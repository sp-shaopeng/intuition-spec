# Tasks — Auth robustness (011)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each task names the owning specialist, traces to a REQ, lists exact files, and a verify step. The Boss dispatches
> the specialist; the orchestrator does not build. **Two lanes now:** frontend lane (REQ-001/002/003 — shares
> api implementation reference/auth Store with spec 012, serialize on those files) + a small **backend lane** (REQ-004 duplicate-phone
> specific message — Auth Service implementation reference, no overlap with any other Wave-2 REQ). Check a box only when evidence is
> captured under the private evidence directory for the requirement.

## Governance gate (FIRST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 1 | Scoped /team-review on this spec (security-expert, software-engineer, frontend-expert, bff-specialist, tech-lead) + Boss audit → sign-off boxes checked → status: APPROVED. **REQ-004 founder decision is RESOLVED (override: make-it-specific + hardening) — security-expert signs off the constant-time/rate-limit guard at this gate; verify REQ-001 await-then-server-revoke ordering and HttpOnly-cookie guarantees before build.** | all | Boss | sign-off boxes checked; status APPROVED; security-expert hardening sign-off recorded | [ ] |

## Build lane (frontend; serialize with spec 012 on api implementation reference)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 2 | REQ-001: add logout-generation flag in api implementation reference; refresh Session/interceptor refresh branch no-op while logging out and suppress replay/store repopulation; logout = set flag → await existing refresh settlement → await server logout/revoke/expiry Set-Cookie → clear local state/caches → navigate; failure shows uncertain-server-sign-out and retry, without claiming cookie deletion | REQ-001 | software-engineer | api implementation reference, auth Store implementation reference | in-flight refresh during logout does not resurrect session | [ ] |
| 3 | REQ-001 security review: confirm no session-resurrection path remains; old refresh cookie rejected post-logout | REQ-001 | security-expert | review of #2 + backend /auth/logout//auth/refresh | reuse/invalid on old refresh after logout | [ ] |
| 4 | REQ-002: set correct autocomplete per field on login/register/reset/forced-change; ensure no autofill auto-submit | REQ-002 | frontend-expert | intuition-the private implementation folder** + forced-change form | attributes correct; no auto-submit on programmatic value-set | [ ] |
| 5 | REQ-003: add/confirm regression test that the unread-count poll is enabled: !user; keep BACKGROUND POLL ROUTES exemption | REQ-003 | frontend-expert | Notification Bell implementation reference, auth Redirect implementation reference, tests | logged-out → 0 fetches; logged-in → enabled | [ ] |
| 6a | REQ-004 (override): replace the registration phone conflict operation fold with a specific PHONE ALREADY REGISTERED exception at Auth Service implementation reference + 244-245; keep registration failed reason is phone conflict metric; backend tests for the specific code/message | REQ-004 | bff-specialist (contract) · software-engineer (impl) | Auth Service implementation reference, error-code consts, tests | specific code/message; metric preserved | [ ] |
| 6b | REQ-004 hardening: retain per-IP register rate-limit on the duplicate-phone path; make register handler constant-time/consistent-latency (no fast early-return on the phone branch); security-expert signs off residual risk; rate-limit + timing-consistency tests | REQ-004 | security-expert | Auth Service implementation reference register paths, tests | rate-limit holds; deterministic spies: exactly one bcrypt on phone-conflict, email-conflict and success; rate-limit precedes membership reads; no wall-clock assertion; sign-off recorded | [ ] |
| 6c | REQ-004 frontend: add PHONE ALREADY REGISTERED to the frontend error map so the register toast shows the specific copy | REQ-004 | frontend-expert | frontend error map / register form | specific toast on duplicate phone | [ ] |
| 7 | frontend tests for REQ-001/002/003 | REQ-001,002,003 | frontend-expert | *.test implementation reference | lint and build and test:ci green ≥97.5/88 | [ ] |

## Evidence + Do D

| # | Task | Traces | Owner | Private evidence requested | Expected result | Status |
|---|---|---|---|---|---|---|
| 8 | Live Playwright: logout-vs-refresh (no resurrection); forced-change autofill (no auto-submit) | REQ-001,002 | qa-engineer · frontend-expert | captures under evidence/ | session ends; no auto-submit | [ ] |
| 9 | Do D: backend gate (the documented operational check ≥97.5/97.5) + frontend gate green; local deploy; live captures; e2e green; virtual machine re-confirm; commit+push on current branch | all | frontend-expert · software-engineer | gates + push | green + pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 10 | /team-review (full panel) → Boss verdict; fix SEND-BACK in place; on SHIP set status: HISTORICALLY MARKED IMPLEMENTED + update requirements traceability matrix | all | Boss | Boss SHIP; requirements traceability matrix updated; evidence opened | [ ] |

## Corrected acceptance — 2026-09-05

- [ ] REQ-001: await in-flight refresh, then await server revoke plus expiry Set-Cookie for both HttpOnly credentials; verify original/rotated refresh rejection. No JavaScript cookie-expiry step.
- [ ] REQ-001 failure path: local caches cleared, automatic refresh suppressed, visible uncertain-server-sign-out state with retry. Do not assert server revocation on a failed call.
- [ ] REQ-004: retain specific phone 409, conflict metric and deterministic single-bcrypt/rate-limit-ordering assertions. Historical timing-measurement wording is superseded.
