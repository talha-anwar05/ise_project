# ADR-002: Data Persistence, Concurrency Control and Immutable Audit

**Status:** Accepted
**Date:** 2026-11-14
**Deciders:** Project team (all members)
**Related:** ADR-001 (modular monolith), ADR-003 (re-check inside write transactions), ADR-004 (outbox, jobs, Calendar)

## Context

- **ASR-1:** three triggers compete to lock a Check-in: Representative submission, the deadline, and supervisor confirmation of Start review. Two supervisors may confirm at the same moment. A confirmation processed after 11:59 PM must lose to the deadline (AC-05g, cases 1–4). Exactly one lock and one audit entry may result. The lock, the missing-contribution records and the audit entry must all be saved, or none of them (NFR-04).
- **ASR-2:** a Group must always have at least one Assigned supervisor, so adding the new supervisor and removing the old one is a single atomic step. Replacing the Representative can race with Consolidated report submission: the first save wins, with never two reports and never a lost draft. Stale actions must be refused, such as a submission by an old Representative or a decision by a replaced supervisor (AC-04d–g, AC-16a, AC-16b, AC-16f). Replacement must never change deadlines, Missed Check-ins or review timers (BR-32).
- **ASR-4:** original contributions, late contributions, corrections, Deliverable versions, deadline history, Missed Check-ins and error annotations are never overwritten. Ordinary users cannot permanently delete records. Each audit entry records the actor, time, changed fields and previous values, and is committed together with the change (BR-25, NFR-04).
- **ASR-8:** the Dashboard combines status and counters across all of a supervisor's Groups, and must load within 2 s (NFR-01).
- **ASR-5 / C-06:** counters and status depend on Blackouts and thresholds that can change, and must then be recalculated immediately.

We need a storage model and a concurrency strategy that guarantee these rules.

## Alternatives considered

1. **Relational ACID database** with atomic conditional updates or row locks, unique constraints, version numbers, append-only history tables, a shared audit hook and an outbox table.
2. **Document store**, for example one Group document holding its Check-ins, roles and Milestones.
3. **Event sourcing**, where every change is stored as an immutable event and current state is rebuilt from projections.

Sub-options considered within option 1:
- **Lock race:** a row lock (`SELECT … FOR UPDATE`) vs. an atomic conditional update.
- **Audit capture:** database triggers vs. an application-layer Unit-of-Work hook.
- **Dashboard data:** a summary table maintained on every write vs. computing status on read with indexed queries.

## Decision

We use **one relational ACID database**. Each module owns its own tables (ADR-001). The design rules are:

1. **Check-in lock.** Locking is a single **atomic conditional update**. It succeeds only if the Check-in is still unlocked **and** the server's current time is before the effective deadline (taken from Calendar & Scheduling, ADR-004). A **unique constraint** on the lock record (one lock per Check-in) makes a second lock impossible even under races. The losing request receives a refusal that names the existing lock (AC-05g). Lock time is the server's processing time.
2. **Supervisor and Representative replacement.** Replacement runs in one transaction. Where a rule spans several rows, the transaction takes a **row lock** on the Group, for example: add the new supervisor, remove the old one, then check that at least one Assigned supervisor remains. Group roles and drafts carry **version numbers** (optimistic concurrency). A form submitted with an outdated version, or by someone who no longer holds the role, is rejected. Replacement never changes deadline, Missed Check-in or review-timer records (BR-32).
3. **Append-only history.** Records that BR-25 says must be preserved are stored as **versions, annotations or linked records**, never overwritten. Ordinary users cancel records with a status change, never a permanent delete. Late contributions are stored separately and linked to the missing record (BR-29). A "review started in error" note is an annotation; the lock is never reversed (BR-06).
4. **Audit capture.** A **shared application-layer Unit-of-Work hook**, owned by the Audit module, records the actor, timestamp, entity, changed fields, previous values and lock trigger for every audited change. It writes **in the same database transaction** as the change. Feature code does not write audit rows itself. Only the Administrator and the Group's supervisors can read audit history (NFR-03).
5. **Outbox.** Any transaction that must cause a notification or other external effect writes an **outbox row** in the same transaction. Delivery happens after commit (ADR-004).
6. **Dashboard and counters are computed on read.** Group status (BR-10), counters and overdue values are **not stored** as separate fields. Each read calculates them from the stored facts with **indexed queries**, using the Calendar & Scheduling module for every date rule. A change to a Blackout or threshold therefore takes effect immediately, with no recalculation job.
7. Transactions stay **short**. No external call (email, file storage, network) happens while a database lock is held.

## Rationale

- ASR-1 and ASR-2 are **concurrency rules across several records**. Relational transactions, conditional updates and unique constraints let the database itself guarantee "exactly one winner" and "all or nothing", instead of relying on application code. Both are easy to show in tests (AC-05g, NFR-04).
- A **conditional update** is simpler than a row lock for the single-row lock decision and never blocks. Row locks are used only where a rule spans rows (the ≥1-supervisor check).
- **Document stores** have weaker transactions across documents, and append-only history makes documents grow without limit. **Event sourcing** fits the history requirement, but its projections lag behind writes (a risk for "lock effective at the deadline instant") and it adds complexity the team and the AI agent would struggle with. Append-only tables give the needed immutability where BR-25 requires it, and nowhere else.
- An **application-layer audit hook** knows the actor and the business reason directly, which database triggers do not. Making it one shared mechanism stops individual features from forgetting the audit.
- **Computing on read** avoids stale stored values when Blackouts or thresholds change (ASR-5, C-06). There is only one source of truth: facts plus the Calendar module.

## Trade-offs / consequences

**Positive**
- The core rules (one lock, ≥1 supervisor, lock + audit committed together) are enforced by the database and can be tested directly.
- History and audit are complete and consistent. Rule changes to status or counters take effect immediately.
- The approach is well understood by the team and by AI coding agents.

**Negative / risks**
- **Performance risk for ASR-8.** Computing on read across many Groups and Check-ins could exceed 2 s. **Mitigation:** indexes on Group, period, deadline and status columns, and queries that work on sets rather than one query per Group. Measure against NFR-01 once OQ-01 is resolved. If targets are missed, a summary table or cache can be added in a new ADR that supersedes item 6.
- History and audit tables grow continuously. Retention is deferred (D-02).
- The audit hook is critical infrastructure. Any change made outside the Unit-of-Work would escape audit. AGENTS.md must forbid writes that bypass it, and tests must check that an audit entry exists for each lock and replacement.
- Version-number rejections mean users sometimes see "this record changed; reload" messages. This is intended (AC-16b) but the UI must handle it clearly.
- The schema for versions and annotations needs careful upfront design. This ADR does not choose a specific database product (C-03).
