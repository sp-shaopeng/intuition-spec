# InTuition — application programming interface Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Use the current page envelope, 30-minute user-session/refresh contract and authenticated detail routes. Endpoint/code anchors copied from older snapshots do not override numbered contracts; the maintained endpoint inventory is reference/backend-api md. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.

**Document Version:** 1.0
**Date:** 2026-05-27
**Status:** Draft — Auth complete; other modules to be added per module completion
**Platform:** InTuition
**Tech Stack:** Java 21 + Spring Boot 3.x

---

## Overview

All endpoints are served from example com/api/v1/** (same-origin production routing per spec 027; the Railway topology is historical). There is no separate application programming interface domain; the SPA and application programming interface share the same origin.

**Base path:** api v1 interface operation
**Content-Type:** application/json on all endpoints
**Auth:** http Only cookie auth=<jwt> (HS256, 30-minute TUTEE/TUTOR access cookie plus spec 004 refresh; ADMIN lifecycle separately scoped)
**cross-origin request permissions:** Not required — same-origin via Cloudflare proxy

---

## Conventions

### Authentication

All endpoints except those marked public require the auth cookie. The Jwt Authentication Filter validates the cookie on every request. Role requirements:

| Marker | Meaning |
|---|---|
| public | No authentication required |
| TUTEE | Authenticated, role = TUTEE |
| TUTOR | Authenticated, role = TUTOR |
| ADMIN | Authenticated, role = ADMIN, multi-factor authentication verified, session active |
| any auth | Authenticated, any role |

### Error Response Format


The error response format example describes information including error, code, message, retry After. Concrete payload values and implementation syntax are omitted.


retry After (seconds) is only present on 429/423 responses.

Since **spec 010** every error body also carries a top-level trace Id and the trace identifier in the error response — see Tracing below.

### Tracing (spec 010)

Every request carries a per-request **trace id**, surfaced consistently so a reported failure can be tied to its log line:

- **X-Correlation-ID response header** — present on **every** response (success, error, 204, redirect, binary); set by request correlation filter and **cross-origin request permissions-exposed** so the SPA can read it. This is the universal carrier.
- **trace Id in every JSON-object body** — added additively (one top-level field; no existing field touched). Success objects and pagination wrappers get a top-level trace Id; **error** bodies get it inside the error object **and** as a top-level field.
- Bodies that are not a JSON object (top-level array, primitive, empty/204, binary, RFC-7807 Problem Detail) carry the id on the **header only**.
- The reserved response trace Id represents this request’s correlation identifier; apply spec 010’s reconciled collision/error-body rules. A pre-existing unrelated application field must not silently masquerade as the tracing carrier.
- **Known gap:** security **401/403** entry-point responses carry the **header** but no body trace Id (they bypass the global exception handler).

### Pagination

Zero-based page and size, using the retained Page Response contract (ADR-001):


The pagination example describes information including data, meta, trace Id. Concrete payload values and implementation syntax are omitted.


Public filtering is available without login. Anonymous size is capped at 20 and pages beyond 4 are empty; oversized valid sizes are clamped, not converted into an empty first page. Detail routes require authentication. Use the endpoint’s documented server maximum for authenticated requests.

---

## Module: Auth (auth interface operation)

Full specification: modules/auth/backend md

| Method | Path | Auth | Summary |
|---|---|---|---|
| POST | /auth/register/tutee | public | Register tutee account |
| POST | /auth/register/tutor | public | Register tutor account (commission ack required) |
| POST | /auth/login | public | Email/password login |
| POST | /auth/login/mfa | mfa session cookie | Admin time-based one-time passcode multi-factor authentication verification |
| POST | /auth/logout | any auth | Logout + invalidate session/jti |
| GET | /auth/me | any auth | Get current user (authorized own contact info; user email/phone storage follows the plaintext amendment) |
| POST | /auth/forgot-password | public | Request password reset one-time passcode |
| POST | /auth/reset-password | public | Submit one-time passcode + new password |
| DELETE | /auth/account | any auth | Personal Data Protection Act erasure — soft delete account |

---

## Module: Account (account interface operation)

Full specification: reference/backend-api md (Account section). Account operations are scoped to the authenticated owning user with cross-site request forgery protection protection on mutations, except the purpose-bound revoke credential described below, which grants cancellation of its one request only.

| Method | Path | Auth | Summary |
|---|---|---|---|
| POST | /account/phone/verify/start | any auth | INT-082 — mint + send WhatsApp one-time passcode (inert an indication of whether phone verification is enabled (disabled in this case) while OFF) |
| POST | /account/phone/verify/confirm | any auth | INT-082 — confirm the 6-digit one-time passcode |
| PUT | /account/phone | legacy | Historical INT-082 direct-change route, superseded by 012. It must not bypass step-up and email-link confirmation; use the contact-change workflow. |
| PUT | /account/whatsapp-consent | any auth | INT-082 — set WhatsApp notification opt-in |
| POST | /account/contact-change | any auth | **spec 012** — initiate a self-edit of email/phone. Body contains the contact field to change, its proposed new value and the current password for identity confirmation. Step-up re-auth required (wrong password → 422 STEP UP REQUIRED); emails a confirm link; live value unchanged until confirmed |
| GET | /account/contact-change | any auth | **spec 012** — current PENDING contact changes (personal information-masked) for the "verification pending" banner |
| POST | /account/contact-change/verify | authenticated owner + cross-site request forgery protection | **spec 012** — confirm a pending change with the request identifier and its confirmation credential from the email link. **POST-only** (anti email-scanner-prefetch). 400 CONTACT CHANGE TOKEN INVALID; uniqueness re-checked at commit → 409 EMAIL ALREADY REGISTERED / PHONE ALREADY REGISTERED. Original migration **0066** (contact change request); new revoke fields/state use an additive versioned changeset. |
| POST | /account/contact-change/revoke | authenticated owner + cross-site request forgery protection OR valid dedicated revoke credential | **012** — lock the same request row as confirmation; PENDING → REVOKED invalidates confirmation, keeps the current contact and records an audit event. Owner body the request identifier, warning-link body the request identifier and its revocation credential; success and valid repeat → 204 with no body. CONSUMED → 409 with recovery guidance. GET links render only. Revoke credentials are independent, hashed, purpose-bound and expire after 30 minutes. |

---

## Module: Posting (api v1 interface operation)

Full specification: modules/posting/backend md

| Method | Path | Auth | Summary |
|---|---|---|---|
| POST | /tutee/posts | TUTEE | Create tutee post (request for tuition). num Students bounded 1–20 (over → 400 VALIDATION ERROR); unknown code **or incoherent subject+level pair** → 422 UNKNOWN SUBJECT OR LEVEL (spec 009 + spec 020 pair-aware; subject/level must share level group, plus the N LEVEL → O LEVEL widen) |
| GET | /tutee/posts | TUTEE | List own tutee posts |
| GET | /tutee/posts/:id | TUTEE (own) | Get tutee post detail |
| PUT | /tutee/posts/:id | TUTEE (own) | Update tutee post. **spec 021:** editable while **PENDING** → 200, stays PENDING and re-enters approval; ACTIVE edit re-queues to PENDING. Same pair-aware taxonomy validation as create |
| DELETE | /tutee/posts/:id | TUTEE (own) | Delete/withdraw tutee post |
| POST | /tutor/posts | TUTOR | Create tutor post (offering) |
| GET | /tutor/posts | TUTOR | List own tutor posts |
| GET | /tutor/posts/:id | TUTOR (own) | Get tutor post detail |
| PUT | /tutor/posts/:id | TUTOR (own) | Update tutor post |
| DELETE | /tutor/posts/:id | TUTOR (own) | Delete/withdraw tutor post |
| GET | /public/tutors | public | Browse approved tutor posts (SEO listing) |
| GET | /tutors/:id | authenticated | Get approved tutor detail by id; preserve login return To |
| GET | /public/assignments | public | Browse bounded, filterable, anonymized approved requests |

---

## Module: Approval (admin interface operation)

Full specification: modules/approval/backend md

| Method | Path | Auth | Summary |
|---|---|---|---|
| GET | /admin/posts | ADMIN | Admin post review queue |
| GET | /admin/posts/:id | ADMIN | Get post detail (incl. contact info — access logged) |
| POST | /admin/posts/:id/approve | ADMIN | Approve post → ACTIVE |
| POST | /admin/posts/:id/reject | ADMIN | Reject post with reason |

---

## Module: Matching (admin interface operation)

Full specification: modules/matching/backend md

| Method | Path | Auth | Summary |
|---|---|---|---|
| GET | /admin/matching/tutors | ADMIN | Search tutor posts for matching (filter by subject/level) |
| GET | /admin/matching/tutees | ADMIN | Search tutee posts for matching |
| POST | /admin/matches | ADMIN | Create match linking an approved tutee post to a tutor user; published tutor listing is optional (INT-034). |
| GET | /admin/matches | ADMIN | List all matches |
| GET | /admin/matches/:id | ADMIN | Match detail |
| PATCH | /admin/matches/:id/status | ADMIN | Update match status (ACTIVE, COMPLETED, CANCELLED) |
| GET | /tutee/matches | TUTEE | Tutee's own matches |
| GET | /tutee/matches/:id | TUTEE (own) | Match detail for tutee |
| POST | /tutee/matches/:id/confirm-session | TUTEE | Confirm first session occurred (BR-010) |
| GET | /tutor/matches | TUTOR | Tutor's own matches |
| GET | /tutor/matches/:id | TUTOR (own) | Match detail for tutor |

---

## Module: Commission (admin interface operation)

Full specification: modules/commission/backend md

| Method | Path | Auth | Summary |
|---|---|---|---|
| GET | /admin/commissions | ADMIN | List commissions |
| GET | /admin/commissions/:id | ADMIN | Commission detail |
| POST | /admin/commissions | ADMIN | Create commission record for a match |
| POST | /admin/commissions/:id/pay | ADMIN | Record agency receipt of the tutor commission as PAID and complete the match. Parent lesson payment or an unpaid dispute resolution cannot authorize this action. |
| GET | /admin/commissions/:id/invoice | ADMIN | Generate invoice (PDF/HTML) |

---

## Module: Interview (api v1 interface operation)

Full specification: modules/interview/backend md

| Method | Path | Auth | Summary |
|---|---|---|---|
| POST | /interviews | TUTEE | Request interview. **spec 021:** allowed when the tutee post is **PENDING** or ACTIVE (contact remains assigned-participant/match-state gated; post approval alone never releases it) |
| GET | /tutee/interviews | TUTEE | List own interviews |
| GET | /tutee/interviews/:id | TUTEE (own) | Interview detail for tutee |
| POST | /tutee/interviews/:id/cancel | TUTEE (own) | Cancel own interview (REQUESTED status only) |
| GET | /tutor/interviews | TUTOR | List own interviews (SCHEDULED + recent) |
| GET | /admin/interviews | ADMIN | Admin interview queue → Admin Interview Response. **spec 019 (ADM-05):** carries tutee Post Subject/tutee Post Level to render the Job column (incl. free-market interviews) |
| GET | /admin/interviews/:id | ADMIN | Interview detail (contact info — access logged) |
| POST | /admin/interviews/:id/schedule | ADMIN | Schedule interview (date + meeting link) |
| PATCH | /admin/interviews/:id/meeting-link | ADMIN | Update meeting link |
| POST | /admin/interviews/:id/complete | ADMIN | Mark interview as completed |
| POST | /admin/interviews/:id/cancel | ADMIN | Cancel interview |
| POST | /admin/interviews/:id/no-show | ADMIN | Mark no-show (TUTEE/TUTOR/BOTH) |

---

## Module: Invitations & Profile gate (Wave 1)

Full specification: reference/backend-api md (INT-044 Job Invitations · M7 Profile).

| Method | Path | Auth | Summary |
|---|---|---|---|
| GET | /tutee/invitations | TUTEE | **spec 013 REQ-003 (Wave 1)** — caller's OWN outbound (sent) invitations, paginated, newest first. Default PENDING only; ?status is ALL for history. personal information-safe Job Invitation Response (tutor handle only); own-only scope by created by |
| GET | /tutor/profile | TUTOR | Own tutor profile. **Wave 1:** now returns **profile Complete (bool)** + **missing Fields (string[])** — the apply-gate single source of truth (true ⇔ missing Fields empty). Stable field keys: Handle, commitment, hourly Rate Min, hourly Rate Max, location Planning Areas, qualifications, subject Levels. The same verdict the apply endpoint enforces (submit operation for tutor applications → 422 TUTOR PROFILE INCOMPLETE) |

---

## Health & System

| Method | Path | Auth | Summary |
|---|---|---|---|
| GET | /actuator/health | public (or internal) | Spring Boot health check (used by Railway + uptime monitor) |
| GET | /actuator/info | public | Build info (version, commit SHA) |

/actuator/** endpoints except /health and /info should be restricted to internal access only (localhost or Railway internal network) — do not expose metrics or env endpoints publicly.

---

## Response Status Codes Reference

| Code | Usage |
|---|---|
| 200 | Success (GET, PATCH, POST with side effects) |
| 201 | Created (POST that creates a resource) |
| 400 | Validation error (malformed input) |
| 401 | Unauthenticated (no valid auth cookie) |
| 403 | Forbidden (authenticated but wrong role or ownership) |
| 404 | Resource not found |
| 409 | Conflict (duplicate email, phone, etc.) |
| 422 | Business rule violation (guard failed — see module error codes) |
| 423 | Account locked (retry After field present) |
| 429 | Rate limit exceeded (retry After field present) |
| 500 | Internal server error |

### Selected domain error codes (additive — see reference/backend-api md for the full catalogue)

| web request protocol | code | When |
|---|---|---|
| 404 | NOT FOUND | No matching handler for the path (spec 016 — was previously a 500); a wrong method on an existing path is still 405 |
| 409 | PHONE ALREADY REGISTERED | Phone uniqueness clash — registration or /account/contact-change/verify; preserve the existing conflict code/message. The old direct-phone route is superseded by 012 |
| 409 | EMAIL ALREADY REGISTERED | Email uniqueness clash on /account/contact-change/verify (re-checked at commit) |
| 422 | STEP UP REQUIRED | submit operation for account contact-change with a wrong/absent current Password |
| 400 | CONTACT CHANGE TOKEN INVALID | submit operation for account contact-change verify token wrong / expired / consumed |
| 400 | VALIDATION ERROR (+field Errors) | Bean-validation failure; carries a field Errors map (field → message) — e g. create-admin submit operation for admin admins |
