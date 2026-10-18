> **Current-file check, 18 October 2026:** The retained first-session confirmation journey is absent from the current website and server. Match completion on agency receipt is present (G02). See the [alignment review](../../../docs/current-file-alignment.md). The requirements remain unchanged.

# InTuition — Matching Module Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** PRD20/INT034 supersede mandatory-listing/admin-only-expression and never-share-contact assumptions. PAID means the agency received commission from the tutor; first-session confirmation alone does not complete the match. Earlier detailed enums/database structure definitions examples are historical where they differ.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Module:** matching  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x  
**Owner:** Tech Lead  
**business requirements document References:** BR-001, BR-003, BR-004, BR-010  
**product requirements document References:** §7, §8, §9 (N-10–N-13), AC-MATCH-01–AC-MATCH-18

---

## 1. Overview

The Matching module is the core operational workflow of the Singapore tuition agency platform. A **match** is an admin-created record linking exactly one approved tutor post to exactly one approved tutee post, representing the agency's formal introduction of a tutor to a tutee. All matches are created exclusively by admin staff — there is no self-selection, algorithmic ranking, or marketplace browsing by tutors for assignment initiation (BR-003).

### 1.1 Core Principles

- **Admin is the gatekeeper.** Tutors may apply and parents may select/request a tutor; only authorized admin confirmation creates the binding assignment under product requirements document §20. Listing publication is optional.
- **Contact information stays private.** Public/applicant/interview-negotiation surfaces remain personal information-free. The existing assignment-stage authorized disclosure gate applies; never claim contact is never shared. Contact is restricted to authorized assigned participants; frontend reveals only for ACTIVE/COMPLETED and hides match-PENDING. Post approval, invitations, interviews and tutor selection do not release contact (current baseline C04).
- **One active arrangement per tutee post.** A tutee post may only be in one PENDING or ACTIVE match at a time. This prevents double-booking a tutee and enforces the agency's curated matching promise.
- **Tutors can scale.** A tutor post may participate in multiple simultaneous matches, reflecting that professional tutors often serve multiple students.
- **Commission follows first session confirmation.** The commission lifecycle is tracked separately on the commissions table. Match status (PENDING/ACTIVE/COMPLETED/CANCELLED) and commission status are independent dimensions.

### 1.2 Scope

- Admin matching dashboard (two-panel post selector)
- Match creation with prerequisite validation
- Match state machine (PENDING → ACTIVE → COMPLETED / CANCELLED)
- First session confirmation flow (BR-010)
- 7-day tutee confirmation timeout and admin escalation
- Notification triggers (N-10, N-11, N-12, N-13)
- Cancellation at any status with required reason
- Tutee post visibility management (MATCHED state in discovery index)

### 1.3 Out of Scope

- Commission calculation, invoice generation, and payment tracking (see commission module)
- Post creation, editing, and admin approval (see posting module)
- User registration and authentication (see auth module)
- SMS or in-platform notifications (Phase 2)

---

## 2. What Is a Match?

A match is a record in the matches table that:

1. Links a **single approved tutor post** (tutor post id) to a **single approved tutee post** (tutee post id)
2. Is created by an **admin user** (created by admin id)
3. Carries a lifecycle status (PENDING → ACTIVE → COMPLETED | CANCELLED)
4. May store an agreed rate sgd (the actual negotiated hourly rate, which may differ from the tutor post's stated min–max range) and an agreed rate sgd override for commission calculation purposes
5. Has an optional admin notes free-text field for internal coordinator notes

The match does **not** contain contact information. Contact details reside on the users table and are accessible only through the admin contact reveal endpoint (logged to audit log).

---

## 3. Match State Machine

### 3.1 States

| State | Description |
|-------|-------------|
| PENDING | Match created by admin. Both parties notified. Admin is conducting off-platform outreach. Tutee post is in MATCHED status and hidden from tutor discovery. |
| ACTIVE | Admin has confirmed that both parties have been contacted and the arrangement is proceeding. First session confirmation request (N-12) sent to tutee. Commission record created with status PENDING; invoicing/due handling retains first-session confirmation. |
| COMPLETED | Agency received tutor commission and admin marked PAID, atomically completing the match. First-session confirmation alone does not complete it; COMPLETED does not mean all teaching has ended. |
| CANCELLED | Admin has cancelled the match at any prior status. Cancellation reason is mandatory. Tutee post is returned to ACTIVE status and re-enters tutor discovery index. |

### 3.2 Historical state diagram — current transitions are in §3.3

The original first-session→COMPLETED and mandatory-two-post diagram below is retained as history; it is superseded by the corrected state table and PRD20/INT034.


The illustration for historical state diagram — current transitions are in §3.3 contains the following information:

- [PRE-MATCH].
- tutor post.
- APPROVED.
- tutee post.
- [admin: POST /admin/matches].
- PENDING.
- [admin: activate] [admin: cancel reason].
- POST .../activate.
- ACTIVE ▶ CANCELLED.
- [tutee confirms first session.
- OR admin override after 7 days].
- POST .../complete [admin: cancel reason].
- CANCELLED COMPLETED.


### 3.3 Valid Transitions

| From | To | Actor | Guard | Side Effects |
|------|----|-------|-------|--------------|
| (none) | PENDING | ADMIN | Canonical tutor profile/action eligibility holds, tutee post is ACTIVE and non-deleted; attached listing is optional; no existing live match for the job | Tutee post status → MATCHED; N-10 sent to tutor; N-11 sent to tutee; commission record NOT yet created |
| PENDING | ACTIVE | ADMIN | Match must be in PENDING status | activated at = NOW operation; N-12 sent to tutee; commission record created with status PENDING; 7-day countdown starts |
| PENDING | CANCELLED | ADMIN | Cancellation reason must be provided | cancelled at = NOW operation; tutee post status → APPROVED; tutee post re-enters discovery index; N-14 sent to both parties (future: notification TBD) |
| ACTIVE | COMPLETED | ADMIN | First-session confirmation precedes collection; agency has received the tutor commission and admin marks PAID | Atomically commission PENDING→PAID and match→COMPLETED; confirmation alone does not complete the match |
| ACTIVE | CANCELLED | ADMIN | Cancellation reason must be provided | cancelled at = NOW operation; tutee post status → APPROVED; re-enters discovery; existing unpaid commission is cancelled atomically with the match under the current assignment lifecycle; paid/terminal recovery policy is not invented here |

### 3.4 Invalid Transitions (Rejected by Match Service)

- COMPLETED → any state (terminal)
- CANCELLED → any state (terminal)
- ACTIVE → PENDING (no backward transitions)
- PENDING → COMPLETED (must pass through ACTIVE; first session confirmation requires ACTIVE state)

---

## 4. Cardinality Constraints

### 4.1 Tutee Post: Maximum 1 PENDING or ACTIVE Match

A tutee post can have **at most one match** in either PENDING or ACTIVE status at any given moment.

**Enforcement mechanism:** A partial unique index on the matches table:


Implementation example omitted. This public edition retains the written requirements in this section.


This constraint is enforced at two layers:
1. **Application layer:** the create operation queries for existing PENDING/ACTIVE matches on the tutee post before append records and returns a 409 Conflict if one exists.
2. **Database layer:** The partial unique index provides a hard guarantee even under concurrent requests.

**Implication:** If a match is CANCELLED, the tutee post can be re-matched. Historical CANCELLED and COMPLETED matches for the same tutee post co-exist in the matches table without violating the constraint (only one non-terminal match at a time).

### 4.2 Tutor Post: Unlimited Simultaneous Matches

A tutor post may participate in any number of simultaneous PENDING or ACTIVE matches. There is no index or application-layer check limiting tutor post match count. This supports tutors who accept multiple students concurrently.

**Display implication:** On the admin matching dashboard, a tutor post is never visually grayed out due to existing matches. The tutor card may display a "Currently in N matches" badge as informational context.

---

## 5. Match Creation Prerequisites

Before admin can create a match, **all of the following must be true**:

1. The **tutor post** has status APPROVED (not DRAFT, PENDING, REJECTED, MATCHED, CLOSED, or EXPIRED)
2. The **tutee post** has status APPROVED (same constraints)
3. The **tutee post** does not currently have a PENDING or ACTIVE match (partial unique index check)
4. The **admin user** has role ADMIN and an active session
5. An optional agreed rate sgd is within a plausible range (application-layer soft warning if outside the tutor post's stated min–max; not a hard block)

Failing conditions return the following errors:
- Tutor post not APPROVED → 422 Unprocessable Entity: TUTOR POST NOT APPROVED
- Tutee post not APPROVED → 422 Unprocessable Entity: TUTEE POST NOT APPROVED
- Tutee post already has PENDING/ACTIVE match → 409 Conflict: TUTEE POST ALREADY MATCHED

---

## 6. Admin Matching Workflow

### 6.1 Two-Panel Dashboard

The admin matching dashboard (/admin/matching) presents a two-panel layout:

- **Left panel — Tutee Posts:** All tutee posts with status APPROVED (no current PENDING/ACTIVE match). Filterable by subject, level, budget range, venue preference, location region.
- **Right panel — Tutor Posts:** All tutor posts with status APPROVED. Filterable by subject, level, hourly rate range, venue types, location region, availability overlap with selected tutee post.

Tutee posts that already have a PENDING or ACTIVE match are **not shown** in the left panel — they are in MATCHED status and removed from the matchable pool.

### 6.2 Match Creation Flow

1. Admin selects a tutee post from the left panel (row click or "Select" button).
2. The right panel auto-filters to show tutor posts with overlapping subject and level (soft suggestion; admin can clear filters).
3. Admin selects a tutor post from the right panel.
4. A **compatibility preview** modal opens showing:
   - Tutee post summary: subject, level, budget range, venue prefs, availability bitmask overlap
   - Tutor post summary: subjects, levels, rate range, venue types, location
   - Availability overlap indicator (bitwise AND of both bitmasks)
   - Any existing active matches for the selected tutor post (informational)
5. Admin optionally enters agreed rate sgd and admin notes.
6. Admin clicks "Create Match" → confirmation dialog → application programming interface call.
7. On success: both posts show updated status; notification queue entries created.

### 6.3 Post-Match Tutee Post State

Once a tutee post is matched (PENDING or ACTIVE match exists):
- Tutee post status → MATCHED
- Tutee post is removed from the tutor discovery index (/posts/tutees)
- Tutee post is **not expired** — it retains all data and the match link
- Tutee post cannot be re-matched while in MATCHED state
- If the match is CANCELLED, tutee post status reverts to APPROVED and re-enters discovery

The tutee's own dashboard (/dashboard/my-matches) shows the match status badge but does not show tutor contact information.

---

## 7. First Session Confirmation Flow (BR-010)

### 7.1 Trigger

When admin activates a match (PENDING → ACTIVE), the system:
1. Creates a session confirmation tokens record: token hash, match id, an expiry seven days after issuance, confirmed at no value, used no
2. Generates a signed signed session token confirmation URL (7-day expiry period): the protected first-session confirmation link
3. Sends N-12 email to the tutee with the confirmation link and a 7-day deadline

### 7.2 Tutee Confirmation

The tutee receives an email with a "Confirm First Session" button. Clicking it:
1. Opens the protected first-session confirmation link (public, unauthenticated page — tutee may not be logged in)
2. System validates the token: checks token hash, the expiry timestamp must be later than the current time, used no
3. If valid: mark the token used and record the current confirmation time. The older design then completed the match; C03 replaces that transition: completion requires agency commission to be PAID.
4. Commission module is triggered: commission status → CONFIRMED
5. Success page shown: "Thank you for confirming. Your coordinator will follow up shortly."

If the token is invalid or expired:
- Show an error page: "This confirmation link has expired or is no longer valid. Please contact us."
- Admin is prompted to resend or manually confirm via admin panel.

### 7.3 7-Day Timeout and Admin Escalation

A scheduled job (runs nightly at 02:00 Singapore time) checks for:


Implementation example omitted. This public edition retains the written requirements in this section.


For each expired unconfirmed token, the scheduler:
1. Sends N-13 (admin escalation alert) to the admin team
2. Creates a system notifications record for tracking
3. Does **not** automatically advance match status — admin must manually decide to override

### 7.4 Admin Override

Admin can manually mark the first session as confirmed from the match detail page (/admin/matches/:id). This:
1. Calls submit operation for admin matches :id complete with an explicit administrator override and its required reason
2. Marks the session confirmation token as used (if one exists)
3. Transitions match → COMPLETED
4. Commission → CONFIRMED

---

## 8. Cancellation

### 8.1 Who Can Cancel

Only admin can cancel a match. Tutors and tutees cannot cancel a match from their own dashboards.

### 8.2 Cancellation at Any Status

| Match Status at Cancellation | Post-Cancellation Behaviour |
|------------------------------|----------------------------|
| PENDING | Tutee post → APPROVED; re-enters discovery; no commission record to handle |
| ACTIVE | Tutee post → APPROVED; re-enters discovery; if commission record exists (should be at AWAITING CONFIRMATION), admin must set it to WAIVED or DISPUTED |
| COMPLETED or CANCELLED | Terminal; cannot be cancelled again |

### 8.3 Cancellation Requirements

- cancellation reason is **mandatory** (validated non-empty string, 500 chars max)
- cancelled at is set to NOW operation
- Admin can include a public-facing reason (future: notification to parties)

### 8.4 Tutee Post Recovery After Cancellation

When a match is cancelled:
1. tutee posts status is set back to APPROVED
2. The partial unique index constraint is released (CANCELLED is not in the partial index scope)
3. The tutee post re-appears in the tutor discovery index
4. The tutee post re-appears in the admin matching dashboard left panel
5. The tutee is not notified automatically in MVP (admin should follow up off-platform)

---

## 9. Notification Triggers

| Notification | Template | Trigger | Recipient | Content |
|-------------|----------|---------|-----------|---------|
| N-10 | match-created-tutor | Match status → PENDING (on create) | Tutor (owner of tutor post) | "Great news! We found a match for your post. Our coordinator will contact you within 1 business day." No contact info. |
| N-11 | match-created-tutee | Match status → PENDING (on create) | Tutee (owner of tutee post) | "Great news! We found a tutor. Our coordinator will contact you within 1 business day." No contact info. |
| N-12 | first-session-confirm | Match status → ACTIVE (on activate) | Tutee | Confirmation link, subject/level context, 7-day deadline. No tutor contact info. |
| N-13 | admin-escalation-timeout | 7 days after activation, tutee has not confirmed | Admin team | Match ID, tutee post summary, activation date, link to admin match detail. |

All notifications are queued as system notifications records with status QUEUED. The notification worker processes them asynchronously via Amazon Web Services SES. No contact personal information (email address, phone) is included in payload json.

---

## 10. Acceptance Criteria

| ID | Criterion |
|----|-----------|
| AC-MATCH-01 | Admin can create a match only when both the tutor post and tutee post have status APPROVED. Attempting to match a post with any other status returns 422. |
| AC-MATCH-02 | Admin cannot create a second PENDING or ACTIVE match for a tutee post that already has one. The system returns 409 Conflict with error code TUTEE POST ALREADY MATCHED. |
| AC-MATCH-03 | A single tutor post can simultaneously be in PENDING or ACTIVE matches with multiple tutee posts. No cardinality limit applies to the tutor post side. |
| AC-MATCH-04 | Upon match creation, the tutee post status transitions to MATCHED and is removed from the tutor discovery index within the same database transaction. |
| AC-MATCH-05 | Upon match creation (status = PENDING), notification N-10 is queued for the tutor and N-11 is queued for the tutee. Neither notification contains a phone number or email address. |
| AC-MATCH-06 | Admin can activate a match (PENDING → ACTIVE). Upon activation, notification N-12 is queued for the tutee with a valid confirmation link (7-day expiry period). |
| AC-MATCH-07 | The session confirmation link in N-12 is a public URL (the protected first-session confirmation link) that does not require the tutee to be logged in. |
| AC-MATCH-08 | When a tutee clicks a valid, unexpired confirmation link, the match transitions to COMPLETED and the session confirmation token is marked used yes in an atomic operation. |
| AC-MATCH-09 | A confirmation link used after its 7-day expiry returns a user-friendly error page and does not advance the match status. |
| AC-MATCH-10 | A confirmation token cannot be reused. A second click on a used yes token returns the error page. |
| AC-MATCH-11 | If the tutee has not confirmed within 7 calendar days of activation, the nightly scheduler sends N-13 (admin escalation alert) exactly once per unconfirmed match. |
| AC-MATCH-12 | Admin can override first session confirmation via the admin match detail page, advancing the match to COMPLETED even if the tutee has not clicked the link. An override reason is required. |
| AC-MATCH-13 | Admin can cancel a match at PENDING or ACTIVE status. Cancellation requires a non-empty cancellation reason. |
| AC-MATCH-14 | When a PENDING match is cancelled, the tutee post status reverts to APPROVED and re-appears in both the tutor discovery index and the admin matching dashboard left panel. |
| AC-MATCH-15 | When an ACTIVE match is cancelled, the same tutee post recovery applies. If a commission record exists, its status is not automatically changed — admin must manually resolve it. |
| AC-MATCH-16 | COMPLETED and CANCELLED matches are terminal states. No further status transitions are accepted by the application programming interface (returns 422). |
| AC-MATCH-17 | The admin matching dashboard left panel only shows tutee posts with status APPROVED and no existing PENDING/ACTIVE match. Tutee posts in MATCHED status are not displayed. |
| AC-MATCH-18 | The "View Contact Info" action on any match detail page creates an immutable entry in audit log with event type CONTACT FIELD ACCESS, resource type match, resource id match id, and fields accessed indicating which fields were viewed. |

---

## 11. Dependencies

| Dependency | Purpose |
|------------|---------|
| posting module | Provides approved tutor and tutee posts; manages post status updates (MATCHED, APPROVED revert) |
| commission module | Creates commission record on match activation; receives COMPLETED signal to advance commission to CONFIRMED |
| auth module | Admin authentication and role enforcement; signed session token validation on all admin endpoints |
| audit log table | Records admin contact info access (append records-only) |
| system notifications table | Queues N-10, N-11, N-12, N-13 email notifications |
| session confirmation tokens table | Stores first-session confirmation tokens for BR-010 flow |
| Amazon Web Services SES ap-southeast-1 | Email delivery for all matching notifications |
| Redis | Rate limiting on confirmation link endpoint (prevent token enumeration) |
| Spring a scheduled task | 7-day timeout check job (nightly at 02:00 Singapore time) |

---

## 12. Open Questions

| # | Question | Owner | Due |
|---|----------|-------|-----|
| OQ-M-01 | Should admin receive notification when a match is created (confirmation to the creating admin)? Currently no such notification is specified. | Product | TBD |
| OQ-M-02 | When a CANCELLED match frees a tutee post back to APPROVED, should the tutee be notified? Currently no N-xx is defined for this event. | Product | TBD |
| OQ-M-03 | Should the N-13 escalation alert go to all admin accounts or to the specific admin who created the match? | Tech Lead | TBD |
| OQ-M-04 | After admin override confirmation (AC-MATCH-12), should the tutee still receive a notification? The product requirements document is silent on this. | Product | TBD |
| OQ-M-05 | The schema uses match status ENUM with CLOSED in the domain schema (schema md line 93) but the product requirements document and task spec use COMPLETED. Confirm canonical terminal-success status name before implementation. This spec uses COMPLETED; migration and ENUM must be reconciled. | Tech Lead | Pre-implementation |

## 2026-09-05 current contract and verification

Current verification: an eligible no-listing tutor can apply/be assigned; parent selection still requires admin assignment confirmation; one active arrangement per job is preserved; public data transfer objects expose handles only; first-session confirmation does not itself set COMPLETED; admin PAID after agency receipt atomically completes the match. Contact release is settled by C04: assigned participants and ACTIVE/COMPLETED only. Exceptional paid cancellation/reversal remains in [current open decisions](../../CURRENT-REQUIREMENTS.md).
