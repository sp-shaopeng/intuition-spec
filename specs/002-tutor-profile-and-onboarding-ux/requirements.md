# Requirements — Tutor profile and onboarding user experience (002)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin**
> acceptance criteria and the **exact evidence** that will prove each scenario. A compound requirement is two
> requirements — split it. See SKILL implementation reference for the EARS patterns and the evidence-type catalogue.

**Source issues (backward trace):**
- GH #195 — Tutor Workflow Step 2 of 4 (Availability): "Allow the Tutor to select timeslot by 1 hour blocks
  or 2 hour blocks" / "The timeslot is too wide." Founder decision (binding): deliver REAL finer-grained
  selectable slots (a backend data-model change), **not** a CSS narrowing of the existing 3-period grid.
- GH #196 — Tutor Workflow Step 3 ("Tell us about yourself"): "Preferred MRT station / area (optional) —
  Allow multiple selection."
- GH #197 — Tutor Profile: (1) Public handle autogenerate-first, editable later; (2) hourly rate / add a
  subject / highest qualification / years of experience / area should be pre-populated based on onboarding;
  (3) "Add a subject" should allow more than one, as per the onboarding page.

**Grounded against real code (verified 2026-06-20):**
- Wizard Create Listing implementation reference (4-step). Step 2 uses
  Availability Grid implementation reference — a coarse **21-bit bitmask** = 7 days × 3
  periods (Morning/Afternoon/Evening), encoded by build Bitmask (Create Listing implementation reference) into
  availability Bitmask. Periods defined in scheduling implementation reference (SLOTS).
- Backend Tutor Post implementation reference — availability bitmask
  whole number (21-bit, lines 49-51). 1h granularity (e g. 7×~14 slots = up to 168 bits) overflows whole number.
- Step 3 MRT field location Mrt is a **single-select** Combobox (Combobox implementation reference, ~140 stations);
  persisted to Tutor Post location Mrt Station (single the varchar operation, Tutor Post implementation reference-74).
- Tutor Profile location Planning Areas (Tutor Profile implementation reference-69) already demonstrates the multi-select
  @Element Collection + junction (tutor profile planning areas) pattern to COPY.
- Tutor profile page Profile implementation reference — Handle is an editable text
  input, prefilled from the GET in the form the reset operation at **Profile implementation reference** as
  Handle: tutor Profile public Handle or otherwise (so a tutor with no handle gets an empty field, NOT an
  auto-generated one). Backend Tutor Profile public Handle the varchar operation
  unique-continuous integration, validated in Profile Service (denylist email/phone/url, Profile Service implementation reference-257). Changeset
  0027-backfill-tutor-profile-rate-area implementation reference (0027b-backfill-provisional-public-handle) already derives a
  provisional handle from display name ("Wei Tan" → "Wei T.") — reuse that algorithm for autogen.
- Subjects: Subject Level Matrix Editor implementation reference supports **multiple**
  subject-level pairs (no numeric cap — the editor adds a per-subject level row for every taxonomy subject and
  re-emits the full pair-set; there is no count limit in the component); backend Tutor Profile subject Levels
  (junction tutor profile subject levels). So #197.3 ("more than one subject") may ALREADY be satisfied — a
  VERIFY requirement, not a build.
- **Onboarding is a 2-PAGE flow, NOT a multi-step capture wizard** (verified 2026-06-20):
  Register implementation reference (account details — name/email/phone/password) →
  Commission implementation reference (commission terms) → POST application programming interface ROUTES auth.register Tutor.
  The profile / listing entries in onboarding implementation reference
  (TUTOR ONBOARDING STEPS) are **Wizard Stepper LABELS only** — there is **no capture page** behind them; no
  profile field is collected at signup today. The in-memory carry
  tutor Registration Draft implementation reference (Tutor Registration Draft) holds **only 4 fields**
  (email, phone, display Name, password); Register implementation reference sets exactly those, and Commission implementation reference
  (payload, ) POSTs only email, phone, password, display Name, commission Acknowledged,
  commission Terms Version.
- Onboarding Register Tutor Request implementation reference declares
  display Name + qualifications + years Experience fields, **but the frontend never sends qualifications or
  years Experience** — the draft + Commission payload omit them (above). So in practice **NO profile field
  (not rate, not area, not subjects, not even qualifications/years) is captured at onboarding today** — every
  one is entered AFTER signup (INT-043). Auth Service register Tutor builds the Tutor Profile at
  Auth Service implementation reference-219. **Founder ruled Option (b) on 2026-06-20:** onboarding must START capturing the
  full prefill set — hourly rate (min/max), planning-areas, region, subjects, commitment **AND** qualifications
  + years Experience (the latter two have no source today either) — OPTIONAL, persisted here. See REQ-008b. The
  target Tutor Profile columns ALREADY EXIST (hourly Rate Min/Max Sgd, tutor profile planning areas,
  location Region, tutor profile subject levels, commitment; qualifications/years Experience are
  scalar columns on Tutor Profile), so **no new migration is needed** for the capture — but it is NOT a
  small edit: it requires a **new onboarding-capture page/route**, an **extended Tutor Registration Draft
  shape**, an **extended Commission/register payload**, plus the new data transfer object fields and the persistence wiring
  (re-costed honestly in tasks md 18a–18e).

**Cross-cutting constraints (apply to every REQ):** soft-delete only; Liquibase-only (no number collisions —
historical migration allocation is **0059/0059b** for availability and **0060** for MRT; **0058** belongs to spec 001); no magic strings (application programming interface paths → Api Routes, enums →
typed constants both ends); existing posts/profiles must not break (backward compat); backend coverage ≥97.5%
line+branch, frontend ≥97.5% line / ≥88% branch as the test-acceptance gate.

---

## RESOLVED — clarification gate (REQ-008 / REQ-008b)  ✅ founder ruled 2026-06-20

> **#197.2 prepopulation** asks to pre-populate **hourly rate, a subject, highest qualification, years of
> experience, and area** "based on onboarding information that was done." When this spec was first drafted,
> onboarding (Register Tutor Request) only captured **display Name, qualifications, years Experience** —
> rate/area/subjects were entered AFTER onboarding (INT-043), so the data the issue assumed did not exist at
> onboarding time. That was the open question.
>
> **Founder decision (binding, 2026-06-20): OPTION (b) — EXPAND ONBOARDING.** Onboarding must START capturing
> the full prefill set — **hourly rate (min/max), serviceable area/planning-areas, region, subject-levels,
> commitment, AND qualifications + years of experience** — so the profile page can fully pre-populate them.
> (qualifications + years are explicitly included because, although Register Tutor Request declares the fields,
> the frontend draft + Commission payload never send them today, so they too lack an onboarding source.) This
> **partially reverses INT-043**: INT-043's intent — *do not FORCE profile completion at signup* — is preserved
> (the new fields stay **OPTIONAL** at signup, signup is not re-blocked, and the post-login completion banner
> still covers anything skipped), but the fields are now **offered and persisted at onboarding** so a tutor who
> fills them does not re-enter them on the profile page. Because today's arc is a **2-page flow with no capture
> page**, this requires a NEW onboarding-capture page/route + an extended draft + an extended payload — costed
> honestly in tasks md (18a–18e), not as a trivial edit.
>
> **Resulting requirement split:**
> - **REQ-008b (NEW, MUST)** — onboarding *captures* hourly Rate / area / region / subject-levels / commitment /
>   qualifications / years and *persists* them onto Tutor Profile at registration (new capture page + extended
>   draft + extended payload + new Register Tutor Request fields + persistence wiring). The fields are OPTIONAL;
>   if omitted, registration still succeeds exactly as today.
> - **REQ-008 (MODIFIED, MUST)** — the profile page pre-populates **ALL** of: hourly rate, subjects,
>   qualification, years of experience, area — from whatever onboarding (REQ-008b) or any prior profile/listing
>   save persisted. Fields the tutor skipped at onboarding render empty-but-prompted (INT-043 completion banner).
>
> **No new migration for the onboarding-capture columns (CONFIRMED 2026-06-20):** every target column already
> exists on Tutor Profile — hourly rate min sgd / hourly rate max sgd (Tutor Profile implementation reference-55),
> commitment (47-49), tutor profile planning areas junction + location region (63-75),
> tutor profile subject levels junction (86-91), and the scalar qualifications / years experience
> columns. Onboarding only needs to POPULATE them at registration; **no Liquibase changeset is required for
> REQ-008/008b** (next-free numbers 0059/0060 are consumed by Slice A/B only). (Slice A/B migrations 0059/0060
> are unaffected — they are for tutor posts, a different table.)

---

## REQ-001 — Tutor selects availability in fine-grained 1-hour blocks
- **Type:** user interface
- **Priority:** Must
- **Source:** GH #195 (Step 2 Availability; founder binding: real finer-grained slots)
- **Rationale / value:** A 3-period (Morning/Afternoon/Evening) grid is "too wide" — a tutor free 7–9pm cannot
  say so without claiming the whole 6–10pm Evening block. 1-hour granularity lets supply advertise real
  availability, improving match precision.
- **Owner (specialist):** frontend-expert (user interface) · uiux-designer (interaction/IA)
- **Acceptance criteria (Gherkin):**


- Scenario: Pick a single one-hour block.
- Given a tutor on Step 2 of the create-listing wizard at 1280px.
- When they toggle the "Tue 19:00–20:00" cell ON.
- Then exactly that one-hour block is selected (and no adjacent hour is selected).


- **Required evidence (defined up front):** Playwright screenshot **@1280 and @390** of Step 2 showing the
  1-hour grid with one hour-block selected; DOM/console clean. (Mobile @390 must show the horizontal-scroll
  affordance still works for the wider grid.)
- **Dependencies:** REQ-003 (storage), REQ-005 (constants)

## REQ-002 — Tutor selects availability in 2-hour blocks via a granularity control
- **Type:** user interface
- **Priority:** Must
- **Source:** GH #195 ("select timeslot by 1 hour blocks **or** 2 hour blocks")
- **Rationale / value:** #195 asks for BOTH 1h and 2h selection. A granularity toggle (1h / 2h) lets a tutor who
  thinks in 2-hour sessions select faster without losing the 1h option. Atomic split from REQ-001 because the
  2h-mode behaviour can pass/fail independently of the 1h-cell behaviour.
- **RESOLVED user experience decision (binding — uiux advisory applied):** a 7-day × ~14-hour 1h grid is ~98 cells, which at
  390px forces rage-scrolling. Therefore **2-hour granularity is the DEFAULT at the @390 mobile breakpoint**
  (1h remains available via the granularity control; @1280 may default to 1h). The @390 evidence MUST prove a
  tutor completes a selection without rage-scrolling, and MUST preserve the INT-074 horizontal-scroll
  affordance (the visible "Swipe to reach Sat & Sun" hint + right-edge fade in Availability Grid implementation reference).
- **Owner (specialist):** frontend-expert (user interface) · uiux-designer (interaction)
- **Acceptance criteria (Gherkin):**


- Scenario: Two-hour granularity selects a contiguous pair of hours.
- Given a tutor on Step 2 with the granularity control set to "2 hours".
- When they toggle the "Wed 14:00–16:00" block ON.
- Then both the 14:00–15:00 and 15:00–16:00 hour-slots are recorded as selected.


- **Required evidence (defined up front):** Playwright screenshot **@1280 and @390** of Step 2 with the
  granularity control set to "2 hours" and one 2-hour block selected; plus the request payload (REQ-004)
  showing two contiguous hour-slots persisted for that block.
- **Dependencies:** REQ-001, REQ-003

## REQ-003 — Backend persists tutor-post availability at 1-hour granularity (junction table)
- **Type:** data
- **Priority:** Must
- **Source:** GH #195 (founder binding: backend data-model change); design md §Data model
- **Rationale / value:** A 21-bit whole number bitmask cannot hold 1-hour-per-day granularity. A normalized
  junction (tutor post availability slots) stores arbitrary fine slots, is queryable/indexable, and follows
  the existing @Element Collection junction pattern (soft constraint: no JSON blob — DBA hard rule).
- **Owner (specialist):** dba (schema/migration) · software-engineer (entity/persistence)
- **Acceptance criteria (Gherkin):**


- Scenario: Selected hour-blocks are stored as junction rows.
- Given a tutor saves a listing with Tue 19:00–20:00 and Wed 14:00–16:00 selected.
- When the listing is persisted.
- Then tutor post availability slots holds rows (Tue,19,60),(Wed,14,60),(Wed,15,60) for that post id.


- **Required evidence (defined up front):** a private inspection of the saved listing’s availability, showing each day, start hour and duration exactly as expected; plus the
  Liquibase changeset id (0059) and that the documented operational check applies it cleanly (databasechangelog row).
- **Dependencies:** none (foundational)

## REQ-004 — Availability application programming interface accepts and returns the fine-grained slot list
- **Type:** application programming interface
- **Priority:** Must
- **Source:** GH #195; design md §application programming interface
- **Rationale / value:** The create/update listing contract must carry the new slot list (replacing — or
  alongside, during transition — availability Bitmask) so the frontend grid round-trips. No magic strings: field
  named via the data transfer object, route via Api Routes.
- **Owner (specialist):** bff-specialist (contract) · software-engineer (mapping)
- **Acceptance criteria (Gherkin):**


- Scenario: Create-listing round-trips the availability slots.
- Given a valid create-listing request whose availability Slots = [day:TUE,start Hour,duration Min].
- When POST to the listing-create route succeeds (201).
- Then GET of that listing returns availability Slots containing day:TUE,start Hour,duration Min.


- **Required evidence (defined up front):** live curl/web request protocol log: POST request body + 201, then GET + 200 body
  echoing the slot list. Include the negative path → covered by REQ-009.
- **Dependencies:** REQ-003

## REQ-005 — Canonical 1-hour slot vocabulary is a single shared source of truth
- **Type:** constraint
- **Priority:** Must
- **Source:** project rule "no magic strings"; scheduling implementation reference precedent
- **Rationale / value:** The hour-slot ordering/labels (days, start hours, allowed durations) must be defined
  once on each end (frontend constant + backend enum/constant) and reused by the grid, encoder, data transfer object and entity — never
  hand-typed. Mirrors the existing DAYS/SLOTS single-source rule.
- **Owner (specialist):** frontend-expert (frontend constant) · software-engineer (backend constant/enum)
- **Acceptance criteria (Gherkin):**


- Scenario: Slot day-of-week is a typed enum, not a literal.
- Given the availability grid, encoder, request data transfer object and entity.
- When the codebase is searched for day-of-week / duration literals in availability logic.
- Then no raw availability string/number literal exists outside the canonical constant/enum.


- **Required evidence (defined up front):** grep/ESLint/Spotless output showing the constant is the only
  source (no stray literals); test-run log of the frontend lint + backend build passing with the shared vocabulary.
- **Dependencies:** REQ-001, REQ-003

## REQ-006 — Existing 21-bit availability is migrated forward to fine-grained slots (no data loss)
- **Type:** data
- **Priority:** Must
- **Source:** constraint "data migration for existing tutor posts … existing 21-bit availability must map
  forward"; design md §Migration
- **Rationale / value:** Every existing approved/active post already carries a 3-period bitmask. On the new
  model those posts must keep advertising the SAME windows, expanded to the corresponding hour-slots, so no
  live listing silently loses its availability.
- **RESOLVED design decision (period→hour-window mapping, binding — not a TODO):** confirmed against the
  caption Availability Grid implementation reference ("Morning: before 12pm · Afternoon: 12–6pm · Evening: 6–10pm"), the
  three windows are fixed as: **Morning = 08:00–12:00** (lower bound set explicitly to 08:00 — the caption's
  "before 12pm" is open-ended, 08:00 is the decided floor), **Afternoon = 12:00–18:00**, **Evening =
  18:00–22:00**. These are the exact hour ranges the 0059b backfill expands each set period into (start hour ∈
  8,9,10,11 / 12,…,17 / 18,…,21, one 60-min row each).
- **RESOLVED bitmask decode (binding):** build Bitmask (Create Listing implementation reference) packs each selected cell
  as bit day Idx*3 + slot Idx, where DAYS Mon,Tue,Wed,Thu,Fri,Sat,Sun (so **day 0 = Mon**) and
  SLOTS Morning,Afternoon,Evening (so **slot 0 = Morning, 1 = Afternoon, 2 = Evening**), per
  scheduling implementation reference. Therefore the 0059b backfill decodes each set bit b as **day b / 3**
  (integer) and **period b % 3**, then expands period into its hour window above. This exact mapping
  MUST be written into the 0059 changeset so migrated posts do not shift days or periods.
- **Owner (specialist):** dba (backfill changeset)
- **Acceptance criteria (Gherkin):**


- Scenario: A legacy Evening bit expands to its hour-slots.
- Given an existing post with the "Mon Evening" bit set and no availability slots.
- When the backfill changeset (0059b) runs.
- Then tutor post availability slots gains the Mon 18:00–22:00 hour rows for that post.
- And the post's other selected periods are likewise expanded with none dropped.


- **Required evidence (defined up front):** database before/after: count/sample of legacy bitmask posts before, and
  the expanded tutor post availability slots rows after the changeset; the changeset id (0059) + clean
  the documented operational check apply. Round-trip check: re-encoding the slots reproduces the original period set.
- **Dependencies:** REQ-003

## REQ-007 — Step 3 "Preferred MRT station / area" accepts MULTIPLE selections
- **Type:** user interface + data
- **Priority:** Must
- **Source:** GH #196
- **Rationale / value:** A tutor often serves several MRT stations/areas. The current single-select Combobox
  forces one. Multi-select (mirroring location Planning Areas' @Element Collection + junction
  tutor post mrt stations) lets a post list every station it serves, widening match reach. Optional field —
  zero selections is valid.
- **Owner (specialist):** frontend-expert (multi-select user interface) · dba (junction tutor post mrt stations +
  migration) · software-engineer (entity/mapping) · bff-specialist (contract)
- **Acceptance criteria (Gherkin):**


- Scenario: Two MRT stations are selected and persisted.
- Given a tutor on Step 3 of the create-listing wizard.
- When they select both "Bishan" and "Tampines" in Preferred MRT station / area.
- Then both stations appear as selected chips.
- And after save, tutor post mrt stations holds rows for Bishan, Tampines for that post id.


- **Required evidence (defined up front):** Playwright screenshot **@1280 and @390** of Step 3 with ≥2 MRT
  chips selected; database query of tutor post mrt stations showing both rows; curl log of the create request
  carrying the list. Changeset id (0060) applies cleanly.
- **Dependencies:** REQ-005 pattern reuse

## REQ-008b — Tutor onboarding captures the full prefill set (rate, area, region, subjects, commitment, qualifications, years) and persists them at registration
- **Type:** user interface + application programming interface + data (capture/persistence)
- **Priority:** Must
- **Source:** GH #197.2; founder decision 2026-06-20 (Option b — expand onboarding)
- **Rationale / value:** So the profile page can pre-populate the **full** REQ-008 set, onboarding must add a
  **new capture page** to today's 2-page arc (Register → [NEW capture] → Commission → POST register Tutor) that
  COLLECTS and PERSISTS onto Tutor Profile at registration: **hourly Rate Min/Max, planning-areas, region,
  subject-levels, commitment, AND qualifications + years Experience.** The last two are explicitly in scope
  because REQ-008's premise ("prefill qualifications/years from onboarding") has **no source today** — the frontend
  draft + Commission payload omit them (see grounding), so without capturing them REQ-008 cannot meet its AC.
  All fields stay **OPTIONAL** (INT-043's intent — never re-block signup is preserved): a tutor who provides
  them prefills the profile; a tutor who skips them registers exactly as today and leans on the completion
  banner. **No new schema** — every target column already exists on Tutor Profile (rate columns,
  planning-areas + subject-levels junctions, location region, commitment, and the scalar
  qualifications/years experience).
- **Scope is NOT a small edit (honest cost):** this requires (1) a **new onboarding-capture page + route**
  inserted between Register and Commission, (2) an **extended Tutor Registration Draft** shape carrying the new
  fields (today only 4), (3) an **extended Commission/register payload** carrying them into
  application programming interface ROUTES auth.register Tutor, (4) the **new Register Tutor Request data transfer object fields**, and (5) the
  **persistence wiring** in Auth Service register Tutor. Costed as tasks 18a–18e (not one task).
- **Owner (specialist):** bff-specialist (data transfer object/contract) · software-engineer (registration persistence wiring)
  · frontend-expert + uiux-designer (new onboarding-capture page + draft/payload carry)
- **Acceptance criteria (Gherkin):**


- Scenario: Provided onboarding fields persist onto the tutor profile at registration.
- Given a tutor registering who enters hourly Rate min,max, planning-areas Bishan,.
- When they complete registration (POST auth register tutor interface operation).
- Then the saved Tutor Profile has hourly rate min sgd is 40, hourly rate max sgd is 60,.


- **Required evidence (defined up front):** live curl log of submit operation for auth register tutor (201) carrying
  the new optional fields; database query of tutor profiles + tutor profile planning areas +
  tutor profile subject levels for that user showing the persisted values (incl. qualifications/years);
  Playwright screenshot **@1280 and @390** of the new onboarding-capture page with rate/area/subjects/quals/years
  entered.
- **Dependencies:** none (columns pre-exist) — REQ-008 prefill consumes its output

## REQ-008c — Onboarding registration succeeds when the new optional fields are omitted (INT-043 backward compat)
- **Type:** application programming interface (negative/compat guard)
- **Priority:** Must
- **Source:** founder decision 2026-06-20 ("don't re-block signup"); INT-043 intent
- **Rationale / value:** The Option-(b) expansion must NOT force completion. Omitting hourly rate / area /
  subject-levels (or sending them empty) must still register the tutor successfully — exactly as INT-043 made
  possible — so the partial reversal does not regress the no-forced-completion guarantee.
- **Owner (specialist):** software-engineer (registration) · bff-specialist (contract — fields nullable, no
  @Not no value)
- **Acceptance criteria (Gherkin):**


- Scenario: A tutor registers with the new onboarding fields omitted.
- Given a register-tutor request that omits hourly Rate Min/Max, location Planning Areas and subject Levels.
- When it is POSTed to auth register tutor interface operation.
- Then registration succeeds (201) and the Tutor Profile is created with no rate/area/subject-levels set.
- And on first /tutor/profile load those fields are empty with the completion prompt (INT-043 banner).


- **Required evidence (defined up front):** live curl log of submit operation for auth register tutor WITHOUT the new
  fields → 201; database query showing the profile row created with null rate / empty junctions; Playwright @1280+@390
  of the completion banner on first profile load.
- **Dependencies:** REQ-008b

## REQ-008 — Tutor profile pre-populates ALL onboarding-derived fields (rate, subjects, qualification, years, area)
- **Type:** user interface + application programming interface
- **Priority:** Must
- **Source:** GH #197.2; founder decision 2026-06-20 (Option b)
- **Status:** **READY** — open question resolved (Option b). Prefill now covers the full field set.
- **Rationale / value:** When a tutor opens the profile page, **all** of hourly rate, subjects, highest
  qualification, years of experience, and area must be pre-filled from whatever was persisted at onboarding
  (REQ-008b) or a prior canonical-profile save. The explicit one-time latest-listing backfill may seed the profile; routine listing saves never synchronize or overwrite it (product requirements document §§5.1a, 20.2). Listing overrides remain attached to that listing. Fields the
  tutor skipped at onboarding (no persisted value) render empty-but-prompted (INT-043 completion banner). Do
  NOT fabricate values for fields with no persisted source.
- **Owner (specialist):** frontend-expert (prefill wiring) · bff-specialist (GET profile shape)
- **Acceptance criteria (Gherkin):**


- Scenario: All onboarding-provided fields prefill on the profile page.
- Given a tutor who at onboarding provided qualifications "NUS BSc", years Experience 5,.
- When they open /tutor/profile.
- Then the Highest-qualification, Years-of-experience, Hourly-rate, Area and Subjects fields are all.
- And any field they did NOT provide at onboarding is empty with the completion prompt.


- **Required evidence (defined up front):** Playwright screenshot **@1280 and @390** of /tutor/profile
  first load for a tutor who completed the expanded onboarding, showing rate + area + subjects + qualifications
  + years all pre-filled (and a skipped field prompted); curl log of GET profile returning those values.
- **Dependencies:** REQ-008b (persists the values the profile reads back)

## REQ-009 — Invalid availability slot input is rejected
- **Type:** security / data (negative guard)
- **Priority:** Must
- **Source:** EARS "If…then" guard derived from REQ-003/REQ-004 (every fine-grained input needs a reject path)
- **Rationale / value:** A client could post an out-of-range start Hour (e g. 25), a non-positive
  duration Min, a duration that overruns the day, or a bad day-of-week. The application programming interface must reject rather than
  persist garbage slot rows.
- **Owner (specialist):** software-engineer (validation) · bff-specialist (error contract)
- **Acceptance criteria (Gherkin):**


- Scenario: Out-of-range start hour is rejected.
- Given a create-listing request with an availability slot day:MON,start Hour,duration Min.
- When it is POSTed to the listing-create route.
- Then the response is 400 VALIDATION ERROR and no row is written to tutor post availability slots.


- **Required evidence (defined up front):** live curl log showing the **rejected** path (400 + error code);
  database query confirming zero slot rows written for that attempt.
- **Dependencies:** REQ-004

## REQ-010 — Public handle is auto-generated on first load, then editable
- **Type:** user interface + application programming interface
- **Priority:** Must
- **Source:** GH #197.1
- **Rationale / value:** Today the profile form's the reset operation prefills the handle as
  Handle: tutor Profile public Handle or otherwise (**Profile implementation reference**), so a tutor with no handle gets an
  empty field and must invent a pseudonym before they can save. The fix is at that exact reset site:
  Handle: tutor Profile public Handle or otherwise suggested Public Handle or otherwise — consuming the new
  suggested Public Handle GET field (REQ, below) computed from display name (the
  0027b-backfill-provisional-public-handle "Wei Tan" → "Wei T." algorithm). This removes friction while
  keeping the field editable and still server-validated (no personal information/profanity, unique-continuous integration, max 40).
- **Owner (specialist):** software-engineer (autogen on profile bootstrap) · frontend-expert (prefill the
  editable input) · bff-specialist (GET returns the suggested handle)
- **Acceptance criteria (Gherkin):**


- Scenario: A tutor with no handle sees an auto-generated, editable one.
- When they open /tutor/profile.
- Then the Public handle field shows an auto-generated value (e g. "Wei T.").
- And the field remains editable and a different valid value can be saved.


- **Required evidence (defined up front):** Playwright screenshot **@1280 and @390** showing the prefilled
  handle on first load and after editing to a new value; curl log of GET returning the suggested handle and
  a subsequent successful PATCH to an edited handle.
- **Dependencies:** none

## REQ-011 — Reject an edited public handle that violates the handle rules
- **Type:** security (negative guard)
- **Priority:** Must
- **Source:** EARS "If…then" guard on REQ-010 (autogen must not weaken existing validation)
- **Rationale / value:** Auto-generation must not bypass the existing server-side guard. If a tutor edits the
  handle to leak personal information (email/phone/URL), use profanity, exceed 40 chars, or collide (unique-continuous integration), it must be
  rejected — the autogen path keeps the same protections.
- **Owner (specialist):** software-engineer (Profile Service validation) · security-expert (review)
- **Acceptance criteria (Gherkin):**


- Scenario: A phone-number handle is rejected.
- Given a tutor on /tutor/profile.
- When they edit Public handle to "[REDACTED PHONE]" and save.
- Then the response is 400/422 with the handle-validation error and the handle is not changed.


- **Required evidence (defined up front):** live curl log of the **rejected** edit (400/422 + error code);
  database query confirming the stored handle is unchanged.
- **Dependencies:** REQ-010

## REQ-012 — Verify the profile user interface already supports adding more than one subject
- **Type:** user interface (verification)
- **Priority:** Should
- **Source:** GH #197.3
- **Rationale / value:** Subject Level Matrix Editor + tutor profile subject levels already support **multiple**
  subject-level pairs (the editor imposes **no numeric cap** — it offers a per-subject level row for every
  taxonomy subject and re-emits the full pair-set), so #197.3 may already be satisfied. This REQ VERIFIES the
  running user interface lets a tutor add a 2nd subject and round-trips it; only if blocked does it convert to a fix (then a
  new REQ is raised).
- **Owner (specialist):** frontend-expert (verify) · qa-engineer (exploratory)
- **Acceptance criteria (Gherkin):**


- Scenario: A second subject can be added and saved.
- Given a tutor on /tutor/profile with one subject-level pair already present.
- When they add a second distinct subject-level pair and save.
- Then both pairs persist and reappear on reload.


- **Required evidence (defined up front):** Playwright screenshot **@1280 and @390** of the profile with two
  subject-level pairs after reload; database query of tutor profile subject levels showing both rows. If the user interface
  blocks a second add, attach the failing capture and STOP (raise a fix-REQ) — do not close as done.
- **Dependencies:** none

---

### Traceability summary

| REQ | Issue | Type | Priority | Owner using s | Status |
|-----|-------|------|----------|----------|--------|
| REQ-001 | #195 | user interface | Must | frontend-expert, uiux-designer | DRAFT |
| REQ-002 | #195 | user interface | Must | frontend-expert, uiux-designer | DRAFT |
| REQ-003 | #195 | data | Must | dba, software-engineer | DRAFT |
| REQ-004 | #195 | application programming interface | Must | bff-specialist, software-engineer | DRAFT |
| REQ-005 | #195 | constraint | Must | frontend-expert, software-engineer | DRAFT |
| REQ-006 | #195 | data | Must | dba | DRAFT |
| REQ-007 | #196 | user interface+data | Must | frontend-expert, dba, software-engineer, bff-specialist | DRAFT |
| REQ-008 | #197.2 | user interface+application programming interface | Must | frontend-expert, bff-specialist | DRAFT (was BLOCKED; founder ruled Option b) |
| REQ-008b | #197.2 | user interface+application programming interface+data | Must | bff-specialist, software-engineer, frontend-expert, uiux-designer | DRAFT (NEW capture page — full prefill set incl. quals/years) |
| REQ-008c | #197.2 | application programming interface (compat) | Must | software-engineer, bff-specialist | DRAFT |
| REQ-009 | #195 | security/data | Must | software-engineer, bff-specialist | DRAFT |
| REQ-010 | #197.1 | user interface+application programming interface | Must | software-engineer, frontend-expert, bff-specialist | DRAFT |
| REQ-011 | #197.1 | security | Must | software-engineer, security-expert | DRAFT |
| REQ-012 | #197.3 | user interface (verify) | Should | frontend-expert, qa-engineer | DRAFT |
