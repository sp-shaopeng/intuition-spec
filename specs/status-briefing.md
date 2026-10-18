# InTuition SG — Current-State Status Briefing

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-05-29 · **From:** the solution architect (Solution Architect), with Tech Lead
**For:** Product & Business teams (roadmap planning)
**Purpose:** An honest, verified snapshot of what is actually deployed and what still
blocks a real launch — not aspirations. Every claim was checked against the code
(controllers, latest Liquibase changeset 0021, User Role, Email Queue Service,
Super Admin Seeder, docker-compose implementation reference), not against the planning docs.

---

> ## 🔎 Verified update — 2026-06-04 (post user experience redesign + admin/tutee user experience wave + lifecycle)
>
> **DEV-ONLY** (nothing new in prod; PROD-DEPLOY-OWED now spans migrations 0033→0039). Shipped to dev
> since 06-03, each team-reviewed → Boss SHIP IT with gates re-verified: **INT-049** all-pages redesign +
> header dashboard-tabs; the **06-03 feature wave** (INT-034/039/044/025/045/043, head 0037); the **INT-053–059
> wave** (INT-056 Sev-1 refresh-logout fix, INT-053 full-stack admin tutor↔tutee+job clickable, + INT-054/055/
> 057/058/059/060); and **INT-062** withdraw/close-live-job cascade (apps+invitations → visible CLOSED, head 0039).
> **Honest gaps:** **INT-064 (S1)** continuous integration not wired to the subrepos → the Testcontainers/IT layer runs nowhere
> (local gate = surefire-unit + jacoco only); **INT-063 (S2)** admin close-job lacks a discoverable surface;
> **INT-061** admin Job-column blank; **INT-060** flaky test. Real launch still blocked by the standing prod
> gaps + the dev→prod deploy (do not prod-deploy until continuous integration runs the integration gate).
>
> ## 🔎 Verified update — 2026-06-02 (the solution architect, post INT batch + Hunter run)
>
> The full backend regression-lab + frontend suites were run green-to-completion (first time in a while) against
> a freshly reseeded dev stack. Sources: issues/README md, issues/BATCH3-REMEDIATION-PLAN md,
> and a direct re-read of Public Browse Controller, Job Application Repository, Resend Email Client,
> Auth Service, Tutor Post Service, Contact Reveal Service.
>
> **Shipped to dev:** founder-reported defect batch INT-001…014 (sign-up link, forgot-password reset
> deep-link, tutor-detail crash fix, nav About→Admin, pre-login filter gating + principal-gated
> anonymous /public/tutors cap, 30-min session, Node 20 pin). Team- + code-reviewed, tested, live-smoked.
>
> **Two infra defects fixed (had silently red-lined the whole regression-lab suite):** INT-018 seed
> password drift; INT-019 a JPQL inline-enum-literal cast (Job Application Status, a non-existent PG
> type) that 500'd match-create/assign. Suite recovered 16→0 batch-caused failures.
>
> **⚠️ Correction to §1 below (verified false on 2026-06-02):** the M18 row's "atomic assign … cancel→reopen
> … contact reveal" does **not** all work on dev. Hunter reproduced **web request protocol 500 on match-cancel and
> contact-reveal** (Lazy Initialization Exception, open-in-view is no) — **INT-016, Sev-1**. Also: an
> owner cannot delete a REJECTED post (INT-015), and a profile-incomplete tutor is blocked from applying
> (INT-017). These are pre-existing, now on the eng plan, and are **launch blockers for the concierge
> operating loop** — staff cannot currently cancel/unwind a match or reveal contact details. Match
> *create/activate/commission/review* paths are verified green.
>
> **Net effect on this briefing:** §2 prod-readiness gaps are unchanged and still hold; add INT-016/015/017
> as **functional** blockers alongside them. No change to the dev-only posture.

---

> ## 🔎 Verified update — 2026-06-03 (the solution architect/Tech Lead, post fix-batch + deploy)
>
> The INT-015…042 batch was implemented, team-reviewed (+ Boss SHIP), code-reviewed, and **deployed to the
> dev stack** (fresh reseed). **Now FIXED & verified green** (the documented operational check, ITs vs the live stack):
> **INT-016 (Sev-1)** match-cancel + contact-reveal 500 — real cause was the modifying operation
> detaching the context mid-tx, **not** open-in-view; **INT-015** REJECTED-post delete 500 (migration **0031**
> relaxes the status-guard trigger for the soft-delete close); **INT-017** seeded tutor can now apply (seed
> completes the canonical profile; clean handles verified). Plus the M6/M17/M18 user experience batch
> (INT-032/033/034-stopgap/035/036/037/038/040/041/042) and the frontend Vitest deadlock (INT-020). Backend 1153
> unit + all ITs green; frontend 1234 tests green.
>
> **Two honest carry-overs:** **INT-047** — local Docker Build Kit is wedged (can't build images; frontend deployed
> via the documented operational check + the documented operational check workaround, persistent; real fix = reboot Docker). **INT-046** — the
> INT-020 fix let the full frontend suite run and enforce coverage for the first time, revealing **95.82% < 97.5%**
> (pre-existing debt in admin/auth files; all tests pass).

---

> ## 🔎 Update — 2026-06-03 (later): feature wave + full user interface redesign HISTORICALLY MARKED IMPLEMENTED to dev
>
> What was previously "deferred" (INT-025/034/039/043/044/045) has since been **built, gate-verified,
> Boss-cleared (SHIP IT), and deployed to dev** — see [work-plan md](work-plan.md) §A and
> [issues/BATCH-WAVE-2026-06-03-built md](issues/BATCH-WAVE-2026-06-03-built.md):
> - **INT-034** free-market assign (matches re-keyed on tutor user id, migration 0033) — and **INT-016
>   (the Sev-1 cancel/contact-reveal 500) is FIXED**, so the earlier §1 correction no longer blocks.
> - **INT-043** tutor onboarding optional + completion banner · **INT-045** analytics date-range + indexes (0037)
>   · **INT-025** POST SUBMITTED admin notify (0036) · **INT-039** interview timeslot negotiation (0034)
>   · **INT-044** tutee→tutor job invitations (0035).
> - **INT-049 — full all-pages user interface redesign** (design tokens + layout-primitive system; header dashboard tabs
>   after login; login→dashboard first tab). frontend 98.25% line / 92.79% branch; backend 98.92% / 98.13%; whole-app
>   token audit 0.
>
> **Schema head is now Liquibase 0037** (0033 INT-034, 0034 INT-039, 0035 INT-044, 0036 INT-025, 0037 INT-045;
> 0032 reserved for M19 OAuth). **Still genuinely deferred:** the §B Tier-D *extensions* (INT-039 product calls,
> INT-043 force-completion, INT-044 invitation extras, Epic O) — each needs its own spec/team-review.
> **§2 prod-readiness gaps below still hold** (this all shipped to DEV only; prod deploy is founder-gated).

---

## 1. What is live and shippable today (DEV stack only)

The platform is **built and running on the dev Docker stack** — Postgres + Redis +
the Spring Boot application programming interface + the React SPA. It has **never been deployed to a real,
internet-facing production environment with real users.** "Shippable" below means the
feature is built, tested, and behaves as described on the dev stack; it does **not**
mean it is safely operable in production. See Section 2 for why.

**Modules built and deployed on the dev stack:**

| Scope | What works |
|---|---|
| **M1–M12** (core marketplace) | Accounts (register as parent/tutee or tutor, login, admin time-based one-time passcode login, password reset/change, account deletion), posting + public paginated browse, admin approve/reject queue, admin match create/activate/cancel, commission (manual "mark paid") + match completion, interview request/schedule/no-show, profiles, match dashboard, in-app notification centre, search + favourites, post-match reviews, and an admin analytics dashboard (database count/funnel queries — not behavioural event tracking) |
| **M15 — Applications** (changeset 0018) | Tutors apply to / withdraw from a tutee's job; admin sees the applicant list. First step away from 100% admin-driven matching; the honest source for a future "N tutors applied" count |
| **M14a — Active Tutors showcase** (0019) | Public showcase of active tutors |
| **M16 — Admin Management & Super-Admin** (0020) | Admin account management and super-admin provisioning surface |
| **M6 interview cap** (0021) | Hard cap of max 3 interviews per posting |
| **M18 — Assignment Lifecycle & Free Marketplace** (changesets 0025–0029, dev only) ⚠️ **see 2026-06-02 correction** | Free-market apply (no listing required); profile as canonical identity (handle/commitment/rate/area); atomic assign (winner→MATCHED, others→SUPERSEDED, job→MATCHED, optimistic-lock 409, cancel→reopen); parent applicant view (personal information-free) + choose-tutor selection requests; contact reveal post-assignment. **✅ 2026-06-03: cancel + contact-reveal FIXED (INT-016) and verified green on dev (create/activate already green); cancel now reopens the job + notifies the former tutor.** **frontend admin surfaces built 2026-06-01:** §G2 selection-request queue + assign, §G3 Matching lifecycle (activate / mark-paid / cancel→reopen), §G4 stale-commission tab. Plus the tutor/parent frontend (profile, cards, apply, my-applications, applicants, contact reveal) from prior passes. |
| **Email consumer** (Resend) | Email Consumer Service reads the email-notifications Redis stream and dispatches via Resend; XCLAIM crash-recovery, Micrometer metrics, HTML-encoded templates, graceful no-key startup. Dev only; no failed-message queue / custom sender domain yet (pre-prod gate). |
| **Account / user interface changes** | Plaintext email & phone shown **to authorized viewers only** in the user interface (per the ratified personal information-visibility model), phone formatting, qualification dropdown, tutor portfolio prompt |

**Latest applied schema (dev):** Liquibase changeset **0031** is the head (0024
audit-log fix; 0025–0029 the M18 assignment-lifecycle set; 0030 interviews-listingless-applicant;
**0031 allow-post-soft-delete-close — INT-015, 2026-06-03**). Next free = 0032 (M19 OAuth).
Controllers now include
the M18 additions (Admin Selection Request, Tutor Selection, plus the parent applicant /
choose-tutor / contact-reveal endpoints) on top of the original 18.

**Company legal entity:** the agency’s registered legal name (UEN
the agency’s registration number) — wired into the frontend (company implementation reference).

**At-rest personal information:** still **AES-SIV (Tink) encrypted** in the database. Plaintext is only
ever rendered to authorized viewers in the user interface; it is not stored or transmitted in
the clear at rest.

---

## 2. Prod-readiness gaps that still block a real launch (each verified)

The feature set is real, but the platform is **not yet safe to operate live.** These
are the gaps that stand between "demos well on the dev stack" and "can take real,
paying users":

1. **Email is now delivered via Resend (dev stack only).** Email Consumer Service is
   deployed: it reads from the email-notifications Redis stream via consumer group
   email-dispatcher and dispatches through the Resend application programming interface. Three templates live:
   forgot password, post approved, post rejected. XCLAIM-based crash recovery is
   implemented (PEL idle ≥ 30s). **Not yet production-safe:** no failed-message queue, no retry counter,
   no custom sender domain (currently sends from REDACTED email which will land
   in spam for real users — must switch to a verified domain before any real-user
   traffic). Remaining 4 templates (interview reminder, match created, commission due,
   review received) are not yet implemented.

2. **No payments.** Commission is the revenue model, but there is no Stripe/Pay Now or
   any gateway. An admin manually marks commission "paid" (Admin Commission Service).
   We cannot collect revenue through the platform.

3. **No automated backups.** No backup/PITR mechanism exists. On the dev stack a disk
   failure or bad command means total, unrecoverable loss of every account, post,
   match, and review.

4. **No continuous integration/continuous deployment.** No .github/workflows, no Git Lab continuous integration, no Jenkinsfile. Every build is
   manual on an engineer's laptop; nothing automatically stops a broken change.

5. **No staging environment.** There is nowhere to validate a change against
   real-shaped data before users see it.

6. **Monitoring is spec-only.** Observability is designed but **not deployed** — no
   live metrics/alerting stack running.

7. **Dev secrets are committed in the repo and MUST be regenerated for prod.**
   docker-compose implementation reference contains plaintext dev-only credentials, explicitly labeled
   "DO-NOT-USE-IN-PROD": database & Redis passwords (REDACTED password), the signed session token keyed message authentication key, and —
   critically — the **Tink AES256_SIV master encryption keyset** (the key that
   protects all personal information at rest). The application itself reads these via an environment-provided configuration value
   references in application implementation reference, which is correct — but the committed dev
   values must all be rotated/regenerated before any prod deploy.

8. **Production has no admin.** Super Admin Seeder is the profile operation only (there
   is no dev profile). On a prod profile, **no admin account is created** — so a
   secure super-admin provisioning path must be built before launch, or the platform
   ships with no way in.

9. **multi-factor authentication is off.** time-based one-time passcode exists in code (Auth Service, Mfa Verify Request) and only
   gates ADMIN login when both a feature flag and the user's own multi-factor authentication flag are on; in
   the dev config the challenge is skipped. It is not enforced.

10. **Analytics is count-only, no real funnel.** Analytics Service computes counts
    and simple funnels from database queries; there is no behavioural event-tracking
    pipeline. Treat the dashboard as a log/count-level stub, not product analytics.

11. **DPO / legal pending.** Privacy and terms are placeholder stubs; a DPO and real
    legal copy are outstanding (required for handling Singapore personal information under Personal Data Protection Act).

**Bottom line for Business:** the product is feature-rich and demos well, but
launching to paying customers in this state risks data loss, silent breakage, users
who never hear from us, an inability to charge, and a Personal Data Protection Act exposure on placeholder
legal terms.

---

## 3. Deferred bets — and what each waits on

| Bet | What it is | Waiting on |
|---|---|---|
| **M14b — Recent Requests** | Public landing showing recent job requests as social proof | **No longer deferred — DEPLOYED (migration 0023).** The consent/legal gate was **waived by the founder (2026-05-30)**; the anonymised strip shipped |
| **M13 — Ops Console + Secure personal information** | A dedicated, audited console for the ops team to safely look up real contact details, under a restricted **OPERATOR** role | **Hard-gated, sequenced last.** User Role is confirmed TUTEE/TUTOR/ADMIN only — **no OPERATOR role exists in code.** Cannot ship until backups exist, the audit log fails *closed*, and bulk reads are hard-capped |

---

## 4. Hosting decision pending (analysis done this session, NOT yet executed)

A deployment-target analysis was completed but **no infrastructure has been stood up
and no runbook written.** Options for Product/Business to choose from:

| Option | Est. monthly cost | Notes |
|---|---|---|
| **GCP managed** | ~US$60–90 | Cloud database query language (Postgres) + Memorystore (Redis) dominate the cost. Most operationally robust; **closes the backups gap** via managed Postgres PITR. No runbook written yet |
| **Single virtual machine** | ~$15–30 | Cheapest cloud option, but a single box = single point of failure; backups are DIY |
| **Digital Ocean droplet** | ~$18–24 | Simplest to operate. DO **managed** Postgres/Redis would close the backups gap |
| **Oracle Cloud Always Free** | Free (ARM) | Zero cost, but ARM + free-tier constraints; DIY backups; weakest support story |

**Architect's flag:** the single cheapest line items aside, **managed Postgres/Redis
(Cloud database query language/Memorystore or DO managed) is the lever that closes the no-backups gap**
without us hand-rolling backup automation. A bare virtual machine or Oracle free tier saves money
but leaves the most dangerous gap (backups) on our plate. **No GCP/DO runbook exists
yet** — that work is still to be done once a target is chosen.

---

## 5. Open decisions the roadmap must resolve

1. **Which hosting target?** (GCP managed vs single virtual machine vs Digital Ocean vs Oracle
   free.) This choice also decides how cheaply we close the backups gap.
2. **Email before or after payments?** Real email is arguably a launch blocker
   (password reset is broken without it); payments gate revenue but carry external
   lead times. Which comes first?
3. **When do we do the prod-hardening block?** (Backups, continuous integration/continuous deployment, staging, secret
   rotation, prod admin provisioning, monitoring deploy, multi-factor authentication-on.) Before more
   features, or interleaved?
4. ~~**M14b consent work** — when do we commission the consent + legal gate that
   unblocks Recent Requests?~~ **Resolved 2026-05-30:** founder waived the gate; M14b
   shipped (migration 0023).
5. **DPO / legal** — when do we engage a DPO and replace the placeholder
   privacy/terms stubs? (Personal Data Protection Act-relevant given we hold real personal information.)

---

## 6. Engineering-recommended sequencing (input — Product & Business own the call)

Framed as cost/value. **You own the roadmap; this is engineering's recommendation.**

**Do first — foundations / prod-hardening (cheap relative to the risk they remove):**
- **Pick hosting + stand up managed Postgres/Redis.** This single move closes the
  catastrophic backups gap.
- **Rotate all secrets** (database, Redis, signed session token, **Tink keyset**) out of the repo into the
  chosen platform's secret store before any real data exists.
- **Secure super-admin provisioning** for the prod profile (today prod has no admin).
- **continuous integration/continuous deployment + staging.** Force-multiplier — unblocks safe deploys and every later step.
- **Deploy monitoring** (the spec already exists).

**Then — user-facing enablers:**
- **Real email sender** (the missing Redis-stream consumer → SES/SMTP). Without it,
  password reset and all alerts are dead.
- **DPO + real legal copy** before public launch (Personal Data Protection Act).

**Plan deliberately, file external dependencies early:**
- **Payments (Stripe/Pay Now)** — large, external verification lead times; start
  paperwork early even if the build lands later.
- **multi-factor authentication-on for admins** and finally **M13 Ops Console / OPERATOR role** — the most
  sensitive surface, intentionally last, only after backups and audit-fail-closed are
  real. *(The M14b consent gate that previously sat here was waived by the founder
  2026-05-30; M14b is deployed.)*

**The headline:** the highest-risk gaps (backups, secret rotation, continuous integration/continuous deployment, staging)
are among the *cheapest* to close relative to the damage they prevent — especially
on a managed-database host. Foundations first; features sit on top of them.

---

*Prepared from a direct read of the codebase on 2026-05-29: 18 backend controllers,
Liquibase head at changeset 0021, User Role = TUTEE, TUTOR, ADMIN (no OPERATOR),
Email Queue Service producer-only with no consumer, the initial administrator seeder restricted to local development,
dev secrets incl. the Tink AES256_SIV keyset committed in docker-compose implementation reference, and
no continuous integration/continuous deployment or payment integration anywhere in the source tree.*

> *Provenance note (2026-06-02): the figures above are the original 2026-05-29 snapshot. As of
> 2026-06-02 the code has moved on — **24 production controllers, Liquibase head 0030, and
> Email Consumer Service now exists in-tree** (the consumer is no longer out-of-tree). See the top
> verified-update banner; User Role / no-continuous integration-continuous deployment / committed-dev-secrets claims still hold.*
