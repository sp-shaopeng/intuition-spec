# Sequence 31 — Admin Management: create + delete admin

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Only a permitted admin creates another admin; retain the folded temporary-password contract and forced first change.
2. Only super-admin may remove an eligible admin. Logical deletion, target-session purge and required audit must be part of the complete action.
3. Production bootstrap is separate from local fixture seeding and requires forced password change plus multi-factor authentication before privileged use.

## Authority

[Current requirements C09](../CURRENT-REQUIREMENTS.md) · [spec md](../modules/admin-management/spec.md) · [spec md](../027-aws-production-deployment/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# Sequence 31 — Admin Management: create + delete admin

**Module:** M16 Admin Management & Super-Admin · **Actors:** Admin, Super-Admin
**Endpoints:** read operation for admin admins, submit operation for admin admins,
delete operation for admin admins id

The founder's named flow: a super-admin onboards an operator admin, who clears its gate
and operates; deletion is super-admin-only.

## Preconditions
- Caller is an authenticated admin (role ADMIN). Delete additionally requires is super admin.

## Flow — create + first login


Implementation example omitted. This public edition retains the written requirements in this section.


## Flow — delete (super-admin only)


Implementation example omitted. This public edition retains the written requirements in this section.


## Postconditions
- New admin exists, is super admin is no, gate set until first change-password.
- Deleted admin soft-removed; disappears from read operation for admin admins.

## Error cases
- create: 409 EMAIL ALREADY REGISTERED; 400 weak temp Password.
- delete: 403 NOT SUPER ADMIN (checked **before** existence, so no existence leak);
  403 CANNOT REMOVE SELF; 404 USER NOT FOUND (super-admin, real missing id).
- 401 unauthenticated; 403 forced-change gate on any mutation pre-change.

## Related
- Gate clearance: [29-change-password md](29-change-password.md)
- Module: [Private supporting material excluded (publication note)](../../docs/publication-notes.md)
- Regression: Admin Bootstrap IT, Admin User Mgmt IT, Admin Authz Matrix IT.
