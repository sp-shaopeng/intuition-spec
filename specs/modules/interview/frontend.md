# InTuition — Interview Module — Frontend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** INT039 replaces REQUESTED-only user interface with proposed slots, counter/accept/decline and participant cancellation;021 allows non-deleted PENDING parent jobs. Current detail authentication and disclosure follow the baseline, not the old public-route label.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Module:** interview  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. New user interface Elements on Existing Pages

### 1.1 Tutor Post Detail Page — "Request Interview" Button

**Route:** the specified tutor accessible to the user (public tutor post detail page — route defined in posting/frontend md)

When a TUTEE user is logged in, a **"Request Interview"** button is added to the tutor post detail page, positioned below the post content and above the "Post a Request" CTA.

**Button states:**

| State | Display | Condition |
|---|---|---|
| Active | Primary button: "Request Interview" | Tutee logged in, tutor post ACTIVE, no cooldown, < 3 open requests for this posting |
| Cooldown | Disabled button: "Interview cooldown active — X days remaining" + tooltip (see below) | 30-day cooldown in effect for this tutee/post pair |
| Limit reached | Disabled button: "Max 3 open interviews for this posting reached" + link to /tutee/interviews | This posting has 3 open requests |
| Not logged in | Text link: "Log in to request an interview" | Unauthenticated visitor |
| Post not active | Button not shown | Tutor post not in ACTIVE status |

**Cooldown tooltip:** Copy must cover both COMPLETED and NO SHOW cases. Use: "You recently had an interview session for this post. You can request again in X days." (Avoid "interviewed" as a verb — it implies the session was successful, which is inaccurate for NO SHOW.)

**Tooltip accessibility:** The disabled button must include aria-describedby pointing to a visually-hidden  containing the full cooldown message. Tooltip must not rely on hover-only — it must be reachable by keyboard focus and screen reader.

**Interview Request Modal:**

On click, a modal opens:
- Title: "Request an Interview"
- Subtext: "Admin will coordinate a 15-minute video call between you and this tutor. Your contact details remain private during interview arrangements."
- Tutor post summary card (headline, subjects, levels, rate range)
- Optional note field:
  - Label: "Add a note for admin (optional)"
  - Placeholder: "e g., Interested in weekly 2-hour sessions for my P6 daughter..."
  - Max 300 characters; character counter shown
- Propose 1–3 slots under INT039; submit button: "Send interview proposal"
- Cancel button

**On submit:**
- POST interviews interface operation
- **Optimistic user interface:** Disable submit button immediately on click to prevent double-submission. Show spinner inside button. Do not close modal until success response received.
- Success: modal closes, toast notification: "Interview proposal sent. You’ll be notified when the tutor responds."
- Error: inline error message per guard (cooldown, limit, post not active); re-enable submit button.

---

## 2. Tutee Pages

### 2.1 My Interviews (/tutee/interviews)

**Layout:** Standard tutee dashboard layout; breadcrumb: Dashboard → My Interviews

**Header:** "My Interview Requests"

**Status tabs:**
- All (default)
- Open (REQUESTED + SCHEDULED)
- Past (COMPLETED + CANCELLED + NO SHOW)

**Interview card:** Each row shows:
- Tutor post headline
- Status badge (color-coded):
  - REQUESTED — Amber: "Awaiting Scheduling"
  - SCHEDULED — Blue: "Scheduled"
  - COMPLETED — Green: "Completed"
  - CANCELLED — Gray: "Cancelled"
  - NO SHOW — Red: "No Show"
- For SCHEDULED: date/time (Singapore time, human-readable) + "Join Meeting" button (links to meeting URL in new tab)
- Requested date
- "View Details" link

**Empty state (no interviews):**
- Icon: calendar/video icon
- Heading: "No interview requests yet"
- Body: "Browse tutor profiles and request an interview to speak with a tutor before committing to a match."
- CTA: "Browse Tutors" → /tutors

---

### 2.2 Interview Detail — Tutee View (the specified interview accessible to the parent)

**Layout:** Standard tutee layout; breadcrumb: Dashboard → My Interviews → Interview Detail

**Content:**

1. **Status banner** — Full-width colored banner with status badge and status description
   - REQUESTED: "Your request has been received. Admin will schedule your interview within 1 business day."
   - SCHEDULED: Green: "Your interview is scheduled. Join the meeting at the time below."
   - COMPLETED: "This interview has been completed."
   - CANCELLED: "This interview was cancelled."
   - NO SHOW: "This interview was marked as no-show."

2. **Tutor post summary card:**
   - Headline, subjects, levels, rate range, location area
   - Link to tutor post: "View tutor profile →"
   - No tutor contact info shown

3. **Interview details (for SCHEDULED):**
   - Date: [Weekday, DD Month YYYY]
   - Time: [HH:MM Singapore time]
   - Duration: 15 minutes
   - "Join Meeting" button (primary, links to meeting URL — opens new tab)
     - **Time-gating:** Button is disabled with countdown ("Join available in 2 hr 30 min") until 30 minutes before scheduled At. Active (clickable) from 30 min before until 60 min after scheduled At.
   - Meeting platform derived from URL (e g., "via Zoom" if zoom us)
   - **P1:** "Add to calendar" link (Google Calendar URL + ICS download)

4. **Your note** (if provided at request time): italic text block

5. **Cancel request button** (only for REQUESTED status):
   - Destructive button: "Cancel Request"
   - Confirmation dialog: "Are you sure you want to cancel this interview request?"
     - Must use role is alertdialog with aria-modal is yes and aria-labelledby pointing to the dialog heading
     - Must implement focus trap: keyboard focus stays within the dialog until dismissed
     - Focus moves to the "Confirm Cancel" button on open; returns to the "Cancel Request" trigger on dismiss

---

## 3. Tutor Pages

### 3.1 My Interviews (/tutor/interviews)

**Layout:** Standard tutor dashboard layout

**Header:** "Scheduled Interviews"

Only shows interviews in SCHEDULED status (upcoming) and recently completed (last 7 days). Tutors do not have visibility into REQUESTED state (only admin sees the queue).

**Interview card:**
- Tutee request summary: subject, level, area, budget — NO tutee name or contact
- Date and time (Singapore time)
- "Join Meeting" button (links to meeting URL — new tab); applies same 30-min-before time-gating as tutee view
- Status badge

**Empty state:**
- "No interviews scheduled."
- Body: "When admin schedules an interview between you and a tutee, it will appear here."

---

## 4. Admin Pages

### 4.1 Interview Queue (/admin/interviews)

**Layout:** Admin panel layout; sidebar nav item: "Interviews" with badge showing count of REQUESTED

**Tabs:**
- **Action Required** (REQUESTED) — sorted by requested at ASC (oldest first)
- **Scheduled** (SCHEDULED) — sorted by scheduled at ASC (soonest first)
- **History** (COMPLETED / CANCELLED / NO SHOW) — sorted by updated at DESC

**Row columns (Action Required tab):**
- Tutee display name (pseudonym)
- Tutor post headline
- Subject + Level
- Requested time (relative: "2 hours ago")
- Tutee note (truncated to 60 chars, "..." if longer)
- "Schedule" button (primary, opens scheduling panel)

**Row columns (Scheduled tab):**
- Tutee display name
- Tutor post headline
- Date/Time (Singapore time)
- Meeting platform (hostname)
- Action buttons: "Mark Completed" / "Mark No-Show" / "Cancel"

---

### 4.2 Interview Detail — Admin View (the specified interview accessible to an administrator)

**Layout:** Admin panel; breadcrumb: Interviews → Interview Detail

**Sections:**

1. **Status header** — Status badge + timestamp of last action

2. **Tutee information panel** (contact info — access logged):
   - Display name, email (decrypted), phone (decrypted)
   - Account link: "View Account →"
   - Tutee's note (if provided)

3. **Tutor post summary:**
   - Headline, subjects, levels, rate range, area
   - Link to tutor post admin view

4. **Schedule panel** (shown for REQUESTED and SCHEDULED):

   For REQUESTED (scheduling form):


Implementation example omitted. This public edition retains the written requirements in this section.


   For SCHEDULED (update form):


Implementation example omitted. This public edition retains the written requirements in this section.


5. **Admin notes log:** Chronological list of notes (editable by admin)

6. **Audit trail:** Read-only list of status transitions with admin user and timestamp

---

### 4.3 Scheduling Form Validation

| Field | Rule | Error Message |
|---|---|---|
| Date & Time | Must be in the future (> now operation + 30 min) | "Please select a future date and time (at least 30 minutes from now)" |
| Meeting Link | Required; must be a valid URL (http/https) | "A valid meeting link URL is required" |
| Meeting Link | Max 2000 characters | "Meeting link is too long" |

---

## 5. Components

### 5.1 Interview Status Badge

Reusable status badge component:


The illustration for interview status badge contains the following information:

- REQUESTED → amber background, "Awaiting Scheduling".
- SCHEDULED → blue background, "Scheduled".
- COMPLETED → green background, "Completed".
- CANCELLED → gray background, "Cancelled".
- NO SHOW → red background, "No Show".


### 5.2 Meeting Link Button

Renders a "Join Meeting" button that opens the meeting URL in a new tab. When the link contains a recognisable hostname, shows a platform icon (Zoom, Google Meet, Microsoft Teams). Falls back to a generic video-camera icon.

**Time-gating:** Accepts a scheduled At prop. Button is disabled with countdown text until 30 minutes before scheduled At. Remains active until 60 minutes after scheduled At, then disabled again (interview window has passed).

**Security:** The URL is passed through the application programming interface as an opaque string. The frontend renders it as an  with target= blank rel is noopener noreferrer. No URL validation or rewriting is performed on the frontend.

### 5.3 Interview Request Modal

Described in §1.1. Shared between the tutor post detail page (the specified tutor accessible to the user) and any inline "Request Interview" CTA.

---

## 6. Empty States and Error States

| Scenario | Message |
|---|---|
| Tutee has no interviews | "No interview requests yet. Browse tutor profiles to get started." |
| Tutor has no scheduled interviews | "No upcoming interviews." |
| Admin queue empty (REQUESTED) | "No pending interview requests." |
| Request failed — cooldown active | "You recently had an interview session with this tutor. You can request again in X days." |
| Request failed — limit reached | "This posting has 3 open interview requests. Complete or cancel one for this posting before requesting again." |
| Scheduling failed — link missing | "A meeting link is required to schedule the interview." |
| Interview in terminal state (edit attempt) | "This interview has already been completed and cannot be modified." |

---

## 7. Keyboard Shortcuts (Admin Interview Queue)

| Shortcut | Action |
|---|---|
| S | Open scheduling panel for highlighted row |
| C | Mark as Completed (for SCHEDULED rows) |
| X | Cancel interview |

Consistent with keyboard shortcuts defined in approval/frontend md.

---

## 8. Mobile Considerations

The tutee interview request flow must be fully usable at 390px (mobile-first per business requirements document §10.7.3):
- "Request Interview" button: full width on mobile
- Interview Request Modal: full-screen drawer on mobile (not a floating dialog)
- Interview list: card layout (not table) at < 640px
- "Join Meeting" button: prominently sized (min 44px tap target) — tutees must be able to tap it quickly on the day of their interview
- Date/time display: always show Singapore time timezone label to avoid confusion

Admin interview pages are desktop-first (exempt from mobile-first requirement per business requirements document §10.7.3).

## 2026-09-05 current contract and verification

Use the INT039 whose Turn-driven actions and supported nonterminal cancel states; preserve the current cooldown/cap/ownership rules. Allow non-deleted PENDING/ACTIVE job invitation/interview initiation without exposing contacts; approval alone never permits disclosure; only authorized assigned participants see contact with frontend ACTIVE/COMPLETED (match-PENDING hidden). Scheduling/short-notice reminders follow006/007. See [current open decisions](../../CURRENT-REQUIREMENTS.md) for invitation acceptance while the parent POST is PENDING (O01); contact disclosure itself is settled by C04. Existing mobile/accessibility/Singapore time requirements remain applicable.
