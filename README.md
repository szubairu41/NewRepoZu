The work is entirely systems based. Everything I do runs through SSMS, Azure DevOps, Confluence, SMAX and the shared drives. The servers I administer are not at my desk, they are accessed remotely whether I am in the building or not. My physical location changes nothing about my access, tools or output.
2.  My work is ticket driven and measurable. Requests come in as DBRs in Azure DevOps, I build, test and deploy against them, and every step is documented in the ticket, the PR and Confluence. The work product is the same from home, and it is auditable either way.
3.  After hours coverage. Backups, SQL Agent jobs and health checks run on the server's schedule, not office hours. A failed job or a refresh that finishes overnight can be checked and acted on first thing from home, instead of waiting until I am physically in the office. That extends the team's coverage window at no extra cost.
4.  Long running operations. Database refreshes, data imports and restore tests run for extended stretches where my role is monitoring and verifying. That work is done from a screen regardless, and starting it early from home means it is further along when the developers start their day.
5.  All collaboration is already virtual. My meetings are on Teams, and day to day coordination with the developers and the production DBA happens through tickets and chat. No part of my workflow depends on someone walking up to my desk.
6.  Deployment work needs focus. Testing scripts, verifying row counts and data changes, and writing documentation are concentration work where an interruption can mean an error in a deployment. A quieter environment directly reduces that risk.
7.  I am a single point of failure for the team. I am the only dedicated Dev/Test DBA. Anything that keeps me from the office, a flooded commute in Hampton Roads being a live example this week, stalls the whole DBR queue. Working from home removes that disruption risk almost entirely.
8.  It directly solves the spacing problem. My seat can go to someone whose role genuinely requires them to be in the building. I occupy a full desk while doing work that has no physical component, which is the definition of a seat that can be freed.


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
