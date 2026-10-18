# INT-074 — Mobile-Web Responsiveness Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Issue:** INT-074 — the user interface is not mobile-web friendly.
**Authors:** frontend-expert (Sam) + frontend-designer + uiux-designer (Jamie).
**Status:** Spec (doc only — no code). frontend-only; application programming interface contract frozen.
**Breakpoint audited:** ~390px (iPhone 12/13/14 logical width; Tailwind <md = <768px).
**Date:** 2026-06-05.
**Dependency:** workstream **A-frontend (admin identity / Identity Link)** touches the SAME admin pages
(Commission, Matching, Interview Queue, Job Applicants, Users). This spec's card renderers MUST layer on
top of A-frontend's identity cells — see §5. Coordinate so neither overwrites the other.

---

## 1. Problem

The INT-049 redesign and the 7-wave flow-restructure gave the **public / tutee / tutor** flagship flows real mobile
polish: a mobile bottom nav (Mobile Bottom Nav implementation reference, rendered by Tutee Layout/Tutor Layout), sticky CTAs with
spacer siblings, grid-cols-1 md:grid-cols-N collapse, and Data List card-mode (<md cards / ≥md table). But
coverage is **uneven**: the **admin surface** never received the same treatment, and a handful of shared
primitives/forms still ship desktop-only layout. At 390px these surfaces produce **horizontal overflow, clipped
content, and sub-44px touch targets** — the admin (the operator's daily console) is effectively unusable on a phone.

The viewport meta tag is correct (index html — width is device-width, initial-scale is 1.0), so this is a layout
problem, not a scaling one.

---

## 2. Current state in code (offender inventory — file:line, ranked by severity)

### Severity 1 — content lost / page wider than the viewport

| # | File:line | Defect | Why it breaks at 390px |
|---|-----------|--------|------------------------|
| 1 | Interview Queue implementation reference | **7-column raw ** wrapped in overflow-hidden | 7 columns (Tutee · Tutor Listing · Job · Status · Requested · Scheduled · Actions) at w-full blow far past 390px; overflow-hidden (not overflow-x-auto) **clips the overflow — the Actions column and schedule controls are simply unreachable** on a phone. Worst offender. |
| 2 | Commission implementation reference | **7-column raw ** (Match · Subjects · Monthly Fee · Commission · Matched · Status · Actions) wrapped in overflow-hidden | Same clip — the "Mark paid" / "Activate" actions and the inline activate-fee row (Commission implementation reference, a 5-col-span input + 3 buttons) overflow and are clipped. |
| 3 | Matching implementation reference | **6-column raw ** (Tutor↔Tutee · Job · Status · Commission · Created · Actions), rows via Match Row (Matching implementation reference), wrapped in overflow-hidden | Actions column clipped; cancel/activate unreachable. |
| 4 | Users implementation reference | Data List used but with **mode is table** — forces table-only at **all** breakpoints, **no card collapse <md** | The one primitive built to fix this is being told NOT to collapse. The user table overflows on a phone despite using Data List. |
| 5 | Admins implementation reference | Same as #4 — Data List mode is table, no <md cards | Admin-accounts table overflows on a phone. |

> Note: overflow-hidden on #1–#3 is *worse* than no wrapper — it clips rather than allowing scroll. The correct
> minimum-bar fallback is overflow-x-auto (proven safe at Rate Guide implementation reference and Data List implementation reference).

### Severity 2 — horizontal scroll, but content reachable

| # | File:line | Defect | Note |
|---|-----------|--------|------|
| 6 | Analytics implementation reference | 3-column raw  (Date · Metric bar · Value) in overflow-hidden | Only 3 cols + an inline progressbar; the **bar-chart rows don't card cleanly** (the bar is the point) — this is the case for the **responsive-table-wrapper fallback** (overflow-x-auto), not a card. Lower severity (3 cols ~ fits) but still clipped by overflow-hidden. |

### Severity 2 — touch targets < 44px (WCAG 2.2 AA 2.5.8 Target Size)

| # | File:line | Element | Measured size |
|---|-----------|---------|---------------|
| 7 | Button implementation reference | size is sm = h-9 (**36px**) | Used pervasively across admin pages (size is sm on every action button in Commission/Matching/Interview Queue). 36px < 44px. |
| 8 | Availability Grid implementation reference | Slot cells w-10 h-10 (**40px**) | 40px < 44px; the grid is min Width (Availability Grid implementation reference) inside overflow-x-auto — scroll OK, but each cell tap target is undersized. |
| 9 | Availability Grid implementation reference | Quick-set chips px-3 py-1.5 text-xs (**~28px tall**) | Well below 44px. |
| 10 | Admin Layout implementation reference | "Log Out" bare text button, no min-height; nav links px-3 py-2 text-body-sm (Admin Layout implementation reference, ~32px tall) | Header actions and the overflow-scroll nav links are under 44px. |
| 11 | Modal implementation reference | Close "×" button — text-xl leading-none, no width/height/padding box | The dialog dismiss affordance is a bare glyph, far under 44×44px. |

### Severity 3 — forms / layout polish

| # | File:line | Defect | Note |
|---|-----------|--------|------|
| 12 | Create Request implementation reference | grid grid-cols-2 gap-4 for Min/Max budget inputs — **no <md collapse** | Two number fields side-by-side on a phone (violates single-column-form rule). Short numeric fields, so usable, but cramped at 390px. |
| 13 | Create Listing implementation reference | Same grid grid-cols-2 gap-4 pair | Same. |
| 14 | Modal implementation reference | w-full max-w-md mx-auto — **no side gutter** (max-w-md = 448px) | At 390px w-full caps to viewport but the dialog touches both screen edges (no mx-4/inset); content padding is px-6. Tight but not overflowing. |

### Confirmed NON-issues (already correct — reuse as the reference pattern, do not "fix")

- **Viewport meta** — index html correct.
- Rate Guide implementation reference — raw table **correctly** wrapped in overflow-x-auto rounded-card border
  (this is the §3 fallback pattern, proven).
- Data List implementation reference — the table already scrolls within its container; display mode, card rendering and visual variant
  already exist and work (auto = cards <md / table ≥md).
- Job Applicants implementation reference — already routed through Data List (default mode is auto → collapses to
  cards <md). **Reference for how the other admin pages should look.**
- Analytics implementation reference — summary stat cards correctly grid-cols-1 sm:grid-cols-2 lg:grid-cols-4.
- Marketplace Filter Bar implementation reference — flex flex-wrap with min-w- 200px; a 200px item fits at
  390px and wraps to single column. Safe.
- Mobile Bottom Nav implementation reference — min-h- 44px targets, spacer sibling, md:hidden. Correct; the
  model for sticky-bar non-occlusion.
- Notification Bell implementation reference min-w- 18px / text-2xs — a numeral count badge, not a tap target. OK.

---

## 3. Design / fix pattern (reuse, don't reinvent)

### 3.1 Admin tables → Data List card-mode <md (S1 #1–#5)

The fix already exists in the codebase. Route each raw-table admin page through the existing Data List with
variant is admin, mode is auto (cards <md, table ≥md), and a render Card that mirrors the desktop row:


Implementation example omitted. This public edition retains the written requirements in this section.


- **Interview Queue / Commission / Matching (S1 #1–#3):** convert the hand-rolled / + inline
  loading/empty/error into columns + render Card. This deletes the bespoke skeleton/empty rows (the page gets the
  5-state contract for free) AND collapses to cards <md. The inline activate-fee editor
  (Commission implementation reference) and inline schedule editor (Interview Queue implementation reference) move into the card body so they
  stack vertically on a phone instead of overflowing a col Span.
- **Users / Admins (S1 #4–#5):** change mode is table → mode is auto and add a render Card. The Data List
  already renders a column-driven default card if render Card is omitted, but these pages have action columns, so a
  purpose-built render Card (one primary action per card, per Von Restorff) is preferred.
- **A-frontend dependency:** A-frontend is adding Identity Link cells to these same admin tables. The render Card here must
  render A-frontend's identity cell (not a duplicate). Land this AFTER or co-ordinated WITH A-frontend — the card render is the
  natural home for the identity cell. Do not both edit the same columns array blind. (SHARED-LESSONS md
  concurrency rule: same-subtree edits run sequentially.)

### 3.2 Responsive-table-wrapper fallback where a card doesn't fit (S2 #6)

Analytics implementation reference is a **bar chart rendered as a table** — the horizontal bar *is* the data, so a stacked card
loses the comparison. Fix = swap overflow-hidden → overflow-x-auto (the Rate Guide implementation reference pattern), and let it
scroll on the narrowest screens. 3 columns means it nearly fits; the wrapper guarantees no clip. This is also the
universal **minimum-bar fallback**: any admin table that can't justify a card before A-frontend lands MUST at minimum have
its overflow-hidden wrapper changed to overflow-x-auto so content is reachable.

### 3.3 Touch targets ≥ 44px (S2 #7–#11)

- **Button sm (Button implementation reference):** raise h-9 → h-11 (44px) for sm, OR (lower-blast-radius) give the admin
  action buttons size is md on the mobile card render. Decide in §7. If h-9 stays for dense desktop tables,
  ensure the **card render uses size is md** so the mobile target is ≥44px.
- **Availability Grid cells (70):** raise w-10 h-10 → w-11 h-11 (44px). Grid is already in overflow-x-auto,
  so the wider grid just scrolls; min Width updates accordingly.
- **Quick-set chips (36,41):** py-1.5 → py-2.5 (or min-h- 44px).
- **Modal × (Modal implementation reference):** give the close button min-h-[44px] min-w-[44px] inline-flex items-center
  justify-center (keep the glyph, enlarge the hit area).
- **Admin Layout Log Out + nav links (56,71):** py-2 → ensure min-h- 44px on the tap target.

### 3.4 Forms single-column <md (S3 #12–#13)

Create Request implementation reference / Create Listing implementation reference: grid grid-cols-2 gap-4 → grid grid-cols-1 sm:grid-cols-2
gap-4. Min/max budget stacks on a phone single-column-form rule , side-by-side fromsm up.

### 3.5 Modal viewport-safe (S3 #14)

Modal implementation reference: w-full max-w-md mx-auto → add a side gutter so the dialog never touches the screen edge, e g.
the calc operation (or max-w-md mx-4). max-w-md (448px) already caps desktop width; the only
change is the 390px inset. No focus-trap change (native dialog the show modal operation already traps).

### 3.6 Admin mobile nav (no change required)

Admin Layout implementation reference keeps the horizontal overflow-x-auto nav (admin is desktop-first by design — design-direction
§4 explicitly excludes admin from the bottom tab bar). Only the per-link **touch target** (§3.3) is in scope. Do NOT
add a Mobile Bottom Nav to admin — that's out of scope and contradicts the design direction.

---

## 4. Acceptance criteria

1. **No horizontal overflow at 390px** on every admin surface (Commission, Matching, Interview Queue,
   Analytics, Users, Admins, Job Applicants) AND the key public/tutee/tutor flows. document document Element
   /body scroll Width ≤ client Width (≤ ~390 + scrollbar) on each route.
2. **No clipped content** — every action (activate, mark-paid, cancel, schedule, choose, delete) is reachable on a
   phone (the S1 overflow-hidden clip is gone).
3. **Touch targets ≥ 44×44px** on all interactive elements on mobile-rendered surfaces (Button sm, Availability Grid
   cells + chips, Modal ×, admin nav/logout).
4. **Readable type** — no body/control text below 14px on mobile (the 12px text-xs helper/caption usages are
   acceptable as secondary captions per design tokens; no *interactive label* below 14px).
5. **Modals fit** — dialog has a side gutter at 390px, content not edge-to-edge, focus trap intact.
6. **frontend gate:** the documented operational check 0, the documented operational check (tsc) 0, the documented operational check ≥ **97.5% line / ≥88% branch** (Vitest;
   current config vite config implementation reference). Existing admin page tests
   (Commission,Matching,Interview Queue,Analytics,Users,Admins,Job Applicants .test implementation reference) updated
   for the card render (assert card mode via the md:hidden/hidden md:block class, NOT DOM removal — jsdom has no
   layout; SHARED-LESSONS md INT-046 rule).
7. **Verified on a real mobile viewport** — the Playwright mobile-chrome project (Pixel 7, playwright config implementation reference,
   currently @full-gated) OR devtools at 390px: walk each admin page + the flagship flows, screenshot, confirm no
   horizontal scrollbar and all actions tappable.

---

## 5. Test notes

- **jsdom can't measure layout.** Assert responsive behaviour via the Tailwind class on the node
  (md:hidden on the card , hidden md:block on the table wrapper — see Data List test implementation reference), never by
  scroll Width/get Bounding Client Rect (always 0 in jsdom) and never by DOM-removal of the hidden sibling.
- **Real-overflow assertion belongs in Playwright** (mobile-chrome), not Vitest: set viewport 390×844, navigate,
  assert check that the page’s content is no wider than the browser viewport and that each action
  button is the to be visible operation + has bounding box height ≥ 44.
- **Data List card-mode coverage footgun (Wave-5 lesson):** for a card-only surface pass mode is cards explicitly;
  these pages stay mode is auto (both layouts in the DOM). Cover BOTH the card render and the table render — the
  render Card arrow is its own branch. Enumerate every modal close path (on Close prop, header ×, each footer
  dismiss) per the INT-046 modal-coverage lesson, asserting closure via the open attribute.
- **Touch-target tests** assert the size class is present (h-11, min-h- 44px) — jsdom can't compute px, so the
  class is the testable contract; the real px check is the Playwright bounding-box assertion.
- **A-frontend shared tests:** if A-frontend has already added identity-cell assertions to these admin test files, extend (don't
  replace) them; run the combined suite once A-frontend merges to confirm no regression on identity rendering.

---

## 6. Open decisions

1. **Button sm globally 44px vs admin-card-only size is md?** Raising Button implementation reference h-9→h-11 fixes every
   sm button at once (lowest LOC, app-wide consistency) but changes dense desktop admin-table button rows (taller
   rows). Alternative: keep h-9 for desktop density, force size is md only in the mobile card render. **Recommend:
   raise sm to h-11** — 36px was already a WCAG fail everywhere, and the desktop row-height change is cosmetic.
   Defer to tech-lead/frontend-designer.
2. **Analytics chart: card or scroll-table?** This spec says scroll-table (the bar is the data). If product wants a
   mobile-native chart (sparkline list), that's a larger redesign — out of INT-074 scope; flag separately.
3. **Create Request/Create Listing budget pair:** strict single-column (grid-cols-1 sm:grid-cols-2) vs leave the 2-up
   (short numeric fields fit). **Recommend single-column <sm** per the form rule; low risk.
4. **A-frontend sequencing:** land A-frontend (identity) first, then this card-render layers the identity cell in — OR build the
   card render with a placeholder and reconcile. **Recommend A-frontend first** to avoid a same-file collision
   (SHARED-LESSONS md 2026-06-04 "ground on the exact element" + concurrency rule).

---

→ Boss review → fix → Boss review → team-review
