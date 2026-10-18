# Design — Auth session hardening 30min expiry period and refresh (004)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> How the requirements in [requirements md](requirements.md) are built. Owned by **security-expert** (lead) with
> **solution-architect** (session design) and **software-engineer** (impl); **frontend-expert** owns the frontend
> silent-refresh. Grounded against the real code read on 2026-06-20.

## Architecture

Two token types, two cookies, one Redis-backed revocation model — reusing the existing signed session token + Redis backbone.


Implementation example omitted. This public edition retains the written requirements in this section.


- **Access token:** unchanged shape (HS256, claims iss/aud/sub/exp/iat/jti/role/session Id, Jwt Token Provider),
  expiry period driven by jwt expiration-seconds is 1800. Already wired (verified in application implementation reference ~54). The auth
  cookie max Age is the SAME property (Auth Controller cookie Max Age), so REQ-001 is config-aligned for all roles.
- **Refresh token:** an **opaque, high-entropy** random value (NOT a signed session token) — server is the only authority on its
  validity. **Entropy + wire format:** the secret is **≥256 bits drawn from java security.Secure Random**, encoded
  base64url; the cookie value is `<family Id>`.`<secret>` (family Id = a unique identifier, secret = the base64url random) so the
  server can look up the family in O using 1 without scanning. The browser holds only this opaque string; all state
  (family id, rotation chain, absolute expiry, creation epoch, owner, role) lives server-side in Redis. This keeps
  reuse-detection and revocation purely server-driven (see ADR-001).
- **Stored secret = SHA-256 hash, constant-time compare:** the server stores only **SHA-256 secret** (hex/base64),
  NEVER the raw secret. A fast cryptographic hash is correct here (NOT bcrypt/argon2): the secret is already ≥256-bit
  high-entropy, so there is nothing to brute-force — a slow password hash would only add latency to the critical auth
  path for zero benefit. Comparison of presented-hash vs stored-hash uses **Message Digest is Equal (constant-time)**;
  the equals operation / equals are **forbidden** (timing-oracle). (ADR-003.)
- **Cookies:** mint via the existing Response Cookie helpers in Auth Controller; the refresh cookie reuses the same
  http Only/secure/same Site is Strict builder but with path is auth refresh interface operation and the refresh max Age.
- **Cookie domain — mirror the existing auth cookie (do NOT add a prod domain):** the current set Auth Cookie/
  clear Cookie helpers declare a configured cookie domain with a fallback value but **never call the domain operation** on the
  builder, so the auth cookie is host-only and works on localhost in dev. The new refresh token builder MUST do
  the same — **omit the domain operation** — so dev/localhost keeps working. Do not "helpfully" wire cookie Domain into the
  refresh builder: that would scope the cookie to example com and break every local/dev/virtual machine run.

## Data model (Redis, no database query language schema change)

No new database query language table. Soft-delete principle honoured: revocation is logical (a tombstone/flag in Redis with a expiry period ≥ the
token lifetime), never a destructive “forget the user” — audit rows persist in audit logs.

New Redis Keys entries (no magic strings — added to the application component Keys):

| Key | Value | expiry period | Serves |
|---|---|---|---|
| refresh:family:`<family Id>` | hash the user and role, current and previous token hashes, previous-token grace expiry, creation and absolute-expiry times, and rotation count | refresh idle expiry period (reset on each rotation) | REQ-002/003/004/007 |
| refresh:revoked:`<family Id>` | 1 (reuse tripped / logout / block / reset) | **exactly absolute Expiry Epoch − now seconds** (closes the replay window precisely) | REQ-003/004/005 |

- The cookie carries `<family Id>`.`<opaque Secret>`; the server stores only **SHA-256 secret** of the current secret
  (current Token Hash) so a Redis dump never reveals a usable token. Reuse detection = presented secret hashes (via
  constant-time Message Digest is Equal) to a value that is NEITHER current Token Hash NOR the in-grace prev Token Hash
  for a still-live family ⇒ trip the family-wide revoke.
- **Grace window (concurrency self-Do S fix, REQ-003):** on each rotation the OLD current Token Hash is copied to
  prev Token Hash with prev Expires At Epoch now + jwt refresh.rotation-grace-seconds. A request presenting
  prev Token Hash while now is at most prev Expires At Epoch is treated as a **concurrent duplicate**, not reuse: it returns 200 with no Set-Cookie and no token body, does NOT rotate again, and does NOT revoke. Only a token matching neither
  current nor in-grace-prev trips reuse.
- refresh:family:* expiry period is **re-set on each rotation** (the sliding/idle window). absolute Expiry Epoch is fixed at
  family creation = now + jwt refresh.absolute-ttl-seconds (REQ-007). created At Epoch is fixed at family creation
  (epoch seconds) and is the value compared is at most against pwreset:after:`<id>` (REQ-004 — the opaque token has no signed session token
  iat). absolute Expiry Epoch is checked **before** rotation on every refresh (REQ-007 precedence), so a valid idle
  window never bypasses the hard cap.
- **Revoked-tombstone expiry period = absolute Expiry Epoch − now** (NOT a loose "≥ absolute expiry period"): a freshly-created family
  whose absolute cap is 30d away gets a ~30d tombstone, but a family already 29d old gets only a ~1d tombstone — the
  tombstone outlives exactly as long as any member of that family could still be replayed, and not longer (precise
  replay-window closure, soft-delete semantics preserved).

New config (in application implementation reference, no magic strings):


Implementation example omitted. This public edition retains the written requirements in this section.


## application programming interface / contracts

New endpoint — constant Api Routes AUTH REFRESH auth refresh interface operation (backend) and application programming interface ROUTES auth.refresh (frontend).

**Security Config registration (REQ-002 — code-grounded gap):** auth refresh interface operation is currently **NOT** in
the permit all operation, so it falls through to the any request operation and would **401 before the
controller runs** — fatal, because refresh is called precisely when the access cookie is expired/anonymous. The
endpoint MUST be added to the permit all operation (via Api Routes AUTH REFRESH, POST) so it reaches the controller as an
anonymous request, while staying **OUT of the cross-site request forgery protection ignoring Request Matchers list** (it is a real state-changing POST
and keeps double-submit XSRF — see ADR-002). This is Task 8 (files include Security Config implementation reference).

Error codes are the shared **Auth Error Code** enum (no magic strings):

| Method/Path | Auth | Success | Failure (Auth Error Code) |
|---|---|---|---|
| submit operation for auth refresh | refresh cookie only (permit All; runs anonymous) | 200 + new auth + new refresh token Set-Cookie | 401 REFRESH INVALID (no/garbage cookie, unknown family, admin context, missing creation-epoch), 401 REFRESH EXPIRED (idle or absolute, REQ-007/008), 401 REFRESH REUSED (genuine beyond-grace reuse, REQ-003), 401/403 ACCOUNT BLOCKED (REQ-004), 401 REFRESH INVALID post-reset (REQ-004), 401 post-logout (REQ-005), 429 RATE LIMITED (REQ-011) |

- Error envelope is the existing an error object containing a code and message; code is the code operation. Every
  negative path returns **401** (the frontend treats 401-on-refresh as "go to login") EXCEPT a confirmed block which may
  surface 403 ACCOUNT BLOCKED consistent with the login path, and an over-limit which returns 429 RATE LIMITED;
  no rejection mints an access cookie. Terminal 401/403 auth rejection clears both cookies; temporary 429 throttling sends no Set-Cookie, preserves the family/cookies and includes Retry-After.
- refresh is on the cross-site request forgery protection-protected POST surface; Same Site is Strict + double-submit XSRF both apply (REQ-006 note).

## Concurrency control (REQ-003 — the #1 risk, atomic CAS + grace)

The naïve "any non-current token from a live family ⇒ revoke" rule self-Do Ses a legitimate user when tabs/stampedes
present the same token. Two layers fix it:

1. **Atomic compare-and-swap on rotation.** The read-current-then-write-successor step runs as a **single atomic Redis
   operation** — a **Lua script** (EVAL) over the refresh:family:`<family Id>` hash (preferred — one round-trip, no
   retry loop), or a WATCH/MULTI/EXEC optimistic transaction with bounded retry. The script: re-reads
   current Token Hash; if it still equals the presented hash → writes the new current Token Hash, shifts old →
   prev Token Hash/prev Expires At Epoch, bumps rotation Count, resets idle expiry period, returns the successor. If
   current Token Hash already advanced (another rotation won) → the script does NOT write; the caller falls to the
   grace check. This makes two concurrent rotations impossible to both-win and impossible to tear the record.
2. **Grace window.** If the presented hash matches prev Token Hash and now is at most prev Expires At Epoch
   (jwt refresh.rotation-grace-seconds, default 30), the request is a **concurrent duplicate**: it returns 200 with no Set-Cookie and no token body, preserving the winning response's standing cookies; it does NOT rotate, does NOT
   revoke. Only a hash matching neither current nor in-grace-prev trips reuse.

**Testable invariants (REQ-003):** (a) on a genuine reuse trip the refresh:revoked:`<family Id>` tombstone write
**commits BEFORE** the 401 is returned; (b) the reuse/grace decision happens **BEFORE** any new token is minted (no
successor is issued to a token that should be rejected). The absolute-cap check (REQ-007) runs **before** rotation, so
an expired-but-idle-alive family is rejected without minting.

## Error handling

- All error codes are members of the shared **Auth Error Code** enum (REFRESH INVALID, REFRESH EXPIRED,
  REFRESH REUSED, ACCOUNT BLOCKED, RATE LIMITED); the controller writes the code operation into the
  existing an error object containing a code and message envelope — no bare string literals (Task 4a; mirrored frontend-side as
  AUTH ERROR CODE).
- Terminal authentication rejection paths (401/403, not 429 throttling) clear BOTH cookies (auth Path=/, refresh token Path is auth refresh interface operation) with max Age is 0,
  so a dead/rotated/abused credential is not re-sent.
- Reuse detection writes refresh:revoked:`<family Id>` (tombstone expiry period = absolute Expiry Epoch − now) + an audit row
  REFRESH REUSE DETECTED + increments an abuse metric (auth refresh.reuse detected), so theft is observable, not
  silent (security-expert non-negotiable: "if exploited, will we know?").
- Success writes audit TOKEN REFRESHED. The endpoint is **rate-limited per-IP/per-user** (reuse the existing
  Rate Limit Service pattern) to blunt refresh-grinding; over-limit → 429 Auth Error Code RATE LIMITED, no mint/rotate
  (REQ-011).

## Testing strategy (backend ≥97.5% line+branch is the gate)

| REQ | Test type | Asserts |
|---|---|---|
| REQ-001 | IT + unit | login Set-Cookie Max-Age is 1800; **effective access max Age is 1800 even though the value operation fallback exists** (guards a dropped property from silently shipping 24h — Jwt Token Provider exp AND Auth Controller cookie); expired access token → 401 on /auth/me |
| REQ-002 | IT | valid refresh → 200 + new access + new refresh cookie (happy path); **/auth/refresh is reachable anonymously** (permit All wired) |
| REQ-003 | IT ×2 | R1 used → R2 issued; replay R1 **beyond grace** → 401 REFRESH REUSED + family revoked-before-401; R2 then fails. **Concurrency:** two refreshes with R1 **within grace** → both non-fatal, family intact (no self-Do S) |
| REQ-004 | IT ×3 | blocked-user refresh → rejected, cookies cleared; family-created-at-or-before-reset (created At Epoch is at most epoch) → 401, cookies cleared; **family-epoch-missing → fail-safe reject** (branch cover) |
| REQ-005 | IT | logout clears+revokes; post-logout refresh → 401 |
| REQ-007 | IT | family whose **idle window is alive but absolute cap is past** → 401 REFRESH EXPIRED (proves absolute check fires on an otherwise-valid rotation — absolute-vs-idle split branch) |
| REQ-008 | IT | idle-expired/unknown refresh → 401 |
| REQ-009 | Vitest + Playwright | pure should Attempt Refresh helper unit-tested (one-shot, single-flight, /auth/refresh excluded → no loop); N concurrent 401s → exactly ONE /auth/refresh; silent refresh+retry stays logged-in; refresh-fail → /login |
| REQ-010 | IT | admin login sets NO refresh cookie; admin sliding session intact; refresh-as-admin → **401 REFRESH INVALID** (defined, branch-covered) |
| REQ-011 | IT | over-limit refresh → **429 RATE LIMITED**, no Set-Cookie, family unchanged (rate-limit branch cover) |

The mandated negative paths each have a dedicated IT capturing the rejected status+body:
genuine reuse-detection (REQ-003), expired-refresh idle+absolute (REQ-007/008), post-logout-refresh (REQ-005),
post-reset-refresh (REQ-004), blocked-user-refresh (REQ-004). **Branch-gate watch (≥97.5% BRANCH):** the four ITs most
likely to bite the branch gate are explicitly required — admin-declined refresh (REQ-010), family-epoch-missing
fail-safe (REQ-004), absolute-vs-idle split (REQ-007), and the 429 rate-limit branch (REQ-011) — plus the concurrency
grace branch (REQ-003). Live regression ITs run against the live stack (Abstract Api IT → localhost) —
rebuild/redeploy before the documented operational check reflects the change.

## Migration / rollout

- **No database query language/Liquibase migration** (Redis-only state). Backward-compatible: existing access tokens keep working until
  their (now 30-min) expiry; a logged-in user without a refresh cookie simply re-logs in once, then has the refresh
  cookie going forward.
- **Rollout order:** ship backend (refresh endpoint + config) and frontend (silent-refresh) together; if frontend lags, users just
  re-login at 30 min (the safe fallback) — no breakage.
- **Rollback:** raising jwt expiration-seconds back up and not setting the refresh cookie reverts to prior behaviour
  without code removal; the refresh endpoint is additive.

## Decisions (ADR log)

### ADR-001 — Opaque server-stored refresh token with rotation + reuse-detection (2026-06-20)
- **Context:** We need a longer-lived credential to avoid 30-min logouts, but a long-lived credential is high-value
  theft bait. Options: (a) a long-lived signed session token refresh token (stateless), (b) an opaque random token whose state lives
  server-side in Redis. A stateless signed session token refresh cannot be individually revoked before expiry without a denylist, and
  cannot detect reuse — defeating the whole point.
- **Decision:** Use an **opaque, high-entropy refresh token** backed by a Redis "family" record, with **rotation on
  every use** (issue new, invalidate old) and **reuse detection** (presenting an already-rotated token from a live
  family revokes the entire family). The browser stores only the opaque string in an Http Only/Secure/
  Same Site is Strict cookie scoped to Path is auth refresh interface operation; the server stores only a hash of the current secret.
  Refresh re-checks the SAME kill-switches the access path enforces (blocked:user:`<id>`, pwreset:after:`<id>` — the
  latter compared is at most against the family's stored created At Epoch since the opaque token has no signed session token iat), plus
  an **absolute lifetime cap** independent of rotation (sliding idle window + hard cap). Rotation is **atomic
  (CAS/Lua)** with a short **grace window** for the immediately-previous token, so concurrent tabs/stampedes do not
  self-Do S the user (REQ-003).
- **Consequences:** (+) Individually revocable; theft is detectable (reuse trips a family-wide revoke + metric); a
  Redis dump leaks no usable token (hash only); consistent with the existing Redis revocation backbone; no database query language
  migration. (−) Refresh requires a Redis round-trip (acceptable — every authed request already hits Redis); Redis is
  now on the critical auth path for refresh (already true for admin sessions/denylist). ADMIN is deliberately excluded
  — it keeps its stateful sliding session (REQ-010), so no behaviour change there.

### ADR-002 — Refresh cookie scoped to auth refresh interface operation, Same Site is Strict (2026-06-20)
- **Context:** The refresh cookie should be sent as rarely as possible to shrink its exposure.
- **Decision:** Scope the refresh token cookie to Path is auth refresh interface operation (not /), so it is attached ONLY to
  the refresh call, never to ordinary application programming interface traffic. Keep Same Site is Strict (mitigates cross-site request forgery protection — the refresh POST cannot be
  forged cross-site) and Http Only (no JS exfiltration).
- **Consequences:** (+) Refresh token never rides along on normal requests; minimal cross-site request forgery protection surface (Strict) layered with
  the existing double-submit XSRF on the POST. (−) Path-scoping is a defence-in-depth nicety, not a guarantee
  (cookies aren't an authz boundary) — the server-side family check remains the real control.

### ADR-003 — Refresh-secret entropy + storage: SHA-256 (fast hash) with constant-time compare, NOT bcrypt (2026-06-20)
- **Context:** We store a representation of the opaque refresh secret server-side so a Redis dump never yields a usable
  token. Options for the at-rest representation: (a) a slow password hash (bcrypt/argon2), (b) a fast cryptographic
  hash (SHA-256). And: how much entropy does the secret carry, and how is it compared?
- **Decision:** The secret is **≥256 bits from Secure Random**, base64url-encoded; the cookie value is
  `<family Id>`.`<secret>`. The server stores **SHA-256 secret** only. Comparison is **constant-time**
  (Message Digest is Equal); the equals operation/equals are forbidden. **bcrypt/argon2 are explicitly rejected here.**
- **Rationale:** Slow hashes exist to defend *low-entropy* human-chosen passwords against offline brute-force. This
  secret is ≥256-bit machine-random — it is computationally infeasible to brute-force regardless of hash speed, so a
  slow hash buys zero security while adding ~50–300ms to the critical auth path on every refresh (and the grinding
  surface is already capped by REQ-011 rate-limiting). SHA-256 is the right tool: fast, collision/preimage-resistant,
  and adequate for a high-entropy secret. Constant-time compare removes the timing-oracle on the hash equality check.
- **Consequences:** (+) No password-hash latency on refresh; a Redis dump leaks only SHA-256 digests (useless without
  the preimage); no timing oracle. (−) SHA-256 of a secret must NEVER be reused for *password* storage — that rule is
  unchanged (passwords stay on the b crypt password encoder operation, Security Config password Encoder); this fast-hash decision
  is scoped strictly to the high-entropy refresh secret.

## 2026-09-05 selected grace and retry branches

The Redis CAS stores hashes only. Winner response sets R2/access; an in-grace previous-hash match returns plain 200 with no Set-Cookie or token body and no extra rotation. Do not attempt to reissue a raw secret from a hash or add a recoverable secret cache. The client shares one refresh promise, replays each protected request once, and never loops on a second 401. If the winner response was lost, R2 cannot be recovered; ask for login after the bounded retry instead of claiming transparent recovery. A temporary 429 RATE LIMITED is a distinct branch: preserve family/cookies/auth store, return Retry-After, show retry guidance and wait for a later deliberate retry. Only terminal 401/403 rejection clears both cookies and redirects. Verify header absence as well as status codes.
