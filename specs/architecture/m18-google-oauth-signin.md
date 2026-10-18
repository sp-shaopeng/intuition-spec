# M19 — Google Sign-up / Sign-in (OAuth2 / OIDC)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Locked §11 decisions govern: Option B proof-of-control linking; tutor role creates a usable account immediately, commission/qualifications gate posting/applying; encrypted stateless OAuth cookies and fresh cross-site request forgery protection cookie. Earlier inline-activation/auto-link alternatives are historical, not current design.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

> **⚠️ Renumbered 2026-06-02 (INT-023): this feature is now M19, not M18.** "M18" is the deployed
> Assignment-Lifecycle & Free-Marketplace (migrations 0025–0030). Any migration reserved by this spec
> takes the **next free number 0031** — the previously-written "migration 0024" is WRONG (0024 is the
> deployed audit-log fix). Treat every "M18"/"0024" below as **M19**/**0031**. (Filename kept as
> m18-google-oauth-signin md to preserve inbound links; content supersedes it.)

Status: **SPEC — team-reviewed + Boss-cleared to implement (behind flag)** · Author: Architect + Product · Date: 2026-05-30
Scope of this document: **specification only — NO code implementation.** Plan → product requirements document →
architecture → data/migration → backend changes → frontend changes → security → privacy →
rollout/roadmap → open questions.

> Grounding (verify exact symbols at implementation time): current auth is
> email/password → **signed session token in an HttpOnly cookie** + cross-site request forgery protection double-submit (bcrypt
> passwords). Endpoints: auth login interface operation, register/tutee, register/tutor,
> register/commission, logout, me, csrf.Userhas plaintextemail (unique),
> phone, password Hash, role (TUTOR/TUTEE/ADMIN), is Super Admin,
> password Change Required; display name lives on the profile. No OAuth deps yet.

---

## 1. Plan (summary)

Add **"Continue with Google"** to the Login and Register screens. Use Open ID Connect
**Authorization Code + PKCE** via Google. After Google verifies the user, the **backend
issues the same signed session token session cookie we already use** — so the rest of the app
(/auth/me, guards, cross-site request forgery protection) is unchanged. A **brand-new** Google user must still choose a
**role** (tutor/tutee) and, if tutor, may defer commission/qualification until posting/applying; role completion creates a usable browse account before
the account is fully active. Existing email/password users can **link** Google to their
account. Phase 1 is tutor/tutee only; admins stay password-only.

**Why:** lower signup friction, faster repeat login, fewer password resets, and a trust
signal ("Sign in with Google") for first-time Singaporean parents and tutors.

---

## 2. product requirements document

### 2.1 Goals
- One-click sign-up and sign-in for tutors and parents via Google.
- Reuse the existing session/authz model (signed session token cookie) — Google is an *identity source*,
  not a new session system.
- Safe account **linking** so a user who used email/password can later use Google with
  the same email (and vice-versa) without creating a duplicate.

### 2.2 Non-goals (phase 1)
- No Google login for **ADMIN** accounts (admins remain email/password per M16).
- No other providers (Apple/Facebook) — designed to extend, not built now.
- No Google scopes beyond identity (openid email profile). No Calendar/Drive/contacts.
- No storing of Google access/refresh tokens — we only consume the ID token at sign-in.
- No Google One-Tap / Fed CM in phase 1 (button only); revisit later.

### 2.3 User stories
- **New parent/tutor:** "I click *Continue with Google*, approve, pick *I'm a
  parent / I'm a tutor*, and I'm in — no password to invent."
- **New tutor specifically:** after choosing *tutor*, I complete the existing
  commission acknowledgement + qualifications step before going live.
- **Existing email/password user:** "I click *Continue with Google* with the email I
  already use → it links to my existing account and signs me in (no duplicate)."
- **Returning Google user:** "One click and I'm in."
- **Security-conscious parent:** "My contact details and child's data are only ever
  shared per the existing privacy rules — Google just proves who I am."

### 2.4 Functional requirements
1. "Continue with Google" button on /login and /register (both role-aware register
   entry points), visually distinct, Google-brand-compliant.
2. New Google user → **role-selection** screen (tutor/tutee). Tutor → existing
   commission/qualification flow. Until role is set, the account is **incomplete** and
   cannot use role-gated features.
3. email verified is yes from Google is **required**; unverified Google emails are
   rejected.
4. **Account linking** by verified email (rule in §7.3).
5. Logout, /auth/me, cross-site request forgery protection, and all guards behave identically afterward.
6. Clear error states: user cancels (access denied), unverified email, linking
   conflict, provider/network error.

### 2.5 Success metrics
- % of new signups via Google; median time-to-first-account.
- Login success rate; password-reset ticket volume (expected ↓).
- Drop-off on the role-completion step (must stay low).

### 2.6 Acceptance criteria (for the eventual build)
- New + returning Google sign-in works for tutor and tutee; new users complete a role.
- No duplicate account is ever created for an email that already exists.
- Admins cannot obtain ADMIN via Google. email verified is no is refused.
- Session post-Google  equals  existing signed session token-cookie session (same /auth/me, cross-site request forgery protection, guards).

---

## 3. Architecture design

### 3.1 Flow (OIDC Authorization Code + PKCE)


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.2 Build approach (recommendation for the team)
Use **Spring Security OAuth2 Login** (spring-boot-starter-oauth2-client) rather than a
hand-rolled token exchange — the framework handles PKCE, state, nonce, and ID-token
signature/claims validation against Google's JWKS. We customize only:
- a **stateless, cookie-based Authorization Request Repository** (so the OAuth handshake
  needs no server-side web request protocol session — consistent with our stateless signed session token model; the
  cookie is short-lived, HttpOnly, Same Site is Lax, signed),
- an **Oidc User Service** that maps the Google identity to our User,
- an **Authentication Success Handler** that mints our existing signed session token cookie (or the
  registration token for new users) and 302s to the frontend,
- an **Authentication Failure Handler** that 302s to /login?error is ....
*(Alternative considered: manual code exchange via a REST endpoint. Rejected — re-implements
PKCE/state/nonce/JWKS validation, more surface for mistakes. Architect to confirm.)*

### 3.3 Session model
After success the user holds the **same HttpOnly signed session token cookie** as today. The OAuth2
client state is used only during the redirect handshake (cookie-based, then discarded).
No Spring Http Session for end users. Admin Redis sessions are unaffected.

### 3.4 Config / redirect URIs
- Google client registration: client-id + client-secret from env (Railway), scopes
  openid email profile, redirect-uri =
  base Url/login/oauth2/code/google.
- Redirect URIs registered in Google Cloud console per environment (local
  http://localhost/..., prod https://example com/...). Allowlist only.

---

## 4. Data model / migration (Liquibase **0024** — spec only)

Add to users:
| Column | Type | Notes |
|--------|------|-------|
| auth provider | **the varchar operation + require auth provider IN LOCAL,GOOGLE** | the enumerated operation on the Java side. **NOT a native Postgres enum** — a native enum in an IS no value filter caused the M17 500 (could not determine data type); use varchar+require like invoice status in 0010. |
| google sub | varchar unique NULL | Google's stable subject id (sub). Unique partial index when required. The real join key (emails can change). |
| email verified | boolean required default no | True once proven (Google verified, or future email-verify flow). |
| password hash | → make **NULLABLE** | Google-only accounts have no password. |

- Backfill: existing rows → auth provider is LOCAL, email verified = (decision §7.2),
  google sub is no value.
- email stays unique (already). New constraint: unique google sub.
- A user may have BOTH a password and a google sub (linked account).

---

## 5. Backend changes (spec)

1. **Dependency:** add spring-boot-starter-oauth2-client.
2. **Config:** spring security.oauth2 client.registration google.* (id/secret/scope/
   redirect) + provider defaults; client secret from env.
3. **Security config:** enable the oauth2 login operation with the custom request-repo, user
   service, success/failure handlers; permit All /oauth2/** and /login/oauth2/**;
   keep the existing signed session token filter for api ** interface operation; keep Session Creation Policy STATELESS;
   cross-site request forgery protection-exempt the OAuth callback (/login/oauth2/**, /oauth2/** — a top-level
   browser redirect carries no cross-site request forgery protection token; state is the protection). **The success
   handler MUST mint a fresh cross-site request forgery protection cookie alongside the signed session token cookie** — otherwise the
   user's first post-login mutating request 403s on a stale cross-site request forgery protection token (the solution architect). All
   OAuth beans/endpoints are gated by the auth google.enabled flag (backend, not just frontend).
4. **User / Auth Service:** find By Google Sub, create-from-Google (no password,
   email verified from token), **link** (set google sub on an existing verified
   account), role assignment deferred for new users.
5. **Registration completion:** submit operation for auth google complete — accepts the
   role (TUTEE/TUTOR) (+ for tutor, hand off to the existing commission/qualification
   step). Authorized only by the short-lived **registration token** minted at callback
   (NOT a full session). Cannot set ADMIN.
6. **Endpoints summary:**
   - read operation for oauth2 authorization google (start) — framework.
   - read operation for login oauth2 code google (callback) — framework + our success handler.
   - submit operation for auth google complete (new) — finish a new Google signup.
   - existing /auth/me, /auth/logout, /auth/csrf unchanged.
7. **Admin exclusion:** a Google identity never maps to or creates an ADMIN; role
   completion offers TUTEE/TUTOR only.
8. **Observability:** counters for google login new, returning, linked, rejected_*,
   failure reasons; audit-log a link event and a new-account-via-google event.

---

## 6. Frontend changes (spec)

1. **"Continue with Google"** button on Login implementation reference and Register implementation reference (and the
   role-aware register entries). Clicking → full-page redirect to
   the Google sign-in entry point, preserving only a safe return path on this site.
   (It is a backend redirect, NOT an axios/XHR call — so it never touches the 401
   interceptor.)
2. **Role-completion page** (new route, e g. /auth/complete): shown when the backend
   redirects a brand-new Google user there. Choose *I'm a parent (tutee)* / *I'm a
   tutor*; tutor → reuse the existing commission/qualification flow; calls
   submit operation for auth google complete; then land on dashboard / return To.
3. use Init Auth / auth Store unchanged (the cookie is set server-side; /auth/me
   resolves the user as usual).
4. **Error handling:** /login?error is access denied|email unverified|link conflict|
   provider error → friendly inline message.
5. **Constants:** application programming interface ROUTES auth.google (start) + auth google Complete; no magic
   strings. Google-brand button asset/guidelines.
6. **user experience:** button placement and copy per Jamie; keep the existing email/password path
   equally prominent (don't force Google).

---

## 7. Security (Casey — to be hammered in review)

1. **ID-token validation (non-negotiable):** verify signature against Google JWKS,
   a recognised Google account issuer, aud equals client id,
   exp not passed, nonce matches. (Framework does this when configured correctly —
   review must confirm it's not bypassed.)
2. **email verified gate:** reject Google identities with email verified differs from yes —
   prevents claiming someone else's email.
3. **PKCE + state + nonce:** mandatory; state is cross-site request forgery protection for the callback. Use the
   signed, HttpOnly, short-expiry period cookie-based authorization-request repo.
4. **Redirect-URI allowlist** in Google console; no wildcard. **return To must be a
   same-origin relative path** (reuse the M17 safe Return To validation) — no open
   redirect through the OAuth flow.
5. **Account-takeover via linking** (the big one — §7.3 decision).
6. **Client secret** only in env/Railway secrets; never in the repo or the frontend bundle.
7. **Registration token** (new-user completion) must be short-lived, single-purpose,
   and unable to act as a full session or self-assign ADMIN.
8. **Rate-limit** the callback + completion endpoints; audit-log link/new events.

### 7.3 Account-linking decision (OPEN — team must choose)
When a Google sign-in's verified email matches an existing **LOCAL** account:
- **Option A (auto-link):** link automatically because Google verified the email.
  Lowest friction; relies entirely on Google's verification.
- **Option B (verify-to-link):** require the user to also enter their existing
  password (or a one-time email confirmation) before linking. Safer against edge cases
  (e g. an email re-registered at Google). Slight friction.
- **Recommendation:** **Option B for accounts that have a password set**; auto-link is
  acceptable only when the existing account has no usable password. Final call: team.

---

## 8. Privacy / Personal Data Protection Act
- Data received from Google: email, name, sub, email verified (and optionally
  picture — **recommend NOT storing the picture** in phase 1 to minimize data).
- Update the Privacy Policy + consent copy to disclose Google sign-in and what is
  received. Data minimization: store only email, display name, google sub,
  email verified. No Google tokens persisted.
- Existing personal information model unchanged (email plaintext; counterparty never sees contact).

---

## 9. Rollout / Roadmap
- **M18 phase 1:** tutor/tutee Google sign-in + linking + role completion; migration
  0024; behind a **feature flag** (auth google.enabled) so the button can be dark-
  launched and rolled back without a deploy.
- **Phase 2 (later):** admin Google sign-in (if ever), One-Tap/Fed CM, possibly Apple.
- Add to intuition-spec/specs/ROADMAP md as **M18** (Tier: next major auth feature),
  with dependency: Google Cloud OAuth client + secrets provisioned in Railway.

---

## 10. Open questions — RESOLVED by team review (2026-05-30)
- **Q1 Linking → Option B (no auto-link).** Resolved §11.1.
- **Q2 email verified backfill → FALSE** for all existing rows. §11.4.
- **Q3 → Spring OAuth2 Login** (with stateless cookie request-repo). §11.6.
- **Q4 Google on showcase CTAs → NO** (phase 1: login/register only; measure first).
- **Q5 Profile picture → SKIP** storing it (no product value, extra personal information surface).
- **Q6 Admins via Google → NO** (password-only; revisit phase 2).
- **Q7 New-tutor path → SPLIT activation.** §11.2.

## 11. Team Review Decisions (LOCKED — panel + Boss, 2026-05-30)

Cleared to implement; these override §1–§9 where they differ. Implementation Boss
review will verify every item.

1. **Account linking = Option B, FINAL.** No silent auto-link, ever. If a Google
   sign-in's verified email matches an existing account that has a usable password, the
   user must **prove control** (enter the existing password OR click an emailed
   one-time link) before google sub is bound. Auto-link permitted ONLY when the
   existing account has **no usable password AND no prior successful login**. Linking is
   always explicit, authenticated, and **audited**. sub is the permanent join key —
   **never re-link by email** after the initial bind. Errors are generic (no account
   enumeration).
2. **Tutor activation = SPLIT, FINAL.** After Google, the user answers ONE question
   (parent or tutor) → a usable account is created immediately (can browse). Commission
   + qualifications gate ONLY *posting a listing / applying to a job* (progressive
   onboarding — no inline paperwork wall at signup). Parent = one tap + role = done.
   Capture the **role-completion funnel** metric (Google return → role chosen →
   [tutor] activated).
3. **Identities model = single-provider shortcut, documented.** Ship auth provider +
   google sub + email verified on users now. Do NOT build the 1:N table yet, but
   it is hereby RECORDED that phase 2 (Apple/FB) migrates to
   the user identities operation — no column-per-provider.
4. **Data layer (DBA):** the varchar operation +
   require auth provider IN LOCAL,GOOGLE + the enumerated operation — **NOT a
   native Postgres enum** (a native enum in an IS no value filter caused the M17 500).
   google sub partial-unique index mirroring idx users phone
   (... when google sub has a value). email verified backfill = **FALSE** for all
   existing rows. Migration 0024-* implementation reference, plain support efficient lookup with an index (Testcontainers-safe).
5. **Login fail-closed:** the password-login path MUST reject password hash IS no value
   with the generic invalid-credentials response **before** invoking the bcrypt encoder
   — with a regression test. A Google-only account can never be password-logged-in.
6. **Statelessness + cross-site request forgery protection:** use a cookie-based Authorization Request Repository that is
   **encrypted** (it carries the PKCE code verifier), HttpOnly, Secure, Same Site is Lax,
   expiry period ≤10 min, deleted on callback. Preserve Session Creation Policy STATELESS
   (oauth2Login's default Http Session repo must NOT ship). The success handler issues
   **both** our signed session token cookie AND a fresh cross-site request forgery protection cookie (else the first post-login mutation
   403s).
7. **Registration token:** signed, single-purpose (scope is registration), bound to
   google sub, expiry period ≤15 min, **rejected by all authenticated endpoints**, carries NO
   role; submit operation for auth google complete **hard-rejects ADMIN** server-side and stays under
   cross-site request forgery protection. Abandoned completion leaves **no durable row** (token, not partial account) —
   no future-signup-blocking orphan.
8. **Feature flag auth google.enabled gates the BACKEND**, not just the button —
   flagged-off → /oauth2/** and /auth/google/** disabled (404). A Google outage must
   not touch the email/password path (no shared failure dependency).
9. **ID-token validation explicitly verified, not assumed:** iss exact-match, aud  equals 
   client idexpnonce sent+verified, JWKS signature w/ key rotation;
   email verified coerced safely (Google may emit string yes) and required true.
   No custom Oidc User Service that bypasses the default OIDC validators.
10. **return To** carried inside signed OAuth state, re-validated same-origin
    server-side (reuse M17 safe Return To); **client secret in Railway env only** (never
    repo/frontend); Privacy Policy updated + a visible **"we only use Google to sign you in —
    we never see your password"** reassurance line (Priya).

## 2026-09-05 current contract and verification

Apply the locked §11 contract to every early outline: Google role selection → usable TUTEE/TUTOR account; a tutor completes commission/qualification only when attempting a gated action. Never silently link an existing password-bearing account by matching email; require password or one-time-link proof, audit the link, and thereafter join by stable sub. Earlier /auth/complete diagrams that imply commission blocks account creation are historical examples. Verify both entry points and missing-cross-site request forgery protection first mutation, without claiming code is changed here.
