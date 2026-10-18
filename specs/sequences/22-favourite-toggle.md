# 22 — Favourite Toggle + Bulk Heart State

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticated parents may favorite eligible tutor posts. Reads and membership queries exclude logically removed rows.
2. Unfavorite marks the retained favorite row removed; it does not physically delete it. Re-favorite reactivates the existing pair without duplicate rows.
3. Keep optimistic user interface rollback on failure, idempotent success and server owner scoping.

## Authority

[Current requirements C13](../CURRENT-REQUIREMENTS.md) · [int-094-batch-soft-delete-integrity-spec md](../architecture/int-094-batch-soft-delete-integrity-spec.md)

## Contract diagram


Implementation example omitted. This public edition retains the written requirements in this section.


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 22 — Favourite Toggle + Bulk Heart State

## Trigger

Tutee clicks the heart on a Tutor Card. Browse page also bulk-loads the
"which of these are favourited?" set on render.

## Participants

- **Tutee**
- **frontend Tutor Card implementation reference** (button + optimistic update)
- **frontend Browse Tutors implementation reference** (bulk membership probe)
- **Favourite Controller** — submit operation for tutee favourites id, delete operation for id,
  read operation for ids?post Ids is ...
- **Favourite Service**
- **Favourite Tutor Post Repository, Tutor Post Repository, User Repository**

## Diagram — bulk membership on browse mount


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — toggle (optimistic + reconcile)


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- frontend bulk fetch: Browse Tutors implementation reference. query parameters that repeat a field name for multiple selected values
  emits repeated post Ids= rather than post Ids — Spring's a list of unique identifier entries
  binding only accepts the repeated form.
- frontend toggle: Tutor Card implementation reference. Optimistic update covers **every** cached
  ids Among query (so the heart flips on Browse, Favourites, etc.). Snapshot
  is restored on error.
- Controller add: Favourite Controller implementation reference — returns 201 when a new
  row is inserted, 200 when the post was already favourited (so the caller can
  distinguish without breaking idempotency).
- Service add: Favourite Service implementation reference. Race-safe via outer existence
  check + catch on Data Integrity Violation Exception.
- Service remove: Favourite Service implementation reference — silently idempotent.
- Bulk-probe service: Favourite Service implementation reference — short-circuits to
  the of operation for empty input (no database hit).

## Failure modes

- 404 TUTOR POST NOT FOUND on add — only checked when row is being inserted
  (the idempotent return path skips it).
- 422 TUTOR POST NOT ACTIVE on add — same.
- 404 TUTEE NOT FOUND — user record gone but signed session token still valid (rare).
- 403 if non-tutee (class-level the pre authorize operation).

## Related flows

- [23-public-browse-with-filters md](23-public-browse-with-filters.md) — what
  populates tutor Post Ids
- [25-csrf-protected-write md](25-csrf-protected-write.md)
