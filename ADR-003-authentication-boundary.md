# ADR-003: Authentication, Authorization and Immediate Revocation Boundary

**Status:** Accepted; amended 2026-11-18 (Amendment 1: MFA for Administrators, TD-11)
**Date:** 2026-11-14
**Deciders:** Project team (all members)
**Related:** ADR-001 (Identity & Access module), ADR-002 (re-check inside write transactions), ADR-005 (file download authorization)

## Context

- **ASR-3:** permissions depend on four things at once:
  - **Role:** Student, Representative, Assigned supervisor or Administrator.
  - **Group membership:** users act only on their own Group's records; Former members are denied (BR-21, EC-13).
  - **Record state:** for example, Start review only while the Check-in is unlocked (BR-28).
  - **Time:** for example, members may replace the Representative only before the deadline (BR-30).

  The Administrator can replace supervisors and override membership, but cannot start a review or approve anything (Assumption 7).
- **NFR-03:** in a scripted access test, 100% of the following must be denied: reading another Group's records, a student reading audit history, starting a review without being an Assigned supervisor, and unauthorized replacement. **Supervisor login requires multi-factor authentication (MFA).**
- **Immediate revocation:** a removed or replaced user loses access at once. A replaced supervisor's in-progress decision must be refused (AC-16b).
- **C-03:** no identity provider is fixed. The institution may later want single sign-on (SSO).
- **OQ-08:** password and session minimums are still open.

We need an authentication boundary and an authorization approach that meet these requirements.

## Alternatives considered

1. **Central permission layer with server-side sessions.** Every request checks permissions against the current database state.
2. **Stateless long-lived tokens (e.g. JWT)** carrying role and Group claims.
3. **Short-lived tokens for login only,** with the permission check of option 1 on every request.
4. **Checks scattered across screens and controllers,** plus hiding buttons in the UI.

## Decision

1. **Authentication**
   - The system has **built-in authentication** inside the Identity & Access module, placed behind an **Identity Provider port**. Institutional SSO (for example, OIDC or SAML) can be added later as another implementation of that port, without changing other modules.
   - **MFA is mandatory for users with the supervisor role.** It is optional for other roles unless policy changes. *(Amended: also mandatory for Administrators; see Amendment 1.)*
   - Password and session limits (OQ-08) are **configuration values**, not hard-coded.
2. **Sessions**
   - Authenticated users get a **server-side session**. The browser holds only an opaque session identifier in a secure, HTTP-only cookie sent only over HTTPS.
   - Sessions can be ended on the server at any time, for example when an account is deactivated or a user signs out.
   - Sessions carry **identity only, never permissions.**
3. **Central policy-as-code guard**
   - All authorization goes through **one guard in Identity & Access**. Each protected action has a named policy written as code, for example `checkin.startReview`, `group.replaceRepresentative`, `group.replaceSupervisor` and `audit.read`.
   - A policy checks role, **current** Group membership from Groups & Roles, record state, and time from Calendar & Scheduling.
   - The guard **denies by default.** An endpoint with no registered policy is refused.
   - It runs on **every API request**, including file downloads (ADR-005).
4. **Re-checking inside write transactions**
   - For every write, the policy is **checked again inside the database transaction** that performs the change (ADR-002).
   - So a user whose role was removed between loading a form and submitting it is refused, even though the earlier check passed.
5. **Client role**
   - The web client may hide controls for convenience, but it is **never** relied on for security.

## Rationale

- **Immediate revocation by design.** Permissions are never stored in a session or token. Every request and every write reads current membership and roles, so a removal or replacement applies to the very next action. Long-lived tokens would keep stale claims until they expire, which breaks ASR-3.
- **One place to test.** NFR-03 demands 100% denial. Putting every rule in one deny-by-default guard means the scripted access test targets a single component, and a forgotten endpoint fails safely. Scattered checks cannot give that guarantee.
- **C-06 compliance.** Permission rules are confirmed rules that may change only through approved changes. Keeping them as named policies in one module makes each change visible and reviewable.
- **Sessions over short-lived tokens.** The system has one web client. Sessions give simple server-side revocation and need no refresh-token handling, which suits a team building one client.
- **Built-in login behind a port** lets the team deliver MFA now without depending on the institution, while leaving SSO open (C-03).

## Trade-offs / consequences

**Positive**
- Meets NFR-03, immediate revocation and AC-16b with one consistent mechanism.
- Adding SSO later changes only the Identity Provider adapter.
- Every permission rule is easy to locate, review and test.

**Negative / risks**
- Each request needs a session lookup plus membership and role queries. This is small, but it adds to the NFR-01 budget. **Mitigation:** caching is allowed only within a single request, never across requests.
- The guard is a critical, high-risk component. Every policy needs positive and negative tests, including Former member, replaced supervisor, Administrator trying to approve, and members acting after the deadline.
- A session store must be operated and protected.
- Building authentication and MFA ourselves means owning password storage, MFA enrolment and recovery, and protection against brute-force login. These must follow established security practice, using vetted libraries, never homemade cryptography.
- Developers, including the AI agent, must register a policy for every new endpoint. AGENTS.md must state this, and deny-by-default ensures a forgotten registration fails closed.

## Amendment 1 (2026-11-18): MFA for Administrators and MFA recovery

**Decision source:** PRD TD-11 (team decision under client-delegated authority), applying the "unless policy changes" clause of Decision 1.

- **MFA is mandatory for Administrators as well as supervisors.** The Administrator can deactivate accounts, replace any supervisor and change deadlines for every Group, so a password alone is not enough protection. MFA stays optional for students.
- **Recovery (PRD BR-33):** 10 single-use recovery codes at enrolment; an Administrator may reset another user's enrolment (never their own); a server-side operator procedure is used only when no other Administrator can act; password reset by an emailed single-use link, with MFA still required afterwards. The last active Administrator cannot be deactivated. Every enrolment, reset, recovery-code use and password reset is audited.
- **Unchanged:** built-in authentication behind the Identity Provider port, server-side sessions, the deny-by-default guard and the re-check inside transactions. The products that implement this amendment are named in ADR-006.
- **Consequence:** the "last active Administrator" check must lock the active Administrator `UserAccount` rows (in ascending ID order, before any `Group` row), so that two Administrators deactivating each other at the same time cannot leave the system with none.
