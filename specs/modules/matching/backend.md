# InTuition — Matching Module — Backend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

This module now follows [product requirements document §7.5/§8.3/§20.3](../../PRD.md): first-session confirmation is authenticated matched-tutee-only and does not itself complete the match; actual agency receipt of tutor commission drives PAID/COMPLETED. Direct anonymous token confirmation and standalone first-session→COMPLETED are superseded. Use [INT-157](../../architecture/int-157-commission-first-month-locked-spec.md) agreed-plan locking, M18 listing-optional assignment, and current outbox/email/scheduler mechanisms. Unapproved cancellation/refund choices remain [pending](../../CURRENT-REQUIREMENTS.md).

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** matching  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. application programming interface Endpoints

### 1.1 Get Matchable Tutor Posts (Admin)


The illustration for get matchable tutor posts admin contains the following information:

- GET admin posts matchable tutors interface operation.
- Authorization: ADMIN.


Returns APPROVED tutor posts available for matching. No filter for existing matches (tutors can be in multiple matches).

**Query Parameters:** subjects, levels, venue Types, location Region, cursor, limit (default 20)

**Response 200:**


The get matchable tutor posts admin example describes information including posts, next Cursor. Concrete payload values and implementation syntax are omitted.


**Tutor Post Matching data transfer object** — extends public data transfer object with:


The get matchable tutor posts admin example describes information including poster, display Name, email, phone, active Match Count. Concrete payload values and implementation syntax are omitted.


---

### 1.2 Get Matchable Tutee Requests (Admin)


The illustration for get matchable tutee requests admin contains the following information:

- GET admin posts matchable tutees interface operation.
- Authorization: ADMIN.


Returns APPROVED tutee posts that do NOT have an existing PENDING or ACTIVE match (enforced by partial unique index check). Gray-out already-matched posts client-side; this endpoint returns only unmatch-eligible posts by default.

**Query Parameter:** include Matched is yes — returns all approved tutee posts including already-matched (for dashboard display with gray-out).

**Response 200:**


The get matchable tutee requests admin example describes information including posts, next Cursor. Concrete payload values and implementation syntax are omitted.


---

### 1.3 Create Match


The illustration for create match contains the following information:

- POST admin matches interface operation.
- Authorization: ADMIN.


**Request Body:**


The create match example describes information including tutor Post Id, tutee Post Id, agreed Rate Sgd, notes. Concrete payload values and implementation syntax are omitted.


**Guards:**
1. Both posts must exist
2. Both posts must have status APPROVED
3. tutee Post Id must NOT already have a PENDING or ACTIVE match → 409 TUTEE ALREADY MATCHED

**Processing (in a single database transaction):**
1. Insert into matches (status = 'PENDING', created by = admin id, agreed rate sgd, notes)
2. Log MATCH CREATED to audit log
3. Queue N-10 (match notification → tutor) to InTuition async email queue (Spring @Async + Redis)
4. Queue N-11 (match notification → tutee) to InTuition async email queue (Spring @Async + Redis)

**Response 201:**


The create match example describes information including id, status, matched At. Concrete payload values and implementation syntax are omitted.


**Errors:** 404 (post not found), 422 (post not APPROVED), 409 (tutee already matched)

---

### 1.4 List Matches (Admin)


The illustration for list matches admin contains the following information:

- GET admin matches interface operation.
- Authorization: ADMIN.


**Query Parameters:** status, tutor Post Id, tutee Post Id, from, to, cursor, limit

**Response 200:** Paginated list with match summary data transfer objects (tutor name, tutee subject/level, rate, status, dates).

---

### 1.5 Get Match Detail (Admin)


The illustration for get match detail admin contains the following information:

- GET admin matches :id interface operation.
- Authorization: ADMIN.


Returns full match with:
- Tutor post details + tutor contact info (decrypted)
- Tutee post details + tutee contact info (decrypted)
- Status timeline (derived from audit log)
- Commission summary if exists

Access is logged to audit log with action ADMIN VIEW personal information.

---

### 1.6 Activate Match (PENDING → ACTIVE)


The illustration for activate match pending → active contains the following information:

- POST admin matches :id activate interface operation.
- Authorization: ADMIN.


**Guards:** Match must be in PENDING status.

**Processing:**
1. Update status ACTIVE, record the current time as match activation time
2. Create commissions record (status = 'AWAITING CONFIRMATION', monthly rate sgd = match agreed rate sgd)
3. Prepare an authenticated tutee-only link to first-session confirmation for the specified match (product requirements document §8.3).
4. Retain a durable first-session confirmation record; a link token, if used for correlation, never replaces owner authentication.
5. Enqueue the confirmation email through the durable outbox/current email provider contract; the legacy direct-SES path is superseded.
6. Schedule the seven-day reminder/admin escalation through the current generic scheduler; retain cancellation/idempotency behavior. Legacy Redis delayed-job mechanics do not override spec 007.
7. Log MATCH ACTIVATED to audit log

**Response 200:** the active state and its recorded time

---

### 1.6b Admin Override: Record First Session Without Tutee Action


The illustration for b admin override: record first session without tutee action contains the following information:

- POST admin matches :id complete-override interface operation.
- Authorization: ADMIN.


Used when tutee has not confirmed within 7 days and admin has verified the first session occurred off-platform.

**Request Body:**


The b admin override: record first session without tutee action example describes information including override Reason. Concrete payload values and implementation syntax are omitted.


**Guards:** Match must be ACTIVE. session confirmation tokens confirmed at IS no value (only needed for override; normal complete requires confirmation).

**Processing:**
1. Record the current time as the token’s confirmation time and mark the token used
2. Cancel InTuition async email queue escalation job a first-session confirmation escalation tied to the match
3. Record first-session confirmation and permit the existing invoice progression; do not mark agency receipt or complete the match.
4. Audit the override reason, actor and time as a first-session-confirmation override.

**Response 200:** first-session confirmation recorded with the override marker; Match Status is unchanged. The old COMPLETED response is superseded by the receipt-based completion rule.

---

### 1.7 Match Completion (receipt of tutor commission)

The former standalone submit operation for admin matches :id complete design is superseded by product requirements document §7.5/§20.3. Neither a
tutee confirmation nor an admin first-session override alone completes the match. They fulfill the retained
first-session/invoice obligation. The admin records actual agency receipt of tutor commission through the
guarded Mark paid action; commission PAID and match COMPLETED are committed atomically. Parent lesson
payment and an amount merely owed do not satisfy receipt. Apply 017/019 confirmation and concurrency guards.

This corrects the required contract; it does not assert a current route has been removed or changed.

### 1.8 Cancel Match (PENDING or ACTIVE → CANCELLED)


The illustration for cancel match pending or active → cancelled contains the following information:

- POST admin matches :id cancel interface operation.
- Authorization: ADMIN.


**Request Body:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Processing:**
1. Guard PENDING/ACTIVE and serialize against assignment/activation/payment. COMPLETED/CANCELLED returns the established terminal-state rejection.
2. Set match CANCELLED and record reason/time; reopen the job to ACTIVE and restore eligible applications according to the assignment lifecycle (not a blanket reset of human rejections).
3. Cancel an unpaid pending commission; preserve any historical PAID receipt. Preserve audit history and cancel outstanding confirmation/reminder work idempotently.
4. Audit the cascade. Paid refunds/clawbacks and cancellation commercial entitlements remain separate open decisions; this operation does not invent a refund or erase received revenue.

**Response 200:** the cancelled state

---

### 1.9 First Session Confirmation (authenticated matched tutee)

The email link opens first-session confirmation for the specified match, authenticated and tutee-only per product requirements document §8.3,
which §20.3 explicitly reuses. After login, return to that page. The matched tutee must explicitly click
"Yes, my first session has taken place"; GET/navigation/email preview never mutates confirmation state.

The write is an authenticated, cross-site request forgery protection-protected POST scoped to the matched tutee. A bearer link token alone is
insufficient. If the legacy matches confirm-session :token interface operation route is retained as an adapter, it must
enforce the same authenticated-owner checks; its old Authorization: None contract is superseded.

Processing: authorize the matched tutee without leaking other matches, then atomically record the first-session
confirmation and its timestamp/audit event. Already-confirmed-or-later requests return 200/current state without
duplicate records (product requirements document AC-13). Invalid/expired optional link context cannot mutate a record. Cancel the overdue
reminder/escalation idempotently. Confirmation does not mark commission PAID or match COMPLETED.

Required evidence for implementation: unauthenticated/wrong-owner rejection, login return path, GET/prefetch
no-op, explicit POST confirmation, idempotent repeat and cancellation race. This documentation correction has
not run these checks.

### 1.10 My Matches (Tutee/Tutor Dashboard)


The illustration for my matches tutee/tutor dashboard contains the following information:

- GET dashboard my-matches interface operation.
- Authorization: TUTOR or TUTEE.


Returns matches where the user's post is involved.

**Response 200 (no contact info):**


The my matches tutee/tutor dashboard example describes information including matches, id, status, matched At, tutor Post Summary, tutee Post Summary, confirmation Pending. Concrete payload values and implementation syntax are omitted.


---

### 1.11 Contact Info Reveal (Admin)


The illustration for contact info reveal admin contains the following information:

- GET admin matches :id contact interface operation.
- Authorization: ADMIN.


Returns decrypted phone and email for both tutor and tutee in the match. Always logged.

---

## 2. Service Layer


Implementation example omitted. This public edition retains the written requirements in this section.


All state-changing operations wrapped in database transactions.

---

## 3. Confirmation Token

Token is a signed signed session token:


The . confirmation token example describes information including sub, purpose, tutee Id, iat, exp. Concrete payload values and implementation syntax are omitted.


Signed with CONFIRMATION TOKEN SECRET (separate from auth signed session token secret). Token hash stored in session confirmation tokens for idempotency checks.

---

## 4. Notifications

| Event | ID | Recipient | Channel | Queue |
|---|---|---|---|---|
| Match created | N-10 | Tutor | Email | InTuition async email queue (Spring @Async + Redis) |
| Match created | N-11 | Tutee | Email | InTuition async email queue (Spring @Async + Redis) |
| Session confirmation request | N-12 | Tutee | Email | Direct SES (critical) |
| Confirmation overdue (7 days) | N-13 | Admin | Email | InTuition async email queue (Spring @Async + Redis) — delayed job |

N-12 is sent as a direct SES call (not via InTuition async email queue) per architecture spec — critical path, cannot tolerate worker queue latency.
