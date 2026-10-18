# Assignment Lifecycle — Product Design (apply → interview → assign → close)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** [CURRENT-REQUIREMENTS](../CURRENT-REQUIREMENTS.md) controls agency commission receipt, separate first-session/lesson-payment events and public access. Post assignment status is MATCHED; contact follows participant authorization plus the ACTIVE/COMPLETED frontend rule. Ordinary in-app notifications stay in-tx; independently committed analytics is deferred after commit. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Status:** ✅ team-reviewed (full debate) + Boss SEND-BACK conditions applied → **APPROVED to drive
the product requirements document** · **Date:** 2026-05-31
**Owner:** Product (Quinn/Dana) + Tech Lead · **Source of truth:** the service code.
**Drives:** product requirements document §4 (journeys), §5 (M4/M6/M15 module specs), §7 (data model), §8 (permissions).

---

## 1. Why

Today the pieces exist but the **assignment lifecycle is not wired end-to-end**. Verified in code
(Job Application Service, Admin Match Service, Admin Application Controller, model/*):

- Real enum is **application states: applied by default, withdrawn, shortlisted, rejected or matched**
  (verified in Job Application Status implementation reference) — **no "superseded"/"not applicable"** state for
  "the job was taken by another tutor." (apply sets APPLIED; withdraw sets WITHDRAWN;
  **SHORTLISTED, REJECTED, and MATCHED appear only in the seeder — no service path transitions an app
  to any of them**, so they are currently dead.)
- **the create match operation** builds a PENDING match
  linked to tutee Post + tutor Post, emits two notifications, and returns. It does **NOT**: (a)
  close the job, (b) mark the winning application MATCHED, (c) supersede the other applications, or
  (d) write any profile record. The service doesn't even inject Job Application Repository. The tutee
  post stays ACTIVE and every other tutor's application stays APPLIED forever.
- The applicant list exists but is **ADMIN-ONLY** (Admin Application Controller list Applicants →
  list Applicants For Job). **Parents cannot see who applied to their own job today** — that is a gap
  the founder's flow requires.
- **Tutor Profile has NO rate/area/commitment columns today** (verified: it holds only
  display Name, qualifications, years Experience, bio, featured Consent). Rate + area live **only on
  Tutor Post** (hourly Rate Min Sgd/Max, location Planning Area/Region/Mrt Station). So
  "profile-based applicant card" is a **hard schema + backfill MUST**, not an "if not present" maybe.
- **the apply operation gates on an ACTIVE tutor listing** (Job Application Service →
  TUTOR NO ACTIVE POST 403). This **contradicts the free-market model** (below) and must be removed.

## 0. MODEL: free marketplace (founder, 2026-05-31)

**Any tutor can express interest / apply to any active job — no listing required.** Applying is an
open expression of interest, not gated behind owning a published listing.

Consequences that ripple through this whole design:
- **Remove the TUTOR NO ACTIVE POST guard** in the apply operation (delete Job Application Service).
  tutor Post Id on the application stays **optional** (it already is) — a tutor may attach one of
  their listings as a pitch, or apply with none.
- **Applicant card must source from the tutor PROFILE, not a listing.** Because an applicant may have
  no post, the rate/area/YOE/PT-FT/rating shown to the parent come from tutor profiles
  (+ aggregate rating), with the *optional* attached listing as extra context. This is the second
  reason **PT/FT lives on the profile** (founder's other call).
- Tutor profile becomes the **canonical public identity** (rate range, area, YOE, PT/FT, rating);
  listings become optional showcases. Profile setup must therefore capture rate range + area + PT/FT.
- Removes the catch-22 where a new tutor can't apply until they publish a listing — friction the
  free-market model explicitly rejects (Wei's pain point).

The founder's target flow (verbatim intent): *tutor applies → application stored → parent sees who
applied and can request an interview → tutor sees their applications + interview sessions → parent
browses tutors → tutor detail shows that tutor's listings → **when a job is taken, it leaves a record
on the profile and other applications become "not applicable"** → admin has dedicated dashboards for
approval, interviews, and job-status/assignment.*

## 2. Domain model — target

### 2.1 Job (Tutee Post) status — REUSE existing Post Status MATCHED, do NOT invent ASSIGNED
Actual Post Status enum (verified): DRAFT, PENDING, ACTIVE, REVISION REQUESTED, REJECTED,
**MATCHED**, CLOSED, EXPIRED. **No new enum value is needed** — "assigned" maps onto the existing
**MATCHED**.
- **ACTIVE**: visible in public browse, accepting applications.
- **MATCHED** (= assigned): a tutor has been chosen; no longer in browse; **not accepting new
  applications**; existing other applications → **SUPERSEDED**.
- **CLOSED**: assignment completed (or job withdrawn).
- On cancel (Q1): **MATCHED → ACTIVE** (reopen).

### 2.4 State-transition table (job × application × match)

| Event | Job (Post Status) | Winning application | Other applications | Match |
|---|---|---|---|---|
| tutor applies (free market) | ACTIVE (unchanged) | — | — (this app → APPLIED) | — |
| parent requests interview / **chooses** | ACTIVE | (no status change yet) | — | — (selection request only) |
| **admin confirms → closes job** (§3.1) | ACTIVE → **MATCHED** | APPLIED/SHORTLISTED → **MATCHED** | APPLIED/SHORTLISTED → **SUPERSEDED** | create **PENDING** (source application id set) |
| admin activates | MATCHED | MATCHED | SUPERSEDED | PENDING → **ACTIVE** (commission PENDING) |
| **agency receives tutor commission → admin marks PAID** | MATCHED | MATCHED | SUPERSEDED | ACTIVE → **COMPLETED** (commission **PAID**) |
| admin cancels (pre-complete) | MATCHED → **ACTIVE** | MATCHED → **APPLIED** | SUPERSEDED → **APPLIED** | → **CANCELLED** (commission CANCELLED) |
| tutor withdraws own app | (unchanged) | — | APPLIED → **WITHDRAWN** | — |

### 2.2 Application status (Job Application Status) — wire it up + ADD SUPERSEDED
Current: APPLIED → WITHDRAWN (only these two transition in code); SHORTLISTED, MATCHED exist
but are never set by a service. Target:
APPLIED → SHORTLISTED → MATCHED (winner) · → WITHDRAWN (tutor) · **→ SUPERSEDED** (NEW — the
job was assigned to someone else; this application is now "not applicable", read-only, shown to the
tutor with a clear label). Optional → REJECTED (parent declines a specific applicant) — **Q5**.

### 2.3 Assignment record (profile history)
When a job is assigned, write a durable record visible on the **tutor profile** ("Assignments")
and the **parent's history**. Minimum: subject/level, period (start/until), status
(ACTIVE/COMPLETED). personal information rules below. *(Implementation: the existing Match row already links
tutee/tutor/posts + timestamps; the profile "assignment record" is a projection over Match — no new
table strictly required, but a Match source application id FK is added to tie the winning app.)*

## 3. Assignment flow (founder, 2026-05-31) — parent-initiated, admin-confirmed

The full lifecycle the founder described, end to end:

1. **Parent acts** on an applicant: either **request an interview** *or* **choose this tutor**
   (a "selection request"). Both are signals from the parent, not the final assignment.
2. **Admin receives the request** (new admin queue item) and **liaises with the tutor** off/on
   platform to confirm fit + availability.
3. **Both confirm** → **admin closes the job** = the atomic "assign" transition (§3.1).
4. Lessons begin. **Parent lesson payment and tutor commission remittance are separate events. Admin marks commission PAID only after the agency receives the tutor commission.**
5. **Admin marks the job/commission PAID** → lifecycle complete.

States map onto existing infra: parent-choose → a **selection request** the admin sees; admin-close
→ Match created/confirmed + tutee Post → MATCHED; PAID → existing **commission** PAID path
(which already sets the match COMPLETED). The "pay after first session" is a real-world step; the
system models it as commission PENDING → PAID.

### 3.1 The atomic "assign" transition (admin closes the job)

**Trigger:** admin confirms the assignment for (tutee Post, chosen tutor) — extend
the create match operation (inject Job Application Repository); the chosen tutor comes from
the parent's selection/interview request.
**New atomic side-effects (all in the one a database transaction):**
1. Winning application (the chosen tutor's app for this job) → MATCHED. Link match source Application.
2. tutee Post status → **MATCHED** (drops out of public browse + blocks new applies).
3. **All other APPLIED/SHORTLISTED applications for this tutee Post → SUPERSEDED.**
4. Assignment record becomes visible on both profiles (projection over the new Match).
5. On match CANCELLED (before completion): reopen — tutee Post MATCHED → ACTIVE, superseded apps →
   back to APPLIED (**Q1 resolved: reopen**).

**Non-functional requirements (Boss/the technical lead conditions — binding):**
- **Set-based supersede:** one update records job applications SET status is SUPERSEDED when tutee post id=?
  AND status IN ('APPLIED','SHORTLISTED') — **never a read-loop** (N+1 + lock fan-out at scale).
  Requires an index on tutee post id, status.
- **Concurrency:** optimistic-version check on the Tutee Post so two admins can't double-assign the
  same job (second commit fails → 409, not a silent double-match).
- **Notifications after-commit + failure-isolated:** the supersede/winner/parent notification fan-out
  runs *after* the tx commits and a notification failure must **not** roll back the assignment
  (in-tx ordinary notification rows with recorded per-recipient isolation; outbox intent in-tx and external dispatch afterward, per the current messaging playbook).

### 3.2 Parent actions feeding the admin queue

- request-interview (exists: M6) — already a parent→admin signal.
- **choose-tutor (NEW)** — parent picks an applicant without an interview; creates a **selection
  request** the admin actions. (backend: new endpoint submit operation for tutee posts id select with
  the tutor-account identifier; surfaces in an admin queue.)

> **Selection request = a NEW tracked request object** (distinct from the Match it may lead to),
> **mutable until an admin confirms** — a parent may change their chosen tutor before the admin acts.
> Its storage + lifecycle (created → admin-actioned → consumed/superseded/expired) is defined in the
> **backend spec**; this design fixes only that it exists, is owner-scoped, and never itself creates a
> match (AC-04 admin-only matching holds).

Guard: apply endpoint must reject applies to a non-ACTIVE job (already returns
TUTEE POST NOT ACTIVE 422 — verified; once the job is MATCHED this naturally blocks new applies).
**The only remaining apply guard is "job must be ACTIVE."** The TUTOR NO ACTIVE POST listing
requirement is **removed** (free market, §0) — note this retires regression test
Guard Negatives IT apply Without Active Listing is403, which must be deleted/replaced when this ships.

## 4. End-to-end journeys (target, per role)

### Tutor (supply — Wei)
1. Browse jobs (/requests) → **job detail** → **Apply / Express interest** (optional pitch +
   *optionally* attach one of their listings). **No listing required** (free market). → app APPLIED.
2. **My Applications**: every job applied to + live status. **Display vocabulary mapped from the enum**
   (Jamie): APPLIED→"Applied", SHORTLISTED→"Shortlisted", interview→"Interview scheduled",
   MATCHED→"Assigned to you", SUPERSEDED→"No longer available", REJECTED→"Not selected",
   WITHDRAWN→"Withdrawn". (No "PENDING"/"ACCEPTED" — those are not enum values.) The
   "No longer available" copy is reassuring closure, not a cold grey-out (Wei).
3. **My Interviews**: interview sessions requested by parents (date, mode, job).
4. **My Profile** shows **Assignments** (jobs won) — builds reputation.

### Parent (demand — Priya)
1. Post a job → admin approves → ACTIVE.
2. **My Job → Applicants**: list of tutors who applied (personal information-safe card sourced from the tutor
   **profile**: handle, subjects, rate, area, YOE, **PT/FT**, rating — **no real name/contact
   pre-assignment**). Actions: **Request Interview** *or* **Choose this tutor**.
3. Parent's request goes to admin → admin liaises with the tutor → both confirm → admin closes the
   job (§3) → other applicants auto-notified "position filled" (SUPERSEDED).
4. After the first session, parent may pay for lessons; **only agency receipt of tutor commission** permits admin to mark commission **PAID**.
5. **Browse Tutors** (profile-based — any tutor with a completed profile, listing optional) → tutor
   detail → that tutor's profile + any optional listings.

### Admin (ops)
Dashboards (largely exist; this clarifies/links them):
- **Approval** — post review queue (approve/reject tutor + tutee posts).
- **Interviews & Selections** — interview requests **+ the new parent "choose-tutor" selection
  requests**; admin liaises, then confirms.
- **Matching / Assignment** — on confirmation, **close the job** (= assign, triggering §3.1);
  activate; mark **PAID** after agency receipt of tutor commission (commission PENDING→PAID, match COMPLETED);
  cancel/reopen. Surfaces job status + applicant pool.

## 5. Tutor profile as canonical identity (free-market) + PART TIME/FULL TIME — MUST

Because tutors apply without a listing (§0), the **profile** carries the public-facing identity. This
is a **hard schema MUST** (confirmed: tutor profiles has none of these columns today):
- New enum part-time or full-time tutoring commitment — required **dropdown at profile setup**.
- **Add to tutor profiles:** commitment, hourly rate min sgd, hourly rate max sgd,
  location planning area, location region (rate + area today live **only on Tutor Post**).
- **Identity model = profile-canonical + listing-overrides-when-attached:** the applicant card / browse
  reads the **profile** rate/area; **if** an application attaches a tutor Post Id, that listing's
  rate/area **overrides** for that card. One source of truth per context — **no sync job**.
- **Backfill migration:** populate each existing tutor's profile rate/area from their most-recent
  Tutor Post; tutors with no prior post get null until they edit their profile (acceptable — they're
  new). Changeset **+ backfill ship in the same pull request** (schema-coupling rule).
- Liquibase also adds match source application id (nullable FK — admin may assign a tutor who never
  formally applied).
- Surfaced on the **public tutor card**, **showcase**, and the **parent applicant card** (personal information-free).

## 5a. personal information contract (Casey/Priya — binding)

- **Parent applicant view is a DISTINCT personal information-free data transfer object** (not a reuse of Admin Application Response),
  with its own **contract test** (mirrors the existing showcase personal information-free data transfer object test): handle, subjects,
  levels, rate, **coarse** area, YOE, PT/FT, rating — **NO real name, email, phone, exact location**.
- **Answer to Priya:** the child's name and the parent's phone are **hidden from applicants until the
  parent commits** = **YES**. Contact is shared only after assignment (counterparty rule).
- **Owner-authorization (IDOR):** the applicant-list and choose-tutor endpoints are scoped to the
  job's owner — 403 if a parent queries/selects on a job that isn't theirs.
- **Assignment record on the public tutor profile** shows **subject / level / period / status only —
  never the counterparty's identity** (no parent or child name).

## 6. Tutor card / showcase redesign (folds in bug ①②)

- **Hide the real/display name** on public cards (personal information; founder). Lead with **subjects → levels**.
- Show: **Rate**, **Area** (coarse), **YOE**, **PT/FT commitment**, rating.
- Fix the showcase "undefined" rate: Showcase Tutor Card reads rate Min Sgd but the data transfer object sends
  hourly Rate Min Sgd — align the frontend type + card (bug ①).

## 7. Bug fixes folded into this delivery

| # | Bug | Fix in this delivery |
|---|---|---|
| ① | showcase rate "undefined" | frontend: Showcase Tutor type + card → hourly Rate Min Sgd/Max |
| ② | reg-test data polluting browse | clean reseed (no test run after); long-term: reg-band purge job |
| ③ | reset/admin password "only old works" | **real dev email via Mailhog** (founder choice) so the one-time passcode outbox actually delivers; backend reset logic already correct |
| ④ | signup link "gone" | frontend: add **Register** to desktop nav |
| — | tutor card name/personal information + PT/FT | §5 + §6 above |

## 8. Open questions (for the product requirements document + team review)

- **Q1** On match cancel, do superseded applications reopen to APPLIED, or stay superseded (parent
  re-opens manually)? *Recommend: reopen job to ACTIVE; superseded → APPLIED.*
- **Q2** RESOLVED (founder): parent **requests interview or chooses** a tutor → admin receives,
  liaises with tutor → **both confirm → admin closes job** → agency receives tutor commission → admin
  marks **PAID**. Browse Tutors is **profile-based** (listing optional). See §3.
- **Q3** RESOLVED: tutor profiles gets rate/area + commitment columns (§5), model =
  profile-canonical + listing-overrides, with backfill. (Profile had none of these today.)
- **Q4** Notifications (carry to product requirements document): templates for application superseded, application matched,
  parent assignment confirmed. (M9 — define template IDs in backend spec.)
- **Q5** RESOLVED: **WON'T v1** — no explicit parent "decline applicant"; decline is implicit via
  SUPERSEDED at assign time. (REJECTED enum value stays but unused by the parent flow in v1.)

## 9. Deliverables that follow this design

1. **product requirements document update** — §4 journeys (assignment lifecycle), §5 (M4/M6/M15 amendments + commitment),
   §7 data model (Application Status+SUPERSEDED, post MATCHED, commitment, match source Application),
   §8 permissions (applicant view, assign action).
2. **backend spec** — Job Application Status SUPERSEDED, Tutor Commitment enum + Liquibase
   (tutor profiles commitment, match source application id), Admin Match Service create Match
   side-effects (winner→MATCHED / others→SUPERSEDED / job→MATCHED / reopen-on-cancel), apply-guard,
   **NEW parent-facing applicant-list endpoint** (today it's admin-only — read operation for tutee posts id applicants, personal information-safe),
   profile assignments projection over Match, Mailhog dev mail (so the reset one-time passcode outbox delivers).
3. **frontend spec** — tutor card/showcase redesign + PT/FT dropdown, parent Applicants view + actions,
   tutor My-Applications statuses (SUPERSEDED), desktop Register link.
4. **UIUX spec** — applicant card (personal information-safe), assignment states, "position filled" empty/▢ states,
   commitment dropdown, card information hierarchy (subjects-first, no name).
5. **Regression** — branch tests for the new transitions: **positive free-market apply** (no-listing
   tutor CAN apply → 201, NEW), assign→winner MATCHED, others SUPERSEDED, job→MATCHED,
   apply-after-assigned→422, cancel→reopen, choose-tutor happy + IDOR-403, parent applicant-list
   owner-scope + personal information-shape contract test. **Retire** Guard Negatives IT apply Without Active Listing is403.
   (~8–10 new IT cases; concurrency double-assign is unit-test territory, not black-box.)

## 10. Acceptance criteria — observability + ops (Nova/Dana/Seb — binding)

- **Metrics:** applications superseded total (per assign), tutor selections pending (gauge =
  admin selection-request backlog), assignments total, commissions pending age (stuck-revenue).
- **Alert:** commissions pending age over threshold — the "agency received tutor commission / admin forgot to mark PAID"
  revenue leak (the pay-after-first-session step is a **manual admin action with no system deadline**;
  this alert is the only safety net — mandatory).
- **Admin view:** stale-PENDING commissions list (Seb) so stuck revenue is actionable.
- **Analytics events** (Blake): application superseded, tutor selected, job assigned,
  commission paid; the application submitted event carries whether a listing was attached (to
  measure free-market spam / applications-per-job).

## 11. Known scaling limit (Boss)

The "admin liaises with the tutor for every selection" step makes **admin throughput the ceiling** for
the core assignment flow. Accepted for v1 (founder's model); the product requirements document names it as a known limit, and
tutor selections pending makes the backlog visible. Mitigations (auto-confirm, tutor self-confirm)
are future work, not v1.

> Each deliverable runs the mandatory full-debate team review + Boss before it is DONE-DEV.
