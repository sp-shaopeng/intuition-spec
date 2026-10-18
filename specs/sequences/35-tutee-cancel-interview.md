# Sequence 35 — Tutee cancels their own interview

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate the parent and verify ownership of the interview.
2. Permit cancellation in the INT-039-defined non-terminal states, including negotiation; reject attempts to modify terminal states.
3. Release the active cap, preserve cause/audit and invalidate the reminder including stale in-flight generations.

## Authority

[Current requirements C07](../CURRENT-REQUIREMENTS.md) · [int-039-interview-negotiation md](../architecture/int-039-interview-negotiation.md) · [requirements md](../007-generic-scheduler-db-scheduler/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# Sequence 35 — Tutee cancels their own interview

**Module:** M6 Interview · **Actors:** Tutee (interview requester)
**Endpoint:** submit operation for tutee interviews id cancel

Complements seq 13 (tutee *requests*) and seq 16 (admin completion paths): here the tutee
withdraws a pending/scheduled interview themselves.

## Preconditions
- Tutee is authenticated and owns the interview.
- Interview is in REQUESTED or SCHEDULED (not a terminal state).

## Flow


Implementation example omitted. This public edition retains the written requirements in this section.


## Postconditions
- Interview status is CANCELLED; counterpart receives an INTERVIEW CANCELLED notification.
- Frees a slot against the max-3-active-interviews-per-posting cap.

## Error cases
- 422 invalid status (already COMPLETED / NO SHOW / CANCELLED).
- 401 unauthenticated; 403 not the owner; 404 not found.
- Interview Status: REQUESTED · SCHEDULED · COMPLETED · CANCELLED · NO SHOW.

## Related
- Request: [13-tutee-request-interview md](13-tutee-request-interview.md)
- Admin paths (schedule/complete/no-show/cancel): [16-interview-completion-paths md](16-interview-completion-paths.md)
- Regression: Interviews IT, Admin Interview Ops IT (admin cancel + no-show).
