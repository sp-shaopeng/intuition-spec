# 21 — Notification Bell + Inbox

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Read and mutate notifications only for the authenticated recipient; another recipient’s notification is not disclosed. Preserve idempotent read and count refresh.
2. Use the INT-147 structured invitation data for humanized job/tutor labels and action destinations; malformed legacy data receives the documented safe fallback.
3. Notification display/acknowledgement is separate from successful email/WhatsApp delivery and from invitation acceptance.

## Authority

[Current requirements C07](../CURRENT-REQUIREMENTS.md) · [int-147-structured-notification-data-spec md](../architecture/int-147-structured-notification-data-spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 21 — Notification Bell + Inbox

## Trigger

Notification Bell mounts in Nav implementation reference for any authenticated user. It polls
/notifications/unread-count every 60 s. Clicking navigates the user to their
role-specific inbox where individual notifications can be marked read.

## Participants

- **User** (any role)
- **frontend Notification Bell implementation reference** — polls unread-count
- **frontend Notifications page** (/tutee/notifications, /tutor/notifications,
  /admin/notifications)
- **Notification Controller** — read operation for unread-count, GET /, submit operation for id read,
  submit operation for read-all
- **Notification Service**
- **Notification Repository**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- frontend bell: Notification Bell implementation reference.
  REFETCH INTERVAL MS 60 000. Display caps at 9+.
- Role-routing on click: lines 17-22 of the bell. The admin path is
  /admin/notifications even though no admin-only logic exists server-side —
  notifications are universally scoped by recipient.
- Controller: Notification Controller implementation reference.
- Service: Notification Service implementation reference. Existence is deliberately not
  leaked on cross-user mark Read — 404, not 403 (lines 89-92).
- mark All Read is a single bulk database query language update (not a loop) — see
  mark All Read For Recipient in the repository.

## Failure modes

- 401 — signed session token missing/invalid.
- 404 NOTIFICATION NOT FOUND — notification id absent OR owned by another user.
- Polling continues on transient errors (React Query default retry semantics);
  the badge stays stale until the next successful fetch.

## Related flows

- Any flow that calls notification Service emit — 09, 10, 11, 13, 14, 16, 19.
- [05-csrf-bootstrap md](05-csrf-bootstrap.md) — marking as read the specified notification as the user is
  a write and so requires the X-XSRF-TOKEN header.
