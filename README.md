-- Replace with the actual table name, run one at a time
SELECT 
    fk.name AS ForeignKeyName,
    OBJECT_NAME(fk.parent_object_id) AS ChildTable,
    OBJECT_NAME(fk.referenced_object_id) AS ReferencedTable
FROM sys.foreign_keys fk
WHERE fk.parent_object_id = OBJECT_ID('dbo.PlanOfWorkBudgetItemWorking')
   OR fk.referenced_object_id = OBJECT_ID('dbo.PlanOfWorkBudgetItemWorking');
