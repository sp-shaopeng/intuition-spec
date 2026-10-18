# InTuition — Module Development Workflow

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Frontend branch threshold is 88%; backend remains 97.5% line and branch. Requirements changes must propagate through design, tasks, acceptance and references. See [current requirements](../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.

**Historical document version:** 1.1

**Original date:** 2026-05-27

**Editorial status, 18 October 2026:** Retained historical procedure with unresolved policy conflicts. This page is not a standalone current operating authority and does not prove that its proposed automation was installed.

For the current explanation of a task from start to finish, read the [workflow walkthrough](../../docs/workflow-walkthrough.md). The [workflow improvement plan](../../improvement-plans/ai-workflow.md) describes proposed process changes, and [testing and release](../../docs/testing-and-release.md) distinguishes inspected checks from unverified or missing automation.

The original page called itself active and claimed precedence over module guidance. That claim cannot settle its own conflicting instructions: the all-phases rule versus the tier table, the 18-agent review versus the older 13-agent list, and the lab-first rule versus release-only browser checks. The founder must reconcile those policies explicitly. This editorial correction selects no reduced workflow and does not waive the standing full-review or coordinator's final-audit requirements. The local notes below identify the conflicts where a reader would otherwise encounter them.

---

## Overview

**Historical all-phases rule:** The original overview required every new or changed module to pass all eight phases without skipping any. It treated the gates marked 🔴 as blockers to the next phase. This conflicts with the [tiered workflow](#tiered-workflow-scope-based-phase-selection) below; a developer must not choose a shortcut from either passage without the founder's reconciliation.


The historical phase sequence was:

- Phase 1: Plan.
- Phase 2: API Contract Review 🔴 All domains agree before any code is written. An API is the interface through which the browser and server exchange requests and responses.
- Phase 3: Database Migration 🔴 Liquibase changeset reviewed, rollback verified.
- Phase 4: Backend Development 🔴 97.5% coverage, all tests green.
- Phase 5: Frontend Development 🔴 97.5% line and 88% branch coverage, all tests green.
- Phase 6: Integration & Postman 🔴 Pact verification passes and the Postman collection is updated; the proposed Pact gate is not established by the inspected implementation.
- Phase 7: DevOps Validation 🔴 Docker build succeeds, deployment files valid.
- Phase 8: Team Review 🔴 Full team review completed, all Group A/B issues fixed.


> **Retained review requirement:** The recorded founder instruction of 30 May 2026 requires the full 18-agent, five-round debate with no quick mode. The standing coordinator's final audit also remains required. Older references below to quick reviews, 13 agents or a technical-lead decision do not authorize reducing those obligations; the founder must reconcile the conflicting historical descriptions.

---

## Workflow Amendments — Adopted 2026-05-30

The historical record describes these four amendments as adopted after a full 18-agent review and the coordinator's audit, with conditions. It assigned them precedence over older conflicting text. They record lessons from an admin-lab session; that adoption history is not proof of current enforcement. Unresolved conflicts remain labelled below rather than being silently resolved by this public edition.

### A1. Automated pre-review gate (lab-first)

Before the **Phase 8 Team Review** is entered, an **automated pre-review gate** must be green:


The required sequence in that amendment was:

- Formatting → build → regression lab on the development stack → coverage check. The corrected thresholds are 97.5% lines and branches for the backend, and 97.5% lines and 88% branches for the frontend.


- If the gate is **RED, fix before any review agent is spawned.** Mechanical/contract bugs die here cheaply, so the panel spends its expensive reasoning on design and risk, not on defects a run would have caught.
- The gate's result is **handed to the panel as evidence** — Reese/Seb cite real runs, not guesses.
- The handoff **must include an explicit NOT COVERED section**: for example, untested rejection of stale signed session tokens, missing load or concurrency cases, or limitations of testing on one shared stack. A passing result must be accompanied by its known gaps.
- **Terminology:** this is an *automated pre-review gate*, **not** a "deterministic" gate. The lab is black-box against one shared dev stack and is **order/load-sensitive** (documented 429 rate-limiter cascades, connection-pool exhaustion, a warm-up race). The durable fix (in-process fresh-database Testcontainers layer) was **de-scoped by the founder** ("don't need fresh db, dev anyway"), so the substrate stays non-deterministic and the flake protocol below is the only mitigation.

**Historical flake protocol:** The original reset step affected shared rate-limit and account-lock state. It is retained here as historical intent, not permission to reset unrelated records. The [current maintenance proposal](../../improvement-plans/ai-workflow.md#maintain-the-lab-with-each-change) requires ownership and target checks; the founder must reconcile the old reset and retry policy before it is used as current operating guidance.

1. Reset rate-limit counters, clear failed-login counts and remove account-lock expiry values for the intended test accounts.
2. On an **environmental** RED (429 cascade / connection-pool exhaustion — the documented signatures only), **one automatic retry**.
3. Only a **second** RED blocks the pipeline and escalates. A product-failure RED (assertion mismatch, 4xx/5xx contract break) blocks immediately — no retry.

**Status recorded in May:** lab-first was local discipline, with enforcement through GitHub Actions still owed. That historical statement is not a current inventory; see [testing and release](../../docs/testing-and-release.md). **The lab gate is not a substitute for production observability**: runtime detection, such as the admin server-error alert, remains a separate obligation.

**Historical coupled requirement (Should):** provide cleanup limited to the synthetic regression-data group so repeated runs do not make the development database unusable for manual testing. This does not authorize deleting unrelated data or settle the separately approved retention and soft-deletion rules.

### A2. Edit→verify reflex

After any edit to registration, configuration or wiring—such as a Liquibase changelog entry, route map, dependency wiring or environment configuration—search the resulting file to confirm the intended edit before building. This extends the coordinator's verify-before-proceed rule to the implementing agent's own edits. The historical record attributes a startup failure and a nearly incorrect passing report to two edits that had not actually taken effect.

### A3. Definition of Done ends at dev — with a non-deletable prod obligation

- **DONE-DEV** = regression lab green on the **dev** stack + team review cleared. The regression lab's scope is **dev/sim database only** — it never requires prod access.
- Any change that fixes or touches a production-affecting defect must also create a **PROD-DEPLOY-OWED** entry in the roadmap. Record the change, migration-before-startup order, production schema-verification procedure and named owner. A PROD-DEPLOY-OWED entry cannot be deleted; it can only move to DEPLOYED after the founder confirms.
- **Review cadence:** the PROD-DEPLOY-OWED register is **opened and reconciled the moment prod access is next available** — it is the first thing checked, so a dev-validated fix for a live prod bug can never silently rot on dev.

### A4. Schema-change coupling rule

An entity change affecting a column ships together with its Liquibase changeset and a verified entry in the master changelog. With schema validation enabled, a mismatch between the entity mapping and database can prevent startup. Assess rollback safety for each migration. For example, converting text back into structured JSON requires every affected value to be valid for that conversion.

**Proposed backstop (Should):** a CI check should fail when an entity column has no corresponding changelog entry. Check that the column exists in the registered changelog rather than merely matching annotation text. The historical proposal retained the coupled-change and file-verification discipline until that check existed; its installation is not established here.

### Accepted risks (eyes open)

- **Always-full-debate × lab-first compounds per-change cost.** Every change now needs a full lab run *and* a full 18-agent debate. This raises fixed cost per change and predictably incentivises **batching** (accumulating changes to amortise review). The founder's always-full-debate decree stands; **tripwire:** if change-batching emerges to dodge review overhead, flag it to the founder to reconsider.
- **Gate on a non-deterministic substrate** (founder de-scoped the fresh-database fix) — mitigated by the flake protocol, not eliminated.
- **Build-time gate ≠ runtime detection** — the admin 5xx alert and prod observability remain separately owed; this amendment must not be used as an excuse to keep deferring them.

---

## Phase 1: Plan

**Before writing a single line of code or changing any file:**

1. Read the module's specification, frontend design, backend design and database design.
2. Write a plan — either as a reply in the session or as a plan file — covering:
   - What is being built or changed (scope)
   - What existing behaviour is affected (blast radius)
   - What acceptance criteria must be satisfied
   - Estimated order of implementation (database → backend → frontend)
3. The plan must be reviewed and approved before proceeding
4. For changes to existing modules, identify affected downstream modules. For example, changing an API response shape may break Interview, Matching or Commission.

**Principle:** No surprises mid-implementation. If the approach cannot be fully articulated before coding starts, more spec reading is required.

---

## Phase 2: API Contract Review

**The API contract defines the boundary between the browser application and the Spring Boot server. All affected domains must agree before implementation.**

### 2.1 Contract Definition

For every endpoint in the module:

1. Write or update the endpoint's entry in the API specification:
   - HTTP method and path
   - Authentication and permission requirements
   - Full request body schema (field names, types, constraints)
   - Full response body schema (all fields, types, nullable/required markers)
   - Every error code, its HTTP status and its meaning, including invalid credentials or an already registered email address. Production responses must not introduce undocumented error codes.
   - The paginated response structure, page size and zero-based page numbering

2. For changed endpoints: explicitly diff the old and new contract. Any breaking change (removed field, type change, renamed field, status code change) requires a version bump or a migration plan.

### 2.2 Cross-Domain Agreement

The historical contract review called for these affected perspectives:
- **Riley (frontend-facing service):** payload shape, pagination, versioning
- **Sam (Frontend):** does the React SPA have everything it needs?
- **Casey (Security):** auth requirements, sensitive field exposure
- **Morgan (DBA):** can the backend produce this efficiently without N+1?
- **Reese (quality assurance):** are all error cases covered?

**🔴 Gate:** Contract is locked only when all five agents have no Group A objections. Once locked, no field may be added, removed, or renamed without repeating this review.

This specialist contract review is not a replacement for the required full deliverable review and coordinator's final audit.

### 2.3 Pact Contract Stub

**Unestablished implementation:** This is the historical Pact proposal. The inspected manifests and workflows do not establish installed Pact consumer/provider verification or a Pact Broker. The existing API inventory and snapshot utilities are different mechanisms. The founder must reconcile the proposed gate with the implemented lab; this note neither installs Pact nor waives contract verification.

Before backend implementation begins, write the Pact consumer test stubs in the React SPA:
- These define the expected request/response shapes in runnable TypeScript
- They act as the living contract that the Spring Boot provider must satisfy
- Publish the consumer contract to the proposed Pact Broker before backend work starts
- **Must include at least one interaction per 4xx variant** — if the endpoint can return 400, 401, 409, and 423, write an interaction for each; do not write only the happy path
- **Rate-limited endpoints must include a 429 interaction** documenting the response's retry-delay information
- Pact files with only 2xx interactions will be rejected at Phase 2 gate review

### 2.4 Postman Collection Update

Add each API endpoint to the shared Postman collection:
- One folder per module
- One request per endpoint, with example request body and expected response
- Environment variables for the application address and separate protected credentials for parent, tutor and administrator roles
- Authentication setup that calls the login endpoint and stores the session cookie
- Tests tab: assert response status code + at minimum one field from the response body

The historical requirement is to commit the Postman collection to a repository and update it with every API change. This statement does not verify the collection's current storage or execution.

---

## Phase 3: Database Migration

**Tool: Liquibase**

Liquibase manages database schema changes. Hibernate checks that the schema matches the entity mappings; it does not replace the migration process.

### 3.1 Changeset Rules

1. Every schema change needs a new, numbered Liquibase changeset registered in the changelog. The historical text allowed XML or YAML; [current requirement C14](../CURRENT-REQUIREMENTS.md) requires XML. A standalone SQL file is not a registered migration.
2. Changesets are immutable once merged to main — never edit a merged changeset
3. Every changeset that creates or modifies a table must include rollback instructions.
4. Express PostgreSQL-specific partial indexes, custom enumerations, triggers and check constraints through SQL inside the registered Liquibase changeset where the standard change types cannot express them.
5. **Historical enum rollback policy:** When adding an enumeration value, document that automatic rollback makes no change and describe the required manual recovery. The original policy proposed recreating the type without the new value and assumed limits on transactional rollback. Those assumptions and effects on dependent objects must be checked for the selected database version; this historical paragraph does not establish a safe recovery procedure. Do not provide an automatic rollback that fails or removes structures still needed by the running application.
6. **Database lock timeout:** Changes to the structure of live tables must first set a three-second limit on waiting for a database lock, preventing indefinite waits during a rolling deployment.
7. Validate the Liquibase changelog in CI so malformed XML and duplicate changeset identifiers fail the build.
8. **Historical migration dry-run requirement:** CI must apply the migrations to a disposable PostgreSQL 16 database. Validating the changelog alone does not prove that its SQL executes. The named database version and the existence of this CI step must be checked against the chosen environment; see the [inspected technology snapshot](../../docs/how-it-is-built.md).

### 3.2 Local Validation

Before committing a changeset, start the local database, apply the migration, verify the schema against the entity mappings, test the rollback and confirm that the database returned to its prior state. The private source supplies the operational commands; they are omitted here.


### 3.3 Changeset Review

**Legacy wording requiring reconciliation:** This section originally requested a quick DBA review, which conflicts with the retained no-quick-mode instruction. The founder must reconcile that wording. A DBA's focused checks remain necessary and do not replace the full review or final coordinator audit:
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

The framework and database versions in this historical section are not the current dependency inventory. See [how the application is built](../../docs/how-it-is-built.md).

### 4.1 Implementation Order


The historical implementation order was: migration, entity mappings, repository queries, services, controllers and request/response data types. It specified manual mapping and deferred MapStruct until after the initial release.


### 4.2 Unit Tests (JUnit 5 + Mockito)

Every service class has companion unit tests:
- All public methods tested
- All guard branches tested (both paths: pass and fail)
- All state machine transitions tested (valid + invalid)
- All error codes tested
- Use Mockito mocks without starting a Spring application context

### 4.3 Repository Tests (Spring Data JPA + Testcontainers)

Every repository with a custom query has companion repository tests:
- Uses a real PostgreSQL 16 container (not H2)
- Tests all custom queries with real data
- Tests partial indexes (concurrent insert tests for concurrency guards)
- Tests soft-deletion filters

### 4.4 Structured Logging and Correlation IDs

Every incoming HTTP request must have a correlation identifier. The server generates one when absent, returns it in the X-Correlation-ID header and includes it in the request's log entries.


Implementation example omitted. This public edition retains the written requirements in this section.


Log pattern must include the request’s correlation identifier in the log entry. This is required — not optional — before Phase 4 gate clears.

**Gate check:** Controller tests must assert that the response contains the X-Correlation-ID header and that the log output contains the same identifier.

### 4.5 Controller and Integration Tests

Every controller has companion tests using Spring's MockMvc:
- Tests the full filter chain (signed session token validation, role guards, rate limits)
- Tests every endpoint's happy path
- Tests every client-error and server-error response, asserting its specific error-code value as well as its HTTP status
- Tests contact field privacy: asserts phone/email are absent from public/pre-assignment views and available only to authorized assigned participants under match-state gates
- Tests X-Correlation-ID propagation (see §4.4)

### 4.6 Coverage Enforcement


Implementation example omitted. This public edition retains the written requirements in this section.


The required build gate fails below 97.5% line or branch coverage. Excluding code from JaCoCo requires explicit justification in the pull request; the existence of that rule is not evidence of achieved coverage.

**Pull request test assertion requirement:** Each test must assert a specific field value, rather than merely checking that a response is non-null or that a mock was called. Rendering a component or calling a service without checking the outcome does not establish correct behavior. Reviewers must check this in the pull request.

**🔴 Gate:** All tests pass, coverage ≥ 97.5%, zero ignored/skipped tests without justification.

---

## Phase 5: Frontend Development

**Historical language/framework choice:** React 18 + TypeScript (Vite); see the [current inspected versions](../../docs/how-it-is-built.md).

**Test framework:** Vitest + React Testing Library  
**Minimum coverage:** 97.5% lines and 88% branches (enforced by Vitest coverage thresholds)

### 5.1 Implementation Order


The proposed implementation order was:

1. Pact consumer tests from Phase 2.3; their implementation remains unestablished.
2. TypeScript types and Zod schemas.
3. TanStack Query hooks for each endpoint.
4. Zustand updates for any new interface state.
5. Route-level page components.
6. Reusable shared components.
7. Form validation with React Hook Form and Zod.


### 5.2 Component Tests (Vitest + RTL)

Every component has companion tests covering:
- Happy path: renders correctly with valid props/state
- Loading state: renders skeleton / spinner
- Error state: renders error message
- Empty state: explains that no data is available
- Form validation: all invalid inputs show correct error messages
- Accessibility: key interactive elements have correct ARIA attributes

### 5.3 Coverage Enforcement


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.4 User Experience Design Sign-Off

Before marking the frontend gate as cleared, complete this 15-minute checklist:

- [ ] New and changed components visually reviewed against approved wireframes
- [ ] All user-facing copy matches the spec's copy deck (button labels, headings, error messages, empty states)
- [ ] Mobile layout (390px) and desktop layout (1280px) spot-checked in browser devtools
- [ ] Loading, error, and empty states rendered and compared against wireframe spec
- [ ] No placeholder text, "TODO", or dev-only labels in the user interface

The historical plan estimated 15 minutes for this check. Its purpose is to find interface regressions before the final review; that estimate is not evidence of how long a particular review takes or permission to ship an unresolved defect.

### 5.5 Analytics Instrumentation Verification

For every new user interaction added in this module:
- [ ] Corresponding analytics event fires and appears in Sentry/Logtail or test transport
- [ ] Event does not contain personal information (email, phone, full meeting link URL)
- [ ] Event property names match the schema in product requirements document §16

**🔴 Gate:** All tests pass, coverage reaches 97.5% of lines and 88% of branches, the user experience checklist is complete, and analytics events are verified.

---

## Phase 6: Integration & Postman Verification

### 6.1 Pact Provider Verification

**Unestablished implementation:** The historical proposal requires the backend to verify every consumer-contract interaction, with none left unverified. Installed Pact verification and a broker have not been established by the inspected files. See [testing and release](../../docs/testing-and-release.md#limits-that-remain-visible); a snapshot comparison alone does not prove this proposed gate exists.


### 6.2 Postman Collection Run

Bring up the full stack locally via Docker Compose and run the Postman collection against it:


Implementation example omitted. This public edition retains the written requirements in this section.


- All requests must return expected status codes
- All response assertions must pass
- Update the collection with any new endpoints or changed response shapes
- Commit the updated collection file

### 6.3 End-to-End Smoke — Conflicting Historical Cadence

The older proposal placed a browser smoke check before each production release, covering registration, posting, interview requests, staff scheduling and joining the meeting. It omitted that check from individual module gates and argued that Pact and Postman supplied enough integration evidence. This conflicts with the lab-first amendment and later browser-regression obligations. API checks do not establish that the website controls work, and the inspected files do not establish the proposed Pact gate. The founder must reconcile the cadence; this historical passage is not permission to omit required browser evidence. See the [current test boundaries](../../docs/testing-and-release.md).

**Historical gate:** Pact provider verification passes; the Postman collection passes its response-body and status assertions against the local Docker stack; the private Postman environment file is excluded from version control. The Pact portion remains unestablished, and naming this gate does not show it passed.

---

## Phase 7: DevOps Validation

**Historical deployment options:** The Railway and later cloud options below describe this document's original plan. They do not replace the approved production arrangement in [current requirement C16](../CURRENT-REQUIREMENTS.md), or establish deployment, installed monitoring or a live service.

**Containers:** Docker  
**MVP deployment:** Railway (PostgreSQL + Redis + Spring Boot, ap-southeast-1). Low ops overhead; appropriate until first paying users or investor funding.  
**Historical post-traction target:** AWS ECS Fargate, AWS App Runner or Google Cloud Run.

**Continuous integration and deployment:** GitHub Actions was the selected tool; the complete proposed pipeline is not established by this page.

### 7.1 Docker Files Required

Every service must have a production-grade Dockerfile:

**Backend (Dockerfile backend):**


Implementation example omitted. This public edition retains the written requirements in this section.


**Frontend (Dockerfile frontend):**


Implementation example omitted. This public edition retains the written requirements in this section.


### 7.2 Docker Compose for Local Development and CI

The historical Docker Compose definition required:
- postgres (PostgreSQL 16, port 5432, seeded with Liquibase)
- redis (Redis 7, port 6379)
- backend (Spring Boot, depends on postgres + redis)
- frontend (Vite dev server or built Nginx, depends on backend)
- Health checks on all services
- Named volumes for postgres data persistence


The omitted example coordinated these services, their dependencies and health checks.


### 7.3 Historical Cloud Deployment Options

The source listed these alternatives; they are not a new platform-selection decision:

| Platform | Service | Use case |
|---|---|---|
| AWS | ECS Fargate | Long-running Spring Boot API; predictable workload |
| Amazon Web Services | App Runner | Simpler ops; auto-scaling; good for low-traffic MVP |
| GCP | Cloud Run | Serverless containers; per-request billing; cold start risk |
| AWS | RDS PostgreSQL | Managed database (alternative to Railway) |
| AWS | ElastiCache Redis | Managed Redis (alternative to Railway Redis) |
| Amazon Web Services | SES | Email (already in spec — keep) |
| Amazon Web Services | S3 | File storage (already in spec — keep) |

Infrastructure-as-code for cloud deployments:
- AWS: define ECS tasks, ECR image repositories, networking and security groups through Terraform or AWS CDK
- All secrets in Amazon Web Services Secrets Manager (not environment variables in task definitions)
- GCP: retain reviewed Cloud Run service configurations

### 7.4 DevOps Validation Checklist

**Legacy wording requiring reconciliation:** The source requested a quick DevOps review here, conflicting with the no-quick-mode instruction. The founder must reconcile that wording. These focused checks do not replace the full review or coordinator's final audit:

- [ ] Container images build successfully
- [ ] Docker Compose starts the full stack and all health checks pass
- [ ] The backend test suite runs inside its container
- [ ] In the historical Railway plan, the release step applies Liquibase migrations before the new application starts, rather than during container startup
- [ ] GitHub Actions workflow syntax is validated
- [ ] Secrets are not hardcoded in container, Compose or workflow definitions
- [ ] Container images tagged with git SHA (not just latest)
- [ ] Cloud deployment config references the correct image tag and secrets source

**🔴 Gate:** Container builds pass, the composed stack is healthy, and the DevOps review validates deployment files.

### 7.5 Observability Gate

Before Phase 7 is considered complete:

- [ ] In the historical Railway setup, logs stream to Logtail and a recent request's correlation identifier can be found
- [ ] A latency alert fires when the 95th-percentile response time across API endpoints exceeds 500 milliseconds for five consecutive minutes
- [ ] An error-rate alert fires when HTTP server errors exceed 1% of API responses for five consecutive minutes
- [ ] Sentry is receiving error events: trigger a deliberate 500 in dev, confirm event appears in the Sentry project within 30 seconds
- [ ] Spring Boot /actuator/health returns the up state from the Railway healthcheck URL
- [ ] The private Postman environment file is excluded from version control, while an example containing only placeholders is committed

---

## Phase 8: Team Review

Every completed deliverable remains subject to full team review and the coordinator's final audit before completion can be claimed.

**Conflicting historical description:** The original Phase 8 list named 13 agents and assigned the final decision to the technical lead. The retained founder instruction names 18 agents and five rounds, and standing instructions also require the coordinator's final audit. The founder must reconcile this list; its smaller count and older decision role do not authorize reducing the required review.

The historical list required:

- Review of the complete implementation by all 13 roles then listed
- Group A (correctness) and Group B (security/privacy) issues are fixed before merge
- Group C and D issues are filed as tickets and addressed in the next sprint
- A decision from the technical lead, followed by the separately required coordinator audit under the standing instructions

**Historical gate:** Resolve all Group A and B issues before creating the pull request. This does not replace the full-review and final-audit obligations or establish that other required acceptance conditions may remain unmet.

---

## Tiered Workflow (Scope-Based Phase Selection)

**Unresolved phase-selection conflict:** This older table allows fewer than eight phases, contradicting the all-phases overview and summary. Its scoped-change and bug-fix rows also omit Phase 7 even though the rules beneath make it mandatory. The founder must publish the applicable phase set before this table can be used to select work. The table is retained for reconciliation, not as permission to skip a gate or reduce review.

| Tier | Trigger | Required Phases |
|---|---|---|
| **New module** | Building a new module from scratch | All 8 phases |
| **Scoped change** | ≤3 endpoints changed, no database schema change | Phases 1, 4 (or 5), 6, 8 |
| **Bug fix** | No API contract change, no schema change | Phases 1, 4 (or 5), 8 |

**Rules retained from that same historical section:**
- If the change touches the database schema → Phase 3 (Liquibase migration) is mandatory regardless of tier
- If the change alters any request/response shape → Phase 2 (API Contract Review) is mandatory regardless of tier
- Phases 7 and 8 are always mandatory before any change merges to main
- If you are uncertain which tier applies, escalate to Phase 2 and run the full contract review

---

## Historical Workflow Summary Card


This summary repeats the original all-phases model. It is subject to the unresolved phase-selection conflict above and is not a separate source of operating authority:

- InTuition — Module Dev Workflow (v1.1).
- 1. PLAN Write plan, get approval.
- 2. API CONTRACT: affected specialists agree on the contract; the historical proposal adds Pact consumer tests.
- → Postman collection updated.
- 3. DATABASE MIGRATION: apply the Liquibase changeset and test rollback locally.
- 4. BACKEND: JUnit, Mockito and Testcontainers checks; at least 97.5% line and branch coverage.
- 5. FRONTEND: Vitest and React Testing Library checks; at least 97.5% line and 88% branch coverage.
- 6. INTEGRATION: the proposed Pact verification and Postman collection pass. Pact implementation remains unestablished.
- 7. DEVOPS: container builds pass, the stack is healthy and deployment configurations are checked.
- 8. TEAM REVIEW: full debate and resolution of Group A/B issues, with the standing coordinator's final audit retained.
- The historical summary required all eight gates before merging; it conflicts with the retained tier table and requires founder reconciliation.


---

## Historical Technical Decisions and Current Boundaries

The source called these decisions fixed for every module. Current requirements govern approved replacements, while the [testing guide](../../docs/testing-and-release.md) identifies what automation was actually found. Proposed tools and pipeline stages below must not be read as installed or verified.

| Concern | Decision |
|---|---|
| Database migration tool | **Liquibase**; the source allowed XML/YAML, while current C14 requires numbered XML changesets and reviewed rollback |
| Database schema management | Hibernate validates entity mappings; Liquibase manages schema changes |
| Backend test coverage | **≥97.5%** line + branch (Jacoco enforced in continuous integration) |
| Frontend test coverage | **≥97.5% lines / ≥88% branches** (Vitest enforced in continuous integration) |
| API contract tool | **Pact**, historically proposed with the browser as consumer and server as provider; installed verification and a broker remain unestablished |
| API documentation | **Postman collection**; the historical requirement is to version it and keep it current |
| Containerisation | **Docker** (Dockerfile per service, docker-compose for local/continuous integration) |
| Cloud deployment | Historical alternatives were **AWS ECS Fargate, App Runner and GCP Cloud Run**; current C16 and the approved deployment package govern the production arrangement |
| Secrets management | **Amazon Web Services Secrets Manager** / **GCP Secret Manager** (never in env files) |
| CI and deployment | **GitHub Actions**; the historical proposed sequence was lint, tests, coverage, Pact verification, container build and deployment. The complete sequence is not established as an enforced pipeline. |
