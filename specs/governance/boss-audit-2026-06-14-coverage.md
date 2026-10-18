# Boss audit — e2e coverage depth (2026-06-14)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Auditor:** Rex (Boss). **Scope:** the "100% coverage" claim vs the founder's bar (pre→action→post,
negatives, role-based access control, transitions). Verified against the real e2e/ specs, coverage/, and the controllers.

## Verdict: ⚠️ SEND BACK

"108/108 = 100%" is **endpoint REACHABILITY**, not flow coverage. It proves each endpoint is referenced
by ≥1 spec — it does NOT prove the before/after state, the negative/role-based access control matrix, or the state machines.
The founder is correct: there are far more FLOWS than endpoints, and the depth bar is not met.

## Verified findings (grounded in the files)

1. **Pre-state not captured (the founder's exact example).** auth-lifecycle spec implementation reference changes the
   password and asserts old-fails/new-works, but never explicitly logs in with pw1 FIRST, and has no
   logout-between. The required shape is: **login using pw1→200 → change → logout → login using pw1→401 →
   login using pw2→200**. Most specs assert only the action's status + a single after-read.

2. **role-based access control is essentially untested.** The entire suite has **one** the to be operation (blocked-user login).
   Untested across the board: a TUTEE hitting /admin/** → 403; a tutor editing another tutor's post
   → 403/404; a non-participant calling the contact-sharing route for the selected match → 403; cross-tenant reads. This is the
   single biggest hole — authz is a security contract and it's unverified.

3. **State machines are sampled, not covered.** Posting (DRAFT/PENDING/ACTIVE/MATCHED/CLOSED/REJECTED/
   REVISION REQUESTED), Matching (PENDING/ACTIVE/COMPLETED/CANCELLED + reopen), Interview (9 states).
   Specs walk ONE path; the illegal transitions (409/422) and reopen/revision paths are mostly absent.

4. **Cross-cutting flows absent:** idempotency REPLAY (same key → same result, not double-effect),
   quota EXHAUSTION (interview cap 3, invite quota 10 → the (N+1)th rejected), rate-limit LOCKOUT
   (5 bad logins → locked), soft-delete PRESERVATION (row still exists/queryable after "delete"),
   pagination correctness, and concurrent conflict beyond the one assign race.

5. **The metric credits skipped/failing tests.** Coverage is a static scan of spec source, so a
   test skip (database-gated) or even a failing test still counts the endpoint. The multi-factor authentication spec once counted
   while failing. Coverage ≠ green.

6. **Inventory < real surface.** 108 (postman-derived) vs ~113 controller @*Mappings. A few real
   endpoints (and request-revision / reopen variants) are not in flows implementation reference at all → invisible.

## Conditions (SEND BACK)

- [ ] **C1 — Depth standard:** every owned module's specs adopt **pre→action→post**: capture the
  before-state, act, assert the after-state. Start with auth (the founder's change-password example).
- [ ] **C2 — role-based access control matrix:** add a per-area authz suite — each protected endpoint refused for the wrong
  role/non-owner (403) and unauthenticated (401), existence not leaked (404 vs 403 per the contract).
- [ ] **C3 — Transition matrices:** for M2/M3/M4/M6/M18, assert the illegal transitions (409/422),
  reopen, and revision-request paths, not just the happy path.
- [ ] **C4 — Cross-cutting:** idempotency replay, quota exhaustion, rate-limit lockout, soft-delete
  preservation, pagination.
- [ ] **C5 — Honest metric:** add a SECOND dimension to coverage mjs — a per-module **flow checklist**
  (happy / negative / role-based access control / transition / cross-cutting) so "depth" is visible, and make the gate
  count only flows whose spec is green (exclude .skip).
- [ ] **C6 — Inventory:** regenerate flows implementation reference from the controllers (not postman alone) so the
  denominator is the true surface; add the missing endpoints.

## Owners (per the ownership matrix)
Each condition is delivered by the module's owner; qa-engineer owns C5 (the metric) and the depth bar;
security-expert owns C2 (role-based access control). The Boss re-audits each module against pre→action→post before SHIP.

**Next milestone:** auth rebuilt to the depth bar as the exemplar + the role-based access control matrix landed; then
module-by-module by owner. Re-audit after the exemplar.
