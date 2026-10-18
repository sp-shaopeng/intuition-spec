# 26 — Email Queue Fanout on Event

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Write the business mutation, required audit, in-app notification rows and durable outbox intent atomically. Dispatch to the approved stream/consumer transport after commit.
2. Email uses the approved Resend path. Missing credentials/disabled transport means deferred work, not successful delivery or ACK.
3. Bound retries and park permanent failures in the failed-message queue with an explicit replay disposition. Preserve replayable work when limiting stream size, and drain in-flight consumers safely on shutdown.
4. Observe pending/delivery/failed-message queue states separately. The old missing-consumer/SES snapshot is historical, not the current delivery contract.

## Authority

[Current requirements C12](../CURRENT-REQUIREMENTS.md) · [transactional-messaging-and-idempotency md](../architecture/transactional-messaging-and-idempotency.md) · [int-087-transactional-messaging-hardening-spec md](../architecture/int-087-transactional-messaging-hardening-spec.md) · [spec md](../027-aws-production-deployment/spec.md)

## Contract diagram


Implementation example omitted. This public edition retains the written requirements in this section.


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 26 — Email Queue Fanout on Event

## Trigger

Any service path that enqueues an email — currently:

- Auth Service forgot Password → template N-05 (string literal, one-time passcode code).
- Admin Post Service queue Approval Email → Email Template Keys POST APPROVED.
- Admin Post Service queue Rejection Email → Email Template Keys POST REJECTED.

There are no other producers in the backend tree today. (User registration,
match notifications, interview transitions etc. only emit in-app notifications
via Notification Service emit; no email is queued.)

## Participants

- **Caller service** (e g. Admin Post Service)
- **Email Queue Service** — @Async Spring bean
- **Redis Stream** — STREAM KEY email-notifications
- **Email consumer** — **GAP**: not present in this monorepo

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Implementation: Email Queue Service implementation reference.
  @Async so the producing transaction is not blocked on Redis I/O. Stream key
  is STREAM KEY email-notifications (line 21).
- The message map includes template Id, to, enqueued At, and any
  caller-supplied params (lines 26-32).
- Template keys are centralised in Email Template Keys implementation reference — but the
  forgot-password producer uses a string literal N-05 rather than a
  constant. Candidate clean-code fix.
- Critical sequencing: producers wrap enqueue in after Commit so a rolled-back
  transaction never produces a stray email. See Auth Service implementation reference and
  Admin Post Service queue methods (the queue methods themselves do NOT wrap;
  they are called inside a database transaction that saves all related changes together service methods, but enqueue happens
  inline since failure is logged-only — see line 422-434).

> Audit: this is a real inconsistency. Auth Service forgot Password defers
> enqueue with after Commit (line 420); Admin Post Service queue Approval Email
> does not. Approve/reject emails will fire even if the post status save fails
> later in the transaction (unlikely but theoretically possible). Worth flagging
> for review.

## Failure modes

- XADD failure → caught in Email Queue Service enqueue (lines 33-36) and
  logged; the caller never sees the failure.
- Consumer outage → messages accumulate on the stream. There is no
  back-pressure or failed-message queue in the producer side.
- No retry/back-off built into producer — emails are fire-and-forget from the
  monolith's perspective.

## Gaps

- **No consumer in this monorepo.** The email-notifications stream is
  produced into but never read. Either an external worker / Lambda exists, or
  no email has actually been delivered in any environment running this exact
  tree. The Dev Ops spec should clarify.
- Email template N-05 is referenced as a string literal in
  Auth Service forgot Password rather than via Email Template Keys.

## Related flows

- [04-forgot-reset-password md](04-forgot-reset-password.md)
- [08-admin-approval md](08-admin-approval.md)
