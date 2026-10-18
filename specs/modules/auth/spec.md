# InTuition — Auth Module Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** [001](../../001-legal-and-compliance-consent-and-policy-pages/spec.md), [002](../../002-tutor-profile-and-onboarding-ux/spec.md), [004](../../004-auth-session-hardening-30min-ttl-and-refresh/spec.md), [012](../../012-tutor-apply-and-profile-ux/spec.md), [plaintext contact storage](../../architecture/plaintext-email-phone.md), M16 and [027](../../027-aws-production-deployment/spec.md) supersede the affected older rules. OQ-1 email-verification onboarding remains pending; a contact-change prerequisite is not an onboarding decision. Local no-multi-factor authentication exception is limited to development; production requires multi-factor authentication. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Module:** auth  
**Version:** 1.2.0
**Status:** Draft  
**Last Updated:** 2026-09-05
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x  
**Owner:** Tech Lead

---

## 1. Overview

The Auth module manages identity, access control, and session lifecycle for the Singapore tuition agency marketplace. It serves three distinct roles — TUTEE, TUTOR, and ADMIN — each with different registration paths, session rules, and trust levels. Apply the retained privacy/provider acceptance in the product requirements document/business requirements document and027; this module document does not certify compliance merely from a region choice.

### 1.1 Scope

- User registration (TUTEE and TUTOR; ADMIN has no public signup; administrator-created accounts and protected bootstrap follow M16/027)
- Email + password authentication with signed session token-in-http Only-cookie session management
- time-based one-time passcode multi-factor authentication for ADMIN accounts (mandatory, enforced at every login)
- Password reset via 6-digit email one-time passcode
- Account lockout after repeated failed login attempts
- Session expiry: 30-minute standard-user access token with rotating refresh under 004; ADMIN retains Redis idle enforcement and receives no refresh token
- Account deletion: Personal Data Protection Act right of erasure (soft delete → 30-day anonymisation pipeline)
- Commission disclosure: mandatory standalone step in TUTOR registration wizard

### 1.2 Out of Scope

- OAuth / social login (future phase)
- SMS one-time passcode for non-admin users (future phase)
- TUTOR profile enrichment beyond initial registration (see matching module)

---

## 2. Roles and Registration Paths

| Role  | Registration | multi-factor authentication | Session Idle Timeout | Notes |
|-------|-------------|-----|----------------------|-------|
| TUTEE | Public /register/tutee | None | 30-minute access + refresh (004) | Standard learner account |
| TUTOR | Public /register/tutor (wizard) | None | 30-minute access + refresh (004) | Must complete commission disclosure |
| ADMIN | M16 admin-created / protected027 bootstrap; no public signup | time-based one-time passcode (mandatory) | 30 minutes | No public registration path |

---

## 3. Functional Requirements

### 3.1 TUTEE Registration

1. User navigates to /register and selects "I am a Tutee".
2. User submits: full name, email, phone (optional), password.
3. System validates inputs (see Zod schema in frontend md), checks normalized email uniqueness under the plaintext-contact amendment.
4. System hashes password (bcrypt, cost factor 12), stores normalized plaintext email/phone under the founder amendment; no user email/phone encryption/keyed message authentication lookup columns.
5. System creates users record (role = TUTEE) and tutee profiles stub record in a single transaction.
6. System issues signed session token, sets http Only cookie, redirects to /dashboard.
7. Required email events use durable email intent and the configured Resend transport; the event catalog and delivery requirements remain authoritative. Enqueue is not proof of delivery.

### 3.2 TUTOR Registration (Multi-Step Wizard)

The TUTOR registration wizard has three steps. Non-secret wizard state may be held in React state or an approved persistence mechanism; passwords/one-time passcode/credentials must never be stored in browser storage. No partial account is created on abandonment. Optional profile fields do not block signup under 002.

**Step 1 — Account Info**
- Fields: full name, email, phone (required for tutors), password, confirm password.
- Validation mirrors tutee registration; phone is mandatory for tutors.

**Step 2 — Commission Disclosure (BR-009)**
- A standalone page displaying the agency's commission structure in plain language (percentage, when charged, what triggers it).
- User must tick an explicit acknowledgement checkbox: "I understand and agree that the agency’s registered legal name charges a commission fee as described above." (legal name from COMPANY legal Name, company implementation reference.)
- This step cannot be skipped. The "Continue" button is disabled until the checkbox is ticked.
- The server records the accepted version, timestamp and trusted request context atomically; client timestamps/IP are not trusted audit authority. Required legal consents follow001.

**Step 3 — Tutor Profile (Initial)**
- Fields: subject using s taught, MOE level using s taught (Primary, Secondary, JC/IP, IB, Poly, Adult), availability (free-text or structured slots), preferred regions (Singapore planning areas).
- Profile enrichment at signup is optional under 002. Subject/level pairs may be captured optionally and, when supplied, use020/025 validation; profile completeness is enforced at Apply under 012/024 rather than making signup enrichment mandatory.

On final submission:
1. All three steps' data are submitted together in one atomic application programming interface call.
2. System creates users record (role = TUTOR), tutor profiles record, and commission acknowledgements record inside a single database transaction.
3. signed session token issued, http Only cookie set, redirect to /dashboard/tutor.

### 3.3 Login

1. User submits email + password at /login.
2. System resolves the normalized plaintext email under the founder storage amendment.
3. System checks locked until: if in the future, reject with 423 and remaining lockout duration.
4. System verifies bcrypt hash. On failure: increment failed login attempts; if count reaches 5, set the account lock to expire 15 minutes after the current time, respond 423.
5. On success: reset failed login attempts to 0, update last login at, issue signed session token, set http Only cookie.
6. If role = ADMIN: do NOT issue full signed session token. Issue a short-lived (5-minute) partial token (mfa pending: yes) and redirect to /login/admin/mfa.
7. Role-based redirect:
   - TUTEE → /dashboard
   - TUTOR → /dashboard/tutor
   - ADMIN → /admin (only after multi-factor authentication success)

### 3.4 Admin Login (multi-factor authentication)

1. After password validation, admin receives mfa pending partial cookie.
2. Admin navigates to /login/admin/mfa (protected: requires mfa pending cookie).
3. Admin enters 6-digit time-based one-time passcode code from authenticator app.
4. System validates time-based one-time passcode against mfa secret (keyed message authentication-SHA1, 30-second window, ±1 step tolerance).
5. On success: invalidate mfa pending cookie, issue full signed session token with role: ADMIN and session Id, store session in Redis with 30-minute expiry period.
6. Every authenticated admin request resets the Redis session expiry period (sliding 30-minute idle window).
7. On multi-factor authentication failure: 3 consecutive failures lock the admin account for 15 minutes (same lockout mechanism as login).
8. Admin accounts without mfa enabled yes cannot log in (enforced in service layer and database constraint via trigger).

### 3.5 Password Reset

1. User submits email at /forgot-password.
2. System always responds 200 (no email enumeration). If the normalized account email exists, system:
   a. Checks rate limit: max 3 requests per email per hour (Redis counter).
   b. Generates a 6-digit numeric one-time passcode.
   c. Hashes one-time passcode with bcrypt (cost 10) and stores in the current reset-token record (account identity, one-time passcode hash,15-minute expiry and unused state; use the current Liquibase schema).
   d. Persists email intent in the business transaction; the outbox dispatches through the configured Resend client with retry/failed-message queue. Production one-time passcode logging is OFF under 027.
3. User navigates to /reset-password, enters email, one-time passcode, new password, confirm new password.
4. System validates one-time passcode: lookup by the normalized account identity, verify bcrypt hash, check expires at > NOW operation, check used at IS NULL.
5. On success: update password hash, record the reset token’s used-at time as the current time, reset failed login attempts, clear locked until.
6. System invalidates all existing sessions for the user (delete Redis keys by pattern server-managed sessions belonging to the specified user).
7. Redirect to /login with success toast.

### 3.6 Account Lockout

- Trigger: 5 consecutive failed login attempts (wrong password) on the same account.
- Effect: lock the account until 15 minutes after the triggering failure.
- Behaviour during lockout: all login attempts immediately return 423 with the time when another attempt is permitted.
- Auto-unlock: locked until is checked at login time; once elapsed, login proceeds normally and failed login attempts resets on the next successful login.
- Admin lockout also triggered by 3 consecutive multi-factor authentication failures (same 15-minute window).

### 3.7 Session Management

**Standard Users (TUTEE, TUTOR):**
- auth is a HttpOnly, Secure, Same Site is Strict access cookie with expiry period/Max-Age1800 seconds.
- Spec004 owns the host-only, refresh-path-scoped rotating refresh cookie, reuse/grace behavior and revocation. Do not replace its server-side refresh-family state with a stateless-only claim.
- Logout is server-driven: revoke the refresh family and expire both relevant cookies using their exact paths; JavaScript cannot clear a HttpOnly auth cookie. Account deletion also revokes the family.

**Admin Users:**
- signed session token expiry period: 1 hour (shorter than standard).
- Redis session key: server-managed sessions belonging to the specified user → stores the user identifier, role and session creation time with expiry period = 1800 (30 minutes).
- Every authenticated admin application programming interface request calls the expire operation (sliding window).
- If Redis key is missing/expired, admin is treated as unauthenticated regardless of signed session token validity; respond 401.
- Logout: clear cookie + delete Redis key.

### 3.8 Account Deletion (Personal Data Protection Act Right of Erasure)

**Phase 1 — Soft Delete (immediate):**
- User submits deletion request at /account/delete (authenticated).
- System records the current time as the deletion timestamp on the users record.
- Server clears the auth/refresh cookies and revokes the refresh family (004), with existing admin-session purge as applicable.
- User can no longer log in.
- Active postings/bookings are cancelled (handled by respective modules).

**Phase 2 — Anonymisation (scheduled, T+30 days):**
- A nightly cron job selects users where a deletion timestamp was present and was more than 30 days old in the historical proposal. The current retention and erasure schedule remains unresolved under O08.
- Anonymisation: anonymize the current plaintext email/phone and profile identity fields with uniqueness-safe placeholders/nulls as permitted, and clear authentication/multi-factor authentication secrets under the retained erasure policy. Do not write removed encrypted/keyed message authentication columns. Scheduling follows007.
- Name fields (in profile tables) are replaced with "Deleted User".
- The users record is retained for referential integrity (invoices, commission records).
- A data erasure log record is created with timestamp, user id, and performing job version.

### 3.9 Commission Disclosure

- Governed by BR-009. The disclosure page is a mandatory wizard step, not a checkbox buried in Terms & Conditions.
- The displayed text is sourced from a CMS/config (not hardcoded) so the agency can update commission rates without a code deploy.
- The commission acknowledgements record must be created atomically with tutor registration. If the database transaction fails, the entire registration is rolled back.
- The stored record includes: tutor id, commission text version (CMS version key), acknowledged at, ip address, user agent.

---

## 4. State Machine

### 4.1 User Account States


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.2 Admin Session States


The illustration for admin session states contains the following information:

- NOT AUTHENTICATED.
- [POST /login - password OK].
- multi-factor authentication PENDING (5-min partial cookie).
- [POST /login/mfa - time-based one-time passcode OK].
- AUTHENTICATED (full signed session token Redis session).
- [30 min idle OR logout OR signed session token exp].


---

## 5. Security Requirements

| Requirement | Implementation |
|-------------|----------------|
| Password storage | bcrypt, cost factor 12 |
| Password minimum | 8 characters, at least 1 uppercase, 1 lowercase, 1 digit |
| User contact storage | Normalized plaintext email/phone under the explicit founder amendment; database access control and encrypted backups remain |
| Unrelated encryption/keyed message authentication uses | Retain their scoped secret-management requirements; the contact-storage amendment does not remove them |
| signed session token signing | HS256, secret min 256-bit, stored in env var |
| Cookie flags | a protected cookie that page scripts cannot read, sent only in the permitted same-site context and over encrypted connections |
| multi-factor authentication (admin) | RFC 6238 time-based one-time passcode, keyed message authentication-SHA1, 30-second step, ±1 tolerance |
| Rate limiting | 10 login attempts/hour/IP; 3 password reset requests/hour/email |
| Account lockout | 5 failed logins → 15 min; 3 failed multi-factor authentication → 15 min |
| time-based one-time passcode secret storage | Encrypted at rest (application-layer AES-SIV encryption or database-level TDE) |
| Residency/provider acceptance | Retained product requirements document/business requirements document requirements and027 evidence; regional hosting alone is not compliance proof |
| Email delivery | Durable outbox + configured Resend transport; disabled transport defers, never records success |
| encrypted web request protocol | Enforced; web request protocol redirects to encrypted web request protocol (approved027 Cloudflare Tunnel topology for production) |
| No email enumeration | Forgot-password always returns 200 |
| database query language injection | Parameterised queries via Spring Data Java persistence mapping |
| cross-site request forgery protection | Same Site is Strict cookie + origin validation on state-changing endpoints |

---

## 6. Acceptance Criteria

| ID | Criterion |
|----|-----------|
| AC-AUTH-01 | A TUTEE can register with a valid email, phone (optional), and password; receives signed session token cookie and is redirected to /dashboard. |
| AC-AUTH-02 | A TUTOR supplies required account/disclosure/consent inputs; optional profile enrichment may be skipped under 002. Abandonment creates no partial account and persists no credential in browser storage. |
| AC-AUTH-03 | The commission disclosure step (Step 2) displays the current commission text. The "Continue" button remains disabled until the acknowledgement checkbox is ticked. |
| AC-AUTH-04 | On successful TUTOR registration, a commission acknowledgements record is created with tutor id, acknowledged at, commission text version, ip address, and user agent. |
| AC-AUTH-05 | Admin registration via any public endpoint returns 404 (route does not exist). |
| AC-AUTH-06 | A user with correct credentials receives a signed session token in a a protected cookie that page scripts cannot read, sent only in the permitted same-site context and over encrypted connections cookie. |
| AC-AUTH-07 | After 5 failed login attempts, the account is locked for 15 minutes; subsequent attempts return web request protocol 423 with retry After timestamp. |
| AC-AUTH-08 | An ADMIN who passes password auth is redirected to the time-based one-time passcode challenge page and cannot access admin routes until time-based one-time passcode is validated. |
| AC-AUTH-09 | An ADMIN session expires after 30 minutes of inactivity regardless of signed session token validity. |
| AC-AUTH-10 | After 3 consecutive time-based one-time passcode failures, the admin account is locked for 15 minutes. |
| AC-AUTH-11 | Password reset one-time passcode is 6 digits, expires in 15 minutes, can only be used once. |
| AC-AUTH-12 | The forgot-password endpoint always returns web request protocol 200 regardless of whether the email exists. |
| AC-AUTH-13 | On successful password reset, all existing sessions for that user are invalidated. |
| AC-AUTH-14 | A user can request account deletion from /account/delete. Immediately after: deleted at is set, cookie is cleared, and the user cannot log in. |
| AC-AUTH-15 | 30 days after soft deletion, the nightly anonymisation job overwrites all personal information fields with placeholder values and logs the erasure. |
| AC-AUTH-16 | /admin/* routes return 403 for TUTEE and TUTOR roles; they return 401 for unauthenticated requests. |
| AC-AUTH-17 | Public/unrelated callers receive no email/phone. Owners may read their own contacts; assigned participants use the contact-reveal contract with auditing and authorization. frontend reveal remains ACTIVE/COMPLETED; match-PENDING hides it. |
| AC-AUTH-18 | The read operation for auth me endpoint returns role-appropriate fields: owner-visible contact/profile fields follow the current self-profile contract; no counterparty data is exposed through /me. |
| AC-AUTH-19 | Login rate limit of 10 attempts per hour per IP is enforced; excess returns web request protocol 429. |
| AC-AUTH-20 | Password reset is rate-limited to 3 requests per hour per email address. |
| AC-AUTH-21 | All auth endpoints are served over encrypted web request protocol only. |
| AC-AUTH-22 | A TUTEE cannot access /dashboard/tutor or any TUTOR-scoped routes. |
| AC-AUTH-23 | A user whose deleted at has a value cannot authenticate; login returns 401. |
| AC-AUTH-24 | time-based one-time passcode setup for a new ADMIN seed account is documented and must be performed before the account can log in. |

---

## 7. Dependencies

| Dependency | Purpose |
|------------|---------|
| bcryptjs or argon2 | Password hashing |
| jsonwebtoken | signed session token issuance and verification |
| otplib | time-based one-time passcode generation and validation |
| ioredis | Redis session management |
| Configured Resend client + outbox/consumer | Email handoff and durable delivery workflow |
| zod | Runtime schema validation |
| Spring Data Java persistence mapping | database access |
| Unrelated encryption provider | Only still-retained non-contact encryption uses; see current backend build dependencies |

---

## 8. Open Questions

| # | Question | Owner | Due |
|---|----------|-------|-----|
| OQ-1 | Should TUTOR accounts require email verification before activation, or is immediate access acceptable for MVP? | Product | TBD |
| OQ-2 | Commission text versioning: use a database table or a CMS version key in environment config? | Tech Lead | TBD |
| OQ-3 | Is a 30-day soft-delete window sufficient for Personal Data Protection Act right-of-erasure, or does the agency need to support shorter on-request erasure timelines? | Legal | TBD |
| OQ-4 | time-based one-time passcode seed/QR code delivery for new admin accounts: manual out-of-band or admin setup endpoint? | Tech Lead | TBD |
