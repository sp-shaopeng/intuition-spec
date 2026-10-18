# 40 — Public Showcase Strips (Active Tutors / Recent Requests)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Serve bounded cached personal information-free showcase envelopes, with public handle/humanized public identity and the retained eligibility/consent rules.
2. Honor the 027 trusted-proxy boundary when deriving client IP; no unconditional trust of CF-Connecting-IP or X-Forwarded-For from arbitrary callers.
3. See-all links lead to bounded guest-filterable browse. Detail login remains separate. Degraded caches use the specified fallback envelope rather than leaking private or unrestricted data.

## Authority

[Current requirements C01](../CURRENT-REQUIREMENTS.md) · [m17-marketplace-ux-overhaul md](../architecture/m17-marketplace-ux-overhaul.md) · [spec md](../027-aws-production-deployment/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 40 — Public Showcase Strips (Active Tutors / Recent Requests)

## Trigger

An anonymous visitor lands on the pre-login marketing surface. Two personal information-free strips render:
**Active Tutors** (M14a) and **Recent Job Requests** (M14b). Both are public, GET-only,
cache-served, and rate-limited.

## Participants

- **Anonymous visitor**
- **frontend** landing showcase strips
- **Showcase Controller** — read operation for showcase tutors, read operation for showcase requests
- **Rate Limit Service** (per-IP), **Showcase Service** (Caffeine/Redis cache — never the database on read)

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Showcase Controller implementation reference. Both endpoints per-IP rate-limited
  (ratelimit showcase.limit is 60/60s) and set Cache-Control: public, max-age is 60.
- Reads hit **Caffeine/Redis only, never the database** (24-25).
- personal information-free by contract: tutors expose handle only (no real name — guarded by a
  reflection personal information-contract test); requests are anonymised (no title/contact/exact date; bucketed
  budget; planning-area location).
- These are the cached *strips*; the full enumerable browse is flow 23 (/public/tutors) and the
  public jobs browse (/public/assignments).
- Client IP precedence: CF-Connecting-IP → X-Forwarded-For 0 → the get remote addr operation.

## Failure modes

- 429 RATE LIMITED — per-IP showcase limit exceeded.
- Stale-but-served: a cache miss is repopulated by the showcase refresh path (out of band); the
  read never blocks on the database.

## Related flows

- [23-public-browse-with-filters md](23-public-browse-with-filters.md) — the full tutor browse
- [24-rate-limited-endpoint md](24-rate-limited-endpoint.md) — the sliding-window limiter
- [36-marketplace-detail-login-gated md](36-marketplace-detail-login-gated.md) — clicking a strip → login-gated detail
