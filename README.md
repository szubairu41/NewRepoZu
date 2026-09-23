SELECT 
    fk.name AS ForeignKeyName,
    OBJECT_NAME(fk.parent_object_id) AS ChildTable,
    OBJECT_NAME(fk.referenced_object_id) AS ParentTable
FROM sys.foreign_keys fk
WHERE fk.referenced_object_id = OBJECT_ID('dbo.Commitment')


TRUNCATE TABLE [dbo].[GrandchildTable];  -- if any exist
TRUNCATE TABLE [dbo].[ChildTable1];
TRUNCATE TABLE [dbo].[ChildTable2];
TRUNCATE TABLE [dbo].[Commitment];
