# Requirements — Password reset failure fix (003)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin**
> acceptance criteria and the **exact evidence** that will prove each scenario. A compound requirement is two
> requirements — split it.
>
> **Source bug:** GitHub issue #193 "Reset account password fail" (REDACTED name /in Tuition) — a screenshot of a
> failing reset with **no reproduction steps and no stated root cause**. This is treated as a **Sev bug**:
> reproduce-before-patching is mandatory. REQ-001 reproduces and captures the *exact* failure; the root cause in
> [design md](design.md) stays a **placeholder** (a list of hypotheses to prove/disprove) until REQ-001
> produces evidence in the build phase. No symptom patching — the corrected-behaviour and messaging REQs
> (002–004) are designed against the *proven* root cause, not a guess.

---

## REQ-001 — Reproduce and capture the exact reset failure before any fix is designed

- **Type:** constraint (reproduction gate)
- **Priority:** Must
- **Source:** Issue #193; CLAUDE md "Reproduce Sev-1s before patching"; software-engineer SHARED-DEBUGGING discipline.
- **Rationale / value:** A tracker-filed bug with only a screenshot is a *hypothesis*, not a known root cause.
  Patching a guessed cause risks fixing nothing (or masking the real defect). The fix MUST be designed against a
  reproduced, evidence-backed failure. This REQ blocks REQ-002…004 design finalisation.
- **Owner (specialist):** qa-engineer (repro) → hands the proven root cause to software-engineer.
- **Acceptance criteria (Gherkin):**


- Scenario: Reproduce the live reset failure end-to-end.
- Given the live system at http://[REDACTED IP ADDRESS] with the reset flow reachable.
- When quality assurance drives forgot-password  leads to  reset-password as a legitimate user via Playwright MCP.
- Then the exact failing step is observed and recorded.
- And the real web request protocol status + error code + response body of the failing POST auth reset-password interface operation is captured.
- And the password reset tokens row state (expires at, used at, attempt count) and the Redis one-time passcode-attempts counter are captured at the moment of failure.
- And each candidate hypothesis (H1-H6 in design md) is marked PROVEN or DISPROVEN with the evidence that decided it.


- **Required evidence (defined up front):** Playwright trace + screenshots of the live reset flow at the failing
  step (@1280 and @390); the captured request/response of the failing submit operation for auth reset-password
  (status, error code, body) via Playwright network capture or curl-with-cross-site request forgery protection; a database dump of the user's
  password reset tokens row using s + the otp:attempts:<token Id> Redis key; a written hypothesis-elimination
  table (H1–H6 → PROVEN/DISPROVEN). Filed under REQ-001 implementation reference.
- **Dependencies:** none — this is the entry gate.

---

## REQ-002 — A valid one-time passcode + compliant new password completes the reset and the user logs in with the new password, NOT bounced to forced-change

- **Type:** functional
- **Priority:** Must
- **Source:** Issue #193; project memory project password reset lockout bug md (prior regression where reset
  left password change required is yes, bouncing the user into Force Change Password).
- **Rationale / value:** This is the core user-visible promise of the feature: a user who proves ownership via a
  valid one-time passcode and chooses a compliant password must end up able to sign in with that password — with no forced
  change-password bounce afterwards.
- **Owner (specialist):** software-engineer.
- **Acceptance criteria (Gherkin):**


- Scenario: Happy-path reset then login.
- Given a user has requested a reset and holds a live, unexpired, unused one-time passcode.
- When they POST auth reset-password interface operation with email, otp, new Password where new Password meets the policy.
- Then the response is 200 ok:true.
- And the user's password hash is updated and password change required is cleared.
- And the token row is marked used (used at set).
- And the user can subsequently POST auth login interface operation with the new password and receive a session.
- And the post-login client is NOT redirected to the forced change-password screen.
- Scenario: Old password no longer works after reset.
- Given a completed reset per the scenario above.
- When the user attempts login with the OLD password.
- Then login is rejected.


- **Required evidence (defined up front):** Playwright before→after of the live reset+login journey
  (@1280, @390) showing the user landing on the authenticated dashboard (not Force Change Password); the captured
  200 response of reset-password; a database before→after dump of users (password hash changed,
  password change required is no, failed login attempts is 0, locked until null) and
  password reset tokens (used at set); the captured login 200 with the new password and the login rejection
  with the old password. Filed under REQ-002 implementation reference.
- **Dependencies:** REQ-001 (root cause proven first).

---

## REQ-003 — Reset error messages are accurate and non-misleading, without leaking account existence

- **Type:** security / user experience (error surfacing)
- **Priority:** Must
- **Source:** Issue #193; candidate causes H1 (user-not-found collapses to "INVALID OTP") and the frontend
  swallow-all catch (H6, Reset Password implementation reference); personal information Visibility Model memory (no enumeration leak);
  software-engineer non-negotiable on accurate error surfacing.
- **Rationale / value:** A legitimate user must be told *something they can act on* (e g. "code expired —
  request a new one", "too many attempts — request a new code", "your new password does not meet the policy"),
  while an attacker must NOT be able to distinguish "no such account" from "wrong one-time passcode". The current frontend collapses
  every server error to one generic string (an empty error handler that hides the failure in Reset Password implementation reference), so even an actionable,
  distinct backend error (429 rate-limit, 429 attempts-exceeded, 400 weak-password) is shown as "invalid or
  expired code" — actively misdirecting a legitimate user. Whether this is the issue-193 root cause is decided
  by REQ-001; regardless, the messaging contract below is required.
- **Owner (specialist):** software-engineer (backend error contract) + frontend-expert (frontend surfacing); security-expert sign-off.
- **Acceptance criteria (Gherkin):**


- Scenario: Wrong-or-expired one-time passcode and unknown-account stay indistinguishable.
- Given a reset attempt with an one-time passcode that is wrong/expired, OR for an email with no account.
- When POST auth reset-password interface operation is called.
- Then both cases return the SAME 400 INVALID one-time passcode with the SAME generic body.
- And these initial unknown-account and wrong/expired-code responses have identical status, code and generic body.
- And comparative timing is checked without claiming the entire response sequence is enumeration-free.
- Scenario: Distinct, actionable failures are surfaced distinctly to the user.
- Given the one-time passcode attempt cap is exceeded (429 one-time passcode ATTEMPTS EXCEEDED) OR the per-account reset rate-limit is hit (429 RATE LIMIT EXCEEDED) OR the new password fails policy (400 validation).
- When POST auth reset-password interface operation is called.
- Then the backend returns the corresponding distinct error code (NOT collapsed to INVALID one-time passcode).
- And the frontend surfaces an accurate, actionable message for that case (request a new code / try again later / password requirements) instead of one blanket "invalid or expired code".


- **Required evidence (defined up front):** captured responses for each branch (unknown-account vs wrong-one-time passcode →
  identical 400 INVALID OTP body; 429 OTP ATTEMPTS EXCEEDED; 429 RATE LIMIT EXCEEDED; 400 weak-password) showing
  status + code + body; Playwright screenshots of the frontend message shown for each branch (proving they are
  distinct and actionable, with no explicit existence disclosure); a security-expert note confirming the identical initial-pair contract and recording the accepted response-sequence/activity oracle, not certifying strict non-enumeration. Filed under REQ-003 implementation reference.
- **Dependencies:** REQ-001.

---

## REQ-004 — A regression test reproduces the defect (FAILS before the fix, PASSES after) in the gated test layer

- **Type:** constraint (test / quality gate)
- **Priority:** Must
- **Source:** CLAUDE md regression-test discipline; software-engineer SHARED-test-driven development ("a fix shipped without a
  failing-then-passing test isn't shipped"); memory note that the existing lab had NO reset-flow coverage.
- **Rationale / value:** The proven root cause must be pinned by an automated test in the gated layer
  (live-stack IT / regression lab, or service-level test as appropriate to where the bug lives) so it can never
  silently regress again — closing the gap that let issue #193 ship. The test must demonstrably FAIL on the
  current (pre-fix) code and PASS after the fix.
- **Owner (specialist):** qa-engineer (authors the test) + software-engineer (fix makes it green).
- **Acceptance criteria (Gherkin):**


- Scenario: Regression test is red before, green after.
- Given a regression test that exercises the proven failing path from REQ-001 (and the happy path of REQ-002).
- When the test is run against the pre-fix code.
- Then it FAILS, reproducing the issue-193 symptom.
- When the test is run against the post-fix code.
- Then it PASSES.
- Scenario: Coverage gate holds.
- Given the full backend gate the documented operational check.
- When it is run after the fix.
- Then it passes with  is at least 97.5% line AND  is at least 97.5% branch (JaCoCo).


- **Required evidence (defined up front):** the test source (path + name); a RED run log against pre-fix code
  showing the assertion that fails; a GREEN run log against post-fix code; the documented operational check summary showing the
  coverage thresholds met (and the documented operational check if frontend error-surfacing per REQ-003 is touched,  is at least 97.5% line /
   is at least 88% branch). Filed under REQ-004 implementation reference.
- **Dependencies:** REQ-001 (defines the path to pin), REQ-002 & REQ-003 (define corrected behaviour).

---

## REQ-005 — one-time passcode debug logging: production OFF, historical local/dev D8 retained

- **Type:** operational; **Priority:** Should; **Source:** June D8 as explicitly superseded for production by [spec027](../027-aws-production-deployment/spec.md).
- Local/dev one-time passcode generation may use the existing flag-gated debug facility with the documented accepted risk. Production must set effective one-time passcode DEBUG LOG is no; it must not log reset or phone-verification OTPs, full recipients or reset links through this facility. The earlier “all profiles including prod/default TRUE” decision is historical, not production launch authority.
- The shared kill switch controls both producers; retain non-sensitive outcome/latency observability and never log email/outbox payload bodies. No new runtime override is authorized here.
- **Evidence required:** inspect effective production configuration; exercise both producers with the flag OFF and verify no one-time passcode/debug payload line, plus local flag behavior where applicable. Existing D8 evidence remains historical and does not prove current production settings.

## 2026-09-05 amendment — production one-time passcode logging and security scope

The June D8/all-profiles logging decision and its completed evidence above are historical. Spec [027](../027-aws-production-deployment/spec.md) supersedes its production setting: one-time passcode DEBUG LOG is no is required in production; accepted local/dev flag-gated behavior remains scoped to those environments. The revisit-before-launch choice is settled OFF, not a pending option. Production verification must inspect effective configuration and confirm neither reset nor phone-verification one-time passcode/recipient payload is logged. No implementation/config change or verification is claimed by this documentation revision.

REQ-003 guarantees generic, identical initial unknown-account and wrong/expired-one-time passcode responses and accurate distinct guard messages. The attempt-cap branch's response-sequence/activity oracle recorded in evidence/REQ-003/SECURITY-REVIEW md remains the expressly accepted residual; broader timing/sequence non-enumeration is not certified by the existing evidence.
