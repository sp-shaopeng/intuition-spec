# 12 — Admin Match Cancel + Commission Cascade + Match State Machine

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authorized cancellation is limited to eligible pre-completion matches. Never cancel a COMPLETED match or erase collected PAID commission through this path.
2. Reopen the job and restore eligible competing applications as defined by the state-machine amendment; retain cause-specific closed/expired states.
3. Required audit is atomic; persist the former tutor’s in-app notification/outbox intent atomically and deliver externally after commit. The old no-notification Gap was already superseded by INT-016.

## Authority

[Current requirements C03](../CURRENT-REQUIREMENTS.md) · [int-093-batch-state-machine-integrity-spec md](../architecture/int-093-batch-state-machine-integrity-spec.md) · [assignment-be-spec md](../architecture/assignment-be-spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 12 — Admin Match Cancel + Commission Cascade + Match State Machine

## Trigger

Admin clicks Cancel on a PENDING or ACTIVE match. If a PENDING commission
exists, the M12 fix cascades the cancellation to that commission row so
"outstanding revenue" analytics don't silently inflate.

## Participants

- **Admin**
- **frontend Matching implementation reference**
- **Admin Match Controller** — submit operation for admin matches id cancel
- **Admin Match Service**
- **Match Repository, Commission Repository**

> **2026-06-03 (INT-016, Sev-1 — FIXED):** cancel previously **500'd** with a
> Lazy Initialization Exception. Root cause was NOT open-in-view: the M18 cancel→reopen step calls
> the reopen applications operation, a the modifying operation query that
> **detaches the persistence context mid-transaction**; the subsequent the get tutor operation/the get subject codes operation
> on the now-detached Match/Tutor Post proxies threw. Fixed by capturing the former-winner id + the
> response's subject codes **before** the clearing query and building the response from primitives.
> Cancel now also **reopens the job** (MATCHED→ACTIVE + winner/superseded apps→APPLIED) and emits an
> after-commit, failure-isolated "match cancelled / job reopened" notification to the former tutor
> (Admin Match Service cancel Match).

## Sequence diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Match status machine


The lifecycle diagram describes these state changes:

- Start leads to PENDING.
- PENDING leads to ACTIVE.
- PENDING leads to CANCELLED.
- ACTIVE leads to COMPLETED.
- ACTIVE leads to CANCELLED.


## Commission status machine (companion)


The lifecycle diagram describes these state changes:

- Start leads to PENDING after activate Match.
- PENDING leads to PAID after pay Commission.
- PENDING leads to CANCELLED.


## Notable details

- Controller: Admin Match Controller implementation reference.
- Service: Admin Match Service implementation reference. The comment block (lines 241-254)
  documents the design intent of the cascade — uncollectable amounts must not
  silently inflate outstanding revenue.
- CANCELLABLE STATUSES PENDING, ACTIVE (line 57).
- Commission PAID is intentionally NOT touched — agency already received payment.
- **Gap**: no notification is emitted on cancel — cancel Match lacks
  notification Service emit calls (unlike create and activate). The match
  dashboard reflects the new status on next fetch, but neither party gets a
  bell ping.

## Failure modes

- 404 MATCH NOT FOUND.
- 422 MATCH NOT CANCELLABLE — already COMPLETED or already CANCELLED.

## Related flows

- [09-admin-match-creation md](09-admin-match-creation.md)
- [10-admin-match-activation md](10-admin-match-activation.md)
- [11-admin-mark-commission-paid md](11-admin-mark-commission-paid.md)
