# M6 Interview — Constraint: Max 3 interviews per parent posting

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

Three is a concurrent ACTIVE interview cap, not a lifetime limit. Include INT-039 PROPOSED/COUNTER PROPOSED/PENDING SCHEDULING states; TUTOR DECLINED/EXPIRED free capacity. [021](../../021-tutee-request-flow/spec.md) permits eligible PENDING owner posts as well as ACTIVE. The old lifetime-cap suggestion is removed; it is not an approved alternative.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** BUILT + DEPLOYED (migration 0021) — interviews now carry tutee post id; founder directive 2026-05-29.
**Driver:** "Add a constraint for parents: for any posting they can ask for a maximum of 3 interviews."

## Decision (founder-ratified)
"Posting" = the **parent's own job request** (tutee posts). A parent may request up to **3
interviews per posting** (i e. interview up to 3 tutors for each request they posted).

## Current model gap
Today an Interview links tutee id + tutor post id only — it is NOT associated with the
parent's posting. To enforce a per-posting cap, interviews must reference the parent's posting.

## Changes

### Data model — migration 0021
- Add interviews tutee post id unique identifier no value FK → the tutee posts operation ON DELETE RESTRICT.
  Nullable so existing interview rows (created before this rule) remain valid; new requests
  require it (see below).
- Add index the interviews operation
  to back the cap count cheaply.
  - **INT-077 (2026-06-06):** this predicate had **drifted** from the 5-status
    ACTIVE STATUSES the count actually uses, so the count was no longer index-covered.
    INT-077 widened the partial-index predicate to match ACTIVE STATUSES.
- Add the matching the many to one operation field to the Interview entity.

### Backend — request Interview
- Create Interview Request gains a **required** tutee Post Id (unique identifier, @Not no value).
- New guards (in addition to the existing tutor-post-active + duplicate guards):
  1. The tutee Post Id must exist, be owned by the requesting tutee, and be ACTIVE or PENDING (later 021 amendment)
     → else 404 TUTEE POST NOT FOUND / 403 (not owner) / 422 TUTEE POST NOT ACTIVE.
  2. **Cap:** count By Tutee Post Id And Status In over the current active states REQUESTED, SCHEDULED, PROPOSED, COUNTER PROPOSED, PENDING SCHEDULING reaches 3
     → 422 INTERVIEW LIMIT REACHED ("You can have up to 3 active interviews per request.").
     Counts **active** interviews only, including INT-039 negotiation states. CANCELLED, COMPLETED, NO SHOW,
     TUTOR DECLINED and EXPIRED do not consume capacity. This is not a lifetime cap; do not change it to all
     statuses without a new approved requirement. Constant MAX INTERVIEWS PER POSTING 3 (no magic number).
- Persist tutee Post Id on the interview.

### Frontend
- The interview-request flow must capture which of the tutee's **active postings** the interview
  is for (selector when the tutee has >1 active posting; auto-select when exactly 1; block with a
  "post a request first" nudge when 0).
- Show remaining capacity ("2 of 3 interviews used for this request") and map
  INTERVIEW LIMIT REACHED to a clear inline message; disable the request action at 3.
- No magic strings.

## Acceptance
- A 4th active interview request for the same posting → 422 INTERVIEW LIMIT REACHED.
- Cancelling one active interview frees a slot (a new request then succeeds).
- Requesting against another tutee's posting → 403; against a non-ACTIVE posting → 422.
- Existing duplicate-tutor guard still applies. Coverage gates hold (backend 97.5%, frontend 97.5%/88%).

## Out of scope
- No change to scheduling/no-show flows beyond carrying tutee Post Id.
- ~~The cap is enforced as a count-then-insert and is **best-effort under concurrency**~~ —
  **RESOLVED by INT-077 (2026-06-06).** The interview cap shared the same TOCTOU race as the
  new application/invite caps; INT-077 hardened it the same way: a transaction-scoped
  the pg advisory xact lock operation is taken **before** the count
  (inside the existing a database transaction that saves all related changes together), so concurrent requesters serialize on the posting key
  and the cap can no longer be exceeded. See
  [backend-api implementation reference](../../reference/backend-api.md) "INT-077 — Marketplace
  quota caps" for the shared design.
