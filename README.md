Subject: DEV Environment Refresh Complete

Hey Team,

The DEV environment refresh (TST -> DEV) is complete. DEV is now available.

Summary:
- DEV was wiped and re-synced with TST (1:1 copy of tables, data, and constraints).
- Permissions from TST/DEV have been retained and verified.
- All DEV-only objects identified beforehand were backed up (schema + data) prior to the refresh. Per our retention policy, these are not automatically restored - if you need any of your DEV-only work reintroduced, please submit a DBR and I'll get it back in.

One data note for visibility: during validation, I found an existing data inconsistency in TST itself - PlanOfWorkBudgetItemWorking has rows referencing SourceOfFundingAllocation IDs that don't exist in that table (which is currently empty in TST). This isn't something introduced by the refresh; DEV now accurately mirrors this same state since the goal was a 1:1 copy. I've left that specific constraint disabled in DEV to reflect TST's actual data rather than force a fix outside my scope. Wanted to flag this to the team in case it's worth addressing at the source.

Please reach out if anything looks off or if you run into issues in DEV following the refresh.

Thanks,

