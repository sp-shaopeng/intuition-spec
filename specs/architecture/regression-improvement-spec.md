# Regression-Test Improvement Spec (M-Reg Hardening, TATAH-aligned)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** SPEC — team-review pending. NO code built yet. Phased; each phase is independently shippable with
its own gate + (at build time) /team-review → Boss.
**Created:** 2026-06-09. **Owners:** Robin (Regression Engineer, lead) + Reese (quality assurance) + the technical lead (Tech Lead);
Sam (frontend testability), Seb (flake/continuous integration), Taylor (continuous integration topology) as needed.
**Builds on:** [regression-labs-spec md](regression-labs-spec.md) (M-Reg-0…3 + §14 TATAH forward plan),
[reference/branch-coverage-matrix md](../reference/branch-coverage-matrix.md) (the GAP ledger),
specs/sequences/*.md (the Use Cases), and the TATAH discipline now in
SHARED-LESSONS md + qa-engineer/tech-lead SKILL md.

---

## 0. Why — current state in one paragraph
M-Reg-0/1/2 are **built and real**: 32 backend flow IT classes under intuition-the private implementation folder,
10 Playwright specs under e2e/tests/ (incl. the crown-jewel journey), per-repo continuous integration green (backend the recorded revision, frontend #22).
But the **quality contract is not yet honest end-to-end**: the branch ledger (reconciled 2026-06-10) has **24 open
✗ GAP rows** (untested business branches — 5 Tier-1 security/authz/money guards, 1 genuinely-uncovered sequence
#34, ~18 lower-value * NOT FOUND/invalid-status twins; see branch-coverage-matrix md Summary + tier list), the
lab has **no frozen seed** (it depends on
Dev Data Seeder, so determinism is incidental not guaranteed), the **live-stack lab runs in continuous integration only via the composed
lane whose POLYREPO PAT run is unverified** (per-repo continuous integration excludes it by design via ci-no-lab), **M-Reg-3 post-deploy
prod-smoke is spec-only**, and existing tests carry **five-factor-taste debt** (vacuous asserts that pass without
verifying the promise — the INT-077 class). This spec closes those, organized by the TATAH loops.

**TATAH framing (the organizing principle):** **GAP** = a promise (UC/sequence) with no gating test (the ✗ rows);
**MISMATCH** = UC↔System structural drift (4 types) found by a bidirectional scan. The Guard Loop = the lab keeps
promises fulfilled (R1, R2, R3); the Foundation Loop = the UCs stay honest (R5 the linter); continuous integration is the carrier (R4).

---

## Phase R1 (S2, highest value) — Close the 24 ✗ GAP rows: negative-path & status-transition coverage
**Problem:** 24 branches in branch-coverage-matrix md are ✗ (untested) — all guard/error paths the happy-path ITs
never exercise. A promise verified only on the happy path is half-honest. Source of truth = that matrix's reconciled
per-service tables + its tier list (the earlier high-value tranche INTERVIEW LIMIT REACHED/review-integrity/
match-cancellable/post-ownership/account-recovery is already **CLOSED** — do not re-target it).
**Scope — add one assertion per OPEN ✗ row, grouped into the EXISTING flow ITs (no new harness), worked in tier
order so the highest-consequence guards land first:**

- **Tier 1 — security / authz / money / account-integrity (close first):**
  - **Auth/account** (Auth Journey IT): EMAIL ALREADY REGISTERED 409 (Auth Service),
    PHONE ALREADY REGISTERED 409 (106), COMMISSION ACK REQUIRED 400 (144/619),
    one-time passcode ATTEMPTS EXCEEDED 429 (535).
  - **Admin** (Admin User Mgmt IT): CANNOT REMOVE SUPER ADMIN 403 (Admin User Service).
  - **Interview ownership/active guards** (Interviews IT): TUTEE POST NOT OWNED 403 (99),
    TUTEE POST NOT ACTIVE 422 (108), TUTOR POST NOT ACTIVE 422 (147).
- **Tier 2 — genuinely uncovered sequence (no IT today):** #34 tutor-availability-toggle
  (Tutor Post Service set Availability 200) — needs a brand-new case, not just an added assert.
- **Tier 3 — lower-value symmetric * NOT FOUND / invalid-status twins:**
  - **Matching** (Admin Matching IT): TUTOR POST NOT FOUND 404 (96), TUTEE POST NOT FOUND 404 (112),
    MATCH NOT FOUND 404 on activate (211) + cancel (263).
  - **Interview** (Interviews IT): TUTOR POST NOT FOUND 404 (139); invalid-status cancel/link/complete 422
    (217/318/340).
  - **Apply** (Applications IT/Guard Negatives IT): TUTEE POST NOT FOUND 404 (96), TUTOR POST INVALID 400
    (139), APPLICATION NOT FOUND 404 withdraw (163).
  - **Favourite** (Favourites IT): TUTOR POST NOT FOUND 404 (64), TUTEE NOT FOUND 404 (84).
  - **Profile/Review** (Profile IT/Review Submit Journey IT): PROFILE NOT FOUND 404 (Profile Service),
    Review Service MATCH NOT FOUND 404 (78).

(#03 multi-factor authentication login, #30 account-deletion, and the M18-removed TUTOR NO ACTIVE POST guard stay 🚫 — note them,
don't build them.)
**Each test obeys the five-factor taste:** assert the **error code + web request protocol status + that the state did NOT change**
(business-semantic, decidable), not merely a status code; one promise per test; cite the source sequence in the
test's javadoc (traceability).
**Acceptance (reachable exit criterion):** (a) every **Tier-1 + Tier-2 row** (the 6 guards + #34) flips ✗→✓ in
branch-coverage-matrix md with its owning *IT method named — this is the **R1 success KPI: open ✗ on
money/personal information/authz → 0**; (b) Tier-3 twins flip ✗→✓ as a stretch (any deferred Tier-3 row stays ✗ with a one-line
"Tranche C" deferral note, NOT silently dropped); (c) no dead/removed branch is targeted — guards no longer in the
SUT stay 🚫. The exit is **not** "✗ count → 0" (the M18-removed guard and multi-factor authentication/rate-limit are permanently 🚫); it is
"every Tier-1/2 row ✓, every remaining ✗ is either Tier-3-deferred or 🚫-by-design." backend verification is initially report-only
— see Sequencing for the proposed enforcement stage.

## Phase R2 (S2) — Five-factor-taste sweep of the existing 32 IT classes (hunt MISMATCH-3 / vacuous tests)
**Problem:** some existing assertions pass without verifying the promise — the INT-077 "stale JSON path → no value Value
asserts vacuously green after an envelope reshape" class; happy-path-only caps; missing per-recipient/negative asserts.
**Scope:** audit each regression/flows/*IT against the five factors. Concrete checks: (a) every negative/no value Value
assert pairs with a **positive sibling on the same node** so the path is proven to exist (the INT-077 rule); (b) every
cap/limit test has a **boundary + over-boundary** pair, not just happy; (c) every "notifies/sends" test asserts the
**per-recipient count** (the INT-085 contract), not a single "sent"; (d) every assert targets the **business outcome**,
not just 200. Record findings as MISMATCH rows; fix in place.
**Acceptance:**
- **Named deliverable:** a five-factor-audit md table (one row per regression/flows/*IT, five columns =
  Business-Semantic-Correctness / Granularity / Anchor-Mandatoriness / Outcome-Decidability / Traceability-Closure,
  each ✓ or a MISMATCH note) committed under intuition-spec/specs/reference/; every flagged gap fixed in place.
- **Spot-break runbook (proves no vacuous green):** for **3 named representative ITs** —
  Assignment Lifecycle IT (the assign→contact-reveal money path), Review Submit Journey IT (per-participant +
  duplicate guards), and Interview Limit IT (the cap boundary) — run the procedure: *temporarily remove the asserted
  field / flip the guarded branch in the SUT → re-run the IT → confirm it turns **RED** → revert.* Record the RED
  output line for each in the audit table. An assertion that stays green under its break is the INT-077 vacuous-test
  defect and must be rewritten before R2 closes.

## Phase R3 (S2) — Frozen, deterministic regression seed (decouple the lab from Dev Data Seeder)
**Problem:** regression-labs-spec md §3.3 mandates a frozen seed at * implementation reference;
it is **not on disk** — the lab relies on Dev Data Seeder, so a seeding change can silently shift fixtures and the
suite's determinism is incidental. (We already hit seed drift: INT-018 password drift, the matches tutor user id
re-key, the repo-IT FK fixtures.)
**Scope:** create the frozen seed (00-reset implementation reference, 10-users implementation reference, 20-posts implementation reference, 30-graph implementation reference) as the immutable
fixture the live-stack lab loads, independent of dev seeding; a Seed Manifest IT (one exists — extend it) asserts the
exact expected row counts/ids so drift fails loudly. Keep Dev Data Seeder for local dev only.
**Pin contract (Seed Manifest IT, exact — not greater Than):** assert the manifest by **exact** values, e g.
users equals N, active Tutor Posts equals N, active Tutee Posts equals N, matches equals N, plus the **stable ids** the flow ITs
depend on (the seeded tutor/tutee/admin account ids, the crown-jewel pair). The current Seed Manifest IT asserts
only seeded equals yes + the greater than operation — R3 tightens those to exact pins so an off-by-one seed change fails RED.
**Synthetic-personal information only:** the frozen database query language contains **only synthetic** emails/phones/names (e g. @regression test
addresses, +65 8000 00 NN numbers) — never a real or production-shaped personal information value; reviewed at commit. The seed is
committed test fixture, so it must be safe to read in any continuous integration log.
**Schema-evolution procedure (so the frozen seed can't rot):** when a Liquibase changeset adds/renames a column the
seed touches, the same pull request must (1) update the frozen * implementation reference, (2) update the Seed Manifest IT pins, (3) re-run the lab
green — enforced because a schema change that skips the seed update makes Seed Manifest IT RED (loud, not silent).
Document this three-step in the seed dir's README.
**Acceptance:** the regression lab loads the frozen seed (not Dev Data Seeder); Seed Manifest IT pins exact
counts/ids; the database query language is synthetic-personal information-only; a deliberate seed change (or an unsynced schema change) makes it RED, not
silently-green.

## Phase R4 (S1, reliability backbone) — Finish the continuous integration topology so the live-stack lab actually gates
**Problem:** the live-stack *IT lab + the Playwright crown-jewel run **only locally or in the composed lane whose
POLYREPO PAT GitHub run is unverified** (INT-064 tail); per-repo continuous integration excludes the lab (ci-no-lab, by design);
**M-Reg-3 post-deploy prod-smoke is spec-only** (no post-deploy-smoke implementation reference). So a regression in a cross-repo journey
can still merge.
**Scope:** (a) verify + green the composed regression implementation reference run (provision POLYREPO PAT;
confirm the crown-jewel @smoke + backend black-box @smoke run against the booted stack with the deploy-SHA gate);
(b) wire **M-Reg-3**: a post-deploy-smoke lane (Newman over postman/ + Playwright @smoke, read-path only) that
hard-gates the virtual machine/Railway deploy and railway rollbacks on red; (c) confirm the nightly @full cron actually fires.
This **completes INT-064** (close it once the composed run is green-verified) and makes the crown-jewel a real gate.
**Acceptance:** a green composed-lane run is linked in INT-064; a pull request that breaks the crown-jewel journey is blocked;
post-deploy smoke runs against the dev virtual machine and fails closed on a stale/broken deploy.

## Phase R5 (S3, the TATAH payoff — advisory-only, biggest build) — UC-linter + GAP report automation
**Problem:** the bidirectional UC↔System scan + GAP ledger is **manual** (the branch matrix is hand-maintained); drift
re-accumulates. This is regression-labs-spec md §14 promoted from "forward note" to a planned, separately-gated phase.
**Scope (advisory-only, NEVER auto-gating — an LLM/scan can be wrong; a human ratifies):**
- A **uc-linter** that does the bidirectional scan: System→UC (every controller endpoint + Global error handler
  error code has a sequences/ entry → flag MISMATCH-1 Scope-Underspec / M3 Branch-Underspec) and UC→System (every
  sequence step maps to a real endpoint → M2 Scope-Overspec / M4 Branch-Overspec). Output: a drift report reconciled
  into branch-coverage-matrix md.
- A **GAP report** generator: the ✗ matrix rows surfaced as a release signal (an open GAP on a money/personal information/revenue
  promise = a promotion blocker), wired into the nightly lane as a report (not a hard gate initially).
**Acceptance:** the linter reproduces (and would have caught) at least the INT-082/INT-009 branch-underspec drift and
the current ✗ GAP set; output is advisory + human-ratified; no false-gate. **This phase gets its own spec + team-review
before build** — listed here for sequencing, not greenlit by this spec.

---

## Sequencing & gates
- **Bundle A (near-term, high-value/low-risk):** R1 + R2 + R3 — close the GAPs, fix taste debt, freeze the seed. One
  /team-review + Boss; each ✗→✓ verified in the matrix.
  - **R1↔R4 gating coupling (the load-bearing caveat):** the new R1 ITs live in regression/flows/ and the per-repo
    backend continuous integration **excludes that path by design** (ci-no-lab profile, pom implementation reference + ci implementation reference). So R1's tests
    **do NOT gate the per-repo pull request build** — they run in the live-stack composed lane (e2e/.github/workflows/
    regression implementation reference), which is exactly what **R4/Bundle B** is finishing (INT-064 unverified). Until R4 lands, R1's
    suite gates **only locally + in the composed lane**; the per-repository backend verification and coverage gate is
    unaffected by these ITs (they are excluded from JaCoCo's lab path). **Do not claim R1 "gates on every pull request" — it
    does not until R4 wires the composed lane as a required check.**
  - **Report-only band first (inherited from regression-labs-spec md §8):** when R1's ITs do enter the composed
    lane they start **report-only for 2 weeks**, auto-promoting to a hard pull request gate only at **<2% flake over 50 runs**.
    New negative-path ITs are exactly where flake hides, so they earn the gate, not assume it.
  - **Flake budget (inherited, regression-labs-spec md §7):** the <2%-flake-over-50-runs ceiling applies to every
    R1/R2 IT before promotion; a flakier test is quarantined, not gated.
  - **Success KPI (Bundle A):** **open ✗ GAP on money / personal information / authz → 0** (the matrix Tier-1 set + #34), the R2
    five-factor-audit table published with 3 spot-break REDs recorded, and the frozen seed loaded with exact pins.
- **Bundle B (reliability):** R4 — finish continuous integration topology + M-Reg-3; closes INT-064 and is what turns R1's suite into a
  real pull request gate. Owner Taylor/Seb.
- **Bundle C (strategic, separately specced):** R5 — the uc-linter automation. Advisory-only.
- Each bundle: build → gates → /team-review → Boss → deploy (per-repo continuous integration green; the live-stack band runs in the
  composed lane, report-only until it earns promotion) → push. The branch matrix is the running ledger updated every
  bundle.

## Out of scope / non-goals
- No change to the WhatsApp OFF toggle or the INT-082 PE gates (the one-time passcode-enqueue PE-3 observation stays under INT-082).
- #03 multi-factor authentication-login (needs a time-based one-time passcode fixture) and #30 account-deletion stay deferred per existing decisions — R1 notes
  them as known exclusions, does not build them.
- R5's agentic tooling is advisory-only forever (never a hard merge gate); a wrong UC must not block a correct change.
- Pact consumer contracts remain deferred (regression-labs-spec md §14).

## Verification (definition of done for the SPEC, this pass)
The spec exists, is internally consistent (every ✗ row it targets is a real branch-coverage-matrix md row; every
named IT/sequence/continuous integration file exists), is phased with testable acceptance criteria, and decides the TATAH framing. **No
production or test code was changed for that historical specification pass. This was a documentation review, not a new application test run.
Next step: /team-review on Bundle A → Boss → build.
