> **Current-file check, 18 October 2026:** Updating the link currently saves it and updates reminder scheduling, but does not send the required new-link notice or write the required audit event (G05). See the [alignment review](../../docs/current-file-alignment.md). The requirements remain unchanged.

# 15 — Admin Sets Meeting Link on Scheduled Interview

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authorized admin updates a valid meeting link on a SCHEDULED interview.
2. Persist the change/audit and resend the updated link to both participants as required by product requirements document AC-17. The old claim of a deliberate silent update is withdrawn; no approved waiver was recovered.
3. A late link must create/update the 007 reminder when eligible. A removed link invalidates reminder delivery; rescheduling uses a fresh generation. WhatsApp OFF remains respected.

## Authority

[Current requirements C07](../CURRENT-REQUIREMENTS.md) · [product requirements document md](../PRD.md) · [requirements md](../007-generic-scheduler-db-scheduler/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 15 — Admin Sets Meeting Link on Scheduled Interview

## Trigger

Admin pastes a Zoom/Meet URL into Interview Detail implementation reference after the
interview has been SCHEDULED.

## Participants

- **Admin**
- **frontend Interview Detail implementation reference**
- **Interview Controller** — update operation for admin interviews id meeting-link
- **Interview Service**
- **Interview Repository**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Interview Controller implementation reference — @Patch Mapping.
- Service: Interview Service implementation reference.
- No notification is emitted on link set — the link is visible the next time
  the tutee opens the specified interview accessible to the parent. This is a deliberate decision (the
  interview is already scheduled; the link is a follow-up admin detail).
- Validation of the URL itself comes from @Valid Set Meeting Link Request (URL
  pattern asserted in the data transfer object).

## Failure modes

- 404 INTERVIEW NOT FOUND.
- 422 INVALID STATUS TRANSITION — interview not SCHEDULED.
- 400 invalid meeting Link format.
- 403 non-admin.

## Related flows

- [14-admin-schedule-interview md](14-admin-schedule-interview.md)
- [16-interview-completion-paths md](16-interview-completion-paths.md)
