# InTuition — Approval Module — Database Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** PENDING-only review, current moderation/reason constraints and atomic audit/outbox intent govern. Legacy current-code omissions, PAUSED FOR REVIEW/drip-state and raw-database structure definitions examples are historical observations; do not reinstate them over later posting/moderation amendments.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Module:** approval  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Tables

### 1.1 post review log

Immutable audit trail for all admin review actions on posts. No update records or DELETE permitted.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2. Constraints

### 2.1 Rejection Reason Required on REJECT

Enforced at the application service layer (the reject operation validates before writing) and additionally guarded by a trigger:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 3. Indexes


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Status-Related Indexes on Post Tables

These indexes exist in the posting module but are critical for the approval queue. Reproduced here for cross-reference:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 5. Permissions


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 6. Audit Log Integration

The approval module writes these action codes to audit log:

| Action Code | Trigger | Data Logged |
|---|---|---|
| POST APPROVED | the approve operation | post id, post type, admin id |
| POST REJECTED | the reject operation | post id, post type, admin id, rejection reason |
| POST REOPENED | the reopen operation | post id, post type, admin id, notes |
| ADMIN VIEW personal information | the get post for review operation | post id, admin id, fields accessed |

The audit log table is defined in the architecture spec (append records-only, GRANT append records ON audit log TO app role).

---

## 7. InTuition async email queue (Spring @Async + Redis) Job Schema (Drip Email Scheduling)

After approving a post, the approval service inserts delayed jobs into InTuition async email queue (Spring @Async + Redis). Jobs are not database-stored, but their job Id pattern is documented here for observability:

| Job ID Pattern | Delay | Notification |
|---|---|---|
| the day-3 follow-up for the specified post | 3 days after approval | N-06: Day 3 check-in email |
| the day-7 follow-up for the specified post | 7 days after approval | N-07: Day 7 shortlisting email |
| the day-14 follow-up for the specified post | 14 days after approval | N-08: Day 14 shortlisting update |
| the day-21 follow-up for the specified post | 21 days after approval | N-09: Admin escalation alert |

On post rejection, all jobs matching the scheduled follow-up messages for the specified post are removed from the queue to prevent sending misleading emails to rejected users.

On PAUSED FOR REVIEW transitions (owner edits), drip jobs are paused but not removed — they resume if the post is re-approved.

## 2026-09-05 current contract and verification

Audit/review facts commit atomically with the approval/rejection; immutable log and required nonblank rejection reason remain. Later moderation lifecycle and reason-length rules supersede the old enum/trigger examples. Any revised migration must preserve applied history and match current enums, not run the illustrative old CREATE TYPE/trigger statements.
