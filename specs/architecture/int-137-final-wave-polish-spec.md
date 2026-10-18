# INT-137 — Final-Wave Polish & Test-Infra Fixes

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** SPEC v2 — revised per team-review + Boss SEND-BACK (2026-06-13); ready to build
**Date:** 2026-06-13
**Authors:** bff-specialist (137a data transfer object) · frontend-expert (137a frontend) · software-engineer (137b) · qa-engineer (137b/c) · devops-engineer (137c)
**Tracker:** [README implementation reference](../issues/README.md) — INT-137 (S4)
**Severity:** S4 (polish + test-infra; no security/data exposure)
**Migration:** none
**Next milestone:** build per the corrected design below

> **Review history.** v1 → team-review (18 seats) → Boss **SEND BACK** (2026-06-13) with 6 conditions, all folded into v2:
> (1) 137a must NOT drop tutee Post Title (it's in two data transfer objects + two frontend pages) — add tutee Level, keep the field, and **sweep the sibling raw-code surfaces** (Job Application Service implementation reference → applicant cards / Applications page), with ONE regression guard over all sites;
> (2) 137b is a documented contract change (api Error implementation reference documents NOT FOUND as 400 for the INT-063 close-job recovery) — apply both-ends (update the doc + pinned tests in the same pull request; the frontend keys on the CODE not the status, api Error implementation reference, so 404 is runtime-safe);
> (3) add an server-error counts grouped by route metric/alert (the detection half the missing-value failure class lacked);
> (4) 137c must pick the isolation approach in the spec (done: option (i));
> (5) [INT-064 conditions live in that spec]; (6) no value Away phased — tracked in the missing-value failure retrospective, not here.

---

## 1. Problem

Three pre-existing defects surfaced during the final-wave proof of the INT-090…136 batch. All filed
(not fixed) per the bug-founder convention. Each verified against source this session.

| # | Defect | Where (verified) |
|---|--------|------------------|
| a | Tutor match-card title renders **raw subject/level codes** (S4 EXP AMATH OLEVEL) | Match Dashboard Service implementation reference; frontend Matches implementation reference |
| b | read operation for assignments id (+ siblings) return **web request protocol 400 with body code NOT FOUND** — status/body contradiction vs the controller javadoc and the rest of the codebase | Tutee Post Service implementation reference |
| c | The regression-lab reseed **destroys demo-account fixtures** (Wave A's mei ling COMPLETED match wiped), so manual demo state isn't re-provable after a lab reseed | Dev Data Seeder ↔ the private implementation folder boundary |

These are independent and can ship in one wave. (a) spans backend+frontend, (b)+(c) are the private implementation folder only.
Within the backend subtree, serialize the builds (one repo, no worktree isolation in this lane).

---

## 2. INT-137(a) — humanise the tutor match-card title

### 2.1 Root cause

Match Dashboard Service implementation reference composes a pseudo-title from raw enum codes:


Implementation example omitted. This public edition retains the written requirements in this section.


Tutor Match Response (Tutor Match Response implementation reference) carries tutee Post Title +
tutee Subject but **no level code**, so the frontend cannot humanise the title with a clean one-liner. The
frontend renders the raw string at Matches implementation reference (the match’s tuition-request title), and sibling raw
the tutee's subject renders exist that the INT-099 regression guard does not cover.

### 2.2 Design — codes in the data transfer object, labels on the frontend (the established pattern)

This mirrors INT-115's interview-data transfer object fix: **the data transfer object carries codes, the frontend humanises via the existing
format implementation reference helpers.** Never compose a display label backend-side (the backend has no humaniser and
shouldn't grow one — labels are an frontend concern, and INT-099 already routes every other listing surface
through the subject label operation/the level label operation).

**Backend — match card (Tutor Match Response).**
- Add the raw level code (e g. text tutee Level) alongside the existing tutee Subject.
- **Do NOT drop tutee Post Title** (Boss SEND-BACK #1). Verified consumers: the field exists on
  **two** data transfer objects — Tutor Match Response implementation reference AND Application Response implementation reference — and is rendered on
  **two** frontend pages — Matches implementation reference AND Applications implementation reference — and typed at
  index implementation reference. Dropping it is a breaking contract change. Keep the field; stop the
  raw-code concatenation at Match Dashboard Service implementation reference (have it carry the subject **code**, or a
  back-compat value the frontend no longer renders raw). Add tutee Level and update the constructor call.

**Backend — SWEEP the sibling raw-code surfaces (Boss SEND-BACK #1).** 137a v1 was scoped to only the
match card. The same the get subject code operation concatenation pattern feeds the applicant /
Applications surfaces — audit and fix in this wave (the consumer-impact-list lesson applied to our own
spec):
- Job Application Service implementation reference and 598-599 (the get subject code operation,
  the get level code operation) → Application Response / applicant cards. If Application Response's
  title/subject is a concatenated raw-code string, carry the codes and humanise frontend-side (same fix);
  if it carries a genuine free-text title, leave it — **read the mapper before changing.**
- Grep the backend data transfer object mappers repo-wide for the code operation string concatenation
  (rg -n get Subject Code\ \ |get Level Code\ \ intuition-the private implementation folder)
  and enumerate every display-composition site; that grep is the authoritative scope of 137a.

**Frontend.**
- Add a small the match title operation helper in format implementation reference, next to the existing
  interview Title helper (format implementation reference), composing
  readable level and subject labels with the same null-safe fallback contract
  (format implementation reference). Reuse the subject label operation (131) and the level label operation (139).
- Render the humanised title at Matches implementation reference and on the Applications page
  (Applications implementation reference); humanise the sibling raw the tutee's subject renders too (route
  them through the subject label operation).

### 2.3 Regression guard

Extend the INT-099 "no raw enum codes" scan / site-pin test to cover **every** site found by the §2.2
grep — match card, applicant cards, Applications page — not just the one match-card line. This class
has regressed multiple times (INT-068, INT-099, 137a); the guard must pin all sites or the next one
rots silently (the SHARED-LESSON: grep the backend data transfer object mappers for code-string concatenation, not only frontend
renders).

### 2.4 Tests

- **backend:** Match Dashboard Service Test — assert Tutor Match Response carries raw tutee Subject +
  tutee Level (not a composed label string); add the equivalent assertion on any
  Job Application Service mapper changed by the sweep.
- **frontend:** Matches test implementation reference (+ Applications test implementation reference if touched) — assert the card
  shows the friendly label (e g. "O-Level · A-Math") for a code input; the extended INT-099
  regression scan stays green.

---

## 3. INT-137(b) — NOT FOUND must be 404, not 400

### 3.1 Root cause

Five call sites throw the bad request operation → web request protocol **400** with
body code NOT FOUND, contradicting the controller javadoc and the rest of the codebase, which uses
the not found operation → 404 (e g. Admin Moderation Service implementation reference):

- Tutee Post Service implementation reference, 431, 446
- Tutor Post Service implementation reference, 338

### 3.2 Design — fix to 404, but it is a CONTRACT change (Boss SEND-BACK #2 — both ends)

Switch all five to the not found operation (helper exists,
App Exception implementation reference). Code label unchanged (NOT FOUND); web request protocol status moves 400 → 404,
aligning status, body, and javadoc.

**This is NOT trivial** — api Error implementation reference documents CLOSE POST ERROR CODE
with **"NOT FOUND (400 — deleted…)"**, the contract the INT-063 close-job stale-recovery relies on
(Active Jobs implementation reference). Note the 5 sites span GET-detail AND mutation endpoints:
- GET-detail (the filed defect): Tutee Post Service implementation reference (find Active Post By Id),
  Tutor Post Service implementation reference.
- mutation / other: close Job As Admin (Tutee Post Service implementation reference), find Own Post
  (Tutee Post Service implementation reference?), Tutor Post Service implementation reference — **read each method before changing** to
  confirm 404 is the correct semantic (resource-not-found) for that operation.

**Runtime-safe, because the frontend keys on the CODE, not the status** — api Error implementation reference
(NOT FOUND: NOT FOUND) is switched by value (verified). So 404 with the same code does not break
the recovery. BUT the both-ends discipline (INT-075 lesson) is mandatory:

### 3.3 Blast radius — update BOTH ends in the same pull request

- **frontend doc + tests:** correct the api Error implementation reference comment "(400)" → "(404)"; run the frontend tests that
  exercise the close-job recovery (Active Jobs implementation reference) to confirm they key on code, not status.
- **backend tests:** audit every test asserting **400** on these five paths and flip to **404**. Candidate
  suites: Tutee Post Controller Test, Tutor Post Controller Test, Public Browse Controller Test, and any
  assignment-detail / close-job IT (rg -n NOT FOUND intuition-the private implementation folder shows the set). Add /
  keep a positive test asserting **404 + NOT FOUND** on a missing/MATCHED post via each endpoint.
- **Decision:** fix all five for consistency (one envelope semantic), with the doc + test sweep above
  in the same change. Do not blanket-flip without the frontend-doc + pinned-test updates.

### 3.4 Observability — add the 5xx-by-route metric (Boss SEND-BACK #3, cross-cutting from the missing-value failure retro)

The missing-value failure cluster (INT-091/092) 500'd the money/review paths with **no alert**. Independently of the
400→404 cosmetic, add an server-error counts grouped by route counter + alert (Micrometer; pairs with the existing
the number of commission-payment attempts, grouped by result from INT-091) so a future shape-regression 500 is *visible*, not silently
dead. This is the detection half that no value Away (compile-time) cannot provide. Confirm the
Global error handler maps App Exception 404 to the same envelope as the existing 404s (it does).

---

## 4. INT-137(c) — isolate demo fixtures from the lab reseed

### 4.1 Root cause

The regression-lab reseed truncates/repopulates shared tables, wiping manually-provisioned demo state
(Wave A's mei ling COMPLETED match). The lab reseed and the demo seed share row space, so re-running
the lab destroys demo data — manual demo flows aren't re-provable after a lab run.

### 4.2 Design — DECISION: scope-limit the lab reseed (option i) (Boss SEND-BACK #4)

**Chosen: option (i) — scope-limit the lab reseed to lab fixtures only.** Namespace lab accounts with
a lab.*/regression.* handle+email prefix and have the reseed delete/insert **only within that
namespace** — never a blanket truncate of shared tables that takes demo rows with it. Demo accounts
(e g. mei ling) live outside the lab namespace and are therefore untouched by a reseed.

Implementation: read both up-paths first (Dev Data Seeder and the private implementation folder)
— this is a seeder-ownership boundary, do not guess — then make the lab teardown delete by the
namespace predicate (e g. when email LIKE lab.%) rather than truncating. If the lab today truncates
whole tables, that truncate is the offending line to replace. Fallback to (ii) additive/idempotent
upsert only if a clean namespace predicate proves impractical (record why). Document the boundary in
specs/ops/dev-sample-data md.

### 4.3 Verify

Run a lab reseed, then assert the demo COMPLETED match (and other demo fixtures) still resolve via the
application programming interface. Add a note to the runbook so future waves stay re-provable.

---

## 5. Verification (end-to-end)

Backend (Java 21; Docker up for Testcontainers):


Implementation example omitted. This public edition retains the written requirements in this section.


- 137a: Match Dashboard Service Test data transfer object-codes assertion green; the §2.2 grep
  (rg -n get Subject Code\ \ |get Level Code\ \ .../service) returns only non-display uses — every
  display-composition site humanised, all pinned by the extended INT-099 guard.
- 137b: controller/IT tests assert **404 + NOT FOUND** on the five endpoints; frontend close-job recovery
  tests green with the api Error implementation reference doc corrected to "(404)".
- 137b observability: server-error counts grouped by route counter present + a 5xx alert wired.
- 137c: lab reseed (namespace-scoped delete) → demo mei ling COMPLETED match still resolves.

Frontend (Node 20):


Implementation example omitted. This public edition retains the written requirements in this section.


- 137a: tutor match-card shows friendly labels; INT-099 regression scan green.
- **Mobile-web check (standing frontend review point):** the humanised tutor match-card label must not overflow at 390px — confirm via the mobile-overflow spec implementation reference gate (a longer "O-Level · A-Math"-style label wraps/truncates rather than widening the card past the viewport). Per the founder directive, every frontend change carries a mobile-web pass even though the app is desktop-optimised.

Live smoke (after redeploy + reseed — regression ITs hit localhost):

- read operation for assignments <missing-id> → **404** body NOT FOUND (was 400).
- Tutor dashboard match card shows e g. "O-Level · A-Math", not "S4_EXP AMATH OLEVEL".

---

## 6. Ship

/team-review → Boss SHIP → build → gates green (re-run by orchestrator, not trusted) → deploy dev +
reseed → push on sp-dev (backend + frontend + spec) with concise messages + the Co-Authored-By trailer.
Stage real source/tests only (never coverage/, .DS Store, .env implementation reference). Update the INT-137 tracker row
to FIXED-DEV with evidence on ship.
