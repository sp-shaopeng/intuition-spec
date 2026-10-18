# 23 — Public Browse and Limited Anonymous Filters

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Render supported filters for both anonymous and signed-in visitors on tutor and assignment lists. Filter changes reset pagination and refetch using the filters and authentication state in the query key.
2. Apply server-side anonymous bounds: pages0–4, size capped at20; page>4 returns an empty page. A larger valid size is clamped to20, not treated as an empty first page. These are per-query result bounds.
3. Return only the approved personal information-free public projection and public handle. Detail/action login is separate and preserves return To. Rate limits do not create a lifetime search quota.

## Authority

[Current requirements C01](../CURRENT-REQUIREMENTS.md) · [m17-marketplace-ux-overhaul md](../architecture/m17-marketplace-ux-overhaul.md) · [int-125-batch-closeout-spec md](../architecture/int-125-batch-closeout-spec.md)

## Contract diagram


Implementation example omitted. This public edition retains the written requirements in this section.


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 23 — Public Browse Tutors with Filters

## Trigger

Anyone (logged in or anonymous) navigates to /tutors. **INT-012:** the filter bar renders
**only for logged-in users**; anonymous visitors get a capped teaser sample (no filter user interface). Each
filter change is a React Query refetch — the query key includes every active filter **and the
authed flag** (so the list refetches at the larger page size the moment the user logs in).

## Participants

- **Visitor**
- **frontend Browse Tutors implementation reference**
- **Public Browse Controller** — read operation for tutors
- **Tutor Post Service find Public Posts Filtered**
- **Tutor Post Repository** (find Active Posts Filtered), **Review Service**,
  **Tutor Profile Repository**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- frontend: Browse Tutors implementation reference. Page is hard-coded to 0 and size to 50; no user interface
  paginator. params Serializer REPEAT ARRAY PARAMS (line 17) enforces
  ?venue is A&venue is B form.
- Filters and their frontend state: Browse Tutors implementation reference. Active filter detection
  drives the "Clear all filters" button visibility (lines 45-52).
- Controller: Public Browse Controller implementation reference. Spring binds venue to
  a list of venue types enum, region and group Size to enum.
- Service: Tutor Post Service find Public Posts Filtered — Tutor Post Service implementation reference. Page size
  is capped at 50 server-side (the min operation).
- load Ratings is called once per page (single batched aggregate query). Tutor
  display name lookup is **not** batched here — the to response operation does an
  individual tutor Profile Repository find By User Id per post (line 305-309).
  This is an N+1 candidate that hasn't been fixed yet (compare to admin queue
  which IS batched in flow 08).
- Endpoint is the permit all operation (Security Config lines 129-130). The cross-site request forgery protection ignore set
  covers ** interface operation too (line 108) but GET is not state-changing so
  this is incidental.
- **INT-012 — pre-login gating + anonymous cap.**
  - frontend (Browse Tutors implementation reference): the Marketplace Filter Bar renders only when user is set; the request
    size is AUTHED PAGE SIZE is 50 for logged-in users and ANON PAGE SIZE is 12 for anonymous; the
    query key includes the yes-or-no value operation so logging in refetches at the larger size.
    Browse Assignments implementation reference gates its filter bar the same way (added use Auth Store).
  - backend (Public Browse Controller browse Tutors): takes @Authentication Principal Authenticated User
    For an anonymous visitor, no signed-in user is attached to the request. The following limits then apply:
    permit All route) AND the max anonymous page operation or the max anonymous size operation, it returns
    an empty page (catalogue is not anonymously enumerable). Authenticated callers of any role bypass
    the cap and page freely. Mirrors the existing /public/assignments cap, but **principal-gated**
    (the assignments cap is unconditional) so it never throttles authed tutors.

## Failure modes

- 400 — invalid enum value for region, group Size, or any element of venue.
  Spring deserialisation will fail before the controller method body runs.
- No 401/403 — endpoint is open.
- Repository exceptions surface as 500.

## Related flows

- [20-public-tutor-detail-reviews md](20-public-tutor-detail-reviews.md) — click-through from a card
- [22-favourite-toggle md](22-favourite-toggle.md) — the heart state companion
- [19-tutee-submits-review md](19-tutee-submits-review.md) — how aggregate ratings get into the listing
