DEV & TST Environment Retention Policy

Purpose
This page defines expectations for how long objects and data may exist in the DEV and TST environments, when they should be cleaned up, and how they're handled when removed. This policy exists because no prior retention policy covered environment object lifecycle (confirmed with Ian Rogers).

1. Guiding Principle

DEV and TST are working environments, not permanent storage. Objects should exist only as long as they serve active development, testing, or validation purposes. Retention expectations differ between the two environments based on their role in the SDLC (see Section 2).

2. Environment Roles (Retention Context)

	•	DEV — active development space. Developers can make direct changes here (per existing Dev Expectations standards), so object turnover is expected and frequent.
	•	TST — integration/regression testing space, meant to closely mirror PRD. No direct developer changes allowed; only DBA-applied, approved changes exist here. As a result, TST retention issues are rare and typically arise only from accumulated approved changes that are no longer relevant, not from ad hoc developer objects.

3. Object Lifecycle Expectations — DEV

Developers are expected to self-manage what they leave in DEV:

	•	Active work — objects tied to a feature currently in development. Expected to remain until complete or paused.
	•	Paused/parked work — objects for work resuming later. Developers should flag these so they aren't mistaken for abandoned.
	•	Completed work — once promoted to TST (or no longer needed), should be removed from DEV by the developer or flagged to the DBA for removal.
	•	Abandoned/stale objects — no clear owner, purpose, or recent activity. Subject to removal at DBA discretion after a reasonable attempt to identify an owner.

4. Object Lifecycle Expectations — TST

Since only approved, DBA-applied changes exist in TST:

	•	Objects in TST are expected to reflect the current, intended state of the system and should closely track PRD.
	•	An object becomes eligible for removal from TST only when it has been superseded, deprecated, or confirmed no longer relevant via the standard DBR/change request process — not through ad hoc developer request, since developers cannot modify TST directly.
	•	TST is treated as the source of truth during a DEV refresh; TST itself is not subject to the same "stale object" cleanup DEV requires, since its contents are already gated by approval.

5. Standard Retention Handling

	•	The DBA is not expected to track the purpose or lifecycle of every object in DEV on an ongoing basis. Developers are responsible for communicating what they're working on in DEV and cleaning up what they no longer need.
	•	Periodically (see Database Refresh Plan), the DBA will perform a DEV/TST compare to identify DEV-only objects and confirm their status with the team.
	•	Any DEV-only object not confirmed as active or intentionally retained by its owner is treated as eligible for removal, subject to the backup process below.
	•	Removal of anything from TST follows the standard DBR/change request process, not this policy's DEV cleanup process.

