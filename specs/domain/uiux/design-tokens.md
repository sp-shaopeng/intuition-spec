# Design Tokens

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Visual tokens do not determine permission or business-state rules. Apply the current shared layouts and 390px accessibility contracts; historical screen examples inherit the current requirements. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.
## Singapore Private Tuition Agency — Design System: Direction B Extension

**Document Version:** 2.1  
**Date:** 2026-05-27  
**Prepared By:** user experience Design  
**Source:** Existing tokens implementation reference (Direction B), business requirements document v1.1, product requirements document v1.1  
**Code Reference:** REDACTED local path

---

## Overview

This document extends the Direction B design system established in tokens implementation reference. All existing tokens are preserved unchanged. New tokens are additive. The admin panel uses a visually distinct shell (purple-tinted header and background) to make it immediately clear when a staff member is in the admin context vs the public-facing application.

---

## 1. Color Tokens

### 1.1 Brand Colors (Preserved — DO NOT CHANGE)

These tokens are defined in tokens implementation reference and must not be altered.

| Token Name | Hex | Usage |
|---|---|---|
| primary | #0 F2138 | Primary navy — headings, buttons, nav background |
| primary Dark | #070 F1 C | Darker navy — hover states on primary elements |
| accent | #C84 B31 | Clay/brick — CTAs, highlights, commission callout border |
| accent Light | #F5 DDD2 | Light clay — hover tint on accent elements |
| cream | #FBF6 EE | Warm cream — page background (public) |
| paper | #FFFFFF | White — card backgrounds |
| section | #F2 EBE0 | Warm off-white — alternating sections, context boxes |
| heading | #0 B1828 | Near-black navy — H1/H2 heading text |
| body | #3 F3 A33 | Dark warm grey — body text |
| muted | #8 C8378 | Medium warm grey — helper text, captions, placeholders |
| border | #E5 DDD0 | Warm light grey — dividers, input borders (default state) |
| whatsapp | #22 A55 D | WhatsApp green — reserved for WhatsApp CTAs |
| success | #1 F8 A5 B | Green — existing success token (see also semantic override below) |
| star | #D69 E2 E | Amber — star ratings |

### 1.2 Semantic Color Tokens (New)

Semantic tokens map intent to colour. Use these instead of raw hex values in components.

#### Status / Feedback Colors

| Token Name | Hex | Usage |
|---|---|---|
| color Success | #16 A34 A | Success states, approved badges, PAID commission, first-session confirmation check |
| color Success Bg | #F0 FDF4 | Success background tint — confirmation screen, PAID row highlight |
| color Success Border | #86 EFAC | Success border — success card outline |
| color Warning | #D97706 | Warning amber — PENDING/UNDER REVIEW badges, Day 7 waiting indicator |
| color Warning Bg | #FFFBEB | Warning background tint — AWAITING rows, service-level agreement approaching indicator |
| color Warning Border | #FDE68 A | Warning border |
| color Error | #DC2626 | Error red — REJECTED badges, form validation errors, destructive actions |
| color Error Bg | #FEF2 F2 | Error background tint — error state cards |
| color Error Border | #FECACA | Error border |
| color Info | #2563 EB | Information blue — AWAITING CONFIRMATION commission status, informational banners |
| color Info Bg | #EFF6 FF | Info background tint |
| color Info Border | #BFDBFE | Info border |

#### Admin-Specific Colors

| Token Name | Hex | Usage |
|---|---|---|
| color Admin | #7 C3 AED | Admin accent purple — admin header, active tab indicator, admin action buttons |
| color Admin Dark | #5 B21 B6 | Darker admin purple — hover states on admin-primary buttons |
| color Admin Light | #A78 BFA | Lighter admin purple — admin chip borders, secondary admin elements |
| color Admin Bg | #F3 F0 FF | Admin panel background tint — replaces cream for all admin pages |
| color Admin Bg Deep | #EDE9 FF | Admin panel section tint — contact info panel, commission acknowledgement panel |
| color Admin Bg Header | #7 C3 AED | Admin header background (solid purple, not tint) |
| color Admin Text | #FFFFFF | Text on purple admin header background |
| color Admin Border | #C4 B5 FD | Admin panel borders — subtle purple-tinted dividers |

### 1.3 Post Status Badge Colors

Each post status has a consistent semantic colour applied to its badge dot, badge background, and badge text.

| Post Status | Badge Dot | Badge Bg | Badge Text | Token Prefix |
|---|---|---|---|---|
| PENDING / UNDER REVIEW | #D97706 | #FFFBEB | #92400 E | status Pending |
| APPROVED / ACTIVE | #16 A34 A | #F0 FDF4 | #14532 D | status Active |
| MATCHED | #0 F2138 | #EFF6 FF | #0 F2138 | status Matched |
| REVISION REQUESTED | #DC2626 | #FEF2 F2 | #991 B1 B | status Revision |
| REJECTED | #DC2626 | #FEF2 F2 | #991 B1 B | status Rejected |
| CLOSED | #8 C8378 | #F5 F5 F4 | #44403 C | status Closed |
| EXPIRED | #8 C8378 | #F5 F5 F4 | #44403 C | status Expired |
| PAUSED FOR RE-REVIEW | #D97706 | #FFFBEB | #92400 E | status Paused Review |
| CONFIRM SESSION (action needed) | #C84 B31 | #FFF3 EE | #9 A1 F0 E | status Confirm |

**Confirm Session card special treatment:**
- Card background: #FFF3 EE (accent Light tint)
- Card border: 1px solid #C84 B31
- CTA button inside card: btn-primary but with accent (#C84B31) background on mobile full-width variant

### 1.4 Commission Status Badge Colors

| Commission Status | Badge Hex | Badge Bg | Badge Label |
|---|---|---|---|
| PENDING | #8 C8378 | #F5 F5 F4 | grey |
| AWAITING CONFIRMATION | #2563 EB | #EFF6 FF | blue |
| CONFIRMED | #D97706 | #FFFBEB | amber — "ready to invoice" |
| INVOICED | #EA580 C | #FFF7 ED | orange — "invoice sent" |
| PAID | #16 A34 A | #F0 FDF4 | green — "paid" |
| DISPUTED | #DC2626 | #FEF2 F2 | red |
| WAIVED | #8 C8378 | #F5 F5 F4 | grey + strikethrough text |

### 1.5 service-level agreement Indicator Colors (Admin Review Queue)

| service-level agreement State | Indicator | Hex | Label |
|---|---|---|---|
| On track | Green dot | #16 A34 A | < 1 business day |
| Approaching | Amber dot + hours label | #D97706 | 1–2 business days |
| Breached | Red badge "OVERDUE" | #DC2626 | > 2 business days |

---

## 2. Typography Scale

### 2.1 Font Families (Preserved)


Implementation example omitted. This public edition retains the written requirements in this section.


### 2.2 Type Scale

All sizes in pixels. Line heights are unitless multipliers. Weights use numeric values.

| Token | Font | Size (px) | Weight | Line Height | Letter Spacing | Usage |
|---|---|---|---|---|---|---|
| type Display | serif | 72 | 400 (regular) | 1.05 | −0.02em | Hero H1 on landing — desktop only |
| type H1 | serif | 48 | 400 | 1.10 | −0.01em | Page H1 — desktop |
| type H1 Mobile | serif | 36 | 400 | 1.15 | −0.01em | Page H1 — mobile |
| type H2 | serif | 40 | 400 | 1.15 | 0 | Section headings — desktop |
| type H2 Mobile | serif | 30 | 400 | 1.20 | 0 | Section headings — mobile |
| type H3 | sans | 24 | 600 | 1.25 | 0 | Card headings, modal titles |
| type H4 | sans | 18 | 600 | 1.30 | 0 | Sub-section labels, sidebar nav items |
| type Body Lg | sans | 20 | 400 | 1.65 | 0 | Hero body copy — desktop |
| type Body | sans | 16 | 400 | 1.60 | 0 | Standard body — forms, cards, emails |
| type Body Sm | sans | 14 | 400 | 1.55 | 0 | Helper text, secondary labels |
| type Caption | sans | 12 | 400 | 1.40 | 0 | Character counts, timestamps, muted info |
| type Eyebrow | sans | 11 | 500 | 1.30 | 0.15em | Eyebrow labels (CAPS) — section identifiers |
| type Label | sans | 13 | 500 | 1.35 | 0 | Form field labels |
| type Code | mono | 14 | 400 | 1.50 | 0 | Commission calculation formulae, code snippets |

**Responsive rule:** On mobile (< 768px), all type H1 through type H2 tokens use their Mobile variants. type Display is not used on mobile — falls back to type H1 Mobile (36px serif).

---

## 3. Spacing Scale

Base unit: **4px**. All spacing values are multiples of the base unit.

| Token | px | Usage |
|---|---|---|
| space1 | 4 | Micro — icon gap, tight inline spacing |
| space2 | 8 | Tight — between label and input, between inline elements |
| space3 | 12 | Small — between form fields (compact), chip gap |
| space4 | 16 | Base — standard component padding (mobile), grid gap (mobile) |
| space5 | 20 | Medium-small — card inner padding (mobile) |
| space6 | 24 | Medium — card inner padding (desktop), section inner padding |
| space8 | 32 | Large — between form sections, card margin |
| space10 | 40 | Section padding (mobile), between major form groups |
| space12 | 48 | Section padding (desktop compact) |
| space16 | 64 | Section vertical padding (desktop), page header padding (mobile) |
| space20 | 80 | Hero padding (desktop sides), between major landing sections (mobile) |
| space24 | 96 | Landing section padding (desktop) |
| space32 | 128 | Hero vertical padding (desktop) |

**Application pattern:**
- Component padding (card, form group, modal body): space6 (24px) desktop / space5 (20px) mobile
- Section vertical padding on landing: space24 (96px) desktop / space16 (64px) mobile
- Gap between form fields: space6 (24px)
- Gap between cards in a grid: space6 (24px)
- Inline gap between chips: space2 (8px)

---

## 4. Border Radius

The system uses a minimal radius approach consistent with the existing B radius 2 token, with progressively larger radii for higher-level containers.

| Token | px | Usage |
|---|---|---|
| radius None | 0 | Flush elements — dividers, full-width mobile buttons (not used often) |
| radius Minimal | 2 | Buttons, inputs, checkboxes, chips — the default; matches existing B radius |
| radius Card | 8 | Cards, filter panels, callout boxes, post cards, tutor cards |
| radius Modal | 12 | Modal dialogs, confirmation overlays, bottom drawer on mobile |
| radius Full | 9999 | Avatar circles, status dot indicators, toggle pills |

---

## 5. Shadow Scale

| Token | Value | Usage |
|---|---|---|
| shadow Sm | the rgba operation | Subtle — input focus ring offset, nav bottom edge |
| shadow Md | the rgba operation | Cards — default card resting state |
| shadow Lg | the rgba operation | Card hover state, sticky headers |
| shadow Xl | the rgba operation | Modal/overlay panels, bottom drawer |

**Usage guidance:**
- Cards rest at shadow Md; on hover: shadow Lg
- Modals and bottom drawers: shadow Xl to distinguish from page content
- Nav (sticky): shadow Lg when user has scrolled > 0px
- Admin panels use the same shadow scale — purple tint distinguishes them, not shadow

---

## 6. Button Variants

All buttons use border-radius: 2px (radius Minimal) unless specified. Heights use the spacing scale.

### 6.1 Button Token Table

| Variant | Token | Background | Border | Text Color | Hover Background | Usage |
|---|---|---|---|---|---|---|
| Primary | btn Primary | #0 F2138 (primary) | none | #FBF6 EE (cream) | #070 F1 C (primary Dark) | Main CTA — "Post Request", "Submit", "Approve" |
| Secondary | btn Secondary | transparent | 2px solid #0 F2138 | #0 F2138 | #EFF6 FF | Secondary action — "Browse Tutors", "Save Draft" |
| Destructive | btn Destructive | #DC2626 (color Error) | none | #FFFFFF | #B91 C1 C | Irreversible actions — "Reject", "Confirm Rejection" |
| Ghost | btn Ghost | transparent | none | #0 F2138 | #F2 EBE0 (section) | Low-emphasis — "Not yet", "Cancel", "Back" |
| Admin Primary | btn Admin Primary | #7 C3 AED (color Admin) | none | #FFFFFF | #5 B21 B6 (color Admin Dark) | Admin CTAs — "Confirm Match", "Generate Invoice" |
| Admin Secondary | btn Admin Secondary | transparent | 2px solid #7 C3 AED | #7 C3 AED | #F3 F0 FF (color Admin Bg) | Admin secondary — "Request Revision", "Override" |

### 6.2 Button Size Variants

| Size | Height | Padding (horizontal) | Font | Usage |
|---|---|---|---|---|
| btn Sm | 36px | 16px | Inter 13px / weight 500 | Quick-select buttons, table row actions |
| btn Md | 44px | 24px | Inter 15px / weight 500 | Standard — most form submit buttons |
| btn Lg | 56px | 32px | Inter 16px / weight 600 | Primary hero CTAs, commission disclosure CTA, full-width mobile primary buttons |

### 6.3 Button States

| State | Visual Treatment |
|---|---|
| Default | As defined in variant table above |
| Hover | Hover background from variant table; subtle shadow Sm |
| Focus | outline: 3px solid #7 C3 AED (color Admin) at 2px offset; visible for keyboard users |
| Active / Pressed | 3% darker than hover background |
| Disabled | 40% opacity; cursor: not-allowed; no hover effect; no pointer events |
| Loading | Spinner icon replaces text; button width maintained; disabled state styling |

---

## 7. Form Input Tokens

### 7.1 Input Appearance

The existing system uses a border-bottom underline style (no box around the input). This is preserved and extended with focus and error state overrides.

| State | Border Treatment | Shadow | Icon / Indicator |
|---|---|---|---|
| Default | border-bottom: 1.5px solid #E5 DDD0 (color Border) | none | none |
| Focus | border-bottom: 2px solid #0 F2138 (primary) | shadow Sm below the input field area | none |
| Error | border-bottom: 2px solid #DC2626 (color Error) | none | alert-circle icon, color Error, 16px trailing |
| Disabled | border-bottom: 1.5px solid #E5 DDD0 at 40% opacity | none | none; cursor: not-allowed; background: #F5 F5 F4 |
| Success | border-bottom: 2px solid #16 A34 A (color Success) | none | check icon, color Success, 16px trailing |

### 7.2 Input Typography

| Token | Value |
|---|---|
| input Font Size | 16px (Inter) — prevents iOS auto-zoom on focus (min 16px required) |
| input Font Weight | 400 |
| input Line Height | 1.5 |
| input Placeholder Color | #8 C8378 (muted) |
| input Text Color | #3 F3 A33 (body) |
| input Label Size | 13px Inter / weight 500 |
| input Label Color | #8 C8378 (muted) — above field; #0 F2138 (primary) when field is focused |
| input Helper Size | 13px Inter / weight 400 |
| input Helper Color | #8 C8378 (muted) |
| input Error Message Color | #DC2626 (color Error) |
| input Error Message Size | 12px Inter / weight 500 |
| input Char Count Color | #8 C8378 (muted) at 0/500; transitions to #DC2626 at 480+/500 |

### 7.3 Checkbox Tokens

| State | Visual |
|---|---|
| Unchecked | 16×16px box; border: 1.5px solid #E5 DDD0; border-radius: 2px; background white |
| Checked | 16×16px box; background #0 F2138 (primary); checkmark icon cream (#FBF6EE); border none |
| Focus | outline: 3px solid #7 C3 AED at 2px offset |
| Disabled | 40% opacity of unchecked state |

**Minimum clickable area (for Personal Data Protection Act consent and commission disclosure checkbox):** 44×44px touch target regardless of visual checkbox size — full label text is also clickable to toggle.

---

## 8. Availability Grid Component Tokens

The availability grid is a core interactive component used in both the tutee request wizard (Step 2) and tutor offering wizard (Step 2).

### 8.1 Desktop Grid (7 × 3 checkbox grid)

| Token | Value | Notes |
|---|---|---|
| avail Grid Cell Size Desktop | 40px × 40px | Each day × time-slot cell |
| avail Grid Cell Border Default | 1.5px solid #E5 DDD0 | Unselected cell border |
| avail Grid Cell Bg Default | #FFFFFF | White — unselected |
| avail Grid Cell Bg Hover | #F2 EBE0 | Warm off-white hover (section colour) |
| avail Grid Cell Bg Selected | #0 F2138 | Primary navy — selected/available |
| avail Grid Cell Checkmark Color | #FBF6 EE | Cream checkmark on selected cell |
| avail Grid Cell Border Radius | 2px | radius Minimal |
| avail Grid Weekend Col Bg | #FDFAF6 | Very subtle tint for Sat/Sun columns |
| avail Grid Header Font Size | 12px | "Mon"–"Sun" column headers; "Morning" row headers |
| avail Grid Header Color | #8 C8378 | muted colour |
| avail Grid Header Font Weight | 500 | |
| avail Grid Legend Font Size | 12px Inter | "Morning: before 12pm; Afternoon: 12pm–6pm; Evening: 6pm–10pm" |
| avail Grid Gap | 4px | Gap between cells in the grid |

### 8.2 Mobile Grid (Accordion-per-day pattern)

| Token | Value | Notes |
|---|---|---|
| avail Grid Cell Size Mobile | 44px minimum height | Touch target minimum for toggle buttons per WCAG 2.1 |
| avail Toggle Bg Selected | #0 F2138 | Navy when time slot selected |
| avail Toggle Bg Unselected | #E5 DDD0 | Warm grey when not selected |
| avail Toggle Text Selected | #FBF6 EE | Cream text when selected |
| avail Toggle Text Unselected | #3 F3 A33 | Body text when not selected |
| avail Toggle Border Radius | 2px | |
| avail Accordion Header Height | 48px | Day accordion header (e g., "MONDAY") — also acts as tap target |
| avail Accordion Header Bg | #FBF6 EE | Cream |
| avail Accordion Header Bg Open | #F2 EBE0 | Section colour when day is expanded |
| avail Accordion Anim Duration | 200ms | Slide animation for expand/collapse |
| avail Accordion Count Badge | #0 F2138 bg, cream text, 6px border-radius | Count of selected slots shown when day is collapsed |
| avail Default Expanded | Saturday, Sunday | These days are expanded by default (high-demand) |
| avail Default Collapsed | Monday–Friday | Collapsed by default; show selection count when user has selected slots |

### 8.3 Quick-Select Button Tokens

| Button | Label | Action |
|---|---|---|
| avail Quick Weekday Evening | "Weekday Evenings" | Selects all Evening slots Mon–Fri |
| avail Quick Weekend | "Weekends" | Selects all slots Sat + Sun |
| avail Quick Clear All | "Clear All" | Deselects all slots |

All quick-select buttons use btn Ghost + btn Sm sizing.

---

## 9. Commission Disclosure Page Tokens

The commission disclosure screen is a full-page standalone step during tutor registration. It is specifically designed to NOT look like a terms-and-conditions scroll. It uses distinct visual treatment for the key monetary values.

| Token | Value | Usage |
|---|---|---|
| commission Page Max Width | 640px | Content column max-width (centered) |
| commission Page Padding Desktop | 64px top/bottom, 40px left/right | |
| commission Page Padding Mobile | 40px top/bottom, 20px left/right | |
| commission Callout Bg | #FBF6 EE | cream — same as page background but boxed |
| commission Callout Border Left | 4px solid #C84 B31 | Clay accent left border |
| commission Callout Heading Size | 24px Instrument Serif | Rate statement ("Commission = 50% of...") |
| commission Callout Heading Weight | 400 | |
| commission Callout Heading Color | #0 F2138 | primary navy |
| commission Example Bg | #F2 EBE0 | Section off-white — worked example box |
| commission Example Border Radius | 4px | radius Minimal |
| commission Example Font Size | 16px Inter | Calculation line |
| commission Example Font Weight | 600 | Semi-bold for the SGD amounts |
| commission Example Color | #0 F2138 | |
| commission Faq Label Size | 13px Inter / weight 500 | Section labels: "WHEN IS COMMISSION DUE?" |
| commission Faq Label Color | #8 C8378 | muted — all caps, small |
| commission Faq Label Spacing | 0.10em letter-spacing | |
| commission Faq Body Size | 15px Inter | Answer text |
| commission Faq Body Color | #3 F3 A33 | body |
| commission Ack Checkbox Label | 15px Inter | Acknowledgement label text (full label is clickable) |
| commission Progress Indicator Active | #C84 B31 (accent clay) | Active step dot in 3-step progress |
| commission Progress Indicator Done | #0 F2138 (primary navy) | Completed step dot |
| commission Progress Indicator Future | #E5 DDD0 (border) | Future step dot |

**Commission page design rules (enforced):**
1. Page title uses "Commission Terms" — never "Terms & Conditions" or "Terms of Service"
2. No main site nav — only logo + progress indicator in header
3. The "50%" and the worked SGD example amount must be the most visually prominent text elements on the page (achieved via callout box + 24px serif heading)
4. Button is btn Lg (56px height) and remains disabled at 40% opacity until checkbox is checked
5. No "Skip", "Back", or alternative routing paths

---

## 10. Admin Panel Shell Tokens

The admin panel uses a distinct visual shell to make it unambiguous that the user is in an administrative context — not the public-facing site.

| Token | Value | Usage |
|---|---|---|
| admin Shell Header Bg | #7 C3 AED | Admin header background (solid purple) |
| admin Shell Header Height | 64px | |
| admin Shell Header Text | #FFFFFF | White text and icons on purple header |
| admin Shell Header Border Bottom | none | No border — the colour contrast defines the edge |
| admin Shell Nav Item Active | underline #A78 BFA (color Admin Light) | Active nav item indicator in header |
| admin Shell Nav Item Hover | background the rgba operation | |
| admin Shell Page Bg | #F3 F0 FF | Admin page background (light purple tint — color Admin Bg) |
| admin Shell Sidebar Bg | #FFFFFF | White sidebar within purple-tinted page (if sidebar used) |
| admin Shell Sidebar Border | 1px solid #C4 B5 FD (color Admin Border) | |
| admin Contact Panel Bg | #EDE9 FF | Contact info panel background (deeper purple tint — color Admin Bg Deep) |
| admin Contact Panel Border | the rgba operation | Subtle purple border |
| admin Contact Panel Border Radius | 8px (radius Card) | |
| admin Contact Panel Label Color | #5 B21 B6 (color Admin Dark) | "ADMIN ONLY — Contact Information" label |
| admin Table Row Hover Bg | #FFFFFF | White hover on purple-tinted page |
| admin Table Row Hover Shadow | shadow Sm | Subtle lift on hover |
| admin Table Header Bg | #FFFFFF | |
| admin Table Header Border Bottom | 2px solid #E5 DDD0 | |
| admin service-level agreement Green | #16 A34 A (color Success) | service-level agreement dot: < 1 business day |
| admin service-level agreement Amber | #D97706 (color Warning) | service-level agreement dot: 1–2 business days |
| admin service-level agreement Red | #DC2626 (color Error) | service-level agreement badge "OVERDUE": > 2 business days |
| admin service-level agreement Overdue Badge Bg | #FEF2 F2 | OVERDUE badge background |
| admin service-level agreement Overdue Badge Text | #991 B1 B | OVERDUE badge text (dark red) |
| admin Commission Overdue Row Bg | #FEF3 C7 | Amber-tinted row for AWAITING commission > 7 days |

---

## 11. Full Token Reference (TypeScript Extension)

The following block shows how to extend the existing tokens implementation reference file. All new tokens use the prefix convention B2 to distinguish from the original B object and avoid collisions.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 12. Component Usage Guidelines

### 12.1 When to Use Which Status Color

| Scenario | Color Token | Rationale |
|---|---|---|
| Post approved and active — tutee/tutor sees "Active" badge | status Active Dot (#16A34A) | Positive, green = go |
| Post under admin review — user sees "Under Review" badge | status Pending Dot (#D97706) | Amber = waiting, not alarming |
| Post matched — tutee sees "Matched" badge | status Matched Dot (#0F2138) | Navy = platform brand, premium feel |
| Post needs revision — user must act | status Revision Dot (#DC2626) | Red = action required |
| Post closed/expired — no action needed | status Closed Dot (#8C8378) | Grey = neutral, historical |
| First-session confirmation needed — tutee must act | status Confirm Dot + status Confirm Bg | Clay = matches brand accent; urgent but not error |

### 12.2 Admin vs Public Visual Distinction

The admin panel must be visually unmistakable from the public-facing application:

1. **Header:** #7 C3 AED solid purple vs #FBF6 EE cream (public)
2. **Page background:** #F3 F0 FF light purple tint vs #FBF6 EE cream (public)
3. **Contact info panel:** #EDE9 FF deep purple tint with purple border — makes admin-only data immediately visually distinct
4. **Primary action button:** btn Admin Primary (purple) vs btn Primary (navy)

These distinctions protect against accidental data exposure by making it instantly clear which context the user is in.

### 12.3 Accessibility Notes

- All colour pairings must meet WCAG 2.1 AA contrast ratio (4.5:1 for body text, 3:1 for large text and user interface components)
- White (#FFFFFF) on color Admin (#7C3AED) passes at ≈4.5:1 — use for admin header text
- Cream (#FBF6EE) on primary (#0F2138) passes at ≈13:1 — use for primary button text
- Muted (#8C8378) on cream (#FBF6EE) is 3.0:1 — acceptable for helper text only; do not use for required labels
- Error red (#DC2626) on white passes at 5.9:1 — acceptable for error messages
- Focus ring (outline: 3px solid #7 C3 AED at 2px offset): admin purple chosen for visibility against both cream and white backgrounds
- Touch targets: 44×44px minimum for all interactive elements on mobile (buttons, checkbox labels, accordion headers, availability toggle buttons)
