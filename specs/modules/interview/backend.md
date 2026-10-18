# InTuition — Interview Module — Backend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Current proposal/scheduling/cancellation routes fold INT039 and021. Reminder handling is006/007, including confirmation-only short notice and safeguards before enablement; the remaining older implementation examples are historical.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Module:** interview  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. application programming interface Endpoints

### 1.1 Propose Interview (Tutee) — current INT039/021 contract

submit operation for interviews creates PROPOSED, not a new REQUESTED row. Follow [INT039](../../architecture/int-039-interview-negotiation.md)'s Propose Interview Request with canonical tutor identity, the linked tutee job and 1–3 proposed slots; a published tutor listing is optional where the canonical profile path applies. Proposal/counter/accept/decline use that spec's state/whose Turn/slot validation.

Spec 021 permits invitation/interview initiation for a non-deleted PENDING or ACTIVE tutee post. Keep application/assignment predicates separate. Removed/rejected/closed jobs reject new actions, and021 requires rejection to void pending arrangements cleanly without contact disclosure. Contact release is settled by current baseline C04: authorized assigned participants only; frontend shows ACTIVE/COMPLETED and hides match-PENDING. Approval, interview or invitation alone never releases contact.

Verify PENDING and ACTIVE success, soft-deleted PENDING rejection, 1–3 valid slots, wrong-role/ownership denial and no-listing canonical identity. Historical REQUESTED rows remain supported by the admin schedule path; new proposal creation does not use that status.

---

### 1.2 Schedule Interview (Admin)


The illustration for schedule interview admin contains the following information:

- POST admin interviews :id schedule interface operation.
- Authorization: ADMIN.


**Request Body:**


The schedule interview admin example describes information including scheduled At, meeting Link, admin Notes. Concrete payload values and implementation syntax are omitted.


**Guards:**
1. Interview must exist → 404
2. Interview must be in PENDING SCHEDULING or legacy REQUESTED status → 422 INVALID STATUS TRANSITION (INT039)
3. scheduled At must be a future datetime (> NOW operation + 30 minutes) → 422 SCHEDULED AT MUST backend FUTURE
4. meeting Link must be a valid URL (http/https scheme, valid hostname) → 422 INVALID MEETING LINK

**Processing:**
1. **In database transaction:**
   - Update interviews: status SCHEDULED, scheduled at, meeting link, admin notes, scheduled by admin id
   - Log INTERVIEW SCHEDULED to audit log
2. **Post-commit (via the after commit operation):**
   - **Resend email delivery through the approved durable notification path**: Send N-22 to tutee and N-23 to tutor (critical path — 5-min service-level agreement)
   - Enqueue N-26 24-hour reminder delayed job: a reminder job uniquely associated with the interview, delay is the remaining time until 24 hours before the scheduled interview. At or within24h, register no reminder: scheduling confirmation only (006/007 Option A). Current generation/durable dedup/reconciliation must follow007; this delayed-job pseudocode is historical.
   - Emit analytics event interview scheduled (with meeting platform domain extracted from URL hostname; full URL never in analytics)

**Why after Commit:** If email dispatch or analytics publish happen inside the transaction and the transaction rolls back, orphan notification jobs are created. Using after Commit ensures side effects only fire when the database write is durable.

**SES failure fallback:** If direct SES delivery fails, immediately enqueue an InTuition async email queue job an email fallback job tied to the notification for retry (max 3 attempts, exponential backoff: 1 min, 5 min, 15 min). If all retries fail, page the on-call admin. The 5-minute service-level agreement clock resets from the first retry attempt.

**Response 200:**


The schedule interview admin example describes information including id, status, scheduled At, meeting Link, meeting Link Domain Warning. Concrete payload values and implementation syntax are omitted.


meeting Link Domain Warning: yes is included in the response if the meeting link domain is not in the known conferencing platforms list (see §3.3). Non-blocking — the schedule action succeeds regardless.

---

### 1.3 Update Meeting Link (Admin)


The illustration for update meeting link admin contains the following information:

- PATCH admin interviews :id meeting-link interface operation.
- Authorization: ADMIN.


**Request Body:**


The update meeting link admin example describes information including meeting Link. Concrete payload values and implementation syntax are omitted.


**Guards:**
1. Interview must be in SCHEDULED status → 422 INVALID STATUS TRANSITION (cannot update link once terminal)

**Processing:**
1. **In database transaction:**
   - Update interviews meeting link
   - Log INTERVIEW MEETING LINK UPDATED to audit log
2. **Post-commit (via the after commit operation):**
   - **Direct SES**: Resend N-22 (tutee) and N-23 (tutor) with updated link. Apply same SES failure fallback as §1.2.
   - Emit analytics event interview meeting link updated with the interview identifier and the meeting provider’s domain

**Response 200:**


The update meeting link admin example describes information including meeting Link, meeting Link Domain Warning. Concrete payload values and implementation syntax are omitted.


meeting Link Domain Warning: yes if the new link's domain is not in the known list (see §3.3).

---

### 1.4 Mark Interview Completed (Admin)


The illustration for mark interview completed admin contains the following information:

- POST admin interviews :id complete interface operation.
- Authorization: ADMIN.


**Guards:** Interview must be in SCHEDULED status → 422

**Processing:**
1. **In database transaction:**
   - Update interviews: status COMPLETED, record the current completion time
   - Log INTERVIEW COMPLETED to audit log
2. **Post-commit (via the after commit operation):**
   - Queue N-25 (admin self-prompt) to async email queue
   - Cancel N-26 delayed reminder job: a reminder job uniquely associated with the interview
   - Emit analytics event interview completed with the interview identifier and the whole hours elapsed since the request
3. Interview-to-match conversion tracking fires when a match is created (see §4)

**Response 200:** the completed state and its recorded time

---

### 1.5 Cancel Interview (Admin)


The illustration for cancel interview admin contains the following information:

- POST admin interviews :id cancel interface operation.
- Authorization: ADMIN.


**Request Body:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Guards:** Interview must NOT be in a terminal state (COMPLETED, NO SHOW) → 422. Can cancel from REQUESTED or SCHEDULED.

**Processing:**
1. **In database transaction:**
   - Update interviews: status CANCELLED, record the current cancellation time, cancellation reason
   - Log INTERVIEW CANCELLED to audit log
2. **Post-commit (via the after commit operation):**
   - Queue N-24 (cancellation notice to tutee and tutor) to async email queue
   - Cancel N-26 delayed reminder job: a reminder job uniquely associated with the interview
   - Emit analytics event interview cancelled with the interview identifier with who cancelled it and its previous state

**Response 200:** the cancelled state

---

### 1.6 Mark No-Show (Admin)


The illustration for mark no-show admin contains the following information:

- POST admin interviews :id no-show interface operation.
- Authorization: ADMIN.


**Request Body:**


The mark no-show admin example describes information including no Show Actor. Concrete payload values and implementation syntax are omitted.


**Guards:** Interview must be in SCHEDULED status → 422

**Processing:**
1. **In database transaction:**
   - Update interviews: status NO SHOW, no show actor, record the current no-show time
   - Log INTERVIEW NO SHOW to audit log
2. **Post-commit (via the after commit operation):**
   - Cancel N-26 delayed reminder job: a reminder job uniquely associated with the interview
   - Emit analytics event interview no show with the interview identifier and the participant recorded as absent

**No tutee/tutor notification is sent on NO SHOW.** Admin follows up off-platform using contact information available in the admin detail view. This is intentional — an automated notification to a no-show participant would be premature and potentially awkward before admin has assessed the situation.

**Response 200:** the NO_SHOW state and which participant was absent

---

### 1.7 List Interviews — Admin Queue


The illustration for list interviews — admin queue contains the following information:

- GET admin interviews interface operation.
- Authorization: ADMIN.


**Query Parameters:**


The illustration for list interviews — admin queue contains the following information:

- status interview status filter (default: REQUESTED for action queue).
- tutor Post Id uuid filter.
- tutee Id uuid filter.
- from ISO8601 filter by created at.
- to ISO8601 filter by created at.
- cursor string.
- limit integer default 20, max 100.


**Response 200:**


The list interviews — admin queue example describes information including interviews, next Cursor. Concrete payload values and implementation syntax are omitted.


**Interview Admin Summary data transfer object:**


The list interviews — admin queue example describes information including id, status, tutee, tutor Post, note, requested At, scheduled At, meeting Link. Concrete payload values and implementation syntax are omitted.


> **Live shape note.** The running response record is Admin Interview Response (a page of admin interview records, with pagination details). **spec 019 (ADM-05):** it carries **tutee Post Subject** + **tutee Post Level** (the job's subject/level codes) so the admin list renders the **Job column** for every interview, including **free-market** interviews with no tutor-post. It also carries tutee Post Id, tutor User Id/tutor Display Name/tutor Public Handle (INT-053). Verify Admin Interview Response implementation reference.

Access to tutee email and tutee phone in this response triggers an audit log entry per BR-001.

---

### 1.8 Get Interview Detail — Admin


The illustration for get interview detail — admin contains the following information:

- GET admin interviews :id interface operation.
- Authorization: ADMIN.


Returns full interview record with decrypted tutee contact info. Access logged per BR-001.

**Response 200 — Interview Admin Detail data transfer object:**


The get interview detail — admin example describes information including id, status, tutee, display Name, email, phone, tutor Post, headline, subjects, levels, hourly Rate Min Sgd, hourly Rate Max Sgd, location Planning Area, note, admin Notes, scheduled At, meeting Link, scheduled By. Additional nested details follow the surrounding requirements. Concrete payload values and implementation syntax are omitted.


---

### 1.9 List Interviews — Tutee Dashboard


The illustration for list interviews — tutee dashboard contains the following information:

- GET tutee interviews interface operation.
- Authorization: TUTEE.


Returns interviews for the authenticated tutee.

**Response 200:**


The list interviews — tutee dashboard example describes information including interviews, next Cursor. Concrete payload values and implementation syntax are omitted.


**Interview Tutee data transfer object:**


The list interviews — tutee dashboard example describes information including id, status, tutor Post, headline, subjects, levels, hourly Rate Min Sgd, hourly Rate Max Sgd, location Planning Area, note, requested At, scheduled At, meeting Link, duration Minutes. Concrete payload values and implementation syntax are omitted.


**Security:** meeting Link is returned only for SCHEDULED status. Once the interview reaches any terminal state (COMPLETED, CANCELLED, NO SHOW), meeting Link is no value in the data transfer object — the link is time-limited and no longer valid after the meeting window.

---

### 1.10 Get Interview Detail — Tutee


The illustration for get interview detail — tutee contains the following information:

- GET tutee interviews :id interface operation.
- Authorization: TUTEE (own interview only).


Returns the same Interview Tutee data transfer object. Ownership check: interviews tutee id req user.id → 403 if mismatch.

---

### 1.11 Cancel Interview Request — Tutee


The illustration for cancel interview request — tutee contains the following information:

- POST tutee interviews :id cancel interface operation.
- Authorization: TUTEE (own interview only).


Tutees may only cancel their own interviews while in REQUESTED status (before admin has scheduled).

**Guards:**
1. Ownership check → 403
2. Status must be an allowed nonterminal state under INT039 §5 → 422 otherwise; participant cancellation is not limited to REQUESTED. Verify owner and each eligible state, including negotiation and scheduled states.

**Processing:**
1. Update interviews: status CANCELLED, record the current cancellation time, cancellation reason CANCELLED BY TUTEE
2. Queue N-21 cancellation notice to admin (variant: "Interview request cancelled by tutee") to InTuition async email queue (Spring @Async + Redis)
3. If interview was in SCHEDULED status: Queue N-24 cancellation notice to tutor (the tutor already received the meeting link in N-23 and must be informed)
4. Emit analytics event interview cancelled with the interview identifier with who cancelled it and its previous state

**Response 200:** the cancelled state

---

### 1.12 List Interviews — Tutor Dashboard


The illustration for list interviews — tutor dashboard contains the following information:

- GET tutor interviews interface operation.
- Authorization: TUTOR.


Returns interviews for the authenticated tutor's posts. Only SCHEDULED and recently terminal (last 7 days).

**Query Parameters:**


The illustration for list interviews — tutor dashboard contains the following information:

- cursor string.
- limit integer default 20, max 100.


**Interview Tutor data transfer object:**


The list interviews — tutor dashboard example describes information including id, tutor Post Id, status, tutee Request Summary, subject, level, location Planning Area, budget Min Sgd, budget Max Sgd, scheduled At, meeting Link, duration Minutes. Concrete payload values and implementation syntax are omitted.


No tutee contact info (name, phone, email) in this data transfer object.

---

## 2. Service Layer


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2.1 Repository Query Patterns (N+1 Prevention)

The Interview entity has the many to one operation associations to User (tutee) and Tutor Post. Fetching a page of 20 interviews with individual lazy loads fires 40 additional read fields queries.

**get Admin Queue — use JPQL JOIN FETCH:**


Implementation example omitted. This public edition retains the written requirements in this section.


**For Interview Admin Summary data transfer object:** Map in the service layer from the joined entities — do NOT trigger a second query for decrypted contact fields. Decrypt email encrypted and phone encrypted once per Interview row, inside the mapping loop.

**Cooldown check:** Use explicit @Query — avoid derived method names. See §3.2.

**Conversion tracking:** Use the find recent completed interview operation — see §4. Never use find Top By Tutee Id And... long derived method names.

---

## 3. Guards Implementation

### 3.1 Concurrency Check


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.2 Cooldown Check

Uses outcome timestamps (completed at / no show at) not updated at, so that administrative edits after the terminal event don't reset or misrepresent the cooldown window.


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.3 Meeting Link Validation

Only https:// is accepted. http:// is rejected — meeting links must be served over TLS to protect any embedded auth tokens or single-use identifiers in the URL.


Implementation example omitted. This public edition retains the written requirements in this section.


The application programming interface response for POST .../schedule and PATCH .../meeting-link includes a warning that the meeting link uses an unexpected domain if the domain is not in the known list — the frontend renders a non-blocking advisory to the admin (see frontend §4.3).

### 3.4 Meeting Platform Domain Extraction (for analytics)


Implementation example omitted. This public edition retains the written requirements in this section.


The full URL is **never** included in analytics event properties.

---

## 4. Interview-to-Match Conversion Tracking

When a match is created (in the create operation), a background check runs:


Implementation example omitted. This public edition retains the written requirements in this section.


This check is best-effort; it does not affect match creation logic.

---

## 5. Notifications

### 5.1 N-21 — Admin Notification (New Request)

**Queue:** InTuition async email queue (Spring @Async + Redis)
**Template variables:** tutee Display Name, tutor Post Headline, tutor Post Id, interview Id, note  
**Body:** "[ACTION REQUIRED] A tutee has requested an interview with tutor post 'tutor Post Headline'. Log in to schedule: admin interview url"

### 5.2 N-22 — Tutee Interview Scheduled

**Queue:** Direct SES (critical — 5-min service-level agreement)  
**Template variables:** tutor Post Headline, scheduled At Singapore time, meeting Link, duration Minutes  
**Body:** Meeting details; joining instructions; privacy reminder ("Do not share personal contact information during the call")  
**Important:** meeting Link rendered as both a styled button AND a plain-text URL fallback (for email clients that don't render buttons)

### 5.3 N-23 — Tutor Interview Scheduled

**Queue:** Direct SES (critical — 5-min service-level agreement)  
**Template variables:** tutee Subject, tutee Level, tutee Area, tutee Budget Min, tutee Budget Max, scheduled At Singapore time, meeting Link, duration Minutes  
**Body:** Tutee request summary (NO tutee contact info); meeting details; joining instructions

### 5.4 N-24 — Interview Cancelled

**Queue:** InTuition async email queue (Spring @Async + Redis)
**Recipients:** Tutee + Tutor  
**Template variables:** tutor Post Headline, reason (if provided), tutee Can Reapply (bool — true if cooldown does not apply)

### 5.5 N-25 — Admin Follow-Up Prompt (Post-Completion)

**Queue:** InTuition async email queue (Spring @Async + Redis)
**Recipient:** Admin  
**Template variables:** tutee Display Name (pseudonym only), tutor Post Headline, interview Id, admin Interview Url  
**Body:** "Interview with tutee Display Name for post 'tutor Post Headline' has been completed. Log in to view contact details and follow up: admin Interview Url"  
**Purpose:** Prompts admin to view the interview record (where decrypted contact info is displayed and access-logged) and contact the tutee off-platform to gauge interest in a formal match.

**Personal Data Protection Act note:** Decrypted phone/email must NOT be placed in the email body. The email is delivered via SES and may be cached by email servers or indexed by spam filters. Contact details are only revealed on-platform after authenticated admin access (which triggers the audit log per BR-001).

### 5.6 N-26 — 24-Hour Reminder (P1)

**Queue:** InTuition async email queue (Spring @Async + Redis) — delayed job
**Scheduled:** When interview transitions to SCHEDULED, enqueue delayed job with a reminder job uniquely associated with the interview, delay = (scheduled At - 24 hours - NOW operation)
**Recipients:** Tutee + Tutor
**On reschedule/cancel:** Cancel the delayed job from InTuition async email queue (Spring @Async + Redis)

---

## 6. Rate Limiting

| Endpoint | Limit |
|---|---|
| submit operation for interviews | 10 requests per tutee per hour |
| All other interview endpoints | Standard rate limits (see domain/the private implementation folder) |

---

## 7. Error Codes

| Code | web request protocol Status | Meaning |
|---|---|---|
| TUTOR POST NOT ACTIVE | 422 | Tutor post is not in ACTIVE status |
| INTERVIEW LIMIT REACHED | 422 | This posting already has 3 open interviews |
| INTERVIEW COOLDOWN ACTIVE | 422 | 30-day cooldown in effect for this tutee/post pair |
| INVALID STATUS TRANSITION | 422 | Attempted transition is not valid for current status |
| SCHEDULED AT MUST backend FUTURE | 422 | scheduled At is in the past or < 30 min from now |
| INVALID MEETING LINK | 422 | Meeting link is not a valid https:// URL |
| INTERVIEW TERMINAL | 422 | Interview is in a terminal state; no changes allowed |

---

## 8. Reschedule Workaround (MVP)

A dedicated "Reschedule" flow is deferred to P1. For MVP, admin uses **cancel-and-rebook**:

1. Admin cancels the SCHEDULED interview via POST .../cancel (admin actor).
2. The CANCELLED state does **not** trigger the 30-day cooldown (cooldown only applies after COMPLETED or NO SHOW).
3. Tutee submits a new interview request for the same tutor post.
4. Admin schedules the new request with the corrected date/time and meeting link.

**Important:** Because tutee re-submits, the 3-open-interview concurrency guard applies. Admin should confirm tutee has capacity before cancelling. The admin cancel email (N-24) should include a note that the tutee may re-submit if admin instructs them to do so.

P1 enhancement: A first-class PATCH .../reschedule endpoint that transitions SCHEDULED → SCHEDULED with a new scheduled at and meeting link, without requiring tutee re-submission.

## 2026-09-05 current contract and verification

All scheduling/lifecycle hooks must preserve spec 007 generation and durable registration/reminder-intent contracts. Counter/accept/decline/force-pending and participant cancellation use INT039 enumerated transitions; neither REQUESTED-only cancellation nor status/link-only reminder guards remain current. Contact release follows settled C04 (assigned participants, frontend ACTIVE/COMPLETED only). Rejection/removal/closure cleans up associated open intake under C06; invitation acceptance while the parent POST is PENDING remains O01, separate from the contact gate.
