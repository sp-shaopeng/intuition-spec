# Design — Landing page trust and social proof (005)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> How the requirements in [requirements md](requirements.md) will be built. Owned with solution-architect
> for non-trivial design; record significant choices as ADRs (Context / Decision / Consequences).

## Architecture

Two independent workstreams on the public landing page (Landing implementation reference):

1. **#161 reframe (REQ-001/002) — already implemented, this spec VERIFIES + closes.**
   The stats band (Landing implementation reference Stat Item + Privacy Glyph, ~lines 59-89 and 386-400) already replaces the
   bare 0 with an accessible shield glyph (announced to assistive technology as “Always private”) and the label
   "Contact info — always private". No code change is owed here — only verification evidence and issue closure.
   A real adoption metric (REQ-002) is explicitly DEFERRED.

2. **Testimonial strip (REQ-003..009) — DESIGN now, BUILD gated on founder content (REQ-007).**
   A new testimonial strip section component, lazy-loaded the same way the existing showcase strips are
   (Active Tutors Strip, Recent Requests Strip are the lazy operation + a reserved loading state with a footprint-reserving fallback —
   Landing implementation reference lines 12-13, 17-30). The strip reuses Star Rating (Star Rating implementation reference) and the
   visual language of the existing Review Card (Review Card implementation reference) but with **semantic
   semantic quotation and attribution elements** for a marketing context.

   **Placement:** inserted between the parents "How It Works" section (Landing implementation reference ~line 258) and the closing
   stats band (~line 388) — the parent-decision moment, on a cream/light surface so the page does not stack two
   dark anchors (USP spotlight … stats) back-to-back. (REQ-003.)

### Component list (new)
- Testimonial Strip implementation reference — the section (heading + grid of cards). Lazy-loaded from
  Landing implementation reference. Conditionally renders nothing when it has zero approved items (REQ-009).
- Testimonial Card implementation reference — one quotation card: optional Star Rating (only for a real supplied rating) + quote text + an attribution element
  attribution. No personal information; attribution per REQ-008.
- (If source option A is chosen) backend: a public featured-reviews read endpoint + repository query (REQ-005).

### Reused existing
- Star Rating, Review Card visual recipe, Skeleton Card, the lazy operation+Suspense+Showcase Fallback pattern,
  application programming interface ROUTES reviews.* (api Routes implementation reference lines 170-175), the Review type (index implementation reference).

## Data model

Review Response (backend Review Response implementation reference) today: id, match Id, rating, comment,
reviewer Display Name, created At. **There is NOfeatured, consent, or moderation flag, and NO global review
listing** — the only public review endpoint is per-tutor-post
(read operation for tutors tutor Post Id reviews, Review Controller PUBLIC BASE, opened in
Security Config via tutors * reviews interface operation permit All).

### Source and public projection (REQ-005/007/008)

Every approved item is a Testimonial Item with source Type (M11 REVIEW or FOUNDER QUOTE), source Id, quote, display Attribution, and optional rating. The internal curation record also retains the consent/publication approval reference; that private record and raw author identity are not part of the public projection.

- M11 REVIEW: resolves to a genuine M11 row, curated either through an admin featured endpoint or a founder-approved review-ID list. The storage/transport selection remains pending implementation design sign-off; it does not exclude the founder-quote source permitted by REQ-004/005. Publishing an M11 review on a tutor detail page does not itself establish homepage marketing consent.
- FOUNDER QUOTE: resolves to actual founder-supplied content with recorded permission for homepage publication and the approved attribution. No invented quote, name, rating, outcome, or placeholder. Render stars only when the source has a real, consented rating; otherwise render the quote without a rating.
- display Attribution is separately approved for homepage display (for example, consented initials or “Verified parent”). Never bind Review Response reviewer Display Name directly to the card. A full name requires explicit permission. Missing publication consent excludes the item; lack of named-attribution consent may use the agreed anonymous label only where publication itself is consented.

### application programming interface / contracts

The strip consumes only the sanitized Testimonial Item projection, regardless of source storage. An optional featured endpoint returns this projection, not raw Review Response; any writes remain admin-only. The existing per-tutor review response can be used internally for curated M11 lookup but must be mapped through the approval/attribution gate before public rendering. Paths use shared constants. Empty or unavailable approved content follows REQ-009.

**Still pending:** the actual quotes/review IDs, consent records and attribution approvals in REQ-007, and the curated source transport implementation choice. See [current open decisions](../CURRENT-REQUIREMENTS.md). This correction authorizes no content and does not unblock the testimonial build.

## Error handling

- **Fetch failure / empty source:** the strip renders its REQ-009 state — preferably the section is simply absent
  (no layout gap), or an honest "Reviews from real families, coming soon" note. NEVER a fabricated quote, never a
  raw error to the user. Below-the-fold, so a failed fetch must not block the rest of the page.
- **Thin volume:** if fewer than the design's minimum (e g. < 2) approved items exist, fall back to REQ-009 rather
  than show a lone card that reads as weak.
- **Missing/blank comment:** a review with a null/empty comment is skipped for the testimonial strip (a star
  with no words is not a testimonial), mirroring Review Card's review comment and guard.
- **CLS footprint-reserve vs absent section (REQ-006 ↔ REQ-009):** these do not conflict; they cover different
  states. **REQ-009 wins on zero approved items** — the section renders nothing and reserves NO box (a
  reserved-but-empty container would be the blank void REQ-009 forbids). The **REQ-006 CLS footprint-reserve
  applies only to the populated case**: when one or more approved testimonials exist and arrive via an async
  fetch, reserve the strip's footprint so the late content does not shift the page (CLS ~0). Rule: reserve when
  populating; render nothing (reserve nothing) when empty. The Playwright CLS evidence must cover both states —
  populated async-fetch (measured CLS ~0, footprint reserved) and zero-item (section absent, no layout shift).

## Testing strategy

- **REQ-001 (verify #161):** live evidence — Playwright @1280 + @390 of the stats band (CAPTURED 2026-06-20 under
  evidence/REQ-001/) + the live a11y snapshot showing img Always private and no bare 0 + a code-read of
  Privacy Glyph/Stat Item. No unit test owed (no code change). issue-engineer closes #161 with this bundle.
- **REQ-003/006/009 (strip frontend):** Vitest unit tests for Testimonial Strip + Testimonial Card covering: renders
  real items, skips comment-less items, renders the empty/absent state on zero items, blockquote/cite semantics,
  no carousel. Must keep the gate ≥ 97.5% line / ≥ 88% branch. axe run on the strip = 0 serious/critical.
- **REQ-005 (source):** Option A → an IT for the featured endpoint (Abstract Api IT, live stack) asserting only
  featured reviews return + public access; Postman/curl captured 200. Option B → captured fetch of each
  approved review ID returning a real row. Both source variants must be tested; a consented founder quote without rating renders no stars, missing publication consent is excluded, and raw reviewer Display Name never enters the public projection.
- **REQ-008 (consent/personal information):** security-expert review note; assert no contact field is in the rendered payload.

Each REQ's Gherkin maps to one of the above; build evidence lands under the private evidence directory for the requirement.

## Migration / rollout

- **#161:** nothing to migrate — already live; this spec only records evidence + closes the issue.
- **Strip:** behind the founder-content gate (REQ-007). Until content lands, the section ships as REQ-009
  (absent/coming-soon) or is not merged at all. Option A adds one Liquibase changeset (featured column) — applied
  via Testcontainers on the documented operational check, backward-compatible (defaults false). Rollback = drop the section render +
  (Option A) leave the nullable/default column. No data migration. Lazy-load keeps LCP unaffected.

## Decisions (ADR log)

- 2026-06-20 — **#161 is verified live, not re-built.** Context: the reframe is already in Landing implementation reference and
  rendered on http://[REDACTED IP ADDRESS]. Decision: this spec's REQ-001 is a verification + issue-close, not a code
  change. Consequence: fast close with hard evidence; no regression risk.
- 2026-06-20 — **Adoption metric DEFERRED (REQ-002).** Context: a real volume number is the strongest signal but
  needs a founder figure or a non-trivial live aggregate. Decision: out of scope for now, explicitly marked.
  Consequence: no fabricated/embarrassing number ships; re-opens when a real source exists.
- 2026-06-20 — **Testimonial source: Option A vs B — OPEN, decided at sign-off.** Context: no global/featured
  review endpoint exists today; Review Response has no featured flag. Decision: pick A (curated endpoint +
  Liquibase featured column, scales) or B (founder-approved real review-ID list, no migration, fastest) at the
  team-review. Consequence: these options govern M11_REVIEW transport only; consented FOUNDER QUOTE items remain allowed without invented ratings. The difference is curation mechanism + whether
  a migration is owed.
- 2026-06-20 — **No carousel (REQ-006).** Context: auto-advancing JS carousels tank LCP/CLS and are a11y traps.
  Decision: static grid (or CSS scroll-snap rail), no carousel dependency. Consequence: trust added with zero CWV
  regression on the highest-traffic page.
