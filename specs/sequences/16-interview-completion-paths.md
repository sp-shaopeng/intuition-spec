# 16 — Interview Terminal Paths (complete / cancel / no-show) + Interview State Machine

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Use the full INT-039 interview state model. Parent/admin cancellation eligibility covers the specified non-terminal negotiation states, not only REQUESTED/SCHEDULED.
2. No-show accepts TUTEE, TUTOR or BOTH under the retained product requirements document requirement. The typed-enum implementation note did not authorize removing BOTH.
3. Completed/cancelled/no-show interviews are terminal. Cancel any pending reminder and reject stale/in-flight delivery through007’s state/generation guards.

## Authority

[Current requirements C07](../CURRENT-REQUIREMENTS.md) · [int-039-interview-negotiation md](../architecture/int-039-interview-negotiation.md) · [product requirements document md](../PRD.md) · [requirements md](../007-generic-scheduler-db-scheduler/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 16 — Interview Terminal Paths (complete / cancel / no-show) + Interview State Machine

## Trigger

After an interview transitions to SCHEDULED, exactly one of three admin endpoints
moves it to a terminal state. There is also a separate tutee-driven cancel
endpoint usable while REQUESTED or SCHEDULED.

## Participants

- **Admin** or **Tutee** (tutee for cancel My Interview only)
- **Interview Controller** — admin: complete, cancel, no-show; tutee:
  cancel My Interview
- **Interview Service**
- **Interview Repository**
- **Notification Service**

## Sequence — complete


Implementation example omitted. This public edition retains the written requirements in this section.


## Sequence — admin cancel


Implementation example omitted. This public edition retains the written requirements in this section.


## Sequence — no-show


Implementation example omitted. This public edition retains the written requirements in this section.


## Sequence — tutee self-cancel


Implementation example omitted. This public edition retains the written requirements in this section.


## Interview status machine


The lifecycle diagram describes these state changes:

- Start leads to REQUESTED after tutee request Interview.
- REQUESTED leads to SCHEDULED after admin schedule Interview.
- REQUESTED leads to CANCELLED after tutee cancel My Interview / admin admin Cancel Interview.
- SCHEDULED leads to COMPLETED after admin complete Interview.
- SCHEDULED leads to CANCELLED after tutee cancel My Interview / admin admin Cancel Interview.
- SCHEDULED leads to NO SHOW.


## Notable details

- Controller: Interview Controller implementation reference (sections marked Admin and Tutee).
- Complete: Interview Service complete Interview — Interview Service implementation reference.
- Admin cancel: Interview Service admin Cancel Interview — Interview Service implementation reference.
- No-show: Interview Service mark No Show — Interview Service implementation reference. Note: the notification goes to the
  **other** party — i e. if the actor operation then the **tutor** is
  notified, and vice versa. The no-show actor is not notified.
- Tutee self-cancel: Interview Service cancel My Interview — Interview Service implementation reference. Ownership check returns
  404 (not 403) when the interview belongs to a different tutee — see
  load Interview Owned By Tutee line 410.
- All terminal-transition notifications use the safe title operation to bound the body
  length.

## Failure modes

- 404 INTERVIEW NOT FOUND — also raised for "not owned by tutee" in self-cancel
  (existence is not leaked).
- 422 INVALID STATUS TRANSITION — wrong source status.
- 400 — No Show Request actor must be TUTEE or TUTOR.
- 403 — non-admin on admin endpoints; non-tutee on tutee endpoint.

## Related flows

- [13-tutee-request-interview md](13-tutee-request-interview.md)
- [14-admin-schedule-interview md](14-admin-schedule-interview.md)
- [15-admin-set-meeting-link md](15-admin-set-meeting-link.md)
- [21-notification-bell-flow md](21-notification-bell-flow.md)
