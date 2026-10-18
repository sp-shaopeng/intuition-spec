> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 017
slug: admin-analytics-and-financial-safety
title: Admin analytics correctness + financial safety — conversion >100% cohort bug, avg-commission calc, Mark-Paid confirm
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED
owner: requirements-engineer   # build owners: software-engineer / data-engineer / bff-specialist / frontend-expert
created: 2026-06-22
updated: 2026-09-05
---

# Admin analytics correctness + financial safety

> **Documentation correction — 2026-09-05:** Agency receipt meaning and later funnel-tile consistency; exact average time cohort explicitly pending. Current clauses in requirements/design/tasks supersede earlier conflicting directions. Historical HISTORICALLY MARKED IMPLEMENTED labels, dated changelog entries and evidence are retained as history; this correction makes no new runtime, test or deployment claim.


Fixes two analytics-correctness defects and adds a financial-safety guard, from the 2026-06-22 sequential re-run
(specs/issues/review-2026-06-22/CONSOLIDATED md):

- **Conversion >100% (S3, cohort bug)** — the interview-funnel "Conversion Rate" showed "400%".
  Analytics Service compute Conversion Rate (158-166) = completed / requested, fed by build Interview Funnel
  (147-155) from two independent **all-time** count By Status gauges (REQUESTED, COMPLETED) — different
  populations, so completed requested → a ratio above 100%. **Fix PINNED (founder/BA 2026-06-22):
  completed-of-requested-in-period** — denominator = interviews with requested at in the period
  (count By Requested At Between); numerator = the subset of that cohort now COMPLETED; numerator ⊆ denominator →
  ≤100% by construction (no clamp). The all-time-funnel and clamp-and-relabel options are REJECTED.
- **Avg Commission / Match (S4, label/calc mismatch)** — the compute avg commission operation (200-207)
  = collected / active Matches, where collected is the (period or grand) total collected and active Matches is a
  point-in-time gauge of ACTIVE matches. Dividing a cumulative total by a current-active count is not "average
  commission per match" in any consistent cohort — the label and the calc disagree.
- **Mark-Paid confirm (S3, financial safety)** — Commission implementation reference fires mark Paid Mutation mutate directly with
  NO confirmation dialog, while lower-consequence actions (cancel match, 201 → cancel Confirm Id) DO confirm.
  Marking a commission PAID is the highest-consequence, hardest-to-undo financial action and must confirm.

backend analytics (Analytics Service) + frontend (Mark-Paid confirm, labels) lane. Touches the analytics service/data transfer object + the
admin Commission/Analytics pages — no overlap with the other review specs.

- **Requirements:** [requirements md](requirements.md) · **Design:** [design md](design.md) · **Tasks:** [tasks md](tasks.md)
- **Evidence:** the private evidence directory for the requirement

## Historical delivery/review status

The record below describes the earlier delivery cycle; it is not approval or verification of the 2026-09-05 correction.

DRAFT — author does not self-approve. Boss convenes a scoped team-review (data-engineer, software-engineer,
bff-specialist, frontend-expert, business-analyst) + audit; flips to APPROVED when sign-off boxes are checked. The
metric semantics (cohort definition for conversion; the correct denominator/label for avg-commission) must be
agreed with the business-analyst/data-engineer before build.

## Review & Sign-Off
- [ ] Author — requirements (requirements-engineer)
- [ ] Counterpart — metric semantics: cohort-consistent conversion + correct avg definition (data-engineer · business-analyst)
- [ ] Counterpart — the analytics query/calc fix (software-engineer)
- [ ] Counterpart — analytics payload/label contract (bff-specialist)
- [ ] Counterpart — Mark-Paid confirm dialog + corrected labels (frontend-expert)
- [ ] Boss (final sign-off → flips status to APPROVED)

## Changelog

- 2026-09-05 — Documentation alignment correction: Agency receipt meaning and later funnel-tile consistency; exact average time cohort explicitly pending. No application code or runtime verification in this change. Earlier entries below are historical.
- 1.2 (2026-06-22) **Founder/BA decision folded (WAVE2-SPEC-REVIEW md §017/REQ-001 + cross-cutting risk #2).**
  REQ-001 conversion denominator PINNED to **completed-of-requested-in-period** (denominator =
  the count by requested at between operation; numerator = the subset of that requested at cohort with status is COMPLETED
  via a new count By Requested At Between And Status-style query; numerator ⊆ denominator → ≤100% by construction).
  REMOVED the two other cohort candidates (all-time funnel; clamp-and-relabel) and the clamp fallback from the EARS,
  design, and open-question. Defined numerator/denominator precisely against the real timestamps (Interview has no
  completed at — completion is status is COMPLETED). Added a cohort test (10 requested-in-period, 4 COMPLETED, +
  out-of-cohort completions → exactly 40%, never >100%). status stays APPROVED.
- 1.1 (2026-06-22) Wave-1 spec-review APPROVED-condition folded in (WAVE1-SPEC-REVIEW md §017). REQ-003: the confirm
  gate now covers **all three** Mark-Paid call sites (Commission implementation reference, 246, 312) — verified in source —
  via a pay Confirm Id dialog, not only the first; the dialog must name the commission amount + the match/tutor
  identity. RTL test asserts confirm-gates + cancel-no-op across all three sites. (REQ-001/002 analytics calc stay
  Wave-2, not gated here.) status stays DRAFT pending Boss re-gate.
- 1.0 (2026-06-22) initial draft — ADDED REQ-001 (conversion rate is cohort-consistent and can never exceed 100%),
  REQ-002 (avg-commission calc + label agree on one cohort/denominator), REQ-003 (Mark-Paid confirmation dialog).
  Grounded against Analytics Service compute Conversion Rate/compute Avg Commission + admin Commission/Analytics pages.
  status DRAFT pending scoped team-review.
