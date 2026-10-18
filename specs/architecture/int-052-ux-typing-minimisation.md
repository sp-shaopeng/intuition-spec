# INT-052 — Minimise user typing / more pickers (user experience spec)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** team-reviewed + Boss-corrected 2026-06-03 (SEND BACK→corrected; build may proceed on this text) ·
**Date:** 2026-06-03
**Owners:** uiux-designer (lead) + frontend-expert + product-designer · frontend-only; no the private implementation folder programming interface change expected
(geo is frontend-canonical — see §1). Builds on the INT-049 design-system + layout primitives.

> **Team-review/Boss decisions folded in (2026-06-03):** dual-handle slider **CUT** — preset bands only (§2);
> onboarding pickers (#6) **moved to INT-043** (don't double-own); one-time passcode = six labelled paste-aware inputs and is
> **polish, not a fix** (the field already has input Mode is numeric + auto Complete is one-time-code,
> Reset Password implementation reference); geo = **frontend-canonical constant** (no backend area table exists), names **≤64 chars**.
> Four factual corrections vs the real code are baked into §1/§2 below. Open decisions closed except two (§"Open").

## Problem
A post-redesign user experience review found the marketplace forms are *mostly* well-constrained already (subjects/levels =
multi-select; student Count/venue/group-size = <select>; region + commitment + qualifications = enum selects;
interview times = datetime-local + 15-min step; invite = post <select>; phone auto-strips; rating = star
picker). But several fields still force **free-text typing** where a picker or a derived value would be faster,
lower-error, and better for downstream data quality. This spec defines those changes, ordered by ROI.

## Scope — changes (current state cited from source)

### 1. Geo fields → searchable combobox (HIGHEST ROI) — user experience *and* data quality
- **Current (verified):** the MRT input registers as location Mrt (Create Listing implementation reference,
  "Preferred MRT station / area (optional)", an optional value limited to 100 characters) and maps to payload location Mrt Station;
  location Planning Area (tutor Profile implementation reference) is free text. location Region is already an enum select
  (Location Region: CENTRAL/NORTH/SOUTH/EAST/WEST, Location Region implementation reference).
- **Proposed:** a type-to-filter **Combobox** primitive (new Combobox implementation reference) for planning area
  and MRT, **cascading from region** (region → planning area → MRT shortens each list). Source the fixed SG
  lists as geo implementation reference: ~5 regions, ~28 planning areas (with region mapping), ~140 MRT stations (with
  area/region mapping). Combobox keeps the field **optional** (don't make geo required).
- **Why it matters beyond typing (corrected):** the public browse filter is **region-enum only** today
  (Marketplace Filter Bar implementation reference — "subject / level / region only … typed enums"); there is **no
  planning-area filter** to break. So the win is **(a)** less typing on the tutor form (revenue: less
  abandonment), **(b)** canonical, consistent area strings for display, and **(c)** *unlocking* future
  area-level filtering / matching / analytics that free text makes impossible. (The earlier "typo'd area never
  matches the filter" framing was wrong — no such filter exists.)
- **Backend note (corrected):** there is **no backend planning-area reference table** — location Planning Area is a
  free text column with **no FK** (unlike subjects/levels, which ARE FK-backed per taxonomy implementation reference). It is
  length 64 on Tutor Profile implementation reference and length 100 on Tutor Post implementation reference/Tutee Post implementation reference. **Hard
  constraint:** every name in geo implementation reference must be **≤64 chars** or a tutor-profile save throws. geo implementation reference therefore
  *is* the de-facto canonical SG geo source — **owned by frontend**; a backend-served reference endpoint is the documented
  **future migration path**, not a v1 dependency. Emit **human-readable display strings** (e g. "Tampines"), not
  codes (no code table exists), so existing rows stay comparable. No application programming interface/contract change.

### 2. Budget / rate → preset bands (slider CUT)
- **Current (verified):** two number inputs — Create Request implementation reference (budget Min/budget Max),
  Create Listing implementation reference (hourly Rate Min/hourly Rate Max). **min max is already prevented** — both
  forms .refine (Create Request implementation reference budget Max is at least budget Min; Create Listing implementation reference
  hourly Rate Max is at least hourly Rate Min). So the issue is **pure typing**, not a validation gap.
- **Proposed:** **preset bands** tied to the existing Rate Guide ($20–30 / $30–40 / $40–60 / $60+), emitting the
  existing min/max numbers. **The dual-handle slider is CUT** (team decision): a 375px dual-thumb slider is a
  fat-finger + a11y liability on mobile; bands are clearer to Wei/Priya and trivially keyboard-accessible. Keep a
  numeric override for power users (preserves the existing .refine). Reuse the S prefix + validation messages.
  Benefit = **typing reduction** (min>max already handled).

### 3. Derive the listing title (don't ask)
- **Current:** Create Listing implementation reference title + 277 headline are free text.
- **Proposed:** auto-compose a **suggested title** from the already-selected subject using s + level using s + area
  (e g. "P5 Math · Tampines"), pre-filled + editable (an "edit title" affordance). Typing only to override.
  headline stays optional free text with an example placeholder.

### 4. one-time passcode → segmented 6-box, paste-aware input (POLISH, not a fix)
- **Current (verified):** Reset Password implementation reference is a single text Input that **already** has
  input Mode is numeric + auto Complete is one-time-code (so the numeric keypad + SG SMS autofill already work)
  and validation requiring exactly six numeric digits. The segmented version is a **polish/affordance** improvement, not a bug
  fix — scope accordingly (Wave 3, "Should").
- **Proposed:** a **Segmented Otp Input** — **six labelled inputs** (aria-label is Digit n of 6 each), auto-advance,
  backspace-to-previous, **paste-spread of a 6-digit code across all cells**, input Mode is numeric,
  auto Complete is one-time-code retained, digits-only sanitisation. New Otp Input implementation reference. (Team
  chose six labelled inputs over one masked field so the test matrix + SR behaviour are explicit.)

### 5. Slot Picker quick-pick chips
- **Current:** Slot Picker implementation reference uses datetime-local + 15-min step (good).
- **Proposed:** add **quick chips** — Today / Tomorrow / This weekend + common times (4pm/5pm/7pm) — that
  prefill a valid 15-min slot; the datetime-local stays for fine control. Fewer scroll/type interactions.

### 6. Onboarding pickers → MOVED to INT-043 (do not double-own)
- **Team decision:** the post-login profile-completion pickers (tutor-type + commitment segmented toggles,
  years-experience stepper 0–1 / 2–4 / 5–9 / 10+, subjects multi-select, no free numbers) belong to **INT-043**
  (tutor onboarding) and are tracked there — INT-052 does **not** implement them, to avoid two issues owning the
  same screen. Kept here only as the design intent INT-043 should honour.

## Broader user experience (lower-effort, bundle where natural)
- **Cascading selects** region → planning area → MRT (keeps every list short).
- **Sensible defaults + remember-last-used** (venue, commitment).
- **Autosave/draft** on the multi-step create forms (they're long).
- **Typeahead inside** the subjects multi-select if the list grows.
- **Make free-text optional** (headline, teaching Approach) with example placeholders — pairs with INT-043 so
  onboarding isn't blocked on prose. Character counters on the remaining textareas (special Notes,
  teaching Approach).

## Guardrails / constraints
- frontend-only; reuse INT-049 layout primitives + tokens; brand hex/fonts LOCKED; no new heavy deps (build Combobox /
  Range Slider / Otp Input as small in-house components, pure Tailwind + the tokens — like the §2.2 primitives).
- No magic strings — geo lists in geo implementation reference mirroring taxonomy implementation reference; emit canonical codes/values.
- No behaviour/application programming interface change except the geo canonicalisation (which must match the backend's expected planning-area set).
- frontend coverage gate ≥97.5% line / ≥88% branch (v8 summary); new primitives ship with their own tests; the
  Combobox + Otp Input need keyboard + paste + a11y tests (WCAG 2.2 AA, aria-activedescendant for the combobox).

## Acceptance criteria
1. Geo: planning-area + MRT are pickers (no free-text), cascading from region; values are canonical
   human-readable strings (**≤64 chars**) emitted into the existing location Planning Area/location Mrt Station
   fields; geo implementation reference is the single source with a region→area→MRT mapping. (No browse-filter assertion —
   no area filter exists today; this enables future area filtering, doesn't fix a current one.)
2. Budget/rate: **preset bands** (slider cut) work on mobile (375px) + keyboard and emit valid min/max; the
   existing .refine (min ≤ max) is preserved by the numeric override.
3. Title: a sensible title is pre-filled from selections and editable; submitting without typing a title works.
4. one-time passcode: a pasted 6-digit code fills all cells; keyboard nav + screen-reader labels work.
5. Slot Picker chips prefill valid future 15-min slots; datetime-local still available.
6. Onboarding fields are pickers/steppers, not free numbers.
7. Gates green; a11y checks on the new inputs; no behaviour/application programming interface regression.

## Open decisions
**Closed by team-review/Boss (2026-06-03):**
- ~~Slider vs bands vs both~~ → **preset bands** (slider cut; mobile a11y).
- ~~Geo canonical list location~~ → **frontend constant geo implementation reference now**, no backend table exists; backend reference endpoint is
  the documented future migration path. Names **≤64 chars**, human-readable strings.
- ~~Scope/sequencing~~ → **three waves** (below); **#1 geo ships first, alone**. **#6 → INT-043.**

**Still genuinely open (decide during build):**
- Geo: **MRT stations vs planning areas vs both** in v1 (lean: planning-area first — it's the matching/display
  unit; MRT is a nicety; both share the geo implementation reference source).
- Derived title (#3): **store the derived string, or keep title nullable and render from parts?** — note
  the historical listing title was required and limited to 5–100 characters, so "store the derived
  string, pre-filled + editable" is the lower-drift path unless the field is made nullable. Decide in Wave 2.

## Rollout (team-reviewed)
- **Wave 1 (Must):** geo combobox + geo implementation reference (#1) — highest ROI, lowest risk. Combobox a11y + branch
  tests land **before** page integration.
- **Wave 2 (Should):** budget/rate **bands** (#2) + derived title (#3).
- **Wave 3 (Should/Could):** one-time passcode segmented input (#4, "Should" — polish) + slot quick-pick chips (#5, "Could").
- **#6 onboarding pickers → tracked in INT-043** (not this issue).
Each wave frontend-only, gate-verified (≥97.5% line / ≥88% branch, v8 summary), Boss-signed before the next.
