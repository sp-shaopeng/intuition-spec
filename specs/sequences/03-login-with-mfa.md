# 03 — Login (with admin multi-factor authentication time-based one-time passcode)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate credentials with the current lockout, generic-error and audit rules.
2. TUTEE/TUTOR receive the 004 30-minute access/refresh lifecycle.
3. Admin multi-factor authentication is environment-scoped: preserve the recorded local/dev deferral;027 production requires multi-factor authentication enrollment/verification and forced password change before privileged use. Do not infer a production waiver from a local flow.

## Authority

[Current requirements C09](../CURRENT-REQUIREMENTS.md) · [requirements md](../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [spec md](../027-aws-production-deployment/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 03 — Login (with admin multi-factor authentication time-based one-time passcode)

## Trigger

User submits the Login form on Login implementation reference. For TUTEE / TUTOR the flow ends
with a signed session token cookie; for ADMIN the backend returns a short-lived mfa session
cookie and the frontend must POST a 6-digit time-based one-time passcode to auth login mfa interface operation to
exchange it for the final signed session token.

## Participants

- **User**
- **frontend Login implementation reference**
- **Auth Controller** — submit operation for auth login, submit operation for auth login mfa
- **Auth Service** — the login operation, the verify mfa operation
- **Rate Limit Service** — per (ip, email Hmac)
- **Password Encoder** — B Crypt using 12
- **User Repository**
- **Redis** — multi-factor authentication session (the sha256 operation), admin session
  (session:admin:<session Id>), attempt counter (...:attempts)
- **Code Verifier** — dev samstevens.totp Default Code Verifier
- **Jwt Token Provider**

## Diagram — Happy path (non-admin)


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — Admin multi-factor authentication branch


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Auth Controller implementation reference.
- Service login: Auth Service implementation reference. Lockout after LOCK AFTER ATTEMPTS is 5
  failures for LOCK DURATION MINUTES is 15.
- Service multi-factor authentication: Auth Service implementation reference. multi-factor authentication SESSION expiry period SECONDS is 300,
  multi-factor authentication MAX ATTEMPTS is 3. Admin Redis session expiry period configurable via
  jwt admin.session-ttl-seconds (default 1800).
- Token storage: Redis key is the sha256 operation with value
  <user Id>:<bcrypt Hash>. The cookie carries the raw token; the backend never stores
  the raw token in Redis.
- Auth Controller multi-factor authentication COOKIE value (withheld), path scoped to
  auth login mfa interface operation.

## Failure modes

- 401 INVALID CREDENTIALS — user not found or password mismatch.
- 423 ACCOUNT LOCKED — locked Until now, with Retry-After seconds.
- 429 RATE LIMIT EXCEEDED — Lua window exceeded.
- 401 INVALID multi-factor authentication SESSION — cookie missing/expired or bcrypt mismatch on stored hash.
- 401 multi-factor authentication NOT CONFIGURED — admin user has no mfa Secret.
- 401 INVALID time-based one-time passcode — counter incremented, session preserved unless cap hit.
- 429 multi-factor authentication ATTEMPTS EXCEEDED — after 3 wrong time-based one-time passcode entries.
- **Gap**: Login implementation reference on Submit always reads res data.user. The multi-factor authentication
  response shape is confirmation that multi-factor authentication is required (no user), so an admin login will
  fall through the catch and display "Invalid email or password". The multi-factor authentication second
  step exists on the backend but is not yet wired into Login implementation reference.

## Related flows

- [24-rate-limited-endpoint md](24-rate-limited-endpoint.md) — login rate limit (10/h per ip+email Hmac)
- [25-csrf-protected-write md](25-csrf-protected-write.md) — login/login-mfa are cross-site request forgery protection-ignored
- [27-encrypted-pii-write-and-lookup md](27-encrypted-pii-write-and-lookup.md) — email Hmac lookup
