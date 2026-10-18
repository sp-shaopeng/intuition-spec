# 18 — Tutor Views Match Dashboard

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate the tutor and scope matches through tutor User Id; a tutor listing is optional.
2. Show how much the tutor owes the agency and whether the agency has received that commission. PAID does not mean money owed to the tutor.
3. Keep humanized job labels, personal information-free list projections and the separately gated contact endpoint; review eligibility follows the matching/review contract.

## Authority

[Current requirements C02](../CURRENT-REQUIREMENTS.md) · [business requirements document md](../BRD.md) · [int-034-free-market-assign md](../architecture/int-034-free-market-assign.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 18 — Tutor Views Match Dashboard

## Trigger

Tutor navigates to Matches implementation reference.

## Participants

- **Tutor**
- **frontend Matches implementation reference**
- **Match Dashboard Controller** — read operation for tutor matches
- **Match Dashboard Service**
- **Match Repository, Commission Repository, Review Repository**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Match Dashboard Controller implementation reference.
- Service: Match Dashboard Service implementation reference.
- Unlike the tutee view, the tutor view exposes the **commission triple**
  (monthly Fee Sgd, commission Amount Sgd, commission Status) because the
  tutor needs to see how much they're owed and whether it has been paid.
- Tutor display name lookup is **not** done — the tutee's identity is the
  tuition subject/level summary, not a display name (privacy).
- **INT-137a:** the data transfer object carries tutee Subject/tutee Level as raw CODES + the real
  tutee Post Title; the frontend composes the card label via the subject label operation/the level label operation
  (shared Job Identity component). The backend no longer renders "S4_EXP AMATH OLEVEL".
- Same my Review Submitted mechanism as the tutee view.

## Failure modes

- 403 if not a TUTOR.
- 401 if signed session token missing/invalid.

## Related flows

- [17-tutee-views-matches md](17-tutee-views-matches.md) — symmetric
- [11-admin-mark-commission-paid md](11-admin-mark-commission-paid.md) — commission Status driver
- [19-tutee-submits-review md](19-tutee-submits-review.md)
