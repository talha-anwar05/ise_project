# FYP Supervision Tracker — Product Requirements Document (Final — Deliverable 3)

**Status:** Final PRD for Deliverable 3. It revises the Deliverable 2 "Final Validated" PRD to incorporate the two changes required by the TA: **TA-1** an explicit review action before student contributions are locked, and **TA-2** replacement of unavailable Representatives and supervisors. Both changes are applied consistently across the Goals, Roles, Glossary, Functional Requirements, Non-Functional Requirements, Business Rules, Main Flows, Edge Cases, Acceptance Criteria, Open Questions and Traceability, and each is demonstrated by a normal case and a failure/boundary case (Section 18).
**Client confirmation:** All decisions introduced by TA-1 and TA-2 (CC-01 to CC-12) were confirmed with the client. They are recorded in Section 17 (Client Confirmation Record).
**Final revision (2026-11-18): open questions closed.** Every open question left after Deliverable 3 (OQ-01, OQ-05 to OQ-11), and two requirement gaps found in the architecture review (now OQ-12 and OQ-13), is resolved. **Decision authority:** the client has delegated requirements decision-making authority to the project team. Decisions TD-01 to TD-13 (TD-11 to TD-13 from the final architecture review) were finalized by the project team under that delegated authority and are recorded in Section 19 (Team Decision Record). They are kept separate from the client-confirmed decisions CC-01 to CC-12 (Section 17); no individual client confirmation is claimed for them.
**Source decisions:** Requirements grilling log, Q1–Q265 (one ChatGPT chat, used jointly by the group), consolidated in Requirements Assessment V3 (21 Oct 2026); the Deliverable 2 critique decisions in `PRD_Critique_and_Validation_Log.md`; and the TA's email for Deliverable 3. **Availability note:** the original grilling log is no longer available to the team. `[Qnnn]` and `[early §x]` citations are kept as historical references to decisions as they were recorded at the time; they can no longer be re-checked against the log. **This PRD, including the Team Decision Record (Section 19), is the authoritative requirements baseline.**
**Reading note:** Rules cite their source decision as `[Qnnn]` (grilling question number), `[early §x]` (early confirmed-decisions summary, sections 1A–1H), `[TA-1]` / `[TA-2]` (TA email, change 1 / change 2), `[CC-nn]` (client confirmation item in Section 17) or `[TD-nn]` (team decision under client-delegated authority, Section 19). Items decided in the grilling but moved out of the core scope are kept in Section 13 (Deferred Decisions), not deleted.

---

## 0. Change Summary (Deliverable 2 → Deliverable 3)

| Artifact | TA-1: Explicit review before lock | TA-2: Replacement of unavailable Representatives / supervisors |
|---|---|---|
| Goals, Roles, Glossary | G-2 revised; Assigned supervisor role; glossary terms *Lock*, *Start review*, *Late contribution* | New G-8; Administrator, Assigned supervisor, Student and Representative roles revised; glossary terms *Unavailable*, *Replacement*, *Vacancy*, *Handover summary* |
| Functional Requirements | FR-03 and FR-05 revised | FR-01, FR-04, FR-06, FR-15 revised; **new FR-16** |
| Non-Functional Requirements | NFR-03 and NFR-04 extended | NFR-03 and NFR-04 extended |
| Business Rules | BR-02 rewritten; BR-03, BR-04, BR-06, BR-25 revised; **new BR-28, BR-29** | BR-05, BR-16, BR-21, BR-22, BR-25 revised; **new BR-30, BR-31, BR-32** |
| Main Flows | F2 revised; **new F8** (+ alternate paths 10.1) | **New F9, F10** (+ alternate paths 10.1) |
| Edge Cases | EC-03 revised; **new EC-21 to EC-23** | **New EC-24 to EC-33** |
| Acceptance Criteria | AC-05a revised; **new AC-05d to AC-05i** | **New AC-04c to AC-04g, AC-06g, AC-16a to AC-16f**; AC-14d and AC-15a revised |
| Open Questions | OQ-02 resolved | OQ-03 and OQ-04 resolved; OQ-10 narrowed |
| New sections | Section 17 (Client Confirmation Record); Section 18 (Demonstration Scenarios) | same |
| Housekeeping | Row AC-15d, which sat inside the Edge Cases table, moved to the Acceptance Criteria table where it belongs | — |

### 0.1 Final revision: closure of open questions and final architecture review (TD-01 to TD-13)

| Artifact | Change |
|---|---|
| Header, Section 19 | Decision authority stated; new Team Decision Record (TD-01 to TD-13), separate from the client-confirmed record in Section 17; grilling-log availability note; PRD declared the requirements baseline |
| Final architecture review (TD-11 to TD-13) | MFA mandatory for Administrators as well as supervisors, with recovery (NFR-03, **new BR-33**, CFG-15, CFG-17); first Representative appointed by an Assigned supervisor before activation (FR-01, F1, BR-30.11); email on by default (BR-22, CFG-18); **new EC-37, EC-38, AC-01f, AC-02e, AC-14h**; AC-01c, AC-02d revised |
| Roles, Glossary | Assigned supervisor sets the Group's deadline setting and reassigns items of a deactivated assignee; terms *Check-in deadline* (revised), *Driving Meeting*, *Configuration default*, *Team decision* |
| Functional Requirements | FR-07 (Dashboard order), FR-14 (Milestone and Action-item reminders) revised |
| Non-Functional Requirements | NFR-01 and NFR-02 verified under the approved test profiles (new Section 7.1); NFR-03 adds password and session rules |
| Business Rules | BR-07 (deadline setting, driving Meeting), BR-14, BR-17 (due date fixed), BR-21 (account deactivation), BR-22 (new event), BR-23 (Digest duplicates), BR-26, BR-30.2 (majority confirmation rules) revised; **new Configuration Defaults CFG-01 to CFG-18** (CFG-18 from TD-13) |
| Edge Cases | **New EC-34 to EC-36** |
| Acceptance Criteria | **New AC-01d, AC-01e, AC-02d, AC-04h, AC-06h, AC-06i, AC-07b, AC-10f, AC-11d, AC-14e to AC-14g**; AC-14d revised |
| Open Questions | OQ-01, OQ-05 to OQ-11 resolved; OQ-12 and OQ-13 added and resolved; no open questions remain |

---

## 1. Product Overview

The FYP Supervision Tracker is a responsive web application that helps supervisors oversee Final Year Project (FYP) Groups. It shows which Groups need attention, standardizes the weekly Check-in, keeps Meeting history and Action items in one place, and produces a Monday Morning Briefing.

No programming language, framework, database, hosting environment, or identity/email provider is selected by this PRD.

## 2. Problem and User Need

**Problem.** A supervisor often learns that a Group has stalled only after a deadline has been missed. Updates arrive in different places and formats. A shared spreadsheet was abandoned after about three weeks. Incomplete supervision notes lead to repeated or contradictory feedback.

**User need (supervisor).** To see early which Groups are inactive or late, to recall previous feedback and open work before each Meeting, and to do this with very little daily effort.

**User need (students).** One predictable way to report Blockers, Progress, and Next Steps, and clear visibility of their assigned work and deadlines.

**User-level goals.**
- UG-1: Know within seconds which Groups need attention.
- UG-2: Receive consistent weekly reports instead of scattered updates.
- UG-3: Prepare for a Meeting using prior feedback and open Action items.
- UG-4: Follow up on work through accountable, dated Action items.

## 3. Product Intent, Goals and Non-Goals

**Product intent.** Give supervisors an early-warning overview and a dependable supervision history, using a lightweight weekly reporting routine, so that routine monitoring stays within about two minutes per day.

### Goals
| ID | Goal | Serves |
|---|---|---|
| G-1 | Show Group status and two independent inactivity counters on one Dashboard. | UG-1 |
| G-2 | Standardize weekly Check-ins (individual contributions plus one consolidated report), with contributions locked only by a defined, audited event and never by merely viewing a page. | UG-2 |
| G-3 | Track Milestones, Deliverables, review decisions, and extensions. | UG-1, UG-4 |
| G-4 | Give every Action item one accountable assignee, due date, evidence, and review. | UG-4 |
| G-5 | Keep Meeting notes, feedback, and linked Action items as history. | UG-3 |
| G-6 | Produce a Monday Morning Briefing of inactive Groups and overdue work. | UG-1 |
| G-7 | Enforce role- and Group-level access and keep an audit history. | all |
| G-8 | Keep supervision continuous when a Representative or supervisor becomes unavailable: replace them through an explicit, audited action without losing pending approvals, Missed Check-in history, or existing deadlines. | UG-1, UG-2, UG-4 |

### Non-Goals (not in scope)
- AI-generated summaries, automatic GitHub tracking, WhatsApp bots, email-reply parsing, QR-code Check-ins, gamification.
- Any specific technology stack or provider.
- Automatic detection of unavailability from inactivity. Unavailability is declared by an authorized user through a replacement action (Assumption 6).
- Items listed in Section 13 (Deferred Decisions): external calendar synchronization, archive/reopen and retention automation, backup/recovery targets, malware scanning, and integration retry/escalation policies. Their decisions are recorded but they are not part of the core build.

## 4. Users and Roles

| Role | Responsibilities and access |
|---|---|
| **Administrator** | Creates accounts and invitations; configures the Official academic time zone, global weekly deadline, Blackout dates, standard Milestone template, default review periods, and default reminder settings; may override supervisors' Group membership decisions [Q136]; may replace an unavailable Assigned supervisor of any Group, including a Group's only supervisor (BR-31); receives the escalation of reviews left undecided (BR-31); resets another user's MFA enrolment when they have lost access (BR-33) [TD-11]. Has no other Group-level supervision rights. |
| **Assigned supervisor** | Manages membership and co-supervisor assignment for own Groups [Q136]; activates a Group; schedules Meetings; edits Meeting notes; approves Deliverables, Action items, and extensions; **starts the Check-in review that locks contributions early (BR-28)**; replaces a Representative at any time (BR-30); replaces another unavailable Assigned supervisor of the same Group (BR-31); views the Group's audit history; sets the Group's Check-in deadline setting (Group override or meeting-relative, BR-07) [TD-09]; reassigns open Action items whose assignee's account is deactivated (BR-21) [TD-07]; appoints the Group's first Representative before activation (BR-30.11) [TD-12]. Multiple supervisors may share a Group [Q1]. |
| **Student / Group member** | Has an individual account; submits Check-in contributions; creates and updates Action items; submits Deliverables and extension requests; views Meeting history and adds comments; can view all Action items of own Group [Q180]; **may add a late contribution after a lock (BR-29); together with the other members may change the Representative before the Check-in deadline (BR-30).** Cannot start a Check-in review. |
| **Representative** | A Group member, first appointed by an Assigned supervisor before the Group is activated (BR-30.11) [TD-12] and afterwards replaceable by the Group before the deadline, or by a supervisor at any time, who edits and submits the consolidated report. Not a separate account type. The role can become vacant (BR-30). |
| **Former member** | A student removed from a Group; has no access to that Group's records. |

## 5. Glossary (agreed terms)

Use these terms exactly. Do not use "task" for Action item, "team" for Group, or "report" without saying Individual contribution or Consolidated report.

| Term | Agreed meaning |
|---|---|
| **Group** | An FYP group with one or more Assigned supervisors and Student / Group members. |
| **Assigned supervisor** | A supervisor authorized to manage a particular Group. |
| **Representative** | The Group member who edits and submits the Consolidated report. |
| **Former member** | A student removed from a Group, with no access to its records. |
| **Check-in** | The weekly reporting cycle: Individual contributions plus one Consolidated report per Group. |
| **Individual contribution** | One member's responses to Blockers, Progress, and Next Steps; "nothing to report" must be stated explicitly. |
| **Consolidated report** | The Group-level weekly report assembled from contributions, edited and submitted by the Representative. |
| **Missed Check-in** | A history entry recorded when no Consolidated report exists at the Check-in deadline. Late submission clears the active warning but not the entry. |
| **Check-in deadline** | The time by which the Consolidated report is due (global weekly, Group override, or meeting-relative: 24 hours before the start of the driving Meeting; BR-07). |
| **Driving Meeting** | For a Group in meeting-relative mode, the earliest non-cancelled Meeting that starts within the Check-in cycle's period; its start time minus 24 hours is the Check-in deadline (BR-07) [TD-09]. |
| **Lock** | The permanent point after which Individual contributions can no longer be edited. It happens only on one of three lock events (BR-02): Representative submission, the Check-in deadline, or an Assigned supervisor's confirmed Start review. Viewing a page is never a lock event. |
| **Start review** | The explicit, confirmed action by which an Assigned supervisor locks the current Check-in's contributions before the deadline (BR-28). |
| **Late contribution** | An Individual contribution submitted after the lock; stored separately from, and linked to, the missing record it answers (BR-29). |
| **Inactivity counter** | One of two independent counters: calendar days since a valid Check-in, and since a valid Deliverable. |
| **Inactivity flag** | Active when either counter is greater than its threshold. |
| **Group status** | One of On Track, At Risk, Delayed (rules in BR-10). |
| **Milestone** | A named project stage with a deadline and a linked Deliverable. Complete only after supervisor approval. |
| **Deliverable** | A submission with a title, a file or URL, and a linked Milestone. Valid submission is separate from approval. |
| **Action item** | A self-contained piece of work with a description, one assignee, creation time, due date, status, evidence, and completion timestamps. |
| **Action-item status** | Open, Blocked, In Review, or Completed. |
| **In Review** | Status after the assignee submits completion evidence; stops the overdue clock. |
| **Overdue** | A calculated condition (not an Action-item status) that applies once an item's deadline timestamp has passed and it is not Completed or In Review (milestones: not approved). |
| **Meeting** | A scheduled supervision meeting with attendees, notes, feedback, and linked Action items. |
| **Unavailable** | A Representative or supervisor who cannot act in the role (removed, account deactivated, absent, or not responding). The system does not detect this automatically; it is declared by the authorized user who performs the Replacement (BR-30, BR-31). |
| **Replacement** | The explicit, audited action that moves a role from an unavailable person to an eligible one (BR-30 for Representative, BR-31 for supervisor). |
| **Vacancy** | A Group's Representative role with no available holder (BR-30). A Group is never left without an Assigned supervisor (BR-31). |
| **Handover summary** | A notification to a newly assigned supervisor listing the Group's pending approvals, overdue items, Missed Check-ins, upcoming Meetings and deadlines (BR-31). |
| **Blackout date** | A date, defined by the Administrator in the Official academic time zone, on which deadlines that fall on it move to the next non-Blackout day and on which inactivity, overdue-duration, and review countdowns pause. Applies to all Groups equally [Q177, Q236]. |
| **Official academic time zone** | The Administrator-set time zone used for all deadline calculation; users see deadlines in their local time. |
| **Urgent notification** | An immediate notification for events listed in FR-14. |
| **Digest** | A user-scheduled bundle of notifications (user picks time, time zone, delivery days). Empty Digests are skipped. |
| **Monday Morning Briefing** | The weekly summary described in FR-15. |
| **Audit history** | The record of important changes: actor, timestamp, changed fields, previous values. |
| **Configuration default** | A value seeded at installation that an authorized user may change without code changes (Section 8, CFG-01 to CFG-18). |
| **Team decision (TD)** | A requirements decision finalized by the project team under the decision-making authority delegated by the client (Section 19). |

## 6. Functional Requirements (16 core)

**FR-01 — Accounts, Group setup, and membership.** The system shall allow the Administrator to create accounts and send invitations (self-registration only if enabled), allow Assigned supervisors to add and remove students and co-supervisors for their Groups (Administrator may override), and allow a supervisor to activate a Group once setup and Milestone assignment are complete and an eligible Representative has been appointed (BR-30.11) [TD-12]. A Group shall always keep at least one Assigned supervisor; the last Assigned supervisor can be removed or deactivated only together with a named replacement (BR-31).

**FR-02 — Access control and audit history.** The system shall restrict every record to users permitted by role and Group, preserve a Group's records when a student leaves, and record important changes in an Audit history viewable only by the Administrator and that Group's Assigned supervisors.

**FR-03 — Individual Check-in contributions.** The system shall allow every Group member to submit Blockers, Progress, and Next Steps for the weekly Check-in, requiring an explicit response for each field, and to edit their own contribution until it locks (BR-02).

**FR-04 — Consolidated Check-in.** The system shall assemble member contributions into a draft that the Representative can edit and submit as one Consolidated report. The Group may change its Representative before the deadline; an Assigned supervisor may replace the Representative at any time, including after the deadline; the draft and its history pass to the new Representative, and a Missed Check-in or the deadline is never changed by a replacement (BR-30).

**FR-05 — Check-in history, lock, and corrections.** The system shall lock contributions only on a defined lock event — Representative submission, the Check-in deadline, or an Assigned supervisor's explicit, confirmed Start review action (opening or viewing a page never locks) — audit every lock, record missing contributions, keep the originals unchanged, and let students add correction comments and late contributions and supervisors append corrected versions or error annotations (BR-02, BR-28, BR-29).

**FR-06 — Check-in deadlines, reminders, and missed Check-ins.** The system shall apply the configured Check-in deadline (global weekly, Group override, or 24 hours before a Meeting), send reminders, and record a Missed Check-in when no Consolidated report exists at the deadline. Replacing a Representative or a supervisor shall not move an existing deadline or remove a Missed Check-in (BR-32).

**FR-07 — Dashboard.** The system shall show each Group's status, its days-since-Check-in and days-since-Deliverable counters, and mark as requiring attention every Group that is Delayed, At Risk, or has an active Inactivity flag. Groups requiring attention are listed first (Delayed, then At Risk; within each status by the larger of the two inactivity counters, highest first, then by Group name A–Z), followed by the other Groups by Group name [TD-08].

**FR-08 — Inactivity tracking and Group status.** The system shall maintain two independent inactivity counters with configurable thresholds, reset only the counter matching a valid submission, and calculate Group status as On Track, At Risk, or Delayed from overdue Milestones and Action items, Missed Check-ins, and the Inactivity flag (BR-10).

**FR-09 — Milestones and Milestone extensions.** The system shall provide a standard Milestone template that supervisors can customize per Group, and allow students to request a Milestone extension that only an Assigned supervisor may approve.

**FR-10 — Deliverable submission and review.** The system shall accept a Deliverable (title, file or URL, linked Milestone), and allow the Assigned supervisor to approve it or reject it for revision with a mandatory reason and required revisions, preserving earlier versions.

**FR-11 — Action item creation and assignment.** The system shall allow supervisors and students to create Action items with a single assignee, apply default due dates, allow reassignment within the Group, and restrict editing as defined in BR-17.

**FR-12 — Action item progress, review, and extension.** The system shall let assignees update status (Open, Blocked, In Review), submit completion evidence, let supervisors approve or reject it, and handle extension requests, and allow an Assigned supervisor to reopen a Completed Action item.

**FR-13 — Meetings, notes, and comments.** The system shall allow Assigned supervisors to create, reschedule, and cancel Meetings and edit notes and feedback, and allow students to view the complete Meeting history and add comments.

**FR-14 — Notifications, reminders, and Digests.** The system shall send in-app and email notifications, send Urgent notifications immediately for the events listed in BR-22, support user-configured Digests, and send reminders before Milestone and Action-item deadlines (BR-14, CFG-05, CFG-06) [TD-03].

**FR-15 — Monday Morning Briefing.** The system shall produce a Monday Morning Briefing listing inactive Groups; overdue Action items with assignees and due dates; Missed Check-ins; missing Individual contributions; upcoming Milestones and deadlines; unresolved Blocked items; Action items and Deliverables awaiting review including overdue reviews; At Risk or Delayed Groups; and Groups whose Representative role is vacant.

**FR-16 — Replacement of unavailable Representatives and supervisors.** The system shall let authorized users replace an unavailable Representative or Assigned supervisor through an explicit, audited action; keep pending approvals, Missed Check-ins, and existing deadlines unchanged and visible to the replacement; never leave a Group without an Assigned supervisor; and notify the affected users (BR-30, BR-31, BR-32).

## 7. Non-Functional Requirements (4)

| ID | Category | Requirement (verifiable) | How verified |
|---|---|---|---|
| **NFR-01** | Performance | Under the agreed test load, at least 95% of Dashboard and Group-detail page loads complete within 2 seconds, and common save and status-update operations complete within 3 seconds. | Timed test run under test profile TP-01 (Section 7.1) [TD-01]. |
| **NFR-02** | Usability / efficiency | In the agreed test scenario, a supervisor identifies all Groups requiring attention within 30 seconds, retrieves a selected Group's previous feedback and open Action items within 30 seconds, and completes routine daily monitoring within 2 minutes. | Scripted scenario and pass/fail rule TP-02 (Section 7.1) [TD-01]; same test data for every run; time measured from the start of the task to its completion. |
| **NFR-03** | Security and privacy | Only authenticated users may access records. In an access test, 100% of attempts to read another Group's records, or for a student to read Audit history, are denied. **Also 100% of attempts to start a Check-in review by anyone who is not an Assigned supervisor of that Group, and to replace a Representative or supervisor by anyone not authorized in BR-30 / BR-31, are denied.** Supervisor and Administrator login requires multi-factor authentication, with the recovery rules of BR-33 [TD-11]. Password, failed-attempt and session rules follow CFG-15 to CFG-17 (aligned with NIST SP 800-63B-4) [TD-05]. | Scripted role/Group access test; AC-02d for password and session rules; AC-02e for MFA and recovery. |
| **NFR-04** | Reliability | Every Check-in, Deliverable, Meeting record, and Action-item decision created during acceptance testing is saved and retrievable (100%). **Every lock (including Start review) and every replacement is saved completely or not at all, with its audit entry, and is retrievable (100%).** With email delivery disabled or failing, records still save. | Acceptance test run with email disabled. |

### 7.1 Approved acceptance-test profiles (resolves OQ-01) [TD-01]

These profiles are the test conditions required by C-05. They were set by the project team under client-delegated authority. They are acceptance-test conditions, not a capacity commitment (capacity remains deferred, D-05). The team may revise them through a recorded TD amendment if institution figures become available before acceptance testing.

| ID | Item | Agreed condition |
|---|---|---|
| TP-01 | NFR-01 dataset | 40 supervisor accounts; 150 active Groups of 4 students (600 students); each supervisor an Assigned supervisor of up to 10 Groups; 16 weeks of history per Group (16 Check-in cycles, about 5 Action items and 1 Meeting per Group per week, the 7 standard Milestones with Deliverables). The Section 18 data and the AC-07 Groups are included. |
| TP-01 | NFR-01 load | 50 concurrent signed-in users (40 supervisors, 10 students) running a scripted mix of Dashboard loads, Group-detail loads and common saves (Individual contribution save, Consolidated report submission, Action-item status update, Meeting note save). |
| TP-01 | NFR-01 environment and measurement | A staging deployment with the production configuration chosen under the technology-stack decision (ADR-006), email enabled to a test sink; measured from a supported browser (C-02). At least 500 measured requests per measured operation. Pass: the NFR-01 targets are met as written. |
| TP-02 | NFR-02 scenario | One supervisor with 10 Groups from TP-01, including the 4 AC-07 Groups; three timed tasks: (1) identify all Groups requiring attention, (2) retrieve a named Group's previous feedback and open Action items, (3) complete the routine daily monitoring script. At least 3 testers who did not build the screens, each after a 5-minute orientation, using the same data. Pass: every timed run meets its limit (30 s, 30 s, 2 min). |

## 8. Business Rules

### Check-in and inactivity
- **BR-01 Explicit responses.** Blank fields are not allowed; "None" or a "nothing to report" toggle is required. [early §1B, Q4]
- **BR-02 Lock events (revised).** Contributions lock at the earliest of three lock events: (a) the Representative submits the Consolidated report; (b) the Check-in deadline passes; (c) an Assigned supervisor confirms **Start review** (BR-28). **Opening, viewing, or printing the Check-in details page is not a lock event and never locks contributions.** Every lock is recorded in Audit history with its trigger, time, actor (Representative for (a), system for (b), the supervisor for (c)), and the affected contributions. A lock does not create a Missed Check-in; that depends only on whether a Consolidated report exists at the deadline (BR-05). Contributions not yet submitted at a lock are recorded as missing (BR-04). [Q4, Q164, TA-1]
- **BR-03 After lock.** A Representative may still submit using saved contributions, including late; locked contributions cannot be edited. A member whose contribution was missing at the lock may add a late contribution (BR-29). Late or missing contributions are recorded separately; the original omission, timestamps, and report are preserved. [Q146, Q209, Q216, Q258, TA-1]
- **BR-04 Missing contributions.** Recorded at the individual deadline or at any lock (including a supervisor's Start review), and the student is notified; remaining reminders for them are cancelled. They do not change Group status when the Consolidated report is on time. [Q162, Q187, Q237, Q246, TA-1]
- **BR-05 Missed Check-in.** If no Consolidated report exists at the deadline, record a Missed Check-in and notify all Group members. A late report clears the active warning but keeps the history. Replacing the Representative or a supervisor never removes, postpones, or excuses a Missed Check-in (BR-30, BR-32). [Q4, TA-2]
- **BR-06 Corrections (revised).** A correction of a locked contribution preserves the original, the student's correction comment, the supervisor's corrected version, the reason, author, timestamp, and the link between versions; original timeliness is unaffected. Locked contributions are never unlocked, including after a Start review made in error (BR-29). [Q139, Q233, Q244, Q263, TA-1]
- **BR-07 Deadline settings.** Default global deadline: Sunday 11:59 PM in the Official academic time zone, changeable by the Administrator. Changes to global deadlines or Group overrides apply to future cycles only. A meeting-relative deadline recalculates only if the Check-in is not submitted or locked; if the result is in the past, the previous deadline is kept, including when a Meeting is scheduled or rescheduled less than 24 hours ahead. [Q4, Q165, Q166, Q235]
  - **Deadline setting and driving Meeting (clarification) [TD-09].** (a) A Group uses the global weekly deadline unless an Assigned supervisor of the Group chooses a Group setting: a different weekly day and time (Group override) or meeting-relative. A change of setting applies from the next cycle. (b) In meeting-relative mode the cycle's deadline is 24 hours before the start of the **driving Meeting**: the earliest non-cancelled Meeting of the Group that starts within the cycle's period (the week that ends at the global weekly deadline). If the period has no such Meeting, the global weekly deadline applies. (c) When a Meeting is scheduled, rescheduled, or cancelled, the driving Meeting is determined again and the deadline recalculated, subject to the rule above: only while the Check-in is neither submitted nor locked, and a result that is not in the future is not applied (the previous deadline is kept). (d) Every recalculation is kept in the deadline history with its previous and new values, and Blackout adjustment (BR-12) applies to the result. (e) In weekly mode (global or Group override) Meetings never change the Check-in deadline.
- **BR-08 Counters.** Two independent counters count crossed calendar-date boundaries in the Official academic time zone; the inactivity flag is active when a count is greater than its threshold (default 7 days; Administrator default, supervisor Group override). The Check-in counter starts at the first scheduled deadline; the Deliverable counter starts at Group activation. A valid submission resets only its own counter and the flag is recalculated immediately. [Q2, Q217, Q218, Q242]
- **BR-09 Valid Deliverable.** A Deliverable is valid when title, file or URL, and Milestone are present, even if the URL is inaccessible (warn the student; supervisor assesses). [Q178, Q228]
- **BR-10 Group status.** Delayed if any non-cancelled Milestone or Action item is overdue; otherwise At Risk if a Check-in was missed (active warning) or the inactivity flag is active; otherwise On Track. Delayed outranks At Risk. Overdue reviews, Blocked items, and missing contributions do not change status by themselves. A replacement of a Representative or supervisor does not change status by itself (BR-32). [early §1C, Q259, Q171, Q182, Q162]

### Check-in review and lock (TA-1)
- **BR-28 Start review (explicit action).** [TA-1, CC-01 to CC-04]
  1. **Who:** only an Assigned supervisor of that Group may start review. Students, the Representative, Former members, the Administrator, and supervisors of other Groups cannot.
  2. **No accidental lock:** opening, viewing, scrolling, or printing the details page changes nothing.
  3. **When available:** Start review is offered only while the current Check-in's contributions are unlocked. Selecting it opens a **confirmation step** showing the Check-in period, how many members have and have not submitted (naming those who have not), that the lock is permanent (BR-06), that members who have not submitted will be recorded as missing (BR-04), and that the Representative can still submit the Consolidated report (BR-03). The lock happens only when the supervisor confirms. Cancelling, closing the dialog, or letting the session expire leaves everything unchanged.
  4. **On confirmation** the system, in one all-or-nothing step: locks all contributions; records missing contributions; writes an audit entry (supervisor, confirmation time, trigger "Start review", contributions locked, contributions recorded missing); and notifies Group members that contributions are locked, and each member recorded missing (BR-04). If saving fails, nothing is locked, the supervisor is told, and they may retry.
  5. **Lock time:** the lock time is the server time at which the confirmation is processed. A contribution saved before that time counts as submitted; one saved afterwards is a late contribution (BR-29).
  6. **Already locked / simultaneous requests:** if a lock already exists (Representative submission, deadline, or another supervisor's Start review), the action is refused with a message naming the existing trigger and time (and supervisor, if any). If two supervisors confirm at the same moment, the first processed wins and no second audit entry is written.
  7. **No side effects on deadlines or status:** Start review does not change the Check-in deadline, does not create a Missed Check-in, and does not change Group status. There is no separate "finish review" step.
- **BR-29 Corrections after a lock.** [TA-1, CC-04, CC-05]
  1. *Error in a locked contribution:* handled by BR-06 (student correction comment, supervisor-appended corrected version, original kept, versions linked).
  2. *Missing because of an early lock:* a member recorded missing may add a **late contribution** at any time. It is stored separately and linked to the missing record; the missing record, its timestamp, and the note "recorded at supervisor-started review" remain. It does not change the Group's on-time status. The Representative and supervisors can see it, and the Representative may use its content while the Consolidated report is still unsubmitted.
  3. *Review started by mistake:* the lock is never reversed. Any Assigned supervisor of the Group may append an audited **"review started in error"** annotation with a reason; it appears beside every missing record it affected, and the affected members are notified. The annotation does not unlock contributions, delete a missing record, or change timeliness or status.

### Deadlines and Blackout dates
- **BR-11 Overdue timing.** A deadline passes at its stored timestamp in the Official academic time zone; date-only deadlines mean 11:59 PM. Time-zone changes affect future deadlines only. [Q140, Q176, Q206]
- **BR-12 Blackout adjustment.** Only a deadline that falls on a Blackout date moves to the next non-Blackout day; each unique Blackout date counts once. A newly declared Blackout adjusts only unlocked deadlines; locked or submitted Check-ins are unchanged. Original and adjusted values are audited. [Q177, Q223, Q224, Q251, Q254]
- **BR-13 Blackout pauses.** During a Blackout, inactivity counting, overdue-duration counting, and review countdowns pause, and affected reminders are suppressed. The Monday Morning Briefing continues. Items already overdue before a Blackout stay Overdue and appear in the Briefing; only the overdue-duration count pauses. [Q199, Q207, Q208, Q226, Q248, Q253]
- **BR-14 Reminder rescheduling.** When an authorized deadline changes, or the Representative is replaced, reminders for the old deadline or old Representative are cancelled and a new schedule is generated. Reminder times and recipients use the configuration defaults CFG-04 to CFG-06, and a reminder is addressed to the role or assignee current when it is sent. [Q212, Q247, TA-2, TD-03]

### Milestones, Deliverables, and Action items
- **BR-15 Milestone completion.** A Milestone completes only on supervisor approval of its Deliverable. An on-time valid Deliverable awaiting approval is pending, not overdue. A rejection requires a reason and specific required revisions, notifies the whole Group, and leaves the Milestone incomplete. After rejection it is overdue only if the applicable approved deadline has passed. A late Deliverable stays overdue while awaiting review. Approval clears the active overdue state but keeps lateness history. [Q141, Q160, Q161, Q172, Q227]
- **BR-16 Review periods (revised).** Default review period for Milestones is 7 calendar days and for Action items 3 calendar days; the Administrator sets defaults and supervisors may override per Group. Overdue reviews appear in the Briefing and are escalated to all Assigned supervisors but do not change Group status. **If a review stays undecided for a further 3 calendar days after it became overdue (Administrator-configurable; paused by Blackout dates), the Administrator is also notified so that a supervisor replacement can be considered (BR-31).** If two supervisors decide conflictingly, the first valid decision stands until an audited reopen. [Q159, Q170, Q171, Q192, Q241, Q249, TA-2, CC-11]
- **BR-17 Action-item rules.** Every Action item has a due date. Default due date: the start time of the Group's next scheduled Meeting that begins after the item is created, otherwise 11:59 PM on the seventh calendar day after creation. Creators and current assignees may edit descriptions; supervisors approve deadline changes; fields are frozen while In Review. Every Group member may view all Group Action items. Reassignment notifies the new and previous assignee (and the creator if different). A due date, once set (including a default taken from a Meeting's start time), does not follow a later reschedule or cancellation of that Meeting; it changes only through an approved deadline change or extension (BR-20) [TD-06]. [Q5, Q137, Q168, Q180, Q196, Q210, Q240]
- **BR-18 Status transitions.** The assignee may move Open→Blocked, Open→In Review, Blocked→Open (with a resolution note). Only a supervisor may mark Completed or return an item to Open with a mandatory reason and required corrections. The student's In Review time is the submission time; the supervisor's approval time is the official completion time. [Q147, Q148, Q200]
- **BR-19 Overdue Action items.** Overdue when the due date passes and status is neither Completed nor In Review. Blocked items stay subject to the due date. A rejected item is overdue only if its applicable approved deadline has passed. A pending extension does not stop the item becoming overdue, including while a supervisor is being replaced (BR-31). [Q5, Q157, Q169]
- **BR-20 Extensions.** An extension request needs a reason and a proposed new date. Only an Assigned supervisor may approve, reject, or propose another date. Original, revised, and effective deadlines are kept as separate audited values. [Q149, Q158, Q224]
- **BR-21 Membership changes (revised).** A student cannot be removed until every non-Completed Action item they own is reassigned. A Former member loses all access; records stay available to authorized users. A newly added member can view the Group's full history. An Assigned supervisor may reassign any open Action item of a student being removed, so removal is never blocked by that student's availability. **If the removed student is the Representative, the role becomes vacant and BR-30 applies; the removal itself is not blocked by the vacancy.** [Q154, Q173, Q232, TA-2]
  - **Account deactivation outside a removal [TD-07].** Deactivating a student's account is never blocked by Action items they own, and those items are never reassigned, cancelled, or closed automatically: assignee, status, due date, and history stay unchanged, and overdue rules still apply (BR-19). Deactivation alone does not make the student a Former member; a later removal follows the rules above. In the same all-or-nothing step, the Assigned supervisors of each affected Group are notified with the affected items (BR-22), and those items are shown as "assignee deactivated" (calculated, not stored). Any Assigned supervisor of the Group may reassign such an item to a current Group member with an active account; the reassignment is audited and notified like any other (BR-17).
- **BR-27 Reopening.** Only an Assigned supervisor may reopen a Completed Action item, and must give a reason. The original approval, completion time, and all later changes are preserved. The item returns to Open under normal role permissions, keeps its existing due date, and is overdue immediately if that date has passed unless a supervisor approves an extension. [Q221, Q239, Q250]

### Replacement of unavailable Representatives and supervisors (TA-2)
- **BR-30 Representative replacement.** [TA-2, CC-06, CC-07]
  1. **When a Representative counts as unavailable:** removed from the Group, account deactivated (the role becomes vacant automatically), or replaced by an authorized user because they cannot act (illness, absence, not responding). The system does not detect unavailability by itself.
  2. **Who may replace, before the Check-in deadline:** (i) the current Representative hands over to another member; (ii) a majority of the Group's current members (more than half, the Representative counted) confirm a new Representative; or (iii) any Assigned supervisor.
     - **Majority confirmation rules [TD-10]:** (a) each current Group member with an active account has at most one confirmation at a time, naming one eligible candidate (item 4); a new confirmation replaces that member's previous one, and a member may withdraw a confirmation until the majority is reached; (b) the majority is counted for a single candidate among the Group's current members as stated in (ii), and the replacement takes effect when the confirmation that creates the majority is saved: collecting confirmations is not a pending replacement (item 5); (c) confirmations count only within the current Check-in cycle and expire at its deadline (item 3); they are never carried into another cycle; (d) when membership changes, confirmations of Former members and deactivated accounts are discarded, and a membership change never completes a replacement by itself: the replacement takes effect only when a confirmation is saved that, counted against current membership, forms a majority; (e) all outstanding confirmations are cleared when the Representative changes by any route, including a vacancy; (f) each confirmation and withdrawal is recorded with member and time.
  3. **Who may replace, from the deadline onward (including after a Missed Check-in):** only an Assigned supervisor, who must give a reason. If the Consolidated report is already submitted, a replacement applies only to later cycles; the submitted report is unchanged.
  4. **Eligible candidate:** a current Group member with an active account. Former members, deactivated accounts, and users outside the Group are refused.
  5. **Effect:** a replacement takes effect immediately; there is no pending state. In one all-or-nothing step the old Representative loses edit and submit rights, the new one gains them, and the draft and its history pass unchanged (AC-04b). If the old Representative submits at the same moment, the first saved action stands and the other is refused with a message (no duplicate report, no lost draft).
  6. **Items the old Representative created or submitted** (comments, extension requests, Deliverables, evidence) stay valid with their original author and timestamps. The Representative role owns no Action items; Action items stay with their assignees.
  7. **Deadlines and reminders:** the Check-in deadline, Milestone deadlines, and Action-item due dates are not changed. Remaining reminders for the old Representative are cancelled and generated for the new one (BR-14).
  8. **Missed Check-in:** a recorded Missed Check-in is never removed or excused. A Missed Check-in recorded while the role was vacant or the Representative unavailable is automatically annotated with that fact; an Assigned supervisor may add a further audited annotation but may not delete or excuse it. After a replacement, the new Representative may still submit late: this clears the active warning but keeps the entry (BR-05).
  9. **Vacancy:** while the role is vacant no Consolidated report can be submitted, reminders go to all members instead of the Representative, Assigned supervisors are notified, the Group appears in the Briefing (FR-15), and the deadline is **not** paused. The Group (item 2) or an Assigned supervisor can fill the role at any time within the rules above.
  10. **Audit and notification:** every replacement is audited (initiator, time, previous and new Representative, reason; reason mandatory for supervisor-initiated). The new Representative, the old Representative (if the account is active), all Group members, and the Assigned supervisors are notified (BR-22).
  11. **Initial appointment (before activation) [TD-12]:** a Group may be created and set up without a Representative. Before activation, only an Assigned supervisor of the Group may appoint the Representative, choosing an eligible candidate (item 4), and may change that appointment while the Group is still in setup; each appointment is audited (no reason required). The Group cannot be activated unless its appointed Representative is still eligible at the moment of activation. Items 1 to 10 (including vacancy, item 9) apply only from activation onward; before activation the only consequence of having no eligible Representative is that activation is refused.
- **BR-31 Supervisor replacement.** [TA-2, CC-08 to CC-12]
  1. **Minimum:** every Group always has at least one Assigned supervisor.
  2. **When a supervisor counts as unavailable:** leaves, account deactivated, on leave, or cannot act. Unavailability is declared by performing the Replacement; it is not detected automatically.
  3. **Who may replace:** the Administrator (any supervisor, including a Group's only supervisor), or another Assigned supervisor of the same Group. A Group's only supervisor cannot remove or replace themselves.
  4. **How:** "Replace supervisor" adds an eligible supervisor (active supervisor account) and removes the unavailable one in one all-or-nothing action, so the Group is never without a supervisor. Removing or deactivating the last Assigned supervisor without a named replacement is refused, and the Administrator is shown every affected Group. Adding a supervisor without removing anyone remains the co-supervisor feature (FR-01).
  5. **Pending approvals:** approvals are a Group-level queue, not owned by one supervisor. Deliverables awaiting approval, Action items In Review, extension requests, and deadline-change approvals stay pending with their original submission timestamps and are visible to all current Assigned supervisors, including the new one. A decision takes effect only when submitted: a half-entered decision by the replaced supervisor is not recorded and its submission is refused. Decisions already made stay valid (BR-16).
  6. **Review periods:** review countdowns are **not** reset by a replacement; they continue from the original submission time (paused only by Blackout dates, BR-13). Overdue reviews keep escalating to all current Assigned supervisors, and to the Administrator as set in BR-16.
  7. **Pending extension requests:** they remain pending and, as BR-19 states, do not stop an item becoming overdue. If the decision comes after the due date, approval stores the original, revised, and effective deadlines (BR-20), clears the active overdue state if the effective deadline is in the future, and keeps the lateness history (BR-15).
  8. **Handover:** the new supervisor can view the Group's full history and receives a Handover summary (pending approvals, overdue items, Missed Check-ins, upcoming Meetings and deadlines).
  9. **Meetings and notes:** scheduled Meetings stay scheduled. The replaced supervisor is removed from the attendee list, and their notes remain as authored; any Assigned supervisor may edit them (audited). A meeting-relative Check-in deadline is unchanged unless a Meeting is later rescheduled or cancelled (BR-07).
  10. **Access:** the replaced supervisor loses access to the Group immediately; their earlier actions remain in Audit history. Reinstating them later is a new assignment.
  11. **Audit and notification:** every replacement is audited (initiator, time, previous and new supervisor, reason). The new supervisor, the replaced supervisor (if the account is active), the remaining Assigned supervisors, and all Group members are notified (BR-22).
- **BR-32 Continuity of pending work after a replacement.** A replacement changes who may act, never what has already been recorded or when it is due.

  | Item | When a Representative is replaced (BR-30) | When a supervisor is replaced (BR-31) |
  |---|---|---|
  | Pending approvals | None exist for the Representative role (replacement is immediate). Items the old Representative submitted stay valid with original author and timestamps. | Stay pending with original timestamps; decided by any current Assigned supervisor; the new supervisor gets a Handover summary; half-entered decisions are not recorded. |
  | Missed Check-ins | Entry never removed or excused; after the deadline only a supervisor may replace; a late report by the new Representative clears the active warning only. | History unchanged and visible to the new supervisor. |
  | Existing deadlines | Check-in deadline, Milestone deadlines, Action-item due dates, and counters unchanged; reminders re-targeted. | All deadlines and counters unchanged; review periods not reset; Meetings stay scheduled. |
  | Group status | Recalculated only by BR-10. | Recalculated only by BR-10. |
  | Audit and notification | Audited; BR-22 recipients. | Audited; BR-22 recipients. |

### Notifications and Meetings
- **BR-22 Urgent events (revised).** Immediate notifications go to relevant users for: Missed Check-in, inactivity, overdue Milestone or Action item, review submission and decision, rejected Deliverable, Meeting change, assignment change, Blocked item (to creator and supervisors), **Representative replaced or role vacant (new and previous Representative if active, all Group members, Assigned supervisors), supervisor replaced (new supervisor, replaced supervisor if active, remaining Assigned supervisors, all Group members), a review undecided for a further 3 days (Administrator), and an Action-item assignee's account deactivated (the Group's Assigned supervisors, with the affected items; TD-07)**. In-app delivery is immediate; email is also sent unless the user has turned email off or the Administrator has turned it off for everyone. Email is on by default (CFG-18) [TD-13]. Users may disable email but urgent in-app notifications stay on. Ordinary (non-urgent) notification: Check-in contributions locked by Start review. [Q152, Q191, Q211, TA-1, TA-2]
- **BR-23 Digests.** The user chooses Digest time, time zone, and delivery days; empty Digests are skipped; urgent events may also appear as Digest history. **Clarification [TD-02]:** an urgent event already delivered appears once in the recipient's next Digest, marked "already notified", and is never presented as a new alert; a Digest whose only entries are already-notified history counts as empty and is skipped. Defaults for users who have not chosen are CFG-13. [early §1F, Q44, TD-02]
- **BR-24 Meeting changes.** Creating, rescheduling, or cancelling a Meeting notifies Group students, Assigned supervisors, and listed attendees. Students cannot edit or delete supervisor-authored records. [early §1A]

### Audit and access
- **BR-25 Audit content (revised).** Important changes record actor, timestamp, changed fields, and previous values. Original contributions, Deliverable versions, deadline history, missed-week history, Meeting history, and corrections are preserved. **Every lock (with its trigger), Start review confirmation, "review started in error" annotation, Representative replacement, and supervisor replacement is also recorded.** Ordinary users cannot permanently delete records; drafts may be cancelled by their author and eligible records cancelled by supervisors, always audited. [Q138, Q174, Q181, TA-1, TA-2]
- **BR-33 Multi-factor authentication and recovery [TD-11].** (a) Supervisors and Administrators must use MFA at every sign-in (a one-time code from an authenticator app), enrolled at their first sign-in; students may enrol optionally. (b) At enrolment the user receives 10 single-use recovery codes; each code can replace one MFA code once, and generating a new set invalidates the old one. (c) A user who has lost both the authenticator and the recovery codes asks an Administrator, who may reset that user's MFA enrolment; the user must enrol again at the next sign-in. An Administrator cannot reset their own enrolment. (d) If no other Administrator can act, a person with operator access to the server may reset an Administrator's enrolment through a server-side administrative procedure; this is the only path outside the application. (e) A forgotten password is reset through a single-use link sent to the account's email address; MFA is still required afterwards. (f) The last active Administrator account cannot be deactivated; at least two active Administrator accounts are recommended. (g) Every MFA enrolment, reset, recovery-code use, and password reset is recorded in Audit history (BR-25).
- **BR-26 Files and URLs.** Deliverable and evidence URLs must use HTTPS only. Uploaded files are validated for extension, actual type, and size, against CFG-10 to CFG-12. [Q262, TD-04]

### Configuration defaults (seeded values) [TD-03, TD-04, TD-05]

These values are seeded at installation and stored as configuration, never written into business logic. The Administrator may change them; security values (CFG-15 to CFG-17) may be made stricter but not weaker than NIST SP 800-63B-4. Values already confirmed elsewhere are listed for completeness with their original source.

| ID | Setting | Default | Source |
|---|---|---|---|
| CFG-01 | Official academic time zone | No product default: the Administrator enters the institution's time zone during system setup. A Group cannot be activated until it is set. | TD-03 (OQ-06) |
| CFG-02 | Blackout calendar | Empty; the Administrator enters each term's Blackout dates from the academic calendar. | TD-03 (OQ-06) |
| CFG-03 | Global weekly Check-in deadline | Sunday 11:59 PM | BR-07 [Q166] |
| CFG-04 | Check-in reminders | 48 h and 24 h before the deadline, to the Representative (all members while the role is vacant) and to members whose contribution is missing | AC-06e, BR-30.9 |
| CFG-05 | Milestone reminders | 48 h and 24 h before the Milestone's effective deadline, to all Group members, only while no valid Deliverable has been submitted for it | TD-03 (OQ-06) |
| CFG-06 | Action-item reminders | 48 h and 24 h before the due date, to the current assignee, only while the item is Open or Blocked | TD-03 (OQ-06) |
| CFG-07 | Inactivity thresholds | 7 days (both counters) | BR-08 |
| CFG-08 | Review periods | Milestones 7 calendar days; Action items 3 calendar days | BR-16 |
| CFG-09 | Administrator escalation delay | 3 calendar days after a review becomes overdue | BR-16 [CC-11] |
| CFG-10 | Allowed file types | .pdf, .docx, .pptx, .xlsx, .txt, .md, .png, .jpg, .jpeg. Archives, macro-enabled Office files and executables are not allowed while malware scanning is deferred (D-04); larger or code artifacts are submitted as an HTTPS URL (BR-09). | TD-04 (OQ-07) |
| CFG-11 | Maximum upload size | 25 MB per file | TD-04 (OQ-07) |
| CFG-12 | Unused-upload clean-up period | 24 hours (content uploaded but never attached to a saved record is removed after this period) | TD-04 (OQ-07) |
| CFG-13 | Digest defaults | Enabled; 08:00; Monday to Friday; in the user's time zone detected at first sign-in, otherwise the Official academic time zone | TD-05 (OQ-08) |
| CFG-14 | Dashboard order | As stated in FR-07 | TD-08 (OQ-11) |
| CFG-15 | Password rules | Minimum 15 characters for accounts without mandatory MFA (students); minimum 8 characters for supervisor and Administrator accounts (MFA mandatory, TD-11). At least 64 characters accepted. No composition rules and no periodic change; a change is required only on evidence of compromise. Every new password is checked against a blocklist of commonly used, expected, or compromised passwords. | TD-05 (OQ-08); NIST SP 800-63B-4 |
| CFG-16 | Failed sign-in attempts | Attempts are rate-limited; after at most 100 consecutive failures the password is disabled until it is reset. | TD-05 (OQ-08); NIST SP 800-63B-4 |
| CFG-17 | Sessions | Ended after 30 minutes of inactivity or 12 hours in total, for every role; a new session needs full sign-in, including MFA for supervisors and Administrators. | TD-05 (OQ-08); NIST SP 800-63B-4 (AAL2: inactivity ≤ 1 h, overall ≤ 24 h) |
| CFG-18 | Email notifications | On globally and on for each new user; the user may turn email off (urgent in-app notifications stay on); the Administrator may turn email off for everyone | TD-13 (BR-22) |

## 9. Constraints

- **C-01** Responsive web application usable on desktop, tablet, and mobile.
- **C-02** Support the latest two stable versions of Chrome, Edge, Firefox, and Safari. [Q175]
- **C-03** No technology stack, identity provider, or email provider is fixed by this PRD. (The stack is decided at architecture level in ADR-006.)
- **C-04** All deadlines are calculated in the Administrator-configured Official academic time zone.
- **C-05** Acceptance-test scenarios and pass/fail thresholds are approved jointly by the client supervisor and the project team. [Q144]
- **C-06** Confirmed rules (deadlines, status calculation, permissions, audit) may be changed only through approved, recorded changes.

## 10. Main Flows

| Flow | Trigger | Actors | Normal sequence | Successful outcome | FRs |
|---|---|---|---|---|---|
| F1 Set up a Group | New supervision cycle | Administrator, Assigned supervisor | Admin creates accounts and sets defaults; supervisor adds members, customizes Milestones, appoints the Representative (BR-30.11), activates the Group. | Group is active and counters start. | FR-01, FR-09 |
| F2 Weekly Check-in (revised) | Check-in period opens | Members, Representative | Members submit contributions (editable until lock); draft is assembled; Representative edits and submits; contributions lock. A supervisor can lock early only through F8. | Consolidated report and individual history both exist. | FR-03–06 |
| F3 Find Groups needing attention | Supervisor opens Dashboard | Assigned supervisor | System shows status and both counters; supervisor opens a flagged Group. | Attention list found within 30 seconds. | FR-07, FR-08 |
| F4 Deliverable review | Student ready to submit | Student, Assigned supervisor | Student submits; supervisor approves or rejects with reason and revisions; Group notified; student may resubmit. | Decision, versions, and any extension recorded. | FR-09, FR-10 |
| F5 Action item lifecycle | Work assigned | Student, Assigned supervisor | Create with assignee and due date; assignee works, may mark Blocked; submits evidence (In Review); supervisor completes or returns. | Item has owner, evidence, decision, timestamps. | FR-11, FR-12 |
| F6 Prepare for a Meeting | Meeting due | Assigned supervisor, students | Supervisor reviews prior notes and open Action items, holds Meeting, records notes; students view and comment. | Meeting history and follow-ups available. | FR-13, FR-11 |
| F7 Monday Morning Briefing | Monday | Assigned supervisor | System compiles categories; shows partial-data warning if one fails. | Complete (or honestly partial) briefing. | FR-15, FR-14 |
| **F8 Start a Check-in review** | Supervisor wants to examine contributions before the deadline or before the Representative submits | Assigned supervisor, Group members | 1. Supervisor opens the Check-in details page (read-only; nothing locks). 2. Supervisor selects Start review. 3. System shows the confirmation (who has and has not submitted, permanence, consequences). 4. Supervisor confirms. 5. System locks contributions, records missing ones, writes the audit entry, notifies members. 6. Representative may still submit before the deadline. | Contributions are locked by an explicit, audited action; originals preserved; no accidental lock. | FR-05, FR-03, FR-04 |
| **F9 Replace an unavailable Representative** | Representative removed, deactivated, or unable to act | Group members or Assigned supervisor; new Representative | 1. Authorized user opens the Group's Representative setting. 2. Selects an eligible member (supervisor also gives a reason). 3. System applies it immediately: transfers draft and history, re-targets reminders, audits, notifies. 4. New Representative edits and submits. | Group has an available Representative; deadline and Missed Check-in history unchanged. | FR-04, FR-16 |
| **F10 Replace an unavailable supervisor** | Supervisor leaves, is deactivated, on leave, or cannot act | Administrator or another Assigned supervisor; new supervisor | 1. Opens the Group's supervisor settings. 2. Chooses Replace supervisor and an eligible supervisor account. 3. System shows the pending-work summary, then adds the new and removes the old supervisor in one step. 4. Audits, notifies, sends the Handover summary. 5. New supervisor works the existing queue. | Group never without a supervisor; pending approvals, Missed Check-ins, and deadlines carried over unchanged. | FR-01, FR-16 |

### 10.1 Alternate and failure paths for the new flows

| Flow | Alternate / failure path | Required result |
|---|---|---|
| F8 | Supervisor only opens the page and leaves. | No lock, no audit lock entry (AC-05a). |
| F8 | Supervisor cancels or closes the confirmation, or the session expires. | No lock (AC-05f). |
| F8 | Anyone other than an Assigned supervisor of the Group tries to start review. | Refused; no lock (AC-05e). |
| F8 | Contributions are already locked (submission, deadline, other supervisor) when the supervisor confirms. | Refused with the existing trigger and time; no second audit entry (AC-05g). |
| F8 | Review was started by mistake. | Lock stays; "review started in error" annotation; affected members can add late contributions (AC-05h, AC-05i). |
| F9 | Candidate is a Former member, deactivated, or outside the Group; or a non-authorized user tries. | Refused; Representative unchanged (AC-04d). |
| F9 | Replacement attempted by Group members after the deadline. | Refused; only a supervisor may replace (AC-04e). |
| F9 | Representative removed or deactivated and nobody is appointed before the deadline. | Role vacant; Missed Check-in recorded and annotated (AC-04f). |
| F9 | Old Representative submits at the moment of replacement. | First saved action stands (AC-04g). |
| F10 | Last Assigned supervisor is removed or deactivated with no replacement, or candidate is not an active supervisor, or a student attempts it. | Refused; affected Groups listed (AC-16b). |
| F10 | Replaced supervisor was mid-decision. | Submission refused; nothing recorded (AC-16f). |
| F10 | An extension request was pending and the due date passed during the gap. | Item is overdue; later approval clears it and keeps history (AC-16c). |

## 11. Edge Cases

| ID | Situation | Required behavior | Rules | Verified by |
|---|---|---|---|---|
| EC-01 | Some contributions missing; Representative submits on time. | Group Check-in counts as on time; missing contributions recorded and notified; status unchanged. | BR-04, BR-10 | AC-05b |
| EC-02 | No Consolidated report at deadline; submitted later. | Missed Check-in recorded; late report clears warning; history kept. | BR-05 | AC-06b |
| EC-03 (revised) | Supervisor opens Check-in details before submission. | **Nothing locks.** Members can still edit their contributions; the Representative can still edit and submit. | BR-02, BR-28 | AC-05a |
| EC-04 | Check-in arrives while Deliverable counter is over threshold. | Only the Check-in counter resets; flag stays active. | BR-08 | AC-08b |
| EC-05 | Deadline falls on a Blackout date; another does not. | Only the affected deadline moves. | BR-12 | AC-06c |
| EC-06 | Blackout added after a Check-in is locked or submitted. | Locked deadline and history unchanged. | BR-12 | AC-06d |
| EC-07 | On-time valid Deliverable; supervisor approval is late. | Milestone stays pending, not overdue. | BR-15 | AC-10b |
| EC-08 | Deliverable rejected after the original deadline. | Milestone incomplete and overdue unless an approved later deadline exists. | BR-15 | AC-10c |
| EC-09 | Action item is Blocked when its due date passes. | Becomes overdue under normal rules. | BR-19 | AC-12c |
| EC-10 | Action item In Review, then rejected after its deadline. | Returns to Open; overdue again if the applicable deadline has passed. | BR-19 | AC-12b |
| EC-11 | Review of an Action item or Deliverable is overdue. | Escalated and in Briefing; Group status unchanged. | BR-16 | AC-15b |
| EC-12 | Student removed while holding open Action items. | Removal blocked until each is reassigned. | BR-21 | AC-01b |
| EC-13 | Former member opens old Group records. | Access denied; records preserved for authorized users. | BR-21, NFR-03 | AC-02b |
| EC-14 | Briefing data category fails to load. | Show available categories, identify the failed one, warn partial, offer retry. | FR-15 | AC-15c |
| EC-15 | User disables email. | Urgent in-app notifications still delivered; records still save. | BR-22, NFR-04 | AC-14b |
| EC-16 | File has a misleading extension or URL is HTTP. | File rejected on type check; HTTP URL rejected. | BR-26 | AC-10d |
| EC-17 | URL is valid but inaccessible. | Accepted as valid with warning; supervisor assesses. | BR-09 | AC-10e |
| EC-18 | Two supervisors decide the same review differently. | First valid decision stands; change needs an audited reopen. | BR-16 | AC-12d |
| EC-19 | A Meeting is scheduled or moved to less than 24 hours ahead during an open Check-in cycle. | The previous deadline is kept; no early Missed Check-in. | BR-07 | AC-06f |
| EC-20 | Monday Briefing falls inside a Blackout. | Briefing is generated; already-overdue items stay listed; reminders suppressed. | BR-13 | AC-15d |
| EC-21 | Supervisor starts review before the deadline while some contributions are missing. | Contributions lock at confirmation; missing ones recorded and notified; Representative can still submit on time; no Missed Check-in; status unchanged. | BR-28, BR-04, BR-03 | AC-05d |
| EC-22 | Start review is attempted by an unauthorized user, abandoned before confirming, requested when already locked, or confirmed by two supervisors at once. | No lock for unauthorized or abandoned attempts; refusal naming the existing lock; exactly one lock and one audit entry for simultaneous confirmations. | BR-28 | AC-05e, AC-05f, AC-05g |
| EC-23 | A member recorded missing because of an early lock submits afterwards, or the review was started by mistake. | Late contribution stored separately and linked; error annotation added; lock never reversed. | BR-29, BR-06 | AC-05h, AC-05i |
| EC-24 | Representative becomes unavailable before the deadline and the Group replaces them. | Replacement immediate; draft and history pass on; reminders re-targeted; deadline unchanged. | BR-30 | AC-04c |
| EC-25 | Representative is replaced after the deadline when a Missed Check-in already exists. | Only a supervisor may replace; Missed entry stays; new Representative's late report clears the warning only. | BR-30, BR-05 | AC-04e, AC-06g |
| EC-26 | Representative is removed or deactivated and no replacement is chosen before the deadline. | Role vacant; removal not blocked; reminders to all members; Missed Check-in recorded and annotated; deadline not paused. | BR-30, BR-21 | AC-04f |
| EC-27 | Replacement candidate is ineligible or the initiator is not authorized. | Refused; Representative unchanged. | BR-30 | AC-04d |
| EC-28 | Old Representative submits at the same moment as the replacement. | First saved action stands; never two reports or a lost draft. | BR-30 | AC-04g |
| EC-29 | A supervisor is replaced while Deliverables, Action items In Review, or extensions are pending. | Items stay pending with original timestamps; review periods not reset; new supervisor receives a Handover summary and can decide. | BR-31, BR-32 | AC-16a |
| EC-30 | Last Assigned supervisor is removed or deactivated without a replacement, or the candidate is ineligible. | Refused; affected Groups listed; Group keeps its supervisor. | BR-31 | AC-16b |
| EC-31 | Extension request pending while the supervisor is unavailable and the due date passes. | Item is overdue (BR-19); a later approval stores revised and effective dates, clears the overdue state, keeps lateness history. | BR-31, BR-19, BR-20 | AC-16c |
| EC-32 | A review stays undecided because the only supervisor is unresponsive. | Escalation to all supervisors when overdue and to the Administrator after a further 3 days; Group status unchanged. | BR-16, BR-31 | AC-16d |
| EC-33 | The replaced supervisor had scheduled a Meeting and the Check-in deadline is meeting-relative; or was mid-decision. | Meeting stays; deadline unchanged; half-entered decision refused and not recorded. | BR-31, BR-07 | AC-16e, AC-16f |
| EC-34 | A student's account is deactivated (not removed) while they own open Action items. | Deactivation not blocked; items unchanged and still subject to overdue rules; supervisors notified; items shown "assignee deactivated"; supervisor may reassign to an active member. | BR-21, BR-22 | AC-01d |
| EC-35 | In meeting-relative mode the driving Meeting is rescheduled or cancelled, or no Meeting falls in the period. | Driving Meeting determined again; deadline recalculated only while the Check-in is unsubmitted and unlocked and only if the result is in the future; otherwise the global weekly deadline or the previous deadline applies. | BR-07 | AC-06h, AC-06i |
| EC-37 | A supervisor tries to activate a Group with no Representative, or whose appointed Representative was removed or deactivated during setup. | Activation refused with a message; Group stays in setup; no vacancy warning or Briefing entry (the Group is not active). | BR-30.11, FR-01 | AC-01f |
| EC-38 | A supervisor or Administrator loses the authenticator device. | Signs in with a recovery code, or an Administrator resets the enrolment (never their own); the last Administrator is recovered only through the server-side procedure. | BR-33, NFR-03 | AC-02e |
| EC-36 | Majority confirmations for a new Representative are withdrawn, span a deadline, or are affected by a membership change. | Withdrawn and expired confirmations do not count; votes never carry across cycles; a membership change never completes a replacement by itself. | BR-30.2 | AC-04h |

## 12. Acceptance Criteria

Each criterion is linked to the requirement it verifies. IDs with a letter (AC-05a) test a specific rule or edge case.

| ID | Verifies | Given | When | Then (pass condition) |
|---|---|---|---|---|
| AC-01a | FR-01 | Admin account; self-registration disabled | Admin creates a student account and invitation; a visitor tries to self-register | Account and invitation are created; self-registration attempt is refused. With registration enabled, it succeeds. |
| AC-01b | FR-01, BR-21, EC-12 | Student has 2 open and 1 Completed Action item | Supervisor removes the student | Removal is refused until both open items are reassigned; after reassignment it succeeds. |
| AC-01c | FR-01 | Group with setup and Milestones done and an eligible Representative appointed | Supervisor activates it | Group shows Active; Check-in counter starts at first scheduled deadline, Deliverable counter at activation date. |
| AC-01d | FR-01, BR-21, BR-22, EC-34 | Student S in Group Alpha owns 2 open Action items (one due tomorrow) and 1 Completed item | The Administrator deactivates S's account; later a supervisor reassigns one item to Member B, then tries to reassign the other to S, to a Former member, and to a user of another Group | Deactivation succeeds (not blocked by the items); both items keep assignee S, status, and due date; each Assigned supervisor is notified with the 2 items; the items show "assignee deactivated"; the item due tomorrow becomes overdue when its date passes (Group Delayed, BR-10); the reassignment to B succeeds and is audited; the three other reassignments are refused; nothing was reassigned or closed automatically. |
| AC-01e | FR-01, CFG-01 | New installation; Official academic time zone not yet set | A supervisor tries to activate a set-up Group; the Administrator sets the time zone; the supervisor retries | The first attempt is refused with a message; after the time zone is set, activation succeeds. |
| AC-01f | FR-01, BR-30, EC-37 | A Group in setup with members A, B, C and no Representative | The supervisor activates it; a student and the Administrator try to appoint A; the supervisor appoints a Former member, then A, then changes it to B; B's account is deactivated; the supervisor activates; the supervisor appoints C and activates | The first activation is refused; the student's and Administrator's appointments and the Former-member appointment are refused; the appointments of A and B are audited; activation with deactivated B is refused; after C is appointed, activation succeeds, C is the Representative, and later changes follow BR-30. |
| AC-02a | FR-02, NFR-03 | Students in Group A and B; a supervisor assigned to A only | Each requests the other Group's records | All cross-Group requests are denied; own-Group requests succeed. |
| AC-02b | FR-02, EC-13 | A Former member with past submissions | They open the former Group | Access is denied; supervisor still sees the past submissions. |
| AC-02c | FR-02, BR-25 | An Action item is edited | Supervisor opens Audit history; a student tries to open it | Entry shows actor, time, changed field, previous value; student is denied. |
| AC-02d | NFR-03, CFG-15 to CFG-17 | Default security configuration | A student sets a 14-character and then a 15-character password; a supervisor and an Administrator each set an 8-character password; a user sets a blocklisted password, a password of one character type, and a 64-character password; 100 consecutive failed sign-ins are made on one account; one session stays idle 30 minutes and another lasts 12 hours | 14 characters refused, 15 accepted; the supervisor's and Administrator's 8 characters accepted, with MFA still required; the blocklisted password refused; no composition rule is enforced; 64 characters accepted; passwords never expire by age; after the 100th failure the password is disabled until reset; both sessions end and a new sign-in (with MFA for the supervisor) is required. |
| AC-02e | NFR-03, BR-33, EC-38 | Administrators X and Y; supervisor S enrolled in MFA with 10 recovery codes | (1) X and S sign in with a correct password but no MFA code. (2) S signs in with a recovery code, then reuses it. (3) S lost both factors: a student and S try to reset S's MFA; X resets it; X tries to reset X's own. (4) X deactivates Y, then tries to deactivate X's own account while X is the last active Administrator | (1) Both refused until the MFA code is given. (2) The first use succeeds; the reuse is refused. (3) The student's and S's attempts are refused; X's reset succeeds and S must enrol again at next sign-in; X's self-reset is refused. (4) Deactivating Y succeeds; deactivating the last active Administrator is refused. Every enrolment, reset and recovery-code use has an audit entry. |
| AC-03 | FR-03, BR-01 | Open Check-in | Member submits with one field blank, then with "None" typed | Blank is rejected with a message; "None" is accepted. |
| AC-04a | FR-04 | Three members submitted | Representative opens the draft, edits, submits | One Consolidated report is saved; contributions are unchanged. |
| AC-04b | FR-04 | Draft in progress | Group changes Representative before deadline | New Representative can open the existing draft with draft history intact. |
| AC-04c | FR-04, FR-16, BR-30, EC-24 | Open Check-in, deadline 3 days away; Representative (Member A) is on leave with a draft in progress; Group of 3 | Members B and C (a majority) confirm Member B as Representative | Replacement is immediate; B opens the existing draft with history; A can no longer edit or submit; A's reminders are cancelled and B's generated; the deadline is unchanged; supervisors and all members are notified; the audit entry shows initiator, time, old and new Representative. |
| AC-04d | FR-04, BR-30, EC-27 | Group has a Former member D and a member E with a deactivated account; Check-in open | One member alone (1 of 3) tries to replace the Representative; a supervisor tries to appoint D, then E; a user from another Group tries | All four attempts are refused with a message; the Representative is unchanged; nothing is applied. |
| AC-04e | FR-04, BR-30, EC-25 | Deadline passed; Missed Check-in recorded; Representative unavailable | A Group majority tries to replace; then an Assigned supervisor replaces with a reason | The majority attempt is refused (after the deadline only a supervisor may replace); the supervisor's replacement succeeds; the Missed Check-in entry and deadline are unchanged. |
| AC-04f | FR-04, BR-30, BR-21, EC-26 | Representative A owns 1 open Action item; deadline in 2 days | Supervisor reassigns A's item and removes A; nobody is appointed before the deadline | Removal succeeds (not blocked by the vacancy); the role shows vacant; reminders go to all remaining members; supervisors are notified; at the deadline a Missed Check-in is recorded, annotated "Representative role vacant at deadline", and all members are notified; Group is At Risk (BR-10); the Briefing lists the vacancy. |
| AC-04g | FR-04, BR-30, EC-28 | Representative A is submitting the Consolidated report | A supervisor replaces A at the same moment | Whichever is saved first stands: if the report, it is A's valid submission and the replacement applies to later cycles; if the replacement, A's submission is refused with a message and the new Representative can submit. Never two reports; the draft is not lost. |
| AC-04h | FR-04, BR-30, EC-36 | Group of 4: Member A (Representative), B, C, D; deadline in 2 days | (1) B and C confirm C; B withdraws; B confirms C again; D confirms C. (2) In a later cycle B and C confirm D, the deadline passes, and in the next cycle C confirms D. (3) In another cycle B and C confirm C, then D is removed from the Group, then B saves the confirmation again | (1) No replacement at 2 of 4 or after the withdrawal; C becomes Representative immediately when D's confirmation makes 3 of 4, with audit and notifications. (2) The earlier confirmations expired at the deadline; C's new confirmation alone (1 of 4) changes nothing. (3) Removing D completes nothing by itself; B's saved confirmation makes 2 of 3 and C becomes Representative. |
| AC-05a (revised) | FR-05, BR-02, BR-28, EC-03 | Check-in open, deadline not reached; 2 of 3 members submitted | Assigned supervisor opens the details page, reads the contributions, and leaves without selecting any action | No lock is created and no lock audit entry exists; the third member can still submit and edit; the Representative can still edit the draft. |
| AC-05b | FR-05, EC-01 | One of three members did not submit | Representative submits on time | Group Check-in is on time; the omission is recorded and the student notified; status unchanged. |
| AC-05c | FR-05, BR-06 | A locked contribution has an error | Student adds a correction comment; supervisor appends a corrected version | Original remains unchanged; both versions and the link between them are visible. |
| AC-05d | FR-05, BR-28, BR-04, EC-21 | Same state as AC-05a (2 of 3 submitted; deadline not reached) | Supervisor selects Start review; the confirmation names the missing member and states the lock is permanent; supervisor confirms | All contributions lock at the confirmation time; the audit entry shows supervisor, time, trigger "Start review", 2 locked, 1 recorded missing; the missing member is notified and Group members are told contributions are locked; the Representative's later on-time submission is accepted using saved contributions; no Missed Check-in; Group status unchanged. |
| AC-05e | FR-05, NFR-03, BR-28, EC-22 | Check-in open | A student, the Representative, the Administrator, and a supervisor of another Group each try to start review | All are refused; no lock is created. |
| AC-05f | FR-05, BR-28, EC-22 | Supervisor has opened the Start review confirmation | Supervisor cancels, closes it, or the session expires | No lock; contributions remain editable; no audit lock entry. |
| AC-05g | FR-05, BR-28, EC-22 | Case 1: Representative already submitted. Case 2: deadline passed. Case 3: confirmation dialog opened at 11:58 PM and confirmed after the 11:59 PM deadline. Case 4: two supervisors confirm at once | Start review is confirmed in each case | Cases 1–3 are refused with a message naming the existing lock trigger and time; no second audit entry. Case 4: exactly one lock and one audit entry; the second supervisor is told who locked first. |
| AC-05h | FR-05, BR-29, EC-23 | A member was recorded missing because of a Start review | The member submits a late contribution | It is stored separately and linked to the missing record; the missing record and the note "recorded at supervisor-started review" remain; the Group's on-time status is unchanged; the Representative can see it. |
| AC-05i | FR-05, BR-29, BR-06, EC-23 | Review was started by mistake and a member is recorded missing | A supervisor appends "review started in error" with a reason; then someone tries to unlock | The annotation appears beside the missing record and the member is notified; the contributions stay locked; the unlock attempt is not possible; the missing record is not deleted. |
| AC-06a | FR-06 | Global deadline Sunday 11:59 PM; Group has an override | A cycle opens | Group uses its override; other Groups use the global deadline; existing active deadlines unchanged after any later change. |
| AC-06b | FR-06, EC-02 | Deadline passes with no report | Time passes; Representative submits next day | Missed Check-in recorded and all members notified at the deadline; after submission the warning clears but the Missed entry remains. |
| AC-06c | FR-06, EC-05 | Two Groups: one deadline on a Blackout date, one not | Deadlines are calculated | Only the first moves to the next non-Blackout day. |
| AC-06d | FR-06, EC-06 | A submitted Check-in | Admin adds a Blackout on its deadline date | Submitted Check-in's deadline is unchanged. |
| AC-06e | FR-06 | Deadline 48 h and 24 h away | Reminder times arrive | Representative and members with missing contributions receive both reminders. |
| AC-06f | FR-06, BR-07, EC-19 | Check-in cycle open with a weekly deadline; no Check-in submitted | A Meeting is scheduled 10 hours ahead, then an existing Meeting is moved to 5 hours ahead | In both cases the previous deadline stays in force and no Missed Check-in is recorded early. |
| AC-06g | FR-06, BR-05, BR-30, EC-25 | Sunday 11:59 PM deadline passed with no report; Missed Check-in recorded; Representative unreachable | On Monday a supervisor replaces the Representative and the new Representative submits at 10:00 | The Missed entry remains with its original timestamp; the submission is recorded as late and clears the active warning; the deadline is not extended; Group status is recalculated by BR-10. |
| AC-06h | FR-06, BR-07, EC-35 | Group in meeting-relative mode; the period ends Sunday 11:59 PM; Meetings on Thursday 14:00 and Friday 10:00; nothing submitted; it is Monday | The deadline is calculated; on Tuesday the Thursday Meeting is cancelled; on Wednesday the Friday Meeting is cancelled | Deadline is Wednesday 14:00; after the first cancellation, Thursday 10:00; after the second, Sunday 11:59 PM (global weekly). Each change is kept in the deadline history with previous and new values. |
| AC-06i | FR-06, BR-07, EC-35 | Group A uses the global weekly deadline; Group B (meeting-relative) has already submitted its Consolidated report | A Meeting is scheduled for Group A on Wednesday; Group B's driving Meeting is rescheduled; a supervisor switches Group A to meeting-relative mid-cycle | Group A's deadline is unchanged; Group B's deadline is unchanged (submitted); Group A's new setting applies from the next cycle. |
| AC-07 | FR-07, FR-08, NFR-02 | 4 test Groups in different states | Supervisor opens Dashboard | Each Group shows status and both counters; every Delayed, At Risk, or inactivity-flagged Group is marked as requiring attention, and a tester finds all of them within 30 seconds. |
| AC-07b | FR-07 | Groups Delta (Delayed, larger counter 3), Bravo (Delayed, 9), Echo (At Risk, 8), Alpha (At Risk, 8), Charlie (On Track) on one supervisor's Dashboard | Supervisor opens the Dashboard | Order is Bravo, Delta, Alpha, Echo (tie broken by name), Charlie; the first four are marked as requiring attention. |
| AC-08a | FR-08 | Threshold 7 days; last Check-in 8 date-boundaries ago | Counter is calculated | Flag is active; with threshold changed to 10 it recalculates to inactive immediately. |
| AC-08b | FR-08, EC-04 | Both counters over threshold | A valid Check-in is submitted | Only the Check-in counter resets; flag remains active. |
| AC-08c | FR-08 | Group states: overdue Action item; missed Check-in only; none | Status recalculates | Delayed; At Risk; On Track respectively. |
| AC-09a | FR-09 | Admin template with 7 Milestones | Supervisor customizes one Group; Admin later edits the template | Group keeps its customization; edited template applies only to Groups created afterward. |
| AC-09b | FR-09, BR-20 | Student requests an extension without a reason | Student then resubmits with reason and proposed date; supervisor approves | First request is refused; approval stores original, revised, and effective deadlines. |
| AC-10a | FR-10 | A Milestone | Student submits a Deliverable without a title, then with all fields | Incomplete is refused; complete is accepted as valid regardless of approval. |
| AC-10b | FR-10, EC-07 | On-time valid Deliverable | Supervisor has not approved after the deadline | Milestone shown as pending, not overdue. |
| AC-10c | FR-10, EC-08 | Deliverable on time | Supervisor rejects after the deadline | Rejection requires reason and revisions; Group is notified; previous version kept; Milestone is overdue. |
| AC-10d | FR-10, EC-16 | Test files and URLs | Upload a .exe renamed .pdf; submit an `http://` link | Both are rejected. |
| AC-10e | FR-10, EC-17 | Valid HTTPS URL that returns not-found | Student submits it | Accepted as valid, warning shown, counter reset. |
| AC-10f | FR-10, BR-26, EC-16 | Default intake configuration (CFG-10, CFG-11) | Upload a 24 MB PDF, a PNG, a 26 MB PDF, a .zip file, and a .docm file | The 24 MB PDF and the PNG are accepted; the 26 MB PDF, the .zip and the .docm are rejected with a message naming the rule, and nothing is attached for them. |
| AC-11a | FR-11, BR-11, BR-17 | Meeting scheduled in 3 days | Create an Action item with no due date; repeat with no future Meeting | Due date equals Meeting start; otherwise 11:59 PM on day 7. |
| AC-11b | FR-11 | Item assigned to A | Member reassigns to B | Change audited; B notified immediately; A notified; creator notified if neither. |
| AC-11c | FR-11 | Item is In Review | Student tries to edit the description | Edit refused. |
| AC-11d | FR-11, BR-17, BR-20 | An Action item created without a due date took the start of Thursday's 10:00 Meeting as its due date | The Meeting is rescheduled to Friday 10:00, then cancelled; later a supervisor approves a deadline change to Monday | The due date stays Thursday 10:00 after the reschedule and the cancellation (the Meeting-change notification is sent, BR-24); after the approval the original, revised, and effective deadlines are stored and audited. |
| AC-12a | FR-12, BR-18 | Open item | Assignee submits URL evidence; supervisor approves | Status In Review stops the overdue clock; after approval it is Completed with distinct submission and approval timestamps. |
| AC-12b | FR-12, BR-18, BR-19, EC-10 | In Review item; due date passed | Supervisor rejects without reason, then with reason and corrections | First attempt refused; second returns to Open, assignee notified, item overdue. |
| AC-12c | FR-12, EC-09 | Blocked item | Due date passes | Item becomes overdue; Group is Delayed. |
| AC-12d | FR-12, EC-18 | Two supervisors review the same item | Both submit conflicting decisions | The first is kept; the second is refused with a message. |
| AC-12e | FR-12, BR-27 | A Completed item whose due date has passed | A student tries to reopen it; then a supervisor reopens it with and without a reason | Student attempt is refused; supervisor without reason is refused; with reason the item is Open and overdue, the original approval remains in history. |
| AC-13a | FR-13 | A Group with supervisor | Supervisor creates, reschedules, cancels a Meeting | Each is saved; students, supervisors, and attendees are notified. |
| AC-13b | FR-13 | Existing supervisor notes | Student views, comments, tries to edit or delete a note | View and comment succeed; edit and delete are refused. |
| AC-14a | FR-14 | An urgent event occurs | Event is processed | Relevant users get an in-app notification immediately; email also if enabled. |
| AC-14b | FR-14, EC-15 | User disabled email | Urgent event occurs | In-app notification still arrives; records save. |
| AC-14h | FR-14, BR-22, CFG-18 | A new user who has never changed notification settings | An urgent event occurs; the user turns email off; another urgent event occurs; the Administrator turns email off for everyone | The first event is delivered in-app and by email; after the user turns email off, in-app only; with email off globally, no email is sent to anyone and all records still save. |
| AC-14c | FR-14 | User sets Digest time and days; one day with nothing to report | Digest time arrives | Digest is sent on selected days only; the empty one is skipped. |
| AC-14d (revised) | FR-14, BR-22 | One test event per urgent event type | Each event occurs: Missed Check-in; inactivity threshold exceeded; Milestone or Action item overdue; review submitted; review decision; Deliverable rejected; Meeting changed; Action item reassigned; Action item Blocked; **Representative replaced or role vacant; supervisor replaced; review undecided for a further 3 days; Action-item assignee's account deactivated** | Recipients are exactly: all Group members for a Missed Check-in; Assigned supervisors for inactivity and for a submitted review; assignee and all Assigned supervisors for overdue; assignee for a decision; whole Group for a rejected Deliverable; affected users for a Meeting change; new and previous assignee (and creator if different) for reassignment; creator and Assigned supervisors, with the blocking reason, for Blocked; **new and previous Representative (if active), all Group members, and Assigned supervisors for a Representative replacement or vacancy; the new supervisor, the replaced supervisor (if active), the remaining Assigned supervisors, and all Group members for a supervisor replacement; the Administrator for a review undecided for a further 3 days; the Group's Assigned supervisors, with the affected items, for a deactivated assignee.** |
| AC-14e | FR-14, BR-23 | User with a daily Digest; an urgent Missed Check-in notification was delivered at 09:00 | The next Digest time arrives with nothing else new; on the following day the Digest has one ordinary notification and one urgent event already delivered | The first Digest is skipped (only already-notified history); the second shows the ordinary notification as new and the urgent event once, marked "already notified". |
| AC-14f | FR-14, BR-14, BR-13, CFG-05, CFG-06 | A Milestone with no Deliverable and an Action item assigned to B (Open), both due Friday 11:59 PM | Wednesday 11:59 PM and Thursday 11:59 PM pass. Repeat with: a valid Deliverable submitted and the item moved to In Review before Thursday; the item reassigned to C before Thursday; Thursday declared a Blackout | Members receive both Milestone reminders and B both item reminders. In the repeats: no further Milestone or item reminder after the Deliverable or In Review; the Thursday item reminder goes to C, not B; the reminder falling on the Blackout is suppressed. |
| AC-14g | FR-14, BR-23, CFG-13 | A new user who has never set Digest preferences; time zone detected at first sign-in | A weekday 08:00 and a Saturday 08:00 arrive (local time), each with something to report | A Digest is delivered on the weekday and none on Saturday; the user can change time, time zone, and days. |
| AC-15a (revised) | FR-15, NFR-02 | Test dataset with known qualifying records in each category | Briefing is generated | Every qualifying record appears (100%), including missing contributions and Blocked items in separate lists, **and Groups whose Representative role is vacant**. |
| AC-15b | FR-15, EC-11 | A review past its deadline | Briefing is generated | Review appears as overdue; Group status unchanged. |
| AC-15c | FR-15, EC-14 | One category is made to fail | Briefing is opened | Other categories shown, failure named, "partial data" warning, retry works, and it is not labelled complete. |
| AC-15d | FR-15, BR-13, EC-20 | An Action item became overdue before a Blackout began | Monday falls inside the Blackout and the Briefing is generated | The item is listed as overdue, its overdue-duration count is paused, and no reminder is sent for it. |
| AC-16a | FR-16, FR-01, BR-31, BR-32, EC-29 | Group with supervisors S1 and S2; S2 is leaving. Pending: a Deliverable awaiting approval for 2 days, an Action item In Review for 1 day, an extension request; one Missed Check-in in history; a Milestone deadline in 4 days; a Check-in deadline in 3 days | The Administrator (or S1) replaces S2 with S3 | S3 is added and S2 removed in one action (the Group never has zero supervisors); S2 loses access; S3 sees the full history and receives a Handover summary listing the pending items, the Missed Check-in, and upcoming deadlines; the pending items stay pending with original timestamps and review periods are not reset; all deadlines and the Missed entry are unchanged; Group status is unchanged; the audit entry and notifications exist; S1 or S3 can approve the Deliverable. |
| AC-16b | FR-16, FR-01, NFR-03, BR-31, EC-30 | Group whose only supervisor is S1 | Administrator tries to deactivate S1 or remove S1 without a replacement; a student tries to replace the supervisor; the Administrator picks a candidate without an active supervisor account; then picks a valid supervisor S4 | The first three attempts are refused (the Administrator is shown the affected Groups) and S1 remains; the valid replacement with S4 succeeds. |
| AC-16c | FR-16, BR-31, BR-19, BR-20, EC-31 | Action item due Friday; the student requested an extension on Thursday; the only supervisor is unreachable; Friday passes | On Saturday the Administrator replaces S1 with S4, and S4 approves the extension to next Wednesday | The item was overdue from Friday and the Group Delayed meanwhile; on approval the original, revised, and effective deadlines are stored, the active overdue state clears, lateness history is kept, and the actions are audited. |
| AC-16d | FR-16, BR-16, EC-32 | A review submitted; only supervisor S1 is unresponsive | The review period passes, then a further 3 calendar days (Blackout days not counted) | S1 is notified when the review becomes overdue; the Administrator is notified at the further 3 days; Group status is unchanged; the Administrator can then replace S1. |
| AC-16e | FR-16, BR-31, BR-07, EC-33 | S2 scheduled a Meeting in 2 days; the Group's Check-in deadline is meeting-relative | S2 is replaced by S3 | The Meeting stays scheduled; S2 leaves the attendee list; the Check-in deadline is unchanged; if S3 later reschedules the Meeting to under 24 hours ahead, BR-07 applies. |
| AC-16f | FR-16, BR-31, EC-33 | S2 has a rejection form open on an Action item In Review | The Administrator replaces S2; S2 then submits the rejection | The submission is refused; nothing is recorded; the item stays In Review with its original timestamp; S3 can decide. |

## 13. Deferred Decisions (decided in grilling; outside the core build)

The grilling log contains confirmed decisions for the items below. They are kept so no decision is lost. **Team decision:** they are not core capabilities for the project duration (feasibility) and are excluded from the build; they may be added after the core release.

| ID | Item | Original FR / decisions | Reason deferred |
|---|---|---|---|
| D-01 | Two-way external calendar synchronization, conflict handling, and integration retry/escalation | V3 FR-28, FR-29, FR-30, FR-39; Q142, Q155, Q184, Q203, Q213, Q222, Q245 | Not part of the validated problem; needs a provider decision and carries high implementation risk. |
| D-02 | Archive/reopen, retention, and deletion automation | V3 FR-36, FR-37, NFR-13; Q145, Q185, Q186, Q256–Q261 | Depends on the university retention policy (not yet available); can be added after core use. |
| D-03 | Backup, recovery, RPO 1 h / RTO 4 h | V3 FR-38, NFR-10; Q188, Q198 | Depends on hosting choice. |
| D-04 | Malware scanning and quarantine | V3 FR-40 (part); Q179, Q189, Q197, Q204, Q219 | Needs a scanning service; basic type, size, and HTTPS checks remain in BR-26. |
| D-05 | Capacity and scalability NFR | V3 NFR-11; Q143 | No numeric target can be verified until institution estimates exist. |

## 14. Assumptions

1. Delayed outranks At Risk. [early §1C]
2. The initial standard Milestone template is Proposal, SRS, Design, Prototype, Midterm, Final Report, and Defense; the Administrator may change it for future Groups. [Q151]
3. Multiple supervisors may share one Group. [Q1]
4. A valid Deliverable can reset the Deliverable counter while approval is pending.
5. Reopening a Completed Action item is limited to Assigned supervisors (resolves the Q221 / Q250 conflict in favor of the stricter rule).
6. The system cannot detect that a Representative or supervisor is unavailable; unavailability is declared by an authorized user performing the Replacement (BR-30, BR-31). [TA-2]
7. The Administrator has no Group-level supervision rights other than membership override and supervisor replacement; in particular the Administrator cannot start a Check-in review or approve Deliverables, Action items, or extensions. [TA-1, TA-2, CC-01, CC-08]
8. A late contribution after a lock has no cutoff other than the Group's records remaining accessible to the member. [CC-05]
9. A Missed Check-in caused by a vacant or unavailable Representative is not excused automatically; it is annotated only. [CC-07]

## 15. Open Questions

### 15.1 Still open

**None.** Every open question is resolved: OQ-02 to OQ-04 by client confirmation in Deliverable 3 (Section 15.2), and OQ-01, OQ-05 to OQ-13 by team decision under client-delegated authority in this final revision (Section 15.3).

### 15.2 Resolved in Deliverable 3

| ID | Question | Resolution | Confirmation |
|---|---|---|---|
| OQ-02 | Should merely opening the Check-in details page lock contributions, or only an explicit "start review" action? | Only the explicit, confirmed Start review action (or Representative submission or the deadline) locks; opening a page never does (BR-02, BR-28). | CC-01 to CC-04 |
| OQ-03 | What happens if a pending review has no available Assigned supervisor, or the responsible supervisor leaves? | Pending approvals stay in the Group-level queue; the Administrator replaces the supervisor (never leaving the Group with none); review periods are not reset; the Administrator is escalated to after a further 3 days (BR-16, BR-31). | CC-08 to CC-11 |
| OQ-04 | Who may replace or stand in for an unavailable Representative after the deadline, and does it change a Missed Check-in? | After the deadline only an Assigned supervisor may replace; a Missed Check-in is never changed or excused by a replacement (BR-30, BR-05). | CC-06, CC-07 |

### 15.3 Resolved in the final revision (team decisions under client-delegated authority)

OQ-12 and OQ-13 were not listed in Deliverable 3; they are requirement gaps found during the architecture review (formerly `ARCHITECTURE.md` items G-20 and G-19) and are recorded here so that every decision is traceable.

| ID | Question | Resolution | Decision | PRD items affected |
|---|---|---|---|---|
| OQ-01 | Test load, dataset, and environment for NFR-01 and NFR-02 | Approved test profiles TP-01 and TP-02 | TD-01 | NFR-01, NFR-02, Section 7.1 |
| OQ-05 | Duplicate presentation of an event that is both urgent and in a Digest | Shown once as "already notified" history; history-only Digests are skipped | TD-02 | BR-23, AC-14e |
| OQ-06 | Official time zone, Blackout calendar, reminder intervals | Time zone and Blackouts are Administrator-entered data; Milestone and Action-item reminders 48 h and 24 h before | TD-03 | FR-14, BR-14, CFG-01 to CFG-06, AC-01e, AC-14f |
| OQ-07 | Allowed file extensions and maximum upload size | Allowlist and 25 MB limit | TD-04 | BR-26, CFG-10 to CFG-12, AC-10f |
| OQ-08 | Default Digest settings and security minimums | Digest defaults; password, failed-attempt, and session rules aligned with NIST SP 800-63B-4 | TD-05 | NFR-03, BR-23, CFG-13, CFG-15 to CFG-17, AC-02d, AC-14g |
| OQ-09 | Does an Action item's default due date follow a rescheduled or cancelled Meeting? | No; it changes only through an approved deadline change or extension | TD-06 | BR-17, AC-11d |
| OQ-10 | Open Action items when a student account is deactivated outside a Group removal | Deactivation not blocked; items unchanged; supervisors notified and may reassign | TD-07 | Roles, BR-21, BR-22, EC-34, AC-01d, AC-14d |
| OQ-11 | Dashboard order of Groups requiring attention | Delayed, then At Risk; by larger counter; then Group name | TD-08 | FR-07, CFG-14, AC-07b |
| OQ-12 | Which Meeting sets a meeting-relative Check-in deadline, and who chooses the Group's deadline setting? | Earliest non-cancelled Meeting in the cycle's period; global weekly deadline if none; set by an Assigned supervisor | TD-09 | Roles, Glossary, BR-07, EC-35, AC-06h, AC-06i |
| OQ-13 | When do majority confirmations for a new Representative expire, and can they be withdrawn? | Valid only within the current cycle until its deadline; withdrawable; recounted on membership change | TD-10 | BR-30.2, EC-36, AC-04h |

## 16. Traceability to the Validated Problem

| Need from Deliverable 1 | Goal | FRs | NFRs |
|---|---|---|---|
| P-1 Supervisor learns of a stalled Group only after a deadline is missed. | G-1, G-6 | FR-07, FR-08, FR-15 | NFR-02 |
| P-2 No single overview; updates arrive in different formats. | G-1, G-2 | FR-03, FR-04, FR-06, FR-07 | NFR-02 |
| P-3 Reporting must be lightweight (the spreadsheet was abandoned). | G-2 | FR-03, FR-04, FR-06 | NFR-02 |
| P-4 Supervisor needs prior feedback and open work before a Meeting. | G-4, G-5 | FR-11, FR-12, FR-13 | NFR-02 |
| P-5 Supervisor asked for a Monday overview of inactive Groups and overdue Action items. | G-6 | FR-15 | NFR-02 |
| Supporting: accountable follow-up, secure access, history | G-3, G-4, G-7 | FR-01, FR-02, FR-09, FR-10 | NFR-03, NFR-04 |
| **TA-1** Supervisor can review contributions without accidentally freezing them; locks are deliberate and auditable (supports P-2, P-3). | G-2, G-7 | FR-03, FR-05 | NFR-03, NFR-04 |
| **TA-2** Supervision continues when a Representative or supervisor is unavailable, without losing pending approvals, Missed Check-in history, or deadlines (supports P-1, P-4). | G-1, G-4, G-8 | FR-01, FR-04, FR-06, FR-15, FR-16 | NFR-03, NFR-04 |
| **Final revision (TD-01 to TD-13)** Every rule can be implemented and tested without guessing: deadlines, reminders, Dashboard order, Digest duplicates, file intake, security values and MFA recovery, deactivation, Representative appointment and confirmations, email default (supports P-1 to P-5). | G-1, G-2, G-4, G-6, G-7, G-8 | FR-01, FR-04, FR-06, FR-07, FR-10, FR-11, FR-14 | NFR-01, NFR-02, NFR-03 |


## 17. Client Confirmation Record (TA-1 and TA-2)

The decisions below were confirmed with the client, as required by the TA. This record covers only the TA-1 and TA-2 decisions; decisions made later by the project team under client-delegated authority are recorded separately in Section 19. Each is incorporated into the requirements, business rules, flows, edge cases, and acceptance criteria listed in the last column.

| ID | Question put to the client | Confirmed decision | Status | PRD items affected |
|---|---|---|---|---|
| CC-01 | Who may start a Check-in review that locks contributions? | Only an Assigned supervisor of that Group. Not students, the Representative, or the Administrator. | Confirmed | BR-28, Roles, AC-05e |
| CC-02 | Should opening the details page ever lock contributions, and should locking need a confirmation step? | Never on opening. Locking needs an explicit Start review action plus a confirmation showing who has not submitted and that the lock is permanent. | Confirmed | BR-02, BR-28, EC-03, AC-05a, AC-05f |
| CC-03 | May the supervisor start review before the deadline, and what happens to members who have not submitted? | Yes. Contributions lock; missing ones are recorded and notified; the Representative can still submit on time; no Missed Check-in; status unchanged. | Confirmed | BR-28, BR-04, EC-21, AC-05d |
| CC-04 | Can a lock be undone if the review was started by mistake? | No. The lock is permanent; a "review started in error" annotation is added instead. | Confirmed | BR-29, BR-06, AC-05i |
| CC-05 | How is a member who missed an early lock allowed to contribute afterwards? | A late contribution at any time, stored separately and linked to the missing record; the omission is preserved. | Confirmed | BR-29, AC-05h |
| CC-06 | Who may replace an unavailable Representative, before and after the deadline? | Before: the Representative's handover, a majority of members, or any Assigned supervisor. From the deadline: only an Assigned supervisor, with a reason. | Confirmed | BR-30, FR-04, AC-04c–e |
| CC-07 | Does replacing or losing a Representative excuse or change a Missed Check-in? | No. The entry is annotated but never removed or excused; late submission clears the active warning only. | Confirmed | BR-30, BR-05, AC-04f, AC-06g |
| CC-08 | Who may replace an unavailable supervisor, and may a Group ever be left without one? | The Administrator, or another Assigned supervisor of the Group. A Group always keeps at least one Assigned supervisor; the last one cannot be removed without a replacement. | Confirmed | BR-31, FR-01, AC-16a, AC-16b |
| CC-09 | What happens to approvals pending with the replaced supervisor? | They stay pending with original timestamps in the Group-level queue; any current Assigned supervisor can decide; half-entered decisions are not recorded; the new supervisor gets a Handover summary. | Confirmed | BR-31, BR-32, AC-16a, AC-16f |
| CC-10 | Do any existing deadlines change when a Representative or supervisor is replaced? | No. Check-in, Milestone, and Action-item deadlines, counters, and review periods are unchanged. | Confirmed | BR-32, FR-06, AC-04c, AC-16a, AC-16e |
| CC-11 | What if the only supervisor is unresponsive with a review pending? | Overdue reviews escalate to supervisors; after a further 3 calendar days (Administrator-configurable) the Administrator is notified to consider replacement. Group status does not change. | Confirmed | BR-16, BR-31, AC-16d |
| CC-12 | What happens to scheduled Meetings and notes of a replaced supervisor? | Meetings stay scheduled; notes stay as authored and any Assigned supervisor may edit them (audited); a meeting-relative Check-in deadline is unchanged unless the Meeting is later changed. | Confirmed | BR-31, AC-16e |

## 18. Demonstration Scenarios (normal case and failure/boundary case for each change)

All data below is sample data. Group "Alpha" has three members: **Member A (Representative)**, **Member B**, **Member C**, and supervisors **S1** and **S2** (S3 and S4 are other active supervisor accounts). The Check-in deadline is Sunday 11:59 PM.

### 18.1 TA-1, normal case: viewing does not lock; Start review locks deliberately (verifies AC-05a, AC-05d)

| Step | Action | Result |
|---|---|---|
| 1 | Friday: Members A and B submit contributions; Member C has not. | Contributions editable; no lock. |
| 2 | Saturday 10:00: S1 opens the Check-in details page and reads the contributions. | **No lock** (BR-02); no lock audit entry; Member C can still submit. |
| 3 | Saturday 10:05: S1 selects **Start review**. | Confirmation appears: "2 of 3 submitted; Member C has not; locking is permanent; Member C will be recorded as missing; the Representative can still submit." |
| 4 | S1 confirms at 10:06. | Contributions lock; audit entry (S1, 10:06, trigger "Start review", 2 locked, 1 missing); Member C is notified; Group members are told contributions are locked. |
| 5 | Saturday 20:00: Member A submits the Consolidated report using saved contributions. | On time; no Missed Check-in; Group status unchanged. |

### 18.2 TA-1, failure and boundary cases (verifies AC-05e, AC-05f, AC-05g, AC-05h, AC-05i)

| Case | Action | Result |
|---|---|---|
| Unauthorized | Member B (a student) tries to start review. | Refused; no lock (AC-05e). |
| Abandoned | S1 opens the confirmation and closes it. | No lock; everything still editable (AC-05f). |
| Time boundary | S1 opens the confirmation at Sunday 11:58 PM and confirms at 12:00 AM; no Consolidated report exists. | Confirmation refused: contributions already locked by the **deadline** at 11:59 PM; no second audit entry. The Missed Check-in is recorded separately at the deadline (AC-05g, BR-05). |
| Simultaneous | S1 and S2 confirm Start review at the same moment. | One lock and one audit entry; the second supervisor is told S1 locked first (AC-05g). |
| Correction | After 18.1, Member C submits a late contribution on Monday. | Stored separately and linked to the missing record; the missing record remains (AC-05h). |
| Mistake | S1 realizes the review was started too early. | Lock cannot be undone; S1 appends "review started in error"; Member C is notified (AC-05i). |

### 18.3 TA-2, normal case A: Representative replaced before the deadline (verifies AC-04c)

| Step | Action | Result |
|---|---|---|
| 1 | Thursday: Member A (Representative) falls ill; a draft exists. | No automatic change; the system does not detect unavailability. |
| 2 | Members B and C (a majority of 3) confirm Member B as Representative. | Replacement is immediate (BR-30). |
| 3 | Member B opens the Consolidated draft. | Draft and history are intact; Member A can no longer edit or submit; reminders for A are cancelled and generated for B; deadline unchanged. |
| 4 | Audit and notifications. | Audit shows initiator, time, old and new Representative; supervisors and all members are notified. |
| 5 | Saturday: Member B submits. | On time; no Missed Check-in. |

### 18.4 TA-2, failure and boundary cases for the Representative (verifies AC-04d, AC-04e, AC-04f, AC-04g, AC-06g)

| Case | Action | Result |
|---|---|---|
| Ineligible / unauthorized | A supervisor tries to appoint a Former member; one member alone tries to replace the Representative. | Both refused; Representative unchanged (AC-04d). |
| After the deadline | Sunday 11:59 PM passes with no report; **Missed Check-in** recorded. On Monday Member C (with B) tries to replace Member A. | Refused: after the deadline only a supervisor may replace (AC-04e). |
| Supervisor replaces, late report | S1 replaces A with C, giving a reason; C submits Monday 10:00. | Submission is late; active warning clears; the **Missed entry remains**; deadline not extended (AC-06g). |
| Vacancy at the deadline | Member A is removed (open items first reassigned) and nobody is appointed before the deadline. | Removal succeeds; role vacant; reminders go to B and C; at the deadline a Missed Check-in is recorded and annotated "Representative role vacant"; Group At Risk; Briefing lists the vacancy (AC-04f). |
| Simultaneous | A submits while S1 replaces A. | The first saved action stands; never two reports (AC-04g). |

### 18.5 TA-2, normal case B: supervisor replaced with pending work (verifies AC-16a)

| Step | Action | Result |
|---|---|---|
| 1 | Group Alpha has S1 and S2. S2 is leaving the university. Pending: a Deliverable awaiting approval (2 days), an Action item In Review (1 day), an extension request. Milestone deadline in 4 days; Check-in deadline in 3 days; one Missed Check-in in history. | State before replacement. |
| 2 | The Administrator (or S1) chooses Replace supervisor: S2 → S3. | S3 added and S2 removed in one action; Group never has zero supervisors. |
| 3 | S3 opens the Group. | Full history visible; Handover summary lists the pending items, the Missed Check-in, and the deadlines. |
| 4 | Check pending work and deadlines. | Items remain pending with **original timestamps**; review periods are **not reset**; Milestone and Check-in deadlines and the Missed entry are **unchanged**; Group status unchanged. |
| 5 | S3 approves the Deliverable. | Normal decision recorded and audited. |

### 18.6 TA-2, failure and boundary cases for supervisors (verifies AC-16b, AC-16c, AC-16d, AC-16f)

| Case | Action | Result |
|---|---|---|
| Last supervisor | Group Beta's only supervisor S1 is unreachable. The Administrator tries to deactivate S1 with no replacement. | Refused; affected Groups listed; Group Beta keeps S1 (AC-16b). |
| Invalid candidate / unauthorized | The Administrator picks a candidate who is not an active supervisor; a student tries to replace S1. | Both refused (AC-16b). |
| Valid replacement | The Administrator replaces S1 with S4. | Succeeds atomically (AC-16b). |
| Pending extension across the gap | A student's extension request (Thursday) is pending; the due date passes on Friday; on Saturday S4 approves a new date next Wednesday. | The item was overdue Friday (BR-19); on approval the revised and effective dates are stored, the overdue state clears, lateness history is kept (AC-16c). |
| Unresponsive supervisor | A review is undecided past its period. | Escalated to supervisors when overdue; Administrator notified after a further 3 days; Group status unchanged (AC-16d). |
| Mid-decision | S2 has a rejection form open when replaced, then submits. | Refused; nothing recorded; item stays In Review (AC-16f). |

## 19. Team Decision Record (final revision, under client-delegated authority)

**Authority.** The client has delegated requirements decision-making authority to the project team. The decisions below were finalized by the project team under that delegated authority on 2026-11-18. They are **team decisions**, not client confirmations: no individual client conversation or confirmation is claimed for them, and they are kept separate from the client-confirmed record in Section 17. Under C-06 this record is the approved, recorded change for each rule it touches; any later change needs a new or amended TD entry. TD-01 to TD-10 closed the open questions; TD-11 to TD-13 came from the final architecture review.

**Principles applied.** Every decision (1) preserves the confirmed requirements and the client decisions CC-01 to CC-12; (2) follows from an existing rule wherever one applies (cited in the Basis column); (3) adds no new capability beyond what the open question required; and (4) fits the accepted architecture (ADR-001 to ADR-005) without a new module, port, or dependency. The requirements-grilling log was not available while these decisions were made, so no decision cites a grilling answer it did not already cite; this PRD is the baseline (see the availability note in the header).

| ID | Resolves | Decision | Basis in existing requirements | Status | PRD items |
|---|---|---|---|---|---|
| TD-01 | OQ-01 | Acceptance-test profiles TP-01 (NFR-01) and TP-02 (NFR-02) as in Section 7.1. They are test conditions, not a capacity target. | C-05 (conditions approved jointly; the team holds the client's delegated authority); D-05 stays deferred | Finalized (team, delegated authority) | NFR-01, NFR-02, Section 7.1 |
| TD-02 | OQ-05 | An urgent event already delivered appears once in the next Digest as "already notified" history, never as a new alert; a history-only Digest is skipped. | BR-23 "urgent events may also appear as Digest history" [Q44]; "empty Digests are skipped" | Finalized (team, delegated authority) | BR-23, AC-14e |
| TD-03 | OQ-06 | The time zone and Blackout calendar are data the Administrator enters (no product default for the time zone; a Group cannot be activated until it is set). Milestone and Action-item reminders 48 h and 24 h before the deadline, to the roles in CFG-05 and CFG-06. | BR-07, BR-11, BR-12, C-04; Check-in reminder offsets from AC-06e; BR-13 suppression; BR-14 re-targeting | Finalized (team, delegated authority) | FR-14, BR-14, CFG-01 to CFG-06, AC-01e, AC-14f |
| TD-04 | OQ-07 | Allowed types .pdf, .docx, .pptx, .xlsx, .txt, .md, .png, .jpg, .jpeg; 25 MB per file; unused uploads cleaned up after 24 h. No archives, macro-enabled files, or executables while malware scanning is deferred. | BR-26, BR-09 (HTTPS URLs for larger artifacts), D-04 | Finalized (team, delegated authority) | BR-26, CFG-10 to CFG-12, AC-10f |
| TD-05 | OQ-08 | Digest default: enabled, 08:00, Monday to Friday, user's time zone. Passwords: minimum 15 characters without mandatory MFA, 8 with mandatory MFA (supervisors; Administrators added by TD-11); at least 64 accepted; no composition rules or periodic change; blocklist check. At most 100 consecutive failed attempts before the password is disabled. Sessions: 30 minutes inactivity, 12 hours total. | NFR-03 (supervisor MFA); NIST SP 800-63B-4 (password length 15 single-factor / 8 within MFA, ≥ 64 maximum, no composition or periodic change, blocklist, ≤ 100 failed attempts; AAL2 reauthentication inactivity ≤ 1 h, overall ≤ 24 h); BR-23 | Finalized (team, delegated authority) | NFR-03, CFG-13, CFG-15 to CFG-17, AC-02d, AC-14g |
| TD-06 | OQ-09 | An Action item's due date, once set, never follows a rescheduled or cancelled Meeting; it changes only through an approved deadline change or extension. | BR-17 (Meeting start used as default at creation), BR-20 (audited deadline changes), BR-32 (deadlines never change as a side effect) | Finalized (team, delegated authority) | BR-17, AC-11d |
| TD-07 | OQ-10 | Deactivating a student is never blocked by their Action items; the items stay unchanged and subject to overdue rules; the Group's Assigned supervisors are notified and may reassign them to an active current member. | BR-19, BR-21 (supervisor may reassign a departing student's items), BR-25 (history preserved), BR-32 (a change of who can act never changes what is recorded or due), Assumption 7 (Administrator cannot reassign items) | Finalized (team, delegated authority) | Roles, BR-21, BR-22, EC-34, AC-01d, AC-14d |
| TD-08 | OQ-11 | Dashboard order: Delayed, then At Risk; within each, larger inactivity counter first, then Group name; then the other Groups by name. | BR-10 and Assumption 1 (Delayed outranks At Risk); FR-07; NFR-02 | Finalized (team, delegated authority) | FR-07, CFG-14, AC-07b |
| TD-09 | OQ-12 (formerly architecture item G-20) | A Group uses the global weekly deadline unless an Assigned supervisor chooses a Group override or meeting-relative mode (from the next cycle). In meeting-relative mode the driving Meeting is the earliest non-cancelled Meeting starting within the cycle's period; with none, the global weekly deadline applies. Recalculation follows the existing BR-07 limits. | BR-07 (three deadline types; changes apply to future cycles; recalculation limits), AC-06a, AC-06f, AC-16e; BR-08 and BR-16 precedent (Administrator default, supervisor Group override); Assumption 7 | Finalized (team, delegated authority) | Roles, Glossary, BR-07, EC-35, AC-06h, AC-06i |
| TD-10 | OQ-13 (formerly architecture item G-19) | One withdrawable confirmation per active current member for one candidate; valid only in the current cycle until its deadline; never carried over; discarded for Former or deactivated members; a membership change never completes a replacement by itself; cleared when the Representative changes; each confirmation recorded. | BR-30.2 (majority of current members), BR-30.3 (members cannot replace from the deadline), BR-30.4 (eligibility), BR-30.5 (no pending replacement), BR-25 | Finalized (team, delegated authority) | BR-30.2, EC-36, AC-04h |
| TD-11 | Final architecture review | MFA mandatory for Administrators as well as supervisors. Recovery: 10 single-use recovery codes; Administrator-assisted reset of another user's enrolment (never one's own); a server-side operator procedure only when no other Administrator can act; emailed single-use password-reset link with MFA still required; the last active Administrator cannot be deactivated. CFG-15: Administrators move to the 8-character minimum because MFA is now mandatory for them. | NFR-03 (MFA for supervisors is a minimum, not a limit); ADR-003 §1 ("optional for other roles unless policy changes"); the Administrator's account-wide powers (Section 4, BR-31); NIST SP 800-63B-4 | Finalized (team, delegated authority) | Roles, NFR-03, BR-33, CFG-15, CFG-17, EC-38, AC-02d, AC-02e |
| TD-12 | Final architecture review | A Group may be created without a Representative; before activation only an Assigned supervisor appoints (and may change) an eligible Representative, audited, no reason required; activation is refused unless the appointed Representative is still eligible; after activation BR-30 applies unchanged. | FR-01 (activation after setup), BR-30.4 (eligibility), BR-30.2 to BR-30.10 unchanged after activation, Assumption 7 (the Administrator has no Group-level appointment right) | Finalized (team, delegated authority) | Roles, FR-01, F1, BR-30.11, EC-37, AC-01c, AC-01f |
| TD-13 | Final architecture review | Email notifications are on by default, globally and for each user; users may turn email off; the Administrator may turn it off for everyone; urgent in-app notifications always stay on. | BR-22 ("users may disable email"), EC-15, NFR-04 | Finalized (team, delegated authority) | BR-22, CFG-18, AC-14h |
