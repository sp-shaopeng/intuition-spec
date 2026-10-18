# Requirements — Admin analytics correctness + financial safety (017)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> EARS + Gherkin per atomic requirement, grounded 2026-06-22. Source: CONSOLIDATED md sequential ADMIN re-run
> (conversion 400%, avg-commission, Mark-Paid confirm).

## Scope note (verified)

- the compute conversion rate operation (158-166) = completed/requested. The inputs
  come from the build interview funnel operation (147-155), which today uses **all-time** the count by status operation
  and the count by status operation — two independent all-time gauges, NOT one cohort. COMPLETED can exceed the
  *currently-REQUESTED* count (interviews that moved on from REQUESTED are no longer counted in the denominator but
  still count as COMPLETED) → ratio >100% ("400%"). The fix (REQ-001) re-bases both on the requested at-in-period
  cohort: denominator = the count by requested at between operation (69-70), numerator = the subset of that cohort with
  status is COMPLETED (a new count By Requested At Between And Status-style query). Note Interview has no completed at
  column — completion is status is COMPLETED, so the numerator is "requested-in-period AND now COMPLETED", which is
  exactly the pinned definition and is a strict subset of the denominator.
- the compute avg commission operation (200-207) = collected/active Matches.
  collected is cumulative (total or period collected); active Matches is a point-in-time gauge → cohort mismatch
  with the "per match" label.
- Commission implementation reference a Mark Paid action that immediately submits the payment update — no confirm; cancel (201) DOES confirm.

---

## REQ-001 — Interview conversion rate is cohort-consistent and never exceeds 100%

- **Type:** functional / correctness · **Priority:** Should · **Severity:** S3 · **Source:** conversion 400%
- **Rationale:** A conversion KPI above 100% is logically impossible and erodes trust in every number on the
  dashboard. The numerator and denominator must describe the same cohort.
- **Behaviour (EARS) — cohort PINNED: completed-of-requested-in-period (founder/BA decision 2026-06-22).** WHEN the
  interview-funnel conversion rate is computed for a period from, to, the system SHALL compute it as:
  - **Denominator** = the count of interviews whose requested at falls in from, to (the cohort of interviews
    REQUESTED in the period — Interview requested at, the only request timestamp on the model; grounded
    Interview Repository count By Requested At Between, 69-70).
  - **Numerator** = the count of THOSE SAME interviews (requested in the period) whose status is now COMPLETED.
  - Because the numerator is strictly a SUBSET of the denominator (same requested at cohort, additionally filtered
    to status COMPLETED), the ratio is naturally in 0, 100% and can NEVER exceed 100% — no clamp, no
    relabel. WHEN the denominator is 0 (no interviews requested in the period), the rate SHALL be 0% (no
    divide-by-zero, no Na N). The cumulative all-time-vs-fixed-window mismatch that produced the prior 400% is
    eliminated by construction. **The other two candidate definitions (all-time funnel; clamp-and-relabel a
    fixed-window numerator) are REJECTED** — clamp-and-relabel hides the cohort bug rather than fixing it.
- **Owner:** data-engineer · business-analyst (definition) · software-engineer (query/calc) · bff-specialist (payload)
- **Acceptance criteria (Gherkin):**


- Scenario: Conversion never exceeds 100%.
- Given any date range with interviews.
- When the conversion rate is computed.
- Then it is between 0% and 100% inclusive.
- Scenario: Cohort-consistent numerator/denominator (numerator ⊆ denominator).
- Given interviews requested in the period.
- When conversion = (those-requested-in-period that are now COMPLETED) / (requested-in-period).
- Then both counts describe the same requested at cohort (no cross-window inflation).
- And the numerator is a strict subset of the denominator.
- Scenario: Exact cohort rate from a known set (never >100%).
- Given exactly 10 interviews requested in the period, of which 4 are now COMPLETED.
- And some interviews COMPLETED in the period were requested BEFORE the period (out of cohort).
- When the conversion rate is computed.
- Then it is exactly 40% (4/10), counting only the in-cohort completions.
- And it is never above 100%.
- Scenario: Zero requested → 0% (guard).
- Given no interviews requested in the period.
- Then the conversion rate is 0% (no divide-by-zero, no Na N).


- **Required evidence:** unit test of the cohort calc with **a known set → an exact rate** (e g. 10 requested-in-period,
  4 of them now COMPLETED → exactly 40%, with out-of-cohort completions present to prove they do not inflate) plus
  the prior 400% scenario now ≤100% and the zero-requested guard; live capture of a sane percentage on the dashboard.
  Under REQ-001 implementation reference.
- **Dependencies:** INT-045 (analytics date range).

## REQ-002 — Avg-commission calc and label agree on one cohort/denominator

- **Type:** functional / correctness · **Priority:** Should · **Severity:** S4 · **Source:** avg-commission
- **Rationale:** Dividing cumulative collected by current-active matches is meaningless as "avg commission per
  match". Either fix the calc to a real per-match average over a consistent cohort, or relabel the metric to what it
  actually measures.
- **Behaviour (EARS):** WHEN the revenue summary presents an average-commission figure, the system SHALL compute it
  over a consistent cohort (e g. total commission collected ÷ number of matches that have a collected commission,
  for the same period) AND label it to match that definition. The numerator and denominator SHALL describe the same
  population; a zero denominator SHALL yield 0 (no divide-by-zero). The metric SHALL NOT divide a cumulative total
  by a point-in-time active-count under a "per match" label.
- **Owner:** data-engineer · business-analyst (definition) · software-engineer (calc) · frontend-expert (label)
- **Acceptance criteria (Gherkin):**


- Scenario: Avg commission is a real per-match average.
- Given commissions collected over a period.
- When avg-commission is computed.
- Then it is total-collected ÷ matches-with-a-collected-commission (same cohort).
- And the label matches that definition.
- Scenario: Zero denominator → 0 (guard).
- Given no matches with a collected commission.
- Then the value is 0 (no divide-by-zero).


- **Required evidence:** unit test of the cohort-consistent calc + zero guard; live capture with the corrected
  label/value. Under REQ-002 implementation reference.
- **Dependencies:** INT-045; the exact time scope/date anchor remains PENDING in
  [current open decisions](../CURRENT-REQUIREMENTS.md). Same-cohort agency-receipt semantics
  and the zero guard are confirmed; design md's selected-paid at proposal is not approval.

## REQ-003 — Mark-Paid requires explicit confirmation (financial safety)

- **Type:** functional / safety · **Priority:** Must · **Severity:** S3 · **Source:** Mark-Paid confirm
- **Rationale:** Marking a commission PAID is the highest-consequence, hardest-to-reverse financial action; it must
  not fire on a single mis-click when every lesser mutation (cancel match) already confirms.
- **Behaviour (EARS):** WHEN the admin invokes Mark-Paid on a commission from ANY of the three Mark-Paid call sites
  (Commission implementation reference, 246, 312), the system SHALL require an explicit confirmation (a dialog naming the
  commission **amount**, **match/tutor identity**, and explicit confirmation that the **agency received the
  tutor commission**, separate from parent lesson payments) before issuing the pay mutation; the pay mutation SHALL
  fire only on confirm and only via that dialog; cancelling the dialog SHALL make no change. The existing
  idempotency/guarded-money behaviour of the pay endpoint SHALL be retained.
- **Owner:** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario Outline: Mark-Paid asks for confirmation at every call site.
- Given a commission eligible to be paid surfaced through each supported payment action.
- When the admin clicks Mark Paid.
- Then a confirmation dialog appears naming the commission amount and the match/tutor.
- And the pay mutation fires only after explicit confirm.
- Examples:.
- Scenario: Cancelling the confirm makes no change (guard).
- Given the Mark-Paid confirm dialog.
- When the admin cancels.
- Then no pay request is sent and the commission stays unpaid.


- **Required evidence:** RTL test that confirm gates the mutation and cancel sends no request **across all three
  call sites**; live capture of the dialog (showing amount + match) before a pay. Under REQ-003 implementation reference.
- **Dependencies:** M5/INT-087 (commission pay endpoint).

---

## Traceability summary

| REQ | One-line | Type | Priority | Severity | Owner |
|---|---|---|---|---|---|
| REQ-001 | Conversion rate cohort-consistent, always ≤100% | functional/correctness | Should | S3 | data-engineer · business-analyst · software-engineer |
| REQ-002 | Avg-commission calc + label agree on one cohort; no cumulative÷active | functional/correctness | Should | S4 | data-engineer · software-engineer · frontend-expert |
| REQ-003 | Mark-Paid confirmation dialog (financial safety) | functional/safety | Must | S3 | frontend-expert |

## Open questions for the Boss / founder

1. **REQ-001 cohort definition: RESOLVED (founder/BA 2026-06-22).** PINNED to **completed-of-requested-in-period**
   (denominator = interviews with requested at in the period; numerator = the subset of those now COMPLETED;
   numerator ⊆ denominator → ≤100% by construction). The all-time-funnel and clamp-and-relabel options are REJECTED.
   No open question remains.
2. **REQ-002 definition:** what "avg commission per match" should mean (collected ÷ matches-with-commission, for
   which window) — business-analyst decides; relabel if the simplest correct definition differs from the current
   label.

## Current correction — 2026-09-05

- REQ-002: agency receipt of tutor commission and a numerator/denominator from one received-commission population are confirmed. The precise time cohort/date anchor remains **PENDING** in [current open decisions](../CURRENT-REQUIREMENTS.md); the selected-paid at formula in design md is a proposal only.
- REQ-001: apply later [026 REQ-007](../026-s3s4-polish-batch/spec.md) so period funnel tiles reconcile; the earlier permission to mix all-time tiles into that funnel is superseded.
- REQ-003: Mark paid explicitly confirms agency receipt of the tutor commission, separate from parent lesson payment. Apply [019](../019-admin-money-safety-and-analytics/spec.md) to the Matching-page entry point as well.
