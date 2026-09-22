Subject: DEV Environment Refresh Tonight at 8 PM

Hey Team,

Just a heads up that I'll be performing the DEV environment refresh (TST -> DEV) tonight starting at 8 PM, after hours to avoid interfering with any active work.

What this means:
- DEV will be wiped and re-synced with TST (1:1 copy of tables and data).
- All DEV-only objects identified through our earlier compare and email thread have already been backed up (schema + data) and are stored safely on my end.
- Per the DEV retention policy, DEV-only objects will NOT be automatically restored after the refresh, since the goal is to bring DEV back in line with TST. If you need any of your DEV-only work reintroduced afterward, please submit a DBR and I'll get it back in.

Expected timeline: refresh should be complete well before the start of business tomorrow. I'll send a follow-up once it's done and DEV is confirmed available.

If anything urgent comes up or you have concerns before I start, let me know as soon as possible.

Thanks,
Solomon

Solomon Zubairu
Database Administrator 2
Newport News Shipbuilding - E50
(757) 688-2648 (o) | (478) 738-1339 (m)
