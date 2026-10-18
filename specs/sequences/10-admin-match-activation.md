# 10 — Admin Match Activation (PENDING → ACTIVE + Commission row)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authorized admin activates an eligible PENDING match after validating the agreed first-month inputs.
2. Activation locks the agreed plan under INT-157; later profile/listing edits do not alter that obligation. The provisional multiplier/rate-clamp decisions remain explicitly recorded.
3. Initiate the retained first-session confirmation/invoice follow-up. Agency receipt of tutor commission is separate from activation and from parent lesson payment.
4. Assigned-participant contact follows its application programming interface authorization and ACTIVE/COMPLETED frontend gates.

## Authority

[Current requirements C03](../CURRENT-REQUIREMENTS.md) · [int-157-commission-first-month-locked-spec md](../architecture/int-157-commission-first-month-locked-spec.md) · [product requirements document md](../PRD.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 10 — Admin Match Activation (PENDING → ACTIVE + Commission row)

## Trigger

Admin enters the agreed monthly fee for a PENDING match and clicks Activate.

## Participants

- **Admin**
- **frontend Matching implementation reference**
- **Admin Match Controller** — submit operation for admin matches id activate
- **Admin Match Service**
- **Match Repository, Commission Repository**
- **Notification Service**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Admin Match Controller implementation reference.
- Service: Admin Match Service activate Match — Admin Match Service implementation reference.
- DEFAULT COMMISSION RATE 0.5 (Admin Match Service implementation reference).
- commission Amount Sgd is computed as 50% of the monthly fee, rounded to a whole Singapore dollar — int rounding.
- Notifications point recipients at their match dashboard, where the
  admin Contact Email (REDACTED email) is now revealed once ACTIVE
  (Match Dashboard Service ADMIN CONTACT EMAIL, line 40).

## Failure modes

- 404 MATCH NOT FOUND.
- 422 MATCH NOT PENDING — already ACTIVE, COMPLETED, or CANCELLED.
- 400 validation — monthly Fee Sgd must satisfy Activate Match Request constraints.

## Related flows

- [09-admin-match-creation md](09-admin-match-creation.md)
- [11-admin-mark-commission-paid md](11-admin-mark-commission-paid.md)
- [12-admin-match-cancel-cascade md](12-admin-match-cancel-cascade.md)
- [17-tutee-views-matches md](17-tutee-views-matches.md), [18-tutor-views-matches md](18-tutor-views-matches.md)
