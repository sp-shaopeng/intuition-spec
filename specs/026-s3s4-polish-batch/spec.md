> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 026-s3s4-polish-batch
status: HISTORICALLY MARKED IMPLEMENTED
version: 0.2
owners: [frontend-expert]
source: specs/issues/review-2026-06-23-postfix/CONSOLIDATED md
github: 218
---

# Spec 026 — S3/S4 polish batch (GH #218)

The lower-severity user experience/label/a11y findings from the 2026-06-23 domain-expert review. All frontend-leaning, low-risk.
Boss-verified from CONSOLIDATED; founder approved fixing the batch. Each REQ ships its regression test. Soft-delete only.

## Changelog
- 0.2 (2026-09-05) — Documentation alignment correction authorized by the user: make minor-polish acceptance explicit while keeping the unrecorded parent-wizard duration choice pending. No application implementation or deployment is claimed.
- 0.1 (2026-06-23) — APPROVED. Boss-verified low-risk polish; founder: fix the batch.

## Requirements

### REQ-001 — show session duration + frequency (S3)
The saved-post **View** dialog and the wizard **Review** step must show the session duration + frequency (not just "20 slots").
- Test: View/Review renders duration + frequency for a post that has them.

### REQ-002 — wizard phantom Step-4 (S4)
The post wizard must not advertise a Step 4 that's silently skipped — fix the stepper count to the real number of steps.
- Test: the stepper shows the actual step count; no unreachable step.

### REQ-003 — wizard back-nav preserves fields (S3)
Navigating Back in the post wizard must not wipe entered fields.
- Test: fill Step-2, go Back to Step-1 then Forward → Step-2 values retained.

### REQ-004 — P1–P4 level label (S4)
Lower-primary levels P1–P4 must not be labelled "(PSLE)" (PSLE is the P5/P6 exam) — show the plain primary-year label.
- Test: P1–P4 render without "(PSLE)".

### REQ-005 — tutor handle display on browse (S3)
Browse must not surface raw seed handles like "tutor joanne chua" as the tutor's name — show the tutor's display name / a humanised handle. (If the public data transfer object lacks a display field, frontend flags the backend need.)
- Test: browse card shows a humanised name, not the raw tutor * handle.

### REQ-006 — handle-collision on first profile save (S3)
The auto-suggested public handle must not 409 on first save — on collision, auto-suffix (or surface a clear inline retry), no dead-end.
- Test: a colliding suggested handle → save succeeds (suffixed) or shows a clear retry, not a silent 409.

### REQ-007 — analytics funnel tile reconciliation (S3)
The interview-funnel tiles (Requested / Completed / rate) must reconcile — the displayed numerator/denominator must match the rate. Confirm OPS-03 (in-period signups/posts flat 30d-vs-7d) is the known data-confound (no code) or fix the query.
- Test: the funnel tiles' numbers are internally consistent with the displayed %.

### REQ-008 — minor polish (S4)
Mobile bottom-nav must not overlap the schedule grid; remove the "Step1 of2" stepper from the one-step tutee registration; clear password-form a11y warnings.
- Tests: at 390px, verify the final schedule-grid row is visible/reachable above the bottom-nav; registration renders no multi-step counter; password inputs have programmatic labels and the cited a11y warnings no longer occur.
- **Duration-default decision pending:** the parent request wizard's "2 hours default reviewed" did not record a resulting choice. Preserve its existing behavior until the owner records retain/change, exact duration and affected viewport/actor. Then add an assertion for the initial displayed value and unchanged duration options. Do not reopen the separate tutor-mobile2-hour default already specified by002.

## Design notes
frontend: the post wizard (Create Request implementation reference) for REQ-001/002/003/004/008; browse/tutor card for REQ-005; tutor
profile handle logic for REQ-006; admin Analytics for REQ-007; layout/CSS + a11y for REQ-008. backend only if REQ-005
(display field) or REQ-006 (server-side handle suffix) or REQ-007 (query) require it — frontend flags precisely.

## Tasks
- [ ] REQ-001..008 + per-item regression test
- [ ] gate frontend ≥97.5/88 (+ backend ≥97.5/97.5 if any backend); live convergence dev→virtual machine; close GH #218

## Review & Sign-Off
- [x] Boss (Rex) — Boss-verified low-risk polish; founder approved batch fix
- [ ] frontend-expert (Sam)
