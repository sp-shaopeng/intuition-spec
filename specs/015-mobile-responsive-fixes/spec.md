> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 015
slug: mobile-responsive-fixes
title: Mobile/responsive fixes — availability-grid page overflow, admin nav at 390px, tutor nav breakpoint consistency
version: 1.1
status: APPROVED
owner: requirements-engineer   # build owners: frontend-expert / uiux-designer
created: 2026-06-22
updated: 2026-06-22
---

# Mobile/responsive fixes

Fixes the ≤390px responsive defects from the 2026-06-22 review (specs/issues/review-2026-06-22/CONSOLIDATED md):

- **INT-210 (S2, regression)** — the availability grid overflows the PAGE at ≤390px. The grid's own
  overflow-x-auto (Availability Grid implementation reference) wraps a min Width table (34), but a flex/grid ancestor
  isn't width-capped (missing min-w-0), so the overflow-x-auto never engages and the whole page scrolls
  sideways instead of just the grid. Regression vs #206; e2e missed it (only checks Step-1 of the wizard).
- **ADM-nav (S3)** — the admin 8-section nav is a cramped ~154px horizontal scroll strip at 390px with no
  hamburger (the public Nav implementation reference has a hidden md:flex desktop nav + a mobile hamburger drawer at 222, but the
  admin sections aren't given the same treatment).
- **T-04 (S3)** — the desktop nav collapses to the bottom mobile nav on /assignments and /tutor/* (links only
  in Mobile Bottom Nav), while /tutor/dashboard shows the full nav — an inconsistent breakpoint across routes.

frontend-only, responsive lane. Touches Availability Grid + its consuming form layouts, Nav implementation reference/admin nav, and
Mobile Bottom Nav — no overlap with the backend specs.

- **Requirements:** [requirements md](requirements.md) · **Design:** [design md](design.md) · **Tasks:** [tasks md](tasks.md)
- **Evidence:** the private evidence directory for the requirement

## Status

DRAFT — author does not self-approve. Boss convenes a scoped team-review (frontend-expert, uiux-designer) + audit;
flips to APPROVED when sign-off boxes are checked.

## Review & Sign-Off
- [ ] Author — requirements (requirements-engineer)
- [ ] Counterpart — flex/grid min-w-0 overflow fix + breakpoint consistency (frontend-expert)
- [ ] Counterpart — admin nav IA at 390px (uiux-designer)
- [ ] Boss (final sign-off → flips status to APPROVED)

## Changelog
- 1.1 (2026-06-22) Wave-1 spec-review APPROVED-condition folded in (WAVE1-SPEC-REVIEW md §015). REQ-001 min-w-0
  target pinned to the exact ancestor: the Step-1 wrapper  at Create Request implementation reference (the CSS-grid item under
  Form Layout implementation reference's grid track wrapping Availability Grid implementation reference) — NOT a blanket on Form Layout (over-broad,
  flagged). e2e must advance the wizard to Step-2 (the availability step, the availability step, second in the wizard) before the 390px
  scroll Width is at most client Width assertion (the gap was checking only Step-1). status stays DRAFT pending Boss re-gate.
- 1.0 (2026-06-22) initial draft — ADDED REQ-001 (availability-grid page overflow: cap the flex/grid ancestor with
  min-w-0 so overflow-x-auto engages), REQ-002 (admin nav usable at 390px — hamburger/drawer), REQ-003 (consistent
  nav breakpoint across /assignments, /tutor/*, /tutor/dashboard). + a 390px e2e guard (e2e missed INT-210).
  Grounded against Availability Grid implementation reference / Nav implementation reference / Mobile Bottom Nav implementation reference. status DRAFT pending scoped team-review.
