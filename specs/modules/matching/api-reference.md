# Module 4 — application programming interface Reference

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Public tutor identity is handle-only. Canonical admin assignment accepts tutor User Id plus tutee Post Id and an optional listing under INT034; the remaining sample responses/old Bearer-header examples are historical projections, not current application programming interface/cookie authority.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

---

## Overview

The Matching module consumes three endpoints:

| Method | Path | Auth required | Purpose |
|---|---|---|---|
| GET | tutors interface operation | No | Load tutor post list for the left column |
| GET | requests interface operation | No | Load tutee request list for the right column |
| POST | admin matches interface operation | Yes — ADMIN role | Create a match between one tutor post and one tutee post |

The two GET endpoints are **public** — no Authorization header is required.  The POST endpoint requires a valid session signed session token and the caller's role must be ADMIN.

---

## GET tutors interface operation

Retrieves a paginated list of active tutor posts.  Used by Admin Matching to populate the tutor column.

### Query Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| subject | string | No | — | Filter to posts whose subjects array contains this value (e g. H2 Chemistry). Omit to return all subjects. |
| level | string | No | — | Filter to posts whose levels array contains this value (e g. JC1). Omit to return all levels. |
| page | integer | No | 0 | Zero-based page number. |
| size | integer | No | 20 | Page size. The matching page always sends size is 100. |

### Response — 200 OK

Returns a page of tutor post records, with pagination details.  All fields are safe for public display — no personal information is exposed.


The response — 200 ok example describes information including data, id, tutor Public Handle, title, headline, subjects, levels, hourly Rate Min Sgd, hourly Rate Max Sgd, venue Types, group Sizes, availability Bitmask, qualifications, years Experience, teaching Approach, location Mrt Station, location Planning Area, location Region. Additional nested details follow the surrounding requirements. Concrete payload values and implementation syntax are omitted.


### Field Notes

- availability Bitmask — legacy 21-bit integer, day × 3 periods (bit = day Index*3+period Index); fine tutor availability is represented by spec002 slots.  Use the format availability short operation to display.
- status — The public endpoint is expected to return only ACTIVE posts.  Non-active posts should be filtered server-side.
- subjects / levels — Free-text arrays sourced from the tutor's original submission.  Display in the user interface using the join operation.

---

## GET requests interface operation

Retrieves a paginated list of active tutee requests.  Used by Admin Matching to populate the tutee column.  All fields are **anonymised** — no tutee personal information (contact details, full name) is included.

### Query Parameters

Same shape as tutors interface operation:

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| subject | string | No | — | Filter by subject code (e g. H2 Chemistry). |
| level | string | No | — | Filter by level code (e g. JC1). |
| page | integer | No | 0 | Zero-based page number. |
| size | integer | No | 20 | Page size. The matching page always sends size is 100. |

### Response — 200 OK

Returns a page of tutee post records, with pagination details.


The response — 200 ok example describes information including data, id, subject Code, level Code, num Students, budget Min Sgd, budget Max Sgd, venue Preferences, availability Bitmask, session Duration Hours, frequency Per Week, preferred Start Date, location Mrt Station, location Planning Area, location Region, status, created At, meta. Additional nested details follow the surrounding requirements. Concrete payload values and implementation syntax are omitted.


### Field Notes

- subject Code / level Code — Single-value strings (unlike tutor posts which use arrays).
- venue Preferences — Array of Venue Type enum values: STUDENT HOME, TUTOR HOME, PUBLIC PLACE, ONLINE, TUITION CENTRE.
- frequency Per Week — Enum: ONE, TWO, THREE, FLEXIBLE.
- availability Bitmask — Same 7-bit encoding as tutor posts.

---

## POST admin matches interface operation

Creates a match between a tutor post and a tutee post.  This is the core write operation of Module 4.

### Authentication

**Required.**  The request must include a valid session signed session token.  The authenticated user's role must be ADMIN.


The illustration for authentication contains the following information:

- Authorization: Bearer <jwt>.


### Request Body

Content-Type: application/json

| Field | Type | Required | Description |
|---|---|---|---|
| tutor User Id | string unique identifier | Yes on canonical free-market path | Tutor account/profile identity. |
| tutor Post Id | string unique identifier | No | Optional attached listing; verify it belongs to tutor User Id. |
| tutee Post Id | string unique identifier | Yes | ID of the tutee request to match. |


The request body example describes information including tutor Post Id, tutee Post Id. Concrete payload values and implementation syntax are omitted.


### Response — 201 Created

The newly created Match object.


The response — 201 created example describes information including id, tutor Post Id, tutee Post Id, status, matched At. Concrete payload values and implementation syntax are omitted.


status begins as PENDING.  Downstream processing (commission tracking, interview scheduling) may advance it to ACTIVE, COMPLETED, or CANCELLED.

### Error Responses

| web request protocol Status | Condition | Notes |
|---|---|---|
| 400 Bad Request | Missing or malformed request body (e g. non-unique identifier value for tutor Post Id). | Fix the payload. |
| 401 Unauthorized | No signed session token supplied, or signed session token is expired/invalid. | Re-authenticate via submit operation for auth login. |
| 403 Forbidden | signed session token is valid but the user's role is not ADMIN. | Do not call this endpoint from non-admin sessions. |
| 404 Not Found | tutor Post Id or tutee Post Id does not exist in the database. | Verify the IDs from the GET list responses. |
| 409 Conflict | A match with this exact tutor Post Id, tutee Post Id pair already exists. | The unique constraint on the matches operation was violated.  The pairing has already been made. |
| 422 Unprocessable Entity | The referenced tutor post or tutee post does not have status ACTIVE. | A post must be approved (Module 3) before it can be matched.  Check the post's current status via the admin post detail endpoint. |

### Error Response Body (example — 409 Conflict)


The error response body example — 409 conflict example describes information including error, message. Concrete payload values and implementation syntax are omitted.


### Error Response Body (example — 422 Unprocessable Entity)


The error response body example — 422 unprocessable entity example describes information including error, message. Concrete payload values and implementation syntax are omitted.


---

## Page Response Envelope

Both GET endpoints wrap their results in a common envelope:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Tan Stack Query Key Conventions

The frontend uses factory key functions from query Keys implementation reference:

| Key factory call | Covers |
|---|---|
| the active-tutor list for the selected filters | Tutor list query — one cache entry per unique filter combination |
| the active tuition-request list for the selected filters | Tutee list query — one cache entry per unique filter combination |
| the lists operation | All match list queries — invalidated on successful match creation |

Changing a filter dropdown updates the query key, causing Tan Stack Query to treat it as a cache miss and re-fetch from the application programming interface.
