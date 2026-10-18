# Requirements — Landing page trust and social proof (005)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin**
> acceptance criteria and the **exact evidence** that will prove each scenario. A compound requirement is two
> requirements — split it. See SKILL implementation reference for the EARS patterns and the evidence-type catalogue.

**Scope:** the public landing page (/, Landing implementation reference). Two issues drive this spec:
- **#161 / INT-178** — homepage stats band read as pre-traction; the isolated 0 pattern-matched to "broken".
- **#166 / INT-183** — no testimonials / social proof on the landing page.

**Honesty constraint (binding, from business-analyst + Boss):** no invented numbers, no fabricated quotes, no
synthetic attribution. Every social-proof element must trace to a real row in the system (a real M11 review) or
to founder-supplied, consented content. A claim with no real source is rejected at sign-off.

**Founder-content gate (binding):** the testimonial BUILD (REQ-004) is **BLOCKED** until the founder supplies the
content/consent in REQ-007. The design, the data-source wiring spec, and the empty-state are *not* blocked and can
be specified/approved now.

---

## REQ-001 — VERIFY the #161 stats-band reframe is live and correct, then close #161

- **Type:** user interface / verification
- **Priority:** Must
- **Source:** GitHub issue #161 (INT-178); Landing implementation reference Stat Item + Privacy Glyph (currently ~lines 59-89, 386-400)
- **Rationale / value:** The bare 0 pattern-matched to "no data / broken" before the label was read
  (pre-attentive misread). The fix reframes it to a positive privacy *guarantee*. We close the issue only on
  evidence the reframe is real in code AND rendered on the live site — not on a claim.
- **Owner (specialist):** uiux-designer (verify) → issue-engineer (close with evidence)
- **Acceptance criteria (Gherkin):**


- Scenario: Stats band shows a privacy guarantee, not a bare zero.
- Given the live landing page at http://[REDACTED IP ADDRESS]/.
- When I view the four-item stats band.
- And no stat item renders an isolated numeric "0".
- Scenario: The reframe is the real source, not a stale build.
- Given Landing implementation reference in intuition-frontend.
- When I read the Stat Item for the privacy stat.
- Then it passes Privacy Glyph (an accessible svg) as the value, not the string "0".


- **Required evidence (defined up front):**
  - Playwright @1280 + @390 full-page screenshots of the live landing page showing the stats band
    (evidence/REQ-001/landing-1280-fullpage png, landing-390-fullpage png — CAPTURED 2026-06-20).
  - Live accessibility snapshot of the labelled statistics section showing img Always private + label
    "Contact info — always private" and NO bare-0 node (captured 2026-06-20).
  - Code reference: Landing implementation reference Privacy Glyph + a privacy symbol in place of an adoption count (verified 2026-06-20).
- **Dependencies:** none. **Status note:** the reframe is verified FIXED-DEV and LIVE (2026-06-20); this REQ's
  outcome is to *close #161* with the captured evidence.

---

## REQ-002 — Real adoption/volume metric stays DEFERRED (out of scope for now)

- **Type:** constraint (scope marker)
- **Priority:** Won't (this spec)
- **Source:** #161 (INT-178) "add one honest volume/outcome metric from the database once non-trivial"
- **Rationale / value:** A real adoption number (families served, matches made, average rating) would be the
  strongest trust signal — but it requires either a founder-supplied figure or a live database aggregate that is
  *non-trivial* (a count of 3 reads worse than no count). Shipping a fabricated or embarrassingly-small number
  violates the honesty constraint. We mark this DEFERRED explicitly so it is not silently dropped.
- **Owner (specialist):** business-analyst (define the metric + the "non-trivial" threshold) + data-engineer
  (confirm the source aggregate exists) — **deferred to a future spec.**
- **Acceptance criteria (Gherkin):**


- Scenario: No fabricated adoption metric ships in this spec.
- Given the landing stats band.
- When this spec is delivered.
- Then no stat displays a hard adoption/volume number unless it is founder-supplied or a real database aggregate above the agreed threshold.


- **Required evidence (defined up front):** N/A this spec — recorded here as an explicit OUT-OF-SCOPE-FOR-NOW
  marker. Re-enters scope when the founder supplies a number OR data-engineer confirms a non-trivial live aggregate.
- **Dependencies:** REQ-007 (founder content) is the unblock path if a founder number is chosen.

---

## REQ-003 — Testimonial strip DESIGN (placement, responsive, a11y) on the landing page

- **Type:** user interface / design
- **Priority:** Should
- **Source:** #166 (INT-183); Landing implementation reference (no testimonial section exists today)
- **Rationale / value:** Every leading SG agency runs named testimonials with outcomes; the product has an M11
  review system but none surface on the homepage. A testimonial strip about the interview-first / video-call
  differentiator is the highest-leverage trust signal we can add honestly. This REQ owns the *design* — the
  build is REQ-004 (gated). Designing it now lets the team approve placement/markup before content lands.
- **Owner (specialist):** uiux-designer (lead) + frontend-expert (CWV/markup feasibility)
- **Acceptance criteria (Gherkin):**


- Scenario: Strip is placed where it reinforces the decision, not buried.
- Given the landing page section order (hero  leads to  jobs strip  leads to  tutors strip  leads to  USP spotlight  leads to  how-it-works parents  leads to  how-it-works tutors  leads to  stats).
- When the testimonial strip is designed.
- Then it sits AFTER the parent "how-it-works" section and BEFORE the closing stats band (the parent-decision moment).
- And it uses one cream/light surface so it does not stack two dark anchors back-to-back.
- Scenario: Quotes use correct semantics.
- Given a rendered testimonial card.
- Then each quote uses a semantic quotation element with an associated attribution.
- And the strip is a section whose accessible name comes from its heading.
- Scenario: Responsive at the two reference widths.
- Given the strip design.
- Then at 1280px it is a 2-3-up row (static, no auto-advancing carousel).
- And at 390px it is single-column, no horizontal overflow, tap targets  is at least  44px, no font < 14px.


- **Required evidence (defined up front):** design spec section in design md (markup + token + responsive
  notes); on build, Playwright @1280 + @390 of the rendered strip; axe run = 0 serious/critical on the strip.
- **Dependencies:** REQ-005 (data source), REQ-006 (a11y), REQ-004 (build, gated).

---

## REQ-004 — Testimonial strip BUILD — **BLOCKED on founder content (REQ-007)**

- **Type:** functional
- **Priority:** Should
- **Source:** #166 (INT-183)
- **Rationale / value:** Renders the approved design with REAL content. **This task does not start until REQ-007
  delivers founder content/consent.** We refuse to ship placeholder or fabricated quotes (honesty constraint).
- **Owner (specialist):** frontend-expert (frontend strip) + bff-specialist/software-engineer (the featured-review
  source per REQ-005) — **dispatched only after REQ-007 is satisfied.**
- **Acceptance criteria (Gherkin):**


- Scenario: Build proceeds only with real, consented content.
- Given REQ-007 has delivered founder-approved real testimonials (real M11 reviews curated, or founder quotes with consent).
- When the strip is built.
- Then every rendered quote, rating, and attribution traces to a real source row or a consented founder quote.
- And no quote, name, rating, or outcome figure is fabricated or hard-coded as a placeholder.
- Scenario: Frontend coverage gate holds.
- Given the new testimonial component using s.
- When run the corresponding frontend quality check runs.
- Then it passes with  is at least  97.5% line and  is at least  88% branch coverage.


- **Required evidence (defined up front):** the documented operational check log meeting the gate; the documented operational check (tsc) clean;
  Playwright @1280 + @390 of the live strip; the content-source mapping (which real reviews/quotes were used) and
  the founder consent record from REQ-007.
- **Dependencies:** **REQ-007 (HARD BLOCKER)**, REQ-003, REQ-005, REQ-006.

---

## REQ-005 — The strip uses traceable real M11 reviews or consented founder quotes

- **Type:** data / contract
- **Priority:** Must (for any build)
- **Source:** #166 ("testimonial strip pulling real M11 reviews"); M11 review system
- **Rationale / value:** There must be a single, honest source of truth for what the strip shows. The existing
  public review endpoint is **per-tutor-post only** (read operation for tutors tutor Post Id reviews) — there is
  **no global / featured-reviews endpoint** and Review Response has **no featured / consent / moderation flag**.
  So "pull real M11 reviews" is not free: it needs either (a) a small new admin-curated featured-reviews read
  endpoint, or (b) a build-time curated list of real review IDs the founder approved. This REQ forces that
  decision to be made explicitly rather than papering over it with hard-coded prose.
- **Owner (specialist):** bff-specialist (contract) + dba (where the data + any featured flag lives) +
  data-engineer (confirm the reviews exist and the field is real)
- **Acceptance criteria (Gherkin):**


- Scenario: Source is real and traceable.
- Given the testimonial strip needs N quotes.
- When the source is defined in design md.
- Then M11_REVIEW items resolve to an existing review row through the selected curated source.
- And FOUNDER QUOTE items resolve to an actual supplied quote and a recorded permission for homepage publication.
- And a founder quote without a real supplied rating has no stars or numerical rating.
- Scenario: No fabrication path exists.
- Given the chosen source.
- Then there is no code path that renders a quote string not backed by a real review row or a consented founder quote.


- **Required evidence (defined up front):** if (a): the new endpoint's contract + a captured 200 response showing
  real review rows (Postman/curl log); if (b): the approved review-ID list + a captured fetch of each via the
  existing public endpoint. Either way: the mapping from source  leads to  rendered card.
- **Dependencies:** REQ-007 (founder picks which reviews / approves featuring), REQ-008 (consent for attribution).

---

## REQ-006 — Testimonial strip meets a11y + CWV budget (no LCP-tanking carousel)

- **Type:** a11y / non-functional requirement
- **Priority:** Must (for any build)
- **Source:** frontend-expert input; project CWV + WCAG 2.2 AA floor
- **Rationale / value:** A heavy auto-advancing JS carousel hurts LCP/CLS and is an a11y trap. The strip must add
  trust without a performance or accessibility regression on the highest-traffic public page.
- **Owner (specialist):** frontend-expert (CWV) + uiux-designer (a11y semantics)
- **Acceptance criteria (Gherkin):**


- Scenario: No heavy carousel.
- Given the strip implementation.
- Then it is a static grid (or a CSS-only scroll-snap rail), with NO auto-advancing JS carousel library.
- And it adds no new runtime dependency for a carousel.
- Scenario: Accessible quotes and motion.
- Given the rendered strip.
- Then quotes have semantic quotation and attribution elements, the section is labelled, contrast is  is at least  WCAG AA 4.5:1.
- And any motion respects prefers-reduced-motion with a static fallback.
- And it is fully keyboard operable with a visible focus indicator.
- Scenario: No CWV regression in the populated (async-fetch) case.
- Given the landing page with the strip rendering one or more approved testimonials fetched async.
- Then the strip below the fold is not render-blocking and reserves its footprint so the async fetch does not shift layout (CLS contribution ~0).
- Scenario: Zero approved items does not reserve a box (REQ-009 wins).
- Given the testimonial source returns zero approved items.
- Then the section is ABSENT — no reserved footprint, no empty container, no layout gap (this is REQ-009, not a CLS-reserve case).


- **Reserve-vs-absent reconciliation (REQ-006 ↔ REQ-009):** these two requirements do not conflict — they govern
  different states. The CLS footprint-reserve in this REQ applies **only to the populated case**, where the section
  IS rendered and its content arrives via an async fetch (reserve the box so the late content does not shift the
  page). When there are **zero approved items, REQ-009 wins**: the section is conditionally ABSENT — nothing is
  rendered and **no box is reserved** (a reserved-but-empty container would itself be a blank void REQ-009 forbids).
  Restated: reserve footprint when populating; render nothing (and reserve nothing) when empty.
- **Required evidence (defined up front):** axe run (0 serious/critical) on the strip; Playwright reduced-motion
  capture; confirmation the strip introduces no carousel dependency (package json diff = none); a **CLS check note
  covering BOTH states** — (1) populated async-fetch: measured CLS ~0 with the footprint reserved; (2) zero-item:
  section absent, so it contributes no layout shift at all.
- **Dependencies:** REQ-003, REQ-004, REQ-009.

---

## REQ-007 — **BLOCKER:** founder supplies real testimonial content OR approval to curate real M11 reviews

- **Type:** constraint (content blocker)
- **Priority:** Must (gates REQ-004)
- **Source:** #166 labelled "needs founder content"; honesty constraint
- **Rationale / value:** We will not fabricate quotes. The build cannot honestly proceed until the founder provides
  ONE of: (1) real parent testimonials (verbatim quote + how to attribute) with consent to publish, OR (2) written
  approval to surface specific real M11 reviews (by review ID or tutor) on the homepage. Until then REQ-004 stays
  BLOCKED and the strip ships, if at all, only as a designed-and-approved empty/coming-soon state (REQ-009).
- **Owner (specialist):** business-analyst (collect + record) <- founder
- **Acceptance criteria (Gherkin):**


- Scenario: Content is supplied and recorded before build.
- Given the founder provides testimonial content or curation approval.
- When it is recorded under evidence/REQ-007/.
- Then each item is either a consented real quote (REQ-008) or an approved real M11 review id.
- And REQ-004 is unblocked.
- Scenario: No content  leads to  no quotes ship.
- Given the founder has not supplied content.
- Then the strip does not render any quote (REQ-009 empty/deferred state applies).


- **Required evidence (defined up front):** evidence/REQ-007/ containing the founder-supplied quotes or the
  approved review-ID list, with the consent note (REQ-008). **No build evidence is accepted before this.**
- **Dependencies:** blocks REQ-004, REQ-005, REQ-008.

---

## REQ-008 — Attribution respects consent and the personal information model

- **Type:** security / privacy
- **Priority:** Must (for any build)
- **Source:** project personal information visibility model; M11 reviewer Display Name
- **Rationale / value:** M11 reviewer Display Name is a profile display name, not raw personal information — but publishing it as a
  *marketing testimonial* on the public homepage is a different, more prominent use than showing it on a tutor
  detail page. That requires explicit consent and an attribution style the founder approves (e g. "Sarah T.,
  parent" vs first-name-only vs initials). No tutor's or parent's contact details ever appear.
- **Owner (specialist):** security-expert (review) + business-analyst (consent capture)
- **Acceptance criteria (Gherkin):**


- Scenario: Attribution is consented and personal information-safe.
- Given a testimonial about to be published on the homepage.
- Then its attribution style is founder-approved and consented.
- And it exposes no contact info, no exact full name unless explicitly consented, and no other personal information.
- Scenario: Anonymous fallback exists.
- Given a real review whose author has not consented to named attribution.
- Then it may be shown as "Verified parent" / initials only, never with an unconsented full name.


- **Required evidence (defined up front):** the consent record in evidence/REQ-007/; security-expert sign-off
  note that the chosen attribution exposes no personal information.
- **Dependencies:** REQ-007.

---

## REQ-009 — Designed empty / "coming-soon" state while content is pending (no blank void)

- **Type:** user interface
- **Priority:** Should
- **Source:** uiux-designer non-negotiable (empty state is the first impression); blocker REQ-007
- **Rationale / value:** If the strip section ships its container before content lands (or review volume is thin),
  it must not be a blank gap. Per the four-states rule, the empty state either (a) renders nothing at all (the
  section is conditionally absent — preferred until content exists) or (b) shows an honest, non-fabricated
  placeholder ("Reviews from real families, coming soon") — never a fake quote.
- **Owner (specialist):** uiux-designer
- **Acceptance criteria (Gherkin):**


- Scenario: No fabricated filler.
- Given the testimonial source returns zero approved items.
- Then the strip section either renders nothing (no empty container, no layout gap) or an honest "coming soon" note.
- And under no circumstance does it render a placeholder/fake quote.


- **Required evidence (defined up front):** design md empty-state spec; on build, a Playwright capture of the
  zero-content state (section absent or honest note), @1280 + @390.
- **Dependencies:** REQ-003, REQ-007.

## 2026-09-05 source/attribution clarification

REQ-005 covers both source classes already permitted by REQ-004: genuine M11 rows and actual consented founder quotes. The public output is a sanitized Testimonial Item, with an independently approved display Attribution; raw review display names and private consent records are never blindly exposed. Publication consent is required even for anonymous attribution. A quote without a source rating omits stars. REQ-007 founder content and approvals remain pending; see [open decisions](../CURRENT-REQUIREMENTS.md).

The historical “Always private” adoption-band evidence is not authority for a never-share-contact promise: current privacy copy must reflect baseline C04 assigned-participant disclosure (frontend ACTIVE/COMPLETED; match-PENDING hidden).
