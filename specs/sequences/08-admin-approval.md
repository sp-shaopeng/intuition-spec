# 08 — Admin Approval (review queue → approve / reject) + Post State Machine

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authorized admin reviews a PENDING post through the paginated/filterable queue. Approve/reject eligibility uses canonical states; PAUSED FOR REVIEW is not a separate state.
2. Required audit and state changes commit atomically. Rejection/closure/removal clean up associated open invitations/applications/interviews according to021 and the state-machine amendments.
3. ACTIVE/PENDING close is a separate authorized action with cause-preserving cascades. Owner edits, committed matches and logical deletion retain their own guards.

## Authority

[Current requirements C06](../CURRENT-REQUIREMENTS.md) · [spec md](../019-admin-money-safety-and-analytics/spec.md) · [spec md](../021-tutee-request-flow/spec.md) · [int-094-batch-soft-delete-integrity-spec md](../architecture/int-094-batch-soft-delete-integrity-spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 08 — Admin Approval (review queue → approve / reject) + Post State Machine

## Trigger

Admin loads Review Queue implementation reference to see the pending queue, then opens a
Review Detail implementation reference to approve or reject.

## Participants

- **Admin** — ROLE ADMIN
- **frontend Review Queue implementation reference / Review Detail implementation reference**
- **Admin Post Controller** — admin posts * interface operation
- **Admin Post Service**
- **Tutor Post Repository, Tutee Post Repository, Tutor Profile Repository, Tutee Profile Repository, Audit Log Repository**
- **Encryption Service** — decrypts personal information for admin view only
- **Email Queue Service** — POST APPROVED or POST REJECTED templates

## Sequence diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Post status machine


The lifecycle diagram describes these state changes:

- Start leads to PENDING.
- PENDING leads to ACTIVE.
- PENDING leads to REJECTED.
- ACTIVE leads to PENDING.
- DRAFT leads to PENDING.
- DRAFT leads to CLOSED.
- REJECTED leads to CLOSED.
- ACTIVE leads to CLOSED.


## Notable details

- Controller: Admin Post Controller implementation reference (BASE PATH admin posts interface operation).
- Service: Admin Post Service implementation reference. Defence-in-depth: class-level
  the pre authorize operation AND Security Config URL rule
  the request matchers operation.
- N+1 fix: pending queue does two batch profile lookups
  (find All By User Id In) — Admin Post Service implementation reference.
- personal information is decrypted via decrypt Safe (line 381) — never returned by public
  endpoints; only by the specified post accessible to an administrator and the specified interview accessible to an administrator.
- Audit log uses target Entity Id post Id, NOT user Id — see comment in
  write Audit Log (line 411).
- Email body intentionally contains only a dashboard URL — never personal information or contact
  details (lines 426, 440).
- Audit failures are swallowed (lines 416-419) so audit infra issues never
  block the admin operation.

## Failure modes

- 404 NOT FOUND — id resolves to neither tutor nor tutee post.
- 409 ALREADY REVIEWED — idempotency guard; the post has already exited PENDING.
- 400 — missing/blank reject reason (@Valid Reject Post Request).
- 403 — non-admin (filter or method security).
- Email enqueue failure swallowed and logged (lines 432, 446).

## Related flows

- [06-tutee-post-creation md](06-tutee-post-creation.md), [07-tutor-post-creation md](07-tutor-post-creation.md)
- [09-admin-match-creation md](09-admin-match-creation.md)
- [26-email-fanout-on-event md](26-email-fanout-on-event.md)
