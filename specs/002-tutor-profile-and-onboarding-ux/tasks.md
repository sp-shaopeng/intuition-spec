# Tasks — Tutor profile and onboarding user experience (002)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each task is small (≈2–5 min of focused work), names the **owning specialist**, traces to one or more
> **REQ-NNN**, lists the **exact files**, and a **per-task verification step**. The Boss **dispatches the
> owning specialist** — the orchestrator does not build these. Check a box only when its evidence is captured
> under the private evidence directory for the requirement and opened by the reviewer.
>
> **Parallelism:** Slice A (availability), Slice B (multi-MRT) and Slice C (profile + onboarding capture) touch
> mostly disjoint files. Within intuition-backend builds serialize; within intuition-frontend builds
> serialize; the two repos run in parallel. Historical migrations are 0059/0059b → 0060; do not allocate these numbers again (original latest
> existing = 0057). **REQ-008 is UNBLOCKED (founder ruled Option b, 2026-06-20): onboarding now captures
> the full prefill set incl. qualifications/years (REQ-008b) and the profile prefills the full set (REQ-008).**
> Slice C needs **NO new migration** — the Tutor Profile columns already exist — but it is **NOT a small edit**:
> today's onboarding is a 2-PAGE flow (Register → Commission) with **no capture page**, so REQ-008b requires a
> NEW capture page/route (18d), an extended draft + payload (18e), the data transfer object fields (18a) and persistence (18b).

## Slice A — Fine-grained availability (#195) — REQ-001..006, 009

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 1 | Add Availability Slot @Embeddable (day Of Week, start Hour, duration Min) + availability Slots @Element Collection on Tutor Post | REQ-003 | software-engineer | Tutor Post implementation reference, new Availability Slot implementation reference | the documented operational check clean | [ ] |
| 2 | Liquibase **0059** changeset: create tutor post availability slots (PK tutor post id+day+start hour, FK cascade) | REQ-003 | dba | 0059-tutor-post-availability-slots implementation reference + master include | the documented operational check applies; databasechangelog row present | [ ] |
| 3 | Add the **0059b** backfill migration step: for each set bit b in availability bitmask decode day of week b/3 (0=Mon) and period b%3 (0=Morning,1=Afternoon,2=Evening), expand to hour rows (Morning 08–12 → 8,9,10,11; Afternoon 12–18 → 12..17; Evening 18–22 → 18..21; duration 60), idempotent when a duplicate is found leave the existing record unchanged + the reviewed database rollback (see design §Slice A backfill) | REQ-006 | dba | same 0059 file | database before/after query: legacy bits → expanded hour rows, days/periods unshifted, none dropped | [ ] |
| 4 | Add Availability Slot Dto + availability Slots to create/update listing request + GET response; map to/from entity; derive legacy availability Bitmask for back-compat | REQ-004 | bff-specialist, software-engineer | *Listing* implementation reference, response data transfer object, mapper/service | curl: POST slots → 201, GET echoes slots | [ ] |
| 5 | Slot validation: day Of Week 0–6, start Hour 0–23, duration Min∈60,120, start+dur≤24, no dup → 400 VALIDATION ERROR, nothing persisted | REQ-009 | software-engineer | request data transfer object (@Valid/custom validator), service | curl: start Hour is 25 → 400; database shows 0 rows for the attempt | [ ] |
| 6 | New frontend hour-slot vocabulary (DAYS already exist; add HOUR SLOTS/start-hour range + duration constants) — single source | REQ-005 | frontend-expert | scheduling implementation reference | the documented operational check clean; constant is sole source (grep) | [ ] |
| 7 | Rework Availability Grid 3-period rows → 1-hour rows; keep @390 horizontal-scroll affordance + aria-pressed cells | REQ-001 | frontend-expert | Availability Grid implementation reference | Playwright @1280+@390: one 1h cell selectable | [ ] |
| 8 | Add a 1h / 2h **granularity control**; in 2h mode a toggle writes two contiguous 60-min slots; **default to 2h at the @390 breakpoint** (user experience decision, REQ-002) while keeping 1h selectable, preserving the INT-074 scroll affordance | REQ-002 | frontend-expert, uiux-designer | Availability Grid implementation reference | Playwright @1280+@390: 2h block → two hour-slots in payload; @390 defaults to 2h and completes without rage-scroll | [ ] |
| 9 | Replace build Bitmask encoder with a slot-list builder feeding availability Slots in the create payload | REQ-004, REQ-005 | frontend-expert | Create Listing implementation reference | curl/network: create request carries slot list | [ ] |
| 10 | frontend Vitest: grid selection, 2h-mode expansion, encoder round-trip | REQ-001,002 | frontend-expert | *.test implementation reference | the documented operational check ≥97.5% line / ≥88% branch | [ ] |
| 11 | backend IT: slot round-trip save/reload + rejected-path (REQ-009) against live stack | REQ-003,004,009 | qa-engineer, software-engineer | *IT implementation reference | the documented operational check ≥97.5% line+branch | [ ] |

## Slice B — Multi-MRT on the post (#196) — REQ-007

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 12 | Add location Mrt Stations a set of stored values (Linked Hash Set) on Tutor Post with the join column operation (NOT tutor user id) and the column operation; retain legacy single column | REQ-007 | software-engineer | Tutor Post implementation reference | the documented operational check clean | [ ] |
| 13 | Liquibase **0060**: create tutor post mrt stations — join column **tutor post id** (FK→the tutor posts operation cascade), the varchar operation (matches legacy the location mrt station operation), PK tutor post id, mrt station + backfill existing single location mrt station into it + the reviewed database rollback | REQ-007 | dba | 0060-tutor-post-mrt-stations implementation reference + master include | the documented operational check applies; database shows backfilled rows; column named tutor post id | [ ] |
| 14 | Add location Mrt Stations list to create/update request + GET response; map to/from entity; keep accepting legacy single field | REQ-007 | bff-specialist, software-engineer | request/response data transfer objects, mapper/service | curl: POST 2 stations → 201, GET echoes both | [ ] |
| 15 | Step 3 single Combobox → multi-select (chips), mirroring location Planning Areas user experience; optional (0 allowed) | REQ-007 | frontend-expert, uiux-designer | Create Listing implementation reference (+ multi-select component) | Playwright @1280+@390: 2 MRT chips selected | [ ] |
| 16 | Wire multi-MRT into the create payload + frontend test | REQ-007 | frontend-expert | Create Listing implementation reference, *.test implementation reference | the documented operational check green; network shows the list | [ ] |
| 17 | backend IT: multi-MRT round-trip + backfill correctness | REQ-007 | qa-engineer | *IT implementation reference | the documented operational check green | [ ] |

## Slice C — Onboarding capture + profile prepopulation + handle autogen + subject verify (#197)

> **NO migration in Slice C** — REQ-008/008b/008c reuse existing Tutor Profile columns (rate columns,
> tutor profile planning areas + tutor profile subject levels junctions, location region, commitment).
> Onboarding only populates them at registration.

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 18a | Add OPTIONAL onboarding-capture fields to Register Tutor Request with names/types **aligned to the existing profile contract** (Update Tutor Profile Request/Tutor Profile): whole number hourly Rate Min Sgd, whole number hourly Rate Max Sgd, a set of values for location Planning Areas (**NOT** planning Areas), Location Region location Region (**enum**, not String), Tutor Commitment commitment (**enum**, not String), a set of values for subject Levels (qualifications/years Experience already on the data transfer object) — **no @Not no value/@Not Blank**; cross-field hourly Rate Max Sgd ≥ hourly Rate Min Sgd when both present; subject/level codes validated → UNKNOWN SUBJECT OR LEVEL | REQ-008b | bff-specialist, software-engineer | Register Tutor Request implementation reference | the documented operational check clean; field names/types match the update contract (no drift, no magic strings) | [ ] |
| 18b | Persist the new fields onto Tutor Profile in Auth Service register Tutor (map rate setters, location Planning Areas, location Region, commitment, subject Levels, qualifications, years Experience when present) — extend the existing the builder operation block | REQ-008b | software-engineer | Auth Service implementation reference (≈212-219) | backend IT: register WITH fields → database shows persisted rate/areas/region/subject-levels/quals/years | [ ] |
| 18c | Backward-compat: register-tutor with the new fields OMITTED still returns 201 and creates the profile with null rate / empty junctions (INT-043 no-forced-completion) | REQ-008c | software-engineer, bff-specialist | Auth Service implementation reference, IT | backend IT/curl: omit fields → 201; database profile created, junctions empty | [ ] |
| 18d | **NEW onboarding-capture page + route** — insert a page into the arc BETWEEN Register and Commission (account → **CAPTURE** → terms): build the optional-capture form (hourly-rate min/max, planning-areas multi-select, region, subject-level matrix via Subject Level Matrix Editor, commitment, qualifications, years Experience); add the route; thread the Wizard Stepper profile label onto this real page; update derive Tutor Onboarding Step. (This is a new page, NOT a label edit — the profile/listing stepper entries are labels only today.) | REQ-008b | frontend-expert, uiux-designer | NEW <Tutor Onboarding Profile> implementation reference, router config, onboarding implementation reference, Register implementation reference (navigate to the new route) | Playwright @1280+@390: new page renders, fields entered | [ ] |
| 18e | **Extend the draft + payload carry** — add the new fields to Tutor Registration Draft (today only email, phone, display name and password); have the new capture page write them into the draft; extend Commission implementation reference payload () to POST them in the single application programming interface ROUTES auth.register Tutor call (names matching the 18a data transfer object) | REQ-008b | frontend-expert | tutor Registration Draft implementation reference, new capture page, Commission implementation reference | Playwright @1280+@390: rate/area/subjects/quals/years entered on capture page → registered; network shows the extended payload | [ ] |
| 18 | Prefill the profile page with the FULL set (qualifications, years, hourly rate, area, region, subjects) from GET at the reset operation site (Profile implementation reference ≈190-208); any field with no persisted value renders empty-but-prompted (INT-043 banner) | REQ-008 | frontend-expert, bff-specialist | Profile implementation reference | Playwright @1280+@390: all onboarding-provided fields prefilled; skipped field prompted | [ ] |
| 19 | Confirm GET profile returns all onboarding-derived values (quals, years, rate, area, subjects) for prefill | REQ-008 | bff-specialist | profile response data transfer object/service | curl GET profile echoes the persisted onboarding values | [ ] |
| 20 | Compute suggested handle when null (reuse 0027b "Wei Tan"→"Wei T." algorithm); return suggested Public Handle on GET profile | REQ-010 | software-engineer, bff-specialist | Profile Service implementation reference, profile response data transfer object | curl GET: null handle → suggested value returned | [ ] |
| 21 | Prefill the editable Public-handle input at the reset operation site **Profile implementation reference** — change Handle: tutor Profile public Handle or otherwise to Handle: tutor Profile public Handle or otherwise suggested Public Handle or otherwise; keep editable | REQ-010 | frontend-expert | Profile implementation reference | Playwright @1280+@390: prefilled handle, edit + save works | [ ] |
| 22 | Confirm/extend the rejected-path guard: edited handle with personal information/profanity/>40/collision → 400/422, unchanged | REQ-011 | software-engineer, security-expert | Profile Service implementation reference (existing validation), IT | curl: handle "[REDACTED PHONE]" → 400/422; database handle unchanged | [ ] |
| 23 | VERIFY (live) the profile user interface adds a 2nd subject-level pair and round-trips; only convert to a fix if blocked | REQ-012 | frontend-expert, qa-engineer | Subject Level Matrix Editor implementation reference, Profile implementation reference | Playwright @1280+@390: 2 pairs after reload; database shows 2 rows | [ ] |

## Notes
- Independent tasks across the two repos run in parallel; within a repo builds serialize.
- A task is DONE only when its REQ-NNN evidence is captured under the private evidence directory for the requirement and opened by the reviewer.
- The REQ-008 open question is RESOLVED (founder ruled Option b, 2026-06-20): tasks 18a–18e (onboarding capture —
  data transfer object 18a, persistence 18b, compat 18c, NEW capture page 18d, draft+payload extension 18e) + 18/19 (full-set
  profile prefill) are all UNBLOCKED. Slice C adds NO migration but IS a multi-task build (new page + extended
  draft/payload), not a 2–5-minute edit.
- On completion of all tasks, set spec md status: HISTORICALLY MARKED IMPLEMENTED and confirm version/## Changelog reflect what shipped.

## 2026-09-05 correction follow-up — implementation not verified

- [ ] **REQ-006/008:** verify fine-slot/legacy-mirror parity in both client directions and that routine listing edits do not overwrite canonical profile fields. Use historical 0059/0059b/0060 evidence; any corrective migration is a new allocation. Profile-prefill evidence must identify the persisted profile source.
