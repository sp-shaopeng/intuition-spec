# Backend Correctness & Security Hardening Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Owners:** software-engineer (Jordan) · security-expert (Casey) · dba
**Status:** Proposed — doc only (no code, no migration applied).
**Date:** 2026-06-05
**Scope:** Seven defects/hardening items found in the live backend. Two are pre-prod blocking
(INT-067 silent feature outage; unbounded page size Do S lever); five are security/correctness
hardening. Every item is grounded in file:line against the current source.

> Convention reminders carried into every fix below: use Java 21 for backend builds;
> apply the agreed formatting rules and verify at least 97.5% line and branch coverage over the measured scope, with the required database services available;
> no magic strings (constants / typed enums both ends); Liquibase-only for schema (one ledger,
> ALTER TYPE ADD VALUE in its own run In Transaction is no changeset committed before code
> references the label); regression ITs run against the **live** stack (Abstract Api IT →
> localhost) so rebuild and reseed before running verification against a corrected application.

---

## Severity ranking (highest first)

| Rank | Item | Severity | Why |
|---|---|---|---|
| 1 | **INT-067** — interview expiry sweep throws type interviewstatus does not exist every run | **S1 / pre-prod blocking** | A whole feature (INT-039 negotiation expiry) silently never runs; stale PROPOSED/COUNTER PROPOSED interviews accumulate forever; interview expired.errors ticks every 15 min. Functional outage masquerading as healthy. |
| 2 | **Unbounded page size** — no spring data.web pageable.max-page-size | **S2 / pre-prod blocking** | ?size is 100000 on any paginated list endpoint = single-request heap pressure / Do S lever. One-line fix. |
| 3 | **Rate-limit IP spoofing** — Auth Controller client Ip trusts CF-Connecting-IP/X-Forwarded-For unconditionally | **S2 / security** | Attacker forges the header per request → bypasses register / forgot-password throttle → email flooding, one-time passcode-cost abuse. Login partly account-scoped; lockout is a backstop, not a fix. |
| 4 | **Password reset does not revoke tutor/tutee JWTs** (Auth Service reset Password value (withheld)) | **S2 / security** | A stolen access token survives a victim password reset for up to its full ~30-min lifetime — the exact window a "change my password because I think I'm compromised" action exists to close. |
| 5 | **signed session token has no iss/aud** (Jwt Token Provider, parse 69-71) | **S3 / security hardening** | Tokens are not bound to issuer/audience; widens token-confusion / cross-environment replay surface. Defense-in-depth. |
| 6 | **cross-site request forgery protection ignore glob /public/**** (Security Config) | **S3 / security hardening** | Broader than the actual unauthenticated bootstrap POST surface; any future state-changing /public/** route silently ships cross-site request forgery protection-exempt. |
| 7 | **scheduled jobs lack coordination between application instances** (Interview Service, Showcase Service) | **S3 / correctness-at-scale** | On >1 replica every scheduled sweep/reroll double-runs. Today single-replica → latent; becomes real on horizontal scale. |

---

## Item 1 — INT-067: interview expiry sweep errors every run (S1, pre-prod blocking)

### Problem
The INT-039 inactivity sweep is wrapped in a try/catch that increments
interview expired.errors and rethrows; it has thrown
org postgresql.util PSQL Exception: ERROR: type interviewstatus does not exist on **every**
fire (default every 15 min, interview expiry.sweep-ms). The sweep therefore never expires
a single stale negotiation — a silent functional outage. The error counter exists but nobody is
alerting on it.

### Current state (file:line)
- The historical interview sweep ran at a configured interval inside a database transaction. It first selected expiry candidates, then marked stale negotiations as expired.
- Queries: Interview Repository implementation reference. Both are JPQL referencing fully-qualified
  enum constants (the application component Status PROPOSED, .COUNTER PROPOSED) and write
  i status the application component Status EXPIRED.
- Interview status was mapped as a required named value, with the application mapping explicitly tied to the database interview-status type.
- PG type: 0011-create-interviews implementation reference —
  Allowed values for interview status: ...; negotiation labels added in
  0034-interview-timeslot-negotiation implementation reference.
- Dialect: application implementation reference — Postgre database query language Dialect.

### Root cause
The PostgreSQL enum **type** is interview status (snake case). But when Hibernate renders a
JPQL predicate/assignment comparing an Enum Type STRING column that is declared with a named
column Definition against enum literals, the dialect emits a database query language cast to a type derived from the
**Java enum's simple name** — Interview Status, which PostgreSQL folds to the unquoted identifier
interviewstatus. No such type exists (the real one is interview status), so the statement fails
to parse → type interviewstatus does not exist. The mismatch is purely *type-name* (Java
Interview Status vs PG interview status), not value-name; that is why no row is ever touched and
the error is identical on every run.

### Fix direction (pick one; (a) is preferred)
- **(a) Make Hibernate stop casting to the Java type name — store the enum as plain text / varchar
  and drop the named column Definition, OR register the column's PG type name explicitly so the
  emitted cast is interview status.** The clean, durable fix is to ensure the *runtime database query language cast
  matches the actual PG type name*. Two concrete options:
  - Convert the predicate to **native database query language** with an explicit, correct cast (status = CAST(:label
    AS interview status)/status IN (...)::interview status) for the two sweep queries only —
    smallest blast radius, keeps the enum column as a true PG enum. Bind values as the **string
    label** (PROPOSED), not the Java enum.
  - OR map the column as a Hibernate Postgre database query language Enum Jdbc Type / custom @Jdbc Type Code so the
    generated cast uses the registered PG type name interview status everywhere (also fixes any
    other JPQL touching this column, not just the sweep).
- **(b) Tactical (only if (a) is deferred):** rewrite both sweep queries as @Query(native Query =
  true)using the **text labels** and the correct::interview status cast, and bind primitives.
- Apply the **same audit** to the sibling enum columns mapped the same way:
  decline reason (column Definition interview decline reason, Interview implementation reference) and
  last actor (column Definition timeslot proposer, Interview implementation reference), and to
  Interview Timeslot slot Status (interview timeslot status) used by
  withdraw Open Slots For Interviews (Interview Repository implementation reference) — any JPQL bulk
  update records/predicate against those columns has the identical latent defect.

### Acceptance criteria
- The scheduled sweep runs to completion with **zero** database query language errors; interview expired.errors
  stays flat (0 increments) across multiple fires.
- A PROPOSED/COUNTER PROPOSED interview whose updated at now - EXPIRY WINDOW is flipped to
  EXPIRED, its open timeslots flipped to WITHDRAWN, interview expired counter incremented,
  and both parties notified (INTERVIEW EXPIRED).
- Re-running the sweep with no new stale rows is a no-op (status-guarded; counter unchanged) —
  idempotency preserved.

### Migration / test notes
- If (a)-Postgre database query language Enum Jdbc Type is chosen there is likely **no migration** (mapping-only change);
  verify boot + full read/write of every interview status value against a real Testcontainers PG.
- **Reproduce before patching** (Sev-1 rule): seed a stale PROPOSED interview, fire the sweep
  against the live stack, capture the actual PSQL Exception from the *real* log, then fix.
- Add an IT (Interview Expiry Sweep IT or extend the negotiation IT) that: seeds a stale negotiation
  → invokes the sweep → asserts status EXPIRED + slots WITHDRAWN + notifications emitted + counter
  incremented + a second invocation is a no-op. ITs hit the **live** stack, so rebuild+reseed first.
- Unit-cover the repository methods against Testcontainers PG (not H2 — H2 has no PG enum types and
  would hide this exact bug).

---

## Item 2 — Unbounded page size (S2, pre-prod blocking)

### Problem
No spring data.web pageable.max-page-size is configured anywhere in
application* implementation reference. Spring Data's default cap is 2000, but several list endpoints accept a
Pageable directly; a request like ?size is 100000 (or repeated large-page scans) is a cheap
single-request heap-pressure / Do S lever and an N-row materialisation amplifier on the hottest
public browse endpoints.

### Current state
- grep for max-page-size / default-page-size / Pageable Handler Method Argument Resolver across
  application implementation reference, application-local implementation reference, application-prod implementation reference,
  application-debug-sql implementation reference → **no match**. No global resolver customisation.

### Fix direction (one line + clamp)
- Set in application implementation reference (so all profiles inherit):
  - spring data.web pageable.max-page-size is 100 (tune per endpoint need; pick the largest a user interface
    legitimately requests).
  - spring data.web pageable.default-page-size is 20 (make the default explicit).
- Spring Data **clamps** (does not 400) size to max-page-size automatically once set — confirm
  the clamp empirically rather than trusting the property name.
- Audit any endpoint that constructs pagination directly from an unchecked request parameter (not
  via the argument resolver) — those bypass the global cap and need an explicit
  a shared maximum-size limit, reducing larger requests to that limit.

### Acceptance criteria
- GET `<any paginated list>`?size is 100000 returns at most max-page-size rows (clamped), not a
  100k-row payload, and does not OOM.
- Default page size is the configured value when size is omitted.

### Test notes
- IT against the live stack: hit a public browse endpoint with ?size is 100000, assert
  content length is at most 100 and pageable page Size equals 100. Add a unit assertion on the configured
  resolver if a manual Page Request path exists.

---

## Item 3 — Rate-limit IP spoofing (S2, security)

### Problem
Auth Controller client Ip (Auth Controller implementation reference) reads CF-Connecting-IP,
then X-Forwarded-For (first hop), with **no trusted-proxy allowlist** and no verification that
the request actually arrived through Cloudflare. Any direct caller can set
CF-Connecting-IP: `<random>` per request. That IP is the rate-limit bucket key for registration
and forgot-password:
- Auth Service register Tutee/register Tutor → the register key operation (pure-IP
  keyed) — Auth Service implementation reference.
- Auth Service forgot Password → the forgot password key operation — Auth Service implementation reference.
- Auth Service login → the login key operation (partly **account-scoped** by email) —
  Auth Service implementation reference.

So a spoofed header lets an attacker flood registrations and forgot-password one-time passcode emails (cost +
inbox abuse of arbitrary victims) while sailing past the per-IP throttle. Login is partially
protected because the bucket also keys on email, and the per-account lockout (recorded in a
REQUIRES NEW transaction, Auth Service implementation reference) is the backstop — but that protects the
*account*, not the *email-sending / signup-spam* surface.

### Fix direction
- Treat CF-Connecting-IP / X-Forwarded-For as **trusted only when the request came from a known
  proxy**. Configure a **trusted-proxy allowlist** (Cloudflare's published IP ranges and/or the
  internal LB CIDR). If the get remote addr operation is NOT in the trusted set, **ignore** the
  forwarded headers and use the get remote addr operation as the client IP.
- Prefer doing this once at the edge of the app via Spring's Forwarded Header Filter +
  server forward-headers-strategy constrained to trusted proxies, rather than hand-parsing in a
  controller. Keep a single shared client Ip resolver — no per-controller duplication.
- Add a **second, account-independent guardrail** on email-sending endpoints (global
  forgot-password / registration send-rate ceiling) so a distributed-IP attack still can't fan out
  unbounded one-time passcode emails.
- Alert on bursts of register / forgot-password 429s and on auth-failure spikes (credential-stuffing
  / spam signal) — per SHARED-LESSONS A01 rate-limit guidance.

### Acceptance criteria
- A request arriving from a non-trusted remote address with a forged CF-Connecting-IP is bucketed
  by its **real** remote Addr, so per-IP limits cannot be bypassed by header rotation.
- A request from a trusted proxy still resolves the true client IP from the forwarded header.
- Forgot-password / registration email fan-out has an account-independent ceiling.

### Test notes
- Unit-test the IP resolver: trusted-proxy remote + forged header → forwarded value used;
  untrusted remote + forged header → remote Addr used.
- IT: loop forgot-password with a rotating CF-Connecting-IP from an untrusted source → must start
  429-ing at the configured limit (proves the bucket no longer follows the spoofed header).

---

## Item 4 — Password reset does not revoke tutor/tutee JWTs (S2, security)

### Problem
Auth Service reset Password (Auth Service implementation reference) updates the password hash and
clears the change-required gate, but performs **no token/session revocation for TUTOR/TUTEE**. For
ADMIN it only logs "active Redis sessions will expire naturally" (557-561) — i e. it does not
revoke admin sessions either. Tutor/tutee are stateless signed session token (no per-request kill-switch other than
the INT-065 block-flag), so a token stolen before the reset remains valid until its ~30-min
expiry. A user resetting their password specifically because they fear compromise does not actually
cut off the attacker.

### Current state (file:line)
- reset Password body: Auth Service implementation reference — no jti revoke, no session purge, no
  per-user token epoch bump.
- Logout-style revocation already exists for admin: logout writes jti:`<jti>` deny + deletes
  session:admin:`<id>` (Auth Service implementation reference); the admin session index
  sessions:by-user:`<user Id>` (Auth Service implementation reference) already supports purging ALL of a
  user's live admin sessions. The INT-065 block-flag (blocked:user:`<id>`, checked in the auth
  filter for all roles) is the existing all-roles per-request kill-switch primitive.

### Fix direction
- On successful reset, **revoke all of the user's outstanding access tokens**, for every role:
  - **TUTOR/TUTEE (stateless):** introduce a **per-user token epoch / tokens valid after
    timestamp** (Redis pwreset:after:`<user Id>` = reset instant, expiry period ≥ max token lifetime). The
    auth filter rejects any token whose iat (issued At) is before that instant — reusing the
    INT-065 pattern (check before setting the principal, expire the cookie max Age is 0). This is the
    stateless equivalent of "log out everywhere."
  - **ADMIN:** purge sessions:by-user:`<user Id>` (delete every session:admin:* it indexes) so the
    reset actually ends live admin sessions instead of waiting them out.
- Do the revocation **after commit** (same after Commit hook already used at 564) so a rolled-
  back reset doesn't strand sessions.
- Emit an audit event (already does PASSWORD RESET) and a notification "all sessions signed out."

### Acceptance criteria
- After a successful reset, a previously-issued tutor/tutee token is rejected (401/redirect to
  login) on its next request — verified with a token minted *before* the reset.
- Admin live sessions are purged at reset time (not left to expiry period).
- A token issued *after* the reset is accepted normally.

### Test / data notes
- Reuse the INT-065 filter pattern + tests: assert the rejected pre-reset token never reaches the
  jti/session has Key check and the response sets the auth cookie max Age is 0.
- IT against live stack: login → capture token → reset password → old token 401, new login works.

---

## Item 5 — signed session token has no iss / aud claims (S3, hardening)

### Problem
Jwt Token Provider generate Token (Jwt Token Provider implementation reference) sets sub, role,
session Id, jti, iat, exp, signs HS256 — but **no iss (issuer) and no aud (audience)**.
Parsing (parse With, 69-71) only verifies the signature; it does not require iss/aud. This
widens token-confusion / cross-environment replay surface (e g. a token minted for a sibling
service or a dev environment sharing a secret would be accepted) and misses an OWASP-recommended
signed session token validation step (validate signature **+ iss + aud + exp**).

### Fix direction
- Include the configured issuer and intended audience when creating a signed session token. The issuer identifies who created it; the audience identifies the service it is intended for.
- When validating the token, require both values in addition to a valid signature and expiry. Reject missing or mismatched values.
- Coordinate rollout with the existing dual-secret rotation (secret Current/secret Previous,
  validate And Parse Claims): both secrets must be parsed with the same iss/aud
  requirement; deploy mint-with-claims, allow a grace window where parse does **not** yet require
  them, then flip parse to require — otherwise in-flight pre-upgrade tokens 401 en masse.

### Acceptance criteria
- New tokens carry iss and aud.
- A token with a missing/incorrect iss or aud is rejected on parse (after the grace flip).
- Rotation (current↔previous secret) still works with the claims required.

### Test notes
- Unit: generate → parse round-trips; tamper iss/aud → Jwt Exception; missing claim →
  rejected. Cover the previous-secret branch with claims required.

---

## Item 6 — cross-site request forgery protection ignore glob /public/** too broad (S3, hardening)

### Problem
Security Config (Security Config implementation reference) disables cross-site request forgery protection for the pre-session
bootstrap POSTs (login, login/mfa, register/**, forgot-password, reset-password) **and** for the
broad glob ** interface operation (147). The public surface is overwhelmingly GET reads (which
don't need cross-site request forgery protection), but the glob means any future state-changing /public/** POST silently ships
cross-site request forgery protection-exempt. The exemption should match the **exact** unauthenticated bootstrap POST surface, not a
wildcard read namespace.

### Fix direction
- Narrow the cross-site request forgery protection ignoring Request Matchers to the exact bootstrap POST paths already listed
  (auth login interface operation, /login/mfa, /register/**, /forgot-password, /reset-password) plus
  /actuator/** (Prometheus basic-auth).
- **Remove the ** interface operation glob from the cross-site request forgery protection ignore list.** Public GETs are cross-site request forgery protection-irrelevant
  (cross-site request forgery protection protects state-changing methods); if a specific public POST genuinely needs exemption, list
  that exact path, not the namespace. Keep cross-site request forgery protection protection ON globally (SHARED-LESSONS: never
  blanket-disable; SPA uses Synchronizer Token + with XSRF Token per INT-031).

### Acceptance criteria
- Public GET reads continue to work unauthenticated (no cross-site request forgery protection token required for GET regardless).
- A hypothetical state-changing POST under /public/** is **not** auto-exempt — it requires a cross-site request forgery protection
  token unless explicitly and individually listed.
- The five named bootstrap POSTs still succeed without a prior cross-site request forgery protection token.

### Test notes
- IT: public GET → 200; the named bootstrap POSTs → 200 with no cross-site request forgery protection token; a mutating request
  outside the exact allowlist without a token → 403.

---

## Item 7 — Coordinate scheduled work across application instances

### Problem
The historical design ran two scheduled jobs independently on every application instance:
- The interview-expiry sweep, running at its configured interval.
- The public-preview refresh, running at its configured interval.

There is **no Shed Lock dependency** in pom implementation reference and no Lock Provider. Today the system runs
single-replica so this is latent, but the moment it scales horizontally, every replica fires the
sweep/reroll simultaneously: redundant database load, duplicate metric increments
(interview expired over-counts), and races on the showcase pool reroll. The expiry sweep's
set-based status-guarded update records is *idempotent* so it won't double-flip rows, but it will still
emit the counter and notifications from each replica.

### Fix direction
- Add **Shed Lock** (net javacrumbs.shedlock) with a **Redis** lock provider (Redis is already a
  dependency / session store) — or a JDBC shedlock table via Liquibase if a database lock is preferred.
- Annotate both scheduled methods with @Scheduler Lock(name is ..., lock At Most For is ..., lock At Least For=
  ...)and enable@Enable Scheduler Lock lock At Most For must exceed the worst-case job runtime so
  a crashed holder eventually releases.
- Keep the sweep idempotent regardless (defence in depth) — leader election removes the duplicate
  *notifications/metrics*, idempotency protects the *data*.

### Acceptance criteria
- With N>1 replicas, each scheduled tick fires the job body on exactly **one** replica per interval;
  interview expired counter and INTERVIEW EXPIRED notifications are emitted once per expiring
  interview, not N times.
- A replica crash mid-lock releases the lock within lock At Most For and the next tick proceeds.

### Migration / test notes
- If a JDBC lock store is chosen, add the shedlock table via a Liquibase changeset (next free
  number, leave reserved gaps). Redis provider needs no migration.
- IT/integration: simulate two scheduler beans / two contexts contending the same lock window →
  assert the job body executes once. At minimum, unit-assert the @Scheduler Lock config + name +
  lock At Most For bounds.

---

## Cross-cutting notes
- **Reproduce S1/S2 before patching** — capture the real PSQL Exception (Item 1) and the real
  oversized payload / spoof-bypass behaviour (Items 2,3) from the live stack before touching code;
  classify each as code vs config vs infra by where it throws.
- **Testcontainers, not H2** — Items 1 (PG enum casts) and 2 (clamp behaviour) must be verified
  against real PostgreSQL; H2 would hide the enum-type bug entirely.
- **No magic strings** — issuer/audience (Item 5), trusted-proxy CIDRs (Item 3), max-page-size
  constant for manual Page Request clamps (Item 2), and Shed Lock lock names (Item 7) all via
  config/constants.
- **Soft-delete only** unchanged — none of these introduce a physical delete.

---

→ **team-review → Boss → build.**
