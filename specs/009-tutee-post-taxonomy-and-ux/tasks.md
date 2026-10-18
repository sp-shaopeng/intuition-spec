# Tasks — Tutee-post taxonomy + #205 + grid + Rate-Guide discoverability (009)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Implementation tasks broken from [design md](design.md). Each is small (≈2–10 min of focused work), names the
> **owning specialist**, traces to one or more **REQ-NNN**, lists the **exact files**, and a **per-task verification**.
> The Boss **dispatches the owning specialist** for each — the orchestrator does not build these itself. Check a box only
> when its evidence is captured under the private evidence directory for the requirement. **Two lanes:** backend (intuition-backend) and frontend
> (intuition-frontend) touch non-overlapping subtrees → they may run in parallel; builds **within** one subtree run
> **sequentially** (repo-concurrency rule).

## Governance gate (FIRST — precedes all build tasks)

| # | Task | Traces to | Owner | Verify | Status |
|---|------|-----------|-------|--------|--------|
| 1 | Scoped /team-review on this DRAFT (requirements-engineer, dba, software-engineer, frontend-expert, uiux-designer, qa-engineer, tech-lead, Technical Writer) + Boss audit → **founder approves the REQ-001/REQ-002 canonical taxonomy tables** → all Review & Sign-Off boxes checked → status: APPROVED. Confirm as conditions: subjects code/levels code unique-constraint (✅ both PK — 0005:67/79), the bitmask-unchanged BLOCKING test (REQ-005), 390px gate, the migrate-then-boot rule for any new enum value | all | Boss | sign-off boxes in spec md checked (incl. founder); status flipped to APPROVED before any build task | [ ] |

## Canonical-list artifact (Technical Writer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 2 | Capture the **founder-approved** canonical subject + level lists (the final REQ-001/REQ-002 tables, exactly as approved) as the build's source-of-truth doc | REQ-001, REQ-002 | Technical Writer | evidence/REQ-001/canonical-subjects md, evidence/REQ-002/canonical-levels md | the doc matches the approved spec tables; dba + frontend-expert seed/mirror from it | [ ] |

## backend lane — taxonomy seed + validation + data transfer object (dba · software-engineer) — sequential within the lane

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 3 | **(only if the approved list needs a new level group value)** Separate ALTER TYPE level group ADD VALUE … changeset, run In Transaction is no, ordered in the master changelog BEFORE the seed. Drafted list needs none → likely a no-op | REQ-002 | dba | 006 X-add-level-group-values implementation reference (+ master) | enum value committed before any reference; the documented operational check applies | [ ] |
| 4 | New seed changeset 0064-add-subjects-levels implementation reference (verify next free number against the master ledger) — two append records-only changesets, when a duplicate is found code leave the existing record unchanged, seeding EXACTLY the approved subjects + levels (with sort order); register in db changelog-master implementation reference; add the reviewed database rollback DELETEs | REQ-001, REQ-002 | dba | 0064-add-subjects-levels implementation reference, db changelog-master implementation reference | the documented operational check applies the changeset (Testcontainers); re-run is a no-op (idempotent) | [ ] |
| 5 | Inject Taxonomy Repository into Tutee Post Service; add a the validate taxonomy operation helper (mirror Profile Service implementation reference) → 422 UNKNOWN SUBJECT OR LEVEL; call it in create And Submit before save (and in update Post if that path sets the codes) | REQ-003 | software-engineer | Tutee Post Service implementation reference | unknown code → 422 (IT); valid → 200; spotless clean | [ ] |
| 6 | Add the max operation to Create Tutee Post Request num Students (and Update Tutee Post Request if present) | REQ-004 | software-engineer | Create Tutee Post Request implementation reference (+ update data transfer object) | num Students is 21 → 400 VALIDATION ERROR; 1–20 accepted | [ ] |
| 7 | backend tests: unknown subject → 422, unknown level → 422, valid → 200 (REQ-003); num Students is 21 → 400, =20/=6 → ok (REQ-004); seed-applied smoke (a seeded new code is FK-valid) | REQ-003, REQ-004, REQ-001/002 | qa-engineer | Tutee Post*IT implementation reference | the documented operational check green ≥97.5/97.5 | [ ] |

## frontend lane — mirror + #205 + grid + links (frontend-expert · uiux-designer) — sequential within the lane

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 8 | Mirror the approved codes into SUBJECT OPTIONS + LEVEL OPTIONS (code+label identical to the seed); extend LEVEL BAND (Pre-School, Polytechnic/University, Adult) + LEVEL BAND ORDER as typed constants; band each new level | REQ-001, REQ-002 | frontend-expert | taxonomy implementation reference | new options render in the pickers, grouped under their band; no magic strings | [ ] |
| 9 | frontend↔database sync guard test: SUBJECT OPTIONS/LEVEL OPTIONS value-set equals the committed expected approved set (drift fails continuous integration) | REQ-001, REQ-002 | qa-engineer · frontend-expert | taxonomy test implementation reference | the guard fails if a code is added on one side only; gate green | [ ] |
| 10 | #205: keep STUDENT COUNTS 1–5, add a "6 or more" option (sentinel 6+) revealing a numeric exact Students input; zod exact Students int ≥6 ≤20 required when sentinel; map payload num Students; edit-prefill (>5 → reopen); Review shows the exact number | REQ-004 | frontend-expert | Create Request implementation reference | reveal works; 1–5 unchanged; 6+ submits exact; out-of-range blocked | [ ] |
| 11 | **Grid transpose (REQ-005):** swap the two  .map loops to **day-rows × hour-cols** (DAYS map rows, HOUR SLOTS map cols); keep toggle/aria-pressed/aria-label/cell Key UNCHANGED; move the helper text next to the 1h/2h toggle. **Do NOT touch** scheduling implementation reference / build Bitmask | REQ-005 | frontend-expert | Availability Grid implementation reference | grid is time×day; helper text by the toggle; toggle/quick-set still work | [ ] |
| 12 | 390px: sticky day-label column (sticky left row-header + opaque bg) inside the existing overflow-x-auto; page does not scroll sideways; ≥44px targets; preserve the swipe-hint/fade | REQ-005 | frontend-expert · uiux-designer | Availability Grid implementation reference | 390px Playwright: no page horizontal overflow; day label stays visible while scrolling time | [ ] |
| 13 | **BLOCKING** golden-bitmask unit test: for a representative selection sweep, build Bitmask output is byte-for-byte identical before/after the transpose (pin expected integers); RTL toggle/quick-set parity + ≥44px | REQ-005 | qa-engineer | Availability Grid test implementation reference, Create Request test implementation reference | the test would FAIL if the encoding changed; passes after the transpose; gate green | [ ] |
| 14 | Three Rate-Guide links: FAQ answer linkifies "Rate Guide" →  in the **visible renderer only** (JSON-LD a string stays plain/valid); footer navigation gets a "Rate Guide" link; wizard Step-3 gets "see detailed rates →" after the Typical-rates box | REQ-006 | frontend-expert · uiux-designer | Faq implementation reference, Footer implementation reference, Create Request implementation reference | each link to=/rates; JSON parse of the JSON-LD still succeeds | [ ] |
| 15 | frontend tests: #205 (RTL — reveal, 1–5, 6+ exact, guards); REQ-006 (each link to=/rates, JSON-LD parses); taxonomy options render | REQ-004, REQ-006, REQ-001/002 | qa-engineer · frontend-expert | *.test implementation reference | the documented operational check green ≥97.5/88 | [ ] |

## Integration + evidence + ship

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 16 | Capture evidence per REQ under the private evidence directory for the requirement: seed changeset + the documented operational check apply log (REQ-001/002); 422 IT log + a live 422 body (REQ-003); #205 RTL log + a live 200 POST with num Students>5 (REQ-004); the golden-bitmask test + 390px screenshots (REQ-005); link tests + a Playwright run landing on /rates (REQ-006) | all | qa-engineer · frontend-expert · dba | files under the private evidence directory for the requirement | the reviewer opens each artifact against the running stack | [ ] |
| 17 | Gates green + local deploy + live convergence + push. backend: the documented operational check (≥97.5/97.5), the documented operational check. frontend: the documented operational check (≥97.5/88) + 390px gate. Redeploy local; Playwright live convergence (new subjects/levels selectable + 200, unknown→422, grid time×day + bitmask identical, 3 rate links resolve); e2e green; virtual machine re-confirm; then commit + push **per repo** on its current branch (concise message + Co-Authored-By trailer; stage real source/tests only) | all | dba · software-engineer · frontend-expert · devops-engineer | both repos | all gates pass at thresholds; live convergence + e2e green; pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces to | Owner | Verify | Status |
|---|------|-----------|-------|--------|--------|
| 18 | After build: /team-review (full panel) → Boss verdict on the built deliverable + evidence; fix every SEND-BACK condition in place and re-verify by re-reading/re-running; on SHIP set spec md status: HISTORICALLY MARKED IMPLEMENTED + confirm version/changelog + update requirements traceability matrix row | all | Boss | Boss SHIP; status HISTORICALLY MARKED IMPLEMENTED; evidence opened against the running system | [ ] |

## Notes

- **Spec-review gate first (Task 1)** — author does NOT self-approve; the founder must approve the canonical taxonomy
  tables before any seed/mirror is written. No build task starts before status: APPROVED.
- **Spec 008 ships first** (the two real bugs, fast lane); this spec follows.
- **BLOCKING (Task 13):** the 21-bit availability Bitmask for any selection is byte-for-byte unchanged after the grid
  transpose. scheduling implementation reference and Create Request implementation reference's build Bitmask/slots From Bitmask are untouched.
- **Migrate-then-boot (Task 3):** any new level group enum value ships in its own run In Transaction is no ALTER
  changeset committed before any seed/code references it. The drafted list needs none.
- **No magic strings:** application programming interface paths via application programming interface ROUTES; UNKNOWN SUBJECT OR LEVEL reuses the existing typed code; new
  LEVEL BAND values are typed constants.
- A task is DONE only when its REQ-NNN evidence is captured under the private evidence directory for the requirement and opened by the reviewer.
- Do D = both gates green + local deploy + Playwright live convergence + e2e green + virtual machine re-confirm + push (per project
  CLAUDE md). On completion set spec md status: HISTORICALLY MARKED IMPLEMENTED, confirm version/changelog, and update the requirements traceability matrix row.

## 2026-09-05 correction follow-up — implementation not verified

- [ ] **REQ-001/002 — dba + frontend-expert + quality assurance:** verify N LEVEL enum-before-seed order, all three subject mappings and frontend band. Replace count/spot parity checks with exact full-set and label/group comparisons; any applied-data correction uses a new migration, with spec025 compatibility preserved.
