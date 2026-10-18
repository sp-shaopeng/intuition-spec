# 20 — Tutor Detail + Reviews Fetch

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **M17 update (deployed):** tutor detail is no longer anonymous. The detail call moved
> from read operation for tutors id (removed) to the **authenticated**
> read operation for tutors id (the is authenticated operation → 401 for anonymous visitors, who are
> sent to login followed by a return to the selected tutor’s page). The Security Config tutors ** interface operation
> glob is narrowed to the LIST only, **except** read operation for tutors * reviews, which
> stays permit All so the reviews strip below is unchanged. The diagram/notes that read
> the specified tutor accessible to a public visitor are retained as the pre-M17 history of the detail call.

## Trigger

A **logged-in** visitor navigates to the specified tutor accessible to the user — i e. Tutor Detail implementation reference
mounts. An anonymous visitor hitting the specified tutor accessible to the user is bounced to
login followed by a return to the selected tutor’s page (server-side 401 on the detail call).

## Participants

- **Visitor**
- **frontend Tutor Detail implementation reference** — fires two parallel queries on mount
- **Tutor detail controller** — read operation for tutors id (**M17: authenticated**, the is authenticated operation; was read operation for tutors id permit All pre-M17)
- **Review Controller** — read operation for tutors tutor Post Id reviews (permit All — kept public under the M17 narrowing for M11 reviews)
- **Tutor Post Service find Active Post By Id**, **Review Service list Reviews About Tutor Post**
- **Tutor Post Repository, Review Repository, Tutor Profile Repository**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- frontend: Tutor Detail implementation reference issues both queries with enabled:
  yes-or-no value using id. They are independent and run in parallel.
- backend controller for detail: **M17 moved this to the login-gated
  Marketplace Detail Controller get Tutor Detail (Marketplace Detail Controller implementation reference,
  the is authenticated operation)** — the old anonymous Public Browse Controller detail endpoint was
  removed. The authenticated detail flow is **[36-marketplace-detail-login-gated md](36-marketplace-detail-login-gated.md)**;
  this flow now documents the (still public) **reviews** fetch.
- backend service for detail: Tutor Post Service find Active Post By Id — Tutor Post Service implementation reference.
  Returns **404** NOT FOUND if the post is missing OR not ACTIVE (INT-137b aligned
  status with the body code — was 400; still 404 for non-ACTIVE too, so enumeration of
  approved-and-then-closed listings is still prevented).
- backend controller for reviews: Review Controller implementation reference.
- backend service for reviews: Review Service list Reviews About Tutor Post line 156-173.
- **M17:** the detail endpoint read operation for tutors id is @Pre Authorize /
  the is authenticated operation (401 for anonymous). The reviews endpoint stays public:
  Security Config narrows the tutors ** interface operation glob to the LIST but keeps
  read operation for tutors * reviews permit All.
- Aggregate computation: see get Ratings For Reviewee Ids line 180-193 and the
  associated find Aggregate By Reviewee Ids repository projection.

## Failure modes

- 404 NOT FOUND from the detail endpoint when the post is non-existent or
  non-ACTIVE (INT-137b — was 400).
- 404 TUTOR POST NOT FOUND from the reviews endpoint (different code path —
  the reviews endpoint hits the repository explicitly to surface 404 cleanly).
- **M17:** the detail endpoint returns **401** to anonymous callers (login-gated); the
  frontend redirects to login followed by a return to the selected tutor’s page. The reviews endpoint remains public
  (no 401/403).

## Related flows

- [13-tutee-request-interview md](13-tutee-request-interview.md)
- [19-tutee-submits-review md](19-tutee-submits-review.md)
- [23-public-browse-with-filters md](23-public-browse-with-filters.md)
