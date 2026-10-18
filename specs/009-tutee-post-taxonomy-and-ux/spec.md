> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 009
slug: tutee-post-taxonomy-and-ux
title: Tutee-post taxonomy expansion + number-of-students + time-slot grid + Rate-Guide discoverability
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED             # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: requirements-engineer   # cross-cutting product/taxonomy spec; build owners per-REQ are frontend-expert / dba / software-engineer
created: 2026-06-21
updated: 2026-09-05
---

> **2026-09-05 requirements correction (documentation only).** The approved N LEVEL group is folded into the canonical table/design and the private implementation folder parity guard now requires full sets and mappings.
> The status and shipment/evidence entries below record the earlier implementation. This revision does not
> certify that the corrected contract is implemented, tested, or deployed; follow-up verification remains open.


# Tutee-post taxonomy expansion + number-of-students + time-slot grid + Rate-Guide discoverability

**Spec B** of the founder #206 batch (the non-bug user experience + taxonomy parts; the two real bugs are the separate, fast-tracked
**Spec 008**). Covers GitHub issues **#203** (subject dropdown missing options), **#204** (level dropdown missing
options), **#205** (number-of-students input), **#202** (FAQ links to a Rate-Guide page users cannot find), and the
**non-bug user experience parts of #206** (time-slot grid too tall → transpose + move helper text; Rate-Guide link in the wizard).

All five issues target the **tutee post creation wizard** (/tutee/posts/new,
Create Request implementation reference) and the public **Rate Guide** (/rates, which already exists).
The spec spans both lanes:

- **backend (dba + software-engineer):** expand the database-seeded subjects/levels reference tables (a new Liquibase
  changeset; subjects code/levels code are both record identifier so when a duplicate is found code leave the existing record unchanged is valid — confirmed
  0005-create-tutor-posts implementation reference); add **taxonomy validation** to Tutee Post Service create And Submit so an unknown
  subject/level code returns a clean **422 UNKNOWN SUBJECT OR LEVEL** instead of the current database-FK **500**; widen
  Create Tutee Post Request num Students with the max operation.
- **frontend (frontend-expert):** mirror the new subject/level codes into taxonomy implementation reference; add the **"6 or more"**
  number-of-students option that reveals an exact-count input (#205); **transpose** Availability Grid from hour-rows ×
  day-cols to **day-rows × hour-cols** and move its helper text — **visual-only, the 21-bit availability bitmask is
  byte-for-byte unchanged**; wire **three Rate-Guide links** (FAQ, Footer, wizard Step-3).

> **FOUNDER APPROVAL REQUIRED (at spec-review).** The canonical subject + level additions in
> [requirements md](requirements.md) (REQ-001 / REQ-002) are a **drafted proposal** based on the issue examples + SG
> norms. The founder approves the exact list at the spec-review gate; the build seeds **exactly the approved set**, no
> more, no less.

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC + the drafted canonical
  taxonomy tables (each REQ-NNN), with up-front required evidence.
- **Design:** [design md](design.md) — taxonomy seeding (migrate-then-boot for any new enum value), the 422 validation
  mirror, the grid-transpose-without-touching-the-bitmask approach, the #205 reveal pattern, link wiring.
- **Tasks:** [tasks md](tasks.md) — small build tasks, each traced to a REQ-NNN + owning specialist.
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario.

## Status

DRAFT — author does NOT self-approve. The Boss convenes a scoped team-review on this DRAFT (tasks md Task 1); only
when every Review & Sign-Off box below is checked — **including the founder's approval of the canonical taxonomy list** —
does the Boss flip status to APPROVED and clear the build. **Spec 008 ships first** (the two real bugs, fast lane);
this spec follows.

## Review & Sign-Off

- [x] Author — requirements + canonical taxonomy draft (requirements-engineer, lead) — SIGN-OFF / 2026-06-21
- [x] Counterpart — Liquibase seed + enum migrate-then-boot + code unique-constraint check (dba) — SIGN-OFF (code is PK; N LEVEL ALTER-first) / 2026-06-21
- [x] Counterpart — Tutee Post Service 422 validation + data transfer object the max operation (software-engineer) — SIGN-OFF / 2026-06-21
- [x] Counterpart — taxonomy implementation reference mirror + #205 reveal + grid transpose + 3 rate links (frontend-expert) — SIGN-OFF / 2026-06-21
- [x] Counterpart — form/user experience flow, grid legibility, 390px cognitive load (uiux-designer) — SIGN-OFF (sticky day-col) / 2026-06-21
- [x] Counterpart — BLOCKING bitmask-unchanged test, unknown-code→422 test, #205 tests, 390px gate (qa-engineer) — SIGN-OFF / 2026-06-21
- [x] Counterpart — delivery vs. quality, the private implementation folder lane sequencing (tech-lead) — SIGN-OFF / 2026-06-21
- [x] Technical Writer (docs + the canonical-list doc) — SIGN-OFF / 2026-06-21
- [x] **Founder — approves the canonical subject + level additions** — APPROVED VIA BOSS/TEAM DELEGATION (founder: "boss review with the team, approve if good") + **N-Level = distinct N LEVEL group** (founder ruling) / 2026-06-21
- [x] Boss (final sign-off → flips status to APPROVED) — APPROVED / 2026-06-21

## Open questions (resolve at spec-review — flagged, not guessed)

1. **Founder taxonomy approval (REQ-001 / REQ-002).** The drafted canonical subject + level lists need founder sign-off
   before the seed changeset is written. The build seeds exactly the approved set.
2. **New level group enum values.** The PG enum today is KINDERGARTEN, PSLE, O LEVEL, A LEVEL, IB, POLY, UNI, ADULT
   (0005-create-tutor-posts implementation reference). The drafted level additions map onto **existing** groups (KINDERGARTEN for
   Pre-School, O LEVEL for N-Level secondary years, POLY/UNI/ADULT already exist). **No new enum value is
   required by the drafted list.** IF the founder's final list needs a label outside these (e g. a distinct N LEVEL
   group separate from O LEVEL), that label is added via a separate ALTER TYPE level group ADD VALUE changeset with
   run In Transaction is no, committed **before** any code/seed references it (migrate-then-boot). Confirm at review
   whether KINDERGARTEN/O LEVEL reuse is acceptable or a new group is wanted.
3. **Taxonomy drift (deferred, NOT this batch).** Subjects/levels are duplicated (database seed + frontend constants, no application programming interface). A
   read operation for taxonomy endpoint would remove the manual sync; filed as a follow-up. This spec keeps the duplicated
   pattern to stay focused — but adds the **422 guard** so a missed sync degrades to a clean error, not a 500.

## Changelog

- 1.2 (2026-06-22) HISTORICALLY MARKED IMPLEMENTED (dev) — built backend+frontend, both gates Boss-reverified (backend 2009 tests ≥97.5/97.5; frontend 2442 tests,
  98.13% line / 92.62% branch). Migrations 0064 (ALTER TYPE level group ADD VALUE N LEVEL run In Transaction is no,
  ordered BEFORE the seed) + 0064 seed (34 subjects + 10 levels; N-Level subjects → N LEVEL). Tutee Post Service 500→422
  UNKNOWN SUBJECT OR LEVEL (create + edit). the max operation num Students. frontend: taxonomy mirror + N LEVEL/Pre-School/Poly-Uni/
  Adult bands; #205 "6 or more"→exact (6–20); grid transposed to time×day + sticky day-column + helper-text move
  (**golden-bitmask test GREEN — build Bitmask/scheduling implementation reference untouched**); 3 rate links (FAQ JSON-LD kept valid).
  **Live convergence:** redeployed local — 0064 applied on PG (N LEVEL enum present, subjects 20→54, levels 18→28, clean
  boot); frontend redeployed (grid/#205/links live). virtual machine deploy + 390px Playwright screenshot pending. status APPROVED→HISTORICALLY MARKED IMPLEMENTED.
- 1.1 (2026-06-21) APPROVED — scoped team-review passed (dba, software-engineer, frontend-expert, uiux-designer,
  qa-engineer, tech-lead, Technical Writer + Boss). **Taxonomy list APPROVED** — founder delegated content approval to
  the Boss+team ("boss review with the team, approve if good"); the ~34 subjects / ~10 levels are sound for SG (a v1,
  extensible later). **MODIFIED REQ-002 — N-Level is now a DISTINCT N LEVEL level group** (founder ruling, vs the
  draft's O LEVEL mapping): build adds ALTER TYPE level group ADD VALUE N LEVEL in its own run In Transaction is no
  changeset ordered BEFORE the seed (migrate-then-boot), remaps the N-Level subjects/levels to N LEVEL, and adds an
  N LEVEL frontend LEVEL BAND. Build conditions: golden-bitmask test BLOCKING; frontend↔database sync-guard; 500→422 validation; 390px
  sticky-day-column. Builds AFTER spec 008 ships (shared Create Request implementation reference). status DRAFT→APPROVED.
- 1.0 (2026-06-21) initial draft — ADDED REQ-001 (subject taxonomy expansion, #203, with a drafted canonical SG subject
  list for founder approval), REQ-002 (level taxonomy expansion, #204, drafted canonical list), REQ-003 (taxonomy
  validation 500→422 in Tutee Post Service create And Submit, bug-prevention), REQ-004 (number of students "1–5 + 6 or
  more" reveal + the max operation, #205), REQ-005 (time-slot grid transpose to time×day + helper-text move, #206 — bitmask
  byte-for-byte unchanged, BLOCKING), REQ-006 (Rate-Guide discoverability — 3 links, #202 + #206-3). Spec B of the #206
  batch. requirements-engineer-authored (taxonomy content) with dba + frontend-expert + software-engineer input. status
  DRAFT pending the scoped team-review + founder taxonomy approval (tasks md Task 1).
