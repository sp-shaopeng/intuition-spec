# Requirements — Auth session hardening 30min expiry period and refresh (004)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin**
> acceptance criteria and the **exact evidence** that will prove each scenario. Negative/guard behaviours are
> their own REQ. Source issue: **INT-013** (GH #33) — _"Auth cookie hardening: HttpOnly, 30-min expiry period, logout
> clears it, signed session token refresh-or-relogin on expiry"_, Area: backend · auth/session · Severity S2.

## Scope note (what is already done — NOT re-specified here)

Verified in code (Auth Controller set Auth Cookie ~204–217, clear Cookie ~220–230): the auth cookie is already
Http Only ✓, Secure (configurable) ✓, Same Site is Strict ✓, Path=/ ✓, and logout clears it via the max age operation ✓.
Those four are **out of scope**. The signed session token validation/revocation backbone (Jwt Authentication Filter) already enforces
the Redis jti:* denylist, blocked:user:`<id>`, and the pwreset:after:`<id>` epoch. This spec closes the **two
remaining gaps**: (1) a 30-min access-token expiry period for TUTOR/TUTEE, and (2) a sliding **refresh** flow so an active
user is not logged out every 30 minutes.

---

## REQ-001 — 30-minute access-token expiry period for TUTOR/TUTEE

- **Type:** security
- **Priority:** Must
- **Source:** INT-013 (GH #33) AC "30-min expiry period"; founder binding decision 2026-06-20.
- **Rationale / value:** A short-lived access token shrinks the replay window of a stolen/leaked token. The issue's
  reopen note (2026-06-18) confirmed regular-user cookies were effectively 12h (Max-Age is 43200) while only ADMIN
  sessions honoured 1800s. The access token (and the auth cookie max Age) must be **1800s (30 min)** for ALL roles.
- **Owner (specialist):** software-engineer (backend) · security-expert (lead)
- **Acceptance criteria (Gherkin):**


- Scenario: A freshly issued access token expires in 30 minutes.
- Given a TUTEE logs in successfully.
- When the Set-Cookie "auth" header is inspected.
- Then its Max-Age is 1800 (30 minutes).
- And the decoded signed session token "exp" - "iat" equals 1800 seconds.
- Scenario: An access token older than 30 minutes is no longer accepted.
- Given an access token whose exp is in the past.
- When the holder calls a protected endpoint (GET auth me interface operation).
- Then the request is treated as anonymous and the protected route returns 401.


- **Required evidence (defined up front):** curl i login response showing Set-Cookie value (withheld); decoded signed session token exp−iat is 1800 (jwt io / jq); curl of /auth/me with an expired token → 401 body+status; IT log. Dev Tools Application→Cookies screenshot of auth showing the 30-min expiry.
- **Dependencies:** none (config jwt expiration-seconds)

## REQ-002 — Refresh endpoint mints a new access token

- **Type:** functional
- **Priority:** Must
- **Source:** INT-013 (GH #33) AC "signed session token refresh-or-relogin on expiry"; founder binding decision (full refresh intent).
- **Rationale / value:** Without refresh, a 30-min access expiry period logs active users out every 30 min — unacceptable. A
  separate, longer-lived **refresh token** lets the client silently obtain a new access token without re-login.
- **Owner (specialist):** software-engineer (backend) · solution-architect (session design)
- **Acceptance criteria (Gherkin):**


- Scenario: Refresh issues a fresh access token for a valid refresh cookie.
- Given a TUTOR holds a valid, unexpired, unused refresh-token cookie.
- When the client POSTs auth refresh interface operation.
- Then the response is 200.
- And a new "auth" access cookie (Max-Age 1800) is set.
- And a new "refresh token" cookie is set (rotation, REQ-003).
- And the access token's subject/role match the original user.


- **Required evidence (defined up front):** curl i POST /auth/refresh with a valid refresh cookie → 200 + two Set-Cookie headers (new auth, new refresh token); decoded new access token claims; IT log (happy path).
- **Dependencies:** REQ-001, REQ-006 (refresh issued at login)

## REQ-003 — Refresh-token rotation with reuse detection (concurrency-safe)

- **Type:** security
- **Priority:** Must
- **Source:** OWASP refresh-token best practice (rotation + reuse detection); security-expert lead.
- **Rationale / value:** A stolen refresh token is high value (long-lived). Rotation (each use issues a NEW refresh and
  invalidates the OLD one) means a stolen-then-replayed token is detectable: if an already-rotated (invalidated)
  refresh token is presented, the whole refresh-token family is revoked and the request is rejected — capping theft.
- **Concurrency self-Do S (the #1 risk — design MUST address):** the naïve "any non-current token from a live family ⇒
  revoke the whole family" rule **inverts the feature into a self-Do S**: multiple browser tabs, or a stampede of
  protected calls that all 401 at once, each present the SAME current refresh token R1. The first request rotates
  R1→R2; every other in-flight request still presents R1, which is now "not current" → each trips reuse-detection
  → the family is revoked → the **legitimate user is logged out** even though nothing was stolen. The chosen stance
  (see design REQ-003 §"Concurrency control"):
  1. **Atomic compare-and-swap on rotation** — the read-current-then-write-new step is a single atomic Redis
     operation (Lua script, or WATCH/MULTI/EXEC optimistic transaction) keyed on current Token Hash, so two
     concurrent rotations cannot both "win" and cannot interleave into a torn family record.
  2. **A short grace window** — the *immediately-previous* token (the one just rotated away, R1 after R1→R2) is
     still ACCEPTED for a bounded grace window (config jwt refresh.rotation-grace-seconds, default 30s) and, when
     presented, returns **200 with no Set-Cookie and no token body** rather than tripping reuse. Only a token that is neither the
     current NOR the immediately-previous-within-grace member of a live family trips the family-wide revoke.
- **Testable invariant (backend):** on a genuine reuse trip, the refresh:revoked:`<family Id>` tombstone write **commits
  BEFORE** the 401 is returned (the family is revoked-then-rejected, never rejected-then-maybe-revoked); and the
  reuse/grace **check occurs BEFORE** a new token is minted (no successor is issued to a token that should be
  rejected).
- **Owner (specialist):** security-expert (lead) · software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: A genuinely-stale (beyond-grace) rotated token trips reuse-detection.
- Given a TUTEE refreshes successfully with refresh token R1 (rotating to R2).
- And the rotation grace window has elapsed.
- When the same R1 is presented again.
- Then the response is 401 with code Auth Error Code REFRESH REUSED.
- And the user's entire refresh-token family is revoked BEFORE the 401 is returned.
- And a subsequent refresh with the rotated R2 also fails (family revoked).
- Scenario: Two concurrent refreshes with the same token do NOT self-Do S (grace window).
- Given a TUTEE holds refresh token R1.
- When two requests present R1 concurrently.
- Then exactly one rotation wins atomically (R1  leads to  R2).
- And the family is NOT revoked and the user stays logged in.


- **Required evidence (defined up front):** IT log: refresh with R1 → 200 (issues R2); replay R1 beyond grace → 401
  Auth Error Code REFRESH REUSED; refresh with R2 after the reuse-trip → 401 (family revoked); a concurrency IT firing
  two refreshes with R1 within grace → both non-fatal, family intact. curl capture of the 401 body+status for the
  genuine-reuse replay.
- **Dependencies:** REQ-002

## REQ-004 — Refresh respects the existing revocation/reset/block kill-switches

- **Type:** security
- **Priority:** Must
- **Source:** Consistency with Jwt Authentication Filter model (blocked-user, pwreset epoch); security-expert lead.
- **Rationale / value:** A refresh token must not be a back door around the per-request kill-switches. If the user is
  blocked (blocked:user:`<id>`) or a password reset epoch (pwreset:after:`<id>`) post-dates the refresh token's
  issuance, refresh must be **rejected** and the stale credential cut off — same guarantees the access token has.
- **Epoch-comparison boundary (explicit — the opaque token has NO iat):** the access filter compares the signed session token's iat
  against the reset epoch with iat is at most pwreset:after:`<id>` (a prior Boss SEND-BACK fixed this off-by-one: a token
  minted in the SAME wall-second as the reset, iat equals epoch, is REJECTED, because epoch-seconds granularity would
  otherwise accept an attacker token minted at the reset instant — see Jwt Authentication Filter is Stale After Password Reset,
  is at most at ~line 156). The opaque refresh token carries **no signed session token and therefore no iat**. So this REQ specifies the
  refresh **family's stored creation-epoch** (refresh:family:`<family Id>`.created At Epoch, epoch seconds, fixed at
  family creation and NOT advanced by rotation) is the value compared against the reset epoch, using the **same is at most
  boundary**: refresh is rejected when family created At Epoch is at most pwreset:after:`<id>`. Fail-safe: if the family record
  is missing its creation-epoch (corrupt/legacy), treat the family as stale and reject (mirrors the filter's
  iat equals no value ⇒ stale branch).
- **Owner (specialist):** security-expert (lead) · software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: A blocked user cannot refresh.
- Given a TUTOR holds a valid refresh token.
- And an admin has blocked that user (blocked:user:`<id>` set).
- When the user POSTs auth refresh interface operation.
- Then the response is 401 (or 403 ACCOUNT BLOCKED) and NO new access cookie is set.
- And both auth and refresh token cookies are cleared.
- Scenario: A refresh family created at-or-before a password reset is rejected.
- Given a TUTEE reset their password (pwreset:after:`<id>` epoch written).
- And the user holds a refresh family whose created At Epoch is  is at most  that reset epoch.
- When the user POSTs auth refresh interface operation.
- Then the response is 401 (Auth Error Code REFRESH INVALID) and no new access cookie is issued.
- And both cookies are cleared.
- Scenario: A refresh family whose stored creation-epoch is missing fails safe.
- Given a refresh family record exists but has no created At Epoch (corrupt/legacy).
- And a password reset epoch is set for that user.
- When the user POSTs auth refresh interface operation.
- Then the family is treated as stale and the response is 401 (no new access cookie).


- **Required evidence (defined up front):** IT logs for the negative paths with rejected status (401/403) + body; curl
  captures of the rejected refresh under block and under post-reset; a branch-covering IT for the family-epoch-missing
  fail-safe; Set-Cookie max Age is 0 on both cookies.
- **Dependencies:** REQ-002, REQ-003

## REQ-005 — Logout clears AND revokes the refresh token

- **Type:** security
- **Priority:** Must
- **Source:** INT-013 AC "logout clears it" extended to the new refresh cookie; security-expert lead.
- **Rationale / value:** Logout must terminate BOTH credentials. Today logout clears auth and denylists the access
  jti. The refresh token must likewise be revoked server-side AND its cookie cleared, or a logged-out (or stolen)
  refresh token would keep minting access tokens after logout.
- **Owner (specialist):** software-engineer (backend) · security-expert (lead)
- **Acceptance criteria (Gherkin):**


- Scenario: Refresh after logout is rejected.
- Given a TUTEE is logged in (holds auth + refresh token).
- When the TUTEE POSTs auth logout interface operation.
- Then both auth and refresh token cookies are cleared (Max-Age 0).
- And the refresh-token family is revoked server-side.
- When the (now-stale) refresh token is replayed to auth refresh interface operation.
- Then the response is 401 and no new access cookie is issued.


- **Required evidence (defined up front):** curl i logout → two Set-Cookie … Max-Age is 0; replay of the pre-logout refresh token → 401 body+status; IT log.
- **Dependencies:** REQ-002, REQ-003

## REQ-006 — Login/register/multi-factor authentication issue both an access cookie and a refresh cookie

- **Type:** functional
- **Priority:** Must
- **Source:** Refresh flow needs a refresh token to exist; solution-architect (session design).
- **Rationale / value:** Every credential-establishing entry point (login, register-tutee, register-tutor, multi-factor authentication verify)
  must additionally set the refresh token cookie so the silent-refresh flow has something to rotate. The refresh
  cookie is [REDACTED CREDENTIAL], Secure, Same Site is Strict, scoped to its own path auth refresh interface operation (not /), with a
  longer idle expiry period and an absolute-lifetime cap (REQ-007).
- **Owner (specialist):** software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: Login sets a path-scoped refresh cookie.
- Given valid credentials (no multi-factor authentication).
- When POST auth login interface operation succeeds.
- Then a Set-Cookie "refresh token" header is present.
- And it is HttpOnly, Secure using config, Same Site is Strict, Path is auth refresh interface operation.
- And its Max-Age is the configured refresh idle expiry period.


- **Required evidence (defined up front):** curl i login/register/multi-factor authentication-verify responses each showing the refresh token Set-Cookie with the documented flags + path; Dev Tools cookie screenshot showing Path is auth refresh. interface operation
- **Dependencies:** REQ-001

## REQ-007 — Sliding window with an absolute-lifetime cap

- **Type:** security
- **Priority:** Must
- **Source:** OWASP session-management (idle vs absolute lifetime); solution-architect. Promoted Should→Must on
  spec-review SEND-BACK: an **uncapped sliding session is itself a security finding** — a compromised refresh family
  could be rotated indefinitely with no forced re-auth, so the absolute cap is not optional.
- **Rationale / value:** A "sliding" refresh that renews indefinitely never forces re-auth. The refresh family carries
  an **absolute** expiry (created-at + N days) independent of rotation; once the absolute cap is reached, refresh
  is rejected and the user must re-login — bounding the lifetime of a compromised long-lived credential.
- **Idle-vs-absolute precedence (explicit):** the two windows are independent. The **absolute-cap check runs even on
  an otherwise-valid rotation** — i e. a family whose idle window is still alive (recently used) but whose
  absolute Expiry Epoch is in the past is STILL rejected. Evaluation order on every refresh: (1) structural/parse →
  (2) family exists + not revoked → (3) **absolute cap** (now is at least absolute Expiry Epoch ⇒ reject REFRESH EXPIRED) →
  (4) rotation/reuse-or-grace (REQ-003) → (5) kill-switches (REQ-004) → (6) mint. The absolute cap is never skipped by
  a valid rotation.
- **Owner (specialist):** solution-architect · software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: Refresh past the absolute cap forces re-login (even on an otherwise-valid rotation).
- Given a refresh-token family whose idle window is alive but whose absolute expiry is in the past.
- When the user POSTs auth refresh interface operation with the current (structurally-valid) refresh token.
- Then the response is 401 with code Auth Error Code REFRESH EXPIRED.
- And no new access cookie is issued.
- And both cookies are cleared.


- **Required evidence (defined up front):** IT log with a clock-skewed/short-cap family whose idle window is still
  alive → 401 Auth Error Code REFRESH EXPIRED (proving the absolute check fires despite a valid idle window); curl
  capture of the rejected status+body.
- **Dependencies:** REQ-002, REQ-003

## REQ-008 — Expired refresh token (idle expiry period elapsed) is rejected

- **Type:** security
- **Priority:** Must
- **Source:** INT-013 "refresh-or-relogin on expiry"; security-expert.
- **Rationale / value:** A refresh token unused past its idle expiry period must be dead. This is the "relogin" half of the AC —
  if neither access nor refresh is valid, the user re-authenticates.
- **Owner (specialist):** software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: An idle-expired refresh token is rejected.
- Given a refresh token whose idle expiry period has elapsed (no server-side family entry).
- When the user POSTs auth refresh interface operation.
- Then the response is 401 (Auth Error Code REFRESH EXPIRED or Auth Error Code REFRESH INVALID).
- And no new access cookie is issued.


- **Required evidence (defined up front):** IT log: refresh with an expired/unknown token → 401 + body; curl capture.
- **Dependencies:** REQ-002

## REQ-009 — frontend single-flight silent-refresh-then-retry on a 401, else redirect to login

- **Type:** user interface
- **Priority:** Must
- **Source:** INT-013 "refresh-or-relogin on expiry"; frontend-expert.
- **Rationale / value:** When a protected call returns 401 because the access token just expired, the SPA must call
  /auth/refresh **once**, and on success transparently retry the original request — the user never sees a logout.
  If refresh fails, THEN the SPA logs out and redirects to login. The retry must be one-shot (no loop), mirroring the
  existing csrf Retried one-shot pattern in api implementation reference.
- **Single-flight (the frontend half of the concurrency self-Do S, REQ-003):** when several protected calls 401 at the same
  moment (e g. a dashboard fan-out), they must NOT each submit operation for auth refresh — concurrent refreshes from the same
  client would race the backend rotation and trip reuse-detection. Instead the interceptor holds **one shared in-flight
  refresh promise**: the first 401 starts the refresh; every concurrent 401 **awaits the same promise** and, on its
  resolution, replays its own original request. The shared promise is cleared once settled (so a later, genuinely-new
  expiry can refresh again).
- **Interceptor ordering + no recursion (grounded in api implementation reference):** today api implementation reference (~lines 99-102) calls
  the should trigger logout operation and logs out immediately on any non-probe 401. The silent-refresh attempt MUST run
  **BEFORE** that should Trigger Logout branch — only if the single-flight refresh returns a terminal 401/403 does control fall
  through to logout/redirect; 429 preserves the auth store and surfaces a temporary retry state. A 401 returned **by /auth/refresh itself** must BYPASS the refresh attempt (no
  recursion / no loop): add application programming interface ROUTES auth.refresh to an exclusion set (mirroring AUTH PROBE ROUTES in
  auth Redirect implementation reference) so a failing refresh goes straight to logout.
- **Pure, testable decision helper (coverage):** the one-shot / single-flight / should-we-attempt-refresh decision
  logic lives in a **pure, exported, unit-tested helper** (e g. the should attempt refresh operation in
  auth Redirect implementation reference, sibling to should Trigger Logout), NOT inline in the coverage-excluded axios block — so the
  ≥97.5% line / ≥88% branch Vitest gate actually sees and exercises the branches.
- **Owner (specialist):** frontend-expert (frontend)
- **Acceptance criteria (Gherkin):**


- Scenario: A 401 from an expired access token is healed by a silent refresh.
- Given a logged-in TUTEE whose access token has just expired.
- When a protected request returns 401.
- Then the api client calls POST auth refresh interface operation once.
- And the silent refresh runs BEFORE the should Trigger Logout branch.
- And on a 200 it replays the original request transparently.
- And the user is NOT redirected to login.
- Scenario: Concurrent 401s share a single in-flight refresh (single-flight).
- Given three protected requests return 401 at the same moment.
- When the interceptor handles them.
- Then exactly ONE POST auth refresh interface operation is issued.
- And all three originals are replayed after that single refresh resolves.
- Scenario: A 401 from /auth/refresh itself does not recurse.
- Given the silent refresh request to auth refresh interface operation returns 401.
- When the interceptor handles it.
- Then it does NOT call /auth/refresh again (no loop).
- And the auth store is cleared and the user is redirected to /login.
- Scenario: Refresh failure redirects to login.
- Given the refresh token is also invalid/expired/revoked.
- When the silent refresh returns 401.
- Then the api client does not loop.
- And the auth store is cleared and the user is redirected to /login.


- **Required evidence (defined up front):** Playwright @1280 + @390: expire the access token, trigger a protected call,
  assert the network shows …/auth/refresh then the original request replayed and the page stays logged-in; a
  fan-out capture proving N concurrent 401s yield exactly ONE /auth/refresh; a capture where refresh 401s → redirect
  to /login. Vitest unit tests of the pure should Attempt Refresh helper (one-shot, single-flight, /auth/refresh
  excluded → no loop). Screenshots both widths.
- **Dependencies:** REQ-002, REQ-003, REQ-008

## REQ-010 — Admin stateful sessions are unchanged (no regression)

- **Type:** constraint
- **Priority:** Must
- **Source:** Founder constraint "do not weaken existing admin session handling".
- **Rationale / value:** ADMIN sessions are stateful (Redis session:admin:`<session Id>` + sliding expiry period extension in the
  filter). The new TUTOR/TUTEE refresh flow must NOT alter admin behaviour. Admins continue with their 1800s sliding
  server-side session and are not issued a refresh-token cookie. **The refresh endpoint must return a DEFINED status
  for an admin context, not an ambiguous no-op:** because admins never receive a refresh token cookie, a refresh
  call carrying no (or a non-admin-owned) refresh cookie resolves to 401 Auth Error Code REFRESH INVALID on the
  ordinary "no valid refresh family" path — that IS the defined behaviour and must be asserted (no access token is
  ever minted for an admin). The decision is pinned to **401 REFRESH INVALID** (not 403, not a silent 200/no-op) so
  the contract is unambiguous; admin stateful sessions are otherwise unchanged.
- **Owner (specialist):** software-engineer (backend) · security-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Admin login does not issue a refresh token and keeps the sliding session.
- Given an ADMIN logs in.
- Then no refresh token cookie is set.
- And the admin session still extends to 1800s on each authenticated request.
- And POST auth refresh interface operation in an admin context returns 401 Auth Error Code REFRESH INVALID.
- And no access token is minted.


- **Required evidence (defined up front):** curl i admin login → no refresh token Set-Cookie; existing admin
  session IT still green; a branch-covering IT for refresh-as-admin → 401 REFRESH INVALID (status+body), no access
  cookie set.
- **Dependencies:** REQ-002

## REQ-011 — Rate-limit on the refresh endpoint (refresh-grinding defence)

- **Type:** security
- **Priority:** Must
- **Source:** security-expert — refresh-grinding threat (promoted from a buried design note to a first-class REQ on
  spec-review SEND-BACK). submit operation for auth refresh is an unauthenticated-at-the-filter surface (it runs precisely when the
  access token is gone), so it must not be a free oracle for guessing/grinding opaque refresh secrets or for hammering
  the rotation machinery.
- **Rationale / value:** Without a limit, an attacker (or a buggy client) can fire /auth/refresh at high volume —
  grinding opaque-secret space and pressuring Redis on the critical auth path. A per-IP **and** per-user (when the
  family resolves) limit, reusing the existing Rate Limit Service pattern, blunts this. Over-limit returns
  **429 Auth Error Code RATE LIMITED** without minting or rotating any token.
- **Owner (specialist):** security-expert (lead) · software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: Refresh-grinding is throttled.
- Given a client has exceeded the per-IP refresh rate limit within the window.
- When it POSTs auth refresh interface operation again.
- Then the response is 429 with code Auth Error Code RATE LIMITED.
- And no new access or refresh token is issued.
- And no rotation occurs (the family is unchanged).


- **Required evidence (defined up front):** IT driving the limiter over threshold → 429 RATE LIMITED (status+body),
  asserting no Set-Cookie and the family record unchanged (negative evidence the grinder gained nothing); curl capture
  of the 429.
- **Dependencies:** REQ-002

---

## Threat model summary (refresh token) — security-expert

| Threat (STRIDE) | Scenario | Mitigation (REQ) |
|---|---|---|
| Spoofing / Theft | Refresh cookie exfiltrated | Http Only (no JS read) + Secure + short-scoped Path is auth refresh interface operation; rotation + reuse-detection caps replay (REQ-003); absolute cap bounds lifetime (REQ-007) |
| Replay (stolen + replayed) | Old rotated token replayed | Reuse detection revokes the whole family → 401 (REQ-003) |
| Elevation via stale credential | Refresh used after block/reset/logout | Kill-switches enforced on refresh (REQ-004, REQ-005) |
| Tampering (cross-site request forgery protection) | Cross-site forced refresh | Same Site is Strict blocks cross-site send; Spring cross-site request forgery protection double-submit still applies to the POST (REQ-006 note) |
| Repudiation | No audit of refresh/abuse | Audit TOKEN REFRESHED + REFRESH REUSE DETECTED rows; reuse increments an abuse counter/metric |
| Do S / brute-force | Refresh-grinding the endpoint | Per-IP/per-user rate-limit → 429 Auth Error Code RATE LIMITED, no mint/rotate (REQ-011) |
| Self-Do S (availability) | Multi-tab / 401-stampede trips reuse-detection on the legit user | Atomic CAS rotation + rotation grace window (backend, REQ-003) + frontend single-flight refresh (REQ-009) |

**Accepted residual:** Same Site is Strict on the refresh cookie means a first navigation from an external link starts
unauthenticated until the SPA bootstraps — acceptable for this app (same posture as the existing auth cookie).

## Error-code constants (no magic strings — both ends)

Every refresh failure code referenced above (REFRESH REUSED, REFRESH EXPIRED, REFRESH INVALID, ACCOUNT BLOCKED,
RATE LIMITED) is a **named constant**, NOT a bare string literal. The backend defines a shared Auth Error Code enum (e g.
the application component Error Code) whose the code operation string feeds the existing an error object containing a code and message
envelope; the frontend mirrors the same values in a typed AUTH ERROR CODE constant map (sibling to application programming interface ROUTES) and matches
on it (the frontend only needs to distinguish "refresh failed → go to login", but it must not compare to a hand-typed
string). This satisfies the project "No magic strings — typed enums/constants both ends" rule and is enforced by Task 4a.

## 2026-09-05 exact refresh response and recovery contract (REQ-003/009/011)

- **Rotation winner:** 200 with new access and refresh cookies. **In-grace duplicate:** 200, no token body, no Set-Cookie, no second rotation or family revoke. This deliberately selects the hash-only-compatible branch: the server cannot recover R2 from its stored SHA-256 hash.
- **429 throttle:** no mint, rotation, revocation, cookie clearing, or auth-store clearing; include Retry-After. The client rejects the pending operation with a temporary retry message, releases the single-flight promise, and does not immediately loop. A later deliberate retry after that delay may refresh normally.
- **Terminal refresh 401/403:** clear both cookies and auth store; require login. A protected request is replayed at most once after refresh success; if that replay remains 401, clear the local auth state and ask for login without another automatic refresh loop.
- **Lost winner response:** no promise of recovering an unseen R2 is made. An in-grace duplicate preserves the family but cannot restore missing cookies. The bounded protected retry may therefore require a fresh login. Test delivered-winner concurrency separately from winner-response-loss; neither case may produce a second in-grace rotation or false family revoke.
- Required verification includes exact response headers for winner/duplicate/429/terminal rejection, browser auth-store behavior for 429, and both concurrency response-order/loss cases. This is a technical clarification within the existing hash-only and one-shot requirements, not runtime evidence.
