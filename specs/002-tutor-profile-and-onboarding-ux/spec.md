> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 002
slug: tutor-profile-and-onboarding-ux
title: Tutor profile and onboarding user experience
version: 2.3
status: HISTORICALLY MARKED IMPLEMENTED            # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: requirements-engineer         # the engineer who owns this change (requirements-engineer for cross-cutting/product specs)
created: 2026-06-20
updated: 2026-09-05
---

> **2026-09-05 requirements correction (documentation only).** Profile prefill now respects canonical profile ownership; the legacy availability mirror must remain synchronized. Migration references fold the recorded 0059/0059b/0060 allocation.
> The status and shipment/evidence entries below record the earlier implementation. This revision does not
> certify that the corrected contract is implemented, tested, or deployed; follow-up verification remains open.


# Tutor profile and onboarding user experience

Sharpens the tutor create-listing wizard and profile page from three founder-filed issues (#195/#196/#197):
(1) **fine-grained availability** — replace the coarse 3-period (Morning/Afternoon/Evening) 21-bit bitmask
with real selectable **1-hour and 2-hour** blocks, backed by a new tutor post availability slots junction
and a forward-migration of every existing post's bitmask (binding founder decision: a real data-model change,
not a CSS narrowing); (2) **multi-select Preferred MRT station/area** on the post (new
tutor post mrt stations junction, mirroring tutor profile planning areas); and (3) **profile polish** —
auto-generate the public handle (then editable), prepopulate the fields onboarding actually captured, and
verify the existing multi-subject editor. Value: tutors advertise their true availability and reach, and
re-enter less known data — improving match precision and onboarding completion. **The REQ-008 open question is
RESOLVED (founder ruled Option b, 2026-06-20):** onboarding now ALSO captures the full prefill set — hourly
rate, area, region, subject-levels, commitment AND qualifications + years (OPTIONAL — INT-043's
no-forced-completion intent preserved) and persists them onto Tutor Profile at registration (REQ-008b), so the
profile page can pre-populate the FULL set (REQ-008). This is a **partial reversal of INT-043** and needs **no
new migration** — every target Tutor Profile column already exists. **Honest scope note (v2.1):** today's
onboarding is a **2-page flow** (Register implementation reference → Commission implementation reference) with **no profile-capture page** — the
profile/listing stepper entries are labels only — so REQ-008b is a multi-task build (a NEW
onboarding-capture page/route + an extended in-memory draft + an extended register payload + data transfer object fields +
persistence wiring), not a small edit.

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC (each REQ-NNN)
- **Design:** [design md](design.md) — architecture, data model, error handling, testing, migration
- **Tasks:** [tasks md](tasks.md) — implementation tasks, each traced to a REQ-NNN + owning specialist
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario

## Status

APPROVED — scoped spec-review gate passed after two revision cycles (v2.0 option-b scope expansion → v2.1
grounding corrections, 2026-06-20). Cleared for build. **Boss build-time condition:** Slice C (onboarding-capture
page, REQ-008b/008c, tasks 18a–18e) is the highest-risk lane (twice mis-grounded at spec stage) — its first build
task gets explicit Boss verification against the running flow before the rest of Slice C proceeds.

## Review & Sign-Off

- [x] Counterpart — data / schema (dba) — SIGN-OFF (backfill windows+decode pinned, MRT col named, v2.1) / 2026-06-20
- [x] Counterpart — application programming interface contracts (bff-specialist) — SIGN-OFF (data transfer object names/types aligned to existing contract v2.1) / 2026-06-20
- [x] Counterpart — wizard / profile user interface (frontend-expert) — SIGN-OFF (Slice C re-grounded + re-costed v2.1) / 2026-06-20
- [x] Counterpart — user experience (uiux-designer) — SIGN-OFF (2h @390 default) / 2026-06-20
- [x] Technical Writer (docs) — SIGN-OFF / 2026-06-20
- [x] Boss (Rex) — APPROVED / 2026-06-20 (founder ruled Option b; v2.1 grounding confirmed; Slice C carries the build-time verification condition above).

## Changelog

- 2.2 (2026-06-21) HISTORICALLY MARKED IMPLEMENTED — built (backend+frontend parallel lanes), gated, evidence-complete, team-reviewed → Boss SHIP.
  backend (migrations **0059** tutor post availability slots + 0059b bitmask→hour backfill, **0060** tutor post mrt stations
  + backfill; Availability Slot model/data transfer object/validation; multi-MRT collection; onboarding-capture fields on
  Register Tutor Request + Auth Service persistence; suggested Public Handle): gate the documented operational check BUILD
  SUCCESS, 1908 unit tests, LINE 98.79% / BRANCH 97.51% (Boss-reverified). frontend (1h/2h Availability Grid w/ 2h-default
  @390; multi-MRT chips; NEW /register/profile onboarding-capture page + extended draft/Commission payload; profile
  full-set prefill + suggested-handle prefill): lint+build+test:ci 2415 tests, LINE 98.14% / BRANCH 92.63%.
  **Backward-compat proven:** legacy availability bitmask still derived (=129 live) + accepted; legacy
  location mrt station retained+synced; onboarding fields optional → omit still 201 (INT-043). Evidence under
  evidence implementation reference — REQ-003/004/006/007/008b/008c/009/010 (live psql + curl: backfill 219/219 posts 0-lost, slot+MRT
  round-trip, validation 400, onboarding persist + omit→201, suggested-handle) + REQ-001/002/005/007/008/008b/010
  Playwright @1280/@390 (grid, 2h-default-@390, multi-MRT chips, onboarding page, profile prefill, handle). REQ-007
  proven via live stack (Testcontainers can't run vs Docker 29.5.2 locally — follow-up F5). Closes #195/#196/#197.
  Tracked follow-ups: 0059b prod-cutover load note; pre-existing tutor profile * junction join-col misnamed
  tutor user id (holds profile PK) — tech-debt.
- 2.1 (2026-06-20) MODIFIED REQ-008b / REQ-006 / REQ-010 / REQ-012 + design + tasks — grounding corrections +
  honest re-costing after a scoped spec-review SEND-BACK (no scope added, no REQ removed). **(A)** Slice C
  re-grounded: onboarding is a **2-page flow** (Register implementation reference → Commission implementation reference) with NO capture page; the
  profile/listing stepper entries are labels only; Tutor Registration Draft carries only 4 fields and
  the Commission payload omits qualifications/years too. REQ-008b now explicitly captures the FULL prefill set
  **incl. qualifications + years Experience** (no onboarding source today). Task 18d split into **18d (NEW
  onboarding-capture page/route)** + **18e (extended draft + payload)** — re-costed as a multi-task build.
  **(B)** REQ-006/0058 backfill pinned as a resolved fact — windows Morning 08–12 / Afternoon 12–18 / Evening
  18–22, and the exact bitmask decode day is b/3, period is b%3 (DAYS day0 is Mon, SLOTS Morning/Afternoon/Evening).
  **(C)** Register Tutor Request field names/types aligned to the existing contract: location Planning Areas
  (not planning Areas), location Region/commitment as **enums** (not String); suggested Public Handle
  confirmed the only new RESPONSE field. **(D)** MRT junction join column named **tutor post id** (not the
  misleading tutor user id); the varchar operation matches legacy. **(E)** Dropped the false "up to 200
  subject-level pairs" claim — Subject Level Matrix Editor has no cap; REQ-012 stays verify-only. **(F)** REQ-010
  handle prefill pinned to **Profile implementation reference** Handle or otherwise suggested Public Handle or otherwise. **(G)** user experience
  decision: **2h granularity is the @390 default** (preserve INT-074 scroll affordance). **NO new migration**
  for Slice C confirmed. MINOR (grounding + re-costing, status stays DRAFT for Boss re-gate).
- 2.0 (2026-06-20) MODIFIED REQ-008 — founder ruled option (b): onboarding now captures rate/area/subjects to
  enable profile prefill; partial INT-043 reversal. REQ-008 unblocked + rewritten to prefill the FULL field set
  (rate/subjects/qualification/years/area); ADDED REQ-008b (onboarding captures + persists the new OPTIONAL
  fields onto Tutor Profile at registration) and REQ-008c (omitting them still registers — INT-043
  no-forced-completion compat). design md: Slice C reworked, Register Tutor Request + Auth Service register Tutor
  wiring documented, ADR-4 SUPERSEDED by ADR-5 (option b). **NO new migration** — all target Tutor Profile
  columns already exist (rate, planning-areas + subject-levels junctions, region, commitment). tasks md: added
  tasks 18a–18d (data transfer object/persistence/compat/wizard) and unblocked 18/19. MAJOR (scope expansion).
- 1.0 (2026-06-20) initial draft — REQ-001..012 authored from GH #195/#196/#197; REQ-008 BLOCKED on the
  prepopulation open question (Option a default vs b onboarding-capture); migrations 0058 (availability slots
  + bitmask backfill) and 0059 (multi-MRT). Counterparts: dba / bff-specialist / frontend-expert /
  uiux-designer / Technical Writer / Boss.
