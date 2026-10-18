# Database Migration Strategy

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Liquibase is the sole migration tool. Old migration ledgers and initial-schema/erasure examples are historical; current changelogs and approved retention decisions control. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.
## Singapore Private Tuition Agency — Liquibase XML Migration + Seed Plan

**Document Version:** 1.1  
**Date:** 2026-05-27  
**Status:** Draft  
**Source Documents:** business requirements document v1.1, product requirements document v1.1, schema md v1.1  

---

## 1. Migration tool: Liquibase XML (ADR-004)

Every schema change is a versioned XML changeset in intuition-the private implementation folder, included by the master changelog. Spring Data Java persistence mapping maps entities and Hibernate uses ddl-auto is validate; it is not a migration generator. Do not use schema prisma, Prisma migration commands or a shadow database for this project.

### Workflow

1. Add the next repository-convention XML changeset with a unique id, author, preconditions, bounded lock behavior and a schema-compatible rollback/forward-repair plan. Never alter an already-applied changeset.
2. Put PostgreSQL-specific database structure definitions in supported Liquibase XML elements or a reviewed <sql> section. The database query language examples below are design examples to embed in changesets, not commands to execute ad hoc against production.
3. Update entity mappings and verify migration plus application behavior on PostgreSQL. Local application boot may enable Liquibase; production uses the migration exit-runner and migrate-before-boot sequence in 027.
4. Review old/new application compatibility before dropping or renaming relations. Roll back the application before dropping a table it queries; an absent relation is not an empty table.
5. Reference taxonomy is versioned data migration. Sample users and regression fixtures are local-only seeders; production uses the separate idempotent bootstrap in 027.

Example verification command (from the backend repository):


Implementation example omitted. This public edition retains the written requirements in this section.


This document records the workflow; this correction did not execute a migration or backend test.

## 2. Zero-Downtime Migration Strategy

At MVP scale (single-server deployment, <1,000 users), strict zero-downtime is not a launch requirement. However, the following patterns are followed by convention to build correct habits for Phase 2 scaling:

### 2.1 Zero-Downtime Patterns by Migration Type

| Migration Type | Safe Approach | Unsafe Approach to Avoid |
|---|---|---|
| **Add column (nullable)** | change the record structure for include the field ... no value — fully safe, no lock | Adding required without a default on a large table (rewrites table) |
| **Add column (required with default)** | change the record structure for include the field ... DEFAULT x required — PostgreSQL 11+ sets default metadata-only; no rewrite | Setting required without a default on an existing column with existing rows |
| **Add index** | support efficient lookup with an index CONCURRENTLY — builds index without holding lock | support efficient lookup with an index (non-concurrent) — holds table lock until complete |
| **Drop index** | DROP INDEX CONCURRENTLY — drops without lock | DROP INDEX — holds lock briefly; safe on small tables |
| **Drop column** | 3-step: (1) stop reading, (2) migrate, (3) drop — OR just change the record structure for remove the field on small tables | Dropping a column that application code still references |
| **Change column type** | Add new column → backfill → switch reads → drop old column (4-step) | change the record structure for change the field TYPE on a column with data (table rewrite) |
| **Add required to existing column** | Backfill nulls first → add require constraint → add required in separate migration | Adding required directly on a non-empty column |
| **Rename table/column** | Add new name, dual-write, migrate reads, drop old name | change the record structure for RENAME while application code references old name |
| **Add FK constraint** | change the record structure for require the rule ... related-record reference ... NOT VALID → VALIDATE CONSTRAINT separately | Full FK validation on large table with lock |

### 2.2 Concurrent indexes through Liquibase

Use a dedicated XML changeset with run In Transaction is no for support efficient lookup with an index CONCURRENTLY, with appropriate PostgreSQL preconditions and rollback/repair handling. Do not rely on magic comments in a database query language file or perform an out-of-band schema change. Put each operation requiring non-transactional execution in a bounded, reviewable changeset; validate partial-failure recovery.

## 3. Historical initial-deployment design (not the current changelog)

This old design inventory is retained for context. Actual execution order comes only from the current Liquibase master changelog:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Seed Data

### 4.1 Admin Seed User

The initial admin user is created with a placeholder password hash and no encrypted contact fields. The actual admin account credentials are set via an out-of-band environment process, not committed to source control.


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.2 System Configuration Table

Platform-level configuration values are stored in the formal system config table defined in schema md §17. The table is created in Migration 001 (not as an ad-hoc seed-only construct). The seed values are applied in Migration 004. Keys are standardized to the canonical names below.


Implementation example omitted. This public edition retains the written requirements in this section.


Key naming note: canonical keys are commission rate default, post auto expiry days, and terms version current (not default commission rate or current terms version — updated in schema md v1.1). Application code must reference these exact key strings.

### 4.3 Singapore Subject Taxonomy (business requirements document §12.2)


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.4 Singapore Education Levels Taxonomy (business requirements document §12.1)


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.5 Current seed entry points

Reference/taxonomy changes use reviewed Liquibase data changesets. The earlier database query language examples are historical candidate data, not the approved current canonical list; use009/020/025 and their decision records. Local sample accounts use the guarded Java application seeder. Production uses027’s separate admin bootstrap; never run local fixture seeds there. No Prisma/TypeScript seed entry point is part of this application’s migration workflow.

## 5. Erasure policy dependency and historical example

This procedure is followed when an account deletion request is received (business requirements document Appendix C §C.4, product requirements document §3.10). The business deletion flow is logical and immediately revokes access. The old anonymization example below is historical: it is not an authorized operational recipe until the retention/erasure policy and current schema are reconciled (O08).

### 5.1 Historical erasure example — do not execute as a runbook

The procedure anonymises personal information in-place (to preserve referential integrity of match, commission, and audit records) and sets deleted at. Business deletion remains logical. No blanket physical-row cleanup after two years is authorized; retention and irreversible anonymization timing are open decision O08 in CURRENT-REQUIREMENTS md.


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.2 Profile Photo Deletion (Out-of-Band)

Profile photos are stored in S3. After the database anonymisation transaction commits, trigger the S3 deletion pipeline:


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.3 Hard Delete Job (2-Year Retention)

A scheduled job runs daily and permanently deletes rows that have been soft-deleted for more than 2 years:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 6. Column-Level Encryption Migration Approach

**Historical contingency, not a user-contact migration requirement.** The founder plaintext email/phone amendment supersedes the old assumption that contact encryption launches in Migration001. The following generic example applies only if a future separately approved field-encryption change requires it; any resulting database structure definitions must use versioned Liquibase XML changesets.

### 6.1 Four-Step Encryption Migration

**Scenario:** An existing plaintext column foo must be replaced with foo encrypted (BYTEA) + foo hmac (VARCHAR).


Implementation example omitted. This public edition retains the written requirements in this section.


### 6.2 Key Management Notes

- AES-SIV encryption key and keyed message authentication key are loaded from environment variables (ENCRYPTION KEY BASE64, keyed message authentication KEY BASE64) at application startup. They are never stored in the database.
- Key rotation requires a backfill of all encrypted values — treat as a separate migration event.
- Keys must be stored in a secrets manager (Amazon Web Services Secrets Manager, Hashi Corp Vault, or equivalent) with access logging. Plain .env implementation reference files are acceptable only for local development, never for staging or production.

---

## 7. Rollback strategy

Use the rollback or forward-repair contract recorded in the Liquibase XML changeset. Reversible changes require a verified the rollback instructions; irreversible data changes require an explicit restore/repair plan, approved retention scope and pre-change backup. Match the application version to the schema before removing a relation or enum value. Production027’s canary requires expand-contract compatibility while old/new tasks coexist; a generic single-server backup recipe does not replace that deployment gate.

## 8. Migration Checklist (Per Migration)

Before merging any migration pull request:

- [ ] Migration database query language is idempotent: all define records for statements use when not already defined; all append records statements use when a duplicate is found leave the existing record unchanged
- [ ] New indexes on existing large tables use support efficient lookup with an index CONCURRENTLY
- [ ] GENERATED ALWAYS AS columns are in the reviewed Liquibase XML/database query language changeset
- [ ] Partial indexes are in custom database query language section
- [ ] GRANT / REVOKE statements are included for new tables
- [ ] audit log and commission acknowledgements use explicit per-table GRANT append records (not blanket GRANT ON ALL TABLES) — see schema md v1.1 Role & Grant Summary
- [ ] Seed data uses when a duplicate is found leave the existing record unchanged (safe to re-run)
- [ ] Pre-migration backup documented in pull request description
- [ ] Migration tested on a local database snapshot that mirrors production row counts
- [ ] Verified Liquibase rollback or explicit forward-repair/restore plan for any destructive operation

---

## Appendix A — Anonymization JSONB Property Key Map

This appendix lists every analytics events event name and the top-level JSONB property keys within the properties column that contain surrogate IDs requiring anonymization by the 90-day pipeline job. The pipeline must anonymize both the top-level columns (user id, post id, match id, admin user id) AND the nested JSONB property keys listed here.

**Anonymization rule:** Replace each listed property value with the sha256 operation in hex. Set anonymized yes when all surrogate IDs in the row have been replaced.

| Event Name | properties Keys to Anonymize | Notes |
|---|---|---|
| user registered | properties user id | Anonymize user identity |
| user email verified | properties user id | |
| user deleted | properties user id | Personal Data Protection Act erasure event |
| tutor post created | properties tutor post id, properties tutor id | |
| tutor post submitted | properties tutor post id, properties tutor id | |
| tutor post approved | properties tutor post id, properties tutor id | |
| tutor post rejected | properties tutor post id, properties tutor id | |
| tutor post revision requested | properties tutor post id, properties tutor id | |
| tutor post expired | properties tutor post id, properties tutor id | |
| tutor post closed | properties tutor post id, properties tutor id | |
| tutee post created | properties tutee post id, properties tutee id | |
| tutee post submitted | properties tutee post id, properties tutee id | |
| tutee post approved | properties tutee post id, properties tutee id | |
| tutee post rejected | properties tutee post id, properties tutee id | |
| tutee post revision requested | properties tutee post id, properties tutee id | |
| tutee post expired | properties tutee post id, properties tutee id | |
| tutee post closed | properties tutee post id, properties tutee id | |
| post status changed | properties post id | Generic post status change; anonymize post id |
| match created | properties tutee post id, properties tutor post id, properties match id | Both post IDs and the match ID |
| match activated | properties match id, properties tutee post id, properties tutor post id | |
| match closed | properties match id, properties tutee post id, properties tutor post id | |
| match cancelled | properties match id, properties tutee post id, properties tutor post id | |
| commission created | properties match id, properties commission id | |
| commission status changed | properties match id, properties commission id | Anonymize match id surrogate |
| commission confirmed | properties match id, properties commission id | |
| commission invoiced | properties match id, properties commission id | |
| commission paid | properties match id, properties commission id | |
| commission disputed | properties match id, properties commission id | |
| commission waived | properties match id, properties commission id | |
| admin review action taken | properties post id, properties admin user id | Anonymize post id and admin actor |
| admin match created | properties match id, properties tutee post id, properties tutor post id, properties admin user id | |
| admin commission updated | properties commission id, properties match id, properties admin user id | |
| admin contact field accessed | properties resource id, properties admin user id | resource id is a post or user unique identifier |
| notification sent | properties recipient user id | Anonymize notification recipient |
| notification bounced | properties recipient user id | |
| session login | properties user id | Session auth events |
| session logout | properties user id | |
| mfa bypass attempt | properties user id | Security event — anonymize user id |
| pdpa erasure executed | properties target user id, properties admin user id | Both the erased user and the acting admin |
| post auto expiry job ran | *(none — aggregate event, no surrogate IDs)* | Job-level event only |
| commission acknowledgement recorded | properties user id | Tutor acknowledging commission terms |

**Pipeline implementation notes:**

1. The historical anonymization example uses the dedicated index to find analytics events that have not been anonymized and occurred more than 90 days ago. The current retention and erasure schedule requires explicit approval.
2. For each row, replace top-level columns user id, post id, match id, admin user id (TEXT fields) with their keyed message authentication-SHA256 hex values, then update all properties JSONB keys listed in the table above using PostgreSQL the jsonb set operation.
3. Apply updates in batches of 500–1000 rows per transaction to avoid lock contention.
4. Set anonymized yes in the same update records statement.
5. The anonymization key (anon key) must be a separate secret from any other application hashing key; user email/phone lookup follows the plaintext-contact amendment. It must be rotated on the same schedule as the other application secrets (per business requirements document Appendix B).
6. Events not listed in this table (e g. aggregate/system events with no surrogate IDs) must still have anonymized set to yes after the 90-day window to prevent them from appearing in subsequent cron job scans.

---

---

## 9. Changeset Ledger — Wave 2026-06-03 (0033–0037)

> **Tool:** Liquibase XML (not Prisma/Spring Data Java persistence mapping Migrate — the spec preamble above pre-dates the actual implementation). Changesets live in intuition-the private implementation folder and are registered in db changelog-master implementation reference. Liquibase runs pending changesets in include order on the documented operational check (Testcontainers fresh database) and on every deploy.

> **0032 reserved** — gap intentional. db changelog-master implementation reference contains the comment  
> 

> **Migration order note:** 0035 (job invitations) was developed after 0036/0037 and its <include> is appended at the **end** of db changelog-master implementation reference (after 0037). Liquibase runs un-run changesets regardless of position in the master file — confirmed clean on a fresh Testcontainers database.

### 0033 — matches tutor user id + free-market dedupe (INT-034)

**File:** 0033-match-tutor-user-id implementation reference  
**Changesets:** 0033a through 0033f (six steps)

Re-keys the matches table on tutor user id so listing-less (free-market) tutors can be assigned without requiring a tutor post id.

| Step | Changeset ID | What it does |
|---|---|---|
| a | 0033a-match-tutor-user-id | Adds nullable tutor user id unique identifier column + FK → users id ON DELETE RESTRICT |
| b | 0033b-backfill-tutor-user-id | Backfills tutor user id from tutor posts tutor id for every existing row (idempotent, skips non-null) |
| c | 0033c-tutor-post-id-nullable | Drops the required constraint on tutor post id (FK + ON DELETE RESTRICT from 0008 unchanged) |
| d | 0033d-tutor-user-id-not-no value | Adds required to tutor user id (safe post-backfill) |
| e | 0033e-match-tutor-user-index | support efficient lookup with an index idx matches tutor user id ON matches tutor user id — replaces tutor-dashboard join |
| f | 0033f-match-free-market-dedupe | require a unique lookup index uq matches tutor user pair ON matches tutor user id, tutee post id when status IN PENDING,ACTIVE — database backstop against concurrent double-assign |

**Why:** contact-reveal and the tutor match-dashboard previously joined through tutor post id; listing-less free-market matches had no tutor post id, breaking both. The partial-unique index scope (PENDING, ACTIVE) mirrors Admin Match Service ACTIVE MATCH STATUSES exactly; CANCELLED rows are excluded so cancel→reopen→re-assign still succeeds. Note: match status values are PENDING/ACTIVE/COMPLETED/CANCELLED — MATCHED belongs to job application status, not this enum.

---

### 0034 — Interview timeslot negotiation (INT-039)

**File:** 0034-interview-timeslot-negotiation implementation reference

Adds two-party timeslot negotiation (ping-pong of proposed 15-min slots) before admin scheduling, reusing the existing interviews row. All enum-extension changesets use run In Transaction is no per the PG rule that a freshly added enum value cannot be used in the same transaction.

**New interview status values** (five ALTER TYPE … ADD VALUE when not already defined, each its own changeset):
PROPOSED, COUNTER PROPOSED, TUTOR DECLINED, PENDING SCHEDULING, EXPIRED

**New PG enum types:**
- interview decline reason — NOT INTERESTED, CANT MAKE IT
- timeslot proposer — TUTEE, TUTOR
- interview timeslot status — PROPOSED, WITHDRAWN, ACCEPTED, REJECTED

**Schema changes on interviews** (changeset 0034-interviews-add-negotiation-cols):
- decline reason interview decline reason no value
- agreed timeslot id unique identifier no value (FK added in later changeset 0034-interviews-agreed-slot-fk)
- last actor timeslot proposer no value
- updated at TIMESTAMPTZ no value

**New table interview timeslots** (changeset 0034-create-interview-timeslots):

| Column | Type | Notes |
|---|---|---|
| id | unique identifier PK | the gen random uuid operation |
| interview id | unique identifier required | FK → interviews id automatically remove dependent records |
| proposed by | timeslot proposer required | |
| round | SMALLINT required | Increments per counter round |
| start at | TIMESTAMPTZ required | 15-min Coordinated Universal Time boundary (enforced by ck timeslot quarter hour) |
| slot status | interview timeslot status required | Default PROPOSED |
| created at | TIMESTAMPTZ required | the now operation |

Constraints: uq timeslot interview round start unique interview id, round, start at prevents duplicates. ck timeslot quarter hour enforces minute % 15 0 AND second 0.

**Index:** idx timeslots interview open ON interview timeslots interview id when slot status PROPOSED

**New notification type values** (five ADD VALUE when not already defined, run In Transaction is no):
INTERVIEW PROPOSED, INTERVIEW DECLINED, INTERVIEW COUNTERED, INTERVIEW AGREED, INTERVIEW EXPIRED

**Cap index rebuild** (changeset 0034-recreate-cap-active-index): The M6/0021 cap index idx interviews tutee post active is dropped and recreated over the widened active set:


Implementation example omitted. This public edition retains the written requirements in this section.


This ensures the "max 3 active interviews per parent posting" cap counts negotiating interviews correctly.

---

### 0035 — Job invitations (INT-044)

**File:** 0035-job-invitations implementation reference  
**Master include position:** appended after 0037 (execution order = append order; Liquibase runs by changeset ID within the file)

New job invitations table allowing a tutee to invite a specific tutor to their ACTIVE job. Accept folds into the existing apply + selection-request flow — the invitation never creates a Match directly (AC-04 holds).

**Table job invitations:**

| Column | Type | Notes |
|---|---|---|
| id | unique identifier PK | the gen random uuid operation |
| tutee post id | unique identifier required | FK → tutee posts id |
| tutor user id | unique identifier required | FK → users id |
| created by | unique identifier required | FK → users id (the tutee owner) |
| message | VARCHAR using 500 | Optional invite note |
| status | VARCHAR using 16 required | Default PENDING; varchar (not PG enum) to avoid ALTER TYPE friction |
| job application id | unique identifier | FK → job applications id ON DELETE SET no value; populated on accept |
| responded at | TIMESTAMPTZ | Set on accept/decline |
| created at | TIMESTAMPTZ required | the now operation |
| updated at | TIMESTAMPTZ required | Maintained by trg job invitations updated at trigger (shared the set updated at operation function from 0001) |

**Partial unique index** (0035b): ux jobinv one pending per pair ON job invitations tutee post id, tutor user id when status PENDING — one OPEN invite per (job, tutor); re-invite after a terminal state succeeds.

**Read indexes** (0035c):
- idx jobinv tutor pending ON job invitations tutor user id, created at DESC when status PENDING
- idx jobinv post status ON job invitations tutee post id, status

**New notification type values** (0035e, run In Transaction is no):
JOB INVITATION RECEIVED, JOB INVITATION RESPONDED

---

### 0036 — POST SUBMITTED notification type (INT-025)

**File:** 0036-add-post-submitted-notification-type implementation reference

Single-changeset run In Transaction is no:


Implementation example omitted. This public edition retains the written requirements in this section.


Enables admin fan-out notifications when a tutee or tutor submits a post for review. No table schema changes — the new enum value is consumed by the emit to all admins operation (called from Tutee Post Service and Tutor Post Service after-commit; see INT-025 in the application programming interface reference).

---

### 0037 — Analytics timestamp indexes (INT-045)

**File:** 0037-analytics-timestamp-indexes implementation reference

Six B Tree DESC indexes to support time-bounded COUNT/SUM analytics queries. All created in a single changeset with SET LOCAL lock timeout 3s:

| Index | Table | Column |
|---|---|---|
| idx users created at | users | created at DESC |
| idx tutee posts created at | tutee posts | created at DESC |
| idx tutor posts created at | tutor posts | created at DESC |
| idx interviews requested at | interviews | requested at DESC |
| idx matches matched at | matches | matched at DESC |
| idx commissions paid at | commissions | paid at DESC |

**Why:** Analytics Service issues per-day COUNT/SUM … BETWEEN queries in a loop (up to 366 iterations for a year window). Without these indexes each iteration was a sequential scan. DESC order matches the BETWEEN range predicate on created at/paid at.

---

*End of migrations md — v1.1 — updated 2026-06-03 (wave 0033–0037)*
