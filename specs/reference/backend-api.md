# InTuition Backend — application programming interface Reference

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 correction:** This code-derived inventory does not approve policy changes. Current financial, contact-release, session, contact-change/revoke and error contracts are the numbered specifications and CURRENT-REQUIREMENTS md; implementation verification is owed for clarified contracts. [Current requirements](../CURRENT-REQUIREMENTS.md). No runtime verification was performed by this documentation correction.

> **Date:** 2026-06-03 — **code-derived** (read directly from the 26 @Rest Controller
> classes under the application component, their request/response record data transfer objects, the model
> enums, and Global error handler). This reference describes the code snapshot;
> approved current requirements govern intended behavior and any mismatch needs reconciliation. This revision (wave 2026-06-03) adds **INT-034** (free-market assign
> re-key), **INT-039** (interview timeslot negotiation), **INT-044** (job invitations),
> **INT-045** (analytics from/to + collected In Period Sgd), and **INT-025**
> (POST SUBMITTED admin fan-out). The prior 2026-06-01 revision added M15/M18; see
> "Corrections vs the prior doc" at the end for that history.

Spring Boot 3.3 / Java 21 REST application programming interface for the InTuition SG tuition marketplace. Handles
authentication, post management, approval workflow, tutor↔tutee matching, the free
marketplace (tutor applications + parent tutor-selection), commission tracking, interview
scheduling, the post-assignment contact reveal, notifications, favourites, reviews, public
showcase strips, and admin analytics.

## Tech Stack

| Component | Version |
|---|---|
| Java | 21 (required) |
| Spring Boot | 3.3 |
| Database | PostgreSQL 15 |
| Cache / Queue | Redis |
| Schema migrations | Liquibase (XML changesets) |
| Test coverage | JaCoCo (≥97.5% lines and branches) |
| Code formatter | Spotless — Google Java Format |

## Prerequisites

- **Java 21** — must be set as JAVA HOME. Java 22+ breaks Lombok annotation processing and JaCoCo instrumentation.
- **Maven 3.9+**
- **Docker Desktop** — running and accessible. Integration tests use Testcontainers to spin up PostgreSQL and Redis; the build will fail if Docker is not available.

### Setting JAVA HOME (mac OS / Homebrew)


Implementation example omitted. This public edition retains the written requirements in this section.


Add to your shell profile or prefix every the documented operational check command with the export.

## Running Tests


Implementation example omitted. This public edition retains the written requirements in this section.


Runs unit tests, integration tests (Testcontainers), and the JaCoCo coverage gate (97.5% line and branch). Docker Desktop must be running.

## Linting / Formatting


Implementation example omitted. This public edition retains the written requirements in this section.


## Running Locally


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Conventions

### Authentication & cookies

- All endpoints are prefixed api v1 interface operation.
- Auth is an **HttpOnly** signed session token cookie named auth (Same Site is Strict, Path=/). **spec 004** — its expiry period is now **1800s (30 min)** (jwt expiration-seconds is 1800; auth cookie Max-Age is 1800).
- **spec 004 — sliding refresh.** Non-admin login/register/multi-factor authentication-swap also sets a second HttpOnly cookie refresh token (Same Site is Strict, **Path is auth refresh interface operation** so the browser sends it ONLY to the refresh endpoint, **host-only** — no Domain, Max-Age = idle expiry period jwt refresh.idle-ttl-seconds, default 604800/7d). submit operation for auth refresh rotates both cookies. **ADMIN users get NO refresh cookie** (they keep the stateful sliding session); a refresh call in an admin context resolves to 401 REFRESH INVALID. logout and delete operation for auth account revoke the whole refresh family server-side and clear both cookies.
- multi-factor authentication-enabled login first sets a short-lived (max Age is 300) mfa session HttpOnly cookie scoped to Path is auth login mfa interface operation; submit operation for auth login mfa exchanges it for the auth cookie.
- cross-site request forgery protection: state-changing requests need the XSRF-TOKEN (bootstrap via read operation for auth csrf).
- Roles are TUTEE, TUTOR, ADMIN (super-admin is an is super admin flag on an ADMIN, not a separate role). Auth is layered: Security Config path matchers **plus** method-level @Pre Authorize.

### Pagination envelope (a page of matching records, with pagination details)


Implementation example omitted. This public edition retains the written requirements in this section.


Default page size is 20 on most paginated endpoints (read operation for admin matches defaults to 100). Query params: page, size. Since spec 010 the wrapper also carries an additive top-level trace Id (the results, pagination details and request trace identifier) — data/meta are unchanged; see [Trace id](#trace-id-spec-010).

### Error envelope

All errors are emitted by Global error handler as:


Implementation example omitted. This public edition retains the written requirements in this section.


retry After (seconds) is present only on rate-limit / lock responses. trace Id (spec 010) is
**always** present — see [Trace id](#trace-id-spec-010).

### Trace id (spec 010)

Every request carries a per-request **trace id** so a reported failure can be tied to its log line in
seconds. It is surfaced three ways, all sharing the **same value** for a given request:

- **X-Correlation-ID response header** — on **every** response (success, error, 204, redirect,
  binary). Set by request correlation filter; cross-origin request permissions-exposed so the SPA can read it cross-origin. This is the
  **universal carrier**.
- **trace Id in every JSON-object response body** — added **additively** (one top-level field;
  never an envelope, no existing field touched) by Trace Id response customization component. Success **objects** and
  pagination wrappers (Page Response the results and pagination details) get a top-level trace Id; **error** bodies get it
  **inside** the error object (an error containing its code, message and request trace identifier).
- Bodies that are **not** a JSON object — a top-level JSON **array/list**, a **String/primitive**, an
  **empty/204** body, a **binary/file** download, or a **Problem Detail** (RFC-7807, e g. the
  analytics-collect 422) — are returned **untouched** and carry the id on the X-Correlation-ID
  **header only**.

The body trace Id always equals that response's X-Correlation-ID header and the correlation Id on
the request's log line. To quote it in a bug report: copy the Reference value shown on the frontend error
toast, or read X-Correlation-ID from the response. The id is **not** shown on successful frontend toasts.

> **Known gap.** Security **401 / 403** entry-point responses (authentication-entry-point and
> access-denied handlers) carry the X-Correlation-ID **header** but **no body trace Id** — they are
> emitted before Global error handler runs, so the body-advice never sees them. Read the header to
> correlate these.

| web request protocol | code | Trigger |
|---|---|---|
| 400 | VALIDATION ERROR | Bean-validation failure, unreadable body, or query-param type mismatch. A bean-validation (Method Argument Not Valid Exception) failure additionally carries a **field Errors** map (Record<string,string> of field → message, first error per field) inside the error body — e g. **create-admin** (submit operation for admin admins with a bad email/blank field) returns 400 VALIDATION ERROR with per-field field Errors the frontend binds to inputs |
| 404 | NOT FOUND | **spec 016 REQ-002.** A request to a path with **no matching handler** (No Handler Found Exception / No Resource Found Exception) → standard error body an error containing its code, message and request trace identifier. Previously fell through to a 500. A WRONG METHOD on an EXISTING path still resolves a handler → **405**, not this. (Domain * NOT FOUND codes — e g. POST NOT FOUND — are thrown via App Exception not Found and are distinct from this no-handler row.) |
| 400 / 401 / 403 / 404 / 409 / 422 |  using domain code_ | Thrown via App Exception.* (e g. bad Request, unauthorized, forbidden, not Found, conflict, unprocessable Entity) — the code is the domain string supplied at the throw site |
| 403 | FORBIDDEN | Spring Access Denied Exception (role/authz denial) |
| 409 | JOB ALREADY ASSIGNED | M18 optimistic-lock race on a Tutee Post @Version (two admins assigning the same ACTIVE job). **Unchanged by INT-087.** |
| 409 | COMMISSION ALREADY PAID | **INT-087 F4.** Optimistic-lock race on the Commission @Version (migration 0048) — two distinct-key concurrent pay Commission calls on one PENDING commission. The commission is paid **exactly once**; the loser gets this 409 |
| 409 | CONCURRENT MODIFICATION | **INT-087 F4.** Generic @Version optimistic-lock fallback (Object Optimistic Locking Failure Exception) for any entity without an entity-specific code — retry with a fresh read |
| 409 | CONSTRAINT VIOLATION | database integrity violation (generic) |
| 422 | UNKNOWN SUBJECT OR LEVEL | **spec 009 + spec 020 (pair-aware).** A subject Code/level Code that is unknown **or an incoherent subject+level pair** — the subject and level must share a level group, **plus the N LEVEL → O LEVEL widen** (N-Level subjects, level group is N LEVEL, are valid against the O LEVEL secondary years — N using A/N using T sit those years, there are no N LEVEL level rows). Enforced on **tutee-post create/update** (POST/replace operation for tutee posts), **tutor listing** (POST/replace operation for tutor posts), and **tutor profile** (replace operation for tutor profile, register onboarding). Verify Taxonomy Repository is Valid Subject Level Pair (single pair) + is Coherent Subject Level Set (multi-pair set). Previously an incoherent pair surfaced as a **500** — now a typed 422 |
| 422 | INVALID STATUS TRANSITION | database-trigger-enforced illegal interview status transition |
| 422 | IDEMPOTENCY PARAMS DIFFER | **INT-087 F5 (Stripe-parity).** An Idempotency-Key was reused with a **different request body** — the stored fingerprint of the first request does not match this one. The action is **NOT** re-run |
| 422 | APPLICATION LIMIT REACHED | INT-077 — tutor already holds the max active applications (quota applications-per-tutor, default 10; active = APPLIED+SHORTLISTED across all jobs). Withdraw one to free a slot |
| 422 | INVITE LIMIT REACHED | INT-077 — tutee already holds the max active (PENDING) invitations on this job posting (quota invites-per-post, default 10). Withdraw one to free a slot |
| 422 | INTERVIEW LIMIT REACHED | Tutee already holds the max active interviews on this posting (cap = **3**, not 10; pre-existing, race-hardened by INT-077). Value has no ERR prefix — the backend house style (constant ERR XXX → value XXX), normalized across all limit-reached codes in INT-079 |
| 422 | PHONE NOT VERIFIED | **INT-082 (default OFF).** A phone-verification-gated action (contact-reveal / select-tutor / invite) was attempted while the gate is **ACTIVE** and the signed session token user's phone is unverified. Inert (never thrown) while the feature is OFF |
| 422 | INVALID one-time passcode | **INT-082.** WhatsApp one-time passcode confirm with a wrong or expired code |
| 422 | one-time passcode ATTEMPTS EXCEEDED | **INT-082.** WhatsApp one-time passcode confirm after 5 failed attempts on the live token; re-request a new one-time passcode |
| 422 | NO PHONE ON ACCOUNT | **INT-082.** one-time passcode start/confirm attempted with no phone stored on the account |
| 409 | PHONE ALREADY REGISTERED | An E.164 phone already on another account. Current sites: **register** (submit operation for auth register * — pre-check + TOCTOU unique-index backstop); **spec 012** submit operation for account contact-change verify (uniqueness re-checked AT COMMIT). Same code + message across current entry points; the historical INT-082 direct-phone route is superseded by 012 |
| 409 | EMAIL ALREADY REGISTERED | **spec 012.** submit operation for account contact-change verify confirming an EMAIL change whose new value collides with another account at commit (TOCTOU-safe uniqueness re-check) |
| 422 | STEP UP REQUIRED | **spec 012.** submit operation for account contact-change with a wrong/absent current Password — the mandatory step-up re-auth failed; the contact change is not initiated |
| 400 | CONTACT CHANGE TOKEN INVALID | **spec 012.** submit operation for account contact-change verify with a token that is wrong, expired, or already consumed (bcrypt-hash mismatch). Confirm is **POST-only** (anti email-scanner-prefetch) |
| 423 |  using domain code_ | the locked operation — account lockout, carries retry After |
| 401 | REFRESH INVALID / REFRESH EXPIRED / REFRESH REUSED | **spec 004 (Auth Error Code).** submit operation for auth refresh rejection — invalid/unknown/admin/post-reset family, idle/absolute expiry period elapsed, or a beyond-grace replayed token (family revoked). See [Session refresh](#session-refresh) |
| 403 | ACCOUNT BLOCKED | **spec 004 (Auth Error Code).** submit operation for auth refresh (or login) by an administratively-blocked user |
| 429 | RATE LIMITED | Per-IP rate limit exceeded (showcase / public assignments; **spec 004** submit operation for auth refresh over the per-IP limit); retry After = the relevant window (3600s default; the refresh window for refresh) |
| 429 | RATE LIMIT EXCEEDED | **INT-082.** WhatsApp one-time passcode send rate-limit hit — per-user Id, ip sliding window (ratelimit whatsapp-otp.*, default 3/3600), the per-user daily cap (whatsapp otp.daily-cap-per-user, default 10), or the global send-ceiling; carries retry After |
| 500 | INTERNAL ERROR | Unhandled exception |

> Domain codes seen in the M15/M18 services include TUTOR PROFILE INCOMPLETE (422),
> JOB ALREADY ASSIGNED (409), and FORBIDDEN (403 IDOR/owner check).

### Model enums (referenced by data transfer objects)

| Enum | Values |
|---|---|
| User Role | TUTEE, TUTOR, ADMIN |
| Post Status | DRAFT, PENDING, ACTIVE, REVISION REQUESTED, REJECTED, MATCHED, CLOSED, EXPIRED |
| Match Status | PENDING, ACTIVE, COMPLETED, CANCELLED |
| Commission Status | (invoice status on commission data transfer objects) |
| Interview Status | REQUESTED, **PROPOSED**, **COUNTER PROPOSED**, **TUTOR DECLINED**, **PENDING SCHEDULING**, SCHEDULED, COMPLETED, CANCELLED, NO SHOW, **EXPIRED** (bold = INT-039 additions, migration 0034) |
| Job Application Status | APPLIED, WITHDRAWN, SHORTLISTED, REJECTED, MATCHED, **SUPERSEDED** (match), **CLOSED** (INT-062 — job withdrawn; migration 0038) |
| Tutor Selection Request Status | CREATED, CONSUMED, SUPERSEDED, EXPIRED |
| Tutor Commitment | PART TIME, FULL TIME |
| Start Window | WITHIN 2 WEEKS, WITHIN A MONTH, FLEXIBLE |
| Job Invitation Status | PENDING, ACCEPTED, DECLINED, WITHDRAWN, EXPIRED (match-filled; INT-044, migration 0035), **CLOSED** (INT-062 — job withdrawn; **Java-enum only**, job invitations status is the varchar operation not a PG enum, so no ALTER TYPE — 0039 is a documented no-op) |
| Decline Reason | NOT INTERESTED, CANT MAKE IT (INT-039, interview decline reason PG type) |
| Others | Location Region, Venue Type, Group Size, Frequency Per Week, Post Type |

---

## M1 — Auth (auth interface operation) — Auth Controller

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | /auth/register/tutee | Public | Register tutee; sets auth cookie; **201** the signed-in user’s account details |
| POST | /auth/register/tutor | Public | Register tutor; sets auth cookie; **201** the signed-in user’s account details |
| POST | /auth/login | Public | Login. **200** the signed-in user’s account details, or confirmation that multi-factor authentication is required + mfa session cookie |
| POST | /auth/login/mfa | mfa session cookie | Verify time-based one-time passcode, swap for auth cookie; **200** the signed-in user’s account details |
| POST | /auth/refresh | refresh token cookie (permit All, runs anonymously) | **spec 004** — sliding refresh. Rotates the auth (+ refresh token, unless a grace duplicate) cookie; **200** a success confirmation. Per-IP rate-limited. On any failure clears BOTH cookies and returns a typed Auth Error Code (401/403/429) |
| POST | /auth/logout | Authenticated | signed session token denylist + clear cookie + **revoke refresh family** (spec 004); **200** a success confirmation |
| GET | /auth/me | Authenticated | Current user → Me Response |
| GET | /auth/csrf | Public | Bootstrap XSRF-TOKEN cookie; **204** |
| POST | /auth/forgot-password | Public | Send reset one-time passcode email; **200** a success confirmation |
| POST | /auth/reset-password | Public | Apply new password via one-time passcode; **200** a success confirmation |
| DELETE | /auth/account | Authenticated | Soft-delete own account (body confirms password); **200** a success confirmation |

**Requests**
- Register Tutee Request / Register Tutor Request: email (≤254, email), phone (SG mobile +65 8/9xxxxxxx), password (8–128; upper+digit+special), display Name (2–80).
  - **spec 001 — consent (both register paths).** Three **required** consent booleans, each @Assert yes → a no/absent value raises **400 VALIDATION ERROR**: terms Accepted (Terms of Use + Privacy Notice), pdpa Consent (Personal Data Protection Act collection/use of personal data), age Confirmed (≥18 or has parent/guardian consent). Plus one **optional** marketing Opt In (bool; no is valid). On success the server stamps the * consent at timestamps + terms version/privacy version, and writes a CONSENT GIVEN audit row (migration **0058** — 10 columns + 4 require constraints).
  - Register Tutor Request also carries qualifications (≤1000, optional), years Experience (Integer, optional), commission Acknowledged (@Assert yes → 400), commission Terms Version (required).
  - **spec 002 — OPTIONAL tutor onboarding-capture** (Register Tutor Request only; all optional, omitting them leaves registration unchanged per INT-043): hourly Rate Min Sgd/hourly Rate Max Sgd (1–1000), location Planning Areas (a list of text values, ≤30 items, each ≤64), location Region (Location Region), commitment (Tutor Commitment), subject Levels (a list of subject-and-level pairs, ≤200). Cross-field hourly Rate Max Sgd ≥ hourly Rate Min Sgd (when both present) and subject/level-code validity are enforced in Auth Service register Tutor → **422**. _(Subject Level Dto: subject Code (≤50), level Code (≤50).)_
- Login Request: email, password.
- Mfa Verify Request: totp (6 digits).
- Forgot Password Request: email.
- Reset Password Request: email, otp (6 digits), new Password (8–128).
- Delete Account Request: confirm Password.

**Auth User Response**: id (unique identifier), role, display Name, is Super Admin (bool), password Change Required (bool).
**Me Response**: id, role, display Name, email, phone, created At, is Super Admin, password Change Required.

#### Password reset — /auth/forgot-password + /auth/reset-password (spec 003)

submit operation for auth forgot-password always returns **200** a success confirmation regardless of whether the email
matches an account (anti-enumeration). When it matches, a 6-digit one-time passcode + reset link are minted and emailed;
the one-time passcode is valid for one-time passcode EXPIRY MINUTES (15).

submit operation for auth reset-password returns **distinct typed errors** (spec 003 corrected the prior single-message
collapse — the frontend blanket-catch that flattened every failure to "Invalid or expired code" was the root cause):

| Status | Code | When |
|---|---|---|
| 200 | — | a success confirmation — valid one-time passcode + compliant password; password is reset and password change required is cleared (user logs in with the new password, NOT bounced to forced-change) |
| 400 | INVALID one-time passcode | one-time passcode is wrong or expired. **Unknown-account ≡ wrong-one-time passcode — the two responses are byte-identical** (anti-enumeration: a non-existent email is indistinguishable from a bad code) |
| 400 | VALIDATION ERROR | Malformed request — one-time passcode not 6 digits, or new Password fails policy (8–128; upper+digit+special) |
| 429 | one-time passcode ATTEMPTS EXCEEDED | 5 failed one-time passcode attempts on the live token (Redis counter, survives the wrong-one-time passcode tx rollback — INT-088); re-request a new one-time passcode.  using Soft activity oracle — see security note._ |
| 429 | RATE LIMIT EXCEEDED | Per-IP/endpoint reset rate-limit hit; identical response for existing vs non-existing accounts (no enumeration oracle) |

**one-time passcode debug logging — otp debug-log enabled (founder directive D8).** A single backend kill-switch flag
the setting that controls diagnostic logging during passcode generation (base default **TRUE**) logs, at one-time passcode-generation time, the
**plaintext reset one-time passcode** (with the full recipient email + validity window + reset link) AND the plaintext
**WhatsApp phone-verification one-time passcode** (recipient phone + validity), in **ALL profiles including prod**. Greppable
markers: one-time passcode-DEBUG-LOG reset / one-time passcode-DEBUG-LOG whatsapp. This deliberately overrides the codebase's
never-log-one-time passcode invariant (recorded as an accepted-risk ADR under spec 003).
> ⚠️ **SECURITY:** deployed logs hold plaintext OTPs + recipient personal information — a **reset-hijack vector**. Anyone with
> log access can complete an account takeover. **Remove or revisit (flip the kill-switch OFF) before public
> launch.**

**Local test hook.** submit operation for regression password-reset issue-otp (the profile operation,
Regression Auth Controller) issues a real reset one-time passcode for a regression-band email and returns the plaintext — see
[Local-only Regression Fixtures](#local-only-regression-fixtures).

#### Session refresh

permit All and **runs anonymously** (it is called precisely when the auth cookie has expired). Reads the
host-only, path-scoped refresh token cookie (Path is auth refresh interface operation), validates + rotates the refresh
family, and re-mints the **30-minute** auth cookie. **Per-IP rate-limited** (jwt refresh.ratelimit limit,
default 30 / jwt refresh.ratelimit window-seconds, default 60s).

| Status | Code (Auth Error Code) | When |
|---|---|---|
| 200 | — | a success confirmation — re-mints the auth cookie; rotates refresh token too (unless a grace-window duplicate) |
| 401 | REFRESH INVALID | no/garbage/unknown-family cookie, an **admin** context, missing creation-epoch (fail-safe), or a post-reset family |
| 401 | REFRESH EXPIRED | idle expiry period elapsed, or the absolute cap reached on an otherwise-valid rotation |
| 401 | REFRESH REUSED | a beyond-grace rotated token replayed — the whole family is **revoked before** the 401 |
| 403 | ACCOUNT BLOCKED | the user is administratively blocked (mirrors the login path) |
| 429 | RATE LIMITED | over the per-IP refresh limit; carries retry After (the refresh window, not the 3600s default). No mint/rotate |

The codes ride the standard an error containing its code and message envelope. On **any** failure the server clears BOTH
the auth and refresh token cookies. **ADMINs are never issued a refresh cookie** (REQ-010) — an admin refresh
falls through to 401 REFRESH INVALID.

---

## M2 — Tutee Posts (tutee posts interface operation) — Tutee Post Controller · role TUTEE

| Method | Path | Purpose |
|---|---|---|
| POST | /tutee/posts | Create + submit (→ PENDING); **201** Tutee Post Response |
| GET | /tutee/posts | List own posts → a list of tutee post entries |
| GET | the specified post accessible to the parent | Own post detail |
| PUT | the specified post accessible to the parent | Update; editable from DRAFT/REJECTED/ACTIVE/**PENDING** (EDITABLE allow-list). **spec 021/REQ-003:** a **PENDING** post is editable → **200**, **stays PENDING** and re-enters the approval queue (re-stamps submitted At); an ACTIVE edit re-queues to PENDING; DRAFT/REJECTED edit in place. MATCHED/CLOSED/EXPIRED rejected |
| DELETE | the specified post accessible to the parent | Soft-delete (DRAFT or REJECTED only); a success confirmation |
| POST | closing the specified post as the parent | **INT-062** — withdraw/close a **LIVE** job (ACTIVE/PENDING) → CLOSED; cascade-closes its open applications (→CLOSED) + pending invitations (→CLOSED), notifies them. 409 INVALID STATE on MATCHED/CLOSED/DRAFT/REJECTED; 403 non-owner. Idempotent |

**Create Tutee Post Request**: subject Code, level Code, num Students (**1–20**; the max operation, over → 400 VALIDATION ERROR), budget Min Sgd/budget Max Sgd (1–1000), venue Preferences (a list of venue types, ≥1), availability Bitmask (1–2097151), session Duration Hours (Big Decimal), frequency Per Week (enum), preferred Start Date (optional Local Date), special Requirements (≤500, optional), location Mrt Station (≤100), location Planning Area (≤100). Update Tutee Post Request mirrors it.

> **spec 009 + spec 020 — taxonomy + pair-aware validation.** subject Code/level Code are validated against the seeded taxonomy (migration 0064 — 34 subjects / 10 levels incl. the distinct N LEVEL level group; **canonical corrections in migration 0067**, below). Validation is **pair-aware** (Taxonomy Repository is Valid Subject Level Pair): the subject and level must share a level group, **plus the N LEVEL → O LEVEL widen** (an N-Level subject is coherent with O LEVEL secondary years — N using A/N using T sit those years, and there are no N LEVEL level rows). An unknown code **or an incoherent pair** returns **422 UNKNOWN SUBJECT OR LEVEL** (previously a 500) on both the create and update paths.
>
> **spec 020 (migration 0067) — canonical taxonomy corrections.** Additions: **Combined Humanities (O-Level)** (HUMANITIES OLEVEL, O LEVEL); **H1 sciences (A-Level)** (PHYS H1 ALEVEL/CHEM H1 ALEVEL/BIO H1 ALEVEL, A LEVEL); **N using T secondary years** S1 NT–S4 NT (O LEVEL group, alongside the existing N using A years). The generic **"Mathematics (O-Level)"** (MATH OLEVEL) is **deprecated** — soft-hidden via is active no (never deleted; FK-safe, dropped from the frontend picker; the validation join does NOT filter on is active, so existing posts referencing it stay valid). Use E-Math (EMATH OLEVEL) / A-Math (AMATH OLEVEL) instead. append records-only / when a duplicate is found leave the existing record unchanged — idempotent; no new level group enum value needed.

**Tutee Post Response** (personal information-free — no tutee id/email/phone/name): id, subject Code, level Code, num Students, budget Min Sgd, budget Max Sgd, venue Preferences (a set of distinct venue types), availability Bitmask, session Duration Hours (Big Decimal), frequency Per Week, preferred Start Date, location Mrt Station, location Planning Area, location Region, status (Post Status), created At.

## M2 — Tutor Posts (tutor posts interface operation) — Tutor Post Controller · role TUTOR

| Method | Path | Purpose |
|---|---|---|
| POST | /tutor/posts | Create + submit (→ PENDING); **201** Tutor Post Response |
| GET | /tutor/posts | List own posts → a list of tutor post entries |
| GET | the specified post accessible to the tutor | Own post detail |
| PUT | the specified post accessible to the tutor | Update (DRAFT only; ACTIVE re-enters PENDING; PENDING blocked) |
| PATCH | changing availability on the specified post as the tutor | M14a — toggle public "Available" chip; body states whether the tutor is available → Tutor Post Response |
| DELETE | the specified post accessible to the tutor | Soft-delete (DRAFT or REJECTED only); a success confirmation |

**Create Tutor Post Request**: title (≤100), headline (≤100, optional), subjects (a list of text values, 1–10), levels (a list of text values, 1–8), hourly Rate Min Sgd/hourly Rate Max Sgd (1–1000), venue Types (a list of venue types, ≥1), group Sizes (a list of group sizes, ≥1), qualifications (≤500), years Experience (0–50), teaching Approach (≤500, optional), location Planning Area (≤100). Update Tutor Post Request mirrors it (all fields optional / partial-update).

> **spec 002 — availability slots + multi-MRT (migrations 0059/0059b, 0060).** Two field-pairs each accept a NEW canonical form alongside the RETAINED-but-deprecated legacy field:
> - **Availability:** availability Slots — a list of availability slots, the new fine-grained 1-hour source. Each slot: day Of Week (Short, 0–6 — 0 = Mon … 6 = Sun), start Hour (Short, 0–23, Singapore time local clock), duration Min (Short, ∈ 60, 120). Per-field range failures → **400 VALIDATION ERROR**; cross-field rules (no overrun past midnight, no duplicate day, start Hour) → enforced in Tutor Post Service. The legacy availability Bitmask (Integer, 0–2097151; deprecated) is still accepted and is **derived** from the slots when a new client sends slots. Tutor Post Service requires the bitmask **OR** the slot list to be non-empty.
> - **MRT:** location Mrt Stations — a list of text entries (≤50 items, each ≤100), the new multi-select. On update, no value = leave unchanged, a provided list (incl. empty) = replace-the-set. The legacy single location Mrt Station (≤100; deprecated) is still accepted and kept in sync.

**Tutor Post Response** (personal information-free — no user Id/email/phone/real name): id, **tutor Public Handle** (M18 — the ONLY identity; pseudonymous handle for the public browse card — the real display Name is intentionally omitted, INT-003), **commitment** (M18 — Tutor Commitment, nullable), title, headline, subjects (Set), levels (Set), hourly Rate Min Sgd, hourly Rate Max Sgd, venue Types (Set), group Sizes (Set), availability Bitmask, **availability Slots** (spec 002 — a list of availability slots; echoed on single-post/detail paths only, empty on list paths), qualifications, years Experience, teaching Approach, location Mrt Station, **location Mrt Stations** (spec 002 — a list of text entries; single-post paths only), location Planning Area, location Region, status, created At, average Rating (nullable), review Count (nullable), subject Levels (INT-153 — capability matrix; single-post paths only, empty on list).

## M2 / M10 / M17 — Public Browse (public interface operation) — Public Browse Controller

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | /public/tutors | Public | Browse ACTIVE tutor posts → a page of tutor post records, with pagination details. Filters: subject, level, region, min Rate, max Rate, venue (repeatable Venue Type), group Size, availability (bitmask), page, size |
| GET | /public/assignments | Public | M17 — anonymised Browse Assignments → a page of showcase records, with pagination details. Filters: subject, level, region, page, size. Per-IP rate-limited (60/min); **anonymous cap**: page ≤ 4, size ≤ 20 (100-result ceiling) — out-of-bounds returns an empty page (not 4xx). personal information-free projection |
| GET | /public/requests | TUTOR or ADMIN | Browse ACTIVE tutee requests → a page of tutee post records, with pagination details (full, MRT-level). TUTOR must own ≥1 ACTIVE post (admins bypass). Filters: subject, level, page, size |

> The anonymous read operation for tutors id detail endpoint is **removed** (M17); tutor detail now lives at the authenticated read operation for tutors id (see Marketplace Detail). read operation for tutors id reviews remains public (M11).

## M17 — Marketplace Detail (api v1 interface operation) — Marketplace Detail Controller · the is authenticated operation

| Method | Path | Purpose |
|---|---|---|
| GET | the specified tutor accessible to the user | Full ACTIVE tutor listing detail → Tutor Post Response. 404 if non-existent/non-ACTIVE (anti-enumeration) |
| GET | the specified tuition request accessible to the user | Full ACTIVE assignment detail → Tutee Post Response. 404 if non-existent/non-ACTIVE |

Anonymous → 401; authenticated → 200.

---

## M3 — Admin Post Approval (admin posts interface operation) — Admin Post Controller · role ADMIN

| Method | Path | Purpose |
|---|---|---|
| GET | /admin/posts/pending | Merged pending tutor+tutee queue (personal information-free) → a list of admin queue item entries |
| GET | /admin/posts/active | **INT-063** — paginated ACTIVE tutee jobs (newest first, personal information-free) → a page of admin queue item records, with pagination details. Powers the admin **Jobs** surface (/admin/posts frontend) so an admin can find + close any live job. NOTE: **paginated** (the results and pagination details), unlike /pending (flat list) |
| GET | the specified post accessible to an administrator | Full detail incl. **authorized contact personal information** → Admin Post Detail Response. 404 if not found |
| POST | approving the specified post as an administrator | Approve (→ ACTIVE) → Admin Post Detail Response |
| POST | rejecting the specified post as an administrator | Reject (→ REJECTED) with mandatory reason. Body contains the mandatory rejection reason. 400 if blank |
| POST | closing the specified post as an administrator | **INT-062** — admin withdraw/close any LIVE job (ACTIVE/PENDING) → CLOSED + same cascade as the tutee close. Errors (the frontend Jobs surface treats both as "list stale → refetch + guide", INT-063): **409 INVALID STATE** (already MATCHED/CLOSED/DRAFT/REJECTED), **404 NOT FOUND** (deleted between render and click; INT-137b aligned status with the body code — was 400) |

**Admin Queue Item Response** (no personal information): id, post Type (Post Type), subjects (List), level Display, rate Display, submitter Display Name, submitted At (Instant), status.
**Admin Post Detail Response** (admin-only personal information): queue fields + submitter Email, submitter Phone, rejection Reason, and the full set of tutor-post / tutee-post detail fields (null on the non-applicable side).

---

## M4 — Admin Matching (admin matches interface operation) — Admin Match Controller · role ADMIN

| Method | Path | Purpose |
|---|---|---|
| GET | /admin/matches | List (matched At desc), commission-enriched → a page of admin match item records, with pagination details (default size **100**) |
| POST | /admin/matches | Create match. Body Create Match Request; **201** Match Response. 404 missing post; 409 existing PENDING/ACTIVE pair; 422 post not ACTIVE |
| POST | activating the specified match as an administrator | Activate PENDING + create commission. Body contains the agreed monthly fee in Singapore dollars, between 1 and 50,000 → Admin Match Item Response. 422 if not PENDING |
| POST | cancelling the specified match as an administrator | Cancel PENDING/ACTIVE → Admin Match Item Response. 422 if COMPLETED/already CANCELLED |

**Create Match Request** (INT-034 — migration 0033): Exactly **one** of tutor Post Id or winning Tutor Id must be supplied (cross-field @Assert yes; both-null or both-set → 400).

| Field | Required | Description |
|---|---|---|
| tutor Post Id | one-of | unique identifier — legacy listing-based assign |
| winning Tutor Id | one-of | unique identifier — free-market assign by tutor user id (no listing) |
| tutee Post Id | required | unique identifier |
| source Selection Request Id | optional | unique identifier — ties this assign to a parent choose-tutor request |

**Match Response**: id, tutor Post Id (nullable for listing-less matches), tutee Post Id, status (Match Status), matched At.
**Admin Match Item Response**: id, tutor Subject, tutee Subject, status, matched At, commission (Admin Commission Item Response or null) — **+ INT-053 admin tutor↔tutee+job enrichment:** tutor User Id, tutor Display Name (null if no profile), tutor Public Handle, tutor Post Id (null for free-market/listing-less matches), tutee User Id, tutee Post Id, tutee Name, job Title. Admin-only; the list query (Match Repository find All For Admin List) uses an @Entity Graph fetch-join + batch profile fetch (no N+1).

### INT-034 — Free-market assign from selection request — Admin Selection Request Controller · role ADMIN

Added by INT-034 to the existing Admin Selection Request Controller (which previously only listed).

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | admin selection-requests id assign interface operation | ADMIN | Assign the tutor named in the selection request by reading tutor User Id + tutee Post Id server-side. **201** Match Response. 404 if request not found; 422 if not CREATED; 409 on duplicate/optimistic-lock |

Path constant: assigning a tutor from the specified tutor-selection request as an administrator.

The frontend sends only the selection-request the selection-request identifier — no tutor Post Id or winning Tutor Id in the body. The server resolves the tutor from the stored request, enabling the "Assign" button on listing-less applicants that was previously disabled.

---

## M5 — Admin Commission (admin commissions interface operation) — Admin Commission Controller · role ADMIN

| Method | Path | Purpose |
|---|---|---|
| POST | recording agency receipt of the specified commission as an administrator | Record agency receipt of tutor commission as PAID + complete the match → Admin Commission Item Response. 404 not found; 409 already PAID |

**Admin Commission Item Response**: id, monthly Fee Sgd (int), commission Rate (double), commission Amount Sgd (int), invoice Status (Commission Status), paid At.

---

## M6 — Interviews — Interview Controller

> **INT-039 (migration 0034)** rewrites the interview initiation from a single-step
> Create Interview Request to a two-party timeslot negotiation. All existing admin scheduling
> and completion endpoints are unchanged; submit operation for interviews now takes
> Propose Interview Request and the controller gains negotiation endpoints for both sides.

### Tutee-facing interview endpoints

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | interviews interface operation | TUTEE | **Propose** interview with 1–3 timeslots (INT-039); **201** Interview Response. Body Propose Interview Request. **spec 021/REQ-001:** allowed when the tutee post is **PENDING** (awaiting approval) **or** ACTIVE — the tutor's CONTACT stays restricted to assigned participants with the retained match-state/user interface gates. Not-open post → 422 TUTEE POST NOT ACTIVE. Verify Interview Service (Post Status ACTIVE or Post Status PENDING) |
| GET | tutee interviews interface operation | TUTEE | Own interviews → a page of interview records, with pagination details |
| GET | tutee interviews id interface operation | TUTEE | Own interview detail with negotiation timeline. 404 if not owned |
| POST | tutee interviews id accept interface operation | TUTEE | Accept one open tutor slot. Body contains the selected time-slot identifier → Interview Response. Status → PENDING SCHEDULING |
| POST | tutee interviews id counter interface operation | TUTEE | Propose fresh 1–3 slots. Body contains the proposed interview time slots → Interview Response |
| POST | tutee interviews id cancel interface operation | TUTEE | Cancel own; **200** (no body); 422 if terminal |

### Tutor-facing interview endpoints

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | tutor interviews interface operation | TUTOR | Received interview requests → a page of interview records, with pagination details |
| GET | tutor interviews id interface operation | TUTOR | Detail with negotiation timeline |
| POST | tutor interviews id accept interface operation | TUTOR | Accept one open tutee slot. Body contains the selected time-slot identifier → Interview Response. Status → PENDING SCHEDULING |
| POST | tutor interviews id decline interface operation | TUTOR | Decline with reason (and optional counter slots). Body contains the reason for declining and any optional alternative time slots → Interview Response |
| POST | tutor interviews id counter interface operation | TUTOR | Propose fresh 1–3 slots. Body contains the proposed interview time slots → Interview Response |
| POST | tutor interviews id cancel interface operation | TUTOR | Cancel an in-flight negotiation; **200** (no body) |

### Admin interview endpoints

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | admin interviews interface operation | ADMIN | All interviews → a page of admin interview records, with pagination details; optional status filter (repeatable Interview Status) |
| GET | admin interviews id interface operation | ADMIN | Single → Admin Interview Response (with negotiation timeline). 404 |
| POST | admin interviews id schedule interface operation | ADMIN | Schedule from PENDING SCHEDULING (INT-039) or REQUESTED. Body contains the scheduled date and time and the optional meeting link. 422 if wrong status |
| POST | admin interviews id force-pending interface operation | ADMIN | **INT-039** — override stalled negotiation → PENDING SCHEDULING. Body contains the selected time-slot identifier if one has been agreed (optional: record agreed slot). **200** Admin Interview Response |
| PATCH | admin interviews id meeting-link interface operation | ADMIN | Set link on SCHEDULED. Body contains the meeting link. 422 if not SCHEDULED |
| POST | admin interviews id complete interface operation | ADMIN | SCHEDULED → COMPLETED. 422 otherwise |
| POST | admin interviews id cancel interface operation | ADMIN | Cancel any non-terminal interview. 422 if terminal |
| POST | admin interviews id no-show interface operation | ADMIN | SCHEDULED → NO SHOW. Body contains the participant who failed to attend: the parent or tutor. 422 otherwise |

### Request / response shapes (INT-039 additions)

**Propose Interview Request** (replaces the old Create Interview Request):

| Field | Required | Description |
|---|---|---|
| tutor Post Id | one-of | unique identifier — listing-based interview |
| tutor User Id | one-of | unique identifier — listing-less free-market interview |
| tutee Post Id | required | unique identifier |
| note | optional | ≤500 characters |
| slots | required | a list of offset date time entries (1–3 items). Each slot must be a future 15-min Coordinated Universal Time boundary within 7 days (service-validated) |

**Accept Timeslot Request**: timeslot Id (unique identifier, required) — the open counterparty slot being accepted.

**Counter Request**: slots (a list of offset date time entries, 1–3 items, required) — fresh round of proposed times.

**Decline Interview Request**: reason (Decline Reason, required: NOT INTERESTED or CANT MAKE IT); slots (a list of offset date time entries, 0–3 optional) — when CANT MAKE IT, slots counter in the same call; when NOT INTERESTED, slots are ignored and the interview reaches a terminal state.

**Force Pending Request**: timeslot Id (unique identifier, optional) — if provided, records that slot as agreed timeslot id before advancing to PENDING SCHEDULING.

**Interview Response** (updated — INT-039 adds negotiation fields):

| Field | Notes |
|---|---|
| id | unique identifier |
| tutor Post Id | unique identifier (nullable — listing-less) |
| tutee Post Id | unique identifier |
| tutor Post Title, tutor Post Subjects, tutor Post Levels, tutor Post Rate Min, tutor Post Rate Max | From tutor post snapshot; null/empty for listing-less |
| note, meeting Link | As before |
| status | Interview Status name (now includes negotiation values) |
| scheduled At, requested At | ISO-8601 strings |
| decline Reason | Decline Reason name or null |
| whose Turn | TUTEE, TUTOR, or null (null in list-view; populated in detail-view) |
| agreed Timeslot Id | unique identifier or null — the accepted slot |
| timeslots | a list of timeslot entries — empty in list-view, full in detail-view |

**Timeslot Response**: id, proposed By (TUTEE or TUTOR), round (int), start At (ISO-8601 string), end At (start At + 15 min, derived), slot Status (PROPOSED, WITHDRAWN, ACCEPTED, or REJECTED).

**Admin Interview Response** (updated — same new fields as Interview Response plus tutee Id, tutee Name, no Show Actor. **INT-053:** now also carries tutee Post Id (the job link — previously absent), tutor User Id, tutor Display Name, tutor Public Handle for the admin clickable tutor/job. **spec 019 (ADM-05):** now also carries **tutee Post Subject** + **tutee Post Level** — the job's subject/level codes — so the admin interview list can render the **Job column** for every interview, including **free-market** interviews that have no tutor-post. Verify Admin Interview Response implementation reference.).

### INT-039 expiry

A scheduled sweep (a scheduled task) marks interviews EXPIRED when no agreement is reached within the expiry period. An interview expired Micrometer counter is incremented per expiry. The expiry set is idempotent — running the sweep twice on the same row is safe.

### Cap index (migration 0034)

The M6 cap of 3 active interviews per parent posting is enforced by the partial index idx interviews tutee post active. Migration 0034 widens the when clause to include the new negotiation statuses:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## M7 — Profile — Profile Controller

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | tutee profile interface operation | TUTEE | Own tutee profile → Tutee Profile Response |
| GET | tutor profile interface operation | TUTOR | Own tutor profile → Tutor Profile Response |
| PUT | tutee profile interface operation | TUTEE | Update own tutee profile → Me Response |
| PUT | tutor profile interface operation | TUTOR | Update own tutor profile → Me Response |
| PATCH | tutor profile featured-consent interface operation | TUTOR | M14a — toggle public-landing featuring consent. Body contains the user’s choice to opt in or opt out → Tutor Profile Response |
| POST | auth change-password interface operation | Authenticated | Change password (rate-limited per IP); a success confirmation |

> Path note: profile endpoints are role profile interface operation (the prior doc listed profile tutor interface operation, which does not exist).

**Update Tutor Profile Request** (M7 base + M18 identity): display Name (≤100), qualifications (≤1000), years Experience (0–50), bio (≤2000), **Handle** (≤40), **commitment** (Tutor Commitment), **hourly Rate Min Sgd**, **hourly Rate Max Sgd**, **location Planning Area** (≤64), **location Region** (Location Region).
**Update Tutee Profile Request**: display Name (≤100).
**Tutor Profile Response** (own profile, self-personal information allowed): display Name, qualifications, years Experience, bio, featured Consent, **Handle**, **commitment**, **hourly Rate Min Sgd**, **hourly Rate Max Sgd**, location Planning Areas (a list of text values), **location Region**, subject Levels (INT-153 — a list of subject-and-level pairs capability matrix), **suggested Public Handle** (spec 002 / REQ-010 — a server-suggested handle derived from display name (the "Wei Tan" → "Wei T." algorithm), computed **only when Handle is null**; null/echo when a handle is already set; the frontend prefills the editable handle field with it), **profile Complete** (bool), **missing Fields** (a list of text values). The bold fields are M18/spec-002 additions.

> **Wave 1 — apply-gate single source of truth (profile Complete + missing Fields).** read operation for tutor profile now returns the gate verdict the frontend consumes to enable/disable Apply. profile Complete is true ⇔ missing Fields is empty. missing Fields is the ordered list of still-missing gate fields, as **stable string keys** (the frontend contract — used to render the "complete your profile" checklist): Handle, commitment, hourly Rate Min, hourly Rate Max, location Planning Areas, qualifications, subject Levels (the 7 gate fields, computed by Tutor Profile Completeness). This is the same verdict the apply endpoint enforces server-side (submit operation for tutor applications → **422 TUTOR PROFILE INCOMPLETE** when the profile is not complete), so the frontend pre-gate and the server gate never drift.
**Tutee Profile Response**: display Name.
**Change Password Request**: current Password, new Password (8–128).

---

## M8 — Match Dashboard — Match Dashboard Controller

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | tutee matches interface operation | TUTEE | Own matches → a page of tutee match records, with pagination details |
| GET | tutor matches interface operation | TUTOR | Own matches → a page of tutor match records, with pagination details |

admin Contact Email is populated only when match status equals ACTIVE.

**Tutee Match Response**: id, status, matched At, tutor Post Title, tutor Subject, tutor Display Name, monthly Fee Sgd, admin Contact Email, my Review Submitted.
**Tutor Match Response**: id, status, matched At, tutee Post Title (the real job title, **nullable**), tutee Subject (subject **code**), tutee Level (level **code**, INT-137a), monthly Fee Sgd, commission Amount Sgd, commission Status (Commission Status), admin Contact Email, my Review Submitted.  using INT-137a: subject/level are raw CODES the frontend humanises via the subject label operation/the level label operation the backend no longer composes a display title._

> **INT-146a — tutee Post Title nullability (consumer contract, regression note).** tutee Post Title
> is intentionally nullable and the backend provides **no** server-side fallback. Consumers MUST derive a
> display label from the Job Identity (subject/level codes humanised client-side) with a or otherwise fallback
> on the title — never render tutee Post Title raw. This is safe today only because both consumers
> already do so; do not remove that fallback. The tutor match-card shows subject·level via Job Identity.
>
> **INT-146c — tutee-vs-tutor match-card identity asymmetry is INTENTIONAL (confirmed, no change).**
> The tutor card surfaces the *job* (subject·level via Job Identity); the tutee card is *name-first*
> (tutor Display Name, no tutor Level). The two personas care about different identities (a tutor
> picks a job by topic; a tutee recognises a person by name), so the asymmetry is by design — not a
> bug. Revisit only if product explicitly wants symmetry.

> There is **no** read operation for matches id on the match dashboard. Per-match contact for an
> assigned match is the M18 read operation for matches id contact (below). The prior doc's
> the specified match accessible to the user row was stale and is removed.

---

## M9 — Notifications (notifications interface operation) — Notification Controller · Authenticated

| Method | Path | Purpose |
|---|---|---|
| GET | /notifications | Paginated own notifications → a page of notification records, with pagination details |
| GET | /notifications/unread-count | the number of unread notifications |
| POST | marking as read the specified notification as the user | Mark one read (404 if not owned — by design) |
| POST | /notifications/read-all | Mark all read |

**Notification Response**: id, type, title, body, link Path, read At, created At.

### INT-025 — POST SUBMITTED admin fan-out (migration 0036)

No new endpoint. When a tutee or tutor submits a post (creates and submits via submit operation for tutee posts or submit operation for tutor posts), both services call the emit to all admins operation **after the transaction commits** (Spring Transaction Synchronization Adapter after Commit). The fan-out is failure-isolated — a notification failure does not roll back the post.

- **Fan-out:** one notifications row per ADMIN-role user (the find by role operation)
- **Type:** POST SUBMITTED (added by migration 0036)
- **Title:** New post awaiting review
- **Link:** /admin/approval
- **Body:** subject + level from the submitted post (e g. a message identifying the subject and level of the newly submitted tuition request)

This resolves the gap noted in the sequences INDEX: Tutee Post Service create And Submit and Tutor Post Service create And Submit previously had no admin notification path.

---

## M10 — Favourites (tutee favourites interface operation) — Favourite Controller · role TUTEE

| Method | Path | Purpose |
|---|---|---|
| GET | /tutee/favourites | List own favourites (newest first) → a page of tutor post records, with pagination details |
| GET | /tutee/favourites/ids | Bulk membership probe; query post Ids (repeatable/CSV) → the matching record identifiers |
| POST | the specified favourite tutor accessible to the parent | Add (idempotent: **201** new, **200** already-favourited) |
| DELETE | the specified favourite tutor accessible to the parent | Remove (idempotent **204**) |

> Path note: favourites live under tutee favourites interface operation (the prior doc listed favourites tutor-posts * interface operation, which does not exist).

---

## M11 — Reviews — Review Controller

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | matches match Id reviews interface operation | Authenticated | Submit a review for a COMPLETED match (participant-validated in service); **201** Review Response. Body contains the review rating and accompanying review details |
| GET | tutee reviews interface operation | TUTEE | Reviews about me (received) → a page of review records, with pagination details |
| GET | tutor reviews interface operation | TUTOR | Reviews about me (received) → a page of review records, with pagination details |
| GET | tutors tutor Post Id reviews interface operation | Public | Reviews for a tutor post → a page of review records, with pagination details |

**Review Response**: id, match Id, rating (int), comment, reviewer Display Name, created At.
**Rating summary:** The tutor listing response includes the average rating and number of reviews. There is no separate rating-summary operation.

---

## M12 — Admin Analytics (admin analytics interface operation) — Analytics Controller · role ADMIN

| Method | Path | Purpose |
|---|---|---|
| GET | /admin/analytics | Platform metrics for from, to → Analytics Response. Query from/to (ISO date YYYY-MM-DD, optional; default to is today, from is to-30d). 400 if the start date is after the end date |

> The endpoint is read operation for admin analytics (no /overview suffix — the prior doc's /admin/analytics/overview was stale).

> **INT-045 (migration 0037):** added collected In Period Sgd to Revenue Summary and six timestamp indexes to support time-bounded analytics queries at scale. The from/to query params were already present; INT-045 makes them range-aware for the revenue field.

**Analytics Response** (fully expanded from Analytics Response implementation reference):


Implementation example omitted. This public edition retains the written requirements in this section.


One Daily Metric entry per day in the inclusive from, to range.

> **spec 017 REQ-001 — conversion Rate = completed-of-requested-in-period (cohort, ≤100% by construction).**
> The conversion tile is re-cohorted onto ONE population: of the interviews **requested within the selected
> from, to period**, the fraction that reached COMPLETED. Both operands come from the same cohort
> (requested-in-period), so the numerator is a strict subset of the denominator and the ratio is in **[0, 1]**
> by construction — no clamp. Returns **0.0** when the cohort is empty. _(The sibling requested/scheduled/
> completed/no Show/cancelled gauges remain all-time status totals — the frontend labels them as such — so only
> the ratio is period-cohorted.)_
>
> **spec 017 REQ-002 + spec 019 — avg Commission Per Match Sgd denominator = matches with a paid commission.** Divides
> the PAID-commission total by the **count of matches that HAVE a paid commission** (the same PAID population), NOT by
> the point-in-time active Matches Count. Commission is 1:1 with Match, so the PAID-commission row-count is exactly the
> number of matches with a collected commission. The dashboard tile labels this denominator **"Across matches with a
> paid commission."** active Matches Count stays in the payload as its own (separate-tile) gauge, unchanged. Verify
> Analytics Service build Revenue Summary (the count by invoice status operation). _(Note: the Revenue Summary record javadoc
> still reads / active Matches — stale; the running code divides by matches With Paid Commission.)_

---

## Analytics — Event Ingest (collect interface operation) — Collector Controller · Public (cross-site request forgery protection)

| Method | Path | Purpose |
|---|---|---|
| POST | /collect | Ingest a front-end measurement event. Body Collect Event Request the event name, optional event time and optional event details. |

- name must be on the **Analytics Event Registry allowlist** (e g. landing page viewed,
  assignment list viewed, posting submitted, application submitted, invite created,
  intuition match created). An **unknown** name → **422** (known:no); a **missing/blank** name →
  **400** (bean validation). A known event → 2xx.
- This is the sink the frontend the track operation calls hit; it was historically absent from this doc, postman, and
  the e2e inventory — added 2026-06-14 after deriving the surface from the controllers
  (e2e/coverage/controller-inventory mjs).

---

## M14 — Public Showcase (showcase interface operation) — Showcase Controller · Public

Unauthenticated, per-IP rate-limited, served with Cache-Control: public, max-age is 60. Reads hit Caffeine/Redis only (never the database). personal information-free by contract; fail-closed degraded envelope on empty/degraded/rate-limited.

| Method | Path | Purpose |
|---|---|---|
| GET | /public/showcase/tutors | M14a Active-Tutors strip → Showcase Envelope<Showcase Tutor Response> |
| GET | /public/showcase/requests | M14b open-requests strip → Showcase Envelope<Showcase Request Response> |

**Showcase Envelope<T>**: data (a list of matching records), meta (Showcase Meta).

**Showcase Tutor Response** (personal information-free — tutor Display Name is the pseudonymous handle only; no MRT): id, tutor Display Name, headline, subjects (Set), levels (Set), group Sizes (a set of distinct text values), venue Types (a set of distinct text values), hourly Rate Min Sgd, hourly Rate Max Sgd, location Planning Area, location Region, commitment (Tutor Commitment, nullable), average Rating, review Count, available (Boolean).

**Showcase Request Response** (personal information-free — also the /public/assignments projection): id, subject Code, level Code, num Students, budget Band Min Sgd (bucketed), budget Band Max Sgd (bucketed), session Duration Hours (Big Decimal), frequency Per Week, start Window (Start Window), location Planning Area (coarse, no MRT), location Region, posted Days Ago.

---

## M15 — Tutor Applications

> **NOTE — this module was missing from the prior doc.** It was deployed (migration 0018)
> before the prior revision but never documented. Captured here from the code.

### Tutor-facing (tutor applications interface operation) — Tutor Application Controller · role TUTOR

| Method | Path | Purpose |
|---|---|---|
| POST | /tutor/applications | Apply to a tutee job. **See M18 free-market change below.** Idempotent per (tutee Post Id, tutor Id); **200** Application Response (now carries quota, INT-077) in all success cases. **422 APPLICATION LIMIT REACHED** at the per-tutor cap |
| GET | /tutor/applications | Own applications (paginated) → a page of application records, with pagination details — each row's Application Response carries the live quota (INT-077) |
| DELETE | the specified application accessible to the tutor | Withdraw own (→ WITHDRAWN). **INT-077:** returns success confirmation and the updated application limit and remaining allowance (was just a success confirmation) — the freed slot is reflected immediately |

**Create Application Request**: tutee Post Id (unique identifier, required), tutor Post Id (unique identifier, **optional** — apply with no listing), message (≤500, optional).
**Application Response**: id, tutee Post Id, tutee Post Title, tutee Post Subject, tutee Post Level, tutor Post Id (nullable), message, status (Job Application Status name), has Interview (bool, M18 D2), interview Id (nullable, M18 D2), created At, updated At, **quota** (Quota Response, INT-077 — the tutor's per-account application quota *after* this operation).  using Never carries tutee/parent contact details._

**Quota Response** (INT-077): limit (int — the configured cap), remaining (int — slots left, **clamped ≥0**). Surfaced on every relevant apply/invite/withdraw response so the frontend can render an authoritative "N of limit left" chip without a separate fetch.

### Admin applicant view — Admin Application Controller · role ADMIN

| Method | Path | Purpose |
|---|---|---|
| GET | admin posts tutee Post Id applications interface operation | Applicants for a job (paginated, **admin sees tutor identity**) → a page of admin application records, with pagination details |

**Admin Application Response**: id, tutee Post Id, tutor Id, tutor Display Name, tutor Post Id (nullable), message, status, created At, updated At.

---

## Current financial and invitation acceptance boundaries

PAID always means agency receipt of the tutor’s commission. Parent lesson payment or an unpaid dispute outcome is not agency revenue. The first-session confirmation/invoice obligations in product requirements document remain; simplified endpoint/status descriptions do not waive them.

Average commission’s exact period definition is pending O02 in CURRENT-REQUIREMENTS md. Older all-time/cumulative code descriptions below are historical implementation observations, not authority for a selected-period metric. Invitation initiation on PENDING is approved; acceptance timing is pending O01 and the older ACTIVE-only acceptance guard remains until explicitly reconciled.

## INT-044 — Job Invitations (migration 0035)

A tutee can directly invite a specific tutor to their job while the post is **PENDING** (awaiting approval) **or** ACTIVE (spec 021/REQ-001) — the tutor's CONTACT stays restricted to assigned participants with the retained match-state/user interface gates. The tutor's accept folds into the existing apply + selection-request funnel — no Match is created by an invitation (AC-04 holds). Status is stored as the varchar operation (not a PG enum) to avoid ALTER TYPE friction.

**Lifecycle:** PENDING → ACCEPTED | DECLINED | WITHDRAWN | EXPIRED

### Tutee-facing — Tutee Invitation Controller · role TUTEE

| Method | Path | Purpose |
|---|---|---|
| GET | tutee invitations interface operation | **spec 013 REQ-003 (Wave 1).** The caller's OWN outbound (sent) invitations, paginated, newest first → **200** a page of job invitation records, with pagination details. Default: PENDING only (excludes withdrawn/soft-deleted); pass ?status is ALL for full history. ?page/?size (default size 20). Scoped by created by in the service, so a tutee only ever reads THEIR OWN sent invitations (own-only / IDOR). personal information-safe (tutor handle only — never a name/email/phone). Path constant TUTEE INVITATIONS tutee invitations interface operation |
| GET | tutee posts id invite-quota interface operation | **NEW (INT-077).** Per-job invite quota for a job you own → **200** the allowed limit and remaining allowance. Powers the pre-click invite chip. Owner-checked: **403** non-owner, **404** missing |
| POST | tutee posts id invite interface operation | Invite a tutor to own job. Body Invite Tutor Request. **201** on create; **200** on idempotent re-invite (existing PENDING returned). **spec 021/REQ-001:** allowed when the post is **PENDING** (awaiting approval) **or** ACTIVE — the tutor's CONTACT stays restricted to assigned participants with the retained match-state/user interface gates; a not-open post → 422 JOB NOT ACTIVE (verify Job Invitation Service). Response now carries quota (per-job invite quota, INT-077). **422 INVITE LIMIT REACHED** at the per-job cap. **INT-082 (default OFF):** phone-verification-gated — **422 PHONE NOT VERIFIED** when the gate is ACTIVE and the parent's phone is unverified |
| DELETE | tutee invitations id interface operation | Withdraw a PENDING invite (soft-delete; status → WITHDRAWN). Idempotent on terminal states (200, no-op). Wrong-owner → 404 (existence not leaked). **INT-077:** returns success confirmation and the updated invitation limit and remaining allowance (was just a success confirmation) |

Path constants from Api Routes: TUTEE INVITATIONS tutee invitations interface operation, the remaining invitation allowance for the specified post, inviting a tutor to the specified post as the parent, the specified invitation accessible to the parent.

The list read operation for tutee invitations reuses this same Job Invitation Response shape (the quota field is null on the list — it is only populated on the invite/withdraw mutation responses).

**Invite Tutor Request**: tutor Id (unique identifier, required), message (≤500, optional).

**Job Invitation Response** (personal information-safe — personal information contract-tested by Job Invitation Response Pii Contract Test):

| Field | Notes |
|---|---|
| id | unique identifier |
| tutee Post Id | unique identifier |
| job Subject, job Level | Anonymised from the job |
| tutor User Id | Opaque unique identifier — never a name, email, or phone |
| tutor Public Handle | Public handle only — never display Name or real name |
| message | Optional invite note |
| status | Job Invitation Status |
| job Application Id | unique identifier or null — populated once ACCEPTED |
| created At, responded At | Offset Date Time |
| quota | the allowed limit and remaining allowance (INT-077) — the **per-job** invite quota after this operation |

### Tutor-facing — Tutor Invitation Controller · role TUTOR

| Method | Path | Purpose |
|---|---|---|
| GET | tutor invitations interface operation | Received invitations, paginated. Default: PENDING only. Pass ?status is ALL for full history → a page of tutor invitation card records, with pagination details |
| POST | tutor invitations id accept interface operation | Accept → creates/reactivates application + auto-creates selection request on tutee's behalf. **200** Job Invitation Response (status is ACCEPTED, job Application Id populated) |
| POST | tutor invitations id decline interface operation | Decline (terminal; does not alter any application). Tutee notified after-commit. **200** Job Invitation Response (status is DECLINED) |

Path constants from Api Routes: TUTOR INVITATIONS tutor invitations interface operation, accepting the specified invitation as the tutor, declining the specified invitation as the tutor.

**Tutor Invitation Card Response** (personal information-safe — no parent/child name, no fine location, no contact info):

| Field | Notes |
|---|---|
| id | unique identifier |
| tutee Post Id | unique identifier |
| job Subject, job Level | Anonymised |
| coarse Area | Region only — NOT planning area, MRT, or address |
| budget Min Sgd, budget Max Sgd | Full SGD range (tutee's posted range, not bucketed) |
| message | Optional invite note |
| status | Job Invitation Status |
| already Applied | bool — hint: tutor already has an APPLIED application for this job |
| created At | Offset Date Time |

### Notification types added (migration 0035e)

- JOB INVITATION RECEIVED — sent to the tutor when invited
- JOB INVITATION RESPONDED — sent to the tutee when the tutor accepts or declines

---

## INT-077 — Marketplace quota caps (migration 0046)

Fairness limits on how many open applications a tutor may hold and how many open
invitations a tutee may extend per job, with the live quota surfaced to the frontend on every
relevant response so the user sees "N of limit left" *before* they click.

### Cap semantics

| Cap | Scope | Counted statuses | Config key (default) | Over-cap error |
|---|---|---|---|---|
| Tutor applications | **per tutor** (across all jobs) | OPEN STATUSES = APPLIED + SHORTLISTED | quota applications-per-tutor (**10**) | 422 APPLICATION LIMIT REACHED |
| Tutee invitations | **per tutee post** (job posting) | PENDING | quota invites-per-post (**10**) | 422 INVITE LIMIT REACHED |
| Interviews (pre-existing) | per tutee post | ACTIVE STATUSES | hard **3** | 422 INTERVIEW LIMIT REACHED |

- **Withdraw frees a slot** — application withdraw (→WITHDRAWN) and invite withdraw
  (→WITHDRAWN) both decrement the active count; the response carries the updated quota.
- **Accept bypasses the tutor cap** — the invitation-ACCEPT path
  (Job Application Service apply Internal via the accept operation) passes
  enforce Cap is no: an accept is the highest-intent funnel step, so it is *not* blocked by
  the tutor's application cap. It still serializes on the advisory lock (below).
- Limits are **config-driven** — changing a cap is a deploy, not a code change.

### Race-safety (advisory lock)

Each count → check → insert runs inside the existing a database transaction that saves all related changes together guarded by a
**transaction-scoped** pg advisory xact lock taken on a server-derived key **before** the
count, so two concurrent writes can't both read 9 and both insert the 11th
(TOCTOU). Keys (derived from the authenticated principal / path, never client input):

| Operation | Lock key |
|---|---|
| Apply | jobapp:<tutor Id> |
| Invite | jobinv:<tutee Post Id> |
| Interview request | interview:<tutee Post Id> |

The lock auto-releases at commit and serializes only the same parent. A partial-unique index
enforces "≤1", not "≤N", so the count is still required — the lock makes it atomic.

### Quota Response shape & where it appears

the allowed limit and remaining allowance — remaining is **clamped ≥0**. It rides on:
submit operation for tutor applications, each row of read operation for tutor applications,
delete operation for tutor applications id (as application Quota),
submit operation for tutee posts id invite, delete operation for tutee invitations id (as invite Quota),
read operation for tutee posts id invite-quota (the NEW pre-click endpoint), and the
Owner Applicants Response envelope on read operation for tutee posts id applicants (as invite Quota).

### Index (migration 0046-quota-cap-indexes implementation reference)

- idx jobapp tutor active — partial index ON job applications using tutor id when status IN
  ('APPLIED','SHORTLISTED') (covers the per-tutor application count).
- The per-job invite count **reuses** idx jobinv post status (migration 0035) — no new
  index added for invites.

---

## M18 — Assignment Lifecycle & Free Marketplace

New free-market surface: tutors apply with **no listing required**, parents browse personal information-free
applicant cards and choose a tutor, admins action open selection requests, and the assigned
counterparty's contact is revealed only post-assignment.

### Free-market apply change — submit operation for tutor applications

tutor Post Id is now **optional** (free-market apply). A tutor with an incomplete profile is
blocked with **422 TUTOR PROFILE INCOMPLETE** ("Complete your tutor profile before applying
to jobs."). Other behaviour (idempotent, **200**, returns Application Response) unchanged.
Job Application Status gained SUPERSEDED (an application superseded when the parent
re-chooses / the job is assigned elsewhere).

### Parent applicant list + choose-tutor — Tutee Application Controller · role TUTEE

Owner-scoped (server-side owner check → 403 IDOR on a job you don't own).

| Method | Path | Purpose |
|---|---|---|
| GET | tutee posts id applicants interface operation | **personal information-free** applicant cards for a job you own (paginated). **INT-077** now returns the Owner Applicants Response envelope the applicant page and the job’s invitation limit and remaining allowance (was the bare Page Response). 200 / 403 IDOR / 404 |
| POST | tutee posts id select interface operation | Choose (or re-choose) a tutor. **201** on create, **200** on re-choose (update). Never auto-matches. Body contains the tutor’s identifier → Selection Request Response. **INT-082 (default OFF):** phone-verification-gated — **422 PHONE NOT VERIFIED** when the gate is ACTIVE and the parent's phone is unverified |

**Applicant Card Response** — **personal information-FREE** (enforced by Applicant Card Response Pii Contract Test;
no name/email/phone/exact-location fields exist): id (unique identifier), tutor User Id (unique identifier — opaque,
personal information-safe; needed to request an interview with a listing-less applicant), tutor Public Handle
(the only identity surfaced), job Subject, job Level, hourly Rate Min Sgd, hourly Rate Max Sgd,
coarse Area, years Experience, commitment (Tutor Commitment), rating (Double),
review Count (long), application Status (Job Application Status), applied At, pitch,
also Teaches (a list of text values).

**Owner Applicants Response** (INT-077 envelope): applicants (a page of applicant card records, with pagination details), invite Quota (the allowed limit and remaining allowance — the per-job invite quota, so the applicants screen can show "N of limit invites left" alongside the list).

**Selection Request Response**: id, tutee Post Id, tutor Id, status
(Tutor Selection Request Status), created At.

### Post-assignment contact reveal — Match Contact Controller · Authenticated, participant-only

| Method | Path | Purpose |
|---|---|---|
| GET | matches id contact interface operation | Reveal the **counterparty's** contact for an assigned match. Bidirectional (tutee sees tutor, tutor sees tutee) → Contact Reveal Response. **403** if not a participant; **404** pre-assignment or after cancel. **INT-082 (default OFF):** when the phone-verification gate is ACTIVE, an unverified caller gets **422 PHONE NOT VERIFIED** — see [Account / phone verification (INT-082)](#int-082--phone-verification-and-whatsapp-notifications) |

> This is the **only** endpoint that crosses the counterparty-personal information line, and only post-assignment.

**Contact Reveal Response**: counterparty Name, counterparty Phone, counterparty Email.

### Admin selection-request queue — Admin Selection Request Controller · role ADMIN

| Method | Path | Purpose |
|---|---|---|
| GET | admin selection-requests interface operation | Open (CREATED) parent "choose-tutor" requests, newest first (paginated) → a page of admin selection records, with pagination details |

**Admin Selection Request Response** (personal information-free identity — tutor surfaced by handle + opaque id
only): selection Request Id, tutee Post Id, job Subject, job Level, tutor Public Handle,
tutor User Id (opaque unique identifier), job Application Id (**null** when the chosen tutor did not apply —
free-market direct selection), status (Tutor Selection Request Status), created At.

### Extended interview request — submit operation for interviews

Create Interview Request now: tutor Post Id (unique identifier, **optional**), tutor User Id (unique identifier,
**optional** — for a free-market applicant who applied with no listing), tutee Post Id (unique identifier,
required), note (≤500, optional). **At least one of tutor Post Id / tutor User Id must be
present** (enforced in the service). For a listing-less interview, Interview Response's
post-derived fields (tutor Post Id, tutor Post Title, subjects/levels/rates) are null/empty.

### Concurrency

Two admins racing to assign the same ACTIVE job hit the Tutee Post @Version optimistic lock
→ **409 JOB ALREADY ASSIGNED** (never a silent double-match).

---

## Local-only regression fixtures

> These beans **do not exist** in the prod profile; never exposed in production. Read-only / no real personal information.

| Method | Path | Controller | Purpose |
|---|---|---|---|
| GET | regression seed-manifest interface operation | Regression Manifest Controller | Seed version + presence + coarse counts → the sample-data version, whether it is loaded, a sample-account check and counts of tutors, tuition seekers and active tutor posts |
| POST | regression password-reset issue-otp interface operation | Regression Auth Controller | Issue a real reset one-time passcode for a regression-band (@regression local) email and return the plaintext → the one-time passcode. 400 REGRESSION EMAIL REQUIRED outside band; 404 no such user |

---

## Controller inventory (29 controllers)

> Wave 2026-06-03 adds Tutee Invitation Controller (INT-044) and Tutor Invitation Controller (INT-044), bringing the total to 26. Wave 2026-06-04 adds Admin Moderation Controller (INT-065), bringing the total to 27. Wave 2026-06-06 adds Account Controller + Whats App Webhook Controller (INT-082, default OFF), bringing the total to 29.

| # | Controller | Base / scope | Endpoints |
|---|---|---|---|
| 1 | Auth Controller | auth interface operation | 10 |
| 2 | Tutee Post Controller | tutee posts interface operation (TUTEE) | 5 |
| 3 | Tutor Post Controller | tutor posts interface operation (TUTOR) | 6 |
| 4 | Public Browse Controller | public interface operation | 3 |
| 5 | Marketplace Detail Controller | api v1 interface operation (auth) | 2 |
| 6 | Admin Post Controller | admin posts interface operation (ADMIN) | 4 |
| 7 | Admin Match Controller | admin matches interface operation (ADMIN) | 4 |
| 8 | Admin Commission Controller | admin commissions interface operation (ADMIN) | 1 |
| 9 | Interview Controller | tutee/tutor/admin interviews (INT-039 expanded) | 20 |
| 10 | Profile Controller | role profile interface operation + change-password | 6 |
| 11 | Match Dashboard Controller | tutee/tutor matches | 2 |
| 12 | Notification Controller | notifications interface operation | 4 |
| 13 | Favourite Controller | tutee favourites interface operation (TUTEE) | 4 |
| 14 | Review Controller | reviews | 4 |
| 15 | Analytics Controller | admin analytics interface operation (ADMIN) — INT-045 | 1 |
| 16 | Showcase Controller | showcase interface operation | 2 |
| 17 | Admin User Controller | admin admins interface operation (ADMIN) | 3 |
| 18 | Tutor Application Controller | tutor applications interface operation (TUTOR) — M15/M18 | 3 |
| 19 | Admin Application Controller | admin posts id applications interface operation (ADMIN) — M15 | 1 |
| 20 | Tutee Application Controller | tutee posts id applicants,select interface operation (TUTEE) — M18 | 2 |
| 21 | Match Contact Controller | matches id contact interface operation (auth) — M18 | 1 |
| 22 | Admin Selection Request Controller | admin selection-requests interface operation (ADMIN) — M18 + INT-034 | 2 |
| 23 | Tutee Invitation Controller | tutee invite + withdraw + invite-quota (TUTEE) — INT-044, INT-077 | 3 |
| 24 | Tutor Invitation Controller | tutor invitations (TUTOR) — INT-044 | 3 |
| 25 | Regression Manifest Controller | regression interface operation (local only) | 1 |
| 26 | Regression Auth Controller | regression interface operation (local only) | 1 |
| 27 | Admin Moderation Controller | admin users,posts interface operation (ADMIN) — INT-065 | 4 |
| 28 | Account Controller | account interface operation (auth) — INT-082 (default OFF) | 4 |
| 29 | Whats App Webhook Controller | webhooks whatsapp interface operation (public-but-signed) — INT-082 | 2 |

**Documented endpoints:** ~107 across all 29 controllers (Interview Controller grew from 12 to 20 in INT-039; INT-065 adds Admin Moderation Controller with 4 endpoints; INT-082 adds Account Controller (4) + Whats App Webhook Controller (2), both shipped OFF).

---

## M16 — Admin Management (admin admins interface operation) — Admin User Controller · role ADMIN

| Method | Path | Purpose |
|---|---|---|
| POST | /admin/admins | Any ADMIN creates a (non-super) admin; **201** the administrator’s account identifier, email and temporary password |
| GET | /admin/admins | List admins → a list of admin list item entries |
| DELETE | the specified administrator account accessible to an administrator | **SUPER-ADMIN only** (service-enforced flag); soft-delete + purge sessions; **204** |

**Admin Created Response**: id, email, temp Password.
**Admin List Item Response**: id, email, is Super Admin, created At, last Login At.

---

## INT-065 — Admin Moderation (block/unblock + delete posting) — Admin Moderation Controller · role ADMIN

> Shipped to dev **2026-06-04**. Lets an admin block/unblock a tutor or tutee (with immediate
> session kill via a Redis flag) and remove a posting as a moderation action — **distinct from
> the INT-062 close**, which is the parent's own lifecycle close. All four endpoints are
> ADMIN-only under admin interface operation. Soft-delete only; every action writes an audit record.

| Method | Path | Purpose |
|---|---|---|
| GET | admin users interface operation | Moderation user list (paginated, filterable) → a page of admin user list item records, with pagination details. Query: role (TUTOR/TUTEE), status (ACTIVE/BLOCKED), q (free-text search on name/email), page, size. role is ADMIN → **400 ROLE NOT MODERATABLE** (admins are managed via M16, not moderated here) |
| POST | admin users id block interface operation | Block a tutor/tutee. Body Moderation Action Request. **200** a success confirmation |
| POST | admin users id unblock interface operation | Unblock a tutor/tutee. **No body.** **200** |
| POST | admin posts id delete interface operation | Admin moderation removal of a posting. Body Moderation Action Request. **200** |

**Admin User List Item Response**: id (unique identifier), full Name (String), email (String), role
(TUTOR/TUTEE), blocked (boolean), block Reason (String|null — Moderation Reason name),
block Note (String|null), blocked At (Instant|null).

**Moderation Action Request**: reason (Moderation Reason, required), note (String,
optional — **required when reason is OTHER**, else **400**).

**Moderation Reason** enum: POLICY VIOLATION, SPAM, ABUSE HARASSMENT, FRAUD,
FAKE PROFILE, OTHER. Stored as the varchar operation (a string column, **not** a PG enum — see
the INT-062 ALTER TYPE lesson), so adding a reason needs no enum migration.

### submit operation for admin users id block — guards & side effects

- **Guards:** target must be a TUTOR/TUTEE — blocking an ADMIN/super-admin → **403
  CANNOT BLOCK ADMIN**; **409** if the user is already blocked; note required when
  reason is OTHER.
- **Immediate enforcement:** writes a Redis kill-flag blocked:user:<id> with **expiry period ≥ token
  lifetime**, so the user's existing signed session token is rejected on the **very next request** (**401**, the
  auth cookie is expired by the filter) — not at token expiry.
- **Content cascade (live content closed):** tutee jobs cascade CLOSED via the INT-062 close
  path; tutor listings, applications and invitations are closed; any **ACTIVE/PENDING match is
  cancelled through the existing cancel path** (PENDING commission → CANCELLED; **PAID
  commission untouched**) so commission is never orphaned.
- **Audit:** USER BLOCKED.

### submit operation for admin users id unblock — side effects

- Clears the Redis flag **and** users blocked at; login is restored.
- **Does NOT auto-reopen** any content that was closed on block (operator must reopen manually).
- **Audit:** USER UNBLOCKED.

### submit operation for admin posts id delete — moderation removal (≠ INT-062 close)

- Soft-deletes the posting (tutee posts deleted at), drops it from all listings, and cascades
  open applications/invitations → CLOSED.
- **404** if the posting does not exist; **409** if it is already deleted.
- **Audit:** POSTING DELETED.
- This is an **admin moderation removal**, semantically distinct from the parent-initiated
  INT-062 close.

### Login behaviour — blocked account

A blocked user calling **submit operation for auth login** receives **403** with
error code is ACCOUNT BLOCKED and a message naming the block reason. This database-backed check
(users blocked at differs from no value, evaluated **before** the password check) is the durable backstop;
the Redis kill-flag is the immediate session terminator. Together: Redis = instant session
kill, database = permanent gate (defence in depth).

### Migrations

- **0040** — users: adds blocked at (timestamp), block reason (**VARCHAR using 50**),
  block note (text), blocked by admin id (FK → users).
- **0041** — tutee posts: adds deleted at (timestamp) for moderation soft-delete.

Both are plain add Column changesets — **no ALTER TYPE … ADD VALUE** (which crash-loops
migrate-then-boot, per the INT-062 lesson).

---

## INT-082 — Phone verification and WhatsApp notifications

> **Status: BUILT & shipped OFF to dev (2026-06-06).** The whole feature is **available-but-disabled** behind the
> whatsapp enabled toggle (default no). While OFF it is **inert**: the one-time passcode endpoints return an indication of whether phone verification is enabled (disabled in this case)
> and send nothing, the action gate is a **no-op** (no 422s), and no WhatsApp messages are dispatched. Flipping it ON in
> any environment is governed by the pre-enablement checklist (PE-0…PE-4). Full design + rationale:
> [int-082-whatsapp-otp-and-notifications-spec implementation reference](../architecture/int-082-whatsapp-otp-and-notifications-spec.md).
> Migration **0047** (additive: users phone verified/phone verified at/whatsapp opt in/whatsapp opt in at +
> phone verification tokens table + one-live-token partial unique index).

### Feature toggles (all default no)

| Property | Default | Effect |
|---|---|---|
| whatsapp enabled | no | Master switch. All sub-features inert while false |
| whatsapp otp.enabled | no | one-time passcode send/verify sub-toggle |
| whatsapp notifications.enabled | no | Curated WhatsApp notification fan-out sub-toggle |

The canonical predicates (one injected Whats App Toggle component) are:


Implementation example omitted. This public edition retains the written requirements in this section.


**Central invariant:** the gate bites **only** when master ON **and** the relevant sub-toggle ON **and** the Meta Cloud
client is healthy (access-token + phone-number-id present). OFF — or **ON-but-unconfigured** (the "brick-on-ON" case)
— always no-ops, so a misconfiguration can never brick the marketplace. With whatsapp enabled is yes but secrets absent,
the app **fails its readiness probe** rather than serving the brick state. Secrets are env-only
(WHATSAPP CLOUD ACCESS TOKEN, WHATSAPP CLOUD PHONE NUMBER ID, WHATSAPP CLOUD APP SECRET,
WHATSAPP WEBHOOK VERIFY TOKEN); never committed.

### Account endpoints — Account Controller · Authenticated

All four operate on the **signed session token user's own** account (no user id in path/body). Standard auth: auth cookie + cross-site request forgery protection
XSRF-TOKEN on the mutating calls.

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | account phone verify start interface operation | Authenticated | Mint + send a 6-digit WhatsApp one-time passcode to the user's **own stored phone**. → **200** an indication of whether phone verification is enabled. **OFF / brick-on-ON ⇒ an indication of whether phone verification is enabled (disabled in this case), nothing sent.** When ON: bcrypt-stores exactly one live token, enqueues the Meta auth template; rate-limited per-user Id, ip + per-user daily cap + global ceiling → **429 RATE LIMIT EXCEEDED**. **422 NO PHONE ON ACCOUNT** if no phone stored. Uniform (enumeration-safe) response |
| POST | account phone verify confirm interface operation | Authenticated | Confirm the one-time passcode. Body contains the one-time passcode (6 digits) → **200** Phone Status Response. On success sets phone verified is yes, phone verified at is now, single-uses the token. **422 INVALID one-time passcode** (wrong/expired) — increments attempt count; **422 one-time passcode ATTEMPTS EXCEEDED** after 5 failed attempts. Inert (an indication of whether phone verification is enabled (disabled in this case) semantics) when OFF |
| PUT | account phone interface operation | Legacy | Historical INT-082 direct-change route, superseded by 012. It must not mutate contact without step-up and email-link confirmation. Apply normalization, uniqueness and phone-verification reset at the confirmed commit. |
| PUT | account whatsapp-consent interface operation | Authenticated | Per-user WhatsApp notification opt-in (Personal Data Protection Act). Body contains the user’s choice to opt in or opt out → **200** Phone Status Response. Setting opt In is yes stamps whatsapp opt in at; opt-out clears it. The flip is **audit-logged** (event, not just the column) so consent-at-time-of-send is reconstructable |

**Requests**
- Phone Verify Confirm Request: otp (6 digits).
- Update Phone Request: phone (SG mobile; normalized to E.164 server-side).
- Whats App Consent Request: opt In (bool).

**Phone Verify Start Response**: enabled (bool) — no when the feature is OFF/unconfigured (nothing was sent);
yes when an one-time passcode was minted and enqueued.

**Phone Status Response** (returned by confirm / phone-change / consent): phone (E.164, the stored number),
phone Verified (bool), whatsapp Opt In (bool), gate Active (bool — whether the verification gate is currently biting,
so the frontend knows if unverified actions will 422).

### spec 012 REQ-004 — Self-service contact change and revocation

The [012 design](../012-tutor-apply-and-profile-ux/design.md) controls. Email changes send confirmation to the **new email**; phone changes send it to the **already-verified account email**. Keep the current contact unchanged until an explicit confirmed POST commits. Require current-password step-up at initiation; warn **both old email and old phone** with revocation access. The old-phone transport while WhatsApp is OFF remains open decision O04; a recorded outbox intent is not evidence of delivery.

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | account contact-change interface operation | Authenticated owner + cross-site request forgery protection | Initiate the contact field to change, its proposed new value and the current password for identity confirmation; validate field/value, enforce step-up (422 STEP UP REQUIRED), supersede the owner's previous PENDING request for that field, and mint independent hashed confirmation/revoke credentials with a 30-minute expiry. Record warning intents and preserve the live value. |
| GET | account contact-change interface operation | Authenticated owner | Return only the owner's pending set, with masked new values and expiry. |
| POST | account contact-change verify interface operation | Authenticated owner + cross-site request forgery protection | Explicit confirm with the request identifier and its confirmation credential; verify the credential and lock the request row. PENDING → CONSUMED atomically swaps the field after uniqueness is rechecked at commit. Invalid/expired/consumed/revoked confirmation → 400 CONTACT CHANGE TOKEN INVALID; collision → 409 EMAIL ALREADY REGISTERED / PHONE ALREADY REGISTERED. |
| POST | account contact-change revoke interface operation | Authenticated owner + cross-site request forgery protection OR valid dedicated revoke credential | Owner body the request identifier; warning-link body the request identifier and its revocation credential. Authorize only cancellation of that request. Lock the same row; PENDING → REVOKED invalidates confirmation, leaves current contact unchanged and audits the action. Successful and valid repeat revoke return 204 with no body; CONSUMED → 409 and recovery guidance, without rolling back the completed change. |

Confirmation and revocation links render landing pages; GET never changes state or consumes credentials. Revoke credentials are purpose-bound to the request, owner and field and cannot confirm a change or reveal contact. Tokens must not enter logs, referrers or analytics. Serialize confirm/revoke on the same row so exactly one transition wins.

Contact Change Response pending contains request Id, field, masked new value and expiry, at most one request per field. Token-only revocation must not return account contact or the owner's other pending requests. The historical 0066 table requires a new additive Liquibase changeset for REVOKED/revoked at, independent revoke credential hash and warning snapshots; do not rewrite an applied migration. Keep user contact storage consistent with the plaintext amendment; passwords and token credentials remain hashed.

### Webhook — Whats App Webhook Controller · public-but-signed

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | webhooks whatsapp interface operation | Public (verify-token) | Meta subscription handshake. Echoes hub challenge (**200** plaintext) when hub mode is subscribe and hub verify token matches whatsapp cloud.webhook-verify-token (constant-time compare); **403** otherwise |
| POST | webhooks whatsapp interface operation | Public (keyed message authentication-signed) | Meta delivery-status callback. Verifies the X-Hub-Signature-256 keyed message authentication against whatsapp cloud.app-secret using a **constant-time** compare (Message Digest is Equal); **rejects unsigned / invalid signatures** (**403**). Handler is **idempotent** (Meta retries) → **200** |

> The webhook path is allow-listed in Security Config (public) but is **authenticated by signature**, not by the auth
> cookie. It is the only INT-082 endpoint that is not signed session token-protected.

### Action gate (effective only when the gate active operation)

A single the assert phone verified operation → **422 PHONE NOT VERIFIED**, applied to the founder-chosen gated set. It is a
**no-op when the gate active operation** (master/sub OFF or client unhealthy), so OFF means **no 422s anywhere**.

| Action | Endpoint | Gated? |
|---|---|---|
| Contact reveal | read operation for matches id contact | **MUST gate** — the personal information-exchange moment |
| Select tutor | submit operation for tutee posts id select | SHOULD gate — parent commitment |
| Invite tutor | submit operation for tutee posts id invite | SHOULD gate — parent commitment |
| Tutor apply | submit operation for tutor applications | **NOT gated** — top-of-funnel supply side stays open |

frontend 422 disambiguation: gated actions switch on error code — PHONE NOT VERIFIED routes to the inline verify flow,
while TUTOR PROFILE INCOMPLETE (apply) routes to profile completion.

### Curated WhatsApp notifications (effective only when the notifications active operation)

**One delivered event by default — INTERVIEW SCHEDULED, to BOTH tutor and tutee** (INT-084/INT-085, founder cost
decision 2026-06-07; MATCH CREATED and contact-revealed are **dormant** — enqueued but dropped at dispatch). The set
is config-driven: whatsapp notifications.events (default INTERVIEW SCHEDULED); re-expansion is config-only but
requires founder cost approval + a lockstep frontend consent-copy update. Fan-out goes to whatsapp opt in is yes +
phone-verified recipients only, alongside the in-app Notification Service emit (both parties), via the transactional
**outbox** (idempotent, source Event Id-deduped). Dropped dispatches increment
a count of dropped WhatsApp notifications grouped by reason and event. Pre-approved Meta templates only (free-form text is disallowed outside
the 24h session window). one-time passcode is transactional (no marketing consent needed); notifications are strictly opt-in.

### Pre-enablement gates (block flipping the toggle ON — see design spec §9.1)

- **PE-0 (ops):** Meta Business verification + one-time passcode/notification template pre-approval.
- **PE-1 (SE + dba):** reconcile registration phone normalization with the E.164 normalization used by registration and the 012 confirmed contact-change commit.
- **PE-2 (SE + security):** build the global send-ceiling auto-trip, or formally accept rate-limit-only.
- **PE-3 (devops + observability): ⛔ STILL OPEN — blocks enablement.** Wire the signature-invalid-spike / send-failure-spike / token pre-expiry alerts **+ an alert/dashboard on whatsapp notification.dropped and whatsapp spend (NOT yet emitted in code — building it is part of this gate), and the kill-switch runbook entry (Boss condition 2026-06-08; see ops/runbook md §"WhatsApp kill-switch").**
- **PE-4:** ✅ DONE 2026-06-08 — backend continuous integration green (the recorded revision): surefire + Testcontainers failsafe band incl. the toggle truth-table (live-stack lab runs post-deploy via the ci-no-lab profile).
- **PE-5:** ✅ DONE 2026-06-07/08 — code aligned to the narrowed event set (INT-084 config filter + consent copy; INT-085 both-parties), team-reviewed + Boss SHIP, deployed dev.

---

## Module Completion Status

| Module | Description | Status |
|---|---|---|
| M1 | Auth — registration, login, multi-factor authentication (time-based one-time passcode), password reset, account deletion | Complete |
| M2 | Posting — tutee requests, tutor listings, public browse | Complete |
| M3 | Approval — admin review queue, approve/reject | Complete |
| M4 | Matching — admin create/activate/cancel | Complete |
| M5 | Commission — payment + match completion | Complete |
| M6 | Interview — request, scheduling, no-show | Complete + deployed |
| M7 | Profile — tutor/tutee profile management (M18 tutor identity fields) | Complete |
| M8 | Match Dashboard — tutee/tutor views with ACTIVE contact reveal | Complete |
| M9 | Notification Centre | Complete |
| M10 | Search & Favourites | Complete |
| M11 | Review & Rating | Complete |
| M12 | Admin Analytics | Complete (verification pending) |
| M14a/b | Public showcase strips (tutors + requests), personal information-free | Deployed |
| M15 | Applications — tutor→job apply/withdraw; admin applicant view | Deployed (migration 0018) |
| M16 | Admin Management & Super-Admin | Deployed (migration 0020) |
| M17 | Marketplace user experience overhaul — public assignments, login-gated detail | Deployed (no migration) |
| M18 | Assignment Lifecycle & Free Marketplace — free-market apply, applicant cards, choose-tutor, selection-request queue, contact reveal, extended interview | Deployed |
| INT-034 | Free-market assign re-keyed on matches tutor user id; submit operation for admin matches accepts winning Tutor Id; new submit operation for admin selection-requests id assign | Deployed (migration 0033) |
| INT-039 | Interview timeslot negotiation — propose/accept/counter/decline/cancel for tutee + tutor; admin force-pending; expiry sweep + Micrometer counter | Deployed (migration 0034) |
| INT-044 | Tutee→tutor job invitations (invite/withdraw/list/accept/decline); accept folds into apply + selection-request funnel; 2 notification types | Deployed (migration 0035) |
| INT-045 | Analytics collected In Period Sgd + 6 timestamp indexes | Deployed (migration 0037) |
| INT-025 | POST SUBMITTED admin fan-out after-commit (no new endpoint; new notification type) | Deployed (migration 0036) |
| INT-065 | Admin moderation — block/unblock tutor/tutee (immediate Redis kill-flag) + delete posting + read operation for admin users; Admin Moderation Controller (4 endpoints); login ACCOUNT BLOCKED | Deployed (migrations 0040–0041) |
| INT-077 | Marketplace quota caps — per-tutor application cap + per-job invite cap (config-driven, advisory-lock race-safe); Quota Response on apply/invite/withdraw; new read operation for tutee posts id invite-quota; Owner Applicants Response envelope; 422 APPLICATION LIMIT REACHED/INVITE LIMIT REACHED | Deployed (migration 0046) |
| INT-078 | Removed account-level tutee relationship (student/parent) field end-to-end incl. tutee profiles relationship column + tutee relationship PG enum. Post-level student Relationship later removed by INT-151 (migration 0053) | Deployed (migration 0045) |
| INT-082 | WhatsApp phone-verification (one-time passcode) + curated WhatsApp notifications — Account Controller (start/confirm/phone-change/consent) + signed Meta webhook; phone-verification gate (422 PHONE NOT VERIFIED) on reveal/select/invite. **Default OFF** (whatsapp enabled); inert until pre-enablement gates clear | Deployed OFF (migration 0047) |

---

## Corrections vs the prior (2026-05-30) doc

1. **Profile paths** — were documented as profile tutor interface operation etc.; code uses tutor profile interface operation, tutee profile interface operation (+ PATCH .../tutor/profile/featured-consent and submit operation for auth change-password).
2. **Favourites paths** — were favourites tutor-posts * interface operation; code uses tutee favourites interface operation, the specified favourite tutor accessible to the parent, and /tutee/favourites/ids.
3. **Match dashboard read operation for matches id** — does not exist; removed. Per-match contact is the M18 read operation for matches id contact.
4. **Analytics path** — was /admin/analytics/overview; code is read operation for admin analytics with from/to query params.
5. **Reviews** — the public route is read operation for tutors tutor Post Id reviews; review submission is submit operation for matches match Id reviews. There is **no** standalone the rating summary for the specified tutor endpoint (aggregate rating rides on Tutor Post Response).
6. **M15 Applications** — entirely absent from the prior doc; now documented.
7. **M16 Admin Management** — present in code but absent from the prior doc; now documented.
8. **M18 additions** — free-market apply (tutor Post Id optional + TUTOR PROFILE INCOMPLETE), Applicant Card Response, choose-tutor, /admin/selection-requests, authorized contact details for the specified match, extended Create Interview Request, and the new Tutor Profile Response/Showcase Tutor Response/Tutor Post Response identity fields — all added.

### OPEN questions (flagged, not guessed)

- **Commission Status enum values** were not read directly (referenced by Admin Commission Item Response/Tutor Match Response); enumerate from Commission Status implementation reference if the exact set is needed.
- **Analytics Response nested record fields** (Interview Funnel, Match Funnel, Revenue Summary, Daily Metric) are summarised, not fully expanded — read Analytics Response implementation reference for the full field list.
- **Seed Manifest seed Version** is hard-coded dev-1 and **sentinel Email** REDACTED email in Regression Manifest Controller; these are local-fixture constants, not a stable public contract.
</content>
</invoke>
