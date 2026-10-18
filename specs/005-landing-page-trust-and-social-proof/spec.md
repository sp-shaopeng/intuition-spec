> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 005
slug: landing-page-trust-and-social-proof
title: Landing page trust and social proof
version: 1.3
status: APPROVED            # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: uiux-designer         # the engineer who owns this change (requirements-engineer for cross-cutting/product specs)
created: 2026-06-20
updated: 2026-09-05
---

> **2026-09-05 requirements correction (documentation only).** Both already-authorized testimonial sources now have a consented public attribution contract. Actual founder content/approval remains pending and the strip build remains gated.
> The status and shipment/evidence entries below record the earlier implementation. This revision does not
> certify that the corrected contract is implemented, tested, or deployed; follow-up verification remains open.


# Landing page trust and social proof

Strengthens the trust/social-proof layer of the public landing page (/). Two drivers: (1) **#161 / INT-178** —
the homepage stats band read as pre-traction and the isolated 0 pattern-matched to "broken"; this is already
reframed to a positive "Always private" privacy guarantee (verified live 2026-06-20) and this spec VERIFIES and
CLOSES it. (2) **#166 / INT-183** — there is no testimonial / social-proof section on the landing page despite an
M11 review system; this spec fully DESIGNS an honest testimonial strip (placement, responsive @1280/@390, a11y,
and how it pulls REAL M11 reviews — no global/featured endpoint exists today, so a source must be chosen) and
gates the BUILD on founder-supplied content/consent. Binding throughout: **no invented numbers or quotes** — every
social-proof element traces to a real review row or consented founder content. A real adoption/volume metric stays
explicitly DEFERRED.

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC (each REQ-NNN)
- **Design:** [design md](design.md) — architecture, data model, error handling, testing, migration
- **Tasks:** [tasks md](tasks.md) — implementation tasks, each traced to a REQ-NNN + owning specialist
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario

## Status

APPROVED for the **deferred-build posture** (v1.1, 2026-06-20): the #161 verify+close lane (REQ-001/002) and the
testimonial DESIGN + source decision (REQ-003/005/006/009) are cleared; the testimonial BUILD (REQ-004/007/008)
stays BLOCKED on founder-supplied content/consent and does not start until that lands.

## Review & Sign-Off

- [x] Counterpart — user interface/user experience (uiux-designer, LEAD) — SIGN-OFF / 2026-06-20
- [x] Counterpart — frontend (frontend-expert, impl/CWV) — SIGN-OFF (CLS reconcile v1.1) / 2026-06-20
- [x] Counterpart — business-analyst (metric/quote honesty) — SIGN-OFF (no fabrication) / 2026-06-20
- [x] Counterpart — data (data-engineer, where a real metric/review comes from) — SIGN-OFF (Review Response contract verified) / 2026-06-20
- [ ] Counterpart — backend (bff-specialist, source endpoint contract — only if Option A) — deferred until source decision at content-unblock
- [x] Technical Writer (docs) — SIGN-OFF / 2026-06-20
- [x] Boss (Rex) — APPROVED for deferred-build posture / 2026-06-20

## Changelog

- 1.2 (2026-06-21) MINOR — VERIFIED + CLOSED REQ-001 (#161 / INT-178 — landing stats-band reframe): verified live
  on the virtual machine and **GitHub issue #161 closed** (2026-06-21). The testimonial BUILD (#166 / INT-183 — REQ-004/007/008)
  remains **BLOCKED on founder-supplied content/consent** per D6, so the spec stays APPROVED (deferred-build) — NOT
  flipped to HISTORICALLY MARKED IMPLEMENTED.
- 1.1 (2026-06-20) MINOR — spec-review doc fixes (scoped review verdict: READY FOR APPROVED, two doc fixes).
  MODIFIED REQ-006 + design md error-handling/testing — reconciled the REQ-006 CLS footprint-reserve against the
  REQ-009 absent-section rule: stated REQ-009 wins on zero approved items (section ABSENT, NO reserved box), the
  CLS-reserve applies only to the populated async-fetch case; added a Playwright CLS note covering BOTH states.
  REVIEWED the design md Review Response data-contract claim against source
  (Review Response implementation reference): the record DOES carry id and match Id (alongside
  rating, comment, reviewer Display Name, created At) and the public per-tutor endpoint
  (read operation for tutors tutor Post Id reviews) returns the full Review Response — so the field list is
  CORRECT as written and Option B's id-bearing read path is valid. No data transfer object-drift correction was needed (the
  reported drift did not exist in the current source).
- 1.0 (2026-06-20) initial draft — ADDED REQ-001..009. Covers #161 verify+close (REQ-001), adoption-metric
  DEFERRED (REQ-002), testimonial strip design (REQ-003), gated build (REQ-004), real-M11-review source
  decision (REQ-005), a11y/CWV no-carousel (REQ-006), founder-content BLOCKER (REQ-007), consent/personal information attribution
  (REQ-008), empty-state (REQ-009). Sign-off rows scoped to uiux-designer (lead) + frontend-expert +
  business-analyst + data-engineer + bff-specialist (Option A only) + Technical Writer + Boss.
