# INT-025 — Admin In-App Notification on Post Submission

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** DRAFT — awaiting team-review + Boss sign-off before implementation
**Date:** 2026-06-03
**Authors:** Software Engineer (lead) + frontend-facing service Specialist + Product Designer
**Tracks:** INT-025 in intuition-spec/specs/issues/README md
**Migration assigned:** 0036
**Severity:** S3 (notification gap; no core-flow breakage)

---

## 1. Problem

The Approval module spec (specs/modules/approval/spec md §7) states that admins should be
notified when a post is submitted. In practice, both Tutee Post Service create And Submit and
Tutor Post Service create And Submit emit **analytics events only**:


Implementation example omitted. This public edition retains the written requirements in this section.


Three blockers prevent a quick patch:

1. **No POST SUBMITTED notification type** — the notification type PostgreSQL enum and the Java
   Notification Type enum have no value for a submission event. Adding one requires a Liquibase
   changeset.
2. **No admin fan-out helper** — Notification Service emit is strictly single-recipient. The
   platform has no helper to broadcast to all ADMIN-role users.
3. **Open product call on noisiness** — if 50 tutors and tutees submit in a day, that is up to 100
   notifications per admin inbox. Notifying on every submit without a product decision is a
   usability trap that erodes the admin notification centre's signal-to-noise ratio. This must be
   decided before code ships.

---

## 2. Proposed Design

### 2.1 Overview

- Add a single new Notification Type value: POST SUBMITTED.
- Add an emit To All Admins helper on Notification Service that fans out to every ADMIN-role user.
- Call the helper from both Tutee Post Service create And Submit and
  Tutor Post Service create And Submit using the established after-commit, failure-isolated pattern.
- Product call: emit one notification **per submission** (not a digest). Rationale and noisiness
  analysis below.

### 2.2 Notification Type Value

New Java enum value: POST SUBMITTED
New PostgreSQL enum value: POST SUBMITTED via ALTER TYPE notification type ADD VALUE.

The value is intentionally generic (not split into TUTEE POST SUBMITTED /
TUTOR POST SUBMITTED) because:
- The admin approval queue already differentiates by post type via its own column.
- Splitting the type would add combinatorial values for a low-gain distinction in the inbox.
- The notification body text carries the post type (see §2.4).

### 2.3 Fan-out Helper

Add emit To All Admins to Notification Service:


Implementation example omitted. This public edition retains the written requirements in this section.


Key design decisions:
- REQUIRES NEW propagation is used so the fan-out runs in its own short transaction, isolated
  from the submission transaction that has already committed. This prevents holding the main tx
  open while inserting N notification rows and avoids rollback coupling.
- the find by role operation is the existing method (line 27 of
  User Repository implementation reference); no new query is needed.
- The loop swallows per-recipient failures (same failure-isolation pattern as
  Admin Match Service safe Notify).

### 2.4 Notification Content

| Field | Value |
|---|---|
| type | POST SUBMITTED |
| title | New post awaiting review |
| body (tutee) | a new tuition-request notification showing readable subject and level names |
| body (tutor) | a new tutor-listing notification showing a readable subject name |
| link Path | /admin/approval (the admin approval queue) |

Subject/level codes are safe to include in the body — they are taxonomy codes, not personal information. No real
names, emails, or phone numbers appear.

### 2.5 Emit Call Site

In both Tutee Post Service create And Submit and Tutor Post Service create And Submit, add an
after-commit fan-out immediately after the existing analytics publish calls:


Implementation example omitted. This public edition retains the written requirements in this section.


The run After Commit helper follows the Admin Match Service pattern using
Transaction Synchronization Manager.

Both services must inject Notification Service. Tutee Post Service already uses constructor
injection; Tutor Post Service uses a supplied application dependency constructor injection — add the parameter to
both.

---

## 3. Schema — Migration 0036

**File:** 0036-add-post-submitted-notification-type implementation reference


Implementation example omitted. This public edition retains the written requirements in this section.


**Migration numbering context:**
- 0031 — allow-post-soft-delete-close (INT-015, deployed)
- 0032 — reserved for M19 Google OAuth
- 0033–0035 — available for other in-flight work (INT-039 timeslot negotiation, etc.)
- **0036 — this changeset (INT-025)**

The migration follows the established pattern from 0013
(add-interview-completed-notification-type) and 0015 (REVIEW RECEIVED): a single-database query language
ALTER TYPE … ADD VALUE when not already defined changeset with a documented no-op rollback.

---

## 4. Fan-out Approach — Recommendation and Rationale

### 4.1 Option A — One notification row per admin (RECOMMENDED)

Emit one notifications row per ADMIN-role user per submission. Each admin sees it in their own
inbox and can mark it read independently.

**Pros:**
- Consistent with how every other notification type works (MATCH CREATED, INTERVIEW SCHEDULED,
  REVIEW RECEIVED — all produce per-recipient rows).
- No new read/delivery tracking surface — read at already handles per-user read state.
- The admin notification bell already routes to /admin/notifications; no routing change needed.
- Simple to implement; mirrors the existing pattern exactly.

**Cons:**
- If there are 5 admins and 100 submissions in a day, that is 500 notification rows. At typical
  Singapore marketplace scale this is not a concern; notifications is already indexed on
  recipient user id, read at and recipient user id, created at.

### 4.2 Option B — Single shared "admin broadcast" row

Store one notification row with a special sentinel recipient user id (e g. a fixed
"ADMIN BROADCAST" unique identifier) and fan-out the unread count / list query to all admins who join against
it.

**Pros:** one database row per event.

**Cons:** requires schema additions (broadcast table or sentinel user), new query logic,
per-admin read-state tracking is re-invented, tested separately, and breaks the uniform contract
that recipient user id identifies a real user. Complexity vastly exceeds the marginal storage
saving. Not recommended.

**Decision: Option A.** The row-per-admin model is correct, simple, and consistent. At current
and near-term scale there is no storage or performance argument for Option B.

---

## 5. Noisiness Decision (Product Designer)

### 5.1 Context

The question is whether notifying every admin on every post submission produces useful signal or
inbox noise.

### 5.2 Per-submit notification (RECOMMENDED for now)

**Rationale:**
- The approval queue already exists at /admin/approval. The notification is not the primary
  work-management surface — it is a **nudge** that new items are queued.
- In the early Singapore launch period, submission volume is low (single to low double digits per
  day). Per-submit notifications have direct operational value: admins are prompted within
  seconds, supporting the 24-hour service-level agreement target (approval/spec md §6).
- The notification body links directly to /admin/approval (the queue), so the admin is one
  tap away from the action. The notification IS the service-level agreement prompt.
- Admins can use "Mark all read" (existing bulk action) to clear the inbox after a review session.

**Against digest:**
- A digest (e g. "5 posts submitted in the last hour") would require a scheduled job,
  job-state tracking, and deduplication logic. This is overengineering for a platform at early
  scale.
- A per-session digest loses the sub-minute latency that makes the notification useful as an service-level agreement
  prompt.

**Against no notification at all:**
- The approval spec §7 explicitly specifies admin notification on submission. Removing it would
  contradict the spec and remove the only automated service-level agreement-reminder path.

### 5.3 Future escape hatch (if volume grows)

If submission volume grows to the point of noise (e g. 50+ submissions per day), two targeted
mitigations exist — neither requires re-architecting this design:

1. **Admin notification preferences** (a future per-user settings flag): admins who prefer not
   to receive POST SUBMITTED notifications opt out at the profile level. The fan-out helper
   skips opted-out admins.
2. **Throttle / de-duplicate** in the helper: if more than N POST SUBMITTED notifications
   have been emitted to an admin in the last M minutes, skip the next one and instead update the
   body of the most-recent unread notification ("3 new posts awaiting review"). This is a
   future-scoped enhancement, not part of this spec.

**Decision: per-submit, to all admins, effective immediately.** Re-evaluate if sustained daily
submission count exceeds ~50.

---

## 6. frontend-facing service Contract

There is no new application programming interface endpoint and no change to the existing notification contract.

The existing read operation for notifications and read operation for notifications unread-count endpoints
serve admin notifications identically to tutee/tutor notifications — they are scoped by the
signed session token-authenticated user Id on the server, so admins automatically see their POST SUBMITTED
rows.

The Notification Response data transfer object already carries type (the string enum value) — the frontend can
optionally style or group by type equals POST SUBMITTED for visual distinction in the admin
inbox, but this is a purely additive frontend concern and does not change the application programming interface contract.

**No frontend-facing service changes required.**

---

## 7. Soft-Delete

No data is hard-deleted by this feature. The notification rows follow the standing soft-delete
principle:
- Marking a notification as read sets read at — the row is never deleted.
- If an admin account is soft-closed in the future, their notification rows are retained for
  audit purposes (same as all other notification rows).
- The emit To All Admins fan-out only queries the find by role operation — accounts that have
  been soft-disabled via a future active flag would be excluded at that layer; this spec does
  not define admin account soft-close (not yet built), but the fan-out query is the correct
  choke point for that exclusion when it arrives.

---

## 8. Acceptance Criteria

**AC-025-01 — POST SUBMITTED type is present in the database enum**
Given: migration 0036 has been applied
When: the enum range operation is executed
Then: POST SUBMITTED appears in the result

**AC-025-02 — Tutee post submission notifies all admins**
Given: at least one ADMIN-role user exists
When: a tutee calls submit operation for tutee posts (creates and submits)
Then: within one transaction cycle, one notifications row exists per admin user with
type POST SUBMITTED, link path /admin/approval, and read at no value

**AC-025-03 — Tutor post submission notifies all admins**
Given: at least one ADMIN-role user exists
When: a tutor calls submit operation for tutor posts
Then: within one transaction cycle, one notifications row exists per admin user with
type POST SUBMITTED and link path /admin/approval

**AC-025-04 — Notification body does not contain personal information**
Given: a submitted post whose owner has a real name and email
When: the POST SUBMITTED notification rows are inspected
Then: title, body, and link path contain no email address, phone number, or real name —
only subject/level codes and the queue link

**AC-025-05 — Admin unread count increments**
Given: an admin with 0 unread notifications
When: any post is submitted
Then: read operation for notifications unread-count returns a result count of one for that admin

**AC-025-06 — One notification failure does not block the submission**
Given: Notification Service emit throws for one admin recipient
When: a post is submitted
Then: the submission returns 201, other admin recipients receive their notification, and the
failure is logged at WARN level (not propagated)

**AC-025-07 — Submission transaction commits before notifications are sent**
Given: any post submission
When: the submission transaction commits
Then: notifications are emitted after-commit (via Transaction Synchronization Manager), not
inside the main transaction — verified by observing that a notification emit failure does not
roll back the committed post

**AC-025-08 — Body text carries post type context**
Given: a tutee post submitted for subject CHN PSLE, level P6
When: the notification row is read
Then: body contains CHN PSLE and P6 (or equivalent formatted codes)

---

## 9. Test Plan

### 9.1 Unit tests (new — Notification Service Test)

- emit To All Admins sends To Each Admin: stub the find by role operation returning 3 admins;
  verify notification Repository save called 3 times with the correct recipient IDs.
- emit To All Admins one Failure does Not Block Others: stub save to throw on the second admin;
  verify the third admin still gets a notification and no exception propagates.
- emit To All Admins no Admins no Op: stub returns empty list; verify zero saves, no exception.

### 9.2 Integration tests (extend Post Lifecycle IT or new Admin Submit Notification IT)

- tutee Post submit notifies Admins: seed 2 admins; tutee calls submit operation for tutee posts;
  assert 2 notifications rows with type POST SUBMITTED, correct link, null read at.
- tutor Post submit notifies Admins: same for tutor post; assert subject code present in body.
- submit notification Body is Pii Free: assert no email/display Name in any notification field.
- submit unread Count Increments For Admin: admin's unread count goes from 0 → 1 after a submission.

### 9.3 Regression

- All existing Tutee Post Service and Tutor Post Service tests must pass unchanged (the new
  Notification Service dependency is injected; test constructors must be updated or use
  @Mock Bean).
- Notification Service Test existing tests must be unaffected.
- Coverage gate: backend line ≥ 97.5%, branch ≥ 97.5% — the new emit To All Admins method must be
  covered by the unit tests above (the failure-path branch is the critical one).

---

## 10. Open Decisions

| # | Decision | Recommendation | Status |
|---|---|---|---|
| OD-1 | Migration number: 0036 or the next available free slot at implementation time? | Use 0036 as assigned; if 0032–0035 are consumed before this ships, take the next free slot and update this spec accordingly. | Open — confirm at implementation |
| OD-2 | Should the admin notification user interface group or visually distinguish POST SUBMITTED rows from other types? | Yes — a small queue-icon or "Review pending" label in the admin notifications page would improve scannability. frontend enhancement, not gated on this spec. | Deferred to frontend polish |
| OD-3 | Future: opt-out per admin for POST SUBMITTED if volume grows. | Implement as a notification preferences table when sustained daily submissions exceed ~50. Out of scope for this spec. | Future |
| OD-4 | Should emit To All Admins also accept a "skip self" parameter (e g. to exclude the admin who triggered an action)? | Not needed here (submissions are tutee/tutor-originated, not admin-originated). But the helper signature should be documented as extensible. | No action now |
| OD-5 | After-commit pattern: use a listener that runs in relation to a database transaction instead of Transaction Synchronization Manager? | Either works. The existing Admin Match Service run After Commit uses Transaction Synchronization Manager directly; use the same pattern for consistency. Migrate both to a listener that runs in relation to a database transaction together as a separate refactor if desired. | No action now |
