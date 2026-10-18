# InTuition — Approval Module — Backend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

Use the canonical eight-state approval model in [approval/spec md](spec.md). The reject guard is PENDING; ACTIVE re-review returns to PENDING, never PAUSED FOR REVIEW. Owner REVISION REQUESTED editing/resubmission remains required under product requirements document and [013](../../013-tutee-dashboard-and-invitations/requirements.md); later 021 permits PENDING tutee edits. Revision messages are delivered to the owner and retained in review history.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** approval  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. application programming interface Endpoints

All endpoints under admin interface operation. All require ADMIN role + valid time-based one-time passcode session (with Admin middleware).

### 1.1 Get Pending Posts Queue


The illustration for get pending posts queue contains the following information:

- GET admin posts pending interface operation.
- Authorization: ADMIN.


**Query Parameters:**


The illustration for get pending posts queue contains the following information:

- type string "TUTOR" "TUTEE" (omit for all).
- cursor string Pagination cursor.
- limit integer Default 20, max 50.
- sort By string "submitted at asc" (default) "submitted at desc".


**Response 200:**


The get pending posts queue example describes information including posts, id, post Type, poster Name, subjects, subject, levels, submitted At, waiting Hours, sla Status, next Cursor, total. Concrete payload values and implementation syntax are omitted.


**service-level agreement calculation (server-side):**
- GREEN: waiting Hours < 24
- AMBER: 24 ≤ waiting Hours < 48
- RED: waiting Hours ≥ 48

---

### 1.2 Get Post for Review (Full Admin Detail)


The illustration for get post for review full admin detail contains the following information:

- GET admin posts :type :id review interface operation.
- Authorization: ADMIN.
- type: "tutor" "tutee".


Returns the full post with decrypted contact info of the poster.

**Response 200 (Tutor Post Admin data transfer object):**


The get post for review full admin detail example describes information including id, post Type, poster, user Id, display Name, email, phone, member Since, subjects, levels, rate Min Sgd, rate Max Sgd, venue Types, group Sizes, availability Bitmask, qualifications, teaching Approach, years Experience. Additional nested details follow the surrounding requirements. Concrete payload values and implementation syntax are omitted.


**Processing:** Decrypts email encrypted and phone encrypted using MEK before returning. Access is logged to audit log with action ADMIN VIEW personal information.

---

### 1.3 Approve Post


The illustration for approve post contains the following information:

- POST admin posts :type :id approve interface operation.
- Authorization: ADMIN.
- type: "tutor" "tutee".


No request body required.

**Guards:**
- Post must exist and be in PENDING status → 409 otherwise
- Idempotency: if already ACTIVE, return 409 with code ALREADY REVIEWED

**Processing:**
1. Update status ACTIVE (state machine trigger validates transition)
2. Record the current time as both the approval and review timestamps, and record the reviewing administrator
3. Set the post expiry to 60 days after the current time
4. Insert into post review log (action = 'APPROVE', reviewed by = admin id)
5. Log POST APPROVED to audit log
6. Queue approval notification (N-03) to InTuition async email queue (Spring @Async + Redis)
7. Schedule waiting-period drip emails (all via InTuition async email queue delayed jobs, the scheduled follow-up messages for the specified post):
   - Day 3: N-05 (check-in: "We're actively matching you")
   - Day 7: N-06 (still shortlisting: tips to improve visibility)
   - Day 14: N-07 (shortlisting update: "You've been shortlisted X times")
   - Day 21: N-08 admin-internal escalation alert

**Response 200:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Errors:** 404 (post not found), 409 (invalid state or already reviewed)

---

### 1.3b Request Revision


The illustration for b request revision contains the following information:

- POST admin posts :type :id request-revision interface operation.
- Authorization: ADMIN.


**Request Body:**


The b request revision example describes information including revision Message. Concrete payload values and implementation syntax are omitted.


**Guards:** Post must be PENDING.

**Processing:**
1. Update status REVISION REQUESTED, record the current review time, reviewed by admin id
2. Insert into post review log (action = 'REQUEST CHANGES', notes = revision Message)
3. Log POST REVISION REQUESTED to audit log
4. Queue notification N-04b (revision request email) to InTuition async email queue (Spring @Async + Redis) with the revision Message

**Response 200:** the revision requested state

---

### 1.4 Reject Post


The illustration for reject post contains the following information:

- POST admin posts :type :id reject interface operation.
- Authorization: ADMIN.


**Request Body:**


The reject post example describes information including rejection Reason, notes. Concrete payload values and implementation syntax are omitted.


**Guards:**
- Post must be PENDING. Admin re-review returns ACTIVE posts to PENDING; there is no PAUSED FOR REVIEW state.
- rejection Reason must be one of the 7 valid enum values → 422 otherwise

**Processing:**
1. Update status REJECTED, rejection reason, rejection notes
2. Insert into post review log (action = 'REJECT', rejection reason, notes)
3. Log POST REJECTED to audit log
4. Queue rejection notification (N-04) with reason to InTuition async email queue (Spring @Async + Redis)
5. Cancel any pending drip emails for this post from InTuition async email queue (remove delayed jobs by job Id pattern the scheduled follow-up messages for the specified post)

**Response 200:**


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.5 Reopen / Force to Pending (ACTIVE → PENDING)


The illustration for reopen / force to pending active → pending contains the following information:

- POST admin posts :type :id reopen interface operation.
- Authorization: ADMIN.


**Request Body:**


Implementation example omitted. This public edition retains the written requirements in this section.


Used when admin needs to pull a live post back for re-review (e g., reported by a user, suspected policy violation). Post returns to PENDING queue for re-review — there is no separate PAUSED FOR REVIEW state.

**Processing:**
1. Update status PENDING
2. Insert into post review log (action = 'REOPEN', notes)
3. Log POST REOPENED to audit log

**Response 200:** the pending state

---

### 1.6 Review History Log


The illustration for review history log contains the following information:

- GET admin posts review-log interface operation.
- Authorization: ADMIN.


**Query Parameters:**


The illustration for review history log contains the following information:

- post Type "TUTOR" "TUTEE".
- action "APPROVE" "REJECT" "REOPEN".
- admin Id uuid filter by reviewing admin.
- from ISO8601 date range start.
- to ISO8601 date range end.
- cursor string.
- limit integer default 20, max 100.


**Response 200:**


The review history log example describes information including entries, id, post Id, post Type, poster Name, action, rejection Reason, notes, reviewed By, reviewed At, next Cursor. Concrete payload values and implementation syntax are omitted.


---

## 2. Service Layer

### Post Approval Service


Implementation example omitted. This public edition retains the written requirements in this section.


All state-changing methods:
1. Validate current post status
2. Call the transition operation (throws if invalid)
3. Insert into post review log
4. Dispatch notifications via the queue operation
5. Write to audit log

Operations are wrapped in a database transaction (database update + log insert must be atomic).

---

## 3. Notification Triggers

| Event | Notification | Channel | Recipient | Queue |
|---|---|---|---|---|
| ACTIVE (approved) | N-03 | Email | Post owner | InTuition async email queue (Spring @Async + Redis) — priority high |
| REJECTED | N-04 | Email | Post owner | InTuition async email queue (Spring @Async + Redis) — priority high |
| REVISION REQUESTED | N-04b | Email | Post owner | InTuition async email queue (Spring @Async + Redis) — priority high |
| Day 3 post-approval | N-05 | Email | Post owner | InTuition async email queue (Spring @Async + Redis) — delayed job |
| Day 7 post-approval | N-06 | Email | Post owner | InTuition async email queue (Spring @Async + Redis) — delayed job |
| Day 14 post-approval | N-07 | Email | Post owner | InTuition async email queue (Spring @Async + Redis) — delayed job |
| Day 21 post-approval | N-08 | Email | Admin (internal) | InTuition async email queue (Spring @Async + Redis) — delayed job |

Drip email jobs use the scheduled follow-up messages for the specified post — deduplicated on job Id in InTuition async email queue. On rejection, cancel all pending drip jobs using the scheduled follow-up messages for the specified post pattern.

---

## 4. Rate Limiting & Security

- All admin approval endpoints exempt from public IP rate limits
- Admin sessions require valid time-based one-time passcode (via Redis session key; idle timeout 30 min)
- personal information decryption (for review page) is logged to audit log with action ADMIN VIEW personal information including which fields were accessed
- Double-approval prevention: service layer checks current status before transitioning; concurrent requests protected by read fields ... with a record lock on the post row

---

## 5. Idempotency

All state-transition endpoints are idempotent per intended outcome:
- Approving an already-approved post → 409 ALREADY REVIEWED
- Rejecting an already-rejected post → 409 ALREADY REVIEWED
- Reopen on non-APPROVED post → 409 INVALID STATE

This prevents double-click bugs from creating duplicate post review log entries.
