PBI 595486 – Plan of Work Table Dependencies
30 Oct 2026 · @Solomon
Summary
Nothing outside Plan of Work (PoW) depends on the 11 PoW tables, so they can be removed in three waves with no foreign key or database code blockers. Verified in DEV and TST on 2026-10-30.
PBI 595486 asks for every dependency of the PoW tables to be documented before they are renamed and then removed from the DEV and TST databases.
Finding
Result
Tables in scope
11 (10 PlanOfWork% tables + SourceOfFundingAllocation)
Foreign keys involving them
34, each with an in-scope table as the child
Foreign keys from tables outside PoW
0
Stored procedures, views, functions, triggers that reference them
0
Other objects found
8 check constraints owned by the tables themselves (removed with them)
DEV compared with TST
Identical
Still to confirm
App code (front end and back end)
Scope and method
The scope is every table named PlanOfWork% plus SourceOfFundingAllocation in the ZEN_ZENITH database. All checks are read-only and were run in both DEV and TST.
Check
Source queried
What it answers
Foreign keys, both directions
sys.foreign_keys
Which tables reference the PoW tables, and which tables the PoW tables reference
Database code
sys.sql_expression_dependencies, text search of sys.sql_modules, triggers on the tables
Which stored procedures, views, functions and triggers use them
Not covered by these checks: dynamic SQL built at runtime, application code (front end and back end), SQL Agent job steps (held for now), and PRD.
Color legend
The same colors are used in every table below, so a table's color tells you its place in the removal order at a glance.
Color
Meaning
Used for
🟩 Green
Wave 1: remove first. Nothing references the table.
PoW tables
🟨 Yellow
Wave 2: remove after wave 1 is gone.
PoW tables
🟧 Orange
Wave 3: remove last. These are the parents the other PoW tables point at.
PoW tables
🟦 Blue
Shared lookup table outside PoW. It stays in the database.
Tables the PoW tables reference
🟥 Red
Blocker: a dependency from outside PoW that would stop removal. None found.
Reserved
Table inventory
The 11 tables fall into three layers. Outgoing FKs are the table's own references to other tables; incoming FKs are other tables pointing at it.
Wave
Table
Outgoing FKs
Incoming FKs
Referenced by
🟩 1
PlanOfWorkBudgetItemYearlyInfoBaselined
1
0
none
🟩 1
PlanOfWorkBudgetItemYearlyInfoWorking
1
0
none
🟩 1
PlanOfWorkLineItemYearlyInfoBaselined
1
0
none
🟩 1
PlanOfWorkLineItemYearlyInfoWorking
1
0
none
🟨 2
PlanOfWorkBudgetItemBaselined
3
1
PlanOfWorkBudgetItemYearlyInfoBaselined
🟨 2
PlanOfWorkBudgetItemWorking
2
1
PlanOfWorkBudgetItemYearlyInfoWorking
🟨 2
PlanOfWorkLineItemBaselined
12
1
PlanOfWorkLineItemYearlyInfoBaselined
🟨 2
PlanOfWorkLineItemWorking
11
1
PlanOfWorkLineItemYearlyInfoWorking
🟧 3
PlanOfWorkBaseline
0
2
PlanOfWorkBudgetItemBaselined, PlanOfWorkLineItemBaselined
🟧 3
PlanOfWorkCategory
0
2
PlanOfWorkLineItemBaselined, PlanOfWorkLineItemWorking
🟧 3
SourceOfFundingAllocation
2
4
PlanOfWorkBudgetItemBaselined, PlanOfWorkBudgetItemWorking, PlanOfWorkLineItemBaselined, PlanOfWorkLineItemWorking
Every table in the Referenced by column is itself one of the 11. No table outside PoW references any of them.
Foreign key relationships
The 34 foreign keys come from nine child tables. Orange, yellow and green entries are PoW tables that go away together; blue entries are shared lookups that stay untouched when the PoW tables are dropped.
Child table
Foreign keys
References inside PoW
References outside PoW (shared lookups)
🟩 PlanOfWorkBudgetItemYearlyInfoBaselined
1
🟨 PlanOfWorkBudgetItemBaselined
none
🟩 PlanOfWorkBudgetItemYearlyInfoWorking
1
🟨 PlanOfWorkBudgetItemWorking
none
🟩 PlanOfWorkLineItemYearlyInfoBaselined
1
🟨 PlanOfWorkLineItemBaselined
none
🟩 PlanOfWorkLineItemYearlyInfoWorking
1
🟨 PlanOfWorkLineItemWorking
none
🟨 PlanOfWorkBudgetItemBaselined
3
🟧 PlanOfWorkBaseline, 🟧 SourceOfFundingAllocation
🟦 WBS
🟨 PlanOfWorkBudgetItemWorking
2
🟧 SourceOfFundingAllocation
🟦 WBS
🟨 PlanOfWorkLineItemBaselined
12
🟧 PlanOfWorkBaseline, 🟧 PlanOfWorkCategory, 🟧 SourceOfFundingAllocation
🟦 Hull, 🟦 NavseaTWPOrg, 🟦 NavyTechCog, 🟦 OrderOfImportance, 🟦 PlantType, 🟦 Priority, 🟦 Vendor, 🟦 WBS, 🟦 WorkStatus
🟨 PlanOfWorkLineItemWorking
11
🟧 PlanOfWorkCategory, 🟧 SourceOfFundingAllocation
🟦 Hull, 🟦 NavseaTWPOrg, 🟦 NavyTechCog, 🟦 OrderOfImportance, 🟦 PlantType, 🟦 Priority, 🟦 Vendor, 🟦 WBS, 🟦 WorkStatus
🟧 SourceOfFundingAllocation
2
none
🟦 CategoryOfWork, 🟦 SourceOfFunds
PlanOfWorkBaseline and PlanOfWorkCategory have no outgoing foreign keys, so they do not appear as child tables. The shared lookups are not part of this removal; Priority, for example, is also used by Commitment.
Removal order
Drop the tables in three waves, child tables before the parents they reference. A wave can only start once the previous one is done.
Wave
Tables
Why this order
🟩 1
PlanOfWorkBudgetItemYearlyInfoBaselined, PlanOfWorkBudgetItemYearlyInfoWorking, PlanOfWorkLineItemYearlyInfoBaselined, PlanOfWorkLineItemYearlyInfoWorking
Nothing points at them, so they can go first.
🟨 2
PlanOfWorkBudgetItemBaselined, PlanOfWorkBudgetItemWorking, PlanOfWorkLineItemBaselined, PlanOfWorkLineItemWorking
Their only incoming reference is from a wave 1 table.
🟧 3
PlanOfWorkBaseline, PlanOfWorkCategory, SourceOfFundingAllocation
They are referenced only by wave 2 tables, so they go last.
This order applies to the drop. The rename test in the PBI does not need a specific order, because renaming a table does not break foreign keys.
Code dependencies
No stored procedure, view, function or trigger in the database references any of the 11 tables. DEV and TST returned the same results.
Object type
Found
Impact on removal
Stored procedures
0
None
Views
0
None
Functions
0
None
Triggers
0
None
Check constraints
8
None. They belong to the tables and are dropped with them.
The 8 check constraints were generated by SQL Server Migration Assistant and guard against zero-length strings. Each belongs to one of the tables below:
Table
Check constraints
🟧 PlanOfWorkBaseline
1
🟧 PlanOfWorkCategory
1
🟨 PlanOfWorkLineItemBaselined
2
🟨 PlanOfWorkLineItemWorking
2
🟧 SourceOfFundingAllocation
2
Open items and next steps
The database side is documented; the remaining work is confirmation from the devs, the rename test, and the PRD plan.
[ ] Devs confirm no front end or back end code still reads these 11 tables.
[ ] Notify the devs before any rename starts, as the PBI requires.
[ ] Rename the tables in DEV, test the application, then repeat in TST.
[ ] Drop the tables in the wave order above once the rename test passes.
[ ] Repeat the foreign key and code checks against PRD with the production DBA.
[ ] Write the PRD delivery plan, to ship with the code change that removes PoW.
[ ] Optional: check whether CategoryOfWork and SourceOfFunds, referenced only by SourceOfFundingAllocation in this scope, are used elsewhere.
[ ] On hold: check SQL Agent job steps for references to these tables.