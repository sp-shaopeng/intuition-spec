# M18 — Assignment Lifecycle & Free Marketplace — Backend Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The final founder decisions remain effective: tutor listings are optional, profile identity is canonical, and earlier OPEN tables are superseded. Use [012](../012-tutor-apply-and-profile-ux/requirements.md) for the single server completeness predicate and email-link contact changes; use [INT-157 v2](int-157-commission-first-month-locked-spec.md) for agreed-plan activation locking. First-session confirmation remains required and authenticated-tutee-only. PAID means agency receipt of tutor commission, not parent lesson payment.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** M18 · **Layer:** Backend (Spring Boot 3.3 / Java 21)
**Status:** Draft for team review · **Date:** 2026-05-31
**Owner:** Backend Tech Lead
**Source of truth:** the service code (founder decree). This spec cites real file:line for every
change against existing code and uses Liquibase for ALL schema (no raw database structure definitions).
**Drives:** implementation of product requirements document §20 + the 3 Boss carry-forward conditions
(set-based supersede, optimistic-lock concurrency, after-commit failure-isolated notifications).

**Ground-truth docs read in full before writing:** product requirements document md §1–§7.6 and §20 (Assignment Lifecycle &
Free Marketplace), architecture/assignment-lifecycle-design md (the approved design, all sections).

> **Conventions (binding, from CLAUDE md):** Java 21; Liquibase for all schema; no magic strings
> (backend application programming interface paths → the application component Routes constants, which mirror the frontend application programming interface ROUTES
> registry; enum values → typed enums/constants); coverage gate
> **≥97.5% line AND branch**; error envelope is nested a structured error containing its error code; errors raised via
> App Exception factory methods (bad Request 400 / unauthorized 401 / forbidden 403 /
> not Found 404 / conflict 409 / unprocessable Entity 422 / too Many Requests 429 / locked 423).

> **Verification status.** All cited code was read directly against the live backend and is
> **CONFIRMED** except where marked **VERIFY-LINE** (exact line may shift). Several design-doc
> assumptions were WRONG against the real code and are corrected below. Under-specified product
> decisions are flagged **OPEN backend** rather than guessed.
>
> **Confirmed ground truth (2026-05-31, read directly) — and corrections to the design doc:**
> - **Every PK/FK is unique identifier** (the generated value operation), NOT bigint. Job Application id,
>   Match id, Tutee Post id, Tutor Post id, Tutor Profile id all unique identifier.
> - **Routes constant class is the application component Routes** (NOT Api Paths, NOT application programming interface ROUTES).
>   Enforced by No Hardcoded Routes Arch Test. It has TUTEE POSTS tutee posts interface operation,
>   TUTOR APPLICATIONS, ADMIN POSTS, ADMIN MATCHES. **There is NO applicants/select constant yet
>   — both must be ADDED.**
> - application states: applied, withdrawn, shortlisted, rejected or matched (CONFIRMED), backed
>   by PG enum job application status created in **0018-create-job-applications implementation reference**
>   (define the application states as applied, withdrawn, shortlisted, rejected and matched).
>   Job Application status is the enumerated operation.
> - **Changelog head = 0024** (0024-audit-log-details-to-text implementation reference). M18 starts at **0025**. CONFIRMED.
> - Job Application: @Many To One Tutee Post tutee Post (tutee post id), @Many To One User tutor
>   (**tutor id** — field is tutor, NOT tutor User), @Many To One Tutor Post tutor Post
>   (tutor post id, **nullable** — optional pitch), message, status, created At, updated At.
>   Repo helpers (CONFIRMED): find First By Tutee Post Id And Tutor Id And Status, count By Tutor Id And Status.
>   **0018 already has a PARTIAL unique tutee post id, tutor id when status WITHDRAWN** and
>   indexes incl. tutee post id when status is APPLIED and tutor post id when required.
> - Match (CONFIRMED): tutor Post, tutee Post (both required), status (match status),
>   **created By unique identifier**, **matched At**, created At, updated At. **No agreed Rate, no
>   match Activated At, no completed At, no source application id.** Matches table = 0009.
> - Tutee Post (CONFIRMED): unique identifier id, tutee User, status (post status, default DRAFT),
>   subject Code, level Code, budget/schedule/location fields. **No @Version today.**
> - Tutor Profile (CONFIRMED): separate @Generated Value unique identifier id PLUS
>   the join column operation (NOT @Maps Id). Only display Name,
>   qualifications, years Experience, bio, featured Consent, created At, updated At. **No
>   rate/area/commitment.** Table created in 0002-create-profiles implementation reference; backfill join key = **user id**.
> - Tutor Post (CONFIRMED): hourly Rate Min Sgd/Max (required), location Planning Area,
>   location Region (enum), location Mrt Station, created At. User FK column **tutor id**
>   (count By Tutor Id And Status).
> - Admin Application Controller (CONFIRMED): the pre authorize operation,
>   the administration operation to read a tuition request’s applications →
>   the list applicants for job operation → a page of administration application results.
>   (NOTE: path is hardcoded as APPLICANTS PATH constant in the controller, not in Api Routes — the
>   new parent endpoint MUST put its path in Api Routes.)
> - **Email is NOT SMTP/Java Mail Sender.** the enqueue operation
>   pushes a **templated message onto a Redis stream** consumed by an out-of-process worker, gated by
>   **email delivery.enabled** (CONFIRMED). application-local implementation reference currently sets
>   **email delivery.enabled is no** (no consumer locally). Auth Service forgot Password
>   (437/) does the after commit operation.
>   **This materially changes Bug #3 (§9): Mailhog needs the worker to point at it, OR the local
>   profile needs an SMTP-direct sender — see §9.**
> - Showcase personal information test = **the application component Tutor Response Pii Contract Test** (CONFIRMED path).
>   Sibling Showcase Request Response Pii Contract Test also exists — both are reflection-over-fields tests.

---

## 0. Scope & traceability

This spec covers, item-for-item, the binding list (A–J). Each maps to product requirements document §20 sub-sections and the
design doc:

| § | Topic | product requirements document §20 | Design § |
|---|---|---|---|
| A | Free-market apply (remove listing guard; profile-completeness gate) | §20.1, §3.6 | §0, §3 guard note |
| B | Tutor profile canonical identity (commitment + rate + area) | §20.2, §5.1a | §5 |
| C | Job Application Status SUPERSEDED | §20.2 | §2.2 |
| D | Atomic assign transition (Admin Match Service create Match) | §20.4 | §3.1, §2.4 |
| E | New endpoints (applicants list, choose-tutor select) | §20.3, §20.6 | §3.2, §4 |
| F | personal information contract + applicant data transfer object contract test | §20.6 | §5a |
| G | Money path (reuse existing commission flow) | §20.5, §8.3 | §3 |
| H | Observability (metrics, events, stale-commission view) | §20.8 | §10 |
| I | Bug #3 — Mailhog dev SMTP | §20.7 (bug ③) | §7 |
| J | Test impact | §20.9 | §9 (deliverable 5) |

---

## 1. Domain model — target state

### 1.1 Post Status (Tutee Post / job) — REUSE MATCHED, do NOT invent ASSIGNED

Existing enum (per design §2.1): DRAFT, PENDING, ACTIVE, REVISION REQUESTED, REJECTED, MATCHED,
CLOSED, EXPIRED. No new state is added. “Assigned” maps to MATCHED**.

- **ACTIVE** — in public browse, accepting applications.
- **MATCHED** (= assigned) — tutor chosen; removed from browse; new applies blocked
  (TUTEE POST NOT ACTIVE 422); other applications → SUPERSEDED.
- On admin cancel before completion: **MATCHED → ACTIVE** (reopen).

CONFIRMED: Tutee Post status is the enumerated operation — a PG
enum post status that ALREADY contains MATCHED. **No enum file change, no changeset for Post Status.**

### 1.2 Job Application Status — add SUPERSEDED (item C)

Current (verified design §2.2): applied by default, withdrawn, shortlisted, rejected or matched.
Add **SUPERSEDED**.

Target transitions (design §2.4):

| From | To | Trigger |
|---|---|---|
| (new) | APPLIED | tutor applies (free market) |
| APPLIED/SHORTLISTED | MATCHED | winner at assign |
| APPLIED/SHORTLISTED | SUPERSEDED | **other** apps at assign (set-based) |
| SUPERSEDED | APPLIED | job reopened on match cancel |
| MATCHED | APPLIED | winner's app on match cancel |
| APPLIED | WITHDRAWN | tutor withdraws own app |

REJECTED enum value stays but is **unused by the parent flow in v1** (design Q5 — WON'T v1).

**Two-part change** because job application status is a **PostgreSQL enum type** (CONFIRMED origin:
0018-create-job-applications implementation reference):

1. **Java enum:** add SUPERSEDED to
   Job Application Status implementation reference.
2. **PG enum value** — its OWN changeset, run In Transaction is no (Postgres forbids
   ALTER TYPE ... ADD VALUE inside a transaction block that later uses the value):


Implementation example omitted. This public edition retains the written requirements in this section.


> **Ordering rule:** this ADD VALUE changeset MUST be a separate changelog file/changeset from any
> changeset that *uses* SUPERSEDED (e g. a backfill or the supersede update records), because the new enum
> label is not usable in the same transaction that adds it. The bulk supersede update records (item D) runs at
> runtime, not in a migration, so it is unaffected.

### 1.3 Match Status — unchanged

pending, active, completed or cancelled (verified). Terminal success = **COMPLETED** (set when
commission PAID), per product requirements document §7.5 terminology note — NOT "CLOSED". No change.

### 1.4 Tutor Commitment — NEW enum (item B)

New file Tutor Commitment implementation reference:
part-time or full-time. Stored on tutor profiles commitment. Decision: store as
**varchar + the enumerated operation** (NOT a PG enum) to avoid the ADD VALUE transaction
friction for future commitment values; matches the lower-risk path for a brand-new column.

---

## 2. Schema changes (Liquibase) — changesets start at 0025

**CONFIRMED changelog head = 0024** (0024-audit-log-details-to-text implementation reference; tail: 0017
cancelled-invoice-status, 0018 job-applications, 0019 tutor-showcase, 0020 admin-management, 0021
interviews-tutee-post, 0022 plaintext-email-phone, 0023 tutee-post-showcase-index, 0024
audit-log-details-to-text). **New changesets start at 0025.** Keep the SUPERSEDED ADD VALUE first.

Place new files in intuition-the private implementation folder and include them from
the master changelog in order.

### 2.1 0025 — add SUPERSEDED PG enum value
See §1.2 (own file, run In Transaction is no).

### 2.2 0026 — tutor profiles canonical-identity columns


Implementation example omitted. This public edition retains the written requirements in this section.


No required constraint at database level (backfill cannot guarantee completeness; the **apply-time
profile-completeness gate**, §3.2, enforces presence at the application boundary instead — this is the
correct place because the requirement is "complete profile to apply," not "complete profile to exist").

> **CONFIRMED:** tutor profiles (table from 0002-create-profiles implementation reference) has its own unique identifier PK id
> plus a user id FK (@One To One, NOT @Maps Id). Add the new columns to it. The Tutor Profile
> entity maps commitment as the enumerated operation.

### 2.3 0027 — backfill profile rate/area from most-recent Tutor Post

Schema-coupling rule: backfill ships in the **same pull request** as 0026. Set-based update records, most-recent post
per tutor wins. commitment is **not** backfillable (it never existed anywhere) → stays null;
those tutors must pick PT/FT at next profile edit before they can apply.

CONFIRMED columns: tutor posts user FK is **tutor id** (not tutor user id), with
hourly rate min sgd, hourly rate max sgd, location planning area, location region,
created at. tutor profiles join key is **user id**. location region is a PG enum on both
tables — the backfill copies it enum-to-enum.


Implementation example omitted. This public edition retains the written requirements in this section.


tutor posts created at is CONFIRMED present, so ordered by created at DESC ("most-recent post") is
sound (OPEN #1 resolved). commitment is not backfillable → stays null until the tutor edits.

### 2.4 0028 — match source application id FK + supersede index + Tutee Post @Version

CONFIRMED: table is matches (0009); job applications id is **unique identifier**;
job applications tutee post id exists (0018); tutee posts has **no @Version** today (0028c needed).


Implementation example omitted. This public edition retains the written requirements in this section.


Match entity gains @Many To One using fetch is LAZY @Join Column using name is source application id Job Application
source Application;.Tutee Post gains a record version used to detect competing changes. (CONFIRMED no pre-existing
@Version — OPEN #2 resolved, 0028c required.)

### 2.5 tutor selection requests — NEW table (item E, selection-request persistence)

All IDs are **unique identifier** (consistent with every other table). Timestamps use Offset Date Time (codebase
convention, e g. Job Application created At).


Implementation example omitted. This public edition retains the written requirements in this section.


> **⚠️ INDEX FIX (team-review block, Morgan/Chris):** the unique index definition block above is
> WRONG — a plain unique tutee post id, status forbids more than one CONSUMED/EXPIRED row.
> **DELETE that index definition block entirely** and use ONLY this partial-unique index (the single
> authoritative definition):
> sql
> require a unique lookup index ux selreq one open per job
>   ON tutor selection requests (tutee post id) when status = 'CREATED';
> 
> Emit via Liquibase raw PostgreSQL-specific SQL block. Single-open-per-job is RESOLVED = yes (§12 #3):
> "mutable until admin confirms" = one editable CREATED request; re-choose UPDATEs the row.

**Selection-request lifecycle (state machine):**


Implementation example omitted. This public edition retains the written requirements in this section.


CONSUMED/SUPERSEDED/EXPIRED are terminal. The selection request **never itself creates a Match**
(AC-04 admin-only matching holds) — it is purely a parent→admin signal that the admin later consumes
by calling the one atomic assign method (§4).

**Entity:** Tutor Selection Request + Tutor Selection Request Status enum
(CREATED, CONSUMED, SUPERSEDED, EXPIRED) + Tutor Selection Request Repository.

---

## 3. Free-market apply (item A)

### 3.1 Remove the TUTOR NO ACTIVE POST guard

Job Application Service implementation reference,
the apply operation (CONFIRMED signature, a database transaction).
**CONFIRMED — delete the active-listing guard at Job Application Service:**


Implementation example omitted. This public edition retains the written requirements in this section.


Also delete the now-unused constant ERR TUTOR NO ACTIVE POST (54) and update the the apply operation
Javadoc which currently says "the tutor must hold at least one ACTIVE tutor post." If
count By Tutor Id And Status becomes unused, leave it (used elsewhere) — verify before removing.
tutor Post Id stays **optional** (CONFIRMED tutor post id nullable; the resolve tutor post operation handles
null).

> Note: TUTOR NO ACTIVE POST ALSO guards **browse** in Tutee Post Service ("must have an active
> tutor post to browse tutee requests"). **OPEN (backend) #14** — the free-market model implies tutors
> should browse jobs without a listing too; the design only names the *apply* guard. Recommend remove
> the browse guard as well for consistency, but flag for product confirmation (out of strict §20 scope).

**Remaining apply guards after this change (CONFIRMED already in the apply operation):**
1. **Job must be ACTIVE** — if (tutee Post get Status operation  differs from  Post Status ACTIVE) throw
   App Exception unprocessable Entity using ERR TUTEE POST NOT ACTIVE, ...422:87-90). Only lifecycle
   guard. Once MATCHED, naturally blocks new applies. **Keep.**
2. **Idempotency (not a 409)** — CONFIRMED the apply operation is **idempotent**: an existing APPLIED row for
   tutee Post Id, tutor Id is returned unchanged; an existing WITHDRAWN row is reactivated to
   APPLIED. **There is NO ALREADY APPLIED 409** — earlier OPEN #4 was wrong. Backed by 0018's
   partial-unique tutee post id, tutor id when status WITHDRAWN. **Keep this behaviour as-is.**
3. **Profile-completeness gate** — NEW, see §3.2.

### 3.2 Profile-completeness gate (replaces the listing gate; throttles spam)

Before persisting the application, load the tutor's Tutor Profile and assert all four are present:

- commitment (PART TIME/FULL TIME) non-null
- hourly Rate Min Sgd AND hourly Rate Max Sgd non-null (and max ≥ min, S$10–1000 per §5.1a)
- location Planning Area non-null (region derivable from it)
- qualifications non-blank

If incomplete →
**the unprocessable entity operation → web request protocol 422**, envelope
an error explaining that the tutor profile is incomplete.

> **Status choice:** 422 (not 403) — the request is well-formed and the tutor is authorized; it is
> rejected on a business pre-condition (incomplete profile). Mirrors TUTEE POST NOT ACTIVE's 422.
> **OPEN (backend) #5** — product requirements document §3.6 says "apply gated only by profile completeness … returning a clear error
> code"; it does not fix the exact code/status. Proposed: TUTOR PROFILE INCOMPLETE / 422. Confirm at
> review. (frontend maps this code to a "Complete your profile to apply" CTA → profile edit.)

Implementation: a private the assert profile complete to apply operation helper in
Job Application Service (inject Tutor Profile Repository), called in the apply operation **after the
job-ACTIVE check and before the idempotency lookup**. Keep it pure (throws or returns) so branch
coverage is clean. Each missing-field branch is a distinct test (item J).

### 3.3 application submitted event carries listing Attached

Emit/extend the existing application submitted analytics event with a boolean prop
listing Attached tutor Post Id differs from no value (design §10 / product requirements document §20.8) to measure free-market spam and
applications-per-job. See §8.

---

## 4. Atomic assign transition (item D)

### 4.1 One method, two callers

Both the existing admin 2-panel match creation AND the parent-selection-driven assign converge on the
**single** extended method
CONFIRMED signature: Admin Match Service create Match(unique identifier tutor Post Id, unique identifier tutee Post Id, unique identifier
admin User Id)Admin Match Service implementation reference:89a database transaction at). Extend it with an
overload/param carrying the chosen unique identifier tutor Id (the winning applicant) + optional unique identifier
source Selection Request Idfor the parent-driven path. **Inject Job Application Repository** (CONFIRMED
not injected today). Today the method only: loads posts, checks tutee Post ACTIVE
(ERR TUTEE NOT ACTIVE 422, 101), checks cardinality
(the exists by tutee post id and status in operation → ERR TUTEE ALREADY MATCHED 409, 107),
creates a PENDING Match (set Created By, set Matched At), and calls the notify match created operation
(already after-commit/failure-isolated — reuse this pattern). It does **NOT** touch applications or the
job status today. Cancel = **the cancel match operation** (256, no admin User Id param);
activate = the activate match operation (198).

Wrap all new side-effects in the existing a database transaction.

### 4.2 Side-effects (all in one transaction)

1. **Resolve the winning application:** the chosen tutor's APPLIED/SHORTLISTED app for this
   tutee Post Id. If found → status MATCHED; set match source Application (FK 0028a). If the admin
   assigns a tutor who never applied (2-panel path), source application id stays null (allowed).
2. **Job → MATCHED:** tutee Post status MATCHED. Saving the entity **bumps @Version** — the
   optimistic-lock point (§4.3). Drops out of public browse + blocks new applies
   (TUTEE POST NOT ACTIVE).
3. **Supersede all others — SINGLE set-based update records** (never a read-loop, design §3.1 / Boss
   condition). Repository method on Job Application Repository:

Job Application maps tutee Post as a @Many To One (no scalar tutee Post Id field) so the JPQL
navigates a tutee Post id. IDs are **unique identifier**. Null-safe winner-exclusion lets the SAME method serve
the 2-panel (no-winner) path:


Implementation example omitted. This public edition retains the written requirements in this section.


   - Returns the affected-row count → feeds applications superseded total (§8) + the
     application superseded event count.
   - 2-panel path with no winning app: pass winning Application Id no value → ALL
     APPLIED/SHORTLISTED for the job superseded.
   - **Uses the tutee post id, status index** (0028b) for the when — no full-table scan, no
     per-row lock fan-out beyond the matched set.
   - clear Automatically is yes so the persistence context doesn't serve stale app entities after the
     bulk update.
4. **Match record:** create the PENDING match (existing create Match behaviour: set Created By,
   set Matched At) and additionally the set source application operation when present. Commission record
   PENDING follows the existing §8.3 flow (unchanged — item G).
5. **Selection request (parent path):** if source Selection Request Id provided, mark it CONSUMED;
   mark any other CREATED selection requests for this job SUPERSEDED.

### 4.3 Concurrency — optimistic version on Tutee Post → 409

a record version used to detect competing changes on Tutee Post (column from 0028c). Two admins (or admin + parent-
consume racing) loading the ACTIVE job and both committing an assign: the **second commit throws
Object Optimistic Locking Failure Exception**. Translate it in the controller-advice / service boundary
to **the conflict operation → web request protocol 409** (envelope
an error explaining that the job is already assigned) — never a silent double-match.

> Belt-and-braces: the assign should also re-assert tutee Post status equals ACTIVE at the top of the tx
> and throw JOB ALREADY ASSIGNED 409 if not — covers the case where the first tx already committed
> before the second loaded. The @Version handles the true read-then-write race; the status check
> handles the sequential-but-stale case. Both are cheap; keep both for clean branch coverage.

### 4.4 Notifications — after-commit + failure-isolated (Boss condition)

Notification fan-out (winner "Assigned", each superseded tutor "position filled", parent
"assignment confirmed") MUST run **after the tx commits** and a failure MUST NOT roll back the
assignment.

- **Reuse the existing the after commit operation helper** already used by Admin Match Service notify Match Created
  and Auth Service forgot Password (CONFIRMED pattern in this codebase) — do NOT invent a new mechanism.
  the enqueue operation pushes to the Redis stream; failures must be swallowed.
- Inside after Commit, iterate recipients and enqueue each notification in a **try/catch per
  recipient — swallow-and-log** (one tutor's failed enqueue must not block the others or the assign).
- Recipients of "position filled" = the set just superseded. Because the bulk update records doesn't return
  rows, capture the recipient user-ids with a read fields of tutor user id for the to-be-superseded
  apps **before** the update records (still inside the tx), and pass that list into the after-commit handler.

**Notification template IDs:** the codebase uses **N-xx template ids** with the enqueue operation
(CONFIRMED). product requirements document §9 reserves N-08/N-09 (match created → tutee/tutor) and N-16/N-17 (match cancelled).
There is no template today for superseded/assigned-to-you/parent-assignment-confirmed. **OPEN (backend) #6**
— allocate new N-xx ids (e g. N-19 application superseded, N-20 application matched, N-21
assignment confirmed — verify next free number against the live template catalog) and add the
template bodies to the M9 worker. The assign winner+parent notifications MAY reuse N-08/N-09 copy, but
the **superseded fan-out needs a new template** ("position filled / no longer available").

### 4.5 Cancel → reopen

Extend **the cancel match operation** (CONFIRMED 256; it already cancels a
PENDING commission and leaves PAID commissions untouched). On cancel **before completion** (match
PENDING/ACTIVE → CANCELLED):

1. tutee Post status MATCHED → **ACTIVE** (bumps @Version).
2. Reopen applications: the winner's app MATCHED → APPLIED; all SUPERSEDED apps for this job →
   APPLIED — a **single set-based update records** covering both source statuses, using the same
   tutee post id, status index:


Implementation example omitted. This public edition retains the written requirements in this section.


3. Commission → CANCELLED (existing path). Post-INVOICED cancel keeps the product requirements document §4 admin-decision rule
   (WAIVED/DISPUTED) — unchanged, not part of this reopen.
4. After-commit notifications (job reopened) — same failure-isolation discipline.

> **Edge:** if the match is already terminal (COMPLETED/CANCELLED), cancel is **not allowed** —
> CONFIRMED cancel Match already throws on terminal statuses (266, "Match cannot be cancelled
> (current status: …)"). Reuse that guard; do not add a second.

---

## 5. New endpoints (item E)

All paths via **the application component Routes** constants (the codebase's registry; enforced by
No Hardcoded Routes Arch Test — so a path literal WILL fail the build). Error envelope nested.

> **Api Routes additions required** (none exist for these yet):
> the tutee’s operation to view applicants for a request and
> the tutee’s operation to select a tutor for a request. (The admin applicant path lives as a private
> APPLICANTS PATH constant inside Admin Application Controller and is left untouched.)

### 5.1 read operation for tutee posts id applicants — parent applicant list (personal information-free)

| | |
|---|---|
| **Method/Path** | read operation for tutee posts id applicants |
| **Auth** | Authenticated TUTEE; **must own the post** (owner-scoped). |
| **Query** | page (0-based), size (default 20), sort (default applied At,desc) — per existing Page Response convention. |
| **Response** | 200 a page of applicant cards |
| **Errors** | 401 not authenticated; 403 FORBIDDEN IDOR (post not owned by caller); 404 POST NOT FOUND. |

This is a **NEW parent-facing** endpoint. CONFIRMED today's only applicant list is **admin-only**:
Admin Application Controller (the pre authorize operation,
the administration operation to read a tuition request’s applications) →
the list applicants for job operation →
a page of administration application results (personal information). Add a controller method on a tutee-scoped controller
(new Tutee Application Controller, the pre authorize operation) bound to
Api Routes TUTEE POSTS APPLICANTS — do **NOT** reuse the admin path or data transfer object. The two surfaces differ:
admin = the administration operation to read applications for the selected post returning personal information; parent = the tutee’s operation to read applicants for the selected post
returning the personal information-free card.

**Applicant Card Response (DISTINCT personal information-free data transfer object — item F):** DISTINCT over applicants for the job
(one card per tutor even if data overlaps). Fields ONLY:


The get tutee posts interface operation id /applicants — parent applicant list personal information-free example describes information including id, tutor Public Handle, job Subject, job Level, hourly Rate Min Sgd, hourly Rate Max Sgd, coarse Area, years Experience, commitment, rating, review Count, application Status, applied At, pitch, also Teaches. Concrete payload values and implementation syntax are omitted.


**Value sourcing — profile-canonical + listing-overrides-when-attached (design §5, item B):**
- Default: read hourly Rate*, commitment, coarse Area, years Experience from the tutor's
  **Tutor Profile**.
- **If** the application has a non-null tutor Post Id: that listing's rate + area **override into the
  flat fields** for that card (one source of truth per context; no sync job; **no structured
  attached Listing object** is returned).
- **Lead subject/level = the applied-to job's job Subject/job Level** (the tutor profile has no
  subject columns — OPEN #7 RESOLVED, §12 #7). If a listing is attached, its subjects populate
  **also Teaches** (a flat string array), rendered as the card's "Also teaches" line.
- pitch = the applicant's optional Job Application message; text only (the personal information contract test, §6,
  must assert it carries no name/phone/email pattern at the data transfer object boundary).
- coarse Area: derive from location region (Central/North/South/East/West) — never the planning
  area (that's finer-grained personal information per the personal information contract).
- tutor Public Handle = the tutor's tutor profiles public handle (§12 #8) — NEVER display Name.

**personal information rule (item F):** never real name, email, phone, exact location. Child name + parent phone are
hidden from applicants until the parent commits (counterparty rule) — not relevant to this endpoint
(parent is the caller) but enforced on the reciprocal tutor-facing views.

### 5.2 submit operation for tutee posts id select — choose tutor (selection request)

| | |
|---|---|
| **Method/Path** | submit operation for tutee posts id select |
| **Auth** | Authenticated TUTEE; **must own the post**. |
| **Body** | the chosen tutor-account identifier — tutor Id is a **unique identifier string** (every PK/FK in this codebase is unique identifier) |
| **Response** | 201 (or 200 on update) Selection Request Response application identifier, tuition-request identifier, tutor identifier, status and creation time (all id fields unique identifier strings) |
| **Errors** | 401 ; 403 FORBIDDEN IDOR; 404 POST NOT FOUND; 422 TUTEE POST NOT ACTIVE (cannot choose on a non-ACTIVE job); 422 TUTOR NOT APPLICABLE if the chosen tutor isn't a valid candidate. |

Behaviour:
- **Mutable until admin confirms** — if a CREATED selection request already exists for this job,
  **update records** it (re-point tutor user id, refresh updated at) and return 200; else append records and
  return 201. Enforced by the partial unique index (§2.5).
- **Never creates a Match** (AC-04 admin-only matching). It only enqueues an admin signal.
- Links job application id if the chosen tutor has an application on this job (so admin/assign can
  resolve the winning app); null otherwise.
- Owner-scope check identical to §5.1 (403 IDOR if tutee Post tutee User Id differs from caller).

Api Routes additions (constants, no literals): TUTEE POSTS APPLICANTS, TUTEE POSTS read fields.

### 5.3 Assignment-record projection (profile history) — item F

The tutor-profile "Assignments" and the parent's history are a **projection over Match** (no new
table — design §2.3). A read data transfer object Assignment Record Response exposing **subject / level / period
(start = match created At or activated At, until = completed At/null) / status (ACTIVE|COMPLETED)
ONLY — never the counterparty identity** (no parent name, no child name, no tutor real name when shown
publicly). Surfaced on the public tutor profile.

> **CONFIRMED:** Match has matched At, created At, updated At but **no match Activated At and no
> completed At** (design doc was wrong). So period start = matched At; there is no clean "until"
> timestamp. **OPEN (backend) #9** — add a match completed at column (set when commission→PAID/match→
> COMPLETED) for an accurate period "until", or accept updated At-at-COMPLETE as a proxy. Recommend
> adding the column (small, in the same M18 pull request).

---

## 6. personal information contract & contract test (item F)

Binding (design §5a):

1. **Applicant data transfer object is a DISTINCT personal information-free type** (Applicant Card Response, §5.1) — NOT
   Admin Application Response. It physically cannot carry name/email/phone/exact-location (no such
   fields).
2. **Contract test — mirror the application component Tutor Response Pii Contract Test** (CONFIRMED
   path: Showcase Tutor Response Pii Contract Test implementation reference; sibling
   Showcase Request Response Pii Contract Test exists too). Those reflect over the data transfer object's
   the get declared fields operation and assert no field name (normalised) matches a personal information denylist. New test
   Applicant Card Response Pii Contract Test (in the application component) applies the **same reflection
   check** to Applicant Card Response class, with a denylist extended to also catch the finer-grained
   location leak this data transfer object must avoid: name, fullname, displayname, email, phone, contact,
   address, planningarea, mrt, mrtstation, exactlocation. Pure (no Spring context); fails
   loudly if a personal information field is added later.
3. **Owner-authorization (IDOR):** applicant-list + choose-tutor are owner-scoped (403 otherwise).
4. **Assignment record** shows subject/level/period/status only — never counterparty identity.
5. **Child name + parent phone hidden from applicants until commit** — enforced on tutor-facing
   match/interview views (contact shared only after assignment, counterparty rule).

---

## 7. Money path — REUSE existing commission flow (item G)

**No new payment gate.** The "pay after first session" real-world step maps onto the existing
product requirements document §8.3 / AC-05 / BR-010 commission state machine:
- assign → match PENDING, commission PENDING.
- admin activates → match ACTIVE, commission AWAITING CONFIRMATION (existing).
- first-session confirmed (tutee or admin override) → commission CONFIRMED → INVOICED → **PAID**;
  marking **PAID** sets the match **COMPLETED** (existing M5 path — unchanged).

This spec adds only the **observability** around stuck PENDING/unpaid commissions (§8), NOT a new
gate. **VERIFY-LINE** confirm the existing PAID→COMPLETED wiring in the commission/match service so
the assign lifecycle terminates correctly.

---

## 8. Observability (item H)

### 8.1 Metrics (Micrometer → Prometheus)

| Metric | Type | When |
|---|---|---|
| applications superseded total | Counter | incremented by the affected-row count of the supersede update records per assign |
| tutor selections pending | Gauge | live count of tutor selection requests when status is CREATED (admin backlog) |
| assignments total | Counter | per successful assign (job→MATCHED) |
| commissions pending age | Gauge (max age, hours/days) | oldest commission stuck in PENDING/AWAITING CONFIRMATION/CONFIRMED-not-PAID |

### 8.2 Alert (mandatory — revenue-leak safety net)

commissions pending age THRESHOLD → alert. This is the **only** safety net for "parent paid /
admin forgot to mark PAID" because pay-after-first-session is a manual admin action with **no system
deadline** (design §10/§11). Threshold value: **OPEN (backend) #10** — propose 14 days (matches the §8
invoice-due window) but Ops/Seb to confirm.

### 8.3 Analytics events

Emit via the existing analytics sink (product requirements document §16 / design §10):
application superseded, tutor selected, job assigned, commission paid, and the
listing Attached boolean prop on the existing application submitted event (§3.3).
**VERIFY-LINE** confirm product requirements document §16.4 event-name conventions (e g. whether job assigned should align
with an existing match created/match closed name) — reuse existing names where one already covers
the transition. **OPEN (backend) #11.**

### 8.4 Admin stale-PENDING-commission view

A read endpoint/query backing the admin "stuck revenue" list (design §10, product requirements document §20.8): commissions in
non-terminal states ordered by age desc, with the match/job/tutor refs the admin needs to chase.
read operation for admin commissions stale?older Than Days=… (admin-only) returning a paged projection.
**OPEN (backend) #12** — confirm whether this is a new endpoint or a filter on the existing commission
ledger list (product requirements document §8.5). Recommend: a status/age filter on the existing ledger to avoid a parallel
surface.

---

## 9. Bug #3 — email delivery (item I) — ⚠️ SUPERSEDED by email-resend-spec md

> **SUPERSEDED (2026-05-31).** The Mailhog plan below is OBSOLETE. Boss audit proved **no email
> consumer exists in this repo** — Email Queue Service (infrastructure/, not service/) is a
> producer only; the Email Consumer Service its Javadoc references is **absent**; there is no
> spring-boot-starter-mail/Java Mail Sender/spring mail.* anywhere. So **no email is sent in any
> profile** — Mailhog cannot fix a missing consumer. Founder decision: integrate **Resend**
> (dev-only key). The full fix is in **architecture/email-resend-spec md** — implement that, not the
> Mailhog options below (retained only as the audit trail).

Backend reset logic is **already correct** (Auth Service forgot Password value (withheld) does
the after commit operation). The bug,
CORRECTED against the real code: **email is NOT SMTP/Java Mail Sender** — it is a **Redis-stream queue**
consumed by an out-of-process worker, gated by email delivery.enabled. In application-local implementation reference
this is **email delivery.enabled is no** today, so the enqueue operation is a **no-op that just logs** and the
one-time passcode is never delivered. That is the actual cause of "reset only old password works": the new password
IS set, but the one-time passcode email never arrives so the dev can't complete the visible flow.

> **This invalidates the design doc's "Mailhog SMTP via spring mail.*" framing.** There is no
> Java Mail Sender to point at Mailhog. Two viable fixes — pick one at review:

**Option A (recommended, smallest, matches existing arch): turn the local queue ON + run the worker
against Mailhog.**
- application-local implementation reference: set **email delivery.enabled is yes** (so the enqueue operation writes to the
  Redis stream locally).
- **docker-compose (dev/local only):** add mailhog (mailhog/mailhog, 1025 SMTP / 8025 user interface) AND
  ensure the email-worker container runs locally with its SMTP host/port pointed at mailhog.
  **NEVER** in the prod compose/profile.
- Net effect: forgot Password → Redis → worker → Mailhog → visible at http://localhost.
- Risk: requires the worker to run locally (it may not today). **OPEN (backend) #13** — confirm the worker
  is runnable in the local stack; if not, Option B.

**Option B (no worker dependency): a the profile operation direct sender.**
- Add a local-only Java Mail Sender bean + a local Email Queue Service variant (or a local consumer)
  that, under the profile operation, **sends directly to Mailhog SMTP** instead of pushing to Redis —
  using spring mail.host is localhost, spring mail.port is 1025, …smtp auth is no,
  …starttls enable is no. The prod profile keeps the Redis-stream path untouched.
- Cleaner local user experience (no worker), but adds a profile-specific code path that needs its own test.

**Guard rail (both options):** prod must keep the Redis-stream + real provider (product requirements document §14.3); add a
startup assert/test that email delivery never targets localhost/mailhog under the prod profile.
**Mailhog NEVER in prod.**

---

## 10. Test impact (item J)

**Retire:** Guard Negatives IT apply Without Active Listing is403 (CONFIRMED:
Guard Negatives IT implementation reference). Delete or replace with the
positive free-market apply (case 1).

**New IT cases (REST Assured / Testcontainers):**
1. **Positive free-market apply → 201** — tutor with a complete profile, **no published listing**, applies → APPLIED.
2. **Profile-incomplete apply → 422 TUTOR PROFILE INCOMPLETE** (each missing field is a branch).
3. **Assign:** winner app → MATCHED, match source application id set; all other apps → SUPERSEDED; job → MATCHED.
4. **Apply-after-assigned → 422 TUTEE POST NOT ACTIVE.**
5. **Cancel → reopen:** job MATCHED→ACTIVE, superseded apps →APPLIED, winner MATCHED→APPLIED, commission CANCELLED.
6. **choose-tutor happy** (201 create, 200 update on re-choose) **+ IDOR-403** (non-owner select).
7. **Applicant-list owner-scope** (owner 200, non-owner 403) **+ personal information-shape contract test** (Applicant Card Response Pii Contract Test).
8. **Assign with no winning app** (2-panel path): all APPLIED/SHORTLISTED → SUPERSEDED, source application id null.
9. **After-commit notification isolation:** one recipient enqueue fails → assignment still committed, others still notified (mock the enqueue to throw for one recipient).

**Unit test (not black-box):**
10. **Concurrency double-assign → 409 JOB ALREADY ASSIGNED** — simulate stale @Version /
    Object Optimistic Locking Failure Exception → asserts 409, not a double-match.

**Coverage:** keep assert Profile Complete To Apply and the assign side-effects pure/branch-clean to hit
the **≥97.5% line AND branch** gate without exclusions. Set-based update methods: assert the returned
affected-row count in tests so the metric wiring is covered.

---

## 11. OPEN (backend) items — ⚠️ SUPERSEDED by §12

> **All items below are RESOLVED in §12 "FOUNDER DECISIONS RESOLVED."** This table is historical —
> do NOT implement its recommendations; use §12. (Item #13 email is further superseded by
> email-resend-spec md.)

Remaining genuinely-open items (the rest were RESOLVED by direct code reads — see below):

| # | Item | Recommendation / status |
|---|---|---|
| 3 | Single-open-per-job selection request vs multiple pending | **OPEN** — recommend single open (mutable) |
| 5 | Exact apply profile-gate error code/status | **OPEN (naming)** — propose TUTOR PROFILE INCOMPLETE / 422 |
| 6 | Notification template ids (N-xx) for superseded/assigned/parent-confirmed | **OPEN** — allocate new N-ids; superseded needs a NEW template; winner/parent may reuse N-08/N-09 |
| 7 | subjects/levels source when no listing attached (profile has no such cols) | **OPEN** — fallback to most-recent Tutor Post; else empty arrays |
| 8 | Pseudonymous handle scheme | **OPEN** — stable, non-reversible, collision-free slug from user id |
| 9 | Match period "until" timestamp | **OPEN** — Match has matched At only (no match Activated At/completed At); add match completed at (recommended) or proxy via updated At |
| 10 | commissions pending age alert threshold | **OPEN** — propose 14 days (invoice-due window); Ops to confirm |
| 11 | Analytics event-name alignment with §16.4 | **OPEN** — reuse existing names where one covers the transition |
| 12 | Stale-commission view: new endpoint vs ledger filter | **OPEN** — recommend filter on existing /admin/commission ledger (product requirements document §8.5) |
| 13 | Mailhog wiring (Redis-worker vs the profile operation direct SMTP) | **OPEN** — Option A if local email-worker is runnable, else Option B (§9) |
| 14 | TUTOR NO ACTIVE POST browse guard (Tutee Post Service) | **OPEN** — free-market implies removing it too; confirm with product (outside strict §20 scope) |

**RESOLVED by direct code read (no longer open):**
- ~~#1 most-recent ordering~~ — tutor posts created at CONFIRMED present → ordered by created at DESC.
- ~~#2 pre-existing @Version~~ — CONFIRMED none on tutee posts; changeset 0028c required.
- ~~#4 duplicate-apply 409~~ — WRONG assumption: the apply operation is **idempotent** (returns existing APPLIED /
  reactivates WITHDRAWN), backed by the uniqueness rule from migration 0018 for each request–tutor pair whose application has not been withdrawn. There is **no ALREADY APPLIED 409**.
- prior "Email Queue Service provider" — RESOLVED: email is a **Redis-stream queue + out-of-process worker
  → SES**, gated by email delivery.enabled (NOT Java Mail Sender). This rewrote §9.

## 12. FOUNDER DECISIONS RESOLVED (2026-05-31) — supersedes the OPEN table above

These resolve every OPEN (backend) item; the table above is historical. Binding for implementation.

### 12.0 Team-review conditions (2026-05-31, full debate + Boss) — BINDING

| # | Condition | Where |
|---|---|---|
| TR1 | **§2.5 index:** delete the broken unique index definition; use ONLY the partial-unique when status is CREATED. | applied in §2.5 |
| TR2 | **Bulk supersede update records** must use the modifying operation (stale-context bug otherwise — same lesson as update Password Hash). | §4 |
| TR3 | **No-application assign branch:** when the admin assigns a tutor who never applied (source application id null), there is no app row to flip → handle explicitly, no missing-value failure; add a test. | §4 |
| TR4 | **handle personal information-leak validation = MUST** (not SHOULD): reject values matching an email/phone pattern + length cap, server-side. | §3/§12 #8 |
| TR5 | **Email = Resend, not Mailhog** — no consumer exists; see email-resend-spec md. | §9 superseded |
| TR6 | **Lock order:** post (@Version) first, then applications bulk-update records, then match insert — consistent so single-row withdraw vs bulk supersede don't deadlock. | §4 |
| TR7 | Observability: add notification failures grouped by message template metric + an INFO log line carrying superseded Count on each assign. | §8 |
| TR8 | Analytics: add had application: bool to the job assigned event (marketplace-use vs cold admin assign). | §8 |
| TR9 | Endpoints: batch the rating lookup in GET applicants (no N+1); define re-select response = **200** (same-row update), not 201. | §5 |
| TR10 | Carry to UIUX/frontend: surface the profile-completeness requirement BEFORE the apply button (don't bounce at apply with an error). | UIUX spec |


| # | Item | RESOLUTION |
|---|---|---|
| 8 | Public handle scheme | **ADD tutor profiles public handle** (founder). New nullable→required varchar using 40 column, set at profile setup; cards/applicant-view/showcase render handle ONLY — real display Name stays admin/self. Changeset **0026** (alongside the other profile cols). Backfill (changeset 0027): derive a provisional handle from display Name (e g. first name + last initial, e g. "Wei T.") for existing tutors; mark profile incomplete so they're prompted to confirm/edit on next login. **Validation = MUST (reconciled with TR4, overrides the earlier "SHOULD"):** server-side reject values matching SG phone an eight-digit Singapore mobile number beginning with 8 or 9, email, or URL patterns; length ≤40; profanity screen; **uniqueness = MUST** (case-insensitive) so a handle is a stable, non-colliding public label. The personal information model is only as strong as this field's validation, so it is not optional. |
| 14 | Remove the browse-jobs listing guard too | **YES — remove it** (founder). Delete the TUTOR NO ACTIVE POST guard at Tutee Post Service as well as the apply guard. Free-market = any authenticated tutor can BROWSE and APPLY to active jobs with no listing. Retire any IT asserting the browse guard. |
| 13 | Mailhog worker wiring | **Mailhog + enable local queue** (founder). docker-compose mailhog service (dev only: SMTP 1025, user interface 8025); application-local implementation reference sets email delivery.enabled is yes + spring mail host is localhost port is 1025; the existing Redis-stream worker must run locally and deliver to Mailhog. NEVER enabled in prod profile. Reset one-time passcode (N-05) then visibly arrives at 8025. |
| 7 | Subjects/levels on applicant card when no listing attached | The free-market applicant has no listing, so subjects/levels are NOT a profile concept yet. **v1: the applicant card shows the tutor's profile fields (handle, rate, area, YOE, PT/FT, rating) + the SUBJECT/LEVEL OF THE JOB THEY APPLIED TO** (always known — it's the tutee Post). If a listing IS attached, also show that listing's subjects as "also teaches". No new profile subject columns in v1. |
| 5 | Profile-completeness error code | TUTOR PROFILE INCOMPLETE (422). Completeness = public handle + commitment + rate range + area + qualifications all present. |
| 3 | Selection request uniqueness | Per (job, tutor); partial-unique index on status is CREATED. One open selection per job at a time enforced at admin-confirm, not at request (parent may change choice → supersede prior CREATED). |
| 6 | Notification template IDs | N-14 (you've been selected → tutor), N-15 (assignment confirmed → tutee+tutor), N-16 (no longer available → tutor) — matches product requirements document §9. |
| 9 | match completed at column | Add it (changeset 0028d), set when commission→PAID/match→COMPLETED; feeds the assignment-record "until" period + days active analytics. |
| 10 | Stale-commission threshold | **14 days** (matches the §8 confirmation window cadence); single source shared with the commissions pending age alert + UIUX stale view. |
| 11 | Analytics event names | Align to product requirements document §16.4: tutor selected, job assigned, application superseded, + listing attached bool on application submitted. |
| 12 | Stale-commission surface | **Filter on the existing admin commission list** (not a new endpoint): ?status is PENDING&older Than Days is 14. |

> **Cross-spec consistency:** the UIUX spec's "handle source" OPEN is resolved by #8 above (real
> handle field). The card therefore shows the handle, never display Name.

Remaining **VERIFY-LINE** anchors to re-confirm before editing (exact lines only — types/names CONFIRMED):
Job Application Service (guard to delete) / 54 (ERR TUTOR NO ACTIVE POST) / 87-90
(job-ACTIVE guard); Admin Match Service create Match / cancel Match / activate Match; the
commission PAID→match-COMPLETED wiring in the M5 commission service (confirm location before relying on
it); product requirements document §16.4 analytics event names.

---

## 13. UIUX-review deltas (2026-05-31) — backend-side items from the 10-reviewer round + Boss re-audit

Binding for backend implementation. Mirrored in the UIUX spec's "TEAM-REVIEW RESOLVED" addendum.

- **D1 — Applicant Card Response gains pitch + also Teaches** (applied in §5.1 JSON above).
  The pitch is the optional application message. Other subjects taught come only from the attached listing (a flat
  array; NO structured attached Listing object). Lead subject/level = the applied-to job's
  job Subject/job Level. Authoritative field names: tutor Public Handle, coarse Area, job Subject,
  job Level (not handle/area/subjects/levels).
- **D2 — My-Applications list data transfer object gains has Interview (bool) + interview Id (nullable unique identifier)**,
  precomputed server-side (single batch/join — NO N+1 per row), so the frontend "Interview scheduled"
  pseudo-status (a status badge that also knows whether an interview exists) has a source. Endpoint:
  Job Application Service list My Applications (163). Confirm the interview link key
  (tutee post id, tutor id or a direct application↔interview FK) before building.
- **D3 — Contact-reveal endpoint (NEW, fully specified):**

  | | |
  |---|---|
  | **Method/Path** | read operation for matches id contact |
  | **Auth** | Authenticated; caller **must be a participant** of the match (its tutor or its tutee). |
  | **Gate** | Match status ∈ PENDING-post-assign, ACTIVE, COMPLETED. **NOT** revealed while only a *selection request* exists (selection is mutable → harvest guard). |
  | **Response** | a successful response containing the authorised other participant’s name, phone and email — the OTHER party relative to caller (bidirectional: tutee sees tutor, tutor sees tutee). |
  | **Errors** | 401 ; 403 not a participant; 404 match not found **or** status not yet revealable **or** match CANCELLED (contact **revoked on cancel for BOTH sides**). |

  The ONLY surface that crosses the counterparty-personal information line, and only post-assignment. Own auth test:
  participant 200, non-participant 403, pre-assignment 404, post-cancel 404.
- **D4 — handle validation = MUST** (= §12 #8 + TR4, reconciled): reject SG-phone an eight-digit Singapore mobile number beginning with 8 or 9,
  email, URL; ≤40 chars; unique (case-insensitive); profanity screen — **server-side** (the form
  mirrors it; the server is the enforcement layer, not a client route guard).
- **D5 — tutor Id is a unique identifier** in submit operation for select (the 123 stub in §5.2 corrected to `<uuid>`).
- **D6 — "Reviewed by our team", NOT "Verified" (Boss trust-and-safety catch):** the parent user interface must
  NOT claim tutors are "verified" — there is **no identity-verification workflow** today (only M3 post
  approval + the profile-completeness gate). Expose at most a boolean from EXISTING signals (profile
  approved / ≥1 completed assignment) labelled "Reviewed". **Do NOT invent a verification workflow
  for v1**; a stronger claim is a separate scoped feature (KYC/cost/ops), out of M18.
- **D7 — owner-auth is server-side (Casey):** /applicants and /select enforce ownership in the
  service/controller (@Pre Authorize + explicit tutee Post tutee User Id equals caller), not a frontend route
  guard. The 403 IDOR test asserts the application programming interface rejects independent of any client.
