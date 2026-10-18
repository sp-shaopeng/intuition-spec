# InTuition — Posting Module — Frontend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Schedule precedes rate/venue, including the actual wizard headings.002/009 own current slot/taxonomy fields,025 owns no-orphan validation, and anonymous filtering is permitted within limited results. Old route/field examples are historical where amended.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Module:** posting  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Routes Overview

| Route | Auth | Role | Description |
|---|---|---|---|
| /posts/create/tutor | Required | TUTOR | Tutor post creation wizard |
| /posts/create/tutee | Required | TUTEE | Tutee request creation form |
| /posts/browse/tutors | Optional | Public | Browse approved tutor posts |
| /posts/browse/requests | Optional | Public | Limited public request browsing with filters; authenticated expansion follows current quota contract |
| /posts/tutor/ id | Required | Authenticated viewer | Tutor detail; public browse cards remain limited/personal information-free |
| /posts/tutee/ id | Required | TUTOR/ADMIN | Tutee request detail |
| /dashboard/my-posts | Required | TUTOR/TUTEE | User's own posts |
| /dashboard/posts/tutor/ id /edit | Required | TUTOR (owner) | Edit tutor post |
| /dashboard/posts/tutee/ id /edit | Required | TUTEE (owner) | Edit tutee post |

---

## 2. Tutor Post Creation Wizard (/posts/create/tutor)

Multi-step form wizard. Progress saved to session Storage on each step. Submitting sends all data in one application programming interface call.

### Step 1 — Subjects & Levels

**Fields:**
- **Subjects** (multi-select, required, 1–10 selections): Searchable dropdown populated from taxonomy subjects interface operation. Groups: PSLE, O-Level, A-Level, IB, Poly, University, Others.
- **Levels** (multi-select, required, 1–8 selections): Populated from taxonomy levels interface operation, filtered to relevant levels for selected subjects.

**Validation:**
- At least 1 subject and 1 level required
- Level-subject compatibility soft warning (e g., if A-Level Math selected but level P1 also selected, show warning)

---

### Step 2 — Availability

**Availability Grid Component:**

Desktop (≥768px): 7-column × 3-row grid.


The illustration for step 2 — availability contains the following information:

- Mon Tue Wed Thu Fri Sat Sun.
- Morning [ ] [ ] [ ] [ ] [ ] [✓] [✓].
- Afternoon[ ] [ ] [ ] [ ] [✓] [✓] [✓].
- Evening [✓] [✓] [✓] [✓] [✓] [ ] [ ].


- Each cell is a 40px × 40px checkbox button
- Selected cells: navy fill (#0F2138), white text
- Unselected: white fill, gray border
- Hover: cream fill (#FBF6EE)

Mobile (<768px): Accordion per day. Each day row expands to show Morning/Afternoon/Evening toggles (44px touch targets). Sat/Sun expanded by default.

**Bit mapping (21-bit integer):**


The illustration for step 2 — availability contains the following information:

- Bit 0 = Mon Morning.
- Bit 1 = Mon Afternoon.
- Bit 2 = Mon Evening.
- Bit 3 = Tue Morning.
- ... (3 bits per day × 7 days = 21 bits).
- Bit 20 = Sun Evening.


At least 1 slot required.

---

### Step 3 — Rate & Experience

**Fields:**
- **Hourly Rate Range** (required): Two SGD currency inputs — Min rate and Max rate. Display as S XX – S XX/hr.
  - Min: 1–500 SGD
  - Max: must be ≥ min, max 1000 SGD
  - Placeholder guidance shown below: "Pri tutors typically S$20–40/hr • Sec: S$30–60/hr • JC: S$50–100/hr"
- **Years of Experience** (required): Number input, 0–50. 0 renders as "Less than 1 year".
- **Group Size** (multi-select, required): Options: 1-on-1, Small group 2–5.
- **Qualifications** (required, textarea): Max 500 chars. Character counter shown.
- **Teaching Approach** (optional, textarea): Max 500 chars.

**Validation:** All required fields checked before Step 4.

---

### Step 4 — Venue & Location

**Fields:**
- **Venue Types** (multi-select, required, 1+):
  - Student's Home
  - Tutor's Home
  - Public Place (library/café)
  - Online (Zoom / Google Meet)
  - Tuition Centre
- **Location** (optional): District picker (D01–D28) or MRT line + station. Toggle between district and MRT mode.

**MRT Station Picker:** Searchable dropdown of all Singapore MRT stations, grouped by line (NSL, EWL, CCL, DTL, TEL, NEL).

---

### Step 5 — Review & Submit

Shows read-only summary of all entered data. User can click "Edit" on any section to jump back.

**personal information Content Warning Banner:**


The illustration for step 5 — review & submit contains the following information:

- ⚠️ Do not include personal contact details (phone number, email address, or full name).
- in your post. Admin-only fields are used for matching — your contact info is protected.


Shown prominently before submit button.

**Submit button:** "Submit for Review" — disabled until all required fields valid.

On success: redirect to /dashboard/my-posts with success toast: "Your post has been submitted for review. We'll notify you within 24 hours."

---

## 3. Tutee Request Creation Form (/posts/create/tutee)

Single-page form (not wizard — fewer fields than tutor post).

**Fields:**
- **Subject** (single-select, required): From taxonomy
- **Level** (single-select, required): From taxonomy, filtered by subject
- **Number of Students** (required): Stepper input, 1–20. Default 1.
- **Student Relationship** (required): Toggle button group — "Parent's child", "I am the student", "Other"
- **Budget Range** (required): Min/Max SGD per hour. SGD currency input component. Min > 0.
- **Session Duration** (required): Radio group — 1 hr, 1.5 hrs, 2 hrs, 2.5 hrs
- **Frequency per Week** (required): Radio group — Once, Twice, Three times, Flexible
- **Preferred Start Date** (optional): Date picker. Must be today or future.
- **Venue Preference** (multi-select, required): Same 5 options as tutor post
- **Location** (required): MRT station picker (or planning area). See §9 for component.
- **Availability** (required): Same 21-bit availability grid component (see §4)
- **Special Requirements** (optional, textarea): Max 500 chars. personal information warning shown below field.

**Submit:** "Submit Request for Review"

**Validation error messages** (from posting/spec md §12.2):
- Subject required: "Select a subject"
- Level required: "Select a level"
- Budget max < min: "Maximum budget must be at least the minimum budget"
- Session duration not in allowed set: "Select a valid session duration (1, 1.5, 2, or 2.5 hours)"
- Availability bitmask = 0: "Select at least one available time slot"
- Preferred start date in past: "Start date must be today or a future date"

---

## 4. Browse Tutor Posts (/posts/browse/tutors)

Public page (no auth required to view; auth required to create request or save).

### Filter Panel (left sidebar on desktop, drawer on mobile)

- **Subject**: Multi-select from taxonomy
- **Level**: Multi-select from taxonomy
- **Rate**: Range slider S$0 – S$200/hr
- **Venue Type**: Checkbox list
- **Location**: District or MRT picker
- **Availability**: Compact availability grid (filter by available slots)
- **Group Size**: Checkbox list

Filters are applied client-side for first 50 results; deep filters call application programming interface with query params.

### Post Cards


The illustration for post cards contains the following information:

- [Avatar] John T. [ACTIVE].
- English, Math • Sec 1–4.
- Venue: Student's Home, Online.
- Bedok / Tampines.
- ⭐ 5 yrs exp • B Ed (NTU).
- [View Profile →].


- No phone/email shown
- Display name shown (not full legal name)
- Pagination: 20 per page, cursor-based, "Load more" button

---

## 5. Browse Tutee Requests (/posts/browse/requests)

Requires TUTOR auth. Shows anonymized tutee posts (no contact info, no display name).

### Filter Panel

- Subject, Level, Venue preference, Location, Budget, Availability

### Request Cards


The illustration for request cards contains the following information:

- Tutee Request [APPROVED].
- Subject: Physics • Level: Sec 4 Exp.
- Budget: S$40–60/hr.
- Venue: Student's Home.
- Location: Bishan area.
- Students: 1 • Posted 2 days ago.
- [View Details →].


No tutee name, no contact info. "Posted by" is omitted entirely.

---

## 6. Post Detail Pages

### Tutor Post Detail (/posts/tutor/ id)

Public. Shows full post details without contact info:
- Subjects, levels, rate range, venue types, group sizes, availability grid (read-only), qualifications, teaching approach, years of experience, location region
- No phone/email/last name

Admin view adds a panel showing decrypted contact info (from admin users :id contact interface operation).

### Tutee Post Detail (/posts/tutee/ id)

Visible only to TUTOR (logged in) and ADMIN.
- Subject, level, budget, venue preferences, availability, student count, special notes
- No contact info for tutors; admin sees contact info in separate panel

---

## 7. My Posts Dashboard (/dashboard/my-posts)

### Post List Table

Columns: Title/Subject, Type (Offering/Request), Status badge, Created, Last updated, Actions.

**Status Badges:**
- DRAFT → gray chip
- PENDING → amber chip "Under Review"
- APPROVED → green chip "Live"
- REJECTED → red chip + reason tooltip
- PAUSED FOR REVIEW → amber chip "Paused — Re-review Pending"
- EXPIRED → gray chip "Expired"

**Actions per status:**
- DRAFT: Edit, Submit, Delete
- PENDING: View (read-only)
- APPROVED: View, Edit (with warning modal), Delete
- REJECTED: View rejection reason, Edit & Resubmit, Delete
- PAUSED FOR REVIEW: View (read-only, pending re-review)
- EXPIRED: View, Resubmit

---

## 8. Edit Post (/dashboard/posts/ type / id /edit)

Pre-fills all form fields with current post data. Same form components as creation.

**Edit-while-active modal:** If post status is APPROVED, show confirmation modal before entering edit:


The illustration for . edit post /dashboard/posts/ type / id /edit contains the following information:

- ⚠️ Edit Live Post?.
- This post is currently live and visible to.
- potential matches. Editing will pause it for.
- re-review. It won't be visible until admin.
- approves the changes (usually within 24 hours).
- [Cancel] [Edit Anyway].


On save: POST status → PAUSED FOR REVIEW. Toast: "Your post has been paused for re-review."

---

## 9. Shared Components

### SGD Currency Input
- Text input with S prefix
- Accepts integers and decimals (max 2 decimal places)
- Rejects non-numeric input
- Format on blur: e g. 35 → S 35.00

### Subject + Level Multi-Select
- Combobox with search
- Grouped by curriculum level (PSLE, O-Level, A-Level, IB, Poly, Uni)
- Max selections enforced with visual counter

### District/MRT Picker
- Toggle: "By District" | "By MRT"
- District: D01–D28 select with area names (e g., D01 – Boat Quay, D15 – East Coast)
- MRT: Line selector → Station selector (searchable)

### Availability Grid
See Step 4 above. Used in both creation forms and browse filters.

---

## 10. Selection-First user interface Principle

**Rule:** Every form field with a bounded set of valid values **MUST** use a selection-based control. Free-text input is reserved exclusively for genuinely open-ended content.

### Required selection controls by field type

| Field | Control | Notes |
|---|---|---|
| Subject | <select> grouped by curriculum (PSLE / O-Level / A-Level / IB / Poly / Uni) | Populated from SG SUBJECTS taxonomy |
| Level | <select> filtered by subject group | Populated from SG LEVELS taxonomy |
| Number of students | <select> (1–5) | "1 student", "2 students", … |
| Student relationship | Toggle button group | "Parent's child" / "I am the student" / "Other" |
| Venue types | Multi-select toggle chips (desktop) / checkbox list (mobile) | "select all that apply"; min 1 required |
| Session duration | Toggle button group | 1 hr / 1.5 hrs / 2 hrs / 2.5 hrs |
| Frequency per week | Toggle button group | Once/week / Twice/week / 3×/week / Flexible |
| Group size (tutor post) | Toggle button group | 1-on-1 / Small group (2–5) |
| Availability | 7 × 3 day/time checkbox-button grid | Mon–Sun × Morning/Afternoon/Evening; quick-select presets |
| Rejection reason (admin) | Preset reason select + optional notes textarea | Min 1 preset required; notes always optional |

### Permitted free-text fields

| Field | Max chars | personal information warning |
|---|---|---|
| Special requirements / notes (tutee) | 500 | Yes — warn not to include phone/email/address |
| Qualifications (tutor post) | 500 | No |
| Teaching approach (tutor post) | 500 | No |
| Additional rejection notes (admin) | 500 | No |

### Rationale

Structured selection:
- Prevents data quality issues (inconsistent spellings, invalid values, personal information entered in wrong field)
- Enables reliable server-side filtering and analytics aggregation
- Reduces cognitive load and time-to-complete for all personas (see product requirements document §2)
- Makes the availability bitmask unambiguous (vs free-text schedule descriptions)

### Availability grid — step placement

The schedule step (availability grid + session duration + frequency) **must come before** the budget/venue step. Users should anchor their availability commitment before deciding on financial commitment. See wireframe Screen 6, Step 2.

## 2026-09-05 current contract and verification

Tutor fine availability follows002; tutee availability follows009. Full-card/detail and action gates use the current baseline; anonymous users can filter the limited browse pool without login. The stronger profile Cartesian warning must not replace025 no-orphan validation. Preserve non-secret draft persistence and accessible controls; do not persist credentials.
