# Tasks — Error copy + handling (016)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Boss dispatches each specialist. backend (no-handler 404) + frontend (copy/fields/tel:). Evidence under the private evidence directory for the requirement.

## Governance gate (FIRST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 1 | Scoped /team-review (software-engineer, frontend-expert, bff-specialist, qa-engineer, tech-lead) + Boss audit → sign-off → status: APPROVED. Verify settled frontend-safe-copy ownership (backend scrub optional) and REQ-002 no-handler config safety | all | Boss | boxes checked; status APPROVED | [ ] |

## backend lane (software-engineer · bff-specialist)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 2 | REQ-002: add No Handler Found/No Resource Found → 404 handler; enable throw-if-no-handler (+ add-mappings is no if needed); confirm spec-010 trace Id/header + SPA fallback unaffected; keep 405 | REQ-002 | software-engineer | Global error handler implementation reference, application* implementation reference/properties | unknown path → 404+trace Id; 405 unchanged; no SPA regression | [ ] |
| 3 | REQ-001 (optional backend): keep error CODEs; confirm no consumer depends on raw not-found message text (so the frontend map is safe) | REQ-001 | bff-specialist | service throw-sites (read), error contract | codes stable; message-text not a contract | [ ] |
| 4 | REQ-003 (backend): confirm Create Admin Request validation yields a field Errors entry per required field | REQ-003 | bff-specialist | Create Admin Request implementation reference, GEH 161 | field Errors cover required fields | [ ] |
| 5 | backend tests for REQ-002 (404 handler) | REQ-002 | software-engineer | GEH tests | the documented operational check ≥97.5/97.5 | [ ] |

## frontend lane (frontend-expert)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 6 | REQ-001: extend the code→friendly-copy map; unknown codes use a safe generic fallback, never raw server text; keep trace Id Reference when present | REQ-001 | frontend-expert | api Error implementation reference, toast handler | mapped and unmapped errors show safe copy; no raw uuid/message; Reference preserved when present | [ ] |
| 7 | REQ-003: surface field Errors per field on the create-admin form | REQ-003 | frontend-expert | the create-admin form page | missing field named | [ ] |
| 8 | REQ-004: safe tel/mailto href helper; apply at Contact Reveal Card + other sites | REQ-004 | frontend-expert | Contact Reveal Card implementation reference (29) + grep tel/mailto | hrefs encoded; display intact | [ ] |
| 9 | frontend tests for REQ-001/003/004 | REQ-001,003,004 | frontend-expert | *.test implementation reference | lint and build and test:ci green ≥97.5/88 | [ ] |

## Evidence + Do D

| # | Task | Traces | Owner | Private evidence requested | Expected result | Status |
|---|---|---|---|---|---|---|
| 10 | Live: curl unknown path → 404 (+trace Id); humanised toast; create-admin field error; tel: with spaces | REQ-001..004 | qa-engineer | captures under evidence/ | all verified | [ ] |
| 11 | Do D: backend + frontend gates green; local deploy; live captures; e2e green; virtual machine re-confirm; commit+push current branch | all | software-engineer · frontend-expert | gates + push | green + pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 12 | /team-review (full) → Boss verdict; fix SEND-BACK in place; SHIP → status: HISTORICALLY MARKED IMPLEMENTED + requirements traceability matrix | all | Boss | Boss SHIP; requirements traceability matrix updated; evidence opened | [ ] |
