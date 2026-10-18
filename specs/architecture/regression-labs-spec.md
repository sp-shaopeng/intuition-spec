# Regression Labs — Detailed Build Spec (M-Reg)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** The explicit crown-jewel pull request guard in [e2e-ci-and-test-health](e2e-ci-and-test-health-spec.md) and [INT-064](int-064-multirepo-ci-spec.md) supersedes any earlier report-only treatment of that lane. Retain fail-closed Docker/deployed-SHA behavior and keep the remaining lab/nightly scope distinct. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


Status: **SPEC — team-reviewed + Boss-cleared to build (M-Reg-0 first)** · Author:
Regression Eng (Robin) + Architect + Dev Ops · Date: 2026-05-30
See §12 for the LOCKED review decisions (they override §1–§11 where they differ).
Supersedes the proposal (regression-labs-proposal md) with the **founder-approved
bundle**:
- frontend = **Playwright**; backend = **REST Assured + Testcontainers (real Postgres/Redis)**;
  Postman/**Newman** = post-deploy smoke (not the gate).
- **Labs only now**; AI regression agent deferred + **advisory-only** when it lands.
- **Sequential**: M-Reg-0 foundation fix FIRST, then labs.
- Gating: end-to-end smoke **report-only → auto hard-gate at <2% flake / 50 runs**; the
  **post-deploy prod smoke hard-gates from day 1**.
- Pact deferred.

> Current state (verified): intuition-backend has only the jacoco plugin (no
> failsafe/surefire/build-helper); tests live in the private implementation folder`<layer>`;
> the repo @Data Jpa Tests **self-skip** via the assume yes operation;
> postman/ exists at repo root; no .github/workflows/ yet; frontend uses Vitest (Playwright
> only transitively present).

---

## 1. Phases (build order)

| Phase | Deliverable | Gate to exit |
|-------|-------------|--------------|
| **M-Reg-0** | Foundation fix: kill the self-skip, green the JaCoCo branch gate | branch ≥0.975 with a clean report; repo ITs **fail-hard** (not skip) when Docker absent in continuous integration |
| **M-Reg-1** | **backend regression lab** (REST Assured + Testcontainers, failsafe) + frozen seed | backend smoke green locally + in continuous integration against real Postgres |
| **M-Reg-2** | **frontend regression lab** (Playwright e2e/) + testability groundwork | frontend smoke green vs compose stack |
| **M-Reg-3** | **Post-deploy prod smoke** (Newman + Playwright @smoke) | hard-gates Railway promotion |
| (later) | AI regression agent — advisory-only | out of scope here |

---

## 2. M-Reg-0 — Foundation fix (BEFORE any lab)

The self-skip is the bug that let the M17 500 reach prod. Fix the foundation first.

### 2.1 Replace the self-skip with a continuous integration-fail-hard base
- New shared base Abstract Integration Test (Testcontainers @Container pg15 + redis7,
  a test-provided service connection/@Dynamic Property Source).
- **Docker-availability policy:** local (no continuous integration env) → may skip with a loud warning;
  **continuous integration is yes → MUST FAIL, never skip.** A required env-driven assertion: in continuous integration a missing
  Docker daemon is a hard error, so the real-database query language path can never silently green again.
- Migrate the 6 existing assume yes repo tests onto this base (delete the assume yes).

### 2.2 Green the JaCoCo branch gate (0.93 → ≥0.975)
- Cover the M17 branches the gate is missing: Public Browse Controller (rate-limit +
  pagination-cap arms), Security Config matchers, and the Tutor Post Repository Impl
  search impl (0/55 lines today). Cover it with a **surefire** @Spring Boot Test+
  Testcontainers *Test (runs under the gate's exec) — NOT via failsafe ITs, which are
  non-gating (see §12.2). The surefire-only gate stays the single coverage authority.
- Acceptance: the documented operational check prints BUILD SUCCESS with branch ≥0.975 and a non-partial
  jacoco csv.

---

## 3. backend Regression Lab (M-Reg-1)

### 3.1 Stack & execution
- **REST Assured** driving a real app: the spring boot test operation +
  Testcontainers (pg15, redis7) — the SAME database query language path that produced the enum-500 runs.
- **Maven separation:** add maven-failsafe-plugin → integration tests named
  ***IT implementation reference** run in the integration-test/verify phase; maven-surefire-plugin
  keeps running unit *Test implementation reference in the test phase. Unit coverage gate stays on
  surefire; IT get their own (non-gating-at-first) report.
- **Tags:** JUnit 5 the tag operation / the tag operation; failsafe Dgroups is smoke for pull request.

### 3.2 Auth/cross-site request forgery protection + assertions (the backend seam)
- A reusable Api Client (REST Assured spec) that performs the real handshake:
  read operation for auth csrf → capture XSRF-TOKEN + auth cookie jar → echo X-XSRF-TOKEN on
  mutations. Helpers for login-as using tutor/tutee/admin.
- Assert the **serialized web request protocol body** (catches Lazy Init-style serialization bugs) AND,
  where needed, query the database to prove state. Deserialize a page of results to typed
  records (no JSON-path archaeology).

### 3.3 Frozen regression seed (shared)
- A **versioned, immutable** fixture — NOT Dev Data Seeder (which mutates with dev
  needs). database query language files applied per test class, reset between classes (truncate + reload, or
  per-test transaction rollback where isolation allows). Lives in
  the private implementation folder (see §6). Deterministic ids, clock, data.

---

## 4. frontend Regression Lab (M-Reg-2)

### 4.1 Stack & execution
- **Playwright** in a **top-level e2e/ workspace** (own package implementation reference — it tests the
  *deployed system*, not the SPA in isolation). Project: chromium (smoke); webkit +
  mobile viewport added for full.
- Runs against a **the documented operational check stack** (backend+pg+redis+frontend) — the one place
  compose is needed. playwright config implementation reference web Server or an external-stack URL.
- **Tags:** title-grep @smoke / REDACTED password @smoke on pull request.

### 4.2 frontend testability groundwork (prerequisite — Sam)
- Add stable **data-testid** to: login email/password/submit, each showcase card, each
  gated CTA, nav intent links (none today → specs would lean on brittle text selectors).
- **storage State auth fixtures** (login once per role, reuse) — fast, non-flaky.
- Deterministic content via the **frozen seed** (so showcase assertions are stable).

### 4.3 Patterns
- **Page Object Model** (pages/) — one class per screen; specs read as journeys.
- Auto-wait only — **no fixed sleep** (ESLint-banned in e2e/). retain-on-failure
  traces/video.

---

## 5. continuous integration/continuous deployment topology


Implementation example omitted. This public edition retains the written requirements in this section.


- Real pg15/redis7 everywhere (no self-skip). Secrets via GH Actions Environments/OIDC;
  a **dedicated Google test OAuth client** for M18 (never prod creds); ephemeral
  per-run signed session token/encryption keys. Artifacts: Playwright trace/video retain-on-failure
  (7d pull request / 30d nightly); backend failing-case logs + database snapshot.

---

## 6. RECOMMENDED FILE STRUCTURE (how to maintain the two labs)

The two labs live where their tooling is idiomatic — **backend inside the Maven module**
(shares entities/application programming interface ROUTES/Testcontainers), **frontend as a top-level module** (cross-origin,
tests the deployed system). One folder = one lab = one owner.


Implementation example omitted. This public edition retains the written requirements in this section.


**Maintenance conventions (so the suites don't rot):**
- **One journey = one file**, named for the flow; tagged @smoke (pull request-blocking subset,
  <5 min) or @full (nightly). New flow → new file in the matching flows/ or tests/
  folder; never append to a mega-file.
- **Shared plumbing only in support//fixtures//pages/** — tests never duplicate the
  cross-site request forgery protection handshake or selectors.
- **The frozen seed is the single source of test data** for BOTH labs; changing it is a
  reviewed change with a version bump.
- **Quarantine lane:** a flaky case moves to a @quarantine tag (excluded from the gate)
  AND auto-opens a ticket with a 14-day expiry period — never silently disabled.
- **Owner:** Robin (Regression Eng) owns both trees, the flake budget, and the runbook.

---

## 7. Flake discipline (Seb — acceptance criteria, not aspiration)
1. No fixed sleep/timeouts — auto-wait only (lint-enforced in e2e/; review-enforced
   in backend IT). 2. Retry-once **with visibility** — a pass-on-retry is recorded as a flake,
   never silently green. 3. Deterministic seed + fixed clock + no shared mutable order.
4. **Quarantine has a expiry period** (14d) + auto-ticket; expiry escalates. 5. **Flake ceiling =
   incident** — if quarantine > N% of smoke, the gate is declared untrustworthy (alert).

## 8. Gating policy
- backend IT smoke + frontend end-to-end smoke: **report-only for 2 weeks**, auto-promote to **hard pull request
  gate** once <2% flake over 50 runs. **Post-deploy prod smoke is a hard gate from day
  1** (deterministic, real config — the layer that would have caught M17).

## 9. Initial test-case catalog (each = one managed case)
@smoke: auth login→me→logout; login return To bounce-back; anon browse tutors/jobs →
detail login-gate; landing both strips render (degraded guard); browse-assignments
(enum-500 + cap) ; post-a-request → appears in showcase. @full: register tutee/tutor +
commission; apply → admin approve → match → interview → review; admin create-admin; M18
Google sign-in (new/returning/link) once built.

## 10. Acceptance criteria
- M-Reg-0: branch gate ≥0.975 green; ITs fail-hard (no skip) in continuous integration.
- backend lab reproduces the enum-500 + Lazy Init classes against real Postgres.
- frontend lab covers the @smoke catalog vs a compose stack, <5 min, <2% flake.
- Post-deploy prod smoke runs after every Railway deploy and can block promotion.
- Both labs runnable with one documented command; runbook covers triage + quarantine.

## 11. Open items — RESOLVED by review
- **O1 →** *IT implementation reference under the private implementation folder + failsafe using includes *IT/surefire(excludes
  *IT); **NO build-helper / separate source root** (Taylor/Boss — operability over
  purist separation for a team new to failsafe).
- **O2 →** truncate+reload per class is the default; per-test txn rollback ONLY for
  read-only journeys (rollback hides commit bugs in mutation flows).
- **O3 →** reused service-container continuous integration job (faster, matches the post-deploy topology),
  not a compose web Server.

## 12. Team Review Decisions (LOCKED — panel + Boss, 2026-05-30)

Cleared to build **M-Reg-0 only**; Boss re-audits at M-Reg-0 exit before any lab code.
These override §1–§11.

1. **File structure / O1 = *IT implementation reference in the private implementation folder** (no build-helper). Plugin set:
   maven-surefire-plugin excludes *IT implementation reference (keeps the unit JaCoCo gate clean);
   maven-failsafe-plugin includes *IT implementation reference, bound to integration-test/verify,
   Dgroups is smoke for pull request.
2. **JaCoCo contradiction RESOLVED:** the 0.975 gate stays **surefire-only**. Cover
   Tutor Post Repository Impl (0/55) with a **surefire @Spring Boot Test+Testcontainers**
   slice (runs under the gate's exec) — NOT by merging the failsafe IT exec into the gate
   (merging masks untested units), NOT by excluding it. Failsafe IT get a **separate,
   non-gating** JaCoCo report. M-Reg-0 exit = gate ≥0.975 via surefire+TC.
3. **Railway gate DOWNGRADED (honest framing):** prod-smoke runs post-deploy → on red,
   **alert + railway rollback + incident**. A true *pre-promotion* hard gate requires a
   **staging environment** (no native Railway gate today) → new roadmap fast-follow. Do
   NOT claim a gate we can't enforce.
4. **Seed contract = a versioned backend endpoint, ADOPTED:** read operation for … regression seed-manifest
   (seed version + known IDs). The frontend lab asserts the seed via this **web request protocol contract**,
   never against the backend's resources/ database query language files (kills the cross-module file
   coupling). One mechanism also satisfies the backend seed-integrity check.
5. **Reese's adds — BLESSED & binding:**
   - **Negative fail-hard test** — prove that in continuous integration with no Docker daemon the ITs go
     **RED, not skipped** (this is the test that proves M-Reg-0 actually fixed the
     foundation). Non-negotiable.
   - **backend authz @smoke** — 401/403 on the tutor's profile-detail page + the selected assignment's page (server half
     of M17), alongside the frontend detail login-gate.
   - **Pagination over-cap negative case** on /public/assignments (not just happy browse).
   - **Flake ceiling = 5% of smoke** → exceed = incident (defines §7.5 "N%").
   - **Pin Testcontainers image digests** (pg15/redis7) so the gate isn't at tag-drift's mercy.
6. **Build order unchanged:** M-Reg-0 (kill 6 self-skips → fail-hard base → negative
   proof test → surefire+TC coverage of Tutor Post Repository Impl → gate green ≥0.975)
   BEFORE M-Reg-1 lab code.

## 13. Coverage map vs documented use-cases (cross-checked 2026-05-31)

> **Branch-level coverage** (every service error/transition branch vs the lab) is tracked
> separately and authoritatively in
> [branch-coverage-matrix implementation reference](../reference/branch-coverage-matrix.md): **136 branches,
> ~36 tested, ~78 untested gaps, ~14 excluded by design.** The flow-level map below is the
> use-case view; the matrix is the exhaustive code-derived view.

Cross-check of the lab against the 28 documented sequences/*.md flows + module behaviours.
**16/28 fully covered, 7 partial, 5 declared exclusions.** Per dev-workflow md A1 (lab-first
gate) this NOT COVERED block must be handed to reviewers alongside the green result — green is
never handed over alone.

**Fully covered (backend ITs):** forgot-reset-password (Password Reset Journey IT), csrf-bootstrap +
login + logout (Auth Journey IT), admin approval / match-create / match-activate / commission-pay /
interview schedule+meeting-link+complete (Admin Post Approval / Matching / Commission /
Interview Ops), tutee+tutor view-matches + public reviews (Dashboard Reviews IT / Detail Authz IT),
notifications (Notifications IT), favourites (Favourites IT), public browse + filters tutors +
assignments (Browse Tutors IT / Browse Assignments IT), tutor apply/withdraw (Applications IT),
admin mgmt / analytics / applicant-view (Admin User Mgmt / Analytics / Applications View),
showcase personal information + Lazy Init (Showcase IT), seed manifest (Seed Manifest IT), profile authz (Profile IT).

**Covered transitively** (precursors, not standalone contract tests): register-tutee/tutor and
tutee/tutor post-creation — every admin-chain run registers + creates + submits these, so a break
surfaces as a chain setup failure.

**Partial — flagged for follow-up:**
- match-cancel **commission cascade** (#12): cancel asserted; commission→CANCELLED cascade is not. **P2.**
- interview **completion paths** (#16): complete asserted; **no-show + cancel** not. **P1.**
- **csrf-protected-write negative** (#25): mutations use cross-site request forgery protection, but no missing-token→403 negative. **P2.**
- encrypted-personal information (#27): personal information *absence* in responses asserted; encrypt/decrypt round-trip not.

**True gaps — documented Complete but not exercised (Tech-Lead-flagged 2026-05-31):**
- **review submission (#19)** — Dashboard only *reads* reviews; POST-review (M11) untested. **P1.**
- **multi-factor authentication login (#03)** — admin-only time-based one-time passcode; needs a time-based-code fixture. **Deferred (P3).**
- **account deletion (M1)** — untested. **Deferred (low frequency).**

**Declared exclusions (by design — not gaps):**
- **rate-limit 429 (#24)** — limiter raised to 100000 in the dev lab on purpose; 429 behaviour out of scope.
- **email fan-out (#26)** — email disabled in dev (email-disabled log); not assertable here.

**ACTIONED (founder 2026-05-31): close P1+P2 now** — review-submit, interview no-show/cancel,
match-cancel commission cascade, cross-site request forgery protection-write negative. multi-factor authentication + account-deletion + the two exclusions
stay deferred/declared.

**✅ CLOSED 2026-05-31 — P1+P2 built, full suite 107/107 PASS, x3 consecutive (was 99):**
- **review submission (#19)** → Review Submit Journey IT (3): a match participant submits 5★ on a
  **COMPLETED** match (201) — completion path is create→activate→pay-commission (pay sets match
  COMPLETED); invalid rating 400; non-participant denied.
- **interview no-show + cancel (#16)** → Admin Interview Ops IT +3: schedule→no-show using actor TUTEE is NO SHOW,
  request→admin-cancel is CANCELLED, no-show-on-unscheduled is 422.
- **match-cancel commission cascade (#12)** → Admin Matching IT cancel Match cascades Commission To Cancelled:
  activate → cancel → commission invoice Status is CANCELLED (live-verified: activate is PENDING, cancel is CANCELLED).
- **cross-site request forgery protection-write negative (#25)** → Csrf Negative IT (@smoke): authed POST the selected saved tutor entry with the
  XSRF cookie but NO X-XSRF-TOKEN header → denied. NOTE: this app returns **401** for a missing cross-site request forgery protection
  header (not the conventional 403); the write is still rejected — minor inconsistency logged, not a hole.
- Still deferred: multi-factor authentication login (#03, needs time-based one-time passcode fixture), account deletion. Still excluded by design:
  rate-limit 429 (#24), email fan-out (#26).

## 14. Forward plan — TATAH-style UC-linter + spec→test GAP/MISMATCH automation (deferred, methodology-aligned 2026-06-09)

The "AI regression agent" line in §1 (deferred, advisory-only) is sharpened by the **TATAH** model (Ant
"Test as the agentic Harness"; adopted as review discipline now — see SHARED-LESSONS md "TATAH
quality-contract model" + qa-engineer/tech-lead SKILL md). TATAH names the quality contract
**UC → TB-UC → TC** and the two dishonesties this lab already fights informally. Mapping to what we have:

- **UC (Use Case · the customer promise)** = specs/sequences/*.md (28 flows). Already our source of truth.
- **TB-UC (Testable Use Case · verifiable form)** = the regression/flows/*IT + the crown-jewel journey,
  anchored to real endpoints with decidable outcomes against the frozen seed (§3.3).
- **TC** = the automated cases themselves.
- **The UC↔test ledger** = reference/branch-coverage-matrix md — this is already a manual GAP report.

**The deferred automation (a future M-Reg-4, advisory-only, NOT built here):**
1. **uc-linter** — a bidirectional **UC↔System scan** that flags the four MISMATCH types and forces UC
   calibration before a sequence enters the lab: **M1 Scope-Underspec** (a controller/endpoint exists with no
   sequences/ entry), **M2 Scope-Overspec** (a sequence step with no implementing endpoint), **M3
   Branch-Underspec** (a thrown error-code/status-transition in a service that no sequence describes — our
   INT-082/INT-009 class), **M4 Branch-Overspec** (a sequence branch the code lacks). Inputs already exist:
   the controllers + Global error handler codes (System side) vs sequences/ (UC side); output reconciles
   into the branch matrix.
2. **GAP report** — every TB-UC promise with no system anchor = the ✗ rows in the branch matrix, surfaced as a
   release signal (an open GAP on a money/personal information/revenue promise blocks promotion).
3. **tbuc-sculptor checklist** — enforce the **five-factor TB-UC taste** (business-semantic-correctness /
   granularity / anchor-mandatoriness / outcome-decidability / traceability-closure) on every new *IT.

**Why advisory-only, never gating (unchanged from §1):** an LLM-derived UC or scan can be wrong; it proposes
matrix rows + flags, a human (Robin/quality assurance) ratifies. The value now is the **vocabulary + discipline** (in the
skills); the tooling is a later, separately-specced, team-reviewed deliverable.
