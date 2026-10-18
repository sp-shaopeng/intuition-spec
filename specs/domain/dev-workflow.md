# InTuition — Module Development Workflow

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Frontend branch threshold is 88%; backend remains 97.5% line and branch. Requirements changes must propagate through design, tasks, acceptance and references. See [current requirements](../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.

**Document Version:** 1.1  
**Date:** 2026-05-27  
**Status:** Active — applies to all module development and changes  
**Authority:** This document overrides any conflicting guidance in individual module specs

---

## Overview

Every module — whether built from scratch or changed — follows this 8-phase workflow. **No phase may be skipped.** The gates marked 🔴 are hard blockers: the next phase cannot begin until the gate is cleared.


The illustration for overview contains the following information:

- Phase 1: Plan.
- Phase 2: application programming interface Contract Review 🔴 All domains agree before any code is written.
- Phase 3: Database Migration 🔴 Liquibase changeset reviewed, rollback verified.
- Phase 4: Backend Development 🔴 97.5% coverage, all tests green.
- Phase 5: Frontend Development 🔴 97.5% coverage, all tests green.
- Phase 6: Integration & Postman 🔴 Pact green, Postman collection updated.
- Phase 7: Dev Ops Validation 🔴 Docker build succeeds, deployment files valid.
- Phase 8: Team Review 🔴 /team-review run, all Group A/B issues fixed.


> **Team review is ALWAYS the full 18-agent 5-round debate. There is no quick mode** (founder decree, 2026-05-30) — this supersedes any "quick /team-review" phrasing elsewhere in this document.

---

## Workflow Amendments — Adopted 2026-05-30

Team-reviewed (full 18-agent debate) + Boss-cleared (ADOPT with conditions). These four amendments are **binding** and take precedence over older conflicting text below until the body sections are next rewritten. Each was earned by a concrete failure in the M-Reg admin-lab session — none is speculative.

### A1. Automated pre-review gate (lab-first)

Before the **Phase 8 Team Review** is entered, an **automated pre-review gate** must be green:


The illustration for a1. automated pre-review gate lab-first contains the following information:

- spotless:apply → build → regression lab (DEV stack) → coverage ≥ 97.5%.


- If the gate is **RED, fix before any review agent is spawned.** Mechanical/contract bugs die here cheaply, so the panel spends its expensive reasoning on design and risk, not on defects a run would have caught.
- The gate's result is **handed to the panel as evidence** — Reese/Seb cite real runs, not guesses.
- The hand-off **must include an explicit NOT COVERED block** (e g. owed security negatives like stale-signed session token bypass; no load/concurrency; single-stack so no real parallelism). *Green is never handed over alone* — green + known-gaps, so a passing gate never lulls the panel into rubber-stamping.
- **Terminology:** this is an *automated pre-review gate*, **not** a "deterministic" gate. The lab is black-box against one shared dev stack and is **order/load-sensitive** (documented 429 rate-limiter cascades, connection-pool exhaustion, a warm-up race). The durable fix (in-process fresh-database Testcontainers layer) was **de-scoped by the founder** ("don't need fresh db, dev anyway"), so the substrate stays non-deterministic and the flake protocol below is the only mitigation.

**Flake protocol (mandatory before a gating run):**
1. Reset rate-limit state: the documented operational check and update records users SET failed login attempts is 0, locked until is no value.
2. On an **environmental** RED (429 cascade / connection-pool exhaustion — the documented signatures only), **one automatic retry**.
3. Only a **second** RED blocks the pipeline and escalates. A product-failure RED (assertion mismatch, 4xx/5xx contract break) blocks immediately — no retry.

**Status:** lab-first is **local discipline today**; it becomes a true enforced gate once the owed GitHub Actions continuous integration run exists (ROADMAP #42 / #39). **The lab gate is NOT a substitute for production observability** — runtime detection (e g. the admin 5xx alert) remains separately owed.

**Coupled requirement (Should):** because the gate now runs on every change, a **band-scoped purge** of reg-band data (*@regression local precursors) must exist so the dev database does not bloat into unusability for manual quality assurance.

### A2. Edit→verify reflex

After **any** edit to a **registration / config / wiring** file — Liquibase master-changelog include entry lines, route maps, DI wiring, env files — **immediately grep for the inserted token before building.** This extends *Boss-verify-before-proceed* to the agent's **own** edits, not only Boss-mandated fixes. (Earned: two silent-failed edits in the M-Reg session — a master-changelog include entry and a docstring — cost a boot crash-loop and a near-false-green report.)

### A3. Definition of Done ends at dev — with a non-deletable prod obligation

- **DONE-DEV** = regression lab green on the **dev** stack + team review cleared. The regression lab's scope is **dev/sim database only** — it never requires prod access.
- **BUT** any change that fixes or touches a **prod-affecting defect** must *also* create a **PROD-DEPLOY-OWED** entry in ROADMAP md. The entry carries: the change, the **migrate-before-boot ordering** (mandatory under ddl-auto is validate), the **prod schema-check command**, and a **named owner**. A PROD-DEPLOY-OWED entry **cannot be deleted — only moved to DEPLOYED** after the founder confirms.
- **Review cadence:** the PROD-DEPLOY-OWED register is **opened and reconciled the moment prod access is next available** — it is the first thing checked, so a dev-validated fix for a live prod bug can never silently rot on dev.

### A4. Schema-change coupling rule

An entity change that touches a column **ships as ONE unit** with: (a) its Liquibase changeset, **and** (b) a **verified** master-changelog include entry (grep the include per A2). Rationale: under ddl-auto is validate, an entity whose column Definition/@Column disagrees with the live schema **crash-loops the app on boot** (exactly the M-Reg-session failure). Rollback-safety must be **reasoned per migration**, not assumed (e g. a text→jsonb USING … jsonb rollback only works if all rows are valid).

**Backstop (Should):** a continuous integration check that fails the build when an entity column has **no corresponding changelog column** — checking *column existence in the changelog*, not naive column Definition string-matching (so the check is not itself flaky). Until it lands, A4's one-unit + grep discipline is the interim guard.

### Accepted risks (eyes open)

- **Always-full-debate × lab-first compounds per-change cost.** Every change now needs a full lab run *and* a full 18-agent debate. This raises fixed cost per change and predictably incentivises **batching** (accumulating changes to amortise review). The founder's always-full-debate decree stands; **tripwire:** if change-batching emerges to dodge review overhead, flag it to the founder to reconsider.
- **Gate on a non-deterministic substrate** (founder de-scoped the fresh-database fix) — mitigated by the flake protocol, not eliminated.
- **Build-time gate ≠ runtime detection** — the admin 5xx alert and prod observability remain separately owed; this amendment must not be used as an excuse to keep deferring them.

---

## Phase 1: Plan

**Before writing a single line of code or changing any file:**

1. Read the full module spec set (spec md, frontend md, backend md, db md)
2. Write a plan — either as a reply in the session or as a plan file — covering:
   - What is being built or changed (scope)
   - What existing behaviour is affected (blast radius)
   - What acceptance criteria must be satisfied
   - Estimated order of implementation (database → backend → frontend)
3. The plan must be reviewed and approved before proceeding
4. For changes to existing modules: identify all downstream modules affected by the change (e g., changing an application programming interface response shape may break Interview, Matching, and Commission modules)

**Principle:** No surprises mid-implementation. If the approach cannot be fully articulated before coding starts, more spec reading is required.

---

## Phase 2: application programming interface Contract Review

**The application programming interface contract is the source of truth for the SPA ↔ Spring Boot boundary. All domains must agree before any code is written.**

### 2.1 Contract Definition

For every endpoint in the module:

1. Write or update the endpoint's entry in domain/the private implementation folder:
   - web request protocol method + path
   - Auth requirement
   - Full request body schema (field names, types, constraints)
   - Full response body schema (all fields, types, nullable/required markers)
   - All error codes with web request protocol status and meaning — enumerate every code string value (e g., INVALID CREDENTIALS, EMAIL ALREADY REGISTERED); no error code may appear in a production response that is not listed in the contract
   - Page Response envelope and zero-based page/size contract

2. For changed endpoints: explicitly diff the old and new contract. Any breaking change (removed field, type change, renamed field, status code change) requires a version bump or a migration plan.

### 2.2 Cross-Domain Agreement

Run a targeted /team-review on the application programming interface contract covering these agents:
- **Riley (frontend-facing service):** payload shape, pagination, versioning
- **Sam (Frontend):** does the React SPA have everything it needs?
- **Casey (Security):** auth requirements, sensitive field exposure
- **Morgan (DBA):** can the backend produce this efficiently without N+1?
- **Reese (quality assurance):** are all error cases covered?

**🔴 Gate:** Contract is locked only when all five agents have no Group A objections. Once locked, no field may be added, removed, or renamed without repeating this review.

### 2.3 Pact Contract Stub

Before backend implementation begins, write the Pact consumer test stubs in the React SPA:
- These define the expected request/response shapes in runnable TypeScript
- They act as the living contract that the Spring Boot provider must satisfy
- Commit the Pact file to the Pact Broker before backend work starts
- **Must include at least one interaction per 4xx variant** — if the endpoint can return 400, 401, 409, and 423, write an interaction for each; do not write only the happy path
- **Rate-limited endpoints must include a 429 interaction** documenting the retry After response shape
- Pact files with only 2xx interactions will be rejected at Phase 2 gate review

### 2.4 Postman Collection Update

Export each application programming interface endpoint to the shared Postman collection (In Tuition postman collection implementation reference at the repo root):
- One folder per module
- One request per endpoint, with example request body and expected response
- Environment variables for the configured application address, the protected credential for that account role, the protected credential for that account role, the protected credential for that account role
- Auth pre-request script: POST to auth login interface operation and store the session cookie
- Tests tab: assert response status code + at minimum one field from the response body

The Postman collection is committed to the repository and kept up to date with every application programming interface change.

---

## Phase 3: Database Migration

**Tool: Liquibase**

InTuition uses Liquibase for all database schema changes. Hibernate ddl-auto is set to validate in all environments — Hibernate never modifies the schema.

### 3.1 Changeset Rules

1. Every schema change is a new Liquibase changeset — file: VXXX description implementation reference (XML) or  implementation format implementation reference. **Never  implementation format implementation reference bare files** — Liquibase ignores unregistered database query language files silently.
2. Changesets are immutable once merged to main — never edit a merged changeset
3. Every changeset that creates or modifies a table must have a the rollback instructions block
4. Partial indexes, custom ENUMs, triggers, and check constraints are written as Liquibase SQL block tags (not create Table — Liquibase's standard tags don't support PostgreSQL-specific syntax)
5. **ENUM ADD VALUE rollback policy:** PostgreSQL ENUM ADD VALUE is non-transactional and cannot be rolled back inside a transaction block. For any changeset that adds an ENUM value, the the rollback instructions block must be a documented no-op with a comment: . Do not write a SQL block rollback that will fail at runtime.
6. **database structure definitions lock timeout:** All changesets that perform database structure definitions on live tables must include a three-second limit on waiting for a database lock as the first statement. Prevents indefinite blocking on long-running queries during rolling deploys.
7. Run liquibase validate in continuous integration — build fails if changelog XML is malformed or changeset IDs are duplicated
8. **Migration dry-run in continuous integration (required — not just local):** continuous integration must spin up a PostgreSQL 16 container and run mvnw liquibase implementation reference against it. liquibase validate alone only checks XML syntax; it does not verify the database query language executes. A changeset with valid XML but broken database query language will silently pass validate and fail in production.

### 3.2 Local Validation

Before committing a changeset:


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.3 Changeset Review

Run a quick /team-review focused on Morgan (DBA) for every non-trivial changeset:
- Correct index coverage for all query patterns
- Partial index syntax correct
- Rollback script complete and tested
- ENUM changes are additive (never remove an ENUM value without a migration plan)

**🔴 Gate:** Changeset applied cleanly locally, rollback tested, DBA review passed.

---

## Phase 4: Backend Development

**Language:** Java 21 + Spring Boot 3.x  
**Test framework:** JUnit 5 + Mockito + Testcontainers (PostgreSQL 16)  
**Minimum coverage:** 97.5% (line + branch)

### 4.1 Implementation Order


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.2 Unit Tests (JUnit 5 + Mockito)

Every the service component class has a companion *Service Test:
- All public methods tested
- All guard branches tested (both paths: pass and fail)
- All state machine transitions tested (valid + invalid)
- All error codes tested
- Zero Spring context: pure Mockito @Mock and @Inject Mocks

### 4.3 Repository Tests (@Data Jpa Test + Testcontainers)

Every @Repository with a custom @Query has a companion *Repository Test:
- Uses a real PostgreSQL 16 container (not H2)
- Tests all custom queries with real data
- Tests partial indexes (concurrent insert tests for concurrency guards)
- Tests soft-delete filters (@Where)

### 4.4 Structured Logging (logging context Correlation IDs)

Every incoming web request protocol request must be assigned a X-Correlation-ID header (generated by the server if absent) and propagated to every log line via logging context:


Implementation example omitted. This public edition retains the written requirements in this section.


Log pattern must include the request’s correlation identifier in the log entry. This is required — not optional — before Phase 4 gate clears.

**Gate check:** *Controller Test must include one test asserting the X-Correlation-ID header appears in the response and that log output contains the same ID.

### 4.5 Controller / Integration Tests (@Spring Boot Test)

Every controller has a companion *Controller Test using Mock Mvc:
- Tests the full filter chain (signed session token validation, role guards, rate limits)
- Tests every endpoint's happy path
- Tests every 4xx/5xx error response — assert specific code field value in response body (not just web request protocol status)
- Tests contact field privacy: asserts phone/email are absent from public/pre-assignment views and available only to authorized assigned participants under match-state gates
- Tests X-Correlation-ID propagation (see §4.4)

### 4.5 Coverage Enforcement


Implementation example omitted. This public edition retains the written requirements in this section.


Build fails if coverage drops below 97.5%. No @Exclude From Jacoco Generated Report without explicit justification in the pull request.

**pull request test assertion requirement:** Every test method must contain at least one assertion on a specific field value — not just the assert not no value operation or the verify operation. A test that renders a component or calls a service without asserting the output is coverage padding, not a test. Enforce via pull request review checklist.

**🔴 Gate:** All tests pass, coverage ≥ 97.5%, zero ignored/skipped tests without justification.

---

## Phase 5: Frontend Development

**Language:** React 18 + TypeScript (Vite)  
**Test framework:** Vitest + React Testing Library  
**Minimum coverage:** 97.5% lines and 88% branches (enforced by Vitest coverage thresholds)

### 5.1 Implementation Order


The illustration for implementation order contains the following information:

- Pact consumer tests (Phase 2.3 — already written).
- → TypeScript types / Zod schemas.
- → Tan Stack Query hooks (use Query / use Mutation per endpoint).
- → Zustand state updates (if any new user interface state).
- → Page components (route-level).
- → Shared components (reusable user interface elements).
- → Form validation with React Hook Form Zod.


### 5.2 Component Tests (Vitest + RTL)

Every component has a companion *.test implementation reference:
- Happy path: renders correctly with valid props/state
- Loading state: renders skeleton / spinner
- Error state: renders error message
- Empty state: renders empty state user interface
- Form validation: all invalid inputs show correct error messages
- Accessibility: key interactive elements have correct ARIA attributes

### 5.3 Coverage Enforcement


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.3 user experience Design Sign-Off (end of Phase 5, before coverage gate)

Before marking the frontend gate as cleared, complete this 15-minute checklist:

- [ ] All new/changed components visually reviewed against approved wireframes in domain/uiux/wireframes md
- [ ] All user-facing copy matches the spec's copy deck (button labels, headings, error messages, empty states)
- [ ] Mobile layout (390px) and desktop layout (1280px) spot-checked in browser devtools
- [ ] Loading, error, and empty states rendered and compared against wireframe spec
- [ ] No placeholder text, "TODO", or dev-only labels in the user interface

This check takes 15 minutes. Skipping it means user experience regressions are caught in Phase 8 team review and filed as Group C tickets — they ship. Do the check now.

### 5.4 Analytics Instrumentation Verification

For every new user interaction added in this module:
- [ ] Corresponding analytics event fires and appears in Sentry/Logtail or test transport
- [ ] Event does not contain personal information (email, phone, full meeting link URL)
- [ ] Event property names match the schema in product requirements document §16

**🔴 Gate:** All tests pass, coverage ≥ 97.5%, user experience sign-off checklist complete, analytics events verified.

---

## Phase 6: Integration & Postman Verification

### 6.1 Pact Provider Verification


Implementation example omitted. This public edition retains the written requirements in this section.


### 6.2 Postman Collection Run

Bring up the full stack locally via Docker Compose and run the Postman collection against it:


Implementation example omitted. This public edition retains the written requirements in this section.


- All requests must return expected status codes
- All response assertions must pass
- Update the collection with any new endpoints or changed response shapes
- Commit the updated collection file

### 6.3 End-to-End Smoke (Pre-launch once, not per module)

A Playwright end-to-end regression suite covers the critical path: register → post → interview request → admin schedules → join meeting. This suite runs once before each production release, not as a per-module gate. Per-module end-to-end on staging adds weeks of overhead without proportional signal — Pact + Postman cover the same contract and integration surface.

**🔴 Gate:** Pact green, Postman collection (all response body assertions, not just status codes) passes against local Docker stack, local postman environment implementation reference confirmed in .gitignore.

---

## Phase 7: Dev Ops Validation

**Containers:** Docker  
**MVP deployment:** Railway (PostgreSQL + Redis + Spring Boot, ap-southeast-1). Low ops overhead; appropriate until first paying users or investor funding.  
**Post-traction target:** Amazon Web Services (container service Fargate or App Runner) or GCP (Cloud Run) — migrate when active user count justifies it or Railway's limitations are hit.  
**continuous integration/continuous deployment:** GitHub Actions

### 7.1 Docker Files Required

Every service must have a production-grade Dockerfile:

**Backend (Dockerfile backend):**


Implementation example omitted. This public edition retains the written requirements in this section.


**Frontend (Dockerfile frontend):**


Implementation example omitted. This public edition retains the written requirements in this section.


### 7.2 Docker Compose (Local Dev + continuous integration)

docker-compose implementation reference must define:
- postgres (PostgreSQL 16, port 5432, seeded with Liquibase)
- redis (Redis 7, port 6379)
- backend (Spring Boot, depends on postgres + redis)
- frontend (Vite dev server or built Nginx, depends on backend)
- Health checks on all services
- Named volumes for postgres data persistence


This example illustrates the manage the required application services


### 7.3 Cloud Deployment (Amazon Web Services / GCP)

Target platforms (choose one per environment):

| Platform | Service | Use case |
|---|---|---|
| Amazon Web Services | container service Fargate | Long-running Spring Boot application programming interface; predictable workload |
| Amazon Web Services | App Runner | Simpler ops; auto-scaling; good for low-traffic MVP |
| GCP | Cloud Run | Serverless containers; per-request billing; cold start risk |
| Amazon Web Services | managed relational database PostgreSQL | Managed database (alternative to Railway) |
| Amazon Web Services | Elasti Cache Redis | Managed Redis (alternative to Railway Redis) |
| Amazon Web Services | SES | Email (already in spec — keep) |
| Amazon Web Services | S3 | File storage (already in spec — keep) |

Infrastructure-as-code for cloud deployments:
- Amazon Web Services: define container service task definitions, container image registry repositories, VPC, and security groups via Terraform or Amazon Web Services CDK
- All secrets in Amazon Web Services Secrets Manager (not environment variables in task definitions)
- GCP: Cloud Run service YAML configs in deploy/gcp/

### 7.4 Dev Ops Validation Checklist

Before any deployment file is considered complete, run a /team-review quick pass focused on Taylor (Dev Ops):

- [ ] Dockerfile builds successfully: the documented operational check
- [ ] docker-compose up brings full stack up and passes health checks
- [ ] docker-compose run backend ./mvnw test runs all tests inside the container
- [ ] Liquibase migration runs via Railway Release Command (pre-deploy) — mvnw liquibase implementation reference succeeds before new JAR starts; migrations do NOT run on container startup
- [ ] GitHub Actions workflow file is syntactically valid: * implementation reference
- [ ] Secrets are NOT hardcoded in any Dockerfile, docker-compose implementation reference, or workflow file
- [ ] Container images tagged with git SHA (not just latest)
- [ ] Cloud deployment config references the correct image tag and secrets source

**🔴 Gate:** Docker build passes, docker-compose stack is healthy, deployment files validated by Taylor team review.

### 7.5 Observability Gate

Before Phase 7 is considered complete:

- [ ] Logtail log drain connected to Railway and streaming — search for correlation Id in a recent request log to confirm propagation
- [ ] At least one **latency alert** configured: fire if P95 response time on ** interface operation exceeds 500 ms for 5 consecutive minutes
- [ ] At least one **error rate alert** configured: fire if web request protocol 5xx rate on ** interface operation exceeds 1% for 5 consecutive minutes
- [ ] Sentry is receiving error events: trigger a deliberate 500 in dev, confirm event appears in the Sentry project within 30 seconds
- [ ] Spring Boot /actuator/health returns the up state from the Railway healthcheck URL
- [ ] local postman environment implementation reference is in .gitignore; local postman environment example implementation reference (with placeholder values) is committed to the repo

---

## Phase 8: Team Review

Per the mandatory process rule (saved in memory), every completed deliverable must pass a /team-review before it is considered done.

For a completed module, run a **full debate** (not quick mode):
- All 13 agents review the complete implementation
- Group A (correctness) and Group B (security/privacy) issues are fixed before merge
- Group C and D issues are filed as tickets and addressed in the next sprint
- Tech Lead the technical lead produces the final decision

**🔴 Gate:** All Group A and B issues from the team review are resolved. pull request is created only after this gate.

---

## Tiered Workflow (Scope-Based Phase Selection)

Not every change requires all 8 phases. Use the tier that matches the scope of the change. When in doubt, use the higher tier.

| Tier | Trigger | Required Phases |
|---|---|---|
| **New module** | Building a new module from scratch | All 8 phases |
| **Scoped change** | ≤3 endpoints changed, no database schema change | Phases 1, 4 (or 5), 6, 8 |
| **Bug fix** | No application programming interface contract change, no schema change | Phases 1, 4 (or 5), 8 |

**Rules:**
- If the change touches the database schema → Phase 3 (Liquibase migration) is mandatory regardless of tier
- If the change alters any request/response shape → Phase 2 (application programming interface Contract Review) is mandatory regardless of tier
- Phases 7 and 8 are always mandatory before any change merges to main
- If you are uncertain which tier applies, escalate to Phase 2 and run the full contract review

---

## Workflow Summary Card


The illustration for workflow summary card contains the following information:

- InTuition — Module Dev Workflow (v1.1).
- 1. PLAN Write plan, get approval.
- 2. application programming interface CONTRACT Riley/Sam/Casey/Morgan/Reese agree Pact stubs.
- → Postman collection updated.
- 3. database MIGRATION Liquibase changeset rollback tested locally.
- 4. BACKEND JUnit5 Mockito Testcontainers ≥97.5%.
- 5. FRONTEND Vitest RTL ≥97.5%.
- 6. INTEGRATION Pact provider green Postman collection passes.
- 7. DEVOPS Docker build docker-compose healthy cloud cfg.
- 8. TEAM REVIEW /team-review full debate, A B issues fixed.
- All 8 gates must be green before the pull request merges to main.


---

## Key Technical Decisions (Fixed for All Modules)

| Concern | Decision |
|---|---|
| database migration tool | **Liquibase** (XML/YAML changesets, rollback required) |
| database schema management | Hibernate ddl-auto is validate — Liquibase owns all schema |
| Backend test coverage | **≥97.5%** line + branch (Jacoco enforced in continuous integration) |
| Frontend test coverage | **≥97.5% lines / ≥88% branches** (Vitest enforced in continuous integration) |
| application programming interface contract tool | **Pact** (consumer-driven; React SPA is consumer, Spring Boot is provider) |
| application programming interface documentation | **Postman collection** (committed to repo, kept current) |
| Containerisation | **Docker** (Dockerfile per service, docker-compose for local/continuous integration) |
| Cloud deployment | **Amazon Web Services container service Fargate or App Runner** / **GCP Cloud Run** |
| Secrets management | **Amazon Web Services Secrets Manager** / **GCP Secret Manager** (never in env files) |
| continuous integration/continuous deployment | **GitHub Actions** (lint → test → coverage check → Pact verify → Docker build → deploy) |
