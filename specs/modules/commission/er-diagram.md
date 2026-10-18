# Module 5: Admin Commission Tracking — Entity-Relationship Diagram

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** The original minimal ER diagram is historical. [INT-034](../../architecture/int-034-free-market-assign.md) makes the tutor user canonical with optional listing; [INT-093](../../architecture/int-093-batch-state-machine-integrity-spec.md) explicitly guards PENDING/PAID/CANCELLED commissions. Agency receipt of tutor commission is PAID; it atomically completes the match. Rich invoice-state proposals require their own approval record. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


## Current identity and commission relationship


Implementation example omitted. This public edition retains the written requirements in this section.


The tutor user is canonical and the listing is optional (INT-034). Commission PENDING/PAID/CANCELLED guards follow INT-093; PAID means agency receipt. Exact column/index migrations come from the maintained Liquibase ledger. The older full diagram below is preserved only as migration history.

## Historical pre-M18 Plant UML ER Diagram (do not implement)


Implementation example omitted. This public edition retains the written requirements in this section.


## Key Design Notes

### Partial unique index vs. unconditional unique constraint

Migration 0008-create-matches implementation reference initially created CONSTRAINT uq matches pair unique tutor post id, tutee post id. This prevented the same pair from being matched more than once ever, including after cancellation. Migration 0009-matches-partial-unique-index implementation reference dropped that constraint and replaced it with uq matches open pair, scoped to rows where status IN PENDING, ACTIVE.

The application-layer duplicate guard in Admin Match Service create Match checks the same scope: it calls exists By Tutor Post Id And Tutee Post Id And Status In with PENDING, ACTIVE. The database index mirrors that logic to prevent any race condition that could slip past the application check.

### Why commissions uses VARCHAR using 20 for invoice status

The commission status value set has only two members (PENDING, PAID) but is stored as the varchar operation rather than a PostgreSQL enum. The Liquibase comment in 0010-create-commissions implementation reference explains this is to simplify future migrations: adding a new enum label to a PostgreSQL enum type requires an ALTER TYPE statement and an exclusive table lock, whereas changing VARCHAR check constraints or adding a new valid string is easier.

### One-to-one relationship between commissions and matches

The commissions match id column carries both required and unique constraints, making it a strict 1:1 relationship. A match without a commission is a PENDING or CANCELLED match that was never activated. An ACTIVE or COMPLETED match always has exactly one commission.

The Java persistence mapping mapping on Commission match uses the one to one operation with the join column operation.

### Referential integrity

Both tutor posts and tutee posts are referenced from matches with ON DELETE RESTRICT, and matches is referenced from commissions with ON DELETE RESTRICT. This means no post or match can be hard-deleted from the database while downstream records exist. Soft deletion (status changes) is the intended cleanup path.
