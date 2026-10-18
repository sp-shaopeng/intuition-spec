# InTuition — Approval Module — Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The canonical state table and revision/resubmission requirements below remain effective; APPROVED in older drip examples means canonical ACTIVE, not another enum. [021](../../021-tutee-request-flow/spec.md) allows PENDING tutee edits; [013](../../013-tutee-dashboard-and-invitations/requirements.md) retains REVISION REQUESTED editing. Delivery must follow current outbox/email/scheduler specifications, not old Spring-Async/Redis job mechanics.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail
## Singapore Private Tuition Agency

**Document Version:** 1.0
**Date:** 2026-05-27
**Status:** Draft
**Module:** Approval
**Source Documents:** business requirements document v1.1, product requirements document v1.1, schema md, design-tokens md

---

## 1. Overview

Every Tutor Post and Tutee Post submitted by a public user must pass an admin review before it becomes visible to the other side of the marketplace. This module governs the full lifecycle of that review: the admin queue, review actions, state transitions, audit logging, service-level agreement enforcement, and outbound email notifications.

The approval module is admin-only. No public user can trigger state transitions directly — they can only submit a post to PENDING via the Posting module. All transitions from PENDING onward are controlled by this module.

---

## 2. Post Status Lifecycle

### 2.1 Status Enum Values

The canonical post status PostgreSQL enum (from posting/spec md):


Implementation example omitted. This public edition retains the written requirements in this section.


This module controls transitions that involve admin decisions on PENDING and REVISION REQUESTED posts. MATCHED, CLOSED, EXPIRED are managed by the Matching and Lifecycle modules.

### 2.2 State Machine

| From State           | To State             | Actor  | Trigger                             | Logged Action      |
|----------------------|----------------------|--------|-------------------------------------|--------------------|
| PENDING            | ACTIVE             | Admin  | Admin clicks Approve                | APPROVE          |
| PENDING            | REJECTED           | Admin  | Admin clicks Reject + reason        | REJECT           |
| PENDING            | REVISION REQUESTED | Admin  | Admin clicks Request Revision + msg | REQUEST CHANGES  |
| REVISION REQUESTED | PENDING            | Owner  | Owner edits post + clicks Resubmit  | *(no log entry — owner action)* |
| ACTIVE             | PENDING            | Owner  | Owner edits a live post             | *(no log entry — owner action)* |

**Invalid transitions** (must be rejected at service layer with web request protocol 409):
- Any transition from DRAFT (post owner must submit first)
- Any transition from EXPIRED, CLOSED, MATCHED, REJECTED
- Re-approving an already-ACTIVE post (idempotency guard)

### 2.3 Transition Rules

- APPROVE: allowed from PENDING only. Transitions to ACTIVE. No rejection reason required.
- REJECT: allowed from PENDING only. rejection reason field is **mandatory** (one of 7 valid enum values). Free-text notes is optional.
- REQUEST CHANGES (REVISION REQUESTED): allowed from PENDING only. Admin provides a mandatory revision message (stored in post review log notes). Post moves to REVISION REQUESTED — owner can edit and resubmit without it counting as a fresh post.
- State is checked atomically before the update; if the current state has already changed, the application programming interface returns web request protocol 409 with INVALID STATE TRANSITION.

**Note on REOPEN:** The previous design had REOPEN (ACTIVE → PAUSED FOR REVIEW). In the canonical enum, there is no PAUSED FOR REVIEW state. If admin needs to pull an ACTIVE post back for review, they use the "Force to Pending" admin action, which transitions ACTIVE → PENDING and logs action REOPEN. The post re-enters the review queue as a normal PENDING item.

---

## 3. Admin Review Queue

### 3.1 What the Queue Shows

The queue at /admin/approval shows all posts with status PENDING, sorted **oldest submitted at first** (FIFO). This ensures the earliest submissions are reviewed first, supporting the 24-hour service-level agreement.

Each row in the queue displays:

| Column           | Source                             | Notes                                          |
|------------------|------------------------------------|------------------------------------------------|
| Post Type        | post type (TUTOR / TUTEE)        | Color-coded chip                               |
| Post Title       | tutor posts title / tutee posts title | Truncated to 60 chars                   |
| Owner Name       | users display name               | Linked to review page                          |
| Subject / Level  | tutor post subjects / tutee posts subject code + level code | Comma-separated for multi-subject tutor posts |
| Submitted        | submitted at                     | Relative time (e g. "3 hours ago") + absolute  |
| service-level agreement              | Computed from submitted at       | Green / Amber / Red badge (see §6)             |

### 3.2 Queue Filters

Admins may filter the queue by:
- **Post type**: All / Tutor / Tutee
- **service-level agreement status**: All / On Track / Approaching / Overdue
- **Subject** (free-text search on subject name)
- **Level** (dropdown from level reference table)

Filters are shallow URL params (?type is TUTOR&sla is OVERDUE) so they can be bookmarked.

### 3.3 Sort Order

Default sort: submitted at ASC (oldest first). No alternative sort exposed in v1 — FIFO is enforced by design to maintain service-level agreement fairness.

### 3.4 Pagination

20 posts per page, cursor-based (keyed on submitted at + id). No offset pagination. Cursor is base64-encoded and passed as ?cursor=`<value>`.

---

## 4. Per-Post Review Workflow

### 4.1 Review Page Layout

At /admin/approval/ id, the admin sees the full post details alongside the action panel. Layout is two-column on desktop: left panel (post details + owner contact info), right panel (action form).

### 4.2 Post Details Panel

Displays all post fields relevant to content review:

**For Tutor Post:**
- Title, headline
- Subjects (all), levels (all), venue types (all), group sizes
- Hourly rate range (SGD)
- Years of experience, qualifications text, teaching approach
- Availability bitmask (decoded to human-readable schedule)
- Location (region, planning area, MRT station)
- Profile photo (rendered from signed S3 URL, 1-hour expiry)

**For Tutee Post:**
- Title (auto-generated if blank)
- Subject, level
- Budget range (SGD), session duration, frequency per week
- Number of students, student relationship
- Preferred start date, special requirements
- Location (region, planning area, MRT station)
- Availability bitmask (decoded)

### 4.3 Owner Contact Info Panel

Admin sees the post owner's contact details, decrypted for review purposes. This panel is styled with color Admin Bg Deep (#EDE9 FF) to visually distinguish it from post content.

| Field        | Source                             |
|--------------|------------------------------------|
| Display name | users display name               |
| Email        | Decrypted from users email encrypted |
| Phone        | Decrypted from users phone encrypted (if present) |
| User ID      | users id (for cross-referencing) |
| Registered   | users created at                 |
| Post count   | Count of other posts by same user  |

Contact info is **never** included in public-facing application programming interface responses. It is only available via the admin-scoped review endpoint (see backend md §3).

### 4.4 Action Panel

The action panel shows:
1. Current post status badge
2. **Approve** button (primary, green)
3. **Reject** button (destructive, red)
4. If post is APPROVED: **Reopen** button (warning, amber) — moves to PAUSED FOR REVIEW

Clicking Approve or Reject opens a confirmation modal before the application programming interface call is made.

---

## 5. Rejection Reasons

### 5.1 Valid Rejection Reason Codes

Rejection reason is **mandatory** when rejecting. The admin must select exactly one from the following enum:

| Code                          | Display Label (Admin user interface)                    | When to Use                                                  |
|-------------------------------|---------------------------------------------|--------------------------------------------------------------|
| INAPPROPRIATE CONTENT       | Inappropriate Content                       | Offensive language, spam, adult content, discriminatory text  |
| MISSING INFORMATION         | Missing Information                         | Required fields are vague, blank, or placeholder text         |
| UNVERIFIABLE QUALIFICATIONS | Unverifiable Qualifications                 | Claims cannot be substantiated (e g. "Ph D from Harvard" with no detail) |
| DUPLICATE POST              | Duplicate Post                              | Same or near-identical post already exists from same user     |
| PRICING OUT OF RANGE        | Pricing Out of Range                        | Rate is unrealistically low (< $10/hr) or unrealistically high (> $500/hr) for Singapore market |
| INCORRECT CATEGORY          | Incorrect Category                          | Subject or level does not match post content                  |
| OTHER                       | Other (specify in notes)                    | Use with mandatory notes explaining the reason                |

### 5.2 Notes Field

notes is a free-text TEXT field. It is:
- **Optional** for all rejection reasons except OTHER, where a clarifying note is strongly recommended (enforced by frontend validation warning, not hard block).
- **Never exposed** to the post owner in the notification email — only rejection reason display label is shown to users. Notes are internal admin records.
- Stored in post review log notes.

### 5.3 User-Facing Rejection Email (N-04)

The notification email (N-04) sent to the post owner includes:
- The display label of the rejection reason (not the code)
- A standard paragraph explaining they can edit and resubmit
- Link to their post in the owner dashboard

---

## 6. service-level agreement Policy

### 6.1 Target

All PENDING posts must be reviewed within **24 hours** of submitted at. This is a design target, not a contractual service-level agreement with users.

### 6.2 service-level agreement Indicator States

Based on the elapsed time since the post was submitted:

| Age                  | service-level agreement State    | Token              | Hex       | user interface Treatment                           |
|----------------------|--------------|--------------------|-----------|----------------------------------------|
| < 24 hours           | On track     | color Success     | #16 A34 A | Green dot + relative time              |
| 24 hours to 48 hours | Approaching  | color Warning     | #D97706 | Amber dot + "X hrs overdue" label      |
| > 48 hours           | Breached     | color Error       | #DC2626 | Red badge "OVERDUE" + hours overdue    |

service-level agreement is computed at query time; no separate column is stored. The queue page refreshes service-level agreement indicators every 5 minutes via React Query polling.

### 6.3 Escalation

When a post has been PENDING for > 48 hours with no review action, the system should surface this prominently. In v1, this is handled by the red OVERDUE badge only. Automated escalation alerts to a supervisor email are tracked as a v2 enhancement.

---

## 7. Notification Triggers

### 7.1 Approval Email (N-03)

Triggered immediately when a post transitions to APPROVED. Queued to InTuition async email queue (Spring @Async + Redis) within the approve transaction commit.

**Recipient:** Post owner (users email encrypted, decrypted at queue-push time)
**Subject:** "Your [tutor/tutee] listing has been approved"
**Content includes:**
- Post title
- Link to the live listing
- What happens next (waiting period explanation)
- Day 0 content: approval confirmation + how matching works

### 7.2 Rejection Email (N-04)

Triggered immediately when a post transitions to REJECTED. Queued to InTuition async email queue (Spring @Async + Redis) within the reject transaction commit.

**Recipient:** Post owner
**Subject:** "Action needed: your [tutor/tutee] listing requires updates"
**Content includes:**
- Post title
- Rejection reason (display label only — internal code not shown)
- Guidance based on rejection reason
- Link to edit the post and resubmit
- Contact admin link for queries

### 7.3 Waiting Period Drip Email Schedule

After a post is APPROVED, the following scheduled emails are queued at the time of approval (using Spring a scheduled task delayed tasks (Redis-backed)):

| Day | Email ID | Trigger Condition            | Content                                           |
|-----|----------|------------------------------|---------------------------------------------------|
| 0   | N-03     | Immediate on approval        | Approval confirmed + what happens next            |
| 3   | N-05     | 3 days after approval        | Check-in: "We're actively matching you"            |
| 7   | N-06     | 7 days after approval        | Still looking: tips to improve visibility         |
| 14  | N-07     | 14 days after approval       | Shortlisting email: "You've been shortlisted X times" |
| 21  | N-08 (admin) | 21 days after approval   | Admin escalation alert: post unmatched > 3 weeks  |

Delayed jobs for Day 3, 7, 14, and 21 are created atomically with the approval action. If a post transitions out of APPROVED before a delayed job fires (e g. matched, closed, or re-opened), the job must check current post status before sending and abort if no longer APPROVED.

---

## 8. Audit Log Requirements

### 8.1 Immutability

Every review action is recorded in post review log. The table is append records-only at the database level (GRANT append records, read fields to app role — no update records or DELETE). This is enforced at the PostgreSQL permission layer, not just the application layer, per business requirements document §10.6.2.

### 8.2 Required Fields per Log Entry

| Field              | Required | Notes                                                    |
|--------------------|----------|----------------------------------------------------------|
| post id          | Yes      | unique identifier of the reviewed post                                |
| post type        | Yes      | TUTOR or TUTEE                                   |
| reviewed by      | Yes      | unique identifier of the admin who performed the action               |
| action           | Yes      | APPROVE, REJECT, REOPEN, or REQUEST CHANGES      |
| rejection reason | Conditional | Mandatory when action REJECT. NULL otherwise.  |
| notes            | No       | Internal notes. Optional for all actions.                |
| reviewed at      | Yes      | Set to the now operation at append records time (default)                   |

### 8.3 Log Retention

post review log rows are retained indefinitely. They are never anonymised or purged — admin actions are not personal information subject to Personal Data Protection Act right-of-erasure. If an admin account is hard-deleted, reviewed by is set to NULL (ON DELETE SET no value) so the log row is preserved.

### 8.4 Review History Page

At /admin/approval/history, admins can view and filter the full post review log. Accessible by all admins. Supports filtering by action type, admin user, date range, and post type.

---

## 9. Admin Authorization Requirements

- All approval endpoints require role ADMIN in the session.
- Admin sessions require time-based one-time passcode multi-factor authentication at login and enforce a **30-minute idle timeout**.
- Sessions past the idle timeout redirect to re-authentication before any review action can be performed.
- All review actions are attributed to the authenticated admin's user id — shared admin logins are not permitted.

---

## 10. Acceptance Criteria

### AC-APPR-01 — Queue shows only PENDING posts

Given: Posts exist in various states (DRAFT, PENDING, APPROVED, REJECTED)
When: Admin loads /admin/approval
Then: Only posts with status PENDING appear in the queue

### AC-APPR-02 — Queue is sorted oldest first

Given: Multiple PENDING posts with different submitted at timestamps
When: Admin loads the queue with no sort override
Then: The oldest submitted at post appears first (FIFO order)

### AC-APPR-03 — service-level agreement indicator is green for posts < 24 hours old

Given: A PENDING post with submitted at = 10 hours ago
When: Admin views the queue
Then: A green service-level agreement indicator is shown for that post

### AC-APPR-04 — service-level agreement indicator is amber for posts 24–48 hours old

Given: A PENDING post with submitted at = 30 hours ago
When: Admin views the queue
Then: An amber service-level agreement indicator showing "6 hrs overdue" is shown

### AC-APPR-05 — service-level agreement indicator is red for posts > 48 hours old

Given: A PENDING post with submitted at = 60 hours ago
When: Admin views the queue
Then: A red OVERDUE badge is shown

### AC-APPR-06 — Approve transitions post to APPROVED

Given: A PENDING tutor post
When: Admin clicks Approve and confirms the modal
Then: tutor posts status = APPROVED, approved at is set, reviewed by is set to admin unique identifier, and an N-03 email is queued

### AC-APPR-07 — Reject requires rejection reason

Given: A PENDING post
When: Admin clicks Reject without selecting a rejection reason
Then: The confirmation modal Confirm button is disabled; the application programming interface rejects the request with web request protocol 422 if called directly

### AC-APPR-08 — Reject with valid reason transitions to REJECTED

Given: A PENDING tutee post
When: Admin selects MISSING INFORMATION and confirms rejection
Then: tutee posts status = REJECTED, rejection reason is stored, N-04 email is queued

### AC-APPR-09 — Every review action is logged in post review log

Given: Admin approves post P1 and rejects post P2
When: Admin checks /admin/approval/history
Then: Two log entries appear, each with correct post id, reviewed by, action, and reviewed at

### AC-APPR-10 — Reopen moves APPROVED post to PAUSED FOR REVIEW

Given: An APPROVED tutor post
When: Admin clicks Reopen and confirms
Then: tutor posts status = PAUSED FOR REVIEW, a REOPEN log entry is created, post disappears from public listing

### AC-APPR-11 — Double-approve is rejected

Given: Post P1 is already APPROVED
When: Admin attempts to approve P1 again (e g. browser back button + re-submit)
Then: application programming interface returns web request protocol 409 with error code INVALID STATE TRANSITION

### AC-APPR-12 — Contact info visible on review page

Given: Admin is on /admin/approval/ id
When: Page loads
Then: Owner's decrypted email and phone are visible in the contact panel

### AC-APPR-13 — Admin notes are not sent to post owner

Given: Admin rejects a post with rejection reason OTHER and notes = "Suspected bot account"
When: Post owner receives N-04 rejection email
Then: Email contains the rejection reason label ("Other") but does NOT contain the admin notes text

### AC-APPR-14 — Day 3 drip email is not sent if post is matched before Day 3

Given: Post is approved and Day 3 delayed job is queued
When: Post transitions to MATCHED on Day 2
Then: Day 3 drip job checks post status, finds it is no longer APPROVED, and skips sending

### AC-APPR-15 — Review history is filterable by admin

Given: Admin A reviewed 5 posts, Admin B reviewed 3 posts
When: Admin loads /admin/approval/history?reviewed by= Admin A unique identifier
Then: Only Admin A's 5 log entries are shown

### AC-APPR-16 — time-based one-time passcode session required for review actions

Given: Admin session has been idle for > 30 minutes
When: Admin attempts to approve a post
Then: Admin is redirected to re-authenticate before the action is processed

### AC-APPR-17 — Approved post with PAUSED FOR REVIEW status appears in queue

Given: An APPROVED post that was reopened (now PAUSED FOR REVIEW)
When: Admin loads /admin/approval
Then: The post does NOT appear in the PENDING queue (queue only shows PENDING)

> Note: PAUSED FOR REVIEW posts are reviewed via a separate filter or the direct URL. In v1, the queue filters only PENDING. Admin reaches PAUSED FOR REVIEW posts via the review history or direct link.
