# 28 — Tutor Applies to / Withdraws from a Job (M15 Applications)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate the tutor and require the canonical profile-completeness predicate. A published listing is not required and tutor Post Id remains optional.
2. Validate the live approved job, deletion predicate, quota and existing application state before applying. Use the INT-093 reapply/withdraw rules rather than the original M15 two-state simplification.
3. Retain rows and idempotency. Parent-owned applicant views are allowed; applying does not reveal contact or create a match.

## Authority

[Current requirements C05](../CURRENT-REQUIREMENTS.md) · [requirements md](../012-tutor-apply-and-profile-ux/requirements.md) · [int-093-batch-state-machine-integrity-spec md](../architecture/int-093-batch-state-machine-integrity-spec.md)

## Contract diagram


Implementation example omitted. This public edition retains the written requirements in this section.


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 28 — Tutor Applies to / Withdraws from a Job (M15 Applications)

## Trigger

An authenticated tutor browsing active tutee requests (/requests,
Browse Requests) clicks **Apply** on a job. A later **Withdraw** flips the
application closed; the same tutor may then re-apply (which reactivates the same
durable row).

> Status: M15 is **implemented and team-reviewed** (2026-05-29). This diagram
> reflects the shipped thin slice
> ([spec implementation reference](../modules/applications/spec.md),
> [ADR-013](../ROADMAP.md)). Only APPLIED/WITHDRAWN transitions
> exist; no applicant count is surfaced yet; no tutee notification is emitted (the
> earlier APPLICATION RECEIVED notification was cut in review).

## Participants

- **Tutor**
- **frontend** — Apply Button (on Browse Requests) / Applications page
- **Tutor Application Controller** — tutor applications interface operation
- **Job Application Service**
- **Tutee Post Repository, Tutor Post Repository, Job Application Repository**
- **Analytics Event Publisher**

## Diagram — Apply (idempotency + withdraw → re-apply reactivation)


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — Withdraw


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- **Idempotency:** a duplicate apply while an APPLIED row exists returns that row
  (200), never a second insert. The partial unique index is the database backstop; a lost
  concurrent race surfaces as a handled **409 CONSTRAINT VIOLATION** (not a 500).
- **Re-apply reactivates in place:** there is exactly one durable row per (job,
  tutor) pair; withdraw → re-apply flips that same row back to APPLIED.
- **Withdraw is a status flip, not a DELETE** — the row is retained
  (db md §4 revokes DELETE); withdrawing an already-withdrawn application is a
  no-op 200; withdrawing another tutor's application returns 404 (no-leak).
- **No tutee notification** in this slice (cut in review). Funnel signal comes from
  the application submitted / application withdrawn analytics events.
- **No count surfaced:** applicant Count is M14's job and renders nothing until M14
  ships; the count query is pinned in
  [db implementation reference](../modules/applications/db.md).

## Failure modes

- 404 TUTEE POST NOT FOUND / 404 APPLICATION NOT FOUND (incl. not-owned, no-leak).
- 422 TUTEE POST NOT ACTIVE — job not accepting applications.
- 403 TUTOR NO ACTIVE POST — applying tutor holds no ACTIVE listing.
- 400 / 403 TUTOR POST INVALID — pinned tutor post not found / not owned.
- 409 CONSTRAINT VIOLATION — lost concurrent double-apply race.
- 403 — non-tutor on a tutor endpoint; non-admin on the applicant list.

## Related flows

- [09-admin-match-creation md](09-admin-match-creation.md) — admin matches from
  the applicant pool (later slice).
- [13-tutee-request-interview md](13-tutee-request-interview.md) — sibling
  supply↔demand signal flow.
- [21-notification-bell-flow md](21-notification-bell-flow.md).
