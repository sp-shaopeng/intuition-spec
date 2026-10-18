# Tasks — Mobile/responsive fixes (015)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Boss dispatches each specialist. frontend-only. Evidence under the private evidence directory for the requirement.

## Governance gate (FIRST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 1 | Scoped /team-review (frontend-expert, uiux-designer, qa-engineer, tech-lead) + Boss audit → sign-off → status: APPROVED. Resolve admin-nav pattern + the 390px e2e-guard scope | all | Boss | boxes checked; status APPROVED | [ ] |

## Build lane (frontend-expert · uiux-designer)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 2 | REQ-001: add min-w-0 to the **Step-1 wrapper  at Create Request implementation reference** (the CSS-grid item under Form Layout implementation reference's grid track) so Availability Grid implementation reference's overflow-x-auto engages; do NOT blanket min-w-0 on the shared Form Layout; grid markup byte-for-byte unchanged | REQ-001 | frontend-expert | Create Request implementation reference (393) [+ Create Listing implementation reference if its grid wrapper shows the same overflow] | body scroll Width ≤ client Width at 390px; grid scrolls internally | [ ] |
| 3 | REQ-002: admin nav mobile drawer (reuse Nav implementation reference hamburger pattern; Escape + focus-trap; all 8 sections) | REQ-002 | uiux-designer · frontend-expert | admin nav component, Nav implementation reference | usable drawer at 390px; a11y | [ ] |
| 4 | REQ-003: one nav-breakpoint policy across /assignments, /tutor/*, /tutor/dashboard | REQ-003 | frontend-expert | Nav implementation reference, Mobile Bottom Nav implementation reference, route layouts | consistent layout; no unreachable link | [ ] |
| 5 | frontend tests for REQ-001/002/003 + a 390px e2e overflow assertion that **advances to Step-2 (the availability step, the availability step, second in the wizard)** before asserting body scroll Width is at most client Width (the gap was checking only Step-1) | REQ-001..003 | frontend-expert · qa-engineer | *.test implementation reference, e2e/** | lint and build and test:ci green ≥97.5/88; e2e advances to grid step + green | [ ] |

## Evidence + Do D

| # | Task | Traces | Owner | Private evidence requested | Expected result | Status |
|---|---|---|---|---|---|---|
| 6 | Live Playwright at 390px: no page overflow on the grid step; admin drawer; consistent nav across the 3 routes | REQ-001..003 | qa-engineer · frontend-expert | captures under evidence/ | no overflow; nav usable + consistent | [ ] |
| 7 | Do D: frontend gate green; local deploy; live captures; e2e green (incl. the new 390px guard); virtual machine re-confirm; commit+push current branch | all | frontend-expert | gates + push | green + pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 8 | /team-review (full) → Boss verdict; fix SEND-BACK in place; SHIP → status: HISTORICALLY MARKED IMPLEMENTED + requirements traceability matrix | all | Boss | Boss SHIP; requirements traceability matrix updated; evidence opened | [ ] |
