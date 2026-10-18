# INT-053–059 — Admin visibility + Tutee user experience + auth-refresh fix (spec)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** spec (needs /team-review → Boss before build) · **Date:** 2026-06-03
**Owners:** frontend-expert (lead) + uiux-designer + bff-specialist (INT-053 data transfer objects) + frontend-designer (INT-054)
Builds on the INT-049 design-system + layout primitives. **6 frontend-only + 1 full-stack (INT-053).** No schema/
migration changes.

## Problem / context
A founder walkthrough of the live dev build surfaced a themed batch: admins and tutees can't *see* or *reach* the
entities they care about (who/what is behind a match or interview; the tutors who applied to a job; how to search
tutors), one **Sev-1 auth bug** (refresh logs you out), and form/layout polish. Each is recorded as an issue
(INT-053…059) in specs/issues/README md; this spec defines the changes, grounded in the real code.

---

## INT-053 — Admin tables: show tutor↔tutee + job posting, each clickable to detail (FULL-STACK)
**Current:** /admin/matching "Match Lifecycle" renders only the subject pair (Matching implementation reference,
the tutor's subject and the tutee's subject). /admin/interviews queue (Interview Queue implementation reference)
links Tutee → interview detail only; "Tutor Listing" is plain text; no job link.
**Why it's full-stack:** the list data transfer objects lack the fields. The entities already carry them —
Match has tutor User/tutor Post/tutee Post (Match implementation reference); Interview has
tutor Post/tutee Post (Interview implementation reference) — so this is **data transfer object + mapper enrichment, no schema change**.
**Proposed:**
- backend: extend Admin Match Item Response implementation reference with tutor identity (tutor User Id + display name/public
  handle, tutor Post Id), tutee identity (tutee Name, tutee Post Id), and the job-post title; update the mapper
  (Admin Match Service). Add tutee Post Id (+ tutor handle) to the admin interview response.
- backend (corrected — Boss): the relations are the many to one operation (Match implementation reference), so the enriched list mapper
  **must JOIN FETCH / use an entity-graph** (tutor User, tutor Post, tutee Post) — a per-row lazy load is **N+1**
  across a paginated admin list. **Null-safe the tutor identity** — tutor User can be null on pre-0033 rows
  (resolve Tutor User Id fallback). The new fields are returned **only from /admin/*** endpoints.
- frontend (v1 — Boss ruling 2026-06-04): replace the Matching "Subject pair" column with **Tutor ↔ Tutee** + a **Job**
  column; in Interview Queue make "Tutor Listing" a link and add the job. **All three clickable via EXISTING routes
  (no new views, modals still DEFERRED):**
  - **Tutor → /admin/review/:tutor Post Id** — Admin Review Detail fetches the generic read operation for admin posts id and
    branches on post Type equals TUTOR (Review Detail implementation reference), so it renders the tutor listing. **Null-safe:**
    free-market matches (INT-034 re-key) can have tutor Post Id is no value → render the tutor name/handle as plain TEXT
    (no dead link).
  - **Tutee → /admin/review/:tutee Post Id** (the job/tutee post detail, same generic page).
  - **Job posting → /admin/posts/:tutee Post Id/applicants** (Admin Job Applicants — higher-value admin triage).
  - Interview queue mirrors this (tutor Post Id already present; job uses the new tutee Post Id).
  - **frontend must verify:** Admin Review Detail renders sanely for an already-approved/MATCHED post — the Approve/Reject
    block must be hidden/disabled when status isn't pending (don't offer "Approve" on a live match).
**Acceptance:** admin sees who↔who + the job on both tables; clicking any opens its existing detail route; subjects
still available (tooltip/secondary line); personal information unchanged (admin already sees tutee names — Interview Queue implementation reference);
a list test asserts a **constant query count** (no N+1).

## INT-054 — Auth Auth Card too narrow on desktop (frontend-only)
**Current:** Auth Card implementation reference is a fixed mx-auto w-full max-w- 420px for every auth page →
the multi-field Register-as-Tutor flow (Register implementation reference) is cramped with big side margins, and short cards
(forgot-password "Check your email", Forgot Password implementation reference) look small/lost on a large viewport.
**Proposed (v1 — Boss):** add a size/width prop (sm | md | lg) to Auth Card; the long register flow
uses a wider card, short forms (login/forgot) keep a tidy default. Replace the arbitrary max-w- 420px with a
**token/standard Tailwind max-w** (it slipped the INT-049 audit). **The 2-column register layout is DEFERRED**
(gold-plating — the size-prop solves the cramping). **Acceptance:** register is comfortable on a laptop (no
oversized side gaps); login/forgot stay tidy; no arbitrary value; works at 375/768/1280.

## INT-055 — Qualifications: free textarea → dropdown (frontend-only)
**Current:** free a multi-line text field (Profile implementation reference, zod the min operation; same in the
/register/commission step). **Proposed:** a Select backed by the **existing** qualifications implementation reference
QUALIFICATION OPTIONS (submits the chosen label verbatim into the existing qualifications string — **no backend
change**), with **"Other" → free-text fallback**; update zod (select required; free text only when "Other").
**Acceptance:** qualification is a picker; "Other" reveals a text input; existing free-text values still display.

## INT-056 — Refresh on a protected page logs you out (Sev-1, frontend-only)
**Current (root cause corrected after grounding — the spec earlier blamed Require Auth; the real gate is the
layouts):** protected routes are gated by the **layouts** — Tutee Layout implementation reference (if using !user Navigate
/login?return To=…Tutor Layout implementation reference Admin Layout implementation reference— not by Require Auth (which only wraps
/tutors/:id + /assignments/:id). Root Layout implementation reference **already** gates the router Outlet on is Loading
(show a loading indicator until the nested screen is ready) and mounts the use init auth operation. **The race:** auth Store implementation reference defaults
is Loading:no, and use Init Auth flips it true only inside a use Effect (which fires *after* the first
render). So on the first synchronous render Root Layout renders <Outlet/ (not the spinner) → the child layout
sees user equals no value → <Navigate to=/login> **before** /csrf→/me resolves. Repro: refresh /tutee/interviews
while logged in → bounced to /login.
**Proposed (corrected — Boss):** **seed is Loading:yes** in the store default (auth Store implementation reference) so Root Layout
shows the bootstrap spinner from the very first render until use Init Auth resolves /me (set User +
the set loading operation in .finally) — no child layout renders a redirect decision until the session is known. (An
initialized flag is an acceptable equivalent.) This single fix covers /tutee, /tutor, AND /admin because
all are Root Layout children (App implementation reference-66). Keep the safe-relative return To logic **unchanged** (built only
from location pathname+search, re-validated in Login implementation reference — no open-redirect regression). **Sub-task:** confirm
no guard sits outside Root Layout, and that the admin multi-factor authentication/password Change Required gate still fires post-init. **Acceptance:** a hard
refresh on /tutee/interviews (and every guarded route, incl. admin) while logged in **stays on the page**; a
genuinely logged-out user still lands on login with a valid return To; tests cover both + the
"refresh-while-/me-in-flight" case. **Ship first (Wave 1).**

## INT-057 — Logged-in tutees can't search/browse tutors (frontend-only)
**Current:** TUTEE TABS (nav Tabs implementation reference) has no tutor-search entry; /tutors (Browse Tutors,
App implementation reference) is already public. **Proposed:** add a Find Tutors navigation item opening the tutor directory to TUTEE TABS
(label TBD: "Find Tutors"/"Browse Tutors"). /tutors/:id detail stays login-gated. **Acceptance:** a logged-in
tutee has a nav entry that opens the tutor search/list.

## INT-058 — Tutees can't reach the applicants for their job (frontend-only)
**Current:** the view exists — /tutee/requests/:job Id/applicants → Applicants implementation reference (with
Applicant Card + Choose Tutor Modal, application programming interface api Routes implementation reference) — but nothing links to it from My Posts/dashboard.
**Proposed:** on each posting in the tutee dashboard/My Posts, add a **"View applicants (N)"** link routing to
/tutee/requests/:job Id/applicants (show the applicant count if available). **Acceptance:** from My Posts a tutee
can open the list of tutors who applied to a given job and proceed to choose-tutor.

## INT-059 — "Reviews About Me" tab → fold into Profile (frontend-only)
**Current:** dedicated tab in both TUTEE TABS (nav Tabs implementation reference) and TUTOR TABS (33)
(/tutee/reviews→Reviews implementation reference, /tutor/reviews→Reviews implementation reference, App implementation reference).
**Proposed (Boss):** remove the tab; render reviews as an **inline section inside the Profile page** for both
roles (not a sub-tab — we removed a tab to cut nav load, don't reintroduce one a level down). **Keep
/tutee/reviews + /tutor/reviews as redirects** (to the Profile section) so bookmarked/old links don't 404.
**Acceptance:** no top-level Reviews tab; reviews show as a Profile section for tutee + tutor; old /…/reviews
links **redirect** (quality assurance asserts the redirect).

---

## Guardrails / constraints
- frontend-only except **INT-053** (backend data transfer object + mapper only — **no schema/migration**). Reuse INT-049 primitives + tokens;
  no magic strings (INT-054 must remove the arbitrary max-w- 420px); brand locked.
- **Security:** INT-056 must not weaken the return To open-redirect protection; INT-053 must keep personal information rules for
  the admin context (confirm what admin may see — names vs handles) with security-expert.
- frontend coverage gate **≥97.5% line / ≥88% branch** (v8 summary); INT-056 ships a "refresh-while-/me-in-flight" test;
  new/changed components ship their own tests.
- Each item builds + previews locally, then is pushed per the definition-of-done.

## Mo S Co W / sequencing (for Product/Tech-Lead)
- **Wave 1 (Must, ship first):** INT-056 (Sev-1 auth bug) — small, high-impact, no dependency.
- **Wave 2 (Should, frontend-only quick wins):** INT-057, INT-058, INT-059 (tutee nav/IA), INT-055 (qualifications
  dropdown), INT-054 (Auth Card width).
- **Wave 3 (full-stack):** INT-053 (data transfer object + mapper + frontend links/modals) — largest; needs backend + frontend coordination.

## Open decisions
**Closed by team-review/Boss (2026-06-04):**
- ~~INT-053 modal vs routes~~ → **link to existing detail routes**; inline modals **deferred**.
- ~~INT-053 admin personal information~~ → admin already sees tutee names (Interview Queue implementation reference); no new exposure; data transfer object admin-only.
- ~~INT-054 size-prop vs default; 2-col~~ → **size-prop**; 2-column register **deferred**.
- ~~INT-059 sub-tab vs section~~ → **inline section** of Profile; old routes redirect.
- ~~INT-056 fix shape~~ → **seed is Loading:yes/initialized flag** (waiting on is Loading alone is insufficient).

**Still genuinely open (decide at build):**
- INT-053: is a dedicated **admin tutor-detail** view needed, or is linking to the tutor post + match/interview
  detail enough? (lean: reuse existing routes; revisit if admins ask for a tutor-centric view.)
- INT-057: label **"Find Tutors" vs "Browse Tutors"** and its position in the tab order (uiux call at build).
