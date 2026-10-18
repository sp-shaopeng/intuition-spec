# InTuition — Posting Module — Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The canonical revision/resubmission and ACTIVE→PENDING rules remain effective. Later [021](../../021-tutee-request-flow/spec.md) permits PENDING tutee edits. M18 removes the active-listing prerequisite for tutor browse/apply; use profile completeness from [012](../../012-tutor-apply-and-profile-ux/requirements.md). Historical availability/schema examples must defer to later shared taxonomy/availability amendments; this pass does not choose a new encoding.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail
## Singapore Private Tuition Agency — Two-Sided Marketplace

**Module:** Posting  
**Document Version:** 1.0  
**Date:** 2026-05-27  
**Status:** Draft  
**Source Documents:** product requirements document v1.1, business requirements document v1.1, schema md v1.1

---

## Table of Contents

1. [Overview](#1-overview)
2. [Post Types](#2-post-types)
3. [Tutor Post Field Specification](#3-tutor-post-field-specification)
4. [Tutee Post Field Specification](#4-tutee-post-field-specification)
5. [Availability Bitmask Specification](#5-availability-bitmask-specification)
6. [Post State Machine](#6-post-state-machine)
7. [Browsing Rules](#7-browsing-rules)
8. [Contact Info Redaction — BR-001](#8-contact-info-redaction--br-001)
9. [Post Expiry](#9-post-expiry)
10. [Edit-While-Active](#10-edit-while-active)
11. [Draft Autosave](#11-draft-autosave)
12. [Validation Rules Summary](#12-validation-rules-summary)
13. [Business Rules](#13-business-rules)
14. [Acceptance Criteria](#14-acceptance-criteria)

---

## 1. Overview

The Posting module is the entry point for both sides of the marketplace. It governs:

- Creation, submission, and lifecycle management of **Tutor Post** and **Tutee Post** records.
- Admin-gated approval flow: every post must be reviewed and approved before it becomes publicly visible.
- Contact information isolation: phone and email are stored encrypted and are never exposed outside admin-authenticated views (BR-001).
- Discovery: approved tutor posts are publicly browseable without login; approved tutee posts are browseable only by logged-in approved tutors (anonymised).

This module does **not** cover the matching workflow (see modules/matching/) or commission calculation (see modules/commission/).

---

## 2. Post Types

### 2.1 Tutor Post

A tutor offering post published by a user with role TUTOR. Describes the tutor's teaching credentials, subjects, levels, rates, availability, and location. A tutor may hold multiple simultaneous active Tutor Posts (e g., one for Maths, one for Science).

Key characteristics:
- Multi-select for subjects, levels, venue types, and group sizes.
- Rate expressed as min/max SGD integer range.
- Qualifications and teaching approach as free text.
- Profile photo stored in S3 (access restricted to admin and post owner).
- Post remains ACTIVE after a match is created — a tutor post can participate in multiple simultaneous matches.

### 2.2 Tutee Post

A tutee request post published by a user with role TUTEE. Describes the tuition need, including subject, level, schedule, budget, and location. A tutee may have multiple posts for different subjects.

Key characteristics:
- Single subject and single level per post (one need per post).
- Budget expressed as min/max SGD integer range.
- Student relationship (self, parent's child, other) and student count.
- Session duration (1.0, 1.5, 2.0, or 2.5 hours) and weekly frequency.
- A tutee post moves to MATCHED when admin creates a match, removing it from the discovery index.

---

## 3. Tutor Post Field Specification

All fields map directly to tutor posts table plus junction tables tutor post subjects, tutor post levels, tutor post venue types, tutor post group sizes.

| Field | database Column | Type | Required | Constraints | Visibility |
|---|---|---|---|---|---|
| Title | title | VARCHAR using 100 | Yes | 1–100 chars | Public |
| Headline | headline | VARCHAR using 100 | No | 0–100 chars | Public |
| Subjects | tutor post subjects junction | TEXT[] (codes) | Yes | Min 1; codes from subjects lookup | Public |
| Levels | tutor post levels junction | TEXT[] (codes) | Yes | Min 1; codes from levels lookup | Public |
| Hourly Rate Min (SGD) | hourly rate min sgd | INTEGER | Yes | > 0; must be ≤ rate max | Public |
| Hourly Rate Max (SGD) | hourly rate max sgd | INTEGER | Yes | > 0; must be ≥ rate min | Public |
| Venue Types | tutor post venue types junction | venue type[] | Yes | Min 1; STUDENT HOME / TUTOR HOME / PUBLIC PLACE / ONLINE / TUITION CENTRE | Public |
| Group Sizes | tutor post group sizes junction | group size[] | Yes | Min 1; ONE ON ONE / SMALL GROUP / BOTH | Public |
| Availability | availability bitmask | INTEGER | Yes | 1–2097151 (at least 1 slot); see §5 | Public |
| Qualifications | qualifications | TEXT | Yes | 1–500 chars | Public |
| Years Experience | years experience | SMALLINT | Yes | 0–50 | Public |
| Teaching Approach | teaching approach | TEXT | No | 0–500 chars | Public |
| Location MRT Station | location mrt station | VARCHAR using 100 | Conditional | Required if any in-person venue type selected; must be valid SG MRT/LRT station name | Public (area only) |
| Location Planning Area | location planning area | VARCHAR using 100 | Conditional | Required if any in-person venue type selected; must be valid URA planning area | Public (area only) |
| Location Region | location region | location region ENUM | Derived | Auto-derived from planning area: CENTRAL / NORTH / SOUTH / EAST / WEST | Public |
| Profile Photo | profile photo s3 key | VARCHAR using 500 | No | JPEG/PNG/Web P; max 5 MB; S3 key only; signed URL generated on access | Admin + own user only |
| Status | status | post status ENUM | System | See §6 | System-managed |
| Rejection Reason | rejection reason | VARCHAR using 50 | System | Set by admin on REJECTED; from product requirements document §6.4 enum | Admin + own user |
| Admin Notes | admin notes | TEXT | System | Internal; never returned to non-admin application programming interface | Admin only |
| Submitted At | submitted at | TIMESTAMPTZ | System | Set on DRAFT → PENDING transition | System |
| Reviewed At | reviewed at | TIMESTAMPTZ | System | Set on admin decision | System |
| Reviewed By | reviewed by | unique identifier (FK users) | System | Admin user ID | Admin only |
| Approved At | approved at | TIMESTAMPTZ | System | Set on APPROVED transition | System |
| Expires At | expired at | TIMESTAMPTZ | System | Set to approved at + 60 days (configurable) | System |

**Contact fields** (on users table, not tutor posts):

| Field | Visibility | Encryption |
|---|---|---|
| Email | Admin only (BR-001) | AES-SIV ciphertext + keyed message authentication index |
| Phone | Admin only (BR-001) | AES-SIV ciphertext + keyed message authentication index |

**Rate benchmarks for validation warnings** (non-blocking):

| Level Group | Suggested Range (SGD/hr) |
|---|---|
| Primary (PSLE) | 20–40 |
| Secondary (O-Level) | 30–60 |
| JC / A-Level | 50–100 |
| IB / Poly / Uni | 50–120 |

Rates outside benchmark trigger an admin flag (RATE OUT OF MARKET) but do not block submission.

---

## 4. Tutee Post Field Specification

All fields map to tutee posts table plus junction table tutee post venue preferences.

| Field | database Column | Type | Required | Constraints | Visibility |
|---|---|---|---|---|---|
| Post Title | title | VARCHAR using 150 | No | 0–150 chars; auto-generated if blank (e g. "P6 Science Tutor Needed — Tampines") | Public (anonymised) |
| Subject | subject code | VARCHAR using 50 FK subjects | Yes | Single code from subjects lookup | Public |
| Level | level code | VARCHAR using 50 FK levels | Yes | Single code from levels lookup | Public |
| Number of Students | num students | SMALLINT | Yes | ≥ 1 | Public |
| Budget Min (SGD/hr) | budget min sgd | INTEGER | Yes | > 0; must be ≤ budget max | Public |
| Budget Max (SGD/hr) | budget max sgd | INTEGER | Yes | > 0; must be ≥ budget min | Public |
| Preferred Venue Types | tutee post venue preferences junction | venue type[] | Yes | Min 1; same enum as Tutor Post venue types | Public |
| Availability (Schedule) | availability bitmask | INTEGER | Yes | 1–2097151 (at least 1 slot); see §5 | Public |
| Session Duration | session duration hours | DECIMAL using 3,1 | Yes | Allowed: 1.0, 1.5, 2.0, 2.5 | Public |
| Frequency per Week | frequency per week | frequency per week ENUM | Yes | ONE / TWO / THREE / FLEXIBLE | Public |
| Preferred Start Date | preferred start date | DATE | No | Must be today or a future date | Public |
| Special Requirements | special requirements | TEXT | No | 0–500 chars | Public |
| Location MRT Station | location mrt station | VARCHAR using 100 | Yes | Must be valid SG MRT/LRT station name | Public (area only) |
| Location Planning Area | location planning area | VARCHAR using 100 | Yes | Must be valid URA planning area | Public (area only) |
| Location Region | location region | location region ENUM | Derived | Auto-derived: CENTRAL / NORTH / SOUTH / EAST / WEST | Public |
| Status | status | post status ENUM | System | See §6 | System-managed |
| Rejection Reason | rejection reason | VARCHAR using 50 | System | Set by admin on REJECTED | Admin + own user |
| Admin Notes | admin notes | TEXT | System | Internal; never returned to non-admin application programming interface | Admin only |

**Contact fields** (on users table, not tutee posts):

| Field | Visibility | Encryption |
|---|---|---|
| Email | Admin only (BR-001) | AES-SIV ciphertext + keyed message authentication index |
| Phone | Admin only (BR-001) | AES-SIV ciphertext + keyed message authentication index |

**Tutee display name on discovery index:** A pseudonym is generated. Full name, email, and phone are never shown to any tutor on any page (BR-001, product requirements document §5.2 note).

---

## 5. Availability Bitmask Specification

### 5.1 Encoding

The availability bitmask is a 21-bit integer. Each bit corresponds to one time slot in the weekly schedule. Bit value 1 = available; 0 = not available.

**Bit position map:**

| Bit | Day | Slot |
|---|---|---|
| 0 | Monday | Morning (approx. 08:00–12:00 Singapore time) |
| 1 | Monday | Afternoon (approx. 12:00–17:00 Singapore time) |
| 2 | Monday | Evening (approx. 17:00–22:00 Singapore time) |
| 3 | Tuesday | Morning |
| 4 | Tuesday | Afternoon |
| 5 | Tuesday | Evening |
| 6 | Wednesday | Morning |
| 7 | Wednesday | Afternoon |
| 8 | Wednesday | Evening |
| 9 | Thursday | Morning |
| 10 | Thursday | Afternoon |
| 11 | Thursday | Evening |
| 12 | Friday | Morning |
| 13 | Friday | Afternoon |
| 14 | Friday | Evening |
| 15 | Saturday | Morning |
| 16 | Saturday | Afternoon |
| 17 | Saturday | Evening |
| 18 | Sunday | Morning |
| 19 | Sunday | Afternoon |
| 20 | Sunday | Evening |

**Pattern:** bit position day index × 3 + slot index, where day index is 0=Mon … 6=Sun and slot index is 0=Morning, 1=Afternoon, 2=Evening.

**Valid range:** 0 to 2097151 (= 2^21 − 1). Value 0 indicates no availability selected; this is invalid for a submitted post (must be rejected at the application layer).

### 5.2 Encoding Examples

- Tutor available only Monday evenings and Saturday mornings: 1 2 | 1 15 = 4 | 32768 = 32772
- Tutor available all weekend: bits 15–20 all set = 1 15 | 1 16 | 1 17 | 1 18 | 1 19 | 1 20 = the recorded revision
- Tutor available all slots: the recorded revision

### 5.3 Decoding (Application Layer)


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.4 Overlap Query (Admin Matching)

To check if tutor and tutee have at least one overlapping available slot:


Implementation example omitted. This public edition retains the written requirements in this section.


This is evaluated in application memory at MVP scale after the subject/level/region filter has already reduced candidate sets. No dedicated bitmask index is created (see indexes md §Index Design Principles).

---

## 6. Post State Machine

### 6.1 States

| State | Description | Visible To |
|---|---|---|
| DRAFT | Post saved but not submitted. Not visible publicly. | Owner, Admin |
| PENDING | Submitted; awaiting admin review. Not publicly visible. | Owner, Admin |
| APPROVED | Deprecated alias for ACTIVE in initial schema. Use ACTIVE. | — |
| ACTIVE | Admin-approved; publicly visible (tutor posts: public listing; tutee posts: discovery index for approved tutors). | Public (Tutor Post) / Logged-in approved tutors (Tutee Post) |
| REVISION REQUESTED | Admin requested changes. Not publicly visible. Owner can edit and resubmit. | Owner, Admin |
| REJECTED | Admin rejected; terminal state for this submission. | Owner, Admin |
| MATCHED | Tutee Post only. A match has been created. Removed from discovery index. | Owner, Admin |
| CLOSED | Owner closed the post voluntarily or admin closed it. Terminal. | Owner, Admin |
| EXPIRED | Auto-expired after configurable period (default: 60 days after approval). Terminal unless renewed. | Owner, Admin |

> **Implementation note:** The prompt context references PAUSED FOR REVIEW as a state. The authoritative schema md defines the enum as above. The "edit-while-active" behaviour is implemented by transitioning ACTIVE → PENDING (re-review required) rather than a separate PAUSED FOR REVIEW state. See §10 for details.

### 6.2 State Transition Table

| From | To | Actor | Guard / Trigger |
|---|---|---|---|
| — | DRAFT | Owner (Tutor/Tutee) | Post created via create form |
| DRAFT | PENDING | Owner | Owner clicks "Submit for Review"; bitmask ≥ 1 slot; all required fields present |
| PENDING | ACTIVE | Admin | Admin clicks Approve; commission acknowledgement log exists (Tutor Post only) |
| PENDING | REJECTED | Admin | Admin selects rejection reason and clicks Reject |
| PENDING | REVISION REQUESTED | Admin | Admin enters revision message and clicks Request Revision |
| REVISION REQUESTED | PENDING | Owner | Owner edits post and clicks Resubmit for Review |
| ACTIVE | PENDING | Owner | Owner edits any field of an active post (re-triggers review) |
| ACTIVE | MATCHED | System | Admin creates a match referencing this Tutee Post |
| ACTIVE | CLOSED | Owner / Admin | Owner clicks Close; or admin force-closes |
| ACTIVE | EXPIRED | System (scheduler) | expires at reached; daily scheduler at 02:00 Singapore time sets status to EXPIRED |
| MATCHED | ACTIVE | System | Admin cancels the associated match; Tutee Post returns to ACTIVE |
| MATCHED | CLOSED | Owner / Admin | Owner closes; or admin force-closes |
| REJECTED | — | — | Terminal state; no outbound transitions |
| CLOSED | — | — | Terminal state; no outbound transitions |
| EXPIRED | PENDING | Owner | Owner clicks Reactivate → resubmits for admin review (admin may extend expires at on re-approval) |

### 6.3 State Machine Diagram (Text)


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 7. Browsing Rules

### 7.1 Public Tutor Listing (Unauthenticated + All Users)

- Route: /posts/browse/tutors
- Shows: Tutor Posts with status ACTIVE
- No login required
- Contact info (email, phone, profile photo) excluded from all application programming interface responses
- Filterable by: subject, level, venue type, location region/MRT, rate range, availability bitmask
- Sorted by: approved at DESC (most recently approved first)
- Pagination: cursor-based (see backend md §Pagination)

### 7.2 Tutee Request Discovery (Logged-in Approved Tutors Only)

- Route: /posts/browse/requests
- Authentication: required; role must be TUTOR; tutor's own post must have at least one ACTIVE post (product requirements document §3.6 — "approved tutors")
- Shows: Tutee Posts with status ACTIVE only (not MATCHED, PENDING, CLOSED, EXPIRED)
- Anonymisation: tutee display name shown as pseudonym (initials or first-name only); no contact info
- Filterable by: subject, level, venue type, location region, budget range, availability bitmask
- Sorted by: approved at DESC
- Pagination: cursor-based

### 7.3 Own Post List (Authenticated, Any Role)

- Route: /dashboard/my-posts
- Shows: all posts belonging to the authenticated user, any status
- Contact info not shown (users see their own data via profile, not via post view)
- Pagination: offset-based (simpler user experience for low-volume personal list)

---

## 8. Contact Info Redaction — BR-001

**Rule BR-001:** Phone and email are never shown on public pages, never shown to tutees viewing tutor posts, and never shown to tutors viewing tutee posts. They are accessible only to admin via the admin panel.

### 8.1 Fields Excluded from Non-Admin Responses

| Field | Location | Excluded From |
|---|---|---|
| email encrypted | users table | All non-admin application programming interface responses |
| phone encrypted | users table | All non-admin application programming interface responses |
| profile photo s3 key | tutor posts table | All unauthenticated responses and non-owner/non-admin authenticated responses |
| admin notes | tutor posts, tutee posts | All non-admin responses |
| reviewed by | Both post tables | All non-admin responses |

### 8.2 data transfer object Pattern

Two data transfer objects per post type govern field inclusion:

- **Public data transfer object** — used for public browse, discovery index, and own-user dashboard. Excludes all contact fields and admin-only fields.
- **Admin data transfer object** — used for admin review queue, admin post detail, and admin matching dashboard. Includes decrypted contact info and all admin fields. Every application programming interface call rendering admin contact fields appends an immutable entry to audit log.

See backend md §Role-Aware data transfer objects for full field lists.

---

## 9. Post Expiry

- Tutor Posts and Tutee Posts expire automatically at expires at.
- Default expires at = approved at + 60 days (configurable via system config; product requirements document §3.3 states "Auto-expiry of unmatched active posts after configurable period (default 60 days)").
- The daily expiry scheduler runs at 02:00 Singapore time. It transitions all posts that are ACTIVE and whose expiry timestamp is at or before the current time to status EXPIRED.

> The approval-based expiry in this module differs from the submission-based clock in the product requirements document. This review preserves both source meanings and flags the conflict; it does not approve a new expiry policy.
- System sends an email notification to the post owner on expiry.
- Admin can extend expires at on a per-post basis from the admin panel (sets a new future timestamp; does not require re-approval).
- A Tutee Post in MATCHED status is not expired by the scheduler (the match takes precedence).
- On expiry, a Tutee Post that was ACTIVE transitions to EXPIRED. Any associated PENDING or ACTIVE matches must be resolved first (admin is alerted).

---

## 10. Edit-While-Active

When an owner edits a post that is currently in ACTIVE status:

1. The edit is saved.
2. Post status transitions from ACTIVE → PENDING (re-enters the admin review queue).
3. The post is removed from the public listing / discovery index immediately on the status transition.
4. The owner is shown a warning before confirming the edit: "Editing your active post will remove it from the listing until admin re-approves it. Do you want to continue?"
5. Admin reviews the updated post as a normal PENDING submission. The review view shows the change history.
6. If admin approves: ACTIVE again; approved at is updated; the expiry timestamp is reset to 60 days after the re-approval time.

**Scope of edit-while-active trigger:** Any field change to a post in ACTIVE status triggers this behaviour, including minor changes such as availability or rate adjustments.

**DRAFT posts:** Editing a DRAFT post never triggers a status change. The owner is free to modify drafts at any time.

**REVISION REQUESTED posts:** Editing a REVISION REQUESTED post updates the draft. The post does not re-enter PENDING until the owner explicitly clicks "Resubmit for Review".

---

## 11. Draft Autosave

- Draft autosave applies to both the Tutor Post creation wizard and the Tutee Post creation form.
- Autosave is triggered:
  - On step navigation (wizard only): when the user moves from one step to the next, the current step's data is persisted.
  - On inactivity: after 30 seconds of no input, a PATCH to the draft endpoint is issued.
  - On page unload: a beforeunload handler fires a synchronous navigator send Beacon with the current form state.
- Autosave is silent (no toast on every save); a discrete "Draft saved" indicator appears in the form header after each successful save.
- If the autosave application programming interface call fails, a visible "Draft not saved — check connection" warning is displayed.
- Posts are created in DRAFT status on the server at the first autosave; subsequent autosaves are PATCHes.
- A user may have at most one DRAFT Tutor Post and one DRAFT Tutee Post in progress simultaneously (per-role limit). If a second draft is started, the system offers to continue the existing draft or discard it.

---

## 12. Validation Rules Summary

### 12.1 Tutor Post Submission Validation

| Rule | Error Message |
|---|---|
| Title required (1–100 chars) | "Title is required (max 100 characters)" |
| At least one subject selected | "Select at least one subject" |
| At least one level selected | "Select at least one level" |
| Rate min > 0 | "Minimum rate must be greater than 0" |
| Rate max  is at least  rate min | "Maximum rate must be at least the minimum rate" |
| At least one venue type | "Select at least one venue type" |
| At least one group size | "Select at least one group size" |
| Availability bitmask  is at least  1 | "Select at least one available time slot" |
| Qualifications 1–500 chars | "Qualifications required (max 500 characters)" |
| Years experience 0–50 | "Years of experience must be between 0 and 50" |
| Location required if in-person venue selected | "Location is required when offering in-person sessions" |
| Commission acknowledgement exists (server-side) | Submission blocked; redirect to acknowledgement flow |

### 12.2 Tutee Post Submission Validation

| Rule | Error Message |
|---|---|
| Subject required | "Select a subject" |
| Level required | "Select a level" |
| num students ≥ 1 | "Enter a valid student count" |
| Student relationship required | "Select student relationship" |
| Budget min > 0 | "Minimum budget must be greater than 0" |
| Budget max  is at least  budget min | "Maximum budget must be at least the minimum budget" |
| At least one venue preference | "Select at least one venue preference" |
| Availability bitmask  is at least  1 | "Select at least one available time slot" |
| Session duration in allowed set | "Select a valid session duration (1, 1.5, 2, or 2.5 hours)" |
| Frequency per week required | "Select a session frequency" |
| Preferred start date not in past | "Start date must be today or a future date" |
| Location required | "Select your location area or MRT station" |

---

## 13. Business Rules

| Rule | Description |
|---|---|
| **BR-001** | Phone and email are never shown publicly or to other users. Accessible by admin only. Every admin access is logged immutably. |
| **BR-002** | Every post must go through admin review and approval before going live. No auto-approval. |
| **BR-003** | Matches are made exclusively by admin; tutors and tutees cannot initiate contact. |
| **BR-004** | Commission is 50% of first month's tuition fee, paid by tutor, triggered by tutee first-session confirmation. |
| **BR-008** | Owners can edit their own posts. Editing an ACTIVE post resets it to PENDING for re-review. |
| **BR-009** | Tutors must acknowledge commission terms before their post can be submitted or approved. |

---

## 14. Acceptance Criteria

### AC-POST-01 — Draft Creation
Given an authenticated tutor, when they start the post creation wizard, then a DRAFT Tutor Post record is created in the database on the first autosave, and the post is not visible in any public listing.

### AC-POST-02 — Submit for Review
Given a Tutor Post in DRAFT status with all required fields populated, when the owner clicks "Submit for Review", then post status transitions to PENDING, submitted at is set to the current timestamp, the post does not appear in the public listing, and the owner receives a confirmation email within 5 minutes.

### AC-POST-03 — Admin Approval
Given a Tutor Post in PENDING status, when admin clicks Approve, then post status transitions to ACTIVE, approved at is set, expires at is set to approved at + 60 days, the post appears in /posts/browse/tutors, and the owner receives an approval email.

### AC-POST-04 — Admin Rejection
Given a Tutor Post in PENDING status, when admin selects a rejection reason and clicks Reject, then post status transitions to REJECTED, rejection reason is recorded from the enum, the post does not appear in any public listing, and the owner receives a rejection email with the reason.

### AC-POST-05 — Revision Request
Given a Tutor Post in PENDING status, when admin enters a revision message and clicks Request Revision, then post status transitions to REVISION REQUESTED, the owner receives a notification with the admin message, and the owner can edit the post and resubmit.

### AC-POST-06 — Resubmission
Given a Tutor Post in REVISION REQUESTED status, when the owner edits the post and clicks Resubmit, then post status transitions back to PENDING and the post re-enters the admin review queue.

### AC-POST-07 — Contact Info Not Exposed
Given a Tutor Post in ACTIVE status, when any unauthenticated user or a logged-in tutee requests the post detail via the public application programming interface, then the application programming interface response does not contain email encrypted, phone encrypted, or any contact field from the users record.

### AC-POST-08 — Discovery Index Restricted to Logged-In Tutors
Given a Tutee Post in ACTIVE status, when an unauthenticated user attempts to access /posts/browse/requests, then they receive a 401 response. When a logged-in tutee attempts access, they receive a 403 response.

### AC-POST-09 — Discovery Index Anonymisation
Given a Tutee Post in ACTIVE status, when an authenticated approved tutor browses the discovery index, then the response contains subject, level, location region, budget range, and availability bitmask, but does not contain the tutee's full name, email, phone, or home address.

### AC-POST-10 — Edit-While-Active Triggers Re-Review
Given a Tutor Post in ACTIVE status, when the owner submits any field change, then post status transitions to PENDING, the post is removed from /posts/browse/tutors immediately, and the owner sees a warning banner in the dashboard.

### AC-POST-11 — Post Auto-Expiry
Given a Tutor Post with status ACTIVE and expires at in the past, when the daily scheduler runs at 02:00 Singapore time, then the post status transitions to EXPIRED, the post is removed from /posts/browse/tutors, and the owner receives an expiry notification email.

### AC-POST-12 — Tutee Post Removed from Discovery on Match
Given a Tutee Post in ACTIVE status, when admin creates a match referencing that post, then the Tutee Post status transitions to MATCHED and it is no longer returned by the discovery index application programming interface.

### AC-POST-13 — Tutee Post Returns to Discovery on Match Cancellation
Given a Tutee Post in MATCHED status, when admin cancels the associated match, then the Tutee Post status transitions back to ACTIVE and it reappears in the discovery index.

### AC-POST-14 — Availability Bitmask Minimum
Given a user submitting a Tutor Post or Tutee Post, when the availability bitmask equals 0, then the submission is rejected with the error "Select at least one available time slot" and no PENDING record is created.

### AC-POST-15 — Rate Range Integrity
Given a user submitting a Tutor Post, when hourly rate min sgd > hourly rate max sgd, then the submission is rejected with the error "Maximum rate must be at least the minimum rate".

### AC-POST-16 — Budget Range Integrity
Given a user submitting a Tutee Post, when budget min sgd > budget max sgd, then the submission is rejected with the error "Maximum budget must be at least the minimum budget".

### AC-POST-17 — Draft Autosave
Given a user completing the Tutor Post creation wizard, when they navigate between wizard steps, then the current step data is persisted to a DRAFT record within 2 seconds, and the user sees a "Draft saved" indicator.

### AC-POST-18 — Profile Photo Restricted
Given a Tutor Post with a profile photo, when any user other than the post owner or an admin requests the post detail, then profile photo s3 key and any signed photo URL are absent from the application programming interface response.

### AC-POST-19 — Commission Acknowledgement Guard
Given a tutor attempting to submit a Tutor Post, when no commission acknowledgement log entry exists for that tutor, then the submission is blocked server-side with a 403 and a redirect URL to the commission acknowledgement flow.

### AC-POST-20 — Personal Data Protection Act Compliance
Given an approved Tutor Post or Tutee Post, when a Personal Data Protection Act data-access request is processed for the owning user, then all post content linked to that user's ID is included in the export, and contact fields are decrypted by admin for the export only.

---

*End of spec md — Posting Module — v1.0 — 2026-05-27*
