# Solution Architecture Document


## October 2026 reading note

This document preserves the original architecture proposal. Use [How it is built](../../docs/how-it-is-built.md) for the inspected implementation and [specification 027](../027-aws-production-deployment/spec.md) for the approved hosting plan. The old proposal below does not describe a live deployment.

| Earlier proposal below | Inspected state or later decision |
|---|---|
| React 18 | The inspected lockfile contains React 19.2.6. |
| PostgreSQL 16 | Development and combined-test definitions use PostgreSQL 15. |
| Railway and the old hosting diagram | The later AWS plan governs production design; deployment is not verified here. |
| Spring scheduled tasks alone | The implementation also uses database-backed scheduling through db-scheduler 16.12.0. |
| Earlier encryption and contact statements | Current contact fields are stored in plain text; access and disclosure restrictions remain required. |
| Queue and retry components | Known delivery and queue-capacity gaps still need correction and recovery tests. |


> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Explicit amendments govern current scope auth, M16/M19 admin/OAuth/cross-site request forgery protection, Resend email,007 scheduler, PRD20/INT034 assignment/contact and027 production. Historical topology/SES/multi-factor authentication/database structure definitions/cron examples and original ADR approval labels do not outrank these. Current decisions and remaining policy questions are in CURRENT-REQUIREMENTS md.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.
## InTuition — Singapore Private Tuition Marketplace Platform

**Document Version:** 2.0 — Post Team Review (Spring Boot SPA Architecture)
**Date:** 2026-05-27
**Status:** Approved for Development
**Prepared By:** Solution Architecture
**Source Documents:** business requirements document v1.1 (2026-05-27), product requirements document v1.1 (2026-05-27)

---

## Table of Contents

1. [Architecture Decision](#1-architecture-decision)
2. [System Architecture Diagram](#2-system-architecture-diagram)
3. [Component Architecture](#3-component-architecture)
4. [Authentication Architecture](#4-authentication-architecture)
5. [Data Architecture](#5-data-architecture)
6. [application programming interface Design](#6-server-interface-design)
7. [Scheduled Jobs Architecture](#7-original-scheduled-jobs-proposal)
8. [Email Architecture](#8-email-architecture)
9. [Security Architecture](#9-security-architecture)
10. [Deployment Architecture](#10-deployment-architecture)
11. [Availability and Disaster Recovery](#11-availability-and-disaster-recovery)
12. [Singapore Compliance](#12-singapore-compliance)
13. [Architecture Decision Records (ADR)](#13-architecture-decision-records-adr)

---

## 1. Architecture Decision

### 1.1 Original stack proposal from May 2026

| Layer | Choice | Justification |
|---|---|---|
| Frontend framework | **React 18 + TypeScript (Vite), SPA** | Decoupled from Java backend; Vite fast dev loop; team preference for separate the private implementation folder |
| Client routing | **React Router v6** | Standard SPA routing; no SSR required for MVP |
| Server state | **Tan Stack Query v5** | Data fetching, caching, background sync against Spring Boot REST application programming interface |
| user interface state | **Zustand** | Lightweight global state for auth context and user interface flags |
| Forms / Validation | **React Hook Form + Zod** | Performant uncontrolled forms; shared Zod schemas for client-side validation |
| Styling | **Tailwind CSS** | Utility-first; no runtime overhead; consistent design tokens |
| Frontend hosting | **Spec027 approved Amazon Web Services deployment** | Current production topology is owned by027; the older Cloudflare/Railway diagram below is historical |
| Backend framework | **Java 21 + Spring Boot 3.x** | Team preference for JVM strong typing; mature ecosystem |
| application programming interface layer | **Spring Web MVC REST controllers (** interface operation)** | Standard Spring REST; no frontend-facing service coupling between frontend and backend codebases |
| Security | **Spring Security 6 + signed session token (HS256)** | Industry-standard; http Only cookie delivery via Cloudflare proxy |
| object-relational mapping / persistence | **Spring Data Java persistence mapping + Hibernate** | Type-safe repositories; works with PostgreSQL; N+1 prevention via JPQL fetch joins |
| Migrations | **Liquibase changesets** | Deterministic, version-controlled database query language; ddl-auto is validate in production |
| Validation | **Jakarta Bean Validation** | Declarative constraint annotations on request data transfer objects |
| data transfer object mapping | **Manual mapping in MVP; Map Struct post-MVP** | Map Struct deferred (ADR-013); manual mapping reduces compile-time setup for the small development effort |
| Boilerplate reduction | **Lombok** | @Data, @Builder, @Required Args Constructor on entities and data transfer objects |
| Email queue | **Spring @Async + Redis Streams** | Non-critical email queue using the same Redis instance already provisioned |
| Scheduled jobs | **Spring scheduled tasks** | Built-in cron execution; no external scheduler service |
| Rate limiting | **Redis sliding window (MVP); Bucket4j post-MVP** | Bucket4j deferred (ADR-014); Redis counters sufficient for MVP volumes |
| Build tool | **Maven** | Standard Spring Boot build; the documented operational check produces fat JAR |
| Backend hosting | **Railway (Spring Boot fat JAR, ap-southeast-1, Java 21 runtime)** | Co-located with PostgreSQL and Redis; simple deploy model |
| Primary database | **PostgreSQL 16 (Railway, ap-southeast-1)** | Relational model fits the marketplace domain; managed backups; same Railway project |
| Cache / queue | **Redis (Railway, ap-southeast-1)** | Sessions, jti keys, email queue, rate limit counters, cron dedup keys |
| Email | **Resend** | Founder email-resend decision supersedes SES; production credentials/operations follow027 |
| Object storage | **Amazon Web Services S3 (ap-southeast-1)** | Singapore residency; server-sent events-S3 encryption; pre-signed URL support |
| Monitoring | **Sentry (React SPA + Spring Boot) + Logtail** | Error tracking; log draining from Railway |

### 1.2 Frontend Stack Justification

**Why React 18 + Vite SPA over Next implementation reference App Router (ADR-001):**

1. **Decoupled architecture** — The team chose a dedicated Java 21 + Spring Boot backend. A Vite SPA is the natural frontend counterpart: it calls Spring Boot REST application programming interfaces directly, with no framework-mandated frontend-facing service layer or server runtime in the frontend codebase.

2. **Simpler mental model** — With a Vite SPA there are no Server Components, no hybrid rendering modes, no server-side data fetching patterns to reason about alongside Spring Boot. All server interactions go through explicit Tan Stack Query hooks.

3. **Cloudflare Pages fit** — A Vite SPA builds to static files. Cloudflare Pages serves them from global CDN with zero configuration. No Node implementation reference runtime is required on the frontend host.

4. **SEO trade-off accepted** — The loss of native SSR for /tutors public browse pages is accepted for v1. Modern Googlebot handles client-rendered React adequately for initial organic traffic. Cloudflare Pages prerender workers for /tutors and the tutor page identified by its public handle are deferred to v2 (ADR-012). This will be revisited if organic ranking is below expectations after 3 months post-launch.

### 1.3 Backend Stack Justification

**Why Java 21 + Spring Boot 3.x over Node implementation reference + TypeScript / Nest JS (ADR-002):**

1. **Team preference** — The team has stronger JVM experience. Spring Boot is the primary backend of choice.

2. **Strong typing at scale** — Java generics, Jakarta Bean Validation annotations, and Map Struct (post-MVP) provide end-to-end compile-time safety that the team values for a financial-adjacent marketplace.

3. **Scope control for the small development effort** — Map Struct, Bucket4j, and Spring Cloud are all deferred. Spring Initializr defaults throughout. The scope is intentionally constrained to keep MVP velocity reasonable.

### 1.4 PostgreSQL + Spring Data Java persistence mapping Justification

- The domain has a well-defined relational structure: users → posts → matches → commissions, with explicit foreign key relationships and cardinality constraints enforced by partial unique indexes.
- Spring Data Java persistence mapping + Hibernate provides type-safe repositories, JPQL fetch joins for N+1 prevention, and lifecycle callbacks for audit logging.
- Liquibase owns all schema changes. spring jpa.hibernate ddl-auto is validate in production — Hibernate validates but never modifies the schema.
- Partial indexes (match cardinality enforcement) are managed exclusively in Liquibase changesets.
- The 21-bit availability bitmask is stored as a PostgreSQL INTEGER, allowing bitwise queries in the admin matching dashboard filter.

### 1.5 Redis Justification

- **signed session token jti tracking** — Admin sessions tracked in Redis jti allowlist (30-min idle expiry period). TUTEE/TUTOR logout jti denylist (the session-revocation record for the token identifier, expiry period = remaining token expiry).
- **Email job queue** — Non-critical emails enqueued to Redis Streams with consumer groups. Spring @Async dispatches to stream; a dedicated consumer group reads and delivers to SES.
- **Rate limit counters** — Redis sliding window counters enforce MVP rate limits.
- **Cron deduplication** — Each scheduled job writes a deduplication key (a scheduled-task record identified by job and Singapore date) to Redis with a 24-hour expiry period.

---

## 2. System Architecture Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


**Data flow summary:**

1. Browser requests arrive at Cloudflare's edge. Static SPA assets (HTML/JS/CSS) are served from Cloudflare Pages CDN. application programming interface requests to example com/api/* are intercepted by a Cloudflare Worker and proxied to the Railway Spring Boot instance at https://api-internal railway.app/api/*.
2. Because the Cloudflare Worker delivers application programming interface responses under the same origin (example com), the browser treats all requests as same-origin. No cross-origin request permissions headers are needed. Same Site is Strict http Only cookies work without any cross-origin configuration.
3. Spring Security 6 validates the signed session token from the http Only cookie on every authenticated request. The signed session token filter chain runs before any controller method.
4. Spring Data Java persistence mapping + Hibernate queries PostgreSQL over an encrypted TCP connection to the Railway-managed PostgreSQL 16 instance in ap-southeast-1.
5. Redis is used for signed session token jti tracking (admin allowlist, tutee/tutor logout denylist), rate limit counters, email job streams, and cron deduplication keys.
6. Critical emails (N-10, N-12, N-13) call Amazon Web Services SES directly and synchronously from within the Spring Boot service method, targeting a 5-minute service-level agreement. Non-critical emails are published to a Redis Stream; the Spring @Async consumer group reads and dispatches them to SES.
7. Profile photos are uploaded to Amazon Web Services S3 (ap-southeast-1) with server-sent events-S3 encryption. application programming interface responses include pre-signed URLs with 1-hour expiry period. URLs are returned only to the owner (tutors) and admin.
8. The Analytics Event Publisher bean is the only path for emitting analytics events. It validates the event schema, strips personal information, and dispatches via Spring @Async outside the active a database transaction boundary (using a Transaction Synchronization after Commit hook).

---

## 3. Component Architecture

### 3.1 Frontend SPA Structure


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.2 Spring Boot Package Structure


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.3 Public Tutor Browse (Client-Rendered)

The /tutors public listing and /tutors/:slug detail pages are React components. On mount, Tan Stack Query fetches from read operation for posts tutors (paginated) and read operation for posts tutors slug respectively. No session cookie is required. Responses are the Tutor Post Public data transfer object — contact information is never present.

SEO note: For v1, modern Googlebot handles client-rendered React. Cloudflare Pages prerender workers for these routes are deferred to v2 (ADR-012).

### 3.4 Authenticated App (Client-Side Navigation)

Tutee and tutor dashboards are React components navigated via React Router v6. Tan Stack Query manages all server state — mutations for post creation/update and queries for matches/posts. Zustand holds the current user's auth state (role, user Id) populated on login and persisted across soft navigation.

**Auth state hydration:** On hard load, the React app calls read operation for auth me. Spring Boot validates the signed session token from the http Only cookie and returns the current user profile. Zustand stores the result. If the cookie is absent or invalid, the user is redirected to /login.

### 3.5 Admin Panel (Desktop-First)

The admin panel lives under /admin/* with a separate React Router layout. Every admin route is wrapped by Admin Mfa Guard which checks: (1) authenticated session, (2) role = ADMIN, (3) time-based one-time passcode multi-factor authentication verified within the current session, (4) session age < 30 minutes since last activity (validated via Redis jti allowlist).

The matching dashboard uses two-panel layout with Tan Stack Query fetching both tutor posts and tutee posts in a single call to read operation for admin matching. Filters trigger client-side refetch with debouncing.

### 3.6 Analytics Event Publisher

Analytics Event Publisher is the **single, mandatory path** for emitting analytics events. No other code path may write to the analytics events table. It:

1. Validates the event payload against the declared event schema.
2. Strips personal information fields: full meeting link URL, tutee id from interview requested events, any email/phone fields that may appear in raw event context.
3. Dispatches the validated, personal information-stripped event via @Async — outside the active a database transaction boundary. Uses the register synchronization operation so events are only emitted after the transaction commits successfully.
4. Persists the event to analytics events.

### 3.7 Audit Log Store

The audit log is an append-only PostgreSQL table in a separate schema (audit log) within the same Railway database instance. Two PostgreSQL roles are used:

- app user — has read fields, append records, update records, DELETE on app schema tables. Has **append records-only** on audit log.*.
- audit reader — has read fields-only on audit log.*. Used exclusively by the DPO audit view in the admin panel.

No application code path has update records or DELETE on audit log.*. This is enforced at the database role level.

---

## 4. Authentication Architecture

### 4.1 Authentication Stack

**Token format:** HS256 signed session token signed with a 256-bit secret (signed session token SECRET Railway environment variable).

**Token delivery:** http Only cookie: [REDACTED CREDENTIAL]

**Why Same Site is Strict works:** Because the Cloudflare Worker proxies api * interface operation under the same origin as the SPA (example com), the auth cookie is always same-origin from the browser's perspective. Same Site is Strict prevents cross-origin delivery entirely. See ADR-003 and ADR-005.

**Current session contract:** spec004 requires30-minute access tokens plus hash-only opaque rotating refresh sessions for TUTEE/TUTOR; single-flight refresh, bounded grace, revocation and idle/absolute caps apply. The no-refresh/24-hour alternative is superseded.

### 4.2 signed session token Payload


The signed session token payload example describes information including sub, role, jti, mfa Verified, iat, exp. Concrete payload values and implementation syntax are omitted.


The kid header claim is included to support key rotation windows (see ADR-004).

### 4.3 Role-Based Access

Three roles are enforced at the Spring Security filter chain level:

| Role | Assignment | Token Lifetime | multi-factor authentication |
|---|---|---|---|
| TUTEE | Set at registration; mutually exclusive with TUTOR | 30-minute access + spec004 refresh | None required |
| TUTOR | Set at registration; mutually exclusive with TUTEE | 30-minute access + spec004 refresh | None required |
| ADMIN | M16 authenticated admin creation; protected super-admin; no public admin signup | 30-min idle | multi-factor authentication deferred for local/dev; production policy remains explicit |

**Role enforcement:** Jwt Authentication Filter validates the signed session token and sets the Spring Security context on every request. Controllers use the pre authorize operation or equivalent annotations. Frontend role guards are usability aids only; they do not constitute access control.

### 4.4 signed session token Filter Chain


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.5 Admin multi-factor authentication Architecture


Implementation example omitted. This public edition retains the written requirements in this section.


**Critical constraint (AC-07):** Any application-level attempt to skip multi-factor authentication verification returns web request protocol 403, logged in the audit log. The multi-factor authentication check is in Admin Session Validator called from Jwt Authentication Filter, not in individual controllers, making it structurally impossible to bypass by omitting a handler-level check.

### 4.6 TUTEE / TUTOR Session Lifecycle

| Event | Action |
|---|---|
| Login | Issue30-minute access signed session token and spec004 refresh cookies |
| Authenticated request | Validate signed session token signature + expiry; check jti denylist in Redis |
| Logout | Delete auth cookie; write record token revocation for the token’s remaining lifetime to Redis denylist |
| Access expiry (30min) | Single-flight refresh and one protected retry under004; terminal failure requires re-login |

### 4.7 Admin Account Creation

M16/admin-management supersedes seed-only creation: authenticated admins can create admins; only the protected super-admin may delete/demote/disable another admin. Bootstrap runs only under exact local profile plus seed admin.enabled, idempotently through the shared current contact-normalization/password-hashing creation path; never test/prod by a broad !prod gate. Enforce password-change-required on admin mutations. multi-factor authentication is deferred in the approved local/dev scope; unresolved production multi-factor authentication policy is not silently approved by the old design.

---

## 5. Data Architecture

### 5.1 Primary Database (PostgreSQL 16 on Railway, ap-southeast-1)

**Critical production configuration:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Schema overview:**


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.2 Liquibase Migration Strategy

All schema changes are managed exclusively through Liquibase changesets. This is non-negotiable in production.

**Configuration:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Migration file naming convention (Liquibase XML/YAML — not Flyway  implementation format implementation reference):**


Implementation example omitted. This public edition retains the written requirements in this section.


Changesets use the Liquibase XML format with id, author, and the reviewed database rollback blocks. **Do not use Flyway-style V1 * implementation reference filenames** — Liquibase does not use that naming convention and does not auto-discover  implementation format implementation reference files outside the master changelog include list.

**database structure definitions lock timeout:** Every changeset that performs database structure definitions on a live table must begin with limit waiting for a database lock to three seconds to prevent indefinite blocking during rolling deploys on long-running queries.

**ENUM ADD VALUE rollback policy:** PostgreSQL ENUM ADD VALUE is non-transactional and cannot be reversed inside a transaction block. Any changeset that adds an ENUM value must use a documented no-op the reviewed database rollback block:


Implementation example omitted. This public edition retains the written requirements in this section.


**Partial indexes are owned by Liquibase:** Every partial index — including match cardinality enforcement indexes — is defined in a Liquibase changeset. Hibernate's database structure definitions generation does not understand partial indexes. ddl-auto is validate will confirm columns and types match entities but will not create or modify indexes.


Implementation example omitted. This public edition retains the written requirements in this section.


**continuous integration/continuous deployment migration sequence (critical):**
Liquibase migrations run BEFORE the new Spring Boot JAR starts accepting traffic. On Railway, this is configured as a Railway Release Command:


Implementation example omitted. This public edition retains the written requirements in this section.


This ensures: (1) new columns/tables exist before the new code runs, (2) rollback is possible if migration fails (new JAR never starts).

### 5.3 Schema Design Decisions

**21-bit Availability Bitmask:**

Stored as a PostgreSQL INTEGER. Bit positions 0–20 correspond to Mon–Sun × Morning/Afternoon/Evening (21 slots). Bitwise AND operations in database query language enable the admin matching dashboard to filter for tutors/tutees with overlapping availability without deserializing the bitmask in application code.


Implementation example omitted. This public edition retains the written requirements in this section.


**Subjects and Levels — Junction Tables (Not Array Columns):**

Tutor posts support multiple subjects and levels. These use dedicated junction tables (tutor post subjects, tutor post levels) rather than PostgreSQL array columns because:
- Junction tables allow referential integrity and indexed lookups by subject code/level code.
- The admin matching dashboard filters on subject/level; indexed FK lookups outperform array containment operators (@ ARRAY ...).
- Spring Data Java persistence mapping handles @One To Many relations cleanly with JPQL fetch joins for N+1 prevention.
- Future normalization (subject/level reference tables) requires no schema change to the junction tables.

Tutee posts use single scalar subject and level columns (one subject, one level per request).

### 5.4 personal information Encryption at Column Level

> **Current contact amendment:** user email/phone use the explicit plaintext contract (C13). The AES-SIV/keyed message authentication contact examples below are historical; do not rebuild their columns or adapters. Other sensitive-data protections retain their own scope.

**Algorithm:** AES-SIV (Synthetic IV) for deterministic authenticated encryption on lookup fields (email, phone). AES-256-GCM with random IV for non-lookup fields (full name, home address, totp secret).

AES-SIV produces the same ciphertext for the same plaintext under the same key, enabling exact-match lookups without a separate plaintext index. It is authenticated (AEAD) — ciphertext tampering is detected.

**Key management:**


Implementation example omitted. This public edition retains the written requirements in this section.


**keyed message authentication index for email/phone:** A keyed message authentication-SHA256 index column is maintained alongside each AES-SIV encrypted lookup field. This provides a secondary non-decryptable index for admin search. The AES-SIV approach already provides determinism, but the keyed message authentication index is retained to support key rotation windows (old and new keyed message authentication values can coexist).

#### MEK Rotation Procedure


Implementation example omitted. This public edition retains the written requirements in this section.


### 5.5 Object Storage (Amazon Web Services S3 ap-southeast-1)

| Aspect | Specification |
|---|---|
| Bucket | tuitionagency-profiles-sg (private, no public access) |
| Key format | a profile-image location formed from the user, upload time, a random suffix and file extension |
| Encryption | server-sent events-S3 (AES-256 managed by Amazon Web Services) |
| Access | Pre-signed URL, 1-hour expiry period, generated per request by Spring Boot |
| Deletion pipeline | Account deletion triggers S3 object deletion within 24 hours |
| Upload size limit | 5 MB enforced by Spring Boot (configured on Multipart Resolver) |
| Accepted formats | JPEG, PNG, Web P (validated server-side by magic byte check, not file extension) |
| Response visibility | Signed URL returned only to: own user (tutors) and admin. Never in public listing. |

### 5.6 Data Retention Implementation

| Data Class | Retention | Implementation |
|---|---|---|
| User personal information (active account) | Indefinite while active | No expiry period; active accounts not purged |
| User personal information (after deletion) | 2 years post-deletion | Historical proposal: select accounts more than two years after deletion. This does not authorize physical deletion; C13/O08 require a separate approved retention and erasure policy |
| Post data (closed/expired) | 1 year post-closure | posts closed at; nightly purge job |
| Match records | 5 years from creation | matches created at; annual purge job |
| Commission records | 7 years from invoice | commissions invoice sent at; annual purge job |
| Audit logs | 5 years; immutable | Never deleted; archived to S3 Glacier ap-southeast-1 after 2 years |
| Analytics events | Anonymized at 90 days; aggregates indefinite | Nightly anonymization job; analytics events.anonymized at |
| Application/server logs | 90 days rolling | Logtail (Better Stack) log drain from Railway; retention configured to 90 days |

---

## 6. Server interface design

### 6.1 REST Conventions

All endpoints follow these conventions:
- Base path: ** interface operation
- web request protocol methods: GET (read), POST (create/action), PATCH (partial update), DELETE (soft-delete where applicable)
- Resource IDs: unique identifier
- Timestamps: ISO 8601 with Coordinated Universal Time offset
- Authentication: http Only signed session token cookie (validated by Spring Security filter chain)
- No request body on GET requests

### 6.2 Endpoint Groups

#### Public Endpoints (No Authentication)

| Method | Path | Description | Response data transfer object |
|---|---|---|---|
| GET | posts tutors interface operation | Paginated public tutor listings | Tutor Post Public data transfer object |
| GET | posts tutors slug interface operation | Single tutor post detail | Tutor Post Public data transfer object |

Tutor Post Public data transfer object includes: headline, subjects, levels, rate range, venue types, availability bitmask (decoded to display slots), location area, qualifications text, years of experience. It explicitly excludes: email, phone, full name, home address, profile photo URL.

#### Tutee-Authenticated Endpoints

| Method | Path | Description |
|---|---|---|
| GET | tutee posts interface operation | Own tutee posts |
| POST | tutee posts interface operation | Create post |
| PATCH | tutee posts id interface operation | Update post (own, when Draft or Revision Requested) |
| POST | tutee posts id close interface operation | Close own post |
| GET | tutee matches interface operation | Own matches |
| GET | tutee matches id interface operation | Own match detail (no tutor contact info) |
| POST | tutee matches id confirm-session interface operation | First-session confirmation (idempotent) |

#### Tutor-Authenticated Endpoints

| Method | Path | Description |
|---|---|---|
| GET | tutor posts interface operation | Own tutor posts |
| POST | tutor posts interface operation | Create listing |
| PATCH | tutor posts id interface operation | Update listing |
| POST | tutor posts id close interface operation | Close listing |
| GET | tutor requests interface operation | Anonymised tutee discovery index (Active posts only) |
| GET | tutor requests id interface operation | Anonymised tutee request detail |
| GET | tutor matches interface operation | Own matches |
| GET | tutor matches id interface operation | Own match detail |
| POST | tutor commission-acknowledgement interface operation | Submit commission disclosure acknowledgement |

#### Admin Endpoints (multi-factor authentication Required)

| Method | Path | Description |
|---|---|---|
| GET | admin posts review interface operation | Pending review queue |
| GET | admin posts review id interface operation | Post full detail (triggers audit log) |
| POST | admin posts id approve interface operation | Approve post |
| POST | admin posts id reject interface operation | Reject post |
| POST | admin posts id request-revision interface operation | Request revision |
| GET | admin matching interface operation | Matching dashboard data (both panels) |
| POST | admin matches interface operation | Create match |
| PATCH | admin matches id interface operation | Update match status |
| POST | admin matches id cancel interface operation | Cancel match |
| POST | admin matches id override-confirmation interface operation | Override tutee first-session confirmation |
| GET | admin commission interface operation | Commission ledger |
| POST | admin commission id invoice interface operation | Generate invoice |
| PATCH | admin commission id interface operation | Update commission status (PAID, DISPUTED, WAIVED) |
| GET | admin users interface operation | User list (triggers audit log on contact field access) |
| GET | admin users id interface operation | User detail |
| POST | admin users id suspend interface operation | Suspend account |
| POST | admin users id ban interface operation | Ban account |
| GET | admin audit interface operation | Audit log (DPO/senior admin only) |

#### Auth Endpoints

| Method | Path | Description |
|---|---|---|
| POST | auth login interface operation | TUTEE / TUTOR login |
| POST | auth admin login interface operation | Admin login (step 1, issues partial signed session token) |
| POST | auth admin mfa verify interface operation | Admin multi-factor authentication verification (step 2, issues full signed session token) |
| POST | auth logout interface operation | Logout (clears cookie, writes jti denylist) |
| GET | auth me interface operation | Returns current user profile (hydrates Zustand on hard load) |

#### Webhook Endpoints

| Method | Path | Description |
|---|---|---|
| POST | webhooks ses-events interface operation | Amazon Web Services SES SNS bounce/complaint webhook |

### 6.3 Role-Aware data transfer object Serialization

**Rule:** Java persistence mapping entity objects are never serialized directly to the web request protocol response. Every response passes through a data transfer object class that applies role-specific field allowlists. Raw entity fields are never leaked.


Implementation example omitted. This public edition retains the written requirements in this section.


Adding a new column to the Java persistence mapping entity does NOT automatically include it in any data transfer object. Every field must be explicitly added to the relevant data transfer object class.

#### Tutee Post Tutor data transfer object — Explicit Field Allowlist

The Tutee Post Tutor data transfer object is the anonymised view of a tutee post served to authenticated tutors via the discovery index. This data transfer object is security-critical.

**Fields explicitly INCLUDED:**

| Field | Notes |
|---|---|
| subject Code | Subject code — not the tutee's identity |
| level Code | Academic level code |
| num Students | Number of students |
| budget Min Sgd | Minimum hourly budget |
| budget Max Sgd | Maximum hourly budget |
| session Duration Hours | Session length in hours |
| frequency Per Week | Sessions per week requested |
| preferred Start Date | Preferred engagement start date |
| special Requirements | Admin-moderated free text; personal information scrubbed before approval |
| location Planning Area | e g., "Tampines" — not street address |
| location Region | e g., "East" |
| preferred Venue Types | Array of venue type codes |
| availability Bitmask | 21-bit availability integer |
| status | Post status (only ACTIVE posts served via this endpoint) |
| created At | Post creation timestamp |

**Fields explicitly EXCLUDED:**

| Field | Reason |
|---|---|
| tutee Id | Directly identifies the tutee user record |
| email Encrypted | Contact information — never exposed to tutors |
| phone Encrypted | Contact information — never exposed to tutors |
| admin Notes | Internal admin field |
| rejection Reason | Internal admin field |

### 6.4 N+1 Prevention

All listing queries use JPQL fetch joins to retrieve related entities in a single query:


Implementation example omitted. This public edition retains the written requirements in this section.


For the admin matching dashboard, both tutor posts and tutee posts are returned in a single application programming interface call to read operation for admin matching to reduce round trips.

### 6.5 Pagination (Cursor-Based)

All listing endpoints use cursor-based pagination. The cursor is a base64-encoded JSON string encoding creation time in a standard format and the record identifier. This provides stable pagination even as new records are inserted during browsing.


The pagination cursor-based example describes information including data, next Cursor, page Size, has More. Concrete payload values and implementation syntax are omitted.


### 6.6 Error Format

All error responses follow a consistent structure:


The error format example describes information including status, error, message, details, timestamp. Concrete payload values and implementation syntax are omitted.


Jakarta Bean Validation errors are caught by @Controller Advice and mapped to this format. web request protocol 400 for validation errors; web request protocol 401 for unauthenticated; web request protocol 403 for unauthorized; web request protocol 404 for not found; web request protocol 429 for rate limited; web request protocol 500 for unexpected server errors.

---

## 7. Original scheduled-jobs proposal

### 7.1 Runner

**Spring scheduled tasks** is the cron job runner. Jobs run within the Spring Boot process. Cron expressions use Coordinated Universal Time; Singapore time offset is handled inside the job handler logic.


Implementation example omitted. This public edition retains the written requirements in this section.


**Singapore time alignment:** Each job handler validates the expected Singapore time time window before executing business logic, to handle any scheduler drift or startup delay.

### 7.2 Job Specifications

#### Job 1: first session confirmation timeout


Implementation example omitted. This public edition retains the written requirements in this section.


#### Job 2: post auto expiry


Implementation example omitted. This public edition retains the written requirements in this section.


#### Job 3: active inventory snapshot


Implementation example omitted. This public edition retains the written requirements in this section.


### 7.3 Idempotency Design

Every cron job uses a Redis deduplication key with format a scheduled-task record identified by job and Singapore date. The key is written at the start of execution with a 24-hour expiry period. If the key already exists, the job exits immediately (preventing duplicate processing on restarts or multi-instance scenarios).

All database mutations are idempotent by design:
- Status transitions are guarded by when status EXPECTED PRIOR STATE.
- update records with the same target state on an already-transitioned record is a no-op.
- Notifications are guarded by flags (confirmation Overdue, etc.) to prevent duplicate sends.

---

## 8. Email Architecture

### 8.1 Provider

**Amazon Web Services SES (Simple Email Service), ap-southeast-1 region (Singapore).**

Amazon Web Services ap-southeast-1 ensures Singapore data residency for all email transmission metadata and bounce/complaint records. SES is Personal Data Protection Act-compatible with a signed DPA. SES supports SPF, DKIM, and DMARC on the sending domain.

### 8.2 Sending Domain Configuration


Implementation example omitted. This public edition retains the written requirements in this section.


### 8.3 Two-Tier Email Delivery

Email delivery is split into two tiers based on business criticality:

**Tier 1 — Critical Path (Direct SES Call, 5-minute service-level agreement):**

Triggered synchronously within the Spring Boot service method or cron job handler. Called before or immediately after the corresponding database transaction commits.

| Notification | Event |
|---|---|
| N-10 | Admin marks match ACTIVE → tutee first-session confirmation email |
| N-12 | 7-day confirmation timeout → tutee reminder |
| N-13 | 7-day confirmation timeout → admin alert |


Implementation example omitted. This public edition retains the written requirements in this section.


**Tier 2 — Non-Critical (Redis Streams with Consumer Groups):**

All other notifications (N-01, N-02, N-03 through N-09, N-11, N-14 through N-20) are published to a Redis Stream. The Spring @Async consumer reads from the stream and dispatches to SES. Delivery target: within 30 minutes of trigger.


Implementation example omitted. This public edition retains the written requirements in this section.


### 8.4 Notification Templates (N-01 through N-20)

Each template is stored in the private implementation folder (Thymeleaf HTML or plain HTML). The template receives a typed context object. No personal information of the other party is included in any notification.

| Template | Subject | Recipient |
|---|---|---|
| N-01 | "We've received your post" | Submitter |
| N-02 | "Your tuition request is now live" | Tutee |
| N-03 | "Your tutor listing is now live" | Tutor |
| N-04 | "Revision needed for your post" | Tutee |
| N-05 | "Revision needed for your listing" | Tutor |
| N-06 | "Update on your tuition request" | Tutee |
| N-07 | "Update on your tutor listing" | Tutor |
| N-08 | "Exciting news — a match has been found!" | Tutee |
| N-09 | "A tutee match has been identified for you" | Tutor |
| N-10 | "Please confirm your first tuition session" | Tutee |
| N-11 | "Your tuition arrangement is active" | Tutor |
| N-12 | "Reminder: Please confirm your first session" | Tutee |
| N-13 | "[ALERT] Tutee confirmation overdue: Match #id" | Admin |
| N-14 | (Internal system event — no email) | — |
| N-15 | "Commission Invoice — Payment Due in 14 Days" | Tutor |
| N-16 | "Your tuition match has been cancelled" | Tutee |
| N-17 | "The tutee match has been cancelled" | Tutor |
| N-18 | "Your post has expired" | Submitter |
| N-19 | "Your post has been closed" | Submitter |
| N-20 | "Important notice regarding your account" | User |

### 8.5 Bounce and Complaint Handling

Amazon Web Services SES → SNS → Spring Boot webhook at submit operation for webhooks ses-events:
- **Bounce:** Mark email as undeliverable in users email delivery status; alert admin for critical bounces (N-10, N-13).
- **Complaint:** Mark email as opted out of marketing. Alert admin.
- SNS endpoint validated using Amazon Web Services SNS signature verification in Webhook Controller implementation reference.

---

## 9. Security Architecture

### 9.1 BR-001 Enforcement: Contact Field Access Control

The most critical security requirement is BR-001: contact information (phone, email, home address) must never be exposed to non-admin users via any application programming interface surface.

**Enforcement layers (defense in depth):**


Implementation example omitted. This public edition retains the written requirements in this section.


### 9.2 cross-origin request permissions Strategy (ADR-003)

**cross-origin request permissions is not required** because the Cloudflare Worker proxies example com/api/* to the Railway Spring Boot backend under the same origin. The browser never makes a cross-origin request. Cors Config implementation reference disables Spring's cross-origin request permissions filter explicitly, with a comment documenting the dependency on the Cloudflare proxy being in place.


Implementation example omitted. This public edition retains the written requirements in this section.


### 9.3 cross-site request forgery protection Decision — amended by M19 and004

Use double-submit XSRF protection for cookie-authenticated mutations, including refresh; bootstrap/refresh the cross-site request forgery protection cookie after login and OAuth success and keep only documented narrow exemptions. Same Site is Strict is complementary, not a reason to disable the filter. Both frontend headers and backend allowed-origin/header configuration must agree. Production origin/cross-origin request permissions topology follows027. The original disabled-filter example is superseded and must not be reused.

### 9.4 Admin multi-factor authentication and Session Security

- time-based one-time passcode enforced on every admin login (see Section 4.5).
- 30-minute idle timeout implemented via Redis jti allowlist with sliding expiry period (see Section 4.4).
- Admin session bypass attempts return web request protocol 403 and are logged in the audit log (AC-07).
- Admin time-based one-time passcode secrets stored AES-256-GCM encrypted; encryption key in Railway environment variable.

### 9.5 Rate Limiting

Rate limiting runs in a Spring Once Per Request Filter that executes before controller methods. Bucket4j is deferred to post-MVP (ADR-014); MVP uses Redis sliding window counters directly.


Implementation example omitted. This public edition retains the written requirements in this section.


### 9.6 IDOR Prevention

Every post and match endpoint validates ownership or admin role before returning data:


Implementation example omitted. This public edition retains the written requirements in this section.


### 9.7 Immutable Audit Log

- PostgreSQL table in a separate schema with append records-only grants for the app user database role.
- No DELETE or update records is physically possible for app user on audit log.*.
- the log contact field access operation is called **before** returning contact field values — the log entry is written first. If the write fails (exception), the contact field is not returned (fail-safe).
- Audit log retained for 5 years; archived to S3 Glacier ap-southeast-1 after 2 years.
- Quarterly restore test: archive is restored to a test PostgreSQL instance and validated for completeness.

### 9.8 Content Security Policy and Security Headers

Applied via a Spring Once Per Request Filter on all responses:


Implementation example omitted. This public edition retains the written requirements in this section.


### 9.9 Input Validation

All Spring Boot controller methods have @Request Body parameters annotated with @Valid. Jakarta Bean Validation constraints on the request data transfer object class are evaluated before the method body executes. Validation failures return web request protocol 400 (mapped by @Controller Advice). No raw user input reaches the Java persistence mapping repository layer without prior validation.

### 9.10 database query language Injection Prevention

Spring Data Java persistence mapping + Hibernate uses parameterized queries for all database operations. No string-concatenated database query language is used. JPQL queries use named parameters (param). Native @Query database query language uses named parameters. Bitwise integer operations on availability bitmask use PostgreSQL integer arithmetic, not string interpolation.

---

## 10. Deployment Architecture

### 10.1 Infrastructure Stack


Implementation example omitted. This public edition retains the written requirements in this section.


**Infrastructure cost estimate:**
- Railway (PostgreSQL + Redis + Spring Boot): USD 20–45/month
- Cloudflare Pages: Free
- Cloudflare Workers: Free tier (100k req/day) or USD 5/month (Workers Paid)
- Amazon Web Services SES: <USD 1/month at MVP volumes
- Sentry: Free tier
- Total: ~USD 25–55/month

### 10.2 Cloudflare Worker Proxy Configuration


Implementation example omitted. This public edition retains the written requirements in this section.


The Railway Spring Boot service listens on its Railway-internal URL (api-internal railway.app). It is not exposed publicly — only via the Cloudflare Worker proxy. This is what enables same-origin delivery without cross-origin request permissions.

### 10.3 Environments


The illustration for environments contains the following information:

- Environment Branch Database URL.
- local feature/* Local PostgreSQL 16 localhost.
- Local Redis (docker-compose implementation reference).
- staging main Railway staging database staging example com.
- Railway staging Redis.
- production release tag Railway prod database example com.
- Railway prod Redis.


### 10.4 continuous integration/continuous deployment Pipeline


Implementation example omitted. This public edition retains the written requirements in this section.


**Coordinated deploy requirement:** Liquibase migrations MUST run and succeed before the new Spring Boot JAR starts accepting traffic. This is configured as a Railway Release Command, not a GitHub Actions step, so Railway's orchestration handles the ordering — if the migration fails, the new JAR deployment is aborted.

### 10.5 Secret Management

All secrets are stored as Railway environment variables (backend) or Cloudflare Pages environment variables (frontend). They are never committed to the codebase.

| Secret | Variable Name | Service |
|---|---|---|
| PostgreSQL connection string | DATABASE URL | Railway Spring Boot |
| Redis connection string | REDIS URL | Railway Spring Boot |
| signed session token signing secret (HS256) | signed session token SECRET | Railway Spring Boot |
| Encryption master key | ENCRYPTION MASTER KEY | Railway Spring Boot |
| keyed message authentication key | keyed message authentication KEY | Railway Spring Boot |
| Amazon Web Services access key ID | Amazon Web Services ACCESS KEY ID | Railway Spring Boot |
| Amazon Web Services secret access key | Amazon Web Services SECRET ACCESS KEY | Railway Spring Boot |
| S3 bucket name | S3 BUCKET NAME | Railway Spring Boot |
| SES sending domain | SES FROM ADDRESS | Railway Spring Boot |
| Sentry DSN | SENTRY DSN | Railway Spring Boot + CF Pages |
| Logtail source token | LOGTAIL SOURCE TOKEN | Railway log drain |
| Cloudflare Worker Railway URL | BACKEND URL | Cloudflare Worker |
| Vite application programming interface base URL | VITE application programming interface BASE URL | Cloudflare Pages build |

### 10.6 Observability (Required Before First User)

All of the following must be live and verified before the platform opens to real users:

| Signal | Tool | Alert Threshold |
|---|---|---|
| Java exceptions / React errors | Sentry | >10 errors/min → Pager Duty / email alert |
| Spring Boot health endpoint | Uptime check (/actuator/health) | >1 min downtime → alert |
| Email worker (Redis consumer) | Uptime check + Sentry heartbeat | Consumer process down → alert |
| SES bounce SNS webhook | Amazon Web Services SNS → webhooks ses-events interface operation | Must be live and verified |
| PostgreSQL connection pool | Railway metrics | >80% utilization → alert |
| Redis memory | Railway metrics | >80% memory → alert |
| Log drain | Logtail (Better Stack) from Railway | Must be receiving logs pre-launch |
| application programming interface error rate (5xx) | Logtail alert | >1% 5xx rate → email alert |

---

## 11. Availability and Disaster Recovery

### 11.1 Availability Targets

| Component | Target | Provider Basis |
|---|---|---|
| Public web app (/ and /tutors) | 99.5% monthly uptime | Cloudflare Pages (99.99% infrastructure); application errors not covered |
| Admin panel | 99.9% during Singapore time business hours (Mon–Sun 08:00–22:00 Singapore time) | Railway managed PostgreSQL service-level agreement + Cloudflare |
| Email delivery (N-10, N-12, N-13) | Within 5 minutes per product requirements document §14.2 | Amazon Web Services SES service-level agreement (99.9%) |

### 11.2 RPO and RTO

| Parameter | Value | Implementation |
|---|---|---|
| RPO | 24 hours | Railway automated daily backups |
| RTO | 4 hours | Railway restore from backup + Railway redeploy |
| Backup frequency | Daily automated | Railway managed backup |
| Backup retention | 30 days rolling | Railway backup policy |
| Backup storage region | ap-southeast-1 (Singapore) | Consistent with data residency requirement |
| Restore test | Quarterly | Full restore to Railway test instance; validate data integrity; document results |

### 11.3 Disaster Recovery Procedure


The illustration for disaster recovery procedure contains the following information:

- Disaster Scenario: Primary PostgreSQL database unrecoverable.
- 1. Detect: Railway database health alert fires.
- 2. Assess: Confirm primary instance is unrecoverable (< 30 min).
- 3. Restore: Railway one-click restore to most recent backup.
- (30 daily snapshots retained).
- 4. Validate: Run the documented operational check to confirm schema consistency.
- 5. Reconnect: Update DATABASE URL in Railway Spring Boot service variables.
- 6. Redeploy: Trigger Railway redeploy (< 5 min).
- 7. Verify: Smoke test — login, tutor listing, admin panel accessible.
- 8. Communicate: Notify stakeholders of incident and data loss window (up to 24h).
- Total elapsed time target: < 4 hours (RTO).
- Data loss window: up to 24 hours since last backup (RPO).


### 11.4 Redis Failure Handling

| Redis Function | Failure Impact | Mitigation |
|---|---|---|
| Admin jti allowlist | All admin users logged out | Re-login + multi-factor authentication required; no data loss |
| Tutee/tutor jti denylist | Logged-out tokens valid until exp | 24h max exposure; acceptable for MVP |
| Rate limiting | Rate limits unenforced temporarily | Acceptable for short outages |
| Email Streams (non-critical) | Non-critical emails paused until Redis recovers | Consumer group resumes from last ACK'd message position |
| Email (critical N-10/N-12/N-13) | Not affected — direct SES call, not streamed | No mitigation needed |
| Cron dedup keys | Cron jobs may run multiple times on recovery | Jobs are idempotent by design |

Railway Redis is deployed with persistence enabled (AOF) to prevent stream/key loss across restarts.

---

## 12. Singapore Compliance

### 12.1 Data Residency

All user data is processed and stored in Singapore-region infrastructure:

| Data | Storage | Region |
|---|---|---|
| User personal information, posts, matches, commissions | PostgreSQL (Railway) | ap-southeast-1 (Singapore) |
| signed session token jti keys, email streams | Redis (Railway) | ap-southeast-1 (Singapore) |
| Profile photos | Amazon Web Services S3 | ap-southeast-1 (Singapore) |
| Email transmission | Amazon Web Services SES | ap-southeast-1 (Singapore) |
| Application backups | Railway automated backups | ap-southeast-1 (Singapore) |
| Audit log archives | Amazon Web Services S3 Glacier | ap-southeast-1 (Singapore) |
| Error monitoring | Sentry (EU/US) | Requires signed DPA; personal information scrubbed before transmission |
| Log draining | Logtail / Better Stack | Requires DPA review; no personal information in logs |

**Cross-border transfer policy:** Any data transfer outside Singapore requires written approval from the designated Data Protection Officer (DPO) prior to implementation. This applies to all environments including staging and development environments handling real user data (business requirements document §10.7.2).

### 12.2 Personal Data Protection Act Compliance Architecture

#### Personal Data Inventory

All personal information fields are encrypted at rest as specified in business requirements document Appendix B:

| Field | Encryption | Access Log | Retention |
|---|---|---|---|
| email | AES-SIV (deterministic) | Yes (admin access) | Account duration + 2 years |
| phone number | AES-SIV (deterministic) | Yes (admin access) | Account duration + 2 years |
| full name | AES-256-GCM | Yes (admin access) | Account duration + 2 years |
| home address | AES-256-GCM | Yes (admin access) | Account duration + 2 years |
| profile photo | S3 server-sent events-S3 | Admin + self only | Account duration + 1 year |

#### Right of Erasure (Account Deletion)


Implementation example omitted. This public edition retains the written requirements in this section.


#### Right of Access (Data Export)

Admin-assisted data export at read operation for admin users id export generates a JSON package containing all data associated with the user's account. Encrypted fields are decrypted and included. The export is logged in the audit log.

#### Personal Data Protection Act Consent

Personal Data Protection Act consent is captured at registration:
1. A Personal Data Protection Act consent checkbox is presented on the React registration form with a link to the Privacy Policy.
2. Consent is sent to Spring Boot and stored in users pdpa consent at (timestamp) and users pdpa consent version (Privacy Policy version).
3. The Privacy Policy version is stored in platform settings; a new version requires re-consent on next login.

### 12.3 Analytics 90-Day Anonymization

The nightly active inventory snapshot job runs the anonymization pipeline on the analytics events table:

1. Selects all events where the event occurred more than 90 days ago and no anonymization time has been recorded, in this historical proposal. The current retention policy remains a separate decision.
2. Replaces user id, post id, match id, admin user id with non-reversible SHA-256 hashes (per-run salt — prevents cross-run linkage).
3. Removes IP addresses and session tokens from the properties JSONB column.
4. Sets the anonymization timestamp to the current time.

The pipeline targets ONLY the analytics schema. It has no database access to audit log.* or commission acknowledgements.

### 12.4 DPO Sign-Off Gate

Per business requirements document Appendix C, the Data Governance and Retention Policy must be reviewed and signed off by the designated Data Protection Officer before the platform opens to real users. This gate must be tracked as a go-live prerequisite in the project plan, separate from technical readiness.

---

## 13. Architecture Decision Records (ADR)

### ADR Summary Table

| # | Decision | Options Considered | Decision Made | Rationale | Date |
|---|---|---|---|---|---|
| ADR-001 | Frontend framework | React 18 + TypeScript (Vite) SPA vs. Next implementation reference App Router | **React 18 + TypeScript (Vite) SPA** | Decoupled frontend + Java backend; Vite fast dev loop; Cloudflare Pages static hosting fit; simpler mental model | 2026-05-27 |
| ADR-002 | Backend framework | Java 21 + Spring Boot 3.x vs. Node implementation reference + Nest JS | **Java 21 + Spring Boot 3.x** | Team JVM preference; compile-time type safety; mature ecosystem; scope controlled by deferring Map Struct, Bucket4j | 2026-05-27 |
| ADR-003 | cross-origin request permissions / origin strategy | cross-origin request permissions configuration vs. Cloudflare reverse proxy | **Cloudflare reverse proxy (same-origin delivery)** | Eliminates cross-origin request permissions entirely; Same Site is Strict cookies work; single origin for browser | 2026-05-27 |
| ADR-004 | signed session token signing algorithm | HS256 vs. RS256 | **HS256 with 256-bit secret** | No third-party verifiers; single issuer (Spring Boot); simpler key management; kid claim for rotation window | 2026-05-27 |
| ADR-005 | cross-site request forgery protection protection | Csrf Filter enabled vs. disabled | **Csrf Filter disabled** | Same Site is Strict cookie + same-origin via Cloudflare proxy prevents cross-origin cookie delivery entirely; precondition: ADR-003 must be in place | 2026-05-27 |
| ADR-006 | signed session token token lifetime | 24h TUTEE/TUTOR vs. refresh token vs. longer session | **24h TUTEE/TUTOR; 30-min idle ADMIN (Redis allowlist)** | 24h eliminates need for refresh token in MVP; admin idle timeout via Redis jti allowlist; forced re-login on idle for admin | 2026-05-27 |
| ADR-007 | Database provider | Railway managed PostgreSQL vs. Amazon Web Services managed relational database | **Railway managed PostgreSQL** | Co-located with Spring Boot (same Railway project, ap-southeast-1); simpler ops; included in project billing; managed relational database adds networking complexity | 2026-05-27 |
| ADR-008 | Schema migrations | Liquibase changesets vs. Hibernate auto-database structure definitions | **Liquibase; ddl-auto is validate in production** | Hibernate database structure definitions generation does not understand partial indexes, triggers, or custom ENUMs; Liquibase gives deterministic, version-controlled migrations | 2026-05-27 |
| ADR-009 | Email queue | Redis Streams (Spring consumer groups) vs. Bull MQ (Node implementation reference) | **Redis Streams with Spring consumer groups** | Same Redis instance already provisioned; avoids a separate Node implementation reference Bull MQ worker process; named streams, consumer groups, retry counter, failed-message queue stream | 2026-05-27 |
| ADR-010 | File storage | Amazon Web Services S3 vs. Cloudflare R2 | **Amazon Web Services S3 (ap-southeast-1)** | Singapore data residency confirmed; server-sent events-S3 native encryption; signed URL support; same Amazon Web Services account as SES simplifies IAM | 2026-05-27 |
| ADR-011 | Email provider | Amazon Web Services SES vs. Postmark vs. Send Grid | **Amazon Web Services SES (ap-southeast-1)** | Singapore data residency; signed DPA; lowest cost at MVP volumes; SPF/DKIM/DMARC support; SNS bounce/complaint webhooks | 2026-05-27 |
| ADR-012 | SEO for public pages | Client-rendered React (v1) vs. Cloudflare Pages prerender (v1) | **Client-rendered React in v1; prerender deferred to v2** | Modern Googlebot handles client-rendered React adequately for initial traffic; revisit after 3 months post-launch if organic ranking is below expectations | 2026-05-27 |
| ADR-013 | data transfer object mapping | Manual mapping (MVP) vs. Map Struct (MVP) | **Manual mapping in MVP; Map Struct in first maintenance sprint** | Map Struct adds compile-time annotation processing setup overhead; 2-person team benefits from deferring this; manual mapping is explicit and reviewable | 2026-05-27 |
| ADR-014 | Rate limiting | Redis sliding window (MVP) vs. Bucket4j (MVP) | **Redis sliding window in MVP; Bucket4j in first maintenance sprint** | Redis already in stack; sliding window is adequate for MVP rate limit volumes; Bucket4j adds library complexity without MVP-scale need | 2026-05-27 |
| ADR-015 | Contract testing | Pact consumer-driven contracts vs. no contract tests | **Pact required quality gate before any module ships** | React SPA defines consumer contracts; Spring Boot verifies them in continuous integration; prevents application programming interface drift between independently developed frontend and backend | 2026-05-27 |

---

### ADR-001: React 18 + TypeScript (Vite) SPA over Next implementation reference App Router

**Status:** Accepted

**Decision:** Vite SPA for all pages. React Router v6 for client-side routing. No SSR in v1.

**Context:** The team is building a decoupled frontend + Java backend architecture. Two options were evaluated: (1) React 18 + Vite SPA with Spring Boot REST application programming interface, (2) Next implementation reference App Router with Server Components and a separate Spring Boot backend.

**Why chosen:**
- User and team preference for a clean the private implementation folder separation with no framework-imposed frontend-facing service layer.
- Vite SPA builds to static files, ideal for Cloudflare Pages.
- Simpler mental model for a 2-person team: all server interactions are explicit Tan Stack Query hooks against Spring Boot REST endpoints.

**Trade-off accepted:** Loss of native SSR for /tutors and the tutor page identified by its public handle public pages. Mitigated by: modern Googlebot can crawl client-rendered React adequately for initial organic traffic. Cloudflare Pages prerender workers for these routes are deferred to v2. Revisit if organic ranking is below expectations after 3 months post-launch.

---

### ADR-002: Java 21 + Spring Boot 3.x over Node implementation reference + TypeScript (Nest JS)

**Status:** Accepted

**Decision:** Spring Boot 3.x with Java 21 runtime on Railway.

**Context:** Backend framework selection. Two options: (1) Java 21 + Spring Boot 3.x, (2) Node implementation reference + TypeScript + Nest JS (same language as frontend).

**Why chosen:**
- Team preference for JVM strong typing and the Spring ecosystem.
- Jakarta Bean Validation, Spring Security, Spring Data Java persistence mapping, and Liquibase are a well-integrated suite with a strong track record for data-sensitive applications.

**Trade-off accepted:** Slower MVP velocity for a 2-person team vs. a full-TypeScript stack. Mitigated by strict scope control: Map Struct deferred (ADR-013), Bucket4j deferred (ADR-014), no Spring Cloud, Spring Initializr defaults throughout.

---

### ADR-003: Cloudflare Reverse Proxy for Same-Origin Delivery

**Status:** Accepted

**Decision:** Route example com/api/* via a Cloudflare Worker to the Railway Spring Boot backend (https://api-internal railway.app/api/*). The Spring Boot backend is not publicly accessible except through this proxy.

**Context:** Without a proxy, the SPA at example com making requests to a Railway Spring Boot URL (e g., example com) would require cross-origin request permissions configuration. Same Site is Strict cookies would not work across different origins.

**Why chosen:**
- Eliminates cross-origin request permissions entirely — browser sees all requests as same-origin.
- Allows Same Site is Strict http Only cookies without any cross-origin cookie configuration.
- Single origin for browser: SPA and application programming interface are both example com.
- Disables cross-site request forgery protection risk entirely at the cookie transport layer.

**Trade-off accepted:** Additional Cloudflare Worker hop (~1–3ms per request). Acceptable.

---

### ADR-004: HS256 signed session token over RS256

**Status:** Accepted

**Decision:** HS256 with a 256-bit secret stored in signed session token SECRET Railway environment variable.

**Context:** signed session token signing algorithm selection for the auth token.

**Why chosen:**
- No third-party services need to verify tokens independently. Spring Boot is the single issuer and verifier.
- HS256 key management is simpler: one secret, one service.
- RS256 is appropriate when multiple services or third parties need to verify tokens using the public key without the signing secret. This does not apply here.

**Key rotation:** Manual on compromise. Rotation invalidates all active sessions (accepted). The kid header claim is included in all tokens to support a rotation window: during rotation, the Spring Boot signed session token filter accepts tokens signed by either the old or new key for 24 hours, then retires the old key.

---

### ADR-005: cross-site request forgery protection — superseded by M19/004

**Current decision:** double-submit XSRF protects cookie-authenticated mutations; fresh cross-site request forgery protection cookie after login/OAuth; refresh remains protected. The original Csrf Filter-disabled rationale is historical and withdrawn from current acceptance.

### ADR-006: signed session token lifetime — superseded by004

**Current decision:**30-minute access token; TUTEE/TUTOR opaque hash-only rotating refresh with idle/absolute caps and bounded grace. Admin retains its30-minute sliding session.004 defines precise duplicate/429/terminal responses and single-flight client recovery; no-refresh is not a current alternative.

### ADR-007: PostgreSQL on Railway over Amazon Web Services managed relational database

**Status:** Accepted

**Decision:** Railway managed PostgreSQL 16 in the same Railway project as the Spring Boot service.

**Why chosen:** Co-located with Spring Boot (same Railway project, ap-southeast-1 region), simpler operational model, included in Railway project billing. Amazon Web Services managed relational database adds VPC networking configuration, separate account setup, and cost overhead that is not justified at MVP scale.

---

### ADR-008: Liquibase over Hibernate Auto-database structure definitions

**Status:** Accepted

**Decision:** Liquibase changesets. spring jpa.hibernate ddl-auto is validate in all non-local environments.

**Why chosen:** Hibernate's database structure definitions generation does not understand partial indexes, custom PostgreSQL ENUMs, triggers, or row-level security policies. Liquibase gives deterministic, reviewable, version-controlled database query language migrations. Every partial index (including match cardinality enforcement) is declared in a Liquibase changeset file. ddl-auto is validate confirms the Java persistence mapping entities match the database schema but never modifies it.

**Enforcement:** The production application implementation reference file hard-codes spring jpa.hibernate ddl-auto is validate. A Railway Release Command runs the documented operational check before the new JAR starts. If migration fails, the deploy is aborted.

---

### ADR-009: Redis Streams for Email Queue over Bull MQ

**Status:** Accepted

**Decision:** Redis Streams with Spring consumer groups for non-critical email delivery.

**Why chosen:** The same Redis instance is already provisioned for jti keys and rate limiting. Redis Streams provide named job streams, consumer groups per job type, message acknowledgement, retry counter in message headers, and a failed-message queue stream for failed jobs. This avoids deploying a separate Node implementation reference Bull MQ worker process.

**What must be built:** Named streams (email-notifications, email-failed failed-message queue), consumer group per stream, retry counter in message metadata, dead-letter on 3 failures, Sentry alert on failed-message queue entry.

---

### ADR-010: Amazon Web Services S3 for File Storage

**Status:** Accepted

**Decision:** Amazon Web Services S3 in ap-southeast-1.

**Rationale:** Singapore ap-southeast-1 data residency, server-sent events-S3 encryption, pre-signed URL support with expiry period, 5 MB object size limit enforced server-side. Same Amazon Web Services account as SES simplifies IAM role management.

---

### ADR-011: Amazon Web Services SES for Email

**Status:** Accepted

**Decision:** Amazon Web Services SES in ap-southeast-1.

**Rationale:** Singapore data residency, SPF/DKIM/DMARC, SNS bounce/complaint webhooks, lowest cost at MVP email volumes. Signed DPA available from Amazon Web Services.

---

### ADR-012: Cloudflare Pages Prerender for SEO (Deferred to v2)

**Status:** Deferred

**Decision:** v1 ships without prerendering for public tutor pages. Client-rendered React is used for /tutors and the tutor page identified by its public handle.

**Rationale:** Modern Googlebot handles client-rendered React adequately for initial organic traffic. Prerender workers add build pipeline complexity not justified before initial user acquisition.

**Revisit trigger:** If organic search ranking for tutor listing pages is below baseline expectations after 3 months post-launch, implement Cloudflare Pages prerender workers for /tutors and the tutor page identified by its public handle.

---

### ADR-013: Map Struct Deferred to Post-MVP

**Status:** Deferred

**Decision:** Manual data transfer object mapping in MVP. Add Map Struct in the first maintenance sprint after launch.

**Rationale:** Map Struct requires annotation processing configuration in pom implementation reference and IDE setup overhead. For a 2-person team in an MVP sprint, explicit manual mapping in service/controller classes is faster to write and easier to review. Map Struct provides compile-time safety benefits worth adding once the data transfer object surface stabilizes post-MVP.

---

### ADR-014: Bucket4j Deferred to Post-MVP

**Status:** Deferred

**Decision:** Redis-backed sliding window counters for MVP rate limiting. Add Bucket4j in the first maintenance sprint.

**Rationale:** Redis is already provisioned. Sliding window counters are sufficient for the MVP rate limit volumes (10/hr registration, 200/min public listing). Bucket4j provides more sophisticated token bucket algorithms and better burst handling, worth adding when traffic patterns are better understood post-MVP.

---

### ADR-015: Contract Testing with Pact (Required Quality Gate)

**Status:** Accepted

**Decision:** Pact consumer-driven contract tests are a required quality gate before any module ships.

**How it works:**
- The React SPA (consumer) generates Pact contract files during test execution, specifying the application programming interface shapes it depends on.
- Spring Boot (provider) runs Pact provider verification in continuous integration, confirming all consumer contracts are satisfied.
- A continuous integration pipeline failure on Pact provider verification blocks the backend from merging.

**Why required:** The React SPA and Spring Boot are developed as separate codebases by a 2-person team. Without contract tests, application programming interface shape drift is detected only at integration time. Pact makes the contract explicit and enforced in continuous integration.

---

*End of Solution Architecture Document*

*Document Version 2.0 — Post Team Review (Spring Boot SPA Architecture) — 2026-05-27. Authored by Solution Architecture. References business requirements document v1.1 and product requirements document v1.1 (2026-05-27). Changes from v1.1: Complete rewrite to reflect correct stack (React 18 + Vite SPA + Spring Boot 3.x). Removed all Next implementation reference, Vercel, Prisma, Bull MQ, and Nest JS references. Added Cloudflare Pages + Worker proxy architecture, Spring Boot package structure, Liquibase changeset strategy, Redis Streams email consumer, Analytics Event Publisher design, Spring Security 6 signed session token filter chain, and 15 ADRs covering all architectural decisions. Architecture changes that affect the security model (Sections 4, 5.4, 9) or Personal Data Protection Act compliance (Section 12) require DPO review before implementation.*

## 2026-09-05 current contract and verification

| Concern | Current normative source / contract |
|---|---|
| Public browse | Anonymous filters allowed inside the limited result pool; detail/action authentication per current baseline |
| Assignment | PRD20/INT034 canonical tutor User Id, optional listing, atomic assignment/intake closure and active-match dedup |
| Contact | User email/phone stored per plaintext amendment C13; public personal information-free; authorized assigned participants only; frontend ACTIVE/COMPLETED visible, match-PENDING hidden. Approval alone never releases contact |
| Commission | PAID means agency receipt of tutor commission; first-session confirmation precedes collection; admin PAID completes match |
| Email | Founder-approved Resend; scoped durable outbox contracts; no full recipient/one-time passcode/body logging in production |
| Scheduler |007 registered jobs; durable generation/recipient intent and reconciliation required before reminder enablement; OFF/no backfill preserved |
| Deployment |027 is production authority; original Railway/Cloudflare/SES cost, command and diagram examples are historical |
| Audit | Folded INT066 Option A: in-transaction business audit; never swallowed best-effort audit failure |

See [current open decisions](../CURRENT-REQUIREMENTS.md) for genuinely unresolved decisions (including environment-specific multi-factor authentication or financial/retention exceptions). Nothing in an old descriptive diagram or status stamp resolves them. Reuse enduring data transfer object allowlists, Liquibase ownership, password hashing and authorization requirements; user email/phone follow the explicit plaintext-contact amendment (C13); verify any eventual implementation independently.
