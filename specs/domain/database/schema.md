# Database Schema

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Current storage/deletion/lifecycle amendments replace the affected May schema assumptions. The retained database structure definitions is explicitly historical; see the current changelogs before implementing a migration. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.
## Singapore Private Tuition Agency — PostgreSQL Schema Specification

**Document Version:** 1.1  
**Date:** 2026-05-27  
**Status:** Draft  
**Source Documents:** business requirements document v1.1, product requirements document v1.1  

---

## Design Decisions

### 1. Separate Post Tables (not polymorphic)
tutor posts and tutee posts are separate tables. A single polymorphic posts table with a discriminator column was rejected because the two post types have fundamentally different field sets (e g., qualifications, years experience, teaching approach, profile photo s3 key are tutor-only; subject code, level code, num students, session duration hours, frequency per week are tutee-only). Separate tables allow required constraints to be applied correctly on required fields without nullable columns that can never be populated for one post type.

### 2. Availability as Integer Bitmask
availability bitmask INTEGER encodes a 21-bit weekly schedule. Bit positions 0–20 map to:
- Bit 0 = Monday Morning, Bit 1 = Monday Afternoon, Bit 2 = Monday Evening
- Bit 3 = Tuesday Morning, Bit 4 = Tuesday Afternoon, Bit 5 = Tuesday Evening
- Bit 6 = Wednesday Morning, Bit 7 = Wednesday Afternoon, Bit 8 = Wednesday Evening
- Bit 9 = Thursday Morning, Bit 10 = Thursday Afternoon, Bit 11 = Thursday Evening
- Bit 12 = Friday Morning, Bit 13 = Friday Afternoon, Bit 14 = Friday Evening
- Bit 15 = Saturday Morning, Bit 16 = Saturday Afternoon, Bit 17 = Saturday Evening
- Bit 18 = Sunday Morning, Bit 19 = Sunday Afternoon, Bit 20 = Sunday Evening

A bit value of 1 means available. Value range: 0 (no availability) to 2097151 (all slots). Stored as INTEGER (PostgreSQL INTEGER is 4 bytes; 21 bits fit). Application layer decodes bits for display. Overlap queries use bitwise AND: the tutor and tutee have at least one available time slot in common.

### 3. Plaintext user contact fields (explicit amendment)

The plaintext-email-phone decision replaces the encrypted/keyed message authentication user-contact model with normalized the varchar operation and the varchar operation. Password hashing, public personal information restrictions, authorized participant disclosure, auditing and protected backups remain separate requirements. The old ciphertext/keyed message authentication database structure definitions below is a historical initial-schema example, not the current migration contract.

### 4. Separate commissions Table
Commission data is stored in a dedicated commissions table (one row per match), not as fields on matches. This separates the match lifecycle (PENDING → ACTIVE → CLOSED/CANCELLED) from the billing lifecycle (PENDING → AWAITING CONFIRMATION → CONFIRMED → INVOICED → PAID/DISPUTED/WAIVED). Each dimension can evolve independently and be queried separately without column-level joins on a wide row.

### 5. Audit Log — append records-Only Grant
The audit log table is granted append records to the application role (app role) only. No update records or DELETE is granted. At the PostgreSQL level, attempts to modify or delete rows return ERROR: permission denied. This enforces immutability at the database layer, not just the application layer (per business requirements document §10.6.2).

### 6. Analytics as Separate Schema
Analytics events live in the analytics schema, separate from the public schema used by the operational database. This allows separate role grants, different retention jobs, and clear separation of the 90-day anonymization pipeline scope (business requirements document Appendix C §C.3, product requirements document §7.8.5). The anonymization pipeline targets only analytics.* tables and must never touch public audit log or public commission acknowledgements.

---

## Historical initial-schema examples — not an executable current schema

The following tables, enums, indexes and database query language examples retain the May design for reference. Do not apply them to a current database. Use versioned Liquibase changelogs, the plaintext-contact amendment, INT-034/INT-093/INT-094 and numbered requirements for the current model. In particular: no mandatory tutor-listing relationship for matches, no keyed message authentication user-contact lookup, no business physical-delete cascade, and COMPLETED follows agency commission PAID. Analytics-store separation is unresolved O07; a same-database schema is a proposal, not an approved waiver.

### Initial enum examples


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Table Definitions

### 1. users

Primary identity table for all platform users (tutees, tutors, admins). Contact fields are encrypted at rest per business requirements document §10.7.1 and Appendix B. Soft-delete via deleted at supports Personal Data Protection Act right-of-erasure: on deletion, personal information fields are anonymised (display name → "Deleted User [id]", email/phone encrypted columns and keyed message authentication columns nulled), and deleted at is set. Hard delete of the entire row follows after the 2-year retention period (business requirements document Appendix C §C.1).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 2. commission acknowledgements

Write-once audit table recording each tutor's explicit acknowledgement of the commission terms at registration. No update records or DELETE is permitted via application role (enforced by GRANT). Retained for 5 years from acknowledgement date (business requirements document Appendix C §C.1).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 3. tutor posts

Tutor offering posts. One post per tutor per listing (tutors may have multiple active posts — product requirements document §3.3). Contact fields are on the users table; this table contains only post content. profile photo s3 key stores the S3 object key only (never a public URL); signed URLs are generated at access time with 1-hour expiry (product requirements document §14.4).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 4. tutor post subjects

Junction table linking tutor posts to subjects (multi-select). Uses the subject code from the subjects reference table. Composite primary key.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 5. tutor post levels

Junction table linking tutor posts to MOE education levels (multi-select).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 6. tutor post venue types

Junction table linking tutor posts to supported venue types (multi-select).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 7. tutor post group sizes

Junction table linking tutor posts to accepted group sizes (multi-select).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 8. tutee posts

Tutee tuition request posts. A tutee post references a single subject and level (unlike tutor posts which are multi-select). The availability bitmask column uses the same 21-bit encoding as tutor posts. The partial unique index on tutee id where status = 'ACTIVE' is NOT required — tutees may have multiple active posts. The cardinality constraint is on matches, not here.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 9. tutee post venue preferences

Junction table linking tutee posts to preferred venue types (multi-select).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 10. post review log

Immutable audit trail of every admin review action on any post. Records approve, reject, and revision-requested actions with the admin who performed them. This is distinct from audit log (which tracks contact field access); post review log tracks moderation decisions.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 11. matches

Admin-created match records linking one tutee post to one tutor post. A tutee post may have at most one match in PENDING or ACTIVE status simultaneously, enforced by a partial unique index. A tutor post may participate in multiple simultaneous matches (no cardinality constraint on tutor side).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 12. commissions

One commission record per match. Separated from matches to decouple the arrangement lifecycle from the billing lifecycle. The calculated amount sgd is a generated stored column (computed on write; no separate computation needed at read time). Commission is unique on match id — only one commission record per match is allowed (BR-005, AC-14).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 13. audit log

Immutable audit log for admin contact field access (business requirements document §10.6.2). BIGSERIAL primary key (not unique identifier) for efficient append ordering. append records granted to app role only; update records and DELETE not granted. Retained for 5 years, archived to cold storage after 2 years (business requirements document Appendix C §C.1).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 14. system notifications

All outbound notification records (email only in MVP; channel enum allows future SMS/push addition without schema change). Tracks delivery status per notification. Payload stored as JSONB for template rendering without schema changes per new notification type.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 15. subjects

Singapore subject reference taxonomy. Used by tutor post subjects subject code and tutee posts subject code. Loaded as seed data (see migrations md for complete seed database query language).


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 16. levels

Singapore MOE education level reference taxonomy. Used by tutor post levels level code and tutee posts level code.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 17. system config

Platform-level key-value configuration store. Replaces the ad-hoc table previously defined only in the migration seed. Formal schema definition enables GRANT/REVOKE control and trigger-based updated at tracking. Admin-only write access; app role has read fields to read live config values (e g. commission rate default, post auto expiry days, terms version current).


Implementation example omitted. This public edition retains the written requirements in this section.


Seed values (applied in Migration 004 — seed taxonomy):


Implementation example omitted. This public edition retains the written requirements in this section.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Analytics Schema

The analytics schema is separate from the operational public schema. The 90-day anonymization job targets only this schema (product requirements document §7.8.5, business requirements document Appendix C §C.3).


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Role & Grant Summary


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Updated at Trigger

All tables with an updated at column use a shared trigger function to keep it current.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## State Machine Guard Triggers

These trigger functions enforce terminal-state immutability at the database layer, preventing application bugs or direct database query language from transitioning posts or commissions out of a terminal state.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Rejection Reason Enum Constraint

Valid rejection reason codes are enforced by a PostgreSQL require constraint on both tutor posts rejection reason and tutee posts rejection reason. This prevents free-text or typo values from entering the database outside the defined set.


Implementation example omitted. This public edition retains the written requirements in this section.


Valid rejection reason codes:

| Code | Description |
|---|---|
| INCOMPLETE INFO | Post is missing required information |
| UNREALISTIC RATE | Stated rate is implausibly high or low |
| INAPPROPRIATE CONTENT | Post contains inappropriate or offensive content |
| DUPLICATE POST | Duplicate of an existing active post by the same user |
| SPAM TEST POST | Identified as spam or a test submission |
| MISSING QUALIFICATIONS | Tutor does not meet minimum qualification requirements |
| RATE OUT OF MARKET RANGE | Rate is outside the acceptable market range per admin policy |

---

*End of schema md — v1.1 — 2026-05-27*
