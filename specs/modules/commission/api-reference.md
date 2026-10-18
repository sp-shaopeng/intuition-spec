# Module 5: Admin Commission Tracking — application programming interface Reference

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The auth, activation and receipt clauses below are corrected. Use [INT-157 v2](../../architecture/int-157-commission-first-month-locked-spec.md) for canonical agreed-plan inputs/immutable whole-SGD amounts. The monthly Fee-only request is a legacy compatibility example. Retain authenticated tutee first-session confirmation under [product requirements document §8.3/§20.3](../../PRD.md); this abbreviated admin reference does not replace that workflow.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

These admin endpoints require an authenticated ROLE ADMIN session. Browser authentication uses the
HttpOnly auth/refresh-cookie contract in spec 004 and cross-site request forgery protection protection for writes. Do not require JavaScript
to read the auth cookie or manufacture a Bearer header; other supported authenticated clients follow the application programming interface contract.

Base URL: api v1 interface operation

---

## GET admin matches interface operation

Returns all matches sorted by matched At descending. Each item optionally includes its associated commission.

### Auth

ROLE ADMIN required.

### Request

No request body. No required query parameters.

The frontend sends page is 0&size is 100 as query parameters (Commission implementation reference line 36) but the backend controller ignores pagination parameters on this endpoint — it calls the find all by order by matched at desc operation which returns all rows.

### Success Response

**web request protocol 200 OK**


The success response example describes information including id, tutor Subject, tutee Subject, status, matched At, commission, monthly Fee Sgd, commission Rate, commission Amount Sgd, invoice Status, paid At. Concrete payload values and implementation syntax are omitted.


The commission field is no value for matches that have not been activated (PENDING or CANCELLED without activation). tutor Subject is the first element of the tutor post's subject Codes list; tutee Subject is the tutee post's subject Code.

### Error Codes

| Status | When                                                 |
|--------|------------------------------------------------------|
| 401    | No or invalid signed session token                                    |
| 403    | Authenticated user does not have ROLE ADMIN          |

---

## POST admin matches interface operation

Creates a new match between an ACTIVE tutor post and an ACTIVE tutee post. Introduced in Module 4; included here for completeness.

### Auth

ROLE ADMIN required.

### Request Body

Content-Type: application/json


The request body example describes information including tutor Post Id, tutee Post Id. Concrete payload values and implementation syntax are omitted.


| Field         | Type   | Constraints                    |
|---------------|--------|--------------------------------|
| tutor Post Id   | unique identifier   | @Not no value — required          |
| tutee Post Id   | unique identifier   | @Not no value — required          |

### Success Response

**web request protocol 201 Created**


The success response example describes information including id, tutor Post Id, tutee Post Id, status, matched At. Concrete payload values and implementation syntax are omitted.


Note: this endpoint returns Match Response (flat structure), not Admin Match Item Response. The commission field is absent because a newly created match is always PENDING and has no commission yet.

### Error Codes

| Status | Error Code            | When                                                              |
|--------|-----------------------|-------------------------------------------------------------------|
| 400    | VALIDATION ERROR      | Missing tutor Post Id or tutee Post Id                           |
| 401    | —                     | No or invalid signed session token                                                 |
| 403    | —                     | Authenticated user does not have ROLE ADMIN                      |
| 404    | TUTOR POST NOT FOUND  | No tutor post exists with the given ID                           |
| 404    | TUTEE POST NOT FOUND  | No tutee post exists with the given ID                           |
| 409    | DUPLICATE MATCH       | A PENDING or ACTIVE match already exists for this pair           |
| 422    | TUTOR POST NOT ACTIVE | Tutor post exists but its status is not ACTIVE                   |
| 422    | TUTEE POST NOT ACTIVE | Tutee post exists but its status is not ACTIVE                   |

---

## POST admin matches id activate interface operation

Activates a PENDING match by recording the agreed first-month plan, computing and locking the agency commission,
and transitioning the match to ACTIVE. The canonical agreed-plan inputs and integer-SGD snapshot are defined by
INT-157 v2. The monthly Fee-only body below is a retained legacy example, not the canonical new-flow request.

### Auth

ROLE ADMIN required.

### Path Parameters

| Parameter | Type | Description               |
|-----------|------|---------------------------|
| id        | unique identifier | The match ID to activate  |

### Request Body

Content-Type: application/json


The request body example describes information including monthly Fee Sgd. Concrete payload values and implementation syntax are omitted.


| Field         | Type    | Constraints                                   |
|---------------|---------|-----------------------------------------------|
| monthly Fee Sgd | Integer | Required; positive whole Singapore-dollar amount; no more than S$50,000        |

Bean Validation is applied by @Valid on the controller parameter. Constraint violations return 400 before the service layer is reached.

### Success Response

**web request protocol 200 OK**


The success response example describes information including id, tutor Subject, tutee Subject, status, matched At, commission, monthly Fee Sgd, commission Rate, commission Amount Sgd, invoice Status, paid At. Concrete payload values and implementation syntax are omitted.


### Error Codes

| Status | Error Code          | When                                                    |
|--------|---------------------|---------------------------------------------------------|
| 400    | VALIDATION ERROR    | monthly Fee Sgd is null, zero, negative, or > 50000    |
| 401    | —                   | No or invalid signed session token                                       |
| 403    | —                   | Authenticated user does not have ROLE ADMIN             |
| 404    | MATCH NOT FOUND     | No match exists with the given id                     |
| 422    | MATCH NOT PENDING   | Match exists but its status is not PENDING              |

The 422 error message includes the current status: Match must be PENDING to activate current status: ACTIVE . This is returned as the message field in the standard App Exception error response body.

---

## POST admin matches id cancel interface operation

Cancels a PENDING or ACTIVE match. Terminal matches (COMPLETED, CANCELLED) cannot be cancelled.

### Auth

ROLE ADMIN required.

### Path Parameters

| Parameter | Type | Description              |
|-----------|------|--------------------------|
| id        | unique identifier | The match ID to cancel   |

### Request Body

None required. The frontend sends an empty body an empty response object (sending the request without application data in its body).

### Success Response

**web request protocol 200 OK**


The success response example describes information including id, tutor Subject, tutee Subject, status, matched At, commission. Concrete payload values and implementation syntax are omitted.


If the cancelled match was ACTIVE and had a commission, commission is present in the response with invoice Status unchanged (the commission record itself is not modified by cancellation).

### Error Codes

| Status | Error Code               | When                                                              |
|--------|--------------------------|-------------------------------------------------------------------|
| 401    | —                        | No or invalid signed session token                                                 |
| 403    | —                        | Authenticated user does not have ROLE ADMIN                      |
| 404    | MATCH NOT FOUND          | No match exists with the given id                              |
| 422    | MATCH NOT CANCELLABLE    | Match status is COMPLETED or CANCELLED                           |

---

## POST admin commissions id pay interface operation

Records that the agency actually received the tutor commission, marks it PAID and sets the associated match
to COMPLETED atomically. It does not record parent lesson payment, an amount merely owed, or a dispute ruling.
Require the amount/identity/receipt confirmation from specs 017/019 and preserve first-session obligations.

### Auth

ROLE ADMIN required.

### Path Parameters

| Parameter | Type | Description                 |
|-----------|------|-----------------------------|
| id        | unique identifier | The commission ID to pay    |

### Request Body

None required. The frontend sends an empty body an empty response object (sending the request without application data in its body).

### Success Response

**web request protocol 200 OK**

Returns the updated Admin Commission Item Response (commission only, not the full match wrapper):


The success response example describes information including id, monthly Fee Sgd, commission Rate, commission Amount Sgd, invoice Status, paid At. Concrete payload values and implementation syntax are omitted.


After receiving this response, Commission implementation reference calls refreshing the cached match lists to refresh the full match list (which now reflects match status COMPLETED).

### Error Codes

| Status | Error Code               | When                                                   |
|--------|--------------------------|--------------------------------------------------------|
| 401    | —                        | No or invalid signed session token                                      |
| 403    | —                        | Authenticated user does not have ROLE ADMIN            |
| 404    | COMMISSION NOT FOUND     | No commission exists with the given id               |
| 409    | COMMISSION ALREADY PAID  | Commission invoice Status is already PAID             |

---

## Standard Error Response Shape

All error responses from App Exception use a consistent body structure:


The standard error response shape example describes information including status, error Code, message. Concrete payload values and implementation syntax are omitted.


Bean Validation failures (400) use Spring's default Method Argument Not Valid Exception response format and include a field Errors array.
