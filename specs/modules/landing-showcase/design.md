# M14 — Landing Showcase: Technical Solution / Design

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

**Status:** **M14a (Active Tutors) DEPLOYED (migration 0019); M14b (Recent Requests)
DEPLOYED (migration 0023)** — consent/legal gate **waived by founder 2026-05-30**.
The §B gating sections below are retained as history; shipped design lives in
[../../architecture/m14b-public-job-postings-showcase md](../../architecture/m14b-public-job-postings-showcase.md).
**Author:** the solution architect (Solution Architect, wearing Tech Lead hat) — revised per the
2026-05-29 review + Boss decision.
**Date:** 2026-05-29 (M14b status updated 2026-05-30)
**Authoritative requirements:** [spec md](spec.md) (Product — what & why; split
into §A / §B).
**Scale posture:** [status-briefing md](../../status-briefing.md) 🚩 — *build for many users; "only bites at scale" is not a valid defer.*
**Roadmap:** [roadmap-next md](../../roadmap-next.md) — M14 = item H, reconciled to
the split status.

**History note:** a **14-person team review + Boss** returned **SEND BACK on the
Recent Requests strip** and gave a **green path to the Active Tutors strip** on
2026-05-29. The founder chose to **ship tutors first and gate requests.** This revision
splits the design into two slices and applies all the must-fix conditions: the
system-watched last active at activity signal is **removed entirely** and replaced
with a **tutor-set Available flag**; migration 0019 drops CONCURRENTLY; the response
becomes an envelope; the observability backstop is in scope; and several verified
doc-accuracy errors are corrected (§0).

This document is the *how*. Product owns the *what* (spec §A/§B); every choice here is
grounded in the actual code as it exists at migration **0018**.

**Document structure:**
- **§0** — grounding (verified code facts), with the corrected citations.
- **§A1–§A12** — M14a (Active Tutors): the buildable architecture.
- **§B** — M14b (Recent Requests): gated; what the build would need once §B1 clears.
- **§9** — decision log.

---

## 0. Grounding — what already exists (verified in code)

| Fact | Evidence |
|---|---|
| Tutee Post Response is already fully personal information-free (no tutee Id, name, email, phone). | Tutee Post Response implementation reference lines 15–34 |
| Tutor Post Response exposes only tutor Display Name, no user Id/contact; average Rating/review Count already nullable & batch-loaded. | Tutor Post Response implementation reference lines 18–39 |
| tutors interface operation is genuinely public; requests interface operation is the pre authorize operation **and** guards on the caller owning an ACTIVE post. | Public Browse Controller implementation reference lines 39–89 |
| Security Config: the public **permit** for tutors interface operation + /tutors/** is at **lines 129–130** (verified); **line 108 is the cross-site request forgery protection ignoring Request Matchers entry for ** interface operation** — the cross-site request forgery protection ignore-list, *not* the permit. Everything else under /public/** falls through to the any request operation (l.189–190). | Security Config implementation reference lines 108 (cross-site request forgery protection ignore), 129–130 (permit), 189–190 (catch-all) |
| Redis is wired (text Redis Template); a reusable atomic **sliding-window** Rate Limit Service exists (Lua, no TOCTOU). | Redis Config implementation reference, Rate Limit Service implementation reference |
| @Enable Scheduling is already active on the app. | Intuition Application implementation reference line 10 |
| Application Ready Event warm-on-startup is available via Spring's standard Application Listener/@Event Listener. | Spring Boot lifecycle |
| Tutor Post tutor → User is the owner; tutor posts has status, created at, location planning area, location region, location mrt station. **tutor posts has no available for students column yet** (added in 0019). | Tutor Post implementation reference |
| Client IP is resolved with precedence **CF-Connecting-IP → X-Forwarded-For[0] → the get remote addr operation**. The showcase rate-limit reuses this same resolution helper. | Auth Controller implementation reference (IP resolution helper) |
| Latest migration is **0018**; next free is **0019**. | db/changelog/changes/ |

Two consequences fall straight out of this grounding:

1. The tutor public data transfer object is *already* a personal information-free superset of what the M14a card needs.
   The showcase projection is a strict **subset** (minus MRT) — this is a deletion
   exercise, not a new exposure of sensitive fields. (Satisfies spec §A2 and the
   contract test.)
2. There is **no** system-watched activity signal anywhere and we are **not** adding
   one — "available" is a tutor-set flag (§A2). The only new public surface for M14a is
   the two unauthenticated showcase GETs (one in this slice — tutors). The Recent
   Requests public surface is **deferred and gated** (§B).

---

# §A — M14a: Active Tutors (BUILDABLE ARCHITECTURE)

## A1. Random-at-scale — **chosen: Redis-cached, per-window precomputed shuffled candidate pool**

### Decision
For the Active Tutors strip, a **scheduled server-side job** runs every **re-roll window (default 90s)**,
selects the eligible rows with a cheap **freshness-first index seek**, draws a randomised
subset in **application memory** (not in database query language), serialises the finished data transfer object list to JSON,
and writes it to a single Redis key. **All visitor reads hit Redis only.** The database
is never touched on the request hot path, and the random operation never runs anywhere.

This is option (a) from spec §A1a: *a periodically-recomputed shuffled candidate pool
cached in Redis, re-rolled per window.*

### Read path (per visitor request) — O using 1, no database


Implementation example omitted. This public edition retains the written requirements in this section.


**The refresh job — not the read path — is the only thing that touches the database.** A
read miss never triggers a database seek; it fails CLOSED to the static fallback (§A3).

### Refresh path (scheduled, off the hot path) — runs once per window per app cluster


Implementation example omitted. This public edition retains the written requirements in this section.


**Warm-on-startup:** an Application Ready Event listener runs the refresh tutors operation once at
boot so the first visitor after a deploy hits a warm cache, not a cold miss/fallback.

**Fail-CLOSED on Redis unreachability:** the refresh **must not** run the database seek if it
cannot acquire/observe the Redis lock — otherwise N instances would each run the seek
uncoordinated. If Redis is unreachable, the refresh returns early and reads serve the
static fallback (§A3). Redis is the coordination point; without it we do not touch the database.

find Showcase Tutor Candidates is **not** a random query. It is a bounded, index-ordered
seek that pulls a **candidate pool** larger than the display ceiling (e g. POOL SIZE 100,
display = 20):


Implementation example omitted. This public edition retains the written requirements in this section.


> The eligibility is **ACTIVE post AND available flag true** (plus owner not
> soft-deleted / not featuring-revoked). There is **no last active at predicate** —
> that surveillance signal was deleted (§A2). The 7-day window in spec §A12 is a
> recency *framing/label* computed in Singapore time, not a query filter on an activity column.

### the freshness first shuffle operation (in-memory) — "freshness-first, random fills the rest"
1. Sort the candidate pool by created at DESC (already ordered from database query language).
2. Take the **freshest K** (default the ceil operation ≈ 7) deterministically — these
   always lead, honouring the liveness promise and the persona test in spec §A1a.
3. **Shuffle the remaining** pool with an **injectable Random/seed** and take
   ceiling − K of them.
4. Concatenate: freshest K ++ random sample. This is the "random *effect*,
   freshness-first" the product requires — not a pure random feed, not the random operation.

The **Random/seed is injectable** (test seam, spec §A10) so the shuffle is
deterministic under test. Cost is trivial: shuffling a ≤100-element list is
microseconds, once per 90s, on one node.

### Complexity at scale (the spec §A1a random-at-scale ask)
| Eligible rows | This design (per *read*) | This design (per *refresh*, every 90s) | the random operation per read (rejected) |
|---|---|---|---|
| 10² | O using 1 Redis GET | index seek of ≤100 rows + in-mem shuffle of ≤100 | full scan + sort of 10² — survivable |
| 10⁴ | O using 1 Redis GET | **same** — LIMIT 100 caps the work regardless of pool size | full scan + sort of 10⁴ on **every request** — already painful |
| 10⁶ | O using 1 Redis GET | **same** — index seek touches only the freshest ≤100 rows | full scan + sort of 10⁶ on every request — **the cliff the banner forbids** |

The key property: **read cost and refresh cost are both independent of total eligible
row count.** The partial created at DESC when status is ACTIVE AND available for students
index + LIMIT 100 means even at 10⁶ rows we only ever read the freshest slice (≤100 rows),
once per re-roll window, off the hot path.

### Rejected alternatives
- **the random operation** — full scan + sort, runs on the **primary** on **every** request.
  Explicitly flagged in the scale-posture banner. Rejected outright.
- **TABLESAMPLE** — samples *physical pages*, not rows; gives a biased, clumpy sample,
  cannot honour "freshness-first," and behaves erratically on small/young tables (exactly
  the launch-week low-volume regime in spec §A6). Block-level sampling also can't express
  "ACTIVE + available" cheaply. Rejected.
- **Random-key / keyset seek** (when id random uuid LIMIT n) — unique identifier v4 PKs are not
  uniformly seekable for "pick N random," needs retry loops near the table tail, and
  still hits the database per request. It also can't do freshness-first without a second pass.
  Rejected as more complex and database-coupled than the cached pool for no benefit.
- **A materialized showcase table / MATERIALIZED VIEW** — durable, but adds a second
  source of truth, a REFRESH lock, autovacuum churn, and a migration for what is a
  pure read cache. Redis already exists and is the right tool for an ephemeral,
  per-window, throw-away projection. Rejected as over-engineering; revisit only if we
  ever need the pool to survive a full Redis flush (we don't — see degraded mode §A3).

---

## A2. The "Available" signal — tutor-SET flag (the last active at design is DELETED)

### What was removed (review-mandated deletion)
The previous design tracked a system-watched **users last active at** activity signal.
The review rejected it as non-consensual surveillance and an autovacuum hazard. The
following are **deleted from this design** and must **not** appear in the build:
- the **users last active at column**;
- the **Redis throttle / write-behind** ("active:seen:" NX gate, enqueue, batched flush);
- the **hot-path interceptor/filter** that wrote activity on each authenticated request;
- the **idx users last active** partial index;
- the related failure modes (old **F2** write-amplification, old **F11** index churn) —
  removed from §A6 because they no longer exist.

### Replacement — a boolean the tutor controls
A new column **tutor posts available for students BOOLEAN required DEFAULT yes**.

**Chosen home — tutor posts, not tutor profiles (justified):**
- The showcase candidate query already selects from tutor posts (§A1); the flag lives
  on the exact row the strip reads — **no extra join, no extra index on another table.**
- Eligibility is post-scoped (ACTIVE *post*), so availability is naturally per-listing: a
  tutor with multiple posts can pause one and keep another open.
- tutor profiles is the account-level identity record; per-listing operational state
  does not belong there. (Account-level **featuring consent** — below — is the only
  account-level switch.)

**Default TRUE** is honest for an advertised listing: an ACTIVE, approved post is already
a public offer to take students. The flag is a **visible, revocable toggle** in the
tutor's listing-management user interface; flipping it FALSE removes the post from the strip on the
next re-roll (≤ window). The card's **"Available" chip** renders from this flag only —
never from inferred activity. No false claim is ever made.

### Featuring consent (account-level, spec §A5)
The tutor opts into being **featured on the public landing page**. Default-ON is
acceptable for an advertised listing, but it is a **visible, revocable toggle**.
Implementation: a tutor profiles featured consent BOOLEAN required DEFAULT yes (the
one genuinely account-wide switch), joined in the candidate query
(AND tp owner profile featured consent yes). Revoking it excludes **all** the
tutor's posts from the strip on the next re-roll, independent of per-post availability.

### Liquibase changeset **0019** (next free) — plain support efficient lookup with an index, no CONCURRENTLY
The tables are tiny and the build applies migrations via **Testcontainers**;
CONCURRENTLY / run In Transaction is no has **zero precedent** in this codebase and
breaks the Testcontainers coverage build. Use a **plain in-transaction support efficient lookup with an index.**


Implementation example omitted. This public edition retains the written requirements in this section.


(Register 0019-.. implementation reference in the master changelog include list, following the 0018
precedent. The M14b tutee posts show on landing column + its index are **NOT** in 0019
— they ship only when the §B gate clears.)

---

## A3. Caching & freshness

### Keys & windows
| Concern | Value |
|---|---|
| Cache key (M14a) | showcase:tutors:v1 — one JSON **envelope** the results, the time they were built and whether the service is operating with reduced information, read in one GET |
| Caffeine micro-cache | **Required** in-process tier 2.5: last-good envelope, ~60s expiry period, sits in front of Redis (§A1 read path) |
| Built-at marker | showcase:tutors:built At (Singapore time-derived, for staleness/observability) |
| Re-roll window | showcase reroll.ms default **90s** (the "per server window" re-roll of spec §A1a) |
| Redis expiry period on the blob | **10 × window (≈15 min)** — deliberately *longer* than the re-roll cadence so a missed/lagging refresh still serves a recent value instead of a miss |
| Refresh trigger | a scheduled task job per the window (not per request) + **warm-on-startup** (Application Ready Event) |
| Single-flight lock | showcase:lock:tutors, SET NX EX 30s (refresh-only; reads never rebuild) |

**Staleness bound:** a visitor sees data at most **window + refresh duration** old in the
normal case (≤ ~95s), and at most **expiry period** old (≤ ~15 min) in the degraded "refresh job
fell behind" case. Both are well inside the product's honesty promise. The re-roll visibly
rotates cards across reloads without per-request database cost.

### Stampede protection + FAIL-CLOSED on miss (no uncoordinated database seek)
The scheduled job is the **only** writer, so steady-state has **no** stampede — reads
**never** trigger a rebuild. The only miss window is cold start (covered by warm-on-startup)
/ Redis flush / key expiry. The single-flight lock is on the **refresh**, not on reads:


Implementation example omitted. This public edition retains the written requirements in this section.


A read miss **fails closed** to the static fallback — it does not opportunistically seek
the database, because across N instances that would be N uncoordinated seeks. Only the
lock-holding refresh touches the database.

### Redis-down degraded mode (the page must still render)
A tiered fallback, in order, so the landing page **never** hard-fails and **never** falls
through to a raw the random operation (explicitly forbidden):

1. **Caffeine hit** — in-process last-good envelope (~60s). A brief Redis blip is invisible.
2. **Redis hit** — normal path; refreshes Caffeine.
3. **Redis miss / down / unreachable** — catch the exception and serve a **static, bundled
   fallback envelope** the results, the time they were built and whether the service is operating with reduced information (no database
   rows, no random query). The strip degrades to its §A6 acquisition-CTA layout. This serve
   increments the static-fallback source metric, emits a WARN log, and flips
   /actuator/health to DEGRADED (§A9).

This means **Redis is a performance dependency, not an availability dependency** for the
landing page — consistent with the architect non-negotiable "every external dependency has
a fallback." (Persistent Redis loss = permanent CTA fallback; durable/replicated Redis is
therefore a real M14a operational dependency — §9.)

---

## A4. New public endpoint

### One endpoint for M14a (the requests endpoint is gated — §B)


The illustration for one endpoint for m14a the requests endpoint is gated — §b contains the following information:

- GET showcase tutors interface operation → Showcase Envelope<Showcase Tutor Response> (≤ limit, default 20).


Registered as a **constant** (no magic strings): backend path constant + frontend application programming interface ROUTES public.showcase Tutors.
The Recent Requests endpoint (showcase requests interface operation) is **NOT** built in
this slice — it is part of the gated M14b (§B). The split-endpoint rationale (independent
cache granularity, failure isolation, parallel fetch, no meaningful round-trip penalty)
still holds for when M14b ships.

Do **not** reuse requests interface operation — it is @Pre Authorize-gated to TUTOR/ADMIN and
guards on owning an ACTIVE post (Public Browse Controller l.75–89). The showcase needs a
genuinely unauthenticated, *fixed-ceiling, no-filter, no-offset* projection.

**No pagination:** fixed limit (default 20, max 20), no offset, no filters. The cheapest
possible read shape — a single Redis GET returning a bounded envelope. A ?limit= query
param (clamped 1–20) is the only knob.

### Response envelope — not a bare array


Implementation example omitted. This public edition retains the written requirements in this section.


The frontend distinguishes **empty-but-fresh** (data: , degraded: no) from
**degraded-fallback** (degraded: yes) and renders the right §A6 state.

### Response data transfer objects — strict personal information-free subset (spec §A2, proves zero personal information)


Implementation example omitted. This public edition retains the written requirements in this section.


> Showcase Request Response is **not** part of this slice — it belongs to the gated
> M14b and is specified in §B2 with a stripped-down, re-identification-safe field set.

**Proof of zero personal information (spec §A2):** Every field above is copied from the existing
already-personal information-free public tutor data transfer object, *minus* fields. There is **no** email, phone, real
full name, user Id, exact address, MRT, or admin Notes. Location is **planning area +
region only** (coarse), and the frontend collapses it to planning Area or otherwise region or otherwise Singapore.
**location Mrt Station is intentionally dropped** even though the public data transfer object carries it.
This invariant is locked by the **personal information-free data transfer object contract test** (AC-5): an automated test
asserting the showcase data transfer object field set is a subset of the allow-list and contains none of the
deny-list field names (reflection over the record components).

**JSON (de)serialization (test seam, spec §A10):** the envelope is serialized on refresh
and deserialized on read through an **injected JSON serializer**. Both **exception branches**
(serialize failure on refresh → log + skip write; deserialize failure on read → treat as a
miss → static fallback) are explicit code paths and are **covered by tests** for the
97.5% line / 88% branch gate.

### Security Config changes — make ONLY this public
Add a single, **specific** GET matcher (not a broad /public/** permit — that would also
open the gated /public/requests):


Implementation example omitted. This public edition retains the written requirements in this section.


Place it **immediately after** the existing public-tutors **permit** block at
Security Config **lines 129–130** (the the request matchers operation block) and **before** the any request operation
(l.189–190). cross-site request forgery protection is already disabled for ** interface operation via the
ignoring Request Matchers entry at **line 108** (the cross-site request forgery protection ignore-list — *not* the permit),
and this is GET-only, so no cross-site request forgery protection change is needed. Restrict the matcher to GET so the
path can never carry a state-changing verb. requests interface operation keeps its
@Pre Authorize gate untouched. The M14b showcase-requests matcher is **NOT** added here —
it ships only with the gated M14b.

### Rate-limit / abuse posture (unauthenticated endpoint — AC-9)
Unauthenticated + cached means the blast radius is low (a flood hits Redis/Caffeine, not the
database), but an open endpoint still needs a ceiling. Reuse the existing
the is allowed operation (atomic sliding window) keyed on
client IP — resolved with the **same precedence already used in Auth Controller:
CF-Connecting-IP → X-Forwarded-For 0 → the get remote addr operation**:


Implementation example omitted. This public edition retains the written requirements in this section.


Add a the showcase key operation method to Rate Limit Service for consistency with the
existing login Key/register Key pattern (no magic strings).

### Cache-Control header (always set)
The endpoint sets **Cache-Control: public, max-age is 60** on every response so browsers and
any future CDN can absorb load at the edge. This is not optional — it is part of the M14a
contract (spec §A10 AC-9), independent of whether a CDN exists yet.

---

## A8. Frontend — dedicated showcase card (do NOT reuse Tutor Card)

The strip renders a **new component**, not Tutor Card:
- Tutor Card renders location Mrt Station via format Location — MRT must not appear on
  the personal information-free showcase card.
- Tutor Card wires **auth-only favourite mutations** that **401-redirect anonymous
  visitors** — fatal on a pre-login page.

Build a Showcase Tutor Card consuming a **new personal information-free frontend type** (mirrors
Showcase Tutor Response), with location rendered as **planning Area or otherwise region or otherwise Singapore**,
an **"Available" chip** from available, and **no** favourite/auth affordances. The
landing page reads from application programming interface ROUTES public.showcase Tutors and renders the §A6 states using
meta degraded / data length.

---

## A9. Observability backstop (IN SCOPE — zero-infra detectors)

The monitoring stack is not deployed, so M14a ships its own detectors:
- **source-split serve metric** — a counter/tag per serve: redis-hit / rebuild /
  static-fallback (the rebuild tag covers the warm-on-startup/scheduled build serve).
- **Structured WARN log** on **every** static-fallback serve **and every** refresh-failure
  (Redis unreachable, lock contention that left the blob stale past threshold, serialize
  exception), with correlation id.
- **/actuator/health DEGRADED indicator** — a custom Health Indicator reports DOWN/
  DEGRADED when the strip is serving fallback (latest serve source static-fallback or
  showcase:tutors:built At age > 3×window). Lets an operator see "the landing page is on
  fallback" with no metrics pipeline.

The applicant count is **omitted** in M14a (no count query, no count field). It is not part
of this slice; see roadmap §4 Decision 1 for the count's eventual home (owner-only, floored).

---

## A6. Failure modes

| # | Condition | Consequence | Mitigation (designed) |
|---|---|---|---|
| F1 | the random operation on primary | Full scan/sort per request; CPU/IO cliff at 10⁴–10⁶ rows | Never used. Cached precomputed pool; refresh is an indexed LIMIT 100 seek (§A1). |
| F3 | Redis down / unreachable | Landing page can't read the cache | Tiered fallback → Caffeine last-good → static CTA envelope (degraded is yes). Refresh **fails closed** (no database seek); read never runs a database query (§A3). |
| F4 | Cache miss (cold start / flush) | Risk of N instances rebuilding at once | Warm-on-startup populates at boot; refresh single-flight SET NX EX lock; read misses serve fallback, never seek the database (§A1/§A3). |
| F5 | Refresh job lags/dies | Cards go stale | expiry period = 10×window cushions a missed cycle; built At marker + health DEGRADED when age > 3×window (§A9). |
| F6 | Multi-instance duplicate refresh | Wasted database work each window | showcase:lock:tutors ensures one instance refreshes per window across the cluster (§A1/§A3). |
| F7 | Zero eligible rows | Blank/broken grid | Refresh writes an empty data (degraded is no); frontend renders the §A6 acquisition CTA. No fabrication. |
| F8 | Low volume (<6) | "Dead" 20-grid look | Honest short data array; frontend compact layout is a user interface concern (spec §A6). Backend never pads. |
| F9 | Showcase endpoint mistaken for gated requests | personal information / authz leak | Separate data transfer object + specific GET-only matcher for /showcase/tutors only; /public/requests gate untouched (§A4). |
| F10 | Scraping / abuse on open endpoint | Junk traffic | IP sliding-window rate limit (reuses Rate Limit Service) + Cache-Control + cache absorbs load (§A4). |
| F12 | JSON serializer serialize/deserialize failure | Bad write / unreadable blob | Explicit, tested branches: serialize-fail → log + skip write; deserialize-fail → treat as miss → static fallback (§A4). |

> Removed in this revision: the old **F2** (last active at written per request → write
> amplification) and old **F11** (last active at index churn). Both were tied to the
> deleted activity signal (§A2) and no longer exist — there is no per-request write.

---

## A7. Scale analysis (summary)

- **Reads:** Caffeine/O using 1 Redis GET, independent of total row count, at every scale
  (10²–10⁶). The database is **not** on the read path. Horizontally scalable: all app instances
  share one Redis blob; adding instances adds no database read load.
- **Refresh:** one indexed LIMIT 100 seek + in-memory shuffle per window per cluster
  (single-flight lock, warm-on-startup at boot). Constant work regardless of eligible-row
  count. Fails closed if Redis is unreachable — never an uncoordinated database seek.
- **Writes:** **none.** Availability is a tutor-set flag mutated only when the tutor edits a
  listing — there is no request-coupled write anywhere (the deleted last active at design
  was the only write hazard). The hottest table sees zero showcase-driven writes.
- **Aligns with the banner:** kills the the random operation cliff, introduces **no** hot-path
  writes, leans on already-durable Redis + the existing rate-limit primitive +
  @Enable Scheduling. The only new infra posture is durable/replicated Redis (already a
  scale-posture requirement).

---

## §B — M14b: Recent Requests strip (HISTORICALLY MARKED IMPLEMENTED — gate waived)

> **STATUS (2026-05-30): DEPLOYED (migration 0023).** The founder **waived** the §B1
> consent/legal gate. The as-shipped design lives in
> [../../architecture/m14b-public-job-postings-showcase md](../../architecture/m14b-public-job-postings-showcase.md);
> the §B content here is retained as history.
>
> *Historical (pre-waiver):* DEFERRED — SEND BACK by the 14-person review + Boss
> (2026-05-29). Kept specced so the work could be picked up cleanly once the gate
> cleared.

### B1. Gate (all must clear before any build)
1. **Opt-in consent column** — tutee posts show on landing BOOLEAN required DEFAULT no
   (a future changeset, **not** 0019). Default FALSE → **no retroactive publication**.
2. **Re-consent path** for existing posters (silence ≠ consent).
3. **Privacy-notice update at point of posting**, enumerating the exact fields shown.
4. **Legal / founder sign-off** on the public exposure + notice wording.

### B2. Stripped-down public card — re-identification-safe contract
The gated Showcase Request Response would carry **ONLY**:


Implementation example omitted. This public edition retains the written requirements in this section.


**Explicitly dropped** (must not appear): budget/band, frequency, venue/at-home, planning
area, MRT, session duration, #children, relationship-to-student, any free-text. The personal information-free
contract test for M14b is a **re-identification deny-list** — it asserts the card contains
**none** of these (not merely "no name/email"), because their combination is re-identifying
in a small market.

### B3. Endpoint + security (when built)
A separate read operation for showcase requests gated on show on landing yes, with
its own specific GET-only permit matcher. **Not** added to Security Config in M14a. It
**cannot** reuse requests interface operation (TUTOR/ADMIN-gated). When built it reuses the same
Redis-cached, fail-closed, envelope, observability, and rate-limit machinery as §A.

---

## 9. Roadmap sequencing & dependencies

The team review + Boss (2026-05-29) split M14: **M14a building, M14b deferred-gated.**

1. **M15 deployed/live** — *not* required for M14a. No applicant count is shown (it is
   omitted entirely in this slice), so M14a does **not** depend on M15.
2. **Migration 0019 (this design)** — adds tutor posts available for students,
   tutor profiles featured consent, and the partial showcase index; plain support efficient lookup with an index,
   no CONCURRENTLY. Required for M14a.
3. **Anon→signup funnel (Epic N, hard co-dependency — spec §A11)** — anon id minted
   client-side, submit operation for events batched ingestion, viewport-based impressions,
   anon id carried through registration, analytics events extended with nullable
   user id + anonymous id. Must ship **alongside** M14a or the thesis is unmeasurable.
4. **Durable/replicated Redis** — load-bearing for the landing page (degraded mode covers a
   blip; persistent loss = permanent CTA fallback). A real M14a operational dependency.
5. **Observability backstop (§A9)** — in scope for M14a, not deferred: source-split serve
   metric, WARN logs, /actuator/health DEGRADED indicator. These are the zero-infra
   detectors since the monitoring stack isn't deployed.
6. **M14b gate (§B1)** — consent column + re-consent + privacy notice + legal sign-off
   before any requests-strip build.

roadmap-next md item H (§2/§3/§5) and modules/INDEX md are reconciled to this split.
The earlier "real count from M15" framing is **dropped** — the count is omitted in M14a and
M14b alike.

---

## 10. Decision log (ADR-style summary)

| Decision | Choice | Rejected |
|---|---|---|
| Slice split | M14a (Active Tutors) builds now; M14b (Recent Requests) deferred-gated | Shipping both strips together (SEND BACK on requests) |
| Random-at-scale | Redis-cached per-window precomputed shuffled pool, in-memory freshness-first shuffle, injectable seed | the random operation, TABLESAMPLE, keyset/random-key seek, materialized view |
| Tutor "active" signal | **Tutor-SET tutor posts available for students flag** (per-post) + tutor profiles featured consent (account) | System-watched users last active at activity surveillance (**deleted**: column, write-behind, interceptor, index) |
| Availability flag home | tutor posts (query reads it directly, per-listing) | tutor profiles (account-level identity record; needs an extra join) |
| Endpoints | M14a: /public/showcase/tutors only, GET-only, envelope, no pagination | Building /showcase/requests now (gated); combined endpoint; reusing /public/requests |
| Response shape | Envelope the results, the time they were built and whether the service is operating with reduced information | Bare array (can't distinguish empty-fresh from degraded) |
| Caching | JSON envelope, 90s re-roll, expiry period 10×window, **required Caffeine tier**, warm-on-startup, single-flight SET NX EX, **fail-closed** static degraded mode | Per-request caching; read-path database rebuild; database fallback |
| Count | **Omitted** in M14a (no field, no query) | Per-card subquery; proxy/fabricated count (permanent no) |
| Migration 0019 | **Plain support efficient lookup with an index** (in-transaction) + 2 boolean columns | CONCURRENTLY / run In Transaction is no (zero precedent; breaks Testcontainers build) |
| Security | One specific GET permit matcher (/showcase/tutors); permit added after l.129–130, before l.189–190; cross-site request forgery protection ignore already at l.108 | Broad /public/** permit; adding the gated requests matcher |
| Observability | In-scope zero-infra backstop: source-split metric, WARN logs, health DEGRADED | Deferring observability to the monitoring epic |

---

*Cross-links:* requirements [spec md](spec.md) · scale posture
[status-briefing md](../../status-briefing.md) · roadmap [roadmap-next md](../../roadmap-next.md).
*Next step:* whole-team review (/team-review) → Boss sign-off before build, per project workflow.
