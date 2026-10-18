# 36 — Marketplace Detail (login-gated tutor / assignment detail)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Trigger

A logged-in user opens a tutor listing (/tutors/:id) or a tuition assignment
(/assignments/:id). M17 SEC-1 moved full detail behind auth — the browse **lists**
stay public (flow 23), but **detail requires login**. The old anonymous
tutors id interface operation detail endpoint was removed; this is its authenticated
replacement plus the new symmetric assignment-detail endpoint.

## Participants

- **User** (any authenticated role)
- **frontend** Tutor Detail implementation reference / Assignment Detail implementation reference
- **Marketplace Detail Controller** — read operation for tutors id, read operation for assignments id
  (the pre authorize operation)
- **Tutor Post Service find Active Post By Id** / **Tutee Post Service find Active Post By Id**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Marketplace Detail Controller implementation reference — both methods the pre authorize operation;
  class the request mapping operation.
- 404 (not 403) for non-existent **or non-ACTIVE** posts — prevents enumeration.
- The tutor detail response is the same personal information-free Tutor Post Response as the browse list
  (handle only, no real name — INT-003). The frontend avatar initials derive from
  tutor Public Handle (INT-009; see flow 20).

## Failure modes

- 401 — anonymous caller (the M17 gate). frontend redirects to login.
- 404 — id not found or post not ACTIVE.

## Related flows

- [23-public-browse-with-filters md](23-public-browse-with-filters.md) — the public LIST the user clicks from
- [20-public-tutor-detail-reviews md](20-public-tutor-detail-reviews.md) — the reviews fetch on the tutor detail page
- [13-tutee-request-interview md](13-tutee-request-interview.md) — the CTA on the tutor detail page
