# Regression Fixtures Spec — Password-Reset Journey (closes the PDO-2 coverage gap)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The folded requirement at the original header for a real forced-password-change mutation takes precedence over the old /me-only fixture check. Tests must demonstrate the actual password-change behavior; the historical 99/99 local fixture record is not evidence for later changes. No new runtime test result is claimed.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** ✅ IMPLEMENTED + verified (DONE-DEV) — team-reviewed (full debate) + Boss-cleared, built, full regression **99/99 green** on dev (2026-05-31). All four review amendments applied (canonical band constant in main; service-layer band guard; POST Security Config matcher; pull request-1 asserts a post-reset admin mutation). No new PROD-DEPLOY-OWED (fixture is the profile operation).
**Date:** 2026-05-31
**Owner:** Regression Engineer (Robin)
**Authority:** subordinate to domain/dev-workflow md (lab-first gate, A1–A4) and
architecture/regression-labs-spec md. Adds one local-only test fixture + one backend journey IT.

---

## 1. Why

The founder hit a Sev-2: an admin with password change required yes could forgot-password →
reset → log in with the new password (web request protocol 200) yet be **bounced straight back** to the forced
change-password screen, because Auth Service reset Password cleared the hash + lockout but **not the
forced-change gate** (frontend Admin Layout intercepts password Change Required). Fixed (reset now calls
update Password Hash And Clear Change Required), verified on dev, logged as **ROADMAP PDO-2**.

It escaped because **the regression lab has zero coverage of forgot → reset → login.**
Auth Journey IT covers login / wrong-pw / logout / me only. This spec closes that gap permanently so
the bug — and the whole reset surface — can never silently regress.

## 2. The hard constraint

The reset flow is **one-time passcode-based**: POST auth reset-password interface operation email, otp (6-digit),
new Password.forgot-password generates a 6-digit one-time passcode, stores **only its bcrypt hash**
(password reset tokens otp hash), and emails the plaintext (dev: it lands in the
an email-disabled diagnostic message with the private passcode omitted log line). **A black-box test cannot read
the plaintext one-time passcode from the database.** Two ways to obtain it for a test:

- **(A) Scrape the dev email log** — fragile (log format, timing, parallelism), couples the test to
  log strings. ❌ Rejected.
- **(B) A the profile operation regression fixture endpoint that issues a reset and returns the
  plaintext one-time passcode directly** — deterministic, no log coupling, mirrors the existing
  Regression Manifest Controller pattern. ✅ Chosen.

## 3. Fixture endpoint (new, local-only)

### 3.1 Contract


Implementation example omitted. This public edition retains the written requirements in this section.


It issues a real reset token (same code path as forgot Password: invalidate prior tokens →
generate one-time passcode → persist otp hash → set expires at) and returns the plaintext one-time passcode **instead of
emailing it**. The reset itself is unchanged — the suite then calls the real
submit operation for auth reset-password with the returned one-time passcode.

### 3.2 Three independent guards (defence in depth — Casey's bar)

1. **the profile operation on the controller** — the bean does not exist in prod (same as
   Regression Manifest Controller). regression ** interface operation is already permit All in Security Config
   precisely because the controller's absence in prod is the real gate.
2. **Reg-band check in the *service*** (not just the controller): the email **must** end with the
   reg-band domain (@regression local, sourced from one shared constant). Any other email →
   REGRESSION EMAIL REQUIRED 400. This holds even if the method is ever called from elsewhere, so
   it can never mint an one-time passcode for a real user (e g. REDACTED email).
3. **No prod wiring path**: the plaintext one-time passcode is returned only from this local controller; the
   production forgot Password path is untouched and still only emails.

### 3.3 Code shape

- the issue reset otp for regression operation → returns plaintext one-time passcode. Normalises email,
  **rejects non-reg-band** (the bad request operation), finds user
  (404 if absent), invalidates prior tokens, generates one-time passcode via the existing private
  generate Numeric Otp, persists the hash + expires at (reuse one-time passcode EXPIRY MINUTES), returns
  plaintext. No email enqueue. Single source of one-time passcode-gen (no duplication).
- New Regression Auth Controller (the profile operation, the request mapping operation) with
  the POST above — kept separate from Regression Manifest Controller for clarity, same gating.
- Reg-band domain constant: reuse the existing one (Reg Admins BAND DOMAIN @regression local);
  expose the bare value to main code via a small shared constant if needed (no test→main dependency:
  put the canonical constant in main, have Reg Admins reference it, OR duplicate the literal with a
  comment — **decision for review**, see §7 Q1).

## 4. Api Client helper

Add to the regression Api Client:


Implementation example omitted. This public edition retains the written requirements in this section.


Re-captures rotated cookies like the other mutating helpers. Returns the otp string.

## 5. The IT — Password Reset Journey IT (extends Abstract Admin IT)

Extends Abstract Admin IT because the headline case needs the reg super-admin to mint a **gated**
admin. All actors are reg-band + self-created → independent + re-runnable.

| # | Test | Asserts |
|---|------|---------|
| **pull request-1** | **Gated admin reset (the PDO-2 regression guard).** super-admin creates a fresh *@regression local admin (pcr is yes) → issue-otp → reset → login with new pw | login 200 **and body password Change Required is no**; the admin can hit an admin read (not bounced). This is the test that would have caught the founder's bug. |
| **pull request-2** | **Normal tutee reset.** register reg-band tutee → issue-otp → reset → login new = 200; **old password now 401** | new works, old rejected (hash actually rotated) |
| **pull request-3** | **Reset clears lockout.** register → 5 wrong logins (lock → 423) → issue-otp → reset → login new = **200** (not 423) | reset restores access from a lockout |
| **pull request-4** | **Wrong one-time passcode rejected.** issue-otp → reset with 000000 (wrong) → 400 INVALID one-time passcode; correct one-time passcode still works afterward | bad one-time passcode doesn't burn the token before max attempts |
| **pull request-5** | **one-time passcode single-use.** issue-otp → reset (200) → reset again same one-time passcode → 400 INVALID one-time passcode | used token can't be replayed |
| **pull request-6** | **Fixture backdoor is reg-band-only (security negative).** issue-otp for REDACTED email → **400 REGRESSION EMAIL REQUIRED** (no one-time passcode minted for a non-band email) | the local fixture cannot target real users |

Notes:
- pull request-3 needs the dev login-rate-limit raised (already RATELIMIT LOGIN LIMIT is 100000 in dev compose)
  and the flake protocol (reset ratelimit:* + failed login attempts) per A1.
- the tag operation for the chain-y ones; pull request-1 + pull request-6 the tag operation (headline + security).

## 6. Lab-first + Do D

- Per **A1**: build → run the lab on the dev stack → coverage, all green BEFORE review. The new IT
  must pass against the live (fixed) backend; the full suite (currently 93) + these 6 must be green.
- Per **A3**: DONE-DEV = lab green + review cleared. PDO-2's prod deploy obligation already exists
  in ROADMAP md; this fixture work itself is dev-only (no prod-affecting change — the fixture is
  the profile operation), so it adds **no** new PROD-DEPLOY-OWED entry.
- Per **A2/A4**: no schema change, no entity column change → A4 N/A. A2 (grep-verify config edits)
  applies to the Security Config/controller wiring.

## 7. Open questions for review

- **Q1 (Morgan/Jordan):** where does the canonical @regression local domain constant live so main
  code (the service guard) and test code (Reg Admins) share ONE source without a main→test or
  test→main smell? Options: (a) a small Regression Constants in main referenced by both; (b) literal
  in main + comment, Reg Admins keeps its own. Recommend (a).
- **Q2 (Casey):** is the profile operation + reg-band service guard sufficient, or should the fixture
  endpoint additionally be compiled out of prod artifacts? (Profile gating is the project's existing
  accepted pattern for Regression Manifest Controller — recommend parity, no new mechanism.)
- **Q3 (Reese):** should pull request-1 also assert via a real admin **mutation** (not just a read) that the
  un-gated admin can now operate, to fully prove the gate lifted? Recommend yes — cheap, stronger.
