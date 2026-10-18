# 05 — cross-site request forgery protection Bootstrap + Auth Hydration on App Boot

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Bootstrap the readable cross-site request forgery protection token before protected writes; never expose the HttpOnly auth token to JavaScript.
2. Anonymous session hydration is a valid state. For an expired recoverable user session, follow004’s single-flight refresh and bounded retry instead of treating every initial401 as final logout.
3. A refresh429 keeps both cookies. A protected write still needs the cross-site request forgery protection header after successful hydration.

## Authority

[Current requirements C09](../CURRENT-REQUIREMENTS.md) · [requirements md](../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 05 — cross-site request forgery protection Bootstrap + Auth Hydration on App Boot

## Trigger

Root Layout mounts and invokes the use init auth operation. This runs **once** per session
to (1) seed the XSRF-TOKEN cookie before any state-changing POST and (2)
hydrate auth Store from /me.

## Participants

- **App / Root Layout** — React component tree
- **use Init Auth implementation reference** — bootstrap hook
- **axios api** — with Credentials: yes, xsrf Cookie Name: REDACTED credential,
  xsrf Header Name: X-XSRF-TOKEN
- **Auth Controller** — read operation for auth csrf, read operation for auth me
- **Auth Service** — the get me operation
- **Encryption Service**
- **Security Filter Chain** — Csrf Cookie Filter, Cookie Csrf Token Repository,
  Jwt Authentication Filter

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- frontend: use Init Auth implementation reference (full file is the canonical
  reference). The /csrf → /me ordering is intentional — the comment on
  line 14-25 explains why: /me's 401 path triggers redirect to /login, so
  the XSRF cookie must already be present before that redirect's login POST.
- axios config: api implementation reference. Names match Spring
  Security defaults so no extra config is needed.
- backend controller: Auth Controller implementation reference — returns 204 No Content; the
  cookie is set by the filter chain, not the controller body.
- cross-site request forgery protection repository: Security Config implementation reference —
  the with http only no operation with
  the secure operation.
- Csrf Cookie Filter forces token materialisation on every response that gets
  past the cross-site request forgery protection filter (Csrf Cookie Filter implementation reference).
- auth csrf interface operation is in the the permit all operation list AND not in the cross-site request forgery protection
  ignoring Request Matchers set — but it's a GET, so cross-site request forgery protection check does not apply.

## Failure modes

- /csrf failure is ignored — handling a failed request without leaving an unhandled error (line 26). The next
  request would still get a fresh Csrf Token via the filter; only a true network
  outage would leave the SPA without a token, in which case /me would also fail
  and the user lands on /login.
- /me 401 → axios interceptor calls the logout operation and dispatches the
  custom event so router can redirect with return To. Anonymous users on
  public pages just see the redirect skipped (frontend checks pathname).

## Related flows

- [25-csrf-protected-write md](25-csrf-protected-write.md) — what the bootstrap enables
- [03-login-with-mfa md](03-login-with-mfa.md) — login is the typical destination of the 401 redirect
