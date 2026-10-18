# Tasks — Flag hygiene, seeder, frontend hygiene (018)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Boss dispatches each specialist. frontend-mostly + a tiny backend seeder change. Evidence under the private evidence directory for the requirement.

## Governance gate (FIRST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 1 | Scoped /team-review (frontend-expert, software-engineer, uiux-designer, tech-lead) + Boss audit → sign-off → status: APPROVED. Verify settled email-link/channel-scoped OFF behavior and the enumerated CTA scope | all | Boss | boxes checked; status APPROVED | [ ] |

## Build

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 2 | REQ-001: suppress WhatsApp verification toast/one-time passcode on profile/phone save while the flag is OFF; retain email-link pending copy and old phone until confirmed | REQ-001 | frontend-expert · software-engineer | Phone Settings Section implementation reference, Profile implementation reference | OFF → no WhatsApp toast/one-time passcode, email-link copy retained; old phone until confirm; ON WhatsApp behavior unchanged | [ ] |
| 3 | REQ-002: save Interview sets the get tutor operation; Dev Data Seeder Test ASSERTS every seeded interview has a non-null tutor user id  equals  its tutor-post's tutor (the shared invariant with create Interview) | REQ-002 | software-engineer | Dev Data Seeder implementation reference (1078), Dev Data Seeder Test | seeded interviews have tutor user id  equals  tutor-post tutor; test asserts it | [ ] |
| 4 | REQ-003: seed variety to the **measurable bar** — at least 8 eligible open posts, ≥8 subjects, ≥5 levels, ≥4 planning-areas across open showcase posts; no (subject,level,budget-bucket,area) tuple >20%; reaching the public strips | REQ-003 | software-engineer · data | Dev Data Seeder implementation reference | seeder/inspection assertion meets the thresholds | [ ] |
| 5 | REQ-004: /dashboard route → role dashboard or /login (before *) | REQ-004 | frontend-expert | App implementation reference (108,,), auth-redirect util | redirect per role + logged-out | [ ] |
| 6 | REQ-005: reconcile the "All optional" step header with actual required fields | REQ-005 | frontend-expert · uiux-designer | Create Request implementation reference (335) | accurate step copy | [ ] |
| 7 | REQ-006: route generic auth CTAs to /login (register reachable); keep create-account → /register | REQ-006 | uiux-designer · frontend-expert | landing/CTA components | generic CTA → login; create-account → register | [ ] |
| 8 | Tests: backend seeder; frontend flag-toast, /dashboard, step copy, CTA | REQ-001..006 | software-engineer · frontend-expert | seeder + *.test implementation reference | the documented operational check ≥97.5/97.5; lint and build and test:ci ≥97.5/88 | [ ] |

## Evidence + Do D

| # | Task | Traces | Owner | Private evidence requested | Expected result | Status |
|---|---|---|---|---|---|---|
| 9 | Live: flag-OFF phone change (no WhatsApp toast; email-link pending copy retained; old phone until confirm); reseed → tutor interviews non-empty + varied showcase; /dashboard redirect; CTA → login | REQ-001..006 | qa-engineer · frontend-expert | captures under evidence/ | all verified | [ ] |
| 10 | Do D: backend + frontend gates green; local deploy (+ reseed); live captures; e2e green; virtual machine re-confirm; commit+push current branch | all | software-engineer · frontend-expert | gates + push | green + pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 11 | /team-review (full) → Boss verdict; fix SEND-BACK in place; SHIP → status: HISTORICALLY MARKED IMPLEMENTED + requirements traceability matrix | all | Boss | Boss SHIP; requirements traceability matrix updated; evidence opened | [ ] |
