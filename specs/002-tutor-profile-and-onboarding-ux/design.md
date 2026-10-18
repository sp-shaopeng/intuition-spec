# Design — Tutor profile and onboarding user experience (002)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> How the requirements in [requirements md](requirements.md) will be built. Co-owned with dba (schema),
> software-engineer (persistence/validation), bff-specialist (contracts), frontend-expert +
> uiux-designer (wizard/profile user interface). Significant choices are recorded as ADRs below.

## Architecture

Three independent slices, two repos. Backend (intuition-backend) and frontend (intuition-frontend) are
separate git repos — parallelize across the repo boundary, serialize builds within a repo.


Implementation example omitted. This public edition retains the written requirements in this section.


Patterns reused (do not reinvent): the @Element Collection + @Collection Table junction pattern
(Tutor Post subject Codes, Tutor Profile location Planning Areas); the single-source scheduling constant
(scheduling implementation reference); the provisional-handle database query language (0027b-backfill-provisional-public-handle); the
Api Routes constant for every path; typed enums both ends.

## Data model

### Slice A — tutor post availability slots (REQ-003, changeset **0059**)
| column | type | notes |
|--------|------|-------|
| tutor post id | uuid required | FK → the tutor posts operation automatically remove dependent records (join column) |
| day of week | smallint required | 0=Mon … 6=Sun (matches frontend DAYS ordering) — OR a PG enum day of week; **DBA to choose** (ADR-1) |
| start hour | smallint required | 0–23, the slot's starting hour (Singapore time local clock) |
| duration min | smallint required DEFAULT 60 | 60 for 1h, 120 for a 2h block stored as two 60-min rows (see ADR-2) |

- PK / unique: tutor post id, day of week, start hour — one row per hour, idempotent re-save.
- Index: tutor post id for the round-trip fetch; consider day of week, start hour for future
  availability search (defer until a query needs it — DBA call, no speculative index).
- Store the listing’s availability as a set of slots, each with day of week, start hour and duration.
- **availability bitmask column is RETAINED** (not dropped) through this change for backward compat and
  rollback; it becomes a derived/legacy mirror. Dropping it is a separate later changeset once nothing reads
  it (soft-delete-of-schema discipline). nullable is no stays satisfied by recomputing the legacy 21-bit coarse mirror from the current fine slots on every slot save; freezing the migrated value is not an allowed alternative. Legacy-only payloads expand their periods into the canonical slots before the mirror is derived.

### Slice A backfill (REQ-006, backfill change Set **0059b**, in the0059 file)
Set-based expansion of every existing post's 21-bit mask into hour rows. The period→hour-window mapping is a
**RESOLVED design decision** (not a clarification TODO), confirmed against Availability Grid implementation reference
("Morning: before 12pm · Afternoon: 12–6pm · Evening: 6–10pm"):

| period | slot Idx | hour window | start hour values | duration min |
|---|---|---|---|---|
| Morning | 0 | **08:00–12:00** (lower bound fixed at 08:00) | 8,9,10,11 | 60 |
| Afternoon | 1 | **12:00–18:00** | 12,13,14,15,16,17 | 60 |
| Evening | 2 | **18:00–22:00** | 18,19,20,21 | 60 |

**Reading the old availability format:** The old format gives each day three consecutive positions: morning, afternoon and evening, starting with Monday. For each selected position, divide its zero-based position number by three: the whole-number result identifies the day and the remainder identifies the period. Expand that period into the hourly rows in the table above. This exact mapping must be used in migration 0059 so saved availability does not move to another day or period. Do not create duplicate rows when the migration is repeated. The reviewed rollback removes the rows added by this migration.

### Slice B — tutor post mrt stations (REQ-007, changeset **0060**)
| column | type | notes |
|--------|------|-------|
| tutor post id | uuid required | FK → the tutor posts operation automatically remove dependent records — **join column named tutor post id** |
| mrt station | varchar using 100 required | station/area label; **the varchar operation matches the legacy the varchar operation** (Tutor Post implementation reference) |

- **DBA note on the join-column name (binding):** name the join column **tutor post id** — do NOT copy the
  misleading name from tutor profile planning areas, whose @Join Column is named tutor user id
  (Tutor Profile implementation reference) but actually FKs the tutor profiles operation, not a user table. This junction FKs
  the tutor posts operation, so its column is honestly tutor post id (matching the Slice A
  tutor post availability slots join column for consistency).
- PK: tutor post id, mrt station. Mapped a set of stored values location Mrt Stations with
  the join column operation (use Linked Hash Set for stable round-trip, like location Planning Areas).
- The legacy single location mrt station column is **retained**; a tiny backfill copies the existing single
  value into the new junction as the first element so existing posts keep their MRT (REQ-007 backward compat).

### Slice C — no schema change (onboarding capture reuses existing Tutor Profile columns)
handle already exists (the varchar operation unique-continuous integration). Suggested-handle is computed at read time in
Profile Service; nothing persists until the tutor saves.

**REQ-008b/008c onboarding capture — NO new migration.** Every target column already exists on Tutor Profile
(verified 2026-06-20 against Tutor Profile implementation reference):
| Onboarding field | Tutor Profile target | Already present |
|---|---|---|
| hourly Rate Min Sgd / hourly Rate Max Sgd | hourly rate min sgd / hourly rate max sgd (whole number, nullable) | yes (lines 51-55) |
| location Planning Areas (multi) | tutor profile planning areas junction (a set of stored values) | yes (lines 63-69) |
| location Region | location region (PG enum location region) | yes (lines 71-75) |
| subject Levels (matrix) | tutor profile subject levels junction (a set of values for) | yes (lines 86-91) |
| commitment | commitment (the varchar operation, enum Tutor Commitment) | yes (lines 46-49) |
| qualifications | qualifications (scalar text) | yes |
| years Experience | years experience (scalar whole number) | yes |

Onboarding only POPULATES these columns at registration; **no Liquibase changeset is required for Slice C.**
(0059/0060 belong to Slice A/B on tutor posts — a different table — and are unaffected.)

## application programming interface / contracts

All paths via Api Routes (Api Routes implementation reference); no literal
URLs. Field names are typed in the request/response records; enums shared via constants.

- **Create/update listing** (Create Listing Request / update data transfer object): **add** a set of values for
  availability slots with day of week, start hour and duration, and a set of values for location Mrt Stations. Keep
  availability Bitmask and location Mrt Station accepted during transition (deprecated; server derives the
  legacy fields from the new lists). GET listing returns both new lists. (REQ-004, REQ-007)
- **Validation (REQ-009):** day Of Week ∈ 0–6; start Hour ∈ 0–23; duration Min ∈ 60,120 and
  start Hour + duration Min/60 ≤ 24; reject duplicates. @Valid on the list + a custom validator → 400
  VALIDATION ERROR. Nothing persists on reject.
- **Register tutor** (Register Tutor Request, Register Tutor Request implementation reference):
  **add OPTIONAL** fields, with **names + types aligned to the EXISTING profile contract**
  (Update Tutor Profile Request / Profile Service / Tutor Profile) so the registration and update paths speak
  the same shape — NO drift:
  - whole number hourly Rate Min Sgd, whole number hourly Rate Max Sgd (matches Tutor Profile hourly Rate Min/Max Sgd).
  - a set of values for location Planning Areas (**NOT** planning Areas — matches Tutor Profile location Planning Areas
    (Tutor Profile implementation reference) and the location planning areas operation consumed at
    Profile Service implementation reference).
  - Location Region location Region (the **enum**, not String — matches Tutor Profile location Region
    (Tutor Profile implementation reference-75) and the update contract).
  - Tutor Commitment commitment (the **enum** Tutor Commitment, not String — matches
    Tutor Profile commitment (Tutor Profile implementation reference-49)).
  - a set of values for subject Levels (matches Tutor Profile subject Levels).
  - qualifications (the size operation) and whole number years Experience already exist on the data transfer object but are
    NOT sent by the frontend today — the new capture page now populates and sends them (REQ-008b).

  **No @Not no value / @Not Blank** on any onboarding-capture field — omitting them keeps registration succeeding
  exactly as today (REQ-008c); INT-043's no-forced-completion intent is preserved. Reuse cross-field bounds
  where they exist (hourly Rate Max Sgd ≥ hourly Rate Min Sgd when both present; subject/level codes validated
  against the reference tables → existing UNKNOWN SUBJECT OR LEVEL). Auth Service register Tutor
  (Auth Service implementation reference-219) maps these onto the builder operation (rate setters, location Planning Areas,
  location Region, commitment, subject Levels, qualifications, years Experience) — the canonical
  persistence site (REQ-008b). frontend carries them in the **extended** in-memory tutor-registration draft
  (tutor Registration Draft implementation reference — today only 4 fields) and POSTs them in the single
  application programming interface ROUTES auth.register Tutor payload from Commission implementation reference, collected on the **new onboarding-capture page**.
  (REQ-008b, REQ-008c)
- **GET tutor profile** (Tutor Profile Response): already returns qualifications, years Experience,
  hourly Rate Min/Max Sgd, location Planning Areas/location Region, and subject Levels. The frontend **prefills ALL of
  these** on the profile page from the GET response; a field with no persisted value renders empty-but-prompted
  (INT-043 banner). No new response field is required for REQ-008 beyond suggested Public Handle below. (REQ-008)
- **GET tutor profile** (Tutor Profile Response): add suggested Public Handle — the **only genuinely new
  RESPONSE field** in this spec (all other prefill values — quals, years, rate, area, region, subjects — are
  already returned). Computed when Handle is null; null/echo when already set. frontend prefills the editable
  field with the saved public handle, or the suggested handle if none is saved, or an empty value if neither exists. (REQ-010)
- **Update profile** (Update Tutor Profile Request, existing): unchanged shape; the existing server-side handle
  validation in Profile Service (Profile Service implementation reference-257, denylist email/phone/url, unique-continuous integration, max 40)
  is the REQ-011 guard — autogen does NOT bypass it. (REQ-011)

## Error handling
- Bad availability slot → 400 VALIDATION ERROR, no rows written (REQ-009). Surface field-level message in
  the wizard near the grid.
- Handle violation on save → existing 400/422 handle-validation error path (REQ-011); the wizard/profile shows
  the inline error and the stored handle is unchanged.
- Backfill (0059/0060) is idempotent (when a duplicate is found leave the existing record unchanged) so a re-run is safe; rollback removes only the
  inserted rows.
- Empty selections are valid for MRT (optional) and for availability the existing "at least one"
  wizard rule (if any) is preserved.

## Testing strategy
- **REQ-001/002/007/008/008b/010/012 (user interface):** Playwright @1280 + @390 captures of each Gherkin state + frontend Vitest
  unit tests on the new grid encoder/granularity logic, the multi-select, the onboarding-capture step, and the
  full-set profile prefill; frontend gate ≥97.5% line / ≥88% branch.
- **REQ-003/006/007 (data):** Liquibase changeset applies under Testcontainers in the documented operational check; database query
  before/after for the backfill; entity round-trip IT (save slots → reload → equal set).
- **REQ-008b/008c (onboarding persistence):** backend IT against the live stack — register-tutor WITH the optional
  fields persists them onto Tutor Profile (database query of the profile row + planning-areas/subject-levels
  junctions); register-tutor WITHOUT them still returns 201 and creates the profile with null rate / empty
  junctions (the INT-043 compat path). No migration → no Liquibase apply step for Slice C.
- **REQ-004/009/010/011 (application programming interface):** REST-Assured/live-stack IT — happy round-trip AND the rejected path (400/422)
  for each guard; backend gate ≥97.5% line+branch.
- **REQ-005 (no magic strings):** ESLint + Spotless + a grep assertion that availability literals live only in
  the canonical constant/enum.
- **Regression:** existing listing-create + profile ITs must stay green with the retained legacy fields
  (backward-compat proof).

## Migration / rollout
- **0059** — create tutor post availability slots + backfill from availability bitmask (two changesets,
  same file). availability bitmask retained. **0060** — create tutor post mrt stations + backfill the
  single location mrt station into it. Next-free numbers verified (latest existing = 0057); leave no
  collision.
- **Slice C (onboarding capture + profile prefill) needs NO migration (CONFIRMED).** REQ-008/008b/008c reuse
  existing Tutor Profile columns (rate, planning-areas junction, subject-levels junction, region, commitment,
  qualifications, years experience); they are data transfer object + persistence-wiring + user interface changes (incl. a NEW
  onboarding-capture page + extended draft/payload — see Slice C tasks 18a–18e). No new changeset, no collision risk.
- **Backward compat:** legacy columns kept and kept in sync; old clients sending only the bitmask /
  single-MRT still work; new clients send the lists. A later spec drops the legacy columns once no reader
  remains.
- **Rollback:** each changeset has a the reviewed database rollback (drop the junction tables / delete backfilled rows). Because
  the legacy columns are retained, rolling back the code path keeps the system functional on the old model.
- **Feature flag:** not required (additive contract + retained legacy fields). The frontend grid swap ships atomically
  with the backend contract; if risk is raised at review, gate the new grid behind a build-time flag (frontend-only).

## Decisions (ADR log)
- **2026-06-20 — ADR-1: junction table over a wider bitmask.** *Context:* 1h granularity = up to 168 bits,
  overflowing whole number; a BIGINT (64 bits) still can't hold 7×24. *Decision:* a normalized
  tutor post availability slots junction (one row per selected hour), mirroring the existing
  @Element Collection pattern. *Consequences:* queryable/indexable, no blob, follows precedent; slightly more
  rows per post (≤168) — acceptable; DBA picks smallint day vs PG enum during build.
- **2026-06-20 — ADR-2: a 2-hour block is stored as two 60-min rows, not one 120-min row.** *Context:* #195
  wants 1h OR 2h selection. *Decision:* the model's atom is the 1-hour slot; 2h selection is a user interface convenience
  that writes two contiguous 60-min rows. *Consequences:* one canonical granularity in storage (simpler
  queries/merge/round-trip), duration Min stays mostly 60; the frontend re-derives "2h" for display. (If review
  prefers preserving the literal 120-min block, flip duration Min to 120 and store one row — captured here as
  the alternative.)
- **2026-06-20 — ADR-3: retain legacy availability bitmask and location mrt station.** *Context:*
  backward compat + safe rollback. *Decision:* keep both columns, derive/sync them, drop later. *Consequences:*
  zero-break migration; small temporary dual-write; a follow-up cleanup spec.
- **2026-06-20 — ADR-4 (SUPERSEDED by ADR-5): prepopulation limited to onboarding-captured fields (REQ-008
  Option a).** *Context:* onboarding only stored display Name/qualifications/years Experience; #197 assumed more.
  *Original decision:* prefill only what exists; flag Option (b) at the spec gate. *Status:* **SUPERSEDED** —
  the founder ruled Option (b) on 2026-06-20 (see ADR-5). Retained for the decision trail.
- **2026-06-20 — ADR-5: EXPAND ONBOARDING to capture rate/area/subject-levels (REQ-008 Option b — founder
  ruling, supersedes ADR-4).** *Context:* #197.2 wants the profile fully pre-populated "based on onboarding";
  the founder chose to make that real rather than narrow the requirement. *Decision:* the onboarding wizard
  captures **hourly rate (min/max), planning-areas and subject-levels** as **OPTIONAL** fields, persisted onto
  Tutor Profile at registration (REQ-008b); the profile page then prefills the full set (REQ-008).
  *Consequences:* (1) a **partial reversal of INT-043** — the fields are now OFFERED at signup — but INT-043's
  core intent (never FORCE completion / never re-block signup) is preserved by keeping them optional + the
  post-login completion banner (REQ-008c). (2) **NO new migration:** every target column already exists on
  Tutor Profile (rate columns, planning-areas + subject-levels junctions, region, commitment, qualifications,
  years experience); only the data transfer object, the new capture page and the persistence wiring change. (3) **NOT a small
  edit** — today's arc is a 2-PAGE flow (Register → Commission) with NO capture page; the profile/listing
  Wizard Stepper entries are labels only. Option (b) therefore needs: a **new onboarding-capture page + route**
  inserted between Register and Commission, an **extended Tutor Registration Draft** (today only 4 fields), an
  **extended Commission/register payload**, **new Register Tutor Request fields** (names/types aligned to the
  existing profile contract — see application programming interface §) + validation, the **persistence mapping**, and the additional
  full-set prefill. Costed in tasks md as 18a–18e (Slice C), not as one task.
  *Alternative kept on file:* Option (a) (ADR-4) — prefill only display Name/quals/years — rejected by the founder.

## 2026-09-05 canonical profile and migration clarification

read operation for tutor profile supplies persisted canonical-profile values from onboarding/profile edits and the approved one-time latest-post backfill. Reading or saving a listing must not silently change profile defaults; its overrides apply only while attached. Cover profile-edit → differing listing-edit → profile-reload: the profile edit survives. Availability saves derive the legacy mirror from fine slots and preserve all period/day mappings; exercise old-client bitmask input and new-client slot input. The historical applied allocation is 0059 (slots), 0059b (backfill), 0060 (MRT), as recorded in spec md's shipment entry; new migrations must use the next unallocated ID, never rewrite applied migrations.
