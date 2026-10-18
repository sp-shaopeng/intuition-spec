> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 025-taxonomy-stream-year-scoping
status: HISTORICALLY MARKED IMPLEMENTED
version: 0.3
owners: [dba, bff-specialist, frontend-expert, qa-engineer]
source: specs/024-postfix-review-fixes/spec md (MOE-T1 deferral); GH #217
supersedes-in-part: 020-taxonomy-integrity (band-level scoping → finer stream×year scoping)
---

# Spec 025 — Taxonomy: stream × year scoping (MOE-T1)

**MOE-T1**, deferred from spec 024 (GH #217). Spec 020 shipped subject↔level coherence at the
**band level** (level group): a pair is valid iff subject and level share a level group, with an
N LEVEL → O LEVEL widen. That is too coarse for Secondary. Because *every* secondary subject and
*every* secondary level both carry level group O LEVEL (and N-Level subjects widen onto it), the
current join accepts **every Secondary subject at every secondary year and stream** — Sec 1–5 ×
Express, N using A, N using T. So the marketplace currently offers curriculum-impossible pairs:
**A-Math at Sec 1**, **A-Math at N using T**, **Combined Humanities at Sec 1**, **Pure Physics at N using A
lower-sec**, etc. A Singapore teacher spots these instantly; they damage trust on the supply side.

This spec adds **finer-than-band, (subject, exact-level) scoping** so each Secondary subject is valid
only for its real (stream, year) set, enforced on the **4 surfaces** that call
is Valid Subject Level Pair (tutee-post, tutor-listing, tutor-profile, **tutor registration**) and mirrored
in the frontend picker so it only offers the curriculum-valid levels.

## Status

The HISTORICALLY MARKED IMPLEMENTED header records the historical delivery claim; the following approval record is still incomplete. The exact canonical stream×year mapping, data-model selection and finer hard-block approval have not been recovered from the package. Appendix A/design retain their proposed status until that record is attached or the outstanding choice is confirmed. Do not infer "nothing was built" from the older DRAFT text, or infer approval of every proposal from HISTORICALLY MARKED IMPLEMENTED. The explicit020 band-level HARD BLOCK remains settled.

## Changelog
- 0.3 (2026-09-05) — Documentation alignment correction authorized by the user: correct E-Math negative-test rationale, unknown-code fallback and rollback order; preserve unresolved approval provenance. No application implementation or deployment is claimed.

- 0.2 (2026-06-23) — DRAFT revision by requirements-engineer addressing the 7 scoped-review SEND-BACK
  conditions (dba + bff + fe + qa). **H1:** tutor registration declared as the **4th** enforcement surface
  (Auth Service sanitize And Validate Onboarding Subject Levels, fed by Register Tutor Request subject Levels
  at 278) — it inherits the two-tier change inside is Valid Subject Level Pair; REQ-001 now asserts a
  AMATH OLEVEL + S1 EXP register → 422. **H2:** stated (design §3 + Appendix A) that the only existing S5
  code is **S5 NA** (13 secondary codes total: Secondary 1–4 in Express, Normal Academic and Normal Technical + S5 NA); the seed must reference
  only those — a non-existent code is an FK hard-fail. **H3:** Appendix A curriculum corrections (Hist/Geog/
  Lit standalone electives → upper-sec only; A-Math S3–S4 EXP + S3–S5 N using A, not N using T; Higher Chinese note);
  Open-Q #3 resolved using the existing convention (N-stream uses * NLEVEL codes; * OLEVEL subjects do
  NOT map to pure-N-year-only levels). **M1:** REQ-004 now mandates a NEW parser keyed on
  subject code, level code (the existing taxonomy Level Group Drift test implementation reference parser is keyed on
  code+level group → would skip the junction and pass vacuously) + a non-empty (>0) assertion.
  **M2:** the N LEVEL→O LEVEL widen fires ONLY in the band-level fallback tier — a junction-scoped subject
  is validated by its precise rows only (design §4 + REQ test). **M3:** fail-open completeness check — every
  Secondary subject in SUBJECT OPTIONS must have ≥1 junction row OR be explicitly listed as
  intentionally-band-level. **L1:** error code pinned to **422 UNKNOWN SUBJECT OR LEVEL** (frontend-contract
  stable); the "frontend-facing service to decide" thread removed. Surface count 3 → **4**.
- 0.1 (2026-06-23) — DRAFT authored by requirements-engineer. Problem framed against the live band-level
  join (Taxonomy Repository is Valid Subject Level Pair, 94 listing, N LEVEL→O LEVEL widen at 69).
  REQ inventory REQ-001…REQ-005. Recommends data-model **Option A** (subject allowed levels junction).
  Three founder-decision points flagged for the spec-review.

## Scope

In scope:
- A precise subject code, level code validity source replacing/augmenting the band-level join, for the
  **Secondary** band first (the band where stream×year matters); other bands keep band-level behaviour.
- Enforcement → **422 UNKNOWN SUBJECT OR LEVEL** (pinned — frontend-contract stable; see L1) on the
  **4 surfaces** that call is Valid Subject Level Pair:
  1. tutee-post create/update (Tutee Post Service validate Taxonomy),
  2. tutor listing (Tutor Post Service validate Listing Taxonomy via is Coherent Subject Level Listing),
  3. tutor profile matrix (Profile Service),
  4. **tutor registration** (Auth Service sanitize And Validate Onboarding Subject Levels, fed by
     Register Tutor Request subject Levels at Auth Service). All four delegate to the same predicate,
     so the two-tier change is inherited by registration with **no extra call-site edit** — but it MUST be
     declared and regression-tested (REQ-001).
- frontend picker cascades to the **precise** valid levels per chosen subject (extends allowed Level Codes For Subject).
- frontend-mirror + a NEW subject code, level code-keyed drift test (non-empty-asserted) + a fail-open
  completeness check (every Secondary subject junction-scoped or explicitly band-level).
- Backward-compat: **validate-on-write only**; existing posts referencing now-invalid pairs are NOT
  retro-invalidated. Keep the N LEVEL → O LEVEL widen consistent.

Out of scope (this spec):
- New subjects/levels beyond what the stream×year mapping needs (spec 020 REQ-003 already added the SG set;
  N using T year rows S1 NT..S4 NT already exist).
- Cross-band changes (Pre-School / Primary / JC / IB / Poly / Uni / Adult keep band-level scoping).
- Public browse-filter scoping (read path) unless the same helper is trivially reused.

## Founder-decision points (resolve at spec-review — do NOT auto-default)

1. **The canonical Secondary stream×year mapping** (requirements md Appendix A) — the exact valid
   subject, level set per Secondary subject. PROPOSED by us from public SG/MOE curriculum knowledge;
   the founder/MOE expert must confirm or correct each row before any seed is written.
2. **Data model** (design md) — Option A junction table (recommended, precise), Option B per-subject
   min/max-year + allowed-stream columns (lighter, less expressive), or Option C band-floor + exclusion
   list (smallest diff, least correct).
3. **Hard-block vs soft-warn** an out-of-curriculum pair. Spec 020 chose **HARD BLOCK** for band-level
   coherence; recommend HARD BLOCK here too for consistency, but the finer rule rejects more, so confirm.

> **Resolved at this revision (no longer founder-decision points):**
> - **Error code (L1)** — pinned to **422 UNKNOWN SUBJECT OR LEVEL** (the code the 4 surfaces already
>   raise; keeping it avoids an frontend error-map change). The earlier "introduce INCOHERENT SUBJECT LEVEL?"
>   open thread is **dropped**.
> - **N-stream subject convention (was Appendix-A Open-Q #3)** — resolved using the **existing** convention:
>   N-stream students use the *** NLEVEL** subject codes at N using A/N using T year levels (consistent with the
>   current N LEVEL→O LEVEL widen); the * OLEVEL subjects therefore do NOT map to pure-N-year-only
>   levels. Appendix A's N using T columns use * NLEVEL accordingly. (The seed *content* of Appendix A is
>   still founder/MOE-refinable — it is low-risk data, not contract.)

## Review & Sign-Off

- [ ] dba (Morgan) — data model + migration (junction table, unique constraint, when a duplicate is found, frontend-mirror impact)
- [ ] bff-specialist (Riley) — validation contract + pinned 422 UNKNOWN SUBJECT OR LEVEL on the 4 surfaces (incl. tutor registration)
- [ ] frontend-expert (Sam) — precise picker cascade + drift check
- [ ] qa-engineer (Reese) — per-REQ regression (incoherent stream×year → 422 on all 4 surfaces incl.
      registration; valid → 2xx; picker; new-parser drift + non-empty assert; no-double-widen REQ-006;
      fail-open completeness REQ-007)
- [ ] product-designer / business-analyst — block-vs-warn + scope (Secondary-only first)
- [ ] technical-writer (Chris) — copy of the 422 message
- [ ] Boss (Rex) — **incl. canonical stream×year mapping approval** (Appendix A) + data-model choice
