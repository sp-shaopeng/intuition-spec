# Admin Identity & Linking — consolidated spec (INT-068/069/070/071 + INT-063)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** spec (needs /team-review → Boss before build) · **Date:** 2026-06-05
**Owners:** bff-specialist (data transfer object/query enrichment) + frontend-expert (shared identity-cell + page wiring) +
uiux-designer (table IA / clickability / headers) · security-expert sign-off on the new admin-only personal information (phone).
**Builds directly on INT-053** (int-053-059-admin-tutee-ux-wave md) which already established the
enrichment pattern: Match Repository find All For Admin List @Entity Graph + batched profile lookups in
Admin Match Service list Matches, and the clickable tutor↔tutee+job cell in Matching implementation reference.
**No schema/migration changes** for INT-068/069/070/071 (data already exists). INT-063 may add ONE read-only
admin list endpoint (no migration).

---

## 1. Problem — the admin-table-identity cluster (one recurring defect, five surfaces)

A founder walkthrough (2026-06-05) found the SAME defect on every admin table that wasn't touched by INT-053:
the row shows **codes / listing-titles / static names**, not **who** (tutor name + tutee/parent name) and
**which job**, and the identities aren't clickable. With seed data full of identical "Reg Parent" / "Reg Tutor"
rows, the tables are operationally unusable (you can't tell rows apart or chase a payment).

| Issue | Surface | Symptom (file:line) | Lane |
|---|---|---|---|
| **INT-068** | Commission Ledger Commission implementation reference | Rows render the subject pair only — the tutor’s subject / the tutee’s subject (Commission implementation reference) and the tutor's subject and the tutee's subject (259); table headers are Match,Subjects,… (217). No tutor/tutee/job. | **frontend-only** |
| **INT-069** | Interviews & Selections queue Interview Queue implementation reference | "Tutor Listing" column shows the listing **title** the interview’s tutor-listing title (297,300), not the tutor's NAME; "Job" is a bare label-less View job link (309); no tutor/tutee NAME columns. | **frontend-only** |
| **INT-070** | Job Applicants Job Applicants implementation reference | Tutor cell is static text the applicant’s tutor display name (121), **not a link**; no differentiating fields (columns are only Tutor/Status/Applied, 114-135); header just says title is Applicants (142) with no job context. **Plus an N+1** (see §2). | **frontend + backend** |
| **INT-071** | Users moderation Users implementation reference | Columns are Name/Email/Role/Status/actions (211-247); Name is static the user’s full name (213); **no phone**; rows indistinguishable. Admin User List Item Response has no phone (Admin User List Item Response implementation reference). | **frontend + backend** |
| **INT-063** | Admin close-job discoverability | The close control (submit operation for admin posts id close, Admin Post Controller implementation reference) lives ONLY on the per-job applicants page (Job Applicants implementation reference "Close job"). No admin list of ACTIVE jobs exists — only read operation for admin posts pending (Admin Post Controller implementation reference). | **frontend + backend** |

### Current state in code — what INT-053 already gave us (reuse, don't rebuild)
- **The Commission Ledger is fed by the SAME endpoint INT-053 enriched.** Commission implementation reference calls
  application programming interface ROUTES admin.matches (admin matches interface operation, api Routes implementation reference) and binds Admin Match Item. That
  payload **already carries** tutor Display Name, tutor Public Handle, tutor Post Id, tutee Name, tutee Post Id,
  job Title, tutor User Id, tutee User Id — added to Admin Match Item Response implementation reference by INT-053 and batched
  with no N+1 in Admin Match Service list Matches (402-460). **⇒ INT-068 is frontend-only**: Commission implementation reference just
  isn't rendering the identity fields it already receives.
- **The interview data transfer object already carries identity too.** Admin Interview Response implementation reference has tutee Post Id,
  tutor User Id, tutor Display Name, tutor Public Handle, tutee Name, plus tutor Post Title. **⇒ INT-069 is
  also frontend-only**: the queue renders tutor Post Title where it should render tutor Display Name, and a bare
  View job where it should render the job title/subject·level. (Verify at build that the LIST factory
  the from operation (47) is the one used by the queue and that the service passes the
  resolved names; if the service stubs them, that's the one backend touch for INT-069 — confirm at build.)
- **The reference clickable cell already exists** at Matching implementation reference: tutor →
  /admin/review/:tutor Post Id (null-safe: plain text when tutor Post Id is null, 393-399), tutee →
  /admin/review/:tutee Post Id (401), job → /admin/posts/:tutee Post Id/applicants (411), with the
  subject pair as a secondary line (405-407). **This is the component to extract.**

---

## 2. INT-070 is the only NEW backend N+1 — fix it with the existing batch pattern

Job Application Service list Applicants For Job (289-304) maps each applicant via
the from operation (300), and
resolve Tutor Display Name (445-450) does **one the find by user id operation per applicant** —
a classic N+1 across the applicant page.

The **sibling owner-facing path already does it right**: list Applicant Cards For Owner (313-356) collects the
distinct tutor ids (331-332) and batch-loads once with
the find all by user id in operation (336) + a batched rating aggregate (339-343). INT-070
must make list Applicants For Job use the **same find All By User Id In batch** (one profile query for the page),
exactly mirroring 331-345 and the Admin Match Service list Matches batch (423-446).

This is the **single shared backend enrichment approach** the cluster needs: a flat to-one @Entity Graph/JOIN on the
list query for the entity edges, then **batch the profile fields by the distinct user-id set** with
find All By User Id In (display name + handle + rate/area/YOE/rating for differentiation). Never a per-row profile
lookup.

---

## 3. Design

### 3.1 Shared backend data transfer object-enrichment approach (one pattern, applied per surface)

**Rule (cluster-wide):** an admin list endpoint that shows identities resolves entity edges via a to-one
@Entity Graph/JOIN-FETCH on the repository query (Postgres-side pagination preserved — all edges are
@Many To One), then batch-loads the off-entity profile fields **once per page** by the distinct user-id set
(find All By User Id In). Return a record projection, never an entity. This is exactly
Match Repository find All For Admin List (70-72) + Admin Match Service list Matches (423-446) — already in code.

Per surface:

- **INT-068 (Commission):** **no backend change.** Admin Match Item Response already carries every identity field and
  is already batched (no N+1). Verified: Commission implementation reference reads application programming interface ROUTES admin.matches.
- **INT-069 (Interviews):** **no data transfer object change** — Admin Interview Response already carries
  tutor name/handle/tutee Name/tutee Post Id/job Title-equivalents. **At build, confirm** the admin interview
  list service actually populates tutor Display Name/tutee Name (the the from operation factory takes them as args,
  Admin Interview Response implementation reference). If those are passed null today, batch them per page with
  find All By User Id In (one tutor-profile query + one tutee-profile query) the same way list Matches does —
  do NOT add a per-row resolve. Add job Subject/job Level to the interview list data transfer object **only if** the queue
  needs a humanized job label and tutor Post Title/job title is null (cf. INT-061); prefer reusing
  tutor Post Subjects/tutor Post Levels (already present, 23-24) and the tutee post's subject/level.
- **INT-070 (Job Applicants):** **backend change.** (a) Fix the N+1 in list Applicants For Job (§2). (b) Extend
  Admin Application Response with the **differentiating fields** the page needs to tell identical-named tutors
  apart: tutor Public Handle, hourly Rate Min Sgd/hourly Rate Max Sgd, location Region (or planning area),
  years Experience, average Rating/review Count — all already available from the batched Tutor Profile +
  the rating aggregate (list Applicant Cards For Owner shows the exact sourcing, listing-override
  included). Keep tutor Id + tutor Post Id for the link. (c) Surface the **job's title + subject·level** for
  the page header — either add job Title/job Subject/job Level to a small wrapper on the applicants response
  (the service already loads the Tutee Post via exists By Id; switch to find By Id to read
  the get subject code operation/the get level code operation/title as list Applicant Cards For Owner does), or have the frontend fetch
  read operation for admin posts job Id for the header. **Decision (lean):** return the job label fields on the applicants
  payload so the header needs no second round-trip (under-fetching is as bad as over-fetching — frontend-facing service owns the
  aggregation).
- **INT-071 (Users):** **backend change — one line of projection.** Add phone to the
  Admin User List Row projection (User Repository implementation reference) and the
  administration user lookup, taking the phone number from the account record, and add phone to Admin User List Item Response (26-34) + the mapper
  (Admin Moderation Service list Users). **Still a single flat query — no N+1.** Optionally add
  created At for context (same projection, one column) — keep Mo S Co W-Could.

**personal information rule (security sign-off — §3.4):** the new phone field is returned **only** from read operation for admin users
(admin-only, already exposes email). It is added to NO public/tutee/tutor data transfer object. The applicant differentiators
(handle/rate/area/YOE/rating) are non-personal information and admin-appropriate.

### 3.2 Shared frontend clickable identity-cell component

Extract the Matching implementation reference markup into ONE reusable component, e g.
Identity Cell implementation reference (admin-scoped; not a generic ui/ primitive — it encodes admin routing).
**Per the react-refresh lesson, it exports the component only**; any label constants live in constants/.

Proposed shape (discriminated, null-safe, keyboard/a11y correct):


Implementation example omitted. This public edition retains the written requirements in this section.


- **Routing (existing routes only — no new views, mirrors INT-053):**
  - tutor → /admin/review/:post Id when post Id present, else **plain text** (free-market applicants /
    pre-0033 matches have a null tutor Post Id — App implementation reference route is /admin/review/:id).
  - tutee → /admin/review/:tutee Post Id.
  - job → /admin/posts/:tutee Post Id/applicants (App implementation reference).
- **Display fallback chain** (reuse Matching implementation reference): name or handle or user Id for the label so a
  missing profile never renders blank.
- **job Title null (INT-061 carry-in):** when job Title is null/blank, render a derived label
  the readable subject and level (humanized via utils/format) instead of an empty link or "View job".
- **a11y:** link text is the human label (not "View job"); use  with hover:underline +
  focus-visible ring (match Users implementation reference); subject·level as a secondary text-caption text-sand line
  (recognition-over-recall — Nielsen). Plain-text fallback uses a , never a dead .

**Consumers (all four pages render the same cell):**
- Commission implementation reference — replace the Match/Subjects columns (217, 255-259) with **Tutor**, **Tutee**,
  **Job** columns using Identity Link; keep a Subject·Level secondary line. (The activating-row branch
  251-301 must mirror the new columns or the row layout breaks — handle in the same edit.)
- Interview Queue implementation reference — render tutor Display Name (not tutor Post Title) as the **Tutor** column via
  Identity Link kind is tutor; keep tutor Post Title/subjects as a secondary line; add a **Tutee** column
  (tutee Name, already shown at 288 linking to the interview detail — keep that as a "Details" affordance);
  replace the bare View job (309) with Identity Link kind is job showing the job title/subject·level.
- Job Applicants implementation reference — make the **Tutor** cell an Identity Link kind is tutor to
  /admin/review/:tutor Post Id (null-safe text when null); add differentiating columns (handle, rate band,
  region/area, YOE, rating); feed the **job title/subject·level** into the Page Header title/subtitle
  (142) from the enriched payload.
- Users implementation reference — add a **Phone** column; make the **Name** an Identity Link to the user's detail (tutor →
  /admin/review/:tutor Post Id IF a listing exists; otherwise the name stays text — a tutee/listing-less tutor
  has no admin detail route today, see open decision OD-2); keep block/unblock + reason dropdown (230-247).

### 3.3 Page headers carry the job context (INT-069/070)
The Job Applicants header (Job Applicants implementation reference) must name the job: the job title used as the label (or the derived
the readable subject and level), subtitle = level/area, sourced from the enriched applicants payload (§3.1). The
Interviews queue and Commission ledger are multi-row tables, so per-row identity cells (§3.2) carry the context
there rather than a single header.

### 3.4 personal information rule (security-expert sign-off)
- **Phone is admin-only.** Added solely to read operation for admin users (already an admin-gated surface exposing
  email — Admin User List Item Response doc, 18). Not added to any public/showcase data transfer object (M14a/M14b stay
  personal information-free), not to tutee/tutor-facing applicant cards (Applicant Card Response stays personal information-free per
  list Applicant Cards For Owner contract, 306-312). Security must confirm the new field appears on no
  non-admin endpoint (grep the data transfer object usages).
- **Names are already admin-visible** (INT-053 closed this: Interview Queue implementation reference, Matching implementation reference); the
  cluster adds NO new name exposure. The applicant differentiators (handle/rate/area/YOE/rating) are non-personal information.

### 3.5 INT-063 — admin close-job discoverable surface
The close right exists but is buried. Add discoverability:
- **backend (one read-only endpoint, no migration):** read operation for admin posts active (or ?status is ACTIVE on a new
  admin posts list) returning paged ACTIVE tutee posts with the SAME identity enrichment (job title/subject·level
  + owner name) — there is currently **only** /admin/posts/pending (Admin Post Controller implementation reference), no ACTIVE
  list. Reuse the §3.1 batch pattern.
- **frontend:** an admin **Active Jobs** view (or section on the Matching surface / an admin posts list) listing live
  jobs with a **Close job** action inline (reusing the existing submit operation for admin posts id close,
  api Routes close Post) — so an admin finds and closes a job without drilling into applicants. Each job row
  uses the §3.2 Identity Link kind is job. **Mo S Co W: Should** (smaller than the identity cluster; can ship in a
  later wave if the cluster is the priority).

---

## 4. Acceptance criteria

**INT-068 (frontend-only):** every Commission row shows the real **Tutor**, **Tutee**, and **Job** (not subject
codes), each clickable to its existing detail route; subject·level remains as a secondary line; the
activating-row branch matches the new columns; no new query (endpoint unchanged); gates green.

**INT-069 (frontend-only, confirm backend name population):** each queue row legibly shows tutor **name** (not listing
title), tutee name, and the **job** (title or subject·level — not "View job"), each clickable; rows are
distinguishable even with identical seed names; a query-count assertion proves no N+1 was introduced.

**INT-070 (frontend + backend):** the applicants list fires a **bounded** query count (one batched profile query per page,
proven by an IT query-count assertion — the N+1 at 300 is gone); each applicant row identifies a **distinct**
tutor (handle/rate/area/YOE/rating differentiators), the tutor is **clickable** to their listing detail (null-safe
text when no listing), and the page **header names the job** (title/subject·level).

**INT-071 (frontend + backend):** read operation for admin users returns phone (admin-only — confirmed on no other endpoint); the frontend
Users list shows Name (clickable where a detail route exists), Email, **Phone**, Role, Status; rows are denser /
distinguishable; the query stays a single flat projection (no N+1); block/unblock unchanged.

**INT-063 (frontend + backend):** an admin can find a list of ACTIVE jobs and close one inline without opening its applicants
page; the close reuses the existing endpoint; the list has no N+1.

**Cross-cutting:** one shared Identity Link cell is used by all four tables (and the active-jobs row); frontend gate
**≥97.5% line / ≥88% branch**; backend gate **≥97.5% line AND branch**; the documented operational check + the documented operational check (tsc) +
test:ci green; the documented operational check + the documented operational check green.

---

## 5. Test notes

- **backend no-N+1 (the load-bearing tests):**
  - INT-070: a @Data Jpa Test/IT on list Applicants For Job with N applicants asserts a **constant** query count
    (mirror the INT-053 Match Repository Enrichment Test approach; note: @Data Jpa Test repo tests show
    Tests run: 0 LOCALLY by Docker Support policy and run in continuous integration — that is expected, not hollow, per
    SHARED-LESSONS 2026-06-04). Assert the page query + ONE find All By User Id In + ONE rating-aggregate, no per-row
    profile query.
  - INT-069: if the service is touched to batch names, the same constant-query assertion on the interview list.
  - INT-071: assert the user list is ONE query and phone is populated; a unit test asserts phone is absent
    from every non-admin data transfer object (grep-style guard or a serialization test).
- **frontend component tests (Identity Link):** tutor with post Id → renders a  to /admin/review/:post Id;
  tutor with null post Id → plain text, **no anchor** (assert the query by role operation is null); job with null
  job Title → renders the derived the readable subject and level label; fallback chain name or handle or user Id; focus-ring
  + hover:underline present; keyboard-reachable.
- **frontend page tests:** Commission renders Tutor/Tutee/Job columns and the links resolve (assert href);
  Job Applicants header shows the job title and the tutor cell is a link; Users shows the Phone column and a
  clickable name where a route exists. Reuse the modal-close enumeration lesson (assert open, not text
  removal) for any dialogs touched.
- **personal information guard:** a test/grep asserting phone is serialized only by the admin users endpoint.

---

## 6. Mo S Co W / sequencing

- **Wave A (Must) — the frontend-only quick wins (no backend):** INT-068 + INT-069 (data already on the wire) + extract the
  shared Identity Link. Highest value-per-effort; unblocks the rest by landing the shared component first.
- **Wave B (Must) — backend + frontend:** INT-070 (N+1 fix + applicant differentiators + job header) and INT-071 (phone
  projection + clickable name). Each is a small, additive backend change reusing the batch pattern.
- **Wave C (Should):** INT-063 (admin active-jobs list endpoint + view with inline close).

Per the holistic-walk lesson (SHARED-LESSONS 2026-06-05): after the waves, do a final admin-role end-to-end walk
(Commission → Interviews → Applicants → Users → Active jobs) to confirm the identity treatment is consistent
across all five surfaces, not polished on some and left on others.

---

## 7. Open decisions

- **OD-1 (job Title null — INT-061):** confirm at build whether real/seeded tutee posts carry a free-text title.
  If consistently null, the derived the readable subject and level label becomes the primary job label everywhere (not a
  fallback). Lean: derive in utils/format so all four surfaces share one humanizer.
- **OD-2 (tutor with no listing):** a free-market applicant / a tutee has **no admin tutor-post detail route**
  (only /admin/review/:post Id exists, App implementation reference). For these the identity stays **plain text** (no dead
  link). Revisit only if admins ask for a tutor-centric admin profile view (deferred, same as INT-053's open
  decision).
- **OD-3 (INT-069 name population):** confirm whether the admin interview list service already passes resolved
  tutor/tutee names into the from operation or stubs them — decides whether INT-069 is truly
  frontend-only or needs the one-query batch on the backend.
- **OD-4 (INT-063 scope):** standalone "Active Jobs" admin page vs a section on the existing Matching surface vs
  a filter on a new admin posts list — uiux call at build; the backend list endpoint is the same either way.
- **OD-5 (Users name link target for tutees):** tutees have no admin detail page; either leave the tutee name as
  text or introduce a minimal admin tutee-detail (deferred — text for now, consistent with OD-2).

---

→ team-review → Boss → build.
