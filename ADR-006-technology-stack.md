# ADR-006: Technology Stack and Implementation Conventions

**Status:** Accepted
**Date:** 2026-11-18
**Deciders:** Project team (all members)
**Related:** ADR-001 (modular monolith), ADR-002 (ACID database, locks, audit, outbox), ADR-003 (sessions, MFA, guard; amended by TD-11), ADR-004 (Clock, jobs, outbox), ADR-005 (file storage). Closes `ARCHITECTURE.md` §9.5 items G-12, G-14, G-16, G-18, G-21, G-22, G-23 and G-24.

## Context

- **C-03** left the technology stack to a separate decision. ADR-001 to ADR-005 fix the *mechanisms* the stack must support:
  - one deployable **modular monolith** of 11 modules with enforceable boundaries (ADR-001);
  - one **ACID relational database** with exclusive and shared row locks, unique constraints, conditional inserts and version checks; a Unit-of-Work audit hook; an outbox row in the same commit (ADR-002);
  - **server-side sessions** in an opaque, secure, HTTP-only cookie; **mandatory MFA for supervisors and Administrators** with recovery (ADR-003, TD-11, PRD BR-33); a central deny-by-default guard re-checked inside transactions;
  - an **injectable Clock**, a **database-backed job runner** with idempotency keys, and a **transactional outbox** with an Email port (ADR-004);
  - **private storage** behind a Storage port, with server-side content-type detection (ADR-005).
- **C-01 / C-02:** a responsive web client for the latest two versions of Chrome, Edge, Firefox and Safari.
- A small student team working for one semester. Development is assisted by Claude Code under `AGENTS.md`, so the architecture rules should be **machine-checked** wherever practical.

## Alternatives considered

1. **Django + Django REST Framework + PostgreSQL; React + TypeScript + Vite + Tailwind CSS** (chosen).
2. **NestJS (TypeScript end to end) + PostgreSQL.** One language, but authentication, sessions, MFA, migrations and CSRF must be assembled from separate packages, which means more security glue code to own.
3. **Spring Boot (+ Spring Modulith) + PostgreSQL.** Excellent module enforcement, but heavier and less familiar to the team.
4. **Django templates + HTMX instead of a React client.** Simpler (one stack, no separate client build), but weaker for the interactive Dashboard and Briefing, and less familiar to the team.
5. **Database alternatives:** MySQL (weaker fit for the locking and constraint features we rely on) and SQLite (no concurrent row locking). Rejected.
6. **Job-runner alternatives:** Celery with Redis (adds a message broker, which ADR-004 rejected) and third-party database job libraries (they own their own tables, which conflicts with Calendar & Scheduling owning `ScheduledJob`). Rejected.

## Decision

1. **Backend.** Python 3.12 or later, **Django 5.2 LTS** and **Django REST Framework**. One Django project; **each of the 11 modules is one Django app**. Exact versions are pinned in lock files when the repository is created.
2. **Database.** **PostgreSQL 16 or later**, at the default READ COMMITTED isolation level, combined with the locking mechanisms of ADR-002.
3. **Frontend.** **React + TypeScript**, built with **Vite**; **React Router**; **TanStack Query** for server state; **Tailwind CSS** for responsive styling. The built client is served from the same origin as the API (`/api/...`), so session cookies and CSRF protection need no cross-origin configuration.
4. **Authentication (ADR-003, TD-11).**
   - Django authentication with **database-backed sessions** (opaque, `Secure`, `HttpOnly`, `SameSite` cookie). Permissions are never stored in the session.
   - **django-allauth** (`account`, `mfa`, `headless`): TOTP authenticator-app MFA, **mandatory for supervisors and Administrators**; 10 single-use recovery codes; emailed single-use password-reset links; login rate limiting.
   - Additional code in Identity & Access, all audited:
     - Administrator-assisted MFA reset of another user;
     - an operator management command for the case where no other Administrator can act (PRD BR-33(d));
     - the CFG-16 rule that disables a password after 100 consecutive failures;
     - the "last active Administrator" guard on deactivation.
   - Password validation follows PRD CFG-15: a minimum length that depends on the role, and Django's common-password blocklist validator. Django's numeric-only validator is **removed**, because NIST SP 800-63B-4 forbids composition rules.
   - Idle and absolute session limits (CFG-17) are enforced by middleware that reads `SecuritySettings`.
5. **Authorization.** The guard is a DRF permission class set as the **default for every view**. It denies any request whose view has not declared a named policy registered in Identity & Access. No third-party permission framework is used.
6. **Unit of Work and audit (closes G-12).** Platform code wraps `transaction.atomic()`. Module repositories register each audited change with the Unit of Work: entity, Group ID (or none for account-level changes), changed fields, previous values, actor, and reason or trigger. The Audit hook writes `AuditEntry` rows just before commit, and domain events are published through `transaction.on_commit`. Django signals, `QuerySet.update()` and `bulk_*` writes are **not used on audited models**.
7. **Locks.** Exclusive locks use `select_for_update()`. Because the ORM has no `FOR SHARE`, `lockGroupShared` and shared `lockUserAccount` use **one platform helper** that runs `SELECT ... FOR SHARE`. Conditional lock inserts rely on unique constraints, using `INSERT ... ON CONFLICT DO NOTHING` or catching `IntegrityError` inside a savepoint.
8. **Jobs and outbox (closes G-14).**
   - A **custom worker** (`manage.py run_worker`) in the same codebase and deployable, run as a **separate process**.
   - It claims due `ScheduledJob` and unsent `OutboxMessage` rows with `SELECT ... FOR UPDATE SKIP LOCKED`, runs the registered handler in its own Unit of Work, and polls on a timer, so no wake-up event is required for correctness.
   - Poll interval, retry count and backoff are **environment configuration**.
   - Recurring work (cycle opening, the Monday Briefing) reschedules its own next job, as `ARCHITECTURE.md` §6.8 describes.
9. **Ports and adapters.**
   - **Email:** a Django email backend (console in development, SMTP in deployment).
   - **Storage:** the Django storage API (`FileSystemStorage` in development; **django-storages** with a private, S3-compatible bucket in deployment).
   - **Content-type detection:** **python-magic** (libmagic).
   - **Clock:** a Calendar & Scheduling adapter (system clock in production, controllable clock in tests).
   - The SMTP service and the storage bucket provider are chosen at deployment and do not affect code.
10. **Persistence conventions (closes G-22).**
    - **No foreign keys between modules.** A reference to another module is a plain ID field.
    - Each module's migrations live in its own app and touch only its own tables. With no foreign keys between modules, migrations need no dependencies across modules.
    - Business timestamps never use `auto_now` or `auto_now_add`; they are set from `Calendar.now()`.
    - Settings use `USE_TZ = True` with UTC. The Official academic time zone is stored in `CalendarSettings`, not in Django settings.
11. **Django admin** is **not used for business data**, because it bypasses the guard, the Unit of Work and audit. It is used only for read-only inspection in development, if at all.
12. **Time-zone scheduling (closes G-21).** Calendar & Scheduling computes the next Digest instant for a user's time zone with Python `zoneinfo`; Notifications does no date arithmetic.
13. **Inactivity alerts (closes G-16).** The alert job uses the idempotency key `inactivity:{groupId}:{counter}:{lastValidSubmissionId}` (or `activation` when there is no submission yet). This gives at most one alert per counter per inactive spell and needs no stored counter. The job runs daily and after every Blackout change.
14. **Enforcement and testing (closes G-24).**
    - **import-linter** contracts: layers L4 → L0; no imports between modules in the same layer except Action Items → Meetings; no imports of another module's `internal`; `platform_core` and `shared_kernel` import no module.
    - **Ruff** banned-API rules: `datetime.now`, `datetime.utcnow`, `django.utils.timezone.now` and `time.time` are not allowed outside the Calendar Clock adapter.
    - A pytest architecture test rejects foreign keys between modules and checks that each module's migrations touch only its own tables.
    - **pytest + pytest-django + factory_boy** for the server; **Vitest + React Testing Library** for the client; **Playwright** for selected acceptance criteria on Chromium, Firefox and WebKit; **Locust** for the TP-01 load test.
    - **GitHub Actions** runs Ruff, import-linter, pytest, `tsc`, ESLint and Vitest on every pull request; `main` is protected.
15. **Client conventions (closes G-23).** The client makes no security decisions and does no date arithmetic. Timestamps arrive as ISO 8601 UTC and are formatted with `Intl.DateTimeFormat`. API types are generated from the **drf-spectacular** OpenAPI schema with **openapi-typescript**.
16. **Repository layout** (Python package names avoid the standard-library names `platform` and `calendar`):
    ```
    /client/                      Vite + React + TypeScript + Tailwind
    /server/config/               Django settings, URLs, WSGI
    /server/shared_kernel/        typed outcomes, IDs, actor, UoW and event contracts
    /server/platform_core/        Unit of Work, audit-hook runner, FOR SHARE helper, on-commit events, worker loop
    /server/api/                  DRF views, guard permission class, outcome mapping, application services for L0/L1 commands
    /server/bootstrap/            composition root: binds the six ports, registers policies, job handlers, audit hook
    /server/modules/<module>/     one Django app per module: public/, internal/ (models, services, policies), adapters/, migrations/
        identity_access  groups_roles  checkins  milestones_deliverables  action_items  meetings
        calendar_scheduling  notifications  audit  files  dashboard_briefing
    /server/tests/                support/, architecture/, access/, acceptance/
    ```
17. **Delivery.** **Docker Compose** for local development (PostgreSQL, app, worker); **Gunicorn** in deployment; the app and the worker run as two processes from one image. **Git + GitHub** with protected `main`. The hosting provider is chosen before acceptance testing (TP-01 staging environment; D-03 backups) and does not affect code.

## Rationale

- Django's apps, migrations per app, sessions, CSRF protection, transactions and row locks map directly onto ADR-001 to ADR-003, so most of what the architecture needs comes with the framework.
- django-allauth gives vetted MFA, recovery codes, password reset and rate limiting (ADR-003: "vetted libraries, never homemade cryptography").
- PostgreSQL supports every concurrency mechanism in ADR-002 natively: `FOR SHARE`, `SKIP LOCKED` and partial unique indexes.
- A custom worker over the tables the architecture already defines keeps ADR-004 unchanged and adds no broker.
- import-linter, Ruff and architecture tests turn the `AGENTS.md` rules into checks that fail the build, which matters for AI-assisted development.

## Trade-offs / consequences

**Positive**
- A mature, well-documented stack with little infrastructure: PostgreSQL plus one application image run as two processes.
- The architecture rules are checked automatically in CI.
- The modular-monolith design and all earlier ADR decisions are unchanged; this ADR only names the products and conventions.

**Negative / risks**
- Two languages (Python and TypeScript). Front-end and back-end types are kept in sync by code generation.
- Django's usual habits (foreign keys between apps, signals, the admin, `ModelSerializer.save()`, `auto_now`) conflict with the module, audit and Clock rules. The conventions above must be enforced in review and CI.
- The custom worker and the MFA-recovery additions are code we own and must test: idempotency, retry, crash recovery and refusal cases (`AGENTS.md` §13).
- On acceptance, `AGENTS.md` §3 and §12 and `ARCHITECTURE.md` §9.5 and §11 are updated to match this ADR.
