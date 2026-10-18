> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 013
slug: tutee-dashboard-and-invitations
title: Tutee dashboard — edit/view post, venue display, and a sent-invitations / outbound-requests view
version: 1.2
status: HISTORICALLY MARKED IMPLEMENTED
owner: requirements-engineer   # build owners: frontend-expert / bff-specialist / software-engineer / dba
created: 2026-06-22
updated: 2026-09-05
---

# Tutee dashboard — edit/view post, venue display, and a sent-invitations / outbound-requests view

> **Documentation correction — 2026-09-05:** Restored product requirements document revision editing and later PENDING-edit permission; exact outbound-list acceptance. Current clauses in requirements/design/tasks supersede earlier conflicting directions. Historical HISTORICALLY MARKED IMPLEMENTED labels, dated changelog entries and evidence are retained as history; this correction makes no new runtime, test or deployment claim.


Closes the tutee-dashboard gaps from the 2026-06-22 review + founder GitHub issues #207/#208
(specs/issues/review-2026-06-22/CONSOLIDATED md):

- **#207.1 (S2)** — no edit / view action for a post after creating it. **The backend already supports it:**
  replace operation for tutee posts id (Tutee Post Controller implementation reference) + read operation for tutee posts id
  (post Detail). The gap is purely frontend — the tutee Dashboard (Dashboard implementation reference) wires only a Resubmit
  link for REJECTED posts, no general view/edit.
- **#207.2 (S3)** — dashboard "shows only 2 venues". **Root cause FOUND:** Dashboard implementation reference renders only
  venue Preferences 0 in the list cell — a display truncation, not a persistence bug (the full list is on the
  post). Fix the display (and surface all venues on view/edit).
- **#208 (S2) + sent-invitations (S3)** — invite-to-job and request-interview created from a tutor's public page
  don't appear in the tutee's "active requests". **Root cause (verified):** (a) **interviews** created via
  submit operation for interviews DO persist and DO appear under read operation for tutee interviews (my List) — the gap is
  the **tutee Dashboard doesn't surface outbound interview requests** alongside posts; (b) **invitations** have a
  create (post Invite) + withdraw, the Job Invitation row persists every field (Job Invitation implementation reference), but there
  is **NO tutee-side GET** for sent invitations — the "Invited" pill is session-only use State
  (Tutor Detail implementation reference), so a refresh loses it and the dashboard never shows it. Add a tutee sent-invitations
  read endpoint + surface both outbound types on the dashboard.

This is a tutee-lane spec; it touches Job Invitation Service + a controller (new read endpoint) on the backend and the
tutee Dashboard + Tutor Detail on the frontend — no overlap with the auth/apply specs.

- **Requirements:** [requirements md](requirements.md) · **Design:** [design md](design.md) · **Tasks:** [tasks md](tasks.md)
- **Evidence:** the private evidence directory for the requirement

## Historical delivery/review status

The record below describes the earlier delivery cycle; it is not approval or verification of the 2026-09-05 correction.

DRAFT — author does not self-approve. Boss convenes a scoped team-review (bff-specialist, frontend-expert,
software-engineer, uiux-designer) + audit; flips to APPROVED when sign-off boxes are checked.

## Review & Sign-Off
- [ ] Author — requirements (requirements-engineer)
- [ ] Counterpart — sent-invitations read endpoint contract (personal information rules) (bff-specialist)
- [ ] Counterpart — the read endpoint + soft-delete-aware query (software-engineer)
- [ ] Counterpart — dashboard IA: posts + outbound requests + venue display (frontend-expert · uiux-designer)
- [ ] Boss (final sign-off → flips status to APPROVED)

## Changelog

- 2026-09-05 — Documentation alignment correction: Restored product requirements document revision editing and later PENDING-edit permission; exact outbound-list acceptance. No application code or runtime verification in this change. Earlier entries below are historical.
- 1.1 (2026-06-22) Wave-1 spec-review SEND-BACK resolved (WAVE1-SPEC-REVIEW md §013). (1) REQ-003 read-endpoint
  contract pinned: reuse the existing personal information-safe Job Invitation Response as-is (field-by-field tutee-visibility table),
  query by Job Invitation created By (40) on Tutee Invitation Controller, + a new DBA index
  idx jobinv createdby status created by, status (no existing index covers created by). (2) REQ-001
  editable-status list RESOLVED from the backend So T (Tutee Post Service implementation reference the historical editable set of draft, rejected and active posts;
  the other 5 Post Status → View-only; ACTIVE-edit re-enters PENDING re-approval; editable fields enumerated). (3) #208
  dashboard IA DECIDED: one dedicated "Your outbound requests" section on /tutee/dashboard below "My posts" listing
  Invitations-sent + Interview-requests with status — an endpoint nobody renders does not close #208. Removed the
  phantom "confirm which statuses PUT allows" backend task; added the DBA index task + own-only/personal information-contract blocking
  tests. All three open questions closed. status stays DRAFT pending Boss re-gate.
- 1.0 (2026-06-22) initial draft — ADDED REQ-001 (frontend edit/view post action — backend PUT/GET already exist), REQ-002
  (venue display: show all, not just [0]), REQ-003 (tutee sent-invitations GET endpoint + dashboard surface),
  REQ-004 (surface outbound interview requests on the tutee dashboard — interviews already in /tutee/interviews).
  Grounded against Tutee Post Controller, Job Invitation using Service, Tutor Detail, tutee Dashboard. status DRAFT pending
  scoped team-review.
