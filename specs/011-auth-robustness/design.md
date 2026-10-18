# Design — Auth robustness (011)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> How the requirements are built. Owned by software-engineer (logout/refresh ordering) + security-expert
> (threat review) + frontend-expert (autofill, poll-pin). Grounded against the real code 2026-06-22.

## REQ-001 — logout vs silent-refresh race

**Current state:** api implementation reference runs single-flight the refresh session operation (one shared promise; refresh Retried
one-shot replay). auth Store implementation reference the logout operation clears client state. submit operation for auth logout revokes the refresh
family server-side. No coordination between the two today.

**Design:**
- Set a logout generation/intent marker before beginning logout; block new automatic refreshes and suppress replay
  or store repopulation from any older refresh response.
- Use **await-then-revoke**: await the existing single-flight refresh promise to settle, then call
  submit operation for auth logout. The browser may already have processed the refresh response's Set-Cookie; the subsequent
  server logout must revoke its refresh family and send expiry cookies for both auth and refresh token, with
  the original path/domain attributes. Both remain **HttpOnly**, as required by spec 004.
- JavaScript does not clear either credential. Aborting a fetch alone is not accepted as proof that its
  Set-Cookie was never processed. The earlier JS-readable-auth/abort guarantee is superseded by this correction.
- On successful server logout, clear store/query caches and navigate. Verify the old and rotated refresh
  credentials are rejected and cookies no longer authorize protected calls.
- On a failed/uncertain server logout, clear local user interface state/caches and keep automatic refresh suppressed. Show
  "We couldn't confirm sign-out from the server. Please retry." Keep a retry action. This local safety behavior
  does not claim server revocation or HttpOnly-cookie deletion while offline.

**Files for a later implementation:** frontend refresh/interceptor and logout orchestration; verify the existing server
revoke/clear contract from [004 requirements](../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md).
This document correction is not evidence that those paths are currently correct.

## REQ-002 — autofill suppression

Audit the auth forms — login, register (tutee+tutor), password-reset, forced password-change — and set per-field
autocomplete: identifier → username/email; login password → [REDACTED CREDENTIAL]; register/reset/forced-change
passwords → new-password. Ensure no on Change/effect auto-submits on a programmatic value set (autofill). Files
live under intuition-the private implementation folder** (login/register/reset) + the forced-change form.

## REQ-003 — poll gate (confirm + pin)

No code change expected. Add/confirm an RTL regression test: Notification Bell with user is no value issues 0
unread-count fetches; with a user it is enabled. Keep enabled: !user (Notification Bell implementation reference) and the
BACKGROUND POLL ROUTES exemption (auth Redirect implementation reference).

## REQ-004 — duplicate-phone SPECIFIC message + anti-enumeration hardening (FOUNDER OVERRIDE)

**Current state:** Auth Service implementation reference (tutee) + 244-245 (tutor) fold a duplicate phone into the generic
the registration phone conflict operation → 422 REGISTRATION FAILED (INT-109, constants 71-81). Email already returns a
SPECIFIC 409 EMAIL ALREADY REGISTERED (183-185). Rate-limit on register exists (121-125,176-177,233-234);
**no constant-time/latency guard exists.**

**Design (override):**

- **bff-specialist — contract PINNED (Wave-2 condition C2): 409 PHONE ALREADY REGISTERED.** The duplicate-phone
  rejection returns **web request protocol 409** with code **PHONE ALREADY REGISTERED** and the human message
  **"This phone number is already registered."** — mirroring the existing email path 409 EMAIL ALREADY REGISTERED
  (Auth Service implementation reference). 409 (resource conflict) is correct here, not 422 (validation-shape). Add the
  PHONE ALREADY REGISTERED code → friendly-copy entry to the frontend error map (api Error implementation reference) **serialised with
  016/REQ-001's edits on the same file** (overlap map §intra-Wave-2). New shared constants on Auth Service mirror
  the email pair: CODE PHONE ALREADY REGISTERED PHONE ALREADY REGISTERED, MSG PHONE ALREADY REGISTERED =
  "This phone number is already registered." (no-magic-strings).

- **software-engineer — de-fold + the timing equaliser:** replace the registration phone conflict operation fold at the two
  pre-check sites (187-189 tutee, 244-245 tutor) with the specific 409 PHONE ALREADY REGISTERED exception;
  KEEP the registration failed reason is phone conflict metric increment (79-81) on the conflict path so internal
  observability survives the de-fold. The save User Or Conflict TOCTOU backstop (880-891, idx users phone) is
  re-pointed to the SAME specific exception so the race path and the pre-check path are identical.

- **security-expert — the CONCRETE timing guard (Wave-2 condition C1, the override's whole premise):** the chosen
  mechanism is **always-run-the-same-work (a deterministic dummy bcrypt on the conflict path), NOT a wall-clock
  floor-sleep.** Grounded fast-path analysis: today both exists By Email (183) and exists By Phone (187)
  early-return **before** build User → build User Account → the encode operation (981), so the
  dominant cost of the SUCCESS path (one bcrypt hash, the deliberately-slow operation) is **absent** from BOTH
  conflict branches — that bcrypt is the measurable branch delta. The guard:
  - On the **duplicate-phone conflict path AND the duplicate-email conflict path**, perform an **equivalent dummy
    bcrypt** (the encode operation whose result is discarded) **before** throwing the conflict
    exception, so the conflict branches incur the same dominant cost as the success branch. This covers
    **phone-conflict, email-conflict, and success** — all three now execute exactly one bcrypt — closing the
    review's "three latency classes" gap (email is already a specific 409, but equalising it too removes the
    success-vs-email timing tell as well).
  - **No artificial sleep / no fixed-millisecond floor** is used (avoids the data-engineer's Do S-amplifier concern:
    a 500ms floor × the rate-limit ties up threads cheaply). The cost is the bcrypt the success path already pays,
    so the guard adds at most one hash-equivalent of work per rejected attempt and cannot be tuned into an
    amplifier. Place the equalising hash behind a single private seam, e g.
    the equalise registration cost operation (calls password Encoder encode and discards), invoked on
    every conflict-reject branch.
  - **Floor budget / target:** the equaliser's cost ≈ one bcrypt at the configured strength (the same
    Password Encoder bean the success path uses). No separate millisecond budget is defined — the budget IS
    "one bcrypt, identical to success" — which is exactly what makes it deterministically testable (below).
  - The rate-limit (ratelimit register.limit / window-seconds, 121-125,176-177,233-234) is **retained on ALL
    register attempts including the duplicate-phone rejection** and is checked FIRST (176-177 / 233-234,
    before any exists By* membership read), so an over-limit attacker is 429 RATE LIMIT EXCEEDED'd **before** any
    phone membership is revealed. security-expert signs the residual-risk box on this concrete design.

- **DETERMINISTIC test (Wave-2 condition C3 — replaces the untestable "statistically indistinguishable"):** assert
  the **equalising work is invoked on the conflict path**, via a SPY/seam — NOT a flaky wall-clock measurement:
  - Spy the Password Encoder (Mockito verify): a duplicate-phone registration → the encode operation is
    invoked **exactly once** (the dummy hash), the SAME number of times as a successful registration; a
    duplicate-email registration → also invoked once. The conflict path can no longer skip the dominant cost.
  - Assert ordering: rate-limit is consulted **before** exists By Phone (over-limit → 429 RATE LIMIT EXCEEDED
    with exists By Phone never called — Mockito the verify operation).
  - Assert the specific contract: duplicate-phone → 409 + code PHONE ALREADY REGISTERED + the message; and the
    the counter for registrations rejected because the phone is already in use counter incremented (Meter Registry assertion).
  - **No continuous integration wall-clock timing assertion** — timing-invariance is proven structurally (same code path executes the
    same bcrypt cost), which is stable in continuous integration.

- This is a backend-lane change (Auth Service implementation reference + tests) PLUS a small frontend error-map entry. **Auth Service implementation reference is a
  shipped-Wave-1-adjacent hot file but is touched only by this REQ in Wave 2** — no other Wave-2 REQ edits it; the frontend
  api Error implementation reference edit serialises with 016/REQ-001.

## Test / coverage

- backend gate the documented operational check ≥97.5/97.5 — NEW/changed tests: 409 PHONE ALREADY REGISTERED
  code/message; **deterministic timing-equalisation test** — spy Password Encoder encode and assert it is invoked
  the SAME number of times (once) on the duplicate-phone, duplicate-email, AND success paths (the conflict branch can
  no longer skip the dominant bcrypt cost), NOT a wall-clock measurement; rate-limit-ordering test (over-limit →
  429 RATE LIMIT EXCEEDED with exists By Phone never reached — the verify operation); preserved
  registration failed reason is phone conflict metric.
- frontend gate lint and build and test:ci ≥97.5 line / ≥88 branch — new tests for REQ-001 (logout/refresh ordering +
  **cookies-not-re-applied**: assert the in-flight refresh's rotated cookie does not survive logout), REQ-002
  (autocomplete + no auto-submit), REQ-003 (poll-gate pin).
- Live (BLOCKING for REQ-001): Playwright logout-vs-refresh capture asserting the **old refresh cookie is rejected
  after logout** — trigger a refresh, click logout, then a subsequent /auth/refresh with the old refresh cookie
  returns REFRESH REUSED/REFRESH INVALID (server-revocation proof; jsdom/RTL alone cannot prove it). Forced-change
  autofill capture.
