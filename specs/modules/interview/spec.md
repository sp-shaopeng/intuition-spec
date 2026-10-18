# InTuition — Interview Module — Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The original five-state/listing path is retained for history. The effective negotiation model is [INT-039](../../architecture/int-039-interview-negotiation.md): PROPOSED/counter/accepted-slot handoff, expanded active-cap set, terminal decline/expiry, and cancellation across the permitted pre-terminal negotiation states. [021](../../021-tutee-request-flow/spec.md) allows PENDING owner posts. Do not restore old listing prerequisites or REQUESTED-only self-cancel as the complete current rule.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** interview  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Overview

The Interview module implements the platform's primary unique selling point (USP): a structured pre-match interview session that allows tutees (parents/students) to have a short conversation with a tutor before committing to a formal match.

Unlike competitors where tutees accept or reject a tutor blindly based on a listing profile, this platform enables a 15-minute admin-coordinated video call. Admin remains the intermediary throughout — no contact information is ever shared between tutee and tutor.

This module covers: interview request creation, admin scheduling workflow, meeting link delivery, outcome tracking, and analytics.

---

## 2. Business Rules

| Rule | Reference |
|---|---|
| Any TUTEE may request an interview with any ACTIVE tutor post | BR-011 |
| Max 3 concurrent open requests (REQUESTED or SCHEDULED) per tutee | BR-011 |
| 30-day cooldown per (tutee, tutor post id) pair after COMPLETED or NO SHOW | BR-011 |
| Admin-mediated scheduling only — no direct tutee-tutor scheduling | BR-013 |
| Max 15 minutes per session; stated in notifications, not technically enforced | BR-012 |
| Admin must provide a meeting link (any URL) to schedule | BR-014 |
| No contact information shared between tutee and tutor through this flow | BR-015 |

---

## 3. Interview Lifecycle

### 3.1 State Machine


Implementation example omitted. This public edition retains the written requirements in this section.


**Terminal states:** COMPLETED, CANCELLED, NO SHOW — no further status changes permitted.

### 3.2 State Definitions

| State | Meaning |
|---|---|
| REQUESTED | Tutee has submitted the request; admin has not yet scheduled |
| SCHEDULED | Admin has set date/time and provided meeting link; both parties notified |
| COMPLETED | Interview took place; admin marked outcome |
| CANCELLED | Interview cancelled (by admin, at any stage before or after scheduling) |
| NO SHOW | Interview was scheduled but one or both parties did not attend |

### 3.3 Valid Transitions

| From | To | Actor | Condition |
|---|---|---|---|
| REQUESTED | SCHEDULED | Admin | Admin enters valid date/time and meeting link |
| REQUESTED | CANCELLED | Admin | Admin cancels before scheduling |
| SCHEDULED | COMPLETED | Admin | Admin marks outcome after scheduled time |
| SCHEDULED | CANCELLED | Admin | Admin cancels after scheduling (before or after meeting time) |
| SCHEDULED | NO SHOW | Admin | Admin marks that one/both parties did not join |

**All transitions from terminal states are rejected (web request protocol 422).**

**Reschedule (MVP workaround):** There is no dedicated RESCHEDULE transition in MVP. To change the date/time or meeting link of a SCHEDULED interview: admin cancels the interview (admin-actor, CANCELLED does NOT trigger cooldown), and tutee re-submits a new request. A first-class reschedule flow is deferred to P1. See backend md §8 for the full cancel-and-rebook procedure.

---

## 4. Interview Request Guards (BR-011)

### 4.1 Concurrency Limit

A tutee may not have more than **3 concurrent open interview requests** (status REQUESTED or SCHEDULED) at any point in time. Attempts to exceed this limit return web request protocol 422: INTERVIEW LIMIT REACHED.

### 4.2 Cooldown Period

A tutee may not submit a new interview request for the **same tutor post** within **30 calendar days** of a previous interview for that post reaching COMPLETED or NO SHOW. Attempts return web request protocol 422: INTERVIEW COOLDOWN ACTIVE.

The cooldown is per tutee id, tutor post id pair. A CANCELLED interview does not trigger the cooldown.

### 4.3 Tutor Post Must Be Active

Interview requests are only accepted for tutor posts with status ACTIVE. If the post has been closed, expired, or is pending review, the request returns web request protocol 422: TUTOR POST NOT ACTIVE.

---

## 5. Meeting Link Policy (BR-014)

- Admin provides any valid URL as the meeting link (Zoom, Google Meet, Microsoft Teams, Jitsi, or any other service)
- The platform does not create or manage video conferencing rooms
- The link is stored in the interviews meeting link column (text of up to 2000 characters)
- The link is delivered to tutee via N-22 and to tutor via N-23
- Admin may update the link while the interview is in SCHEDULED status; both parties are re-notified
- Meeting links must **never** appear in analytics event properties (they may contain authentication tokens or single-use identifiers). Only the hostname (e g., "zoom us") is captured for analytics.

---

## 6. Privacy Rules (BR-015)

The interview flow preserves the same privacy model as the rest of the platform:

| Party | Can See | Cannot See |
|---|---|---|
| Tutee | Tutor post content (subjects, levels, rate, area) + meeting link + date/time | Tutor's name, phone, email |
| Tutor | Tutee request summary (subject, level, area, budget) + meeting link + date/time | Tutee's name, phone, email |
| Admin | All contact info (access logged per BR-001 / §10.6.2) | — |

The meeting link is the only information shared between tutee and tutor. Admin contacts the tutor via admin-held contact info (not on the platform) to coordinate scheduling.

---

## 7. Notifications

| ID | Trigger | Recipient | Channel | service-level agreement |
|---|---|---|---|---|
| N-21 | Tutee submits request | Admin | Email | 30 min |
| N-22 | Admin schedules → tutee | Tutee | Email | 5 min |
| N-23 | Admin schedules → tutor | Tutor | Email | 5 min |
| N-24 | Admin or tutee cancels | Tutee + Tutor | Email | 30 min |
| N-25 | Admin marks COMPLETED | Admin (self-prompt) | Email | 30 min |
| N-26 | 24-hour reminder (P1) | Tutee + Tutor | Email | — |

**N-22 and N-23 must be delivered within 5 minutes** — tutee and tutor need the link promptly.

**Admin scheduling service-level agreement:** Admin must schedule a REQUESTED interview within **1 business day** of receipt. This is the core USP commitment. See business requirements document §9.5 for the corresponding metric (admin scheduling turnaround hours).

**No-show notifications:** NO SHOW status does **not** trigger an automated notification to tutee or tutor. Admin follows up off-platform using the contact information in the admin detail view. This is intentional — automated notification is inappropriate before admin has assessed the situation and decided on next steps.

### 7.1 N-22 Content (Tutee — Interview Scheduled)

- Subject: "Your interview is confirmed — join details inside"
- Body must include:
  - Tutor post headline (not tutor's real name or contact)
  - Date and time (Singapore time, human-readable)
  - Duration: "This is a 15-minute interview session"
  - Meeting link (the full URL, as a clickable button)
  - Instructions: "Join at the scheduled time by clicking the link above."
  - Important: "Do not share your personal contact information during the interview."

### 7.2 N-23 Content (Tutor — Interview Scheduled)

- Subject: "Interview request scheduled — join details inside"
- Body must include:
  - Tutee request summary: subject, level, area, budget range (no tutee name, phone, email)
  - Date and time (Singapore time, human-readable)
  - Duration: "This is a 15-minute interview session"
  - Meeting link (the full URL, as a clickable button)
  - Instructions: "Join at the scheduled time. Your commission terms apply to any resulting match."

---

## 8. Acceptance Criteria

### AC-INT-01 — Request Eligibility: Active Post
**Given** a tutee is logged in and views a tutor post with status ACTIVE,  
**When** they submit an interview request,  
**Then** the request is accepted and the interview record is created with status REQUESTED.

### AC-INT-02 — Request Blocked: Inactive Post
**Given** a tutee attempts to request an interview for a tutor post that is not ACTIVE,  
**When** the request is submitted,  
**Then** the application programming interface returns web request protocol 422 with code TUTOR POST NOT ACTIVE.

### AC-INT-03 — Concurrency Limit
**Given** a tutee already has 3 interviews in REQUESTED or SCHEDULED status,  
**When** they attempt to submit another request,  
**Then** the application programming interface returns web request protocol 422 with code INTERVIEW LIMIT REACHED.

### AC-INT-04 — Cooldown Enforcement
**Given** a tutee completed (or had a no-show) an interview for tutor post X within the last 30 days,  
**When** they attempt to request another interview for the same tutor post X,  
**Then** the application programming interface returns web request protocol 422 with code INTERVIEW COOLDOWN ACTIVE.

### AC-INT-05 — Scheduling Requires Meeting Link
**Given** admin opens a REQUESTED interview and attempts to schedule it,  
**When** the form is submitted without a meeting link,  
**Then** the application programming interface returns web request protocol 422 with a validation error: "Meeting link is required".

### AC-INT-06 — Both Parties Notified on Schedule
**Given** admin schedules an interview with a valid date/time and meeting link,  
**When** the action is saved,  
**Then** N-22 is delivered to the tutee within 5 minutes AND N-23 is delivered to the tutor within 5 minutes, both containing the meeting link.

### AC-INT-07 — No Tutee Contact in Tutor Notification
**Given** an interview is scheduled,  
**When** notification N-23 is generated for the tutor,  
**Then** the notification contains the tutee's subject, level, area, and budget but does NOT contain the tutee's name, phone number, or email address.

### AC-INT-08 — No Tutor Contact in Tutee Notification  
**Given** an interview is scheduled,  
**When** notification N-22 is generated for the tutee,  
**Then** the notification contains the tutor post headline, rate range, and meeting details but does NOT contain the tutor's name, phone number, or email address.

### AC-INT-09 — Terminal State Immutable
**Given** an interview is in a terminal state (COMPLETED, CANCELLED, or NO SHOW),  
**When** any application programming interface call attempts to change the interview status,  
**Then** the application programming interface returns web request protocol 422 with message: "Interview is in a terminal state and cannot be modified".

### AC-INT-10 — Audit Log on Admin Contact Access
**Given** admin views an interview detail page and the tutee's contact information is displayed,  
**When** the contact field values are included in the admin application programming interface response,  
**Then** an immutable audit log entry is created (consistent with BR-001 / §10.6.2 of the business requirements document).

### AC-INT-11 — Meeting Link Never in Analytics
**Given** an interview is scheduled with a meeting link,  
**When** the interview scheduled analytics event is emitted,  
**Then** the event properties contain only the derived meeting platform domain (hostname extracted from the URL), and the full meeting link URL is absent from all analytics event properties.

### AC-INT-12 — Cancelled Interview Does Not Trigger Cooldown
**Given** a tutee had an interview with tutor post X that was CANCELLED,  
**When** fewer than 30 days have elapsed,  
**Then** the tutee may submit a new interview request for the same tutor post X without cooldown restriction.

### AC-INT-13 — Tutee Self-Cancel (REQUESTED Only)
**Given** a tutee has an interview in REQUESTED status,  
**When** the tutee submits submit operation for tutee interviews :id cancel,  
**Then** the interview transitions to CANCELLED and an admin notification (N-21 variant) is queued.

**Given** a tutee attempts to self-cancel an interview that is in SCHEDULED status,  
**When** the request is submitted,  
**Then** the application programming interface returns web request protocol 422 with code INVALID STATUS TRANSITION and message "Cannot cancel a scheduled interview — contact admin."

### AC-INT-14 — Meeting Link Update
**Given** an interview is in SCHEDULED status,  
**When** admin submits update operation for admin interviews :id meeting-link with a valid https:// URL,  
**Then** the meeting link is updated, N-22 and N-23 are resent to tutee and tutor respectively with the new link.

**Given** admin attempts to update the meeting link when the interview is in any other status,  
**When** the request is submitted,  
**Then** the application programming interface returns web request protocol 422.

### AC-INT-15 — Scheduling Time Floor Validation
**Given** admin submits a scheduling request with scheduled At less than 30 minutes from now,  
**When** the request is processed,  
**Then** the application programming interface returns web request protocol 422 with code SCHEDULED AT MUST backend FUTURE and message "Please select a date and time at least 30 minutes from now."

---

## 9. Service Layer


Implementation example omitted. This public edition retains the written requirements in this section.


All state-changing operations:
1. Validate transition is legal (reject with 422 otherwise)
2. Check eligibility guards where applicable
3. Execute within a database transaction
4. Enqueue notifications to InTuition async email queue (Spring @Async + Redis) (except N-22/N-23 which go direct SES — critical)
5. Emit analytics event

---

## 10. Interview-to-Match Conversion Tracking

When admin creates a match, the system checks whether any COMPLETED interview exists between the same tutee and the same tutor post within the last 14 days. If found, the analytics layer emits interview converted to match with interview id and days interview to match.

This check is a best-effort analytics event only — it does not affect match creation logic.
