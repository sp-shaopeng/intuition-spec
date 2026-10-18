# Five-Factor-Taste Audit — Regression Lab ITs (R2 deliverable)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Phase:** R2 of [architecture/regression-improvement-spec md](../architecture/regression-improvement-spec.md) §R2.
**Date:** 2026-06-10. **Auditor:** Reese (quality assurance).
**Scope:** every *IT implementation reference (32 classes).

## Method
Each IT class is scored against the **five-factor TB-UC taste** (MULTIPLICATIVE — any one ≈0 ⇒ the
test is worthless), per the TATAH discipline in
[Private supporting material excluded (publication note)](../../docs/publication-notes.md):

| Factor | What it demands |
|---|---|
| **BSC** Business-Semantic-Correctness | asserts the **promise** (entity status / field / error code + state-unchanged), NOT bare web request protocol 200. |
| **GRA** Granularity | one promise per test; no fuzzy multi-promise assertion. |
| **AM** Anchor-Mandatoriness | every assert targets a node that MUST exist (every no value Value paired with a positive sibling on the SAME JSON node — the INT-077 rule). |
| **OD** Outcome-Decidability | pass/fail is unambiguous; cap tests have boundary + over-boundary; multi-recipient tests assert per-recipient. |
| **TC** Traceability-Closure | javadoc cites the source sequence / UC / matrix branch. |

The hunt target is **MISMATCH-3 / vacuous tests** (the INT-077 class): an assertion that stays GREEN
even when its target field/branch is removed. Findings recorded as MISMATCH rows; **every finding fixed
in place** and re-run GREEN against the live stack at http://localhost.

**Cell legend:** ✓ = passes the factor. **MISMATCH → fix** = a vacuous/weak assertion that was found and
**fixed in this audit** (the fix is described). The lab was re-run after every fix; final full-lab result
**161 run / 0 failures / 1 skipped** (the skip = the @Disabled INT-088 test, expected).

---

## Audit table (one row per IT class)

| IT class | BSC | GRA | AM | OD | TC | Notes / fix |
|---|:---:|:---:|:---:|:---:|:---:|---|
| Account Lifecycle IT | ✓ | ✓ | ✓ | ✓ | ✓ | change-pw + delete assert error code + state-unchanged (old pw still/again works). |
| auth/Auth Journey IT | ✓ | ✓ | ✓ | ✓ | ✓ | register dup-email/phone assert error code 409 + /me is 401 (no second account). Cites sequences/01-register-*. |
| auth/Detail Authz IT | ✓ | ✓ | ✓ | ✓ | ✓ | authed-detail asserts id matches discovered id + lazy subjects present (guards the 500). |
| auth/Role Denial Status IT | ✓ | ✓ | ✓ | ✓ | ✓ | 401(anon) vs 403(wrong-role) on both gate styles. Cites INT-029. |
| Csrf Negative IT | ✓ | ✓ | ✓ | ✓ | ✓ | documents the control (header-present → 404) proving the header is what's enforced. |
| Guard Negatives IT | ✓ | ✓ | ✓ | ✓ | ✓ | every guard asserts error code; cites Job Application Service:NN branches. |
| Interview Limit IT | ✓ | ✓ | ✓ | ✓ | ✓ | **boundary+over-boundary pair** (3 OK → 4th 422 INTERVIEW LIMIT REACHED). **Spot-break #3 → RED (below).** |
| Password Reset Journey IT | ✓ | ✓ | ✓ | ✓ | ✓ | reset asserts gate cleared **+ a real admin mutation succeeds**; one-time passcode single-use/wrong-one-time passcode assert error code. pull request-7 @Disabled for **INT-088** (a real source bug it surfaced, filed not papered). |
| Post Lifecycle IT | ✓ | ✓ | ✓ | ✓ | ✓ | edit/delete guards assert INVALID STATE; #34 availability toggle asserts owner 200 + id echo + non-owner FORBIDDEN. |
| Review Submit Journey IT | ✓ | ✓ | ✓ | ✓ | ✓ | per-participant (tutee **and** tutor) 201; duplicate guard asserts DUPLICATE REVIEW; non-completed → INVALID STATUS. **Spot-break #2 → RED (below).** |
| Seed Manifest IT | ✓ | ✓ | ✓ | ◑ | ✓ | OD note: seeded equals yes + the greater than operation (not exact pins) — **R3 tightens to exact counts/ids by design**, out of R2 scope; left as-is. |
| dashboard/Dashboard Reviews IT | ✓ | ✓ | ✓ | ✓ | ✓ | read-paths both roles + public reviews open; anon 401. |
| notification/Notifications IT | ✓ | ✓ | ✓ | ✓ | ✓ | mark-all-read **drives unread count to 0** (the outcome, not just 200). |
| application/Applications IT | ✓ | ✓ | ✓ | ✓ | ✓ | apply→appears-in-list (asserts the job id is present)→withdraw; self-cleaning. |
| favourite/Favourites IT | ✓ | ✓ | ✓ | ✓ | ✓ | add→list-contains→remove→list-absent (round-trip outcome); unknown post → TUTOR POST NOT FOUND. |
| profile/Profile IT | ✓ | ✓ | ✓ | ✓ | ✓ | own-profile 200 + cross-role 403; display Name not no value on the happy read. |
| interview/Interviews IT | ✓ | ✓ | ✓ | ✓ | ✓ | ownership/active guards assert error code; terminal-cancel-twice → INVALID STATUS TRANSITION. Cites Interview Service:NN. |
| interview/Interview Negotiation IT | ✓ | ✓ | ✓ | ✓ | ✓ | each transition asserts status + whose Turn; assert Not500 guards the INT-016 lazy-init signature. |
| marketplace/Assignment Lifecycle IT | **MISMATCH → fix** | ✓ | **MISMATCH → fix** | ✓ | ✓ | **contact-reveal asserted only counterparty Email not no value Value — VACUOUS** (would pass if the endpoint echoed the caller's OWN email or any non-null value). **Fix:** assert the exact **cross-party** identity — parent sees the equal to operation, tutor sees the equal to operation, + counterparty Name/counterparty Phone not no value (full contact promise). Added email to the Tutor Actor record. **Spot-break #1 proves the fix (below).** |
| marketplace/Browse Assignments IT | ✓ | ✓ | **MISMATCH → fix** | ✓ | ✓ | **response Body is Pii Free the does not contain operation passes VACUOUSLY on an empty data array.** **Fix:** added a positive sibling — assert the size operation before treating personal information-absence as evidence. |
| marketplace/Browse Tutors IT | ✓ | ✓ | **MISMATCH → fix** | ✓ | ✓ | two vacuous-on-empty asserts: List body is Contact Pii Free (personal information scan) **and** …Expose Public Handle not Real Name (the tutor Display Name equals no value personal information negative under the if operation). **Fix:** assert data/handles non-empty first, then the personal information negatives become load-bearing. |
| showcase/Showcase IT | ✓ | ✓ | **MISMATCH → fix** | ✓ | ✓ | both personal information-free showcase tests the does not contain operation on a body that could be empty. **Fix:** assert the size operation in each before the personal information scan (the non-empty was only proven in a sibling test, not the personal information test itself). |
| admin/Admin Analytics IT | ✓ | ✓ | ✓ | ✓ | ✓ | asserts the sectioned metric shape + counters reconcile (total Users 0); invalid date-range → 400. |
| admin/Admin Applications View IT | ✓ | ✓ | ✓ | ✓ | ✓ | anon/cross-role denied; operator gets the data envelope. |
| admin/Admin Authz Matrix IT | ✓ | ✓ | ✓ | ✓ | ✓ | full anon-401 / wrong-role-denied / operator-200 matrix across every admin surface. |
| admin/Admin Bootstrap IT | ✓ | ✓ | ✓ | ✓ | ✓ | create-admin asserts email/temp Password/id; **gate-blocked mutation → 403** until pw change, then operates. |
| admin/Admin Commission IT | ✓ | ✓ | ✓ | ✓ | ✓ | pay→PAID; **double-pay (distinct key) rejected** (not an idempotent replay); unknown → 404. |
| admin/Admin Interview Ops IT | ✓ | ✓ | ✓ | ✓ | ✓ | full lifecycle asserts terminal COMPLETED/NO SHOW/CANCELLED; no-show-on-unscheduled → 422. |
| admin/Admin Matching IT | ✓ | ✓ | ✓ | ✓ | ✓ | create/activate/cancel assert status transitions; **cancel cascades commission → CANCELLED**; NOT FOUND twins assert error code. |
| admin/Admin Post Approval IT | ✓ | ✓ | ✓ | ✓ | ✓ | approve **removes from pending queue** (the outcome); reject-without-reason → 400; double-approve rejected. |
| admin/Admin User Mgmt IT | ✓ | ✓ | ✓ | ✓ | ✓ | delete is super-only (flag-driven); **CANNOT REMOVE SUPER ADMIN 403 + asserts the target is still present & still super** (state-unchanged). |
| admin/Free Market Assign IT | **MISMATCH → fix** | ✓ | ✓ | ✓ | ✓ | same vacuous contact-reveal pattern as Assignment Lifecycle IT — counterparty Email not no value Value only. **Fix:** assert exact cross-party email (parent↔tutor); added email to the Listingless Tutor record. (Already pairs tutor Post Id no value Value with positive siblings — AM was ✓.) |

**Totals:** 32 IT classes audited · **5 MISMATCH/vacuous findings fixed in place** (across 5 classes:
Assignment Lifecycle IT, Free Market Assign IT, Browse Assignments IT, Browse Tutors IT, Showcase IT) ·
1 OD note left to R3 by design (Seed Manifest IT exact pins). No vacuous test was found to be hiding a
*new* real source bug (the one real bug in this surface — INT-088 — was already surfaced + filed by R1's
Password Reset Journey IT, not papered over).

---

## Spot-break runbook (proves no vacuous green)

For 3 named representative ITs, the asserted thing was broken **in the SUT** (not the assertion), the dev
stack rebuilt+redeployed, the IT re-run to confirm **RED**, then the break reverted + redeployed. Method =
SUT-break for all three. **Procedure used each time:** edit the service → manage the required application services
backend→ wait health 200 →run the selected server integration test → observe RED →
revert → rebuild → re-run GREEN.

> **Methodology gotcha discovered & recorded:** the documented operational check **alone does NOT recompile
> test sources** — it runs the previously-compiled *IT class. A spot-break (or any test edit) MUST be
> preceded by the documented operational check (or run the documented operational check), or the break runs against **stale test bytecode**
> and falsely shows GREEN. This bit the contact-reveal break before it was caught (see SHARED-LESSONS).

| # | IT (method) | SUT break applied | RED failure line observed |
|---|---|---|---|
| 1 | marketplace/Assignment Lifecycle IT#contact Reveal four States | Contact Reveal Service counterparty User Id is Tutor ? tutee User Id : tutor User Id → inverted to is Tutor ? tutor User Id : tutee User Id (each caller gets their OWN contact) | Assignment Lifecycle IT contact Reveal four States → Expected: tutor-0372135e-…@regression local Actual: tutee-a6437bd7-…@regression local — i e. the parent received its OWN email. **The OLD the not no value value operation assertion stayed GREEN under this exact break; the new the equal to operation catches it — direct proof the fix removed a vacuous test.** |
| 2 | Review Submit Journey IT#review duplicate By Same Reviewer is422 | Review Service ERR DUPLICATE REVIEW value → DUPLICATE REVIEW BROKEN (body still 422, but the promised error code changes) | Review Submit Journey IT review duplicate By Same Reviewer is422:163 → Expected: DUPLICATE REVIEW Actual: DUPLICATE REVIEW BROKEN — proves the test verifies the **business error code**, not just the 422 status. (NB: breaking only the pre-check would NOT have gone RED — the database unique-constraint catch re-throws the same code; the error-code break is the precise vacuity probe.) |
| 3 | Interview Limit IT#fourth Active Interview on Same Posting is422 limit Reached | Interview Service MAX INTERVIEWS PER POSTING 3 → 99 (cap disabled) | Interview Limit IT fourth Active Interview on Same Posting is422 limit Reached → Expected status code <422> but was <201> — the 4th interview was accepted, proving the over-boundary assertion is load-bearing. |

All 3 breaks reverted; backend rebuilt from clean committed source; **full lab re-run GREEN (161/0/1)**;
A source search found no remaining temporary fault-injection changes. The dev stack at 8080 ends healthy and matches committed source.
