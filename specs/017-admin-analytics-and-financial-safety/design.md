# Design — Admin analytics correctness + financial safety (017)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Owned by data-engineer + business-analyst (semantics) + software-engineer (calc) + bff-specialist (payload) +
> frontend-expert (confirm/labels). Grounded 2026-06-22.

## REQ-001 — conversion cohort (PINNED: completed-of-requested-in-period)

**Cohort PINNED (founder/BA 2026-06-22) — the clamp/relabel and all-time options are REJECTED.** build Interview Funnel
(Analytics Service implementation reference) today feeds the compute conversion rate operation (158-166) two
**all-time** count By Status gauges — different populations, hence >100%. Re-base both on ONE cohort:

- **Denominator** = the count by requested at between operation (already exists, 69-70) — the
  interviews requested in the period.
- **Numerator** = the subset of THAT cohort whose status COMPLETED. Add a repository query
  the count by requested at between and status operation (mirror the existing count By Requested At Between
  derived query, +status). Interview has no completed at column (verified) — completion is status is COMPLETED, so
  "requested-in-period AND now COMPLETED" is the numerator. It is a STRICT SUBSET of the denominator → ratio ∈ [0,1]
  by construction; **no clamp is required** (and none is added — a clamp would mask a cohort regression).
- compute Conversion Rate keeps its requested equals 0 → 0.0 guard (159-161); now requested IS the cohort
  denominator so a zero cohort → 0% cleanly.
- build Interview Funnel must take from Ts, to Ts (like build Match Funnel/build Revenue Summary already do,
  170,189) so the conversion is period-scoped. Under later [026 REQ-007](../026-s3s4-polish-batch/spec.md),
  tiles presented as this funnel must reconcile with the same selected requested-at cohort. Clearly separated
  lifetime metrics may remain totals; do not mix them into the displayed period funnel. data transfer object field
  name conversion Rate unchanged (additive query only).
- **Test:** a known fixture — e g. 10 interviews with requested at in-period, 4 of them COMPLETED, PLUS some
  COMPLETED interviews requested BEFORE the period — asserts the rate is exactly 0.40 (4/10), proving out-of-cohort
  completions do not inflate; the prior 400% scenario now ≤100%; zero-cohort → 0%.

## REQ-002 — avg-commission cohort (exact time cohort PENDING)

**Confirmed:** collected commission means money actually received by the agency from the tutor. The numerator
and distinct-match denominator must describe the same received-commission population; an ACTIVE-match gauge
cannot be its denominator. Parent lesson payments are not agency collection.

**Pending business decision:** selected-period versus lifetime scope and the exact date anchor. The later 019
statement that code is correct is historical implementation evidence, not ratification of this metric. Track
the decision in [CURRENT-REQUIREMENTS open decisions](../CURRENT-REQUIREMENTS.md).

**Proposal only, not approved:** sum agency commission receipts whose paid at is in the selected period,
divide by distinct matches contributing those receipts, return 0 for an empty cohort, and label it
"Average commission received per paid match (selected period)". Do not implement or mark this exact definition
accepted until the founder/BA decision is recorded. Existing received-money semantics and the same-cohort
invariant are not pending.

## REQ-003 — Mark-Paid confirm

Mirror the existing cancel-confirm pattern (Commission implementation reference cancel Confirm Id → 325-348 confirm dialog). Add a
pay Confirm Id state (carrying the commission id + the match/amount to display) + a confirm dialog that **names the
commission amount and the match/tutor identity** so the admin verifies WHICH commission before this irreversible,
highest-consequence action. The confirmation explicitly states that the agency has received the tutor
commission; it is not confirmation of parent lesson payment or of an amount merely owed.
mark Paid Mutation mutate fires only on explicit confirm. Later 019 extends the same guard to Matching.

**Gate ALL THREE Mark-Paid call sites — not just the first.** mark Paid Mutation mutate is invoked from three
on Mark Paid handlers (verified in source):

- Commission implementation reference — the match-card action that records receipt for its commission
- Commission implementation reference — the commission-list action that records receipt for the selected commission
- Commission implementation reference — the match-detail action that records receipt for its commission

Every one must route through the pay Confirm Id dialog (set pay Confirm Id instead of calling .mutate directly);
the mutation is only ever called from the dialog's confirm handler. A confirm on one path is a false sense of safety.
Keep the guarded-money idempotency key (85/86). frontend-only.

## Test / coverage

- backend gate ≥97.5/97.5 — new calc unit tests (the 400% case now ≤100%; avg over the correct cohort; zero guards).
- frontend gate ≥97.5/88 — Mark-Paid confirm gates the mutation; corrected label rendering.
- Live: dashboard shows a sane conversion % + corrected avg label; Mark-Paid dialog before paying.
