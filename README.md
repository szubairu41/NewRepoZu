-- Code dependencies on PlanOfWork% tables + SourceOfFundingAllocation
-- Read-only. Finds procs, views, functions and triggers that reference them.
;WITH Scope AS (
    SELECT object_id, name
    FROM sys.tables
    WHERE name LIKE 'PlanOfWork%'
       OR name = 'SourceOfFundingAllocation'
),
Hits AS (
    -- 1) SQL Server's own dependency tracking
    SELECT d.referencing_id AS object_id, s.name AS ReferencedTable, 'Dependency metadata' AS DetectedBy
    FROM sys.sql_expression_dependencies d
    JOIN Scope s ON d.referenced_id = s.object_id
    UNION
    -- 2) Table name found in the object's code (catches what metadata misses)
    SELECT m.object_id, s.name, 'Text search'
    FROM sys.sql_modules m
    JOIN Scope s ON m.definition LIKE '%' + s.name + '%'
    UNION
    -- 3) Triggers that live on the tables themselves
    SELECT tr.object_id, s.name, 'Trigger on table'
    FROM sys.triggers tr
    JOIN Scope s ON tr.parent_id = s.object_id
)
SELECT
    OBJECT_SCHEMA_NAME(h.object_id) + '.' + OBJECT_NAME(h.object_id) AS ReferencingObject,
    o.type_desc                                                       AS ObjectType,
    h.ReferencedTable,
    STRING_AGG(h.DetectedBy, ', ') WITHIN GROUP (ORDER BY h.DetectedBy) AS DetectedBy
FROM Hits h
JOIN sys.objects o ON o.object_id = h.object_id
WHERE o.is_ms_shipped = 0
  AND h.object_id NOT IN (SELECT object_id FROM Scope)   -- ignore the PoW tables themselves
GROUP BY h.object_id, o.type_desc, h.ReferencedTable
ORDER BY ReferencingObject, h.ReferencedTable;
