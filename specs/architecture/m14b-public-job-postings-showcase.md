# M14b — Public Job-Postings Showcase + Two-Audience Landing

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Status: **DEPLOYED (migration 0023)** · Author: Architect · Date: 2026-05-30
Supersedes the M14b deferral (consent/legal gate **waived by founder 2026-05-30**:
"it is ok to show posting of the job, no issue, i wants that").

## 1. Problem

The pre-login landing shows only the **Active Tutors** strip (M14a). But two
distinct audiences arrive logged-out:

- **Parents (demand):** "can I find a good tutor?" → served by the tutors strip.
- **Tutors (supply):** "are there real jobs worth signing up for?" → **not served
  at all today.** A tutor must register *and* post before they can see any job
  (requests interface operation is gated to TUTOR/ADMIN). That is backwards for
  acquisition: we ask tutors to commit before showing them the value.

We need a **public, anonymised, recency-based showcase of open job postings**, and
a landing that speaks to **both** audiences.

## 2. Goals / Non-goals

**Goals**
- A public, unauthenticated **Recent Job Requests** strip on the landing, mirroring
  the M14a tutors strip exactly (same cache/fail-closed/rate-limit/CLS posture).
- Reframe the landing so a logged-out **tutor** immediately sees "real jobs +
  list yourself", and a **parent** sees "real tutors + post a request".
- Reuse the existing anonymised tutee-post projection (BR-001 invariant).

**Non-goals**
- No consent flag / migration (founder waived). No change to the gated
  /public/requests browse (that stays for logged-in tutors with full filters).
- No applicant counts on the public strip (M15 applicant count stays admin-only,
  consistent with M14a "no applicant count").
- No new filtering user interface on the strip (it is a curated showcase, not the browse page).

## 3. Privacy posture (what is and isn't shown)

The tutee-post data is **already** visible to any TUTOR (free signup) via
/public/requests; this lowers the barrier from "sign up" to "none" for an
**anonymised subset**. Hard invariants, enforced by a personal information-free data transfer object contract test
(mirroring Showcase Tutor Response):

- **NEVER**: tutee user Id, name, email, phone, special Requirements (free text),
  admin Notes, location Mrt Station (too precise — dropped exactly as M14a drops
  tutor MRT).
- **Free-text title**: parent-authored → may inadvertently carry a child's name.
  **Decision for team:** default = **drop the raw title**; the frontend composes a label
  from subject + level (e g. "Secondary 3 · A-Math"). (Open item O-1.)
- **Shown (coarse, non-identifying):** post id (links into signup/login, not a
  user Id), subject, level, num Students, budget min/max,
  session Duration Hours, frequency Per Week, preferred Start Date,
  location Planning Area, location Region, created At.

## 4. Backend design

Mirror M14a 1:1 — same infra, second cache key.

- **Endpoint:** read operation for showcase requests on Showcase Controller
  (already showcase interface operation). Unauthenticated; per-IP rate-limited via the
  existing Rate Limit Service showcase Key ;  Cache-Control: public, max-age is 60.
  Returns Showcase Envelope<Showcase Request Response>.
- **data transfer object:** new Showcase Request Response record — the §3 "shown" fields only. Add a
  personal information-free contract test asserting the record has no forbidden components.
- **Service:** extend Showcase Service (or a sibling) with a second cached pool —
  Caffeine key + Redis blob key showcase:requests:v1, built-at key, single-flight
  lock showcase:lock:requests, the **same** freshness-first shuffle (select),
  static fail-CLOSED fallback (degraded:yes, empty list), warm-on-startup +
  scheduled re-roll. Reads NEVER hit the database; never the random operation.
- **Repository:** the find showcase candidates operation:


Implementation example omitted. This public edition retains the written requirements in this section.


  (No consent join — founder waived. LIMIT pool Size via Pageable, indexed on
  status, created at; verify index exists or add via Liquibase — O-2.)
- **Security:** Security Config must permit All showcase ** interface operation
  (already true for /showcase/tutors; confirm the glob covers /requests).
- **Observability:** reuse the showcase serve counter with a strip is requests tag
  (or a parallel metric); extend Showcase Health Indicator to cover both pools so a
  stale/degraded requests pool is visible (Nova/Seb gate).

## 5. Frontend design

- **New component** Recent Requests Strip implementation reference — clone of Active Tutors Strip:
  Tan Stack Query, retry:no, degrade-to-CTA on error/degraded/empty, skeleton
  CLS reservation, low-volume centred layout. **Never** trips the 401→login
  interceptor (endpoint is public; failures are caught). Tutor-facing CTA:
  "See all jobs" → /register?role is tutor / /login.
- **New card** Showcase Request Card implementation reference — subject·level label, students +
  relationship, budget range, frequency + duration, coarse location, "posted N days
  ago". No title, no contact.
- **Landing reframe** (Landing implementation reference) for **two audiences**:
  - Hero stays trust-led but adds a dual CTA: **"Find a tutor"** (parent) and
    **"Find tuition jobs"** (tutor).
  - Section order: Hero → **Active Tutors** (parents) → **Recent Job Requests**
    (tutors) → How-it-works (both) → trust/stats. Both strips lazy-loaded.
- **Constants:** new route API ROUTES public.showcase Requests =
  the public request-preview operation and its matching cache entry; response types
  Showcase Request + reuse Showcase Envelope. No magic strings.

## 6. Acceptance criteria

1. Logged-out visitor on / sees **both** a tutors strip and a job-requests strip;
   neither requires login; no redirect, no 401 bounce.
2. read operation for showcase requests returns 200 anonymous with an envelope;
   contains **none** of the §3 forbidden fields (contract test green).
3. Empty/degraded/rate-limited → acquisition CTA, never an error screen.
4. Coverage gates hold: backend ≥97.5% line+branch, frontend ≥97.5% line/88% branch.
5. Landing addresses both personas (tutor "find jobs" path is reachable pre-login).

## 7. Team Review Decisions (LOCKED — 2026-05-30)

Resolved by the panel + Tech Lead; these override §3–§5 where they differ.

- **O-1 (title):** DROP raw title. frontend composes a label from subject + level.
- **O-2 (index):** BLOCKING. New migration **0023** (head is 0022):


Implementation example omitted. This public edition retains the written requirements in this section.


  (Plain support efficient lookup with an index, not CONCURRENTLY — Testcontainers transactional build.)
- **O-3 (structure):** Generify into a reusable Showcase Pool<T> engine
  (owns its Caffeine key + Redis blob/built At keys + single-flight lock +
  Supplier loader + mapper + isolated the refresh operation/fail-closed serve + per-pool
  telemetry). Showcase Service holds TWO Showcase Pool instances. A failing
  requests refresh MUST NOT degrade the tutors pool (separate locks + guarded
  ticks).
- **O-4 (observability):** showcase serve gains a strip is tutors|requests tag
  (typed constant). Per-pool degraded/built At state (no shared scalars).
  Showcase Health Indicator reports each pool independently; requests-pool
  degraded is surfaced at LOWER severity than tutors (tutor vs parent acquisition
  blast radius). Alert: the age of the request preview exceeding three refresh intervals sustained.

### Privacy — minimum-disclosure projection (LOCKED)
Showcase Request Response exposes ONLY:
- post id (links to signup/login; NOT a user Id)
- subject Code, level Code, num Students
- budget Band — bucketed (NOT exact min/max); e g. round to nearest $10 band
- session Duration Hours, frequency Per Week
- start Window — coarse enum derived from preferred Start Date
  (WITHIN 2 WEEKS / WITHIN A MONTH / FLEXIBLE), NOT the exact date
- location Planning Area, location Region (coarse; kept for tutor travel
  decisions — founder-prioritised supply user experience; Casey's dissent: region-only)
- posted Days Ago / relative recency (NOT exact created At)

FORBIDDEN (asserted by BOTH a forbidden-field test AND a frozen positive
allow-list test): tutee user Id/name/email/phone, title, special Requirements,
admin Notes, location Mrt Station, exact budget Min/Max, exact dates.

Mapper touches SCALAR columns only — never the get venue preferences operation / the get tutee operation.

### Frontend (LOCKED)
- Hero keeps ONE primary CTA (parent "Find a tutor") + ONE secondary, relabelled
  "Find tuition jobs" → /register?role is tutor. Do NOT add a third CTA.
- Recent Requests Strip defers its fetch until in-view (Intersection Observer or
  enabled flag); own Suspense; identical skeleton CLS reservation; own unique
  section heading id.
- Regression test: a mocked 401 on the showcase-requests endpoint renders the CTA
  and does NOT trigger the redirect.
- Post-creation form (Create Request) shows a visible privacy line: "Your name,
  contact, and exact address are never shown publicly."

### Boss conditions (LOCKED — must hold at implementation review)
- Security Config: add **explicit** request Matchers(GET,
  "showcase requests interface operation").permit All operation— NO/showcase/** glob
  exists today; prove with an anonymous-200 integration test.
- Location override conditions ship TOGETHER (budget bucketed, start-window enum,
  relative recency, title dropped) — proven by forbidden-field AND frozen
  positive-allow-list tests. If any slips, location reverts to region-only.
- Mapper scalar-only; N+1 guard test (no extra database query language per element).
- Showcase Pool<T> isolation test: forced requests-pool refresh failure leaves
  the tutors pool serving fresh.
- Showcase Health Indicator → per-pool reporting (requests at lower severity);
  document the new /actuator/health showcase detail shape (changes a deployed
  contract).
- Reuse existing the find all active posts operation — do NOT add a
  duplicate find Showcase Candidates.

### Wire contract (LOCKED field names — keep the private implementation folder in sync)
Showcase Request Response: id:unique identifier, subject Code:text, level Code:text,
num Students:whole number, budget Band Min Sgd:whole number
(floor to $10), budget Band Max Sgd:whole number (ceil to $10), session Duration Hours,
frequency Per Week:enum, a start window of within two weeks, within a month or flexible
(from preferred Start Date: ≤14d / ≤31d / else|null), location Planning Area:text,
location Region:enum, posted Days Ago:whole number (from created At, server clock).

### Residual risk (accepted, pending Boss/founder)
Keeping planning area + bucketed budget + coarse start window is a deliberate
supply-user experience-over-strict-minimisation trade the founder asked for. Re-identification
by combination is reduced (bucketed budget, coarse dates, dropped title) but not
zero. Documented here for explicit sign-off.
