# 11 — Admin Marks Commission Paid (match → COMPLETED)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. On both Matching and Commission admin surfaces, show a confirmation dialog before sending the financial mutation. Cancel produces no mutation.
2. Confirm that the agency received the tutor commission; parent lesson payment or winning an unpaid dispute is insufficient.
3. The server enforces eligible state, idempotency and authorization, records the receipt/audit atomically, and completes the match when commission becomes PAID.
4. The simplified collection flow does not remove first-session confirmation or invoicing requirements and does not define partial/advance/refund handling.

## Authority

[Current requirements C02](../CURRENT-REQUIREMENTS.md) · [requirements md](../017-admin-analytics-and-financial-safety/requirements.md) · [spec md](../019-admin-money-safety-and-analytics/spec.md) · [spec md](../modules/commission/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 11 — Admin Marks Commission Paid (match → COMPLETED)

## Trigger

Admin operates Commission implementation reference, clicks Mark Paid on a PENDING commission.

## Participants

- **Admin**
- **frontend Commission implementation reference**
- **Admin Commission Controller** — submit operation for admin commissions id pay
- **Admin Commission Service**
- **Commission Repository, Match Repository**
- **Notification Service**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Admin Commission Controller implementation reference.
- Service: Admin Commission Service implementation reference.
- Only the **tutor** is notified — the tutee is not notified on commission
  payment. The status change is observable in their dashboard.
- Match transition ACTIVE → COMPLETED is what unlocks review submission
  in Review Service submit Review (see flow 19).
- Idempotency: re-calling on a PAID commission throws 409.

## Failure modes

- 404 COMMISSION NOT FOUND.
- 409 COMMISSION ALREADY PAID.
- 403 non-admin.

## Related flows

- [10-admin-match-activation md](10-admin-match-activation.md) — where Commission was created
- [12-admin-match-cancel-cascade md](12-admin-match-cancel-cascade.md) — interaction with PENDING commission
- [19-tutee-submits-review md](19-tutee-submits-review.md) — unlocked by COMPLETED match
