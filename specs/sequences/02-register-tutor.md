# 02 — Tutor Registration (two-step)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Collect001 required consents and the tutor commission acknowledgement. Keep the password draft in memory, not session Storage.
2. The002 profile capture page is optional and skippable. Applying later requires the canonical profile-completeness predicate; signup does not require a published listing.
3. Persist legal/commission acknowledgements and account audit atomically. Initialize access/refresh sessions under 004.

## Authority

[Current requirements C05](../CURRENT-REQUIREMENTS.md) · [requirements md](../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [requirements md](../002-tutor-profile-and-onboarding-ux/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 02 — Tutor Registration (two-step)

## Trigger

User selects "I am a tutor" on Register implementation reference. The first form does **not** call
the application programming interface — it stashes the form values in session Storage under
intuition tutor Registration and navigates to /register/commission. The
commission page collects the binding acknowledgement and posts the full payload
to auth register tutor interface operation.

## Participants

- **User**
- **frontend Register implementation reference** — Step 1 ([REDACTED AUTHORIZATION] + Personal Data Protection Act)
- **session Storage** — short-lived stash bridging two pages
- **frontend Commission implementation reference** — Step 2 (qualifications + commission acknowledgement)
- **Auth Controller** — submit operation for auth register tutor
- **Auth Service** — the register tutor operation
- **Rate Limit Service, Encryption Service**
- **User Repository / Tutor Profile Repository / Commission Acknowledgement Repository / Audit Log Repository**
- **Analytics Event Publisher**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Stash key constant: Register implementation reference — TUTOR REGISTRATION STASH KEY.
- Stash payload shape: Register implementation reference — Tutor Registration Stash.
- Mount guard + beforeunload wipe: Commission implementation reference. Stash holds the
  plaintext password until the user submits step 2 — the beforeunload listener
  proactively wipes it on tab close/refresh.
- Commission terms version constant: Commission implementation reference — COMMISSION TERMS VERSION 2026-01-01.
- backend controller: Auth Controller implementation reference.
- backend service: Auth Service implementation reference. Ack check is the **first** validation
  (line 137) — before rate limit.
- Two audit log entries written on commit: USER REGISTERED and
  COMMISSION ACKNOWLEDGED (lines 183-195).

## Failure modes

- 400 COMMISSION ACK REQUIRED — the commission acknowledged operation is false.
- 429 RATE LIMIT EXCEEDED — same window as tutee registration (5/h/IP).
- 409 EMAIL ALREADY REGISTERED / 409 PHONE ALREADY REGISTERED.
- Stash missing on Commission mount → toast + redirect (no application programming interface call).
- 500 if encryption fails.

## Related flows

- [01-register-tutee md](01-register-tutee.md)
- [25-csrf-protected-write md](25-csrf-protected-write.md) — this endpoint is in the cross-site request forgery protection ignoring Request Matchers set
- [24-rate-limited-endpoint md](24-rate-limited-endpoint.md)
- [27-encrypted-pii-write-and-lookup md](27-encrypted-pii-write-and-lookup.md)
