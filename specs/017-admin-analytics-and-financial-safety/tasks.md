# Tasks — Admin analytics correctness + financial safety (017)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Boss dispatches each specialist. backend analytics calc + frontend confirm/labels. Evidence under the private evidence directory for the requirement.

## Governance gate (FIRST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 1 | Scoped /team-review (data-engineer, business-analyst, software-engineer, bff-specialist, frontend-expert, tech-lead) + Boss audit → sign-off → status: APPROVED. **Conversion is settled; record the pending exact average-commission time cohort before implementing that metric.** | all | Boss | boxes checked; status APPROVED; metric definitions recorded | [ ] |

## backend lane (data-engineer · software-engineer · bff-specialist)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 2 | REQ-001: implement the agreed cohort-consistent conversion (query + calc); guarantee [0,100%] + zero-requested guard | REQ-001 | data-engineer · software-engineer | Analytics Service implementation reference (145-166), repo query | calc ≤100%; cohort-consistent; zero guard | [ ] |
| 3 | REQ-002: PENDING approved time-cohort decision; then implement receipt-based numerator and distinct-match denominator from that same cohort plus zero guard | REQ-002 | data-engineer · software-engineer | Analytics Service implementation reference (181-207) | real per-match avg over one cohort; zero guard | [ ] |
| 4 | bff: confirm the analytics payload stays additive (field names stable or versioned) | REQ-001,002 | bff-specialist | analytics data transfer object | contract additive | [ ] |
| 5 | backend tests (the 400% case now ≤100%; avg over correct cohort; zero guards) | REQ-001,002 | software-engineer | Analytics Service Test | the documented operational check ≥97.5/97.5 | [ ] |

## frontend lane (frontend-expert)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 6 | REQ-003: Mark-Paid confirm dialog naming **amount + match/tutor + agency receipt of tutor commission**; route **all three** on Mark Paid handlers (199, 246, 312) through a pay Confirm Id dialog (no direct .mutate); mutation only on confirm; keep idempotency key (85) | REQ-003 | frontend-expert | Commission implementation reference (85, 199, 246, 312) | confirm gates pay at all 3 sites; cancel = no request | [ ] |
| 7 | REQ-002: after the pending cohort decision, label the exact received-commission population; do not present the proposal as approved | REQ-002 | frontend-expert | Analytics implementation reference (256) | label matches calc | [ ] |
| 8 | frontend tests for REQ-002 label + REQ-003 confirm | REQ-002,003 | frontend-expert | *.test implementation reference | lint and build and test:ci green ≥97.5/88 | [ ] |

## Evidence + Do D

| # | Task | Traces | Owner | Private evidence requested | Expected result | Status |
|---|---|---|---|---|---|---|
| 9 | Live: dashboard shows sane conversion % + corrected avg; Mark-Paid dialog before paying | REQ-001..003 | qa-engineer · frontend-expert | captures under evidence/ | metrics sane; pay confirmed | [ ] |
| 10 | Do D: backend + frontend gates green; local deploy; live captures; e2e green; virtual machine re-confirm; commit+push current branch | all | software-engineer · frontend-expert | gates + push | green + pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 11 | /team-review (full) → Boss verdict; fix SEND-BACK in place; SHIP → status: HISTORICALLY MARKED IMPLEMENTED + requirements traceability matrix | all | Boss | Boss SHIP; requirements traceability matrix updated; evidence opened | [ ] |
