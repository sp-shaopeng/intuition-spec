# Database Index Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Removed the obsolete user-contact keyed message authentication lookup contract. Retained index examples require reconciliation with current Liquibase schema and logical deletion. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.
## Singapore Private Tuition Agency — PostgreSQL Indexes

**Document Version:** 1.1  
**Date:** 2026-05-27  
**Status:** Draft  
**Source Documents:** business requirements document v1.1, product requirements document v1.1, schema md v1.1  

---

## Index Design Principles

1. **Every FK column is indexed.** PostgreSQL does not auto-index foreign keys. Without explicit indexes on FK columns, JOIN and ON DELETE operations trigger sequential scans.
2. **Normalized plaintext email has the approved unique index.** User-contact keyed message authentication columns were removed by the explicit plaintext amendment. Phone uniqueness/validation follows the current account contract; do not infer a unique phone constraint from the historical example.
3. **Partial indexes for filtered queries.** The most common filtered queries (e g., active posts, open matches) use partial indexes scoped to the relevant status values, keeping index size small and scan efficiency high.
4. **Composite indexes ordered by selectivity.** High-cardinality columns are placed first in composite indexes to narrow the scan window before lower-selectivity columns.
5. **Covering indexes where beneficial.** For the public browse queries (tutor listing, tutee discovery), composite indexes are designed to cover the filter columns so the planner can use index-only scans and avoid heap fetches.
6. **Bitmask overlap queries.** PostgreSQL cannot use a B-tree index on availability bitmask & 1 differs from 0. These queries do a sequential scan of the filtered result set after status/subject/level filtering has already reduced the candidate set. No dedicated bitmask index is created; the multi-column filter indexes are sufficient at MVP scale (target: 500 active tutor posts, 300 active tutee posts).

---

## Table: users

### Current contact lookup

Use normalized users email and its approved unique index. Contact migration0022 and later Liquibase changesets define actual index names and active/deleted-row behavior. The old email/phone keyed message authentication indexes are obsolete and must not be recreated. Query tuning must preserve soft-delete and actor-access predicates.

### Remaining initial index examples

The examples below are historical planning material. Reconcile each table, column, status and soft-delete predicate with current changelogs before use; none is an instruction to run standalone database structure definitions.

### idx users role active — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Support admin user management queries filtered by role and active status.
- **Lookup purpose:** Find active tutor accounts that have not been deleted for the administration user list.

### idx users deleted at — BTREE (partial: deleted accounts)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Support the Personal Data Protection Act hard-delete retention job that targets soft-deleted accounts older than 2 years.
- **Query pattern:** The historical example selects accounts with a recorded deletion timestamp more than two years old; it does not authorize deletion under the unresolved current retention policy.

---

## Table: commission acknowledgements

### idx commission ack user id — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Support FK lookup and the admin check "has this tutor acknowledged commission terms?".
- **Lookup purpose:** Find the latest commission acknowledgement for a user before enabling post approval.

### idx commission ack user terms — BTREE (composite)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Validate that the acknowledged terms version matches the current active version (AC-10).
- **Lookup purpose:** Check whether a user acknowledged a particular version of the commission terms.

---

## Table: tutor posts

### idx tutor posts tutor id — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup and tutor's own post list view.
- **Lookup purpose:** List a tutor’s posts, newest first.

### idx tutor posts status — BTREE (partial: non-terminal)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin review queue and matching dashboard filter by status. Partial index excludes terminal states to keep index size proportional to the active working set.
- **Lookup purpose:** List posts awaiting review in submission order, oldest first.

### idx tutor posts active region status — BTREE (composite, partial)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Public tutor browse filtered by region (fast path before subject/level join).
- **Lookup purpose:** Find active tutor posts in the selected region; East is the example.

### idx tutor posts approved at desc — BTREE (partial: active posts)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Public browse default sort (most recently approved first), covering only active posts.
- **Lookup purpose:** Find the twenty most recently approved active tutor posts.

### idx tutor posts reviewed by — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup; support admin workload queries ("which posts has admin X reviewed?").

### idx tutor posts submitted at — BTREE (partial: pending)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin review queue default sort (oldest submission first, for service-level agreement compliance).
- **Lookup purpose:** List pending tutor posts in submission order, oldest first.

---

## Table: tutor post subjects

### idx tps subject code — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Filter tutor posts by subject. Used by public browse, admin matching dashboard left panel filter, and the JOIN path for subject-based matching queries.
- **Lookup purpose:** Find tutor posts that offer the selected subject.

### idx tps tutor post id — BTREE (covers reverse lookup)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Note:** The composite PK tutor post id, subject code already supports both when tutor post id 1 (leading column lookup for post subject retrieval) and when subject code 1 requires the separate index above.

---

## Table: tutor post levels

### idx tpl level code — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Filter tutor posts by level. Used by public browse, admin matching dashboard, and subject+level combined filtering.
- **Lookup purpose:** Find tutor posts that offer the selected education level.

---

## Table: tutor post venue types

### idx tpvt venue type — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Filter tutor posts by venue type.
- **Lookup purpose:** Find tutor posts that support the selected venue type.

---

## Combined Subject + Level Filter Query Pattern

The admin matching dashboard and public browse both filter by subject AND level simultaneously. The optimal query plan for this compound filter:


Implementation example omitted. This public edition retains the written requirements in this section.


This query uses:
1. idx tps subject code to find candidate tutor post id values for the subject
2. idx tpl level code to narrow by level (or vice versa depending on selectivity)
3. idx tutor posts active region status to filter by region + status

---

## Table: tutor post group sizes

### idx tpgs group size — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Support filtering by group size (one-on-one vs small group) on browse pages.
- **Lookup purpose:** Find tutor posts that support the selected group size.

---

## Table: tutee posts

### idx tutee posts tutee id — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup and tutee's own post list view.
- **Lookup purpose:** List a tutee’s requests, newest first.

### idx tutee posts status — BTREE (partial: non-terminal)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin review queue filter; matching dashboard left panel.

### idx tutee posts active discovery — BTREE (partial: active, composite)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Tutor discovery index primary filter path (product requirements document §3.6). Only posts in ACTIVE status are shown in the discovery index (not MATCHED, PENDING, etc.). The composite subject code, level code, location region covers the most common filter combination.
- **Lookup purpose:** Find active requests matching the selected subject, level and region, most recently approved first.

### idx tutee posts submitted at pending — BTREE (partial: pending)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin review queue sort (oldest first for service-level agreement compliance — product requirements document §6.1).

### idx tutee posts approved at desc active — BTREE (partial: active)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Tutor discovery index default sort (most recently approved first).

### idx tutee posts reviewed by — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup; admin workload queries.

### idx tutee posts active budget — BTREE (partial: active, composite)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin matching dashboard budget/rate overlap filter. Admin filters tutee posts by budget range overlap with tutor rate ranges.
- **Lookup purpose:** Find active requests whose budget range overlaps the tutor’s rate range: the request maximum must be at least the tutor minimum, and the request minimum no greater than the tutor maximum.

---

## Table: tutee post venue preferences

### idx tpvp venue type — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Filter tutee posts by venue preference in discovery index and matching dashboard.

---

## Table: post review log

### idx prl post id type — BTREE (composite)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Retrieve review history for a specific post (admin per-post review view shows revision history — product requirements document §6.2).
- **Lookup purpose:** List review history for a specified post type and identifier, newest first.

### idx prl admin id — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin workload reporting; audit of which admin moderated which posts.

---

## Table: matches

### idx matches tutee post active unique — unique (partial) [CRITICAL — defined in schema md]


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Enforce the tutee post cardinality rule: at most one PENDING or ACTIVE match per tutee post (business requirements document §4, product requirements document §7.4, AC-04). This is a business-critical constraint enforced at the database level.
- **Query pattern:** append records or update records that would create a second PENDING/ACTIVE match for the same tutee post id raises unique violation (SQLSTATE 23505).

### idx matches tutee post id — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup and tutee's match history view. Composite with status supports filtered queries (e g., find the active match for a tutee post).
- **Lookup purpose:** Find an active match for the selected tuition request.

### idx matches tutor post status — BTREE (composite)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Find all active matches for a tutor post. Supports the matching dashboard view and the tutor's "My Matches" page. Referenced explicitly in product requirements document §7.8.1 as a recommended index.
- **Lookup purpose:** Find pending or active matches for the selected tutor listing.

### idx matches created by admin — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup; admin workload reporting ("matches created by admin X this month").

### idx matches status activated at — BTREE (partial: awaiting confirmation)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Support the first-session confirmation timeout cron job (product requirements document §7.7.1). The cron selects active matches whose commission awaits first-session confirmation and whose activation was more than seven days ago. This partial index covers the status filter; activated at ASC ordering finds the oldest overdue matches first.
- **Query pattern (cron):** Find active matches with a corresponding commission awaiting confirmation, activated more than seven days ago.

---

## Table: commissions

### idx commissions match id — unique [defined in schema md via unique constraint]


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Enforce one commission per match (BR-005, AC-14). Enables fast lookup of the commission record for a given match.
- **Lookup purpose:** Find the commission record for a specified match.

### idx commissions status — BTREE (partial: non-terminal)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin commission ledger view filtered by open commission statuses. Partial index excludes terminal states.
- **Lookup purpose:** List commissions awaiting first-session confirmation, newest first.

### idx commissions tutor post id — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup; admin commission ledger filtered by tutor.

### idx commissions tutee post id — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup; denormalized column for query efficiency (product requirements document §7.8.2).

### idx commissions created by admin — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup.

### idx commissions invoice sent awaiting payment — BTREE (partial: INVOICED)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin commission ledger sorted by invoice date to track outstanding invoices (oldest first = highest urgency). Payment due in 14 days per product requirements document §8.4.

---

## Table: audit log

### idx audit log admin user created at — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Personal Data Protection Act access reports and DPO review: "all contact field accesses by admin X in date range Y–Z". Also supports individual admin accountability queries.
- **Lookup purpose:** Find an administrator’s recorded actions within the selected date range, newest first.

### idx audit log resource — BTREE (composite)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Look up all audit entries for a specific resource (e g., "all admin accesses to tutor post X"). Supports per-record access history in the admin audit log view.
- **Lookup purpose:** Find recorded actions concerning the selected tutor post, newest first.

### idx audit log event type created at — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Filter audit log by event type for operational monitoring (e g., count of CONTACT FIELD ACCESS events per day, or multi-factor authentication BYPASS ATTEMPT alerts).
- **Query pattern:** Count recorded MFA-bypass-attempt events from the past 24 hours.

---

## Table: system notifications

### idx sysnotif recipient user id — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** FK lookup; tutee/tutor notification inbox view (P1 feature). User's notification history.

### idx sysnotif status scheduled — BTREE (partial: queued)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Notification dispatch worker: find the next batch of queued notifications to send. Only indexes QUEUED records.
- **Query pattern (worker):** Find queued notifications whose scheduled time has arrived, ordered by earliest scheduled time, taking at most 100 records per batch.

### idx sysnotif event type — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Admin notification audit; delivery rate monitoring per event type.

---

## Table: analytics events

### idx analytics events event name occurred — BTREE


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Analytics queries aggregating by event type and date range (business requirements document §9 KPI tracking).
- **Query pattern:** Count match-created analytics events from the past 30 days.

### idx analytics events anonymized occurred — BTREE (partial: not yet anonymized)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** 90-day anonymization job: find records not yet anonymized that are older than 90 days. Partial index covers only un-anonymized rows.
- **Query pattern (cron):** The historical anonymization example selects up to 1,000 analytics events not yet anonymized and older than 90 days. Apply only an explicitly approved current retention policy.

### idx analytics events user id not anonymized — BTREE (partial: not yet anonymized)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Purpose:** Personal Data Protection Act data-access requests: within the 90-day window before anonymization, regulators or DPO staff may request all analytics events associated with a specific user. This partial index on user id (TEXT — see schema md v1.1 Change 1 note) supports when user id 1 AND anonymized no lookups without scanning the full events table. The partial condition when anonymized no keeps the index size proportional to the live (non-anonymized) working set only.
- **Lookup purpose:** For a data-access request, find that user’s non-anonymised analytics events, newest first.
- **Notes:** user id is TEXT type in schema v1.1 to accommodate both unique identifier strings (pre-anonymization) and keyed message authentication-SHA256 hex strings (post-anonymization). This index operates exclusively on the pre-anonymization rows (partial: when anonymized no).

---

## Index Inventory Summary

| Table | Index Name | Type | Columns | Partial | Purpose |
|---|---|---|---|---|---|
| users | idx users email hmac | unique BTREE | email hmac | No | Login / dedup lookup |
| users | idx users phone hmac | unique BTREE | phone hmac | when required | Phone dedup |
| users | idx users role active | BTREE | role, is active | when NOT deleted | Admin user list |
| users | idx users deleted at | BTREE | deleted at | when required | Retention cleanup job |
| commission acknowledgements | idx commission ack user id | BTREE | user id | No | Acknowledgement lookup |
| commission acknowledgements | idx commission ack user terms | BTREE | user id, terms version | No | Version validation |
| tutor posts | idx tutor posts tutor id | BTREE | tutor id | No | Own post list |
| tutor posts | idx tutor posts status | BTREE | status | when non-terminal | Review queue |
| tutor posts | idx tutor posts active region status | BTREE | location region, status | when ACTIVE | Browse by region |
| tutor posts | idx tutor posts approved at desc | BTREE | approved at DESC | when ACTIVE | Browse default sort |
| tutor posts | idx tutor posts reviewed by | BTREE | reviewed by | when required | FK + admin workload |
| tutor posts | idx tutor posts submitted at | BTREE | submitted at ASC | when PENDING | Review queue sort |
| tutor post subjects | idx tps subject code | BTREE | subject code | No | Subject filter |
| tutor post levels | idx tpl level code | BTREE | level code | No | Level filter |
| tutor post venue types | idx tpvt venue type | BTREE | venue type | No | Venue filter |
| tutor post group sizes | idx tpgs group size | BTREE | group size | No | Group size filter |
| tutee posts | idx tutee posts tutee id | BTREE | tutee id | No | Own post list |
| tutee posts | idx tutee posts status | BTREE | status | when non-terminal | Review queue |
| tutee posts | idx tutee posts active discovery | BTREE | subject code, level code, location region | when ACTIVE | Discovery index filter |
| tutee posts | idx tutee posts submitted at pending | BTREE | submitted at ASC | when PENDING | Review queue sort |
| tutee posts | idx tutee posts approved at desc active | BTREE | approved at DESC | when ACTIVE | Discovery sort |
| tutee posts | idx tutee posts reviewed by | BTREE | reviewed by | when required | FK + admin workload |
| tutee posts | idx tutee posts active budget | BTREE | budget min sgd, budget max sgd | when ACTIVE | Budget overlap filter |
| tutee post venue preferences | idx tpvp venue type | BTREE | venue type | No | Venue filter |
| post review log | idx prl post id type | BTREE | post type, post id, created at DESC | No | Per-post review history |
| post review log | idx prl admin id | BTREE | admin id, created at DESC | No | Admin workload |
| matches | idx matches tutee post active unique | unique BTREE | tutee post id | when PENDING/ACTIVE | Cardinality enforcement |
| matches | idx matches tutee post id | BTREE | tutee post id, status | No | Tutee match lookup |
| matches | idx matches tutor post status | BTREE | tutor post id, status | No | Tutor match lookup |
| matches | idx matches created by admin | BTREE | created by admin id, created at DESC | No | Admin workload |
| matches | idx matches status activated at | BTREE | activated at ASC | when ACTIVE | 7-day timeout cron |
| commissions | *(unique constraint auto-index)* | unique BTREE | match id | No | One commission per match |
| commissions | idx commissions status | BTREE | status, created at DESC | when non-terminal | Commission ledger |
| commissions | idx commissions tutor post id | BTREE | tutor post id | No | FK |
| commissions | idx commissions tutee post id | BTREE | tutee post id | No | FK |
| commissions | idx commissions created by admin | BTREE | created by admin id | No | FK |
| commissions | idx commissions invoice sent awaiting payment | BTREE | invoice sent at ASC | when INVOICED | Outstanding invoices |
| audit log | idx audit log admin user created at | BTREE | admin user id, created at DESC | No | Personal Data Protection Act access reports |
| audit log | idx audit log resource | BTREE | resource type, resource id, created at DESC | No | Per-record access history |
| audit log | idx audit log event type created at | BTREE | event type, created at DESC | No | Security monitoring |
| system notifications | idx sysnotif recipient user id | BTREE | recipient user id, created at DESC | No | Notification inbox |
| system notifications | idx sysnotif status scheduled | BTREE | scheduled at ASC | when QUEUED | Dispatch worker |
| system notifications | idx sysnotif event type | BTREE | event type, created at DESC | No | Delivery monitoring |
| analytics events | idx analytics events event name occurred | BTREE | event name, occurred at DESC | No | KPI aggregation |
| analytics events | idx analytics events anonymized occurred | BTREE | occurred at ASC | when NOT anonymized | Anonymization cron |
| analytics events | idx analytics events user id not anonymized | BTREE | user id (TEXT) | when NOT anonymized | Personal Data Protection Act data-access requests (90-day window) |

---

*End of indexes md — v1.1 — 2026-05-27*
