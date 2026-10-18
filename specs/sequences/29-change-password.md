# Sequence 29 — Change Password (authenticated)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate and re-confirm the current password. Wrong current password returns422 INVALID CREDENTIALS without destroying the valid session.
2. Apply the password-change/reset session rules, audit and forced-change completion under the current auth contracts.
3. For production admins, forced password change alone does not enable privileged actions multi-factor authentication requirements also apply.

## Authority

[Current requirements C09](../CURRENT-REQUIREMENTS.md) · [int-095-batch-auth-session-hardening-spec md](../architecture/int-095-batch-auth-session-hardening-spec.md) · [spec md](../027-aws-production-deployment/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# Sequence 29 — Change Password (authenticated)

**Module:** M1 Auth / M16 Admin · **Actors:** Any authenticated user; gated admin
**Endpoint:** submit operation for auth change-password

Distinct from the forgot/reset flow (seq 04): the user is already logged in and proves
identity with their *current* password. This is also the path a freshly-created or seeded
admin uses to clear the forced-change gate (password change required).

## Preconditions
- User is authenticated (valid auth cookie).
- Knows the current password.

## Flow


Implementation example omitted. This public edition retains the written requirements in this section.


## Postconditions
- password hash rotated; failed login attempts is 0, locked until is no value,
  password change required is no (gate lifted — admin can now operate).

## Error cases
- 400 INVALID PASSWORD — current password wrong.
- 400 validation — new password fails complexity (1 upper + 1 digit + 1 special, 8–128).
- 401 unauthenticated.
- 429 — per-IP change-password rate limit (5 / 15 min default).

## Related
- Reset (no current pw): [04-forgot-reset-password md](04-forgot-reset-password.md)
- Module: M16 admin gate; Regression: Admin Bootstrap IT (gate→change→operate).
