# 42 — Interview Timeslot Negotiation (INT-039)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Use INT-039’s role/turn/slot-validation contract and the per-posting active cap. Proposal timestamps remain Coordinated Universal Time instants.
2. A parent counter transitions to COUNTER PROPOSED; previous round slots become WITHDRAWN. Do not substitute PROPOSED/REJECTED from the older descriptive diagram.
3. Preserve the specified error statuses and terminal/cancellation rules. Accepted slots lead to PENDING SCHEDULING; assignment/contact release is a separate flow.

## Authority

[Current requirements C07](../CURRENT-REQUIREMENTS.md) · [int-039-interview-negotiation md](../architecture/int-039-interview-negotiation.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 42 — Interview Timeslot Negotiation (INT-039)

## Trigger

A tutee wants to arrange an interview with a tutor. Instead of the admin picking a
time, the two parties propose and counter 15-minute Coordinated Universal Time timeslots until one is
mutually accepted. The agreed slot then lands in PENDING SCHEDULING for the admin
to finalise with a meeting link, or the admin can force-advance a stalled negotiation.

## Participants

- **Tutee / Tutor** — the two negotiating parties
- **Admin** — schedules or force-advances a stalled negotiation
- **frontend** — My Interviews / Tutor Interviews / Admin Interviews pages + Slot Picker/Whose Turn/Timeline user interface components
- **Interview Controller** — all interview endpoints (CREATE PATH, TUTEE BASE, TUTOR BASE, ADMIN BASE)
- **Interview Service** — state-machine logic, expiry sweep
- **database** — interviews table (extended with negotiation columns) + interview timeslots child table (migration 0034)

## Interview Status State Machine (INT-039)


The lifecycle diagram describes these state changes:

- Start leads to PROPOSED after tutee proposes 1-3 slots.
- PROPOSED leads to PENDING SCHEDULING after tutor accepts a slot.
- PROPOSED leads to COUNTER PROPOSED.
- PROPOSED leads to TUTOR DECLINED.
- PROPOSED leads to CANCELLED after tutee or tutor cancels.
- PROPOSED leads to EXPIRED after expiry sweep.
- COUNTER PROPOSED leads to PENDING SCHEDULING after tutee accepts a slot.
- COUNTER PROPOSED leads to PROPOSED after tutee counters.
- COUNTER PROPOSED leads to CANCELLED after tutee or tutor cancels.
- COUNTER PROPOSED leads to EXPIRED after expiry sweep.
- PENDING SCHEDULING leads to SCHEDULED.
- PENDING SCHEDULING leads to CANCELLED after admin cancels.
- PENDING SCHEDULING leads to PENDING SCHEDULING.
- SCHEDULED leads to COMPLETED after admin marks complete.
- SCHEDULED leads to NO SHOW after admin marks no-show.
- SCHEDULED leads to CANCELLED after admin cancels.


> **Legacy path:** REQUESTED (the pre-INT-039 initial status) is still present in
> the enum for backward compat; new interviews start in PROPOSED.  
> **Cap:** The 3-active-interview cap per parent posting counts
> REQUESTED, PROPOSED, COUNTER PROPOSED, PENDING SCHEDULING, and SCHEDULED
> — i e. any in-flight status. Index: idx interviews tutee post active (migration 0034).

## Diagram — Happy path: propose → accept → admin schedule


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — Counter round: tutor declines with alternative slots


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — Tutor declines (NOT INTERESTED) → terminal


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — Admin force-pending (stalled negotiation override)


Implementation example omitted. This public edition retains the written requirements in this section.


## Expiry sweep

A a scheduled task background task in Interview Service queries interviews with
status in the proposed and counter-proposed states past the expiry period and transitions them to
EXPIRED. The sweep is idempotent; running it twice on the same row is safe
(already-EXPIRED rows are excluded by the query). Each expiry increments the
interview expired Micrometer counter for monitoring.

Participants notified: INTERVIEW EXPIRED → tutee and tutor.

## Notable details

- Propose Interview Request enforces 1–3 slots; Counter Request and the slots
  field in Decline Interview Request (for CANT MAKE IT) do the same.
- All slot timestamps must be future 15-min Coordinated Universal Time boundaries within 7 days
  (validated in the service, not bean-validation, for typed error messages).
- database constraint ck timeslot quarter hour double-enforces the 15-min alignment
  at the database layer: the date part operation.
- whose Turn is a derived field computed in the service (not stored);
  it is present in detail-view responses and null in list-view responses.
- agreed Timeslot Id on the interviews row is the single source of truth for
  which slot was mutually agreed; Interview Response agreed Timeslot Id surfaces it.
- Source: Interview Controller implementation reference (all 20 endpoints), Interview Service implementation reference,
  migration 0034-interview-timeslot-negotiation implementation reference.

## Failure modes

- 422 INVALID STATUS TRANSITION — accept/counter/decline when it is not the
  caller's turn, or the interview is in a terminal/incompatible status.
- 404 — wrong-owner TUTEE or TUTOR access (existence not leaked).
- 403 — non-ADMIN on admin endpoints (Security Config + @Pre Authorize).
- 400 VALIDATION ERROR — empty slots list, > 3 slots, missing timeslot Id/reason.

## Related flows

- [13-tutee-request-interview md](13-tutee-request-interview.md) — the pre-INT-039 flow (request only; now superseded for new interviews but kept for reference)
- [14-admin-schedule-interview md](14-admin-schedule-interview.md) — admin scheduling (unchanged; PENDING SCHEDULING is now the expected entry status)
- [16-interview-completion-paths md](16-interview-completion-paths.md) — complete / cancel / no-show
- [43-tutee-tutor-job-invitation md](43-tutee-tutor-job-invitation.md) — job invitations (a separate signal path for tutee→tutor)
