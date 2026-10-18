# Batch Fix Review — 2026-06-03 (Phases 2-3: team review + Boss + code review)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Scope: the Tier A–C fix batch — INT-015, INT-016, INT-017, INT-020, INT-021, INT-032, INT-033,
INT-034 (frontend stopgap), INT-035, INT-036, INT-037, INT-038, INT-040, INT-041, INT-042.
(INT-025 deferred; Tier-D features each get their own spec.)

## Code review (per-change)

- **INT-016 — Admin Match Service cancel Match** ✅ Captures former Winner + tutor Subject/tutee Subject
  BEFORE the the modifying operation reopen Applications clears the context; builds the
  response from primitives (no lazy re-touch). Record field order verified against Admin Match Item Response.
  Live 200 + ITs green. Other callers (create Match to Response, activate Match) only touch the get id operation
  after their clear or have no clear → unaffected.
- **INT-015 — migration 0031** ✅ CREATE OR REPLACE of the guard fn; permits REJECTED/EXPIRED→CLOSED ONLY,
  still blocks CLOSED→* and REJECTED→ACTIVE. Rollback restores the original. Soft-delete preserved (no hard
  delete) — aligns with the founder soft-delete-only principle.
- **INT-017 — Dev Data Seeder** ⚠️→✅ **FINDING (fixed in review):** the handle was built from
  the first name operation, but seed first names contain spaces (e g. Wei Jie) → a handle
  with a space, and name-derived handles risk collision. **Fixed:** derive from the unique email local part,
  sanitized to a-z0-9, well under the 40-char column. (Lands on the FINAL reseed.)
- **INT-035 — schedule meeting link** ✅ Optional meeting Link (≤500) on Schedule Interview Request; set in
  schedule Interview only when non-blank; standalone PATCH untouched. All 5 record constructor call-sites in
  Interview Service Test updated; new branch test added.
- **INT-033 — admin Job Applicants** ✅ New Admin Layout route + page reads r data.data (Page Response items
  field is data); link repointed off the TUTEE-only route. Test added (incl. error/empty states).
- **INT-036** ✅ Both interview lists read r data.data. **Riley's note actioned:** audited other list pages —
  Applicants implementation reference already correct; no other r data-as-array misuse found.
- **INT-034 frontend stopgap** ✅ Button disabled + note when tutor Post Id equals no value; test added. Full backend fix tracked.
- **INT-020** ✅ pool:forks — full 1234-test suite runs without the deadlock (validated).
- **INT-021 + sibling drift** ✅ Scoped multi-match queries; fixed Selection Request Queue (code AMATH OLEVEL
  + admin link) and Stale Commission List (aria-label) surfaced by the full-suite run.
- **INT-041/042 (Register)** ✅ Phone strips non-digits on input; confirm Password refine, not sent to application programming interface
  (payload built field-by-field). Tests cover spaced-phone-accepted + mismatch-blocks-submit.
- **INT-037/038/040/032 (nav/layout)** ✅ Role-honest; Nav coverage-excluded but tests added for behavior.

## Team review (debate)
- **Jordan (SWE):** INT-016 was reproduced before patching (Sev-1 rule) — the real cause (clear Automatically
  detach) differs from the tracker's open-in-view guess; fix is correct and minimal. ✅
- **Dana (DBA):** 0031 is a clean CREATE OR REPLACE, idempotent, with a real rollback; no data migration
  needed. Numbering sequential after 0030. ✅
- **Casey (Security):** No personal information surface change; soft-delete keeps the audit trail; admin Job Applicants is
  ADMIN-gated (existing endpoint). ✅
- **Reese (quality assurance):** Every backend fix maps to a named IT now green post-reseed; frontend full suite green incl. coverage.
  The handle finding shows the review caught a real defect pre-deploy. ✅
- **Sam (Frontend):** r data.data reused consistently; no silent no-op left (INT-034 stopgap). ✅
- **the technical lead (Tech Lead):** Ship the batch; INT-025 + Tier-D correctly deferred to their own specs. ✅

## Boss (Rex) — verdict
Verified against files: Admin Match Service cancel Match (pre-clear capture), 0031 trigger fn, Dev Data Seeder
handle (now email-derived/sanitized), Schedule Interview Request/schedule Interview, Job Applicants route.
The one material defect (space in seeded handle) was found and fixed in review. **Verdict: ✅ SHIP** with
conditions: (1) the handle fix lands on the FINAL reseed (not the interim one); (2) the documented operational check green against
the freshly-seeded stack before declaring done; (3) INT-025 + Tier-D remain OPEN with their own specs.
