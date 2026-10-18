# Sequence 30 — Account Deletion (Personal Data Protection Act right to erasure)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate and re-confirm credentials; wrong password returns422 INVALID CREDENTIALS while preserving the existing session.
2. Logically delete the account, revoke access/refresh sessions across devices and preserve the required audit/cascade behavior.
3. Do not physically remove financial/audit rows through this action. Erasure/anonymization timing remains O08; the old single-token denylist example does not prove full revocation.

## Authority

[Current requirements C13](../CURRENT-REQUIREMENTS.md) · [int-095-batch-auth-session-hardening-spec md](../architecture/int-095-batch-auth-session-hardening-spec.md) · [requirements md](../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# Sequence 30 — Account Deletion (Personal Data Protection Act right to erasure)

**Module:** M1 Auth · **Actors:** Any authenticated user
**Endpoint:** delete operation for auth account

## Preconditions
- User is authenticated; re-confirms identity with their password.

## Flow


Implementation example omitted. This public edition retains the written requirements in this section.


## Postconditions
- User row soft-deleted (deleted at), excluded from all queries by @database query language Restriction.
- Active signed session token denylisted so the current session cannot continue.
- Email/phone freed only per the unique-index policy (soft-deleted rows can still trip the
  partial unique index — see registration TOCTOU backstop).

## Error cases
- 400 / 401 — wrong password / unauthenticated.

## Notes
- Soft delete (not hard) preserves referential integrity for matches/commissions/audit.
- Personal Data Protection Act erasure of personal information beyond soft-delete (anonymisation) is handled by the scheduled
  anonymisation job — see find Users For Anonymisation and ROADMAP Tier 1 #7a.

## Related
- Anonymisation: ROADMAP #7a Personal Data Protection Act Data Export & Erasure.
