# Spec 025 — Requirements (stream × year scoping, MOE-T1)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Format: EARS statement + Gherkin (positive + negative) + the regression test, per REQ. All Gherkin uses
the current live mechanism as the baseline (band-level join at Taxonomy Repository is Valid Subject Level Pair).

> The exact subject, level validity set referenced below is **Appendix A**, which is PROPOSED and
> pending founder/MOE approval at the spec-review. The REQs are stable; the *data* in Appendix A is the
> founder-decision input.

---

## REQ-001 — server-side stream×year scoping, single-pair surfaces (tutee-post + tutor profile matrix + tutor registration)

**EARS:** WHEN a tutee-post (create or update), a tutor-profile teaching-matrix entry, **or a tutor
registration onboarding subject-level entry** supplies a subject Code, level Code pair that is NOT in the
approved curriculum set for that subject, the system SHALL reject the request with
422 UNKNOWN SUBJECT OR LEVEL (pair-aware message) and SHALL NOT persist. WHEN the pair IS in the approved
set, the system SHALL accept it.

This changes the single shared predicate Taxonomy Repository is Valid Subject Level Pair from a band-level
join to a precise subject, exact-level check (Appendix A / the chosen data model). All three single-pair
call-sites delegate to it and so inherit the finer rule with no per-call-site edit:
Tutee Post Service, Profile Service, and **Auth Service sanitize And Validate Onboarding Subject Levels**
(reached from tutor registration via Register Tutor Request subject Levels at Auth Service; it already
raises the same 422 UNKNOWN SUBJECT OR LEVEL). Because registration shares the predicate, it is a
**declared 4th surface** that MUST carry its own regression assertion (below), not an assumed side effect.


- Scenario: A-Math at Sec 1 is rejected (curriculum-impossible — A-Math is upper-secondary only).
- Given an authenticated tutee.
- When they POST tutee posts interface operation with subject Code "AMATH OLEVEL" and level Code "S1_EXP".
- Then the response is 422 with error code "UNKNOWN SUBJECT OR LEVEL".
- And no tutee post row is created.
- Scenario: A-Math at N using T is rejected (A-Math is not offered in the Normal-Technical stream).
- Given an authenticated tutee.
- When they POST tutee posts interface operation with subject Code "AMATH OLEVEL" and level Code "S4_NT".
- Then the response is 422 with error code "UNKNOWN SUBJECT OR LEVEL".
- Scenario: A-Math at Sec 4 Express is accepted (a real curriculum pair).
- Given an authenticated tutee.
- When they POST tutee posts interface operation with subject Code "AMATH OLEVEL" and level Code "S4_EXP".
- Then the response is 2xx and the tutee post is created.
- Scenario: Combined Humanities at Sec 1 is rejected (upper-secondary subject only).
- Given an authenticated tutor updating their teaching matrix.
- When they submit a matrix entry subject Code "HUMANITIES OLEVEL" level Code "S1_EXP".
- Then the response is 422 with error code "UNKNOWN SUBJECT OR LEVEL".
- Scenario: Combined Humanities at Sec 4 Express is accepted.
- Given an authenticated tutor updating their teaching matrix.
- When they submit a matrix entry subject Code "HUMANITIES OLEVEL" level Code "S4_EXP".
- Then the response is 2xx.
- Scenario: A-Math at Sec 1 is rejected at tutor REGISTRATION (4th surface, shared predicate).
- Given an unauthenticated visitor registering as a tutor.
- When they POST auth register tutor interface operation with subject Levels containing  "AMATH OLEVEL", "S1_EXP" .
- Then the response is 422 with error code "UNKNOWN SUBJECT OR LEVEL".
- And no tutor account is created.
- Scenario: A-Math at Sec 4 Express is accepted at tutor REGISTRATION.
- Given an unauthenticated visitor registering as a tutor.
- When they POST auth register tutor interface operation with subject Levels containing  "AMATH OLEVEL", "S4_EXP" .
- Then the response is 2xx and the tutor account is created.
- Scenario (negative — cross-band still rejected, regression of spec 020): A-Math at PSLE P6.
- Given an authenticated tutee.
- When they POST tutee posts interface operation with subject Code "AMATH OLEVEL" and level Code "P6".
- Then the response is 422 (still rejected — finer rule must not regress the band-level guard).
- Scenario (negative — N using T subject at its real years still accepted): N-Level Math at Sec 3 N using T.
- Given the N-Level Math subject "MATH NLEVEL" is approved for N using T Sec 1–4 in Appendix A.
- And an authenticated tutee.
- When they POST with subject Code "MATH NLEVEL" and level Code "S3_NT".
- Then the response is 2xx.


**Regression test (REQ-001):** backend integration test (live-stack Abstract Api IT, localhost) asserting,
per surface — **tutee-post create, tutee-post update, tutor-profile matrix, AND tutor registration** (the
4th surface): the incoherent stream×year pairs AMATH OLEVEL, S1 EXP and AMATH OLEVEL, S4 NT → 422;
the valid pair AMATH OLEVEL, S4 EXP → 2xx; the cross-band pair AMATH OLEVEL, P6 → 422 (spec-020
regression); a known N using T pair → 2xx. The registration assertion specifically: a tutor-register request
carrying Additional Mathematics paired with Secondary 1 Express → 422 UNKNOWN SUBJECT OR LEVEL (no account created),
and Additional Mathematics paired with Secondary 4 Express → 2xx — proving the shared-predicate change reaches the onboarding path.

---

## REQ-002 — server-side stream×year scoping, tutor-listing surface (multi-subject × multi-level set)

**EARS:** WHEN a tutor listing supplies a subjects × levels set whose implied cross-product
contains NO curriculum-valid subject, level pair for some supplied subject (an "orphan" subject that
cannot pair with any supplied level), the system SHALL reject it with 422 UNKNOWN SUBJECT OR LEVEL.
WHEN every supplied subject pairs with at least one supplied level (and vice versa) under the approved
curriculum set, the system SHALL accept it.

This refines is Coherent Subject Level Listing from a band-set coverage check to a curriculum-set
coverage check. The listing stores subjects and levels as independent sets; the finer rule must keep the
"no orphan code" semantics (each subject pairs with ≥1 level and each level with ≥1 subject) but evaluated
against the precise subject, level set, not the level group cross-product.


- Scenario: listing offering A-Math but only lower-sec levels is rejected.
- Given an authenticated tutor creating a listing.
- When subjects = "AMATH OLEVEL" and levels = "S1_EXP","S2_EXP".
- Then the response is 422 "UNKNOWN SUBJECT OR LEVEL" (A-Math pairs with no supplied level).
- Scenario: listing offering A-Math at upper-sec Express is accepted.
- Given an authenticated tutor creating a listing.
- When subjects = "AMATH OLEVEL" and levels = "S3_EXP","S4_EXP".
- Then the response is 2xx.
- Scenario (negative — mixed set with one orphan subject): A-Math + E-Math at only Sec 1.
- Given an authenticated tutor creating a listing.
- When subjects = "AMATH OLEVEL","EMATH OLEVEL" and levels = "S1_EXP".
- Scenario (positive — broad but coherent secondary listing).
- Given an authenticated tutor creating a listing.
- When subjects = "EMATH OLEVEL","AMATH OLEVEL" and levels = "S2_EXP","S3_EXP","S4_EXP".
- Scenario (regression — empty set vacuously coherent, unchanged from spec 020).
- Given an authenticated tutor creating a listing with no subjects or no levels.
- When validated.
- Then the listing-taxonomy check does not reject on coherence (data transfer object owns "at least one").


**Regression test (REQ-002):** backend integration test on the tutor-listing surface: orphan-subject sets
Additional Mathematics paired with Secondary 1 Express and the mixed Additional and Elementary Mathematics paired with Secondary 1 Express → 422; the coherent
Elementary and Additional Mathematics paired with Secondary 2, 3 and 4 Express → 2xx; empty set vacuously OK.

---

## REQ-003 — frontend picker cascades to the precise valid levels per subject

**EARS:** WHEN a subject is selected in a marketplace picker (tutee Create Request, tutor Create Listing,
tutor-profile Subject Level Matrix Editor), the level options SHALL be restricted to exactly the
curriculum-valid levels for that subject (the precise set, not the whole level group band), so an
incoherent stream×year submit is blocked client-side before the server 422.

This extends allowed Level Codes For Subject in taxonomy implementation reference from a level group-set filter to a precise
per-subject allowed-level-code set (mirroring the chosen backend data model). The multi-subject helpers
(allowed Level Codes For Subjects, level Options For Subjects) compose from the same precise set. The
**fail-open** rule for an unknown subject (return all levels) is retained — the backend stays authority.


- Scenario: selecting A-Math offers only upper-sec Express/N using A years.
- Given the Create Request level picker.
- When subject "AMATH OLEVEL" is selected.
- Then the level options include "S3_EXP" and "S4_EXP".
- And the level options exclude "S1_EXP", "S2_EXP", "S4_NT", "P6", "JC1".
- Scenario: selecting Combined Humanities offers only upper-sec O-Level/N using A years.
- Given the Create Listing level picker.
- When subject "HUMANITIES OLEVEL" is selected.
- Then the level options exclude "S1_EXP" and "S2_EXP".
- Scenario (negative — unknown subject fails open): a brand-new subject not yet in the mirror.
- Given a subject code absent from BOTH SUBJECT ALLOWED LEVELS and the canonical subject/level-group mirror.
- When it is selected.
- Then the picker shows ALL levels as the existing display fallback.
- Scenario (known subject without precise rows): retain band fallback.
- Given a known canonical subject with no SUBJECT ALLOWED LEVELS entry.
- When it is selected.
- Then the picker shows only its band-level options, including the existing N LEVEL-to-O LEVEL fallback when applicable.
- And it does not show every band.
- Scenario (multi-subject union): A-Math + E-Math selected together.
- Given the tutor listing picker with subjects "AMATH OLEVEL","EMATH OLEVEL".
- Then the offered levels are the UNION of each subject's precise allowed levels.


**Regression test (REQ-003):** Vitest component/unit tests on allowed Level Codes For Subject,
level Options For Subject, allowed Level Codes For Subjects: AMATH OLEVEL excludes S1/S2/N using T/cross-band;
HUMANITIES OLEVEL excludes lower-sec; subject unknown to both mirrors → all levels for display only; known subject without precise rows → band-only fallback; multi-subject union correct. Plus a
render test that the Create Request level dropdown does not list the excluded years after selecting A-Math.

---

## REQ-004 — frontend-mirror drift check extended to the precise (subject, level) set

**EARS:** WHEN the frontend precise subject → allowed levels mirror diverges from the backend curriculum source
(the seeded junction / metadata of the chosen data model), the build SHALL fail (continuous integration red), so the two cannot
silently drift.

**A NEW parser is required — the existing one cannot be reused.** The current drift guard
(taxonomy Level Group Drift test implementation reference, parse Seed) is keyed on **code → level group**:
it reads only the code and level group columns and returns a flat Record<string,string>. The
subject allowed levels junction seed has **neither column as the key** (its rows are
subject code, level code with no level group), so feeding it to the existing parser yields an EMPTY
map and the assertion would pass **vacuously** — a silent no-guard. REQ-004 therefore mandates a **new
parser keyed on subject code, level code** that parses 0068-subject-allowed-levels implementation reference into a
a mapping from each subject to its allowed levels.

To make a failed/empty parse fail LOUD (not vacuously green), the test SHALL assert the parsed backend row-set
is **non-empty** — the expect operation (the same
parsed-no-rows-guard pattern the existing drift test already uses at
taxonomy Level Group Drift test implementation reference/162). If Option B (metadata columns) is chosen instead, the new
parser reads the min/max-year + stream metadata, with the same non-empty guard.


- Scenario: frontend precise mirror matches the backend curriculum seed.
- Given the committed expected (subject → allowed level codes) set.
- When the drift test runs.
- Then SUBJECT ALLOWED LEVELS equals the set parsed from the backend seed changeset.
- Scenario (negative — drift fails the build).
- Given a subject's allowed-level set is changed on ONE side only.
- When the drift test runs.
- Then the test fails (build red), surfacing the divergence loudly.


**Regression test (REQ-004):** a drift case using the **new subject code, level code parser**: parse
0068-subject-allowed-levels implementation reference into a mapping from each subject to its allowed levels; assert
the keys operation (loud-fail on a broken/empty parse — guards against the vacuous-pass
trap); assert it equals the committed EXPECTED SUBJECT ALLOWED LEVELS; a deliberate one-side mutation in
a fixture proves the guard fails red.

---

## REQ-005 — backward-compatibility: validate-on-write only, no retro-invalidation

**EARS:** WHEN spec 025 ships, existing tutee posts / tutor post * / tutor profile matrix rows that
reference a now-curriculum-invalid subject, level pair SHALL remain valid and readable (no migration
flips, deletes, or status changes them). The finer rule SHALL apply ONLY on write (create/update). The
N LEVEL → O LEVEL widen SHALL remain consistent between backend and frontend.


- Scenario: an existing post with a now-invalid pair still renders and is not deleted.
- Given a tutee post created before spec 025 with (AMATH OLEVEL, S1_EXP).
- Then it renders normally and its row is unchanged (soft, validate-on-write only).
- Scenario: editing that existing post forces it to a valid pair.
- Given the same pre-existing (AMATH OLEVEL, S1_EXP) post.
- When the owner submits an update keeping (AMATH OLEVEL, S1_EXP).
- Then the update is 422 "UNKNOWN SUBJECT OR LEVEL" (write-path enforces the finer rule).
- Scenario: the N LEVEL→O LEVEL widen is consistent on both ends.
- Given an N-Level subject approved for its N using A/N using T years in Appendix A.
- When validated on backend and offered in the frontend picker.
- Then both treat the same year set as valid (no end-only divergence).


**Regression test (REQ-005):** backend test that seeds (via a fixture, not migration) a pre-existing invalid
pair and asserts a READ succeeds while an update records re-validates → 422; a backend+frontend parity assertion that an
N-Level subject's valid year set is identical on both ends (covered jointly by REQ-001 and REQ-004).

---

## REQ-006 — no double-widen: a junction-scoped subject is validated by its precise rows ONLY

**EARS:** WHEN a subject has ≥1 subject allowed levels junction row (it is junction-scoped), the system
SHALL validate its pairs **solely** against those precise rows; the N LEVEL → O LEVEL band widen SHALL
NOT fire for that subject. The widen SHALL fire ONLY in the band-level fallback tier — i e. for subjects
with NO junction rows. This prevents an * NLEVEL subject that has been given precise junction rows from
being silently re-broadened back to the whole O LEVEL band by the legacy widen.


- Scenario: a junction-scoped N-Level subject is NOT widened onto the full O LEVEL band.
- Given "MATH NLEVEL" has junction rows for its N using A S1–S5 and N using T S1–S4 years only (Appendix A).
- When a pair "MATH NLEVEL" + "S4_EXP" (an Express level, outside its junction rows) is validated.
- Then the response is 422 "UNKNOWN SUBJECT OR LEVEL" (the N LEVEL→O LEVEL widen does NOT apply).
- Scenario: a NON-junction N-Level subject still uses the band widen (unchanged fallback).
- Given a hypothetical N-Level subject with NO junction rows.
- When one of its N using A/N using T/O LEVEL-band pairs is validated.
- Then the band-level fallback (incl. N LEVEL→O LEVEL widen) applies as before spec 025.


**Regression test (REQ-006):** backend unit test (Testcontainers, seeded junction): for a junction-scoped
* NLEVEL subject, an Express-level pair outside its junction rows → invalid (widen suppressed); for a
non-junction subject, the band widen still resolves. Asserts the two tiers are mutually exclusive: a subject
never gets BOTH precise rows AND the widen.

---

## REQ-007 — fail-open completeness check: no Secondary subject silently falls back to permissive band-level

**EARS:** WHEN spec 025 ships, EVERY Secondary subject present in the frontend SUBJECT OPTIONS
(taxonomy implementation reference) SHALL EITHER have ≥1 subject allowed levels junction row OR be explicitly
listed in a committed INTENTIONALLY BAND LEVEL SUBJECTS allow-list. A Secondary subject that is in
neither set (a forgotten seed row) SHALL fail a test — because the two-tier predicate would otherwise let
it **silently fall back to the permissive band-level join** (every Sec year/stream accepted), re-opening
exactly the curriculum-impossible pairs this spec exists to close. The check fails the build, not
production.


- Scenario: every Secondary subject is either junction-scoped or explicitly band-level.
- Given the set of Secondary subjects in SUBJECT OPTIONS (level group O LEVEL or N LEVEL).
- When the completeness check runs.
- Then each one is a key in SUBJECT ALLOWED LEVELS OR in INTENTIONALLY BAND LEVEL SUBJECTS.
- Scenario (negative — a forgotten subject fails the build).
- Given a new Secondary subject added to SUBJECT OPTIONS with no junction rows and no allow-list entry.
- When the completeness check runs.
- Then the test fails (build red), naming the un-scoped subject.


**Regression test (REQ-007):** Vitest test enumerating Secondary SUBJECT OPTIONS codes and asserting each
is covered by SUBJECT ALLOWED LEVELS (junction-scoped) or INTENTIONALLY BAND LEVEL SUBJECTS (explicit
opt-out); a fixture adding an un-scoped Secondary subject proves the guard fails. (The allow-list documents
*why* a Secondary subject is intentionally band-level, so the fail-open path is a conscious choice, not an
omission.)

---

## Appendix A — PROPOSED Secondary stream × year mapping (PENDING founder/MOE approval)

> **This table is PROPOSED, not approved** (founder/MOE may refine the seed content — it is data, low-risk
> to adjust). It encodes the requirements-engineer's reading of the public SG/MOE secondary curriculum.
> **Every row must be confirmed or corrected by the founder/MOE expert** before any seed/metadata is
> written. Streams: **EXP** = Express (S1 EXP..S4 EXP), **NA** = Normal Academic
> (S1 NA..S4 NA + S5 NA), **NT** = Normal Technical (S1 NT..S4 NT). "Upper-sec" = Sec 3–4 (EXP/NA)
> plus Sec 5 (NA 5th year). Non-Secondary bands are unchanged (band-level scoping retained).
>
> **⚠ S5 ASYMMETRY (H2) — only S5 NA exists.** The 5th secondary year is a Normal-Academic-only year.
> The live taxonomy has **exactly 13 secondary level codes**: Secondary 1–4 in Express, Normal Academic and Normal Technical (12) **plus
> S5 NA** (1). There is **NO S5 EXP and NO S5 NT** (verified against
> 0064-add-subjects-levels implementation reference and 0067-canonical-taxonomy implementation reference). Therefore an Appendix-A row's
> Express column tops out at **S4**, its N using A column may extend to **S5_NA**, and its N using T column tops out
> at **S4**. The subject allowed levels seed MUST reference ONLY these 13 existing codes — a row naming a
> non-existent code (e g. S5 EXP, S5 NT) is an **FK hard-fail at migration time** (the FK to
> the levels operation rejects it; it is NOT silently skipped). The seed writer expands "S3–S5 N using A" to exactly
> Secondary 3, 4 and 5 Normal Academic, never an S5 EXP/S5 NT.

| Subject (code) | Express years | N using A years | N using T years | Notes (proposed, verify) |
|---|---|---|---|---|
| E-Math (EMATH OLEVEL) | S1–S4 | S1–S5 | — | core EXP/NA; N using T sits N using T Maths, not E-Math |
| A-Math (AMATH OLEVEL) | S3–S4 | S3–S5 | — | upper-sec elective; **EXP S3–S4, N using A S3–S5**; NOT N using T; NOT lower-sec (H3 confirmed) |
| Pure Physics (PHYS OLEVEL) | S3–S4 | S3–S5 | — | pure sciences are upper-sec EXP/NA only |
| Pure Chemistry (CHEM OLEVEL) | S3–S4 | S3–S5 | — | upper-sec EXP/NA |
| Pure Biology (BIO OLEVEL) | S3–S4 | S3–S5 | — | upper-sec EXP/NA |
| Combined Science (SCI OLEVEL) | S3–S4 | S3–S5 | — | upper-sec; verify N using T takes a separate science |
| Combined Humanities (HUMANITIES OLEVEL) | S3–S4 | S3–S5 | — | upper-sec O-Level/N using A |
| History (HIST OLEVEL) | S3–S4 | S3–S5 | — | **upper-sec standalone elective** (H3): EXP S3–S4, N using A S3–S5. Lower-sec History is taught inside the integrated Humanities programme, not as a standalone marketplace subject |
| Geography (GEOG OLEVEL) | S3–S4 | S3–S5 | — | **upper-sec standalone elective** (H3): EXP S3–S4, N using A S3–S5 |
| Literature (LIT OLEVEL) | S3–S4 | S3–S5 | — | **upper-sec standalone elective** (H3): EXP S3–S4, N using A S3–S5 |
| Social Studies (SS OLEVEL) | S3–S4 | S3–S5 | — | upper-sec component of Combined Humanities |
| POA (POA OLEVEL) | S3–S4 | S3–S5 | — | upper-sec elective |
| English (ENG OLEVEL) | S1–S4 | S1–S5 | — | core all years EXP/NA |
| Chinese / MT (CHN OLEVEL,MALAY OLEVEL,TAMIL OLEVEL) | S1–S4 | S1–S5 | — | core; Higher MT upper-sec — verify |
| Higher Chinese (HCHN OLEVEL) | S1–S4 | S1–S5 | — | **default S1–S4 EXP / S1–S5 N using A** (H3 note): Higher Mother Tongue runs across lower- and upper-sec for MT-biased pupils; left at the full secondary range pending founder confirmation (do NOT narrow to upper-sec without sign-off). Mirror the same range for Higher Malay/Tamil if those codes are added |
| English N-Level (ENG NLEVEL) | — | S1_NA–S5_NA | S1_NT–S4_NT | N-stream English — * NLEVEL is the canonical N-stream code (Open-Q #3 resolved); N using A → S1 NA..S5 NA, N using T → S1 NT..S4 NT |
| Math N-Level (MATH NLEVEL) | — | S1_NA–S5_NA | S1_NT–S4_NT | N-stream Maths — * NLEVEL canonical (no * OLEVEL at pure-N years) |
| Science N-Level (SCI NLEVEL) | — | S1_NA–S5_NA | S1_NT–S4_NT | N-stream Science — * NLEVEL canonical |

**Resolved at this revision:**
- **(was Q1) Lower-sec humanities** — Hist/Geog/Lit are taken as **upper-sec standalone electives only**
  (S3–S4 EXP, S3–S5 N using A); lower-sec humanities is the integrated programme, not a standalone marketplace
  subject. Rows corrected above (H3). *Founder may still re-confirm; low-risk data.*
- **(was Q3) N-stream subject convention** — RESOLVED using the **existing** convention: N-stream uses the
  *** NLEVEL** subject codes at N using A/N using T year levels (consistent with the live N LEVEL→O LEVEL widen
  and REQ-006). The * OLEVEL subjects therefore do **NOT** map to pure-N-year-only levels; Appendix-A
  N using T columns are * NLEVEL only.

Still open for the founder/MOE expert (resolve at review — low-risk data):
- Should N using A subjects all extend to S5 NA, or only the exam-year subjects? (the table currently extends
  the cores + N-stream subjects to S5_NA)
- Combined Science vs Pure sciences by stream — exact split (e g. does N using A take Combined or a single Pure
  science).
- Higher Chinese range — left at full secondary (S1–S4 EXP / S1–S5 N using A) pending confirmation (H3 note).
- **S5 (H2):** confirmed asymmetric — only S5 NA exists; no Express/N using T 5th year. The seed must never
  emit S5 EXP/S5 NT (FK hard-fail).
