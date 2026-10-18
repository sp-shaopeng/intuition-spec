> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 020-taxonomy-integrity
status: HISTORICALLY MARKED IMPLEMENTED
version: 0.3
owners: [dba, bff-specialist, frontend-expert]
source: specs/issues/review-2026-06-22-dev-personas/CONSOLIDATED md
---

# Spec 020 — Taxonomy integrity (subject↔level scoping)

MOE-2 + taxonomy dupes/gaps from the 2026-06-22 review. Founder decision: **HARD BLOCK** incoherent combos.
Standing rule: regression test per REQ. Soft-delete only.

## Changelog
- 0.3 (2026-09-05) — Documentation alignment correction authorized by the user: separate retained hard blocking from the missing canonical-list approval record. No application implementation or deployment is claimed.
- 0.1 (2026-06-22) — DRAFT. Team-review: Option B (level group join, no new table); frontend scoped picker + drift check.
- 0.2 (2026-06-23) — REQ-001 (3-surface 422 + N LEVEL widen) + REQ-002 (frontend scoped pickers + drift) HISTORICALLY MARKED IMPLEMENTED dev+virtual machine. REQ-003 (canonical SG list) DEFERRED pending founder approval.

## Decision record status

REQ-001/002 band-level hard blocking is explicitly decided. REQ-003's v0.2 deferral and025's later statement that the SG set was added are historical records of different stages; neither provides the exact approved canonical list here. Recover and attach that list and its approval before authorizing additional seed content. Do not interpret this record gap as proof that approval never happened. Spec025 supersedes only the finer Secondary stream/year predicate; other bands retain this contract.

## Requirements

### REQ-001 — subject→level scoping, server (S2, HARD BLOCK)
A subject must constrain its valid levels; an incoherent pair is rejected.
- **Given** a request/listing/profile with subject AMATH OLEVEL + level in a different level group (e g.
  Primary), **when** submitted, **then** the server responds **422 UNKNOWN SUBJECT OR LEVEL** (pair-aware message).
- **Given** a coherent pair (subject and level share level group), **when** submitted, **then** it succeeds.
- Design (Option B, no new table): subjects + levels already carry level group; replace the independent
  Taxonomy Repository existing Subject Codes/existing Level Codes (26,) with is Valid Subject Level Pair
  (join on level group); enforce in Tutee Post Service validate Taxonomy **and** the tutor-listing +
  tutor-profile validation paths (mirror the pattern). No migration to schema; reuse existing level group.
- **Regression test (e2e, 3 surfaces):** POST tutee-post / tutor-listing / tutor-profile with AMATH OLEVEL+
  PSLE → 422 each; valid pair → 2xx.

### REQ-002 — subject→level scoping, frontend picker (S2)
The level picker shows only levels valid for the chosen subject; frontend blocks an incoherent submit before the server.
- **Given** a subject is selected, **when** the level dropdown opens, **then** only valid levels appear.
- Design: taxonomy implementation reference has no level group (only band) → derive a subject→allowed-levels map (or add level group
  to the mirror) + filter the picker in Create Request, Create Listing, Subject Level Matrix Editor. **Add a
  build-time frontend-mirror drift check** (the frontend map must match the backend taxonomy).
- **Regression test (component):** selecting AMATH OLEVEL → the level options exclude Primary/Poly/Uni; a continuous integration
  drift check fails if the frontend map diverges from the backend.

### REQ-003 — canonical SG subject×level list (S3, FOUNDER DECISION)
Remove the duplicate generic "Mathematics (O-Level)" (vs E-Math/A-Math); add missing lower-primary (P1–P4)
subjects, N using T stream, Combined Humanities, H1 sciences. **The canonical list is proposed here and approved by
the founder at this spec's review** before the seed changeset is written.
- Design: Liquibase changeset append records … when a duplicate is found leave the existing record unchanged for the approved additions; if a new level group
  label is needed → its own ALTER TYPE … ADD VALUE run In Transaction is no changeset, ordered before seed.
  Sync taxonomy implementation reference.
- **Regression test (unit):** the approved subjects/levels exist + map to the right level group.

## Design notes
backend: Taxonomy Repository implementation reference, Tutee Post Service implementation reference, tutor-listing + profile validation,
006 X-* implementation reference (+ master). frontend: taxonomy implementation reference, Create Request implementation reference,
Create Listing implementation reference, Subject Level Matrix Editor implementation reference. **Long-pole — build 020 first** (backend→frontend).

## Tasks
- [ ] REQ-001 is Valid Subject Level Pair join + enforce on 3 surfaces → 422 + IT/e2e
- [ ] REQ-002 frontend scoped picker + drift check + component test
- [ ] REQ-003 founder-approved canonical list → seed changeset + taxonomy implementation reference sync + unit test
- [ ] gates backend ≥97.5/97.5, frontend ≥97.5/88; live convergence dev→virtual machine (incoherent pair → 422 on all 3 surfaces)

## Review & Sign-Off
- [ ] dba (Morgan) — model + migration
- [ ] bff-specialist (Riley) — validation contract
- [ ] frontend-expert (Sam) — picker + drift
- [ ] qa-engineer (Reese)
- [ ] technical-writer (Chris)
- [ ] Boss (Rex) — incl. canonical-list approval
