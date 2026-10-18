# Wireframes

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Retained wireframes are historical layout sketches, not permission or financial-state definitions. Guest filters are visible; details require login; contact requires assigned-participant and match-state authorization; PAID means agency receipt. Use current route/component and numbered requirements for implementation. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.
## Singapore Private Tuition Agency — Text-Based Screen Specifications

> **⚠️ 2026-06-03 — the screens below predate the INT-049 all-pages redesign.** The shipped user interface now composes a
> shared **layout-primitive system** (the private implementation folder: Page Header / Section / Container[app|prose] /
> Card Grid / Form Layout / Data List[cards<md, table≥md] / Tab Nav / Auth Card / Article Layout; ui/: Stat Card,
> Status Timeline) on the locked brand tokens. **Header IA:** logged-out = marketing nav; logged-in **tutee/tutor
> see their dashboard tabs in the top Nav** (role-honest, mobile parity) and land on the dashboard first tab
> after login; **admin** = the purple shell with its header tabs (now token-sourced admin.*). Every data page
> follows the **four-state rule** (skeleton-parity loading / empty+action / error+retry / success). Use the
> primitive contracts + reference/frontend-routes md as the current source; the text screens below are
> historical reference for content, not the current layout.

**Document Version:** 2.1  
**Date:** 2026-05-27  
**Prepared By:** user experience Design  
**Primary Viewport:** 390px (mobile-first); Desktop: 1280px  
**Admin:** Desktop only (768px minimum usable; 1280px primary)

---

## Conventions

-  = interactive button
-  = text input field
- v = dropdown select
- x = checkbox (unchecked) / ✓ = checked
-  = icon
-  = horizontal rule / divider
- | = column separator in two-column layouts
- █ = filled / selected state
- · = empty / unselected state
- Dimensions and notes in  parentheses

---

## Screen 1: Landing Page (/)

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


> **As shipped (Footer implementation reference):** all strings are sourced from
> COMPANY in company implementation reference (brand, legal Name, UEN, registered Address).
> The earlier "Operated in Singapore. Registered under Personal Data Protection Act." copy is **superseded** by
> the legal-name + operated-by (UEN + address) + Personal Data Protection Act-reference lines shown above.

### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Copy requirements:**
1. Hero H1 MUST be "Meet your tutor before you commit." — this is the primary USP statement
2. Hero body MUST include both the interview feature and "never shared with any tutor, ever"
3. Step 2 card for tutees MUST carry the "Only on InTuition" chip — visual differentiator
4. Step 2 for tutees MUST say "within 1 business day" — sets the scheduling service-level agreement expectation
5. The comparison nudge copy ("Others give a profile. We give a conversation.") MUST appear
6. Footer must include Privacy Policy and Terms & Conditions (Personal Data Protection Act requirement)
7. USP Spotlight section is mandatory — do not remove or collapse it below the fold

---

## Screen 2: Register Page

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Interactive element specs:**
- Role selection cards: hover state shows navy 2px border; tap on mobile shows brief press highlight
- Password field: eye icon toggles character visibility
- Personal Data Protection Act consent checkbox: required; Create Account is disabled without it
- Email: validate format on blur; phone: validate Singapore format (+65 8/9XXXXXXX) on blur
- Submit: shows loading spinner while creating account; disabled state: 50% opacity

---

## Screen 3: Commission Disclosure Screen (Tutor Registration — Step 2 of 3)

This is a standalone full-page screen. It does NOT share the main site nav. It looks purposeful and document-like, not like a T&C scroll.

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Design constraints (mandatory):**
- Commission rate "50%" and the worked example SGD amount must be visually largest elements
- Label the page "Commission Terms" — not "Terms & Conditions"
- Button disabled state: visually obvious (opacity 40%, cursor: not-allowed, no hover effect)
- Browser back: native "Leave page?" prompt (beforeunload event)
- No way to reach Step 3 (profile creation) without this step

---

## Screen 4: Tutee Dashboard — My Posts

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Status badge colour system:**
- UNDER REVIEW: amber dot (#D97706)
- ACTIVE (searching): green dot (#16A34A)
- PAUSED FOR RE-REVIEW: amber dot (#D97706) with sub-label "PAUSED FOR RE-REVIEW" — same visual as UNDER REVIEW but distinct sub-label text
- MATCHED: navy dot (#0F2138)
- REVISION REQUIRED: red dot (#DC2626)
- CONFIRM SESSION: orange dot + accent card background (#FFF3EE, border #C84B31)
- CLOSED / EXPIRED: grey dot (#8C8378)
- REJECTED: red dot with strikethrough title text

**Edit Request — Confirmation Modal (shown when tutee clicks "Edit Request" on an Active post):**


Implementation example omitted. This public edition retains the written requirements in this section.


**State transition after edit submission:**
1. Tutee clicks "Edit Request" on Active post → confirmation modal appears
2. Tutee clicks "Yes, Edit" → wizard opens with current values pre-filled
3. Tutee submits edited post → post status immediately changes to "Under Review" (amber PENDING / PAUSED FOR RE-REVIEW badge)
4. Post is removed from the tutor-browseable index until re-approved by admin
5. Admin reviews edited post (target: 24 hours); on approval → post returns to Active (green)

**Important:** "Confirm First Session" card must visually stand out from all others using the accent-tinted background and clay-coloured full-width button. This is the action that unlocks commission invoicing.

---

## Screen 5: Waiting Period State (Approved but Unmatched)

### Desktop Variant (1280px) — shown on My Posts when post is Active and unmatched


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Screen 6: Create Tutee Request — 4-Step Wizard

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)

Each step is full-screen scroll. Progress bar at top (4 segments). Sticky footer with Back / Next buttons.


Implementation example omitted. This public edition retains the written requirements in this section.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Screen 7: Create Tutor Offering — 5-Step Wizard

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)

Same accordion-per-day availability pattern as Screen 6 (tutee wizard Step 2). Steps 1, 3, 4, 5 scroll vertically within each step; sticky footer has Back/Next buttons.

---

## Screen 8: Browse Tutor Listings (/tutors) — Public

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Tutor card elements (NO contact info — enforced):**
- Avatar: initials-only circle (no profile photo on public page; photo is admin-only)
- Headline: max 2 lines, truncated
- Subject chips: max 3 shown + "+N more" label
- Level chips: max 2 shown
- Rate range: SGD X–Y/hr
- Area: planning area name
- Years experience: number
- Availability summary: abbreviated (e g., "Tue/Thu evenings · Weekends")
- No: email, phone, full legal name, home address, profile photo

### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Mobile filter sheet:** Bottom drawer (slides up from bottom); overlays content; same filter controls as desktop sidebar; "Apply Filters" full-width primary button at bottom of drawer; "Clear All" ghost button above it.

---

## Screen 8a: Tutor Post Detail (the specified tutor accessible to the user) — Public

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Request Interview Modal** (appears on sidebar button click):


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Tutor post detail design constraints:**
- No contact info (phone, email, last name) shown anywhere on the page — enforced
- "Request Interview" button placement: always above the secondary "Post a Request" CTA
- On mobile, the Request Interview CTA block appears immediately after the meta row (rate, area, availability summary) — before qualifications — so tutees see it without excessive scrolling
- Cooldown and limit-reached disabled states must be keyboard-accessible with full tooltip text available via aria-describedby
- The modal/drawer process summary (3-item row) reinforces admin coordination and zero-contact-sharing at the point of action — this is the key trust signal

---

## Screen 9: Browse Tutee Requests (/tutor/requests) — Authenticated Tutors Only

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Enforced: no "Apply", "Contact", "Express Interest", or "Request Match" button anywhere on tutee request card or detail page.**

**No-contact copy (shown on every tutee request card):**


The illustration for desktop variant 1280px contains the following information:

- < lock icon > Contact information is managed by our team.
- If this assignment is a good match, our coordinator will reach out to you directly.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Screen 10: First Session Confirmation Page (first-session confirmation for the specified match)

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Screen 11: Admin Review Queue (/admin/posts/review) — Desktop Only


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Screen 12: Admin Post Review Detail (the specified post accessible to an administrator) — Desktop Only


Implementation example omitted. This public edition retains the written requirements in this section.


**Contact info panel design notes:**
- Purple-tinted background (#EDE9FF) distinguishes admin-only fields from post content
- Access-logged notice must appear every time the panel is visible (not just on hover)
- Approve button is disabled if commission acknowledgement is missing on a tutor post

---

## Screen 13: Admin Matching Dashboard (/admin/matching) — Desktop Only

**Breakpoint behaviour:**
- At **1024px and above**: two panels side by side (50/50 split) as shown in the wireframe below.
- At **768px–1023px**: panels stack vertically — tutee posts panel above, tutor posts panel below; shared filter bar collapses to a drawer triggered by a "Filters" button (same bottom-drawer pattern as mobile filter sheet on public pages).
- **Below 768px**: admin panel is not supported — redirect to a desktop prompt page ("Please use a desktop or laptop to access the admin panel.").


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Screen 14: Admin Commission Ledger (/admin/commission) — Desktop Only


Implementation example omitted. This public edition retains the written requirements in this section.


**Commission ledger copy requirements:**
- INVOICED rows: show invoice date and due date in tooltip or inline ("Due: 2026-06-10")
- AWAITING rows overdue (>7 days): amber row background + "OVERDUE" sub-label beneath status badge
- Empty state: "No commission records match your filters. Try adjusting the date range or status filter."
- Tutee reference format: first word of display name + "—" + subject abbreviation + level (e g., "Mei—P6Sci")

---

## Screen 15: My Interviews — Tutee (/tutee/interviews)

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Status badge colour system (interview cards):**
- AWAITING SCHEDULING (REQUESTED): amber (#D97706) — "Awaiting Scheduling"
- SCHEDULED: blue (#1D4ED8) — "Scheduled"
- COMPLETED: green (#16A34A) — "Completed"
- CANCELLED: grey (#8C8378) — "Cancelled"
- NO SHOW: red (#DC2626) — "No Show"

**Join Meeting button time-gating rules:**
- More than 30 min before scheduled At: disabled; shows countdown "Available in X hr Y min"
- 30 min before to 60 min after scheduled At: active; links to meeting URL in new tab
- More than 60 min after scheduled At: disabled; shows "Meeting window has passed"

---

## Screen 16: Interview Detail — Tutee View (the specified interview accessible to the parent)

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Interview detail design constraints:**
- Status banner is full-width above all content — never inside a card — so tutee immediately sees state on arrival
- Tutor post summary card: zero contact info; the lock note reaffirms the privacy model at the exact moment the tutee is looking at the tutor
- Join Meeting button: must be the largest, most prominent interactive element on the page when status is SCHEDULED — this is the primary action
- Time-gating countdown must update in real-time (client-side interval; no polling required)
- Cancel button must use role is alertdialog, aria-modal is yes, aria-labelledby on dialog heading, and focus trap per WCAG 2.1 success criterion 2.1.2
- "Cancel a scheduled interview" copy directs tutee to email — no self-service cancel for SCHEDULED status (per AC-INT-13)
- Your Note block is omitted entirely (not shown as empty) when no note was provided at request time

---

## Screen 17: Tuition Rate Guide (/rates)

**Access:** Public. No login required.  
**Purpose:** Reference page showing typical Singapore private tuition rates. Helps tutees set realistic budgets and tutors benchmark their rates. Linked from Landing, post-creation forms, and Nav.

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Design constraints (mandatory):**
- Rate table must be the largest element on the page — this is the primary information
- All rate figures formatted as S XX–XX/hr with the S$ prefix (never just numbers)
- Disclaimer chip visible in the hero before any rate is shown — sets context immediately
- Mobile: Ex-MOE column may be collapsed but must be accessible (expandable row or tap-to-expand)
- "Post a Request" CTA must show a lock icon and redirect unauthenticated users to /login?return To=/rates
- Do NOT show actual tutor names, contact info, or individual tutor rates on this page — this is market reference data only
- Source attribution footnote required ("Singapore market benchmarks 2026") — not "InTuition rates"

---

## Screen 18: Browse Tuition Requests (/requests) — Tutor Discovery

**Access:** Requires TUTOR login (role guard; redirect to /login?return To=/requests if unauthenticated).  
**Purpose:** Tutors browse anonymised tutee requests to see available assignments. Counterpart to /tutors (tutees browsing tutors). Completes the bidirectional marketplace.

### Desktop Variant (1280px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Request card design rules:**
- NEVER show tutee name, contact info, full address, or any personal information
- Show location as broad area only (e g., "East Singapore", "Bishan area", "Central") — never MRT station or block number
- Budget shown as range (S$XX–XX/hr) — never exact figure if only one value
- "Posted X days ago" relative timestamp — never exact date (prevents inferring identity pattern)
- No "Apply" or "Contact" button — tutors cannot self-initiate; they view details and admin handles matching
- Card max-height: 220px; no truncation of key fields; secondary details hidden behind "View Details"

### Mobile Variant (390px)


Implementation example omitted. This public edition retains the written requirements in this section.


**Navigation placement:**
- Desktop nav: add "Browse Requests" link visible only when logged in as TUTOR
- Mobile nav drawer: "Browse Requests" appears under "For Tutors" section
- Tutor dashboard sidebar: "Browse Requests" as primary nav item alongside "My Posts"
- Landing page "For Tutors" section: "Step 2 — Browse Requests" card links here

---

## Screen 19: Tutor Dashboard (/tutor/dashboard)

**Route:** /tutor/dashboard  
**Layout:** Tutor Layout (TUTOR role guard)  
**Purpose:** Tutor's home — view and manage their listing posts; see match notifications


Implementation example omitted. This public edition retains the written requirements in this section.


**Mobile:** Stats stack vertically (1-col grid); listing cards full-width; [+ New Listing] button pinned to bottom.

**Design constraints:**
- Status badge colors: PENDING = navy/10 + navy text; APPROVED = green-100 + green-800; REJECTED = red-50 + red-800
- "Edit Draft" only shown for DRAFT or REJECTED status
- Contact info never shown — admin mediates all matches
- Commission count is S$0 until Module 6 ships (show "—" instead of S$0 in Phase 2)

---

## Screen 20: Create Tutor Listing (/tutor/listings/new)

**Route:** /tutor/listings/new  
**Layout:** Tutor Layout (TUTOR role guard)  
**Purpose:** Multi-step wizard for tutor to create a listing post  
**Gate:** Commission acknowledgement must be confirmed before wizard begins


Implementation example omitted. This public edition retains the written requirements in this section.


**Navigation:** ← Back / Continue → at each step; Back on step 1 returns to dashboard.

**Design constraints:**
- Commission gate shown only when tutor commission Acknowledged equals no; once acknowledged, go straight to wizard
- Rate range linked to Rate Guide — show "View rate guide" link near rate inputs
- Do NOT show contact fields in the form — they are already stored from registration
- Rejection reason shown as inline notice if resubmitting after rejection: "Your previous listing was rejected: [reason]. Please review and correct before resubmitting."
- Availability grid identical component to tutee Create Request (Availability Grid)
