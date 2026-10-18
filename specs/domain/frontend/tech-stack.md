# InTuition — Frontend Tech Stack

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Display scheduling in Singapore time while preserving each endpoint’s wire format. Use spec 004 for access/refresh cookies and ADR-005 for the frontend 97.5% line/88% branch threshold; older framework/provider snapshots are not new deployment decisions. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.

**Document Version:** 1.0  
**Date:** 2026-05-27  
**Status:** Draft  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite)

---

## 1. Stack Summary

| Concern | Library | Version | Notes |
|---|---|---|---|
| Framework | React | 18 | Concurrent mode, Suspense |
| Language | TypeScript | 5.x | strict: yes |
| Build tool | Vite | 5.x | SPA mode |
| Routing | React Router | v6 | create Browser Router |
| Server state | Tan Stack Query | v5 | Query + mutation; no Redux |
| user interface state | Zustand | 4.x | Auth user, user interface flags only |
| Forms | React Hook Form | 7.x | + Zod resolvers |
| Validation | Zod | 3.x | Frontend validation only |
| Styling | Tailwind CSS | 3.x | + design tokens (see design-system md) |
| user interface components | Headless user interface | 2.x | Accessible primitives |
| Icons | Heroicons | 2.x | |
| web request protocol client | Axios | 1.x | Configured with cookie credentials |
| Date/time | date-fns | 3.x | Singapore time timezone handling |
| Testing | Vitest + RTL | latest | |
| Contract tests | Pact (consumer) | 11.x | |
| end-to-end | Playwright | latest | Staging only |
| Hosting | Cloudflare Pages | — | Static SPA, CDN-served |

---

## 2. Project Structure


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 3. Authentication Pattern

The auth http Only cookie is set by Spring Boot on login. The React SPA does not access the cookie directly.

On app load:
1. auth Store is initially an unresolved sign-in check, with no user loaded yet
2. read operation for auth me is called (cookie sent automatically by browser, with Credentials: yes)
3. On 200: store the signed-in user’s account details after the sign-in check finishes
4. On 401: store a completed sign-in check with no signed-in user — user is not authenticated

Protected routes (<Require Auth role is TUTEE>) check auth Store user — redirect to /login if null.

---

## 4. Axios Configuration


Implementation example omitted. This public edition retains the written requirements in this section.


No Authorization: Bearer header. Auth is entirely cookie-based.

---

## 5. Page Rendering Strategy

| Page type | Rendering | SEO | Cache |
|---|---|---|---|
| / (landing) | Client-rendered SPA | Cloudflare prerender (v2) | Edge cache 60s |
| /tutors (listing) | Client-rendered SPA | Cloudflare prerender (v2) | Edge cache 60s |
| /tutors/:slug (detail) | Client-rendered SPA | Cloudflare prerender (v2) | Edge cache 60s |
| /login, /register | Client-rendered SPA | Not indexed | No cache |
| /tutee/*, /tutor/*, /admin/* | Client-rendered SPA | Not indexed | No cache |

**v1 SEO note:** Cloudflare Pages prerender workers are deferred to v2 (ADR-012). In v1, Google's modern crawler handles client-rendered React well enough for initial organic traffic. Monitor Google Search Console indexing rate after launch — if issues arise, accelerate the v2 prerender implementation.

**SPA route fallback:** public/ redirects file must contain:


The illustration for . page rendering strategy contains the following information:

- /* /index html 200.


This ensures all routes are served the index html shell, and React Router handles client-side routing. Without this, direct URL access to /tutors/some-slug returns a 404 from Cloudflare.

---

## 6. Bundle Size Budget

| Bundle | Max size (gzip) | Enforcement |
|---|---|---|
| Initial JS (vendor) | 150 KB | Vite build rollup Options output.manual Chunks |
| Initial JS (app) | 80 KB | |
| Route chunk (authenticated pages) | 40 KB per chunk | Lazy-loaded via the lazy operation |
| CSS (Tailwind purged) | 30 KB | |

Budgets enforced via vite-bundle-visualizer in continuous integration — build warning if exceeded.

**Lazy loading:** All authenticated page components are lazy-loaded:


Implementation example omitted. This public edition retains the written requirements in this section.


Public pages (landing, browse tutors, tutor detail) are eager-loaded in the main bundle for fast first paint.

---

## 7. Loading State Spec

| Scenario | Loading user interface |
|---|---|
| Tutor card grid (initial load) | Skeleton cards: 6 × card-shaped gray placeholders |
| Tutor post detail | Skeleton: headline block + 3 content rows |
| Interview list | Skeleton: 3 × row-shaped placeholders |
| Form submit | Spinner inside button, button disabled |
| Global initial auth check (/auth/me) | Full-page loading spinner for max 2s; then render (or redirect if 401) |

No layout shift permitted on skeleton → content transition: skeleton dimensions must match actual content dimensions.

---

## 8. Singapore time Timezone Handling

All dates and times are stored and transmitted as Coordinated Universal Time ISO 8601 strings. Display in Singapore Time (Singapore time, Coordinated Universal Time+8):


Implementation example omitted. This public edition retains the written requirements in this section.


Always show the Singapore time label in the user interface — never show timezone-naive times.

---

## 9. Currency Formatting


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 10. TypeScript Config


The . type script config example describes information including compiler Options, strict, no Unchecked Indexed Access, exact Optional Property Types, target, lib, module, module Resolution, jsx, paths. Concrete payload values and implementation syntax are omitted.
