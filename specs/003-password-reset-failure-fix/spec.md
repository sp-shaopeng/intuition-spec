> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 003
slug: password-reset-failure-fix
title: Password reset failure fix
version: 1.6
status: HISTORICALLY MARKED IMPLEMENTED        # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: software-engineer         # the engineer who owns this change (requirements-engineer for cross-cutting/product specs)
created: 2026-06-20
updated: 2026-09-05
---

> **2026-09-05 requirements correction (documentation only).** The security guarantee reflects the accepted response-sequence residual. Spec 027 supersedes the historical D8 production logging setting: production one-time passcode debug logging is OFF.
> The status and shipment/evidence entries below record the earlier implementation. This revision does not
> certify that the corrected contract is implemented, tested, or deployed; follow-up verification remains open.


# Password reset failure fix

Fixes GitHub issue #193 "Reset account password fail": users report that resetting their account password
fails. The issue arrived as a screenshot only — no repro steps, no root cause — so this spec treats it as a
**Sev bug under reproduce-before-patching discipline**: the live failure is reproduced and captured first
(REQ-001), the root cause is named from evidence (not guessed), and the fix delivers a working reset (a valid
one-time passcode + compliant new password resets the password, the user can log in with the new password and is NOT bounced
to the forced change-password screen — REQ-002), with accurate, non-account-leaking error messaging (REQ-003)
and a regression test that fails before and passes after the fix (REQ-004). Value: restores a core,
trust-critical account-recovery flow.

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC (each REQ-NNN)
- **Design:** [design md](design.md) — architecture, data model, error handling, testing, migration
- **Tasks:** [tasks md](tasks.md) — implementation tasks, each traced to a REQ-NNN + owning specialist
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario

## Status

HISTORICALLY MARKED IMPLEMENTED — built, gated, and live-verified on the dev virtual machine (2026-06-20). See Changelog 1.5.

## Review & Sign-Off

- [x] Counterpart — backend / implementation (software-engineer) — SIGN-OFF / 2026-06-20
- [x] Counterpart — quality assurance reproduction & regression test (qa-engineer) — SIGN-OFF / 2026-06-20
- [x] Counterpart — auth / anti-enumeration (security-expert) — SIGN-OFF / 2026-06-20
- [x] Counterpart — frontend error surfacing (frontend-expert) — SIGN-OFF / 2026-06-20
- [x] Technical Writer (docs) — SIGN-OFF / 2026-06-20
- [x] Boss (Rex) — APPROVED / 2026-06-20

**Build-time conditions (non-blocking, fold into build):** (1) add a "could-not-reproduce after N attempts → escalate / capture working trace as disproof" branch to REQ-001; (2) Task 6 security review must assert the 429 RATE LIMIT EXCEEDED response is identical for existing vs non-existing accounts (no enumeration oracle); (3) make the frontend RTL test + the documented operational check gate UNCONDITIONAL (H6 blanket-catch fix is certain) and pin the server validation-error body shape so the weak-password 400 branch doesn't fall through to the generic message.

## Changelog

- 1.5 (2026-06-20) HISTORICALLY MARKED IMPLEMENTED to dev virtual machine — backend 1b194af / frontend 08be1ae / spec 63b04f3; full mvn verify + frontend test:ci green;
  e2e green; deployed + live-verified on [REDACTED IP ADDRESS] (one-time passcode-DEBUG-LOG live, distinct reset errors, reset completes).
- 1.4 (2026-06-20) MODIFIED REQ-005 — **FOUNDER DIRECTIVE D8 (special approval) OVERRIDES the local-dev-only
  posture: one-time passcode DEBUG LOGGING for EVERY one-time passcode, ALL profiles INCLUDING PROD.** The founder explicitly overrode the
  codebase's never-log-one-time passcode invariant: log both the password-reset one-time passcode AND the WhatsApp phone-verification one-time passcode at
  generation time, including the **recipient** (email / phone, full — not masked) and the **validity/expiry**
  window, in all environments incl. prod. Replaced the two-condition (email dev-log-otp flag + prod hard-skip)
  gate with a SINGLE kill-switch flag the historical development setting for diagnostic code logging (base default TRUE); removed
  the Environment/prod hard-skip from Auth Service. Added the WhatsApp one-time passcode log in
  Phone Verification Service start Verification; greppable marker one-time passcode-DEBUG-LOG reset / whatsapp. Updated
  the now-inaccurate never-log-one-time passcode code comments (Email Payload implementation reference, Whats App Queue Service implementation reference,
  Phone Verification Service javadoc) to the new founder-approved behaviour + D8/REQ-005/accepted-risk citation.
  Recorded the override as an ADR (accepted security risk: deployed logs hold plaintext OTPs + recipient personal information;
  **revisit-before-public-launch** kill-switch-off follow-up). Updated requirements REQ-005 (EARS + Gherkin: (a)
  flag-on reset logged w/ recipient+validity+link, (b) flag-on whatsapp logged w/ recipient+validity, (c) flag-off
  no log — the kill-switch), design REQ-005 section + ADR, tasks 9–11, and 4 regression unit tests (reset + whatsapp
  × on/off). Status stays IN-PROGRESS.
- 1.3 (2026-06-20) ADDED REQ-005 — **Local-dev-only reset-one-time passcode observability (prod-forbidden)** (founder
  directive). Surfaces the plaintext reset one-time passcode + reset link in the LOCAL backend log so the forgot→reset flow is
  testable on a workstation whose free Resend key can't deliver one-time passcode emails. Two-condition gate makes it
  impossible in prod: config flag email dev-log-otp (base default FALSE, TRUE only in
  application-local implementation reference) AND a runtime Environment hard-skip under the prod profile; logged at
  one-time passcode-generation time in Auth Service forgot Password (NOT via the Email Payload/poller path — keeps the
  never-log-the-payload invariant). Added requirements REQ-005 (EARS + Gherkin incl. the NEGATIVE prod-profile
  scenario + off-by-default), design gating section + ADR (crossing the never-log-one-time passcode invariant in local only),
  tasks 9–11, and a 3-case security regression unit test. Status stays IN-PROGRESS.
- 1.2 (2026-06-20) MODIFIED design md — **root cause PROVEN from REQ-001 live reproduction**: the defect is
  the frontend blanket an empty error handler that hides the failure (Reset Password implementation reference) collapsing every server error (weak-password 400
  VALIDATION ERROR, 429 OTP ATTEMPTS EXCEEDED, 429 RATE LIMIT EXCEEDED) to one "Invalid or expired code"
  message; backend guards verified correct (H1/H3/H4 fire as designed, H2/H5 disproven as the cause). Added a
  security note (OTP ATTEMPTS EXCEEDED is a soft existence/activity oracle in the response sequence). Status →
  IN-PROGRESS.
- 1.0 (2026-06-20) initial draft — software-engineer + qa-engineer. REQ-001..004 (reproduce-first gate,
  corrected reset behaviour, accurate non-leaky messaging, failing-then-passing regression test);
  root cause is a placeholder pending the build-phase live reproduction.
