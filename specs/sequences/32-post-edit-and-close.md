# Sequence 32 — Post lifecycle: edit + close

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Scope edits to the owner/admin action and the canonical post state. PENDING owner edits are allowed by021; ACTIVE edits requeue to PENDING; committed match states remain restricted.
2. Distinguish logical removal of a draft/rejected post from closing a live ACTIVE/PENDING job.
3. Live close preserves the documented reason and cascades to open applications/invitations/interviews without inventing a timeout or physically deleting records.

## Authority

[Current requirements C06](../CURRENT-REQUIREMENTS.md) · [spec md](../021-tutee-request-flow/spec.md) · [int-062-cascade-close-spec md](../architecture/int-062-cascade-close-spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# Sequence 32 — Post lifecycle: edit + close

**Module:** M2 Posting · **Actors:** Tutee (job post owner), Tutor (listing owner)
**Endpoints:** replace operation for tutee,tutor posts (own active),
replace operation for tutee,tutor posts id, delete operation for tutee,tutor posts id

Covers what happens to a post *after* creation (seq 06/07): the owner edits it or closes it.

## Preconditions
- Owner is authenticated and owns the post.

## Flow — edit


Implementation example omitted. This public edition retains the written requirements in this section.


## Flow — delete (soft-delete → CLOSED; owner-initiated, restricted by status)


Implementation example omitted. This public edition retains the written requirements in this section.


## Postconditions
- Edit: fields updated; status unchanged (a PENDING post stays PENDING; re-review rules
  per module spec apply if a material field changes).
- Delete: status is CLOSED (soft delete, not a row delete — founder soft-delete-only rule) — post
  leaves browse/queues. Only a DRAFT or REJECTED post can be owner-deleted; an ACTIVE/MATCHED post
  is NOT owner-deletable (protects live matches) and returns 409 INVALID STATE.
- **2026-06-03 (INT-015 — FIXED):** deleting a **REJECTED** post previously **500'd** — the soft-delete
  sets CLOSED, but the post-status-guard trigger (changeset 0005) forbade ANY change out of a terminal
  state (REJECTED is terminal). **Migration 0031** (allow-post-soft-delete-close) relaxes the trigger to
  permit the close transition (REJECTED/EXPIRED→CLOSED) while still forbidding all other terminal changes,
  so DRAFT and REJECTED deletes now behave identically.

## Error cases
- 403 not owner / wrong role; 404 post not found.
- 409 INVALID STATE — "Only DRAFT or REJECTED posts can be deleted" (an ACTIVE/PENDING/
  MATCHED post cannot be owner-deleted).
- Post Status: DRAFT · PENDING · ACTIVE · REVISION REQUESTED · REJECTED · MATCHED · CLOSED · EXPIRED.

## Related
- Creation: [06-tutee-post-creation md](06-tutee-post-creation.md),
  [07-tutor-post-creation md](07-tutor-post-creation.md)
- Module: [Private supporting material excluded (publication note)](../../docs/publication-notes.md)
