# Module 3 — Admin Post Approval: application programming interface Reference

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** PENDING-only review, current moderation/reason constraints and atomic audit/outbox intent govern. Legacy current-code omissions, PAUSED FOR REVIEW/drip-state and raw-database structure definitions examples are historical observations; do not reinstate them over later posting/moderation amendments.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Base path:** admin posts interface operation
**Auth requirement:** All four endpoints require a valid signed session token with ROLE ADMIN. The check is applied at two layers: the pre authorize operation on the controller class, and the Spring Security filter chain.

A non-admin signed session token returns **403 Forbidden**. A missing or expired signed session token returns **401 Unauthorized**.

---

## Endpoint 1 — Get Pending Queue


The illustration for endpoint 1 — get pending queue contains the following information:

- GET admin posts pending interface operation.


Returns all posts (tutor and tutee) currently in PENDING status, merged into a single list and sorted by submitted At descending (most recently submitted first). Nulls (posts with no submitted At) are sorted last.

**No query parameters.** The current implementation returns all pending posts without pagination. The spec defines cursor-based pagination with type, cursor, limit, and sort By parameters — these are not yet implemented.

**Response 200 — Success**


The endpoint 1 — get pending queue example describes information including id, post Type, subjects, level Display, rate Display, submitter Display Name, submitted At, status. Concrete payload values and implementation syntax are omitted.


**Field notes:**

| Field | Source |
|---|---|
| id | tutor posts id or tutee posts id |
| post Type | Hardcoded TUTOR or TUTEE |
| subjects | Junction table tutor post subjects (multi-value) or single tutee posts subject code wrapped in a list |
| level Display | tutor post levels joined with  or tutee posts level code |
| rate Display | min–max SGD/hr from rate/budget columns |
| submitter Display Name | tutor profiles display name or tutee profiles display name; falls back to Tutor / Tutee if profile not found |
| submitted At | submitted at from the post table, converted to Instant |
| status | Always PENDING for posts in this queue |

**No personal information is returned in this response.** Email and phone are absent.

**Error responses:**

| Status | Condition |
|---|---|
| 401 Unauthorized | Missing or expired signed session token |
| 403 Forbidden | signed session token present but user does not hold ROLE ADMIN |

---

## Endpoint 2 — Get Post Detail


Implementation example omitted. This public edition retains the written requirements in this section.


Returns the full post record for admin review, including decrypted contact information (personal information). The service tries tutor posts first, then tutee posts. Post type determines which fields are populated.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| id | unique identifier | Post ID (from the queue list) |

**Response 200 — Tutor Post**


The endpoint 2 — get post detail example describes information including id, post Type, subjects, level Display, rate Display, submitter Display Name, submitted At, status, submitter Email, submitter Phone, rejection Reason, title, headline, qualifications, teaching Approach, levels, hourly Rate Min Sgd, hourly Rate Max Sgd. Additional nested details follow the surrounding requirements. Concrete payload values and implementation syntax are omitted.


**Response 200 — Tutee Post**


The endpoint 2 — get post detail example describes information including id, post Type, subjects, level Display, rate Display, submitter Display Name, submitted At, status, submitter Email, submitter Phone, rejection Reason, title, headline, qualifications, teaching Approach, levels, hourly Rate Min Sgd, hourly Rate Max Sgd. Additional nested details follow the surrounding requirements. Concrete payload values and implementation syntax are omitted.


**Post-type field mapping:**

| Field group | TUTOR post | TUTEE post |
|---|---|---|
| title, headline, qualifications, teaching Approach | Populated | no value |
| levels, hourly Rate Min Sgd, hourly Rate Max Sgd | Populated | no value |
| venue Types, group Sizes, years Experience | Populated | no value |
| availability Bitmask, location Mrt Station, location Planning Area | Populated | no value (tutee fields not mapped here) |
| subject Code, level Code, budget Min Sgd, budget Max Sgd | no value | Populated |
| num Students, venue Preferences | no value | Populated |
| frequency Per Week, session Duration Hours, preferred Start Date | no value | Populated |
| special Requirements | no value | Populated (stored as plaintext) |

**Error responses:**

| Status | Code | Condition |
|---|---|---|
| 404 Not Found | NOT FOUND | No post with that ID exists in either tutor posts or tutee posts |
| 401 Unauthorized | — | Missing or expired signed session token |
| 403 Forbidden | — | Not an admin |

**Status note (INT-137b, 2026-06-13):** NOT FOUND now correctly maps to **web request protocol 404** — the service uses the not found operation across Admin Post Service, Tutee Post Service, and Tutor Post Service (the earlier 400 was an implementation mismatch, now resolved repo-wide). The body code stays NOT FOUND.

---

## Endpoint 3 — Approve Post


Implementation example omitted. This public edition retains the written requirements in this section.


Sets the post status to ACTIVE, records the review timestamp, writes an audit log entry, and queues an approval notification email to the post owner.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| id | unique identifier | Post ID |

**Request body:** None required. The admin identity is read from the signed session token principal (@Authentication Principal Authenticated User).

**Response 200 — Success**

Returns the full Admin Post Detail Response (same shape as Endpoint 2) with status updated to ACTIVE.


The endpoint 3 — approve post example describes information including id, post Type, status, submitter Email. Concrete payload values and implementation syntax are omitted.


**Side effects on success:**

1. tutor posts status or tutee posts status → ACTIVE
2. approved at → the now operation
3. reviewed at → the now operation
4. audit log append records (event type ADMIN POST APPROVE)
5. Redis Stream email-notifications XADD (template post approved)

**Error responses:**

| Status | Code | Condition |
|---|---|---|
| 404 Not Found | NOT FOUND | Post not found |
| 401 Unauthorized | — | Missing or expired signed session token |
| 403 Forbidden | — | Not an admin |

**No 409 guard:** The current implementation does not check if the post is already ACTIVE before updating. Calling this endpoint on an already-approved post will silently re-set approved at and reviewed at without error, and queue a second approval email. This matches the observation in the codebase — the spec's 409 guard is not yet implemented.

---

## Endpoint 4 — Reject Post


Implementation example omitted. This public edition retains the written requirements in this section.


Sets the post status to REJECTED with a mandatory rejection reason, records the review timestamp, writes an audit log entry, and queues a rejection notification email to the post owner.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| id | unique identifier | Post ID |

**Request body:**


The endpoint 4 — reject post example describes information including reason. Concrete payload values and implementation syntax are omitted.


| Field | Type | Required | Constraint |
|---|---|---|---|
| reason | String | Yes | @Not Blank — must be non-null, non-empty, non-whitespace |

The reason string is a free-text field in the current implementation. The frontend sends either a preset string or a combined "preset — notes" string. The database column rejection reason is the varchar operation. If the composed reason exceeds 50 characters, the database will truncate or error depending on PostgreSQL mode. The spec defines a controlled enum of 7 values for post review log rejection reason, but the service layer does not currently enforce enum validation for the tutor posts rejection reason column.

**Response 200 — Success**

Returns the full Admin Post Detail Response with status: REJECTED and rejection Reason populated.


The endpoint 4 — reject post example describes information including id, post Type, status, rejection Reason, submitter Email. Concrete payload values and implementation syntax are omitted.


**Side effects on success:**

1. tutor posts status or tutee posts status → REJECTED
2. rejection reason → reason string
3. reviewed at → the now operation
4. audit log append records (event type ADMIN POST REJECT, includes rejection Reason in details JSON)
5. Redis Stream email-notifications XADD (template post rejected)

**Error responses:**

| Status | Code | Condition |
|---|---|---|
| 400 Bad Request | (validation) | reason is blank or missing — Spring returns a 400 with Method Argument Not Valid Exception detail |
| 404 Not Found | NOT FOUND | Post not found |
| 401 Unauthorized | — | Missing or expired signed session token |
| 403 Forbidden | — | Not an admin |

**Validation error body example (blank reason):**


The endpoint 4 — reject post example describes information including timestamp, status, errors, field, message. Concrete payload values and implementation syntax are omitted.


The exact error body shape depends on the global exception handler configuration. Check Global error handler implementation reference (or equivalent) for the actual response structure.

---

## Common web request protocol Error Shapes

All App Exception instances are serialised by the global exception handler. The expected shape is:


The common web request protocol error shapes example describes information including code, message. Concrete payload values and implementation syntax are omitted.


| web request protocol Status | Meaning |
|---|---|
| 400 | Bad request — validation errors (e g. blank reason).  using INT-137b: NOT FOUND is now 404, not 400._ |
| 404 | Not found — NOT FOUND from the approval/post services (post does not exist) |
| 401 | Unauthenticated |
| 403 | Forbidden (wrong role) |
| 409 | Conflict (not currently returned by this module — reserved for future idempotency guard) |
| 500 | Unhandled server error |
