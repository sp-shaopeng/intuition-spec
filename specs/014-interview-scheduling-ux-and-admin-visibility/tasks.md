# Tasks — Interview scheduling user experience + admin visibility (014)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Boss dispatches each specialist. Mostly frontend; REQ-002/005 may add a response field (backend). Files: frontend
> Interview Queue implementation reference, Interview Detail implementation reference, the datetime control + Applicants screens;
> backend Admin Interview Response/Interview Response if a field is added. Evidence under the private evidence directory for the requirement.

## Governance gate (FIRST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 1 | Scoped /team-review (frontend-expert, uiux-designer, bff-specialist, software-engineer, tech-lead) + Boss audit → sign-off → status: APPROVED. Resolve REQ-001 grouping + REQ-004 picker choice | all | Boss | boxes checked; status APPROVED | [ ] |

## frontend lane (frontend-expert · uiux-designer)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 2 | REQ-001: redefine status Params so the tab union  equals  the full Interview Status enum (action=REQUESTED,PROPOSED,COUNTER PROPOSED,PENDING SCHEDULING,TUTOR DECLINED, scheduled=SCHEDULED, history=COMPLETED,CANCELLED,NO SHOW,**EXPIRED**); add an **enum-driven set-equality** contract test (compute the enum set at test time; assert union  equals  enum — not a frozen list) | REQ-001 | frontend-expert | Interview Queue implementation reference (52-57), status implementation reference | PENDING SCHEDULING in action tab; EXPIRED in history; union equals enum (set-equality) | [ ] |
| 3 | REQ-002: resolve the accepted slot from timeslots by agreed Timeslot Id (already on Admin Interview Response) and pre-fill the schedule datetime when present, empty otherwise — frontend-only | REQ-002 | frontend-expert | Interview Detail implementation reference | pre-fills accepted slot | [ ] |
| 4 | REQ-003: wrap/stack/group the action buttons so none clips at any viewport (incl. 390px) | REQ-003 | frontend-expert · uiux-designer | Interview Detail implementation reference (255-285) | all buttons visible at desktop + 390px | [ ] |
| 5 | REQ-004: usable validated datetime control (admin + tutee/tutor proposers); Singapore time defaults; past-datetime guard; unchanged wire format | REQ-004 | uiux-designer · frontend-expert | the datetime control component + its usages | usable, a11y, validated, format unchanged | [ ] |
| 6 | REQ-005: render tutor Display Name (not raw handle) on the admin Interview + Applicants screens; friendly fallback | REQ-005 | frontend-expert | Interview Detail implementation reference, Job Applicants implementation reference | name shown, not slug | [ ] |
| 7 | frontend tests for REQ-001..005 | REQ-001..005 | frontend-expert | *.test implementation reference | lint and build and test:ci green ≥97.5/88 | [ ] |

## backend lane

**NONE.** Both REQ-002 (agreed Timeslot Id+timeslots, Admin Interview Response) and REQ-005
(tutor Display Name, Admin Interview Response) read fields that **already exist** — verified against source at
spec-review. The v1.0 phantom "confirm/add if absent" backend task is REMOVED so the Boss does not dispatch a no-op backend
agent. This is an frontend-only spec.

## Evidence + Do D

| # | Task | Traces | Owner | Private evidence requested | Expected result | Status |
|---|---|---|---|---|---|---|
| 9 | Live Playwright: accepted interview in queue; pre-filled schedule; all buttons at 390px; usable picker; display-name on Interview/Applicants | REQ-001..005 | qa-engineer · frontend-expert | captures under evidence/ | all verified live | [ ] |
| 10 | Do D: backend (if changed) + frontend gates green; local deploy; live captures; e2e green; virtual machine re-confirm; commit+push current branch; close GH #210/#211 if founder verifies | all | software-engineer · frontend-expert | gates + push | green + pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 11 | /team-review (full) → Boss verdict; fix SEND-BACK in place; SHIP → status: HISTORICALLY MARKED IMPLEMENTED + requirements traceability matrix | all | Boss | Boss SHIP; requirements traceability matrix updated; evidence opened | [ ] |

## Endpoint-format correction — 2026-09-05

- [ ] REQ-004: test admin local scheduled At separately from proposal/counter Coordinated Universal Time slots. A shared picker must use endpoint-specific serializers and explicit Singapore time conversion; cover a non-Singapore time browser and Coordinated Universal Time agreed-slot prefill.
- [ ] REQ-001: retain TUTOR DECLINED in the action tab for follow-up, while asserting it is terminal and does not consume interview capacity. The tab is not a lifecycle-state classification.
