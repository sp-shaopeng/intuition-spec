# InTuition — Applications Module (M15) — Frontend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** [012](../../012-tutor-apply-and-profile-ux/requirements.md)/[024](../../024-postfix-review-fixes/spec.md) own completion feedback on detail and list cards. M18 removes the listing prerequisite and adds the parent applicant view; INT-093 owns current withdraw/reapply states. Old composer/count/notification limitations describe the initial slice. This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Module:** applications
**Version:** 1.2.0
**Status:** Implemented; team-reviewed (2026-05-29)
**Last Updated:** 2026-09-05
**Platform:** InTuition
**Tech Stack:** React 19 + TypeScript (Vite)

> Authoritative spec: [spec md](spec.md). Backend contract:
> [backend md](backend.md). All application programming interface paths go through the application programming interface ROUTES constants
> map (no raw path literals — ADR-006).

The thin slice ships two tutor-facing capabilities: an **Apply / Withdraw** action
on the browse-requests surface, and a **My Applications** page. No applicant count
is rendered anywhere yet (that arrives with M14).

> **Shipped user interface is a lean subset of the design sketched below.** As built: Apply is a
> **single-click primary button** (no composer/modal, no message field, no listing
> picker — tutor Post Id/message are omitted); Withdraw uses a **window confirm**
> dialog; "My Applications" is a **flat paginated list** (no status tabs). The
> aspirational composer/tabs/tooltip in §2.2/§3.2 below are **not** in this slice.

---

## 1. api Routes keys

New entries added to the frontend api Routes constants map:

| Key | Path | Method | Endpoint (backend §) |
|---|---|---|---|
| application programming interface ROUTES applications.create | tutor applications interface operation | POST | Apply (§1.1) |
| application programming interface ROUTES applications.my List | tutor applications interface operation | GET | List own (§1.3) |
| the withdraw operation | tutor applications id interface operation | DELETE | Withdraw (§1.2) |

The admin applicant endpoint (admin posts id applications interface operation, §1.4) has
**no frontend consumer in this slice**, so it is not added to application programming interface ROUTES yet.

The status enum is consumed through a typed frontend union
type Application Status APPLIED | WITHDRAWN | SHORTLISTED | REJECTED | MATCHED
(matches the named PG enum), but only APPLIED / WITHDRAWN are produced by the
thin-slice endpoints.

---

## 2. Apply / Withdraw on Browse Requests

**Surface:** the tutor browse-requests page (/requests, Tutor Browse Requests)
and each tutee-request detail card. A tutor browsing active jobs gets an
**Apply** action per job.

### 2.1 Button states

| State | Display | Condition |
|---|---|---|
| Apply (active) | Primary button: "Apply" | Tutor logged in, job ACTIVE, tutor passes the shared012 completeness gate, no active application for this job |
| Applied | Secondary/"applied" button: "Applied ✓" + adjacent "Withdraw" | Tutor already has an APPLIED application for this job |
| Needs profile completion | Replace Apply with an accessible "Complete profile" banner/link; stale422 shows friendly feedback | Tutor fails the shared012 predicate (TUTOR PROFILE INCOMPLETE), independent of whether a listing exists |
| Job not active | Button not shown | Tutee post not in ACTIVE status |
| Not a tutor | Action not shown | Tutee/admin/anonymous (only TUTOR sees the action) |

The profile-completion explanation must be keyboard- and screen-reader-reachable
via aria-describedby pointing to a visually-hidden span (consistent with the
interview module's disabled-button accessibility pattern).

### 2.2 Apply interaction

On clicking **Apply**, an optional inline composer / modal opens:

- Optional message field — label "Add a note for the tutee (optional)",
  max 500 chars with a character counter.
- Optional listing picker — "Apply with listing" dropdown of the tutor's ACTIVE
  posts; defaults to none (sends tutor Post Id: no value).
- Submit: "Send Application".

**On submit:**

- POST api Routes tutor Applications with the tuition-request identifier, optional tutor-listing identifier and optional message.
- Disable submit immediately to prevent double-submit; show in-button spinner.
- **Idempotent success:** the endpoint always returns **200** (created, reactivated,
  or already-applied) and the card flips to the **Applied** state — the user interface treats
  all cases identically, so a double-click never produces an error toast. Toast:
  "Application sent. Our coordinator will be in touch."
- Error mapping (shipped):
  - 403 TUTOR NO ACTIVE POST → "Set up an active tutor listing before applying to
    jobs." (the shipped button is shown to all tutors; this actionable toast handles
    the no-listing case rather than a pre-disabled tooltip).
  - All other errors → generic "Could not send your application. Please try again."
  - Re-enable submit on error.

### 2.3 Withdraw interaction

From the **Applied** state (on the browse card or My Applications page):

- "Withdraw" → confirmation dialog (role is alertdialog, aria-modal is yes,
  focus trap), copy: "Withdraw your application for this job? You can apply again
  later."
- Confirm → the tutor application operation.
- Success: card flips back to the **Apply** state (re-apply is allowed —
  backend §3). Toast: "Application withdrawn."

---

## 3. My Applications Page

**Route:** /tutor/applications (page Tutor Applications) — **TUTOR role-gated**.

### 3.1 Routing and role-gating

- Added to the router behind the shared Protected Route with role is TUTOR
  (same guard used for /tutor/dashboard, /tutor/interviews, etc.). A TUTEE or
  ADMIN hitting /tutor/applications is redirected per the existing
  unauthorized-route behaviour; anonymous → login with return To.
- New nav entry **"My Applications"** in the tutor navigation, placed alongside
  "My Interviews" / "My Matches".
- Listed in [frontend-routes implementation reference](../../reference/frontend-routes.md)
  under the **Tutor (TUTOR role required)** section.

### 3.2 Layout

- Standard tutor dashboard layout; breadcrumb: Dashboard → My Applications.
- Header: "My Applications".
- Status tabs (optional, mirroring the interview page):
  - **All** (default)
  - **Active** (APPLIED)
  - **Withdrawn** (WITHDRAWN)
- Data via GET api Routes tutor Applications (paginated Page Response,
  newest first).

### 3.3 Application card

Each row shows:

- Job summary: tutee-request headline, subject, level, area (no tutee personal information).
- Status badge (see §4).
- Applied date (created At, Singapore time, human-readable).
- The message the tutor sent (if any), and the listing pinned (if any).
- Actions:
  - APPLIED → "Withdraw" (opens the §2.3 confirm dialog).
  - WITHDRAWN → "Apply again" (re-runs the §2.2 apply flow for the same job).

### 3.4 Empty state

- Icon: paper-plane / application icon.
- Heading: "No applications yet".
- Body: "Browse open tutee requests and apply to start matching."
- CTA: "Browse Requests" → /requests.

---

## 4. Status Badge States

A reusable Application Status Badge component. Only the first two render in the
thin slice; the remaining three are defined for forward-compatibility (the
endpoints never emit them yet):

| Status | Badge | Colour |
|---|---|---|
| APPLIED | "Applied" | Blue |
| WITHDRAWN | "Withdrawn" | Gray |
| SHORTLISTED | "Shortlisted" | Amber (reserved — not emitted yet) |
| REJECTED | "Not selected" | Red (reserved — not emitted yet) |
| MATCHED | "Matched" | Green (reserved — not emitted yet) |

Badge colour tokens follow the platform palette already used by
Interview Status Badge (blue = active/scheduled, gray = closed, green = terminal
success, red = terminal negative, amber = pending action).

---

## 5. Mobile Considerations

Consistent with the platform mobile-first requirement (usable at 390px):

- "Apply" button: full width on mobile cards.
- Apply composer: full-screen drawer on mobile (not a floating dialog).
- My Applications list: card layout (not table) below 640px.
- Tap targets ≥ 44px (Apply / Withdraw / Apply again).

---

## 6. Out of Scope (this slice)

- Applicant **count** on any job card — added by M14 once this module ships.
- Admin applicant-pool user interface for read operation for admin posts id applications —
  consumed by M4 matching screens in a later slice; the endpoint exists but no
  dedicated admin page is built here.
- Shortlist / reject / matched status user interface (those transitions are deferred).
