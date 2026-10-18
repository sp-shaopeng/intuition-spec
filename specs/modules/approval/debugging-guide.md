# Module 3 — Admin Post Approval: Developer Debugging Guide

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Use current plaintext user email/phone fields, authorized reads and the current Liquibase ledger, not the encrypted/keyed message authentication database query language below. Diagnose business transaction→durable email outbox→synchronous stream handoff→consumer/retry/failed-message queue under [INT-087](../../architecture/int-087-transactional-messaging-hardening-spec.md). A user interface toast or queue row is not proof of recipient delivery. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


This guide is for developers diagnosing approval-related issues in production. All database query language examples assume a the documented operational check session with app role or a DBA role.

---

## 1. Check a Post's Current Status

You do not know ahead of time whether a post ID belongs to a tutor post or a tutee post — query both tables.


Implementation example omitted. This public edition retains the written requirements in this section.


**Interpreting status:**

| Status | Expected Behaviour |
|---|---|
| DRAFT | Not submitted yet. Will not appear in the admin queue. |
| PENDING | In the queue. Admin has not acted yet. |
| ACTIVE | Approved. Publicly visible. |
| REJECTED | Rejected. rejection reason should be non-null. |
| REVISION REQUESTED | Admin requested changes — not yet implemented in the service layer. |
| MATCHED | Tutee post matched. Outside the scope of this module. |
| CLOSED / EXPIRED | Terminal. Not in approval queue. |

**Post not appearing in admin queue despite being submitted:**
Check if status PENDING. If status DRAFT, the user did not complete the submit step and submitted at will be no value. If status is anything other than PENDING, it has already been reviewed or is in a non-reviewable state.

---

## 2. Log Patterns for Approval/Rejection Issues

All approval logs use the Admin Post Service logger. In production, filter logs by class name:


The illustration for . log patterns for approval/rejection issues contains the following information:

- logger: com intuition service Admin Post Service.


### 2.1 Successful Approve


The illustration for successful approve contains the following information:

- DEBUG c i service Admin Post Service - (no explicit debug log on success — check audit log instead).


On success there is no explicit INFO log from the service. Confirm success by querying audit log (see §3) or checking the post status in the database.

### 2.2 Decryption Failed


The illustration for decryption failed contains the following information:

- ERROR c i service Admin Post Service - Failed to decrypt field.
- com google crypto tink General Security Exception: ciphertext too short.


This appears when the decrypt safe operation fails. The field returns no value in the response instead of the plaintext. Causes:
- The ENCRYPTION MASTER KEY KEYSET env var is wrong or was rotated without re-encrypting existing data.
- The email encrypted column value is corrupted or was written with a different key version.

### 2.3 Audit Log Write Failed


The illustration for audit log write failed contains the following information:

- ERROR c i service Admin Post Service - Failed to write audit log for action is APPROVE post Id=<uuid>.
- org springframework dao Data Integrity Violation Exception: .


The audit log failure does **not** roll back the approval/rejection. The post status change has already been committed. You must manually insert the missing audit log entry if compliance requires it.

### 2.4 Email Enqueue Failed


The illustration for email enqueue failed contains the following information:

- ERROR c i infrastructure Email Queue Service - Failed to enqueue email template is post approved to=[REDACTED EMAIL].
- org springframework data redis Redis Connection Failure Exception: .


Redis is unreachable. The email is **lost** — there is no retry mechanism in the current implementation. The approval itself is committed. You must manually trigger the notification or re-queue it.

### 2.5 Email Consumer Failed (Email Module Logs)


The illustration for email consumer failed email module logs contains the following information:

- ERROR c i email Email Consumer Service - .


If the enqueue succeeded but the user did not receive an email, check the Email Consumer Service logs for SES errors or dead-letter queue entries.

---

## 3. Verify the Audit Log Was Written


Implementation example omitted. This public edition retains the written requirements in this section.


**Expected row for an approval:**


The . verify the audit log was written example describes information including event type, actor id, user id, details. Concrete payload values and implementation syntax are omitted.


**Expected row for a rejection:**


The . verify the audit log was written example describes information including event type, actor id, user id, details. Concrete payload values and implementation syntax are omitted.


**Important:** audit log user id holds the **post unique identifier**, not the post owner's user unique identifier. Do not use this column to look up the owner.

To find the admin who performed the action:


Implementation example omitted. This public edition retains the written requirements in this section.


Note: admin email is stored encrypted — you cannot read it directly from this query. Use the keyed message authentication for lookup or find the admin account in the admin user list.

---

## 4. Check Whether the Notification Email Was Queued

### 4.1 Using Redis CLI


Implementation example omitted. This public edition retains the written requirements in this section.


Each stream entry looks like:


The illustration for using redis cli contains the following information:

- template Id post approved.
- to [REDACTED EMAIL].
- dashboard Url https://example com/dashboard.
- enqueued At 1748390400123.


### 4.2 If the Email Was Not Queued

If you cannot find the entry in the stream and the ERROR Email Queue Service log is absent, confirm:

1. The owner's email encrypted column is not null. A null value causes the decrypt safe operation to return no value, and the queue approval email operation returns early without enqueuing.


Implementation example omitted. This public edition retains the written requirements in this section.


2. The @Async thread pool executor is not saturated. Check JVM thread dumps if emails are consistently missing under high load.

---

## 5. Contact Info Not Decrypting

**Symptom:** read operation for admin posts id returns submitter Email: no value or submitter Phone: no value.

**Step 1 — Check if the column is populated:**


Implementation example omitted. This public edition retains the written requirements in this section.


If email set no, the user was registered without an email (should not happen — email encrypted is required on the schema). If so, the user record is corrupt.

**Step 2 — Verify the encryption key:**

The Encryption Service loads the Tink keyset from ENCRYPTION MASTER KEY KEYSET at startup (via @Post Construct). If this env var is wrong, the service fails to start. However, if the key was rotated after some records were written, old records encrypted with the old key will fail to decrypt with the new key.


The illustration for . contact info not decrypting contains the following information:

- ERROR c i service Admin Post Service - Failed to decrypt field.
- General Security Exception: decryption failed.


Check the startup log for:


The illustration for . contact info not decrypting contains the following information:

- INFO c i infrastructure Encryption Service - (no explicit log — absence of startup error is the signal).


If the app started without error, the keyset parsed correctly. If decryption still fails, the stored ciphertext was written with a different key version.

**Step 3 — Test decryption in isolation:**

There is no admin endpoint to test decryption. To debug, you would need to write a test using Encryption Service directly, passing the raw bytes from the database.

---

## 6. Common Issues and Root Causes

| Symptom | Most Likely Cause | Fix |
|---|---|---|
| Post not in review queue | status differs from PENDING — not submitted or already reviewed | Check post status in database (§1) |
| Post in queue but frontend shows 0 items | the queue operation React Query stale; or application programming interface returning empty array | Check browser network tab; check read operation for pending response |
| Approve returns 404 NOT FOUND | Post unique identifier is wrong, or post was deleted | Confirm unique identifier from URL matches database |
| Approve silently double-approves | No 409 guard in current impl | Known gap; check spec §BR-Approval-5 in module-overview md |
| Email not delivered but no error | Redis stream enqueued but consumer failed | Check Email Consumer Service logs and SES delivery status |
| Email not delivered, enqueue error | Redis unavailable at time of approval | Check SPRING DATA REDIS HOST and Redis health; re-queue manually |
| submitter Email = null in detail response | Decryption failed OR email encrypted is null | See §5 |
| submitter Phone = null | Phone not collected at registration (nullable field) | Normal — phone is optional |
| Audit log missing | Audit Log Repository save operation threw an exception during the action | Check ERROR logs around the action timestamp (§2.3) |
| Rejection reason truncated | Reason string > 50 chars (VARCHAR using 50 limit) | Frontend presets are short; custom "Other" notes can exceed limit |

---

## 7. Environment Variables Affecting the Approval Flow

| Variable | Module | Effect if Wrong or Missing |
|---|---|---|
| ENCRYPTION MASTER KEY KEYSET | Encryption Service | App fails to start (Post Construct throws) |
| ENCRYPTION keyed message authentication SECRET | Encryption Service | keyed message authentication lookups fail; login broken; approval not directly affected |
| SPRING DATA REDIS HOST | Email Queue Service | Email enqueue fails silently; approval still commits |
| SPRING DATA REDIS PORT | Email Queue Service | Same as above |
| SPRING DATASOURCE URL | All database operations | App fails to start |
| signed session token SECRET (or equivalent) | Spring Security | Admin auth fails; all 4 endpoints return 401 |

---

## 8. Quick Production Checklist

When a developer reports "the approval is broken", run through this sequence:

1. Can the admin log in and get a valid signed session token? (Auth module issue if not)
2. Does read operation for admin posts pending return 200? (Network / auth issue if not)
3. Is the specific post in PENDING status in the database? (Submission issue if not)
4. Does read operation for admin posts id return the post with email populated? (Encryption issue if email is null)
5. After approve: is status ACTIVE in the database? (database write issue if not)
6. After approve: is there an ADMIN POST APPROVE row in audit log? (Audit issue if not — post still approved)
7. After approve: is there a matching entry in the Redis stream email-notifications? (Email queue issue if not)
8. Did the owner receive the email? (SES / Email Consumer Service issue if steps 1-7 all passed)
