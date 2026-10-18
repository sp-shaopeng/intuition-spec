# 17 — Tutee Views Match Dashboard

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate the parent and scope all matches to that account. Batch enrichment retains optional-listing-safe tutor identity and canonical profile fallbacks.
2. Render public handle/humanized labels without assuming tutor Post exists. Keep administrative contact metadata distinct from counterparty contact reveal.
3. Counterparty contact requires participant authorization; frontend reveal is limited to ACTIVE/COMPLETED. Review actions retain their participant/completed-match guards.

## Authority

[Current requirements C05](../CURRENT-REQUIREMENTS.md) · [int-034-free-market-assign md](../architecture/int-034-free-market-assign.md) · [int-125-batch-closeout-spec md](../architecture/int-125-batch-closeout-spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 17 — Tutee Views Match Dashboard

## Trigger

Tutee navigates to Matches implementation reference.

## Participants

- **Tutee**
- **frontend Matches implementation reference**
- **Match Dashboard Controller** — read operation for tutee matches
- **Match Dashboard Service**
- **Match Repository, Commission Repository, Tutor Profile Repository, Review Repository**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Match Dashboard Controller implementation reference.
- Service: Match Dashboard Service implementation reference.
- ADMIN CONTACT EMAIL REDACTED email — only revealed when match is
  ACTIVE (line 96). PENDING / CANCELLED / COMPLETED matches see no value.
- Three batch lookups avoid N+1: commissions, tutor display names, reviewed
  match IDs (lines 65-83).
- my Review Submitted boolean drives the "Leave review" CTA in Matches implementation reference.

## Failure modes

- 403 if not a TUTEE.
- 401 if signed session token missing/invalid.

## Related flows

- [10-admin-match-activation md](10-admin-match-activation.md) — reveals admin email
- [18-tutor-views-matches md](18-tutor-views-matches.md) — symmetric tutor view
- [19-tutee-submits-review md](19-tutee-submits-review.md) — what my Review Submitted gates
