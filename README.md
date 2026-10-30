-- Every PlanOfWork% table (+ SourceOfFundingAllocation) with FK counts
SELECT
    DB_NAME() AS CurrentDatabase,
    t.name    AS TableName,
    (SELECT COUNT(*) FROM sys.foreign_keys fk
      WHERE fk.parent_object_id = t.object_id)      AS OutgoingFKs,   -- this table references others
    (SELECT COUNT(*) FROM sys.foreign_keys fk
      WHERE fk.referenced_object_id = t.object_id)  AS IncomingFKs    -- other tables reference this one
FROM sys.tables t
WHERE t.name LIKE 'PlanOfWork%'
   OR t.name = 'SourceOfFundingAllocation'
ORDER BY (SELECT COUNT(*) FROM sys.foreign_keys fk
           WHERE fk.parent_object_id = t.object_id
              OR fk.referenced_object_id = t.object_id), t.name;
