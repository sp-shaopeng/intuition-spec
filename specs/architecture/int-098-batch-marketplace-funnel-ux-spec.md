# INT-098…INT-120 — Wave E: Marketplace & Funnel user experience remediation batch

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** **team-reviewed 2026-06-11 — conditions folded in** → Boss · **Date:** 2026-06-10
**Owners:** uiux-designer (lead) + frontend-expert + bff-specialist · supporting: software-engineer (the private implementation folder transfer object), dba (migration 0052), qa-engineer (regression tests)
**Tracker:** INT-098, INT-099, INT-100, INT-101 (all S2); INT-113…INT-120 (S3 sub-batch)
**Source review:** [issues/PROJECT-REVIEW-2026-06-10.md](../issues/PROJECT-REVIEW-2026-06-10.md) Part 1 S2/S3 + Part 3 B1–B4
**Migration:** **0052 SURRENDERED back to the ledger (DECIDED, team-review 2026-06-11)** — verified: 0007's **up-path** already does DROP CONSTRAINT IF EXISTS ... rejection reason check + widens to VARCHAR using 500 on **both** tables (the require is re-added only in rollback). No migration in this batch; the fix is **frontend-only** — REASON MAX LENGTH 50→280 + the owner-data transfer object rejection Reason.

> **Design principle (all items):** reuse the INT-049 primitives — Page Header / Section / Container / Card Grid / Tab Nav / Status Timeline / Success Panel / Empty State / Data List / Modal / Marketplace Filter Bar / the Favourites implementation reference pager. **Invent no new visual system.** This batch is narrative + symmetry + plumbing, not pixels (Part 3 verdict: "the headroom is NOT visual").

---

## Scope & the realised Part-3 user experience wave

This batch realises the **P1 cluster B1–B4** from the review's Part 3 table:

| Part-3 P1 | Realised by |
|---|---|
| **B1** — status-driven "what happens next" timeline on every artifact | INT-098 (rejection narrative), INT-118 (commission-pending narrative), INT-115 (interview context); the Status Timeline/Success Panel reuse threaded through each |
| **B2** — close the M18 loop in the user interface | (INT-090/INT-097 are a sibling spec) — INT-117 (real un-choose) is the funnel-symmetry half that lives here |
| **B3** — unify tutor jobs-browse (filters + pager) | INT-101 |
| **B4** — label hygiene + "no raw enum" regression test | INT-099 |

S3 items INT-113…INT-120 are grouped into a tidy sub-batch at the end (§S3-BATCH). Three items carry a **frontend-facing service contract change** (flagged ⚙️): INT-098 (rejection Reason on the owner data transfer object), INT-115 (interview data transfer object job/counterpart context), INT-117 (a real withdraw-selection endpoint).

---

## INT-098 (S2) — Rejection reason never reaches the post owner; REJECTED / REVISION REQUESTED are dead ends ⚙️

### Problem
A tutee whose post is rejected is told it failed but never told *why* in-app, and must re-type the entire post from a blank form. REVISION REQUESTED ("Revision Needed") has no edit route at all — a literal dead end.

### Current state (verified, file:line)
- **The owner data transfer object has no rejection Reason.** Tutee Post Response implementation reference (the public-safe record) carries status but no rejection field; its header comment enumerates exactly what is omitted. Tutee Post Controller list Own Posts (42-48) and get Own Post (50-57) both return this same data transfer object — so the **owner of the post cannot see the reason on their own posts**. Only the admin path exposes it: Admin Post Detail frontend type has rejection Reason (index implementation reference); Tutee Post frontend type does NOT (index implementation reference).
- **The data exists on the entity.** Tutee Post implementation reference — the column operation. The value is persisted; it's just never projected to the owner.
- **Resubmit is a blank form.** Dashboard implementation reference — the REJECTED row's only action is Resubmit → the create wizard with no prefill.
- **REVISION REQUESTED has no route.** format implementation reference maps it to "Revision Needed"; grep REVISION REQUESTED src finds the label only — App implementation reference has no edit/revise route for it.
- **The 50-char cap is a frontend constant, NOT the database column.** Review Detail implementation reference REASON MAX LENGTH 50 ;  computes notes Max Length REASON MAX LENGTH - reject Preset length - 3, so the longest preset ("Contact details in post content (personal information)") leaves ~9 chars of notes. **The database column is already the varchar operation** — migration 0007-admin-post-approval-fixes implementation reference bumped both tutor posts and tutee posts rejection reason from 50→500 *specifically because admin notes were being truncated*. The current 50 cap is a self-inflicted frontend regression that re-introduces the exact truncation 0007 fixed.

### Design / fix
**Three coordinated changes — one the private implementation folder transfer object, one migration, two frontend.**

1. **⚙️ frontend-facing service contract — surface rejection Reason to the owner.** Add text rejection Reason to the owner projection. **Do NOT add it to the public discovery Tutee Post Response** (that record feeds public browse — personal information/leak surface; rejection reason is owner-private). Two clean options — recommend **(a)**:
   - **(a) New Own Tutee Post Response record** returned by list Own Posts/get Own Post only = the public fields + rejection Reason + (future) admin-notes-free narrative. Keeps the public data transfer object an invariant personal information-free record.
   - (b) Add a nullable rejection Reason to Tutee Post Response and null it on every public build site. Riskier (one missed build site = a leak) and contradicts the record's "NO ..." invariant comment — reject.
   - Mirror frontend: add rejection Reason: string | no value to the Tutee Post frontend type (index implementation reference).

2. **frontend render — REJECTED narrative (reuse Status Timeline + Empty State/Section).** On the tutee/Dashboard REJECTED row and on a post-detail surface, render a Section tone is muted block with the exact final copy (team-review 2026-06-11): **"Not approved — reason. Update the details below and resubmit — most fixes take a minute."** (true because Resubmit pre-fills, §3) with the Status Timeline showing submitted → reviewed → **rejected (reason)** → resubmit. The reason text uses the full 500-char value.

3. **frontend — pre-fill the Resubmit wizard.** Change Dashboard implementation reference Resubmit to a new tuition-request form copied from a selected earlier request (or push the post into router state); the create wizard hydrates its RHF defaults from the rejected post (subject/level/budget/venue/availability/requirements) so the owner edits, not re-types. Guard: only prefill from a post the caller owns (it comes from their own list Own Posts).

4. **REVISION REQUESTED — DECIDED (team-review 2026-06-11): remove it from the frontend display mapping** (format implementation reference). The status is unreachable — no admin path sets it (admin only sets REJECTED/ACTIVE). The **backend enum value stays** (no ALTER TYPE risk); rejected posts use the INT-098 rejection flow above. No edit route is built for a state nothing produces; a distinct "send back for edit" admin action would be its own future issue.

5. **frontend — lift the reason cap.** Set REASON MAX LENGTH 280 in Review Detail implementation reference (the column holds 500; 280 leaves generous headroom for preset + notes and matches the review's "~280" recommendation). Recompute notes Max Length against 280.

6. **Migration 0052 — SURRENDERED back to the ledger (DECIDED, team-review 2026-06-11).** Audit verified: 0007's **up-path** already does DROP CONSTRAINT IF EXISTS tutee posts rejection reason check / tutor posts rejection reason check + widens to VARCHAR using 500 on **both** tables (the require is re-added only in the rollback). No live require remains and the column is already VARCHAR using 500 — **no migration is needed; do not file an empty changeset.** The fix is frontend-only: REASON MAX LENGTH 50→280 (§5) + the owner-data transfer object rejection Reason (§1).

### Test plan (lanes, team-review 2026-06-11: **[live-lab]** = Abstract Api IT → localhost, excluded from per-pull request continuous integration via ci-no-lab, needs rebuild+redeploy+reseed; **[Testcontainers/per-pull request]** = runs in per-pull request continuous integration, incl. Vitest)
- **[live-lab]** backend: Tutee Post Controller IT — submit → admin reject with a 200-char reason → read operation for tutee posts and the selected tuition request return rejection Reason equal to the full stored value (asserts §1 + no 500-truncation).
- **[live-lab]** backend: assert the **public** /public/assignments / discovery data transfer object does **not** carry rejection Reason (personal information-private guard).
- **[Testcontainers/per-pull request]** backend persistence test (Testcontainers PG): a 280-char reason persists without a require violation — pins the §6 finding that 0007's up-path already dropped the require (no migration).
- **[Testcontainers/per-pull request]** frontend: Dashboard test implementation reference — a REJECTED post renders the exact rejection copy via Status Timeline/Section; the Resubmit link carries the identifier of the request to copy.
- **[Testcontainers/per-pull request]** frontend: wizard prefill test — a new tuition-request form copied from a selected earlier request hydrates subject/level/budget from the rejected post.
- **[Testcontainers/per-pull request]** frontend: Review Detail test implementation reference — the longest preset + notes ≤ 280 submits without client truncation.

### Acceptance
1. A rejected tutee sees the admin's full reason in-app on their dashboard/detail (Status Timeline narrative), not just "Rejected".
2. Resubmit opens a **pre-filled** wizard, not a blank form.
3. Admin can write a meaningful reason (≥ ~250 chars usable) without silent truncation, and it persists.
4. REVISION REQUESTED is removed from the frontend display mapping (DECIDED; backend enum value stays) — no dead-end status ships.
5. The public discovery data transfer object still carries no rejection reason (personal information-private).

---

## INT-099 (S2) — Raw subject/level enum codes on tutor-listing surfaces (B4)

### Problem
The highest-traffic public conversion surface (tutor cards a parent first sees) renders database enum codes (MATH PSLE, S4 EXP) instead of friendly labels — marketplace-credibility damage. The humanisers exist and are already used on the *jobs* side, so this is pure routing-through.

### Current state (verified, file:line)
Raw tutor subjects / tutor levels rendered without the subject label operation/the level label operation:
- Tutor Card implementation reference (the slice operation) and 134 (the join operation) — public browse cards.
- Tutor Detail implementation reference (the map operation into Badge) and 248 (the join operation).
- Dashboard implementation reference — the raw subject and level codes (parent sees CHN PSLE — PRI 6).
- Invite Tutor Modal implementation reference — the posting label operation returns the raw subject and level codes.
- Review Detail implementation reference (title), 159-160 (Subject/Level Row values) — tutee branch renders post subject Code/post level Code raw.
- **Already correct (do not touch, use as the pattern):** Browse Requests implementation reference already does the readable subject and level labels — proof the humanisers work and are the established idiom.

The humanisers: format implementation reference the subject label operation, 136 the level label operation — both fall back to the raw value for unknown codes (never blank). Source of truth taxonomy implementation reference.

### Design / fix
Route **every** listed render through the subject label operation/the level label operation. No new component — wrap the existing render expressions. For the map operation keep s as the React key, label the display text: a subject badge showing its readable label.

**Add a regression "no raw enum" test** (this class has regressed before — INT-068 fixed the admin ledger only, this is the second regression). Two layers:
- A **grep guard** in continuous integration / a unit test: assert no listing-surface JSX renders .subjects/.levels/.subject Code/.level Code directly inside JSX without a the label operation wrapper. Implement as a vitest test that reads the listed files and asserts the raw-interpolation pattern is absent, OR a package implementation reference lint-grep step.
- A **rendering unit test**: render Tutor Card with subjects: MATH PSLE, assert the DOM shows the friendly label and **not** the string MATH PSLE.

### Test plan (all **[Testcontainers/per-pull request]** — Vitest)
- Tutor Card test implementation reference / Tutor Detail test implementation reference — assert friendly label rendered, raw code absent.
- Invite Tutor Modal test implementation reference — option text is humanised.
- Review Detail test implementation reference — tutee subject/level rows humanised.
- The grep/AST "no raw enum on listing surfaces" guard fails if any of the six sites regress.

### Acceptance
Parents and tutors never see a raw subject/level code on any tutor-listing, invite, dashboard, or admin-review surface; a regression test fails the build if the pattern returns.

---

## INT-100 (S2) — Browse CTAs send logged-in users to /login ; /login & /register have no authed redirect

### Problem
A logged-in parent clicking the platform's main conversion CTA ("Post a Request") lands on a login form. Authenticated users reaching /login//register directly see the form instead of being bounced to their dashboard.

### Current state (verified, file:line)
- Browse Tutors implementation reference — "Post a Request" →  unconditionally (this is a tutee primary tab).
- Browse Assignments implementation reference — "Are you a tutor? … Log In" →  unconditionally.
- Login implementation reference / Register implementation reference — never read use Auth Store user; an authed visitor sees the form.
- The role-landing helper already exists (auth Redirect implementation reference login Landing For Role per the review / INT-029 lessons) — reuse it.

### Design / fix
1. **Role-aware CTAs.** Read use Auth Store in both browse pages. Logged-in TUTEE → "Post a Request" links to /tutee/posts/new; logged-in TUTOR → hide the cross-role "Post a Request" CTA (and on Browse Assignments, a logged-in TUTOR's "Log In to apply" becomes the inline Tutor Apply Control/"apply" path, while a logged-in TUTEE sees no tutor CTA). Anonymous → keep /login.
2. **Authed redirect off /login and /register.** In Login implementation reference/Register implementation reference, if use Auth Store user is set, redirect to the landing page for the signed-in role while replacing the current history entry. Use replace so the back button doesn't trap them on the form.
3. Couple-guard (SHARED-LESSONS 2026-06-05): the redirect target MUST equal the role's NAV primary 0 landing — assert in a test, or it silently drifts.

### Test plan (all **[Testcontainers/per-pull request]** — Vitest)
- Browse Tutors test implementation reference — anon: CTA → /login; authed TUTEE: CTA → /tutee/posts/new; authed TUTOR: cross-role CTA hidden.
- Login test implementation reference/Register test implementation reference — authed user is redirected to the login landing for role operation (one case per role).

### Acceptance
No logged-in user is ever sent to a login form from a browse CTA; visiting /login//register while authed redirects to the role landing.

---

## INT-101 (S2) — Authed marketplace surfaces hard-capped at 50, no pager; /requests empty-state lies (B3)

### Problem
The tutor's **login landing** /requests shows the first 50 jobs with no filters and no pager — and its empty state claims "matching your subjects" when no such matching exists. Job #51+ is unreachable. The anonymous /assignments teaser has *more* capability (filters) than the supply side's #1 revenue screen.

### Current state (verified, file:line)
- Browse Requests implementation reference — use Query with fixed the first page with up to fifty results, no Marketplace Filter Bar, no pager. **Empty-state copy lies:** 60-61 heading "No open requests" / description "No open requests matching your subjects at the moment." — there is no subject-matching logic; it's a flat list.
- Browse Tutors implementation reference AUTHED PAGE SIZE 50 ;  page: 0 fixed (it *has* filters — subject/level/region state at 28-30 + Marketplace Filter Bar — but **no pager**, and total Results can exceed 50, so #51+ is unreachable).
- Invitations implementation reference — size: 50 fixed similarly.
- The reusable parts both exist: Marketplace Filter Bar implementation reference (subject/level/region + opt-in rate/venue/group Size) and the Favourites implementation reference pager (page state, total Pages from meta, Prev/Next nav with disabled states + a11y label).

### Design / fix
**DECIDED (team-review 2026-06-11): UNIFY — /requests becomes a role-aware /assignments (realises B3). Conditions: reuse the existing card/filter/pager primitives ONLY, zero visual redesign, delete Browse Requests.**
- Browse Assignments implementation reference already renders the anonymised open-jobs surface with Marketplace Filter Bar. Make it role-aware: a logged-in TUTOR gets the inline Tutor Apply Control/Apply Button per card (already the Browse Requests behaviour) + filters + pager; anon keeps the teaser. **Delete Browse Requests implementation reference** and point the tutor login-landing route at /assignments. One card system to maintain; tutors get search on their #1 tab. Update nav Tabs implementation reference + the login-landing coupling test (the landing must equal NAV primary 0 for TUTOR).
- (The bolt-on-filters fallback was REJECTED in team-review.) **Kept regardless:** add the pager everywhere a 50-cap exists (Browse Tutors implementation reference, Invitations implementation reference included), and FIX the empty-state copy — drop "matching your subjects", say "No open requests right now — check back soon" (truthful; ties to B11 empty-state truthfulness pass).
- Pager: lift the Favourites implementation reference pattern verbatim (page state → query param → meta total Pages → Prev/Next nav). Filters: reuse Marketplace Filter Bar (subject/level/region; the jobs surface omits rate/venue/group Size like Browse Assignments already does).

### Test plan (all **[Testcontainers/per-pull request]** — Vitest)
- Browse Assignments test implementation reference (the unified surface) — authed TUTOR sees filters + apply control + pager; changing a filter refetches; Next advances the page; empty state shows the truthful copy (no "matching your subjects").
- Pager test: meta total Pages 3 → Next enabled on page 0, disabled on page 2; Prev disabled on page 0.
- Coupling test: TUTOR login-landing route  equals  NAV primary 0 (/requests is retired — DECIDED).
- Retirement guard: grep Browse Requests src returns zero route references after unification.

### Acceptance
A tutor can filter and page through ALL open jobs (not just 50); the empty state names a real next step; the anonymous teaser is no longer more capable than the supply side's primary screen.

---

## §S3-BATCH — Moderate funnel/narrative defects (INT-113…INT-120)

A tidy sub-batch: small, low-risk, mostly frontend. Two carry a frontend-facing service contract change (INT-115 ⚙️, INT-117 ⚙️).

### INT-113 (S3) — Notifications page has no pager
**Current (Notifications implementation reference):** use Query consumes a page of notifications with no page params and no pager user interface; older notifications are permanently unreachable. **Fix:** add the Favourites implementation reference pager (page state → the selected page and page size → meta total Pages → Prev/Next). **Test:** meta total Pages is 2 → Next loads page 2; page-1 items differ. **Accept:** all notifications reachable.

### INT-114 (S3) — Interview create never invalidates the lists operation
**Current:** Applicants implementation reference (request Interview Mutation on Success only toasts) and Tutor Detail implementation reference (interview-create on Success only resets form + toasts) — neither calls invalidate Queries. Every other interview mutation invalidates. With stale Time: 30 000 the tutee opening "My Interviews" sees a stale list = looks like silent failure. **Fix:** add refresh cached interview lists after a change to both on Success (key helper exists at query Keys implementation reference). **Test:** mock invalidate; assert it's called with the lists operation after a successful create. **Accept:** a freshly requested interview appears in the list without a manual refresh.

### INT-115 (S3) ⚙️ — Interview cards lack counterpart/job context
**Current:** the Interview data transfer object carries only tutor Post* fields (index implementation reference; backend Interview Response implementation reference — tutor Post Title, subjects, levels, rate, all no value for free-market). Interviews implementation reference titles every card with the tutor's OWN tutor Post Title (3 negotiations = 3 identical cards); Interviews implementation reference renders tutor Post Title with no fallback → blank for free-market interviews. **Fix (⚙️ frontend-facing service contract):** extend Interview Response with the **job's** subject/level (from the get tutee post operation — already partly resolved at Interview Response from 50 for tutee Post Id) + an **anonymised counterpart handle** (tutor's public handle for the tutee view; the tutee is personal information-anonymised for the tutor view — carry only a non-personal information label, never a name). Render: tutor card titled by *job* subject/level + "with tutee Handle-or-anon"; tutee card titled by job subject/level + "with tutor Handle", with a fallback so free-market cards are never blank. personal information rule: the data transfer object stays name/email/phone-free (the record's existing invariant). **Test:** backend — Interview Response from for a free-market interview carries job subject/level (not null) + a handle; frontend — three negotiations render three distinct titles; a free-market card is non-blank. **Accept:** every interview card is self-identifying; INT-039 negotiation is no longer guesswork.

### INT-116 (S3) — Interview cancel fires on first click; three inconsistent destructive-confirm patterns
**Current:** Interview Detail implementation reference and Interview Detail implementation reference (mirror) — Cancel calls the mutate operation directly, no confirm, even for SCHEDULED. The codebase has THREE patterns: close-job = design-system Modal, decline-invitation = Modal, withdraw-application = window confirm (Applications implementation reference). **Fix:** one shared confirm. Recommend a small Confirm Modal wrapper over the design-system Modal (reuse the INT-046 close-path conventions — assert closure via the open attribute, not text removal). Add it to interview cancel; replace the window confirm in Applications implementation reference. **Test (per the INT-046 lesson):** open → confirm fires the mutation; each close affordance (footer Cancel/Keep, header ×, native close) is its own case asserting dialog not open; assert no window confirm remains (grep window confirm src → only its removal). **Accept:** every destructive action (cancel/withdraw/decline/close) routes through one consistent confirm; no first-click-fires destructive action.

### INT-117 (S3) ⚙️ — "Change choice" / "Change my choice" change nothing; the modal over-promises
**Current:** Applicants implementation reference "Change my choice" only the set chosen handle operation (dismisses the success panel; the selection request stays CREATED). Applicant Card implementation reference "Change choice" run the choose-tutor action when selected re-opens the modal for the **same** tutor. Choose Tutor Modal implementation reference promises "You can change your choice until our team confirms" — but **no withdraw-selection endpoint exists** (the source search found no withdraw-selection operation). You can pick a *different* tutor, never return to "no one chosen". **Fix — DECIDED (team-review 2026-06-11): option (a), a real withdraw-selection endpoint** — the modal already promises changeability, so make the promise true; the withdraw is a **soft-delete status flip** on the selection request, never a row delete:
- **(a) ⚙️ Wire a real withdraw-selection (backend+frontend).** Add submit operation for tutee posts id select withdraw (or delete operation for tutee posts id selection) that, owner-gated, moves the parent's CREATED tutor selection request → a terminal WITHDRAWN/CANCELLED state (**soft-delete only** — status flag, never a row delete, per the founder rule). Idempotent (no-op if none CREATED); 409 if already admin-confirmed/assigned. frontend: "Change my choice" calls it, invalidates the selection + applicants queries, returns the panel to "no one chosen". This makes the modal's promise TRUE (claims-trace-to-code, SHARED-LESSONS 2026-06-05). **Couple-guard:** the withdraw must not strand the chosen application — leave it APPLIED/SHORTLISTED, not orphaned.
- (b) Relabel + soften — **REJECTED in team-review** (the modal already promises changeability; honour it).
**Test:** **[live-lab]** backend — choose A → withdraw → request is WITHDRAWN, app un-orphaned; withdraw with nothing chosen = 200 no-op; withdraw after admin-confirm = 409. **[Testcontainers/per-pull request]** frontend — "Change my choice" returns to no-one-chosen. **Accept:** the parent can truly un-choose; the modal's promise matches what the system does.

### INT-118 (S3) — Tutor "Commission pending" is a money dead end (B1 narrative)
**Current:** Matches implementation reference — a pending-style badge labelled “Commission pending” + the amount (50-52), with no explanation of how/when/where to pay; payment exists only as the admin's mark-paid. A debt-looking badge with no narrative is the #1 anxiety in an admin-mediated marketplace. **Fix:** one explanatory line under the badge (reuse Section tone is muted or inline caption): "We'll contact you to collect the one-time placement fee after your first session — nothing to do yet." + admin contact, **appending the exact final copy (team-review 2026-06-11): "The fee applies only once lessons begin — if the arrangement ends early, email us at admin Email and we'll sort it out."** No new component. **Test:** Matches test implementation reference — a PENDING-commission ACTIVE match renders the explanatory copy. **Accept:** the tutor understands the badge is informational, not an overdue bill.

### INT-119 (S3) — Apply fallback 422 is silent
**Current:** Tutor Apply Control implementation reference — on a server TUTOR PROFILE INCOMPLETE 422 the sheet closes and the handler returns with **no toast/banner** (a stale cached profile → Apply → sheet vanishes → nothing said; INT-004 silent-failure class). The adjacent quota-422 path (111-116) correctly toasts. **Fix:** on TUTOR PROFILE INCOMPLETE, show the existing Apply Gate Banner / a toast routing to /tutor/profile ("Complete your tutor profile to apply") instead of returning silently. **Test:** mock a 422 TUTOR PROFILE INCOMPLETE; assert the banner/toast renders with the profile link. **Accept:** the apply fallback always explains why and where to go; no silent sheet-vanish.

### INT-120 (S3) — "Complete portfolio" → new-listing wizard → duplicate listings; competing onboarding prompts (B8 seed)
**Current:** Dashboard implementation reference "Complete portfolio" →  — no listing-EDIT route exists, so following it creates a **duplicate** listing. Meanwhile Tutor Layout implementation reference Profile Completion Banner and the apply gate both point at /tutor/profile. Two simultaneous prompts, different fields, different destinations. **Fix (scope fence, team-review 2026-06-11):** repoint the prompt at editing the existing listing; the full B8 consolidation stays P2/roadmap. Point "Complete portfolio" at editing the existing identity (/tutor/profile, the canonical M18 tutor identity), or remove the dashboard prompt and rely on the single Profile Completion Banner. Do NOT route to a creation wizard for a "complete" action. (Full B8 onboarding consolidation — one prompt, first-week checklist — is a P2 enhancement, out of this defect's scope; this fix just stops the duplicate-listing trap and the competing destinations.) **Test:** Dashboard test implementation reference — "Complete portfolio" links to /tutor/profile, never /tutor/listings/new ; inspect every completion prompt to confirm none links to the new-listing form. **Accept:** one onboarding destination; following any completeness prompt never creates a duplicate listing.

---

## Cross-cutting test & quality notes
- **No magic strings:** every new label routes through the existing humanisers/typed enums (INT-099); new endpoints reference application programming interface ROUTES constants (INT-117).
- **Soft-delete only:** INT-117's withdraw-selection flips a status, never deletes a row.
- **personal information invariants hold:** INT-098's rejection Reason is owner-private (not on the public data transfer object); INT-115's interview context carries no name/email/phone.
- **Coverage gates:** frontend ≥97.5% line / ≥88% branch (Vitest); backend ≥97.5%/97.5% (JaCoCo). Modal close-path enumeration for INT-116 (assert open attribute, per the INT-046 lesson). backend data transfer object/migration changes verified on Testcontainers PG, not H2.
- **Couple-guards (SHARED-LESSONS 2026-06-05):** login-landing  equals  NAV primary 0 test for INT-100/INT-101; "no raw enum" guard for INT-099; "no window confirm" guard for INT-116.
- **Analytics (team-review 2026-06-11):** add events **selection withdrawn** (INT-117) and **post resubmitted** (INT-098 prefill) to the Analytics Event Registry additions; the existing job assigned / tutor selected / application superseded events cover the rest of this batch.
- **Coverage note (team-review 2026-06-11):** Wave E adds the most untested frontend branches of any wave (role-aware CTAs, pager states, wizard prefill, the /assignments unify, 6 humaniser sites) — the test plan must **enumerate branch tests per surface** to hold the **88% frontend branch** gate.
- **Test lanes (team-review 2026-06-11):** all §S3-BATCH tests are **[Testcontainers/per-pull request]** (Vitest) except INT-117's backend withdraw tests, which are **[live-lab]** (Abstract Api IT → localhost; excluded from per-pull request continuous integration via ci-no-lab; rebuild+redeploy+reseed first).

## Owners
| Item | Lead | Supporting |
|---|---|---|
| INT-098 ⚙️ | bff-specialist (data transfer object) + uiux-designer (narrative) | software-engineer (backend), dba (0052 surrendered — no migration), frontend-expert (wizard prefill) |
| INT-099 (B4) | frontend-expert | uiux-designer (label review), qa-engineer (regression guard) |
| INT-100 | frontend-expert | uiux-designer (CTA copy) |
| INT-101 (B3) | uiux-designer (lead) + frontend-expert | bff-specialist (pager params); UNIFY decided (team-review 2026-06-11) |
| INT-113/114/116/118/119/120 | frontend-expert | uiux-designer (copy/narrative) |
| INT-115 ⚙️ | bff-specialist | software-engineer (backend data transfer object), frontend-expert (cards) |
| INT-117 ⚙️ | bff-specialist + software-engineer (backend) | frontend-expert (frontend); option (a) decided (team-review 2026-06-11) |

## Migration ledger note (dba)
**0052 SURRENDERED back to the ledger (DECIDED, team-review 2026-06-11).** Latest applied = 0048. Audit verified: rejection reason is ALREADY the varchar operation and 0007's **up-path** already does DROP CONSTRAINT IF EXISTS ... rejection reason check on **both** tables (the require is re-added only in the rollback) — no live require remains. No changeset is filed (no empty/no-op migration); the fix is frontend-only (REASON MAX LENGTH 50→280 + the owner-data transfer object rejection Reason). No other item in this batch needs a migration.

---

*Realises Part-3 P1 wave B1 (INT-098/115/118 narrative), B3 (INT-101 unify), B4 (INT-099 label hygiene); B2's funnel-symmetry half (INT-117 un-choose). INT-090/097 (B2 assign + tutor reveal) are a sibling spec. B5–B11 are P2/P3 enhancements (ROADMAP, founder call), not in this defect batch.*
