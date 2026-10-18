# InTuition SG — Engineering Roadmap

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

**Last updated:** 2026-06-04 (Singapore time)
**Owner:** Engineering
**Audience:** Engineers picking up tickets. Not investors.

> ## 🔎 2026-06-04 — Progress update (user experience redesign + admin/tutee user experience wave + lifecycle)
>
> Source of truth = code + issues/README md. All of the below is **DEV-ONLY** — none is in prod
> (see **PROD-DEPLOY-OWED**; the dev↔prod gap now spans migrations 0033→0039).
>
> **Shipped to DEV since 2026-06-03 (team-reviewed → Boss SHIP IT each; gates re-verified by the orchestrator):**
> - **INT-049 — all-pages user interface redesign** + layout-primitive system + **dashboard tabs moved into the top header**
>   after login (sidebars retired). frontend ~98% line / ~92% branch.
> - **2026-06-03 feature wave** (logged below): INT-034/039/044/025/045/043 — schema head **0037**.
> - **INT-053–059 wave (2026-06-04):** INT-056 refresh-logout **Sev-1** fix; INT-054 auth/Register width;
>   INT-055 qualifications picker; INT-057 tutee "Find Tutors" nav; INT-058 tutee applicants link;
>   INT-059 reviews→Profile; INT-060 flaky-test fix; **INT-053** full-stack admin match/interview tables
>   show **tutor↔tutee + job, clickable** (data transfer object @Entity Graph, no N+1).
> - **INT-062 (2026-06-04):** withdraw/close a **live job** (the authorized owner or staff action for closing the selected post) →
>   cascade-close applications + invitations to a visible **CLOSED "job no longer available"** status
>   (rows stay listed). Migrations **0038** + **0039** (no-op); schema head now **0039**.
> - **INT-065 (2026-06-04):** admin moderation — **block/unblock** tutor/tutee + **delete posting** +
>   **read operation for admin users** (Admin Moderation Controller); immediate **Redis kill-flag** enforcement (blocked
>   signed session token rejected next request → 401) + login **ACCOUNT BLOCKED** 403 backstop. Migrations **0040**+**0041**;
>   schema head now **0041**. Follow-ups **INT-066/INT-067** filed.
>
> **Tracker reconciled (2026-06-04, code-verified):** the issues tracker had ~11 stale-OPEN rows for work
> that's actually BUILT (INT-034/039/043/044/045 06-03 wave; INT-053–059 this session) or RESOLVED (INT-046/051
> coverage — now frontend 98.36/92.88, backend met). The **true open backlog is small** — see issues/README md top banner.
>
> **Genuinely OPEN backlog (code-verified):** **INT-064 (🔴 S1)** continuous integration: backend+frontend ci implementation reference wired+pushed but the GitHub
> run is **unconfirmed** (no gh) + the multi-repo regression/e2e continuous integration is still a follow-up; **INT-063 (S2)** admin
> close-job discoverable surface; **INT-050 (S2)** idempotency rule review; **INT-047 (S2)** local Docker Build Kit
> stall; **INT-061 (S3)** admin "Job" column blank (job Title null → frontend fallback); **INT-060 (S3)** flaky
> Tutee Profile test. That's the whole real backlog — mostly polish + the idempotency review; **no feature wave is
> pending.**
>
> **Still owed for prod:** all of the above → **PROD-DEPLOY-OWED** (migrate-then-boot 0033→0039).
> **Do NOT prod-deploy until INT-064 (continuous integration) actually runs the integration gate somewhere.**

> ## 🔎 2026-06-02 — Full status review (post INT bug-fix batch + Hunter run)
>
> The whole backend regression-lab + frontend suite were run green-to-completion for the first time in a while.
> Source of truth = code; details in issues/README md + issues/BATCH3-REMEDIATION-PLAN md.
>
> **Shipped to DEV this pass (founder-reported defects INT-001…014):** "New to InTuition?" link
> (INT-001), forgot-password email now carries a reset deep-link (INT-006), tutor-detail crash fixed
> = the INT-003 regression (INT-009), Nav About removed + role-gated Admin entry (INT-010), pre-login
> filter gating + **principal-gated** anonymous cap on /public/tutors (INT-012), 30-min session expiry period
> (INT-013), dev runtime pinned to **Node 20** + the documented operational check clears the vitest hang (INT-014). Team-
> reviewed + code-reviewed; new unit/Vitest tests green; stack reseeded + redeployed; live-smoked.
>
> **Infra defects fixed (were blocking the entire regression-lab suite — NOT from this batch):**
> INT-018 seed-password drift (Seed Users REDACTED password→REDACTED password); INT-019 Job Application Repository
> JPQL inline enum literals → Hibernate cast Job Application Status (absent PG type) → 500 on
> match-create/assign/cancel → bound the status set as a parameter. These two recovered the suite (16→0
> batch-caused failures).
>
> **NEW pre-existing defects found by Hunter — must clear before launch (see BATCH3-REMEDIATION-PLAN):**
> - **INT-016 (🔴 Sev-1):** Lazy Initialization Exception (Tutor Post proxy, no Session) → **500 on
>   match-cancel and contact-reveal** (core admin flows). open-in-view is no; fetch-join in-tx.
> - **INT-015 (🟠 Sev-2):** owner cannot delete a REJECTED post — soft-delete sets CLOSED, database trigger
>   forbids terminal→CLOSED → 500.
> - **INT-017 (🟠 Sev-2):** seeded tutor blocked from applying (profile-completeness gate / seed data).
> - **INT-020 / INT-021 (Sev-3, test-infra):** host vitest full-suite worker-pool deadlock (passes in
>   chunks + Docker build); Applicants test implementation reference choose-flow query drift.
>
> **Still prod-affecting & owed (move to PROD-DEPLOY-OWED on next prod access):** INT-002 (stale-cookie
> 401 lockout) + INT-003 (public personal information leak) were FIXED-DEV in an earlier pass; INT-009 (tutor-detail
> crash) was fixed in **this** pass. All three affect prod and must ship there. **Carry-forward config:**
> set APP FRONTEND BASE URL for any non-local deploy (INT-006).

## Status Legend

| Status | Meaning |
| --- | --- |
| Planned | Scoped, not started |
| In Progress | Active sprint |
| Blocked | Waiting on dependency or decision |
| Shipped | Merged to main, deployed |

## 🚀 PROD-DEPLOY-OWED — fixes validated on DEV, not yet shipped to prod

> **Why this exists:** per domain/dev-workflow md Amendment **A3** (adopted + Boss-cleared 2026-05-30),
> a change is DONE-DEV when the regression lab is green on the **dev** stack and team review is
> cleared. Any change that fixes a **prod-affecting** defect must *also* land here so the fix can
> never silently rot on dev while the live bug persists.
>
> **Rules:** an entry is **never deleted — only moved to ✅ DEPLOYED** after the founder confirms the
> prod deploy. Each entry carries the migrate-before-boot ordering, the prod schema-check command,
> and a named owner.
>
> **Review cadence:** this register is **opened and reconciled the moment prod access is next
> available** — it is the first thing checked before any other prod work.

| # | Change | Severity | Prod status | Owner | Deploy notes |
|---|--------|----------|-------------|-------|--------------|
| PDO-1 | **audit log details jsonb→text** — Liquibase 0024-audit-log-details-to-text implementation reference + Audit Log entity column Definition is text. Without it, **every admin mutation 500s** (approve/reject post, match create/activate/cancel, commission pay, interview ops): the entity binds details as text/varchar but the column is jsonb, so the append records fails at commit and rolls back the whole mutation. | 🔴 Sev-1 (total admin outage) | **PRESUMED-LIVE in prod — VERIFY FIRST.** Inferred from shared migration history; not confirmed (no prod access as of 2026-05-30). Fixed + verified on dev (full regression 93/93 green). | Founder | **1. Verify first:** \d audit log on prod — confirm details is jsonb; if already text, prod was never broken → close N/A. **2. Migrate-before-boot:** run Liquibase 0024 (Railway release phase) **before** the new jar boots — under ddl-auto is validate the text entity crash-loops against a jsonb column. **3. Post-deploy:** run the admin chain suite vs prod/staging as a smoke gate. **4. Separately owed:** admin-endpoint 5xx alert (this outage was invisible — no paging). |
| PDO-2 | **Password reset did not clear the forced-change gate** — Auth Service reset Password called update Password Hash (which sets hash + clears failed login attempts/locked until) but **left password change required untouched**. So an admin with the gate set (seeded REDACTED email, or any super-admin-assigned password) resets via one-time passcode, logs in with the new password — login returns **200** — but the frontend Admin Layout intercepts password Change Required is yes and **forces them onto the change-password screen**, i e. "reset, log in with new password, still can't get in." Fix: reset now calls the **existing** update Password Hash And Clear Change Required (same method Profile Service change Password uses) — clears hash + lockout + gate. **Pure code fix — no migration, no new method.** | 🟠 Sev-2 (admin self-service recovery broken — gated admin bounced after a correct reset) | **PRESUMED-LIVE in prod** (same code path). Fixed + verified on dev. | Founder | **1.** No schema change — plain app deploy (jar). **2.** Post-deploy smoke: admin forgot→reset using one-time passcode→login reaches the dashboard with password Change Required is no (not bounced to forced-change). **3. Coverage gap owed:** the regression lab does NOT cover forgot→reset→login — this bug escaped because no test exercised it. Closing it black-box needs a the profile operation reg-band one-time passcode-issue test endpoint (plaintext one-time passcode only ever exists in the email; the database stores only otp Hash), then a Password Reset Journey IT asserting a gated admin is NOT forced to change after reset. That test surface is itself a deliverable requiring lab-first + full team review. |
| PDO-3 | **M18 Assignment Lifecycle & Free Marketplace (backend core)** — migrations 0025–0029 (SUPERSEDED enum; tutor profiles canonical-identity cols incl handle+commitment+rate+area; backfill; matches source application id+match completed at; tutee posts version @Version; tutor selection requests table) + free-market apply (both TUTOR NO ACTIVE POST guards removed, profile-completeness gate), atomic assign (winner→MATCHED / others→SUPERSEDED / job→MATCHED / optimistic-lock→409 / after-commit notifications / cancel→reopen), 3 new endpoints (read operation for tutee posts id applicants personal information-free, POST .../select, read operation for matches id contact). | 🟢 New feature (not a prod-bug fix; listed for the deploy-ordering record) | **DEV-ONLY — DONE-DEV 2026-06-01** (team-reviewed full debate + Boss SHIP IT; 1087 tests 0F/0E; JaCoCo 99.18% line / 98.35% branch — excludes verified honest, IT-only harness; migrations applied + verified on live dev database, backend HEALTHY). NOT in prod. | Founder | **1. Migrate-before-boot:** run Liquibase 0025–0029 (Railway release phase) **before** the new jar boots — ddl-auto is validate will crash-loop otherwise (esp. 0028c @Version + the location region named-enum col). 0025 (ALTER TYPE ADD VALUE) is run In Transaction is no — ensure it commits before any code references SUPERSEDED. **2. Backfill** 0027/0027b populate profile rate/area + provisional handle from most-recent post / display name — verify post-migrate. **3. Post-deploy smoke:** free-market apply (no listing→201/200), applicant view personal information-free, choose-tutor→admin queue, assign→supersede, contact-reveal gated. **4. Owed (Boss conditions, track in next passes):** (a) test distinguishing [REDACTED PASSWORD] from eager-status-409; (b) tighten handle phone regex to catch +65/spaced forms; (c) distinct in-app Notification Type category via a future 0030 changeset. **5. frontend — admin assignment-lifecycle surfaces BUILT (2026-06-01):** §G2 Interviews&Selections (selection-request queue + assign), §G3 Matching rewrite (Pending Selections + Match Lifecycle: activate / mark-paid / cancel→reopen), §G4 stale-commission tab. Required a backend contract fix shipped with it — Admin Selection Request Response gains nullable **tutor Post Id** (batch-loaded from the applicant's application in Tutor Selection Service list Open Requests) so the frontend assigns via submit operation for admin matches; the prior the private implementation folder Queue payload sent tutor User Id (would 400) — corrected. Spec: architecture/admin-matching-fe-spec md (team-reviewed + Boss SEND BACK resolved). Tests: Matching 22/22, backend 1142 pass (incl. new tutor Post Id batch-load + personal information-contract update). **Deploy note:** the tutor Post Id enrichment is a backend change — ships with the 0025–0029 M18 release. The rest of the M18 frontend (tutor profile §A, cards §B, apply §C, my-applications §D, parent applicants §E, contact reveal §F) was built in prior passes. |

## Current Baseline (as of 2026-05-30)

- **Backend:** Spring Boot 3.3 / Java 21, **30 Liquibase migrations (0001–0030**; 0017=add-cancelled-invoice-status, 0018=create-job-applications, 0019=tutor-showcase, 0020=admin-management, 0021=interviews-tutee-post, 0023=public-job-requests-showcase (M14b), 0024=audit-log-details-to-text, **0025–0029=M18 assignment-lifecycle/free-market (superseded-status, tutor-profile canonical identity, backfill, assign-transition support, tutor-selection-requests), 0030=interviews-listingless-applicant, **0031=allow-post-soft-delete-close (INT-015)**), JaCoCo line+branch gate 97.5%. **Next free migration number is 0032** (M19 OAuth, previously slated for 0031, now takes 0032). **⚠️ Branch coverage gate was RED on M17 (0.93 vs 0.975)** — see Tier 1 #38 M17 coverage close-out.
- **Frontend:** React 19 / Vite, Vitest 97.5% line / 88% branch coverage.
- **Modules complete:**
  - M1 Auth — registration, login, multi-factor authentication (time-based one-time passcode), password reset, account deletion
  - M2 Posting — tutee requests, tutor listings, public browse
  - M3 Approval — admin review queue, approve/reject
  - M4 Matching — admin match creation, activation, cancellation
  - M5 Commission — commission payment, match completion
  - M6 Interview — interview request, scheduling, no-show tracking
  - M7 Profile — tutor/tutee profile management
  - M8 Match Dashboard — tutee/tutor views with admin contact reveal on ACTIVE
  - M9 Notification Centre — in-app bell + page + 10 notification types
  - M10 Search & Favourites — filter sidebar, heart, favourites page
  - M11 Review & Rating — post-match stars, aggregate on tutor cards
  - M12 Admin Analytics — platform metrics dashboard (verification pending — see Tier 1 #41)
  - M14a Active Tutors showcase — public pre-login strip (deployed, migration 0019)
  - M14b Recent Job Requests showcase — public anonymised open-jobs strip (deployed, migration 0023)
  - M15 Applications — tutor → job apply/withdraw (deployed, migration 0018)
  - M16 Admin Management & Super-Admin (deployed, migration 0020)
  - M17 Marketplace user experience overhaul — public Browse Assignments page (/assignments) + login-gated detail; deployed, no schema change (see Tier 1 #38–#40 fast-follows)
  - M18 Assignment Lifecycle & Free Marketplace — free-market apply, profile-as-identity, atomic assign (winner→MATCHED/others→SUPERSEDED), choose-tutor selection, contact reveal; Deployed (dev), migrations 0025–0031; **INT-016 (cancel/contact-reveal Sev-1 500) FIXED**.
  - **Feature wave (2026-06-03, deployed dev)** — INT-034 free-market assign real fix (0033), INT-039 interview timeslot negotiation (0034), INT-044 job invitations (0035), INT-025 admin POST SUBMITTED notify (0036), INT-045 analytics date-range (0037), INT-043 tutor onboarding optional; schema head now **0037** (0032 reserved M19). See [work-plan md](work-plan.md) + [issues/BATCH-WAVE-2026-06-03-built md](issues/BATCH-WAVE-2026-06-03-built.md).
  - **INT-049 all-pages user interface redesign (deployed dev)** — design tokens + layout-primitive system; every route modernized; dashboard tabs moved into the top header after login + login→dashboard first tab. frontend 98.25% line / 92.79% branch. Spec: [uiux/all-pages-redesign-spec md](uiux/all-pages-redesign-spec.md).
- **In design:** **M19 — Google Sign-up / Sign-in (OAuth2/OIDC)** — spec team-reviewed + Boss-cleared to implement behind flag (architecture/m18-google-oauth-signin md; doc rename to M19 pending). "Continue with Google" on login + register; OIDC Auth-Code+PKCE via Spring OAuth2 Login → mints the existing signed session token cookie; new Google users pick role (parent activates immediately; tutor commission gates only posting/applying); Option-B account linking (prove ownership, no silent auto-link); admins password-only (phase 1). Reserves **migration 0031** (users: auth provider varchar+require, google sub partial-unique, email verified, nullable password hash) — note 0024 is already the audit-log fix and 0025–0030 are M18, so OAuth takes the next free number; needs a Google Cloud OAuth client + client secret in Railway; feature-flagged auth google.enabled. Not yet implemented (spec only).
- **Repository layout:** monorepo at REDACTED local path containing intuition-backend, intuition-frontend, intuition-spec.

---

## Tier 1 — Required Before Launch (P0)

Hard blockers. Nothing goes to production without these.

### 1. Email Consumer Service

- **Status:** In Progress — v1 deployed on dev; hardening (10 fixes) under review. Spec: architecture/email-consumer-hardening md.
- **What is built (dev stack):** In-process a framework-managed scheduled method XREADGROUP consumer (Email Consumer Service) + Resend SDK dispatcher (Resend Email Client). Consumer group email-dispatcher. 3 templates live: forgot password, post approved, post rejected. XCLAIM-based crash recovery (PEL idle ≥ 30s). Pod-unique consumer name. HTML encoding on all params. Micrometer counters email sent/email failed + email dispatch.latency timer. Graceful startup when RESEND application programming interface KEY absent.
- **Sender:** REDACTED email (Resend sandbox). **Must switch to a verified platform domain before any production traffic.**
- **Deferred (pre-production gate):**
  - Retry counter + exponential backoff (3 attempts), then dead-letter to email-notifications-dlq *(AC3 — deferred)*
  - Custom sender domain / DKIM/SPF/DMARC alignment *(required before prod)*
  - Remaining 4 templates: interview-reminder, match-created, commission-due, review-received
  - Resend webhook for delivery/bounce/complained events
  - Template engine externalization (Thymeleaf)
- **Dependencies:** Redis (live), Resend application programming interface key (set via RESEND application programming interface KEY env var — see .env implementation reference).
- **Effort:** L (original); v1 shipped M, hardening S.
- **Acceptance criteria:**
  1. Triggering forgot-password results in an email delivered to a real inbox within 30s. ✅ (dev, Resend sandbox)
  2. Killing the consumer and restarting it does not lose un-acked messages (XCLAIM reclaim). ✅
  3. ~~SES throwing Throttling retries 3x with backoff, then DLQs.~~ **DEFERRED** — no failed-message queue, no retry counter yet. Failure stays in PEL indefinitely until manually cleared or failed-message queue is built.
  4. Metrics the email-delivery counter grouped by template and result increment on success. ✅ (email sent, email failed counters live)
  5. ~~All 7 templates render without unresolved placeholders.~~ **PARTIAL** — 3 of 7 templates implemented.

### 2. Security Headers (CSP, HSTS, X-Frame-Options)

- **Status:** Planned
- **Why:** Current Security Config implementation reference sets no response headers. Pen-testers and Lighthouse will both flag this. CSP is the most effective XSS mitigation we have given React renders user-controlled tutor bios.
- **Scope:**
  - Add headers via Spring Security's the headers operation DSL.
  - HSTS with a one-year transport-security policy covering subdomains and requesting preload.
  - CSP: a proposed browser policy limiting resource origins and preventing the page being embedded by another site.
  - X-Frame-Options: DENY, X-Content-Type-Options: nosniff, Referrer-Policy: strict-origin-when-cross-origin, Permissions-Policy: geolocation= , microphone= , camera is .
- **Dependencies:** Confirm Tailwind compiled CSS (no runtime inline ), confirm no inline inline script blocks blocks in React build output.
- **Effort:** S
- **Implementation outline:**
  1. Extend Security Config with the headers operation.
  2. Audit Vite build output for inline scripts (vite-plugin-csp if needed; otherwise use nonces).
  3. Stripe implementation reference requires script-src https://js stripe.com and frame-src https://js stripe.com. Add when item #4 lands.
  4. Test on staging with Content-Security-Policy-Report-Only first, watch reports for a week, then enforce.
- **Acceptance criteria:**
  1. the documented operational check shows all six headers.
  2. Mozilla Observatory grade A or better.
  3. Lighthouse "Best Practices" score 100.
  4. Zero CSP violations in production logs after 7-day report-only soak.

### 3. cross-site request forgery protection Protection

- **Status:** Planned
- **Why:** Currently the csrf operation. Cookie-based session auth without cross-site request forgery protection is a textbook vulnerability. signed session token in Authorization header is partially protective, but our refresh-token flow uses an HttpOnly cookie which is exploitable.
- **Scope:**
  - Spring Security the with http only no operation.
  - frontend axios interceptor reads XSRF-TOKEN cookie, sends as X-XSRF-TOKEN header.
  - Exempt auth login interface operation, auth register interface operation, payments webhook ** interface operation (Stripe signed).
- **Dependencies:** None.
- **Effort:** S
- **Implementation outline:**
  1. the csrf operation.
  2. Frontend api Client implementation reference interceptor: read document cookie for XSRF-TOKEN, attach to outgoing non-GET requests.
  3. Update e2e tests to handle the round-trip.
- **Acceptance criteria:**
  1. POST without X-XSRF-TOKEN header returns 403.
  2. Login → token issued → subsequent mutations succeed.
  3. Stripe webhook still verifies via signature (cross-site request forgery protection bypassed).

### 3a. Mandatory multi-factor authentication for Privileged Roles

- **Status:** Planned
- **Why:** Tutors will receive Pay Now payouts once #4 ships; admins approve posts and trigger refunds. Admin multi-factor authentication (time-based one-time passcode) exists today but is opt-in. Tutors have no multi-factor authentication at all. Account takeover on either role is direct financial loss.
- **Scope:**
  - Feature flag mfa required.roles is ADMIN,TUTOR.
  - 14-day grace period from rollout date for existing users to enrol.
  - Enrolment flow user experience: QR provisioning + verify-on-first-login wizard.
  - Backup codes: 10 single-use codes, downloadable as a text file, regeneratable from settings.
  - Login enforcement: after grace period, login without multi-factor authentication on privileged roles is blocked at the auth layer.
- **Dependencies:** Existing time-based one-time passcode infrastructure from M1 Auth.
- **Effort:** S
- **Implementation outline:**
  1. Extend Mfa Service to issue and verify 10 hashed backup codes per user.
  2. users mfa grace period ends at column to track per-user grace deadline.
  3. Auth Filter reads mfa required.roles; if role matches and mfa Enrolled no and the enrolment grace period has ended, return 401 with multi-factor authentication ENROLMENT REQUIRED.
  4. frontend: forced-enrolment wizard intercepts post-login on privileged roles.
  5. Settings page exposes backup-code download + regenerate.
- **Acceptance criteria:**
  1. Admin login without multi-factor authentication after grace period returns 401 multi-factor authentication ENROLMENT REQUIRED.
  2. Tutor login without multi-factor authentication after grace period returns 401 multi-factor authentication ENROLMENT REQUIRED.
  3. Backup code consumes on use; second attempt with same code rejected.
  4. Regenerating backup codes invalidates the previous batch atomically.

### 3b. Refresh Token Rotation

- **Status:** Planned
- **Why:** Carry-over from Solution Architect P0-5. Current signed session token expiry period is 24h with no refresh path — every session expires hard, users re-auth daily, and revoked tokens stay valid until they expire. SA flagged this as a P0 launch blocker.
- **Scope:**
  - Short-lived access token (15min) + long-lived refresh token (30d rolling).
  - Refresh tokens stored server-side in Redis (refresh token value (withheld) → user + family + expiry).
  - Refresh-token family detection: reuse of a rotated token revokes the entire family (theft signal).
  - New endpoint submit operation for auth refresh reading refresh token from HttpOnly cookie, returning new access + new refresh.
  - Logout revokes refresh token family in Redis.
- **Dependencies:** Redis (live), #3 cross-site request forgery protection (refresh endpoint is mutating).
- **Effort:** M
- **Implementation outline:**
  1. Split current signed session token into access Jwt (15min) and refresh Jwt (30d), separate HttpOnly cookies.
  2. the rotate operation: validate, mark used, issue new pair, store family id linkage.
  3. Reuse detection: if a used is yes refresh token is presented again, delete all refresh token value (withheld) with matching family id and force re-auth.
  4. frontend axios interceptor: on 401 from access token, call /auth/refresh once; on failure, redirect to login.
  5. Logout path revokes by family.
- **Acceptance criteria:**
  1. Access token expires after 15min; frontend silently refreshes once and retries the request.
  2. Refresh token expires after 30d of inactivity; rotates on every use.
  3. Reusing a rotated refresh token revokes the whole family within the same request.
  4. Logout clears both cookies and revokes the family in Redis.

### 4a. Stripe Test-Mode Integration

- **Status:** Planned
- **Why:** Commission is the revenue model. Card payments are the broadest-coverage rail and the lowest-risk path to validate the end-to-end flow. Ships as test-mode only — no live keys touch the codebase until #4c.
- **Scope:**
  - Stripe Checkout (test mode) for card payments only.
  - New Payment Service orchestrating intents, confirmation, webhook handling.
  - Liquibase migration for payment intents table.
  - Webhook endpoint submit operation for payments webhook stripe with signature verification.
  - Idempotency keys on all create-intent calls.
- **Dependencies:** Stripe account created, **business verification kicked off before Sprint 2** (5–10 business-day lead time; without it #4c cannot land).
- **Effort:** L
- **Implementation outline:**
  1. Liquibase 0022-add-payment-intents implementation reference (0001–0021 are taken — see baseline):


Implementation example omitted. This public edition retains the written requirements in this section.


  2. the create checkout session operation → returns Stripe Checkout URL.
  3. Webhook handler: verify signature, lookup by provider intent id, transition commission state on checkout session.completed.
  4. Frontend Commission Payment Page implementation reference → redirect to Stripe Checkout, return to /payments/return.
- **Acceptance criteria:**
  1. Tutor completes Stripe Checkout (test card 4242) → commission status flips to PAID.
  2. Webhook replay with same event ID is a no-op (idempotency).
  3. Failed payment surfaces a clear retry CTA, does not lock the commission.
  4. All payment events are recorded in audit log.

### 4b. Pay Now Integration via Stripe

- **Status:** Planned
- **Why:** SG market expects Pay Now QR. Stripe supports Pay Now as a payment method in SG; Hit Pay is fallback if Stripe Pay Now has capability gaps.
- **Scope:**
  - Pay Now payment method added to existing Stripe Checkout session.
  - provider enum on payment intents extended to STRIPE PAYNOW.
  - MAS regulatory check on Pay Now merchant rules.
- **Dependencies:** #4a complete; Stripe business verification complete; MAS check passed.
- **Effort:** M
- **Implementation outline:**
  1. Extend Stripe checkout session creation with payment method types= card, paynow.
  2. Webhook handler covers Pay Now-specific events (asynchronous confirmation can take minutes).
  3. frontend shows QR code via Stripe implementation reference inline element.
- **Acceptance criteria:**
  1. Tutor scans Pay Now QR → on confirmation, commission state transitions to PAID.
  2. Pay Now timeout (15min) surfaces a clear retry CTA.
  3. Audit log distinguishes STRIPE CARD vs STRIPE PAYNOW.

### 4c. Live-Mode Cutover + Reconciliation

- **Status:** Planned
- **Why:** Test-mode flows do not collect real money. Live cutover requires verified Stripe account, daily reconciliation against bank settlement, and a documented incident playbook.
- **Scope:**
  - Live Stripe keys rotated in secret manager (Amazon Web Services Secrets Manager or Fly secrets).
  - Daily reconciliation job comparing payment intents PAID set vs Stripe's Balance Transaction application programming interface.
  - Discrepancy alert to Pager Duty if reconciliation drift > 0.
  - Runbook for refund, chargeback, and stuck-intent recovery.
- **Dependencies:** #4a, #4b, Stripe business verification approved, #6a Staging Environment (for live-mode smoke test in isolation).
- **Effort:** M
- **Implementation outline:**
  1. Reconciliation Quartz job runs at 02:00 Singapore time daily.
  2. Materialised view payment reconciliation drift for ops dashboard.
  3. Stripe Sigma query for monthly settlement check.
  4. Runbook in intuition-spec/runbooks/payments md.
- **Acceptance criteria:**
  1. First live transaction completes end-to-end (admin smoke test, $1).
  2. Reconciliation job runs for 7 consecutive nights with zero drift.
  3. Refund flow exercised in live mode against the smoke-test transaction.

### 5. File Upload (Cloudinary)

- **Status:** Planned
- **Why:** Tutors need to upload profile photos and qualification certificates; tutees need profile photos for trust signal. Tuition agencies expect to see NIE certs, NRIC redacted scans. We cannot launch with text-only profiles.
- **Scope:**
  - Tutee profile photo, tutor profile photo, tutor qualification certificate using s.
  - Pre-signed URL pattern (browser → Cloudinary direct, server only signs).
  - media uploads table tracking ownership, type, status.
  - Virus scan: Cloudinary built-in content moderation + malware scan.
  - Image processing: resize profile photos to 400x400 and 1200x1200 derivatives.
- **Dependencies:** Cloudinary account. Provider choice ratified by **ADR-010** in the Decision Log.
- **Effort:** L
- **Implementation outline:**
  1. Liquibase 0023-add-media-uploads implementation reference (next free after 0022-add-payment-intents implementation reference):


Implementation example omitted. This public edition retains the written requirements in this section.


  2. submit operation for media signed-upload → returns Cloudinary signed params (signature, timestamp, folder).
  3. Browser uploads directly to Cloudinary.
  4. submit operation for media confirm → server fetches metadata, marks status READY.
  5. Reject MIME types outside image/jpeg, image/png, application/pdf. Reject size > 10MB.
- **Acceptance criteria:**
  1. Tutor uploads a JPEG → profile photo renders on their public profile.
  2. Tutor uploads a 12MB file → rejected with clear error.
  3. Tutor uploads evil exe renamed to evil png → Cloudinary's content scan rejects.
  4. Deleting an upload removes both database row and remote asset.

### 6. continuous integration/continuous deployment Pipeline

- **Status:** Planned
- **Why:** Every merge to main today is a manual local build. We will ship a regression within two weeks of going live without automated checks on pull requests.
- **Scope:**
  - ci implementation reference — runs on every pull request. Backend: the documented operational check with JaCoCo gate. Frontend: the documented operational check.
  - cd implementation reference — runs on push to main. Deploys to staging.
  - Cache Maven and npm dependencies. PostgreSQL service for backend tests.
- **Dependencies:** Staging environment provisioning (Dev Ops track).
- **Effort:** M
- **Implementation outline:**
  1. continuous integration: matrix of backend, frontend. Java 21 (Temurin), Node 22.
  2. Backend job spins up postgres service container. Sets JAVA HOME correctly (Java 26 system default breaks Lombok — pin to 21).
  3. Upload JaCoCo report as artifact, post coverage delta as pull request comment.
  4. continuous deployment: build Docker images, push to container image registry (or Fly io registry), wrangler deploy or flyctl deploy.
  5. Required status checks on main branch.
- **Acceptance criteria:**
  1. pull request with failing test cannot merge.
  2. pull request with coverage < 97.5% on changed files cannot merge.
  3. Merge to main produces a staging deploy within 10 minutes.
  4. Build cache reduces full continuous integration run to under 4 minutes warm.

### 6a. Staging Environment Provisioning

- **Status:** Planned
- **Why:** #6 continuous integration/continuous deployment's continuous deployment job needs somewhere to deploy. No staging today means every code path is first exercised in production. Hard prerequisite for the continuous deployment half of #6 and for #4c live-mode smoke testing.
- **Scope:**
  - Hosting region selection (Fly io sin or Amazon Web Services ap-southeast-1).
  - Postgres 16 + Redis 7 provisioned with separate credentials from production.
  - Secret management via Amazon Web Services Secrets Manager or Fly secrets.
  - DNS: staging example.com with TLS cert (Let's Encrypt or provider-managed).
  - Seed dataset: anonymised sample of 100 tutors, 100 tutees, 50 posts.
- **Dependencies:** Cloud provider account, DNS access for example com.
- **Effort:** M
- **Implementation outline:**
  1. Provider decision logged as ADR (Fly io vs Amazon Web Services).
  2. Terraform or Fly fly toml for infra-as-code; checked into intuition-infra/ (new repo or intuition-spec/infra/).
  3. Staging Postgres uses Timescale database-compatible image to ease future Post GIS / Timescale upgrade.
  4. Secrets pulled at boot via provider SDK; no .env implementation reference files on disk.
  5. Smoke-test endpoint /actuator/health/liveness reachable from public over TLS.
- **Acceptance criteria:**
  1. https://staging example.com/actuator/health returns 200 with all components UP.
  2. continuous deployment pipeline from #6 deploys a merge to main to staging within 10 minutes.
  3. Staging database is isolated from production (separate VPC / network).
  4. Rolling deploy completes with zero 5xx on a steady 1 rps load test.

### 7. Reschedule Interview

- **Status:** Planned
- **Why:** Today interviews can only be cancelled or completed. Real life has reschedules. Currently the admin team manually deletes + recreates, losing the audit trail.
- **Scope:**
  - update operation for admin interviews id reschedule accepting the proposed interview time and reason.
  - Service guard: only when status SCHEDULED. Reject if within 4 hours of original slot (configurable).
  - Emit INTERVIEW RESCHEDULED notification to tutor and tutee.
  - Emit a new email-stream message for both parties.
- **Dependencies:** #1 Email Consumer for the email notifications to actually send.
- **Effort:** S
- **Implementation outline:**
  1. Extend the reschedule operation.
  2. Append to audit log with old Scheduled At and new Scheduled At.
  3. Admin user interface: button on Interview Detail Page, modal with datepicker.
  4. Unit + integration tests including guard conditions.
- **Acceptance criteria:**
  1. Reschedule a SCHEDULED interview → both parties notified, status remains SCHEDULED.
  2. Reschedule a COMPLETED interview → 409 Conflict.
  3. Reschedule within 4-hour window → 400 with clear error.
  4. Audit log shows the change with actor user ID.

### 7a. Personal Data Protection Act Data Export & Erasure

- **Status:** Planned
- **Why:** Singapore Personal Data Protection Act grants users the rights to data access, portability, and erasure. Launching without these is direct legal exposure under the Personal Data Protection Act and adjacent regimes (and a hard blocker for any future EU traffic under GDPR).
- **Scope:**
  - User-initiated data export from the profile page: produces a JSON + CSV bundle of every entity owned by the user (posts, matches, interviews, reviews, notifications, audit-log entries scoped to them). Emailed link valid 7 days, single-use.
  - Admin-initiated hard erasure: removes personal information from users, profiles, posts, messages; keeps anonymised tombstone rows in audit log for forensic continuity.
  - Documented retention schedule per entity type, published as a public policy doc.
- **Dependencies:** #1 Email Consumer (for the export-ready email), #5 File Upload (for the bundle hosting on Cloudinary signed URL).
- **Effort:** M
- **Implementation outline:**
  1. submit operation for me data-export enqueues a job; worker assembles bundle, uploads to a private bucket, emails signed link.
  2. submit operation for admin users id erase performs cascading personal information redaction inside a transaction; replaces names/emails/phone with REDACTED:`<sha256>` tombstones in audit log.
  3. Retention policy doc at intuition-spec/policy/retention md enumerating per-entity expiry period.
  4. Quartz job runs daily to expire data per the retention schedule (e g. soft-deleted posts purged after 90d).
- **Acceptance criteria:**
  1. Any authenticated user can request export from profile → email arrives within 5 minutes with a working signed link.
  2. Export bundle includes posts, matches, interviews, reviews, notifications.
  3. Admin erase removes all personal information fields but retains an anonymised audit row referencing the original user unique identifier.
  4. Retention policy is published at /policy/retention and matches what the daily job actually does.

### 23. Notification Email Delivery (promoted from Tier 3)

- **Status:** Planned
- **Why:** Without it, the email consumer (#1) only handles password reset and interview reminders. Match activation, commission due, and other high-stakes events stay in-app only. Users who don't open the app miss the moments that matter most.
- **Scope:** Notification Service enqueues an email-stream message when type is in the notification events for match activation, interview scheduling, interview rescheduling and commission due. Honours email opt out flag.
- **Effort:** S
- **Dependencies:** #1 Email Consumer.
- **Acceptance criteria:**
  1. Match activation triggers both in-app notification and email.
  2. User-set email opt out suppresses the email but not the in-app entry.
  3. Each of the four event types covered by an integration test.

### 34. Database Backups + DR Runbook (promoted from Tier 5)

- **Status:** Planned
- **Why:** Single-node Postgres without backups is an unrecoverable risk. Critical for production. Promoted to Tier 1 — cannot launch without it.
- **Scope:**
  - pg dump nightly to S3 with 30-day daily retention.
  - Weekly full backup retained 12 months.
  - Documented PITR (point-in-time recovery) procedure using WAL archiving.
  - Quarterly restore drill exercised against the staging environment.
- **Dependencies:** #6a Staging Environment (for the restore drill).
- **Effort:** S
- **Implementation outline:**
  1. Nightly job runs in production: pg dump | gzip | aws s3 cp to versioned, encrypted S3 bucket.
  2. WAL archiving configured for PITR window (≥ 24h).
  3. Restore runbook at intuition-spec/runbooks/db-restore md.
  4. Quarterly drill on the calendar; outcome logged to intuition-spec/dr-drills/.
- **Acceptance criteria:**
  1. Nightly backup completes in < 5min and lands in S3.
  2. Quarterly restore drill succeeds in < 1h on staging.
  3. RPO ≤ 24h and RTO ≤ 4h documented and tested.

### 38. M17 Coverage Close-Out — Branch Gate is RED

- **Status:** In Progress
- **Why:** M17 shipped public Browse Assignments + login-gated detail, but backend JaCoCo **branch coverage is RED (0.93 vs 0.975 required)**. The gate is the only honest signal that the new auth-narrowing and rate-limit branches are exercised; it is failing now, so the coverage promise in the Cross-Cutting table and ADR-005 is not being met for the M17 code.
- **Scope:**
  - Public Browse Controller — tests for the assignments interface operation paths: filter combinations, the anonymous pagination cap (page > 4 / size > 20 → rejected, never deeper data), the 60/min rate-limit branch, and the empty/degraded envelope.
  - Security Config M17 branches — the narrowed /public/tutors/** → LIST-only glob, /public/tutors/*/reviews kept public (M11), and the is authenticated operation 401-anon vs 200-authed branches on read operation for tutors id and read operation for assignments id.
  - Tutor Post Repository Impl — Testcontainers integration gap; the query branches are not exercised against a real Postgres (see #39).
- **Dependencies:** #39 (Testcontainers bump makes the local/continuous integration signal trustworthy).
- **Effort:** S
- **Acceptance criteria:**
  1. the documented operational check passes the 97.5% line **and** branch gate with no new exclusions.
  2. Each M17 Security Config branch (anon 401, authed 200, reviews-public, list-glob) is covered by a test.
  3. Tutor Post Repository Impl query paths run under Testcontainers.

### 39. Testcontainers / docker-java Version Bump

- **Status:** Planned
- **Why:** The current Testcontainers / docker-java pairing does not run cleanly against the local/continuous integration Docker engine, so the integration suite (and the JaCoCo branch signal that depends on it) is not honest — gaps like Tutor Post Repository Impl (#38) go unmeasured. A version bump restores a trustworthy coverage signal locally and in continuous integration.
- **Scope:** Bump Testcontainers + docker-java to a compatible pair; confirm Postgres + Redis containers start under the local Docker engine and in the continuous integration runner (ties into #6 continuous integration/continuous deployment Docker-in-continuous integration gate).
- **Dependencies:** None to start; unblocks the honest re-run of #38.
- **Effort:** S
- **Acceptance criteria:**
  1. the documented operational check spins up Postgres + Redis via Testcontainers locally without manual workarounds.
  2. The same suite runs green in continuous integration (#6).
  3. The branch-coverage number reported locally matches continuous integration (no environment-dependent skips).

### 40. M17.1 — Marketplace user experience Minors

- **Status:** Planned
- **Why:** Three known user experience gaps from the M17 deploy that did not block ship but degrade the experience. Surfaced explicitly so they are not lost.
- **Scope:**
  - **"Post a Request" anon CTA** — label/destination: the anonymous CTA currently routes to /login; confirm copy + intent (post-login return To back to the post-a-request flow) so an anonymous poster lands where they expect.
  - **Assignment Detail Retry** — the error-state Retry should re-fetch within the SPA, not trigger a full page reload.
  - **Mobile drawer "About"** — the mobile navigation drawer is missing the "About" entry present on desktop; restore parity.
- **Dependencies:** None.
- **Effort:** S
- **Acceptance criteria:**
  1. Anonymous "Post a Request" lands the user on the post flow after login via return To, with copy matching intent.
  2. Assignment Detail Retry re-fetches in place (no full reload), verified by test.
  3. Mobile drawer lists "About"; desktop/mobile nav parity verified.

### 41. M12 Analytics Verification Close-Out

- **Status:** In Progress
- **Why:** M12 Admin Analytics is built and deployed but carries a standing "verification pending" caveat across the docs. Close it out so the dashboard numbers are trusted, or file the specific gaps.
- **Scope:** Verify the analytics dashboard metrics against the source-of-truth event/data store; document the verification result and clear the "verification pending" caveat in CLAUDE md, modules/INDEX md, and backend-api md (or convert it to a tracked defect list).
- **Dependencies:** None.
- **Effort:** S
- **Acceptance criteria:**
  1. Each dashboard metric reconciled against source data within an agreed tolerance.
  2. The "verification pending" caveat is either removed across the docs or replaced with a concrete defect list.

### 42. M-Reg — Regression Labs (frontend + backend end-to-end)

- **Status:** Spec team-reviewed + Boss-cleared to build (M-Reg-0 first)
- **Why:** Unit/slice tests mock the repo and the only real-database tests **self-skip** when Docker is absent — that blind spot let the M17 enum-500, the showcase Lazy Initialization bug, and the broken login return To reach prod / escape continuous integration. Nothing exercises full journeys against a real running stack.
- **Spec:** architecture/regression-labs-spec md (proposal: architecture/regression-labs-proposal md).
- **Scope:** Two labs — **frontend Playwright** (top-level e2e/) + **backend REST Assured + Testcontainers** (*IT implementation reference via failsafe, real Postgres/Redis); Postman/**Newman** repurposed as post-deploy smoke. Phases:
  - **M-Reg-0 (foundation, FIRST):** kill the 6 assume yes self-skips → fail-hard-in-continuous integration Testcontainers base → a **negative test proving no-Docker goes RED, not skip** → cover Tutor Post Repository Impl via a **surefire** @Spring Boot Test+TC test → green the branch gate ≥0.975 (this is also Tier 1 #38's close-out path).
  - **M-Reg-1** backend lab · **M-Reg-2** frontend lab · **M-Reg-3** post-deploy prod smoke.
- **Gating:** end-to-end smoke report-only → hard-gate at <2% flake (ceiling 5% = incident); post-deploy prod-smoke = alert + railway rollback now. A true *pre-promotion* hard gate is **blocked on a Railway staging environment** (Railway has no native promotion gate) — see Tier 1 #6a Staging Environment.
- **Out of scope:** the AI "regression agent" — deferred, advisory-only (never gating) when it lands.
- **Owner:** Regression Engineer (Robin).
- **Effort:** L
- **Dependencies:** #6 continuous integration/continuous deployment; #6a Staging Environment (for the prod-smoke pre-promotion gate only); #39 Testcontainers bump (honest coverage signal).
- **Boss precondition:** M-Reg-0 must exit (gate green + self-skip killed + negative proof test) **before any lab code**; Boss re-audits at M-Reg-0 exit.

### 42a. cross-site request forgery protection rejection returns 401 instead of 403 (minor consistency — backlog)

Found 2026-05-31 by Csrf Negative IT: an authenticated state-changing request carrying valid auth +
XSRF cookies but **omitting the X-XSRF-TOKEN header** is rejected with **web request protocol 401** (UNAUTHORIZED),
not the conventional **403**. The write IS denied (no security hole), but a missing cross-site request forgery protection token
surfacing as an auth failure is misleading for application programming interface clients. Consider standardising cross-site request forgery protection rejection to
403. Low priority. Csrf Negative IT asserts denial (401/403), so it won't break when this is fixed.

### Carry-Over from Solution Architect Review

Cross-references between the SA P0 findings and the Tier 1 roadmap. None of these are net-new work; they map SA-numbered findings to the items above so engineers picking up tickets can see the audit trail.

| SA Finding | Description | Roadmap Item |
| --- | --- | --- |
| P0-1 | Pagination contract divergence (Reviews / Notifications / Interviews) | IN PROGRESS — task #29 integration-audit fixes |
| P0-2 | Email queue has no consumer | Tier 1 #1 Email Consumer |
| P0-3 | Timestamp + timezone correctness | IN PROGRESS — task #30 SA P0 fixes |
| P0-4 | Two frontend logout paths skip server invalidation | IN PROGRESS — task #30 SA P0 fixes |
| P0-5 | No refresh token; 24h signed session token | Tier 1 #3b Refresh Token Rotation (new) |
| P0-6 | No continuous integration/continuous deployment pipeline | Tier 1 #6 continuous integration/continuous deployment Pipeline |

---

## Tier 2 — Strong Differentiators (P1)

Not launch-blockers, but they materially separate us from incumbents within the first 90 days.

### 8. Google / Microsoft Calendar Sync

- **Status:** Planned
- **Why:** Tutors juggle 5–15 students; missing an interview because it lived only in our system is a churn event.
- **Scope:** OAuth2 connect flow, calendar integrations table, one-way iCal feed, two-way Google Calendar application programming interface.
- **Dependencies:** Google Cloud project, OAuth verification (4–6 weeks).
- **Effort:** L
- **Implementation outline:**
  1. Ics Export Service produces an authenticated .ics feed per user (calendar feed token .ics interface operation). Ship this first — works for both Google and Outlook subscribers.
  2. For two-way: store refresh token encrypted with key management service, sync via Google Calendar application programming interface on interview create/update/delete.
  3. Webhook from Google for external changes back into our system (optional v1.1).
- **Acceptance criteria:**
  1. Subscribing the ICS URL in Google Calendar shows all upcoming interviews.
  2. Two-way connect: creating an interview adds a Google Calendar event within 60s.
  3. Revoking access in Google deactivates the integration on next sync attempt.

### 9. Auto Meeting Links

- **Status:** Planned
- **Why:** Admins currently paste meeting links manually. This is operational toil and a security risk (link reuse).
- **Scope:** Zoom signed session token/Server-to-Server OAuth or Google Meet via Calendar application programming interface.
- **Dependencies:** #8 (Google Meet route), or Zoom developer account.
- **Effort:** M
- **Implementation outline:**
  1. On Interview Service schedule, call provider application programming interface to create unique meeting.
  2. Store meeting link, meeting id, meeting provider on interviews.
  3. Send link via email + in-app notification; never expose in public feeds.
- **Acceptance criteria:**
  1. Newly scheduled interview has a meeting link auto-populated.
  2. Cancelling an interview deletes the meeting on the provider.
  3. Reschedule keeps the same meeting link.

### 10. Pseudonymous Chat

- **Status:** Planned
- **Why:** Pre-match parties need to ask logistics questions without exchanging real names or numbers. WhatsApp leakage is our biggest privacy risk.
- **Scope:**
  - conversations and messages tables.
  - Web Socket via STOMP (Spring native) for real-time delivery; long-polling fallback.
  - Display name from a generated pseudonym (Tutor-7 G2 K, Parent-9 X1 P).
  - personal information scrubber on send: regex for SG phone numbers, common email patterns. Warn + soft-block.
- **Dependencies:** None.
- **Effort:** XL
- **Implementation outline:**
  1. Liquibase migrations for the two tables.
  2. the send operation — applies personal information scrubber, persists, broadcasts.
  3. Spring @Enable Web Socket Message Broker with /ws endpoint.
  4. frontend chat component, optimistic send, retry queue.
  5. Per-conversation rate limit (10 msg/sec/user).
- **Acceptance criteria:**
  1. Two parties can exchange messages without seeing real names.
  2. Pasting REDACTED phone triggers a warning, can be sent with override.
  3. After match is ACTIVE, conversation auto-archives.

### 11. Admin Mobile Responsive

- **Status:** Planned
- **Why:** Ops team triages from phones during off-hours. Today admin pages overflow on tablet.
- **Scope:** Refactor admin pages to be tablet-usable (≥768px). Phone is stretch.
- **Effort:** M
- **Implementation outline:** Audit each admin page with Tailwind responsive utilities; collapse multi-column layouts; convert action bars to bottom sheets on narrow viewports.
- **Acceptance criteria:**
  1. All admin pages usable at 768x1024 without horizontal scroll.
  2. Tap targets ≥ 44x44px.

### 12. Tutor Verification Badges

- **Status:** Planned
- **Why:** Trust signal. Parents repeatedly ask "is this person actually MOE-registered?"
- **Scope:** Boolean flags on Tutor Profile: moe Registered, nie Trained, nus Graduate, ntu Graduate, verified At, verified By.
- **Dependencies:** #5 File Upload (for the supporting documents).
- **Effort:** S
- **Implementation outline:**
  1. Liquibase add columns.
  2. Admin endpoint update operation for admin tutors id badges.
  3. Public profile renders badge icons.
- **Acceptance criteria:**
  1. Admin can flip badges, change reflected on public profile within request lifetime.
  2. Audit log records who set each badge and when.

### 13. Public Tutor Profile URL

- **Status:** Planned
- **Why:** Tutors want a shareable link for their socials. SEO matters for organic growth.
- **Scope:** the public tutor page addressed by its permitted public identifier route. Slug derived from display Name + 6-char unique identifier suffix.
- **Effort:** S
- **Implementation outline:**
  1. Add slug column on tutor profiles, unique, generated on first publish.
  2. Public read endpoint read operation for tutors slug returns only non-personal information fields.
  3. SSR/SSG consideration deferred — for now, client-render with React Helmet for meta tags.
- **Acceptance criteria:**
  1. Visiting /t/john-tan-7g2k shows public profile.
  2. OG tags render for link previews on WhatsApp / Telegram.
  3. Unpublished tutors return 404.

### 13a. Policy Versioning + Re-consent

- **Status:** Planned
- **Why:** Personal Data Protection Act / Terms / Privacy change over time. Without versioning we cannot prove a given user agreed to a specific version, and we have no clean re-consent flow when a policy materially changes.
- **Scope:**
  - the policy versions operation table — kind ∈ the terms, privacy and commission policy categories.
  - users last consented version map column (or join table).
  - Login interceptor: if user's last consented version for any required kind is below current, surface a modal requiring re-consent before continuing.
- **Effort:** S
- **Implementation outline:**
  1. Liquibase migration for policy versions and user policy consents.
  2. Admin user interface to publish a new version (markdown editor, effective-at picker).
  3. frontend: post-login guard renders re-consent modal; cannot dismiss without acknowledging.
- **Acceptance criteria:**
  1. Publishing a new TERMS version causes next-login re-consent modal for every active user.
  2. Consent rows are append-only — historical record of every accepted version per user is preserved.
  3. Admin can see "X% of users have accepted v3" report.

### 14. Geospatial Search (Post GIS)

- **Status:** Planned
- **Why:** "Tutors within 3km of Tampines MRT" is the most-asked search. Current haversine in app code does not scale.
- **Scope:** Enable Post GIS, add the geography operation to posts. Index. Replace search database query language.
- **Dependencies:** Confirm the Postgres provider used in #6a Staging Environment supports Post GIS before kickoff. Amazon Web Services managed relational database and Fly Postgres both support it; verify the chosen image ships with the extension.
- **Effort:** L
- **Implementation outline:**
  1. Liquibase: enabling the database's geographic-query extension
  2. Backfill location from existing latitude, longitude.
  3. GIST index. Use the st d within operation.
  4. Seed MRT station coordinates (~150 stations).
- **Acceptance criteria:**
  1. Search "within 5km of Bedok MRT" returns correctly bounded results.
  2. Query planner uses GIST index (verified via EXPLAIN).
  3. p95 search latency < 200ms at 10k posts.

### 15. Subject Taxonomy

- **Status:** Planned
- **Why:** Flat strings like "math", "Math", "Mathematics", "A Math" fragment search and reporting.
- **Scope:** Hierarchy Subject → Topic → Level. New subjects table. Migrate existing post strings.
- **Effort:** L
- **Implementation outline:**
  1. Schema: the subjects operation self-referential.
  2. Seed MOE syllabus structure (Primary / Secondary / JC / IB).
  3. Migration script to map free-text values; manual review queue for ambiguous.
  4. frontend: cascading dropdown.
- **Acceptance criteria:**
  1. New posts must select from taxonomy.
  2. Backfill leaves no orphan strings.
  3. Search filters cascade correctly (selecting "Mathematics" auto-includes "A Math", "E Math").

### 15a. Cookie Consent + Privacy Center

- **Status:** Planned
- **Why:** Personal Data Protection Act notice-and-consent expectation in Singapore. Gets the project GDPR-ready for any future EU expansion, and removes ambiguity for tracking scripts.
- **Scope:**
  - Banner on first visit with three toggles: Essential (always on), Analytics, Marketing.
  - Preference center under user settings — re-open at any time.
  - Audit of every tracking script (Plausible, Mixpanel, Sentry, whichever lands) and which consent category it sits under.
  - Reject-all option as a first-class button (no "manage" trap).
- **Effort:** S
- **Implementation outline:**
  1. Lightweight cookie-consent component, no external library if avoidable (stay within bundle budget).
  2. consent analytics / consent marketing flags in local Storage + mirrored to user preferences once authenticated.
  3. Tracking scripts conditionally injected via the consent Zustand slice.
  4. Privacy policy page links to the preference center.
- **Acceptance criteria:**
  1. First visit shows the banner with a single click to accept-all or reject-all.
  2. Rejecting analytics prevents the analytics script from loading at all (verified via Dev Tools network panel).
  3. Preference center is reachable from the footer of every page.

---

## Tier 3 — Engagement / Growth (P2)

### 16. Saved Searches + Alerts

- **Status:** Planned
- **Why:** Demand-side users (parents) return weekly; we should pull them back daily with relevant matches.
- **Scope:** saved searches table. Daily Quartz job runs each search, emails matching new posts since last run.
- **Dependencies:** #1 Email Consumer.
- **Effort:** M
- **Acceptance criteria:** Saving a search with criteria X → next day email lists only new matches.

### 17. Referral Codes

- **Status:** Planned
- **Why:** CAC is high. Word-of-mouth from tutors is our cheapest channel.
- **Scope:** the referrals operation. First-month commission discount for inviter once invitee's first match activates.
- **Effort:** M
- **Dependencies:** #4a Stripe Test-Mode Integration (commission must be billable to apply the discount).
- **Acceptance criteria:** Code applied at signup → both parties see credit applied to next commission.

### 18. Tutor Reply to Reviews

- **Status:** Planned
- **Why:** Tutors with a low review need a chance to respond publicly. Current system is one-sided.
- **Scope:** Extend reviews with tutor reply TEXT, tutor replied at TIMESTAMPTZ. One reply per review.
- **Effort:** S
- **Acceptance criteria:**
  1. Tutor can post one reply per review.
  2. Reply visible on public profile.
  3. Reply edits create an audit entry.

### 19. Group Classes

- **Status:** Planned
- **Why:** Tutors increase per-hour earnings; tuitees split cost.
- **Scope:** Extend Match to allow multiple tutee Post Ids pointing to a single tutor Post Id. Commission scales per head.
- **Effort:** L
- **Dependencies:** #15 Subject Taxonomy (level must match).
- **Acceptance criteria:**
  1. Admin can group up to 6 tutee Posts under one tutor Post.
  2. Commission billed per tutee Post, with group discount logic.
  3. Interview model adapts to multi-party.

### 20. Tutor Analytics Dashboard

- **Status:** Planned
- **Why:** Tutors want to know what's working. Today they have zero visibility.
- **Scope:** Per-tutor: profile views, interviews requested, conversion %, earnings rollup, monthly trend.
- **Effort:** M
- **Dependencies:** M12 Analytics events (already in place).
- **Acceptance criteria:**
  1. Dashboard loads in <1s.
  2. Numbers match the source-of-truth event store within 5%.

### 21. Tutor Subscription Tier

- **Status:** Planned
- **Why:** Recurring revenue, not just commission slices.
- **Scope:** Stripe Subscriptions. subscription tier enum on tutor profiles: FREE, FEATURED. Featured tutors appear first in search.
- **Effort:** L
- **Dependencies:** #4a Stripe Test-Mode Integration.
- **Acceptance criteria:**
  1. Tutor subscribes → tier flips, featured boost active within seconds.
  2. Failed renewal grace period 7 days, then auto-downgrade.

### 22. Promo Codes

- **Status:** Planned
- **Why:** Marketing levers for launch campaigns and partnerships.
- **Scope:** the promo codes operation. Applied at commission activation.
- **Effort:** S
- **Dependencies:** #4a Stripe Test-Mode Integration.
- **Acceptance criteria:**
  1. Code reduces commission, recorded in audit.
  2. Expired or over-used code returns 410 Gone.

### 23. Notification Email Delivery

> **Promoted to Tier 1** — see the Tier 1 listing alongside #1 Email Consumer.

---

## Tier 4 — Admin / Ops (P2)

### 24. Admin User Management

- **Status:** Planned
- **Why:** Today user actions are database query language-shell only. Operationally fragile.
- **Scope:** Search users, suspend, view audit log, view linked profiles.
- **Effort:** M
- **Dependencies:** None — Audit Log entity already exists.
- **Acceptance criteria:**
  1. Admin can search by email / name.
  2. Suspend flips users status SUSPENDED, blocks login.
  3. Audit log surfaces last 100 events per user.

### 25. Bulk Approve Queue

- **Status:** Planned
- **Why:** Post moderation backlog grows fast. One-by-one approval does not scale.
- **Scope:** submit operation for admin posts bulk-approve accepting the selected post identifiers. Max 50 per call.
- **Effort:** S
- **Acceptance criteria:**
  1. Bulk approve transitions all to APPROVED, emits notifications.
  2. Partial failure returns per-id status.

### 26. Refund / Dispute Workflow

- **Status:** Planned
- **Why:** Disputes are inevitable in a marketplace. Without workflow, every one is a manual fire.
- **Scope:** the disputes operation. Tutee opens, admin reviews, Stripe refund on resolution.
- **Effort:** L
- **Dependencies:** #4a Stripe Test-Mode Integration.
- **Acceptance criteria:**
  1. Tutee opens dispute → admin notified.
  2. Resolution REFUND triggers Stripe refund + audit entry.
  3. Match status updates accordingly.

### 27. Internal Admin Notes

- **Status:** Planned
- **Why:** Ops team needs scratch space per match (e g. "called tutee, awaiting reply").
- **Scope:** the match notes operation. Admin-visible only.
- **Effort:** S
- **Acceptance criteria:**
  1. Note creation visible only to admin role.
  2. Append-only (no edit) for forensic clarity.

### 28. Status Page

- **Status:** Planned
- **Why:** Trust during incidents. Reduces support volume.
- **Scope:** Public uptime page consuming the existing /actuator/health endpoint + a simple incident feed.
- **Effort:** S
- **Acceptance criteria:**
  1. Public route /status renders current uptime + last 90 days.
  2. Admin can post incident updates.

---

## Tier 5 — Quality / Scale (P3)

### 29. OpenAPI (springdoc)

- **Status:** Planned
- **Why:** Frontend / future mobile / partner integrations need a contract document. Hand-maintained docs go stale within a sprint.
- **Scope:** Add springdoc-openapi-starter-webmvc-ui. Auto-generates /v3/api-docs and Swagger user interface at /swagger-ui html.
- **Effort:** S
- **Acceptance criteria:**
  1. All 16 controllers appear in Swagger user interface.
  2. Schemas reflect data transfer object annotations correctly.
  3. signed session token auth flow exercisable from Swagger user interface.

### 30. end-to-end Tests (Playwright)

- **Status:** Planned
- **Why:** Unit + integration tests miss cross-system regressions. Payment + auth flows must be covered end-to-end.
- **Scope:** Golden flows: register → post → match → interview → commission. Browser matrix Chromium + Web Kit.
- **Effort:** L
- **Dependencies:** #6 continuous integration/continuous deployment, #4a Stripe Test-Mode Integration.
- **Acceptance criteria:**
  1. continuous integration runs Playwright suite on every pull request.
  2. Five golden flows pass green for 7 consecutive days.

### 31. i18n (EN / ZH / MS / TA)

- **Status:** Planned
- **Why:** SG's four official languages. ZH is a major demand-side segment.
- **Scope:** i18next on frontend, messages * implementation reference on backend (used for emails).
- **Effort:** L
- **Acceptance criteria:**
  1. Language picker switches all visible strings.
  2. Emails sent in the user's preferred language.
  3. Date / currency localized.

### 32. OAuth Login (Google Sign-In)

- **Status:** Planned
- **Why:** Drops signup friction; password reset volume drops; phishing surface reduces.
- **Scope:** Spring Security OAuth2 client; link to existing user by verified email.
- **Effort:** M
- **Acceptance criteria:**
  1. New user can sign up via Google.
  2. Existing email-password user can link Google to their account.

### 33. PWA / Install Prompt

- **Status:** Planned
- **Why:** SG users are mobile-first. App-store presence without app-store cost.
- **Scope:** manifest webmanifest, service worker, offline shell for cached profile + dashboard pages.
- **Effort:** M
- **Acceptance criteria:**
  1. Chrome shows install prompt on second visit.
  2. Offline shell renders cached data for last-viewed pages.
  3. Lighthouse PWA score ≥ 90.

### 34. Database Backups + DR Runbook

> **Promoted to Tier 1** — see the Tier 1 listing alongside #6 continuous integration/continuous deployment and #6a Staging Environment.

### 35. A11y Tests (axe-core)

- **Status:** Planned
- **Why:** SG IMDA requires WCAG 2.1 AA for many government-adjacent services; tuition crosses into this scope.
- **Scope:** vitest-axe per component test; fail build on critical violations.
- **Effort:** M
- **Acceptance criteria:**
  1. Zero critical axe violations across all rendered pages.
  2. continuous integration enforces.

### 36. Performance Budget

- **Status:** Planned
- **Why:** Without a budget, perf degrades silently. SG mobile-first users feel it first.
- **Scope:** Lighthouse continuous integration in GitHub Actions. Fail build if LCP > 2.5s, CLS > 0.1, TBT > 300ms on the home and search pages.
- **Effort:** S
- **Dependencies:** #6 continuous integration/continuous deployment.
- **Acceptance criteria:**
  1. pull requests with perf regressions fail continuous integration.
  2. Reports archived per pull request for comparison.

---

## Cross-Cutting Concerns (Continuous)

These are not items — they are disciplines applied to every ticket above.

| Concern | Rule | Enforcement |
| --- | --- | --- |
| Test coverage | backend ≥ 97.5% line, frontend ≥ 97.5% line / 88% branch | JaCoCo + Vitest gates in continuous integration |
| Security review | Each module reviewed by security agent before merge | /security-review skill on pull request |
| Documentation | Every new endpoint documented; every new env var added to .env implementation reference | pull request template checklist |
| Liquibase discipline | All schema via Liquibase XML in db/changelog/. No ALTER in code. No raw database query language outside changelogs | pull request review |
| Code style | Spotless (backend) + ESLint + Prettier (frontend) | Pre-commit + continuous integration |
| No magic strings | application programming interface paths → application programming interface ROUTES constants; enums typed; no raw literals in logic | Code review |

---

## Suggested Sprint Sequencing

| Sprint | Window | Items | Theme |
| --- | --- | --- | --- |
| 1 | This week | #2 Security Headers, #3 cross-site request forgery protection, #6 continuous integration/continuous deployment, START #1 Email Consumer (infra layer only) | Unblock launch — security + continuous integration |
| 2 | Next week | FINISH #1 Email Consumer (templates + failed-message queue + #23 wiring), #7 Reschedule Interview, START #5 File Upload, START #4a Stripe test-mode, #36 Performance Budget | Email live + media + revenue start |
| 3 | Week 3 | FINISH #4a Stripe test-mode, #4b Pay Now, #6a Staging Environment, #34 database Backups, #8 Calendar Sync, #9 Auto Meeting Links | Payments + infra hardening |
| 4 | Week 4 | #4c Live-mode cutover, #3a Mandatory multi-factor authentication, #3b Refresh Token Rotation, #7a Personal Data Protection Act Export & Erasure | Compliance + live revenue |
| 5 | Week 5 | #10 Chat, #11 Admin mobile, #12 Verification badges | Trust + comms |
| 6 | Week 6 | #13 Public profile URL, #14 Post GIS, #15 Subject taxonomy | Search + SEO |
| 7+ | Ongoing | Tier 3 + 4 by business priority | Engagement + ops |

Backlog items in Tier 5 are quality investments; slot one per sprint as a "tax" item.

### Blockers / Lead Times

External dependencies that have to be filed early because the wait blocks the sprint they land in.

| Sprint | Blocker / Lead Time |
| --- | --- |
| 1 | Amazon Web Services SES production access (file day 1, ~7d lead) for #1 Email Consumer. |
| 2 | Stripe business verification (file day 1, 5–10 business-day lead) for #4a; Cloudinary account provisioned. |
| 3 | Google OAuth verification kickoff (4–6 week lead) — file in week 3 to land in time for #8 Calendar Sync in week 7. MAS regulatory check for Pay Now (#4b). |
| 4 | Production secret manager (Amazon Web Services Secrets Manager or Fly secrets) provisioned for #4c live-mode keys. |

---

## Decision Log

Architecture Decision Records live here. Append-only. Entries up to ADR-009 are retrospective — they capture decisions made during M1–M12 that were never written down. ADR-010 onwards are forward-looking.

| # | Decision | Context | Status | Date |
| --- | --- | --- | --- | --- |
| ADR-001 | A shared page-of-results response for paginated responses | Spring's Page`<T>` leaks Pageable internals over the wire and makes the contract brittle for the private implementation folder A flat a result list with its page, size, total records and total pages is easier to version and document. | Accepted | 2026-02-10 |
| ADR-002 | Auth transport via HttpOnly session cookie | Bearer token in Authorization header requires frontend to handle storage (XSS risk in local Storage) and complicates cross-site request forgery protection. HttpOnly cookie + cross-site request forgery protection token (item #3) is the standard browser-safe pattern. | Accepted | 2026-02-12 |
| ADR-003 | personal information encryption via Tink AES-SIV | Deterministic encryption needed for searchable personal information (email lookup at login). Tink AES-SIV provides this without the IV-reuse pitfalls of GCM-SIV used carelessly. | Accepted | 2026-02-20 |
| ADR-004 | Schema migrations via Liquibase XML changesets only | Mixing Flyway/Liquibase/raw database structure definitions causes drift. XML chosen over YAML for stricter validation and IDE tooling. No raw database structure definitions anywhere in the codebase. | Accepted | 2026-02-22 |
| ADR-005 | Coverage gate: backend JaCoCo ≥97.5% line+branch; frontend Vitest ≥97.5% line / ≥88% branch | Raised from 95% on 2026-05-28 after repeated regressions slipped through. Branch threshold lower on frontend because component branches inflate the denominator artificially. | Accepted | 2026-05-28 |
| ADR-006 | Magic strings forbidden — application programming interface ROUTES constants for all paths; typed enums for all enumerated values | Refactoring a path or renaming an enum value should be a single-file edit, not a grep-and-pray. Enforced at code review. | Accepted | 2026-03-05 |
| ADR-007 | Java toolchain pinned to Java 21 | System default Java 26 breaks Lombok annotation processing and JaCoCo instrumentation. Pinned via JAVA HOME in build commands and in continuous integration. | Accepted | 2026-03-12 |
| ADR-008 | Integration testing via Testcontainers (Postgres + Redis); Docker Desktop required | H2 / embedded Redis diverge from production behaviour (timestamp handling, Stream semantics). Testcontainers makes the test environment match prod. | Accepted | 2026-03-18 |
| ADR-009 | Mandatory /team-review after every module deliverable | Solo review missed cross-cutting concerns repeatedly during M1–M4. Forcing a multi-agent review surfaces issues earlier and cheaper. | Accepted | 2026-04-22 |
| ADR-010 | Cloudinary for v1 file uploads (#5) | Choice was Cloudinary vs S3 + Clam AV sidecar. Cloudinary bundles signed uploads, image transforms, content moderation, and CDN; Clam AV would require operating a sidecar and a separate transform pipeline. Revisit at scale if Cloudinary cost becomes material. | Accepted | 2026-05-29 |
| ADR-011 | M13 operator personal information access defaults to **decrypt-on-the-fly**; a stored ops pii plaintext directory only if partial/substring search is a proven operator need, and only after database backups exist | Real 17-agent review + Boss SEND BACK. AES-SIV is deterministic (verified in Encryption Service), so exact-match search already works over ciphertext — no second store needed for exact lookup. A plaintext copy reintroduces ADR-003's blast radius (same database instance, same pg dump, same app credential + Tink keyset). If a stored directory is justified, it ships via a transactional outbox with a repair path (not "single-writer + nightly parity"), audit **fail-closed** (503 on audit-write failure), a **blocking** bulk-read cap, named-multi-factor authentication OPERATOR accounts, and Personal Data Protection Act erasure cascade. M13 is sequenced **last**. | Superseded prior informal note; Accepted | 2026-05-29 |
| ADR-012 | M14 landing showcase = public, cached, **split** (/showcase/jobs, /showcase/tutors), personal information-free Page Response endpoints; sequenced **after** M15; applicant count comes **only** from real M15 applications | Live social proof (recent jobs + active tutors). 7-day job window; users last active at via a **guarded** write (update last activity only after the configured interval has elapsed since the previous recorded activity, Redis-gated — never per-request). Random via Redis cache (~60s, stampede-locked, defined Redis-down degraded mode); frontend freshness-first ordering + low-volume layout. Showcase funnel events required or the thesis is untestable. **Update 2026-05-30:** M14a deployed (migration 0019) and M14b (Recent Requests) deployed (migration 0023) after the founder waived the consent/legal gate; both shipped without an applicant count, consistent with ADR-013. | Accepted; M14a+M14b deployed | 2026-05-29 |
| ADR-013 | A real "Apply" concept (M15 job applications) is the **only** source of the applicant count; the interview-signal proxy is **CUT** | Build M15 **first**. The proxy (interview requests relabelled "applied") was rejected by Architect, Product, BA, Data, Writer, and both Platform Users as dishonest. Count = distinct tutors with an APPLIED application to the job. Partial the unique operation (re-apply must work), count index, named job application status enum. No count is shown until M15 ships. **Update 2026-05-29:** spec finalised and M15 moved from *approved* → *in implementation* (backend+frontend in parallel, team review pending). Thin slice ratified: APPLIED/WITHDRAWN only; idempotent apply; withdraw→re-apply; admin-only applicant visibility (ADMIN, not OPERATOR — that role does not exist yet); Liquibase 0018-create-job-applications implementation reference. Module docs: [Private supporting material excluded (publication note)](../docs/publication-notes.md) (spec, db, backend, frontend) + sequence [28](sequences/28-tutor-apply-withdraw.md). | Accepted; in implementation | 2026-05-29 |

When a Tier 1–2 item is picked up, the engineer logs the key decision here before merging. Format:

- **Decision:** One-line statement of what was chosen.
- **Context:** Two-line summary of alternatives and why this won.
- **Status:** Proposed / Accepted / Superseded.
- **Date:** YYYY-MM-DD.

---

## Notes for Engineers Picking Up Tickets

1. Read the **Why** before the **Scope**. If the why doesn't hold up today, raise it; don't silently build the wrong thing.
2. Dependencies are hard. If item Y says it needs X, do not start Y in parallel — coordinate.
3. Effort sizes are rough: S ≤ 2 days, M ≤ 1 week, L ≤ 2 weeks, XL > 2 weeks. If your estimate doubles, surface it.
4. Every Tier 1 ticket needs a security review and a coverage gate pass. No exceptions.
5. Add an ADR entry when you make a non-obvious technical choice. Future you will thank present you.
