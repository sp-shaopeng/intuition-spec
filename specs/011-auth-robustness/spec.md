> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 011
slug: auth-robustness
title: Auth robustness — logout-vs-refresh race, autofill suppression, logged-out poll storm, duplicate-phone error
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED       # DRAFT → APPROVED-pending → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED). Boss re-gates Wave-2 slices.
owner: requirements-engineer   # build owners: software-engineer / security-expert / frontend-expert / bff-specialist
created: 2026-06-22
updated: 2026-09-05
---

# Auth robustness — logout-vs-refresh race, autofill suppression, logged-out poll storm, duplicate-phone error

> **Documentation correction — 2026-09-05:** HttpOnly server logout ordering and honest network-failure semantics; deterministic bcrypt task checks. Current clauses in requirements/design/tasks supersede earlier conflicting directions. Historical HISTORICALLY MARKED IMPLEMENTED labels, dated changelog entries and evidence are retained as history; this correction makes no new runtime, test or deployment claim.


Hardens the authentication surface across four independent defects found in the 2026-06-22 external review
(specs/issues/review-2026-06-22/CONSOLIDATED md). None is a confirmed cross-user breach — the Boss verified authz
is intact and deep-link/refresh auth holds — but each is a real reliability / hygiene / privacy gap on the auth path:

- **T-02 (S3*)** — the frontend "Log Out" click races the silent token-refresh loop (Spec 004); a refresh in flight can
  re-mint cookies after logout, so "logout may not stick". Backend revocation is correct; the fix is frontend-side: halt
  the refresh loop and await revocation before navigating.
- **autofill (S3/S4)** — auth forms don't resist stale browser autofill; saved creds can override typed values and
  one case auto-submitted a forced password-change. Needs correct autocomplete handling.
- **logged-out poll storm (S4)** — the notification bell polls /notifications/unread-count while logged OUT,
  producing ~190× 401s. Gate the poll on an authenticated session.
- **#209.2 (S3) — FOUNDER OVERRIDE (MAKE IT SPECIFIC):** duplicate-phone registration returns a generic "couldn't
  complete registration". The founder overrode the INT-109 generic-fold default and chose a SPECIFIC "this phone
  number is already registered" message, WITH compensating security-expert hardening — retain the per-IP register
  rate-limit AND add a constant-time / consistent-latency guard so the specific message is not a fast enumeration
  oracle. Founder accepts the residual phone-disclosure/Personal Data Protection Act trade-off (see REQ-004).

This spec is the **auth lane**; it shares no source files with the other review specs except the shared frontend auth
store / api client (coordinate with spec 012's PROFILE INCOMPLETE work, which touches the same api client).

- **Requirements:** [requirements md](requirements.md) — EARS + Gherkin AC per REQ-NNN
- **Design:** [design md](design.md)
- **Tasks:** [tasks md](tasks.md)
- **Evidence:** the private evidence directory for the requirement

## Historical delivery/review status

The record below describes the earlier delivery cycle; it is not approval or verification of the 2026-09-05 correction.

APPROVED-pending — the founder overrides are folded in; the Boss re-gates the Wave-2 slices before build. The
author does NOT self-approve. The Boss convenes a scoped team-review (security-expert, software-engineer,
frontend-expert, bff-specialist) on the Wave-2 slices; only when every Review & Sign-Off box is checked does the
Boss flip status to APPROVED and clear build. REQ-004 is now a FOUNDER-RESOLVED override (make-it-specific +
security hardening) — security-expert must sign off the constant-time/rate-limit guard at the Wave-2 gate.

**Wave-2 (S3) slices in this spec:** REQ-001 (logout-vs-refresh race, S3*), REQ-002 (autofill suppression),
REQ-003 (poll-storm regression-test — already-gated, confirm+pin), REQ-004 (duplicate-phone specific message +
hardening — founder override). See specs/issues/review-2026-06-22/WAVE2-SCOPE md.

## Review & Sign-Off
- [ ] Author — requirements (requirements-engineer)
- [ ] Counterpart — logout/refresh-loop halt + revocation ordering (software-engineer)
- [ ] Counterpart — session-resurrection threat + duplicate-phone enumeration tradeoff (security-expert)
- [ ] Counterpart — frontend autofill, poll-gating, logout sequencing (frontend-expert)
- [ ] Counterpart — duplicate-phone error contract shape (bff-specialist)
- [ ] Boss (final sign-off → flips status to APPROVED)

## Changelog

- 2026-09-05 — Documentation alignment correction: HttpOnly server logout ordering and honest network-failure semantics; deterministic bcrypt task checks. No application code or runtime verification in this change. Earlier entries below are historical.
- 1.2 (2026-06-22) **Wave-2 spec-review conditions folded (WAVE2-SPEC-REVIEW md §011).** REQ-004 DESIGN completed
  (was SEND-BACK on the under-specified timing guard): contract PINNED **409 PHONE ALREADY REGISTERED** (mirrors
  EMAIL ALREADY REGISTERED, C2); timing-equalisation PINNED to **always-run-the-same-work — a dummy bcrypt on the
  phone- AND email-conflict paths so all three branches incur one password Encoder encode** (no sleep/floor → no Do S
  amplifier), covering phone-conflict/email-conflict/success (C1); the untestable "statistically indistinguishable"
  AC REPLACED by a **deterministic Password Encoder encode spy** assertion (same invocation count per path) + a
  rate-limit-ordering assertion (exists By Phone never reached when over-limit), C3; metric registration failed
  reason is phone conflictpreserved. REQ-001 cookie-neutralisation made EXPLICIT clear/expire theauth cookie
  client-side on logout, not just call the application programming interface; abort-before-Set-Cookie OR await-then-server-clear ordering) + the
  blocking live "old refresh cookie rejected after logout" test pinned. No status change (still APPROVED-pending;
  Boss re-gates).
- 1.1 (2026-06-22) **FOUNDER OVERRIDE on REQ-004 (#209.2):** rewrote from "default NO CHANGE / Won't" to **Must —
  MAKE IT SPECIFIC**. Duplicate-phone now returns a specific "this phone number is already registered" message
  (aligning phone with the existing specific email path EMAIL ALREADY REGISTERED, Auth Service implementation reference),
  WITH security-expert hardening to soften enumeration: retain the per-IP register rate-limit
  (Auth Service implementation reference) AND add a constant-time/consistent-latency guard (net-new — no timing
  guard exists today) so the specific branch is not a fast oracle; keep INT-109's registration failed
  reason is phone conflict metric. Founder accepts the residual phone-disclosure/Personal Data Protection Act trade-off (recorded in REQ-004).
  Status DRAFT → APPROVED-pending (Boss re-gates Wave-2 slices). Wave-2 (S3) scope clarified: REQ-001/002/003/004.
- 1.0 (2026-06-22) initial draft — ADDED REQ-001 (logout halts silent-refresh + awaits revocation before nav),
  REQ-002 (auth-form autofill suppression), REQ-003 (gate the unread-count poll on auth), REQ-004 (specific,
  non-enumerating duplicate-phone registration error). Grounded against CONSOLIDATED md (T-02, autofill, S4 poll
  storm, #209.2) + Spec 004 (auth session hardening + refresh). status DRAFT pending scoped team-review.
