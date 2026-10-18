# InTuition — Approval Module — Frontend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

**Module:** approval  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Routes Overview

| Route | Auth | Description |
|---|---|---|
| /admin/approval | ADMIN | Pending posts queue |
| /admin/approval/ type / id | ADMIN | Review a single post |
| /admin/approval/history | ADMIN | Review history log |

All admin pages use the admin panel layout: fixed sidebar navigation + main content area. Sidebar nav links: Dashboard, Approval Queue, Matching, Commissions, Settings.

---

## 2. Approval Queue (/admin/approval)

### 2.1 Page Layout


The illustration for page layout contains the following information:

- Admin Panel [Admin Name ▼].
- Approval Queue [Tutor Tutee All].
- Links 23 posts pending review.
- • Queue [Table: see below].
- • Match.
- • Comm [Pagination].
- • Config.


### 2.2 Queue Table

Columns: Type | Posted By | Subject using s / Level using s | Submitted | Waiting | service-level agreement | Action


The illustration for queue table contains the following information:

- Type Posted By Subject / Level Submitted Waiting service-level agreement Action.
- TUTOR John T. Math, Phys • S3–S4 27 May, 10:30 14h ● Amber [Review].
- TUTEE Parent (Bishan) English • P5 27 May, 08:00 17h ● Amber [Review].
- TUTOR Sarah L. Chinese • P1–P6 26 May, 14:00 42h ● Red [Review].


**service-level agreement Indicator:**
- Green dot ● — < 24 hours since submission
- Amber dot ● — 24–48 hours
- Red dot ● — > 48 hours (overdue)

**Sort:** Default oldest first (highest urgency at top). User can re-sort by clicking column headers.

**Tabs (status filter):** "Pending Review" (PENDING) / "Needs Revision" (REVISION REQUESTED) / "All". Sub-tabs by type: "All" / "Tutor" / "Tutee".

The "Needs Revision" tab surfaces REVISION REQUESTED posts — posts where admin already requested changes and is waiting for the owner to resubmit. Without this tab these posts would be invisible in the queue.

**Empty state:** "No posts pending review. You're all caught up! 🎉" shown when queue is empty.

---

## 3. Post Review Page (/admin/approval/ type / id)

### 3.1 Layout

Two-panel layout (desktop ≥1024px):
- Left panel (60%): Post details + poster information
- Right panel (40%): Action panel (approve/reject)

Mobile (<1024px): Stacked — details first, action panel below.

### 3.2 Left Panel — Post Details

**Header section:**


The illustration for left panel — post details contains the following information:

- ← Back to Queue TUTOR POST — PENDING [service-level agreement: 14h • ● Amber].


**Poster Information (admin-only section, visually distinct with purple tint background):**


The illustration for left panel — post details contains the following information:

- 🔒 Poster Contact (Admin Only).
- Name: John Tan Wei Ming.
- Email: [REDACTED EMAIL].
- Phone: [REDACTED PHONE].
- User since: 15 Jan 2025.


**Post content sections:**
- Subjects: "Mathematics (O-Level), Physics (O-Level)"
- Levels: "Secondary 3 (Express), Secondary 4 (Express)"
- Rate: "S$35 – S$50/hr"
- Venue: "Student's Home, Online"
- Group Size: "1-on-1"
- Availability: Read-only availability grid (same component as creation)
- Experience: "5 years"
- Qualifications: "B Sc (Physics), NTU. MOE-trained teacher. 5 years private tutoring."
- Teaching Approach: "Concept-first approach. Exam technique coaching for O-Level papers."
- Location: "East Coast (D15)"

### 3.3 Right Panel — Review Actions


The illustration for right panel — review actions contains the following information:

- Review Actions.
- [✓ Approve Post].
- Rejection Reason *.
- Select reason... ▼.
- Additional Notes (optional).
- [✗ Reject Post].
- Previous Reviews.
- No prior reviews for this post.


**Rejection reason dropdown options:**
1. Inappropriate Content
2. Missing Information
3. Unverifiable Qualifications
4. Duplicate Post
5. Pricing Out of Range
6. Incorrect Category
7. Other

**Approve button** → Confirmation modal:


The illustration for right panel — review actions contains the following information:

- Confirm Approval?.
- "This post will go live and be visible to potential matches.".
- [Cancel] [Yes, Approve].


**Reject button** → Requires reason selected → Confirmation modal:


The illustration for right panel — review actions contains the following information:

- Confirm Rejection?.
- "John T. will be notified with reason: Missing Information".
- [Cancel] [Yes, Reject].


**Request Revision button** (amber, between Approve and Reject):


The illustration for right panel — review actions contains the following information:

- Revision Message *.
- "Please clarify which exam board...".
- [Cancel] [Send Revision Request].


The revision message is sent to the post owner and stored in post review log notes. Post moves to REVISION REQUESTED status.

**Force to Pending button** (shown only when post is ACTIVE): pulls post back to PENDING for re-review. Requires a reason note. Used when admin receives a complaint about a live post.

**Keyboard shortcuts** (shown in tooltip on hover):
- A = Approve (opens confirmation modal)
- R = Focus rejection reason dropdown
- V = Open revision request form

---

## 4. Review History (/admin/approval/history)

### 4.1 Filters

Date range picker (from/to), Action (Approve/Reject/Reopen), Admin (dropdown of all admin users), Post type (All/Tutor/Tutee).

### 4.2 History Table


The illustration for history table contains the following information:

- Date & Time Admin Post Type Subject Action Reason.
- 27 May 10:45 Admin A TUTOR Math • S3–4 APPROVED —.
- 27 May 10:20 Admin A TUTEE English • P5 REJECTED Missing Info.
- 26 May 16:00 Admin B TUTOR Chinese • P1 APPROVED —.


Click any row → opens read-only view of that review (post snapshot + action taken).

---

## 5. Notification Toast States

All approval actions show inline toasts (top-right, 4s auto-dismiss):

- Approve success: Post approved. John T. has been notified.
- Reject success: Post rejected. Reason sent to John T.
- Error (network): Something went wrong. Please try again.
- Idempotency error (already reviewed): This post has already been reviewed. Refresh the queue.

---

## 6. Waiting Period Email Status (Informational Only)

In the post detail view for APPROVED posts, show the email drip status tracker:


The illustration for . waiting period email status informational only contains the following information:

- Post Approval Email Schedule.
- Day 0 ✓ Approval email sent (27 May 10:45).
- Day 3 ○ Check-in email (scheduled 30 May).
- Day 7 ○ Still shortlisting (3 Jun).
- Day 14 ○ Shortlisting update (10 Jun).
- Day 21 ○ Admin escalation alert (17 Jun).


Admins use this to know when to intervene if a match is taking long.
