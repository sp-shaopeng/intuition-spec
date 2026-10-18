# Spec 025 — Design (stream × year scoping, MOE-T1)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## 0. Current state (grounded against live code)

- **backend single-pair check** — the is valid subject level pair operation
  (Taxonomy Repository implementation reference) does a native join:
  subjects s JOIN levels l ON (s level group = l level group OR (s level group is N LEVEL AND
  l level group is O LEVEL)). **Band-level only.** All Secondary subjects + all secondary levels are
  O LEVEL, so every Sec subject × every Sec year/stream passes.
- **backend listing check** — the is coherent subject level listing operation (94) compares
  DISTINCT level group sets (subject side normalises N LEVEL → O LEVEL at 119); accepts iff the two
  group-sets are equal. Same band-level coarseness.
- **4 enforcement surfaces** (unchanged by this spec, only the shared predicate behind them changes):
  - tutee-post create/update → Tutee Post Service validate Taxonomy → 422 UNKNOWN SUBJECT OR LEVEL.
  - tutor listing → Tutor Post Service validate Listing Taxonomy → is Coherent Subject Level Listing → 422.
  - tutor-profile matrix → Profile Service per-pair is Valid Subject Level Pair → 422.
  - **tutor registration** → Auth Service sanitize And Validate Onboarding Subject Levels per-pair
    is Valid Subject Level Pair → 422 (reached from Register Tutor Request subject Levels at Auth Service).
    Spec 020 already added this call; it was MISSED in spec-025 v0.1's surface inventory. Because the
    two-tier change lives **inside** is Valid Subject Level Pair, registration inherits it with no call-site
    edit — but it must be declared (here) and regression-tested (REQ-001).
- **frontend mirror** — taxonomy implementation reference: SUBJECT LEVEL grouped by CODE, LEVEL grouped by CODE,
  allowed Level Codes For Subject (level group filter + N LEVEL FALLBACK GROUP O LEVEL),
  the level options for subject operation, allowed Level Codes For Subjects. Drift test:
  taxonomy test implementation reference asserts the flat code-sets + maps mirror the seed changesets.
- **Seed** — 0064-add-subjects-levels implementation reference, 0052-add-secondary-na-levels implementation reference,
  0067-canonical-taxonomy implementation reference; master ledger last include is 0067 (line 187). Next free number: **0068**.

## 1. Data-model options (founder-decision #2)

### Option A — explicit subject allowed levels junction table  ★ RECOMMENDED


Implementation example omitted. This public edition retains the written requirements in this section.


- **Validation rule:** a subject-and-level pair is valid when the approved mapping contains that exact pair. A listing must give each selected subject at least one compatible selected level, and each selected level at least one compatible selected subject.
- **+** Exact and arbitrary: encodes any real curriculum shape, including A-Math's "S3–S4 EXP + S3–S5 NA,
  no NT". Queryable, indexable, auditable. A new curriculum correction is one append records/soft-row.
- **+** Self-documenting: the table *is* the curriculum map; no implicit widen logic.
- **−** Content-heavy seed: ~ (Secondary subjects × valid years) rows. Mitigated by seeding **Secondary
  only** (the band where it matters) and keeping a band-level fallback for subjects with no junction rows
  (so Pre-School/Primary/JC/IB/Poly/Uni/Adult need no rows — see §2 "scoping the rollout").
- **−** Two sources of truth to keep mirrored (table + frontend) — handled by the drift test (REQ-004).

### Option B — per-subject metadata columns (min/max year + allowed-stream)


Implementation example omitted. This public edition retains the written requirements in this section.


- **+** Lighter: a handful of columns, no row explosion.
- **−** Less expressive: cannot encode irregular shapes cleanly (e g. "S3–S4 EXP but S3–S5 NA" needs
  per-stream max-year; "available EXP+NA but not NT" needs a set). Pushes complexity into parsing
  allowed streams and per-stream year ranges — magic-string-y, fights the No-Magic-Strings rule.
- **−** Levels must also be decomposed into (year, stream) columns to join on — a wider migration than A's
  one table, touching the hot levels reference table.

### Option C — band-level floor + small exclusion list


The illustration for option c — band-level floor + small exclusion list contains the following information:

- subject level exclusions (subject code, level code) -- the KNOWN-BAD combos only.


- **+** Smallest diff; keep is Valid Subject Level Pair and subtract exclusions.
- **−** Wrong default: "everything valid unless listed bad" means every NEW curriculum gap silently passes
  until someone files it. The founder's complaint is exactly that impossible combos are offered; an
  allow-list (A) is correct-by-construction, a deny-list (C) is whack-a-mole. **Not recommended.**

**Recommendation: Option A**, scoped to **Secondary subjects only** in the first cut (DBA to confirm at
review). Correctness-first matches spec 020's HARD-BLOCK stance and the founder's "a teacher spots it"
concern.

## 2. Scoping the rollout (keeps Option A's seed bounded)

The predicate is **two-tier**:
1. If the subject has ANY subject allowed levels rows → the pair is valid iff it is one of those rows
   (precise stream×year scoping). This is seeded for **Secondary subjects** only.
2. Else → fall back to the existing band-level join (level group match + N LEVEL→O LEVEL widen) — so
   Pre-School / Primary / JC / IB / Poly / Uni / Adult subjects keep working with no junction rows and no
   behaviour change.

This bounds the seed to the Secondary curriculum (Appendix A) and makes REQ-005's "other bands unchanged"
true by construction. The same two-tier rule drives the frontend mirror: a subject present in
SUBJECT ALLOWED LEVELS uses its precise set; otherwise allowed Level Codes For Subject keeps the current
level group logic. (DBA/frontend-facing service confirm the two-tier vs full-junction choice at review.)

## 3. Migration notes (Option A, Liquibase)

- New changeset 0068-subject-allowed-levels implementation reference, appended to the master ledger
  **after** 0067 (line 187). New number 0068 (no collision; leave gaps for reserved numbers per
  CLAUDE md).
- database structure definitions via create Table + add Primary Key (PK on subject code, level code = the unique constraint) +
  two FKs ON DELETE RESTRICT (soft-delete only — never physically remove a reference). **No raw database structure definitions.**
- Seed via append records ... when a duplicate is found subject code, level code leave the existing record unchanged → idempotent / re-runnable,
  matching the 0064/0067 pattern. The seed rows are the **founder-approved Appendix A** (do not write the
  seed until Appendix A is approved at review).
- **No ALTER TYPE needed** — no new enum value (we reuse existing subject/level codes; N using T year rows
  S1 NT..S4 NT already exist from 0067). If the founder approves brand-new codes, those go in a separate
  prior changeset (codes first, junction after), in ledger order.
- **S5 asymmetry (H2) — seed against the 13 existing secondary codes ONLY.** The live taxonomy has exactly
  Secondary 1–4 in Express, Normal Academic and Normal Technical + **S5 NA** (verified: 0064-add-subjects-levels implementation reference defines S5 NA;
  0067-canonical-taxonomy implementation reference adds the N using T rows). There is **no S5 EXP / S5 NT** — the 5th year is
  Normal-Academic only. The junction seed expands an Appendix-A "S3–S5 N using A" cell to exactly
  Secondary 3, 4 and 5 Normal Academic and never emits S5 EXP/S5 NT. Because both FKs are ON DELETE RESTRICT and
  reference the levels operation, a seed row naming a **non-existent level code is an FK hard-fail at migration
  time** (the documented operational check red via Testcontainers) — it is NOT silently skipped, so a typo cannot slip through
  to production as a missing-row fail-open.
- The reference tables have **no Java persistence mapping entity** (per Taxonomy Repository doc); subject allowed levels is
  likewise driven by native queries in Taxonomy Repository — no entity, no @Version.
- Index: the PK subject code, level code covers the point lookup; add a secondary index on
  level code only if the listing coherence query benefits (DBA to confirm with EXPLAIN; the sets are tiny).

## 4. backend validation changes (predicate behind the 4 surfaces)

- is Valid Subject Level Pair (Taxonomy Repository) → two-tier: junction-membership when the subject has
  rows, else the existing band join. Single method change; the **4 call-sites** (Tutee Post Service,
  Profile Service, Auth Service registration, and the listing path) are **untouched in shape** —
  they keep raising the pinned **422 UNKNOWN SUBJECT OR LEVEL**.
- **Error code pinned (L1) — 422 UNKNOWN SUBJECT OR LEVEL, no new code.** The spec commits to the
  existing code; a more specific INCOHERENT SUBJECT LEVEL is explicitly **not** introduced — keeping the
  current code means **zero frontend error-map change** and a stable frontend contract. The earlier "frontend-facing service to decide"
  thread is closed.
- **Two-tier is mutually exclusive — no double widen (REQ-006/M2).** The N LEVEL → O LEVEL widen lives
  **only** in tier-2 (the band-level fallback) and fires **only** for a subject with NO junction rows. A
  junction-scoped subject (tier-1) is validated by its precise rows alone; the widen never re-broadens it
  back onto the full O LEVEL band. Concretely: an * NLEVEL subject that HAS junction rows is matched
  against those rows verbatim (so e g. MATH NLEVEL + S4 EXP is rejected even though S4 EXP is in the
  O LEVEL band), because tier-1 short-circuits before the widen. Implement so the band/widen branch is
  unreachable once any precise allowed-level entries exist for that subject.
- is Coherent Subject Level Listing (94) → replace the level group-set coverage with curriculum-set
  coverage: every supplied subject must pair (per the two-tier predicate) with ≥1 supplied level, and every
  supplied level with ≥1 supplied subject. Implement as a single membership query over the candidate
  cross-product (kept small by the supplied set sizes) rather than N+1 per-pair calls.
- Keep the N LEVEL → O LEVEL widen ONLY in the band-level fallback tier (REQ-005 consistency).

## 5. frontend-mirror + drift sync

- taxonomy implementation reference: add the browser's mapping from each subject to its allowed levels (or string) for the
  Secondary subjects (mirrors the junction seed). allowed Level Codes For Subject becomes two-tier: if the
  subject is a key in SUBJECT ALLOWED LEVELS → return that precise set; else keep the existing
  level group-filter + N LEVEL FALLBACK GROUP path. the level options for subject operation,
  allowed Level Codes For Subjects, is Valid Subject Level Pair compose unchanged on top. A code absent from the precise map but present in the canonical subject/group mirror takes the band fallback. Only a code unknown to BOTH mirrors uses the existing all-levels display fallback; the backend still rejects unknown or invalid submitted pairs.
- Pickers (Create Request implementation reference, Create Listing implementation reference,
  Subject Level Matrix Editor implementation reference) need **no structural change** — they already call the
  helpers; they automatically cascade to the precise set.
- **Drift check (REQ-004 / M1) — a NEW parser is required, not a reuse.** The existing drift guard
  taxonomy Level Group Drift test implementation reference (parse Seed) is keyed on code → level group and
  returns a flat Record<string,string>; it has no notion of a subject code, level code row, so
  pointing it at the junction yields an **empty map → a vacuous pass** (a silent no-guard). Add a **new
  parser** that reads 0068-subject-allowed-levels implementation reference into a mapping from each subject to its allowed levels (keyed on
  the junction's two columns), assert the parsed backend row-set is **non-empty**
  (the expect operation — the same loud-fail guard the existing test
  uses at 147/162), then assert it equals a committed EXPECTED SUBJECT ALLOWED LEVELS. A one-side
  change → continuous integration red; a broken parse → continuous integration red (not green).
- **Fail-open completeness check (REQ-007 / M3).** Add a committed INTENTIONALLY BAND LEVEL SUBJECTS
  allow-list and a test that enumerates the **Secondary** subjects in SUBJECT OPTIONS
  (taxonomy implementation reference; level group ∈ O LEVEL, N LEVEL) and asserts each is EITHER a key in
  SUBJECT ALLOWED LEVELS (junction-scoped) OR in INTENTIONALLY BAND LEVEL SUBJECTS. A Secondary subject
  in neither set would silently take the permissive band-level fallback (re-opening the impossible pairs
  this spec closes) — the test fails the build instead, naming the un-scoped subject. The allow-list makes
  any intentional band-level fallback an explicit, documented choice.

## 6. The 4-surface enforcement (unchanged surfaces, finer predicate)

| Surface | Entry | Predicate | Result |
|---|---|---|---|
| Tutee-post create/update | Tutee Post Service validate Taxonomy | is Valid Subject Level Pair (two-tier) | 422 UNKNOWN SUBJECT OR LEVEL |
| Tutor listing create/update | Tutor Post Service validate Listing Taxonomy → is Coherent Subject Level Listing | curriculum-set coverage | 422 UNKNOWN SUBJECT OR LEVEL |
| Tutor profile matrix | Profile Service per-pair | is Valid Subject Level Pair (two-tier) | 422 UNKNOWN SUBJECT OR LEVEL |
| **Tutor registration** | Auth Service sanitize And Validate Onboarding Subject Levels per-pair (from Register Tutor Request subject Levels, Auth Service) | is Valid Subject Level Pair (two-tier) | 422 UNKNOWN SUBJECT OR LEVEL |

All four delegate to the single shared is Valid Subject Level Pair (or, for the listing, the membership
relation it exposes), so the two-tier predicate change is the only code touched — no call-site adds a new
branch. Registration was the surface missed by v0.1's "3-surface" framing.

## 7. Test plan

- **backend unit** — Taxonomy Repository (Testcontainers PG, seeded junction): is Valid Subject Level Pair
  two-tier truth table (junction subject precise; non-junction subject band fallback; cross-band reject;
  N-Level widen still valid for a NON-junction subject); **REQ-006/M2** — a junction-scoped * NLEVEL
  subject is matched against its precise rows ONLY (e g. MATH NLEVEL + S4 EXP → invalid; the widen is
  suppressed), proving tier-1 short-circuits before the band/widen branch; is Coherent Subject Level Listing
  orphan-coverage with the precise set.
- **backend integration (live stack, Abstract Api IT)** — REQ-001 across all **4 surfaces** (tutee create+update,
  profile matrix, **tutor registration**) and REQ-002 (listing) per the Gherkin: incoherent stream×year →
  422; valid → 2xx; cross-band regression → 422; existing-row read OK + update re-validates (REQ-005). The
  registration assertion: submit operation for auth register tutor with Additional Mathematics paired with Secondary 1 Express →
  422 (no account); with Additional Mathematics paired with Secondary 4 Express → 2xx.
- **frontend unit/component (Vitest)** — REQ-003 helper assertions (A-Math excludes S1/S2/N using T/cross-band;
  Humanities excludes lower-sec; unknown-to-both-mirrors subject → all for display only; known non-precise subject → band-only; multi-subject union); a Create Request render
  test that the excluded years are absent after selecting A-Math.
- **frontend drift (Vitest)** — REQ-004 NEW subject code, level code parser: non-empty (>0) assertion +
  precise-map equality + a one-side-mutation fixture proving the guard fails.
- **frontend completeness (Vitest)** — REQ-007/M3: every Secondary SUBJECT OPTIONS subject is junction-scoped
  or in INTENTIONALLY BAND LEVEL SUBJECTS; a fixture adding an un-scoped Secondary subject proves the
  guard fails.
- **Gates** — backend the documented operational check ≥97.5% line AND branch (JaCoCo, Docker up); frontend the documented operational check +
  the documented operational check (tsc) + the documented operational check ≥97.5% line / ≥88% branch.
- **Live convergence** dev → virtual machine: incoherent stream×year pair (AMATH OLEVEL+S1 EXP) → 422 on all 4
  surfaces against the running stack; valid (AMATH OLEVEL+S4 EXP) → 2xx; frontend picker visibly omits the
  excluded years. Evidence under private evidence grouped by the requirement in specification 025.

## 8. Risk / rollback

- **Risk:** a wrong Appendix A row rejects a legitimate pair → supply-side friction. Mitigation: founder/MOE
  signs off Appendix A at review; the junction is data, so a correction is a single re-runnable
  append records ... when a duplicate is found changeset, not code.
- **Rollback:** a missing table is not an empty table: a query referencing a dropped relation fails before its no-row branch. Roll back/deploy the compatible prior band-only application and frontend mirror first, verify no running query references subject allowed levels, then run the reviewed Liquibase rollback that drops the table. Never drop the relation under the running two-tier application.
- **Rollback acceptance:** while the two-tier app runs, an existing empty table reaches the band fallback without a missing-relation error; separately rehearse old-app/old-mirror first → table rollback → band-level read/write success on all 4 surfaces. Capture order and results. This is rollback verification, not approval to silently disable the precise rule in production.
- **No retro-invalidation** (REQ-005): no data migration touches existing posts; write-path only.
