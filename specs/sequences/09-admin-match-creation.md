# 09 — Admin Match Creation

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Parent choosing is a selection request, not a match. Only authorized admin assigns after validating the approved open job, tutor identity/profile and current selection.
2. Use tutor User Id; an attached tutor Post Id is optional. Atomically create the PENDING match, set the job MATCHED, select the winner and supersede competing intake; enforce uniqueness and idempotency.
3. Required audit is atomic. Persist in-app participant notification rows/outbox intents in the transaction; dispatch external delivery after commit. No contact is released by choosing alone.

## Authority

[Current requirements C05](../CURRENT-REQUIREMENTS.md) · [int-034-free-market-assign md](../architecture/int-034-free-market-assign.md) · [assignment-be-spec md](../architecture/assignment-be-spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 09 — Admin Match Creation

## Trigger

Admin operates Matching implementation reference: selects an ACTIVE tutor post and an
ACTIVE tutee post, hits Create Match.

## Participants

- **Admin**
- **frontend Matching implementation reference**
- **Admin Match Controller** — submit operation for admin matches
- **Admin Match Service**
- **Tutor Post Repository, Tutee Post Repository, Match Repository**
- **Notification Service**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Admin Match Controller implementation reference. Returns 201 Created.
- Service: Admin Match Service implementation reference.
- Class-level comment notes that contact details are NOT shared at this stage —
  the frontend success screen reads "No contact details have been shared yet" (line 41-45 of service comment).
- Both notifications are emitted within the same transaction; they will
  rollback together if the match save fails downstream.
- matched At uses Zone Offset Coordinated Universal Time.

## Failure modes

- 404 TUTOR POST NOT FOUND / 404 TUTEE POST NOT FOUND.
- 422 TUTOR POST NOT ACTIVE / 422 TUTEE POST NOT ACTIVE.
- 409 DUPLICATE MATCH — only PENDING or ACTIVE statuses count as open;
  COMPLETED or CANCELLED matches do not block re-matching.
- 403 non-admin via dual security layers.
- 400 validation errors on Create Match Request.

## Related flows

- [10-admin-match-activation md](10-admin-match-activation.md)
- [12-admin-match-cancel-cascade md](12-admin-match-cancel-cascade.md)
- [21-notification-bell-flow md](21-notification-bell-flow.md)
