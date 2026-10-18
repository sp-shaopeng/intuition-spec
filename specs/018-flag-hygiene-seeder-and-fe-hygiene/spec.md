> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 018
slug: flag-hygiene-seeder-and-fe-hygiene
title: Flag hygiene, seeder fix, and small frontend hygiene — WhatsApp toast while OFF, seeded interview tutor user id, /dashboard 404, step copy, CTA intent, demo-data variety
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED
owner: requirements-engineer   # build owners: frontend-expert / software-engineer / uiux-designer / data
created: 2026-06-22
updated: 2026-09-05
---

# Flag hygiene, seeder fix, and small frontend hygiene

> **Documentation correction — 2026-09-05:** Pending phone/email-confirmation copy and a minimum 8-post fixture without weakening variety floors. Current clauses in requirements/design/tasks supersede earlier conflicting directions. Historical HISTORICALLY MARKED IMPLEMENTED labels, dated changelog entries and evidence are retained as history; this correction makes no new runtime, test or deployment claim.


Sweeps up the low-severity but real items from the 2026-06-22 review
(specs/issues/review-2026-06-22/CONSOLIDATED md) that don't belong to the larger lanes:

- **T-05 (S4, flag hygiene)** — saving the profile fires a WhatsApp "Sending verification code…"/verification toast
  while WhatsApp is OFF. Phone Settings Section reconciles off an a response indicating that WhatsApp is disabled flag (23) for the modal, but
  a phone-change on profile save still surfaces verification messaging that should be suppressed when the feature is
  OFF. Gate the toast/one-time passcode-send on the WhatsApp-enabled flag.
- **T-08 (low-risk, test-data)** — Dev Data Seeder save Interview (1078-1097) sets tutor Post/tutee/tutee Post/
  statusbut **never setstutor User** → seeded interviews have no valuetutor user id, so the tutor list
  (list Tutor Interviews → find By Tutor User Id) returns empty (this is the T-08 "views disagree" root cause; live
  Interview Service create Interview already sets tutor User at 268, so only seeded rows are affected). Backfill
  tutor User in the seeder.
- **demo-data (S4, data)** — the landing / /assignments are dominated by ~8 identical "Primary 5 · Math (PSLE) ·
  S$30-60" cards. Seeder variety, not code — undercuts "real jobs now".
- **T-01 (S3, routing)** — /dashboard (no role prefix) → hard 404 (App implementation reference catch-all); real dashboards are
  /tutee/dashboard (108) / /tutor/dashboard (123). Add a /dashboard redirect to the role dashboard (or
  login).
- **T-03 (S4, copy)** — a step header reads "All optional" while Subjects & levels is *required
  (Create Request implementation reference) — copy/validation mismatch. Reconcile.
- **CTA-intent (S3, user experience)** — generic /register + "Post a request" CTAs bounce to /login. Default unauthenticated
  CTAs to login (register only on create-account), consistent with M17.

frontend-mostly (routing, copy, CTA, flag toast) + a tiny backend seeder change. The seeder is dev/test-data only — no schema,
low risk. No overlap with the production-logic specs except shared spec conventions.

- **Requirements:** [requirements md](requirements.md) · **Design:** [design md](design.md) · **Tasks:** [tasks md](tasks.md)
- **Evidence:** the private evidence directory for the requirement

## Historical delivery/review status

The record below describes the earlier delivery cycle; it is not approval or verification of the 2026-09-05 correction.

DRAFT — author does not self-approve. Boss convenes a scoped team-review (frontend-expert, software-engineer,
uiux-designer) + audit; flips to APPROVED when sign-off boxes are checked.

## Review & Sign-Off
- [ ] Author — requirements (requirements-engineer)
- [ ] Counterpart — WhatsApp flag-gating + seeder tutor user id (software-engineer · frontend-expert)
- [ ] Counterpart — /dashboard redirect + CTA intent + step copy (frontend-expert · uiux-designer)
- [ ] Boss (final sign-off → flips status to APPROVED)

## Changelog

- 2026-09-05 — Documentation alignment correction: Pending phone/email-confirmation copy and a minimum 8-post fixture without weakening variety floors. No application code or runtime verification in this change. Earlier entries below are historical.
- 1.2 (2026-06-22) **Wave-2 spec-review conditions folded (WAVE2-SPEC-REVIEW md §018/REQ-006 + cross-cutting risk
  #5).** REQ-006: ENUMERATED the exact in-scope CTAs — FLIP set (generic public /register CTAs → /login, e g.
  Rate Guide implementation reference) vs KEEP set (explicit/role-qualified create-account → /register: Nav implementation reference "Sign up",
  Nav implementation reference ?role is tutee, the ?role is tutor strips; Browse Assignments implementation reference already /login); the RTL test
  enumerates both. REQ-001: added the MANDATORY cross-wave NOTE that the WhatsApp-copy suppression is
  **channel-scoped to WhatsApp/one-time passcode only** and MUST NOT suppress 012/REQ-004's email-link phone-verification copy.
  status stays APPROVED.
- 1.1 (2026-06-22) Wave-1 spec-review APPROVED-condition folded in (WAVE1-SPEC-REVIEW md §018). REQ-002:
  Dev Data Seeder Test must ASSERT the invariant (non-null tutor user id  equals  the tutor-post's tutor; the shared
  invariant with create Interview), not merely run. REQ-003 demo-variety: replaced the unmeasurable "looks
  realistic" with a measurable, assertable bar (≥8 subjects / ≥5 levels / ≥4 planning-areas across open showcase
  posts; no single (subject,level,budget-bucket,area) tuple >20%) — kept in Wave 1 with that threshold, noted as the
  cleanest deferral if the Boss trims scope. status stays DRAFT pending Boss re-gate.
- 1.0 (2026-06-22) initial draft — ADDED REQ-001 (suppress WhatsApp verification toast/one-time passcode while the feature is
  OFF — T-05), REQ-002 (seeder backfills tutor user id on interviews — T-08), REQ-003 (seed-data variety —
  demo-data), REQ-004 (/dashboard redirect — T-01), REQ-005 (reconcile "All optional" vs required step copy —
  T-03), REQ-006 (unauthenticated CTAs default to login — CTA-intent). Grounded against Phone Settings Section,
  Dev Data Seeder save Interview, App implementation reference routes, Create Request implementation reference. status DRAFT pending scoped team-review.
