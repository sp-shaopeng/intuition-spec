# Requirements — Auth robustness (011)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> EARS + Gherkin per atomic, independently-testable requirement. Grounded against the real code on 2026-06-22.
> Source: specs/issues/review-2026-06-22/CONSOLIDATED md (T-02, autofill, S4 poll storm, #209.2).

## Scope note (what already exists / what was verified)

- **Spec 004** added single-flight silent refresh: api implementation reference — the first 401 on a
  protected route POSTs /auth/refresh once (the refresh session operation), stores the in-flight promise, replays the
  original request (refresh Retried one-shot). Backend logout revokes the refresh family (Spec 004); the gap is
  purely the frontend ordering between a "Log Out" click and an in-flight the refresh session operation.
- **The logged-out poll storm is ALREADY FIXED** (INT-166): Notification Bell implementation reference
  enabled: !user already gates the unread-count query on an authenticated user, and auth Redirect implementation reference
  exempts the poll route from the logout-trigger. **REQ-003 is therefore a confirm-and-pin (regression guard), not a
  net-new fix** — see the review caveat below.
- **The duplicate-phone "generic message" WAS INTENTIONAL** (INT-109): Auth Service implementation reference folds
  EVERY duplicate-phone path into one neutral 422 REGISTRATION FAILED with a generic message, on purpose, to deny
  a phone-membership enumeration oracle (SG mobile space is 8 digits = enumerable) and on Personal Data Protection Act grounds. The
  reviewer's #209.2 ("make it a specific 'phone already in use'") conflicts with this. **FOUNDER OVERRIDE (2026-06-22):
  MAKE IT SPECIFIC** — REQ-004 is rewritten below to surface a specific message WITH compensating security-expert
  hardening (rate-limit retained + constant-time/consistent-latency on the register path) so the specific message
  does not become a fast enumeration oracle. The founder accepts the residual disclosure trade-off (see REQ-004).
- **Asymmetry note (verified):** duplicate EMAIL already returns a SPECIFIC 409 EMAIL ALREADY REGISTERED
  (Auth Service implementation reference); only PHONE is folded into the generic 422. The override aligns phone with the
  email treatment, with the added timing guard.
- **Rate-limiting on the register path already exists** (Auth Service implementation reference ratelimit register.limit /
  window-seconds; enforced 176-177 tutee, 233-234 tutor, keyed by client IP via register Key). The
  override KEEPS this. **No constant-time/consistent-latency guard exists yet** on the register path — that is
  net-new work REQ-004 introduces so the specific message is not a fast oracle.

---

## REQ-001 — Logout reliably revokes and halts the silent-refresh loop before navigation

- **Type:** functional / reliability
- **Priority:** Must
- **Severity:** S3* (single-user impact: "logout may not stick"; the cross-user "hijack" was a shared-browser artifact)
- **Source:** T-02.
- **Rationale / value:** A user who clicks "Log Out" must end the session deterministically. Today a the refresh session operation
  promise in flight (api implementation reference) can resolve *after* logout fires, re-minting the auth+refresh cookies and
  resurrecting a session the user believes is dead — a trust and (on shared devices) safety problem.
- **Behaviour (EARS):** WHEN the user invokes logout, the system SHALL prevent new silent refresh and
  suppress refresh-result replay, then await any in-flight refresh to settle before calling submit operation for auth logout.
  The server SHALL revoke the refresh family and expire BOTH auth and refresh cookies using matching cookie
  path/domain attributes. Both credentials remain HttpOnly; the frontend SHALL NOT attempt to read or expire them
  with JavaScript. After successful server logout, clear client auth state/caches and navigate to the logged-out
  view. The late refresh cannot survive because the server clear is processed last.
  IF logout fails, clear local auth state/caches, suppress automatic refresh/re-authentication, show that server
  sign-out could not be confirmed, and offer retry. A network failure does not prove cookie expiry or server
  revocation; do not claim protected calls will necessarily 401 until logout succeeds.

- **Owner (specialist):** software-engineer (logout/refresh-loop ordering in api implementation reference + auth Store) ·
  security-expert (session-resurrection threat review)
- **Acceptance criteria (Gherkin):**


- Scenario: Logout halts an in-flight silent refresh.
- Given a silent /auth/refresh is in flight (a 401 just triggered refresh Session).
- When the user clicks Log Out.
- Then no new silent refresh starts.
- And the rotated cookies from the in-flight refresh do NOT re-establish the session.
- And after logout completes the user is on a logged-out view and protected calls 401.
- Scenario: Logout awaits backend revocation before navigating.
- Given the user clicks Log Out.
- When logout runs.
- Then POST /auth/logout completes (refresh family revoked) before client navigation.
- And a subsequent /auth/refresh with the old refresh cookie is rejected (REFRESH REUSED/INVALID).
- Scenario: Logout fails closed on a network error (guard).
- Given POST /auth/logout fails (network error).
- When logout runs.
- Then local auth state is still cleared and the user is treated as logged out.
- And automatic refresh/re-authentication remains suppressed.
- And the user sees that server sign-out could not be confirmed and can retry.
- And no claim is made that HttpOnly cookies or server credentials were cleared.


- **Required evidence:** RTL/unit test of the logout-vs-refresh ordering (in-flight refresh settles before server clear; cookies do not survive successful logout); a live Playwright capture: trigger a refresh, click logout, confirm session does not resurrect +
  old refresh cookie is rejected. Under REQ-001 implementation reference.
- **Dependencies:** Spec 004 (refresh family + revocation).

## REQ-002 — Auth forms resist stale browser autofill

- **Type:** functional / hygiene
- **Priority:** Should
- **Severity:** S3/S4
- **Source:** autofill finding.
- **Rationale / value:** Saved browser credentials overrode typed values, and in one case a forced
  password-change form auto-submitted from autofill — surprising and risky behaviour on security-sensitive forms.
- **Behaviour (EARS):** WHEN an auth form renders (login, register, password-reset, forced password-change), the
  system SHALL set the correct autocomplete semantics per field so the browser does not silently override a
  user-typed value or auto-submit: autocomplete is username/email on identifier fields, current-password
  on login, new-password on register / reset / forced-change password fields, and SHALL NOT auto-submit a form
  purely from autofilled values (a user action is required to submit). The system SHALL NOT use autocomplete is off
  where it breaks legitimate password-manager save (use the specific token instead).
- **Owner (specialist):** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Forced password-change form is not auto-submitted by autofill.
- Given the forced password-change form.
- When the browser autofills a saved password.
- Then the form is NOT auto-submitted.
- And submission requires an explicit user action.
- Scenario: New-password fields advertise new-password, not current-password.
- Given the register or reset-password form.
- Then each new-password input has autocomplete is new-password.
- And the login password input has autocomplete is current-password.


- **Required evidence:** component test asserting the autocomplete attribute per field + no auto-submit on
  programmatic value-set; a live capture on the forced-change form. Under REQ-002 implementation reference.
- **Dependencies:** none.

## REQ-003 — The notification unread-count poll is gated on an authenticated session (confirm + pin)

- **Type:** non-functional / regression guard
- **Priority:** Should
- **Severity:** S4
- **Source:** sequential re-run S4 (logged-out bell polled ~190× 401).
- **Rationale / value:** **Already implemented** (INT-166: Notification Bell implementation reference). The reviewer
  hit the storm because of the shared-browser stale render. This requirement is a **confirm-and-pin** — add/keep a
  regression test so the gate cannot silently regress — NOT a re-implementation.
- **Behaviour (EARS):** WHILE there is no authenticated user, the system SHALL NOT poll /notifications/unread-count
  (the query is enabled: !user), and SHALL NOT trigger the global logout/redirect handler from that route's 401
  (route stays in auth Redirect implementation reference BACKGROUND POLL ROUTES).
- **Owner (specialist):** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: No unread-count poll while logged out.
- Given no authenticated user.
- When the Notification Bell mounts (or remains after logout).
- Then it issues zero requests to /notifications/unread-count.
- Scenario: Poll resumes once authenticated.
- Given a logged-in user.
- Then the unread-count query is enabled and polls on its interval.


- **Required evidence:** RTL test (bell with no user → 0 fetches; with user → enabled); confirm enabled: !user
  still present. Under REQ-003 implementation reference.
- **Dependencies:** none.

## REQ-004 — Duplicate-phone registration returns a SPECIFIC message, with anti-enumeration hardening (FOUNDER OVERRIDE of INT-109)

- **Type:** functional / contract / security
- **Priority:** Must (founder override accepted 2026-06-22)
- **Severity:** S3
- **Source:** #209.2. **Founder decision: MAKE IT SPECIFIC** (overrides the INT-109 generic-fold default).
- **Rationale / value:** Tutors/tutees who re-register with an already-used phone got an opaque "couldn't complete
  registration" and could not self-correct. The founder chose a specific, actionable "this phone number is already
  registered" message (matching how email already behaves — EMAIL ALREADY REGISTERED, Auth Service implementation reference).
  Because a specific message re-opens the phone-membership oracle INT-109 closed (8-digit SG mobile space is
  enumerable), the founder accepts that residual disclosure risk ON CONDITION that security-expert keeps the
  signup path hard to abuse as a *fast, scalable* oracle: the existing per-IP **rate-limit is retained**, and the
  register path is made **constant-time / consistent-latency** so the duplicate-phone branch is not a measurable
  timing/early-return side channel.
- **Behaviour (EARS):**
  - WHEN a registration (tutee or tutor) is submitted with a phone that already belongs to an account, the system
    SHALL return a SPECIFIC, attributing error — **web request protocol 409 with code PHONE ALREADY REGISTERED** and the human
    message "This phone number is already registered." — so the user can correct it. This mirrors the existing email
    path 409 EMAIL ALREADY REGISTERED (Auth Service implementation reference); 409 (resource conflict), not 422 (PINNED by
    bff-specialist, Wave-2 condition C2).
  - The system SHALL retain the per-IP register rate-limit (ratelimit register.limit / window-seconds,
    Auth Service implementation reference) on every registration attempt, including duplicate-phone rejections,
    AND the rate-limit gate SHALL be consulted **before** any phone-membership read (exists By Phone) so an
    over-limit attempt is 429 RATE LIMIT EXCEEDED'd before membership is revealed.
  - The register path SHALL execute the **same dominant work cost on the conflict branches as on the success branch**
    so the duplicate-phone branch is NOT a fast early-return that leaks membership via response time. CONCRETE
    mechanism (PINNED, Wave-2 condition C1): on the duplicate-phone AND duplicate-email conflict paths the system
    SHALL perform an equivalent **dummy bcrypt** (the same password Encoder encode the success path runs at
    981, result discarded) before throwing — so phone-conflict, email-conflict, and success each incur exactly
    one bcrypt. NO artificial sleep / fixed-millisecond floor is used (avoids a Do S-amplifier). This is asserted by a
    DETERMINISTIC spy test (the equalising hash is invoked the same number of times on each path), NOT a wall-clock
    timing measurement.
  - The system SHALL keep INT-109's observability (the registration failed metric with reason is phone conflict,
    Auth Service implementation reference) so the now-specific path stays measurable internally.
  - **Accepted residual risk (founder, recorded):** the specific message does disclose phone membership to an
    attacker willing to register-probe under the rate-limit; the timing guard + rate-limit reduce but do not
    eliminate this. Personal Data Protection Act: confirming a phone belongs to a registered user is a personal-data disclosure the founder
    has accepted for the registration user experience benefit.
- **Owner (specialist):** security-expert (timing-equalisation + residual-risk sign-off) · bff-specialist (the
  specific error contract — code/status/message, symmetry with the email path) · software-engineer (implement the
  branch + constant-time path)
- **Acceptance criteria (Gherkin):**


- Scenario: Duplicate-phone returns a specific, actionable message.
- Given a phone already registered.
- When a new registration uses that phone.
- Then the response is web request protocol 409 with code PHONE ALREADY REGISTERED.
- And the message tells the user the phone number is already registered.
- Scenario: Rate-limit still guards the register path (anti-enumeration, ordering).
- Given more than the configured register limit of attempts from one IP within the window.
- When the next registration is attempted.
- Then it is rejected with 429 RATE LIMIT EXCEEDED before any phone-membership is revealed.
- And exists By Phone is never reached on the over-limit attempt.
- Scenario: Duplicate-phone is not a timing oracle (deterministic — equalising work runs).
- Given a duplicate-phone registration and a successful registration and a duplicate-email registration.
- When each is processed.
- Then the password-hash (bcrypt) is invoked the SAME number of times on each path (the conflict branch does not skip the dominant cost).
- And this is asserted via a spy on Password Encoder encode, not a wall-clock latency measurement.
- Scenario: Internal observability preserved.
- Given a duplicate-phone rejection.
- Then the registration failed metric increments with reason is phone conflict.


- **Required evidence:** backend test asserting 409 PHONE ALREADY REGISTERED + message; a rate-limit-ordering test
  (over-limit → 429 before exists By Phone); the DETERMINISTIC timing-equalisation test (spy Password Encoder encode
  → invoked the same number of times on phone-conflict, email-conflict, and success — no fast early-return), reviewed
  by security-expert; preserved the counter for registrations rejected because the phone is already in use metric; the founder accepted-risk
  decision recorded. Under REQ-004 implementation reference.
- **Dependencies:** INT-109 (the control being overridden); Spec 001 (consent stamping on register, same path).

---

## Traceability summary

| REQ | One-line | Type | Priority | Severity | Owner |
|---|---|---|---|---|---|
| REQ-001 | Logout cancels in-flight silent refresh + awaits revocation before nav; refresh cannot resurrect session | functional | Must | S3* | software-engineer · security-expert |
| REQ-002 | Auth-form autofill suppression (correct autocomplete, no auto-submit) | functional | Should | S3/S4 | frontend-expert |
| REQ-003 | Unread-count poll gated on auth — confirm+pin (already fixed, INT-166) | non-functional | Should | S4 | frontend-expert |
| REQ-004 | Duplicate-phone returns a SPECIFIC message + retained rate-limit + constant-time guard (FOUNDER OVERRIDE of INT-109) | functional/security | Must | S3 | security-expert · bff-specialist · software-engineer |

## Open questions for the Boss / founder (flagged, not guessed)

1. **REQ-004 — duplicate-phone error: RESOLVED + DESIGN PINNED (founder override 2026-06-22 — MAKE IT SPECIFIC; Wave-2
   spec-review conditions C1/C2/C3 folded 2026-06-22).** The founder accepts the phone-enumeration + Personal Data Protection Act-disclosure
   residual risk in exchange for the registration user experience. Contract PINNED: **409 PHONE ALREADY REGISTERED** (mirrors
   EMAIL ALREADY REGISTERED). Timing guard PINNED: **dummy-bcrypt on the conflict paths** (phone + email) so all
   three branches incur one bcrypt — no sleep/floor — asserted by a deterministic Password Encoder spy, not a
   wall-clock measurement. Rate-limit retained and checked before any membership read. No open design choice remains.
2. **REQ-001 ordering: corrected 2026-09-05.** Await the in-flight refresh, then server revoke/expiry of both
   HttpOnly credentials. A generation guard prevents replay but does not delete cookies; abort alone is not
   evidence of cookie neutralization. Network failure remains an explicitly uncertain server-sign-out result.
