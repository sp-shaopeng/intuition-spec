# M14 — Landing Showcase (Active Tutors + Recent Requests)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

Use public handles, not real/display names. The founder-approved [M14b amendment](../../architecture/m14b-public-job-postings-showcase.md) supersedes early consent-gate narrative, and M18 makes tutor profile identity canonical. Anonymous visitors may filter limited public results (confirmed 2026-09-05); full detail remains login-gated. Old data transfer object/eligibility/filter examples below are historical where they disagree. This does not invent a new availability/recency eligibility decision.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** **M14a (Active Tutors) DEPLOYED (migration 0019); M14b (Recent Requests)
DEPLOYED (migration 0023)** — the consent/legal gate was **waived by founder
2026-05-30** ("it is ok to show posting of the job"). The §B gating narrative below
is retained as history; the shipped design is in
[../../architecture/m14b-public-job-postings-showcase md](../../architecture/m14b-public-job-postings-showcase.md).
**Author:** Quinn (Product Designer) → revised by the solution architect (Solution Architect) + Chris
(Technical Writer) per the 2026-05-29 review + Boss decision.
**Date:** 2026-05-29 (M14b status updated 2026-05-30)
**Supersedes:** the DEFERRED Solution Architect draft of 2026-05-29 (same path).

**History note:**
- An earlier draft proposed a per-job **applicant-count proxy** from interview
  signals; that proxy was **CUT** in team review + Boss as dishonest (both Platform
  Users rejected it). The proxy is dropped permanently.
- On 2026-05-29 a **14-person team review + Boss** returned **SEND BACK on the Recent
  Requests strip** (a publicly broadcast projection of tutee request data with no
  opt-in consent, no privacy-notice update, and no legal sign-off) and gave a **green
  path to the Active Tutors strip**. The founder chose to **ship the tutors strip
  first and gate requests.** This revision splits M14 into two independently-shippable
  slices accordingly.

**The split (founder decision, 2026-05-29):**
- **M14a — Active Tutors strip: PROCEEDING to build.** Buildable now; this spec/design
  is implementation-ready.
- **M14b — Recent Requests strip: DEPLOYED (migration 0023).** Originally deferred-gated
  on consent + legal sign-off; the **founder waived that gate on 2026-05-30** and the
  strip shipped. §B below is retained as the historical gated design.

**Driver:** Founder — "On the main landing page, *before login*, show ~10–20 tutors
available now and ~10–20 recent postings; always random, but real rows from the database."
The active-tutor signal is a **tutor-set "Available" status**, not a system-watched
activity signal (see §A4) — surveillance of users was rejected in review.
**Cross-ref:** [roadmap-next md](../../roadmap-next.md) item H §2/§3/§5 (M14 status
reconciled to this split). Reactivating M14a **re-sequences the roadmap** — see §9.

---

## 1. Goal & user value

Up to two public, unauthenticated showcase strips on the landing page (/), seen
**before** a visitor logs in or registers:

1. **Active Tutors (M14a — building)** — ACTIVE tutor listings whose owner has set
   themselves **"Available for students"** and has opted into being featured.
2. **Recent Requests (M14b — DEFERRED, gated)** — tutee job posts created recently,
   shown **only** with explicit opt-in consent (§B).

**User job (JTBD):** A first-time visitor is deciding *"is this marketplace real and
worth signing up for?"* in the first 10 seconds. An empty or static page answers
"no." Live, real, available tutors answer "yes — there are real tutors here ready to
take students."

**Why it serves acquisition/trust:**
- **Demand-side visitor (a parent/tutee):** sees real, available *supply* ("there are
  tutors here ready to take my child now") → reason to post a request. (M14a serves
  this directly.)
- **Supply-side visitor (a tutor):** sees real, recent *demand* → reason to register.
  (M14b serves this — gated.)
- **Liveness/trust:** for M14a the honesty mechanism is the tutor-controlled
  **Available** flag plus the ACTIVE-post + 7-day recency context — it proves the
  marketplace has tutors *currently inviting students*, not a graveyard of stale
  listings. We do **not** infer "active" by watching users (rejected in review); the
  tutor declares availability.

This is **social proof, not a search tool.** It is a teaser that drives signup; it is
not the browse experience (that already exists post-login under M2/M10).

---

## Document structure — the two slices

This spec is split so M14a can be implemented standalone:

- **§A — M14a (Active Tutors): the buildable slice.** Scope, eligibility, the
  tutor-set Available flag, featuring consent, empty/low-volume behaviour, Mo S Co W,
  acceptance criteria, and the co-dependency on the anon→signup funnel.
- **§B — M14b (Recent Requests): DEFERRED, gated.** Kept specced with its consent +
  privacy + legal gate and a stripped-down personal information-free card contract. **Not for build.**
- **§9 — roadmap re-sequencing** (applies to both slices).

The architecture/mechanism (design md) is split the same way (§A / §B).

---

# §A — M14a: Active Tutors strip (BUILDING)

## A1. Scope — what the Active Tutor card shows

The fields below already exist on the public-safe data transfer object (Tutor Post Response) verified
in code, **except** the tutor-set Available flag (§A4). The showcase card is a
**subset** of that existing public projection — never a superset. No new personal information fields
are introduced. The frontend renders a **dedicated showcase card**, not the existing
Tutor Card (§A8).

Current card identity follows M18 profile-canonical privacy:
- Tutor **public handle only** (Handle / tutor Public Handle), never a real/display name, email, phone or user id
- Headline / title
- Subjects + levels
- Hourly-rate band (hourly Rate Min Sgd–hourly Rate Max Sgd)
- General location only — **planning Area or otherwise region or otherwise Singapore** (coarse; MRT is
  **never** sent on the showcase card — see §A8)
- average Rating + review Count (already public, nullable when no reviews)
- An **"Available" chip** rendered from the tutor-set available for students flag
  (§A4) — **not** from any system-watched activity signal.

### A1a. Counts, window, ordering
| Aspect | Product decision |
|---|---|
| **How many** | Target **10–20**. Treat as a *display ceiling*, not a guarantee — show what exists (§A6). Single tunable limit (default 20). |
| **Eligibility** | ACTIVE tutor post **AND** available for students yes **AND** owner opted into featuring (§A4/§A5). |
| **Recency context** | The 7-day window + any "N days ago" label are computed in **Singapore time** (rolling 7×24h) — see §A12. There is **no** system "active in 7 days" surveillance signal; "active" is the tutor-declared Available flag. |
| **Random requirement** | Selection must be **randomised across the eligible pool** so the strip feels fresh and no single post dominates. Product requires the *effect* of randomness; it does **not** prescribe the random operation — see design md §A1. |
| **Within-strip ordering (user experience)** | **Freshness-first**: newest eligible items lead, randomisation fills the remaining slots. A pure random feed tested as "unhelpful" with the supply-side persona; newest-first respects the liveness promise while still rotating. |
| **Refresh behaviour** | The selection is **re-rolled on an interval** (server-side, per refresh window), not per request. A visitor reloading need not see identical cards, but the page must not thrash. |

---

## A2. Eligibility & exclusions (privacy + content rules)

**Only ACTIVE/approved content is eligible.**
- tutor posts status ACTIVE only, and the owning tutor account must be
  approved/active. No unapproved tutors, no suspended accounts, no soft-deleted users.
- tutor posts available for students yes (§A4).
- Owner has not revoked featuring consent (§A5).

**Zero personal information — hard rule (pre-login, fully public, unauthenticated).** The card must
expose **no** email, phone, real full name, exact address, MRT station, or any
user Id. Tutors surface **the public handle only**, not tutor Display Name; the old data transfer object name must not be
read as permission to publish a real/display name. Location is **coarse only** (planning area /
region / "Singapore" fallback). This invariant is enforced by a **personal information-free data transfer object
contract test** (the showcase data transfer object is a strict subset of the already-personal information-free public
data transfer object — see design md §A4).

---

## A4. The "Available" signal — tutor-SET status (NOT system-watched activity)

The review **rejected** a system-watched activity signal (users last active at
written on every authenticated request): it is surveillance the user never consented
to, and it created an autovacuum/write-amplification hazard on the hottest table. It
is **removed entirely** (no column, no Redis write-behind, no hot-path interceptor, no
index — see design md §A2 for the deletion).

**Replacement: a boolean the tutor controls.** A new
tutor posts available for students BOOLEAN required DEFAULT yes.

**Chosen home — tutor posts, not tutor profiles** (architect call):
- The showcase query already selects from tutor posts; the flag lives on the row the
  strip reads, with no extra join.
- Eligibility is already post-scoped (ACTIVE *post*), so availability is naturally
  per-listing — a tutor with two posts (e g. one subject paused, one open) can mark
  them independently.
- tutor profiles is the account-level identity record; per-listing availability does
  not belong there. (Featuring *consent* in §A5 is the broader account-level switch;
  availability is the per-post operational state.)

**Default TRUE** is acceptable: an ACTIVE, approved listing is, by its nature, an
advertisement that the tutor wants students. The flag exists so a tutor can **opt out
of the showcase / mark themselves full** without taking the whole listing down. It is
a **visible, revocable toggle** in the tutor's listing management user interface.

The **"Available" chip** on the card renders from this flag only. We never claim
"active this week" off surveillance; the chip means "the tutor says they're taking
students." This is honest by construction.

---

## A5. Tutor featuring consent

Separate from the per-post Available flag, the tutor **opts into being featured
publicly** on the pre-login landing page. Because an ACTIVE listing is already an
advertised, public-facing offer, **default-ON is acceptable**, but it **must be a
visible, revocable toggle** in the tutor's account/listing settings. A tutor who
opts out is excluded from the strip immediately on the next re-roll, regardless of the
Available flag. (Implementation home and contract: design md §A2/§A4.)

---

## A6. Empty / low-volume behaviour (a real product decision)

This matters most **pre-launch and in the first weeks**, exactly when the page is
seen by the earliest, most skeptical visitors — and exactly when volume is lowest.

**Rules (product, decisive):**
1. **Never fabricate or pad.** No fake rows, no demo data, no "ghost" cards. The
   founder's "real rows from the database" is a hard honesty constraint.
2. **Show what exists, down to zero.** If only 4 eligible tutors exist, show 4. The
   strip degrades gracefully, it does not break.
3. **Dedicated low-volume layout.** 3 cards must not look like a broken 20-card grid.
   Below a threshold (recommend < ~6), render a **deliberately compact layout**
   (centered row / carousel) so few cards read as "curated," not "dead." This is a
   real layout requirement for the user interface/user experience designer, not an afterthought.
4. **True-empty state (zero eligible).** If the strip has **0** eligible items, **hide
   it entirely** and show a forward CTA in its place — e g. *"Tutors: list your
   profile."* An empty strip is converted into an acquisition prompt rather than an
   embarrassing blank. The envelope (§A7) carries meta degraded so the frontend can tell
   an honest empty-but-fresh strip from a degraded fallback.
5. **The Available/eligibility definition is NOT relaxed to fill the grid.** Do not
   silently include unavailable or non-featured tutors to pad. If volume is
   chronically too low, that is a marketplace-liquidity signal for Business, not a
   reason to weaken eligibility.

---

## A7. Response shape — envelope, not bare array

Each showcase endpoint returns an **envelope**, not a bare array, so the frontend can
distinguish *empty-but-fresh* from *degraded-fallback*:


The a7. response shape — envelope, not bare array example describes information including data, meta, built At, degraded. Concrete payload values and implementation syntax are omitted.


Routes are registered in application programming interface ROUTES public (frontend) and as application programming interface ROUTES/path constants
(backend) — **no raw string literals**. See design md §A4.

---

## A8. Dedicated frontend showcase card (do NOT reuse Tutor Card)

The Active Tutors strip uses a **new, dedicated showcase card component** — it must
**not** reuse the existing Tutor Card:
- Tutor Card renders location Mrt Station via format Location — MRT is personal information-adjacent
  precise location that the showcase **must not** expose.
- Tutor Card wires **auth-only favourite mutations** that **401-redirect anonymous
  visitors** — fatal on a pre-login page.

The showcase card consumes a **new personal information-free type** (mirrors the backend Showcase Tutor Response
data transfer object), with location computed as **planning Area or otherwise region or otherwise Singapore** and **no**
favourite/auth affordances. (Component + type: design md §A8 / frontend notes.)

---

## A9. Observability backstop — IN SCOPE (not deferred)

The monitoring stack is not deployed, so M14a ships its own **zero-infra detectors**
(in scope, not deferred):
- A **source-split serve metric** for each strip: redis-hit / rebuild /
  static-fallback.
- A **structured WARN log** on **every** static-fallback serve and **every**
  refresh-failure.
- An **/actuator/health DEGRADED indicator** when a strip is serving fallback.

These let an operator detect "the landing page is on fallback" without a metrics
pipeline. (Design: design md §A9.)

---

## A10. Mo S Co W & acceptance criteria (M14a)

### Mo S Co W
**Must Have**
- Public, unauthenticated **Active Tutors** strip: ACTIVE tutor posts with
  available for students yes and featuring consent, randomised (freshness-first),
  coarse fields only, personal information-free.
- Tutor-set **Available** toggle (per-post) + featuring-consent toggle (account),
  both visible and revocable.
- ACTIVE-only eligibility; Singapore time 7-day window/labels.
- Envelope response (the results, the time they were built and whether the service is operating with reduced information); routes in application programming interface ROUTES public.
- Dedicated personal information-free frontend showcase card (not Tutor Card).
- Graceful low-volume + true-empty behaviour (§A6).
- personal information-free data transfer object contract test; rate-limited public endpoint; Cache-Control: public, max-age is 60.
- Observability backstop (§A9): source-split metric, WARN logs, health DEGRADED indicator.
- Caching machinery: Redis precomputed per-window shuffled pool, **warm-on-startup**
  (Application Ready Event), refresh **fail-CLOSED** if Redis is unreachable, a
  **required Caffeine micro-cache tier** in front of Redis, and Cache-Control: public,
  max-age is 60 headers (design md §A1/§A3).
- **Anon→signup funnel co-dependency (§A11)** ships alongside M14a.

**Should Have**
- Geo-relevant ordering (e g. nearest region first) — iteration 2 with data.

**Won't Have (this cycle)**
- Any fabricated/padded rows or demo data — **permanent no.**
- Any system-watched "active" surveillance signal — **rejected; permanent no.**
- Filtering/search/pagination on the strip (teaser, not browse — M2/M10 own browse).
- Applicant count (that lives in M14b/M15 territory and is omitted here).

### Acceptance criteria — "done when…"
1. An unauthenticated visitor at / sees up to 20 Active Tutor cards, drawn from
   **real ACTIVE database rows** with available for students yes and featuring consent.
2. Every card is from an ACTIVE tutor post whose owner is approved/active and has
   not revoked featuring; no unavailable/non-featured/suspended tutor ever appears.
3. The "Available" chip renders **only** from available for students — no system
   activity signal exists anywhere in the build.
4. The selection visibly rotates across refreshes (re-rolled per window), randomised
   across the eligible pool, freshness-first within the strip; the shuffle uses an
   injectable seed and the window uses an injectable Clock (§A10 test seams).
5. **No card** exposes email, phone, real full name, exact address, MRT station, or
   any user Id — proven by an automated personal information-free data transfer object contract test (allow-list +
   deny-list).
6. With < 6 eligible items the strip renders the compact low-volume layout; with 0
   eligible items the strip is replaced by its acquisition CTA. meta degraded
   distinguishes empty-but-fresh from fallback. No blank grid, no fabricated rows.
7. The response is the envelope the results, the time they were built and whether the service is operating with reduced information; frontend routes
   come from application programming interface ROUTES public.
8. On Redis unavailability the endpoint serves the static fallback with
   meta degraded yes, emits a WARN log, increments the static-fallback metric,
   and /actuator/health reports DEGRADED — and **never** runs a database query
   uncoordinated (refresh fails CLOSED — design md §A1/§A3).
9. The public endpoint is rate-limited, serves without authentication, and sets
   Cache-Control: public, max-age is 60.
10. Anon→signup funnel events (§A11) are captured (no personal information) so the funnel is
    measurable from day one — hard co-dependency, M14a is not "done" without it.

### Test seams (for the 97.5% line / 88% branch gate)
- The freshness-first **shuffle takes an injectable Random/seed**.
- The **7-day window + any throttle use an injectable java time.Clock** (Singapore time).
- JSON (de)serialization goes through an **injected JSON serializer** with its
  **exception branches explicitly noted/tested** (serialize-fail on refresh,
  deserialize-fail on read → counts as a fallback path).

---

## A11. Hard co-dependency — the anon→signup funnel

M14a's entire thesis ("the showcase drives signups") is **unmeasurable** without an
anonymous→signup funnel. This is a **hard co-dependency**: it must ship **alongside**
M14a, not after.

Scope of the funnel (rides Epic N instrumentation):
- **anon id minted client-side** (first landing-page visit; stored locally).
- **submit operation for events** ingestion endpoint (batched, no personal information).
- **Viewport-based impressions** (card seen, not just rendered), card-click, and
  signup-start events — **batched**, keyed on anon id + post Id, **never personal information**.
- The anon id is **carried through registration** so an anonymous session can be
  joined to the resulting account.
- The **Epic N analytics events schema is extended** with nullable user id **and**
  anonymous id columns to support the pre-/post-signup join.

Without this, AC-10 cannot pass and M14a is not "done."

---

## A12. Timezone

The 7-day recency context and any **"N days ago"** label are computed in **Singapore time**
(Asia/Singapore), as a **rolling 7×24h** window (not a calendar week). The injectable
Clock (§A10) makes this testable. (Tutor availability is a flag, not time-based, so
Singapore time applies to the recency framing and labels, not to an activity threshold.)

---

# §B — M14b: Recent Requests strip (HISTORICALLY MARKED IMPLEMENTED — gate waived)

> **STATUS (2026-05-30): DEPLOYED (migration 0023).** The founder **waived** the
> consent/legal gate below ("it is ok to show posting of the job"). The original
> DEFERRED / DO-NOT-BUILD banner is retained as history. The as-shipped design is in
> [../../architecture/m14b-public-job-postings-showcase md](../../architecture/m14b-public-job-postings-showcase.md).
>
> *Historical (pre-waiver):* Returned SEND BACK by the 14-person review +
> Boss on 2026-05-29: publicly broadcasting tutee request data — even
> already-anonymised — with no opt-in consent, no privacy-notice update, and no legal
> sign-off was not acceptable. Kept specced so it could be picked up cleanly once the
> gate below was cleared.

## B1. The gate (all must clear before any build)

1. **Opt-in consent at the data level:** a new
   tutee posts show on landing BOOLEAN required DEFAULT no. Default **FALSE** —
   **no retroactive publication** of existing rows. A request appears on the landing
   page **only** if its owner explicitly opted in.
2. **Re-consent path:** existing posters must be given an explicit, revocable opt-in;
   silence ≠ consent.
3. **Privacy-notice update at the point of posting:** the post-creation flow must
   state, at the moment of opt-in, that the (stripped-down) request may be shown
   publicly on the pre-login landing page, with the exact fields enumerated.
4. **Legal / founder sign-off** on the public exposure and the privacy-notice wording.

## B2. Stripped-down public card — personal information-free contract

When (and only when) the gate clears, the public Recent Request card shows **ONLY**:
- Region
- Subject
- Level
- **Day-granularity** recency ("posted ~3 days ago" — never finer than a day)

**Explicitly dropped** (must NOT appear on the public card): budget / budget band,
frequency per week, venue / at-home preference, planning area, MRT station, session
duration, number of children, relationship-to-student, and any free-text. The card is
deliberately coarser than the existing Tutee Post Response.

The personal information-free contract test for M14b is a **re-identification deny-list**: it asserts
the public card contains **none** of the dropped fields above (not merely "no name/
email") — because the combination of budget + frequency + planning-area + MRT +
duration + #children is itself re-identifying in a small market.

## B3. Auth-boundary note (carried from the original spec)

The existing tutee **requests** browse (read operation for requests) is
@Pre Authorize-gated to TUTOR/ADMIN and additionally guards on the caller owning
an ACTIVE post. M14b's strip is **pre-login**, so it **cannot reuse that endpoint** —
it would need its own unauthenticated projection gated on show on landing yes.
This is precisely the new public surface the review refused to approve without the §B1
gate.

---

## B4. Public browse pagination-cap contract (INT-012, amended INT-136e 2026-06-12)

The public browse endpoints (read operation for tutors, read operation for assignments)
carry an **anonymous anti-enumeration cap**: page ≤ 4 (0-based) and size capped at 20
— a hard ceiling of **100 anonymously reachable rows** (5 pages × 20). /assignments is
additionally rate-limited per-IP (60/min).

**Exact behaviour (INT-136e):**
- **size 20 → CLAMPED to 20** (web request protocol 200, meta page Size 20, first 20 rows served). A
  too-large size is an innocent client mistake; the clamp serves exactly what page 0 of a
  compliant request returns, so the ceiling is untouched. (Pre-fix this returned a confusing
  silent-empty page that cost our own IT author a failed run.)
- **page 4 → EMPTY page** (web request protocol 200, data). A too-deep page is the enumeration
  vector itself — clamping it would re-serve page-4 content under a lying page number
  (dishonest pagination), so the empty page stays. **Do not "fix" this into full
  pagination** — the empty page IS the INT-012 anti-enumeration control.
- **size 1 or page 0 → EMPTY page** (malformed, not innocent).

**Caller scope (Boss build-time note, 2026-06-12):**
- /public/tutors: the cap (page bound + size clamp) applies to **anonymous callers only**
  — authenticated users of any role page freely.
- /public/assignments: the cap applies to **ALL callers** — the endpoint is
  anonymous-only by design and has **no authenticated bypass**, unlike /tutors.

Live guards: Browse Assignments IT / Browse Tutors IT (clamp serves rows; deep page stays
empty); per-pull request matrix in Public Browse Controller Test.

---

## 9. Roadmap re-sequencing

In [roadmap-next md](../../roadmap-next.md), M14 is item **H**. The team review +
Boss decision (2026-05-29) splits it:

**M14a (Active Tutors) — building.** The product spec and architecture are
implementation-ready. M14a does **not** show any applicant count and does **not**
depend on M15 to render. Its real gates are:
- The **anon→signup funnel** ships alongside it (hard co-dependency, §A11), riding
  Epic N instrumentation (with the analytics events schema extended for
  anonymous id + nullable user id).
- Durable/replicated Redis (the strip's cache and refresh lock; degraded mode covers a
  blip, persistent loss = permanent CTA fallback — design md §A3).

**M14b (Recent Requests) — DEFERRED, gated.** Not built. Gated on opt-in consent
(tutee posts show on landing, default FALSE, no retroactive publication), re-consent
path, privacy-notice update at point of posting, and legal/founder sign-off (§B1).

roadmap-next md item H (§2/§3/§5) and modules/INDEX md are reconciled to this
split status. The earlier "real count from M15" framing is **dropped** — the applicant
count is **omitted** in M14a, and M14b carries no count either.
