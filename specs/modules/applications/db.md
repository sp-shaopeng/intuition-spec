# InTuition — Applications Module (M15) — Database Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** M18 [free-market eligibility](../../PRD.md), [012 completeness](../../012-tutor-apply-and-profile-ux/requirements.md) and [INT-093 state rules](../../architecture/int-093-batch-state-machine-integrity-spec.md) supersede the initial listing prerequisite/two-state lifecycle. This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Module:** applications
**Version:** 1.2.0
**Status:** Implemented; team-reviewed (2026-05-29)
**Last Updated:** 2026-09-05
**Platform:** InTuition
**Tech Stack:** Java 21 + Spring Boot 3.x (PostgreSQL 15)

> Authoritative spec: [spec md](spec.md). Decision record:
> [ADR-013](../../ROADMAP.md). This file documents the schema only;
> endpoint behaviour lives in [backend md](backend.md).

---

## 1. Tables

### 1.1 job application status Enum

status is a **named PostgreSQL enum** — every other status column in this
codebase (interview status, commission status, post status, match status)
is a named enum, and this follows that pattern (DBA mandate; see ADR-013).


Implementation example omitted. This public edition retains the written requirements in this section.


**Thin slice:** only APPLIED and WITHDRAWN are written. The remaining three
values exist so the column is forward-compatible — no endpoint transitions into
them yet (see [spec md §5](spec.md)). The CREATE TYPE runs in **its own
changeset** (default transactional), which commits before the define records for
changeset that references it — so no run In Transaction is no is needed here
(that attribute is only required for ALTER TYPE ... ADD VALUE on an existing enum,
which this migration no longer does).

---

### 1.2 job applications


Implementation example omitted. This public edition retains the written requirements in this section.


**Column notes:**

| Column | Notes |
|---|---|
| tutee post id | The job. FK to the tutee posts operation. The application only succeeds when this post is status ACTIVE (enforced in the service layer, [backend md §1.1](backend.md)). |
| tutor id | The applying tutor's user id. FK to the users operation. No active listing is required under M18. Eligibility uses the shared tutor-profile completeness gate (012); an attached listing stays optional. |
| tutor post id | **Nullable.** Optional pin to the specific listing the tutor applied with. NULL is valid — the applicant need not name a listing, only be eligible. |
| message | Optional pitch, max 500 chars. Not personal information-bearing by design (free text shown to admin only). |
| status | Named enum; defaults to APPLIED. Withdraw flips it to WITHDRAWN. |
| created at / updated at | updated at maintained by the shared the set updated at operation trigger (defined in auth/db md §4.1). |

The original M15 schema illustrated only APPLIED/WITHDRAWN. Current lifecycle includes M18/INT-093 states and guards: withdraw only APPLIED/SHORTLISTED; WITHDRAWN no-op; reapply resolves an existing pair across every state. Use the canonical Liquibase ledger for schema rather than applying this historical database query language sketch.

---

## 2. Constraints and Indexes

Each is justified below; all are mandated by the DBA in ADR-013 (follows the
migration-0009 precedent for partial constraints).

### 2.1 Partial unique — one *active* application per (job, tutor)


Implementation example omitted. This public edition retains the written requirements in this section.


**Rationale.** A tutor may have at most one *non-withdrawn* application per job.
This is what makes **idempotent apply** and **withdraw → re-apply** both work:

- The when status WITHDRAWN clause excludes withdrawn rows from the
  constraint, so when a tutor who once withdrew re-applies, the service can
  **reactivate that same WITHDRAWN row in place** (status → APPLIED) without
  tripping the unique index. A plain the unique operation would not be
  needed for that (the row is reused, not re-inserted) — but the partial clause is
  what guarantees there is never more than **one** active row per pair.
- It prevents two simultaneous APPLIED rows for the same pair (the idempotency
  backstop at the database layer; the service also checks first — a lost concurrent race
  surfaces as a handled 409).

### 2.2 Count index — the applicant count (M14's only source)


Implementation example omitted. This public edition retains the written requirements in this section.


**Rationale.** applicant Count using job = COUNT using DISTINCT tutor id when
tutee post id = ? AND status = 'APPLIED'. M14's landing showcase reads this
across ~20 job cards per page; without a partial index that is ~20 sequential
scans per page render. The when status APPLIED partial index keeps the
count cheap and is the **only** source of the applicant count (the interview-signal
proxy was cut — ADR-013). No count is surfaced until M14 ships.

### 2.3 List index — tutor's "My Applications" page


Implementation example omitted. This public edition retains the written requirements in this section.


**Rationale.** Backs read operation for tutor applications (the tutor's own paginated
list, newest first). Composite tutor id, created at DESC serves both the
filter and the sort from the index.

### 2.4 Partial index on tutor post id


Implementation example omitted. This public edition retains the written requirements in this section.


**Rationale.** tutor post id is frequently NULL (the pin is optional). A partial
index excludes NULL rows so the index stays small and is useful only for the
lookups that actually pin a listing (e g. "applications made with this listing").

---

## 3. Updated at Trigger


Implementation example omitted. This public edition retains the written requirements in this section.


the set updated at operation is the shared function defined in auth/db md §4.1.

---

## 4. Permissions


Implementation example omitted. This public edition retains the written requirements in this section.


Withdrawal is modelled as a status transition (APPLIED → WITHDRAWN), never a row
delete — the single durable row per pair is preserved and reactivated on re-apply.
Funnel history (apply/withdraw/reactivate) is captured via Analytics Event Publisher
events, not by accumulating rows.

---

## 5. Liquibase Changeset

All schema ships as a single XML changeset (no raw database structure definitions anywhere — ADR-004).

**File:** 0018-create-job-applications implementation reference
(registered in db changelog-master implementation reference).

The changeset, in order:

1. **Allowed values for job application status: ...** — in its own changeset
   (default transactional), which commits before the table changeset references it.
2. **define records for job applications** — columns, PK, FKs, defaults as in §1.2.
3. The four constraints/indexes from §2:
   - uq job applications active pair (partial unique, when status WITHDRAWN)
   - idx job applications count (partial, when status APPLIED)
   - idx job applications tutor (tutor id, created at DESC)
   - idx job applications tutor post (partial, when tutor post id has a value)
4. **trg job applications updated at** trigger wiring the set updated at operation.

Partial indexes and the partial unique are expressed via Liquibase
SQL block/index definition with a when clause (Liquibase native index definition
does not express partial predicates, so these use raw SQL block inside the
changeset, consistent with the 0009 precedent).

the documented operational check applies this changeset against the Testcontainers Postgres image.

---

## 6. Entity-Relationship Note


Implementation example omitted. This public edition retains the written requirements in this section.


**Cardinality:**

- job applications N tutee posts — many applications to one job.
- job applications N users — many applications by one tutor (across jobs).
- job applications N tutor posts — **optional** (dashed); the pin to a
  specific listing may be NULL.

The pair tutee post id, tutor id is unique among non-withdrawn rows, which is
why a tutor can re-appear for the same job only after withdrawing the prior one.
