# Requirements — Tutee-post taxonomy + number-of-students + grid + Rate-Guide discoverability (009)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin** acceptance
> criteria and the **exact evidence** that will prove each scenario. Negative/guard behaviours are their own scenarios.
> Source: founder GitHub issues **#203** (subjects), **#204** (levels), **#205** (number of students), **#202** (Rate
> Guide hard to find), and the **non-bug user experience parts of #206** (time-slot grid too tall; Rate-Guide link in the wizard) —
> filed 2026-06-21 against the tutee post creation wizard — plus the Boss's pre-spec synthesis plan
> (twinkly-percolating-glade md, "Spec B"). The two real **bugs** of #206 (Step-4 auto-submit, Step-3 budget
> validation) are **out of scope** here — they are Spec 008.

## Scope note (what already exists — grounded against the real code, 2026-06-21)

- **Taxonomy is database-seeded with an frontend mirror — there is no taxonomy application programming interface.** Backend source of truth: the subjects and
  levels reference tables, seeded in 0005-create-tutor-posts implementation reference
  (subjects 20 rows ; levels 16 rows ), extended by 0052-add-secondary-na-levels implementation reference (S1_NA/S2_NA).
  Both tables are the varchar operation (0005:67, 0005:79) → when a duplicate is found code leave the existing record unchanged is valid.
  tutee posts subject code/level code are the subjects operation
  (0006-create-tutee-posts implementation reference). frontend mirror: taxonomy implementation reference — SUBJECT OPTIONS
  (20 codes, ) and LEVEL OPTIONS (18 banded codes, ). The two lists are **manually kept in sync** (the
  file header at  says so). Any seed addition must add the matching frontend code/label, or the picker won't offer it.
- **level group is a PG ENUM** = KINDERGARTEN, PSLE, O LEVEL, A LEVEL, IB, POLY, UNI, ADULT (0005:48–50). No
  PRE SCHOOL/N LEVEL value exists; POLY/UNI/ADULT already do.
- **Tutee Post Service create And Submit (Tutee Post Service implementation reference) does NOT validate the subject/level
  codes.** It builds Tutee Post straight from the subject code operation/the level code operation () and saves — an unknown
  code hits the subjects operation FK and throws a **500**, not a clean 422. The validation
  pattern that returns a clean **422 UNKNOWN SUBJECT OR LEVEL** already exists in Profile Service (, via
  the existing subject codes operation); Tutee Post Service does **not** inject
  Taxonomy Repository today.
- **Number of students:** frontend has a fixed STUDENT COUNTS 1,2,3,4,5 selection control (Create Request implementation reference,
  rendered ), zod the string operation (), submitted as the parse int operation ().
  backend data transfer object Create Tutee Post Request num Students has the min operation only — **no upper bound**
  (Create Tutee Post Request implementation reference). database require num students is at least 1.
- **Time-slot grid:** Availability Grid implementation reference renders **hour-rows × day-cols** — HOUR SLOTS map(hour  leads to 
  … DAYS map using day leads to … (), ~14 hour rows → a tall page. Each cell's identity is
  the cell key operation (availability Slots implementation reference → day-and-hour identity), **independent of render order**. The tutee
  wizard encodes the grid's hour cell-keys into the **legacy 21-bit 3-period bitmask** via build Bitmask
  (Create Request implementation reference, mask | 1 day Idx*3 + slot Idx), submitted as availability Bitmask (). Helper
  text "Tap an hour to toggle it…" sits at the **bottom** of the grid (Availability Grid implementation reference); the 1h/2h
  granularity toggle is at the **top** ().
- **Rate Guide already exists:** Rate Guide implementation reference, route /rates live (App implementation reference). #202 is
  **link-wiring**, not page-building. Today the FAQ answer mentions "our Rate Guide page" as **plain text**
  (Faq implementation reference), the Footer nav has no rate link (Footer implementation reference), and the wizard's Step-3 "Typical rates" box
  (Create Request implementation reference) has no link out. The FAQ JSON-LD FAQ Page block is generated from the same FAQS
  q/a **strings** (Faq implementation reference) — so the a value must stay a plain renderable string for schema validity.

---

## REQ-001 — Subject taxonomy expansion (#203)

- **Type:** functional + data (reference-data seed) · **Priority:** Must
- **Source:** Founder issue #203 ("subject dropdown missing options — Pre-School, N-Level, A-Level Econs, O-Level POA,
  IB, Poly/Uni, Adult…"); plan Spec B / B1.
- **EARS:** *When* a tutee opens Step-1 of the create-request wizard, *the system shall* offer the full canonical
  Singapore subject list (the seeded subjects reference table mirrored in SUBJECT OPTIONS), *so that* common tuition
  subjects (e g. A-Level Economics, O-Level Principles of Accounts, IB subjects, Pre-School, Poly/Uni, Adult) can be
  selected and a post submitted.
- **Rationale / value:** Today only 20 subjects exist; real demand spans many more. A missing subject blocks the post
  entirely (the field is required). Expanding the canonical list is the direct fix.
- **Owner (specialist):** requirements-engineer (drafts the canonical list — this doc) · dba (seed changeset) ·
  frontend-expert (frontend mirror) · Technical Writer (the canonical-list doc artifact)

### Drafted canonical subject additions — **FOUNDER-APPROVE at spec-review**

> Proposal only. The build seeds **exactly the founder-approved set**. Codes follow the existing
> ^ A-Z0-9 + / SUBJECT LEVELSUFFIX convention; level group reuses an existing enum value (no new enum needed).
> Existing 20 rows are untouched (the new changeset is append records-only with when a duplicate is found code leave the existing record unchanged).

| code | label (name) | level group | issue driver |
|---|---|---|---|
| MATH PRESCHOOL | Numeracy (Pre-School) | KINDERGARTEN | #203 Pre-School |
| ENG PRESCHOOL | Literacy / English (Pre-School) | KINDERGARTEN | #203 Pre-School |
| CHN PRESCHOOL | Chinese (Pre-School) | KINDERGARTEN | #203 Pre-School |
| PHONICS PRESCHOOL | Phonics (Pre-School) | KINDERGARTEN | #203 Pre-School |
| MALAY PSLE | Malay (PSLE) | PSLE | SG MTL norm |
| TAMIL PSLE | Tamil (PSLE) | PSLE | SG MTL norm |
| HCHN PSLE | Higher Chinese (PSLE) | PSLE | SG MTL norm |
| MALAY OLEVEL | Malay (O-Level) | O LEVEL | SG MTL norm |
| TAMIL OLEVEL | Tamil (O-Level) | O LEVEL | SG MTL norm |
| HCHN OLEVEL | Higher Chinese (O-Level) | O LEVEL | SG MTL norm |
| POA OLEVEL | Principles of Accounts (O-Level) | O LEVEL | **#203 O-Level POA** |
| HIST OLEVEL | History (O-Level) | O LEVEL | SG humanities |
| GEOG OLEVEL | Geography (O-Level) | O LEVEL | SG humanities |
| LIT OLEVEL | Literature (O-Level) | O LEVEL | SG humanities |
| SCI OLEVEL | Combined Science (O-Level) | O LEVEL | SG norm |
| SS OLEVEL | Social Studies (O-Level) | O LEVEL | SG norm |
| ENG NLEVEL | English (N-Level) | N LEVEL | **#203 N-Level** |
| MATH NLEVEL | Mathematics (N-Level) | N LEVEL | **#203 N-Level** |
| SCI NLEVEL | Science (N-Level) | N LEVEL | **#203 N-Level** |
| ECON ALEVEL | H2 Economics (A-Level) | A LEVEL | **#203 A-Level Econs** |
| MATH H1 ALEVEL | H1 Mathematics (A-Level) | A LEVEL | SG A-Level norm |
| ECON H1 ALEVEL | H1 Economics (A-Level) | A LEVEL | SG A-Level norm |
| HIST ALEVEL | H2 History (A-Level) | A LEVEL | SG humanities |
| GEOG ALEVEL | H2 Geography (A-Level) | A LEVEL | SG humanities |
| LIT ALEVEL | H2 Literature (A-Level) | A LEVEL | SG humanities |
| ENG IB | English (IB) | IB | **#203 IB** |
| ECON IB | Economics (IB) | IB | **#203 IB** |
| BIO IB | Biology (IB) | IB | **#203 IB** |
| MATH POLY | Mathematics / Engineering Maths (Poly) | POLY | **#203 Poly/Uni** |
| ACCT POLY | Accounting (Poly) | POLY | **#203 Poly/Uni** |
| MATH UNI | University Mathematics | UNI | **#203 Poly/Uni** |
| STATS UNI | University Statistics | UNI | **#203 Poly/Uni** |
| ENG ADULT | English (Adult / Conversational) | ADULT | **#203 Adult** |
| CHN ADULT | Chinese (Adult / Conversational) | ADULT | **#203 Adult** |

- **Acceptance criteria (Gherkin):**


- Scenario: A newly-seeded subject is selectable and a post submits.
- Given the approved subject codes are seeded into the subjects reference table.
- And SUBJECT OPTIONS mirrors them with matching code + label.
- When a tutee opens Step-1 of the create-request wizard.
- Then the new subject (e g. "H2 Economics (A-Level)") appears in the subject picker.
- And selecting it and completing the wizard issues a POST that returns 200.
- And the created post's subject code equals the selected code.
- Scenario: The seed changeset is idempotent (re-runnable).
- Given the new subject codes are already present.
- When the changeset is applied again.
- Then no duplicate-key error occurs (when a duplicate is found (code) leave the existing record unchanged) and no row is duplicated.
- Scenario: frontend mirror stays in sync with the database (guard).
- Given SUBJECT OPTIONS in taxonomy implementation reference.
- Then every value posted by the picker exists as a subjects code after the seed.
- And the unit guard test that compares the frontend list against the seeded set passes.


- **Required evidence (up front):** the approved canonical list committed as a doc under REQ-001 implementation reference; the new
  Liquibase changeset diff + the documented operational check log showing it applies (Testcontainers); a Playwright
  screenshot of the Step-1 picker showing a new subject selectable + a live POST returning 200 with the new
  subject code; the frontend↔database sync guard test run log.
- **Dependencies:** founder approval of this table (spec-review). REQ-003 (422 guard) covers the unknown-code path.

## REQ-002 — Level taxonomy expansion (#204)

- **Type:** functional + data (reference-data seed) · **Priority:** Must
- **Source:** Founder issue #204 ("level dropdown missing options — Sec-5 N using A, Pre-School, Poly, Uni…"); plan Spec B/B1.
- **EARS:** *When* a tutee opens Step-1 of the create-request wizard, *the system shall* offer the full canonical level
  list (the seeded levels reference table mirrored in LEVEL OPTIONS), *so that* levels such as Pre-School, Secondary
  5 (N using A), Polytechnic, and University can be selected.
- **Rationale / value:** A missing level blocks the post (required field). Sec-5 N using A is a real SG path (N using A students
  sit O-Levels in a 5th year); Pre-School / Poly / Uni / Adult tuition are real demand segments.
- **Owner (specialist):** requirements-engineer (canonical list) · dba (seed + any enum migrate-then-boot) ·
  frontend-expert (frontend mirror + LEVEL BAND extension) · Technical Writer (doc)

### Drafted canonical level additions — **FOUNDER-APPROVE at spec-review**

> Proposal only. level group reuses **existing** enum values (KINDERGARTEN for Pre-School, O LEVEL for the N-Level
> 5th year, POLY/UNI/ADULT already exist) — **no ALTER TYPE is required by this drafted list.** The frontend mirror
> also needs new LEVEL BAND values (Pre-School, Poly/Uni, Adult) since LEVEL OPTIONS rows are banded
> (taxonomy implementation reference). sort order keeps the per-group ordering convention from 0005/0052.

| code | label (name) | level group | sort order | frontend band (new) | issue driver |
|---|---|---|---|---|---|
| N1 PRESCHOOL | Nursery 1 | KINDERGARTEN | 10 | Pre-School | #204 Pre-School |
| N2 PRESCHOOL | Nursery 2 | KINDERGARTEN | 20 | Pre-School | #204 Pre-School |
| K1 PRESCHOOL | Kindergarten 1 | KINDERGARTEN | 30 | Pre-School | #204 Pre-School |
| K2 PRESCHOOL | Kindergarten 2 | KINDERGARTEN | 40 | Pre-School | #204 Pre-School |
| S5 NA | Secondary 5 (Normal Acad) | O LEVEL | 70 | Secondary (O-Level) | **#204 Sec-5 N using A** |
| POLY1 | Polytechnic Year 1 | POLY | 10 | Polytechnic / University | **#204 Poly** |
| POLY2 | Polytechnic Year 2 | POLY | 20 | Polytechnic / University | **#204 Poly** |
| POLY3 | Polytechnic Year 3 | POLY | 30 | Polytechnic / University | **#204 Poly** |
| UNI | University (Undergraduate) | UNI | 10 | Polytechnic / University | **#204 Uni** |
| ADULT | Adult Learner | ADULT | 10 | Adult | #204 Adult (pairs ADULT subjects) |

- **Acceptance criteria (Gherkin):**


- Scenario: A newly-seeded level is selectable and a post submits.
- Given the approved level codes are seeded into the levels reference table.
- And LEVEL OPTIONS mirrors them with matching code + label + band.
- When a tutee opens Step-1 of the create-request wizard.
- Then the new level (e g. "Secondary 5 (Normal Acad)") appears in the level picker, grouped under its band.
- And selecting it and completing the wizard issues a POST that returns 200.
- And the created post's level code equals the selected code.
- Scenario: No new enum value is referenced before it is committed (migrate-then-boot guard).
- Given the drafted list maps every new level onto an existing level group enum value.
- Then the seed changeset references only KINDERGARTEN/O LEVEL/POLY/UNI/ADULT (all pre-existing).
- And IF the approved list ever needs a new level group label.
- Then that label is added in a separate ALTER TYPE ... ADD VALUE changeset (run In Transaction is no) committed before any seed/code references it.
- Scenario: Seed changeset is idempotent (re-runnable).
- Given the new level codes are already present.
- When the changeset is applied again.
- Then no duplicate-key error occurs and no row is duplicated.


- **Required evidence (up front):** approved canonical list under REQ-002 implementation reference; changeset diff + the documented operational check
  apply log; Playwright screenshot of the level picker showing the new level under its band + a 200 POST carrying the new
  level code; the frontend↔database level sync guard test run.
- **Dependencies:** founder approval; if a new level group is wanted (open question #2), that ALTER changeset precedes
  the seed.

## REQ-003 — Taxonomy validation: unknown subject/level returns 422, not 500 (bug-prevention)

- **Type:** functional (defensive / error-contract) · **Priority:** Must
- **Source:** plan Spec B / B1 bug-prevention note; grounded at Tutee Post Service implementation reference (no code validation) vs
  the FK at 0006-create-tutee-posts implementation reference.
- **EARS:** *When* a create-request is submitted with a subject Code or level Code not present in the subjects/
  levels reference tables, *the system shall* reject it with **web request protocol 422 UNKNOWN SUBJECT OR LEVEL** *before* the
  insert reaches the foreign key, *so that* a stale frontend mirror (or a bad client) yields a clean, typed error instead of a
  500 from the database FK violation.
- **Rationale / value:** The taxonomy is duplicated (database seed + frontend constants, no application programming interface) so drift is possible; today drift →
  500. Mirroring the existing Profile Service guard (276–284) degrades drift to a clean 422 and protects the
  contract regardless of the taxonomy-application programming interface follow-up.
- **Owner (specialist):** software-engineer (inject Taxonomy Repository into Tutee Post Service, validate before save)
  · qa-engineer (the unknown-code IT)
- **Acceptance criteria (Gherkin):**


- Scenario: Unknown subject code is rejected with a typed 422.
- Given a create-request payload with subject Code = "NOT A REAL CODE" and a valid level Code.
- When POST tutee posts interface operation is called.
- Then the response is 422 with error code UNKNOWN SUBJECT OR LEVEL.
- And no tutee posts row is created.
- Scenario: Unknown level code is rejected with a typed 422.
- Given a create-request payload with a valid subject Code and level Code = "NOT A REAL CODE".
- When POST tutee posts interface operation is called.
- Then the response is 422 with error code UNKNOWN SUBJECT OR LEVEL.
- And no tutee posts row is created.
- Scenario: Valid codes still succeed (no regression).
- Given a create-request with a seeded subject Code and level Code.
- When POST tutee posts interface operation is called.
- Then the response is 200 and the post is created.


- **Required evidence (up front):** an integration test (ci-no-lab profile / Testcontainers) asserting unknown
  subject → 422 UNKNOWN SUBJECT OR LEVEL, unknown level → 422, valid → 200; the documented operational check run
  log; a captured live response (Playwright/Postman) showing the 422 body — committed under REQ-003 implementation reference.
- **Dependencies:** none (independent of REQ-001/002, but it is what makes a missed mirror-sync safe).

## REQ-004 — Number of students: "1–5 + 6 or more" with an exact-count reveal (#205)

- **Type:** functional · **Priority:** Should
- **Source:** Founder issue #205 + founder decision this session ("keep the 1–5 dropdown + a '6 or more' option that
  reveals an exact count"); plan Spec B / B2.
- **EARS:** *When* a tutee selects "6 or more" in the number-of-students control, *the system shall* reveal a numeric
  input requiring an exact integer count between 6 and 20 inclusive, *and* the backend shall accept that exact count
  (the max operation added to num Students), *so that* large-group requests capture the real headcount.
- **Rationale / value:** The fixed 1–5 select can't express a 6+ group; an unbounded number is abuse-prone. "6 or more"
  → exact count (≥6, ≤20) is the founder's chosen balance. Today the data transfer object has the min operation but **no @Max** — an arbitrarily
  large count would be accepted; this REQ also closes that.
- **Owner (specialist):** frontend-expert (the reveal + zod) · software-engineer (the max operation on the data transfer object) · qa-engineer
- **Acceptance criteria (Gherkin):**


- Scenario: Selecting "6 or more" reveals an exact-count input.
- Given the tutee is on Step-1 with the number-of-students control.
- When they choose "6 or more".
- Then a numeric "exact number of students" input appears.
- And it is required, integer, minimum 6, maximum 20.
- Scenario: 1–5 selection submits unchanged.
- Given the tutee chooses "3" from the 1–5 options.
- Then no exact-count input is shown.
- And the submitted num Students is 3.
- Scenario: 6+ exact count submits the exact number.
- Given the tutee chooses "6 or more" and enters 8.
- When they submit the request.
- Then the submitted num Students is 8 and the POST returns 200.
- Scenario: Out-of-range exact count is rejected (guard, both ends).
- Given the tutee chose "6 or more".
- When they enter 5 (below 6) or 21 (above 20) or a non-integer.
- Then the frontend shows a validation error and does not advance.
- And if such a payload reaches the backend, num Students > 20 returns 400 VALIDATION ERROR (@Max using 20).


- **Required evidence (up front):** RTL tests — reveal-on-"6 or more", 1–5 unchanged, 6+ exact submit, and the
  below-6/above-20/non-integer guards; a backend validation test for the max operation (num Students is 21 → 400 VALIDATION ERROR); a
  Playwright screenshot of the revealed input + a live 200 POST with num Students > 5 — committed under
  REQ-004 implementation reference.
- **Dependencies:** none.

## REQ-005 — Time-slot grid transpose to time×day + helper-text move (#206 user experience) — **bitmask byte-for-byte unchanged (BLOCKING)**

- **Type:** functional (user interface layout) · **Priority:** Should
- **Source:** Founder issue #206 (non-bug user experience part — "time-slot grid is too tall; make it time × day and move the helper
  text"); plan Spec B / B3.
- **EARS:** *When* the availability grid renders, *the system shall* lay it out as **day-rows × hour-columns** (x-axis =
  time, y-axis = day) instead of hour-rows × day-columns, *and* relocate the helper text to sit beside the 1h/2h
  granularity toggle, *while* the encoded 21-bit availability Bitmask produced for any given selection remains
  **byte-for-byte identical** to the pre-transpose encoding.
- **Rationale / value:** The current ~14 hour-rows make the page very tall (the founder's complaint). Transposing to
  ~14 hour-columns × 7 day-rows is far shorter vertically. **The data must not move** — cell identity is
  the cell key operation (availability Slots implementation reference), independent of render order, and the wizard's build Bitmask
  (Create Request implementation reference) keys off the period index for hour operation; transposing only the  rows/cols
  changes none of that. This is the single highest-risk REQ for a silent regression, so the bitmask-unchanged test is
  **BLOCKING**.
- **Owner (specialist):** frontend-expert (the transpose + helper-text move) · uiux-designer (390px legibility) ·
  qa-engineer (the BLOCKING bitmask-unchanged test + 390px gate)
- **Acceptance criteria (Gherkin):**


- Scenario: Grid renders as day-rows × hour-columns.
- Given a tutee is on the Schedule step.
- Then the grid has one row per day (Mon…Sun) and one column per selectable hour.
- And the helper text "Tap an hour to toggle it…" appears next to the 1h/2h toggle, not below the grid.
- Scenario: BLOCKING — the encoded bitmask is byte-for-byte unchanged after the transpose.
- Given a fixed set of selected cells (e g. Mon 19:00, Tue 14:00, Sat 10:00).
- When build Bitmask is computed before and after the transpose for the identical selection.
- Then the two bitmask integers are exactly equal.
- And a representative sweep of selections all produce identical bitmasks across the change.
- Scenario: 390px mobile has no horizontal page overflow and keeps usable targets.
- Given the viewport is 390px wide.
- Then the grid is wrapped in a horizontal-scroll container with a sticky day-label column.
- And no part of the page overflows the 390px viewport horizontally (the page itself does not scroll sideways).
- And every toggle cell is at least 44×44px.
- Scenario: Toggling and quick-set shortcuts still work after the transpose.
- Given the transposed grid.
- When the tutee taps a cell or a quick-set shortcut.
- Then the same cells select/deselect as before and aria-pressed reflects state.


- **Required evidence (up front):** the **BLOCKING** unit test asserting build Bitmask output is identical for a
  representative selection sweep before/after (a golden-bitmask test); a 390px Playwright screenshot showing no page
  horizontal overflow + the sticky day column + the time×day layout + helper text by the toggle; an RTL test for
  toggle/quick-set parity and ≥44px targets — committed under REQ-005 implementation reference.
- **Dependencies:** none. **Must not touch** scheduling implementation reference (DAYS/HOUR SLOTS/cell Key encoding) or
  Create Request implementation reference's build Bitmask/slots From Bitmask.

## REQ-006 — Rate-Guide discoverability: three links to the existing /rates page (#202 + #206-3)

- **Type:** functional (navigation / IA) · **Priority:** Should
- **Source:** Founder issue #202 ("FAQ links to a Rate Guide page users can't find") + #206-3 (a rate link in the
  wizard); plan Spec B / B4. The page **already exists** (/rates, App implementation reference).
- **EARS:** *When* a user reads the FAQ "typical rates" answer, browses the Footer, or reaches the wizard's Step-3 budget
  step, *the system shall* present a working link to the existing /rates Rate Guide page, *so that* the page founder
  authored is actually reachable.
- **Rationale / value:** The content exists but is undiscoverable — the FAQ names it in plain text, the Footer has no
  link, the budget step (the exact moment a user wonders about rates) has no link. Three wired links close #202.
- **Owner (specialist):** frontend-expert (all three links) · uiux-designer (placement)
- **Acceptance criteria (Gherkin):**


- Scenario: FAQ answer links to the Rate Guide and the JSON-LD stays valid.
- Given the FAQ "What are the typical tuition rates?" answer.
- Then it renders a link to /rates ("Rate Guide").
- And the FAQPage JSON-LD main Entity answer text remains a valid plain string (schema org unbroken).
- Scenario: Footer links to the Rate Guide.
- Given any page with the site Footer.
- Then the Footer navigation includes a "Rate Guide" link to /rates.
- Scenario: Wizard Step-3 links to the Rate Guide.
- Given the tutee is on Step-3 (Venue & Budget), after the "Typical rates" box.
- Then a "see detailed rates →" link to /rates is present.
- Scenario: All three links resolve to the Rate Guide page (guard).
- When each link is followed.
- Then the /rates Rate Guide page renders (no 404).


- **Required evidence (up front):** RTL/render tests asserting each of the three links has to=/rates, plus an
  assertion the FAQ JSON-LD still parses (the a string is intact); a Playwright run clicking each link and landing on
  /rates — committed under REQ-006 implementation reference.
- **Dependencies:** none. The page is reused as-is (no Rate Guide change).

---

## Traceability summary

| REQ | One-line | Issue | Type | Priority | Owner using s |
|---|---|---|---|---|---|
| REQ-001 | Subject taxonomy expansion — seed canonical SG subjects + frontend mirror (founder-approved list) | #203 | functional+data | Must | requirements-engineer · dba · frontend-expert |
| REQ-002 | Level taxonomy expansion — seed canonical levels + frontend mirror/band (founder-approved; no new enum needed) | #204 | functional+data | Must | requirements-engineer · dba · frontend-expert |
| REQ-003 | Unknown subject/level → 422 UNKNOWN SUBJECT OR LEVEL (mirror Profile Service) not database-FK 500 | (B1 guard) | functional | Must | software-engineer · qa-engineer |
| REQ-004 | Number of students "1–5 + 6 or more" reveal (≥6,≤20) + data transfer object the max operation | #205 | functional | Should | frontend-expert · software-engineer · qa-engineer |
| REQ-005 | Time-slot grid transpose to time×day + helper-text move — **bitmask byte-for-byte unchanged (BLOCKING)** + 390px | #206 | functional | Should | frontend-expert · uiux-designer · qa-engineer |
| REQ-006 | Rate-Guide discoverability — 3 links (FAQ, Footer, Step-3) to existing /rates; JSON-LD intact | #202/#206 | functional | Should | frontend-expert · uiux-designer |

## Constraints (recorded for the build + the done-gate)

- **Two lanes.** backend (dba: Liquibase seed [+ any enum ALTER]; software-engineer: Tutee Post Service 422 + data transfer object @Max) and
  frontend (frontend-expert: taxonomy implementation reference mirror, #205, grid, links). The two lanes touch non-overlapping subtrees so they may
  proceed in parallel; builds **within** a subtree run sequentially (repo-concurrency rule).
- **Migrate-then-boot.** Any new level group enum value (only if the approved list needs one) ships in its own
  ALTER TYPE level group ADD VALUE changeset with run In Transaction is no, committed **before** any seed/code
  references the label. The drafted list needs **none**.
- **when a duplicate is found code leave the existing record unchanged** for all reference-data inserts (idempotent; both tables PK on code).
- **Soft-delete only** convention applies to any future deactivation (is active flag exists on both tables); this spec
  only INSERTs, never deletes a taxonomy row.
- **Bitmask invariant (REQ-005) is BLOCKING** — the 21-bit availability Bitmask for any selection is byte-for-byte
  unchanged; scheduling implementation reference and Create Request implementation reference's build Bitmask are untouched.
- **Gates:** frontend the documented operational check at **≥97.5% line / ≥88% branch** (Vitest) + **390px
  no-overflow**; backend the documented operational check at **≥97.5% line / ≥97.5% branch** (JaCoCo), Docker up for
  Testcontainers, the documented operational check before commit.
- **No magic strings:** application programming interface paths via application programming interface ROUTES; the UNKNOWN SUBJECT OR LEVEL error code is a typed constant on both
  ends; new LEVEL BAND values are typed constants, not raw strings.
- **Live convergence is mandatory before SHIP:** on the deployed local stack — the new subjects/levels are selectable and
  a post submits 200; an unknown code returns 422; the grid renders time×day with an identical bitmask; the three rate
  links resolve to /rates; e2e green. Then re-confirm on the virtual machine (the issues were filed against the virtual machine).

## 2026-09-05 taxonomy parity clarification (REQ-001/002)

The approved distinct N LEVEL grouping applies to all three N-Level subjects and corresponding frontend band/mapping; it is not conditional. Validation compares the complete approved code sets and their code→label/group mappings between frontend options and Liquibase seed data. Include negative missing/extra/misgrouped-item cases; count equality or a spot-set does not meet the existing exact parity requirement. Later subject-level compatibility rules in spec025 remain authoritative; this correction does not invent a new compatibility matrix or approve every Cartesian pair.
