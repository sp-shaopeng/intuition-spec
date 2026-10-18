# Product Instrumentation Spec — Event Store, frontend the track operation, and Real Funnels

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Preserve [PRD14.5](../PRD.md) storage separation. The older same-connection table is an unapproved candidate, not a relaxation of that requirement. Conversion follows017; exact average date cohort remains pending there. personal information registry and anonymization requirements remain intact. This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Owners:** Blake (data-engineer) · Riley (bff-specialist) · Sam (frontend-expert)
**Status:** Draft — doc only. Drives a follow-up backend + frontend build.
**Finding:** S1 funnel-blind (= follow-up **#62 measurement**). InTuition is shipping marketplace
funnels it cannot measure: the server "publishes" events to a log file with no store and no consumer,
the frontend emits nothing at all, and the M12 "Analytics" dashboard relabels point-in-time status
counts as funnels. We are **metrics-blind on the core revenue path** (apply → choose → match → commission).

> **Scope guard (product-designer / business-analyst):** this is instrumentation + one event store +
> a thin frontend collector + making M12 real. It is **NOT** a data warehouse, not dbt/Airflow, not an A/B
> platform, not Post Hog-on-Singapore-infra. Those are explicitly deferred (§11). The whole point is to
> stop being blind cheaply, on rails that already exist.

---

## 1. Problem (grounded in file:line)

### 1.1 The server "publisher" is a log info stub — no store, no consumer
Analytics Event Publisher implementation reference (21 lines):


Implementation example omitted. This public edition retains the written requirements in this section.


Every emit goes to stdout and is gone on the next log rotation. There is **no analytics events
table** (grep of db/changelog/ finds none) and **no consumer**. product requirements document §14.5 ("Analytics Event Sink")
and §7.8.5 ("the analytics event store and the audit log are SEPARATE data stores") describe a sink
that does not exist. This is the literal "we can pull it from the logs" anti-pattern — logs are for
debugging, not product analytics.

**The one piece of good news:** the emit *hooks are already wired*, mostly inside the after commit operation
callbacks, so this is **low-risk** — we change what the publish operation does, not where it is called. Verified
call sites (the existing 14 emitted events):

| Service (file) | Events emitted |
|---|---|
| Auth Service implementation reference | user registered (×2 role), login failed (×2 reason), login success |
| Tutee Post Service implementation reference | posting created, posting submitted, (+ close pair) |
| Tutor Post Service implementation reference | posting created, posting submitted |
| Job Application Service implementation reference | application submitted, application withdrawn |
| Tutor Selection Service implementation reference | tutor selected, selection request created |
| Admin Match Service implementation reference | job assigned, application superseded |
| Job Invitation Service implementation reference | invite created/withdrawn/accepted/declined/expired |

That is **~14 distinct event names actually emitted** against the **product requirements document §16 taxonomy of ~92 designed
events** across §16.1–16.8. The high-value funnel middle and the entire pre-login top-of-funnel are
unemitted (see §1.3).

### 1.2 The frontend emits NOTHING — no analytics dep, no the track operation
package implementation reference dependencies: @heroicons/react, @hookform/resolvers,
@tanstack/react-query, autoprefixer, axios, clsx, date-fns, postcss, react, react-dom,
react-hook-form, react-router-dom, tailwindcss, zod, zustand. **No Segment / Amplitude / Post Hog /
Mixpanel / GA.** the private implementation folder contains api, api Routes, auth, auth Redirect, nav Tabs, pagination,
query Client, query Keys, use Init Auth— **notrack/ noanalyticsmodule.** Agrep for
track\b | send Beacon | anonymous id in src/ returns only the M12 admin "Analytics" page and
incidental comment text. So:

- **No client-side events at all.** Landing-page views, /assignments browse, filter usage, CTA
  clicks, detail-page views, registration-CTA intent — **product requirements document §16.3 and §16.7 are 0% instrumented.**
- **No anonymous identity.** Pre-login traffic (M14a/M14b showcase, M17 /assignments) is invisible,
  so activation/conversion from "visited" → "registered" → "applied" **cannot be computed.**

### 1.3 M12 "Analytics" is point-in-time status counts mislabelled as funnels
Analytics Service implementation reference (Module 12) computes everything from the count by status operation snapshots:
build Posting Funnel (127), build Interview Funnel (147), build Match Funnel (171),
build Revenue Summary (189). The class names say Posting Funnel, Interview Funnel, Match Funnel
but, as INT-045's own scoping table admits, these are **point-in-time gauges** — e g.
Interview Funnel conversion Rate completed / requested over the *current* status distribution
(162), NOT a cohort that entered at the top and was followed to the bottom. A post approved last
month and a match created today are counted in the same "funnel" with no shared cohort key and no time
ordering. **You cannot answer "of the jobs posted in May, what % reached a match, and where did the
rest drop off?"** — which is exactly the #62 question.

**Net finding:** the product is **funnel-blind on its revenue path**. We can see stock (how many
ACTIVE posts exist) but not flow (how a cohort moves apply → choose → match → paid).

---

## 2. Current state summary

| Layer | What exists | Gap |
|---|---|---|
| backend emit hooks | 14 events wired, mostly after Commit | publish operation only logs |
| backend store | none | no analytics events table / no consumer |
| backend read model | Analytics Service status counts | mislabelled "funnels"; no cohort/time-ordered query |
| frontend collector | none | no the track operation, no anonymous id, no /collect POST |
| frontend events | none | §16.3 browse / §16.7 funnel-CTA 0% instrumented |
| Identity stitch | none | pre-login → user id never aliased |
| Tracking plan | product requirements document §16 prose tables | no machine registry, no write-time validation |

---

## 3. Design

### 3.1 Architecture (one diagram in words)


Implementation example omitted. This public edition retains the written requirements in this section.


**Both paths land in ONE store.** The frontend collector funnels client events through the SAME
the publish operation so the store is the single sink (product requirements document §7.8.5) and enrichment/personal information
rules apply once. The existing server-side after Commit hooks keep working untouched — we only swap
the body of the publish operation from "log" to "log + append to store."

### 3.2 Event store sink — separated analytics store; deployment choice pending

**Retained requirement:** PRD14.5 requires an analytics data store separate from the operational database and immutable audit log. A Postgres analytics store remains a technical candidate; the following local-table sketch is historical and is not approval to use the operational connection. Record the approved storage boundary/deployment before build. No amendment permitting a shared operational sink was recovered.

Historical candidate rationale (subject to that boundary):

- The whole point of #62 is **queryable cohort funnels**. M12 already aggregates over Postgres
  repositories; a Postgres analytics store supports cohort aggregation. Cross-store data needed for a cohort must have an explicit ingestion/query contract; do not promise same-operational-connection joins or no additional storage boundary. A Redis stream would need a separate consumer + a sink anyway → more moving parts for a
  "stop being blind" deliverable.
- Volume is low (a marketplace, not ad-tech): tens of thousands of rows/month. jsonb + a GIN index
  is more than enough; partition by month only if/when row count warrants (deferred, §11).
- We already run Postgres + Redis; **Redis stays the immediate-session/kill-switch + rate-limit store
  (INT-065), not the analytics log.** Keep responsibilities separate.

> **dba non-negotiables baked in:** name is the varchar operation, occurred at TIMESTAMPTZ NOT
> NULLpropsis required structured information, empty by default never a freetext blob), and every id column is
> typed. Indexes on name, occurred at and occurred at for the cohort/range scans, plus a GIN
> on props for property filters. This is **append-only** (soft-delete is irrelevant — events are
> immutable facts; the only mutation is the 90-day anonymizer rewriting id fields).

**Liquibase migration 0042-analytics-events implementation reference** (next free number; ledger head is 0041,
confirmed in db changelog-master implementation reference):

| Column | Type | Notes |
|---|---|---|
| id | uuid PK (the gen random uuid operation) | |
| name | the varchar operation | the event name; must match the registry (§3.6) |
| occurred at | timestamptz required | client-supplied for frontend events (clamped, §6), server the now operation for backend |
| received at | the now operation | server ingest time (clock-skew guard) |
| actor user id | uuid no value (FK soft — no hard FK to keep store decoupled per §7.8.5) | resolved user, null when anonymous |
| role | the varchar operation | TUTEE\|TUTOR\|ADMIN\|ANON enum-as-text |
| anonymous id | the varchar operation | stable client id; the alias key |
| props | required structured information, empty by default | typed registry-validated properties only |
| schema version | int required DEFAULT 1 | per-event version for breaking changes (§3.6) |

Indexes: idx analytics events name time name, occurred at, idx analytics events time
(occurred at)idx analytics events actor (actor user id)GINidx analytics events props
(props). **No named PG enum** forname/roleavarchar + app-level registry check — avoids the
ALTER TYPE ... ADD VALUE migrate-then-boot crash-loop that bit INT-034/INT-065; new event names must
never require a schema migration).

### 3.3 Analytics Event Publisher becomes a real sink (low-risk swap)
Keep the signature; add an overload carrying anonymous id/role/occurred at for frontend events. Body:

1. Validate name against the registry (§3.6); in dev, **reject** unknown names / missing required
   props (fail loud); in prod, **drop + count a analytics rejected metric** (never throw into a
   business path — instrumentation must never break the feature it measures).
2. Strip any property not in the registry's allow-list (personal information guard, §3.7).
3. append records one row via Analytics Event Store (a thin @Repository).
4. Keep the existing log info line (cheap, aids debugging).

Because the 14 existing hooks already run in after Commit, a store write that fails is logged and
swallowed — it cannot roll back the user's posting/match. **Verify** the swap is non-breaking by
re-running the services' existing unit tests (they mock the publisher; behaviour unchanged).

### 3.4 frontend the track operation + a server collector

**New analytics implementation reference (~80 LOC, no new dependency — frontend-expert
non-negotiable: zero npm cost):**


Implementation example omitted. This public edition retains the written requirements in this section.


- **Transport:** the send beacon operation (fire-and-forget, survives page
  unload, off the main thread — no INP cost), falling back to send the request so it may finish while the page is closing.
  **Never** block a click handler or add to LCP/INP.
- **anonymous id:** generated the random unique identifier operation on first load, persisted in local Storage
  (itn anon id) AND mirrored to a cookie so the server reads it without a body field. Stable across
  the pre-login session.
- **No personal information ever leaves the client:** the track operation only forwards a fixed registry-validated prop set
  (ids, enums, bucketed values) — the TS type for Analytics Props is a discriminated union per event
  so passing a name/email/phone is a **compile error**, not a runtime hope.
- **cross-site request forgery protection:** collect interface operation is a public, side-effect-free-for-the-user append endpoint; exempt it
  from the cross-site request forgery protection filter (it writes only to the analytics store, never to a user resource) and rate-limit
  it per-IP (Redis, reuse the INT-065 limiter pattern) so it cannot be used to flood the store.

**Collector contract (bff-specialist owns it):**
submit operation for collect (add application programming interface ROUTES collect constant frontend-side + Collector Controller BASE PATH
backend-side — **no magic strings**, per project rule).

- Body: event name, standard-format occurrence time and event properties (anonymous id from cookie).
- RFC 9457 application/problem+json error envelope on a bad event (422 unknown-name /
  missing-required-prop in dev; 204 No Content always in prod so the beacon never surfaces an error
  to the user).
- **Versioned in the path** (collect interface operation), additive-only thereafter.
- The server resolves actor user id/role from the auth cookie if present (frontend never asserts its own
  user id — anti-spoofing). Anonymous when no session.

### 3.5 Identity stitching (anonymous → known)
On successful **login or register**, the frontend calls the identify operation which emits an
identity aliased event carrying anonymous identifier, user identifier and role. The store now has the bridge row;
funnel queries can COALESCE an event's actor user id with the user id aliased from its
anonymous id, so a visitor's pre-login landing page viewed / assignment browse and their
post-login application submitted resolve to **one person**. Without this, activation rate is wrong
from day one (data-engineer non-negotiable).

### 3.6 The tracking plan IS the registry (governed, versioned)
Promote product requirements document §16's prose tables into a single **machine-readable registry** checked into the repo —
the canonical source both ends import:

- backend: analytics/Event Registry (a typed map: name → required props + types + status
  active|proposed|deprecated|removed + schema version). the publish operation validates against it.
- frontend: analytics Events implementation reference mirrors the same names + a discriminated-union props type
  (single source for the the track operation signature). A test asserts frontend names ⊆ backend registry (no drift).

**Naming convention (locked):** object action, lowercase snake case, past-tense
(application submitted, match created, contact revealed). **Variations are properties, never new
events** — one application submitted with  job id, source: 'free market'|'invite'|'listing',
is invite accept: bool  so the INT-044 invite-accept folds into the SAME apply→choose→match funnel
step (data-engineer rule). Breaking a prop = deprecate + bump schema version, never silently
repurpose.

### 3.7 personal information boundaries (security-expert + data-engineer, enforced by the registry)
**Allowed props:** ids (user id, job id/tutee post id, match id, application id,
interview id), enums (role, status, source, subject, level, via), bucketed/coarse values
(budget band, planning-area location — mirroring the M14b anonymised-showcase rules), counts/booleans
(superseded count, had application). **Never:** names, emails, phones, exact addresses, raw message
text, or full meeting links (product requirements document §16 note: only meeting platform domain, e g. zoom us). The
contact revealed event records *that* contact was revealed (ids + timestamp) — **never the contact
details**. Enforcement is structural: the registry allow-list strips unknown props server-side, and
the frontend props type makes personal information a compile error.

**Retention (product requirements document §16.8 / §7.8.5):** a daily cron anonymizes events older than 90 days —
actor user id, and the id props inside props, replaced with non-reversible hashes; aggregates may
be kept indefinitely. The cron runs **only** on analytics events, never on the audit log (which is a
separate audit log table, exempt). This is a fast-follow after the store lands (§10 phasing).

### 3.8 Emit the missing high-value events
Close the gap on the four funnels that earn money or feed iteration. Names below are the §16 registry
entries; **bold = newly instrumented** (the rest already emit and only need to reach the store).

**A. Apply funnel (#62 core — one shared cohort key tutee post id):**
post submitted → post approved* → application submitted → **applicant viewed** (parent opens
the personal information-free applicant list) → selection request created → tutor selected → job assigned
(= match created) → **contact revealed** (read operation for matches id contact). Every step carries
tutee post id (+ application id once it exists) so funnel queries join the *same* posting.
(* post approved/post rejected fire from the admin approval service — add the hook there.)

**B. Onboarding completion (INT-043 banner has no measurement):**
**registration started** (frontend CTA) → user registered → **commission disclosure viewed** /
**commission disclosure accepted** (tutor) → **tutor profile completed** (the INT-043 banner's
goal — fire when handle/commitment/rate/area are all set). Answers "do tutors who hit the banner
actually complete?"

**C. Choose → match (M18 selection path):** already mostly wired (selection request created,
tutor selected, job assigned, application superseded) — just needs to reach the store + the
shared tutee post id key for cohorting.

**D. Browse / CTA top-of-funnel (frontend, §16.3 + §16.7 — 0% today):**
**landing page viewed**, **assignment list viewed** (/assignments, with
subject, level and area filters, plus the result count), **tutor list viewed**,
**assignment detail viewed** / **tutor detail viewed** (gated detail), **registration cta clicked**
(button location, intended user role and source page). These are the only way to compute visit→register→apply
activation, and they ride the new the track operation + anonymous id.

---

## 4. North-Star + guardrails (business-analyst + data-engineer)

- **North-Star:** **completed matches per month** — the single metric that means the marketplace
  created real tuition value. Derived from match closed (terminal COMPLETED, §16.4) cohorted by
  month. This is flow, not the M12 stock count of ACTIVE matches.
- **Guardrails (must not be sacrificed to grow the NSM):**
  - **Take-rate / monetisation:** commission collected ÷ matched value, and PAID ÷ invoiced
    collection rate (from commission marked paid / invoice generated). Growth that doesn't
    monetise is vanity.
  - **Trust / quality:** match cancellation rate (match cancelled ÷ match created), interview
    no-show rate (interview no show), and (when M11 review events are added) average rating. Don't
    pump matches that cancel.
- **Primary funnel conversion (the #62 deliverable):** apply→select %, select→match %,
  match→contact-revealed %, and **time-to-match** (days from post approved to match created) — each
  tied to its instrumented step so the dashboard answers "did this move?" with no hand-written database query language.

---

## 5. M12 dashboard becomes real cohort funnels

Today Analytics Service (75 get Analytics) returns status snapshots. The change:

1. **Relabel the existing point-in-time gauges honestly** — keep them (they're useful stock
   metrics) but call them "Current State" not "Funnel" on the frontend. INT-045 already documents which
   fields are point-in-time vs range-scoped; reuse that classification.
2. **Add cohort funnel queries over analytics events** — e g. for a from,to posting cohort:
   the number of distinct tuition requests with a post-approved event in the selected cohort as the denominator, then the count that
   also reached application submitted, tutor selected, job assigned, match closed — a true
   ordered drop-off table with the same cohort key. Returned as a new cohort Funnel block on the
   existing read operation for admin analytics?from&to response (additive, no breaking change — bff rule),
   rendered as a funnel/drop-off chart (route-lazy-loaded chart, frontend-expert: don't bloat the
   admin bundle).
3. **Surface the North-Star + guardrails** as the dashboard's top row (Stat Cards already exist).

The acceptance bar: an admin can answer "of jobs approved in May, what % reached a match and where did
applicants drop off?" **without anyone writing database query language.** If it needs ad-hoc database query language, the instrumentation is
incomplete.

---

## 6. Data-quality + correctness notes

- **Clock skew:** frontend supplies occurred at; server clamps it to received at − 24h, received at
  and always stores received at separately. Order funnels by received at server-trust where
  ordering matters.
- **At-least-once / dedupe:** send Beacon can fire twice on flaky unload. For idempotent counting,
  the frontend includes a client-generated event id (uuid) in props for de-dupable events; the store keeps
  it and the funnel counts each distinct cohort key once, rather than counting raw event rows, so a double-fire does not
  double-count a person through a step.
- **Instrumentation must never break the feature:** prod the publish operation failures are logged + counted,
  never thrown; the collector returns 204 regardless; frontend the track operation is fire-and-forget.
- **Volume sanity:** add a analytics events.ingested / analytics events.rejected micrometer
  counter so a sudden drop (instrumentation regression) or spike (loop/abuse) is observable.

---

## 7. Acceptance criteria

1. **Store exists:** migration 0042-analytics-events implementation reference applies cleanly (Testcontainers);
   analytics events present with the columns + 4 indexes in §3.2; databasechangelog head = 0042.
2. **backend sink works:** every one of the 14 existing emit hooks now writes a row (verified by an IT that
   performs a register/apply/assign flow against the live stack and asserts the matching rows exist
   with correct name, actor user id, role, and registry-valid props).
3. **Registry enforced:** in dev/test, the publish operation of an unknown name or missing-required-prop is
   rejected (unit test); an unlisted prop is stripped before insert (personal information test asserting a name/email
   passed in props never lands in the row).
4. **frontend collector:** submit operation for collect accepts a beacon, resolves anonymous vs authed actor
   server-side (frontend-asserted user id is ignored), returns 204 in prod / 422 problem+json on bad
   event in dev; rate-limited per-IP.
5. **the track operation/the identify operation:** anonymous id is stable across reloads; the identify operation at login/register
   emits identity aliased; a funnel test shows a pre-login landing page viewed and a post-login
   application submitted resolve to one person.
6. **Missing events emitted:** the four funnels in §3.8 each fire end-to-end (apply funnel A proven by
   the §7.2 IT; browse funnel D proven by an frontend test asserting the track operation
   fires with the filter props on /assignments).
7. **M12 real funnel:** the admin analytics response includes a cohort Funnel block answering the §5
   acceptance question for a from,to cohort; the frontend labels the legacy gauges "Current State", not
   "Funnel".
8. **Gates:** backend the documented operational check ≥97.5% line AND branch; frontend run the corresponding frontend quality check  and  run the corresponding frontend quality check  and 
   run the corresponding frontend quality check≥97.5% line / ≥88% branch. No magic strings collector route in application programming interface ROUTES +
   a backend BASE PATH constant; event names from the registry only).
9. **No personal information regression:** security spot-check confirms no name/email/phone/exact-address/full-link in
   any stored prop; the frontend props type makes personal information a compile error.

---

## 8. Test notes

- **backend unit:** registry validation (accept/reject/strip-prop) on Analytics Event Publisher; collector
  controller actor-resolution (anonymous vs cookie-authed, frontend-asserted id ignored).
- **backend IT (live stack, Abstract Api IT → ):** full apply funnel writes the expected ordered rows;
  re-seed before running (regression ITs hit the live stack — rebuild/redeploy + reseed first).
- **frontend (Vitest):** the track operation calls send Beacon with the right name+props; personal information prop is a type error
  (a // @ts-expect-error test); anonymous id stability across mount/unmount; the identify operation on login.
  the documented operational check (tsc) mandatory — the discriminated-union props type must compile.
- **Funnel query test:** seed a small cohort, assert the cohort funnel drop-off numbers and
  time-to-match are correct (no Na N on empty cohort — mirror compute Conversion Rate's zero-guard at
  Analytics Service implementation reference).
- **Docker Support:** @Data Jpa Test repo tests showing Tests run: 0 locally is expected (local-skip
  policy) — the surefire/Mockito unit tests carry the JaCoCo gate.

---

## 9. Migration / coordination notes

- **Number:** 0042 (ledger head 0041 confirmed in db changelog-master implementation reference ; 0038–0041 taken).
  Plain create Table + create Index — **no ALTER TYPE ADD VALUE** (name/role are varchar, so new
  event names never need a migration → no migrate-then-boot crash-loop).
- **Seeder:** Dev Data Seeder does NOT need to seed analytics events (it's an append log; a green
  boot doesn't depend on it). If a demo funnel is wanted later, seed it in its own committed block.
- **Audit log untouched:** this store is separate from audit log (product requirements document §7.8.5); the 90-day anonymizer
  must target analytics events only.

---

## 10. Build phasing (low-risk first)

1. **P1 (unblocks #62):** migration 0042 + Analytics Event Store + swap the publish operation body + the
   registry. The 14 existing hooks immediately persist. *No frontend change yet — instant server-side win on
   rails already placed.*
2. **P2:** add the missing backend hooks (post approved/rejected, tutor profile completed, applicant
   view, contact-revealed) + the M12 cohort funnel query/response + dashboard relabel.
3. **P3:** frontend analytics implementation reference (track/identify/anonymous id) + collector endpoint + browse/CTA
   events + identity stitching.
4. **P4 (fast-follow):** 90-day anonymizer cron + micrometer counters.

---

## 11. Explicitly out of scope (deferred)

Data warehouse / dbt / Airflow; Post Hog-on-Singapore-infra (product requirements document §14.5 long-term); A/B experimentation
platform; tutee/tutor-facing analytics (product requirements document §15 Phase 2 — admin-only in MVP); table partitioning
(add when row volume warrants). This spec stops the bleeding (funnel-blind) on existing rails; it does
not build a BI stack.

---

→ team-review → Boss → build.
