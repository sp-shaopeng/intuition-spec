> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 024-postfix-review-fixes
status: HISTORICALLY MARKED IMPLEMENTED
version: 0.1
owners: [frontend-expert, bff-specialist]
source: specs/issues/review-2026-06-23-postfix/CONSOLIDATED md
---

# Spec 024 — Post-fix review fixes (2026-06-23)

6 Boss-verified defects from the 3-persona domain-expert review. Sign-off basis: Boss live-verification of each
root cause (CONSOLIDATED md) + founder decisions (fix-all-6; planning-areas REQUIRED). Each REQ ships its
regression test. Soft-delete only.

## Changelog
- 0.1 (2026-06-23) — DRAFT→APPROVED. Defers MOE-T1 (stream×year scoping, larger). S3/S4 → GH issues.

## Requirements

### REQ-001 — P1 invite-tutor 404 (S2, broken feature)
"Invite tutor to job" must succeed from the tutor page.
- **Root cause:** Tutor Detail implementation reference passes the selected tutor's account identifier (the public tutor-POST/handle id), but
  Job Invitation Service does the find by id operation (USER id) → 404 TUTOR NOT FOUND. The public
  data transfer object hides user Id by design.
- **Fix:** backend resolves the tutor from the **public identity the frontend actually has** — accept the tutor **public
  handle** OR the **tutor-post id** and resolve to the owning TUTOR user server-side (don't require the raw
  user Id on the public surface). frontend sends that identifier. Keep the "don't leak / 404 if not a tutor" behaviour.
  Surface the frontend error if it still fails (no silent close).
- **Guard:** inviting a non-existent/non-tutor → 404 (no leak); inviting self → 422; PENDING post still allowed.
- **Regression test (e2e):** drive the frontend invite-from-tutor-detail flow → 201, appears in the outbound list.
  (This is the gap that masked the bug — no e2e drove the real frontend invite.)

### REQ-002 — P2 wizard Step-1 jam (S2)
Reverting "6 or more" → "1 student" must not block Continue.
- **Fix:** clear the exact-count field's value + required/validation state when the student count leaves "6 or
  more". Create Request implementation reference.
- **Regression test (component):** pick "6 or more", set a count, revert to "1" → Continue enabled, submits.

### REQ-003 — MOE-T3 list-card silent Apply (S2, money)
The browse **list-card** Apply must surface the same typed feedback as the detail page.
- **Fix:** on a 422 TUTOR PROFILE INCOMPLETE from the list-card apply, show the inline guidance / disabled-apply
  reason (reuse the detail-page handling). No silent failure on the core money action.
- **Regression test (component):** list-card apply with an incomplete profile → visible PROFILE INCOMPLETE message.

### REQ-004 — MOE-T4 planning-areas required (S2, founder: REQUIRED)
Reconcile completeness vs the apply gate by making **planning-areas required**.
- **Fix:** planning-areas counts toward profile completeness (the "100%" only shows once set) and is marked
  **required** (not "(optional)") in the profile user interface — so the apply gate (Tutor Profile Completeness, which already
  requires location Planning Areas) never contradicts the banner/percentage. Confirm the backend completeness predicate
  and the frontend completeness/labels agree on planning-areas.
- **Regression test (component):** a profile without planning-areas → not 100%, banner lists it, apply gated;
  with it → 100%, apply allowed.

### REQ-005 — MOE-T2 band label (S2, label)
The level-group header "SECONDARY (O-LEVEL)" must not mislabel the N using A/N using T levels.
- **Fix:** rename the band header to "Secondary" (it spans Express/N using A/N using T). taxonomy implementation reference / the picker grouping.
- **Regression test (unit):** the band label reads "Secondary", not "Secondary (O-Level)".

### REQ-006 — OPS-01 deprecated MATH OLEVEL label (S3)
Existing posts referencing the soft-deprecated MATH OLEVEL must show a friendly label, not the raw enum.
- **Fix:** add a display label/alias for MATH OLEVEL ("Mathematics (O-Level)") in the frontend label map (the subject
  is hidden from the picker but still referenced by old data).
- **Regression test (unit):** MATH OLEVEL → friendly label.

## Deferred / filed
- **MOE-T1** (within-Secondary stream×year scoping) — larger taxonomy-model change; follow-up spec.
- S3/S4 (raw handles, View drops duration/frequency, funnel tile reconcile, phantom Step-4, handle-collision 409,
  P1–P4 "(PSLE)" label, back-nav wipes, mobile nav overlap, a11y) → GH issues.

## Design notes
backend: Job Invitation Service implementation reference (+ controller/data transfer object) for REQ-001 tutor-identity resolution. frontend:
Tutor Detail implementation reference + Invite Tutor Modal implementation reference (REQ-001), Create Request implementation reference
(REQ-002), the browse list-card apply control (REQ-003), tutor profile completeness/labels (REQ-004),
taxonomy implementation reference (REQ-005/006). Build backend (REQ-001) then frontend; backend∥frontend otherwise.

## Tasks
- [ ] REQ-001 backend resolve-by-handle/post-id + frontend call + e2e
- [ ] REQ-002 wizard revert + test
- [ ] REQ-003 list-card apply feedback + test
- [ ] REQ-004 planning-areas required (frontend+backend agree) + test
- [ ] REQ-005 band label + test
- [ ] REQ-006 MATH OLEVEL label + test
- [ ] gates backend ≥97.5/97.5, frontend ≥97.5/88; live convergence dev→virtual machine; file S3/S4 GH issues

## Review & Sign-Off
- [x] Boss (Rex) — live-verified each root cause; founder approved scope + planning-areas-required
- [ ] frontend-expert (Sam)
- [ ] bff-specialist (Riley)
- [ ] qa-engineer (Reese)
