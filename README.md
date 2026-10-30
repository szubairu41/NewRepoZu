-- PlanOfWork% tables (+ SourceOfFundingAllocation) with NO foreign key dependencies
-- Read-only. Remove the SourceOfFundingAllocation line if you only want PlanOfWork tables.
SELECT
    t.name   AS TableName,
    'None'   AS ForeignKeyName,
    'None'   AS ChildTable,
    'None'   AS ReferencedTable_Parent
FROM sys.tables t
WHERE (t.name LIKE 'PlanOfWork%'
       OR t.name = 'SourceOfFundingAllocation')   -- remove this line if not needed
  AND NOT EXISTS (
        SELECT 1
        FROM sys.foreign_keys fk
        WHERE fk.parent_object_id = t.object_id
           OR fk.referenced_object_id = t.object_id
      )
ORDER BY t.name;
