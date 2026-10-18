# INT-141 / INT-142 — Marketplace card hierarchy, density & trust (design-review build)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Retain the explicit reviewed prohibition on a universal Verified badge; real review/ratings evidence only. The old verified+rating shorthand is corrected below and is not approval to add a badge. [INT-143](../architecture/int-143-codereview-fixes-spec.md) pins full-filter-signature browse-event counting. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Status:** SPEC v2 — team-reviewed + Boss SEND-BACK (2026-06-13), 4 conditions folded; ready to build

> **Review history.** v1 → team-review → Boss SEND BACK (2026-06-13), folded into v2:
> (1) **P1 hierarchy is trust/fit-first, NOT price-first** (Jamie/BA) — a parent decides on fit+trust before price;
> (2) **drop the universal "Verified" badge** (Casey/Quinn) — every browsable tutor is already admin-approved, so a
> badge that's always-true is noise/misleading; keep the **rating** (the real signal) and a neutral "admin-vetted
> marketplace" line at the section level, not a per-card medal — unless a genuinely higher bar (identity/credential
> check) is introduced, which is out of scope;
> (3) **confirm the rating aggregate is on the public list data transfer object before promoting it** (Riley) — if absent, add a
> read-only aggregate (no personal information, no new column; compute from reviews); don't assume;
> (4) **add a conversion measurement note** (Blake) — instrument browse→profile view using →choose so the wave is
> falsifiable; analytics is count-only today.
**Date:** 2026-06-13
**Authors:** uiux-designer + frontend-designer (P1/P2/P3) · frontend-expert (impl/motion/mobile) · bff-specialist (rating/verified data transfer object) · product-designer (Mo S Co W)
**Source:** [frontend-design-review-2026-06-13.md](frontend-design-review-2026-06-13.md) (live-user interface verification, P1–P8)
**Tracker:** [README implementation reference](../issues/README.md) — INT-141 (S3, high-value), INT-142 (S4, distinctiveness)
**Constraints (hard):** brand LOCKED (navy/clay/cream + Instrument Serif/Inter, design-tokens md); **no new deps**; reuse existing tokens (tailwind config implementation reference font Size scale + box Shadow) + Job Identity; personal information-free (handle-only); mobile 390px overflow gate green; the INT-140 raw-code ESLint guard stays green; prefers-reduced-motion respected. No application programming interface/schema change unless a field is genuinely absent (see §rating/verified).

---

## 1. Problem

The two highest-traffic conversion surfaces read as dense, low-hierarchy data tables, not a curated marketplace
(evidence: ui-review-01-landing png, ui-review-02-browse-tutors png). Correctness/brand are fine; the headroom
is structural (hierarchy / density / trust). Grounding refined during spec: Tutor Card implementation reference already carries
shadow-md + hover-lift, so "flat" is specific to the **landing showcase cards**, not Browse.

---

## 2. INT-141 — the high-value wave (P1 + P2 + P6)

### P1 — Tutor Card hierarchy (Browse) — Tutor Card implementation reference
Today: equal-weight Rate/Area/Experience/Schedule/Commitment rows; nothing leads the eye. Rebuild the card to
ONE **trust/fit-first** hierarchy (Boss condition #1 — NOT price-first):
1. **Heading:** subject·level via Job Identity / subject Level Label (already humanised — keep). The *what*.
2. **Trust row:** rating (★ + count) — the lead trust signal — + years-experience as a compact credential.
   (No per-card "Verified" medal — see condition #2; admin-vetting is stated once at the section level.)
3. **Rate:** secondary, clear but not dominant — e g. "S$40–60/hr" in the body type, not the hero figure.
4. **Meta line:** area + a compact "also teaches" chip row.
5. **Demote, don't remove** (condition, Jamie): Experience/Commitment stay on-card as a single **muted**
   secondary line (some parents filter on full-time/part-time); Schedule can move to the detail page.
Keep the existing elevation/hover. No new fields rendered raw (INT-140 ESLint guard).

### P2 — Curate the landing strips — Active Tutors Strip implementation reference / Recent Requests Strip implementation reference (+ Showcase Tutor Card/Showcase Request Card)
Today: ~20+ tiny cards per strip = a wall of text that dilutes the hero. Cap the rendered set to **6–8**, enlarge
each Showcase*Card (more padding, stronger heading, the rate/subject promoted), and add a prominent **"See all
→"** to /tutors and /assignments. Social proof = quality of a few, not volume. (The strips already fetch
personal information-free showcase data transfer objects — this is a render/slice + card-size change, not a data change.)

### P6 — Trust signals at the decision point (Boss-corrected)
- **NO per-card "Verified" medal** (condition #2): every browsable tutor is already admin-approved, so a
  per-card badge would be universally true → noise, and "Verified" implies a higher bar than approval. Instead:
  state **"Every tutor is admin-vetted; contact stays private until you choose"** ONCE at the section/Browse
  header (honest, sets the trust frame without a meaningless medal). A real per-tutor badge returns only if a
  genuine identity/credential check is introduced (out of scope).
- Promote the **review aggregate** (★ + count) as the card's lead trust signal (null → hide; never "0 reviews").
- One-line **privacy microcopy** near the Browse/Choose CTA ("Contact stays private until you choose — Personal Data Protection Act").

**Rating data note (bff-specialist, condition #3):** confirm the public browse/showcase tutor data transfer object already carries
the aggregate rating (the live user interface shows stars on only some cards → likely sparse/optional, so verify it's on the
LIST data transfer object, not just the detail). If absent on the list data transfer object, the one permitted backend change is a **read-only rating
aggregate** computed from existing reviews — **no new column, no personal information**. Do NOT assume it's present.

---

## 3. INT-142 — distinctiveness pass (P3/P4/P5/P7/P8) — S4, follow-on

- **P3 section rhythm** (Landing implementation reference): alternate surface tones (cream/white/navy-tint) per section + serif
  eyebrows; let the hero breathe before the first strip.
- **P4 depth** (hero/CTA bands only — Browse cards already have elevation): subtle gradient-mesh/grain in the navy
  hero via tokens; hairline clay accent on showcase-card hover.
- **P5 typography**: promote section headings + key figures to Instrument Serif; widen the type scale so hierarchy
  comes from type, not weight. Fonts stay locked.
- **P7 visual anchoring**: deterministic **monogram avatar** from the public handle (colour derived from subject)
  + a subject colour-chip — personal information-free, adds scannability/recall.
- **P8 motion**: one orchestrated staggered reveal on hero + first strip; card hover elevation. CSS-only;
  prefers-reduced-motion short-circuits.

---

## 4. Mo S Co W (product-designer — keep it proportionate; these are quality, not defects)
- **MUST:** P1 (Browse card hierarchy), P2 (curate landing density) — the two surfaces every user hits.
- **SHOULD:** P6 (review evidence + actual rating + privacy microcopy) — conversion/trust, cheap.
- **COULD (INT-142):** P3/P4/P5/P7/P8 — distinctiveness; ship as a separate follow-on, don't block MUST.

## 4b. Measurement (Boss condition #4 — data-engineer)
This is a **conversion** change; it must be falsifiable. Instrument the funnel so we can tell if it worked
(analytics is count-only today, so this is additive events, not a pipeline):
- browse tutor card viewed / browse tutor profile opened (the click-through the hierarchy is meant to lift),
- landing see all clicked (does the curated 6–8 + "See all" drive into Browse?),
- downstream tutor chosen already exists — join to compute browse→choose.
Success metric: **browse→profile-open rate** up after INT-141 vs the dense baseline; "See all" CTR > 0. Without
these events the wave is redecoration we can't grade — they are part of the MUST.

## 5. Verification


Implementation example omitted. This public edition retains the written requirements in this section.


- Component tests: Tutor Card test asserts the new hierarchy + Verified chip + rating render (and humanised, never raw codes); strip tests assert the 6–8 cap + "See all".
- Visual re-verify on the dev virtual machine via Playwright (re-shoot ui-review-01/02); confirm density reduced + hierarchy reads.
- No brand drift: whole-app arbitrary-value audit stays 0 (INT-049 rule); no new deps in package implementation reference.

## 6. Ship
Per governance: /team-review → Boss → build MUST (P1/P2) + SHOULD (P6) → frontend gate + mobile gate → redeploy dev →
push sp-dev; INT-142 (COULD) as a follow-on wave. Update tracker rows on ship.
