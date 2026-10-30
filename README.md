/*
=========================================================
PBI 595486 - Document Plan of Work Tables for Removal
Dependency Log (read-only; nothing here changes the database)

Run against DEV first, then TST, and save results from each.
Scope = every table named PlanOfWork% (pattern match, so no typo
risk), plus any extra tables you add to @Extra below.

Result sets, in order:
  1. Scope + row counts
  2. Foreign keys (both directions, classified)
  3. Code dependencies (procs/views/functions/triggers)
  4. Triggers that live ON the PoW tables
  5. Synonyms
  6. SQL Agent job steps (needs msdb access, fails gracefully)
  7. Suggested removal order (FK-safe)
=========================================================
*/
SET NOCOUNT ON;

DECLARE @Pattern SYSNAME = N'PlanOfWork%';

-- Candidate PoW-only lookup tables that don't match the name pattern.
-- Adding them here tells you whether anything OUTSIDE PoW still uses them.
IF OBJECT_ID('tempdb..#Extra') IS NOT NULL DROP TABLE #Extra;
CREATE TABLE #Extra (TableName SYSNAME);
INSERT #Extra (TableName) VALUES
    (N'SourceOfFundingAllocation')
    -- , (N'AnotherCandidate')
;

IF OBJECT_ID('tempdb..#PoW') IS NOT NULL DROP TABLE #PoW;

SELECT t.object_id,
       SCHEMA_NAME(t.schema_id) AS SchemaName,
       t.name                   AS TableName
INTO #PoW
FROM sys.tables t
WHERE t.name LIKE @Pattern
   OR t.name IN (SELECT TableName FROM #Extra);

-----------------------------------------------------------
-- 1. Scope + row counts (confirm the list matches Object Explorer)
-----------------------------------------------------------
SELECT p.SchemaName, p.TableName,
       SUM(ps.row_count) AS [RowCount]
FROM #PoW p
JOIN sys.dm_db_partition_stats ps
  ON ps.object_id = p.object_id AND ps.index_id IN (0, 1)
GROUP BY p.SchemaName, p.TableName
ORDER BY p.TableName;

-----------------------------------------------------------
-- 2. Foreign keys, both directions, classified
--    "INCOMING from outside PoW" = the rows that can block removal
--    "Outgoing to other table"   = shared lookups; these STAY
--                                  (e.g. Priority is also used by Commitment)
-----------------------------------------------------------
SELECT
    CASE
        WHEN c.object_id IS NOT NULL AND r.object_id IS NOT NULL THEN '1 - PoW to PoW (internal)'
        WHEN c.object_id IS NOT NULL                              THEN '2 - Outgoing to other table (likely shared, stays)'
        ELSE                                                           '3 - INCOMING from outside PoW (REVIEW - blocks removal)'
    END AS Relationship,
    fk.name AS ForeignKeyName,
    OBJECT_SCHEMA_NAME(fk.parent_object_id) + '.' + OBJECT_NAME(fk.parent_object_id)         AS ChildTable,
    (SELECT STRING_AGG(COL_NAME(fkc.parent_object_id, fkc.parent_column_id), ', ')
       FROM sys.foreign_key_columns fkc WHERE fkc.constraint_object_id = fk.object_id)        AS ChildColumns,
    OBJECT_SCHEMA_NAME(fk.referenced_object_id) + '.' + OBJECT_NAME(fk.referenced_object_id) AS ReferencedTable,
    (SELECT STRING_AGG(COL_NAME(fkc.referenced_object_id, fkc.referenced_column_id), ', ')
       FROM sys.foreign_key_columns fkc WHERE fkc.constraint_object_id = fk.object_id)        AS ReferencedColumns
FROM sys.foreign_keys fk
LEFT JOIN #PoW c ON c.object_id = fk.parent_object_id
LEFT JOIN #PoW r ON r.object_id = fk.referenced_object_id
WHERE c.object_id IS NOT NULL OR r.object_id IS NOT NULL
ORDER BY Relationship, ChildTable, ReferencedTable;

-----------------------------------------------------------
-- 3. Code dependencies: procs, views, functions, triggers
--    Two methods, because dependency metadata can be stale:
--      Metadata   = SQL Server's own dependency tracking
--      TextSearch = literal table name found in the object's code
--    (TextSearch can over-match on similar names; verify by eye)
-----------------------------------------------------------
SELECT 'Metadata' AS Source,
       OBJECT_SCHEMA_NAME(d.referencing_id) + '.' + OBJECT_NAME(d.referencing_id) AS ReferencingObject,
       o.type_desc AS ObjectType,
       p.TableName AS PoWTable
FROM sys.sql_expression_dependencies d
JOIN sys.objects o ON o.object_id = d.referencing_id
JOIN #PoW p        ON p.TableName = d.referenced_entity_name
WHERE o.type IN ('P', 'V', 'FN', 'IF', 'TF', 'TR')
UNION
SELECT 'TextSearch',
       OBJECT_SCHEMA_NAME(m.object_id) + '.' + OBJECT_NAME(m.object_id),
       o.type_desc,
       p.TableName
FROM sys.sql_modules m
JOIN sys.objects o ON o.object_id = m.object_id
JOIN #PoW p        ON m.definition LIKE N'%' + p.TableName + N'%'
WHERE o.is_ms_shipped = 0
ORDER BY ReferencingObject, PoWTable, Source;

-----------------------------------------------------------
-- 4. Triggers that live on the PoW tables
-----------------------------------------------------------
SELECT tr.name AS TriggerName,
       OBJECT_SCHEMA_NAME(tr.parent_id) + '.' + OBJECT_NAME(tr.parent_id) AS OnTable
FROM sys.triggers tr
JOIN #PoW p ON p.object_id = tr.parent_id;

-----------------------------------------------------------
-- 5. Synonyms
-----------------------------------------------------------
SELECT s.name AS SynonymName, s.base_object_name
FROM sys.synonyms s
WHERE EXISTS (SELECT 1 FROM #PoW p WHERE s.base_object_name LIKE N'%' + p.TableName);

-----------------------------------------------------------
-- 6. SQL Agent job steps that mention PoW tables
-----------------------------------------------------------
BEGIN TRY
    SELECT j.name AS JobName, s.step_id, s.step_name, s.database_name
    FROM msdb.dbo.sysjobsteps s
    JOIN msdb.dbo.sysjobs j ON j.job_id = s.job_id
    WHERE s.command LIKE N'%PlanOfWork%';
END TRY
BEGIN CATCH
    SELECT N'Could not read msdb job steps: ' + ERROR_MESSAGE() AS Note;
END CATCH;

-----------------------------------------------------------
-- 7. Suggested removal order among the PoW tables
--    Wave 1 = nothing in PoW references it, so it goes first.
--    If this errors on recursion, there is an FK cycle - tell me.
-----------------------------------------------------------
;WITH Edges AS (
    SELECT DISTINCT fk.parent_object_id AS ChildId, fk.referenced_object_id AS ParentId
    FROM sys.foreign_keys fk
    JOIN #PoW c ON c.object_id = fk.parent_object_id
    JOIN #PoW r ON r.object_id = fk.referenced_object_id
    WHERE fk.parent_object_id <> fk.referenced_object_id
),
Chain AS (
    SELECT object_id, 0 AS Depth FROM #PoW
    UNION ALL
    SELECT e.ParentId, ch.Depth + 1
    FROM Chain ch
    JOIN Edges e ON e.ChildId = ch.object_id
)
SELECT MAX(ch.Depth) + 1 AS RemovalWave,
       p.SchemaName + '.' + p.TableName AS TableName
FROM Chain ch
JOIN #PoW p ON p.object_id = ch.object_id
GROUP BY p.SchemaName, p.TableName
ORDER BY RemovalWave, TableName
OPTION (MAXRECURSION 100);

DROP TABLE #PoW;
DROP TABLE #Extra;
