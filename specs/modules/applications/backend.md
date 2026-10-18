# InTuition — Applications Module (M15) — Backend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The apply guard and applicant audience below are corrected to [M18](../../PRD.md). Listing is optional; use [012](../../012-tutor-apply-and-profile-ux/design.md) canonical profile completeness. Follow [INT-093](../../architecture/int-093-batch-state-machine-integrity-spec.md) for the expanded idempotent apply/reapply/withdraw state table. The parent has a separate own-job personal information-free applicant view; the admin endpoint remains admin-only.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** applications
**Version:** 1.1.0
**Status:** Implemented; team-reviewed (M15 review + Boss, 2026-05-29)
**Last Updated:** 2026-05-29
**Platform:** InTuition
**Tech Stack:** Java 21 + Spring Boot 3.x

> Authoritative spec: [spec md](spec.md). Schema: [db md](db.md).
> Frontend: [frontend md](frontend.md). Decision: [ADR-013](../../ROADMAP.md).
> Review record: [reviews/M13-M14-M15-team-review md](../../architecture/reviews/M13-M14-M15-team-review.md).

All paths are referenced through route constants (no raw string literals in logic
— ADR-006). The thin slice writes only the APPLIED and WITHDRAWN statuses;
SHORTLISTED/REJECTED/MATCHED exist in the enum but no endpoint sets them
(see [spec md §5](spec.md)).

> **Team-review corrections applied (2026-05-29):** the scope-creep
> APPLICATION RECEIVED tutee notification was **cut** (it dead-ended on a
> non-existent route and was outside the ratified slice); re-apply
> **reactivates the single durable row in place** (it does not insert a new row);
> withdraw returns **200**. The historical active-listing guard is superseded by M18; see the corrected guards below.
> This doc reflects the shipped code.

---

## 1. application programming interface Endpoints

### 1.1 Apply to a Job (Tutor)


The illustration for apply to a job tutor contains the following information:

- POST tutor applications interface operation.
- Authorization: TUTOR.


**Request body (Create Application Request):**

| Field | Type | Required | Notes |
|---|---|---|---|
| tutee Post Id | unique identifier | yes | The job being applied to |
| tutor Post Id | unique identifier | no | Optional pin to the listing applied with; NULL allowed |
| message | string | no | Optional pitch, max 500 chars |

**Guards (in order):**

1. Caller is TUTOR → else 403 (Security Config + @Pre Authorize).
2. tutee Post Id must exist → 404 TUTEE POST NOT FOUND.
3. Tutee post must be status ACTIVE → 422 TUTEE POST NOT ACTIVE
   (only ACTIVE jobs accept applications).
4. **No published tutor listing is required (M18).** Check the canonical server profile-completeness
   predicate from 012; an incomplete profile receives the existing TUTOR PROFILE INCOMPLETE gate.
   Remove the former TUTOR NO ACTIVE POST requirement; tutor Post Id remains optional.
5. If tutor Post Id is supplied, it must exist (→ **400 TUTOR POST INVALID**) and
   be owned by the calling tutor (→ **403 TUTOR POST INVALID**).
6. **Quota cap (INT-077):** the tutor may hold at most quota applications-per-tutor
   (**default 10**) **active** applications across all jobs (active = APPLIED +
   SHORTLISTED). At the cap → **422 ERR APPLICATION LIMIT REACHED**. Withdraw frees a
   slot. The count→check→insert is serialized by a transaction-scoped
   the pg advisory xact lock operation taken before the count (race-safe;
   full design in [int-077-marketplace-quota-caps implementation reference](../../architecture/int-077-marketplace-quota-caps.md)).
   The invitation-**ACCEPT** path bypasses this cap
   (enforce Cap is no) but still takes the lock.

**Idempotency & withdraw → re-apply (single durable row per pair):**

The action is **idempotent per tutee Post Id, tutor Id**:

- If a non-withdrawn (APPLIED) application already exists for the pair, the
  endpoint returns that existing application unchanged (web request protocol **200**, no insert,
  no error). The partial unique index uq job applications active pair is the
  database-level backstop.
- If the only prior application for the pair is WITHDRAWN, that **same row is
  reactivated in place** (status → APPLIED, message/tutor-post refreshed). No new
  row is inserted — there is exactly **one durable row per (job, tutor) pair**, ever
  (ratified model — see §3 and [db md](db.md)).

**Processing:**

1. In a transaction, create the row (first apply) or reactivate the WITHDRAWN row:
   status APPLIED, tutee post id, tutor id, optional tutor post id,
   optional message. created at is set by database default on insert; updated at is
   maintained by the trg job applications updated at trigger and also set in the
   service.
2. **Analytics:** emit application submitted via Analytics Event Publisher with
   the tuition-request identifier, tutor-listing identifier and whether the application was reactivated (matches the sibling-service pattern).

**Response 200 — Application Response** (always 200, whether created, reactivated,
or an idempotent hit):


The apply to a job tutor example describes information including id, tutee Post Id, tutee Post Title, tutee Post Subject, tutee Post Level, tutor Post Id, message, status, created At, updated At, quota. Concrete payload values and implementation syntax are omitted.


> **INT-077:** every Application Response (apply + each row of read operation for tutor applications)
> now carries quota (the allowed limit and remaining allowance, remaining clamped ≥0), and the
> withdraw response is success confirmation and the updated application limit and remaining allowance (was just
> a success confirmation) so the frontend shows the freed slot immediately.

**Errors:** 401 (unauthenticated), 403 (not TUTOR /
TUTOR POST INVALID not-owned), 404 TUTEE POST NOT FOUND, 422
TUTEE POST NOT ACTIVE, 400 TUTOR POST INVALID (not-found). A concurrent
double-apply that trips the partial unique surfaces as **409 CONSTRAINT VIOLATION**
via the global handler (not a 500).

---

### 1.2 Withdraw an Application (Tutor)


Implementation example omitted. This public edition retains the written requirements in this section.


Despite the DELETE verb, this is a **status flip, not a row delete** — the row is
retained (APPLIED → WITHDRAWN) so re-apply (reactivation) and any later analytics
stay intact (db md §4 revokes physical DELETE on the table).

**Guards:**

1. Application the selection-request identifier must exist → 404 APPLICATION NOT FOUND.
2. Application must be owned by the calling tutor → else 404 APPLICATION NOT FOUND
   (404-no-leak: a tutor is never told another tutor's application exists).
3. Withdrawing an **already-WITHDRAWN** application is an idempotent **no-op** (200).

**Processing:**

1. In a transaction, set status WITHDRAWN; updated at bumped by the trigger.
2. Clearing the active row frees the tutee post id, tutor id slot in the partial
   unique, so the tutor may re-apply later (reactivating this same row).
3. Emit application withdrawn via Analytics Event Publisher the tuition-request identifier and application identifier.

**Response:** 200 a success confirmation.

**Errors:** 401, 403 (not TUTOR), 404 APPLICATION NOT FOUND (missing or not owned).

---

### 1.3 List Own Applications (Tutor)


Implementation example omitted. This public edition retains the written requirements in this section.


Backs the "My Applications" page. Returns the calling tutor's applications, newest
first, served by idx job applications tutor tutor id, created at DESC.

**Response 200 — a page of application records, with pagination details** (the standard envelope):


The list own applications tutor example describes information including data, id, tutee Post Id, tutee Post Title, tutee Post Subject, tutee Post Level, tutor Post Id, message, status, created At, updated At, meta. Concrete payload values and implementation syntax are omitted.


Includes the tutor's own withdrawn applications (so the page can show history). The
job-summary fields (tutee Post Title/Subject/Level) come from the tutee post; **no
personal information** about the tutee is included.

**Errors:** 401, 403 (not TUTOR).

---

### 1.4 View Applicants for a Job (Admin)


Implementation example omitted. This public edition retains the written requirements in this section.


The admin's applicant pool, feeding M4 matching. **ADMIN only** in the thin slice —
the OPERATOR role does not exist yet (User Role = TUTEE/TUTOR/ADMIN, verified by
the M13/M14/M15 review). When M13 adds OPERATOR this endpoint can additively accept
it; no contract change today.

**Guards:**

1. Caller is ADMIN → else 403.
2. tutee Post Id must exist → 404 TUTEE POST NOT FOUND.

**Response 200 — a page of admin application records, with pagination details:**


The view applicants for a job admin example describes information including data, id, tutee Post Id, tutor Id, tutor Display Name, tutor Post Id, message, status, created At, updated At, meta. Concrete payload values and implementation syntax are omitted.


The tutee/parent does not reach this ADMIN endpoint. M18 additionally requires an OWN-post, personal information-free parent
applicant projection; this admin endpoint is no longer the only place applicant
identities are exposed (admin-mediated privacy model). It returns all applications
for the job newest-first (including withdrawn) in this slice.

> **Known limitation (tracked fast-follow):** tutor Display Name is resolved
> per-row (a find By User Id per applicant) — an N+1 to batch with find By User Id In
> before this endpoint sees volume.

**Errors:** 401, 403 (not ADMIN), 404 TUTEE POST NOT FOUND.

---

## 2. Authorization Matrix

| Endpoint | TUTEE | TUTOR | ADMIN | Anonymous |
|---|---|---|---|---|
| submit operation for tutor applications | 403 | ✅ (own) | 403 | 401 |
| delete operation for tutor applications id | 403 | ✅ (own only) | 403 | 401 |
| read operation for tutor applications | 403 | ✅ (own) | 403 | 401 |
| read operation for admin posts id applications | 403 | 403 | ✅ | 401 |

- Tutor endpoints are gated the has role operation on the tutor applications ** interface operation
  matcher in Security Config; ownership of the specific application is enforced in
  the service layer (a tutor withdrawing another's application gets **404-no-leak**).
- Admin endpoint gated the has role operation on the admin ** interface operation matcher.
- Tutees and parents have **no** endpoint that returns applicant identities — by
  design (resolved decision 2, [spec md §5](spec.md)).

---

## 3. Idempotency and Withdraw → Re-apply Semantics

The behaviours that drive the schema design in [db md §2.1](db.md):

| Scenario | Existing rows for (job, tutor) | Result |
|---|---|---|
| First apply | none | Insert APPLIED, **200** |
| Duplicate apply (still active) | one APPLIED | Return existing row, **200**, no insert (**idempotent**) |
| Withdraw | one APPLIED | Flip to WITHDRAWN, **200** a success confirmation |
| Re-apply after withdraw | one WITHDRAWN | **Reactivate that same row in place** → APPLIED, **200** |
| Concurrent first-apply race | none | One wins; the loser trips the partial unique → **409 CONSTRAINT VIOLATION** (handled, not 500) |

The partial unique when status WITHDRAWN permits exactly one active row per
pair. Re-apply reactivates the (single) withdrawn row rather than accumulating rows,
so the M14 count (status APPLIED) can never double-count.

> The concurrent-race 409 is a graceful conflict, but the spec's "idempotent 200"
> intent is only fully met for the sequential duplicate-apply path. Catching the
> constraint violation to re-resolve to 200 under the race is a tracked fast-follow.

---

## 4. Analytics Touchpoints

| Event | Emitted on | Properties |
|---|---|---|
| application submitted | Successful apply / reactivation (§1.1) | tutee Post Id, tutor Post Id, reactivated (bool) |
| application withdrawn | APPLIED → WITHDRAWN flip (§1.2) | tutee Post Id, application Id |

Emitted via Analytics Event Publisher (currently a structured-log stub — wired to a
real pipeline in a later phase, platform-wide). No tutee notification is emitted in
this slice (the earlier APPLICATION RECEIVED notification was cut in review).

---

## 5. Out of Scope (this slice)

- Admin-set SHORTLISTED / REJECTED endpoints and their notifications.
- M4 matching transitioning related applications to MATCHED.
- Any tutee-facing notification or applicant view (the parent is served by the
  admin-mediated match flow; revisit with the admin triage slice / M13 OPERATOR).
- Any applicant Count endpoint or field — added by **M14** once M15 ships. The count
  query is pinned in [db md](db.md):
  count distinct tutors with an APPLIED application to the selected tuition request.
