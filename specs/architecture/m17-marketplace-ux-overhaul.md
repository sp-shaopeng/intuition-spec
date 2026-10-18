# M17 — Public Marketplace user experience Overhaul (+ M14b regression fix)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Status: **DEPLOYED (no schema change)** · Author: Architect + user interface/user experience · Date: 2026-05-30
Founder directive (2026-05-30): browse assignments is missing; auth CTAs wrongly
push register; tutors don't show on the landing; dummy data must be fully linked;
profile detail must require login; all flows must be sound. "Boss must ensure user interface/user experience
is correct." Founder decisions: **detail pages require login**; landing order
**Jobs → Tutors**.

## P0 — BUG-1: Tutors showcase is broken (M14b regression)

**Symptom:** read operation for showcase tutors → an empty result list explicitly marked as a degraded response despite 8
eligible ACTIVE tutor posts → landing shows the empty CTA, not tutors.
**Root cause (CORRECTED per Boss — the earlier "transaction dropped" diagnosis was
WRONG):** the refresh operation STILL carries the transactional operation,
but it is **defeated by Spring self-invocation** — the warm on startup operation and
the scheduled refresh operation call the refresh operation internally, which the AOP proxy does NOT
intercept, so no transaction ever opens. The tutors mapper then reads LAZY
@Element Collections (Tutor Post group Sizes, venue Types) →
Lazy Initialization Exception: no Session → refresh fails → fail-closed fallback.
(The requests pool survives only because its mapper is scalar-only.) Do NOT "restore"
a database transaction on the refresh operation — it is already there and inert; relying on it lets the
bug silently return.
**Fix:** inject a Transaction Template into Showcase Pool and wrap ONLY
the select operation (database read + data transfer object mapping) in read Only is yes. This opens a REAL
transaction regardless of proxy self-invocation, and keeps the Redis single-flight
lock + Redis I/O OUTSIDE the JDBC transaction. Reject JOIN FETCH (two Set
@Element Collections → cartesian product + Hibernate in-memory pagination, breaks the
LIMIT seek).
**Regression test (blocking):** MUST drive refresh through the real self-invocation
entry point — call the warm on startup operation (or the scheduled path) on the PROXIED Spring bean
in a @Spring Boot Test/@Data Jpa Test with a REAL Tutor Post Repository, seed a tutor
with non-empty group Sizes/venue Types, assert the served envelope is degraded:no
with non-empty collections. A test that calls the refresh operation / the refresh operation
DIRECTLY does NOT reproduce the production failure (no proxy boundary) and is explicitly
INSUFFICIENT. The mocked-repo Showcase Service Test returns pre-hydrated entities and is
structurally blind to this — keep it, add the new one.
**Process lesson:** deploy verification must assert the strip shows real cards
(degraded:no, data length>0), not merely web request protocol 200.

## FEAT-1: Public "Browse Tuition Assignments" page

Symmetry gap: there is Browse Tutors but no public way to browse jobs.

- **backend:** read operation for assignments — PUBLIC, paginated + filtered
  (subject, level, region, budget band, page, size). Returns the **anonymised**
  projection (reuse Showcase Request Response field set: no title/contact/exact
  date; bucketed budget; planning area + region; posted Days Ago). This is the
  paginated sibling of the capped/cached showcase. The gated /public/requests
  (richer, MRT-level, full Tutee Post Response) stays TUTOR/ADMIN-only.
  DBA: reuse the idx tutee posts showcase partial index; scalar-only mapper.
- **frontend:** new route /assignments (public) — Browse Assignments implementation reference mirroring
  Browse Tutors implementation reference (filter bar, paginated grid of Showcase Request Card-style
  cards, empty/error states). Nav gains "Browse Assignments" beside "Browse Tutors".
- Clicking an assignment → /assignments/:id **detail requires login** (see SEC-1).

## user experience-1: Auth-CTA principle — default to LOGIN, register only on "Create account"

Founder rule (verbatim): "the only time you register is you click create account
else should be login." Audit EVERY auth CTA / protected redirect:
- Generic acquisition CTAs ("Sign up to apply", "List your profile", "Get matched",
  showcase CTAs, "Find tuition jobs") → **/login** (the login page itself carries a
  "Don't have an account? Create account" link → /register).
- ONLY explicit "Create account" / "Register" buttons → /register.
- Hero: primary "Find a tutor" → /tutors (public browse); secondary "Find tuition
  jobs" → /assignments (public browse) — neither forces auth.
- Protected-route bounces already go to /login?return To=… (keep).
- Login implementation reference must have a clear "Create account" link to /register (the one
  sanctioned register entry, plus role-select).

## user experience-2: Landing order (founder: Jobs → Tutors)

Hero → **Recent Job Requests** → **Tutors Available Now** → how-it-works → trust.
Both strips eager-fetch (BUG-1 fix makes tutors actually render). Friendly labels
(below) so cards read "Math (PSLE) · Primary 5", not "MATH PSLE · P5".

## SEC-1: Detail pages require login (lists stay public)

- Browse **lists** (/tutors, /assignments) and the landing showcases stay PUBLIC.
- **Detail** views require auth:
  - frontend: /tutors/:id and /assignments/:id redirect anonymous users to
    /login?return To=… before rendering detail.
  - backend: the detail data endpoints require authentication. Move tutor detail to an
    authenticated endpoint (read operation for tutors id the is authenticated operation), and add
    an authenticated assignment detail endpoint. Remove/!permit the anonymous
    the public tutor-detail route. (List endpoints stay permit All.)
- Own account/dashboard profiles already require the owning role (verified) — keep.

## DATA-1: Dummy data completeness + integrity

Counts show tutors/profiles/tutee-posts all linked 8/8 and eligible — the "no data"
perception was BUG-1 (degraded showcase). Still, make the seed a COMPLETE, realistic
graph for flow-testing:
- Every seeded entity references a real users row (tutors→profiles→ACTIVE posts
  with group Sizes+venue Types+consent+available; tutees→ACTIVE posts).
- Add linked cross-entity rows so flows are demoable: ≥a few applications,
  interviews, matches, reviews — each FK-valid to seeded users/posts.
- Fix nonsensical seed values (e g. planning area/region mismatches like
  "Tampines/NORTH"); use a consistent area→region map.
- Idempotent + crash-safe (existing Transaction Template pattern). Gated behind
  seed sample-data enabled (local only).

## POLISH-1: Friendly subject/level labels

Add a code→label map (frontend constant, sourced from the same subjects/levels reference
the Create forms use) so MATH PSLE→"Math (PSLE)", P5→"Primary 5", S4 EXP→
"Sec 4 (Express)", JC1→"JC1", etc. Apply to both showcase cards, both browse
cards, and detail. No raw codes shown to users. No magic strings (central map).

## FLOW-1: Full flow audit (user interface/user experience + Frontend gate)

Walk and assert every anonymous + authed flow is sound:
1. Anon landing → both strips render real data → CTAs go to the right place (browse,
   not register).
2. Anon Browse Tutors / Browse Assignments → cards render → click detail → login
   prompt (return To preserved) → after login, land back on the detail.
3. Register only reachable via an explicit "Create account" control.
4. Authed tutor: browse assignments → view detail → apply. Authed parent: post
   request → it appears (anonymised) in the public strip/browse.
5. Nav is consistent (both browse links present; login/register correct).
6. No public endpoint leaks personal information; no anonymous access to any detail/profile.

## Acceptance criteria
- read operation for showcase tutors → degraded:no with ≥1 tutor; landing shows tutor
  cards. read operation for assignments → 200 paginated anonymised. Detail endpoints →
  401 anonymous, 200 authed. No CTA leads to /register except explicit create-account.
  Friendly labels everywhere. backend ≥97.5% line+branch, frontend ≥97.5% line/88% branch.

## Team Review Decisions (LOCKED — 2026-05-30)

**BUG-1 fix (the solution architect):** inject a Transaction Template into Showcase Pool; wrap ONLY
the select operation (the database read + data transfer object mapping, where the lazy access happens) in
read Only is yes. Do NOT put the transaction on the refresh operation (that would
hold the Redis single-flight lock + Redis I/O open inside a JDBC tx). Reject JOIN FETCH
(two Set @Element Collections → cartesian product + Hibernate in-memory pagination,
breaks the LIMIT seek). Regression test = @Data Jpa Test/@Spring Boot Test with a REAL
Tutor Post Repository, seed a tutor with non-empty group Sizes/venue Types, invoke
refresh from a non-transactional caller, assert degraded:no + non-empty collections.
(Mocked-repo Showcase Service Test is structurally blind to this — keep it, add the new one.)

**FEAT-1 security (Casey, all three blocking):** /public/assignments MUST (1) be
per-IP rate-limited via Rate Limit Service (as Showcase Controller does); (2) cap
anonymous pagination depth / total-result ceiling so the full pool isn't enumerable;
(3) location at planning-area max, never finer. Anonymised projection (reuse
Showcase Request Response). On Public Browse Controller, permit All. Emit access logs.

**SEC-1 (Casey, server-side):** detail endpoints return **401 to anonymous**
(the is authenticated operation). Narrow the Security Config tutors ** interface operation glob to
the LIST only; remove anonymous tutor-detail. Add authz tests asserting 401 anon / 200
authed. Accept the SEO cost of gating detail (founder chose gate-the-detail).

**Login return To (Sam+Casey, BLOCKER — pre-existing bug):** Login implementation reference currently
ignores return To and routes by role. Implement: read return To, validate it is a
same-origin RELATIVE path (leading /, reject //, \, https, encoded tricks),
navigate there post-login; else fall back to role home. Required for FLOW-1.2.

**user experience (Jamie + Wei/Priya, LOCKED):**
- Nav labels by INTENT: **"Find a Tutor"** (/tutors) and **"Find Tuition Jobs"**
  (/assignments) — not "Browse Assignments"/jargon.
- /login shows a LOUD, role-aware create-account split: "New here? Create a parent
  account / Create a tutor account" (→ /register?role=…) — both users demanded this.
- Detail login-gate renders a value-framed interstitial ("Log in to see the full
  profile and message them"), return To deep-links back to that exact detail.
- Each landing strip header gets "See all →" deep-linking to the (pre-filtered) browse
  page; card click → detail (gated). Strip = teaser, browse = catalog.
- ONE shared card component across both strips + both browse pages; ONE shared
  marketplace filter bar; designed empty state ("No matches — clear filters") +
  skeleton loading on every browse page. Budget band visible on every card.
- Hero PRIMARY CTA stays "Find a Tutor" (parent-first) even though Jobs strip is first
  (order ≠ priority — mitigates Priya's "mis-served" concern). Secondary "Find tuition
  jobs" → /assignments.

**frontend structure (Sam):** shared route wrapper that requires authentication and preserves the return destination for /tutors/:id
+ /assignments/:id (redirect BEFORE the authed fetch fires — no 401 flash); list
routes stay public/outside the guard. application programming interface ROUTES public.assignments; new
the list operation (do NOT collide with request Keys list). Friendly
label maps subject Label/level Label in format implementation reference (typed Records sourced from
the Create-form reference) applied to BOTH strips, BOTH browse cards, and detail (fix
posting Label raw codes). Run the documented operational check (tsc) AND test:ci.

**Auth-CTA final mapping:** find/browse → public browse pages; apply/contact/protected
→ /login; explicit "Create account"/"List as a Tutor"/"Sign up to apply" → /register.

**return To is a LIVE PRODUCTION BUG (Boss):** not merely an M17 prerequisite —
login bounce-back is broken RIGHT NOW for every protected route (reviews, dashboards,
tutor detail all send /login?return To=… which Login implementation reference ignores). FLOW-1 must
verify the fix on the EXISTING protected routes too, not only the new detail pages.

**Nav implementation reference in user experience scope (Boss):** the anonymous "Post a Request" button currently
hard-links to the protected /tutee/posts/new (a user experience-1 violation) → change to /login
(or remove). The MOBILE drawer must reach parity with both intent links ("Find a
Tutor" / "Find Tuition Jobs") — SG is mobile-first; desktop-only is half a fix.

**FEAT-1 concrete limits (Boss — numbers, not adjectives):** per-IP rate limit on
/public/assignments = **60 req/min** (reuse Rate Limit Service, dedicated key);
anonymous pagination hard cap = **max 5 pages, page size ≤ 20, 100-result ceiling**
(requests beyond → 400/empty, not deeper data); location **planning-area max**.

**user experience-2 measurement hook (Boss):** instrument landing-fold engagement so the Jobs-first
decision is reviewable against real parent behaviour — emit a lightweight counter/event
for each strip's view + "See all →" click (landing strip.view/.cta with a
strip is jobs|tutors tag via the existing meter registry; frontend fires on strip mount/click).
If data shows parents bounce off a Jobs-first fold, revisit the order.

## Open items for the team (resolved at deploy)
- O-1: **Resolved** — BUG-1 fixed with a Transaction Template in Showcase Pool plus
  materialised lazy sets (self-invocation lazy-init), not JOIN FETCH.
- O-2: **Resolved** — the anonymous the public tutor-detail route was **removed**; the whole
  tutor detail is login-gated via the authenticated read operation for tutors id. The
  /public/tutors/** glob is narrowed to the LIST (reviews kept public for M11).
- O-3: Budget-band display on the public browse filter (filter by band vs exact) — see
  M17.1 follow-ups in ROADMAP md #40 for any remaining user experience minors.
