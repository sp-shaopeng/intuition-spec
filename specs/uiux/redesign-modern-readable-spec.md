# Public user interface/user experience Redesign — Modern & Reader-Friendly (INT-049)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** The never-ever contact claim is replaced by scoped privacy copy. Public filtering is allowed within limited results; current authorized assigned-participant disclosure applies, with frontend ACTIVE/COMPLETED visible and match-PENDING hidden. Approval alone never releases contact.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

Status: **OPEN — spec authored, awaiting team-review → Boss → implementation** ·
Authors: user interface/user experience Designer (Jamie, lead) + Frontend Designer + Frontend Expert ·
Date: 2026-06-03 · Tracks: INT-049 (Sev S2, frontend · user interface/user experience)

Founder directive (2026-06-03): the public surface is judged dated and not
reader-friendly. **Modernize WITHIN the existing brand.** The navy/clay/cream
palette and the Instrument Serif / Inter pairing in
[design-tokens implementation reference](../domain/uiux/design-tokens.md) are **LOCKED**
— this spec changes spacing, type scale, hierarchy, elevation, component polish,
and readability ONLY. No hex changes, no font changes, no new brand colours.

**Sequencing (fixed): PUBLIC-FIRST.** Landing → Browse Tutors → Browse
Assignments → Login → Register → then tutee/tutor dashboards → admin **last**.

**Non-negotiables preserved verbatim:**
1. Hero headline "Meet your tutor before you commit." (Landing implementation reference).
2. The privacy promise — "your contact details stay private during browsing and interview arrangements" and every personal information-free public surface.
3. Admin purple separation (#7 C3 AED shell) — untouched; admin is the LAST phase.
4. Commission **50% / SGD** prominence on the tutor registration commission page.

This spec ADDS to design-tokens md *usage and scale guidance only* (§2). It does
not redefine any locked token.

---

## 1. Audit — "not modern / not reader-friendly" findings (public surface)

Findings are concrete and cite file:line. Severity: **P0** blocks the redesign
goal, **P1** material, **P2** polish.

### 1.0 System-level (the root causes)

- **P0 — No design-system scale wired into Tailwind.** tailwind config implementation reference
  extends colors and font Family only. There is **no** box Shadow,
  border Radius, font Size, spacing, line Height, or screens extension.
  The rich scale already specified in design-tokens md §3–§5 (4px spacing,
  4-step shadow, radius ladder, 13-step type scale) exists **on paper but not in
  code** — so every component reinvents values inline (text- 72px,
  text- 48px, rounded-lg, rounded-xl, rounded, shadow-md). This is the
  single biggest "not modern" driver: inconsistency reads as unpolished.
- **P0 — No global readability baseline.** index css sets only
  font-family, background, and font-smoothing on body. There is **no base
  line-height, no base color, no base font-size, and no prefers-reduced-
  motion block.** Body copy inherits browser defaults; long paragraphs run the
  full container width with no measure cap (see §1.1).
- **P0 — No prefers-reduced-motion handling anywhere.** animate-pulse
  (Skeleton implementation reference), animate-spin (Spinner implementation reference, Button implementation reference), and
  every transition-* run unconditionally. WCAG 2.2 + founder a11y rule require
  honouring the OS reduced-motion setting.
- **P1 — Badge/status colours bypass the locked palette.** Badge implementation reference uses
  raw Tailwind palette (bg-amber-100 text-amber-800, bg-green-100,
  bg-blue-100, bg-emerald-100) instead of the semantic status tokens defined
  in design-tokens md §1.3. The off-brand greens/blues are a visible "generic
  Tailwind" tell and violate the No-Magic-Strings spirit.
- **P1 — Inputs suppress the focus ring.** Input implementation reference is focus:outline-none
  with only a border-colour change and **no focus-visible ring** — buttons get a
  ring (Button implementation reference) but text inputs do not, an inconsistent and weaker
  keyboard-focus affordance.

### 1.1 Landing (Landing implementation reference)

- **P0 — Hero type is desktop-hardcoded, not responsive.** Landing implementation reference
  text- 72px leading- 1.1 with no mobile override. At 375px a 72px serif
  headline of "Meet your tutor before you commit." overflows / wraps brutally.
  design-tokens md §2.2 already mandates type Display (72) desktop →
  type H1 Mobile (36) mobile; the code ignores it.
- **P0 — Whole page is desktop-grid-locked.** Section padding is fixed px-8
  (Landing implementation reference); the USP and "Three steps" use grid-cols-2 /
  grid-cols-3 with **no responsive prefix** (Landing implementation reference). On a
  phone these become three cramped columns, not a stack. The page was never built
  mobile-first despite 60%+ parent traffic being mobile.
- **P1 — Hero body line-length far exceeds comfortable measure.** 59
  max-w- 560px at 20px ≈ 75–80ch. Comfortable is 60–70ch. Combined with
  leading-relaxed it still reads dense on a dark field.
- **P1 — Stat row will not wrap.** 275 flex divide-x with no wrap; four
  Stat Items with border-r (40) collapse illegibly below ~640px.
- **P1 — Emoji icons mixed with line icons.** 243–245,249 the tutor "How it
  works" uses emoji (📋 📄 🤝) while the parent section uses crafted SVGs
  (185,197,216). Visual inconsistency cheapens the page.
- **P2 — Raw inline button bypasses the Button primitive.** 161 "Join Meeting"
  is a hand-rolled <button class Name is bg-clay … — off-system, no focus ring.
- **P2 — Section vertical rhythm is ad-hoc.** py-24, py-20, py-18
  (109,175,273) — py-18 is not even a default Tailwind step, so it silently
  no-ops. Rhythm should come from the spacing scale (§2.4).

### 1.2 Browse Tutors (Browse Tutors implementation reference)

- **P0 — H1 is fixed text- 48px** (124) with no mobile variant → wraps/
  overflows at 375px (design-tokens md §2.2 wants type H1 Mobile 36).
- **P0 — Loading state is a bare centred spinner** (157–161), not skeleton
  cards matching the final grid. This guarantees layout shift (CLS) when results
  pop in, and is the textbook "not modern" loading pattern. A Skeleton Card
  already exists (Skeleton implementation reference) and is used on Landing — but not here.
- **P1 — Page padding/width are desktop-only.** 123 max-w-7xl mx-auto px-8;
  the intro paragraph (125) runs full 7xl width — ~120ch line length, the
  opposite of reader-friendly.
- **P1 — Bottom CTA bypasses the Button primitive and won't stack.** 194–202
  is a raw flex items-center justify-between with an ** (169,180)
  styled as text links — fine, but inconsistent with the system link treatment.

### 1.3 Browse Assignments (Browse Assignments implementation reference)

- Mirrors Browse Tutors: **P0** fixed text- 48px H1 (63); **P0** bare
  spinner loading (90–94) instead of skeleton grid; **P1** full-width intro
  copy (64) and **P1** non-stacking raw- bottom CTA (136–147).
- **P1 — "Open" badge uses variant is scheduled** (blue) in
  Showcase Request Card implementation reference — semantically an *active/open* job is green per
  design-tokens md §1.3 status Active; blue is reserved for *informational*.
  The badge meaning is carried by text (good, Showcase Request Card implementation reference) but
  the colour is off-token.

### 1.4 Login (Login implementation reference)

- **P1 — No card/surface; the form floats on cream.** 83 is a bare
  max-w- 420px column directly on the page background — no paper card, no
  shadow, no radius. Modern auth screens sit the form on an elevated surface for
  focus and trust. (design-tokens md §5 shadow Md + §4 radius Card exist for
  exactly this.)
- **P1 — Error banner uses raw bg-red-50 border-red-200** (113) instead of
  the semantic color Error Bg/color Error Border tokens (design-tokens md §1.2).
- **P2 — H1 fixed text- 36px** (85) — acceptable on mobile but does not
  scale up; fine to keep modest here but should map to the type scale.

### 1.5 Register (Register implementation reference)

- **P0 — Single long scroll of 6+ fields, no chunking.** 160–277 stacks Name,
  Email, Phone, Password, Confirm, relationship radios, Personal Data Protection Act — a wall that
  violates Miller's law (chunk to ≤5–7) and feels heavy. (Tutor flow already
  defers commission to step 2 — good — but the first step itself is unchunked.)
- **P1 — Password-strength meter uses raw palette + off-system greys.** 64
  ['bg-gray-200','bg-red-400','bg-orange-400','bg-yellow-400','bg-green-400',
  'bg-green-500'] — none are brand tokens; yellow/orange are not in the system.
- **P1 — Role-selection cards bypass elevation + use emoji.** 83 border-2 …
  hover:border-navywith emoji icons🔍 📚 andtext-xl font-bold
  (85) — font-bold on a sans heading is off-scale (system uses 600). No
  shadow lift on hover; these are the first decision a new user makes and should
  feel premium.
- **P1 — Error banner + field error glyphs** reuse raw bg-red-50 (269) and a
  literal ⚠ (243,264) rather than tokenised error styling.
- **P2 — No card surface** (same as Login, 154).

### 1.6 Shared chrome (Nav / Footer / cards)

- **P1 — Nav is not sticky-elevated.** Nav implementation reference sticky top-0 … border-b
  with **no shadow on scroll** (design-tokens md §5 says nav gets shadow Lg
  once scroll Y>0). Flat border-only sticky reads dated.
- **P1 — Nav pre-login CTAs are confusing weight.** Nav implementation reference shows "Post a
  Request" (ghost) + "Log In" (ghost) + "Sign up" (primary) — three competing
  buttons, two of them ghost and visually identical, violating one-primary-action.
- **P2 — Cards use mixed radii + a single hover token.** Tutor Card implementation reference
  rounded-lg … hover:shadow-md, Showcase Request Card implementation reference same, but
  Skeleton Card is rounded-xl (Skeleton implementation reference) and Contact Reveal Card/
  Review Card are rounded-xl (17,11). Radius is inconsistent across the
  card family.

---

## 2. Refreshed design system — within the LOCKED tokens

Everything here is **additive scale/usage**. Locked hex (design-tokens md §1.1)
and fonts (§2.1) are untouched. The fix for nearly every P0 above is the same:
**wire the existing token scale into tailwind config implementation reference and index css, then
make components consume it.**

### 2.1 Wire the scale into Tailwind (Tailwind v3, no new deps)

Extend tailwind config implementation reference theme extend with the scales already defined in
design-tokens md (values copied verbatim from §2–§5; nothing new invented):


Implementation example omitted. This public edition retains the written requirements in this section.


Spacing already uses Tailwind's 4px base — keep the default scale (it equals
design-tokens md §3). Add screens only if a custom breakpoint is needed;
otherwise use stock sm/md/lg (mobile-first). **Do not** add a user interface/animation
library — all of this is config + utility classes.

### 2.2 Readability baseline (index css)

Add to @layer base (no font/colour token changes — uses the locked ink/heading):


Implementation example omitted. This public edition retains the written requirements in this section.


Readability rules enforced product-wide:
- **Body 16px / line-height 1.6**; hero/intro lead copy 20px / 1.65 (body-lg).
- **Measure 60–70ch** for any paragraph (cap at max-w-prose / .measure).
  Stat/label/grid text is exempt.
- **Contrast ≥ WCAG 2.2 AA**: 4.5:1 body, 3:1 large text / user interface. muted (#8C8378)
  on cream is 3.0:1 — **helper/caption only, never required labels or body**
  (carry-over rule from design-tokens md §12.3). Intro paragraphs currently use
  text-sand (e g. Browse Tutors implementation reference) at body size — **change to ink**.
- **4px vertical rhythm**: all margins/padding from the spacing scale; retire the
  silent-no-op py-18 (Landing implementation reference) → py-16/py-20.

### 2.3 Type-scale steps (the ladder, mobile→desktop)

| Role | Mobile | Desktop | Class (mobile→md) |
|---|---|---|---|
| Hero display | 36 serif | 72 serif | text-h1m md:text-display |
| Page H1 | 36 serif | 48 serif | text-h1m md:text-h1 |
| Section H2 | 30 serif | 40 serif | text-h2m md:text-h2 |
| Card / modal title | 24 sans 600 | 24 | text-h3 |
| Sub-label | 18 sans 600 | 18 | text-h4 |
| Lead / intro body | 18–20 | 20 | text-body-lg |
| Body | 16 | 16 | text-body |
| Helper / secondary | 14 | 14 | text-body-sm |
| Caption / meta | 12 | 12 | text-caption |
| Eyebrow (CAPS) | 11 | 11 | text-eyebrow uppercase |

This replaces every text- N Npx arbitrary value on the public pages.

### 2.4 Modern feel — elevation, radius, whitespace, motion

- **Elevation scale** (shadow-sm/md/lg/xl): cards rest at shadow-md, hover
  shadow-lg; nav gains shadow-lg on scroll; modals/drawers shadow-xl.
  Elevation, not heavier borders, conveys hierarchy → the single biggest "feels
  modern" lever within a flat palette.
- **Radius usage** (consistent across the card family): buttons/inputs/chips
  rounded-minimal (2px); cards/panels/CTAs rounded-card (8px); modals/drawers
  rounded-modal (12px); avatars/dots/pills rounded-full. Fix the
  rounded-lg/rounded-xl/rounded drift (Tutor Card / Skeleton / Review Card).
- **Whitespace**: generous, rhythmic. Section padding py-16 mobile / py-24
  desktop; card padding p-5 mobile / p-6 desktop; grid gap gap-6. Let copy
  breathe — density was a top "dated" signal.
- **Motion** (all behind prefers-reduced-motion): hover/focus transitions
  120–150ms; card hover = shadow + 1px translate-y lift; skeleton shimmer only if
  motion allowed (else static tint). No parallax, no autoplay, no scroll-jack.

### 2.5 Interactive states — buttons / inputs / badges

**Buttons** (extend Button implementation reference): keep variants, add a true brand-primary on
light surfaces. The current primary (Button implementation reference, cream-on-navy) is built
for the navy hero; public light-surface pages need navy-on-light. Map to
design-tokens md §6:

| Variant | Default | Hover | Focus-visible | Disabled |
|---|---|---|---|---|
| Primary | bg-navy text-cream | bg-navy-dark + shadow-sm | ring-2 ring-clay ring-offset-2 | opacity-40 cursor-not-allowed |
| Secondary | border-2 border-navy text-navy | bg-cream-muted | same ring | same |
| Ghost | text-navy | bg-cream-muted | same ring | same |
| Danger | bg-clay text-white | bg-clay-light | same ring | same |

(Hero/dark-surface buttons keep the cream-on-navy treatment via an on Dark prop
or the existing classes — do not regress the hero.) Sizes sm/md/lg →
36/44/56px height (design-tokens md §6.2); **lg is the hero/CTA size**. Loading
swaps spinner for text, preserves width (already done, Button implementation reference).

**Inputs** (extend Input implementation reference): keep the underline aesthetic but **add a visible
focus treatment** — focus-visible raises the border to 2px navy **and** adds
shadow-sm below the field (design-tokens md §7.1); error → 2px clay +
tokenised message; disabled → 40% opacity + cursor-not-allowed. Keep 16px font
(iOS no-zoom). Add the missing focus ring parity so keyboard users see the field.

**Badges** (rewrite Badge implementation reference to consume status tokens, not raw palette): map
each variant to design-tokens md §1.3 (status Active green, status Pending
amber, status Matched navy-on-info, status Rejected red, status Closed grey).
Keep **text-carries-meaning** (colour is never the only signal — already true).
Fix Showcase Request Card "Open" → live/active (green), not scheduled (blue).

**Focus-visible rings, globally**: every interactive element gets
focus-visible:ring-2 ring-clay ring-offset-2 (clay reads on both cream and
white; admin keeps its purple ring). No outline:none without a replacement ring
(fixes Input implementation reference).

### 2.6 Component upgrades — the ~18 primitives

| Primitive | Change | Why |
|---|---|---|
| Button | navy-on-light primary + on Dark; shadow-on-hover; tokenised sizes; ring parity | §2.5; current primary is hero-only (Button implementation reference) |
| Input | visible focus (border + shadow-sm), tokenised error/disabled, focus ring | Input implementation reference suppresses focus |
| Badge | consume status* tokens, drop raw palette | Badge implementation reference off-brand |
| Status Pill | no change (delegates to Badge) | inherits Badge fix |
| Modal | radius modal (12), shadow-xl, focus-trap audit, ESC already native | align radius (Modal implementation reference rounded-2xl) |
| Empty State | swap emoji default for a line-icon slot; body measure on description | Empty State implementation reference emoji; modern empties use line art |
| Skeleton / Skeleton Card | radius card to match real cards; shimmer respects reduced-motion | Skeleton implementation reference rounded-xl ≠ card |
| Spinner | reduced-motion fallback (static ring); keep role is status | Spinner implementation reference always spins |
| Toast | radius card, shadow-lg, tokenised success/error/info, slide-in respects reduced-motion | Toast implementation reference uses bg-green-700 raw |
| Toggle | focus ring already good; align off-track to cream-border token | Toggle implementation reference fine, minor |
| Commitment Badge | inherits Badge | — |
| Contact Reveal Card | radius card, shadow-md; keep aria-live | 17 rounded-xl |
| Pii Notice | radius card; replace 🛈 emoji with line icon | Pii Notice implementation reference emoji |
| Profile Completeness Bar | tokenise the amber Apply Gate Banner to warning* tokens | Profile Completeness Bar implementation reference raw amber |
| Review Card | radius card; description measure | 11 rounded-xl |
| Star Rating | no visual change; verify contrast of star amber | already accessible |
| Error Boundary | give it the Empty State treatment (icon + retry) | consistency |
| Notification Bell | focus ring + badge uses error/accent token | consistency |

No primitive is removed; no new primitive is added (no scope creep). Card radius
converges on rounded-card across the whole family.

---

## 3. Page redesigns — mobile-first (375px), wireframe-level

Each page is specified at 375px first, then the desktop adaptation. All preserve
the non-negotiables. Four states (empty / loading / error / success) are
first-class for every data page.

### 3.1 Landing


Implementation example omitted. This public edition retains the written requirements in this section.


- One primary CTA emphasis: "Find a Tutor" is primary, "Find tuition jobs" is
  secondary (dual-audience, but parent-first — preserves founder order).
- Strips reserve their footprint (existing Showcase Fallback, Landing implementation reference)
  → zero CLS.

### 3.2 Browse Tutors


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.3 Browse Assignments

Identical skeleton to 3.2 with assignment copy. Specifics:
- H1 "Browse Tuition Jobs" text-h1m md:text-h1; intro to text-ink measure.
- **Loading = skeleton grid** (fix bare spinner 90).
- Showcase Request Card "Open" badge → **green active** token (fix
  Showcase Request Card implementation reference); card converges on rounded-card.
- Bottom "Are you a tutor?" CTA → Button primitive, stacks on mobile (fix 136).
- personal information-free projection (no title/contact/exact date; bucketed budget; planning
  area) — **unchanged**.

### 3.4 Login


Implementation example omitted. This public edition retains the written requirements in this section.


- Single primary action (Log In); create-account is clearly secondary (keep the
  role-honest split, Login implementation reference).

### 3.5 Register


Implementation example omitted. This public edition retains the written requirements in this section.


- Progressive disclosure preserved: tutor still defers to the commission step
  (Register implementation reference) — and the **commission page's 50% / SGD prominence is a
  non-negotiable, untouched here** (it is its own spec, design-tokens md §9).
- Password strength bars → brand tokens (navy/clay/warning/success), drop
  yellow/orange (Register implementation reference).
- Error banner + field glyphs → tokenised error styling (fix 243,264,269).

---

## 4. Guardrails

- **Tailwind v3 only.** No upgrade to v4, no CSS-in-JS, no component-kit
  (Headless user interface / Radix / shadcn / MUI). All work = tailwind config implementation reference +
  index css + class changes. The native <dialog> modal stays (Modal implementation reference).
- **No heavy deps without written justification.** No animation library
  (Framer/GSAP), no icon package install (reuse inline SVGs; standardise on one
  line-icon style). Any proposed dep must state bundle cost + the cheaper
  alternative rejected, and pass Frontend Expert review.
- **Core Web Vitals budget (public pages, mobile 4G):** **LCP ≤ 2.5s**
  (hero text is the LCP element — must not depend on a lazy chunk; keep the font
  display is swap, already set index css), **CLS ≤ 0.1** (skeletons reserve
  every async region — Browse pages MUST adopt Skeleton Card grids; strips already
  reserve, Landing implementation reference), **INP ≤ 200ms** (filter/heart interactions stay
  optimistic — Tutor Card already is, 40–56).
- **No bundle regression.** Public route JS must not grow beyond current +5KB
  gzip without justification; keep the existing code-split of the showcase strips
  (Landing implementation reference).
- **WCAG 2.2 AA contrast** verified for every pairing (4.5:1 body, 3:1 large/user interface);
  muted stays helper-only; intro copy moves off sand to ink.
- **Keyboard navigation**: visible focus-visible ring on **every** interactive
  element (buttons, links, inputs, toggles, cards-as-links, hamburger); logical
  tab order; modal focus-trap + ESC (native <dialog>); skip-to-content link in
  the layout; no keyboard trap.
- **Reduced motion**: the index css media block (§2.2) plus per-component
  fallbacks (Spinner static ring, Skeleton static tint) — no motion-only signal.
- **Brand lock honoured**: zero changes to locked hex / fonts; status & admin
  colours come from design-tokens md tokens, never raw Tailwind palette.
- **No-Magic-Strings parity**: status colours via the token map, not literals.

---

## 5. Rollout — public-first, dashboards next, admin last

Each phase ends with /team-review → Boss verdict → fix-in-place → re-verify
before the next phase begins (founder governance rule).

- **Phase 0 — Foundation (enables everything):** wire the scale into
  tailwind config implementation reference + index css (§2.1–§2.2); upgrade the ~18 primitives
  (§2.6). No page visually "moves" until classes are swapped, so this lands
  safely first. Verify: visual-regression snapshot of primitives, a11y axe pass,
  bundle delta.
- **Phase 1 — Public pages (the INT-049 headline), in order:**
  Landing → Browse Tutors → Browse Assignments → Login → Register. Verify per
  page: 375px screenshot, CWV (LCP/CLS/INP) on throttled mobile, axe clean,
  keyboard walkthrough, non-negotiables present (hero copy, privacy line, badge
  tokens). Then /team-review → Boss.
- **Phase 2 — Authenticated dashboards** (tutee then tutor): reuse the upgraded
  primitives + layouts (Tutee Layout/Tutor Layout); same verification bar.
- **Phase 3 — Admin (LAST):** apply the modern primitives **inside** the existing
  purple shell — the #7 C3 AED separation and admin token set
  (design-tokens md §10) are preserved exactly; admin only inherits the radius/
  shadow/type/readability upgrades.

Rollout is incremental and reversible (per-page class swaps); no big-bang.

---

## Acceptance Criteria

A page is "done" only when ALL hold:
1. **Tokens wired**: tailwind config implementation reference exposes font Size/box Shadow/
   corner radius and maximum width follow section 2.1; the shared stylesheet contains the reading defaults and
   prefers-reduced-motion block per §2.2. No text- N Npx / rounded using generic
   / raw-palette status colours remain on the page.
2. **Responsive 375px**: every public page stacks cleanly at 375px with no
   horizontal scroll and no clipped headline; type uses the mobile→md ladder
   (§2.3). Verified by screenshot.
3. **Readability**: body 16/1.6, lead 20/1.65, paragraphs ≤68ch; intro copy on
   ink not sand; all pairings pass WCAG 2.2 AA (axe + manual contrast).
4. **Modern feel**: cards rest shadow-md / hover shadow-lg+lift; nav elevates
   on scroll; radius family converges on rounded-card; consistent line icons
   (no emoji on Landing/Register/Empty State/Pii Notice).
5. **States**: Browse pages use Skeleton Card grids (not bare spinners); empty,
   error using +retry, and success states present and tokenised.
6. **Interactive states**: every control has hover + focus-visible ring +
   disabled treatment; inputs show a visible focus (fix Input implementation reference).
7. **CWV budget met** on throttled mobile: LCP ≤ 2.5s, CLS ≤ 0.1, INP ≤ 200ms;
   public JS within +5KB gzip.
8. **Reduced motion** honoured (no animation when the OS requests reduce).
9. **Keyboard**: full tab traversal, visible focus, skip link, modal trap+ESC.
10. **Non-negotiables intact**: hero "Meet your tutor before you commit." verbatim;
    privacy promise present on Landing + both Browse pages; admin purple shell and
    commission 50%/SGD prominence unchanged.
11. **Brand lock**: no locked hex/font changed (design-tokens md §1.1/§2.1 diff
    is empty); status/admin colours sourced from tokens, not literals.
12. **Governance**: each phase passed /team-review + Boss SHIP.

## Open Decisions (for team-review)

1. **Login/Register card surface** — do we elevate the auth forms onto a paper
   card (§3.4/§3.5, recommended) or keep the minimalist float? Founder taste call.
2. **Line-icon source** — standardise on hand-authored inline SVGs (zero dep, our
   recommendation) vs. installing a tree-shaken icon set (bundle cost). Frontend
   Expert to rule.
3. **Mobile filter pattern** on Browse pages — bottom sheet vs. collapsible
   accordion vs. inline (current is desktop-only bar; mobile has none).
4. **Button primary on light surfaces** — introduce navy-on-light as the new
   primary and gate the hero's cream-on-navy behind an on Dark prop (§2.5), or
   add a separate primary Dark variant? Affects how many call-sites change.
5. **Stats row at 375px** — 2×2 grid (recommended) vs. horizontal scroll-snap.
6. **Register chunking** — visual groups on one screen (recommended, §3.5) vs. a
   true multi-step wizard (heavier; the tutor flow is already 2-step).

---

### Reusable lesson (proposed for SHARED-LESSONS md — not edited here)

**2026-06-03 — A design system on paper is not a design system in code
(uiux-designer + frontend-expert).** design-tokens md defined a full 4px
spacing / 4-step shadow / radius / 13-step type scale, but tailwind config implementation reference
extended only colors+font Family, so every component reinvented values inline
(text- 72px, rounded-lg/rounded-xl/rounded, raw bg-amber-100 badges) —
which reads as "not modern" and silently violates No-Magic-Strings. **Rule:** when
a token spec exists, the FIRST redesign task is to *wire the scale into the
framework config and make components consume it*; audit for arbitrary values
(text, bare rounded, raw Tailwind palette like bg-*-100) as the signal
that the system isn't actually being used. Verify with a grep, not a vibe.

---

## Boss-review conditions (2026-06-03)
1. **personal information non-regression (Casey/Security).** After any card/identity restyle, re-run and extend the
   handle-only personal information contract test (the Showcase Tutor Response personal information-contract style) so no public card or
   the new invitation surface can render a real name. Verify in the Network tab, not just the test.
2. **Sequencing.** Do **Phase 0 (wire tokens into tailwind config implementation reference) FIRST**, then let INT-046 write
   its snapshot-sensitive admin tests against the settled markup — avoids coverage rework (Sam/Reese).
3. **Domain owner.** One named engineer owns the matching/applications entity model across
   INT-034/039/044 so the three specs don't fork it (Rex).
