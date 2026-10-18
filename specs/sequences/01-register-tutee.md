# 01 — Tutee Registration

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Collect the three required legal/age consents and optional marketing consent under 001. Persist the actual consented policy versions and server timestamps; missing required consent creates no account.
2. Keep any registration draft containing a password in memory only. Persist account and required audit atomically; durable delivery is separate from the account transaction.
3. Issue or hydrate the user session under 004; cookie-only historical examples do not replace the refresh contract.

## Authority

[Current requirements C09](../CURRENT-REQUIREMENTS.md) · [requirements md](../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [requirements md](../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 01 — Tutee Registration

## Trigger

User selects "I need a tutor" on Register implementation reference, fills the form, accepts Personal Data Protection Act
consent, and submits.

## Participants

- **User** — site visitor on the registration form
- **frontend (Register implementation reference)** — form, validation, axios call
- **Auth Controller** — submit operation for auth register tutee
- **Auth Service** — the register tutee operation
- **Rate Limit Service** — Redis sliding-window check (Lua)
- **Encryption Service** — deterministic AEAD + keyed message authentication-SHA-256
- **User Repository / Tutee Profile Repository / Audit Log Repository** — Java persistence mapping writes
- **Analytics Event Publisher** — fires user registered after commit

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- frontend: Register implementation reference — on Submit posts to
  application programming interface ROUTES auth.register Tutee. (INT-078, 2026-06-06: the account-level
  relationship student/parent field was removed end-to-end — signup no longer collects or
  sends it; the post-level student Relationship on tutee posts was a separate field, since removed by INT-151 (migration 0053).)
- backend controller: Auth Controller implementation reference.
- backend service: Auth Service implementation reference. Rate limit
  (register Limit is 5, register Window Seconds is 3600) is enforced **before** the
  keyed message authentication lookup.
- Side effects (audit Log save, analytics Event Publisher publish) are deferred
  via the after commit operation — Auth Service implementation reference.
- signed session token issued by the issue auth cookie operation (line 199) — Http Only,
  Secure, Same Site is Strict, path /, configurable jwt expiration-seconds
  (default 86400).

## Failure modes

- 429 RATE LIMIT EXCEEDED — Lua script returns 0 ( is at least 5 attempts/hour for this IP).
- 409 EMAIL ALREADY REGISTERED / 409 PHONE ALREADY REGISTERED — keyed message authentication index hit.
- 400 from @Valid field validation (zod regex also enforces frontend-side).
- 500 if Encryption Service encrypt throws — wrapped as
  the illegal state exception operation.

## Related flows

- [02-register-tutor md](02-register-tutor.md) — two-step variant
- [05-csrf-bootstrap md](05-csrf-bootstrap.md) — cross-site request forgery protection cookie precedes this POST
- [24-rate-limited-endpoint md](24-rate-limited-endpoint.md) — Lua sliding window
- [26-email-fanout-on-event md](26-email-fanout-on-event.md) — no welcome email is queued today (gap)
- [27-encrypted-pii-write-and-lookup md](27-encrypted-pii-write-and-lookup.md) — email Hmac/phone Hmac mechanics
