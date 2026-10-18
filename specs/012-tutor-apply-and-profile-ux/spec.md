> **Current-file check, 18 October 2026:** Contact-change delivery, new-email verification, old-phone warnings and owner revocation have confirmed implementation gaps (G03). See the [alignment review](../../docs/current-file-alignment.md). The requirements remain unchanged.

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 012
slug: tutor-apply-and-profile-ux
title: Tutor apply feedback + profile user experience — silent apply-403, completeness criteria parity, sticky-Apply, profile personal information editability
version: 1.4
status: HISTORICALLY MARKED IMPLEMENTED       # Wave-1 REQ-001 APPROVED; REQ-004 rewritten to the email-link fallback (v1.3) — Boss re-gates the Wave-2/REQ-004 slice
owner: requirements-engineer   # build owners: frontend-expert / bff-specialist / software-engineer / uiux-designer
created: 2026-06-22
updated: 2026-09-05
---

# Tutor apply feedback + profile user experience

> **Documentation correction — 2026-09-05:** Email-confirmation task plan, independent revoke contract and old-contact outbox/delivery accounting; old-phone transport remains pending. Current clauses in requirements/design/tasks supersede earlier conflicting directions. Historical HISTORICALLY MARKED IMPLEMENTED labels, dated changelog entries and evidence are retained as history; this correction makes no new runtime, test or deployment claim.


Fixes the tutor-side "apply" and profile-edit experience flagged in the 2026-06-22 review
(specs/issues/review-2026-06-22/CONSOLIDATED md):

- **T-07 (S2, core money action)** — applying with an incomplete profile fails with no usable frontend feedback. The
  backend ALREADY throws a typed 422 TUTOR PROFILE INCOMPLETE ("Complete your tutor profile before applying to
  jobs") at Job Application Service implementation reference, and the frontend ALREADY has a pre-gate (Apply Gate Banner) + a 422
  toast (Tutor Apply Control implementation reference). **The real defect is a frontend↔backend completeness-criteria MISMATCH** plus a
  possible role-403 path: the frontend is Profile Complete (Tutor Apply Control implementation reference) checks location Region +
  qualifications length is at least 10, while the backend checks location Planning Areas (NOT region), commitment, handle, rate
  min/max, qualifications (no length floor) and subject Levels (Job Application Service implementation reference). A tutor the frontend
  thinks is "complete" can be rejected by the backend — and the reviewer saw a bare 403 Access denied with zero
  feedback, so the build must (a) reconcile the criteria to a single source of truth, and (b) prove the gate failure
  always surfaces as a friendly, actionable message (never a bare 403).
- **T-06 (S4)** — the bottom sticky "Apply →" rail on Assignment Detail implementation reference persists after "✓ Interest
  sent" (it's a separate jump-link from the stateful Tutor Apply Control). It should reflect the applied state.
- **#209.1 (S3)** — tutor subject dropdown taxonomy: **ALREADY FIXED.** Create Listing implementation reference and the tutor
  Profile's Subject Level Matrix Editor both consume the shared SUBJECT OPTIONS (spec 009 taxonomy). REQ is
  confirm-live + close on GitHub.
- **#209.3 (S3, FOUNDER OVERRIDE — SELF-EDIT VIA EMAIL-LINK CONFIRMATION, v1.3)** — tutor (and applicable users) can
  edit email/phone from the profile; the change is confirmed via an **email link** before the new value becomes the
  contact of record. Email change → link to the NEW email; **phone change → link to the EXISTING verified account
  email** (the founder's chosen fallback — NOT WhatsApp, NOT defer). The link lands a page requiring an explicit
  **Confirm (POST)** — no bare-GET auto-commit (anti-scanner-prefetch). Token is hashed at rest, single-use, 30-min
  expiry period, bound to the user, changed field, requested new value and confirmation channel. **Step-up re-auth required to initiate; old email AND old
  phone notified with a revoke affordance; uniqueness re-checked AT commit (→409); old value stays of record until
  commit; audited.** Pending state held in a new contact change request table (migration). No dependency on INT-082.
  Scoped as its own Wave-2 slice; security-expert + dba re-gate.

- **Requirements:** [requirements md](requirements.md) · **Design:** [design md](design.md) · **Tasks:** [tasks md](tasks.md)
- **Evidence:** the private evidence directory for the requirement

## Historical delivery/review status

The record below describes the earlier delivery cycle; it is not approval or verification of the 2026-09-05 correction.

APPROVED-pending — REQ-001 was APPROVED at the Wave-1 gate (WAVE1-SPEC-REVIEW md §012). REQ-004 was SEND-BACK at the
Wave-2 gate (the v1.2 spec encoded the SUPERSEDED "phone deferred/pending behind WhatsApp" behaviour) and is now
**rewritten to v1.3 — the email-link confirmation fallback** with the full anti-account-takeover set (step-up,
single-use+30-min-expiry period hashed token, confirm-click landing page, old-email+old-phone notify, uniqueness-at-commit,
contact change request migration). The Boss re-gates the REQ-004 / Wave-2 slice via a scoped team-review
(software-engineer, security-expert, dba, bff-specialist, frontend-expert/uiux-designer) before build. Author does
not self-approve. **No INT-082 dependency — phone re-verify uses the account email, not WhatsApp.**

## Review & Sign-Off
- [ ] Author — requirements (requirements-engineer)
- [ ] Counterpart — single-source completeness criteria + 422 contract (bff-specialist)
- [ ] Counterpart — gate-failure surfacing + role-403 vs 422 path; REQ-004 pending-value model + verify-then-swap (software-engineer)
- [ ] Counterpart — apply control + sticky-rail state + banner user experience; REQ-004 edit + pending-state user experience (frontend-expert / uiux-designer)
- [ ] Counterpart — REQ-004 verification-channel integrity + step-up/anti-takeover on contact change (security-expert)
- [ ] Counterpart — REQ-004 pending-contact / audit schema + migration (dba)
- [ ] Boss (final sign-off → flips status to APPROVED)

## Changelog

- 2026-09-05 — Documentation alignment correction: Email-confirmation task plan, independent revoke contract and old-contact outbox/delivery accounting; old-phone transport remains pending. No application code or runtime verification in this change. Earlier entries below are historical.
- 1.3 (2026-06-22) **Wave-2 spec-review SEND-BACK cleared (WAVE2-SPEC-REVIEW md §012/REQ-004).** REQ-004 fully
  REWRITTEN from the SUPERSEDED "phone deferred/pending-only behind WhatsApp (INT-082 OFF, open infra-dependency)"
  to the founder-decided **EMAIL-LINK confirmation fallback**: email change → link to the NEW email, phone change →
  link to the EXISTING verified account email; the link lands a page requiring an **explicit Confirm (POST)** — no
  bare-GET auto-commit (email-scanner-prefetch class). MANDATED the full anti-account-takeover set: **step-up re-auth
  to initiate; secure-random token hashed at rest, single-use (consumed at), 30-min expiry period, bound to
  the user, changed field, requested new value and confirmation channel; notify the OLD email AND old phone with a revoke affordance at initiation;
  uniqueness re-checked AT commit inside the a database transaction swap (→409 PHONE ALREADY REGISTERED/email conflict),
  not at request time; old value of record until commit; audit row; replay/expired = no-op.** Added the dba
  contact change request table + Liquibase migration (VARCHAR field/status/channel + CHECKs, soft-delete; token
  Password Reset Token-modelled). Pinned the initiate/verify endpoint contract + profile pending Contact Change
  fields + the 8 BLOCKING tests. Removed the INT-082 dependency. status stays APPROVED-pending; Boss re-gates v1.3.
- 1.2 (2026-06-22) **FOUNDER OVERRIDE on REQ-004 (#209.3):** rewrote from "default read-only + explain / Could" to
  **Must — BUILD A SELF-EDIT FLOW WITH RE-VERIFICATION**. User can edit email/phone from the profile; a change is a
  PENDING value re-verified before it becomes the contact of record (email link/one-time passcode; phone via INT-082 WhatsApp/one-time passcode
  if enabled, else a pending-verification state — **INT-082 is OFF, infra-dependency flagged**); old value stays
  active until verified; verify→atomic swap + audit row; uniqueness enforced at commit. Adds a backend lane (pending-value
  model, likely a migration) + dba/security-expert owners. Status APPROVED (REQ-001) → APPROVED-pending (Boss
  re-gates the REQ-004/Wave-2 slice).
- 1.1 (2026-06-22) Wave-1 spec-review APPROVED-condition folded in (WAVE1-SPEC-REVIEW md §012). REQ-001 single-source-
  of-truth DECIDED: backend returns profile Complete + missing Fields from one shared predicate (the 7 apply-gate fields;
  the frontend-only qualifications is at least 10 floor is DELETED, not ported); the frontend is Profile Complete predicate
  (Tutor Apply Control implementation reference) is DELETED and consumes the server signal. Aligned-constant fallback REJECTED unless
  bff records a concrete blocker. Blocking tests pinned: frontend↔backend parity over a field matrix + "403 never silent" (typed
  message, no bare "Access denied"). status stays DRAFT pending Boss re-gate.
- 1.0 (2026-06-22) initial draft — ADDED REQ-001 (reconcile frontend↔backend profile-completeness criteria + ALWAYS surface a
  friendly actionable gate message, never a bare 403), REQ-002 (sticky-Apply rail reflects applied state — T-06),
  REQ-003 (confirm tutor subject taxonomy already shared — #209.1, close), REQ-004 (phone/email editability —
  founder decision — #209.3). Grounded against Job Application Service / Tutor Apply Control / Assignment Detail /
  Create Listing / taxonomy implementation reference. status DRAFT pending scoped team-review.
