# ParentIDtrg
For Solo
========================================================
README - Deployment Package
DBR: Commitment ParentID Fix (Promoted + Closed Commitments)
========================================================

Prepared by: Solomon Zubairu, Database Administrator 2
Target DBA: Chi Lee (PRD DBA)
Date Prepared: [DATE]
Expected Deployment Window: [TO BE DISCUSSED WITH CHI]

--------------------------------------------------------
1. SUMMARY
--------------------------------------------------------
Issue: Commitments that are promoted have their ParentID
incorrectly set to '0' instead of the correct parent
commitment ID. This affects the Open commitments queue
and has also been identified in Closed commitments.

Currently identified as 22+ affected Open commitments
(Louis Currier working these manually) and an unknown
number of affected Closed commitments (pending review).

Root cause: [TO BE FILLED IN - trigger/stored procedure
responsible for setting ParentID on commitment promotion.
Ben Fulk/Louis Currier investigating.]

--------------------------------------------------------
2. TARGET SERVER / DATABASE
--------------------------------------------------------
RSNV43-PWNSQDB5 - [ZEN_ZENITH_PRD]

--------------------------------------------------------
3. OBJECTS AFFECTED
--------------------------------------------------------
PART A - Trigger/Stored Procedure Fix
  - Object name: [TO BE FILLED IN - trigger or SP name]
  - What is being altered: [TO BE FILLED IN]
  - Script attached: [FILENAME].sql

PART B - Data Correction (Open + Closed Commitments)
  - Table: CommitmentChangeRequest (or relevant commitment table)
  - Column: ParentID
  - Records affected: 22 Open commitments identified so far;
    Closed commitments pending count
  - Script attached: [FILENAME].sql

--------------------------------------------------------
4. SCRIPTS ATTACHED
--------------------------------------------------------
  [ ] Trigger/SP fix script (.sql)
  [ ] Data correction script - Open commitments (.sql)
  [ ] Data correction script - Closed commitments (.sql)
      (only if Ben is unavailable and this is handled
      separately - see note below)

--------------------------------------------------------
5. TESTING REQUIREMENTS
--------------------------------------------------------
Has the script been tested? [YES/NO]
Tested on: [SERVER/ENVIRONMENT]
Testing results: [SUMMARY]

Validation after deployment:
  - Confirm no remaining commitments (Open or Closed)
    have ParentID = '0' where a valid parent exists
  - Confirm newly promoted commitments get correct
    ParentID going forward (trigger/SP fix validation)

--------------------------------------------------------
6. IMPACT ANALYSIS
--------------------------------------------------------
What could go wrong: [TO BE FILLED IN]
Impact/Security: [TO BE FILLED IN]
Impact: Loss of functionality/incorrect parent-child
commitment relationships for affected users until
corrected.

--------------------------------------------------------
7. ROLLBACK INSTRUCTIONS
--------------------------------------------------------
[TO BE FILLED IN - provide SQL or explanation for
rollback in case of issue during change. If backing up
affected rows before update, note that here, e.g.:
"Affected rows backed up to [location] prior to update;
rollback = restore ParentID values from backup table."]

--------------------------------------------------------
8. PERMISSIONS
--------------------------------------------------------
No new permissions required. [UPDATE IF NEEDED]

--------------------------------------------------------
9. EXPECTED TIMELINE
--------------------------------------------------------
[Immediate / During Sprint / Before Release / Other]

--------------------------------------------------------
10. NOTES
--------------------------------------------------------
- Open commitments (22 identified) being handled manually
  by Louis Currier in the interim; this script formalizes
  the fix for consistency/repeatability.
- Closed commitments: Ben Fulk's availability is limited
  over the next two weeks. If Ben is unavailable to
  review/fix the trigger/SP root cause in time, Solomon
  can run a data-only correction on Closed commitments
  (setting ParentID based on confirmed correct values
  from the dev team) as a stopgap - this does NOT replace
  the need for the underlying trigger/SP fix, which
  remains dev-owned.
- Correct ParentID values/logic for the data correction
  must be confirmed by the dev team (Louis/Ben) before
  Solomon runs any update - not something Solomon
  determines independently.
========================================================
