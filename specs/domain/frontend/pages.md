# InTuition — Pages Inventory

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Current route names and authentication boundaries follow reference/frontend-routes md. Public lists and their filters remain accessible to guests; details preserve return To through login. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.

> **⚠️ 2026-06-03 — superseded layout model below; see this banner + reference/frontend-routes md.**
> The **INT-049 all-pages redesign** changed the page/layout model. Pages no longer hand-roll headers/spacing/
> loading — they compose a shared **layout-primitive system** in intuition-the private implementation folder:
> **Page Header, Section, Container** (app|prose width), **Card Grid, Form Layout, Data List** (cards <md /
> table ≥md), **Tab Nav, Auth Card, Article Layout** (+ ui/Stat Card, ui/Status Timeline) — each unit-tested,
> all consuming the design tokens (whole-app arbitrary-value audit = 0). **IA change:** the per-role dashboard
> tabs moved **into the top Nav header** after login (the secondary in-page tab bar in Tutee Layout/
> Tutor Layout is removed; admin keeps its purple header tabs); login lands on the dashboard first tab. The
> per-page tables below predate this and describe the old sidebar/tab-bar layout — treat the layout columns as
> historical; the old route list is also historical (see reference/frontend-routes md for current routes and guards).

**Document Version:** 1.0  
**Date:** 2026-05-27  
**Status:** Draft  
**Platform:** InTuition

---

## Route Table

All routes are client-side (React Router v6). The Cloudflare Pages redirects file serves index html for all paths.

| # | Route | Auth Guard | Layout | Module | SEO |
|---|---|---|---|---|---|
| **Public** | | | | | |
| P-01 | / | none | Public Layout | — | Yes (landing) |
| P-02 | /tutors | none | Public Layout | Posting | Yes (tutor listings) |
| P-03 | /tutors/:id | authenticated | Public Layout | Posting | No (login-gated detail) |
| P-04 | /about | none | Public Layout | — | Yes |
| P-05 | /faq | none | Public Layout | — | Yes |
| P-06 | /contact | none | Public Layout | — | Yes |
| P-07 | /privacy | none | Public Layout | — | Yes |
| P-08 | /terms | none | Public Layout | — | Yes |
| **Auth** | | | | | |
| A-01 | /login | redirect if authed | Auth Layout | Auth | No |
| A-02 | /register | redirect if authed | Auth Layout | Auth | No |
| A-03 | /register/tutee | redirect if authed | Auth Layout | Auth | No |
| A-04 | /register/tutor | redirect if authed | Auth Layout | Auth | No |
| A-05 | /register/tutor/commission | redirect if authed | Auth Layout | Auth | No |
| A-06 | /forgot-password | none | Auth Layout | Auth | No |
| A-07 | /reset-password | none | Auth Layout | Auth | No |
| **Tutee** | | | | | |
| T-01 | /tutee/dashboard | TUTEE | Tutee Layout | — | No |
| T-02 | /tutee/posts | TUTEE | Tutee Layout | Posting | No |
| T-03 | /tutee/posts/new | TUTEE | Tutee Layout | Posting | No |
| T-04 | /tutee/posts/:id | TUTEE (own) | Tutee Layout | Posting | No |
| T-05 | /tutee/matches | TUTEE | Tutee Layout | Matching | No |
| T-06 | /tutee/matches/:id | TUTEE (own) | Tutee Layout | Matching | No |
| T-07 | /tutee/matches/:id/confirm-session | TUTEE (own) | Tutee Layout | Matching | No |
| T-08 | /tutee/interviews | TUTEE | Tutee Layout | Interview | No |
| T-09 | /tutee/interviews/:id | TUTEE (own) | Tutee Layout | Interview | No |
| T-10 | /tutee/account | TUTEE | Tutee Layout | Auth | No |
| T-11 | /tutee/notifications | TUTEE | Tutee Layout | — | No (P1) |
| **Tutor** | | | | | |
| R-01 | /tutor/dashboard | TUTOR | Tutor Layout | — | No |
| R-02 | /tutor/posts | TUTOR | Tutor Layout | Posting | No |
| R-03 | /tutor/posts/new | TUTOR | Tutor Layout | Posting | No |
| R-04 | /tutor/posts/:id | TUTOR (own) | Tutor Layout | Posting | No |
| R-05 | /tutor/requests | TUTOR | Tutor Layout | Posting | No |
| R-06 | /tutor/matches | TUTOR | Tutor Layout | Matching | No |
| R-07 | /tutor/matches/:id | TUTOR (own) | Tutor Layout | Matching | No |
| R-08 | /tutor/interviews | TUTOR | Tutor Layout | Interview | No |
| R-09 | /tutor/commission-terms | TUTOR | Tutor Layout | Commission | No |
| R-10 | /tutor/account | TUTOR | Tutor Layout | Auth | No |
| R-11 | /tutor/notifications | TUTOR | Tutor Layout | — | No (P1) |
| **Admin** | | | | | |
| AD-01 | /admin/login | redirect if admin-authed | Admin Auth Layout | Auth | No |
| AD-02 | /admin/mfa | admin-partial-auth | Admin Auth Layout | Auth | No |
| AD-03 | /admin/mfa/enroll | admin-auth no-mfa | Admin Auth Layout | Auth | No |
| AD-04 | /admin/dashboard | ADMIN | Admin Layout | — | No |
| AD-05 | /admin/posts | ADMIN | Admin Layout | Approval | No |
| AD-06 | /admin/posts/:id | ADMIN | Admin Layout | Approval | No |
| AD-07 | /admin/matching | ADMIN | Admin Layout | Matching | No |
| AD-08 | /admin/matches | ADMIN | Admin Layout | Matching | No |
| AD-09 | /admin/matches/:id | ADMIN | Admin Layout | Matching | No |
| AD-10 | /admin/interviews | ADMIN | Admin Layout | Interview | No |
| AD-11 | /admin/interviews/:id | ADMIN | Admin Layout | Interview | No |
| AD-12 | /admin/commissions | ADMIN | Admin Layout | Commission | No |
| AD-13 | /admin/commissions/:id | ADMIN | Admin Layout | Commission | No |
| AD-14 | /admin/commissions/:id/invoice | ADMIN | Admin Layout | Commission | No |
| AD-15 | /admin/users | ADMIN | Admin Layout | Auth | No |
| AD-16 | /admin/audit | ADMIN | Admin Layout | Auth | No |
| AD-17 | /admin/settings | ADMIN | Admin Layout | — | No |

---

## Auth Guards

| Guard | Behaviour |
|---|---|
| none | No redirect |
| redirect if authed | Redirect to role dashboard if already authenticated |
| TUTEE | Redirect to /login if not authenticated or not TUTEE |
| TUTOR | Redirect to /login if not authenticated or not TUTOR |
| ADMIN | Redirect to /admin/login if not authenticated, multi-factor authentication-unverified, or not ADMIN |
| TUTEE own | 403 if authenticated TUTEE but resource doesn't belong to them |
| TUTOR own | 403 if authenticated TUTOR but resource doesn't belong to them |
| admin-partial-auth | Only accessible mid-multi-factor authentication flow (mfa session cookie valid) |
| admin-auth no-mfa | Admin authenticated but multi-factor authentication not yet enrolled (enroll flow) |

---

## Layouts

| Layout | Description |
|---|---|
| Public Layout | Sticky nav (Logo + Browse Tutors + How It Works + Log In/Register CTA) + footer |
| Auth Layout | Centered card, no nav, InTuition logo |
| Tutee Layout | Sidebar nav (Dashboard, My Posts, My Matches, My Interviews, Account) |
| Tutor Layout | Sidebar nav (Dashboard, My Posts, Requests, My Matches, My Interviews, Commission Terms, Account) |
| Admin Layout | Sidebar nav (Dashboard, Posts, Matching, Matches, Interviews, Commissions, Users, Audit, Settings) + "Interviews" badge showing REQUESTED count |
| Admin Auth Layout | Same as Auth Layout |

---

## Page Metadata (SEO pages only)

| Route | page title element |  |
|---|---|---|
| / | InTuition — Find a Private Tutor in Singapore | Singapore's curated tuition marketplace. Meet your tutor before you commit. |
| /tutors | Browse Tutors — InTuition Singapore | Find verified private tutors for PSLE, O-Level, A-Level and more. |
| /tutors/:slug | the listing headline — InTuition Singapore | the tutor’s subjects, teaching levels, hourly price range and service area |

Open Graph tags (og:title, og:description, og:type) mirror the page title element and  values.
