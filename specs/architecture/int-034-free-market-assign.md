# INT-034 — Free-Market Assign (key on tutor user Id, not tutor Post Id) — Backend Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Issue:** INT-034 (Sev-1) · **Layer:** Backend (Spring Boot 3.3 / Java 21)
**Status:** Draft for /team-review → Boss verdict before build
**Owners:** software-engineer (lead) · dba · bff-specialist · **Date:** 2026-06-03
**Source of truth:** the service code (founder decree). Every change cites real file:line against
the live backend. Liquibase for ALL schema (no raw database structure definitions). No magic strings.

> **Conventions (binding, from CLAUDE md):** Java 21; Liquibase for all schema; no magic strings (backend
> application programming interface paths → the application component Routes, mirroring frontend application programming interface ROUTES; enums → typed
> enums/constants); coverage gate **≥97.5% line AND branch**; nested error envelope
> a structured error containing its error code via App Exception factories; **soft-delete only** (status/flag, never a
> hard row delete); **regression ITs hit the LIVE stack** at localhost (rebuild+deploy+reseed
> before the documented operational check).

> **Changeset number:** **0033** is assigned to this issue. **0032 is reserved for M19 OAuth** — do
> not reuse. Current changelog head is 0031-allow-post-soft-delete-close implementation reference (CONFIRMED by
> ls db/changelog/changes/). 0033 follows 0031 (0032 is held for M19, applied independently).

---

## 1. Problem

M18 made the **tutor profile the canonical identity**: a tutor applies to a job (or is chosen by a
parent) **without any tutor listing**. Job Application tutor Post is nullable (Job Application implementation reference)
precisely to support this listing-free apply. This is the **default** free-market flow.

But the admin assign endpoint is keyed on a **tutor Post Id** (a legacy listing), and that key is
load-bearing all the way down:

1. **Contract** — Create Match Request (Create Match Request implementation reference) requires
   @Not no value tutor Post Id. A free-market applicant has no tutor Post Id to send.
2. **Service** — the create match operation (Admin Match Service implementation reference)
   **first** loads the Tutor Post by id (404 TUTOR POST NOT FOUND if missing) and **requires it
   ACTIVE** (422 TUTOR POST NOT ACTIVE, lines 134-138). With no listing there is nothing to load.
3. **Entity / schema** — Match tutor Post is the join column operation (Match implementation reference)
   backed by the tutor posts operation
   (0008-create-matches implementation reference). A match **cannot be persisted** without a
   tutor listing. There is **no column that records the matched tutor's user Id** — the tutor's
   identity is only reachable via the get tutor post operation.
4. **Downstream readers all dereference the get tutor post operation** to resolve the tutor:
   - Contact Reveal Service reveal Contact (Contact Reveal Service implementation reference) —
     the get tutor post operation. **missing-value failure on a free-market match.**
   - Match Dashboard Service list Tutee Matches (Match Dashboard Service implementation reference)
     — tutor identity, subject, and title all come from the get tutor post operation.
   - Match Dashboard Service list Tutor Matches query
     find By Tutor Post Tutor Id Order By Matched At Desc (Match Repository implementation reference) — a
     free-market match has **no tutor Post**, so it would **never appear in the tutor's dashboard**.
   - Admin Match Service mappers to Response / to Admin Match Item Response / cancel Match
     (Admin Match Service implementation reference) dereference the get tutor post operation for subject/ids.

**Net effect:** the headline M18 flow — *parent chooses a free-market applicant → admin assigns* — is
**impossible**, not merely mis-disabled. Tutor Selection Service list Open Requests already concedes
this: it surfaces a **null tutor Post Id** for listing-less applicants
(Tutor Selection Service implementation reference, data transfer object comment Admin Selection Request Response implementation reference).

**Current state:** frontend shipped a **stopgap** — the assign button is disabled and explains "applied
without a listing" when tutor Post Id equals no value (no more silent no-op). The **real fix is backend** and
is the subject of this spec.

---

## 2. Proposed design

**Re-key the match on the tutor user Id, with the optional tutor Post as an enrichment, not a
requirement.** A Job Application already carries the authoritative tutor (tutor → tutor id,
Job Application implementation reference) and so does a Tutor Selection Request (tutor User Id,
tutor User Id/job Application Id resolved in Tutor Selection Service). The tutor's public identity is
the **canonical Tutor Profile** (display Name, Handle, rate, area — Tutor Profile implementation reference),
which exists for every tutor regardless of listings.

### 2.1 Core changes

1. **New nullable column matches tutor user id** (changeset 0033) + Match tutor User (User,
   nullable). It is the **authoritative matched-tutor identity** going forward. tutor post id
   becomes **nullable** (the listing is now optional enrichment).
2. **create Match accepts a tutor user Id OR a tutor Post Id** (exactly one required). New overload
   resolves the tutor user Id from whichever is supplied, loads the Tutor Post **only if** a
   tutor Post Id is given, and persists tutor user id **always** (tutor post id when known).
3. **All readers resolve the tutor via tutor user id first**, falling back to
   the get tutor operation only for legacy rows where tutor user id is null (backfill closes this —
   see §4). Subject/title that previously came from tutor Post fall back to the Tutor Profile /
   selection context when no listing exists.
4. **Update existing matches:** find the tutor account that owns each match’s linked listing and store that account identifier directly on the match. Migration 0033 must be safe to repeat.
5. **tutor Post Id path kept working unchanged** for the 2-panel legacy admin match.

### 2.2 The single atomic assign — unchanged guarantees

The existing atomic transition (Admin Match Service create Match, lines 117-227) is preserved exactly:
winner app → MATCHED, all other open apps → SUPERSEDED (single set-based update records, Job Application Repository supersede Other Applications), job ACTIVE → MATCHED (bumps @Version →
optimistic-lock 409 JOB ALREADY ASSIGNED), match persisted PENDING, selection requests consumed,
after-commit failure-isolated notifications. **The only change is how the tutor is identified** and
that tutor Post may be null.

### 2.3 Caller contract (which key wins)

the create match operation
already exists (Admin Match Service implementation reference). Today it derives the tutor as
the get tutor operation (line 165-166) — i e. it
**already prefers an explicit winning tutor user Id**. The blocker is the **mandatory Tutor Post load
at the top** (lines 124-138) and the **required tutor post id** at persist. We make tutor Post Id
optional and tutor user id the persisted key.

Resolution rule (exactly one tutor source required, else 400 MATCH TARGET REQUIRED):

| Caller | Sends | Tutor user Id resolved from | tutor Post |
|---|---|---|---|
| Free-market (selection-driven) | winning Tutor Id (+ optional source Selection Request Id) | winning Tutor Id | resolved from winner app if it carried a listing, else **null** |
| Legacy 2-panel | tutor Post Id | the get tutor operation | the loaded listing |

If winning Tutor Id is present, the Tutor Post load is **skipped** unless a tutor Post Id is also
sent. The optional tutor Post link on the match is taken from the **winning application's**
tutor Post (the get tutor post operation), which may be null — that is the whole point.

---

## 3. Schema changes (changeset 0033)

New file: 0033-match-tutor-user-id implementation reference, registered after 0031 in
db changelog-master implementation reference. (0032 reserved for M19 — leave the gap.)


Implementation example omitted. This public edition retains the written requirements in this section.


**DBA notes.**
- ON DELETE RESTRICT matches the existing created by/post FKs (0008:32-35) — users are
  **soft-deleted**, never hard-deleted (founder directive), so RESTRICT never actually fires; it is a
  safety net consistent with the rest of the schema.
- The **duplicate-pair guard** uq matches pair unique tutor post id, tutee post id (0008:37)
  is now **ineffective for free-market matches** (both rows would have tutor post id no value, and
  Postgres treats NULLs as distinct, so the unique index does not collide). **Decision:** the
  duplicate guard moves into the service (a partial unique index on tutee post id would over-block
  legitimate reopen/re-assign cycles). The job ACTIVE → MATCHED transition under @Version
  (tutee posts version, 0028c) is the real concurrency guard and already returns 409 — see §2.2.
  We **add** an application-level check (existing lines 157-162) extended to also reject an open match
  for the same tutee Post Id, tutor User Id pair. No new database unique index (keeps reopen working).

---

## 4. Entity / data transfer object / application programming interface contract changes

### 4.1 Entity — Match

Match implementation reference: add a nullable tutor User, make tutor Post nullable.


Implementation example omitted. This public edition retains the written requirements in this section.


Add a small helper to centralise the resolve-fallback (used by every reader):


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.2 Service — Admin Match Service create Match

- **Skip the Tutor Post load when tutor Post Id equals no value** (lines 124-138 become conditional). When a
  tutor Post Id IS given, keep the 404/422 ACTIVE checks unchanged (legacy path).
- Resolve winning Tutor User Id as today (line 165-166) but tolerate a null tutor Post:
  the get tutor operation — tutor Post is only
  dereferenced when it was loaded.
- If neither tutor Post Id nor winning Tutor Id is provided → 400 MATCH TARGET REQUIRED.
- Set the get reference by id operation and
  the get tutor post operation (either may be
  null) in the builder (lines 195-203).
- Extend the duplicate guard (lines 157-162) to also block an open match for the same
  tutee Post Id, winning Tutor User Id via a new repo method (see §4.5).
- Mappers to Response (497), to Admin Match Item Response (504), and the cancel Match response build
  (440-489) must use the resolve tutor user id operation and null-safe subject extraction:
  the get subject codes operation.

### 4.3 Service — Contact Reveal Service

Contact Reveal Service implementation reference → resolve the tutor-account identifier directly from the match
(no more the get tutor post operation). Name already resolves via Tutor Profile (lines 80-88) — works
for listing-less tutors unchanged.

### 4.4 Service — Match Dashboard Service

- **Tutee view** (list Tutee Matches, lines 74/91/93/102): tutor user Id via the resolve tutor user id operation;
  display name still from Tutor Profile (lines 77-79) — keyed on user Id, works without a listing.
  tutor Subject/title fall back: when tutor Post equals no value, use the **tutee post's** subject
  (the get tutee post operation) for the subject and the Tutor Profile display Name for the
  card title (a listing-less tutor has no listing title).
- **Tutor view** (list Tutor Matches): replace the dashboard query
  find By Tutor Post Tutor Id Order By Matched At Desc (Match Repository implementation reference) with a new
  the find by tutor user id order by matched at desc operation keyed on the new column
  (index idx matches tutor user id). This is what makes a free-market match **appear in the tutor's
  dashboard at all**.

### 4.5 Repository — Match Repository

Add (keyed on the new column):


Implementation example omitted. This public edition retains the written requirements in this section.


The legacy find By Tutor Post Tutor Id Order By Matched At Desc (line 52) and
exists By Tutor Post Id And Tutee Post Id And Status In (line 37) are **retained** but no longer the dashboard
path; remove the legacy dashboard call once the new query is wired (keep the method only if another
caller uses it — confirm by grep before deleting, per coverage gate).

### 4.6 frontend-facing service / contract — request & response

**No magic strings: all paths stay in Api Routes** (Api Routes implementation reference) — **no new path is
needed.** The existing endpoints already carry the tutor identity the frontend has:

- submit operation for admin matches (Admin Match Controller implementation reference, path constant BASE PATH).
- Parent selection submit operation for tutee posts id select (Api Routes TUTEE POSTS read fields).
- Admin queue read operation for admin selection-requests (Api Routes ADMIN SELECTION REQUESTS).

**Create Match Request** (Create Match Request implementation reference): relax the contract so admin can
assign by tutor user Id. **One of** tutor Post Id / winning Tutor Id is required (cross-field
validation), tutee Post Id always required:


Implementation example omitted. This public edition retains the written requirements in this section.


Controller Admin Match Controller create Match (lines 66-75) calls the 5-arg overload:
create Match(request tutor Post Id operation, request tutee Post Id operation, principal user Id operation,
request winning Tutor Id operation, request source Selection Request Id operation).

**Selection-driven assign (preferred frontend path).** Rather than the frontend assembling a winning Tutor Id, the
cleaner contract is a dedicated **admin "confirm selection request"** action that already knows the
tutor: submit operation for admin selection-requests id assign (new constant
the administration operation to assign a tutor from a selection request). It reads the
Tutor Selection Request (which carries tutor User Id + tutee Post Id + job Application Id,
Tutor Selection Request implementation reference) and calls the same create Match overload with
the get tutor user id operation and the get id operation. This removes the
frontend's dependency on tutor Post Id entirely and is what un-disables the stopgapped button.
**Open decision OD-1** below weighs this vs. extending the existing submit operation for admin matches.

**Responses unchanged in shape.** Match Response, Admin Match Item Response,
Admin Selection Request Response, Tutee Match Response, Tutor Match Response keep their fields.
Admin Selection Request Response tutor Post Id stays nullable (correctly), and its data transfer object comment
(Admin Selection Request Response implementation reference) is updated: the assign button is **no longer disabled**
for a null tutor Post Id — assign keys on tutor User Id. frontend index implementation reference and application programming interface ROUTES mirror
the new optional fields / the new assign route.

---

## 5. Acceptance criteria

- **AC-1** A parent chooses a tutor who **applied without a listing** (tutor Post Id equals no value); admin
  confirms → a PENDING match is created, matches tutor user id set, matches tutor post id NULL.
  The previously-dead button now succeeds (201). *(This is the headline fix.)*
- **AC-2** The winning listing-less application → MATCHED; every other open application on the job →
  SUPERSEDED (single set-based update records); job ACTIVE → MATCHED. Counts logged + metered as today.
- **AC-3** Concurrency: two admins confirm the same job simultaneously → one 201, the other **409**
  JOB ALREADY ASSIGNED (optimistic lock on tutee posts version). No partial state.
- **AC-4** Legacy 2-panel assign by tutor Post Id still works: 404 TUTOR POST NOT FOUND,
  422 TUTOR POST NOT ACTIVE, 409 duplicate — **all unchanged**.
- **AC-5** Contact reveal on a free-market match returns the tutor's Tutor Profile display Name +
  plaintext phone/email (no missing-value failure; was the INT-016-adjacent 500 surface).
- **AC-6** A free-market match **appears in the tutor's "My Matches" dashboard** (list Tutor Matches)
  and the **tutee's dashboard** with the tutor's display name (no listing required).
- **AC-7** Cancel a free-market match → match CANCELLED, job reopens ACTIVE, winner + superseded
  apps reopen to APPLIED, pending commission CANCELLED. No 500 (INT-016 pre-detach capture must
  use the resolve tutor user id operation / null-safe subject, since tutor Post may be null).
- **AC-8** Sending neither tutor Post Id nor winning Tutor Id (or both) → **400** with the
  cross-field validation message. No partial work.
- **AC-9** Migration 0033 applied to the existing seeded database: every pre-existing match has
  tutor user id populated (backfill), tutor post id retained, required on tutor user id holds.
- **AC-10** Duplicate guard: an open (PENDING/ACTIVE) match already exists for job, tutor →
  409 DUPLICATE MATCH, regardless of whether the tutor has a listing.
- **AC-11** No magic strings: any new path is a constant in Api Routes (and mirrored in frontend
  application programming interface ROUTES); No Hardcoded Routes Arch Test passes. Spotless + ESLint clean.

---

## 6. Test plan

Coverage gate **≥97.5% line AND branch** must hold (every new branch — null tutor Post,
fallback resolve, both-targets-400 — needs a test). New + amended tests:

### 6.1 Unit / slice (Mockito + @Data Jpa Test)
- **Admin Match Service Test** (Admin Match Service Test implementation reference): add cases —
  (a) free-market assign by winning Tutor Id, tutor Post Id null → match persisted with tutor User
  set, tutor Post null, winner app MATCHED, others SUPERSEDED; (b) legacy tutor Post Id path
  unchanged; (c) neither target → 400; (d) duplicate job, tutor → 409; (e) cancel of a
  listing-less match (no missing-value failure, INT-016 pre-detach uses the resolve tutor user id operation).
- **Contact Reveal Service Test**: reveal on a tutor Post equals no value match → name from Tutor Profile,
  no missing-value failure.
- **Match Dashboard Service Test**: free-market match appears in both tutor and tutee views; subject
  falls back to tutee post subject when no listing.
- **Match Repository (@Data Jpa Test)**: find By Tutor User Id Order By Matched At Desc and
  exists By Tutee Post Id And Tutor User Id And Status In return the right rows; the named-PG-enum bind for
  status IN :statuses renders correctly (mirror the Job Application Repository pitfall,
  Job Application Repository implementation reference).
- **Create Match Request validation**: both-null and both-set → invalid; exactly-one → valid.

### 6.2 Controller (@Web Mvc Test / Mock Mvc)
- Admin Match Controller Test: POST with winning Tutor Id only → 201; both targets → 400; non-admin →
  403; (if OD-1 picks the dedicated route) Admin Selection Request Controller Test for
  the assignment action for the selected record.

### 6.3 Integration tests against the **live stack** (regression suite, Abstract Api IT → localhost)
> Per SHARED-LESSONS + MEMORY: regression ITs hit the running app, **not** Testcontainers. **Rebuild +
> redeploy + reseed** before the documented operational check, else seed-dependent ITs fail spuriously.

- **Admin Matching IT** (Admin Matching IT implementation reference): new end-to-end —
  seed a tutor with a Tutor Profile but **no Tutor Post** → tutor applies to a job (listing-less) →
  parent submit operation for tutee posts id select → admin assigns (selection-request assign / winning Tutor Id)
  → assert 201, job MATCHED, winner app MATCHED, contact reveal 200 for both parties, match visible in
  both dashboards.
- **Concurrency IT**: two parallel assigns on one job → exactly one 201, one 409.
- **Cancel IT**: cancel the free-market match → reopen verified (job ACTIVE, apps APPLIED), no 500.
- **Migration IT**: assert databasechangelog head includes 0033d, tutor user id required,
  tutor post id nullable, all existing matches backfilled.

---

## 7. Open decisions

- **OD-1 (bff-specialist) — endpoint shape.** Two options: (A) extend submit operation for admin matches
  with optional winning Tutor Id (smaller surface, frontend must hold the tutor user Id); (B) add
  submit operation for admin selection-requests id assign (server reads the selection request, frontend sends
  nothing but the id — cleanest, kills the frontend tutor Post Id dependency outright). **Lead recommends B**
  for the parent-driven flow and keeping A for the 2-panel path. Confirm at /team-review.
- **OD-2 (dba) — duplicate guard.** App-level tutee Post Id, tutor User Id open-match check (this
  spec) vs. a partial unique index when status IN PENDING,ACTIVE. A database index is stronger but
  must not block legitimate reopen→reassign; app-level + the @Version job lock is the safer default.
  Decide whether to also add a partial unique index for defense-in-depth.
- **OD-3 — drop legacy dashboard query.** Whether to remove
  find By Tutor Post Tutor Id Order By Matched At Desc (Match Repository implementation reference) entirely once the
  user Id-keyed query lands. Keep only if a non-dashboard caller exists (grep before removing — a dead
  method would fail the coverage gate).
- **OD-4 — matches tutor post id long term.** Now optional; do we keep it for analytics/audit
  (yes — soft-delete philosophy: retain history) or migrate it out later? Recommend **keep**.
- **OD-5 — selection-request job Application Id null but a listing-less app exists.** Confirm
  the find open application operation (Job Application Repository implementation reference) is the single
  resolver of the winning app at assign time so the winner is found even when the selection request
  was created before the tutor applied.

---

## 8. Impacted files (cited, for the implementer)

| File | Change |
|---|---|
| 0033-match-tutor-user-id implementation reference | **NEW** — column + FK + backfill + nullable flip + required + index |
| db changelog-master implementation reference | register 0033 (after 0031; leave 0032 gap for M19) |
| Match implementation reference | add tutor User (required FK), make tutor Post nullable, add the resolve tutor user id operation |
| Admin Match Service implementation reference | conditional Tutor Post load, persist tutor User, null-safe mappers, extended duplicate guard |
| Contact Reveal Service implementation reference | resolve via the resolve tutor user id operation |
| Match Dashboard Service implementation reference | user Id-keyed tutor identity + dashboard query + subject fallback |
| Match Repository implementation reference | find By Tutor User Id Order By Matched At Desc, exists By Tutee Post Id And Tutor User Id And Status In |
| Create Match Request implementation reference | optional tutor Post Id/winning Tutor Id (exactly-one), source Selection Request Id, @Assert yes |
| Admin Match Controller implementation reference | pass new fields to the 5-arg overload |
| Api Routes implementation reference | (OD-1=B) ADMIN SELECTION REQUESTS ASSIGN constant + new controller method |
| Admin Selection Request Response implementation reference | update data transfer object comment (button no longer disabled) |
| frontend index implementation reference, application programming interface ROUTES, Matching implementation reference, Selection Request Queue implementation reference | un-disable assign, call user Id/selection-request assign |

---

## Boss-review condition (2026-06-03) — 0033f duplicate-match guard
Team review (Morgan/DBA) found the unique operation becomes a no-op
for free-market matches (both tutor post id NULL) → no database-level dedupe under concurrent admins.
**Required addition** to changeset 0033:


Implementation example omitted. This public edition retains the written requirements in this section.


Keep an app-level pre-check for a friendly 409 (MATCH ALREADY EXISTS); the partial-unique is the
integrity backstop. Mirror the active-status set used elsewhere if it differs.
