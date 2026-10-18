# INT-149…164 — live-virtual machine review batch (spec + fix plan)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** SPEC — pending team-review per wave before build
**Date:** 2026-06-16
**Source:** founder live-virtual machine quality assurance pass (http://[REDACTED IP ADDRESS]/); filed as GitHub issues #5–#20 +
mirrored in specs/issues/README md.
**Pipeline (per wave):** fix → /team-review → update docs → update e2e → update postman → deploy
local + virtual machine → push (sp-dev the private implementation folder, main e2e). Liquibase: reserve 0052/0053/0054 in order.

## Scope
**In scope (13):** INT-149, 150, 151, 152, 154, 155, 156, 158, 159, 160, 162, 163, 164.
**Founder-gated (excluded until a decision):** INT-153 (tutor subject→levels matrix — needs the
matching-vs-display + required-at-signup decision); INT-157 (commission "first month" calc — policy).
**Excluded:** WhatsApp/email integration. **Out of scope (separate triage):** the ~21 older
bulk-imported "OPEN" rows (INT-001…147) — their OPEN status is stale-from-import and must be triaged
against current code before any are touched (many are already deployed/fixed).

## Waves

### Wave 1 — frontend-only user experience/copy (one team-review, one frontend deploy)
**Build status (2026-06-17):** INT-155/156/164 shipped (frontend the recorded revision). INT-149/150/154/158/159
built + gates green (frontend the recorded revision — lint clean, 2188 tests, 98.07% line / 92.83% branch); pending
team-review → deploy → push. INT-162 deferred to live-virtual machine inspection (no leading space in source —
The phone settings control shows “Change number” when a number exists and “Add a number” otherwise, with no stray
space; the reported gap is a layout artifact to confirm on the running virtual machine). **No e2e/postman update
needed for Wave 1** — all changes are user interface-only and the /auth/register/tutor application programming interface contract is
unchanged (e2e registers via Actors implementation reference direct application programming interface, not the two-step user interface).

- **INT-155** (copy): Create Request implementation reference legend When are you available? → Tuition time slot.
- **INT-156** (bug): add a global Scroll To Top (use Location pathname effect → the scroll to operation)
  in App implementation reference so every navigation starts at top (fixes /register/commission landing mid-page).
- **INT-162** (copy): Phone Settings Section implementation reference — add the missing leading space before "Change number".
- **INT-164** (ux): profile-completeness bar uses bg-clay (reads as error) → positive/green fill
  (or graduate by %).
- **INT-149** (layout): Form Layout implementation reference sticky footer overlaps — make the in-page form submit inline
  (drop sticky) OR add a bottom spacer + scroll context (mirror Assignment Detail).
- **INT-150** (ux): tutor register — surface commission ONCE (not step1 fee-note + step2 terms) and
  don't re-collect the password in step 2 (carry state / acknowledge-only). Register implementation reference/Commission implementation reference.
- **INT-154** (bug): interview datetime inputs (Interview Queue implementation reference schedule + propose/counter) →
  min is now, step is 900 (15-min), max=+7d (mirror backend SLOT NOT ON GRID/window). Shared Slot Input.
- **INT-158** (bug): pre-validate phone uniqueness at register step 1 (or map the dup-phone 409 back to
  the account step) so the error doesn't surface on the terms step.
- **INT-159** (bug): back button on /register/commission should step back to the wizard (preserve state),
  not pop history to the signup page.

### Wave 2 — functional bug (investigate-first)
- **INT-163** (S2): tutor listing creation → "Unexpected error", listing not shown after submit.
  **Investigation (2026-06-17, against the running local stack):** the hard failure is NOT reproducible
  in current source. backend submit operation for tutor posts is robust across every input class — 201 for valid
  payloads incl. PUBLIC PLACE/BOTH (frontend↔backend venue/group enums agree); clean 400 for bitmask is 0
  ("Select at least one available time slot"), min>max ("Maximum rate must be at least the minimum
  rate"), and a label-not-code enum ("Request body is missing or malformed"); the admin-notify is
  after-commit + failure-isolated (Tutor Post Service implementation reference, never rolls back the post). The created
  PENDING listing IS returned by read operation for tutor posts and the dashboard renders it without crashing. frontend
  form mapping is sound (Multi Toggle stores labels → codes via require Venue Value/subject Code For Label;
  build Bitmask parses Availability Grid's Mon-Evening keys correctly). The founder hit it on a
  3-day-old virtual machine build (since redeployed with current source in Wave 1).
  **Root fix at the right altitude:** Create Listing on Error swallowed the server's specific message
  and showed a generic "Failed to submit listing" — the source of the opaque "Unexpected error"
  perception. Now surfaces the format api error operation (e g. "Maximum rate must be at least the minimum rate")
  so any failure is actionable; falls back to the generic only when the error carries no message.
  Regression tests added (frontend Create Listing test implementation reference). **Follow-up for founder:** if the error still
  reproduces on the redeployed virtual machine, please capture the exact inputs + the Network-tab response so the
  specific 4xx/5xx can be pinned. No e2e/postman change (user interface-only; create contract unchanged).

### Wave 3 — data / backend+database (each: migration + backend + frontend + tests + docs + e2e + postman + contract)
**Governance note:** INT-160 is a trivial validated reference-data addition (2 level rows + 2 frontend
entries); the full 18-seat team-review is reserved for the substantial Wave-3 items (INT-151 remove
student Relationship, INT-152 planning-area multi) and will run holistically over the wave.
**Environment note:** the Testcontainers IT layer cannot run in this session — the Docker socket
returns an empty /info (Status 400) to docker-java though the docker CLI works (3 strategies
tried: bg, DOCKER HOST-set, fg+sandbox-disabled). Per [[regression-its-live-stack]], backend+database changes
this wave are validated via the **live Liquibase migrate-then-boot** (backend container rebuild) + the unit
gate, not Testcontainers.
- **INT-160** (data) — **BUILT + VALIDATED (2026-06-17).** Liquibase 0052-add-secondary-na-levels
  (idempotent append records, sort order 45/48 keeping the O LEVEL EXP-then-NA grouping; applied FRESH by the
  backend Liquibase runner after deleting manual rows — recorded EXECUTED, rows present). frontend taxonomy implementation reference
  now lists all 18 codes (Express+Normal-Acad per secondary year, interleaved) + taxonomy test implementation reference
  locks the frontend↔backend mirror. backend unit 1814✓/coverage met; frontend 2199✓ 98.13/92.94. No e2e/postman/contract
  change (levels are reference DATA, not application programming interface enums; e2e uses P5,P6, unaffected).
- **INT-151** (remove student Relationship system-wide) — **BUILT + VALIDATED (2026-06-17).** Deleted
  the Student Relationship enum; removed the field from Tutee Post, 6 data transfer objects, 3 services, Dev Data Seeder,
  and ~25 backend tests; removed the frontend selector + types + format student Relationship Label + 3 consumers
  (Showcase Request Card/Review Detail/Assignment Detail) + 18 frontend tests. Liquibase **0053** (drop column then
  DROP TYPE, mirroring 0045) applied live (column+type count 0/0, backend healthy, changeset EXECUTED). e2e
  Actors post Body + postman bodies cleaned; OpenAPI snapshot regenerated via contract:update (now the
  tool's canonical pretty form — semantic delta is student Relationship-only, verified) + contract:diff
  clean. backend unit 1814✓/coverage met; frontend 2192✓ 98.12/92.98; live smoke: POST tutee post w/o relationship
  → 201, response clean. num Students retained. Done via parallel backend+frontend lane-agents (one crashed to a
  server 500 mid-run; resumed). Pending: holistic team-review → virtual machine deploy → push.
- **INT-152** (tutor-profile planning area → multi-select + "Island-wide") — **BUILT + VALIDATED (2026-06-17).**
  Scope = Tutor Profile ONLY (tutee/tutor *post* areas left single). backend: location Planning Area String →
  a collection of planning-area names on the profile; Liquibase migration 0054
  creates junction tutor profile planning areas (FK→tutor profiles automatically remove dependent records, composite PK),
  backfills the scalar (796 live rows), drops the column, with rollback; data transfer objects Update Tutor Profile Request/
  Profile and administrator-application responses return the selected planning areas as a list; the profile service
  + Job Application Service updated. frontend: tutor Profile implementation reference single combobox → Multi Toggle over
  Island-wide, …PLANNING AREAS (region-independent); local Tutor Profile Data field → string; no frontend
  display consumer reads the profile area (Tutor Detail uses MRT; cards use post data transfer objects). "Island-wide" is a
  plain string member (no enum). backend unit 1814✓/coverage met; frontend lint+build+2196✓ 98.12/92.99; 0054 applied
  live (column gone, junction exists); OpenAPI snapshot regen via contract:update (semantic delta =
  string→array on the 3 data transfer objects only) + contract:diff clean; e2e Actors post Body profile + profile spec
  → array. postman unaffected (its 2 refs are post-level). Done via parallel backend+frontend lane-agents. Pending:
  team-review → virtual machine deploy → push.

## Governance & gates (every wave)
/team-review (18-seat + Boss) on the wave → fold conditions → backend build and test verification
(≥97.5/97.5) + frontend lint/build/test:ci (≥97.5/88) + e2e specs green + contract:diff clean → deploy
dev (rebuild stack, reseed) + development host (rebuild the application and verify its externally reachable behaviour) → push.
Per [[sync-issues-with-github]]: on fixing each issue, gh issue close it (reason completed) AND flip
its tracker row to FIXED-DEV.

## Founder decisions needed (to unblock INT-153 / INT-157)
- INT-153: does the subject→levels matrix DRIVE matching/filtering, or display-only? required at signup?
- INT-157: how is "first month" sessions count fixed for the 50% commission, and recomputed on change?
