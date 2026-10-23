 “Good morning, and thank you for meeting with us. A little background may help explain why I submitted this request.

Project Zenith was originally developed by an external vendor, and the internal team took ownership in November 2025. The documentation from that period is incomplete. In the team’s Pilot 0 lessons learned, it states that the DEV database was created by a former NNS employee who left no transition documentation, and I have not been able to locate a vendor Installation and Administration Guide.

During Pilot 0, the team initially expected TST to be in the same location as DEV and believed they would have administrative rights. Instead, TST was separately secured, the initial AD group permissions were configured incorrectly, and the launch was ultimately unblocked when advanced permissions were provided outside the normal process. Those permissions then allowed the team to perform hotfixes after launch. The record also shows that AD groups were previously created through SMAX and that DBR Form 3912 was used to request Database Administrator access to TST.

CCB is different. It is an internally developed prototype that was recently moved to SQL Server, so there is no vendor or vendor administration guide for CCBDB.

I am now the first dedicated Dev/Test DBA supporting these systems. The dashboards and health checks I have built are read-only, but my current access does not return all the DMV and definition information needed for diagnosis, and I can only see SQL Agent jobs that I created. I am requesting VIEW SERVER STATE, VIEW DATABASE STATE, and VIEW ANY DEFINITION on the four Dev/Test servers, granted through a limited AD group rather than to me individually. I am not requesting production access.

My goal is a documented and coordinated access model that avoids repeating the Pilot 0 situation, where work was either blocked by insufficient access or exposed to risk through permissions that were too broad.”

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
