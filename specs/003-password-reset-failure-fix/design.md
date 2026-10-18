# Design — Password reset failure fix (003)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> How the requirements in [requirements md](requirements.md) will be built. Owned with solution-architect
> for non-trivial design; significant choices recorded as ADRs.
>
> **Root-cause-first discipline (Sev bug).** The ## Root cause section below is a **PLACEHOLDER**. Per
> REQ-001 it MUST NOT be filled with a guess. It is completed only after the live reproduction (build phase)
> proves exactly one of the candidate hypotheses. The fix described in ## Fix design is conditional on that
> proof — the candidate-by-candidate fix sketch exists so the design is ready to converge fast once the cause
> is known, NOT to pre-commit to a cause.

## Architecture

The reset flow spans two boundaries; the defect could live in either, so both are in scope until REQ-001
narrows it:

- **Backend** — Auth Controller (submit operation for auth forgot-password ~L162, submit operation for auth reset-password
  ~L169) → Auth Service forgot Password (~L496) / Auth Service reset Password (~L582). forgot Password issues a
  6-digit one-time passcode via issue Reset Token (~L541): invalidates prior tokens, stores a **bcrypt hash** of the one-time passcode in
  password reset tokens with a **15-min** expiry (uq pvt one live = at most one live token per user), and
  enqueues the one-time passcode email on the durable outbox. reset Password (~L582): per-(ip,account) rate-limit (429
  RATE LIMIT EXCEEDED) → resolve user (else 400 INVALID one-time passcode) → latest valid token (else 400 INVALID one-time passcode) →
  **Redis** one-time passcode-attempt cap (429 one-time passcode ATTEMPTS EXCEEDED) → the matches operation (else bump
  Redis counter + 400 INVALID one-time passcode) → mark token used → update Password Hash And Clear Change Required (clears the
  hash, failed login attempts, locked until, AND password change required) → audit row in-tx →
  after Commit session revocation.
- **Frontend** — Forgot Password implementation reference (request a code) and
  Reset Password implementation reference (submit email+otp+new Password). Reset Password on Submit (L41–52) POSTs to
  application programming interface ROUTES auth.reset Password; on ANY thrown error it runs an empty error handler that hides the failure (L49) and sets a single hardcoded
  root error: **"Invalid or expired code. Request a new code."** — discarding the real server status/code/body.

Reused patterns: App Exception typed errors → single exception→web request protocol-status advice; Rate Limit Service (Redis);
Outbox Service enqueue Email; bcrypt password Encoder. No new components are anticipated; the fix is expected to
be a corrective change to one of the existing call sites and/or the frontend error surfacing.

## Root cause — PROVEN (REQ-001 reproduction, 2026-06-20)

**The defect is on the frontend, not the backend.** The reset endpoint and all of its guards behave correctly:
the unknown-account ≡ wrong-one-time passcode collapse to 400 INVALID one-time passcode is intentional anti-enumeration; the per-token
one-time passcode attempt cap (429 one-time passcode ATTEMPTS EXCEEDED) and the per-(ip,account) rate-limit (429 RATE LIMIT EXCEEDED)
fire accurately; a non-compliant new password returns 400 VALIDATION ERROR. **The root cause of issue #193
is the blanket an empty error handler that hides the failure in Reset Password implementation reference**, which discards the
real server status/code/body and hardcodes **one** message — *"Invalid or expired code. Request a new code."*
— for **every** failure. A legitimate user whose new password is too weak, or who has been temporarily
rate-limited, or who exhausted their one-time passcode attempts, is told their **code** is invalid (when the code was never
the problem) and loops forever requesting new codes instead of correcting the real issue.

Captured evidence (REQ-001 implementation reference): live user interface screenshots (repro-wrong-otp-1280.png,
repro-wrong-otp-390.png), per-branch backend responses (branch-responses txt, rate-limit-responses txt),
console/network log (console log), and the full H1–H6 elimination table (REPRODUCTION md):
- **H6 — PROVEN, this is the root cause.** Weak-password 400 VALIDATION ERROR, 429 one-time passcode ATTEMPTS EXCEEDED,
  and 429 RATE LIMIT EXCEEDED are all shown to the user as "Invalid or expired code."
- **H1, H3, H4 — PROVEN as correct backend behaviour** (anti-enum collapse / attempt-cap / hash mismatch fire
  as designed); they are surfaced *opaquely* by the frontend — symptoms of H6, not separate bugs. No backend
  semantic change required for these.
- **H2 — DISPROVEN as the cause** (reproduced with a fresh token; failure was a wrong code, not expiry).
- **H5 — DISPROVEN/not regressed** (Auth Service implementation reference still calls
  update Password Hash And Clear Change Required, which clears password change required); REQ-002 re-verifies the
  no-forced-change-bounce promise end-to-end in the gated regression test, since the live email one-time passcode is
  unreadable from the test harness.

**Additional security finding (handed to REQ-003 / security-expert):** the RATE LIMIT EXCEEDED *body* is
byte-identical for existing vs non-existing accounts, but one-time passcode ATTEMPTS EXCEEDED is observable **only** for an
account that holds a live reset token (an account with no token throws INVALID one-time passcode first and never reaches
the attempt-cap branch) — a soft existence/activity oracle in the response *sequence*. The messaging contract
below keeps the bodies anti-enumerating; this oracle is noted for security sign-off (no message change leaks it
further, and the frontend shows the same "too many attempts" copy for both 429 codes).

### Candidate hypotheses to prove / disprove (REQ-001 must mark each PROVEN or DISPROVEN)

- **H1 — User-not-found masquerades as INVALID OTP.** the find by email operation
  (Auth Service implementation reference). A legitimate user who mistypes their email (or whose email was normalised
  differently than at signup) gets "Invalid or expired one-time passcode" with no hint it was the *email* that didn't match.
  *Intentional* anti-enumeration design, but a usability trap for a real user. Disprove by reproducing with the
  exact registered email.
- **H2 — 15-min one-time passcode expiry → INVALID OTP.** find Latest Valid Token returns empty once expires at has passed
  (Auth Service implementation reference), collapsing to 400 INVALID OTP. A user slow to check email / reusing an old email
  link hits this. Check expires at vs the attempt time in the captured token row.
- **H3 — one-time passcode attempt cap → 429 OTP ATTEMPTS EXCEEDED.** Redis otp:attempts:<token Id> ≥ one-time passcode MAX ATTEMPTS
  (Auth Service implementation reference). A user who fumbles the code a few times is locked to that token and must request
  a new one. Check the Redis counter value at failure time.
- **H4 — one-time passcode hash mismatch (wrong/garbled code).** password Encoder matches false (Auth Service implementation reference) —
  e g. a transcription error, an email-client mangling the code, or a stale code from a *superseded* token
  (issue Reset Token invalidates prior tokens, so an older emailed code is dead). Reproduce with the freshest
  emailed code.
- **H5 — Forced-change bounce after a SUCCESSFUL reset (the known prior regression).** Reset returns 200 and
  login with the new password succeeds, but the frontend Admin Layout/route guard intercepts a residual
  password change required is yes and bounces the user into Force Change Password — perceived as "reset failed,
  still can't get in." The current code calls update Password Hash And Clear Change Required
  (Auth Service implementation reference) which DOES clear the flag — so this should be fixed; REQ-002 **must verify it has not
  regressed** (confirm password change required is no in the database after reset and no frontend bounce).
- **H6 — frontend swallows the real error (Reset Password implementation reference).** The blanket an empty error handler that hides the failure shows "Invalid or expired
  code" for *every* failure — a 429 rate-limit, a 429 attempts-exceeded, a 400 weak-password validation error, a
  network/cross-origin request permissions/cross-site request forgery protection failure, even a 500. A legitimate user is actively misdirected and can't self-correct (e g.
  is told "invalid code" when the real problem is "password too weak" or "try again in a minute"). This is a
  defect regardless of whether it is *the* issue-193 cause, and is owned by REQ-003.

## Fix design — converges after the cause is proven

The corrected behaviour is fixed by REQ-002/003 regardless of which hypothesis lands; the *mechanism* depends
on the proven cause:

- If **H1/H2/H3/H4** (a guard firing correctly but opaquely): the backend error contract stays anti-enumeration for
  the wrong-one-time passcode/unknown-account pair (REQ-003 scenario 1 — keep 400 INVALID OTP identical), while the
  **distinct, non-enumerating** failures (429 attempts/rate-limit, 400 weak-password) are surfaced accurately to
  the user via the frontend (REQ-003 scenario 2). No backend semantic change unless the repro shows an *incorrect*
  guard (e g. a normalisation mismatch in H1 that rejects a legitimate email — that would be a real backend fix).
- If **H5** (forced-change regression): the backend fix is to ensure password change required is cleared on reset
  (already coded — verify, and add the regression test that would have caught the original) and/or the frontend guard
  not to bounce post-reset. REQ-002 evidence proves it.
- If **H6** (frontend swallow): replace the blanket an empty error handler that hides the failure in Reset Password implementation reference with a handler that reads the
  server error code and maps it to an accurate, actionable message (INVALID OTP → "invalid or expired code, request
  a new one"; OTP ATTEMPTS EXCEEDED / RATE LIMIT EXCEEDED → "too many attempts, request a new code / try again
  later"; password-policy 400 → the password-requirements message), with a safe generic fallback for unmapped
  errors. No new account-existence signal is introduced.

**Non-negotiable:** no symptom patch. If the proven cause is a backend guard rejecting a *legitimate* request
(e g. an email-normalisation mismatch), fix the guard at root — do not merely soften the frontend message over a real
backend bug.

## REQ-005 — Current one-time passcode debug logging design

Use the existing single kill switch for both reset and phone-verification producers. Spec027 requires effective one-time passcode DEBUG LOG is no in production; production must not inherit the original base TRUE default. Local/dev may retain the flag-gated D8 behavior and its recorded risk. The old runtime prod hard-skip is not silently restored by this doc: verify effective config for each deployed environment and both producer paths. Do not log payload bodies from the queue/outbox. The original D8 ADR below is historical and superseded on production configuration by spec027.

## Error handling

| Condition | Backend | What the user must see (frontend) |
|---|---|---|
| Wrong/expired one-time passcode **or** unknown account | 400 INVALID one-time passcode (identical body — no enumeration) | "Invalid or expired code. Request a new code." |
| Too many wrong OTPs for this token | 429 one-time passcode ATTEMPTS EXCEEDED | "Too many attempts. Request a new reset code." |
| Too many reset attempts (per ip+account) | 429 RATE LIMIT EXCEEDED | "Too many attempts. Please try again later." |
| New password fails policy | 400 validation | the password-requirements message (8+, upper, number, special) |
| Network / cross-site request forgery protection / unexpected 5xx | n/a | safe generic fallback (no leak), with a retry hint |

Anti-enumeration property (security-expert sign-off): the unknown-account and wrong-one-time passcode branches must remain
**byte-identical** in status, code, and body, and the timing difference must not be attacker-exploitable.

## Testing strategy

- **REQ-001** — manual/Playwright live reproduction (not a continuous integration test): drive the live reset, capture
  status/body/database/Redis, fill the hypothesis table.
- **REQ-002** — regression test in the gated layer covering happy-path reset → login with new password →
  assert NOT forced-change, plus old-password-rejected. Maps to a live-stack IT (Abstract Api IT) or the
  regression lab using the local-only issue Reset Otp For Regression fixture (which returns the plaintext one-time passcode for
  regression-band emails) so the full forgot→reset→login journey is automatable. Mirror with a service/unit test
  asserting password change required is cleared.
- **REQ-003** — tests per error branch: unknown-account vs wrong-one-time passcode return identical INVALID OTP;
  attempts-exceeded → 429; rate-limit → 429; weak password → 400; frontend component/RTL test asserting each code maps
  to its distinct message and none leaks existence.
- **REQ-004** — the REQ-002/003 tests are the regression; demonstrate RED on pre-fix code, GREEN on post-fix;
  the documented operational check ≥97.5% line+branch (and the documented operational check ≥97.5% line / ≥88% branch if frontend touched).

## Migration / rollout

No schema change anticipated (the token table and flags already exist). No feature flag. Backend change ships
via the standard rebuild→redeploy→reseed before the live-stack gate reflects it; frontend via the standard build +
test:ci. If the repro reveals a data issue (e g. stale/duplicate tokens), any schema fix is Liquibase-only and
forward-only (never edit a committed changeset). Rollback = revert the diff; no data migration to unwind.

## Decisions (ADR log)

- 2026-06-20 — **Defer root-cause naming to the build-phase reproduction.** *Context:* issue #193 is a screenshot
  with no repro and several plausible causes (H1–H6). *Decision:* the spec enumerates hypotheses and gates the
  fix behind a live reproduction (REQ-001); ## Root cause stays a placeholder until proven. *Consequences:* no
  guessed/symptom patch; design converges fast once REQ-001 lands, at the cost of one explicit reproduction step
  before code.
- 2026-06-20 — **Preserve anti-enumeration while making legitimate failures actionable.** *Context:* the
  unknown-account and wrong-one-time passcode paths intentionally collapse to one error; the frontend then collapses *everything*
  else into it too. *Decision:* keep the two enumeration-sensitive branches identical, but surface the distinct
  non-enumerating errors (429s, weak-password) accurately. *Consequences:* better user experience for real users with no new
  enumeration signal; requires security-expert sign-off on the identical-branch pair.
- 2026-06-20 — **(REQ-005, D8) FOUNDER OVERRIDE of the never-log-one-time passcode invariant — log EVERY one-time passcode (reset + WhatsApp),
  recipient + validity, in ALL profiles INCLUDING PROD.** *Status:* SUPERSEDES the prior REQ-005 ADR immediately
  below (the local-only / prod-impossible gate). *Context:* the founder (special approval 2026-06-20, directive
  D8) requires operational visibility of every in-flight one-time passcode — the password-reset one-time passcode and the WhatsApp
  phone-verification one-time passcode — including the recipient (email / phone) and the validity/expiry window, in every
  environment incl. prod. *Decision:* remove the prod hard-skip (and the Environment injection on Auth Service);
  log at one-time passcode-generation time gated ONLY by a single kill-switch flag otp debug-log enabled (base default TRUE,
  one-time passcode DEBUG LOG is no to disable). Reset one-time passcode logged in Auth Service forgot Password (recipient + one-time passcode + validity +
  reset link); WhatsApp one-time passcode logged in Phone Verification Service start Verification (full recipient phone + one-time passcode +
  validity). Update the now-inaccurate never-log-one-time passcode code comments (Email Payload implementation reference, Whats App Queue Service implementation reference,
  Phone Verification Service javadoc) to state the new behaviour + cite D8 / REQ-005 / the accepted risk.
  *Accepted security risk (founder-approved, explicit override):* a deployed (incl. prod) log holds plaintext OTPs +
  recipient personal information; anyone with log access can read in-flight OTPs and recipients. This is a deliberate trade-off the
  founder has signed off; it is NOT an oversight. *Follow-up (mandatory):* **REVISIT — turn one-time passcode DEBUG LOG is no
  before public launch.** A pre-launch checklist item owns flipping the kill-switch off (or narrowing it). *Pinned
  by:* gated-layer unit tests — reset flag-on → logged (recipient+validity+link), whatsapp flag-on → logged
  (recipient+validity), both flag-off → not logged. *Alternatives rejected:* (a) keeping the prod hard-skip —
  directly contradicts D8; (b) masking the recipient — directive requires the full recipient; (c) a separate
  dev-only endpoint — does not meet the "visible in prod logs" requirement.
- 2026-06-20 — **(REQ-005 — SUPERSEDED by the D8 ADR above) Cross the never-log-one-time passcode invariant in LOCAL only, behind
  a prod-impossible two-condition gate.** *Context:* the codebase deliberately NEVER logs OTPs (Email Payload / Phone Verification Service comments),
  but the free local Resend key can't deliver one-time passcode emails, so the reset flow is untestable locally. *Decision:* add
  a config flag email dev-log-otp (base default FALSE, TRUE only in application-local implementation reference) AND a runtime
  Environment-based hard-skip under the prod profile; log the one-time passcode+link at one-time passcode-generation time in
  Auth Service forgot Password (NOT via the Email Payload/poller path, which keeps its invariant). *Alternatives
  rejected:* (a) logging via the email poller — would violate the Email Payload-never-log invariant and leak the
  one-time passcode wherever the poller runs; (b) flag-only with no prod guard — a single misconfig would leak OTPs in prod; (c)
  a separate dev-only endpoint returning the one-time passcode — larger attack surface than a gated log. *Consequences:* the
  secret stays out of every deployed log (two independent conditions must BOTH hold, one of which is provably
  false in prod); the crossing is contained to a developer workstation; pinned by a security regression unit test
  covering flag-on/non-prod (logged), flag-on/prod (NOT logged), flag-off (not logged). *Residual:* a developer's
  own local log holds their own dev OTPs — acceptable (own-data, local-only, no real user personal information since delivery is
  sandboxed).

## 2026-09-05 amendment — production one-time passcode logging and security scope

The June D8/all-profiles logging decision and its completed evidence above are historical. Spec [027](../027-aws-production-deployment/spec.md) supersedes its production setting: one-time passcode DEBUG LOG is no is required in production; accepted local/dev flag-gated behavior remains scoped to those environments. The revisit-before-launch choice is settled OFF, not a pending option. Production verification must inspect effective configuration and confirm neither reset nor phone-verification one-time passcode/recipient payload is logged. No implementation/config change or verification is claimed by this documentation revision.

REQ-003 guarantees generic, identical initial unknown-account and wrong/expired-one-time passcode responses and accurate distinct guard messages. The attempt-cap branch's response-sequence/activity oracle recorded in evidence/REQ-003/SECURITY-REVIEW md remains the expressly accepted residual; broader timing/sequence non-enumeration is not certified by the existing evidence.
