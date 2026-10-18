# InTuition — Posting Module — Backend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

Keep the folded PENDING-tutee-edit amendment from 021 and the required REVISION REQUESTED revise/resubmit path. Historical APPROVED/PAUSED aliases must defer to the canonical eight-state posting model. M18 removes active-tutor-listing gates from browse/apply; [012](../../012-tutor-apply-and-profile-ux/design.md) supplies the single server completeness predicate. Use [013](../../013-tutee-dashboard-and-invitations/requirements.md) for approval-preserving owner edits.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** posting  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. application programming interface Endpoints

All endpoints prefixed api v1 interface operation. Cursor-based pagination on all list endpoints.

### 1.1 Create Tutor Post


The illustration for create tutor post contains the following information:

- POST posts tutor interface operation.
- Authorization: TUTOR role required.


**Request Body:**


The create tutor post example describes information including subjects, levels, rate Min Sgd, rate Max Sgd, venue Types, group Sizes, availability Bitmask, qualifications, teaching Approach, years Experience, location Region. Concrete payload values and implementation syntax are omitted.


**Validation (Jakarta Bean Validation on request data transfer object):**
- subjects: array, 1–10 items, each must exist in subjects lookup table
- levels: array, 1–8 items, each must exist in levels lookup table
- rate Min Sgd: number, 1–1000, max 2 decimal places
- rate Max Sgd: number ≥ rate Min Sgd, max 1000
- venue Types: array, min 1, values from enum STUDENT HOME|TUTOR HOME|PUBLIC|ONLINE|CENTRE
- group Sizes: array, min 1, values from enum ONE ON ONE|SMALL GROUP
- availability Bitmask: integer, 1–2097151 (at least 1 of 21 bits set)
- qualifications: string, 1–1000 chars
- teaching Approach: string, 0–500 chars (optional)
- years Experience: integer, 0–50
- location Region: string, optional, matches the letter D followed by two digits or MRT station code

**Processing:**
1. Create tutor posts row with status DRAFT
2. Insert into tutor post subjects junction (post id, subject code for each)
3. Insert into tutor post levels junction (post id, level code for each)
4. Log POST CREATED to audit log

**Response 201:**


The create tutor post example describes information including id, status, created At. Concrete payload values and implementation syntax are omitted.


---

### 1.2 Create Tutee Post


The illustration for create tutee post contains the following information:

- POST posts tutee interface operation.
- Authorization: TUTEE role required.


**Request Body:**


The create tutee post example describes information including subject, level, budget Min Sgd, budget Max Sgd, venue Prefs, availability Bitmask, student Count, special Notes, location Region. Concrete payload values and implementation syntax are omitted.


**Validation:** Similar pattern; subject and level are single values (not arrays). special Notes max 300 chars. Student count is bounded **1–20** (over → 400 VALIDATION ERROR). An unknown code **or an incoherent subject+level pair** → **422 UNKNOWN SUBJECT OR LEVEL** (spec 009 + spec 020 pair-aware; previously a 500). The pair is coherent iff the subject and level share a level group, **plus the N LEVEL → O LEVEL widen** (N-Level subjects are valid against O LEVEL secondary years — verify Taxonomy Repository is Valid Subject Level Pair). Taxonomy is the seeded set (migration 0064 — 34 subjects / 10 levels incl. the distinct N LEVEL group; **canonical corrections migration 0067:** +Combined Humanities (O-Level), +H1 sciences (A-Level), +N using T levels S1 NT–S4 NT; generic MATH OLEVEL "Mathematics (O-Level)" **deprecated** — soft is active is no, use E-Math/A-Math).

**Response 201:** the new record’s identifier, DRAFT state and creation time — plus a top-level trace Id (spec 010; every JSON-object response body carries it, mirrored by the cross-origin request permissions-exposed X-Correlation-ID header).

---

### 1.3 Submit Post for Review


The illustration for submit post for review contains the following information:

- POST posts tutor :id submit interface operation.
- POST posts tutee :id submit interface operation.
- Authorization: Owner only (TUTOR/TUTEE).


No request body. Transitions post from DRAFT or REJECTED → PENDING.

**Guards:**
- Post must be owned by requesting user
- Post must be in DRAFT or REJECTED status (state machine guard)
- All required fields must be populated (re-validate)

**Processing:**
1. Update status PENDING
2. Record the current time as the post submission timestamp
3. Queue N-02 (post submitted confirmation) to InTuition async email queue (Spring @Async + Redis)
4. Log POST SUBMITTED to audit log

**Response 200:** the pending state

**Errors:** 403 (not owner), 409 (invalid state transition), 422 (incomplete fields)

---

### 1.4 Update Post


The illustration for update post contains the following information:

- PATCH posts tutor :id interface operation.
- PATCH posts tutee :id interface operation.
- Authorization: Owner only.


Partial update — only provided fields are updated. Junction tables are replaced in full (delete all then reinsert) if subjects or levels provided.

**Status transition on edit:**
- If current status = ACTIVE → set status = PENDING (re-enters review queue)
- If current status = DRAFT, REJECTED, or REVISION REQUESTED → status unchanged (edits allowed freely)
- If current status = PENDING:
  - **Tutee post (spec 021/REQ-003):** editable → **200**, stays PENDING and re-enters the approval queue (re-stamps submitted At). Tutee EDITABLE allow-list = DRAFT/REJECTED/ACTIVE/PENDING.
  - **Tutor post:** still **409** (cannot edit while under review; admin must act first) — Tutor Post Service update rejects PENDING.

**Response 200:** Updated post object (Tutor Post Owner data transfer object / Tutee Post Owner data transfer object)

---

### 1.5 Delete Post


The illustration for delete post contains the following information:

- DELETE posts tutor :id interface operation.
- DELETE posts tutee :id interface operation.
- Authorization: Owner only.


**Guards:** Only allowed if status = DRAFT or REJECTED. Posts with APPROVED/PENDING/PAUSED status cannot be deleted (only expired or cancelled via admin).

**Processing:** Hard delete (posts with no matches/reviews can be permanently deleted). If post has review log entries → soft delete (set deleted at).

**Response 200:** a success confirmation

---

### 1.6 Browse Tutor Posts (Public Listing)


The illustration for browse tutor posts public listing contains the following information:

- GET posts tutors interface operation.
- Authorization: None (public).


**Query Parameters:**


The illustration for browse tutor posts public listing contains the following information:

- subjects[] string[] Filter by subject code.
- levels[] string[] Filter by level code.
- venue Types[] string[] Filter by venue type.
- rate Min number Minimum rate.
- rate Max number Maximum rate.
- location Region string District (D01–D28) or MRT station code.
- availability integer Bitmask — posts must have ≥1 matching bit.
- group Sizes[] string[] Filter by group size.
- cursor string Cursor for next page.
- limit integer Default 20, max 50.


**Response 200:**


The browse tutor posts public listing example describes information including posts, next Cursor, total. Concrete payload values and implementation syntax are omitted.


**Tutor Post Public data transfer object** (contact info excluded per BR-001):


The browse tutor posts public listing example describes information including id, display Name, subjects, levels, rate Min Sgd, rate Max Sgd, venue Types, group Sizes, availability Bitmask, qualifications, years Experience, location Region, created At. Concrete payload values and implementation syntax are omitted.


**Availability filter logic:**


Implementation example omitted. This public edition retains the written requirements in this section.


Returns posts with at least one overlapping slot.

---

### 1.7 Browse Tutee Requests (Logged-in Approved Tutors Only)


The illustration for browse tutee requests logged-in approved tutors only contains the following information:

- GET posts requests interface operation.
- Authorization: TUTOR role required.
- Guard: requesting tutor must have ≥1 post with status = 'ACTIVE'.


Same query param structure. Returns Tutee Post Tutor data transfer object — subject, level, budget, venue prefs, availability, student count, location. Excludes: tutee user id, display name, phone, email, special notes with identifying info.

**Active-post guard (service layer):**


Implementation example omitted. This public edition retains the written requirements in this section.


Rationale: only tutors with an approved, live post are part of the active marketplace. This prevents unvetted tutors from browsing demand signals before being approved.

---

### 1.8 Get Single Post


The illustration for get single post contains the following information:

- GET posts tutor :id interface operation.
- GET posts tutee :id interface operation.


**Authorization and data transfer object selection:**
- Public / TUTEE requesting tutor post → Tutor Post Public data transfer object
- TUTOR requesting tutee post (own check) → Tutee Post Tutor data transfer object
- ADMIN → Tutor Post Admin data transfer object / Tutee Post Admin data transfer object (includes decrypted contact info)
- Owner requesting own post → Tutor Post Owner data transfer object (includes all fields, no other users' data)

Returns 404 if post not found. Returns 404 (not 403) if post is PENDING/DRAFT and requester is not owner or admin (prevent enumeration).

---

### 1.9 My Posts


The illustration for my posts contains the following information:

- GET dashboard my-posts interface operation.
- Authorization: TUTOR or TUTEE role required.


**Query Parameters:** type is tutor|tutee, status is DRAFT|PENDING|APPROVED|REJECTED|PAUSED FOR REVIEW|EXPIRED, cursor, limit

Returns all posts owned by the authenticated user (both types if no type filter). Includes status, rejection reason if REJECTED, match count for APPROVED posts.

---

### 1.10 Taxonomy Endpoints


The illustration for taxonomy endpoints contains the following information:

- GET taxonomy subjects interface operation.
- GET taxonomy levels interface operation.


Public, cached. Returns full list of valid subject code and level code values with display labels, grouped by curriculum tier.

### 1.11 Tutee outbound invitations (spec 013 REQ-003, Wave 1)


Implementation example omitted. This public edition retains the written requirements in this section.


TUTEE only. The caller's OWN sent invitations (own-only scope by created by — IDOR-safe), returned as the personal information-safe
a page of job invitation records, with pagination details (tutor handle only — never a name/email/phone). Full contract +
invite/withdraw/accept lifecycle: reference/backend-api md → INT-044 Job Invitations.

---

## 2. Service Layer

### Post Service


Implementation example omitted. This public edition retains the written requirements in this section.


### Post State Service

Encapsulates the state machine. Throws the response status exception operation (→ 409) for illegal transitions.

Valid transitions:


The illustration for post state service contains the following information:

- DRAFT → PENDING (on submit).
- REJECTED → PENDING (on resubmit after rejection).
- REVISION REQUESTED → PENDING (on resubmit after revision request).
- EXPIRED → PENDING (on reactivation).
- PENDING → ACTIVE (admin approve).
- PENDING → REJECTED (admin reject).
- PENDING → REVISION REQUESTED (admin request changes).
- ACTIVE → PENDING (owner edits live post).
- ACTIVE → MATCHED (system: match created — Tutee Post only).
- MATCHED → ACTIVE (system: match cancelled — Tutee Post only).
- ACTIVE → CLOSED (owner/admin closes).
- MATCHED → CLOSED (owner/admin closes).
- ACTIVE → EXPIRED (scheduler, 60 days post-approval).


---

## 3. data transfer object Field Allowlists

### Tutor Post Public data transfer object — allowed fields:
id, display Name, subjects, levels, rate Min Sgd, rate Max Sgd, venue Types, group Sizes, availability Bitmask, qualifications, teaching Approach, years Experience, location Region, created At, status

### Tutor Post Admin data transfer object — extends public, adds:
tutor Id, email (decrypted), phone (decrypted), full Name, rejection Reason, submitted At, reviewed At, reviewed By

### Tutee Post Tutor data transfer object — allowed fields:
id, subject, level, budget Min Sgd, budget Max Sgd, venue Prefs, availability Bitmask, student Count, location Region, created At, status

### Tutee Post Admin data transfer object — extends tutor-visible, adds:
tutee Id, email (decrypted), phone (decrypted), display Name, relationship, special Notes, rejection Reason, submitted At

---

## 4. Rate Limiting

Public listing endpoints (posts tutors interface operation, posts requests interface operation): 100 requests/min per IP, applied at Spring Boot filter layer.

Post creation: 10 new posts per user per day (enforced at service layer, counting PENDING + APPROVED + DRAFT posts created today).

---

## 5. Availability Bitmask Query Logic

The 21-bit availability bitmask enables efficient overlap queries without unpacking to rows:


Implementation example omitted. This public edition retains the written requirements in this section.


Client sends filter as integer; server validates range (1–2097151) before executing query.
