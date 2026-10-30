-- FK record for PlanOfWork% tables + SourceOfFundingAllocation
-- Read-only. Includes any FK where either side is one of those tables.
SELECT
    fk.name                                  AS ForeignKeyName,
    OBJECT_NAME(fk.parent_object_id)         AS ChildTable,
    OBJECT_NAME(fk.referenced_object_id)     AS ReferencedTable_Parent
FROM sys.foreign_keys fk
WHERE OBJECT_NAME(fk.parent_object_id)     LIKE 'PlanOfWork%'
   OR OBJECT_NAME(fk.referenced_object_id) LIKE 'PlanOfWork%'
   OR OBJECT_NAME(fk.parent_object_id)     = 'SourceOfFundingAllocation'
   OR OBJECT_NAME(fk.referenced_object_id) = 'SourceOfFundingAllocation'
ORDER BY ChildTable, ReferencedTable_Parent, ForeignKeyName;
