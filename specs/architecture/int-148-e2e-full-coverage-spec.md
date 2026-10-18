# INT-148 — Full e2e flow coverage (Playwright) + a coverage metric

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Endpoint-touch coverage is distinct from critical use-case, permission and transition coverage. Its original Postman denominator is a dated snapshot; update a versioned endpoint→requirement/test map through027 before claiming current full coverage. Existing percentages do not certify later features. See [current coverage review](../reviews/2026-09-05-requirements/alignment/coverage.md). Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Status:** SPEC — pending team-review + Boss audit before build
**Date:** 2026-06-13
**Authors:** qa-engineer (lead) · frontend-expert (browser flows) · bff-specialist (application programming interface funnels) · data-engineer (coverage metric)
**Owner repo:** e2e/ (polyrepo, main). No app-code change.
**Source of truth for flows:** the postman collection (In Tuition postman collection implementation reference) → 109 unique endpoints across 21 modules; this is the coverage DENOMINATOR.

> **Why now.** The lab guards the crown-jewel funnel + a handful of recent fixes, but there was **no way to
> say how much of the product is covered.** Measured baseline (the new coverage/coverage mjs): **15.6%
> (17/109)** flow coverage. M6 Interview (0/20), M2 Posting, M4 Matching, M5 Commission, M7 Profile, M9–M16
> are largely uncovered. This spec defines the metric, sets a gate, and plans the waves to raise it.

---

## 1. The coverage metric (the "find a way to determine coverage" deliverable)

For a black-box lab that tests the DEPLOYED system over web request protocol, **line coverage is the wrong metric** (we don't
import app source). The right metric is **FLOW coverage**: of the application programming interface flows the product exposes, how many does
≥1 spec exercise?

**Mechanism (committed, re-runnable, continuous integration-gateable):**
- flows implementation reference — the canonical flow inventory (method + normalized path + module), generated from
  the postman collection and committed so the lab is self-contained (continuous integration needs no wrapper file).
- e2e/coverage/coverage mjs — scans tests/ + pages/ and marks a flow covered when a spec either
  (1) calls the operation directly in the test, or
  (2) **declares it** via a // @covers METHOD ... interface operation annotation — the honest, reviewer-checkable bridge
  for endpoints a BROWSER spec exercises indirectly (a page render that fetches them).
  Path comparisons treat concrete record identifiers and placeholder identifiers as the same route, so a test call can be matched to the operation inventory.
- Output: a per-module table + overall %, with min `<pct>` to **fail continuous integration below the gate** and gaps to list
  uncovered flows.
- Provide a coverage report and a separate minimum-coverage check. Add them to the frontend-regression
  workflow as a non-blocking report first, then a hard gate at the agreed floor.

**Honesty rules (Boss-auditable):** a @covers annotation is only legitimate if that spec genuinely drives the
endpoint through the running stack (a real page render or a real click that triggers the fetch). @covers must
never be used to claim a flow no test actually exercises — the Boss spot-checks annotations against the spec body.

> **Team-review folds (2026-06-13 → Boss SEND BACK, 3 conditions):**
>
> **(1) Denominator fidelity (Blake).** The inventory is postman-derived = **109** endpoints; the REAL controller
> surface is **113** controller route declarations, counted from the private source
> — so postman is **~96%** of the true surface (4-endpoint gap). Acceptable as a proxy, but it means a green
> 100% could hide ≤4 flows. **Action:** treat postman as the working denominator, BUT cross-check the count
> against the controller-mapping grep whenever flows json is regenerated, and prefer regenerating flows json from
> the controllers/OpenAPI once available. Record the 113-vs-109 delta in the coverage README so the gap is visible.
>
> **(2) Reachability ≠ correctness (Reese).** This metric measures whether a flow is **exercised at least once**,
> NOT that its contracts are verified. A 60% coverage with all-200 happy paths is a FLOOR, not proof of quality.
> **Action:** every MUST-module flow must additionally exercise its **critical negative/transition cases** — e g.
> matching: create→activate→cancel→reopen + the 409/422 conflict; auth: wrong-password lockout + wrong-multi-factor authentication;
> applications: withdraw + re-apply; assignment: cancel→reopen + 401/403/404. The wave specs list these per flow;
> the Boss checks that MUST coverage is transition-complete, not just endpoint-touched.
>
> **(3) README + continuous integration (Chris/Taylor).** Add a coverage/README md (how to read the matrix, the @covers contract,
> the historical inventory-count discrepancy) and the report and minimum-coverage checks; add the latter to the
> frontend-regression workflow — **report-only first**, ratchet to a hard min per §5.

## 2. Mo S Co W prioritization of the 92 uncovered flows

**MUST (revenue + security critical) — target 100%:**
- M1 Auth — login, multi-factor authentication verify, logout, forgot/reset password, change password, delete account, register (×2), /me.
- M2 Posting (tutee + tutor) — create/list/get/update/close using soft-delete/availability.
- M3 Approval — pending queue, post detail (personal information), approve, reject, active jobs.
- M4 Matching — list, create, activate, cancel.
- M5 Commission — pay commission (the money endpoint).
- M7 Profile — get/update tutee + tutor profile.
- M8 Match Dashboard — tutee + tutor matches (the humanisation surface, INT-137a).
- M15 Applications — apply, list, withdraw, applicants using admin.
- M17 Marketplace Detail — tutor detail, assignment detail, browse assignments.
- M18 Assignment Lifecycle — applicant cards, choose tutor, selection queue, reveal contact, invite quota/invite/withdraw.

**SHOULD — target ≥80%:**
- M6 Interview (20) — request, schedule, meeting-link, complete, cancel, no-show, AND the INT-039 timeslot
  negotiation (propose/accept/decline/counter → PENDING SCHEDULING → admin schedule). Largest single gap.
- M9 Notifications — list, unread-count, mark-one-read, mark-all-read (+ INT-025 POST SUBMITTED to admins).
- M10 Favourites — add, bulk-probe, remove, list.
- M11 Reviews — submit, my-reviews using ×2, public reviews.
- M2 Public Browse + M14 Landing Showcase — browse tutors/requests, showcase strips.

**COULD — target ≥50%, last:**
- M12 Admin Analytics (date-range, INT-045), M16 Admin Management, Admin Moderation (INT-065).
- INT-082 WhatsApp / phone-verification — feature-toggled **OFF**; cover the OFF contract only (endpoints
  gated/no-op), not the live WhatsApp path.

## 3. Test-style assignment (per the lab's two idioms)

- **application programming interface-funnel (Journey Api over [REDACTED IP ADDRESS])** — multi-role state machines + status/shape contracts: auth,
  posting, approval, matching, commission, applications, assignment lifecycle, interview, profile, favourites,
  reviews, notifications, admin. Deterministic + hermetic; the bulk of coverage. Shared arrange in Actors implementation reference.
- **Browser (Playwright MCP-authored)** — render/journey-specific behaviour: humanisation (no raw codes),
  login-gate redirects, dashboard tabs (INT-049), nav IA, the match-dashboard humanised cards, mobile 390px.
  Authored by exploring the live app with the Playwright plugin, then committed as POM-backed specs; browser
  specs credit their indirect endpoints via @covers.

## 4. Wave plan (spec → implement → measure → repeat)

Each wave: implement the specs, run them green against the live stack, then regenerate the coverage report and record
the lift. One journey per file; @smoke for the critical subset, @full for the long tail.

- **Wave 1 (MUST, application programming interface-funnel):** auth lifecycle, posting (tutee+tutor) create, read, update and delete+close, approval, matching+commission,
  profile, match dashboard, applications, assignment-lifecycle remainder. Target: MUST ≈ 100%.
- **Wave 2 (SHOULD, application programming interface-funnel):** interview full lifecycle + INT-039 negotiation, notifications, favourites,
  reviews. Target: overall ≥ 70%.
- **Wave 3 (browser, MCP-authored):** match-dashboard humanisation, dashboard-tab IA, public browse render,
  showcase strips, mobile 390px sweep — with @covers. Target: overall ≥ 80%.
- **Wave 4 (COULD):** admin analytics/management/moderation, WhatsApp-OFF contract. Target: overall ≥ 85%.

## 5. Targets & gate

- Land the gate at **min 60** immediately after Wave 1–2 (report-only before), then ratchet to **80** after
  Wave 3. A flow dropping below the gate fails continuous integration (a deleted/❌ test surfaces as a coverage regression).
- MUST-module flows must reach 100% before the gate ratchets — partial MUST coverage blocks the ratchet.

## 6. Verification

- The report shows coverage by operation, a gap list names every uncovered flow, and the minimum check fails below the agreed threshold.
- Every new spec runs green against the live stack (application programming interface BASE URL is http:// REDACTED ip address  BASE URL is http:// REDACTED ip address ).
- Browser specs verified with the Playwright plugin (real render, captured snapshot) before commit; @covers
  annotations spot-checked by the Boss against the spec body.
- Code-rule checks pass; browser tests wait for observable conditions instead of fixed delays; shared setup is kept in one place.

## 7. Ship

/team-review → Boss → implement waves (measure coverage each wave) → push main. Definition of done per wave =
specs green + coverage lift recorded + pushed.
