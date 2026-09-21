Step 4 — Perform the refresh

DEV is wiped and re-synced with TST (1:1 copy of tables and data).
Permissions previously existing in TST/DEV are retained per acceptance criteria in PBI 583205.

Step 5 — Do NOT auto-restore DEV-only objects

Per DEV Retention Policy: DEV-only objects are backed up but not automatically restored after the refresh, since the goal is DEV/TST alignment.
Devs who need to resume DEV-only work after the refresh must submit a new DBR to reintroduce the specific object(s).

Step 6 — Validation

DBA confirms structural integrity of DEV post-refresh.
Devs confirm any objects they expected to persist (i.e., objects that existed in both TST and DEV) came through correctly.

Step 7 — Communication close-out

Team notified that DEV refresh is complete and environment is available.
3. Current Refresh — Findings (as of [DATE])
Schema compare completed between DEV and TST.
6 tables identified as DEV-only (1 containing data, archived to CSV).
3 schemas identified as DEV-only.
3 users identified as owners of DEV-only schemas.
Full backups (CREATE scripts + data export) completed and stored per Step 3.
Refresh execution currently paused while documentation is finalized. Backups are current as of the compare date; a fresh compare should be run immediately before execution if significant time has passed, since DEV may continue to change in the interim.
4. Rollback Considerations

If the refresh causes unexpected issues in DEV, the CREATE scripts and data exports stored in the shared drive backup folder can be used to reconstruct the pre-refresh DEV-only objects. This does not roll back the TST→DEV copy itself — only restores what existed in DEV before the refresh.
