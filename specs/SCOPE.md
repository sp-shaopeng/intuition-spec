# Historical scope and implementation notes — Architect + Tech Team → Product

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-05-29 · **From:** Solution Architect + Tech Lead
**To:** Product (roadmap owner) · **Purpose:** Tell Product exactly what is built,
what is in design, and what is not, so the roadmap and the business update reflect
reality.

## 1. Dated implementation notes; production launch unverified (M1–M12)

| # | Module | Behavior described in the dated implementation record |
|---|---|---|
| M1 | Auth | Register (tutee/tutor), login, admin time-based one-time passcode multi-factor authentication, password reset, change password, delete account |
| M2 | Posting | Tutees post requests; tutors post listings; public browse with filters |
| M3 | Approval | Admin reviews the queue and approves/rejects posts (sees authorized contact on post detail) |
| M4 | Matching | Admin creates, activates, and cancels matches |
| M5 | Commission | Admin marks commission paid; match completes |
| M6 | Interview | Tutee requests interview; admin schedules, sets link, completes, cancels, marks no-show |
| M7 | Profile | Tutor/tutee manage their profile |
| M8 | Match Dashboard | Tutee/tutor see their matches; counterparty contact revealed only when ACTIVE |
| M9 | Notifications | In-app bell + page, ~10 notification types |
| M10 | Search & Favourites | Filter sidebar, favourite heart, favourites page |
| M11 | Reviews | Post-match star reviews; aggregate rating on tutor cards |
| M12 | Admin Analytics | Platform metrics dashboard (verification pending) |

**Platform baseline:** Spring Boot 3.3 / Java 21, 14 controllers, 17 Liquibase
migrations, JaCoCo 97.5% line+branch gate. React 19 / Vite, 39 pages, Vitest
97.5% line / 88% branch. The later approved contact-storage amendment uses plaintext contact fields with strict access and disclosure controls; the earlier encryption design is superseded.

## 2. Honest gaps Product should know (not yet built)

- **No transactional email is actually sent** — the queue has no consumer
  (Roadmap Tier 1 #1). Password resets, reminders, match emails sit unsent.
- **No payments** — commission is marked paid manually; no Stripe/Pay Now yet.
- **No continuous integration/continuous deployment, no staging, no database backups** — all Tier 1 launch blockers.
- **No "apply" action** — matching is fully admin-driven (addressed by M15 below).
- **Security headers, cross-site request forgery protection enforcement, refresh tokens** — still open (Tier 1).

## 3. In design / newly approved (this session)

| # | Module | Status |
|---|---|---|
| M13 | Operations / People Directory + Secure personal information (isolated ops pii, OPERATOR role, audited ops console, /ops entry point, operator agent) | **Approved to build** (ADR-011) |
| M14 | Landing Showcase — 10–20 recent jobs (14-day window) + 10–20 active tutors (last active at ≤ 7d), random order, per-job applicant count | **Plan approved** (ADR-012) |
| M15 | Applications — tutor → job "apply"; powers the M14 applicant count and feeds matching | **Approved concept** (ADR-013) |

## 4. What Product should do with this

1. **Update the roadmap** — M13/M14/M15 added with ADR-011/012/013 (done in
   ROADMAP md); reflect that the landing showcase + apply concept are now in the
   plan ahead of generic Tier-2 items, per founder priority.
2. **Update the business team** — see [business-update md](business-update.md):
   what's live, what's genuinely missing (email, payments), and the new
   operations + marketplace-liquidity bets.
3. **Hold the launch-blocker line** — none of the new work removes the Tier-1
   blockers (email, payments, continuous integration/continuous deployment, staging, backups). Sequence accordingly.
