# INT-046 — Frontend Coverage Closeout Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** SPEC — quality assurance lead + frontend-expert  
**Date:** 2026-06-03  
**Issue:** [README implementation reference](../issues/README.md)  
**Tracker:** INT-046 (S2 OPEN) — frontend coverage gate red, 95.82% lines < 97.5%

---

## 1. Problem

The Vitest worker-pool deadlock (INT-020) was fixed by switching to pool: forks + max Workers: 4.
As a side-effect, the FULL ~91-file suite now runs in a single the documented operational check invocation for
the first time. All **1234 tests PASS**. However, the global coverage report now enforces what
the chunked runs had been hiding: the project is below its own gate.


The illustration for . problem contains the following information:

- Gate (vite config implementation reference): lines ≥ 97.5 %, branches ≥ 88 %.
- Actual (post-INT-020): lines = 95.82 %.


The gap is approximately **1.68 percentage-point** on lines. This is coverage **debt**, not
regressions; the files were under-covered before the deadlock fix made the number visible.

The deficit is dominated by three files:

| File | Estimated coverage | Primary uncovered surface |
|---|---|---|
| Interview Queue implementation reference | ~78 % lines | SELECTIONS mode, no-show confirm modal, set-link error, cancel-link-modal-X-close |
| Matching implementation reference | ~82 % lines | activate error toast, cancel error toast, generic assign error, mark-paid error, refetch button, COMPLETED/CANCELLED filter tabs, commission-null Match Row |
| use Init Auth implementation reference | ~87 % lines | handle Unauthorized event-handler body (navigate call + return To construction) |

All other files are at or above the 97.5 % line threshold; these three are the blocking gap.

---

## 2. Coverage Gap Analysis (per file)

### 2.1 Interview Queue implementation reference — ~78 % lines

Reading the source (Interview Queue implementation reference lines 1–505) against the existing test file
(Interview Queue test implementation reference — 539 lines), the following branches/lines are NOT
exercised by the current tests:

**A. SELECTIONS mode (lines 193–217)**  
The component has two top-level modes controlled by use State<Queue Mode> INTERVIEWS. The
existing tests never click the "Selection requests" tab (mode equals SELECTIONS), so the entire
show the selections view or the alternate view according to the selected mode true-branch is dead code in the coverage report.
Uncovered paths:
- Selections loading skeleton (line 195–198): selections Loading equals yes path.
- Selection Request Queue rendered under SELECTIONS mode (line 200).
- Close Job Assign Modal in SELECTIONS mode (line 202–216): on Confirm → assign Mutation mutate
  with valid tutor Post Id, success toast, and error handling (both JOB ALREADY ASSIGNED and
  generic error).

**B. Confirmation modal — no Show Tutor path (line 491)**  
The confirm modal dispatches on action:


Implementation example omitted. This public edition retains the written requirements in this section.


The TUTOR no-show dispatch branch (no Show Tutor) is confirmed tested (test line 427–437).
The error toast for no-show (line 501–509 in the test) covers both actors implicitly, but the
explicit no Show Tutor confirmation branch should be verified to have a path to the confirm click.

**C. set Link Mutation on Error (line 87)**  
The existing test suite covers on Success for the set-link mutation but there is no test that
makes mock Patch reject and asserts the "Failed to set meeting link" error toast. The on Error
handler on line 87 is uncovered.

**D. assign Mutation on Error — INTERVIEWS mode (lines 139–150)**  
When the component is in INTERVIEWS mode (the default), the assign Mutation block is present
but inaccessible: in INTERVIEWS mode there is no visible assign button because
Close Job Assign Modal and Selection Request Queue are rendered only in SELECTIONS mode. The
assign Mutation is therefore never triggered from INTERVIEWS mode, and coverage tools see the
on Error block as uncovered even though that is correct behaviour. The fix is to exercise
both the JOB ALREADY ASSIGNED code path and the generic error from within SELECTIONS mode.

**E. Confirm modal — confirm Pending equals no value early-return guard (line 487)**  
do nothing until the required confirmation is pending is a defensive guard; reaching it requires clicking Confirm
when confirm Pending is null. This path is unreachable in normal user interface flow (the button is
only visible when the modal is open and confirm Pending differs from no value). See Section 4 for the
alternative of excluding this guard or documenting it as a known untestable defensive branch.

---

### 2.2 Matching implementation reference — ~82 % lines

Reading the source (Matching implementation reference lines 1–437) against Matching test implementation reference
(347 lines), the following branches/lines are uncovered:

**A. activate Mutation on Error (line 110)**  
The existing test confirms on Success ("Match activated." toast) but no test makes
mock Post reject on the activate endpoint. The on Error handler → "Couldn't activate
the match. Please try again." toast is not exercised.

**B. cancel Mutation on Error (lines 133–135)**  
The cancel error path (Couldnt cancel the match. Please try again.) is never exercised.
The existing test only covers success. The cancel on Error also calls the set cancel confirm id operation
to close the modal — both effects are uncovered.

**C. assign Mutation on Error — generic error branch (line 95–97)**  
JOB ALREADY ASSIGNED is covered. The else branch ("Could not assign the job.") is not.
The generic error is reached when err response.data error.code is anything other than
JOB ALREADY ASSIGNED.

**D. mark Paid Mutation on Error (line 120)**  
No test makes the mark-paid POST fail. The Couldnt mark as paid. toast handler is uncovered.

**E. Matches error state — Refresh button (lines 221–225)**  
The matches Error alert contains a a Refresh button that reloads matches.
The existing test asserts the alert renders but does not click the Refresh button, so the
the refetch matches operation call is uncovered.

**F. Match filter tabs — COMPLETED and CANCELLED (lines 193–209)**  
match Tab filtering is confirmed for the "Active" tab in the existing test. The "Completed"
and "Cancelled" tab clicks are not tested. These tabs update match Tab state and trigger the
filtered Matches filter expression (only matches whose status matches the selected tab), which
runs different filtered Matches length 0 + Match Row branches.

**G. Match Row — commission equals no value branch (line 393)**  
MATCH PENDING has commission: no value. When commission is null, the commission cell renders
—. The current test does render MATCH PENDING in the table, but the commission-null
cell — string and the Match Row commission-null ternary (commission ? ... : —)
may not be specifically asserted, leaving the null branch uncovered in v8 branch tracking.

**H. Match Row — can Mark Paid no (commission present but not PENDING) (line 381)**  
MATCH COMPLETED has invoice Status: PAID. can Mark Paid is false when invoice Status differs from PENDING.
The COMPLETED match is in the fixture but the test does not specifically tab to COMPLETED and
assert that "Mark PAID" does NOT appear, leaving the can Mark Paid no branch undercount.

---

### 2.3 use Init Auth implementation reference — ~87 % lines

Reading the source (use Init Auth implementation reference lines 1–40) against use Init Auth test implementation reference
(102 lines), the uncovered surface is:

**A. handle Unauthorized body — navigate call with return To (lines 33–35)**  


Implementation example omitted. This public edition retains the written requirements in this section.


The existing test (line 91–101) confirms that the add event listener operation is
called on mount and remove Event Listener on unmount, but it never fires the auth:unauthorized
event. Therefore:
- The return To ternary (both branches: pathname  differs from  '/login' and pathname  equals  '/login') is uncovered.
- The the navigate operation call is never reached.

The existing 5 tests cover the cross-site request forgery protection/me promise chain thoroughly but leave the entire event
handler body untouched, which is the main reason this file sits at ~87 %.

---

## 3. Test Plan

All tests use RTL + Vitest with the existing Memory Router + Query Client Provider wrapper
pattern established in the file suite. Mocking strategy: the mock operation as done
in all admin tests.

### INT-020 / INT-021 lessons applied to this plan

- **Multiple-element query drift (INT-021):** some user interface surfaces render the same label in multiple
  places (e g. Selection Request Queue row button + Close Job Assign Modal confirm button both
  read "Assign tutor & close job"). Use the find all by role operation to scope to the modal, or
  the within operation scoping. Never use bare get By Role on strings that appear in both the list
  and the modal.
- **forks pool isolation (INT-020):** tests share no worker state. Each test that needs fresh
  query state must create a new Query Client with retry: no. The the make qc operation factory pattern
  already does this; use it consistently.
- **Always use screen find By* (async) after user interactions** to allow React Query mutations
  to settle, not get By* directly after click.

---

### 3.1 New tests for Interview Queue implementation reference

File: Interview Queue test implementation reference — ADD to the existing file.

A SELECTION fixture is needed. Define once at the top of the test file:


Implementation example omitted. This public edition retains the written requirements in this section.


The mock Get must respond differently for the interviews URL vs the selection-requests URL.
Use a test response selected according to the requested address branching on the includes operation.

**TC-IQ-01 — Mode tab: switch to SELECTIONS shows Selection Request Queue**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-IQ-02 — SELECTIONS mode: loading skeleton**


Implementation example omitted. This public edition retains the written requirements in this section.


Note: implement mock Get so the interviews call resolves (so the tab text "0" renders)
but the selection-requests call hangs.

**TC-IQ-03 — SELECTIONS mode: assign success toast**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-IQ-04 — SELECTIONS mode: JOB ALREADY ASSIGNED error toast**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-IQ-05 — SELECTIONS mode: generic assign error toast**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-IQ-06 — Set Link modal: error toast on PATCH failure**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-IQ-07 — Confirm modal: no Show Tutor body text**


Implementation example omitted. This public edition retains the written requirements in this section.


This exercises the confirmation modal body text branch for no Show Tutor.

---

### 3.2 New tests for Matching implementation reference

File: Matching test implementation reference — ADD to the existing file.

**TC-M-01 — Activate error toast**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-M-02 — Cancel error toast**


Implementation example omitted. This public edition retains the written requirements in this section.


Also assert: wait until the cancellation warning is no longer displayed
to verify the modal closes on error (the on Error sets the cancel confirm id operation).

**TC-M-03 — Generic assign error toast**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-M-04 — Mark PAID error toast**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-M-05 — Matches error: Refresh button calls refetch**


Implementation example omitted. This public edition retains the written requirements in this section.


Implementation note: use the mock rejected value once operation then the mock resolved value operation.

**TC-M-06 — Filter tab: COMPLETED shows only COMPLETED matches**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-M-07 — Filter tab: CANCELLED empty state**


Implementation example omitted. This public edition retains the written requirements in this section.


This exercises the empty-state message format: an empty-state message naming the selected match status

**TC-M-08 — Match Row commission is no value shows "—"**


Implementation example omitted. This public edition retains the written requirements in this section.


**TC-M-09 — Match Row can Mark Paid is no: no "Mark PAID" for COMPLETED match**


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 3.3 New tests for use Init Auth implementation reference

File: use Init Auth test implementation reference — ADD to the existing the describe operation block.

**TC-user interface-01 — fires auth:unauthorized → navigates to /login with return To**


Implementation example omitted. This public edition retains the written requirements in this section.


Implementation note: to verify navigation in a render Hook context, wrap in a Memory Router
that exposes its current location via a test helper component, or mock react-router-dom's
use Navigate to capture calls:


Implementation example omitted. This public edition retains the written requirements in this section.


Then: verify that navigation reaches sign-in with the intended return path and replaces the current history entry.

**TC-user interface-02 — fires auth:unauthorized from /login → navigates to /login with no return To**


Implementation example omitted. This public edition retains the written requirements in this section.


This covers the ternary false-branch (pathname equals /login → empty return To).

**TC-user interface-03 — auth:unauthorized listener is removed on unmount (already covered — verify)**  
The existing test (TC line 91–101) covers this. No new test needed.

---

## 4. Re-baseline Alternative

Instead of writing new tests, the team could lower the gate to reflect the actual coverage the
codebase reliably achieves on the admin pages.

**Option A — lower lines gate to 95 %**  
Set the historical thresholds of 95% lines and 88% branches in vite config implementation reference.

- Pros: zero test-writing effort; continuous integration green immediately; honest about what is actually verified.
- Cons: signals that uncovered admin flows are acceptable; the ~78 % Interview Queue and ~82 %
  Matching pages have meaningful untested mutation-error paths and the SELECTIONS mode entirely
  absent from tests. Future regressions in these flows would be invisible. Sets a precedent that
  coverage thresholds can be dialled down when inconvenient.

**Option B — exclude the three files from coverage**  
Add Interview Queue implementation reference, Matching implementation reference, and
use Init Auth implementation reference to the coverage exclude list in vite config implementation reference.

- Pros: quick; preserves 97.5 % discipline on all other files.
- Cons: the excluded files are among the most consequential admin flows (interview lifecycle,
  match assignment, auth init). Explicitly opting out of coverage on them is difficult to justify
  to any external reviewer and hard to walk back once excluded.

**Option C — exclude only the defensive dead-code branches (preferred partial)**  
The do nothing until the required confirmation is pending guard in Interview Queue (line 487) is a legitimate
defensive branch with no test path. Use an inline /* c8 ignore next */ comment on that single
line. This is not the same as excluding the whole file.

- Pros: surgical; documents the reasoning inline; leaves the rest of the file subject to the gate.
- Cons: requires discipline to not abuse c8 ignore elsewhere.

**Recommendation:** implement Option C (inline c8 ignore for the one untestable guard) combined
with the new tests from Section 3. Do NOT lower the gate; the 97.5 % target exists to catch
regressions on meaningful flows, and the admin pages are exactly the flows most likely to
regress.

---

## 5. Acceptance Criteria

1. the documented operational check exits 0 with the coverage report showing lines is at least 97.5 %.
2. All 1234 existing tests continue to pass (no regressions).
3. The new tests described in Section 3 are all green on first run.
4. Interview Queue implementation reference coverage  is at least  97.5 % lines.
5. Matching implementation reference coverage  is at least  97.5 % lines.
6. use Init Auth implementation reference coverage  is at least  97.5 % lines.
7. No new coverage exclude entries are added for these three files.
8. No new /* c8 ignore */ annotations except the one defensive guard in Interview Queue implementation reference
   line 487 (the do nothing until the required confirmation is pending guard), if the team decides Option C.
9. INT-046 status updated to FIXED-DEV in the issue tracker after gate passes.

---

## 6. Open Decisions

**D1 — use Navigate mocking strategy for use Init Auth event tests**  
use Init Auth calls the use navigate operation at hook init but only invokes the navigate operation inside the
event listener. The current test wrapper is a plain Memory Router. Two approaches:
(a) Mock react-router-dom's use Navigate with vi mock and capture calls (simpler, unit-y).
(b) Use a real Memory Router + a helper child that exposes the use location operation to inspect the
    navigation result (integration-y, proves the router actually updated).
The mock approach is recommended here for consistency with the rest of the suite, which already
mocks api rather than standing up real network. Decision: team picks approach before
implementation begins.

**D2 — Whether to add the SELECTION fixture to Interview Queue test implementation reference globally or per-suite**  
SELECTION PAGE and SELECTION REQ should be defined at the module level alongside the
existing IV REQUESTED/IV SCHEDULED fixtures. The mock Get must be extended to branch on
URL in the suites that test SELECTIONS mode. Existing tests that do NOT call the render page operation with
the new overriding mock should remain unaffected (they call mock Get mock Resolved Value directly
and the before Each the clear all mocks operation guarantees isolation).

**D3 — Confirm modal defensive guard (line 487 of Interview Queue implementation reference)**  
The do nothing until the required confirmation is pending guard is unreachable in practice (the Confirm button is only
rendered when the modal is open and confirm Pending differs from no value). Options: (a) accept the uncovered
branch and use /* c8 ignore next */; (b) refactor the handler to remove the guard (slightly
less safe). Recommendation: c8 ignore with a comment explaining it is unreachable by design.

**D4 — Match Row commission is no value assertion robustness**  
the get all by role operation contains the full cell text and is fragile to layout
changes. Consider using within scoping with a test-id on the commission cell, or accepting
the looser check the expect operation combined with verifying
"Mark PAID" does not appear. Team to decide before implementation.
