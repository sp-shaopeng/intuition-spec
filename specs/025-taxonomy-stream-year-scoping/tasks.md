# Spec 025 — Tasks (stream × year scoping, MOE-T1)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Status: DRAFT. **No task may start until the Boss-convened spec-review APPROVES the spec** (esp. Appendix A
mapping, the data-model choice, and block-vs-warn). Then build backend → frontend (backend is the long pole; same lane
sequencing as spec 020). Backend and frontend are separate subtrees — safe to parallelize once Appendix A
is fixed.

## Pre-build gate (spec-review — Boss-convened)
- [ ] T0 — Founder/MOE approves **Appendix A** (Secondary stream×year mapping); each row confirmed/corrected.
- [ ] T0 — Founder picks the **data model** (recommend Option A junction, Secondary-scoped, two-tier).
- [ ] T0 — Founder confirms **HARD BLOCK** vs soft-warn (recommend HARD BLOCK).
- [ ] T0 — Sign-off checklist in spec md all checked → flip status DRAFT → APPROVED, bump version.

> **Resolved at v0.2 (no longer founder gates):** error code pinned to 422 UNKNOWN SUBJECT OR LEVEL (L1);
> N-stream subject convention = * NLEVEL codes (Open-Q #3); Hist/Geog/Lit = upper-sec standalone electives
> (H3); S5 = S5 NA-only asymmetry (H2). Appendix-A seed *content* remains founder/MOE-refinable (data).

## Build — Backend (dba + bff-specialist) — REQ-001, REQ-002, REQ-005, REQ-006
- [ ] T1 (dba) — 0068-subject-allowed-levels implementation reference: create Table + PK using subject code,level code + 2 FKs
      ON DELETE RESTRICT; seed approved Appendix A via append records ... when a duplicate is found leave the existing record unchanged; the reviewed database rollback
      drops the table ONLY after compatible prior band-only app and frontend mirror are running and no query references it (design§8). Rehearse that order separately from the existing-empty-table fallback test. Append to master ledger after 0067 (line187). No raw database structure definitions, no ALTER TYPE.
      **H2:** the seed references ONLY the 13 existing secondary codes (Secondary 1–4 in Express, Normal Academic and Normal Technical + S5 NA) —
      expand "S3–S5 N using A" to Secondary 3, 4 and 5 Normal Academic; **never emit S5 EXP/S5 NT** (FK hard-fail). N-stream
      rows use * NLEVEL codes (Open-Q #3 resolved).
- [ ] T2 (bff/swe) — Taxonomy Repository is Valid Subject Level Pair → two-tier (junction membership when the
      subject has rows; **else** existing band join + N LEVEL→O LEVEL widen). Keep method signature.
      **REQ-006/M2:** tier-1 must short-circuit BEFORE the widen — a junction-scoped subject is matched on
      its precise rows ONLY; the widen never re-broadens it (no double widen).
- [ ] T3 (bff/swe) — is Coherent Subject Level Listing → curriculum-set coverage (no-orphan over the precise
      predicate); avoid N+1.
- [ ] T4 — **4 surfaces** unchanged in shape; confirm 422 UNKNOWN SUBJECT OR LEVEL still raised at
      Tutee Post Service, listing path, Profile Service, **AND Auth Service (tutor
      registration, from Register Tutor Request subject Levels at Auth Service)**. Error code pinned to
      UNKNOWN SUBJECT OR LEVEL (L1) — no new code, no frontend error-map change.
- [ ] T5 (qa) — backend unit (Testcontainers) truth table incl. **REQ-006/M2** (junction-scoped * NLEVEL not
      widened) + integration (live Abstract Api IT) per REQ-001/002/005/006 Gherkin **across all 4 surfaces
      incl. tutor registration**, incl. existing-row read-OK / update-revalidates.

## Build — Frontend (frontend-expert) — REQ-003, REQ-004
- [ ] T6 — taxonomy implementation reference: add SUBJECT ALLOWED LEVELS; make allowed Level Codes For Subject two-tier; keep
      all-levels display fallback ONLY for codes unknown to both precise and canonical mirrors; known subjects without precise rows retain band-only fallback. backend validation remains authoritative; composing helpers unchanged.
- [ ] T7 — Pickers (Create Request, Create Listing, Subject Level Matrix Editor) verified to cascade to
      the precise set (no structural change expected).
- [ ] T8 (qa) — Vitest helper + render tests (REQ-003); drift check (REQ-004/M1) using a **NEW parser keyed
      on subject code, level code** (the existing taxonomy Level Group Drift test implementation reference parser is keyed on
      code+level group and would skip the junction → vacuous pass) + a **non-empty (>0) assertion** +
      one-side-mutation fixture; **completeness check (REQ-007/M3)** — every Secondary SUBJECT OPTIONS
      subject is junction-scoped OR in INTENTIONALLY BAND LEVEL SUBJECTS, else build red.

## Gates & convergence
- [ ] T9 — backend the documented operational check ≥97.5/97.5 (Docker up); frontend the documented operational check + the documented operational check + the documented operational check
      ≥97.5 line / ≥88 branch.
- [ ] T10 — Live convergence dev → virtual machine: AMATH OLEVEL+S1 EXP → 422 on all 4 surfaces (incl. tutor register); AMATH OLEVEL+
      S4 EXP → 2xx; frontend picker omits the excluded years. Capture evidence under
      evidence/REQ-001/, evidence/REQ-002/, evidence/REQ-003/.
- [ ] T11 — /team-review (full 18-agent) → Boss verdict; fix SEND-BACK conditions in place + re-verify.
- [ ] T12 — Commit + push (polyrepo, per-subrepo, current branch; no new branch). Update CLAUDE md module
      table + requirements traceability matrix. Close GH #217 with evidence.

## Dependencies
- T0 blocks everything. T1 blocks T2/T3/T5 (seed must exist for IT). T2/T3 block T4/T5. T6 depends on the
  approved Appendix A (same data as T1's seed) but not on T1's code — the private implementation folder parallelizable post-T0. T8
  depends on T6 and on the seed for the drift parse. T9–T12 are terminal.
