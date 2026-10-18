# 27 — User Contact Storage and Lookup (Plaintext)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Normalize and store user email and phone as plaintext under the explicit storage amendment; use the normalized email lookup/index.
2. Do not encrypt/keyed message authentication those two user fields or recreate the retired contact columns. Password hashing remains unchanged.
3. Storage does not authorize disclosure: public/pre-assignment views omit contact; assigned participants and admins follow their respective access/audit gates.
4. Use logical account deletion and revoke sessions; retention/anonymization timing remains separately unresolved.

## Authority

[Current requirements C13](../CURRENT-REQUIREMENTS.md) · [plaintext-email-phone md](../architecture/plaintext-email-phone.md) · [int-094-batch-soft-delete-integrity-spec md](../architecture/int-094-batch-soft-delete-integrity-spec.md)

## Contract diagram


Implementation example omitted. This public edition retains the written requirements in this section.


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 27 — Encrypted personal information Write + keyed message authentication Lookup

## Trigger

Any flow that creates a User row (register tutee, register tutor) or that
looks up a user by email / phone (login, forgot-password, reset-password,
duplicate-account check on registration).

## Participants

- **Caller** — typically Auth Service
- **Encryption Service** — Deterministic Aead (Tink) for email Encrypted /
  phone Encrypted columns; keyed message authentication-SHA-256 for email Hmac / phone Hmac lookup
  columns
- **Postgres** — User table with email encrypted, email hmac,
  phone encrypted, phone hmac
- **User Repository** — find By Email Hmac, find By Phone Hmac,
  exists By Email Hmac, exists By Phone Hmac

## Diagram — write (register)


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — lookup (login / forgot-password)


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Implementation: Encryption Service implementation reference.
- encrypt / decrypt use Tink's Deterministic Aead so the same plaintext
  always produces the same ciphertext — important for any future need to
  index/lookup the encrypted column directly. AAD is empty (an empty binary response) on
  both sides.
- compute Hmac lowercases the input before hashing (line 67) so email lookups
  are case-insensitive. Phone is already canonicalised at the frontend (+65 prefix).
- Master keyset is provided via encryption master-key-keyset (Tink JSON
  proto). keyed message authentication secret via encryption hmac-secret. Both must be supplied as
  Spring properties / env.
- keyed message authentication columns (email hmac, phone hmac) are the **only** way the backend
  exists/find By Email/find By Phone. Plaintext email / phone is never indexed
  or stored. The duplicate-account check (exists By Email Hmac) and the login
  lookup (find By Email Hmac) both go through the keyed message authentication.
- Decryption is only invoked when personal information needs to leave the backend in plaintext —
  /auth/me, admin post detail (admin-only contact reveal), email enqueue
  (recipient address).

## Failure modes

- the illegal state exception operation if Tink
  throws during register (Auth Service build User line 504-517).
- the illegal state exception operation on /me if ciphertext
  becomes unreadable (key rotation without re-encrypt, corrupted column).
- decrypt Safe (used in admin post detail) catches and returns no value instead
  of throwing — admin sees a null contact field rather than a 500.
- keyed message authentication mismatch (e g. secret rotated without backfill) would manifest as 401
  INVALID CREDENTIALS on login or false 409 on register.

## Related flows

- [01-register-tutee md](01-register-tutee.md), [02-register-tutor md](02-register-tutor.md)
- [03-login-with-mfa md](03-login-with-mfa.md)
- [04-forgot-reset-password md](04-forgot-reset-password.md)
- [05-csrf-bootstrap md](05-csrf-bootstrap.md) — /me decrypts on read
- [08-admin-approval md](08-admin-approval.md) — admin-only personal information reveal
