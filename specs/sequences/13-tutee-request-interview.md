# 13 — Tutee Requests Interview (from Tutor Detail modal)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate the parent and verify ownership of a live, non-deleted PENDING or ACTIVE job under 021.
2. Identify the tutor by user identity with optional listing; validate the proposed slots and enforce the per-posting active interview cap.
3. Use the INT-039 proposal/accept/counter state machine. Initiation does not publish the post, create a match or reveal contact.

## Authority

[Current requirements C06](../CURRENT-REQUIREMENTS.md) · [spec md](../021-tutee-request-flow/spec.md) · [int-039-interview-negotiation md](../architecture/int-039-interview-negotiation.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 13 — Tutee Requests Interview (from Tutor Detail modal)

## Trigger

Authenticated tutee on Tutor Detail implementation reference clicks "Request Interview".
The modal opens; tutee enters an optional note and submits.

## Participants

- **Tutee**
- **frontend Tutor Detail implementation reference** (Modal)
- **Interview Controller** — submit operation for interviews
- **Interview Service**
- **Tutor Post Repository, Interview Repository, User Repository**
- **Notification Service**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- frontend controller: Tutor Detail implementation reference.
- backend controller: Interview Controller implementation reference.
- backend service: Interview Service implementation reference.
- safe Title truncates the tutor-post title to 100 chars (97 + "...") before
  interpolating it into the notification body — see Interview Service implementation reference.
  Without this guard a long title could overrun the varchar using 500 body column
  and roll back the parent transaction.
- The admin fan-out is a per-admin loop (the find by role operation)
  inside the same transaction.

## Failure modes

- 404 TUTOR POST NOT FOUND.
- 422 TUTOR POST NOT ACTIVE.
- 422 DUPLICATE INTERVIEW — tutee already has REQUESTED or SCHEDULED interview
  for this tutor post.
- 403 if non-tutee.
- 404 USER NOT FOUND if tutee somehow does not exist (should not happen with
  valid signed session token).

## Related flows

- [14-admin-schedule-interview md](14-admin-schedule-interview.md)
- [15-admin-set-meeting-link md](15-admin-set-meeting-link.md)
- [16-interview-completion-paths md](16-interview-completion-paths.md)
- [21-notification-bell-flow md](21-notification-bell-flow.md)
