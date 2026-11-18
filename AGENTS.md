# AGENTS.md — Operating Instructions for AI Coding Agents

**Project:** FYP Supervision Tracker
**Applies to:** any AI coding agent (and any human) generating, changing or reviewing code in this repository.
**Status:** Accepted by the project team, 2026-11-14; updated 2026-11-18. Aligned with the PRD final revision (team decisions TD-01 to TD-13), `ARCHITECTURE.md` **Revision 5** and ADR-001 to ADR-006 (ADR-003 as amended).

> **Purpose of this file.** `ARCHITECTURE.md` explains *what* the architecture is and *why*. This file tells you, the agent, *how to work inside it*. The architecture has already been decided by humans. Your job is to implement it faithfully, not to redesign it. If a rule here seems wrong or blocks you, **stop and ask** (Section 11). Do not work around it.

---

## 1. Read these first, every session

| Order | File | Use it for |
|---|---|---|
| 1 | `PRD_Final_FYP_Supervision_Tracker_Deliverable3.md` | **What** the system must do: FR, NFR, BR, EC, AC, C, D, OQ, CFG, TP and TD IDs, and the glossary. It is the source of every requirement. Section 8 (Configuration defaults) holds every seeded value; Section 19 records the team decisions that closed the open questions. |
| 2 | `ARCHITECTURE.md` | **Where** each thing lives: modules, owned entities, interfaces, transactions, ports, constraints, and still-open architecture items (§9.5). |
| 3 | `ADR/ADR-001` … `ADR/ADR-006` | **Binding decisions** and their rationale (ADR-003 includes Amendment 1; ADR-006 names the technology stack and Django conventions). Never contradict an accepted ADR. |
| 4 | `AGENTS.md` (this file) | **How** to work: rules, forbidden actions, approval gates. |

Before changing any code, find the PRD IDs it implements and the module that owns them (`ARCHITECTURE.md` §5.2–5.3). Cite those IDs in your plan, your commit message and your tests.

---

## 2. Product intent (keep it in mind)

The system gives supervisors an **early-warning view** of FYP Groups that need attention, a **trustworthy supervision history**, and a **lightweight weekly Check-in routine**, so that daily monitoring takes about two minutes. It must also:
- lock student contributions **only** through deliberate, audited events (TA-1);
- keep supervision going when a Representative or supervisor becomes unavailable, **without losing pending work** (TA-2).

**Correctness of locks, roles, permissions, audit and deadlines matters more than features.** When you have to choose between them, choose correctness.

**Use the PRD glossary terms exactly** in code, UI text and tests: Group (not "team"), Action item (not "task"), Individual contribution / Consolidated report (not "report"), Assigned supervisor, Representative, Missed Check-in, Lock, Start review, Blackout date, and so on.

---

## 3. Technologies (decided in ADR-006)

**Architecture mechanisms (ADR-001 to ADR-005):**
- A **responsive web client** and a **REST API**.
- **One server application**: a modular monolith of 11 modules (ADR-001), plus non-module code: shared kernel, platform, API / application boundary, bootstrap (`ARCHITECTURE.md` §5.4).
- **One ACID relational database** (ADR-002).
- **Private binary storage** behind a Storage port (ADR-005).
- **Server-side sessions**: an opaque, secure, HTTP-only, HTTPS-only cookie (ADR-003).
- **Mandatory MFA for supervisors and Administrators**, with the recovery rules of PRD BR-33 (ADR-003 Amendment 1, TD-11).
- A **database-backed job runner** and a **transactional outbox** (ADR-004).
- Ports for the **Identity Provider**, **Email**, **Storage** and the **pre-commit malware-scan hook**.

**Products (ADR-006, accepted):**

| Area | Use exactly this |
|---|---|
| Backend | Python 3.12+, Django 5.2 LTS, Django REST Framework; one Django app per module |
| Database | PostgreSQL 16+ |
| Authentication | Django auth + database sessions; django-allauth (`account`, `mfa`, `headless`) |
| Jobs and outbox | Custom worker `manage.py run_worker` (`SELECT ... FOR UPDATE SKIP LOCKED`); **no Celery, Redis or broker** |
| Ports | Django email backend (console / SMTP); Django storage API (`FileSystemStorage` / django-storages S3-compatible, private); python-magic |
| Frontend | React + TypeScript, Vite, React Router, TanStack Query, Tailwind CSS; API types from drf-spectacular via openapi-typescript |
| Quality and tests | Ruff, import-linter, pytest + pytest-django + factory_boy, Vitest + React Testing Library, Playwright, Locust (TP-01) |
| Delivery | Docker Compose (development), Gunicorn, Git + GitHub, GitHub Actions |

**Rules:**
- Use **only** what ADR-006 names. Any other library, service or infrastructure needs approval (Section 10). Name it in your plan as a draft ADR or ADR amendment, and wait.
- Pin exact versions in the lock files (`pyproject`/`requirements` lock and `package-lock.json`). Do not upgrade a major version without approval.
- Follow the Django implementation conventions in Section 12.1. They exist because Django's usual habits break the module, audit and Clock rules.

---

## 4. Module map and boundaries (binding)

There are exactly **11 modules**. **Do not add, merge, split or rename modules** without human approval and a new ADR.

| Layer | Module | Owns (only this module touches these tables) | Must NEVER |
|---|---|---|---|
| L0 | **Audit** | `AuditEntry` (append-only) | Be bypassed. Be written to directly by feature code. |
| L0 | **Calendar & Scheduling** | `CalendarSettings` (incl. Group deadline settings), `BlackoutDate`, `DeadlineRecord`, `DeadlineHistory`, `MeetingDeadlineSource`, `ScheduledJob` | Depend on any business module (the job runner calls registered handlers through an interface). Import Identity & Access (its user commands are authorized by the application boundary, Section 6). |
| L1 | **Identity & Access** | `UserAccount`, `Credential`, `MfaEnrolment`, `Session`, `Invitation`, `SecuritySettings` | Store permissions in sessions or tokens. Read feature tables. Own Group membership. Deactivate the last active Administrator, or let a user reset their own MFA enrolment (BR-33). **Call Notifications or Files** (deactivation notifications are enqueued by Groups & Roles inside ◆ `DeactivationImpactHandler`). |
| L1 | **Notifications** | `Notification`, `OutboxMessage`, `EmailDeliveryAttempt`, `NotificationPreference`, `DigestPreference`, `DigestEntry`, `NotificationSettings` | Send email inside a business transaction. Assemble Briefing or Handover content itself. **Call Identity & Access** (contact details and Administrators come through ◆ `RecipientDirectory`). Accept a final recipient list from a caller. |
| L1 | **Files** | `Attachment`, `AttachmentVersion`, `UrlReference`, `IntakeSettings` | Expose direct or public storage URLs. Call storage while a database lock is held, or inside a transaction (`stageFile` runs before; `commitFile` touches only the database). Fetch a URL before accepting it. Delete content referenced by a committed version. Call any other L1 module. |
| L2 | **Groups & Roles** | `Group`, `GroupMembership`, `SupervisorAssignment`, `RepresentativeAssignment`, `RepresentativeConfirmation` | Depend on any L3 module (use ◆ `OpenActionItemsChecker` and ◆ `CheckInStatusProvider` instead). Change deadlines, Missed Check-ins, review timers, drafts or approvals (BR-32). Activate a Group without an eligible appointed Representative, or allow `appointRepresentative` after activation (BR-30.11). |
| L3 | **Check-ins** | `CheckInCycle`, `IndividualContribution`, `ContributionCorrection`, `CheckInLock`, `MissingContribution`, `LateContribution`, `ConsolidatedDraft`, `ConsolidatedReport`, `MissedCheckIn`, `CheckInAnnotation` | Unlock. Delete a missing record or Missed Check-in. Lock because a page was viewed. **Open a cycle from a read** (only `openWeeklyCycle`, run by the `cycle-opening` job). Write anything from its ◆ `CheckInStatusProvider` implementation. |
| L3 | **Milestones & Deliverables** | `MilestoneTemplate`, `MilestoneTemplateItem`, `Milestone`, `Deliverable`, `DeliverableVersion`, `DeliverableReview`, `MilestoneExtensionRequest` | Store binaries. Do date arithmetic. Overwrite a version. Send Milestone reminders to anyone but the Group's members, or after a valid Deliverable exists (CFG-05). |
| L3 | **Action Items** | `ActionItem`, `ActionItemTransition`, `ActionItemEvidence`, `ActionItemReview`, `ActionItemExtensionRequest` / `DeadlineChangeRequest`, `ActionItemReopen` | Decide Group status. Let anyone but an Assigned supervisor complete or reopen an item. Move a due date because a Meeting changed (BR-17, TD-06). Reassign, cancel or close items because an account was deactivated (TD-07). |
| L3 | **Meetings** | `Meeting`, `MeetingAttendee`, `MeetingNote`, `MeetingComment` | Let students edit or delete supervisor-authored records. Move Action-item due dates (BR-17, TD-06). Choose the driving Meeting or compute a Check-in deadline (it passes Meeting facts to `Calendar.onMeetingScheduleChanged`; Calendar applies BR-07). |
| L4 | **Dashboard & Briefing** | **No business tables** | Write to any other module's records. Its only command use is asking Notifications to send a notification, from its two jobs. Store status, counters or overdue values. |

Responsibilities you must keep in their single home (C-06):

| Rule area | Single home |
|---|---|
| Lock rules and lock arbitration; opening weekly cycles | Check-ins (`ARCHITECTURE.md` §6.5, §6.8) |
| Role, membership and replacement rules; ≥ 1 supervisor; majority confirmation rules (BR-30.2, TD-10) | Groups & Roles |
| All date, deadline, Blackout, counter, overdue and countdown arithmetic; "now"; Group deadline setting and the driving-Meeting rule (BR-07, TD-09) | Calendar & Scheduling |
| All permission rules (named policies) | Identity & Access guard |
| BR-22 recipient rules; Digest rules, including the BR-23 "already notified" rule (TD-02) | Notifications (callers supply target facts only, Section 6 step 3.5) |
| Group status (BR-10), the inactivity flag and the Dashboard order (FR-07, TD-08) | Dashboard & Briefing |
| File acceptance rules | Files |

If you find yourself writing one of these rules anywhere else, **stop**: call the owning module's interface instead.

**Notification payload rule (BR-22, C-06).** When your module causes a BR-22 event, call `Notifications.enqueue(eventType, context)` with the Group ID, the source record reference and **target facts**: the IDs of the specific people involved that only your module knows, for example `assigneeId`, `previousAssigneeId`, `creatorId`, `attendeeIds`, `missingMemberIds`, `newRepresentativeId`, `previousRepresentativeId`, `newSupervisorId`, `replacedSupervisorId`, `deactivatedUserId` with `actionItemIds`. **Never** build, filter or pass a final recipient list, and never decide urgency. Notifications applies the BR-22 rule, adds role-based recipients through ◆ `RecipientDirectory` (all members, Assigned supervisors, Representative, Administrators), drops ineligible users and removes duplicates.

---

## 5. Dependency and interface rules

1. **Downward dependencies only:** L4 → L3 → L2 → L1 → L0. **No compile-time cycles.**
2. **No calls within a layer**, with one exception: inside L3, **Action Items → Meetings** (`nextMeetingStart`) is allowed. Do not add any other same-layer dependency. In particular, **no L1 module calls another L1 module** (Identity & Access, Notifications and Files are mutually independent), and Calendar & Scheduling and Audit never call each other except through the Unit-of-Work hook.
3. **Upward needs go through ports declared by the lower module.** These **six** are the **only** ports allowed:

   | Port | Declared by | Implemented by |
   |---|---|---|
   | ◆ `MembershipProvider` | Identity & Access | Groups & Roles |
   | ◆ `DeactivationImpactHandler` | Identity & Access | Groups & Roles |
   | ◆ `RecipientDirectory` | Notifications | Groups & Roles |
   | ◆ `NotificationContentProvider` | Notifications | Dashboard & Briefing |
   | ◆ `OpenActionItemsChecker` | Groups & Roles | Action Items |
   | ◆ `CheckInStatusProvider` | Groups & Roles | Check-ins |

   Adding a new port, or a new upward call, needs human approval.
   - *Example:* **do not** import Action Items from Groups & Roles. Call ◆ `OpenActionItemsChecker`.
   - *Example:* **do not** import Check-ins from Groups & Roles to check the deadline or whether the Consolidated report is submitted (BR-30.2–30.3). Call ◆ `CheckInStatusProvider.currentCycleStatus(groupId)` and compare the deadline with Calendar.
   - *Example:* **do not** make Identity & Access query Group tables or call Notifications. Use ◆ `MembershipProvider` or ◆ `DeactivationImpactHandler` (whose Groups & Roles implementation also enqueues the deactivation notifications).
   - Port implementations used inside a transaction (◆ `OpenActionItemsChecker`, ◆ `CheckInStatusProvider`) are **read-only** and **join the caller's Unit of Work**.
4. **Public interface only.** Each module exposes one public interface (queries + commands). Never import another module's internal classes, repositories or schema.
5. **No cross-module data access.** Never read or write another module's tables, and never write SQL joins across module boundaries. Compose data through query interfaces. Store cross-module references as plain IDs.
6. **Batch queries for reads.** Dashboard & Briefing and any list view must use set-based interfaces (for example `overdueItems(groupIds[])`). Never run one query per Group (NFR-01).
7. **External services only through their ports:** Identity Provider, Email, Storage, scan hook, Clock. No module calls a provider SDK directly.
8. **Domain events are only for after commit.** Use them to wake the outbox dispatcher, or for optional, idempotent, non-essential reactions. **Never** depend on an event handler for anything that must happen, and never create notifications from an event handler. Anything that must happen is written in the transaction (in-app rows, outbox row, audit) or as a `ScheduledJob`.
9. **Locks offered across modules.** To take another module's row lock, use its public command: `IdentityAndAccess.lockUserAccount(userId)` and `GroupsAndRoles.lockGroupShared(groupId)`. Both join the caller's Unit of Work. Respect the lock order in Section 7.
10. **Non-module code** (`ARCHITECTURE.md` §5.4): modules may import only the **shared kernel** and the **platform** Unit-of-Work contract. Platform imports only the shared kernel. Only **bootstrap** (the composition root) may see every module, and only to wire them; it binds exactly the six ports above.

---

## 6. The write-path recipe (every state-changing command)

Follow this order for every command. Do not skip any step (`ARCHITECTURE.md` §6.3).

1. **Register a named policy** for the endpoint (for example `checkin.startReview`). **The guard denies by default:** an endpoint with no policy is refused. Never mark an endpoint "public" without approval.
2. The API layer calls `authorize(policy, context)`.
   - **Uploads:** before any transaction, the owning feature module calls `Files.stageFile(...)` (intake checks, scan hook, storage write). It returns a staged-file handle. A rejected file is `ValidationFailed` and nothing is written.
3. **Open one Unit of Work. Who opens it:**
   - **Commands owned by L2–L4 modules, or by Identity & Access:** the **owning module** (`ARCHITECTURE.md` §5.3) opens it.
   - **User-initiated commands owned by an L0 or L1 module other than Identity & Access** — currently `Calendar.addBlackout`, `Calendar.removeBlackout`, `Calendar.updateSettings` (including a Group's deadline setting, policy `calendar.setGroupDeadline`, Assigned supervisor only), `Notifications.setEmailPreference`, `Notifications.setDigestPreference`, `Notifications.markRead`: the **API / application boundary** opens it, runs the inside-transaction `IdentityAndAccess.authorize(...)` re-check (step 3.1) itself, and only then calls the L0/L1 command, which joins the Unit of Work. **Never** make the L0/L1 module import Identity & Access, and **never** add an authorization port to L0.

   **Inside the transaction:**
   1. Re-run the same policy against **current** membership, roles and record state (ADR-003 §4) — in the owning module, or in the application boundary for the L0/L1 commands above. Record state is passed in as policy context.
   2. Get time from `Calendar.now()` and deadlines from Calendar. **Never** read the system clock or do date math yourself.
   3. Apply the change using the required concurrency mechanism (Section 7).
   4. Calls to other modules' commands **join this Unit of Work**. They never open their own. For uploads, call `Files.commitFile(handle, …)` here to write the version record.
   5. Call `Notifications.enqueue(eventType, context)` with **target facts, not recipients** (Section 4, notification payload rule). It writes in-app rows and the outbox row in this transaction.
   6. The Unit-of-Work audit hook records audit entries. **Never write audit rows by hand. Never change data outside the Unit of Work.**
4. Commit (all or nothing). Then publish domain events.
5. Return a typed outcome: `Forbidden`, `Conflict` (naming the existing lock, or stating "record changed; reload"), `ValidationFailed` or `NotFound`. Do not use exceptions for normal refusals.
6. **No external I/O inside the transaction:** no email, no storage writes, no HTTP calls. For uploads, storage is written *first* by `stageFile`, then the version is committed by `commitFile` (ADR-005 §5). Staged content that is never committed is removed by the Files orphan-cleanup job.
7. **Views are pure queries.** Opening, viewing or printing a page must never change state. **A GET request never writes,** not even to materialize a deadline lock or to open a missing Check-in cycle.

---

## 7. Invariants you must preserve (with the mechanism to use)

| Invariant | Required mechanism | Reference |
|---|---|---|
| **Exactly one Check-in lock per cycle** | Unique `CheckInLock(cycleId)`. Conditional insert with "no lock AND `now < effectiveDeadline`" for `SUBMISSION` and `START_REVIEW`. `DEADLINE` lock is effective at `effectiveDeadline` and materialized idempotently by `materializeDeadlineOutcome(cycleId)` (from the job or from any later write on the cycle), with `lockedAt = effectiveDeadline`. | `ARCHITECTURE.md` §6.5; BR-02, BR-28, AC-05g |
| **Exactly one Check-in cycle per Group and period; cycles opened proactively** | `openWeeklyCycle` run only by the `cycle-opening` job (scheduled at Group activation, then by each run for the next period); unique `CheckInCycle(group, period)`; idempotency key `open-cycle:{group}:{period}`. Never opened by a read. | §6.8 |
| **Late report never changes the lock** | If the cycle is already locked or the deadline has passed, still save `ConsolidatedReport` (`isLate = false` if before the deadline under a `START_REVIEW` lock; `isLate = true` if at or after the deadline). One report per cycle. A late report clears the active warning; the `MissedCheckIn` entry stays. | §6.5; BR-03, BR-05, AC-05d, AC-06g |
| **Missed Check-in depends only on whether an on-time report existed at the deadline** | Never on when a job ran | §6.5; BR-05 |
| **Representative-replacement rules use current Check-in state** | Inside the transaction, read ◆ `CheckInStatusProvider.currentCycleStatus(groupId)` and compare with `Calendar.now()`. Member-initiated replacement only before the deadline; after it, only an Assigned supervisor with a reason; if the report is submitted, the replacement applies to later cycles. Deadline state is derived from time, even if the deadline job has not run. | §6.7; BR-30.2–30.3, AC-04e |
| **A Group always has ≥ 1 Assigned supervisor** | Group row lock; add the new supervisor, remove the old one, then assert ≥ 1 before commit | §6.7; BR-31, AC-16b |
| **Multi-Group deactivation is atomic** | One Unit of Work owned by Identity & Access. `lockUserAccount` (exclusive), then every affected `Group` in ascending ID order through ◆ `DeactivationImpactHandler`. If any Group would have zero supervisors, roll back and return the **exact list of blocking Groups**. Otherwise Groups & Roles applies the Group consequences **and enqueues the BR-22 notifications**; Identity & Access deactivates and ends sessions; commit. | §6.6; AC-16b |
| **Lock order** | `UserAccount` rows before `Group` rows; `Group` rows in ascending ID. Replacements take a shared `lockUserAccount` on the candidate before the Group lock. | §6.6 |
| **First save wins between Representative replacement and report submission** | Group row lock (`lockGroupShared` for submission, exclusive for replacement) + role version + draft version | §6.7; BR-30.5, AC-04g |
| **Stale actors are refused** | Re-check the policy inside the transaction; version numbers on roles, drafts and reviewable items | ADR-002, ADR-003; AC-16f |
| **First valid review decision wins** | Version-checked conditional update; a change needs an audited reopen | BR-16, AC-12d |
| **History is never overwritten or hard-deleted** | Insert versions, annotations and linked records; cancel by status change (audited) | `ARCHITECTURE.md` §8.4; BR-25 |
| **The system always has at least one active Administrator** | Deactivating an Administrator locks all active Administrator `UserAccount` rows in ascending ID order and refuses if none would remain | BR-33(f), TD-11; AC-02e |
| **Supervisors and Administrators always use MFA** | Enforced at sign-in by Identity & Access (django-allauth); recovery only through recovery codes, Administrator-assisted reset of another user, or the operator command; all audited | NFR-03, BR-33; AC-02e |
| **An active Group always had an eligible Representative at activation** | `activateGroup` re-checks the appointed Representative's eligibility (shared `lockUserAccount`, then Group lock) inside the transaction; `appointRepresentative` exists only in setup | BR-30.11, TD-12; AC-01f |
| **Majority confirmations never carry over and never apply silently** | One active confirmation per active current member; bound to the current cycle and counted only before its deadline; recounted against current membership; the replacement is applied only when a saved confirmation creates the majority (never by a membership change); confirmations closed by status change when the Representative changes | BR-30.2, TD-10; AC-04h |
| **Deactivation never changes Action items** | Inside ◆ `DeactivationImpactHandler`: read items through ◆ `OpenActionItemsChecker`, enqueue `assigneeDeactivated` with target facts; no reassignment, cancellation or closure; "assignee deactivated" computed on read | BR-21, TD-07; AC-01d |
| **Due dates never follow Meetings** | `nextMeetingStart` only sets the default at creation; later changes only through BR-20 | BR-17, TD-06; AC-11d |
| **Meeting-relative deadlines follow the driving Meeting** | Meetings passes facts to `Calendar.onMeetingScheduleChanged`; Calendar chooses the earliest non-cancelled Meeting in the period (fallback: global weekly), recalculates only unfrozen deadlines with a future result, writes `DeadlineHistory`, reschedules jobs | BR-07, TD-09; AC-06h, AC-06i |
| **Replacement changes who may act, never what is recorded or when it is due** | Groups & Roles never touches deadlines, Missed Check-ins, review timers, drafts or approvals | BR-32 |
| **Derived values are computed on read** | Group status, counters, inactivity flag, overdue state and duration, and review countdowns are **never stored**. Compute them with Calendar. | ADR-002 §6 |
| **Blackouts adjust only unfrozen deadlines** | New `DeadlineHistory` rows (audited); owning modules freeze a deadline at lock or submission | BR-12; `ARCHITECTURE.md` §6.8 |
| **Reminders target roles, resolved when they fire** | Never bake a user ID into a Representative reminder | BR-14, BR-30.9 |
| **Jobs are idempotent and re-read state** | Unique idempotency key per job; the handler re-reads state and Calendar | ADR-004 §3 |
| **BR-22 recipients are decided only by Notifications** | Callers pass target facts; Notifications applies the rule and resolves role-based recipients via ◆ `RecipientDirectory` | `ARCHITECTURE.md` §6.2 rule 7 |
| **Email is optional and isolated** | Outbox + dispatcher; at-least-once delivery with duplicate mitigation (unique outbox key, `EmailDeliveryAttempt` check, idempotency key passed to the Email port where supported). **Never claim exactly-once.** | `ARCHITECTURE.md` §6.4 |
| **Files are private and checked on the server** | Extension allowlist + actual content type + size, from `IntakeSettings`; HTTPS URL format only, no fetch; `stageFile` before the transaction, `commitFile` inside it; downloads through the guard; versioned keys; orphan cleanup never deletes committed content | ADR-005; `ARCHITECTURE.md` §5.2 (Files) |

---

## 8. Configuration, not constants

**Never hard-code** these values. Read them from the owning module's configuration:

| Value | Source (PRD) | Owning module (entity) |
|---|---|---|
| Official time zone (no default; Group activation refused until set), Blackout calendar (empty) | CFG-01, CFG-02 | Calendar & Scheduling (`CalendarSettings`, `BlackoutDate`) |
| Reminder offsets for Check-ins, Milestones and Action items (48 h / 24 h) | CFG-04 to CFG-06 | Calendar & Scheduling (`CalendarSettings`) |
| Global weekly deadline (default Sunday 11:59 PM); Group deadline settings | CFG-03, BR-07 | Calendar & Scheduling (`CalendarSettings`) |
| Inactivity thresholds (default 7) | CFG-07, BR-08 | Calendar & Scheduling (`CalendarSettings`) |
| Review periods (7 / 3 days) | CFG-08, BR-16 | Calendar & Scheduling (`CalendarSettings`) |
| Administrator escalation delay (3 days) | CFG-09, BR-16 | Calendar & Scheduling (`CalendarSettings`) |
| Allowed file types, maximum size (25 MB), staged-file grace period (24 h) | CFG-10 to CFG-12 | Files (`IntakeSettings`) |
| Password, failed-attempt and session rules (NIST SP 800-63B-4; may be made stricter, never weaker) | CFG-15 to CFG-17 | Identity & Access (`SecuritySettings`) |
| Digest defaults (enabled, 08:00, Monday–Friday, user's time zone) | CFG-13 | Notifications (`NotificationSettings`) |
| Email on by default (globally and per user) | CFG-18 | Notifications (`NotificationSettings`, `NotificationPreference`) |
| Email on / off (global) | — | Notifications (`NotificationSettings`); per-user in `NotificationPreference` |
| Milestone template | — | Milestones & Deliverables (`MilestoneTemplate`) |

Defaults from the PRD may be **seeded** as configuration values, never written as literals in business logic. Deployment settings (database connection, adapter selection, worker poll intervals) belong in environment configuration, not in business code.

---

## 9. Resolved decisions, remaining open items and deferred scope

All PRD open questions are **resolved** (PRD §15.3). Decisions TD-01 to TD-13 (PRD §19) were made by the project team under client-delegated authority. They are binding requirements: implement them as written, cite the TD and AC IDs, and **do not** mark them provisional or `TODO(decision)`.

| Item | What you must do |
|---|---|
| **OQ-01** → TD-01 | NFR-01 / NFR-02 are measured under TP-01 / TP-02 (PRD §7.1). Still do not add caches, summary tables or denormalized counters "for performance": indexes and set-based queries only; anything more needs a superseding ADR. |
| **OQ-05** → TD-02 | Implement the BR-23 rule **only inside Notifications**: an already-delivered urgent event appears once in the next Digest as "already notified"; a history-only Digest is skipped (AC-14e). |
| **OQ-06, OQ-07, OQ-08** → TD-03 to TD-05 | Seed CFG-01 to CFG-18 as configuration (Section 8). Never hard-code them. |
| **OQ-09** → TD-06 | Due dates never follow a rescheduled or cancelled Meeting (AC-11d). |
| **OQ-10** → TD-07 | Deactivation is not blocked by Action items and never changes them; Groups & Roles enqueues `assigneeDeactivated` inside ◆ `DeactivationImpactHandler`; Action Items lets any Assigned supervisor reassign such items to an active current member (AC-01d). |
| **OQ-11** → TD-08 | Dashboard order per FR-07, computed on read (AC-07b). |
| **OQ-12 (was G-20)** → TD-09 | Implement the BR-07 driving-Meeting rule in Calendar & Scheduling only (AC-06h, AC-06i). |
| **OQ-13 (was G-19)** → TD-10 | Implement the BR-30.2 majority confirmation rules in Groups & Roles only (AC-04h). |
| **TD-11** (MFA for Administrators) | MFA mandatory for supervisors and Administrators; recovery exactly as BR-33; last-Administrator guard (AC-02e). |
| **TD-12** (first Representative) | `appointRepresentative` only in setup, by an Assigned supervisor; activation refused without an eligible appointed Representative; afterwards BR-30 only (AC-01f). |
| **TD-13** (email default) | Email on by default; user and global switches; urgent in-app always on (AC-14h). |
| **ADR-006** (technology stack) | **Accepted.** Use the stack in Section 3 and the conventions in Section 12.1. |
| **Architecture items** (`ARCHITECTURE.md` §9.5) | **All closed** (by ADR-006 and TD-09 / TD-10). If you find a new gap, treat it under Section 11: choose the most conservative behaviour, mark `TODO(decision)`, and raise it. |
| **D-01 to D-05** (calendar sync, retention/archive, backup/RPO-RTO, malware scanning, capacity) | **Do not build.** Keep the extension points (the scan hook stays a no-op). Do not remove them. |

## 10. Actions that require human approval first

**Stop and ask before you:**

1. Add, replace or upgrade (major version) any language, framework, library, database, provider, broker, cache, workflow engine or other infrastructure not named in ADR-006.
2. Add, remove, merge, split or rename a module; move an entity to a different module; or change table ownership.
3. Add a dependency that breaks the layer rules, any same-layer dependency other than Action Items → Meetings (including any L1 → L1 call), or a seventh port.
4. Read or write another module's tables, or join across modules.
5. Change any confirmed business rule (C-06), for example:
   - lock events or arbitration;
   - permissions or policies;
   - recipient lists;
   - deadline, Blackout or status calculation;
   - replacement eligibility;
   - audit content.
6. Make an endpoint public, skip the guard, relax a policy, skip the inside-transaction re-check (including in the application boundary), or let the client enforce security.
7. Add any hard delete, overwrite of history, unlock, or a way to excuse a Missed Check-in.
8. Store derived values (status, counters, overdue) or add caching across requests.
9. Make any external call inside a database transaction, or hand clients direct storage URLs.
10. Implement a deferred item (D-01 to D-05), decide an `ARCHITECTURE.md` §9.5 item beyond Section 9, or change or reinterpret a team decision (TD-01 to TD-13) or a configuration default (CFG-xx).
11. Edit, supersede or contradict an ADR, `ARCHITECTURE.md`, the PRD or this file, or depart from a Section 12.1 convention. Propose the change instead.
12. Disable, skip or weaken tests, including architecture or dependency tests and access tests.
13. Run destructive operations: dropping or rewriting data or migrations that lose history, force-pushing, or deleting files outside your task.

When you ask, state:
- what you need;
- which rule or ADR blocks it;
- two or more options with trade-offs;
- your recommendation.

For architectural changes, give it as a **draft ADR**.

---

## 11. Handling ambiguity and conflicts

1. **Precedence:**
   - **Requirements:** the PRD wins.
   - **Structure and decisions:** accepted ADRs win, then `ARCHITECTURE.md`, then this file.
   - If a lower document contradicts a higher one, follow the higher one **and report the contradiction**.
2. **If the PRD is silent or ambiguous, do not guess.** Pick the most conservative behaviour (deny, refuse, preserve history, change nothing), mark it `TODO(decision)` with the question, and raise it in your summary.
3. **If a task conflicts with an ADR or with this file,** stop and ask (Section 10). Do not "temporarily" bypass a rule.
4. **If you think the architecture is wrong,** say so with evidence (requirement IDs, a failing scenario), and propose a draft ADR. Keep implementing the current architecture until a human decides.
5. **Never invent requirements, IDs, roles, statuses or glossary terms.** If something is missing, ask.

---

## 12. Project structure conventions (ADR-006)

Python package names use underscores and avoid the standard-library names `platform` and `calendar`.

```
/client/                         Vite + React + TypeScript + Tailwind (no security logic, no date arithmetic)
  /src/api/                      Generated OpenAPI types (openapi-typescript) and the API client
/server/
  /config/                       Django settings (UTC, USE_TZ = True), URLs, WSGI. No business rules.
  /shared_kernel/                Contracts only: typed outcomes, plain ID types, actor context,
                                 Unit-of-Work and domain-event contracts. Imports nothing.
  /platform_core/                Unit of Work over transaction.atomic(), audit-hook runner,
                                 FOR SHARE lock helper, transaction.on_commit event publisher,
                                 worker loop (run_worker) for the job runner and outbox dispatcher.
                                 Imports /shared_kernel/ only — never a module.
  /api/                          DRF views per module, the default-deny guard permission class,
                                 outcome-to-HTTP mapping, application services that open the Unit of Work
                                 for L0/L1-owned commands (Section 6, step 3). Imports modules' public/ only.
  /bootstrap/                    Composition root (a Django app whose ready() wires everything): binds exactly
                                 the six ports, registers policies, job handlers and the audit hook.
                                 The ONLY place that sees every module. No business rules.
  /modules/
    /identity_access/            One Django app per module:
      /public/                     the module's ONLY importable surface: interface, DTOs, events, declared ports
      /internal/                   models.py, services, policies, repositories (not importable by others)
      /adapters/                   port adapters (here: Identity Provider via django-allauth)
      /migrations/                 this module's tables only
    /groups_roles/  /checkins/  /milestones_deliverables/  /action_items/  /meetings/
    /calendar_scheduling/  /notifications/  /audit/  /files/  /dashboard_briefing/
      (/adapters only in modules that declare external ports; port implementations live in the
       implementing module's internal/ and are bound in bootstrap; dashboard_briefing has no models/migrations)
  /tests/
    /support/                    Controllable clock, email-off/failing adapters, in-memory storage, PRD §18 data
    /architecture/               pytest checks: no foreign keys between modules, migration ownership, exactly six ports
    /access/                     NFR-03 scripted access tests (positive + negative per policy)
    /acceptance/                 One test per PRD AC ID, named by ID (e.g. test_AC_05g)
  importlinter config            Layers L4→L0, same-layer independence (except action_items → meetings),
                                 no imports of another module's internal, platform_core/shared_kernel import no module
/.github/workflows/              CI: Ruff, import-linter, pytest, tsc, ESLint, Vitest on every pull request
/docker-compose.yml              PostgreSQL, app, worker
/ (root)                         PRD, ARCHITECTURE.md, AGENTS.md, CLAUDE.md (imports AGENTS.md), ADR/
```

**Conventions:**
- **One top-level folder (one Django app) per module.** Other modules may import only from its `public/` part.
- **Modules import only `shared_kernel` and the `platform_core` Unit-of-Work contract** besides other modules' `public/` parts. `platform_core` never imports a module, and `bootstrap` is never imported by anything.
- **Migrations and schema are owned per module.** A module's migration may only create or alter that module's tables.
- **Port implementations are wired in `bootstrap`.** A lower module never imports the implementing module.
- **Name things with PRD terms and IDs:** policies as `<area>.<action>` (for example `checkin.startReview`, `group.replaceRepresentative`, `group.appointRepresentative`, `account.resetMfa`, `file.download`); tests and commits cite FR, BR, AC or EC IDs.

### 12.1 Django implementation conventions (binding, ADR-006)

| Do | Never |
|---|---|
| Reference another module's record by a plain ID field (for example `group_id = UUIDField()`) | A `ForeignKey`, `OneToOneField` or `ManyToManyField` to another module's model; `select_related` / `prefetch_related` across modules |
| Write only through module commands inside the `platform_core` Unit of Work; register audited changes with it | `ModelSerializer.save()` or `ModelViewSet` writes on business models; Django signals for business behaviour; `QuerySet.update()` or `bulk_create` / `bulk_update` on audited models |
| Take exclusive locks with `select_for_update()`; take shared locks only through the `platform_core` `FOR SHARE` helper | Raw SQL elsewhere; locking another module's rows except through `lockUserAccount` / `lockGroupShared` |
| Use unique constraints + `ON CONFLICT DO NOTHING` or a savepoint for conditional inserts (locks, cycles, idempotency keys) | Check-then-insert without a unique constraint |
| Set every business timestamp from `Calendar.now()` | `auto_now`, `auto_now_add`, `timezone.now()`, `datetime.now()` or `time.time()` outside the Calendar Clock adapter (Ruff enforces this) |
| Keep Django settings in UTC; read the Official academic time zone and all CFG values from the owning module's settings tables | Business configuration in `settings.py` (only deployment settings belong there: database URL, adapters, worker poll interval) |
| Set the guard permission class as the DRF default; declare a named policy on every view | `AllowAny`, `IsAuthenticated` alone, or a view without a policy |
| Run background work through `ScheduledJob` rows executed by `run_worker` | Celery, threads, `on_commit` callbacks or in-memory timers for anything that must happen |
| Use django-allauth for login, MFA, recovery codes, password reset and rate limiting | Homemade cryptography, OTP or password hashing |
| Use the Django admin, if at all, read-only in development | Registering business models for editing in the Django admin |
| Return timestamps as ISO 8601 UTC; let the client format them with `Intl.DateTimeFormat` | Deadline, overdue or countdown calculation in the client |

---

## 13. Testing requirements (definition of done)

A change is **done** only when all of these hold:

- [ ] Every new endpoint has a registered policy, with a **positive and a negative** access test. Cover, as relevant:
  - Former member;
  - another Group's user;
  - student vs. supervisor;
  - Administrator trying to approve or start review;
  - replaced supervisor;
  - members acting after the deadline.
- [ ] L0/L1-owned commands (`addBlackout`, `removeBlackout`, `updateSettings`, `setEmailPreference`, `setDigestPreference`, `markRead`) have a test showing the application boundary re-checks the policy **inside** the transaction (for example, an Administrator deactivated between the first check and the commit is refused).
- [ ] Every audited change has a test asserting its audit entry exists and was written in the same commit. Cover especially locks and replacements.
- [ ] Time-dependent logic is tested with the **injectable Clock** at the boundaries:
  - 11:59:59 PM vs. after the deadline;
  - the window where **the deadline has passed but the job has not run** (including Representative replacement through ◆ `CheckInStatusProvider`);
  - Blackout days.
- [ ] Concurrency rules have race tests:
  - two Start reviews;
  - Start review vs. deadline;
  - late report vs. deadline lock;
  - replacement vs. submission;
  - two review decisions;
  - multi-Group deactivation with one blocking Group (asserts the exact list returned and that nothing changed, **including that no notification rows exist**);
  - two `cycle-opening` runs for the same Group and period (exactly one cycle).
- [ ] Notification tests assert recipients per BR-22 event type are decided by Notifications from the supplied target facts plus ◆ `RecipientDirectory` (AC-14d), and that Former members and deactivated accounts are dropped.
- [ ] Upload tests cover `stageFile` rejection (nothing written), a failed business transaction after `stageFile` (no version record; content later removed by orphan cleanup), and that cleanup never removes committed content.
- [ ] With email disabled or failing, all saves still succeed (NFR-04). Dispatcher tests cover retry and duplicate mitigation.
- [ ] Architecture or dependency tests pass (layering, no same-layer calls, exactly six ports, no cross-module table access).
- [ ] Final-revision rules have tests named by their AC IDs: AC-01d (deactivated assignee, including the refused reassignments), AC-01e (activation refused until the time zone is set), AC-02d (password, failed-attempt and session rules), AC-04h (majority confirmations: withdrawal, expiry at the deadline, membership change), AC-06h / AC-06i (driving Meeting, fallback, frozen deadline, next-cycle setting change), AC-07b (Dashboard order), AC-10f (intake defaults), AC-11d (due date fixed), AC-14e to AC-14g (Digest duplicates, reminders, Digest defaults).
- [ ] TD-11 to TD-13 rules have tests: AC-01f (appointment only in setup, by an Assigned supervisor; activation refused without an eligible Representative), AC-02e (MFA for supervisors and Administrators, single-use recovery codes, Administrator-assisted reset never self, last-Administrator guard, including two Administrators deactivating each other concurrently), AC-14h (email default and switches).
- [ ] CI is green: Ruff (including the banned clock APIs), import-linter contracts, pytest (including `tests/architecture`), `tsc`, ESLint and Vitest.
- [ ] Each implemented acceptance criterion has a test named by its AC ID.
- [ ] No hard-coded configuration values. No date math outside Calendar & Scheduling. No direct clock reads. No GET that writes.

---

## 14. Quick self-check before you submit any change

1. Which PRD IDs does this implement, and which module owns them?
2. Did I touch only my module's tables and other modules' **public** interfaces or one of the six declared ports?
3. Is there a registered policy, re-checked inside the transaction — by the owning module, or by the application boundary for an L0/L1-owned command?
4. Does every write go through the Unit of Work, with audit, plus in-app notifications and the outbox if needed?
5. Did I pass **target facts, not recipient lists**, to `Notifications.enqueue`?
6. Did I use `Calendar.now()` and Calendar for every date rule?
7. Did I keep history append-only and avoid storing derived values?
8. Did I avoid external I/O inside transactions (files staged first, committed inside)?
9. Did I avoid any same-layer call (especially L1 → L1) and any port beyond the six?
10. Did I use only the technologies in ADR-006 and follow the Django conventions in Section 12.1? Did I introduce any technology, module, dependency or rule change that needs approval? If yes, did I stop and ask?
11. Did I implement the resolved decisions (TD-01 to TD-13, CFG-xx) exactly as written, raise any new gap instead of deciding it, and avoid building deferred scope?
12. Are tests in place for the positive case, the negative/forbidden case, the time boundary and the race (where relevant)?
