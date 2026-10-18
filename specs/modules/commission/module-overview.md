# Module 5: Admin Commission Tracking — Overview

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The simple match/collection status table below is not the full money workflow. Activation locks the agreed first-month plan (INT-157), then first lesson → authenticated matched-tutee confirmation (or audited allowed admin override) → existing invoice/payment progression. Mark paid confirms agency receipt of tutor commission and atomically completes the match; parent lesson payment is separate. See [product requirements document §8.3/§20.3](../../PRD.md). A confirmation stage may be represented separately from collection status; it is not waived by this shorter table. Listing-less M18 assignment is allowed.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

## Purpose

Module 5 gives admin users a single surface to manage the full tutor-tutee match lifecycle and record commission payments. It extends the match record introduced in Module 4 by adding a financial layer: when a match is activated the system records the agreed monthly fee, calculates a 50% agency commission, and tracks whether the agency has actually received the tutor commission. Parent lesson payments are separate.

The admin Commission Ledger page (Commission implementation reference) is the primary user interface for this module. It shows all matches in a single table and surfaces per-row actions (Activate, Mark Paid, Cancel Match) based on the current match and commission state.

## Actors

Module 5 is accessible only to users with ROLE ADMIN. All four application programming interface endpoints enforce this via the pre authorize operation on the controller classes and Spring Security's request-level authentication check. A request from a non-admin authenticated user returns 403 Forbidden. An unauthenticated request returns 401 Unauthorized.

## Match Lifecycle State Machine


The illustration for match lifecycle state machine contains the following information:

- (admin creates match).
- [ PENDING ].
- (activate) / \ (cancel).
- [ ACTIVE ] [ CANCELLED ].
- (mark paid) \ (cancel).
- [ COMPLETED ] [ CANCELLED ].


Valid transitions:

| From      | To          | Trigger                                              |
|-----------|-------------|------------------------------------------------------|
| PENDING   | ACTIVE      | POST /activate with valid monthly Fee Sgd              |
| PENDING   | CANCELLED   | POST /cancel                                         |
| ACTIVE    | COMPLETED   | Admin records actual agency receipt of the tutor commission through the confirmed Mark paid action |
| ACTIVE    | CANCELLED   | POST /cancel                                         |
| COMPLETED | (terminal)  | No further transitions allowed                       |
| CANCELLED | (terminal)  | No further transitions allowed; pair can be re-matched (see below) |

Attempting to transition out of a terminal state (COMPLETED or CANCELLED) returns 422 Unprocessable Entity.

## Commission Calculation

A Commission record is created when a match is activated. The commission amount is computed once and stored:


Implementation example omitted. This public edition retains the written requirements in this section.


- commission Rate is stored as 0.5 (50%) on every commission row.
- The rate is fixed in Admin Match Service DEFAULT COMMISSION RATE 0.5.
- commission Amount Sgd is an INTEGER in the database (SGD cents not used; amounts are whole dollars).
- Math round rounds half-up on tie (standard Java rounding), so SGD 1 rounds to SGD 1.

Example: monthly Fee Sgd 360 → commission Amount Sgd 180.

## Business Rules

### Duplicate match guard (partial unique index)

Only one open match (PENDING or ACTIVE) may exist for a given tutor post id, tutee post id pair at any time. This is enforced at two levels:

1. **Application layer** — Admin Match Service create Match calls the exists by tutor post id and tutee post id and status in operation and throws 409 Conflict with error code DUPLICATE MATCH if an open match exists.

2. **Database layer** — a partial unique index backs this constraint:


Implementation example omitted. This public edition retains the written requirements in this section.


   This index was added in changeset 0009-add-partial-unique-index (migration file 0009-matches-partial-unique-index implementation reference), replacing the unconditional the unique operation constraint from changeset 0008. Without the partial index, attempting to re-match a previously CANCELLED pair would produce a PostgreSQL constraint violation (unhandled 500) rather than succeeding with 201.

CANCELLED pairs can therefore be re-matched. There is no limit on the number of CANCELLED matches for a pair.

### Monthly fee ceiling

Activate Match Request monthly Fee Sgd is annotated the max operation. Requests with a value above 50,000 SGD fail Bean Validation and return 400 Bad Request before reaching the service layer.

### Commission rate

The commission rate is not configurable at runtime. It is a compile-time constant (DEFAULT COMMISSION RATE 0.5) in Admin Match Service. Changing the rate requires a code deployment.

### One commission per match

The commissions match id column carries a unique constraint and a required constraint. Each match can have at most one commission. A second activation of the same match is blocked at the application layer (the match must be PENDING to activate, and activation transitions it to ACTIVE), so the database constraint acts as a safety net rather than the primary guard.

### Post status prerequisite

Both tutor and tutee posts must be in ACTIVE status at the time of match creation. This is enforced in Admin Match Service create Match. It is not re-checked at activation time; if a post's status changed between creation and activation that does not block activation.

## Integration Points

- **Module 3 (Post Approval)** — Posts must reach ACTIVE status through the M3 approval workflow before they can be matched. Admin Match Service create Match returns 422 with TUTOR POST NOT ACTIVE or TUTEE POST NOT ACTIVE if this prerequisite is not met.
- **Module 4 (Admin Matching)** — Module 4 introduced the matches table, the create Match flow, and the Match Status enum. Module 5 adds the activate, cancel, and pay flows on top of that foundation. Both modules share Admin Match Service; Module 4 methods live in the same class but are documented separately.
- **No outbound notifications** — Notification delivery on match events is noted as a future enhancement in the Admin Match Service Javadoc. The frontend success screen already reads "No contact details have been shared yet. Please contact them directly."
