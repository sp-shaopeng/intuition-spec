# end-to-end continuous integration & Test-Health Spec — close the "crown-jewel journey gated nowhere" gap (S1)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Keep the crown-jewel prerequisites, exact-SHA gate and secret-safe composed continuous integration from [INT-064](int-064-multirepo-ci-spec.md); public provenance uses [027](../027-aws-production-deployment/spec.md). This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Owners:** devops-engineer (Taylor) + qa-engineer (Reese)
**Severity:** S1 (the revenue path has no automated gate that would catch an INT-016-class Sev-1)
**Date:** 2026-06-05
**Status:** SPEC — Boss SEND-BACK conditions closed (§1.5); build of workstream F (composed continuous integration lane +
crown-jewel journey) in progress against this corrected spec.

> ## Boss SEND-BACK conditions — and how each is closed (§1.5)
> The first Boss audit of this spec issued **SEND BACK** with five conditions. Each is closed below;
> the canonical fix is folded into the design/AC sections cited:
>
> 1. **Deploy-gate SHA check** — the deploy job MUST verify the *deployed* artifact's SHA equals the
>    commit under test, because the regression ITs hit the LIVE stack at localhost, so a stale
>    deploy = a false green. → closed in **§3.4** (deploy-SHA gate) + **AC C-9**.
> 2. **IT tiering** — heavy ITs are @full/nightly; the **pull request guard** is the Playwright **crown-jewel
>    journey** (fast, always-on), NOT the full IT suite. → closed in **§3.0** (tiering table) +
>    **§3.1** (pull request vs nightly split) + **AC B-6**.
> 3. **docker-compose ci implementation reference is the single source** for the continuous integration stack (no local/continuous integration drift). → closed
>    in **§3.2** (single-source + drift-guard) + **AC A-2/A-2a**.
> 4. **POLYREPO PAT acceptance criteria** — the cross-repo checkout token's storage, scope, and
>    consumption are specified. → closed in **§3.1a** (POLYREPO PAT contract) + **AC A-1a**.
> 5. **IT paths** — the IT invocation uses the correct module paths. → closed in **§3.1b** (IT path
>    correctness) + **AC A-1b**.

Grounding note: every file:line/path below was read on disk before writing. Where a claim
depends on a run-time observation (a GitHub Actions run, a flake rate), it is called out as
**unverified-pending-run**, not asserted.

---

## 1. Problem

The crown-jewel journey of this marketplace —
**register → (tutee) post a job → (tutor) apply → admin/parent assign → contact reveal** —
is the exact flow that broke in INT-016 (the cancel/contact-reveal Sev-1 500, root-caused to
a bulk database update clearing loaded entity state before the transaction finished). That whole class
of defect is caught **only** by tests that drive the *composed, running* stack end to end:

1. the **black-box backend regression ITs** — Abstract Api IT
   (Abstract Api IT implementation reference)
   drives the live stack over web request protocol via REST Assured, base Url REGRESSION BASE URL default
   http://localhost (Abstract Api IT implementation reference); and
2. the **Playwright e2e** in the e2e repo (REDACTED local path).

**Neither runs anywhere on GitHub today.** Two distinct gaps:

### 1a. The composed regression/e2e continuous integration lives only in the unversioned wrapper
INT-064 fixed and pushed **per-repo standalone continuous integration**:
- backend CI workflow — build and verification with CI mode enabled,
  temurin 21 (ci implementation reference). This runs the @Data Jpa Test Testcontainers repo tests + the
  JaCoCo ≥0.975/0.975 gate. Its own header (ci implementation reference) states the black-box *IT suite
  **self-skips** here because no live stack is up — "expected and non-gating."
- frontend CI workflow — dependency installation, code-rule checks, build and tests on Node 20,
  Vitest ≥97.5% line / ≥88% branch (ci implementation reference).

But the workflows that boot the **composed multi-repo stack** —
REDACTED local path and
regression-fe implementation reference — live in the **unversioned monorepo wrapper**
(REDACTED local path), which is **not a git repo** (SHARED-LESSONS
"POLYREPO" entry; devops SKILL 2026-06-04 INT-064). GitHub therefore **never sees them — they run
nowhere.** They also cannot be dropped into a subrepo as-is because they assume the monorepo:
- regression-be implementation reference / regression-fe implementation reference: starting the combined application requires the
  wrapper-only REDACTED local path (which builds
  context: ./intuition-backend and intuition-frontend implementation reference — both **sibling** subrepos);
- working-directory: intuition-backend (regression-be implementation reference) and e2e (regression-fe implementation reference);
- paths: intuition-backend/** | intuition-frontend/** | e2e/** pull request triggers — monorepo paths that
  don't exist inside any single subrepo.

### 1b. Historical gap: the main marketplace journey lacked browser coverage
At the time of this proposal, the recorded inventory had nine navigation, authentication and showcase smoke-test files. This is a historical count; see the current [testing inventory](../../docs/testing-and-release.md) for the October inspection.


The historical inventory covered navigation, authentication and showcase checks. It did not yet include the complete marketplace journey described below.


All are @smoke, all chromium (playwright config implementation reference). **None exercises
register → post → apply → assign → contact-reveal.** The single most valuable browser journey —
the one that earns commission and the one INT-016 broke — has **zero** Playwright coverage.

On the backend side the journey IS covered by black-box ITs that exist but run nowhere on continuous integration:
- Free Market Assign IT implementation reference free Market select And Assign reveal Contact then Cancel no Listing
  (asserts assign by tutor User Id, contact reveal 200 for **both** parties — the INT-016 missing-value failure/500
  guard — then cancel → reveal 404);
- Assignment Lifecycle IT implementation reference (choose-tutor 201/re-choose/IDOR-403; applicant list
  personal information-free; contact-reveal four states; pre-assignment reveal 404).
These belong to the full-test group and depend on the running application with prepared test data. They are the precise suites that
catch INT-016-class regressions — and they fire on **no GitHub trigger**.

### Net effect (the S1)
The dev backend image is built Dskip Tests (Dockerfile backend.dev), so **continuous integration is the only place
the real gate runs** (devops SKILL git lesson). With the composed lane absent, a broken
contact-reveal can be built into the dev image, deployed, and only discovered by Hunter or a user.
Branch protection does not yet require any cross-repo journey gate, so the Dskip Tests image can
**outrun the gate**.

---

## 1.5. Boss SEND-BACK conditions — closure detail

### C1 — Deploy-gate SHA check (the false-green killer)
Because Abstract Api IT drives the LIVE stack (Abstract Api IT implementation reference base Url default
http://localhost) and the dev image is built Dskip Tests (Dockerfile backend.dev), a green
regression run only means something if the stack under test is the **commit under test**. A stale
deploy (an older image still running) would let the regression lab pass against code that no longer
matches HEAD — a **false green**. The composed lane therefore **stamps and verifies a build SHA**:
- Stamp the tested revision into the development image. After the application becomes healthy, compare that revision with both container metadata and the running application's version information before starting regression tests. A mismatch stops the workflow.
- Missing, empty or unknown build identity is also a failure. A check that only rejects a known mismatch can accidentally accept an unverifiable installation.
- The historical correction added the application-level check after an earlier proposal assumed that information was already exposed. This is a requirement to verify, not proof of a current running environment. Production uses the minimal version response specified by the current requirements; protected operational information must not be made public to satisfy this development check.
- The development release check must likewise confirm that the running application reports the revision intended for deployment. See §3.4 and acceptance criterion C-9.

### C2 — IT tiering (pull request guard = crown-jewel, not the full suite)
The full black-box *IT suite (32 *IT files) and webkit/mobile Playwright are too slow for a
<5-minute pull request gate. Tiering (→ **§3.0**):
- **pull request guard (always-on, blocking):** the Playwright **crown-jewel journey** (@smoke, chromium) +
  only the backend integration tests assigned to the smoke group (the existing split recorded in
  regression-be implementation reference). Fast, deterministic.
- **Nightly (@full):** the whole IT suite, webkit + mobile Playwright projects, the cancel→reopen→
  reveal-404 tail of the crown-jewel, PIT/Stryker report jobs.
The crown-jewel journey is the new @smoke Playwright spec (§3.3); the longer integration tests remain in the full-test group
(Free Market Assign IT implementation reference).

### C3 — docker-compose ci implementation reference is the single source for the continuous integration stack
There must be exactly **one** versioned compose file the continuous integration lane consumes — docker-compose ci implementation reference
(→ **§3.2**). To stop continuous integration↔local drift, a **drift-guard** keeps the wrapper local docker-compose implementation reference
and docker-compose ci implementation reference in lockstep: the continuous integration lane runs a diff (modulo the documented
path/port deltas) and **fails if they diverge** beyond the allowed delta. No third copy is permitted;
the two wrapper regression workflows are deleted (→ **AC A-3**).

### C4 — POLYREPO PAT acceptance criteria (cross-repo checkout token)
The composed lane checks out intuition-backend + intuition-frontend as siblings of the e2e
repo, which requires a token with read access to those private repos (→ **§3.1a**):
- **Storage:** a GitHub **Actions secret** named POLYREPO PAT on the e2e repo (or an org/repo
  fine-grained PAT stored in repo secrets). NEVER committed; referenced only as
  a reference to the repository-read credential in the CI secret store. The founder provisions the value; the workflow does not contain it.
- **Scope (least privilege):** a **fine-grained PAT** limited to the two repos
  (in Tuition-be, in Tuition-fe), **Contents: Read-only** — no write, no other repos, no admin.
  (A classic PAT would need repo scope; the fine-grained variant is preferred.)
- **Consumption:** passed to actions/checkout@v4 with token for the backend and frontend checkout steps
  ONLY; the e2e self-checkout uses the default GITHUB TOKEN. Rotation: PAT expiry ≤90 days,
  rotated via repo settings; a 401 on checkout = expired/over-scoped PAT (runbook note).
- The secure remote access deploy key is **never** referenced here (encrypted web request protocol + PAT only). → **AC A-1a**.

### C5 — IT paths (correct module paths for the IT invocation)
The composed lane runs the backend ITs with working-directory: intuition-backend (the checkout path),
invoking Maven failsafe (*IT implementation reference under
intuition-the private implementation folder**). The artifacts it uploads are
intuition-the private implementation folder** (failsafe, not surefire — ITs are failsafe-bound).
The Playwright specs live under e2e/tests/** (run from the e2e checkout root). → **§3.1b**,
**AC A-1b**.

---

## 2. Current state (verified on disk)

| Asset | Path | Runs on GitHub? |
|---|---|---|
| backend per-repo unit/JaCoCo + Testcontainers gate | ci implementation reference | **Yes** (pushed, INT-064). Run itself unverified-pending — no gh locally. |
| frontend per-repo lint/build/test:ci gate | ci implementation reference | **Yes** (pushed, INT-064). Same caveat. |
| backend composed regression ITs workflow | regression-be implementation reference | **No** — unversioned wrapper. |
| frontend composed Playwright workflow | regression-fe implementation reference | **No** — unversioned wrapper. |
| Composed stack definition | docker-compose implementation reference | Unversioned; builds both sibling subrepos. |
| Playwright crown-jewel journey | — | **Does not exist.** |
| backend black-box assign/contact-reveal ITs | Free Market Assign IT implementation reference, Assignment Lifecycle IT implementation reference (+30 more *IT) | Exist; gated nowhere. |
| e2e repo branch | e2e on main → intuition-e2e git | — |

Polyrepo topology (SHARED-LESSONS): four independent repos — intuition-backend (sp-dev),
intuition-frontend (sp-dev), intuition-spec (sp-dev), e2e (main). The wrapper dir is
**not** a repo. e2e/.gitignore exists; frontend still tracks coverage/ (stage explicitly, never
a blanket staging operation).

### Adjacent items folded in by the founder brief
- **INT-060** (flaky Tutee Profile test) — tracker row reads OPEN, **but code shows it is fixed
  deterministically**: Profile test implementation reference. The fix waits
  until the form displays the loaded server value, then waits until the user’s replacement value is visible before clicking
  Save (135), with the comment explicitly citing INT-060 (127, 134). This is the correct
  fix (qa SKILL: "await the submit settle / find By*"), not a retry crutch. **Action: CONFIRMED
  FIXED — close INT-060; correct the stale README row.**
- **INT-047** (local Docker Build Kit/bake wedge) — README row already reads **RESOLVED 2026-06-03**
  (root cause virtual machine↔registry stale routes under VPN; cleared by full Docker Desktop restart; devops
  SKILL + SHARED-LESSONS capture the diagnose-then-pkill/restart playbook). **Action: CONFIRM
  RESOLVED — close INT-047.** Note its operational implication for §3: the composed lane must
  tolerate the bake/buildkit wedge class (legacy-builder fallback / clean-slate retry) so a transient
  build hang is diagnosed, not waited out.

### Test-health gaps (qa)
- **No mutation testing** (PIT for backend / Stryker for frontend) on the high-stakes logic that protects
  revenue/authz: Admin Match Service implementation reference, Contact Reveal Service implementation reference,
  Jwt Authentication Filter implementation reference. Line coverage proves a line *ran*, not that a test would
  *fail if the logic broke* (qa SKILL). A surviving mutant on the tutor user id re-key, the
  MATCHED/SUPERSEDED transition, the optimistic-lock 409, or the per-role block-flag check
  (INT-065) is a real assertion gap the ≥97.5% gate can't see.
- **No frontend↔backend contract test (Pact).** application programming interface ROUTES pins paths but not **payload shape**; a backend field
  rename passes both per-repo gates and breaks prod (qa SKILL).
- **Broad JaCoCo **/dto/** exclude** — pom implementation reference. data transfer objects that carry mapper /
  branch logic (null-coalescing, bucketed-budget, label derivation) are excluded wholesale, hiding
  real branches. (Other excludes at pom implementation reference: builders, config/**, *Application,
  common/exception/**, Regression*, *Repository Impl — review dto/** specifically.)
- **No true concurrent double-assign test.** Assignment Lifecycle IT implementation reference states the
  optimistic-409 / supersede side-effects are *unit-covered*, not raced. There is no test that fires
  two assigns at the **same** selection request concurrently and asserts exactly one 2xx + one 409
  (qa edge case: "two users do the same action simultaneously").

---

## 3. Design

### 3.0 Test tiering — pull request guard vs nightly (closes Boss condition C2)
| Tier | Trigger | What runs | Budget |
|---|---|---|---|
| Proposed pull-request check | Pull request | Main Playwright journey in Chromium and the backend smoke-test group. | Target: under five minutes. |
| **Nightly** (@full) | schedule cron | full *IT suite + webkit/mobile Playwright + crown-jewel cancel/404 tail + PIT/Stryker report jobs | unbounded |
| **On-demand** | workflow dispatch | either tier, selectable | — |

The **pull request guard is the crown-jewel journey, NOT the full IT suite** — the full suite would blow the
<5-min budget and flake on a shared-IP rate-limit (SHARED-LESSONS 429). The crown-jewel is the one
fast browser path that, if red, means the revenue funnel is broken (it would have caught INT-016).

### 3.1 Where the composed continuous integration lane lives — the e2e repo, with the compose committed into it
**Decision:** the cross-repo lane lives in **regression implementation reference** (the e2e
repo is the natural home — it already depends on the running stack and on no source). The wrapper
docker-compose implementation reference is moved into the e2e repo as **docker-compose ci implementation reference** so continuous integration can reach
a versioned compose file. The job checks out backend + frontend as **siblings** so the compose
build context: ./intuition-backend / intuition-frontend implementation reference resolve.

Rationale for compose-in-e2e over each-subrepo: the compose builds **both** sibling subrepos —
it cannot live in intuition-backend or intuition-frontend alone without the other being a peer.
The e2e repo is the one consumer that needs all three (backend, frontend, e2e). Keep the frontend build-arg
VITE application programming interface BASE URL and the dev env block (rate-limit overrides, SEED SAMPLE DATA ENABLED is yes)
identical to the wrapper file so continuous integration reproduces the dev stack exactly.


The proposed combined-test setup builds the backend and frontend from their checked-out revisions and starts PostgreSQL 15 and Redis 7 beside them. It supplies temporary test settings, waits for application health and then runs the selected regression groups. Credentials must be supplied separately, never copied from development configuration.


Notes that are load-bearing:
- continuous integration: yes makes Abstract Api IT **fail hard** when the stack is unreachable
  (Abstract Api IT implementation reference Assumptions abort is the *local* path; continuous integration must not silently green a
  zero-IT run — the M-Reg-0 discipline already encoded in regression-be implementation reference).
- Dgroups is smoke is the existing pull request/nightly split (regression-be implementation reference); the new
  critical-journey integration tests (§3.3) must belong to the smoke group for inclusion in the proposed pull-request check.
- Surefire + JaCoCo are **skipped** in this lane (the unit gate is the separate per-repo
  ci implementation reference — do not double-run or double-gate).
- Caching per devops web-research lesson: setup-java cache: maven, setup-node cache: npm.
- Email is disabled (RESEND application programming interface KEY=) — the crown-jewel journey must drive the funnel via
  app actions/web request protocol, never an email link, so it is hermetic.

### 3.1a POLYREPO PAT contract (closes Boss condition C4)
The cross-repo checkout token, specified for storage / scope / consumption / rotation:
- **Storage:** GitHub Actions secret POLYREPO PAT on the e2e repo. Provisioned by the founder;
  never committed; supplied through the configured CI secret reference. The secure remote access deploy key path is
  **never** referenced (encrypted web request protocol checkout + PAT only).
- **Scope (least privilege):** a **fine-grained PAT** limited to in Tuition-be + in Tuition-fe,
  **Contents: Read-only**. No write, no other repos, no admin scopes.
- **Consumption:** actions/checkout@v4 with token value (withheld) for the backend and frontend
  checkout steps ONLY (ref: sp-dev). The e2e self-checkout uses the default GITHUB TOKEN.
- **Rotation / failure mode:** expiry ≤90 days; a 401 on the the private implementation folder checkout step = expired or
  over-scoped PAT → rotate in repo settings (runbook). → **AC A-1a**.

### 3.1b IT path correctness (closes Boss condition C5)
- backend ITs run with working-directory: intuition-backend (the sibling checkout path), invoking Maven
  failsafe over *IT implementation reference under
  intuition-the private implementation folder** (the crown-jewel ITs
  Free Market Assign IT/Assignment Lifecycle IT live under …/regression/flows/**).
- IT reports are **failsafe** reports — upload path intuition-the private implementation folder**
  (NOT surefire-reports; ITs are failsafe-bound). Surefire + JaCoCo are skipped in this lane.
- Playwright specs run from the e2e checkout root over e2e/tests/**; the crown-jewel is
  crown-jewel spec implementation reference. → **AC A-1b**.

### 3.2 Where docker-compose ci implementation reference must live, and the wrapper file's fate
- **Move** the wrapper docker-compose implementation reference content into **docker-compose ci implementation reference** (committed
  to the e2e repo, main). It must keep build context: ./intuition-backend /
  intuition-frontend implementation reference (relative to the e2e checkout root, where the two repos are checked out as
  siblings). Pin Postgres 15-alpine / Redis 7-alpine exactly as the wrapper file
  (docker-compose implementation reference) so continuous integration = dev parity.
- **Single source for continuous integration (closes Boss condition C3):** docker-compose ci implementation reference is the ONE file
  the composed lane consumes (manage the required application services
  copy exists. The wrapper docker-compose implementation reference **stays** for local dev (runbook references it); to
  stop continuous integration↔local drift a **drift-guard step** in the composed lane runs a normalized diff of the
  two files (ignoring the documented path/port deltas) and **fails the lane** if they diverge beyond
  the allowed delta — so a dev-only env change can never silently make continuous integration test a different stack
  (qa: "or they silently drift"). → **AC A-2a**.
- The two wrapper workflows (regression-be implementation reference, regression-fe implementation reference) are **superseded** by the
  single e2e composed workflow; delete them from the wrapper (they run nowhere anyway) to avoid a
  third stale copy.

### 3.3 The crown-jewel Playwright journey (new — Lane B)
Add **crown-jewel spec implementation reference**, tagged @smoke, one journey per file (repo
convention, e2e/README md). Use role/accessible-name locators and Playwright auto-wait — no
fixed sleeps (lint-enforced; qa SKILL). The journey, against the live seeded stack:

1. **Register** a fresh tutee + a fresh tutor (unique emails per run, cf. Free Market Assign IT implementation reference
   random suffix pattern) via the real /register flow.
2. **Tutee posts** a job (Create Request), assert PENDING and visibility in the admin queue; admin approves it and the test asserts ACTIVE.
3. **Complete the tutor profile** under 012's shared completeness predicate, then **Tutor applies** to that ACTIVE job with no listing required. Assert APPLIED. PENDING invitation/interview initiation under021 does not waive the ACTIVE-job Apply guard.
4. **Assign and confirm** — exercise parent choose-tutor → admin selection-request → assign (submit operation for admin selection-requests id assign), winner application→MATCHED; activate/confirm the match before testing the frontend contact button. Match-PENDING still hides that button per INT-125.
5. **Contact reveal** — both parties hit the post-assignment contact reveal and see real details
   (the INT-016 guard, mirrored from Free Market Assign IT implementation reference); assert **no 500**.
6. (Nightly @full) **Cancel** → reopen → contact reveal now **404**
   (Free Market Assign IT implementation reference).

Page Objects extend e2e/pages/ (POM convention). Trace/video retain-on-failure already configured
(playwright config implementation reference). This is the thin top of the Testing Trophy (qa SKILL) — reserved for
exactly this revenue-critical journey; everything else stays at the integration/IT layer.

### 3.4 Branch protection — stop the Dskip Tests image outrunning the gate
- On intuition-backend (sp-dev) and intuition-frontend (sp-dev): require the existing per-repo
  verify check (INT-064) as a **required status check** before merge/deploy.
- On e2e (main): require the new composed regression / composed check (pull request smoke lane).
- **Cross-repo trigger:** a backend or frontend change must not deploy until the composed lane has run against
  it. Because GitHub branch protection is per-repo, wire one of:
  (a) the composed workflow repository dispatch-triggered from the private implementation folder continuous integration on success (the private implementation folder continuous integration sends
  a dispatch to e2e with the commit SHAs; composed lane checks them out and gates), **or**
  (b) the dev-deploy step (devops runbook) is the gate: deploy is allowed only after a green
  composed run for the SHAs being deployed.
  Decision deferred to build with devops; (a) is preferred (automated, no human gate). The
  **non-negotiable**: the dev image (built Dskip Tests) may be **built** anytime, but must not be
  **deployed/promoted** without a green composed-lane run for those exact SHAs.
- **Deploy-gate SHA check:** embed the expected image SHA with ARG/ENV GIT SHA; before any live-stack test, compare both the container value and public read operation for version JSON field git Sha with the checked-out SHA. Spec027 supersedes the older anonymous /actuator/info probe; that endpoint remains401 for anonymous callers.
- **Fail closed:** absent, empty, unknown or mismatched git Sha fails the lane before the suite runs. A valid response status alone is insufficient.
- The read-only version response must report a non-empty, known build identifier that matches the expected revision; merely exposing a version operation is insufficient.
- The same exact-SHA gate applies before dev deployment is declared verified (AC C-9). Capture the result as evidence; the historical claim that anonymous info access was enabled is superseded.

### 3.5 Test-health hardening (qa)
- **PIT (backend)** scoped to service/Admin Match Service, service/Contact Reveal Service,
  security/Jwt Authentication Filter only (whole-repo PIT is too slow — qa SKILL). Track *mutation
  score*; a surviving mutant on the re-key / status-transition / optimistic-409 / block-flag logic is
  a real gap. Run as a **separate, non-pull request-blocking** job first (report-only), promote to gating once
  baseline mutation score is set.
- **Stryker (frontend)** scoped to the assignment/contact-reveal/auth-guard components only; same
  report-first → gate cadence.
- **Pact contract** at the frontend↔backend seam for the crown-jewel endpoints (apply, selection-request,
  assign, contact reveal): consumer contracts generated from frontend tests, verified against the live
  Spring Boot provider in the composed lane. Use **loose/type matchers** for volatile fields
  (timestamps, generated IDs, bucketed budget) — exact-match on dynamic data is the #1 flaky-contract
  source (qa SKILL).
- **Narrow the **/dto/** JaCoCo exclude** (pom implementation reference): keep excluding pure record/data carriers,
  but **stop excluding data transfer objects/mappers that contain branch logic** (label/bucket/null-coalesce). Read
  each dto/** class before deciding (CLAUDE md: don't add/keep exclusions to close coverage gaps
  without reading the source; an uncoverable branch may be a real bug). Then re-baseline ≥0.975.
- **Concurrent double-assign IT** — new smoke-group integration test (or method on Free Market Assign IT) that
  fires two assigns at the same selection request from two threads and asserts exactly **one 2xx +
  one 409** (optimistic-lock), and that losers end SUPERSEDED. This is the real race the current
  unit-only coverage (Assignment Lifecycle IT implementation reference) does not exercise.

### 3.6 Flake discipline (carried into the composed lane)
- Playwright retries: 1 in continuous integration already records pass-on-retry as a flake (playwright config implementation reference);
  a pass-on-retry is a defect signal, log it (qa SKILL).
- Seed-dependent ITs: **reseed before the run** (the lane seeds via SEED SAMPLE DATA ENABLED is yes
  on a fresh compose), never retry — the flake is environmental (qa SKILL;
  regression-ITs-live-stack memory). Also clear ratelimit:* keys if the suite trips per-IP limits
  (SHARED-LESSONS 429 lesson) — the dev compose already sets high RATELIMIT * limits
  (docker-compose implementation reference), mirror that in docker-compose ci implementation reference.

---

## 4. Acceptance criteria

**A. Composed continuous integration lane exists and runs on GitHub**
1. regression implementation reference exists in the e2e repo (main), checks out backend +
   frontend as siblings, boots the stack via docker-compose ci implementation reference, and runs **both** Lane A
   (black-box ITs) and Lane B (Playwright). **unverified-pending-run** until the first GitHub run is
   confirmed green (no gh locally — confirm on GitHub user interface, per INT-064 follow-up discipline).
**A-1a (repository-read credential).** Frontend and backend checkout steps consume a credential supplied by the CI secret store (limited to the required repositories,
Contents:Read-only, in Tuition-be+in Tuition-fe only); the e2e self-checkout uses GITHUB TOKEN;
the secure remote access key path is never referenced. The PAT value is provisioned by the founder, not committed (§3.1a).
**A-1b (IT paths).** backend ITs run working-directory: intuition-backend over
the private implementation folder**; artifacts upload target/failsafe-reports/**;
Playwright runs e2e/tests/** (§3.1b).
2. docker-compose ci implementation reference is committed to the e2e repo and is byte-equivalent (modulo paths)
   to the wrapper docker-compose implementation reference (PG 15-alpine, Redis 7-alpine, dev env block,
   SEED SAMPLE DATA ENABLED is yes, high RATELIMIT *).
**A-2a (single-source + drift-guard).** docker-compose ci implementation reference is the ONLY compose file the
composed lane consumes; a drift-guard diff step fails the lane if it diverges from the wrapper
docker-compose implementation reference beyond the documented path/port delta (§3.2, closes C3).
3. The two wrapper workflows are deleted (they run nowhere); no third stale compose copy remains
   undocumented.
4. With continuous integration is yes, an unreachable stack **fails the lane** (the black-box *IT does NOT silently
   skip) — proven by a deliberate failed-boot run showing red, not green.

**B. Crown-jewel journey is gated**
5. crown-jewel spec implementation reference exists, @smoke, covers register → post → apply →
   assign → contact-reveal-200-both-parties, role/name locators, no fixed sleeps (passes
   eslint .).
6. The pull request smoke lane runs this journey; a forced contact-reveal 500 (INT-016 reproduction) turns the
   lane **red** — i e. the gate would have caught INT-016.
7. The cancel → reopen → reveal-404 step runs nightly (@full).

**C. Branch protection**
8. verify (backend), verify (frontend), and regression / composed (e2e pull request smoke) are required status
   checks on their repos' protected branches.
9. The dev image (built Dskip Tests) cannot be **deployed/promoted** without a green composed-lane
   run for the deployed SHAs (mechanism per §3.4 (a) or (b), decided at build).

**D. Adjacent items**
10. **INT-060** confirmed fixed in Profile test implementation reference (wait-for-prefill + find By Display Value
    guard, 127/134 cite INT-060); README row corrected from OPEN → CLOSED/RESOLVED.
11. **INT-047** confirmed RESOLVED (README already says so; full-Docker-restart playbook in devops
    SKILL); row closed. The composed lane tolerates a buildkit/bake wedge (legacy-builder fallback or
    clean-slate retry) rather than waiting out a hang (diagnose-before-retry).

**E. Test-health**
12. PIT (backend) wired report-only on Admin Match Service/Contact Reveal Service/Jwt Authentication Filter
    with a recorded baseline mutation score; surviving mutants on re-key / status-transition /
    optimistic-409 / block-flag logic are triaged (killed or justified).
13. Stryker (frontend) wired report-only on the assignment/contact-reveal/auth-guard components with a
    baseline.
14. Pact contract for the crown-jewel endpoints verified against the live provider in the composed
    lane, using loose matchers for volatile fields.
15. **/dto/** JaCoCo exclude narrowed — every dto/** class read; branch-logic data transfer objects no longer
    excluded; repeat backend verification and meet the 97.5% line and branch thresholds over the measured scope.
16. A concurrent double-assign IT asserts exactly one 2xx + one 409 and losers SUPERSEDED; runs in
    the smoke lane.

**Global gates still hold** (CLAUDE md): backend verification at 97.5% line and branch coverage with required database services available; frontend build and tests at 97.5% lines and 88% branches over their measured scopes; no magic strings; Liquibase-only;
spotless/eslint clean. Polyrepo: stage source/tests explicitly (frontend tracks coverage/), commit on
the repo's current branch with a meaningful message + Co-Authored-By trailer; push only after
verify + local deploy.

---

## 5. Out of scope / deferred
- Prod (non-dev) deploy pipeline, blue/green, SLO/burn-rate alerting (devops SKILL web-research) —
  separate spec; this spec gates dev.
- SLSA provenance / Action SHA-pinning / Dependabot on the polyrepo — track as a follow-up hardening
  item, not a blocker for closing the S1.
- Webkit/mobile Playwright projects stay nightly (playwright config implementation reference), not pull request-blocking.

---

→ **team-review** (full 18-agent debate) → **Boss** (Rex verifies these claims against the real
files/paths/line numbers above) → **build**.
