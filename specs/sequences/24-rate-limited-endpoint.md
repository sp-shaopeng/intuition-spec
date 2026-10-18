# 24 — Rate-Limited Endpoint (Login example)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Use atomic rate-limit decisions and Retry-After with the endpoint’s documented scope. A composite IP+email bucket must not be presented as proof of a retained per-IP ceiling.
2. Retain the auth module’s per-IP requirement unless an explicit scoped replacement is approved; additional composite/account buckets may coexist. Existing implementation still requires verification against that contract.
3. Derive client IP only through the trusted proxy boundary in027. Reject or ignore spoofable forwarding headers from untrusted peers. Refresh429 does not clear cookies.

## Authority

[Current requirements C09](../CURRENT-REQUIREMENTS.md) · [spec md](../modules/auth/spec.md) · [spec md](../027-aws-production-deployment/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 24 — Rate-Limited Endpoint (Login example)

## Trigger

Any request to a rate-limited endpoint. Login is the worked example because it
is the strictest (10 / hour per IP+email Hmac). Registration and forgot-password
follow the same shape with different limits/keys.

## Participants

- **Caller** (frontend)
- **Security Filter Chain** — request correlation filter → Jwt Authentication Filter
  → controller
- **Auth Controller** — submit operation for auth login
- **Auth Service**
- **Rate Limit Service** — atomic Lua script on text Redis Template
- **Redis** — sorted set ratelimit:login:<ip>:<email Hmac>

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Implementation: Rate Limit Service implementation reference. The Lua script is held in a
  Default Redis Script< so it is sent once and EVALSHA-cached by Redis on
  subsequent calls.
- Keys: the login key operation, the register key operation,
  the forgot password key operation, the change password key operation.
- Defaults (configurable via Spring properties):
  - login: 10 / 3600 s
  - register: 5 / 3600 s
  - forgot-password: 3 / 3600 s
- Sliding-window algorithm: ZREMRANGEBYSCORE prunes entries older than now Ms -
  window Seconds*1000, ZCARD counts what's left, ZADD inserts the current
  timestamp on success. The Lua script is executed atomically — no TOCTOU gap.
- The client Ip derivation in the client ip operation prefers
  CF-Connecting-IP (Cloudflare), falls back to X-Forwarded-For first hop,
  then the get remote addr operation (lines 239-246).

## Failure modes

- 429 RATE LIMIT EXCEEDED — Lua returned 0.
- forgot-password is the exception: a rate-limit hit returns silent 200, not 429
  (to avoid leaking limiter state — see flow 04).

## Related flows

- [03-login-with-mfa md](03-login-with-mfa.md) — login uses this gate
- [01-register-tutee md](01-register-tutee.md), [02-register-tutor md](02-register-tutor.md)
- [04-forgot-reset-password md](04-forgot-reset-password.md)
