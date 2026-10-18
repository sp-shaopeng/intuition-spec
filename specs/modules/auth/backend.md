# InTuition — Auth Module — Backend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Current contracts are001 consent/version carry,002 onboarding/profile,003 reset with production logging OFF via027,004 access/refresh/cross-site request forgery protection,012 contact change and M16 local admin bootstrap. Earlier multi-factor authentication/seed and request examples are historical where those amendments differ.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Module:** auth  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. application programming interface Endpoints

All endpoints are prefixed auth interface operation. All responses use Content-Type: application/json.

### 1.1 Register Tutee


The illustration for register tutee contains the following information:

- POST auth register tutee interface operation.
- Authorization: None (public).


**Request Body:**


The register tutee example describes information including email, phone, password, display Name. Concrete payload values and implementation syntax are omitted.


**Processing:**
1. Validate all fields with Jakarta Bean Validation
2. Check canonical email uniqueness — 409 if duplicate
3. Check canonical phone uniqueness — 409 if duplicate
4. Hash password with bcrypt (cost factor 12)
5. Normalize and persist email under the current plaintext-contact contract
6. Normalize and persist phone under the current plaintext-contact contract
7. Insert users row (role = TUTEE)
8. Insert tutee profiles row (display Name)
9. Issue signed session token, set http Only cookie
10. Log USER REGISTERED to audit log

**Response 201:**


The register tutee example describes information including user, id, role, display Name. Concrete payload values and implementation syntax are omitted.


**Errors:** 400 (validation), 409 (email/phone already registered)

---

### 1.2 Register Tutor


The illustration for register tutor contains the following information:

- POST auth register tutor interface operation.
- Authorization: None (public).


**Request Body:**


The register tutor example describes information including email, phone, password, display Name, qualifications, years Experience, commission Acknowledged, commission Terms Version. Concrete payload values and implementation syntax are omitted.


**Processing:**
1. Validate all fields with Jakarta Bean Validation. If commission Acknowledged differs from yes → 400 with code COMMISSION ACK REQUIRED
2. Check canonical email and phone uniqueness — 409 if duplicate
3. Hash password, normalize/persist plaintext contact fields (same as tutee flow)
4. Insert users row (role = TUTOR)
5. Insert tutor profiles row
6. Insert commission acknowledgements row (user id, version, ip address)
7. Issue signed session token, set http Only cookie
8. Log USER REGISTERED and COMMISSION ACKNOWLEDGED to audit log

**Response 201:** same shape as tutee, role = TUTOR

**Errors:** 400 (validation, missing commission ack), 409 (email/phone conflict)

---

### 1.3 Login


Implementation example omitted. This public edition retains the written requirements in this section.


**Request Body:**


The login example describes information including email, password. Concrete payload values and implementation syntax are omitted.


**Processing:**
1. Normalize input email to the canonical lookup form
2. Lookup users by canonical email — generic 401 if not found (no email enumeration)
3. Check deleted at IS no value — 401 if soft-deleted
4. Check locked until — 423 with retry After if account locked
5. Verify the compare operation — increment failed login attempts on failure; 401 after 5 attempts set the account lock to expire 15 minutes after the current time
6. If role = ADMIN → do NOT issue signed session token yet; return mfa Required value (withheld) → client must call /login/mfa
7. If role = TUTEE/TUTOR → reset failed login attempts 0, update last login at, issue signed session token, set cookie

**Response 200 (TUTEE/TUTOR):**


The login example describes information including user. Concrete payload values and implementation syntax are omitted.


**Response 200 (ADMIN):**


Implementation example omitted. This public edition retains the written requirements in this section.


(plus Set-Cookie value (withheld))

**Errors:** 401 (invalid credentials), 423 (account locked with retry After)

---

### 1.4 Admin multi-factor authentication Verification


The illustration for admin multi-factor authentication verification contains the following information:

- POST auth login mfa interface operation.
- Authorization: mfa session cookie (5-min expiry period).


**mfa session token format:** 128-bit cryptographically random bytes, base64url-encoded (not unique identifier, not sequential). Generated via java security.Secure Random. Stored as bcrypt hash in Redis with 5-min expiry period; plaintext never stored.

**Request Body:**


The admin multi-factor authentication verification example describes information including totp. Concrete payload values and implementation syntax are omitted.


**Processing:**
1. Validate mfa session cookie — 401 if expired/missing
2. Look up user from session token
3. Verify time-based one-time passcode against mfa secret using dev samstevens.totp (Java time-based one-time passcode library) — time-based one-time passcode window = ±1 step
4. Invalidate mfa session cookie
5. Issue full signed session token, set auth http Only cookie
6. Store session key in Redis with 30-minute idle expiry period (for admin idle timeout)
7. Log ADMIN LOGIN to audit log

**Response 200:**


The admin multi-factor authentication verification example describes information including user. Concrete payload values and implementation syntax are omitted.


**Errors:** 401 (invalid/expired time-based one-time passcode or session), 429 (3 failed time-based one-time passcode attempts → require re-login)

---

### 1.5 Logout


The illustration for logout contains the following information:

- POST auth logout interface operation.
- Authorization: auth cookie.


**Processing:**
1. Clear auth http Only cookie (Set-Cookie with Max-Age is 0)
2. If ADMIN: delete Redis jti allowlist key
3. If TUTEE/TUTOR: add jti to Redis denylist — key = a stored revocation entry for the session token’s unique identifier, value = 1, expiry period = remaining token exp seconds (calculated as exp - current Epoch Seconds)
4. Log USER LOGOUT to audit log

**Response 200:** a success confirmation

---

### 1.6 Get Current User


The illustration for get current user contains the following information:

- GET auth me interface operation.
- Authorization: auth cookie required.


**Response 200:**


The get current user example describes information including id, role, display Name, email, phone, created At. Concrete payload values and implementation syntax are omitted.


Note: email and phone are decrypted here because the user is viewing their own profile. Admin viewing other users' contact info goes through admin users :id interface operation.

---

### 1.7 Forgot Password


The illustration for forgot password contains the following information:

- POST auth forgot-password interface operation.
- Authorization: None (public).
- Rate limit: 3 requests / hour / email.


**Request Body:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Processing:**
1. Normalize email, lookup user — always return 200 (no email enumeration)
2. If found and not deleted: generate 6-digit one-time passcode, hash with bcrypt (cost 10), insert password reset tokens (otp hash, expires at = NOW operation + 15 min, user id)
3. Invalidate any prior unused tokens for this user
4. Queue email N-05 (password reset one-time passcode) to InTuition async email queue (Spring @Async + Redis)

**Response 200:** a success confirmation (always, regardless of whether email exists)

---

### 1.8 Reset Password


The illustration for reset password contains the following information:

- POST auth reset-password interface operation.
- Authorization: None (public).


**Request Body:**


The reset password example describes information including email, otp, new Password. Concrete payload values and implementation syntax are omitted.


**Processing:**
1. Lookup user by canonical email
2. Find latest unused, non-expired password reset tokens record for user
3. Verify the compare operation — 400 on mismatch
4. Mark token as used (record the current time as the token’s used-at time)
5. Update password hash with new bcrypt hash
6. Reset failed login attempts 0, clear locked until
7. Invalidate all active sessions (clear Redis keys for admin)
8. Log PASSWORD RESET to audit log

**Response 200:** a success confirmation

**Errors:** 400 (invalid one-time passcode, expired one-time passcode)

---

### 1.9 Delete Account (Personal Data Protection Act Erasure)


The illustration for delete account personal data protection act erasure contains the following information:

- DELETE auth account interface operation.
- Authorization: auth cookie required.


**Request Body:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Processing:**
1. Verify password before deletion (prevent accidental/hijacked deletion)
2. Record the current time as the user’s deletion timestamp (soft delete)
3. Clear auth cookie / Redis session
4. Schedule anonymisation pipeline to run after 30 days
5. Log ACCOUNT DELETION REQUESTED to audit log

**Response 200:** a success confirmation

Note: business deletion is logical. Anonymization timing, retained financial/audit fields and FK-safe erasure require O08 in the current decision register; do not treat the older30-day job as approved policy.

---

## 2. Middleware

### 2.1 Jwt Authentication Filter

Extends Once Per Request Filter. Validates auth http Only cookie:
1. Verify signed session token signature (HS256 with signed session token SECRET CURRENT; also try signed session token SECRET PREVIOUS during rotation window) and expiry
2. Check kid claim — if present, use corresponding key version
3. If ADMIN: check Redis jti allowlist key exists and refresh its expiry period (30-min idle). Missing key → 401.
4. If TUTEE/TUTOR: check Redis jti denylist — a stored revocation entry for the session token’s unique identifier. Present key → 401 (token revoked by logout).
5. Set Security Context Holder with authenticated Username Password Authentication Token carrying the authenticated user’s identifier and role

### 2.2 @Pre Authorize / Security Config role checks

Configured in Spring Security Security Filter Chain. After auth, checks role. Returns 403 if not permitted.

### 2.3 Admin route guard

All admin ** interface operation routes require ROLE ADMIN in Spring Security configuration.

---

## 3. signed session token Structure


The . signed session token structure example describes information including sub, role, session Id, iat, exp. Concrete payload values and implementation syntax are omitted.


**Algorithm:** HS256 (symmetric). Secret: minimum 256 bits, stored in signed session token SECRET environment variable — never in code or database.

**Key rotation:** Include kid (key ID) header claim. During rotation: for 24 hours accept tokens signed by either old or new key, then retire old key. All sessions signed with old key are invalidated after the 24-hour window. Rotation procedure: (1) generate new secret, (2) deploy with both secrets in signed session token SECRET CURRENT + signed session token SECRET PREVIOUS, (3) wait 24 hours, (4) remove signed session token SECRET PREVIOUS.

**Why HS256 over RS256:** No third-party token verifiers needed. Single issuer (Spring Boot). Symmetric secret is simpler to manage for a 2-person team. If external verifier integration is ever required, migrate to RS256 at that point.

**TUTEE/TUTOR access-token lifetime:**30 minutes under spec004, with opaque hash-only rotating refresh sessions (7-day idle/30-day absolute defaults). On logout, add jti to Redis denylist: key = a stored revocation entry for the session token’s unique identifier, value = 1, expiry period = remaining token exp seconds. Jwt Authentication Filter checks a stored revocation entry for the session token’s unique identifier — return 401 if present (token revoked).

**Admin token:** Session tracked in Redis jti allowlist (30-min idle expiry period). On idle expiry, admin must re-authenticate.

---

## 4. Cookie Attributes


Implementation example omitted. This public edition retains the written requirements in this section.


**cross-origin request permissions strategy:** The SPA (example com) and application programming interface (example com/api/*) share the same origin via Cloudflare reverse proxy — api * interface operation requests are routed by a Cloudflare Worker to the Railway Spring Boot backend. Same-origin delivery means Same Site is Strict cookies work without cross-origin cookie delivery. cross-origin request permissions headers are NOT required between the SPA and the application programming interface.

**cross-site request forgery protection decision:** double-submit XSRF protection is required on cookie-authenticated mutations, including refresh; Same Site is Strict is additional defense. Preserve the narrowly specified auth/one-time passcode exemptions, bootstrap a fresh cross-site request forgery protection cookie after password/Google login, and include the header on both frontend and backend contract paths. The old Csrf Filter-disabled design is superseded by M19 and spec004. Production origin/cross-origin request permissions behavior follows027, not an assumed Cloudflare proxy.


## 4.1 Java persistence mapping Soft Delete Filtering

All users queries must exclude soft-deleted records. Enforce via Hibernate's @Where annotation on the User entity:


Implementation example omitted. This public edition retains the written requirements in this section.


This means all Spring Data Java persistence mapping queries (including derived methods and @Query JPQL) automatically append AND deleted at IS no value. No application-layer discipline required.

**Historical anonymization-job example:** The older design selected users with a recorded deletion timestamp at least 30 days old, using a dedicated query that can include logically deleted accounts. Current O08 must settle the retention and erasure schedule; this example does not approve a 30-day policy. The ordinary user lookup excludes deleted accounts, so the dedicated job cannot rely on that lookup.

---

## 5. Contact storage — explicit plaintext amendment

Current baseline C13 supersedes the former AES-SIV/keyed message authentication lookup design: user email/phone are stored using the approved plaintext-contact model and canonical normalized uniqueness. Passwords remain hashed; public data transfer object allowlists, ownership/contact-release authorization, audit and no-sensitive-log rules remain mandatory. Do not recreate encrypted-contact columns or require Encryption Service for registration/login based on the old examples. Approved anonymization/retention is O08; no unconditional30-day physical deletion is specified here.

## 6. Rate Limiting

Implemented at Spring Boot filter layer (not business logic):

| Endpoint | Limit | Window | Key |
|---|---|---|---|
| auth login interface operation | 10 requests | 1 hour | IP + email hmac |
| auth forgot-password interface operation | 3 requests | 1 hour | IP + email hmac |
| auth register * interface operation | 5 requests | 1 hour | IP |
| auth login mfa interface operation | 3 attempts | 5-min session | session token |

Rate limit state stored in Redis with sliding window counters.

---

## 7. Error Response Format


The . error response format example describes information including error, code, message, retry After. Concrete payload values and implementation syntax are omitted.


Error codes never reveal whether an email exists (use generic INVALID CREDENTIALS for both "email not found" and "wrong password").

---

## 8. Account self-service — contact change (spec 012 REQ-004, Wave 2)

Account Controller (account interface operation, authenticated) lets a user change their own **recovery contact** (account
email or phone) through an **email-link confirmation** flow. The live users email / users phone is **unchanged
until the user confirms** the change via a link sent to their verified account email, so the OLD value stays active
for all auth/recovery ops while a change is pending. PENDING requests live in contact change request
(**migration 0066**: field EMAIL|PHONE + require, channel EMAIL LINK + require, status
PENDING|CONSUMED|EXPIRED|SUPERSEDED + require — soft-delete only; bcrypt token hash; single-use consumed at;
partial index user id, field when status is PENDING).

| Method | Path | Body | Result |
|---|---|---|---|
| POST | /account/contact-change | the contact field to change, its proposed new value and the current password for identity confirmation | **200** Contact Change Response. **current Password = mandatory step-up re-auth** (wrong → **422 STEP UP REQUIRED**). Bad email/phone or unknown field → **400 VALIDATION ERROR**. Supersedes any existing PENDING for that field; mints a token; **emails the confirm link to the verified account email**. Live email/phone NOT touched |
| GET | /account/contact-change | — | **200** Contact Change Response — current PENDING changes (drives the "verification pending" banner) |
| POST | /account/contact-change/verify | the confirmation credential | **200** Contact Change Response (remaining pending set). **POST-only — never a bare-GET side-effect** (email scanners prefetch links; a GET auto-commit would let a prefetcher silently commit). frontend landing page POSTs the token on an **explicit Confirm click**. Wrong/expired/consumed → **400 CONTACT CHANGE TOKEN INVALID**. Uniqueness re-checked AT COMMIT (TOCTOU-safe) → **409 EMAIL ALREADY REGISTERED / PHONE ALREADY REGISTERED** |

**Contact Change Response** = the list of contact changes still awaiting confirmation; **Pending Contact Change** =
the contact field, a partially hidden version of its proposed new value and its expiry time.

**Anti-account-takeover (3 defences):** (1) **step-up** — initiate requires the current password; (2) **notify-both** —
the change is surfaced so a silent re-point is visible; (3) **confirm-click** — the commit is a POST behind an explicit
user action, never a prefetchable GET. The OLD recovery contact stays live until the user actively confirms.

> PHONE ALREADY REGISTERED here shares the exact code + message used by registration and replace operation for account phone —
> the three sites must not drift.

## 2026-09-05 current contract and verification

M16 explicitly permits local admin creation and defers multi-factor authentication in its scope; do not use the old mandatory-multi-factor authentication section as dev acceptance criteria or infer production multi-factor authentication approval. Password policy must be the single current shared rule on registration/reset/change, not differing comments in this legacy data transfer object sample; unresolved policy details belong to [current open decisions](../../CURRENT-REQUIREMENTS.md). Refresh grace duplicates are plain200/no Set-Cookie; temporary429 preserves state with Retry-After, terminal401/403 clears per004. Verify each relevant request producer and consumer; no runtime claim is added here.
