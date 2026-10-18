> **Current-file check, 18 October 2026:** The ordinary renewal path is present, but a preceding authentication filter may clear a cookie before a 429 response; the combined path still needs an integration check (G10). See the [alignment review](../../docs/current-file-alignment.md). The requirements remain unchanged.

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 004
slug: auth-session-hardening-30min-ttl-and-refresh
title: Auth session hardening 30min expiry period and refresh
version: 1.5
status: HISTORICALLY MARKED IMPLEMENTED            # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: security-expert         # the engineer who owns this change (requirements-engineer for cross-cutting/product specs)
created: 2026-06-20
updated: 2026-09-05
---

> **2026-09-05 requirements correction (documentation only).** 429 throttling preserves the session. In-grace duplicates use plain 200/no Set-Cookie with hash-only storage and an explicit bounded lost-response re-login fallback.
> The status and shipment/evidence entries below record the earlier implementation. This revision does not
> certify that the corrected contract is implemented, tested, or deployed; follow-up verification remains open.


# Auth session hardening 30min expiry period and refresh

Hardens TUTOR/TUTEE session security (issue **INT-013** / GH #33) by closing the two remaining gaps after the
already-done HttpOnly/Secure/Same Site is Strict/logout-clears work: (1) a **30-minute access-token expiry period** for all roles
(today regular users were effectively 12h), and (2) a **sliding refresh-token flow** — an opaque, server-stored
refresh token (HttpOnly, Same Site is Strict, path-scoped to auth refresh interface operation) with **atomic** rotation-on-use +
reuse detection **plus a short grace window** (so concurrent tabs/401-stampedes don't self-Do S the user), an idle
window plus an absolute lifetime cap (Must), consistent with the existing Redis kill-switches (jti denylist,
blocked:user, pwreset:after) so refresh respects block/reset/logout, and a per-IP/user rate-limit on the endpoint.
The frontend **single-flights** the silent refresh on a 401 and retries once (refresh runs before the logout branch, a 401
from refresh itself does not recurse), else redirects to login — so active users are NOT logged out every 30 minutes
while the replay window of a stolen token stays short. ADMIN stateful sessions are unchanged.

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC (each REQ-NNN)
- **Design:** [design md](design.md) — architecture, data model, error handling, testing, migration
- **Tasks:** [tasks md](tasks.md) — implementation tasks, each traced to a REQ-NNN + owning specialist
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario

## Status

APPROVED — scoped spec-review gate passed after one SEND-BACK revision cycle (v1.1, 2026-06-20). Cleared for build.

## Review & Sign-Off

- [x] Author / Counterpart — security (security-expert, lead) — SIGN-OFF (conditions applied v1.1) / 2026-06-20
- [x] Counterpart — session design (solution-architect) — SIGN-OFF (CAS+grace, idle/absolute precedence v1.1) / 2026-06-20
- [x] Counterpart — backend impl (software-engineer) — SIGN-OFF (permit All matcher, Redis Keys, fallback assert v1.1) / 2026-06-20
- [x] Counterpart — frontend refresh handling (frontend-expert) — SIGN-OFF (single-flight, no-recursion, pure helper v1.1) / 2026-06-20
- [x] Technical Writer (docs) — SIGN-OFF / 2026-06-20
- [x] Boss (Rex) — APPROVED / 2026-06-20

## Changelog

- 1.4 (2026-06-21) **RE-HISTORICALLY MARKED IMPLEMENTED — REQ-005 fix verified live.** MODIFIED REQ-005 — logout AND delete-account now
  revoke ALL refresh families for the authenticated the user id operation via a new refresh:user-families:<uid>
  Redis index (maintained on mint + every revoke path incl. the Lua reuse-revoke), eliminating the cookie-dependent
  the revoke family operation no-op. The Refresh Session IT was hardened to send logout WITHOUT the refresh cookie
  (reproducing the live Sev-1). Also fixed the REQ-011 429 retry After mislabel (3600→60). Gate re-green (BUILD
  SUCCESS, ≥97.5% line+branch). **Re-proven LIVE** (private logout and account-deletion evidence grouped under requirement 005): logout/delete →
  replay refresh **401 REFRESH INVALID**, Redis family revoked using tombstone is 1+gone using family is 0+index cleared; regression
  REQ-002/003/011 all re-confirmed PASS. Status IN-PROGRESS→HISTORICALLY MARKED IMPLEMENTED.
- 1.3 (2026-06-21) **SEND-BACK (Boss live-audit) — REQ-005 logout FAIL, status HISTORICALLY MARKED IMPLEMENTED→IN-PROGRESS.** The Boss's
  live convergence audit caught that logout/delete-account do NOT revoke the refresh family server-side: they read
  the path-scoped refresh token cookie (Path is auth refresh interface operation) which the browser never sends to
  /auth/logout, so the revoke family operation no-ops — an exfiltrated/logged-out refresh token keeps minting access
  tokens. Live repro in evidence/REQ-005/logout txt. (Process note: the 1.2 SHIP verdict was issued on a partial
  evidence audit before REQ-005 was opened — a Boss miss; corrected here.) Fix in progress: revoke ALL families for
  the authenticated the user id operation (user→families Redis index), not via the cookie. Status returns to HISTORICALLY MARKED IMPLEMENTED
  once REQ-005 passes live.
- 1.2 (2026-06-21) HISTORICALLY MARKED IMPLEMENTED — built (backend+frontend), gated, evidence-complete, security-reviewed, team-reviewed → Boss SHIP.
  backend (no migration, Redis-only): 30-min access expiry period + cookie Max-Age is 1800; Refresh Token Service (≥256-bit secret,
  SHA-256-only storage, atomic CAS Lua rotation, reuse-detection→family-revoke with tombstone-before-401, grace
  window, absolute-before-rotation, kill-switch re-checks, host-only refresh cookie, admin-skip, 429 rate-limit);
  submit operation for auth refresh (permit All, cross-site request forgery protection retained); Auth Error Code. Gate BUILD SUCCESS, 98.78% line / 97.51% branch
  (Boss-reverified). frontend: pure should Attempt Refresh helper + single-flight silent-refresh (N concurrent 401s → ONE
  refresh, no recursion, probe-401 no loop); lint+build+test:ci 2430 tests, 98.15% line / 92.64% branch. Backward-
  compatible (purely additive). Evidence under evidence implementation reference — live curl: Max-Age is 1800/exp−iat is 1800, refresh 200+2
  Set-Cookie, reuse→401 REFRESH REUSED+family revoked (Redis revoked is 1/family is 0), post-logout/post-reset 401,
  blocked 403, 429 RATE LIMITED (no Set-Cookie), admin no-refresh-cookie; silent-refresh Playwright @1280/@390;
  manual security-expert review (REQ-003/backend-security-review md, 2 accepted residuals). Closes #33.
  Follow-up (founder ruling): ADMIN 30-min no-refresh session friction.
- 1.1 (2026-06-20) spec-review SEND-BACK revision (security-expert). MODIFIED REQ-003 — concurrency self-Do S fix:
  atomic CAS rotation (Lua/WATCH) + rotation grace window (backend) with testable invariants (tombstone-before-401,
  reuse-check-before-mint). MODIFIED REQ-009 — frontend single-flight refresh, interceptor runs before should Trigger Logout,
  /auth/refresh excluded (no recursion), decision moved to a pure exported should Attempt Refresh helper for the
  coverage gate. MODIFIED REQ-004 — opaque-token epoch semantics: family created At Epoch compared is at most against
  pwreset:after:<id>; epoch-missing fail-safe. MODIFIED REQ-007 Should→**Must** + idle-vs-absolute precedence
  (absolute check runs even on a valid rotation). ADDED REQ-011 — per-IP/user rate-limit → 429 RATE LIMITED
  (refresh-grinding). MODIFIED REQ-010 — admin refresh returns a DEFINED 401 REFRESH INVALID. Design: named the hash
  (SHA-256, constant-time compare, bcrypt forbidden — ADR-003), pinned ≥256-bit Secure Random entropy + <family Id>.<secret>
  wire format, refresh:revoked tombstone expiry period = absolute Expiry Epoch − now, refresh cookie omits the domain operation,
  shared Auth Error Code enum (no magic strings). Tasks: ADDED Security Config the permit all operation (kept out of
  cross-site request forgery protection-ignore) to Task 8, ADDED Task 4a (Auth Error Code) + Task 14a (single-flight), Task 1 asserts effective access
  max Age is 1800 (guards the 86400 fallbacks), Task 12 adds branch-covering ITs (admin-declined, epoch-missing,
  absolute-vs-idle, 429, grace). status stays DRAFT — Boss re-gates.
- 1.0 (2026-06-20) initial draft — ADDED REQ-001…REQ-010 (30-min access expiry period + sliding refresh w/ rotation,
  reuse-detection, kill-switch consistency, frontend silent-refresh, admin no-regression). security-expert-authored
  with solution-architect + software-engineer input. status DRAFT pending team-review.
