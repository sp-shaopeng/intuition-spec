# 14 — Admin Schedules Interview (REQUESTED → SCHEDULED)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authorized admin schedules from the permitted source state, including PENDING SCHEDULING, using the accepted slot where present.
2. Display Singapore time; serialize the admin endpoint using its retained format. Participant proposals keep their separate Coordinated Universal Time-instant contract.
3. Notify both participants. If a link exists and timing qualifies, register the 007 reminder. WhatsApp delivery remains conditional on all enablement/consent requirements.

## Authority

[Current requirements C10](../CURRENT-REQUIREMENTS.md) · [requirements md](../014-interview-scheduling-ux-and-admin-visibility/requirements.md) · [int-039-interview-negotiation md](../architecture/int-039-interview-negotiation.md) · [requirements md](../007-generic-scheduler-db-scheduler/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 14 — Admin Schedules Interview (REQUESTED → SCHEDULED)

## Trigger

Admin opens Interview Detail implementation reference, picks a date/time via the frontend
datetime-local input (which emits wall-clock time in the admin's local zone),
and submits.

## Participants

- **Admin** — SG admin, local zone Asia/Singapore
- **frontend Interview Detail implementation reference**
- **Interview Controller** — submit operation for admin interviews id schedule
- **Interview Service**
- **Interview Repository**
- **Notification Service**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Interview Controller implementation reference.
- Service: Interview Service implementation reference.
- Time conversion: the frontend datetime-local input emits a wall-clock
  Local Date Time (no zone info). The backend deserialises that into
  the scheduled at operation, then interprets it as **Singapore time** via
  the at zone operation and stores as Offset Date Time
  (Interview Service implementation reference). Coordinated Universal Time clients posting raw Coordinated Universal Time would still
  end up shifted into Singapore time — by design, since the only zone the frontend form supports
  is the admin's local SG time.
- Only the tutee is notified — not the tutor. (Tutor was already notified at
  the request stage in flow 13.)

## Failure modes

- 404 INTERVIEW NOT FOUND.
- 422 INVALID STATUS TRANSITION — interview not REQUESTED.
- 400 — invalid Schedule Interview Request (missing/malformed scheduled At).
- 403 non-admin.

## Related flows

- [13-tutee-request-interview md](13-tutee-request-interview.md)
- [15-admin-set-meeting-link md](15-admin-set-meeting-link.md)
- [16-interview-completion-paths md](16-interview-completion-paths.md)
