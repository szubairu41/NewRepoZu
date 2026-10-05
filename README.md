/*
=========================================================
Discovery Script: JobCategoryID -> PriorityID Rename
Table: dbo.Commitment
Purpose: Find every object that references JobCategoryID
before renaming, since sp_rename does NOT update code 
that references the column by name.
=========================================================
*/

-----------------------------------------------------------
-- SECTION 1: Confirm the column exists and get its details
-----------------------------------------------------------
SELECT 
    c.name AS ColumnName,
    t.name AS DataType,
    c.max_length,
    c.is_nullable,
    c.default_object_id
FROM sys.columns c
JOIN sys.types t ON c.user_type_id = t.user_type_id
WHERE c.object_id = OBJECT_ID('dbo.Commitment')
    AND c.name = 'JobCategoryID';

-----------------------------------------------------------
-- SECTION 2: Any indexes referencing this column
-- (sp_rename on the column does NOT rename these automatically
--  if they have the old name baked into the index name itself)
-----------------------------------------------------------
SELECT 
    i.name AS IndexName,
    i.type_desc AS IndexType,
    c.name AS ColumnName
FROM sys.indexes i
JOIN sys.index_columns ic ON i.object_id = ic.object_id AND i.index_id = ic.index_id
JOIN sys.columns c ON ic.object_id = c.object_id AND ic.column_id = c.column_id
WHERE i.object_id = OBJECT_ID('dbo.Commitment')
    AND c.name = 'JobCategoryID';

-----------------------------------------------------------
-- SECTION 3: Default or check constraints on this column
-----------------------------------------------------------
SELECT 
    dc.name AS DefaultConstraintName,
    c.name AS ColumnName,
    dc.definition
FROM sys.default_constraints dc
JOIN sys.columns c ON dc.parent_object_id = c.object_id AND dc.parent_column_id = c.column_id
WHERE dc.parent_object_id = OBJECT_ID('dbo.Commitment')
    AND c.name = 'JobCategoryID';

SELECT 
    cc.name AS CheckConstraintName,
    cc.definition
FROM sys.check_constraints cc
WHERE cc.parent_object_id = OBJECT_ID('dbo.Commitment')
    AND cc.definition LIKE '%JobCategoryID%';

-----------------------------------------------------------
-- SECTION 4: Foreign keys involving this column (incoming 
-- or outgoing) - unlikely given the name, but worth checking
-----------------------------------------------------------
SELECT 
    fk.name AS ForeignKeyName,
    OBJECT_NAME(fk.parent_object_id) AS ParentTable,
    COL_NAME(fkc.parent_object_id, fkc.parent_column_id) AS ParentColumn,
    OBJECT_NAME(fk.referenced_object_id) AS ReferencedTable,
    COL_NAME(fkc.referenced_object_id, fkc.referenced_column_id) AS ReferencedColumn
FROM sys.foreign_keys fk
JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id
WHERE (fkc.parent_object_id = OBJECT_ID('dbo.Commitment') 
       AND COL_NAME(fkc.parent_object_id, fkc.parent_column_id) = 'JobCategoryID')
   OR (fkc.referenced_object_id = OBJECT_ID('dbo.Commitment') 
       AND COL_NAME(fkc.referenced_object_id, fkc.referenced_column_id) = 'JobCategoryID');

-----------------------------------------------------------
-- SECTION 5: THE IMPORTANT ONE - Views, Stored Procedures, 
-- Functions, Triggers that reference "JobCategoryID" anywhere
-- in their actual code (text search across all object definitions)
-----------------------------------------------------------
SELECT 
    o.name AS ObjectName,
    o.type_desc AS ObjectType,
    m.definition
FROM sys.sql_modules m
JOIN sys.objects o ON m.object_id = o.object_id
WHERE m.definition LIKE '%JobCategoryID%'
ORDER BY o.type_desc, o.name;

-----------------------------------------------------------
-- SECTION 6: Column-level dependency tracking (catches 
-- references that Section 5's text search might structure 
-- differently, e.g. via dynamic SQL is NOT caught here - 
-- manual review of app/dev code still needed for that)
-----------------------------------------------------------
SELECT 
    referencing_schema_name,
    referencing_entity_name,
    referencing_id,
    COALESCE(referenced_entity_name, '') AS ReferencedEntity
FROM sys.dm_sql_referencing_entities('dbo.Commitment', 'OBJECT')
    CROSS APPLY (SELECT referencing_id = OBJECT_ID(referencing_schema_name + '.' + referencing_entity_name)) x;

-----------------------------------------------------------
-- SECTION 7: Synonyms (unlikely but cheap to check)
-----------------------------------------------------------
SELECT name, base_object_name 
FROM sys.synonyms 
WHERE base_object_name LIKE '%Commitment%';
