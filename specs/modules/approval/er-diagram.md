# Module 3 — Admin Post Approval: ER Diagram

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The abbreviated historical diagram/table below is not the full current lifecycle. Use [approval/spec md](spec.md) canonical states, including REVISION REQUESTED, and preserve owner revision/resubmission plus the required review/audit history. Claims about what earlier implementation logged do not waive either record.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

This diagram covers all database entities involved in the approval workflow. Tables owned by other modules (posting, auth) are shown here for cross-reference because the approval service reads and writes them.

---

## Plant UML ER Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Table Notes

### tutor posts / tutee posts — Approval-Relevant Columns

| Column | Set By | On Which Transition |
|---|---|---|
| status | Admin (via service) | PENDING → ACTIVE or REJECTED |
| rejection reason | Admin | PENDING → REJECTED |
| admin notes | Admin (optional) | Any review action |
| reviewed at | System | Set to NOW operation on any admin decision |
| reviewed by | System | Set to admin user ID on any decision |
| approved at | System | Set to NOW operation on ACTIVE transition only |

### audit log — Data Written by Approval Module


The audit log — data written by approval module example describes information including event type, actor id, user id, details, action, post Id, post Type. Concrete payload values and implementation syntax are omitted.


The audit log — data written by approval module example describes information including event type, actor id, user id, details, action, post Id, post Type, rejection Reason. Concrete payload values and implementation syntax are omitted.


### post review log — database-Level Constraint on Rejection

The database enforces that rejection reason is not null when action REJECT via a trigger (trg prl rejection reason). If the application omits the reason, the append records will fail with a PostgreSQL exception. This is a defence-in-depth guard — the controller layer validates @Not Blank before reaching the database.

### password reset tokens — Not Involved in Approval

The password reset tokens table exists in the auth module and is not read or written by the approval workflow. It is shown here only for completeness as a related table on the users entity.

### post review log vs audit log — When to Use Each

| Table | Purpose | Queried By |
|---|---|---|
| audit log | Security audit trail; terse; actor + target + event | Security audits, compliance exports |
| post review log | Operational review history; richer; includes notes, rejection reasons | Admin review history user interface, service-level agreement reporting |

The current Admin Post Service implementation writes to audit log only. The post review log write (specified in the backend spec) is not yet implemented in the running code — see Debugging Guide §4.

---

## Partial Index — Approval Queue Performance

These indexes are critical for the the get pending queue operation call performance at scale:


Implementation example omitted. This public edition retains the written requirements in this section.


Without these partial indexes, the queue query performs a full table scan filtered by status.
