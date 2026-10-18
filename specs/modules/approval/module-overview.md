# Module 3 — Admin Post Approval: Overview

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Atomic business/audit/outbox intent replaces the old best-effort audit rule. Earlier “current implementation” observations are historical, not current requirements.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Module:** approval
**Version:** 1.0.0 (developer documentation)
**Last Updated:** 2026-05-28
**Source:** Derived from live backend source, frontend source, and approval spec v1.1.0

---

## 1. Purpose

The Approval module is the mandatory gate between a user's post submission and its public visibility on the InTuition marketplace. No post goes live without an admin decision. This module governs:

- The admin review queue that aggregates pending TUTOR and TUTEE posts in one unified view.
- The individual post detail view with decrypted contact information (personal information) visible only to admin.
- The approve and reject actions, each of which triggers a status transition, an immutable audit log entry, and an async email notification to the post owner.

This module does **not** cover: the matching workflow (modules/matching/), the posting creation flow (modules/posting/), or commission calculation (modules/commission/).

---

## 2. Actors

| Actor | Description |
|---|---|
| **Admin** | Internal InTuition staff. The only actor permitted to access all four approval endpoints. Must hold ROLE ADMIN. |
| **Tutor** | Marketplace user with ROLE TUTOR. Submits Tutor Posts via the posting wizard. Receives email notifications on approval/rejection. Cannot access admin endpoints. |
| **Tutee** | Marketplace user with ROLE TUTEE. Submits Tutee Posts via the request form. Receives email notifications on approval/rejection. Cannot access admin endpoints. |
| **System** | Spring Boot application. Handles async email dispatch via Redis Streams (Email Queue Service) and writes audit logs. The system never approves or rejects autonomously — that decision belongs to Admin. |

---

## 3. Post Types

Two post types share the same approval flow. They differ only in their field sets.

| Attribute | Tutor Post | Tutee Post |
|---|---|---|
| database table | tutor posts | tutee posts |
| Owner role | TUTOR | TUTEE |
| Subjects/levels | Multi-select (junction tables) | Single subject + single level |
| Rate fields | hourly rate min sgd / hourly rate max sgd | budget min sgd / budget max sgd |
| Extra fields | Qualifications, teaching approach, years experience, group sizes, profile photo | Session duration, frequency per week, num students, student relationship, special requirements |
| Post volume per user | Multiple simultaneous active posts allowed | Multiple posts for different subjects allowed |

Both types use the same Post Status enum and follow the same PENDING → ACTIVE / REJECTED state transitions.

---

## 4. Post Status Lifecycle

The full lifecycle is defined in the application component Status:


The illustration for . post status lifecycle contains the following information:

- DRAFT → PENDING → ACTIVE.
- → REVISION REQUESTED → PENDING (re-enter queue on owner resubmit).
- → REJECTED (terminal).
- ACTIVE → MATCHED (Tutee Post only, via matching module).
- → CLOSED (owner or admin).
- → EXPIRED (daily scheduler, 02:00 Singapore time).
- → PENDING (if owner edits an active post — edit-while-active).


**This module is only responsible for the PENDING → ACTIVE and PENDING → REJECTED transitions.** All other transitions are managed by the posting module, matching module, or scheduled jobs.

---

## 5. Business Rules

| Rule | Detail |
|---|---|
| **BR-001 (Privacy)** | Email and phone are stored encrypted (email encrypted, phone encrypted as AES-SIV ciphertext on the users table). They are decrypted **only** in Admin Post Service for admin-facing responses. They are **never** included in any public or owner-facing application programming interface response. |
| **BR-002 (No auto-approval)** | Every post must pass through admin review. There is no automated approval pathway. |
| **BR-Approval-1 (Start state)** | Admin can only approve or reject a post in PENDING status. Attempting to act on any other status results in a 404 (the current service implementation does not validate status before acting — see Debugging Guide §4 for implications). |
| **BR-Approval-2 (Rejection reason required)** | A rejection requires a non-blank reason field in the request body, validated by @Not Blank at the controller and enforced by a database trigger on post review log. |
| **BR-Approval-3 (Audit required)** | Every approve and reject action writes an entry to audit log (table: audit log, event types: ADMIN POST APPROVE, ADMIN POST REJECT). The audit row commits in the same transaction as the approval/rejection; audit persistence failure rolls back the action. An after Commit or swallowed-error audit is not compliant (durable-side-effects spec INT-066 Option A). |
| **BR-Approval-4 (Email notification)** | Every approve and reject action queues an async email to the post owner via Redis Streams key email-notifications. External delivery failure is retried separately, but the required durable notification/outbox intent commits atomically with the action; do not equate provider failure with failure to persist the intent. |
| **BR-Approval-5 (No double-review guard in current impl)** | The current Admin Post Service does **not** check current post status before updating. A 409 idempotency guard (specified in the backend spec) is not yet implemented in the running code. See Debugging Guide §4. |

---

## 6. Privacy and Encryption Architecture

Contact fields reside on the users table, not on post tables. This is intentional — post records are safe to return publicly because they contain no personal information.


The illustration for . privacy and encryption architecture contains the following information:

- users.
- email encrypted (byte[]) AES-SIV deterministic ciphertext (Google Tink).
- email hmac (String) Hmac SHA256 of lowercased email, used for login lookup.
- phone encrypted (byte[]) AES-SIV deterministic ciphertext, nullable.
- phone hmac (String) Hmac SHA256 of phone, nullable.


Encryption Service holds the Tink keyset in memory (master Key Keyset env var parsed at startup). the decrypt safe operation calls the decrypt operation and logs an error (never throws to the caller) if decryption fails.

**Env vars that control encryption:**

| Env Var | Purpose |
|---|---|
| ENCRYPTION MASTER KEY KEYSET | JSON Tink keyset for AES-SIV. Required at startup; app fails to start if absent. |
| ENCRYPTION keyed message authentication SECRET | Secret for Hmac SHA256 email/phone lookups. Required at startup. |

---

## 7. Email Notification Architecture

Email dispatch is **asynchronous and non-blocking**. The approval service calls the enqueue operation which is annotated @Async. This publishes a message to the Redis Stream key email-notifications. A separate consumer group (Email Consumer Service, in the Email module) reads from this stream and dispatches to Amazon Web Services SES.

| Template ID | Trigger | Recipient |
|---|---|---|
| post approved | the approve post operation | Post owner |
| post rejected | the reject post operation | Post owner |

The email includes a dashboard Url parameter (https://example com/dashboard). No rejection reason is currently included in the email payload — only the dashboard link.

**Env vars that affect email:**

| Env Var | Purpose |
|---|---|
| SPRING DATA REDIS HOST | Redis host for the email stream |
| SPRING DATA REDIS PORT | Redis port (default 6379) |

---

## 8. Audit Log

Every approve/reject writes to the audit log table:

| Column | Value |
|---|---|
| event type | ADMIN POST APPROVE or ADMIN POST REJECT |
| actor id | Admin's user Id from the signed session token principal |
| user id | The post's unique identifier (used as the target entity reference — see note below) |
| details | JSON string: the moderation action, post identifier, post type and rejection reason |
| ip address / user agent | Not populated by the approval service (populated by auth middleware for login events only) |

**Note:** The audit log user id column conventionally holds the user affected by the action. In the approval service, it holds the post unique identifier. This is a known data model tension; do not use audit log user id to look up the post owner's user ID from this module's records.

---

## 9. Frontend Integration

| Page | Route | Purpose |
|---|---|---|
| Review Queue implementation reference | /admin/review | Fetches read operation for admin posts pending via React Query; renders a table of pending posts |
| Review Detail implementation reference | /admin/review/:id | Fetches read operation for admin posts :id; renders post detail with admin personal information panel; triggers approve/reject mutations |

Both pages use @tanstack/react-query with query key factory approval Keys. On approve or reject success, both invalidate approval Keys all and navigate back to /admin/review. Reject reason is composed client-side from a preset selector plus optional free-text notes; the final string is sent as the reason field in the request body.
