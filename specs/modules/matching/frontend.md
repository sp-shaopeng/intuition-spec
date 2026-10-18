# InTuition — Matching Module — Frontend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** First-session confirmation is a separate event; agency receipt of tutor commission permits PAID and match completion. Assigned participants can reveal contacts under [INT-125](../../architecture/int-125-batch-closeout-spec.md): ACTIVE/COMPLETED frontend cards allow it, PENDING/CANCELLED hide it, with application programming interface authorization preserved. Earlier blanket no-contact and decrypted-contact wording is superseded. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Module:** matching  
**Version:** 1.2.0
**Status:** Draft  
**Last Updated:** 2026-09-05
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Routes Overview

| Route | Auth | Description |
|---|---|---|
| /admin/matching | ADMIN | Matching dashboard (two-panel search) |
| /admin/matching/create | ADMIN | Confirm and create a match |
| /admin/matches | ADMIN | All matches list with filters |
| /admin/matches/ id | ADMIN | Match detail + status management |
| /dashboard/my-matches | TUTOR/TUTEE | User's own match history |
| /confirm-session/ token | Public (token) | Tutee first-session confirmation |

---

## 2. Matching Dashboard (/admin/matching)

### 2.1 Page Layout (Desktop ≥1024px)


The illustration for page layout desktop ≥1024px contains the following information:

- Matching Dashboard.
- Find compatible tutor and tutee posts to create a match.
- TUTOR POSTS TUTEE REQUESTS.
- [Subject ▼] [Level ▼] [Location ▼] [Subject ▼] [Level ▼] [Location ▼].
- ○ John T. Math•Phys S3–4 D15 ○ P6 Math Request P6 D19.
- S$35–50 • 1-on-1 • Online Home S$25–40 • Online • 1 student.
- ○ Sarah L. Chinese P1–P6 D12 ○ Sec 4 Physics Req S4 D15.
- S$30–45 • 1-on-1 • Student Home S$40–60 • Any venue • 1 student.
- ○ [grey] Already matched ○ English P5 Req P5 D08.
- David C. English P4–P6 S$20–35 • Home • 2 students.
- [Load More] [Load More].
- Selected: John T. (Math•Phys S3–4) Selected: Sec 4 Physics Req.
- [Preview Match →].


**Mobile (<1024px):** Vertical stack — Tutor search above, Tutee search below, Preview button fixed at bottom.

### 2.2 Post Cards (Tutor Side)


The illustration for post cards tutor side contains the following information:

- ○ John Tan Wei Ming.
- Math (O-Level), Physics (O-Level).
- Sec 3 (Exp), Sec 4 (Exp).
- Student's Home, Online • East Coast.
- ★ 5 yrs exp.


**Radio button selection** — only one tutor post can be selected at a time.

**Grayed out cards** — tutee posts that already have a PENDING or ACTIVE match. Show tooltip: "Already has an active match."

### 2.3 Post Cards (Tutee Side)


The illustration for post cards tutee side contains the following information:

- ○ Physics Request.
- Physics (O-Level) • Sec 4 (Express).
- Budget: S$40–60/hr.
- Student's Home • East Coast (D15).
- 1 student • Posted 3 days ago.


No tutee name/contact shown on cards.

### 2.4 Compatibility Indicator

When both a tutor and tutee post are selected, show a compatibility summary bar above the "Preview Match" button:


The illustration for compatibility indicator contains the following information:

- Match Compatibility Check.
- ✓ Subject: Physics (O-Level) matches.
- ✓ Level: Sec 4 (Express) matches.
- ✓ Venue: Student's Home — both compatible.
- ✓ Location: Both in East Coast area (D15).
- ◐ Availability: 3 overlapping slots (Mon/Wed/Fri evenings).
- ● Rate: Tutor S$35–50 Tutee budget S$40–60 — compatible.


---

## 3. Match Creation (/admin/matching/create)

After clicking "Preview Match", navigate to a confirmation page.

### 3.1 Match Preview Page

Shows side-by-side full details of both selected posts, including:
- Tutor contact info (current user-contact fields — authorized admin view)
- Tutee contact info (current user-contact fields — authorized admin view)

**Form fields:**
- **Agreed Rate (SGD/hr)** — number input, required. Pre-filled with midpoint of overlapping rate ranges. Admin must confirm actual agreed rate.
- **Internal Notes** — textarea, optional. Admin's notes about this match.


The illustration for match preview page contains the following information:

- Create Match.
- TUTOR TUTEE REQUEST.
- John Tan Wei Ming Physics • Sec 4 (Express).
- 📧 [REDACTED EMAIL] 👤 Parent (East Coast).
- 📱 [REDACTED PHONE] 📧 [REDACTED EMAIL].
- Math, Physics • S$35–50/hr 📱 [REDACTED PHONE].
- Student Home, Online Budget: S$40–60/hr.
- Student's Home preferred.
- Agreed Rate (SGD/hr): [S$45.00].
- Internal Notes: [____________].
- [← Back to Search] [Create Match].


**Create Match button** → Confirmation modal:


Implementation example omitted. This public edition retains the written requirements in this section.


On success: redirect to /admin/matches/ new Match Id with success toast.

---

## 4. All Matches List (/admin/matches)

### 4.1 Filters

Status (All/PENDING/ACTIVE/COMPLETED/CANCELLED), date range, subject, level.

### 4.2 Table


The illustration for table contains the following information:

- ID Tutor Tutee Request Rate Status Matched Action.
- #1023 John T. Phys • S4 Exp S$45/hr PENDING 2 days ago [View].
- #1022 Sarah L. Chinese • P6 S$35/hr ACTIVE 5 days ago [View].
- #1021 David C. English • P5 S$30/hr COMPLETED 30 days ago [View].


Status badges: PENDING is amber, ACTIVE is green, COMPLETED is navy, CANCELLED is gray.

---

## 5. Match Detail Page (/admin/matches/ id)

### 5.1 Layout

Top section: Match summary + status badge + action buttons.  
Middle: Two columns — Tutor details | Tutee details (with contact info).  
Bottom: Status history timeline + notes.

### 5.2 Contact Reveal Panel


The illustration for contact reveal panel contains the following information:

- 🔒 Contact Information (Admin Only).
- Tutor: John Tan Wei Ming.
- 📧 [REDACTED EMAIL] 📱 [REDACTED PHONE].
- Tutee: Parent (East Coast).
- ⚠ Contact access logged for audit.


### 5.3 Status Action Buttons

Shown based on current status:

**PENDING:**
- Activate Match → PENDING → ACTIVE (triggers N-12 tutee confirmation)
- Cancel Match → opens cancel modal

**ACTIVE:**
- Mark Complete → ACTIVE → COMPLETED (only if tutee has confirmed session)
- Cancel Match

**COMPLETED:** Read-only. Link to commission record.

**CANCELLED:** Read-only.

**Cancel Modal:**


The illustration for status action buttons contains the following information:

- Cancel Match?.
- Reason: [________________________] (required).
- [Back] [Confirm Cancel].


### 5.4 Status Timeline


The illustration for status timeline contains the following information:

- ● Match Created 27 May 10:30 by Admin A.
- ● Match Activated 28 May 14:00 by Admin A.
- ○ Awaiting Tutee Confirmation (sent 28 May 14:00, expires 4 Jun).


---

## 6. My Matches (/dashboard/my-matches)

Tutee or Tutor view. Contact remains gated to assigned participants; ACTIVE/COMPLETED match cards offer the reveal control, while PENDING/CANCELLED do not.

**Tutee view:**


The illustration for . my matches /dashboard/my-matches contains the following information:

- Match #1022 — Physics Tutor.
- Status: ACTIVE.
- Matched: 28 May 2025.
- "We've matched you with a tutor for your Physics (Sec 4) request.
- Our coordinator will be in touch shortly.".
- [Confirm First Session] ← shown only when status = ACTIVE confirmation pending.


**Tutor view:**


The illustration for . my matches /dashboard/my-matches contains the following information:

- Match #1022 — Physics Tutee Request.
- Status: ACTIVE.
- Matched: 28 May 2025.
- "We've matched you with a student for your Physics post.
- Our coordinator will contact you within 1 business day.".


---

## 7. First Session Confirmation (/confirm-session/ token)

Public page, accessed via email link. Token is a signed signed session token (7-day expiry period).

**States:**

**Valid, unconfirmed:**


The illustration for . first session confirmation /confirm-session/ token contains the following information:

- Singapore Tuition Agency.
- ✓ Confirm Your First Session.
- Hi [Parent Name],.
- Has your first tuition session taken place?.
- By confirming, you let us know the match is working.
- well. We will then process the tutor's commission.
- [Yes, First Session Happened].
- If you have any concerns, please contact us at.
- [REDACTED EMAIL].


**Already confirmed:**


The illustration for . first session confirmation /confirm-session/ token contains the following information:

- ✓ You've already confirmed your first session. Thank you!.


**Token expired:**


The illustration for . first session confirmation /confirm-session/ token contains the following information:

- This link has expired. Please contact [REDACTED EMAIL].


Confirmation button calls submit operation for matches confirm-session :token — idempotent.
