# INT-053–059 — Build & Delivery Plan

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** plan (spec is team-reviewed + Boss-corrected) · **Date:** 2026-06-04
**Spec:** [int-053-059-admin-tutee-ux-wave md](int-053-059-admin-tutee-ux-wave.md) · **Tracker:** README implementation reference

## Context
The spec for INT-053–059 is approved to build **per wave** (Boss: Wave 1 = INT-056 first). This plan defines, for
each wave, the **five workstreams the founder requires — Build · Review · Docs · e2e · Postman** — who owns each
(delegated to the expert agent; orchestrator verifies every gate itself), the concurrency/collision map, and the
definition-of-done (gate-green → deploys locally → push on the **current branch**, concise message).

**Repos (polyrepo, see [[git-polyrepo-topology]]):** intuition-frontend (sp-dev), intuition-backend (sp-dev),
intuition-spec (sp-dev), e2e (main, Playwright vs the live stack). **Postman** (* implementation reference) lives in the
**unversioned wrapper** — edited but not pushed to any repo (note in reference/postman md).

## Global rules (every wave)
- **Delegate to the owner; verify the gate yourself.** frontend → frontend-expert (+ uiux-designer / frontend-designer);
  backend data transfer object/contract → bff-specialist (+ software-engineer, dba for query); each wave → /team-review → **Boss**.
- **Gates:** frontend the documented operational check (**≥97.5% line / ≥88% branch**, v8 summary);
  backend JAVA HOME is 21 mvn spotless:apply and mvn verify (**≥97.5% line+branch**, Docker up, surefire-only coverage).
  Capture exit codes on their own line (PIPESTATUS).
- **Deploy + reseed to dev after backend changes** (regression ITs + e2e run against the **live stack** at  — see
  [[regression-its-live-stack]]); frontend-only waves just need the documented operational check.
- **Definition of done per wave:** gate green (re-run by me) → deploys/preview locally → push each touched repo on
  its current branch (no branch change) with a concise message → docs/e2e/postman updated → Boss SHIP.
- **Lessons captured** in the owning SKILL md (+ SHARED-LESSONS md); Boss audits.

---

## WAVE 1 — INT-056: refresh-logout (Sev-1) · frontend-only · **ship first**
**Build (frontend-expert):**
- auth Store implementation reference: seed is Loading: yes (or add initialized:no). use Init Auth implementation reference: set
  initialized is yes in .finally. Require Auth implementation reference: while not-initialized render a spinner/no value;
  redirect only when initialized and !user. **Do not touch return To** (location-only, re-validated in Login implementation reference).
- **Audit the role guards** for the same !user-only gap — locate the tutor/admin guards (e g. Require Role/
  Require Admin) and apply the same wait-for-init; admin guard also depends on multi-factor authentication state.
**Review:** /team-review → Boss (focus: open-redirect safety + the initialized-state correctness + no flash).
**Docs:** reference/frontend-routes md — note the auth-bootstrap/guard behaviour (waits for /me before gating).
**e2e (auth-refresh spec implementation reference):** login as tutee → go to /tutee/interviews → the reload operation → assert
URL still /tutee/interviews (NOT /login) and content rendered; repeat for a tutor + an admin route; plus a
logged-out visitor → guarded URL → lands on /login?return To=… (safe relative).
**Postman:** none (no endpoint change).
**Gate + push:** frontend gate (incl. a "refresh-while-/me-in-flight" unit test); push intuition-frontend sp-dev +
e2e main. **Lesson:** auth-rehydration guard pattern → frontend-expert/SKILL md.

---

## WAVE 2 — Tutee user experience + form/layout quick wins · frontend-only
Items: **INT-057** (Find-Tutors tab), **INT-058** (applicants link), **INT-059** (reviews→Profile),
**INT-055** (qualifications dropdown), **INT-054** (Auth Card width).
**Concurrency / collision map (critical — same-file edits must be sequential):**
- nav Tabs implementation reference ← INT-057 (add tab) **and** INT-059 (remove Reviews tab) → **same file**.
- Profile implementation reference ← INT-055 (qualifications Select) **and** INT-059 (reviews section) → **same file**
  (+ tutee Profile).
- → **Lane A (one frontend agent, sequential):** INT-059 + INT-057 + INT-055 (they share nav Tabs + Profile).
- → **Lane B (parallel, worktree):** INT-054 Auth Card implementation reference (independent).
- → **Lane C (parallel, worktree):** INT-058 — the tutee My Posts/dashboard page link to /tutee/requests/:job Id/applicants (independent file).
- Lanes B/C may run in parallel via the documented operational check; **one final combined gate run** on
  the merged branch (avoid the two-vitest coverage race — single test:ci).
**Build:** frontend-expert (impl) + uiux-designer (IA: tab order, reviews-as-inline-section, "Find Tutors" label) +
frontend-designer (Auth Card size prop, remove arbitrary max-w- 420px). INT-055 uses existing
qualifications implementation reference + "Other" free-text fallback (legacy stored value not in list → preselect Other).
INT-059 keeps /tutee/reviews + /tutor/reviews as **redirects**.
**Review:** /team-review → Boss (focus: IA coherence, no-magic-value on Auth Card, legacy-qualification value).
**Docs:** reference/frontend-routes md (Find-Tutors tab, Reviews folded into Profile + redirects, applicants
link); domain/the private implementation folder + domain/uiux/wireframes md (nav/IA); domain/uiux/design-tokens md (Auth Card
sizes / removed arbitrary width).
**e2e (e2e/tests/):** tutee-find-tutors spec implementation reference (nav tab → /tutors list), tutee-applicants spec implementation reference
(My Posts → "View applicants" → applicants list), qualifications-select spec implementation reference (Select + Other path),
reviews-redirect spec implementation reference (/tutee/reviews → Profile section).
**Postman:** none (frontend-only).
**Gate + push:** frontend gate; push intuition-frontend sp-dev + e2e main.

---

## WAVE 3 — INT-053: admin tutor↔tutee + job, clickable · FULL-STACK
**Sequence: backend → deploy+reseed → frontend** (frontend needs the new fields; regression/e2e hit the live stack).
**Build backend (bff-specialist + software-engineer, dba on query):**
- Extend Admin Match Item Response implementation reference (+ Admin Interview Response) with tutor identity
  (tutor User Id, display name/public handle, tutor Post Id), tutee identity (tutee Name, tutee Post Id), job-post
  title. Update the mapper (Admin Match Service / interview admin service).
- **JOIN FETCH / @Entity Graph** the lazy tutor User/tutor Post/tutee Post (Match implementation reference) — no N+1.
  **Null-safe** tutor User (pre-0033 rows). data transfer object returned **only** from /admin/*.
- Test: list endpoint asserts a **constant query count** (statement-count assertion), not O using rows.
**Build frontend (frontend-expert + uiux-designer):**
- index implementation reference Admin Match Item + Admin Interview: add the new fields. Matching implementation reference:
  replace "Subject pair" with **Tutor ↔ Tutee** + **Job** columns, each a link to the existing detail route
  (/admin/posts/:tutee Post Id, /admin/matches/:id); subjects move to a secondary line/tooltip.
  Interview Queue implementation reference: make "Tutor Listing" a link + add a Job link. **Inline modals DEFERRED.**
**Review:** /team-review → Boss (focus: N+1 proof, personal information admin-only, null-safety, link targets).
**Docs:** reference/backend-api md (update Admin Match Item Response at ~line 240 + Admin Interview Response ~299
to the new fields); reference/frontend-routes md (admin tables now link to detail); modules/INDEX md + tracker.
**e2e (admin-match-detail spec implementation reference):** admin login → /admin/matching → row shows tutor/tutee/job →
click each → lands on the right detail route; same on /admin/interviews.
**Postman:** update read operation for admin matches + read operation for admin interviews **example responses** to the new
data transfer object shape in In Tuition postman collection implementation reference; sync reference/postman md (note: wrapper-local, not
pushed to a repo).
**Gate + push:** backend the documented operational check (re-run by me) → deploy+reseed → frontend gate → push intuition-backend sp-dev,
intuition-frontend sp-dev, e2e main. **Lesson:** admin-list JOIN FETCH (no N+1) → bff-specialist/dba SKILL.

---

## Critical files
- **frontend:** auth Store implementation reference, use Init Auth implementation reference, Require Auth implementation reference (+ role guards), nav Tabs implementation reference,
  Auth Card implementation reference, Profile implementation reference (+ tutee Profile), Matching implementation reference,
  Interview Queue implementation reference, tutee My-Posts/dashboard page, index implementation reference, qualifications implementation reference.
- **backend:** Admin Match Item Response implementation reference, Admin Interview Response implementation reference, the admin match/interview
  service + repository (JOIN FETCH), Match implementation reference, Interview implementation reference.
- **e2e:** new specs under e2e/tests/ (page objects under e2e/pages/, seed in seed implementation reference).
- **Docs:** the frontend-route, backend-interface and request-testing reference documents, domain/the private implementation folder,
  the wireframe and visual-style documents, modules/INDEX md, issues/README md.
- **Postman:** In Tuition postman collection implementation reference (wrapper-local).

## Verification (per wave + overall)
1. Gate re-run **by orchestrator** (not trusted): frontend 97.5/88; backend 97.5/97.5; paste the real summary.
2. Deploy+reseed; the new **e2e specs pass** against the live stack (BASE URL is http://localhost); INT-056's
   refresh spec is the headline.
3. Docs match shipped reality (frontend-routes nav, backend-api data transfer object fields); Postman example responses  equals  the new
   data transfer objects; reference/postman md count/baseline synced.
4. Each wave **Boss SHIP** with the capability-upgrade lesson captured; tracker rows flipped to BUILT.
5. Pushed per repo on the current branch (sp-dev / main); Postman noted as wrapper-local.

## Sequencing summary
**Wave 1 (INT-056)** → ship + verify refresh-stays-logged-in (highest trust ROI) → **Wave 2** (5 frontend quick wins,
Lanes A sequential / B+C parallel) → **Wave 3** (INT-053 full-stack, backend→deploy→frontend). Each wave is independently
shippable and Boss-gated.
