# Frontend Design Review — user interface quality verification (2026-06-13)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

This is an advisory visual review. The universal verified-badge claim is withdrawn below; badges require a defined approved verification criterion and tutor-specific evidence. Contact copy follows the ACTIVE/COMPLETED mutually visible participant exception. The review does not establish a product policy or new runtime verification.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Method:** the frontend-design plugin rubric (distinctive direction · typography · colour/theme · motion ·
spatial composition · depth/atmosphere · avoid generic AI aesthetics) applied to the **live dev-virtual machine user interface**
(http:// REDACTED ip address) captured via Playwright. Evidence screenshots: ui-review-01-landing png,
ui-review-02-browse-tutors png, vm-smoke-int137-match-card png (wrapper root).

**Scope constraint:** the brand is **LOCKED** (navy/clay/cream + Instrument Serif/Inter, design-tokens md,
INT-049 non-negotiables). This review does **not** propose a re-skin or new fonts/colours — it grades
*execution quality within the brand* and how to make it more distinctive and trustworthy using the existing tokens.

---

## Verdict

The user interface is **clean, consistent, accessible, and on-brand** — INT-049's token system and the humanisation work
(INT-137a/138/140, verified live: match card reads "H2 Chemistry (A-Level) · Junior College 2", zero raw codes)
hold up. What it is **not yet** is *distinctive or high-trust*: the two highest-traffic conversion surfaces
(landing social-proof strips, Browse) render as **dense, uniform, flat, label:value grids** that read like a data
table, not a curated marketplace. The headroom is structural (hierarchy / density / depth / rhythm / motion),
not cosmetic. This matches the PROJECT-REVIEW-2026-06-10 Part-3 conclusion ("the headroom is NOT visual" — it's
narrative and emphasis).

---

## Findings (prioritised)

### P1 — Card hierarchy: lead with the decision, demote the rest
**Rubric:** spatial composition / hierarchy. **Evidence:** Browse Tutors (ui-review-02) — every card is a flat
stack of equal-weight Label: value rows (Rate / Area / Experience / Schedule / Commitment + "View Profile") on
a thin-bordered white box; nothing leads the eye. **Why it matters:** a parent scans for *subject·level → price →
trust (rating/experience)*; right now those compete with "Schedule" and "Commitment" at the same weight.
**Recommendation:** restructure the card to one clear hierarchy — (1) subject·level (already humanised) as the
bold heading, (2) **rate** as a large clay/serif figure, (3) a **rating row plus a verification badge only where an approved, evidenced eligibility rule is met**, (4) area + a
"also teaches" chip row; collapse Experience/Schedule/Commitment into a single muted meta line or onto the detail
page. Where: Tutor Card implementation reference (and the parallel job/assignment card).

### P2 — Density: curate, don't dump (landing social-proof strips)
**Rubric:** "generous negative space OR *controlled* density." **Evidence:** the landing (ui-review-01) renders
**~20+ tiny cards** each in "Recent job requests" and "Tutors available now" — an uncontrolled wall of text that
reads as a data dump and dilutes the hero. **Recommendation:** show a curated **6–8** with a prominent "See all"
→ the full Browse; make each shown card larger and higher-contrast. Social proof persuades by quality of a few,
not volume. Where: the landing showcase strips (Active Tutors Strip + recent-requests strip) + their page section.

### P3 — Section rhythm on the landing
**Rubric:** colour/theme cohesion + composition. **Evidence:** the page goes navy-hero → a long **flat cream**
middle (two big grids) → navy CTA → cream → cream. The long monotone cream stretch flattens the scroll.
**Recommendation:** alternate surface tones (cream / white / a subtle navy-tint band) per section, give each
section a serif eyebrow + clearer top/bottom rhythm, and let the hero "breathe" before the first grid.

### P4 — Depth & atmosphere (everything is flat)
**Rubric:** backgrounds & visual details — "atmosphere and depth rather than solid colours." **Evidence:** cards
use a thin border + near-zero shadow; the navy hero is a flat fill. **Recommendation (within tokens):** a soft
elevation ladder on cards (token shadow-sm→shadow-md on hover), a subtle gradient-mesh or grain texture in
the navy hero/CTA bands, and a hairline accent (clay) on card hover. Small, token-driven, high perceived-quality.

### P5 — Use the serif for distinctiveness; build a real type scale
**Rubric:** typography. **Evidence:** Instrument Serif appears only in the hero; everything else is Inter at
similar sizes, so cards/sections feel uniform. **Recommendation:** promote **section headings + key figures**
(rates, the "50% commission" callout) to Instrument Serif, and widen the type scale (bigger headings, smaller
muted meta) so hierarchy comes from type, not just weight. Fonts stay locked — just *used* more expressively.

### P6 — Trust signals at the decision points (conversion psychology)

**Corrected recommendation (2026-09-05):** promote available review aggregates and explain that contact remains
private until the team confirms an ACTIVE match, when mutual participant reveal is allowed. The earlier claim
that every tutor is admin-vetted was not established by this visual review. M18 permits complete-profile tutors
without published listings; profile completeness is not verification.

A "Verified by InTuition" badge is conditional on a defined, approved verification criterion with evidence for
that tutor. Do not place it on every card by default. The badge meaning/eligibility remains a product decision;
this advisory design review does not approve a universal vetting guarantee.

### P7 — Visual anchoring without photos (privacy-safe)
**Rubric:** depth / memorability. **Evidence:** no avatars (handle-only, personal information-free by design) makes every card
pure text → low recall. **Recommendation:** a generated **monogram/initial avatar** from the public handle
(deterministic colour from the subject), and a small **subject icon/colour chip** per card. Adds visual rhythm
and scannability with zero personal information.

### P8 — Motion: one orchestrated load + hover feedback
**Rubric:** motion — "one well-orchestrated page load beats scattered micro-interactions." **Evidence:** the user interface
is static. **Recommendation:** a single staggered reveal on the hero + first strip (respecting
prefers-reduced-motion, already a token concern), and card hover elevation/translate. CSS-only; cheap; high
delight-per-line.

---

## Confirmed strong (keep)
- **Login/auth** (Auth Card) — centred, restrained, correct (good minimal execution).
- **Humanisation** — match/browse/applications all read friendly labels, zero raw codes (INT-137a/138/140, live-verified).
- **Token system + a11y + the four states** — INT-049 did its job; this review builds *on* it.
- **Privacy posture** — handle-only, contact-gated; the Personal Data Protection Act footer is present.

## Recommendation
P1 + P2 are the highest value (the two surfaces every user hits) and are a focused frontend wave within the existing
tokens — no brand change, no new deps. Suggest filing P1/P2/P6 as an INT issue cluster (user experience-polish) and treating
P3–P5/P7–P8 as a follow-on "distinctiveness" pass. None are correctness defects; all are quality/conversion.

*Reviewer: frontend-design rubric × live-user interface verification (Playwright). Brand locked; recommendations are
execution-within-brand only.*

**Later disclosure amendment:** [INT-125/129](../architecture/int-125-batch-closeout-spec.md) keeps participant
reveal controls on ACTIVE and COMPLETED indefinitely; match-PENDING stays hidden in the frontend. It does not widen
unrelated ACTIVE-only sections or public/pre-assignment disclosure.
