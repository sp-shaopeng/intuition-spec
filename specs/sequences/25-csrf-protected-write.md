# 25 — cross-site request forgery protection-Protected Write (double-submit cookie)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Trigger

Any state-changing frontend request (POST, PUT, PATCH, DELETE) to a non-ignored
endpoint. The pattern is the same for every such call — this diagram uses the
illustrative example of submit operation for tutee posts.

## Participants

- **frontend** (axios with xsrf Cookie Name / xsrf Header Name configured)
- **Browser** — owns the XSRF-TOKEN cookie (Http Only is no, Secure,
  Same Site is Strict)
- **Security Filter Chain**
  - Cookie Csrf Token Repository — backed by the XSRF-TOKEN cookie
  - Csrf Filter — compares cookie vs X-XSRF-TOKEN header
  - Csrf Cookie Filter — refreshes the cookie on the response
- **Controller / Service**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- axios config: api implementation reference. The names
  XSRF-TOKEN / X-XSRF-TOKEN are axios defaults and match Spring's defaults
  — no further wiring needed.
- cross-site request forgery protection repository + handler config: Security Config implementation reference.
  the with http only no operation means the cookie is readable by JS so axios can echo it.
  the set csrf request attribute name operation opts out of attribute-name resolution (SPA
  uses the header directly).
- Csrf Cookie Filter (Csrf Cookie Filter implementation reference) is registered
  the add filter after operation — runs late enough that the
  cookie reflects post-auth state.
- Endpoints in the ignoring Request Matchers set bypass the cross-site request forgery protection check
  (Security Config implementation reference): login, login/mfa, register/**,
  forgot-password, reset-password, ** interface operation, /actuator/**.
  Every other write requires the header.

## Failure modes

- 403 Invalid cross-site request forgery protection token — header missing, header malformed, or header value
  does not match cookie value. Common causes: SPA POST happens before flow 05
  completes (mitigated by the set loading operation in use Init Auth), third-party
  embed without the cookie, malicious cross-origin POST.
- A Same Site is Strict cookie is not sent on cross-origin or top-level
  navigation from another site, providing defence-in-depth against cross-site request forgery protection.

## Related flows

- [05-csrf-bootstrap md](05-csrf-bootstrap.md) — how the cookie gets there in the first place
- Every other write flow (06–22)
