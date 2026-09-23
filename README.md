/*
=========================================================
Dependency Check Script
Purpose: Identify all dependencies for tables being 
considered for archival (empty, unused PlanOfWork-related 
tables in TST/DEV).

Instructions: Update the @TableList variable below with 
the exact table names to check, then run each section.
=========================================================
*/

-- Set the tables you're evaluating here (comma-separated, no schema prefix needed)
DECLARE @TableList TABLE (TableName SYSNAME);
INSERT INTO @TableList (TableName) VALUES
    ('PlanOfWorkBudgetItemWorking'),
    ('SourceOfFundingAllocation')
    -- add the other 2 tables here
;

-----------------------------------------------------------
-- SECTION 1: Foreign Keys - Tables that REFERENCE these 
-- tables (i.e., what breaks if these are dropped)
-----------------------------------------------------------
SELECT 
    fk.name AS ForeignKeyName,
    OBJECT_NAME(fk.parent_object_id) AS ChildTable,
    OBJECT_NAME(fk.referenced_object_id) AS ReferencedTable,
    'Incoming - would break if referenced table is dropped' AS Note
FROM sys.foreign_keys fk
WHERE OBJECT_NAME(fk.referenced_object_id) IN (SELECT TableName FROM @TableList);

-----------------------------------------------------------
-- SECTION 2: Foreign Keys - Tables that THESE tables 
-- reference (their own outgoing dependencies)
-----------------------------------------------------------
SELECT 
    fk.name AS ForeignKeyName,
    OBJECT_NAME(fk.parent_object_id) AS ChildTable,
    OBJECT_NAME(fk.referenced_object_id) AS ReferencedTable,
    'Outgoing - this table depends on the referenced table' AS Note
FROM sys.foreign_keys fk
WHERE OBJECT_NAME(fk.parent_object_id) IN (SELECT TableName FROM @TableList);

-----------------------------------------------------------
-- SECTION 3: Views, Stored Procedures, Functions, Triggers
-- that reference these tables anywhere in their definition
-----------------------------------------------------------
SELECT DISTINCT
    o.name AS ReferencingObjectName,
    o.type_desc AS ObjectType,
    t.TableName AS ReferencedTable
FROM sys.sql_expression_dependencies d
JOIN sys.objects o ON d.referencing_id = o.object_id
JOIN @TableList t ON d.referenced_entity_name = t.TableName
ORDER BY t.TableName, o.type_desc, o.name;

-----------------------------------------------------------
-- SECTION 4: Indexed Views - additional check in case 
-- Section 3 misses cross-schema or synonym-based references
-----------------------------------------------------------
SELECT 
    v.name AS ViewName,
    t.TableName AS ReferencedTable
FROM sys.views v
JOIN sys.sql_expression_dependencies d ON d.referencing_id = v.object_id
JOIN @TableList t ON d.referenced_entity_name = t.TableName;

-----------------------------------------------------------
-- SECTION 5: Row counts - confirm current empty/data status
-- for each table before making the archival decision
-----------------------------------------------------------
SELECT 
    t.name AS TableName,
    p.rows AS RowCount
FROM sys.tables t
JOIN sys.partitions p ON t.object_id = p.object_id
WHERE t.name IN (SELECT TableName FROM @TableList)
    AND p.index_id IN (0, 1); -- heap or clustered index

-----------------------------------------------------------
-- SECTION 6: Synonyms pointing at these tables (easy to miss)
-----------------------------------------------------------
SELECT 
    s.name AS SynonymName,
    s.base_object_name AS PointsTo
FROM sys.synonyms s
WHERE EXISTS (
    SELECT 1 FROM @TableList t 
    WHERE s.base_object_name LIKE '%' + t.TableName
);
