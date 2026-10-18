# All-Pages user interface Redesign — Modern & Reader-Friendly (every route)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

Preserve locked visual tokens and recorded design decisions. Privacy copy must describe the approved ACTIVE/COMPLETED mutual-participant exception, not never-share-ever. Anonymous visitors may filter limited public results; full detail remains login-gated. Apply [018](../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md) enumerated login versus role/create-account CTA intent; generic login wording does not override explicit registration intent.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

Status: **OPEN — spec authored, awaiting /team-review → Boss → implementation** ·
Authors: user interface/user experience Designer (Jamie, lead) + Frontend Designer + Frontend Expert ·
Date: 2026-06-03 · Tracks: extends INT-049 to the full app ·
Supersedes scope of: [redesign-modern-readable-spec md](redesign-modern-readable-spec.md)
(the public-page + Phase-0 spec). This document **EXTENDS, never contradicts** it.

Founder directive (2026-06-03): the public surface redesign is in progress; the
founder now wants **EVERY page** modernized to the same bar — reader-friendly,
mobile-first, on-brand. This is the single authoritative spec for that whole-app
redesign. It builds on the **already-shipped Phase-0 design-system foundation**
(the token scale is now wired into tailwind config implementation reference,
the private implementation folder, and the private implementation folder* — confirmed below in §2).

> This is a **spec / design document**. It produces **no code** — only this
> Markdown under intuition-spec/. Implementation happens in the rollout waves
> (§6), each gated by /team-review → Boss.

---

## 0. Non-negotiables (carried forward, verbatim)

These are **locked** and must survive every page redesign. A page is not "done"
if any is regressed:

1. **Brand hex + fonts are LOCKED.** Navy #0 F2138 / clay #C84 B31 / cream
   #FBF6 EE + Instrument Serif / Inter, per
   [design-tokens implementation reference](../domain/uiux/design-tokens.md) §1.1/§2.1.
   No new brand colours, no font swaps. Status & admin colours come from the
   token map, never raw Tailwind palette (bg-amber-100 etc. are banned).
2. **Hero headline** "Meet your tutor before you commit." (Landing) stays verbatim.
3. **The privacy promise** — contact details remain private on public/pre-match surfaces. After the team
   confirms an ACTIVE match, both matched participants may reveal each other's contact details. Explain this
   mutual visibility before it occurs (INT-090/097). Public cards remain handle-only; never promise "never shared, ever".
4. **Intent-based, role-honest nav** — TUTEE sees "Find a Tutor", TUTOR sees "Find
   Tuition Jobs", logged-out sees both; "How It Works" hidden once logged in;
   logo + Dashboard route by role (INT-029/032/037/038/040). The redesign keeps
   this logic exactly; it only restyles the chrome.
5. **Commission 50% / SGD prominence** on the tutor-registration commission page —
   the "50%" and the worked SGD example remain the most prominent elements
   (design-tokens md §9). Untouched by this redesign.
6. **Admin purple distinction is intentional** (design-tokens md §10, §12.2). The
   admin shell stays unmistakably purple vs the cream public app. We modernize
   *within* purple; we do not flatten the distinction. (Note: current
   Admin Layout implementation reference uses a hardcoded bg- #3 B1 F6 B and bg-gray-50 page —
   §5.4 reconciles these to the admin* tokens.)
7. **Auth-CTA principle (M17)** — the only path to /register is an explicit
   "Create account" control; every other auth CTA defaults to /login. Restyling
   buttons must not re-point them.

---

## 1. Scope & inventory — EVERY route

Source of truth: [frontend-routes implementation reference](../reference/frontend-routes.md)
+ the actual pages under intuition-the private implementation folder** and layouts under
the private implementation folder*. Every route below is in scope. "Current state" cites the dominant
"not-modern" tells (hardcoded type, text-sand intro copy, bare-spinner loading,
ad-hoc per-page layout); "Redesign intent" states the target. **Wave** maps to §6.

### 1.1 Public (no auth) — Public Layout

| # | Route | Page file | Current state | Redesign intent | Wave |
|---|---|---|---|---|---|
| P1 | / | Landing implementation reference | text- 72px hero, desktop-grid-locked, emoji icons, non-wrapping stats | Mobile-first hero ladder, line icons, stacking stats, dual-CTA (parent-first) | A |
| P2 | /tutors | Browse Tutors implementation reference | text- 48px H1, bare spinner, full-width intro on sand | H1 ladder, Skeleton Card grid, ink measure intro, mobile filter sheet | A |
| P3 | /assignments | Browse Assignments implementation reference | mirrors P2; "Open" badge blue | As P2; "Open" → green active token; personal information-free projection kept | A |
| P4 | /tutors/:id | Tutor Detail implementation reference | auth-gated detail; dense, no Page Header | Page Header + Section primitives; reviews block; handle-only; contact gated | A |
| P5 | /assignments/:id | Assignment Detail implementation reference | auth-gated detail; dense | As P4; anonymised job detail; apply CTA → login if anon | A |
| P6 | /login | Login implementation reference | form floats on cream, raw bg-red-50 error | Elevated paper Auth Card, tokenised error, one primary action | A |
| P7 | /register | Register implementation reference | 6+ field wall, emoji role cards, off-palette strength meter | Role Selection step + chunked Form Layout on a card | A |
| P8 | /register/commission | Commission implementation reference | standalone commission step | Polish within design-tokens md §9 — **50%/SGD prominence LOCKED** | A |
| P9 | /forgot-password | Forgot Password implementation reference | bare form | Auth Card pattern (shared with Login) | A |
| P10 | /reset-password | Reset Password implementation reference | bare form | Auth Card pattern | A |
| P11 | * | Not Found implementation reference | minimal 404 | Empty State-styled 404 with route-home CTA | A |

### 1.2 Info / content — Public Layout

| # | Route | Page file | Current state | Redesign intent | Wave |
|---|---|---|---|---|---|
| I1 | /about | About implementation reference | raw prose, text- 40px, text-sand sub, full-width | Article Layout: max-w-prose, type ladder, Section rhythm | D |
| I2 | /contact | Contact implementation reference | contact form, ad-hoc | Form Layout on Auth Card-style surface + contact details Section | D |
| I3 | /faq | Faq implementation reference | flat Q/A list | Accordion (-based, zero-dep), Section grouping | D |
| I4 | /privacy | Privacy implementation reference | long legal prose | Article Layout, max-w-prose, anchored sub-headings | D |
| I5 | /terms | Terms implementation reference | long legal prose | Article Layout (shared with Privacy) | D |
| I6 | /rates | Rate Guide implementation reference | rate tables, full-width | Article Layout + responsive Data Table; SGD prominence | D |

### 1.3 Tutee dashboard (TUTEE) — Tutee Layout

| # | Route | Page file | Current state | Redesign intent | Wave |
|---|---|---|---|---|---|
| T1 | /tutee/dashboard | Dashboard implementation reference | text- 36px, text-sand sub, bare spinner | Page Header + Card-grid list, Skeleton List, Empty State | B |
| T2 | /tutee/posts/new | Create Request implementation reference | 508-line single-scroll wizard | Form Layout sections + availability grid (design-tokens md §8) | B |
| T3 | /tutee/profile | Profile implementation reference | ad-hoc form | Form Layout + Profile Completeness Bar | B |
| T4 | /tutee/matches | Matches implementation reference | match list + contact reveal | Card grid, status badges, Contact Reveal Card (gated on ACTIVE) | B |
| T5 | /tutee/interviews | Interviews implementation reference | interview list | Card/list + status badges, Skeleton List | B |
| T6 | /tutee/interviews/:id | Interview Detail implementation reference | detail + cancel | Page Header + Section, status timeline | B |
| T7 | /tutee/favourites | Favourites implementation reference | tutor cards | Tutor Card grid (shared), Empty State | B |
| T8 | /tutee/reviews | Reviews implementation reference | review list | Review Card list (shared), Empty State | B |
| T9 | /tutee/requests/:job Id/applicants | Applicants implementation reference | personal information-free applicant cards + choose-tutor | Card grid, handle-only, selection CTA, Pii Notice | B |
| T10 | /matches/:match Id/review | Submit Review implementation reference | review form (match-scoped) | Form Layout + Star Rating, Auth Card surface | B |
| T11 | /notifications | Notifications implementation reference | notification list (shared route) | List primitive, read/unread states, Empty State | B |

### 1.4 Tutor dashboard (TUTOR) — Tutor Layout

| # | Route | Page file | Current state | Redesign intent | Wave |
|---|---|---|---|---|---|
| R1 | /tutor/dashboard | Dashboard implementation reference | listings + availability toggle + showcase settings | Page Header + Card-grid, Toggle, Skeleton List | B |
| R2 | /tutor/listings/new | Create Listing implementation reference | single-scroll wizard | Form Layout sections + availability grid | B |
| R3 | /tutor/profile | Profile implementation reference | canonical identity (handle/rate/area) | Form Layout + Profile Completeness Bar (M18 canonical profile) | B |
| R4 | /tutor/matches | Matches implementation reference | match list + contact reveal | Card grid + Contact Reveal Card (gated on ACTIVE) | B |
| R5 | /requests | Browse Requests implementation reference | active tutee requests browse | Card grid + filter, Skeleton Card, Empty State | B |
| R6 | /tutor/interviews | Interviews implementation reference | interviews for own posts | Card/list + status badges | B |
| R7 | /tutor/interviews/:id | Interview Detail implementation reference | detail | Page Header + Section, status timeline | B |
| R8 | /tutor/reviews | Reviews implementation reference | reviews received | Review Card list, aggregate header, Empty State | B |
| R9 | /tutor/applications | Applications implementation reference | apply/withdraw (M15/M18) | Card list + status badges, withdraw action | B |
| R10 | /tutor/invitations | Invitations implementation reference | selection-request invitations | Card list + accept/decline, handle-only | B |

> Inventory note: Applications implementation reference and Invitations implementation reference exist under
> the private implementation folder and are in scope even where frontend-routes md predates
> their final route wiring — the redesign treats them as first-class tutor pages.

### 1.5 Admin console (ADMIN) — Admin Layout (purple shell — preserved)

| # | Route | Page file | Current state | Redesign intent | Wave |
|---|---|---|---|---|---|
| A1 | /admin → /admin/review | redirect | — | unchanged | C |
| A2 | /admin/review | Review Queue implementation reference | queue table + service-level agreement dots | Admin Data Table, service-level agreement indicators (design-tokens md §1.5), Admin Page Header | C |
| A3 | /admin/review/:id | Review Detail implementation reference | detail + approve/reject | Admin Page Header + Section, contact panel (admin Contact Panel tokens), Modal confirms | C |
| A4 | /admin/matching | Matching implementation reference | match create/manage + modals | Admin Data Table + Modal flows (radius/shadow upgrade) | C |
| A5 | /admin/posts/:job Id/applicants | Job Applicants implementation reference | admin applicant list | Admin Data Table, applicant count admin-only | C |
| A6 | /admin/commission | Commission implementation reference | commission tracking | Admin Data Table, commission status badges (design-tokens md §1.4), overdue row tint | C |
| A7 | /admin/interviews | Interview Queue implementation reference | filterable queue | Admin Data Table + sub-nav/filter tabs, Modal flows | C |
| A8 | /admin/interviews/:id | Interview Detail implementation reference | schedule / no-show | Admin Page Header + Section, scheduling form, Modal confirms | C |
| A9 | /admin/analytics | Analytics implementation reference | metrics + daily chart + metric tabs | Admin Page Header, Stat Card grid, tab/sub-nav, responsive chart | C |
| A10 | /admin/admins | Admins implementation reference | admin management + super-admin | Admin Data Table + create-admin Modal; super-admin protected | C |
| A11 | ADMIN CHANGE PASSWORD PATH | Force Change Password implementation reference | full-screen forced gate (M16) | Full-screen Auth Card (purple-accented), no escape nav | C |

**Total in scope: 11 public + 6 info + 11 tutee + 10 tutor + 11 admin = 49 routes/pages.**

---

## 2. Design-system consolidation — confirm Phase-0, add layout primitives

### 2.1 Phase-0 is shipped — confirmed, do not re-litigate

The token scale from design-tokens md is **already wired** into
tailwind config implementation reference (verified 2026-06-03):
font Size (display→eyebrow ladder), box Shadow (sm/md/lg/xl), border Radius
(none/minimal/card/modal/full), max Width prose (68ch), the locked brand colors,
and the semantic status.* map. The readability baseline + prefers-reduced-motion
block live in the private implementation folder, and the ~18 primitives in the private implementation folder*
consume them. **This spec consumes that foundation; it does not redefine a single
token.** Phase-0 is the prerequisite, not a deliverable of this spec.

The whole-app redesign rule (carry-over lesson): **consume the system, do not
reinvent it.** The audit grep
the rounded operation
must return **zero** new hits on every redesigned page. Every text- N Npx,
generic rounded-lg, text-sand intro paragraph, and raw-palette status colour
found in §1's current-state notes is a defect to eliminate, not to copy.

### 2.2 Shared page-layout primitives (the system layer this redesign adds)

The single biggest "not-a-system" gap today is that **every page hand-rolls its
own header, spacing, loading, and empty markup** (text- 36px + text-sand sub
on Dashboard; raw prose blocks on About; bare <Spinner> centring everywhere).
The redesign introduces a small set of **composable layout primitives** so pages
become declarative compositions, not bespoke markup. These are new
the private implementation folder (or the private implementation folder) components — **no new deps**,
pure Tailwind + the wired tokens. They are the reusable contract every page in
§3–§5 builds on.

| Primitive | Responsibility | Key props / behaviour | Tokens used |
|---|---|---|---|
| **Page Header** | The standard page title block (replaces every per-page <h1>+sub) | title, subtitle?, actions? (right-aligned CTA slot), eyebrow?, back? | text-h1m md:text-h1 serif title, text-body-sm/ink subtitle (NOT sand), actions stack below title on mobile |
| **Section** | A titled content band with consistent vertical rhythm | title? (text-h2m md:text-h2 or text-h3), description?, tone? (default/muted/callout) | py-12 md:py-16 rhythm; callout → bg-cream-muted + clay left-border |
| **Container** | Responsive width wrapper, two reading modes | width = app (max-w-7xl) \| prose (max-w-prose, ~68ch); px-4 md:px-8 | max Width prose for all long-form reading (info pages, legal) |
| **Card Grid** | Responsive card grid (the marketplace + dashboard list pattern) | cols = one column on small screens, two on medium screens and three on large screens; gap-6 | renders children as shadow-md cards, hover shadow-lg+lift |
| **Form Layout** + Form Layout Section | Chunked forms (≤5–7 fields/group, Miller's law) | section legend, description?; consistent field gap gap-6; sticky submit bar on mobile | inputs from Input primitive (visible focus); error tokenised |
| **Empty State** | Already exists — standardised empty slot | icon (line-icon, no emoji), title, description (measure), action? | from §2.6 of base spec |
| **Data List / Data Table** | The list/table pattern (dashboards = cards on mobile, table on desktop; admin = table) | responsive: **stacks to cards < md, table ≥ md**; columns, rows, row Action?, loading → Skeleton Rows, empty → Empty State | shadow-sm rows, hover lift; admin variant uses admin Table Header/admin Table Row Hover tokens |
| **Tab Nav / Sub Nav** | In-page tab strip (dashboards, admin filters, analytics metric tabs) | tabs, active; underline-active pattern (matches existing Tutee Layout tab bar) | active border-clay text-navy (public) / border-admin text-admin (admin) |
| **Auth Card** | Centred elevated surface for auth + single-form pages | title, eyebrow?, slot for form; max-w- 420px centred | paper bg, rounded-card, shadow-md, p-6 |
| **Article Layout** | Long-form reading (info/legal pages) | Container width is prose, type ladder, anchored h2 sub-headings, generous Section rhythm | max-w-prose, body/body-lg, serif headings |
| **Stat Card** | Single metric tile (analytics, landing stats) | label, value, delta?, tone? | paper/shadow-sm, value text-h2, label text-caption uppercase |
| **Status Timeline** | Vertical status progression (interview/match detail) | steps, current | status dots from status.* tokens, rounded-full |

**Loading parity (system-wide rule):** every data page renders a **skeleton that
matches its final layout** — Card Grid → Skeleton Card grid; Data List/Data Table
→ Skeleton rows; detail pages → skeleton header + section blocks. The bare-centred
<Spinner> loading state (currently on Tutee Dashboard, both Browse pages, etc.) is
**retired** for full-page loads to kill CLS. Spinner is retained only for inline
button/control busy states.

**State completeness rule:** every data page specifies all four states —
**loading (skeleton)**, **empty (Empty State + primary action)**, **error
(Empty State-style + retry)**, **success**. No data page ships with fewer than four.

### 2.3 Primitive upgrades inherited from the base spec

The ~18 ui/* primitive upgrades in redesign-modern-readable-spec md §2.6
(Button on Dark/navy-on-light + shadow-on-hover, Input visible focus, Badge
status tokens, Modal radius/shadow, Empty State line-icon, Skeleton radius,
Spinner/Toast reduced-motion, Contact Reveal Card/Pii Notice/Review Card radius) are
**Phase-0/Wave-A prerequisites** and are not re-specified here. This spec assumes
them done and composes the new layout primitives (§2.2) on top.

---

## 3. Per-group redesign direction — mobile-first (375px), wireframe-level

Public pages (P1–P11) are fully specified in
redesign-modern-readable-spec md §3 (Landing, Browse Tutors, Browse
Assignments, Login, Register) and are **not duplicated here** — this section adds
the **detail pages, info pages, and both authenticated dashboards + admin** that
the base spec deferred. All wireframes are 375px-first, then the ≥md adaptation.

### 3.1 Public detail pages (P4 /tutors/:id, P5 /assignments/:id)


The illustration for public detail pages p4 /tutors/:id, p5 /assignments/:id contains the following information:

- 375px — Tutor Detail.
- ‹ Back to tutors (Page Header back slot.
- eyebrow: VERIFIED TUTOR).
- H1: @handle (handle-only!) text-h1m — NO real name (personal information lock).
- subjects · levels · ★4.8 (12) Star Rating count.
- Section "About this tutor" measure ≤68ch body.
- Section "Rate & commitment" Commitment Badge, SGD rate.
- Section "Availability" read-only availability grid.
- Section "Reviews" (Review Card reviews stay PUBLIC (M17).
- list, stacks).
- sticky CTA bar (mobile): one primary action.
- [ Request an interview ] → login if anon (return To).
- ≥md: 2-col — sticky right rail (rate/CTA card) left content column.


- **Assignment Detail** mirrors this: anonymised job (no title/contact/exact date;
  bucketed budget; planning-area location — personal information-free projection **unchanged**),
  Sections "Requirements / Schedule / Location", primary CTA "Apply" → login if
  anon. Reviews block omitted.
- Both: Page Header + Section + Status Timeline (where a match/application status
  exists) primitives; loading = skeleton header + section blocks (not spinner).

### 3.2 Info / content pages (I1–I6)

All use **Article Layout** (Container width is prose, max-w-prose ~68ch) — the
fix for the current full-width raw-prose blocks (About implementation reference runs ~100ch on sand).


The illustration for info / content pages i1–i6 contains the following information:

- 375px — About / Privacy / Terms / Rate Guide.
- Page Header (eyebrow H1 text-h1m md:text-h1.
- subtitle on INK not sand).
- prose column (≤68ch).
- Section h2 (serif h2m→h2) anchored sub-headings.
- body 16/1.6, lists, links links clay→navy underline.
- Section ….


- **About (I1):** mission + how-it-works + verification + contact link. Replace
  text- 40px/text-sand with Page Header; body in Container width is prose.
- **FAQ (I3):** **Accordion** built on native / (zero-dep,
  keyboard + screen-reader native), grouped by Section; respects reduced-motion.
- **Privacy (I4) / Terms (I5):** Article Layout, anchored h2 sub-headings, last-
  updated caption. Reuse one Legal Page composition.
- **Rate Guide (I6):** Article Layout + responsive Data Table for rate bands
  (stacks to labelled cards < md); SGD figures prominent (text-h4/600).
- **Contact (I2):** intro Section + Form Layout (name/email/message) on an
  Auth Card-style surface + a contact-details Section beside it (≥md two-col).

### 3.3 Tutee dashboard (T1–T11)

Shared shell: Tutee Layout (top Nav + tab bar + max-w-7xl main). The tab bar
underline pattern is the Tab Nav reference. Every page below opens with a
Page Header and uses the four-state rule.


Implementation example omitted. This public edition retains the written requirements in this section.


- **Create Request (T2) / [tutor] Create Listing (R2):** the 500-line single-scroll
  wizards become **Form Layout with chunked Form Layout Sections** (Your details
  / Subject & level / Budget / Availability / Venue), each ≤5–7 fields. The
  **availability grid** follows design-tokens md §8 exactly (desktop 7×3 grid;
  mobile accordion-per-day, 44px touch targets, weekend default-expanded). Sticky
  submit bar on mobile.
- **Profile (T3) / Tutor Profile (R3):** Form Layout + Profile Completeness Bar
  header. Tutor profile is the M18 canonical identity (handle/commitment/rate/area)
  — keep those fields prominent.
- **Matches (T4):** Card Grid of match cards; status badge; **Contact Reveal Card
  only on ACTIVE** (admin-gated reveal — personal information rule). Loading skeleton cards.
- **Interviews (T5) + Interview Detail (T6):** list = Data List with status badges;
  detail = Page Header + Section + **Status Timeline** (requested → scheduled →
  completed / no-show) + cancel action (Modal confirm).
- **Favourites (T7):** shared Tutor Card grid (handle-only); Empty State → "Browse
  tutors" CTA.
- **Reviews (T8):** shared Review Card list; Empty State.
- **Applicants (T9):** Card Grid of **personal information-free** applicant cards (handle only) +
  Pii Notice; choose-tutor selection CTA per card (Modal confirm). This is a
  high-personal information-risk surface — handle-only is mandatory and re-tested (§5 guardrail).
- **Submit Review (T10):** Form Layout + Star Rating on an Auth Card surface.
- **Notifications (T11):** Data List of notifications, read/unread visual states
  (unread = cream-muted row + clay dot), mark-all-read action, Empty State.

### 3.4 Tutor dashboard (R1–R10)

Identical system to §3.3 under Tutor Layout. Page-specifics:

- **Tutor Dashboard (R1):** Page Header + Data List of listings; per-row
  Toggle (Available for students) — keep optimistic update; Showcase Settings
  panel becomes a Section (featured consent / M14a availability). Empty State →
  "Create a listing".
- **Browse Requests (R5):** mirrors public Browse — Card Grid + filter + Skeleton Card
  grid + Empty State; handle-only on any tutee-facing card.
- **Matches (R4) / Interviews (R6) / Interview Detail (R7) / Reviews (R8):** same
  primitives as the tutee equivalents (Card grid / Status Timeline / Review Card).
- **Applications (R9):** Data List of the tutor's applications with status badges
  (APPLIED / SUPERSEDED / WITHDRAWN / MATCHED) + withdraw action (Modal confirm).
- **Invitations (R10):** Data List of selection-request invitations (handle-only)
  + accept/decline actions; Empty State.

### 3.5 Admin console (A1–A11) — modernize WITHIN the purple shell

The purple distinction is **kept and reinforced**, only modernized. The admin
shell adopts the layout primitives **in their admin variant** (purple active
states, admin* tokens, admin Table Header/admin Contact Panel styling). See §5.4
for the shell-token reconciliation.


The illustration for admin console a1–a11 — modernize within the purple shell contains the following information:

- 375px — Admin Review Queue (A2).
- Admin top bar (PURPLE, sticky) unmistakable admin context (LOCKED).
- Admin Page Header H1 queue count.
- Sub Nav / filter tabs (purple Tab Nav admin variant.
- active underline).
- Data Table (admin variant):.
- stacks to cards < md mobile parity (admins on phones too).
- [service-level agreement dot] post · age · action service-level agreement dots: green/amber/red OVERDUE.
- row hover = white shadow Sm admin Table Row Hover tokens.
- LOADING: skeleton rows.
- EMPTY: "Queue clear" Empty State.


- **Review Detail (A3) / Interview Detail (A8):** Admin Page Header + Sections; the
  **contact panel** uses admin Contact Panel tokens (#EDE9 FF deep tint + purple
  border, "ADMIN ONLY — Contact Information" label) — this identifies the admin authorization context;
  own-profile and ACTIVE/COMPLETED matched-participant contact reveal have their separately authorized contexts. The panel is visually unmistakable
  (design-tokens md §12.2). Approve/reject/schedule via upgraded Modal
  (radius/shadow), with every close-path test-covered (SHARED-LESSONS modal lesson).
- **Matching (A4) / Job Applicants (A5):** admin Data Table + Modal create/assign
  flows; applicant count is **admin-only**.
- **Commission (A6):** admin Data Table with commission status badges
  (design-tokens md §1.4) + overdue-row amber tint (comm Overdue Row Bg); SGD +
  50% figures prominent.
- **Interview Queue (A7):** admin Data Table + filter Tab Nav (all statuses).
- **Analytics (A9):** Admin Page Header + **Stat Card grid** (signups / interviews
  / matches / commission) + metric Tab Nav + responsive daily chart (chart stacks/
  scrolls on mobile; SVG, no chart-lib dep). Quick-range presets as btn Sm chips.
- **Admins (A10):** admin Data Table + create-admin Modal; super-admin row
  protected (no delete/demote affordance); password change required indicator.
- **Force Change Password (A11):** full-screen Auth Card (purple-accented), **no
  escape nav** — the M16 forced gate behaviour is preserved exactly.

---

## 4. Non-negotiables verification matrix

Each row maps a §0 non-negotiable to where it is checked during the redesign.

| Non-negotiable | Pages it touches | Verification |
|---|---|---|
| Hero copy verbatim | Landing (P1) | Snapshot/string assert "Meet your tutor before you commit." |
| Privacy promise | Landing, Browse Tutors, Browse Assignments | Text-present assert on all three |
| Handle-only personal information | All public cards/details, Favourites, Applicants, Invitations, Browse Requests | personal information-contract test extended; Network-tab check |
| Role-honest nav | Nav, all layouts | Existing nav logic untouched; role-render tests kept |
| Commission 50%/SGD | Commission (P8), Admin Commission (A6), Rate Guide (I6) | Prominence assert; design-tokens md §9 unchanged |
| Admin purple distinction | Admin Layout + all A* pages | Purple shell present; §5.4 token reconciliation |
| Auth-CTA → login (register only on explicit create) | Nav, Login, all acquisition CTAs | CTA href assert; restyle must not re-point |
| Brand hex/fonts locked | every page | audit grep returns zero new arbitrary values |

---

## 5. Guardrails

- **Tailwind v3 only.** No upgrade to v4, no CSS-in-JS, no component kit (Headless
  user interface / Radix / shadcn / MUI). The new layout primitives (§2.2) are plain
  React + Tailwind + the wired tokens. Native <dialog> Modal and native
   Accordion (FAQ) stay — zero new deps.
- **No heavy deps.** No animation library, no icon package, **no chart library**
  (Analytics chart is hand-authored SVG). Any proposed dep states bundle cost +
  the rejected cheaper alternative and passes Frontend Expert review.
- **Core Web Vitals budget** (public + first authenticated paint, throttled
  mobile 4G): **LCP ≤ 2.5s** (hero/title text is LCP — no lazy-chunk dependency;
  font display is swap kept), **CLS ≤ 0.1** (skeletons reserve every async region —
  every data page adopts skeleton-matched-to-layout; no bare-spinner full loads),
  **INP ≤ 200ms** (toggles/hearts/filters stay optimistic). No public-route JS
  regression beyond **+5KB gzip** per wave without written justification; keep the
  showcase-strip code-split.
- **WCAG 2.2 AA.** 4.5:1 body, 3:1 large/user interface; muted/sand is helper-only (every
  intro/subtitle moves off sand to ink); axe-clean per page.
- **Keyboard nav.** Visible focus-visible ring on every interactive element;
  logical tab order; modal focus-trap + ESC (native <dialog>); FAQ accordion
  native keyboard; skip-to-content link in each layout; no keyboard trap.
- **prefers-reduced-motion.** The index css block + per-component fallbacks
  (Spinner static ring, Skeleton static tint, accordion instant) — no motion-only
  signal; card-hover lift suppressed under reduce.
- **frontend coverage gate — every page keeps it.** the documented operational check must stay
  **≥97.5% lines / ≥88% branches** (Vitest). Tests are **updated, not deleted**.
  When restyling a page with modals, enumerate every close-path as its own case
  and assert closure via the open attribute (SHARED-LESSONS 2026-06-03 modal
  lesson) — do not lower the gate or exclude files. Sequence Phase-0/Wave-A markup
  to settle **before** snapshot-sensitive admin tests are rewritten (Boss
  condition, avoids coverage rework).
- **personal information non-regression.** After any card/identity restyle, re-run + extend the
  handle-only personal information-contract test (the Showcase Tutor Response style) so no public
  card, applicant card, or invitation can render a real name. Verify in the Network
  tab, not just the test.
- **No-Magic-Strings parity.** Status colours via the status.* token map; application programming interface
  paths via application programming interface ROUTES; enum values via typed enums — never literals.

### 5.4 Admin shell token reconciliation (defect to fix during Wave C)

Admin Layout implementation reference currently uses bg-gray-50 (page) and a **hardcoded**
bg- #3 B1 F6 B header — neither matches the design-tokens md §10 admin tokens
(admin Page Bg #F3 F0 FF, admin Header Bg #7 C3 AED). The redesign reconciles the shell
to the **named admin tokens** (add them to tailwind config implementation reference as an admin.*
colour group, same way status.* was added in Phase-0) so the purple distinction
is token-sourced, not a magic string. The *intent* (unmistakable purple) is
unchanged; only the source of the colour is corrected. Active-tab underline →
admin Light #A78 BFA; focus ring stays admin-purple per design-tokens md §12.3.

---

## 6. Rollout waves

Each wave ends with /team-review (full 18-agent debate, no quick mode) → Boss
verdict → fix every condition in place + re-verify by re-reading → only then the
next wave. Incremental, reversible (per-page class swaps + additive primitives),
no big-bang.

> **Prerequisite — Phase 0 (DONE):** token scale wired into tailwind config implementation reference
> + index css + the ~18 ui/* primitive upgrades. Confirmed shipped (§2.1).
> The new **layout primitives (§2.2)** land at the **start of Wave A** (they are
> the system layer the rest of the app composes) and are unit-tested in isolation
> before any page consumes them.

| Wave | Scope | Pages | Status |
|---|---|---|---|
| **A** | Public + auth + layout primitives | P1–P11 (+ §2.2 primitives) | **In progress** (base spec §3) |
| **B** | Authenticated dashboards (tutee then tutor) | T1–T11, R1–R10 | Pending |
| **C** | Admin console (LAST) — within purple shell | A1–A11 (+ §5.4 token reconcile) | Pending |
| **D** | Info / content pages | I1–I6 | Pending |

### 6.1 Per-wave acceptance criteria

**Every wave** (applies to each page in it):
1. Composed from §2.2 primitives — no per-page bespoke header/spacing/loading.
2. Audit grep returns **zero** new arbitrary values (text, generic
   rounded-lg/xl, raw bg-*-100, text-sand on intro/body).
3. 375px: stacks cleanly, no horizontal scroll, no clipped headline; type uses the
   mobile→md ladder. Verified by screenshot.
4. Four states present on every data page (loading skeleton / empty+action /
   error+retry / success).
5. Readability: body 16/1.6, lead 20/1.65, long-form ≤68ch; intro/subtitle on
   ink; WCAG 2.2 AA (axe + manual contrast).
6. Modern feel: cards shadow-md→hover shadow-lg+lift; radius family on
   rounded-card; line icons (no emoji).
7. Interactive states: hover + focus-visible ring + disabled on every control;
   inputs show visible focus.
8. CWV budget met (LCP ≤ 2.5s, CLS ≤ 0.1, INP ≤ 200ms); JS within +5KB gzip.
9. prefers-reduced-motion honoured.
10. Keyboard: full traversal, visible focus, skip link, modal trap + ESC.
11. **frontend coverage ≥97.5% line / ≥88% branch** — tests updated not deleted.
12. Non-negotiables intact (§4 matrix for the pages in the wave).
13. /team-review + Boss **SHIP**.

**Wave-specific additions:**
- **A:** hero copy verbatim; privacy promise on Landing + both Browse; detail
  pages handle-only + login-gate on anon apply/interview; Auth Card on Login/
  Register/Forgot/Reset; commission 50%/SGD prominence unchanged.
- **B:** personal information-contract test extended to Favourites/Applicants/Invitations/
  Browse Requests + match cards; Contact Reveal Card only on ACTIVE; availability grid
  matches design-tokens md §8; optimistic toggles/hearts preserved.
- **C:** purple shell present & token-sourced (§5.4); admin contact panel uses
  admin Contact Panel tokens; service-level agreement + commission badges from §1.4/§1.5; super-admin
  protected; M16 forced-password gate behaviour preserved; modal close-paths fully
  covered.
- **D:** all info pages Container width is prose (≤68ch); FAQ accordion native
  ; legal pages share Legal Page; Rate Guide SGD prominence.

### 6.2 Verification checklist (run per page before claiming done)


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 7. Acceptance criteria (whole programme)

The all-pages redesign is "done" only when **all 49 routes** (§1) pass §6.1 + the
per-page §6.2 checklist, AND:

1. Every page is a composition of the §2.2 layout primitives — the codebase has a
   **system**, not 49 bespoke layouts; the audit grep is clean repo-wide.
2. No locked hex/font changed (design-tokens md §1.1/§2.1 diff empty);
   status/admin colours token-sourced (incl. the §5.4 admin-shell reconciliation).
3. Every §4 non-negotiable verified on every page it touches.
4. frontend coverage gate green at ≥97.5% line / ≥88% branch across the whole suite,
   tests updated not deleted.
5. CWV budget met on all public + first-authenticated-paint pages.
6. Every wave (A/B/C/D) passed /team-review + Boss **SHIP**, conditions fixed in
   place and re-verified.

---

## 8. Open decisions (for team-review)

1. **Layout-primitive home** — the private implementation folder (alongside the existing
   primitives) vs a new the private implementation folder for the page-level ones
   (Page Header/Section/Container). Recommendation: page-level → layout/,
   atomic (Stat Card/Status Timeline) → ui/. Frontend Expert to rule.
2. **Data List ↔ Data Table responsive split** — one primitive that stacks to cards
   < md (recommended, less surface area) vs two separate primitives. Affects test
   surface and the coverage gate.
3. **Dashboard list density** — cards-on-mobile / table-on-desktop (recommended)
   vs cards everywhere (simpler, more scroll on desktop). user experience call.
4. **FAQ accordion** — native  (zero-dep, recommended) vs a controlled
   React accordion (animation control, +state/+tests). Frontend Expert.
5. **Admin shell tokens** — add an admin.* colour group to tailwind config implementation reference
   now (recommended, kills the bg- #3 B1 F6 B/bg-gray-50 magic strings) vs defer
   to Wave C. DBA/No-Magic-Strings owner.
6. **Analytics chart** — hand-authored SVG (recommended, zero-dep) vs a tiny
   tree-shaken chart lib (bundle cost). Frontend Expert + Data Engineer.
7. **Wave D position** — info pages last (current) vs folded into Wave A (they are
   public). Recommendation: keep last — low-traffic, low-risk, frees Wave A to
   focus on conversion surfaces.
8. **Create Request/Create Listing** — chunked Form Layout on one screen
   (recommended) vs a true multi-step wizard (heavier, more state/tests).

---

### Reusable lesson (proposed for SHARED-LESSONS md — not edited here)

**2026-06-03 — A redesigned page must compose shared layout primitives, not
hand-roll its own chrome (uiux-designer + frontend-expert).** Phase-0 wired the
*token* scale, but pages still hand-rolled headers (text- 36px + text-sand
sub), spacing, and bare-<Spinner> loading per page — so even on-token pages drift
and reinvent. **Rule:** after the tokens are wired, the next system layer is a small
set of composable **layout primitives** (Page Header / Section / Container / Card Grid
/ Form Layout / Data List / Empty State / Tab Nav / Auth Card); every page redesign is a
*composition* of these, and "done" includes a grep proving no page reintroduced
bespoke text, text-sand-on-body, or bare-spinner full-page loads. A system is
primitives consumed, verified by grep — not a palette on paper.

---

## 9. Team-review + Boss verdict (2026-06-03) — ✅ SHIP IT (proceed to Waves B→C→D)
18-agent review + Boss. Spec approved to implement; conditions folded in:
1. **Primitive-first with tests** — each new §2.2 layout primitive (Page Header/Section/Container/Card Grid/
   Form Layout/Data List/Tab Nav/Auth Card/Article Layout/Stat Card/Status Timeline) ships with its OWN unit test in
   the wave it's introduced. **Data List MUST be tested at both <md (cards) and ≥md (table)** breakpoints.
2. **§5.4 admin tokens** — add an admin.* colour group to tailwind config implementation reference (mirroring how status.* was
   added in Phase-0) and make Admin Layout consume it; the purple *intent* is unchanged, only the source.
   Verified defect: Admin Layout implementation reference bg-gray-50, 45 bg- #3 B1 F6 B.
3. **Coverage discipline** — per wave: update tests (never delete assertions), re-run the personal information contract tests,
   keep the v8 coverage summary ≥97.5% line / ≥88% branch.
4. **No feature-creep** — redesign is visual/structural only; no behaviour or application programming interface change rides in.
Sequence: **Wave B (tutee+tutor dashboards) → Wave C (admin, purple) → Wave D (info)**, each gate-verified by the
orchestrator + Boss-signed. Open decisions §8 resolved within their relevant wave.

**Later disclosure amendment:** [INT-125/129](../architecture/int-125-batch-closeout-spec.md) keeps participant
reveal controls on ACTIVE and COMPLETED indefinitely; match-PENDING stays hidden in the frontend. It does not widen
unrelated ACTIVE-only sections or public/pre-assignment disclosure.
