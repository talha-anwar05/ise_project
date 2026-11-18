# ADR-001: Architecture Style — Modular Monolith with Domain Modules

**Status:** Accepted (wording synced with `ARCHITECTURE.md` Revision 3 on 2026-11-14; the decision itself is unchanged)
**Date:** 2026-11-14
**Deciders:** Project team (all members)
**Related:** ADR-002, ADR-003, ADR-004, ADR-005 · Source: `PRD_Final_FYP_Supervision_Tracker_Deliverable3.md` · `ARCHITECTURE.md` Section 3 (ASRs)

## Context

The FYP Supervision Tracker must satisfy several requirements that shape the overall structure:

- **ASR-1 (Check-in lock invariant):** a lock has exactly one winner among Representative submission, the deadline and supervisor confirmation (BR-02, BR-28, AC-05g). The lock, the missing-contribution records and the audit entry must be saved all or nothing (NFR-04).
- **ASR-2 (role replacement):** a Group must always have at least one Assigned supervisor, so adding the new supervisor and removing the old one is a single step. Representative replacement can race with Consolidated report submission, and the first save wins (BR-30, BR-31, BR-32).
- **ASR-4 (append-only history and audit):** each audit entry must be committed in the same transaction as the change it records.
- **ASR-5 (time as an input):** deadlines, Blackouts and counters must be calculated the same way in every feature (C-04, C-06).
- **ASR-8 (responsiveness):** Dashboard and Group pages must load within 2 s, on desktop, tablet and mobile browsers (NFR-01, C-01, C-02).
- **C-03:** the PRD does not fix the technology stack.
- **C-06:** confirmed rules may change only through approved, recorded changes. To make that controllable, each rule must live in one place.
- **Team feasibility:** a small student team builds the system within one semester. The architecture must also be clear enough for an AI coding agent to follow without drifting.

We need an overall style and clear module boundaries that satisfy these drivers.

## Alternatives considered

1. **Layered monolith.** One deployable, organised by technical layer (UI → controllers → services → data access).
2. **Modular monolith with domain modules.** One deployable and one database, divided into domain modules with explicit interfaces and table ownership.
3. **Microservices.** Separate services, each with its own database, talking over HTTP or messages.

## Decision

We adopt a **modular monolith**: one deployable server application and one relational database (ADR-002), divided into **11 domain modules**:

| # | Module | Responsibility (owns) |
|---|---|---|
| 1 | **Identity & Access** | User accounts, authentication, MFA, sessions, and the central policy/guard layer (ADR-003). |
| 2 | **Groups & Roles** | Groups, membership, the Representative and Assigned-supervisor roles, replacement (FR-01, FR-16, BR-30, BR-31), and the ≥1-supervisor invariant. |
| 3 | **Check-ins** | Weekly Check-in periods, individual contributions, Consolidated reports, locks and lock triggers, Start review, missing and late contributions, Missed Check-ins (FR-03 to FR-06, BR-02 to BR-06, BR-28, BR-29). |
| 4 | **Milestones & Deliverables** | Milestones and templates, Deliverable versions, review decisions, extension requests (FR-09, FR-10, BR-15, BR-20). |
| 5 | **Action Items** | Action items, their status changes, evidence, review and reopen (FR-11, FR-12, BR-17 to BR-19, BR-27). |
| 6 | **Meetings** | Meetings, notes and comments (FR-13, BR-24). |
| 7 | **Calendar & Scheduling** | The Official time zone, the Blackout calendar, effective deadlines, counters, the clock, and time-triggered jobs (ADR-004). |
| 8 | **Notifications** | In-app notifications, Digests, the email outbox and dispatcher, and the Email port (FR-14, BR-22, BR-23; ADR-004). |
| 9 | **Audit** | The shared audit-capture hook and audit records, plus audit read access (BR-25; ADR-002). |
| 10 | **Files** | File and URL intake checks, the Storage port, file versions, and authorized downloads (BR-26; ADR-005). |
| 11 | **Dashboard & Briefing** | Read-only views that combine data from other modules: the Dashboard, Group status display and the Monday Morning Briefing (FR-07, FR-08, FR-15). |

**Interaction rules**

- Each module exposes a **public interface** (queries and commands). Other modules must call only that interface. They must not read or write another module's tables or internal classes.
- **Synchronous queries and commands** between modules go through those interfaces. For example, Check-ins asks Groups & Roles who the current Representative is, and asks Calendar & Scheduling for the effective deadline.
- Modules are arranged in **layers with downward-only dependencies** and no calls between modules of the same layer (one documented exception: Action Items → Meetings). Where a lower module needs something from a higher one, the lower module declares a **port** and the higher module implements it. The layers and the complete, closed list of ports are defined in `ARCHITECTURE.md` §5.1; adding a port needs human approval.
- **Post-commit side effects** use **in-process domain events** published after the transaction commits (for example `CheckInLocked`, `SupervisorReplaced`). Their main use is to wake the Notifications outbox dispatcher; notification records themselves are written **inside** the originating transaction by calling the Notifications interface, never by an event handler (ADR-004 §4). An event handler must never be needed for the originating transaction to succeed.
- Every state-changing command is authorized through **Identity & Access** (re-checked inside its transaction) and audited through **Audit**. Neither may be bypassed. For commands owned by a module below Identity & Access's layer (for example Calendar & Scheduling settings), the API / application boundary performs the in-transaction re-check, so that dependencies stay downward-only (`ARCHITECTURE.md` §6.2).
- **Dashboard & Briefing** is read-only. It calls the query interfaces of other modules and never changes their data.
- Each table is **owned by exactly one module**.

**Client/server split:** a **responsive web client** talks to the server through a **REST API**. Specific frameworks and languages are not chosen in this ADR (C-03). Choosing them later is a separate decision that needs human approval.

## Rationale

- **Atomicity without distributed transactions.** Everything runs in one process against one database, so the ASR-1 lock, the ASR-2 replacement and the ASR-4 audit entry can each be committed in one ACID transaction. Microservices would need sagas or two-phase commit, and those cannot easily guarantee "≥1 supervisor at all times" or "lock + audit in one commit".
- **Explicit boundaries for C-06 and the AI agent.** A layered monolith tends to spread each business rule across several services. Domain modules give every rule one home: lock rules in Check-ins, replacement rules in Groups & Roles, date logic in Calendar & Scheduling, permissions in Identity & Access. That makes rule changes controllable (C-06) and gives AGENTS.md concrete boundaries to enforce.
- **Shared capabilities appear once.** Calendar, Audit, Notifications and Identity & Access are separate modules, so every feature reuses them instead of reimplementing them (ASR-5, ASR-4, ASR-6, ASR-3).
- **Right size.** One institution's FYP cohort does not need independent scaling, and no ASR requires it. One deployable keeps building, testing and deploying achievable for the team.
- **Events after commit** keep side effects such as email out of the critical path, which supports ASR-6 and NFR-04.

## Trade-offs / consequences

**Positive**
- Strong consistency for every cross-entity invariant. Simple deployment, debugging and testing.
- Clear ownership. Each requirement can be traced to one module, which helps the demo and AI-agent guidance.
- A module can later be extracted into a service if ever needed, because its interface already exists.

**Negative / risks**
- The network does not enforce boundaries. They can erode if code reaches into another module's tables. **Mitigation:** dependency rules in AGENTS.md, an agreed folder structure with one top-level package per module, and architecture or dependency tests in CI.
- The whole application scales and fails as one unit. This is acceptable at this scale. NFR-01 must still be measured once OQ-01 is resolved.
- Table ownership has to be fixed early. Shared concepts (for example, Group membership needed by Check-ins, approvals and authorization) are reached only through Groups & Roles' interface, which adds some indirection.
- In-process events are lost if the process crashes before a handler runs. Side effects that must survive a crash, such as notifications, therefore go through the database outbox (ADR-004), not only through in-memory events.
- Choosing frameworks is deferred. The web client and REST API must still be designed to meet C-01, C-02 and NFR-01.
