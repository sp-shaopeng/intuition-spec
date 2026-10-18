# INT-044 — Tutee → Tutor Job Invitations — Feature Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** [021](../021-tutee-request-flow/spec.md) permits PENDING initiation/editing and requires rejection cleanup; acceptance before approval remains explicitly pending there. [024](../024-postfix-review-fixes/spec.md) uses Handle OR tutor Post Id resolved by the backend. [INT-062](int-062-cascade-close-spec.md) distinguishes EXPIRED-on-assignment from CLOSED-on-withdrawal and removes invitation timeout. In-app notifications are in-tx under INT-125. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Issue:** INT-044 · **Type:** FEATURE (M) · **Layer:** Full-stack (backend + frontend)
**Status:** Draft for team review · **Date:** 2026-06-03
**Lead:** Solution Architect · **Contributors:** frontend-facing service Specialist, DBA, user interface/user experience Designer
**Source of truth:** the service code (founder decree). Every claim below was read against the live
backend; file:line cited for each touched file. Liquibase for ALL schema (no raw database structure definitions).

> **Addendum (INT-077, 2026-06-06):** invitations now carry a **per-tutee post cap of 10
> active (PENDING) invites** (config quota invites-per-post), enforced race-safely in
> the invite operation (advisory lock jobinv:`<tutee Post Id>`); over-cap → 422
> ERR INVITE LIMIT REACHED. Every invite/withdraw/applicants response now returns a
> the quota limit and remaining allowance, and a new read operation for tutee posts id invite-quota
> powers the pre-click chip. Full design:
> [int-077-marketplace-quota-caps md](int-077-marketplace-quota-caps.md).

> **Conventions (binding, from CLAUDE md):** Java 21; Liquibase for all schema; **no magic strings**
> (backend application programming interface paths → the application component Routes constants, enforced by No Hardcoded Routes Arch Test;
> frontend paths → application programming interface ROUTES in api Routes implementation reference; enum values → typed
> enums/constants on frontend and backend); **soft-delete only** (never hard-delete a row — status/flag);
> coverage gate **≥97.5% line AND branch (backend)**, **≥97.5% line / ≥88% branch (frontend)**; error envelope is
> nested a structured error containing its error code raised via App Exception factories (bad Request 400 /
> unauthorized 401 / forbidden 403 / not Found 404 / conflict 409 / unprocessable Entity 422).

> **Ground truth read in full before writing:** Job Application / Job Application Status /
> Job Application Repository / Job Application Service ;  Tutor Selection Request /
> Tutor Selection Request Status / Tutor Selection Service / Admin Selection Request Controller /
> Tutee Application Controller ;  Notification Type (+ changeset 0012); changesets 0018, 0025–0031;
> Api Routes and frontend application programming interface ROUTES;  Tutee Post / post statuses; see the assignment backend specification
> (M18) for format and the assign/cancel lifecycle this feature must interlock with.

---

## 1. Problem

The marketplace today is **tutor-initiated**. The only two directed paths into a job are:

1. **Apply (M15/M18)** — a *tutor* finds an ACTIVE job in their **Browse Requests** tab
   (read operation for requests, authenticated) and applies. This creates a durable
   Job Application (Job Application Service apply, Job Application Service implementation reference).
2. **Choose-tutor / selection request (M18)** — a *tutee* who already has applicants picks one via
   submit operation for tutee posts id select. This is constrained to tutors who **already applied** (the
   service links job application id via find Open Application, Tutor Selection Service implementation reference); it
   is a parent→admin signal, never a match.

There is **no way for a tutee to proactively reach a specific tutor**. A parent who browses tutors
(public tutor list / authenticated tutor detail) and finds someone they like has **no button** to say
"I'd like *you* on *my* job." Conversely a tutor has **no surface** to see jobs that were brought to
them — their Browse Requests tab is the undifferentiated open marketplace.

This is a real funnel gap: the highest-intent demand signal (a parent who picked a specific tutor) is
unrepresented, so it cannot be measured, surfaced, or converted. INT-044 closes it with a **job
invitation**: a tutee invites their own ACTIVE or PENDING job to a chosen tutor; the tutor sees a dedicated
"Invitations" surface and can **accept** (→ becomes a normal application, entering the existing assign
flow) or **decline**.

### 1.1 Goals / non-goals

| | |
|---|---|
| **Goal** | A tutee can invite an ACTIVE or PENDING job they OWN to a specific tutor from the tutor detail/browse surface. |
| **Goal** | A tutor sees invitations distinct from open Browse Requests, and can accept/decline. |
| **Goal** | Accept folds cleanly into the existing application + selection-request + assign machinery — **no parallel matching path**. |
| **Goal** | Every state is a durable, soft-deletable record (audit/analytics/undo). |
| **Non-goal** | Invitations do **NOT** create matches (AC-04 admin-only matching holds, unchanged). |
| **Non-goal** | No personal information leak — neither side sees the other's real name/contact pre-assignment (counterparty rule). |
| **Non-goal** | No bulk/blast invite (one tutor per invite); no tutor→tutee "I want this job beyond apply" (that *is* apply). |

---

## 2. Proposed design — **record-based invitation** (recommended)

### 2.1 Decision: a new job invitations table + status enum, NOT notification-only

**Recommended:** persist each invitation as a durable row (job invitations) with an explicit
lifecycle, mirroring how Job Application and Tutor Selection Request are modelled. A notification is a
**side-effect** of the record, not the record itself.

**Rationale (why record, not notification-only):**

1. **Accept/decline needs state.** A notification is fire-and-forget; an invitation has a lifecycle
   (PENDING → ACCEPTED / DECLINED / WITHDRAWN / EXPIRED). The tutor surface must list *pending*
   invitations and reflect terminal outcomes — that requires a queryable row, not a notification feed.
2. **Idempotency / dedupe.** A parent must not be able to spam the same tutor with N invites for the
   same job. A partial-unique index tutee post id, tutor user id when status PENDING (the
   exact pattern proven in 0018 uq job applications active pair and 0029
   ux selreq one open per job) makes "one open invite per (job, tutor)" a database invariant. Notifications
   cannot enforce this.
3. **Measurement (data-engineer).** The business value of INT-044 is the high-intent signal; that is
   only measurable if it is a row we can count (invites sent, acceptance rate, time-to-accept,
   invite→assign conversion). A notification leaves no analyzable trace.
4. **Soft-delete directive.** Withdraw/decline must preserve the record (status flip), never delete —
   consistent with the founder soft-delete rule. Notification-only has nothing to soft-delete.
5. **Consistency.** The codebase already has *two* near-identical signal tables (job applications,
   tutor selection requests) with the same unique identifier-PK + named/varchar-status + partial-unique-index +
   updated at trigger shape. A third follows the established, reviewed pattern — lowest-risk.

We also emit a **notification** to the tutor on invite (and to the tutee on accept/decline) — but the
notification's link path points at the invitation surface; the row is the source of truth.

### 2.2 Decision: dedicated tutor **"Invitations" tab**, NOT a flag inside Browse Requests

**Recommended:** a dedicated tutor surface (read operation for tutor invitations) rendered as its own tab,
distinct from Browse Requests.

**Rationale (uiux + bff):**

- **Different mental model.** Browse Requests = "jobs I can hunt"; Invitations = "jobs that chose me."
  Conflating them with a badge buries the high-signal item in a long open-market list and forces the
  tutor to scan. A distinct tab with a count badge surfaces intent immediately (conversion psychology:
  a personalised invite converts far better than a cold listing — give it its own stage).
- **Different actions.** Browse Requests cards have one action (Apply). Invitation cards have **two**
  (Accept / Decline) plus an expiry. Overloading one card with conditional actions raises cognitive
  load and the chance of mis-tap.
- **Different data shape.** An invitation carries an optional parent message and an expiry; the open
  request card does not. A separate data transfer object keeps both payloads lean (no over-fetch — bff).
- **Clean instrumentation.** A separate surface gives clean impression/click events per tab.

Browse Requests stays exactly as-is. Cross-link: an invited job that is *also* visible in Browse
Requests shows a small "Invited" pill so the tutor isn't confused if they encounter it in both places
(read-only hint; the action lives on the Invitations tab).

### 2.3 Decision: accept = create/reactivate an application (fold into existing flow)

**On accept**, the invitation does **not** introduce a new matching mechanism. It calls the **existing**
the apply operation semantics for tutee Post Id, tutor Id:

- If no application exists → create one (status APPLIED).
- If a WITHDRAWN application exists → reactivate to APPLIED (existing withdraw→re-apply path,
  Job Application Service implementation reference).
- If an APPLIED application already exists → idempotent, reuse it.

The accept also **auto-creates/updates the tutee's selection request** for that job pointing at this
tutor (calling the existing the choose tutor operation path), because the tutee already
expressed the intent "I want this tutor" by inviting them. This means an accepted invitation lands the
job in the **admin selection-request queue** with job application id populated — i e. it converges on
the exact admin-driven assign flow M18 already ships (Admin Match Service create Match), with **zero new
assign logic**.

> **Design note (architect):** This is the load-bearing decision. By making accept a thin orchestration
> over the apply operation + the choose tutor operation, INT-044 adds a *front door* to the funnel without forking the
> conversion machinery. The invitation table is a **signal/state layer**; applications and selection
> requests remain the **canonical funnel**. No new path to a Match.

### 2.4 Interaction matrix with existing flows

| Scenario | Behaviour |
|---|---|
| Tutee invites a tutor who already applied | Allowed. Invite row is PENDING; the tutor sees "you applied + you're invited". Accept is a no-op on the application (idempotent) but flips invite → ACCEPTED and (re)points the selection request. |
| Tutor declines | Invite → DECLINED (terminal, soft). The tutor's application (if any) is untouched — declining the invite ≠ withdrawing an application. Tutee notified. |
| Tutor accepts | Invite → ACCEPTED; application created/reactivated; selection request created/updated → job enters admin queue. |
| Tutee withdraws invite before action | Invite → WITHDRAWN (terminal, soft). No notification spam beyond a single "invite withdrawn" (optional). |
| Job becomes non-ACTIVE (MATCHED / CLOSED / EXPIRED) | Pending invites become EXPIRED when the job is assigned elsewhere, CLOSED when intake is withdrawn/closed/rejected under INT-062/021. There is no timer expiry. Pre-approval acceptance remains pending the user decision; retain the existing ACTIVE acceptance guard until amended. |
| Admin assigns the job to *another* tutor | Job → MATCHED triggers the PENDING-invite → EXPIRED sweep (§6.3). |
| Tutee re-invites the same tutor after a DECLINE/WITHDRAW | Allowed — the partial-unique index only forbids a second *PENDING* row, so a new invite after a terminal one succeeds (same cycle as withdraw→re-apply). |
| Tutee invites the same tutor twice while one is PENDING | Idempotent: returns the existing PENDING invite (200), no duplicate (mirrors the apply operation idempotency). |
| Tutee invites a tutor for a job they don't own | 403 IDOR (owner-scope, server-side). |

---

## 3. Domain model — target state

### 3.1 New enum Job Invitation Status

New file Job Invitation Status implementation reference:


Implementation example omitted. This public edition retains the written requirements in this section.


PENDING is the only mutable/open state. ACCEPTED|DECLINED|WITHDRAWN|EXPIRED|CLOSED are terminal.

**Storage decision (DBA):** store as **the varchar operation + the enumerated operation**, NOT a PG named
enum. Rationale: Tutor Selection Request Status (0029) already set this precedent for new
signal-tables, and it avoids the ALTER TYPE ... ADD VALUE transaction friction (documented as a real
hazard in assignment-be-spec md §1.2 and in Job Application Repository implementation reference) when future
statuses are added. The two older status columns are PG enums only for historical reasons; new
tables use varchar.

### 3.2 New entity Job Invitation

Job Invitation implementation reference — shape mirrors
Tutor Selection Request (scalar unique identifier columns, not @Many To One, to match the M18 convention and keep
the assign/sweep set-based queries simple):

| Field | Column | Type | Notes |
|---|---|---|---|
| id | id | unique identifier PK | the gen random uuid operation default |
| tutee Post Id | tutee post id | unique identifier, required | the invited job (FK → tutee posts) |
| tutor User Id | tutor user id | unique identifier, required | the invited tutor (FK → users) |
| created By | created by | unique identifier, required | the inviting tutee user ( equals  post owner) |
| message | message | varchar using 500, nullable | optional parent note to the tutor; **text only, no personal information** (contract-tested) |
| status | status | varchar using 16, required, default PENDING | the enumerated operation |
| job Application Id | job application id | unique identifier, nullable | set on ACCEPT to the resulting application (FK → job applications, ON DELETE SET no value) |
| responded At | responded at | timestamptz, nullable | set when tutor accepts/declines |
| created At | created at | timestamptz, required | the now operation default |
| updated At | updated at | timestamptz, required | maintained by shared the set updated at operation trigger |

### 3.3 Notification Type — add JOB INVITATION RECEIVED (+ optional tutee-side)

Notification Type (Notification Type implementation reference) is a **PG named enum** notification type (changeset
0012). Add **JOB INVITATION RECEIVED** (tutor-side, on invite) and **JOB INVITATION RESPONDED**
(tutee-side, on accept/decline). This is a **two-part change** (Java enum + PG ADD VALUE), and the
ADD VALUE MUST be run In Transaction is no in its own changeset, per the documented PG hazard
(see 0013 precedent which added INTERVIEW COMPLETED, and assignment-be-spec md §1.2). See §4.2.

> Reuse, don't reinvent: the existing M9 notification machinery (notifications table, link path,
> unread count) carries these — no new delivery mechanism.

---

## 4. Schema (Liquibase) — **changeset 0035**

Changelog head is **0031** (0031-allow-post-soft-delete-close implementation reference); **0032 is reserved for M19**,
so INT-044 is assigned **0035**. New file
0035-job-invitations implementation reference, included from
db changelog-master implementation reference in order. All IDs unique identifier; timestamps timestamptz.

### 4.1 0035 — job invitations table + indexes + trigger


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.2 Notification enum value (separate changeset, run In Transaction is no)

Because notification type is a PG enum, the two new labels are added in their **own** changeset that
is NOT in the same transaction as any code using them. This can ride in the same 0035 file or a
sibling; keep the ADD VALUE changeset isolated:


Implementation example omitted. This public edition retains the written requirements in this section.


> **DBA note:** No @Version is needed on job invitations — accept races are resolved by the
> partial-unique PENDING index plus the application-layer idempotency of the apply operation. The optimistic lock
> that matters (double-assign) lives on tutee posts (M18, 0028c) and is untouched here.

---

## 5. application programming interface contracts

All backend paths via **the application component Routes** constants; all frontend paths via application programming interface ROUTES. Error
envelope nested a structured error containing its error code. Pagination via the existing a page of results convention
(page 0-based, size default 20).

### 5.1 Api Routes additions (backend)


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.2 application programming interface ROUTES additions (frontend, api Routes implementation reference)


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.3 submit operation for tutee posts id invite — tutee invites a tutor

| | |
|---|---|
| **Auth** | Authenticated **TUTEE**; **must own the post** (owner-scope, server-side, 403 IDOR). |
| **Path** | the requested record's identifier = tutee post id |
| **Body** | Target identifier is Handle OR tutor Post Id per024 (not a frontend-required private user unique identifier); optional message≤500 chars. backend resolves the tutor and preserves404 privacy/self-invite guards. |
| **Response** | 201 Job Invitation Response on create; 200 on idempotent re-invite (existing PENDING returned). |
| **Errors** | 401 ; 403 FORBIDDEN (not owner); 404 POST NOT FOUND; 404 TUTOR NOT FOUND (tutor Id not a TUTOR user); 422 JOB NOT ACTIVE (post neither ACTIVE nor PENDING for initiation); 422 CANNOT INVITE SELF if tutor Id equals caller (defensive). |

Behaviour: load post → owner check → ACTIVE/PENDING initiation check → resolve Handle/tutor Post Id to tutor (must be a TUTOR-role, non-deleted
user) → idempotent lookup of an existing PENDING invite for post Id, tutor Id; if present return it
(200), else insert PENDING (201). After commit, emit JOB INVITATION RECEIVED notification to the
tutor (link path → tutor invitations tab) and an invitation sent analytics event. Validation: a
invalid public handle/listing target → 404 TUTOR NOT FOUND (do not leak whether it's a real
non-tutor user).

**Job Invitation Response (shared data transfer object, personal information-safe):**


The post tutee posts interface operation id /invite — tutee invites a tutor example describes information including id, tutee Post Id, job Subject, job Level, tutor User Id, tutor Public Handle, message, status, job Application Id, created At, responded At. Concrete payload values and implementation syntax are omitted.


### 5.4 read operation for tutor invitations — tutor's received invitations

| | |
|---|---|
| **Auth** | Authenticated **TUTOR**. |
| **Query** | status (default PENDING; also accepts ALL), page, size, sort created At,desc. |
| **Response** | 200 a page of tutor invitation cards. |

**Tutor Invitation Card Response (personal information-safe — the job, not the family):**


The get tutor invitations interface operation — tutors received invitations example describes information including id, tutee Post Id, job Subject, job Level, coarse Area, budget Min Sgd, budget Max Sgd, message, status, already Applied, created At. Concrete payload values and implementation syntax are omitted.


> **personal information (security + bff):** the tutee/parent is **never** named to the tutor pre-assignment. Only the
> anonymised job (subject/level/coarse region/budget) + optional message. Same counterparty rule M14b
> applies to anonymised job showcases. coarse Area derives from location region, never
> location planning area/location mrt station (those are finer-grained personal information). Batch-resolve job
> fields + already Applied (one query over job applications) — **no N+1**.

### 5.5 submit operation for tutor invitations id accept — tutor accepts

| | |
|---|---|
| **Auth** | Authenticated **TUTOR**; must be the **invited tutor** (else 404 — do not leak existence). |
| **Response** | 200 Job Invitation Response (status ACCEPTED, job Application Id populated). |
| **Errors** | 401 ; 404 INVITATION NOT FOUND (missing or not addressed to caller); 409 INVITATION NOT PENDING (already ACCEPTED/DECLINED/WITHDRAWN/EXPIRED); 422 JOB NOT ACTIVE (job no longer ACTIVE); 422 TUTOR PROFILE INCOMPLETE (the existing apply gate, reused). |

Behaviour (single a database transaction):
1. Load invite addressed to caller; assert PENDING (409 otherwise); assert job ACTIVE (422).
2. **Reuse apply to the tuition request as that tutor with no message** — create/reactivate
   the application, including the existing assert Profile Complete To Apply gate (so an incomplete-profile
   tutor gets the same 422 they'd get applying normally, and the frontend routes them to profile edit).
3. Set invite job Application Id application id, status ACCEPTED, record the current response time.
4. **Auto-point the selection request:** call Tutor Selection Service choose Tutor(tutee Post Id,
   post tutee User Id, tutor Id) on behalf of the owner so the accepted invite lands in the admin
   selection queue (the tutee already signalled intent by inviting). Implemented by extracting the
   post-owner-id-scoped core of choose Tutor so the accept path can invoke it without re-checking
   "caller owns post" (the accept is system-initiated on the owner's prior intent). **OPEN #2.**
5. Persist the ordinary JOB INVITATION RESPONDED notification in-tx; publish independently committed invitation accepted analytics after commit under the messaging playbook.

> The accept introduces **no new assign code** — it composes the apply operation + the choose tutor operation, both
> already covered + tested. AC-04 admin-only matching is preserved: the job still requires an admin to
> consume the selection request via Admin Match Service create Match.

### 5.6 submit operation for tutor invitations id decline — tutor declines

| | |
|---|---|
| **Auth** | Authenticated **TUTOR**; must be the invited tutor (else 404). |
| **Response** | 200 Job Invitation Response (status DECLINED). |
| **Errors** | 401 ; 404 INVITATION NOT FOUND; 409 INVITATION NOT PENDING. |

Behaviour: assert PENDING → status DECLINED, record the current response time. **Does not touch any application**
(decline ≠ withdraw). After commit: JOB INVITATION RESPONDED to the tutee; invitation declined
event.

### 5.7 delete operation for tutee invitations id — tutee withdraws

| | |
|---|---|
| **Auth** | Authenticated **TUTEE**; must be the **inviter** (created by equals caller, else 404). |
| **Response** | a successful result. |
| **Errors** | 401 ; 404 INVITATION NOT FOUND; idempotent — withdrawing an already-terminal invite returns 200 (no-op), matching the the withdraw operation application precedent. |

Behaviour: if PENDING → status WITHDRAWN, record the current response time (soft-delete via status). No
notification needed for the tutor on withdraw (avoid noise); emit invitation withdrawn event.

---

## 6. Service & lifecycle wiring

### 6.1 New Job Invitation Service

Holds: invite, list For Tutor, accept, decline, withdraw. Injects
Job Invitation Repository, Job Application Service, Tutor Selection Service, Tutee Post Repository,
User Repository, Tutor Profile Repository (handles/labels), Notification Service (existing M9),
Analytics Event Publisher. All owner/recipient checks server-side; "not addressed to you" → 404 (no
existence leak, matching load Owned Application, Job Application Service implementation reference).

### 6.2 Job Invitation Repository


Implementation example omitted. This public edition retains the written requirements in this section.


Plus a set-based expiry sweep (§6.3). Follow the bound-parameter rule from
Job Application Repository implementation reference for any status IN :statuses query.

### 6.3 Expiry sweep on job exit-from-ACTIVE

When a job leaves ACTIVE — admin **assign** (job→MATCHED), admin **cancel** does NOT re-expire (cancel
reopens to ACTIVE; invites stay terminal as already swept), and **close/soft-delete-close** (0031) —
PENDING invites for that job must become EXPIRED so they vanish from tutors' tabs. A single set-based
update records (never a read-loop), invoked **inside** the existing transactions:


Implementation example omitted. This public edition retains the written requirements in this section.


Wire this call into the same place M18 supersedes applications on assign
(Admin Match Service create Match, see assignment-be-spec md §4.2) and into the close path (0031).
**OPEN #3** — confirm exact injection points against the live Admin Match Service.

> **No new scheduler in v1.** expiry period-based expiry (e g. invites older than N days auto-EXPIRE without a
> job state change) is **deferred** — the job-exit sweep covers the funnel-correctness cases. A nightly
> sweep is a fast-follow if invite staleness becomes a problem (measured via the age gauge, §8).

---

## 7. user experience (uiux-designer)

### 7.1 Where the tutee initiates

**Primary entry — tutor detail page** (the tutor's profile-detail page, authenticated per M17). Add an **"Invite to my
job"** primary-adjacent button alongside Favourite. Behaviour:

- Click → a modal listing the tutee's **own ACTIVE jobs** (subject · level chips) with a radio select,
  plus an optional message textarea (≤500 chars, live counter). If the tutee has **no** ACTIVE job →
  the modal shows an empty state with a "Post a job" CTA (don't dead-end).
- Submit → submit operation for tutee posts id invite. On 201/200 → toast "Invitation sent to handle" and the
  button flips to a disabled "Invited" pill for that (tutor, job) pair.
- On 422 JOB NOT ACTIVE → inline error; on 404 TUTOR NOT FOUND → toast generic error.

**Secondary entry — tutor browse list cards** (/tutors): a compact "Invite" affordance on each card
opening the same modal, for parents who triage from the list. (Mo S Co W: **Should**; can land as a
fast-follow after the detail-page entry which is **Must**.)

### 7.2 Tutor surface

**New "Invitations" tab** in the tutor area, sibling to **Browse Requests** and **My Applications**,
with an unread-count badge (count of PENDING). Each invitation card shows: subject · level (lead),
coarse area, budget range, the parent's optional message, "invited relative time", and an
already Applied pill when relevant. Two actions:

- **Accept** → submit operation for tutor invitations id accept. On success → toast "You accepted — the family has
  been notified", card moves to a collapsed "Accepted" state, and a hint links to **My Applications**
  (since accept created the application). On 422 TUTOR PROFILE INCOMPLETE → modal routing to profile
  edit (reuse the existing apply-gate user experience). On 409 INVITATION NOT PENDING (raced) → refresh the list.
- **Decline** → confirm dialog → submit operation for tutor invitations id decline. Card moves to "Declined".

**Cross-link (Browse Requests):** an open-market card whose job the tutor was invited to shows a small
read-only **"Invited"** pill linking to the Invitations tab — so the two surfaces never feel
contradictory.

**Empty states:** Invitations tab with none → "No invitations yet. Keep your profile sharp so parents
reach out." (links to profile).

### 7.3 Notifications

- Tutor on invite: in-app JOB INVITATION RECEIVED — "A family invited you to a subject level
  job." → Invitations tab.
- Tutee on accept/decline: JOB INVITATION RESPONDED — "handle accepted|declined your invitation."
  → applicant view (accept) / post detail (decline).

---

## 8. Observability & analytics (data-engineer)

Events via the existing Analytics Event Publisher (pattern: Job Application Service implementation reference):

| Event | Actor | Props |
|---|---|---|
| invitation sent | tutee | tutee Post Id, tutor User Id, had Prior Application, message Provided |
| invitation accepted | tutor | invitation Id, tutee Post Id, application Reactivated |
| invitation declined | tutor | invitation Id, tutee Post Id |
| invitation withdrawn | tutee | invitation Id, tutee Post Id |
| invitation expired | system | tutee Post Id, count (from sweep affected-rows) |

Metrics (Micrometer): invitations sent total, invitations accepted total,
invitations declined total, invitations pending (gauge), invitations pending age (gauge, max age
— feeds the future expiry period decision). **Headline funnel KPI:** invite→accept rate and invite→assign
conversion (join invite rows to consumed selection requests).

---

## 9. Acceptance criteria

1. **AC-1** A TUTEE can submit operation for tutee posts id invite for an ACTIVE job they own, creating a PENDING
   Job Invitation; the invited TUTOR receives a JOB INVITATION RECEIVED notification.
2. **AC-2** Inviting the same (job, tutor) twice while PENDING is **idempotent** (200, no second row);
   the partial-unique index enforces one PENDING per pair at the database level.
3. **AC-3** Inviting a job the caller does not own → **403**; inviting on a non-ACTIVE job → **422
   JOB NOT ACTIVE**; inviting a non-tutor/unknown id → **404 TUTOR NOT FOUND**.
4. **AC-4** A TUTOR sees only **their own** invitations via read operation for tutor invitations (default PENDING),
   personal information-free (no parent/child name, no fine location); list is paginated and N+1-free.
5. **AC-5** **Accept** flips the invite to ACCEPTED, creates **or reactivates** the tutor's application
   (idempotent with any existing one), populates job Application Id, and lands the job in the **admin
   selection-request queue** pointing at this tutor — with **no new match created** (AC-04 holds).
6. **AC-6** Accept on a non-ACTIVE job → **422**; accept by a profile-incomplete tutor → **422
   TUTOR PROFILE INCOMPLETE** (same gate as apply); accept of a non-PENDING invite → **409**; accept of
   an invite not addressed to the caller → **404**.
7. **AC-7** **Decline** flips to DECLINED and **does not** alter any application; the tutee is notified.
8. **AC-8** **Withdraw** (tutee) flips a PENDING invite to WITHDRAWN (soft; row preserved); idempotent
   on a terminal invite; non-inviter → 404.
9. **AC-9** When a job leaves ACTIVE (assigned / closed), all its PENDING invites become **EXPIRED**
   via a single set-based update records and disappear from tutors' tabs.
10. **AC-10** Re-inviting the same tutor **after** a DECLINE/WITHDRAW/EXPIRE succeeds (partial-unique
    only forbids a second PENDING).
11. **AC-11** No raw path literals (backend Api Routes, frontend application programming interface ROUTES); no hard-deletes (all terminal
    transitions are status flips); Job Invitation Response/Tutor Invitation Card Response carry no personal information
    (contract test).
12. **AC-12** Coverage gates hold: backend ≥97.5% line AND branch; frontend ≥97.5% line / ≥88% branch.

---

## 10. Test plan

### 10.1 Backend — unit (Job Invitation Service Test)
- invite: happy 201; idempotent 200 on existing PENDING; 403 non-owner; 404 post-not-found; 404
  tutor-not-found; 422 non-active job; 422 self-invite; message persisted; notification + event emitted.
- accept: happy (new application) + happy (reactivate WITHDRAWN application); idempotent over an
  existing APPLIED app; 404 wrong-tutor; 409 non-pending; 422 non-active; 422 profile-incomplete (gate
  delegated to apply); selection request created/updated; job Application Id set; tutee notified.
- decline: happy; 404 wrong-tutor; 409 non-pending; application untouched (assert no app row change).
- withdraw: happy; idempotent on terminal; 404 non-inviter.
- expiry sweep: PENDING→EXPIRED only; terminal rows untouched; affected-row count returned; called
  from assign + close (verify via service-level test or Admin Match Service test extension).
- Each status IN/enum branch as a discrete test (clean branch coverage — the gate is strict).

### 10.2 Backend — repository (Job Invitation Repository Test, Testcontainers)
- partial-unique: 2nd PENDING for same pair → constraint violation; PENDING + DECLINED for same pair OK
  (re-invite cycle); find First By… returns the open one.
- find By Tutor User Id And Status… ordering + paging; expire Pending For Job set-based correctness.

### 10.3 Backend — controller / integration
- @Pre Authorize role gates: TUTEE-only on invite/withdraw, TUTOR-only on list/accept/decline; 403 on
  wrong role (401-vs-403 lesson — authenticated wrong role MUST be 403/404, not 401).
- Full happy path IT: tutee invites → tutor accepts → job appears in admin selection queue → admin
  create Match succeeds (proves no new assign path and the funnel converges). Run against the live
  stack per the "regression ITs hit the live stack" lesson (reseed/redeploy first).

### 10.4 Backend — data transfer object contract
- Job Invitation Response Pii Contract Test + Tutor Invitation Card Response Pii Contract Test — reflect over
  fields, assert denylist (name, fullname, displayname, email, phone, contact, address,
  planningarea, mrt, mrtstation) absent (mirror Showcase Tutor Response Pii Contract Test).

### 10.5 Frontend — Vitest
- Invite modal: ACTIVE-jobs list, empty state → Post-a-job CTA, message counter, submit success flips
  to "Invited" pill, 422/404 error paths.
- Invitations tab: PENDING list render, accept (→ My Applications hint), decline (confirm), 409 race
  refresh, 422 profile-incomplete → profile-edit route, empty state.
- Cross-link "Invited" pill on Browse Requests cards.
- api Routes constants used (no literals); a11y on modal + tab (focus trap, aria-labels).

---

## 11. Open decisions

1. **OPEN #1 — self-serve vs admin-mediated accept.** Recommended: accept auto-creates the selection
   request so the job enters the admin queue (admin still assigns). Alternative: accept only creates
   the application and the tutee must then explicitly choose-tutor. Recommend the former (lower funnel
   friction; the invite already encodes the tutee's choice). **Confirm at review.**
2. **OPEN #2 — refactor for choose Tutor reuse.** Accept must (re)point the selection request *on the
   owner's behalf* without the caller-owns-post check. Cleanest: extract a package-private
   the choose tutor internal operation core that choose Tutor wraps after its
   owner check. Confirm this refactor is acceptable vs duplicating a few lines.
3. **OPEN #3 — exact sweep injection points.** Confirm against the live Admin Match Service create Match
   / cancel / close-post service where expire Pending For Job is called so it shares the assign/close
   transaction (and that cancel→reopen does NOT resurrect expired invites — by design it should not).
4. **OPEN #4 — invite cap per job / per tutee.** Should a tutee be limited to N PENDING invites per job
   (anti-spam, like the M6 "max 3 active interviews")? Recommend a soft cap (e g. 10 PENDING per job)
   to prevent blast-invite abuse; value TBD by product. Not blocking v1 if deferred.
5. **OPEN #5 — expiry period auto-expiry.** v1 expires only on job-exit-from-ACTIVE. Decide whether a nightly expiry period
   sweep (invites older than N days) is needed; gate on the invitations pending age gauge.
6. **OPEN #6 — secondary entry (list-card Invite).** Confirm Mo S Co W: detail-page invite = Must;
   list-card invite = Should (fast-follow) — or both in v1.
7. **OPEN #7 — notify tutee on withdraw-of-own-invite?** Recommend no notification (it's their own
   action). Confirm.

---

## Boss-review conditions (2026-06-03)
1. **Instrumentation (Blake/Data).** Build is blocked until this spec names the events:
   invite created, invite accepted, invite declined, invite expired (with tutee/tutor/job ids,
   personal information-free) so invite→accept→match funnel is measurable. Add a Measurement section.
2. **Accept-error contract (Riley/frontend-facing service).** Accept folds into the apply operation, which can return
   TUTOR PROFILE INCOMPLETE (422) from the INT-017/043 gate. The accept response must surface that
   distinctly so the frontend routes the tutor to profile-completion, not a dead error. Finalize this
   contract AFTER INT-043 sets the gating matrix (INT-043 builds first).
