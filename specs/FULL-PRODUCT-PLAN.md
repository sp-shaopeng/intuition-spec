# InTuition — Full Product Plan

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Coverage thresholds and disclosure wording follow current decisions. Delivery checklists/statuses are historical snapshots, not current certification. See [current requirements](CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.

**Version:** 1.1  
**Date:** 2026-05-28  
**Status:** Active — Amended after Boss Review v1.0 (SEND BACK → 10 conditions resolved)  
**Previous status:** v1.0 SEND BACK by Rex. All 10 conditions addressed in this version.

---

## Governing Principles

1. **Plan first.** No code changes begin without a plan approved via team review + boss review.
2. **Compilability gate.** Both frontend (the documented operational check) and backend (mvnw package -Dskip Tests implementation reference) must pass before starting the next step. Broken builds block all progress.
3. **One module at a time.** Backend builds one module; team review + boss review; swagger + postman generated; frontend integrates; team review + boss review; move to next.
4. **Review is not optional.** Every deliverable — spec update, backend module, frontend integration, architecture change — gets a team review AND a boss review before the next step begins.
5. **Selection-first user interface.** All bounded-value form fields use dropdowns, toggle buttons, or grids. Free text only for genuinely open-ended content.
6. **Privacy invariant.** Contact info is never public or pre-assignment. Authorized admin access and assigned-participant disclosure follow product requirements document §20.6 and INT-125.

---

## Part 1 — Frontend Architecture Redesign

### 1.1 Current State Problems

| Problem | Impact |
|---|---|
| Flat components/ui/ and components/modules/ — no feature grouping | Unscalable; hard to add features without polluting shared space |
| No lib/ layer — no application programming interface client, no query Client config | Every future application programming interface call has no standard pattern |
| No hooks/ layer — data fetching logic will pollute page components | Hard to test, hard to reuse |
| No features/ isolation — auth logic, posting logic, admin logic all in pages/ | Cross-feature imports, no clear ownership |
| Missing primitives: no Select, no Modal, no Toast, no Skeleton | Every page will reinvent these |
| Tutee Layout wraps tutor routes — wrong role boundary | Auth guard cannot enforce TUTOR role |
| **No auth guard on protected routes** | **LIVE VULNERABILITY: any visitor can navigate to /admin/review, /tutee/dashboard etc. without authentication. Admin Layout does not check user role equals ADMIN. Tutee Layout has no null-user guard. This is the highest-priority defect.** |
| Login implementation reference is fully hardcoded dummy data | REDACTED email email string sets role — no real application programming interface call, no real signed session token |
| No /auth/me rehydration on app mount | Zustand state wiped on hard refresh — users become unauthenticated |

### 1.2 Target Folder Structure


Implementation example omitted. This public edition retains the written requirements in this section.


### 1.3 Migration Strategy

**Single baseline wave — not incremental.** The architecture migration happens once, as Phase 1 and Phase 2 work. By the time Module 2 (Posting) integration begins, all imports, folder paths, and layout wrappers must already match the target structure.

Incremental migration is prohibited. It produces a codebase where half the files use old patterns and half use new ones. New engineers copy from whatever file they open first. By Phase 4 you have four different import patterns and no linter can save you.

**Cutover rule:** Phase 2 exits only when every existing page has its imports updated to lib/, features/, and components/ paths. An ESLint no-restricted-imports rule banning modules implementation reference paths is added in Phase 1 and must pass from day one.

### 1.4 New Primitives Required Before Integration Begins

These components must exist before any backend integration work starts, because every page will need them for loading states, errors, and empty states:

| Component | Used for | When to build |
|---|---|---|
| Spinner | Button loading state, page-level loading | Phase 2 — before first integration |
| Skeleton | Card/table loading placeholders | Phase 2 — before first integration |
| Toast | Mutation success/error feedback | Phase 2 — before first integration |
| Empty State | Empty lists after real application programming interface calls | Phase 2 — before first integration |
| Error Boundary | Catch render errors from bad application programming interface responses | Phase 2 — before first integration |
| Modal | Confirm dialogs (approve, reject, create match) | Phase 3 — before Posting integration |
| Select | Use a consistently styled selection control | Phase 3 — before Posting integration |
| Textarea | Styled textarea with char count | Phase 3 — before Posting integration |

---

## Part 2 — Complete Page Inventory

### 2.1 All Pages (34 total)

| # | Route | Page Component | Status | Auth | Role Guard | Layout |
|---|---|---|---|---|---|---|
| 1 | / | Landing | ✅ Built | None | None | Public |
| 2 | /login | Login | ✅ Built | None | None | Public |
| 3 | /register | Register | ✅ Built | None | None | Public |
| 4 | /register/commission | Commission Disclosure | ✅ Built | None | None | Public |
| 5 | /forgot-password | Forgot Password | ❌ Missing (P0) | None | None | Public |
| 6 | /reset-password | Reset Password | ❌ Missing (P0) | None | None | Public |
| 7 | /tutors | Browse Tutors | ✅ Built | None | None | Public |
| 8 | /tutors/:slug | Tutor Detail | ✅ Built | None | None | Public |
| 9 | /requests | Browse Requests | ✅ Built (wrong layout) | Login gate | TUTOR | Tutor Layout |
| 10 | /rates | Rate Guide | ❌ Missing (P0) | None | None | Public |
| 11 | /tutee/dashboard | Tutee Dashboard | ✅ Built | Required | TUTEE | Tutee Layout |
| 12 | /tutee/posts/new | Create Request | ✅ Built | Required | TUTEE | Tutee Layout |
| 13 | /tutee/posts/:id | My Post Detail | ❌ Missing | Required | TUTEE | Tutee Layout |
| 14 | /tutee/interviews | Tutee Interviews | ✅ Built | Required | TUTEE | Tutee Layout |
| 15 | /tutee/interviews/:id | Interview Detail | ✅ Built | Required | TUTEE | Tutee Layout |
| 16 | /tutor/dashboard | Tutor Dashboard | ❌ Missing | Required | TUTOR | Tutor Layout |
| 17 | /tutor/listings/new | Create Listing | ❌ Missing | Required | TUTOR | Tutor Layout |
| 18 | /tutor/listings/:id | My Listing Detail | ❌ Missing | Required | TUTOR | Tutor Layout |
| 19 | /tutor/matches | Tutor Matches | ❌ Missing | Required | TUTOR | Tutor Layout |
| 20 | /admin | Admin Dashboard | ❌ Missing (→ /admin/review redirect currently) | Required | ADMIN | Admin Layout |
| 21 | /admin/review | Admin Review Queue | ✅ Built | Required | ADMIN | Admin Layout |
| 22 | /admin/review/:id | Admin Review Detail | ✅ Built | Required | ADMIN | Admin Layout |
| 23 | /admin/matching | Admin Matching | ✅ Built | Required | ADMIN | Admin Layout |
| 24 | /admin/matching/:id | Admin Match Detail | ❌ Missing | Required | ADMIN | Admin Layout |
| 25 | /admin/commission | Admin Commission | ✅ Built | Required | ADMIN | Admin Layout |
| 26 | /account | Account Settings | ❌ Missing | Required | Any | Current layout |
| 27 | * | Not Found | ❌ Missing | None | None | Public |

**Summary:** 15 built, 12 missing (including 3 P0 items).

### 2.2 P0 Pages (Boss SEND BACK — build before module queue)

1. /forgot-password — Forgot Password: email field → sends reset link email
2. /reset-password — Reset Password: token + new password fields  
3. /rates — Rate Guide: rate table by level and tutor type, public, no auth

---

## Part 3 — Backend Module Build Queue

Backend implements modules in this order. Each module is fully complete (migrations + service + tests + swagger) before the next begins.

### Module Order


The illustration for module order contains the following information:

- Auth (✅ DONE) → Posting → Approval → Matching → Interview → Commission.


### 3.1 Module 2: Posting

**Spec:** specs/modules/posting/  
**Priority:** This is the core marketplace action — without it nothing else can function.

**Liquibase migrations needed:**
- 0005-create-tutor-posts implementation reference — tutor posts table (id, tutor id FK, subjects[], levels[], rate min, rate max, venue types[], group sizes[], availability JSONB, qualifications, bio, status, rejection reason, created at, updated at, deleted at)
- 0006-create-tutee-posts implementation reference — tutee posts table (id, tutee id FK, subject, level, budget min, budget max, venue types[], availability JSONB, session duration, frequency, student count, relationship, special notes, status, rejection reason, created at, updated at, deleted at)
- 0007-create-posting-indexes implementation reference — indexes on (status), (tutor id), (tutee id), (subject, level)

**Key endpoints (from api-spec md):**


The illustration for module 2: posting contains the following information:

- POST tutor posts interface operation — create tutor listing (TUTOR).
- GET tutor posts interface operation — list own tutor posts (TUTOR).
- GET tutor posts :id interface operation — get own tutor post (TUTOR).
- PUT tutor posts :id interface operation — update draft (TUTOR, DRAFT only).
- DELETE tutor posts :id interface operation — soft delete (TUTOR, DRAFT only).
- POST tutee posts interface operation — create tutee request (TUTEE).
- GET tutee posts interface operation — list own tutee posts (TUTEE).
- GET tutee posts :id interface operation — get own tutee post (TUTEE).
- PUT tutee posts :id interface operation — update draft (TUTEE, DRAFT only).
- DELETE tutee posts :id interface operation — soft delete (TUTEE, DRAFT only).
- GET tutors interface operation — list APPROVED tutor posts (public, paginated).
- GET tutors :slug interface operation — get single APPROVED tutor post (public).
- GET requests interface operation — list APPROVED tutee posts (TUTOR auth, paginated).


**Coverage gate:** 97.5% line coverage (unit + integration tests)  
**Swagger output:** posting-api implementation reference  
**Postman:** posting implementation reference

### 3.2 Module 3: Approval

**Spec:** specs/modules/approval/

**Liquibase migrations needed:**
- 0008-create-post-reviews implementation reference — post reviews table (id, post id, post type ENUM, reviewed by FK, action ENUM APPROVED|REJECTED, rejection reason, reviewed at)

**Key endpoints:**


The illustration for module 3: approval contains the following information:

- GET admin review-queue interface operation — list PENDING posts (ADMIN, paginated, filterable by type).
- GET admin review :type :id interface operation — get full post with contact info (ADMIN).
- POST admin review :type :id approve interface operation — approve post (ADMIN).
- POST admin review :type :id reject interface operation — reject post with reason (ADMIN).


**Coverage gate:** 97.5%  
**Swagger output:** approval-api implementation reference

### 3.3 Module 4: Matching

**Spec:** specs/modules/matching/

**Liquibase migrations needed:**
- 0009-create-matches implementation reference — matches table (id, tutor post id FK, tutee post id FK, created by admin FK, status ENUM, matched at, updated at)

**Key endpoints:**


The illustration for module 4: matching contains the following information:

- GET admin matching tutors interface operation — search APPROVED tutor posts (ADMIN, filter by subject/level).
- GET admin matching requests interface operation — search APPROVED tutee posts (ADMIN, filter by subject/level).
- POST admin matches interface operation — create match (ADMIN) trigger commission stub creation.
- GET admin matches interface operation — list all matches (ADMIN, paginated with meta envelope).
- GET admin matches :id interface operation — get match detail (ADMIN).
- PATCH admin matches :id status interface operation — update match status (ADMIN).
- GET tutee matches interface operation — list own matches (TUTEE).
- GET tutor matches interface operation — list own matches (TUTOR).


**Commission stub (required in Module 4):** When submit operation for admin matches creates a match, it also creates a pending commission record with: match Id, tutor Id, status: PENDING, monthly Rate: no value, commission Rate: 0.50, commission Amount: no value. This stub record allows admin to later fill in the rate when Module 6 ships. The commission record ID is returned in the match creation response.

**Coverage gate:** 97.5%

### 3.4 Module 5: Interview

**Spec:** specs/modules/interview/

**Liquibase migrations needed:**
- 0010-create-interviews implementation reference — interviews table (id, match id FK, scheduled at, status ENUM, meeting link encrypted, notes, created at, updated at)

**Key endpoints:**


The illustration for module 5: interview contains the following information:

- POST tutee interviews :match Id interface operation — request interview (TUTEE).
- GET tutee interviews interface operation — list own interviews (TUTEE).
- GET tutee interviews :id interface operation — get interview (TUTEE).
- GET tutor interviews interface operation — list own interviews (TUTOR).
- GET tutor interviews :id interface operation — get interview (TUTOR).
- PATCH admin interviews :id schedule interface operation — schedule with time link (ADMIN).
- PATCH admin interviews :id complete interface operation — mark completed (ADMIN).
- PATCH admin interviews :id cancel interface operation — cancel (ADMIN).


**Coverage gate:** 97.5%

### 3.5 Module 6: Commission

**Spec:** specs/modules/commission/

**Liquibase migrations needed:**
- 0011-create-commissions implementation reference — commissions table (id, match id FK, monthly rate, commission rate, commission amount, invoice status ENUM, paid at, created at)
- 0012-create-commission-acknowledgements implementation reference — already in auth module migration (check if exists; skip if present)

**Key endpoints:**


The illustration for module 6: commission contains the following information:

- POST admin commissions interface operation — create commission record (ADMIN).
- GET admin commissions interface operation — list all (ADMIN, filterable).
- GET admin commissions :id interface operation — get detail (ADMIN).
- PATCH admin commissions :id pay interface operation — mark paid (ADMIN).
- GET admin commissions summary interface operation — totals: outstanding, collected, count.


**Coverage gate:** 97.5%

---

## Part 4 — Per-Module Integration Workflow

Every module follows this exact sequence. No step is skippable.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Part 5 — Immediate Work Queue (Before Module Queue Starts)

These fixes are required by the Boss SEND BACK. They must be done and pass boss review before any backend module work begins.

### Phase 1: P0 Missing Pages + Architecture Fixes

**Items in priority order:**

1. **Admin Layout role guard** — CRITICAL SECURITY FIX (LIVE VULNERABILITY)  
   File: Admin Layout implementation reference  
   Require a signed-in administrator before rendering this screen; otherwise send the visitor to sign-in.  
   Also add is Hydrated gate (see Phase 2 #4) — stub with user differs from undefined check until /me rehydration is built

2. **Tutee Layout auth guard** — the navigate operation  
   File: Tutee Layout implementation reference  
   Add: redirect unsigned-in visitors to login while preserving their intended destination

3. **Tutor Layout** — New layout, TUTOR role guard  
   File: Tutor Layout implementation reference  
   Move /tutor/requests (Browse Requests) from Tutee Layout to Tutor Layout in App implementation reference

4. **Forgot Password page** — email field, submits to submit operation for auth forgot-password  
   File: Forgot Password implementation reference  
   Backend endpoint already implemented; wire form → toast success message

5. **Reset Password page** — reads ?token=[REDACTED]] from URL query param + new password fields  
   File: Reset Password implementation reference  
   Backend endpoint: submit operation for auth reset-password

6. **Rate Guide page** — static rate table (no backend required), public  
   File: Rate Guide implementation reference  
   Rate data hardcoded from Singapore market benchmarks (SGD; Pri/Sec/JC by tutor type)

7. **Not Found page** — catch-all 404 route  
   File: Not Found implementation reference

8. **Spec fix** — specs/modules/posting/frontend md §3 relationship label must be Parents child (already correct in code; update spec to match)

9. **Wireframes for Tutor Dashboard and Create Listing** — delivered as text wireframes in specs/domain/uiux/wireframes md before Module 2 sprint planning. These are not implemented in Phase 1, but their design must be locked before implementation begins.

**Compilability check:** the documented operational check (zero TypeScript errors) + the documented operational check (all tests pass) required after Phase 1.

**Phase 1 exit criterion:** "Can an unauthenticated visitor no longer reach /admin/review or any admin route by navigating directly to the URL?"

**Review gate:** Team Review + Boss Review of Phase 1. Must receive ✅ SHIP IT before Phase 2.

### Phase 2: Infrastructure Only (No Feature Pages)

Once Phase 1 passes boss review, Phase 2 delivers the infrastructure that every future module integration depends on. Nothing else.

1. **application programming interface client** — api implementation reference
   - axios instance with base URL implementation reference
   - signed session token is delivered via HttpOnly cookie — no local Storage, no Zustand signed session token storage
   - 401 interceptor: dispatch Zustand the logout operation action + emit auth:unauthorized custom event (NOT direct the navigate operation inside interceptor — router listens to the event)
   - Api Error interface: the response status, error code, message and optional field-specific errors
2. **Query client** — query Client implementation reference (stale Time 30s, retry 1, global on Error → toast)
3. **Query key factories** — query Keys implementation reference — typed factories for every feature: tutor Keys, request Keys, match Keys, approval Keys, interview Keys, commission Keys
4. **/me rehydration** — use Init Auth implementation reference — called in App implementation reference on mount, calls read operation for auth me, populates Zustand user, sets is Hydrated: yes; all layout guards wait for is Hydrated before evaluating user
5. **Primitives** — Spinner, Skeleton (shimmer), Toast (6s, pause-on-hover, aria-live), Empty State, Error Boundary, Modal (focus trap, aria-modal, aria-labelledby, two-level confirm support)
6. **Pagination strategy decision** — committed in pagination implementation reference:
   - Use **offset-based pagination** (simpler for admin use cases; cursor adds complexity not justified at this scale)
   - Standard envelope: a list of results with page number, page size, total records and total pages
   - All list endpoints and Tan Stack Query use Query hooks use this shape
7. **ESLint migration boundary rule** — no-restricted-imports banning modules implementation reference and modules implementation reference paths; all existing pages migrated to new lib/, features/, components/layout/ imports

**Explicitly NOT in Phase 2:**
- Admin Dashboard — deferred until there is real data to show (post-Module 3)
- Account Settings — deferred until auth integration complete (post-Module 2)
- Tutor Matches, Tutor Dashboard — deferred to Module 4 and Module 2 integration respectively
- Any feature page that has no backend data yet

**Phase 2 exit criterion:** "Can a user load the app, have their session restored from the server, and be correctly redirected to /login if unauthenticated?"

**Review gate:** Team Review + Boss Review of Phase 2. Must receive ✅ SHIP IT before any backend module integration begins.

---

## Part 6 — Review Gates Checklist

Every team review + boss review must verify all applicable items below.

### Backend Module Gate

- [ ] Liquibase migrations run cleanly on a clean schema (no conflicts with prior migrations)
- [ ] All endpoints match the spec in the relevant module's backend design document
- [ ] No contact data (email, phone) returned in non-admin endpoints
- [ ] Soft deletes implemented correctly (deleted at IS NULL filter on all queries)
- [ ] Test coverage ≥ 97.5% line **AND** branch coverage (verified by JaCoCo report)
- [ ] PIT mutation score ≥ 80% (verified by Pitest report) — JaCoCo alone is not sufficient
- [ ] **Module 1 (Auth) Personal Data Protection Act gate:** data erasure endpoint exists; consent logged at registration; audit log records all admin contact-field access
- [ ] **Module 3 (Approval) Personal Data Protection Act gate:** rejection reason stored; no contact data in notification email body (deep link only); audit log on every approve/reject action
- [ ] **Module 4 (Matching) Personal Data Protection Act gate:** no tutee id in analytics event properties; contact reveal logged in audit log; meeting links never returned in list endpoints
- [ ] Analytics events defined in module's Analytics Contract (see Part 7) and wired via Analytics Event Publisher; Personal Data Protection Act compliance of each event payload verified
- [ ] mvnw package implementation reference produces a runnable JAR (**COMPILABILITY GATE**)
- [ ] Swagger YAML generated (springdoc-openapi-maven-plugin with app running on test profile) and matches endpoint spec
- [ ] Postman collection generated (npx openapi-to-postmanv2 -s swagger implementation reference -o postman implementation reference) with the configured service address environment variable
- [ ] No secrets in code, migration files, or test fixtures

### Frontend Integration Gate

- [ ] All dummy data for this module replaced with Tan Stack Query hooks using query Keys implementation reference factories
- [ ] Loading state: Skeleton or Spinner shown while query is pending
- [ ] Error state: Toast or Empty State shown on query error
- [ ] Mutation feedback: success/error Toast on create/update/delete
- [ ] Auth guards present and correct for all protected routes
- [ ] Role guards enforced: TUTEE pages not accessible to TUTOR and vice versa
- [ ] Privacy invariant: no email/phone rendered on public or tutor pages (verify in JSX — no @ or +65 in test output)
- [ ] the documented operational check produces zero TypeScript errors (**COMPILABILITY GATE**)
- [ ] the documented operational check all tests pass
- [ ] Frontend Vitest coverage ≥ 97.5% lines, ≥ 88% branches (enforced via vite config implementation reference thresholds block — build fails if below)
- [ ] New pages have tests; privacy invariant test present for every page that renders tutor/tutee data
- [ ] All imports use new lib/, features/, components/layout/ paths (no modules implementation reference — ESLint rule enforced)

### Boss Review Criteria (applies to every gate)

- [ ] Claims verified against actual files (not agent summaries)
- [ ] Business readiness: would a Singapore parent trust this today?
- [ ] No sugarcoated issues — anything fatal is marked fatal
- [ ] Verdict: ✅ SHIP IT before proceeding

---

## Part 7 — Analytics Contract (Gated Before Module 3)

The analytics architecture artifact must be delivered and reviewed before Module 3 (Approval) sprint planning begins. It is not optional.

### 7.1 Analytics Architecture

- **Publisher:** Analytics Event Publisher implementation reference — already exists; events are published via this class
- **Storage:** Analytics events written to an analytics events table (or forwarded to an external sink — decision to be made in analytics contract spec)
- **Personal Data Protection Act constraints (non-negotiable):**
  - No personal information in event properties: no email, no phone, no full name
  - No tutee id in interview requested event
  - No meeting link in any event property
  - 90-day anonymization: after 90 days, user id in events replaced with hash

### 7.2 Required Events Per Module

| Module | Event | Trigger | Key Properties | Personal Data Protection Act Check |
|---|---|---|---|---|
| Phase 2 | rate guide viewed | /rates page load | none | ✅ no personal information |
| Phase 2 | tutor filter applied | filter change on /tutors | the subject and level | ✅ no personal information |
| Module 2 | posting created | tutor/tutee post saved as DRAFT | the post type, subject and level | ✅ no personal information |
| Module 2 | posting submitted | post status → PENDING | the post type, subject and level | ✅ no personal information |
| Module 3 | posting approved | admin approves | the post type | ✅ no personal information |
| Module 3 | posting rejected | admin rejects | the post type and rejection category | ✅ no personal information |
| Module 4 | match created | admin creates match | the subject and level | ✅ no tutee id |
| Module 5 | interview requested | tutee requests interview | the subject and level | ✅ no tutee id, no link |
| Module 5 | interview scheduled | admin schedules | the subject and level | ✅ no link |
| Module 6 | commission recorded | admin records payment | the commission amount band | ✅ no personal information |

### 7.3 Analytics Contract Delivery

Before Module 3 sprint planning, deliver specs/domain/analytics-contract md containing:
- Full event register (above table expanded with property types and constraints)
- Personal Data Protection Act compliance sign-off per event
- Anonymization implementation plan (90-day job)
- Analytics Event Publisher wiring pattern (how services call it)

---

## Part 8 — Full Module Build Timeline


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Part 9 — Component Programming Standards

All new React components must follow these standards to ensure extensibility.

### React Version Decision

**Targeting React 18 with forward Ref.** Do NOT upgrade to React 19 mid-project. The forward Ref deprecation in React 18.3 is a warning, not a breaking change. React 19 refs-as-props is cleaner but introduces a required migration for existing components. Commit to React 18 patterns now; schedule React 19 upgrade as a post-launch tech debt item.

### TypeScript


Implementation example omitted. This public edition retains the written requirements in this section.


### Composition over configuration


Implementation example omitted. This public edition retains the written requirements in this section.


### Data fetching pattern


Implementation example omitted. This public edition retains the written requirements in this section.


### Auth guard pattern


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Part 10 — Compilability Gate Protocol

Before starting any new step, the current step's code must compile cleanly.

**Frontend check:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Backend check:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Environment requirement:**
- Frontend: .env implementation reference with VITE application programming interface BASE URL is http://localhost (committed as .env implementation reference)
- Backend: application-local implementation reference with local database credentials (NOT committed; .gitignore'd)

If either check fails, the current step is **not complete**. The next step does not begin.

---

*This plan supersedes any prior partial plans. All team members operate under this plan.*
