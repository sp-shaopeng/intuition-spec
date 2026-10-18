# Module 3 — Admin Post Approval: Sequence Diagrams

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Current contact fields follow the [plaintext amendment](../../architecture/plaintext-email-phone.md). Replace direct async email origination with in-tx durable intent and later poller/consumer dispatch per [INT-087](../../architecture/int-087-transactional-messaging-hardening-spec.md); keep ordinary in-app rows in the business transaction. Approval success is separate from email delivery. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


All diagrams use Plant UML syntax and can be rendered at https://plantuml com/plantuml.

---

## Diagram 1 — Tutor Submits Post → Admin Approves

This covers the full path from post submission through to the owner receiving an approval email.


The lifecycle diagram describes these state changes:

- ACTRL leads to SVC.
- SVC leads to ACTRL.
- SVC leads to SVC.
- SVC leads to AUDIT.


---

## Diagram 2 — Admin Rejects Post

Same flow as Diagram 1 up to the point of the admin decision. Only the reject path is shown in detail.


The lifecycle diagram describes these state changes:

- CTRL leads to SVC.
- SVC leads to AUDIT.
- SVC leads to CTRL.


---

## Diagram 3 — Admin Loads the Review Queue


The lifecycle diagram describes these state changes:

- CTRL leads to SVC.
- SVC leads to SVC.
- SVC leads to CTRL after a list of admin queue item entries.


---

## Diagram 4 — Admin Loads Post Detail (personal information Decryption)


The lifecycle diagram describes these state changes:

- CTRL leads to SVC.
- SVC leads to ENC.
- ENC leads to SVC.
- SVC leads to CTRL.


---

## Diagram 5 — Email Notification Flow (Redis Streams)


The lifecycle diagram describes these state changes:

- SES leads to OWNER after Email delivered.
