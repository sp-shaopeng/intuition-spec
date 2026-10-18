# 19 — Tutee Submits Review

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Trigger

From the match dashboard, tutee clicks "Leave review" on a COMPLETED match
where my Review Submitted equals no. Lands on Submit Review implementation reference.

## Participants

- **Tutee** (or **Tutor** — same endpoint, role-agnostic)
- **frontend Submit Review implementation reference**
- **Review Controller** — submit operation for matches match Id reviews
- **Review Service**
- **Match Repository, Review Repository**
- **Notification Service**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- frontend: Submit Review implementation reference.
- Controller: Review Controller implementation reference.
- Service: Review Service implementation reference.
- Race-safety: an idempotent pre-check at line 106 plus database unique constraint on
  match id, reviewer id — concurrent submission triggers
  Data Integrity Violation Exception, translated to the same 422 DUPLICATE REVIEW
  the frontend already handles (lines 121-130).
- Reviewer display name is resolved via the reviewer's tutor profile (preferred)
  or tutee profile fallback (resolve Display Name line 211-219). Never the bare
  user id.
- Aggregate refresh: the tutor's average Rating / review Count is computed on
  every tutor-post list/detail via
  the get ratings for reviewee ids operation and surfaced through
  the to response operation (file
  Tutor Post Service implementation reference). It is **not** stored as a denormalised
  column on Tutor Post — every list query recomputes it via database query language aggregation.

## Failure modes

- 404 MATCH NOT FOUND.
- 422 INVALID STATUS — match not COMPLETED.
- 403 NOT PARTICIPANT — reviewer is neither the matched tutee nor the
  matched tutor.
- 422 DUPLICATE REVIEW — pre-check or unique-constraint race.
- 400 — Submit Review Request validation (rating bounds, comment max length).

## Related flows

- [17-tutee-views-matches md](17-tutee-views-matches.md) (my Review Submitted flag)
- [18-tutor-views-matches md](18-tutor-views-matches.md)
- [20-public-tutor-detail-reviews md](20-public-tutor-detail-reviews.md) — where aggregates surface publicly
- [21-notification-bell-flow md](21-notification-bell-flow.md)
