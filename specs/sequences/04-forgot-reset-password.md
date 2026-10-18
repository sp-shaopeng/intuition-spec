# 04 — Forgot Password + Reset

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Use 003 for reset request/one-time passcode expiry, attempt counting, error handling and the documented non-enumeration exception.
2. Complete password reset, token consumption and required audit atomically; revoke pre-reset access/refresh sessions as specified in003/004.
3. Production one-time passcode logging is OFF under 027. A queued reset email is not proof of delivery.

## Authority

[Current requirements C09](../CURRENT-REQUIREMENTS.md) · [requirements md](../003-password-reset-failure-fix/requirements.md) · [requirements md](../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [spec md](../027-aws-production-deployment/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 04 — Forgot Password + Reset

## Trigger

User clicks "Forgot password?" on Login implementation reference, lands on Forgot Password implementation reference,
enters email. Backend issues an one-time passcode via email; user enters one-time passcode + new password on
Reset Password implementation reference.

## Participants

- **User**
- **frontend Forgot Password implementation reference / Reset Password implementation reference**
- **Auth Controller** — submit operation for auth forgot-password, submit operation for auth reset-password
- **Auth Service** — the forgot password operation, the reset password operation
- **Rate Limit Service**
- **Encryption Service** — compute Hmac for email lookup
- **User Repository, Password Reset Token Repository, Audit Log Repository**
- **Password Encoder** — B Crypt using 12 for one-time passcode hash + new password hash
- **Email Queue Service** — Redis stream enqueue (template N-05)

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Auth Controller implementation reference.
- forgot Password service: Auth Service implementation reference. Constants —
  one-time passcode LENGTH is 6, one-time passcode EXPIRY MINUTES is 15, one-time passcode MAX ATTEMPTS is 5,
  forgot Password Limit is 3 per (ip, email Hmac) per hour.
- reset Password service: Auth Service implementation reference. Existing valid tokens are
  invalidated up-front (line 408) — only the latest is checked at reset time.
- Email is enqueued **after commit** (line 420) via
  the after commit operation so a rolled-back transaction never
  produces a stray one-time passcode email.
- Email template key: Email Template Keys FORGOT PASSWORD (forgot password).
- **INT-006 — reset deep-link.** forgot Password now enqueues the one-time passcode and account email (was the one-time passcode).
  Resend Email Client renders a **"Reset your password"** button linking to
  the password-reset page, with the account email supplied as a link parameter (base URL is trusted config, so it
  bypasses the the safe url operation open-redirect guard; the email is URL-encoded + HTML-escaped). The **one-time passcode is
  never in the link** — the user types it. Reset Password implementation reference prefills the email field from the
  ?email query param via use Search Params (absent param = empty field, prior behaviour).
  Source: Auth Service forgot Password, the reset password url operation,
  Reset Password implementation reference.
- Admin password reset does NOT invalidate active Redis admin sessions — a
  warning is logged (line 454). They expire naturally.

## Failure modes

- forgot Password always returns 200, even on rate-limit or unknown email — by
  design (no enumeration).
- 400 INVALID one-time passcode — wrong/expired one-time passcode, increments attempt Count.
- 429 one-time passcode ATTEMPTS EXCEEDED — attempt Count is at least 5.
- Email enqueue failure swallowed inside Email Queue Service (log only).

## Related flows

- [24-rate-limited-endpoint md](24-rate-limited-endpoint.md)
- [26-email-fanout-on-event md](26-email-fanout-on-event.md) — XADD path
- [27-encrypted-pii-write-and-lookup md](27-encrypted-pii-write-and-lookup.md) — keyed message authentication lookup
