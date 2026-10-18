> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 022-tutor-profile-and-fe-polish
status: HISTORICALLY MARKED IMPLEMENTED
version: 0.2
owners: [frontend-expert, uiux-designer]
source: specs/issues/review-2026-06-22-dev-personas/CONSOLIDATED md
---

# Spec 022 — Tutor profile & frontend polish

MOE-3 + S3/S4 polish from the 2026-06-22 review. Standing rule: regression test per REQ.

## Changelog
- 0.2 (2026-09-05) — Documentation alignment correction authorized by the user: resolve student-count verification against009's existing1–5 plus6-or-more/exact6–20 decision. No application implementation or deployment is claimed.
- 0.1 (2026-06-22) — DRAFT.

## Requirements

### REQ-001 — MOE-3 reconcile completeness banner (S3)
The dashboard "still needed: public handle" banner must agree with the apply-gate predicate (profile shows 100%).
- **Given** a tutor whose profile satisfies Tutor Profile Completeness (apply-gate passes), **when** they view the
  dashboard, **then** the banner does not claim a field is "still needed".
- Design: drive the banner from the same profile Complete/missing Fields source as the apply gate
  (Tutor Profile Completeness), not a separate stale check.
- **Regression test (component):** profile-complete tutor → banner absent; incomplete → banner lists the real missing field.

### REQ-002 — listing wizard pre-fill (S3)
The create-listing wizard must pre-fill qualification/experience already on the profile (don't re-ask).
- **Regression test (component):** the wizard fields are pre-populated from the profile.

### REQ-003 — availability chip spacing (S4)
"Mon Evening" must render "Mon Evening".
- **Regression test (unit):** the chip label has the space.

### REQ-004 — commission line on landing (S4)
The landing page should state the commission model (currently only in FAQ/signup).
- **Regression test (component):** the landing renders the commission line.

### REQ-005 — "6 or more" students drift check (S4, FOUNDER-FLAG)
Retain009's intended "6 or more" choice and exact6–20 input; verify the user interface against the recorded contract below. No new product confirmation is needed.
- **Regression test (unit):** the dropdown matches the Spec-009 contract.

## Retained student-count contract

REQ-005 is a regression check against [009 REQ-005](../009-tutee-post-taxonomy-and-ux/requirements.md): retain choices1–5 and "6 or more"; selecting "6 or more" reveals the exact numeric input constrained to6–20. Verify the option remains visible and values6 and20 are accepted while5/21 in the exact-input branch are rejected. This is an existing decision, not a new founder question.

## Design notes
frontend only: the tutor dashboard banner, Create Listing implementation reference, availability chip component, landing page, the
students dropdown. No backend. Build last (parallel with 019 frontend).

## Tasks
- [ ] REQ-001 banner from completeness source + test
- [ ] REQ-002 wizard pre-fill + test
- [ ] REQ-003 chip spacing + test
- [ ] REQ-004 landing commission line + test
- [ ] REQ-005 "6 or more" drift check + test
- [ ] gate frontend ≥97.5/88; live convergence dev→virtual machine

## Review & Sign-Off
- [ ] frontend-expert (Sam)
- [ ] uiux-designer (Jamie)
- [ ] qa-engineer (Reese)
- [ ] technical-writer (Chris)
- [ ] Boss (Rex)
