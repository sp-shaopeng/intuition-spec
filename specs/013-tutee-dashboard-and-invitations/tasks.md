# Tasks — Tutee dashboard + invitations (013)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Boss dispatches each specialist. backend adds the invitations read endpoint and completes any required revise/resubmit support; frontend adds dashboard actions/surfaces. Files: backend
> Job Invitation Service + a tutee invitations controller endpoint; frontend Dashboard implementation reference,
> Tutor Detail implementation reference, the create-post wizard (edit mode). Check a box only with evidence under
> the private evidence directory for the requirement.

## Governance gate (FIRST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 1 | Scoped /team-review (bff-specialist, software-engineer, frontend-expert, uiux-designer, tech-lead) + Boss audit → sign-off → status: APPROVED. Verify the effective editable-status contract and the already-decided outbound-list IA | all | Boss | boxes checked; status APPROVED | [ ] |

## backend lane (bff-specialist · software-engineer · dba)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 2 | REQ-003: add read operation for tutee invitations LIST to Tutee Invitation Controller (own, default PENDING, ?status is ALL, soft-delete-aware, paginated newest-first) + service query by created By; **reuse the existing Job Invitation Response data transfer object as-is** (no new data transfer object); add TUTEE INVITATIONS to Api Routes implementation reference + tutee invitations to api Routes implementation reference | REQ-003 | bff-specialist · software-engineer | Tutee Invitation Controller, Job Invitation Service, Job Invitation Repository (find By Created By…), Api Routes implementation reference, api Routes implementation reference | own-only; authz; soft-delete excluded by default; reuses Job Invitation Response | [ ] |
| 3 | REQ-003: DBA — add Liquibase changeset for idx jobinv createdby status created by, status so the own-only filtered list is index-backed (no existing index covers created by) | REQ-003 | dba | db/changelog/ (new changeset) | index present; the documented operational check applies it | [ ] |
| 4 | backend tests for the sent-invitations endpoint: own-only authz (A never sees B); default excludes WITHDRAWN/soft-deleted, ?status is ALL includes history; extend Job Invitation Response Pii Contract Test for the LIST path | REQ-003 | software-engineer | service/controller tests, Job Invitation Response Pii Contract Test | the documented operational check ≥97.5/97.5 | [ ] |

> REQ-001 now follows the product requirements document plus 021: DRAFT, REJECTED, REVISION REQUESTED, PENDING and ACTIVE are editable.
> The earlier source-code exclusion of REVISION REQUESTED is historical, not a waiver.

- [ ] REQ-001 backend: complete any missing own-post revise/resubmit support, preserve explicit submission and approval,
  and test all eight statuses plus a concurrent MATCHED transition. Owner: software-engineer/bff-specialist.

## frontend lane (frontend-expert · uiux-designer)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 5 | REQ-001: dashboard View + Edit actions; Edit offered ONLY for the amended editable set of draft, rejected, revision-requested, pending and active posts (read shared POST STATUS constant); revision edit exposes reason and explicit resubmit; PENDING stays under review; ACTIVE-edit warns it re-enters approval; edit reuses the wizard, pre-fills from post Detail, submits PUT; surface 409 INVALID STATE (never silent) | REQ-001 | frontend-expert | Dashboard implementation reference, create-post wizard, status implementation reference | edit round-trips; only editable statuses offer Edit; ACTIVE warning; 409 surfaced | [ ] |
| 6 | REQ-002: render all venues (list "+N more" / detail full) instead of 0 | REQ-002 | frontend-expert | Dashboard implementation reference (94) | all venues represented | [ ] |
| 7 | REQ-003: replace session-only invited Post Ids with a query against the new endpoint (server-backed, not session); render the "Invitations sent" sub-group in the new outbound section; withdraw on the persisted row | REQ-003 | frontend-expert | Tutor Detail implementation reference (74-75), Dashboard implementation reference | Invited state survives refresh (server-backed) | [ ] |
| 8 | REQ-004: render the dedicated "Your outbound requests" section below "My posts" (Dashboard implementation reference) with the Invitations-sent + Interview-requests sub-groups (each LISTS rows with status/date/link), incl. empty-state hints | REQ-004 | frontend-expert · uiux-designer | Dashboard implementation reference | outbound section lists invitations + interview requests with status | [ ] |
| 9 | frontend tests for REQ-001/002/003/004 | REQ-001..004 | frontend-expert | *.test implementation reference | lint and build and test:ci green ≥97.5/88 | [ ] |

## Evidence + Do D

| # | Task | Traces | Owner | Private evidence requested | Expected result | Status |
|---|---|---|---|---|---|---|
| 10 | Live Playwright: edit round-trip; 3-venue display; invite→refresh→persists; interview-request→dashboard | REQ-001..004 | qa-engineer · frontend-expert | captures under evidence/ | all surfaced + durable | [ ] |
| 11 | Do D: backend + frontend gates green; local deploy; live captures; e2e green; virtual machine re-confirm; commit+push current branch | all | software-engineer · frontend-expert | gates + push | green + pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 12 | /team-review (full) → Boss verdict; fix SEND-BACK in place; SHIP → status: HISTORICALLY MARKED IMPLEMENTED + requirements traceability matrix; close GH #207/#208 if founder verifies | all | Boss | Boss SHIP; requirements traceability matrix updated; evidence opened | [ ] |
