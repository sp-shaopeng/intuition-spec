# InTuition — Work Plan (state + forward)

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-06-03 · **Owner:** Engineering (orchestrated) · **Audience:** founder + team
**Purpose:** one source for *where we are* (what shipped this session, live on dev) and *what's next*
(sequenced remaining work). Cross-links the issue tracker and ROADMAP rather than duplicating them.

---

## A. State — shipped this session (live on the DEV stack)

All items below are **built, gate-verified by the orchestrator, Boss-cleared (SHIP IT), and deployed to dev**
(backend + frontend containers healthy on  / ). They are **not** in prod (see §B).

### A1. Feature wave
| Issue | Feature | Migration | Notes |
|---|---|---|---|
| INT-034 | Free-market assign — matches re-keyed on tutor user id; fixes contact-reveal/dashboard; 0033f dedupe | 0033 | submit operation for admin selection-requests id assign + winning Tutor Id; admin-only matching preserved |
| INT-043 | Tutor onboarding fields optional at signup + post-login completion banner | — | empty-string sentinel keeps the apply-gate intact (INT-017) |
| INT-045 | Admin analytics from/to + collected In Period Sgd + 6 timestamp indexes | 0037 | interview funnel stays point-in-time |
| INT-025 | POST SUBMITTED admin-fan-out notification | 0036 | after-commit, failure-isolated, the find by role operation |
| INT-039 | Interview timeslot negotiation (state machine, Slot Picker/Whose Turn/Timeline) | 0034 | 3 Boss conditions met: idempotent expiry sweep + interview expired metric, INT-016 lazy-init IT, recreated cap index |
| INT-044 | Tutee→tutor job invitations | 0035 | accept folds into apply→choose funnel (no new Match path); 5 measurement events |
| INT-046 / INT-051 | frontend coverage debt + backend branch debt cleared (Wave-1) | — | merge-gate hygiene |

### A2. user interface redesign — INT-049 (Waves A–F)
- Design tokens wired into Tailwind + a shared **layout-primitive system** (the private implementation folder:
  Page Header, Section, Container, Card Grid, Form Layout, Data List, Tab Nav, Auth Card, Article Layout; ui/:
  Stat Card, Status Timeline) — each unit-tested.
- **Every route redesigned** — public, tutee+tutor dashboards, admin (purple shell, new admin.* token group
  replacing the Admin Layout magic strings), info/legal pages, + the 5 audit-caught pages.
- **Header dashboard tabs**: tutee/tutor tabs moved into the top Nav after login (role-honest); **login lands
  on the dashboard's first tab** (honours return To); admin already had header tabs.

### A3. Verified gate numbers (orchestrator-run)
- **Backend:** the documented operational check BUILD SUCCESS — **98.92% line / 98.13% branch**, ~1324 unit + 139 IT, 0 failures.
- **Frontend:** lint+build+test:ci green — **98.25% line / 92.79% branch**, 1492 tests; **whole-src
  arbitrary-value audit = 0** (pages + components + layouts, incl. semantic colors).
- Non-negotiables intact (hero line, privacy promise, Commission 50% serif callout, admin purple, handle-only
  personal information); brand hex/fonts locked; no new deps; no behaviour/application programming interface change beyond the intended ones.
- Records: issues/BATCH-WAVE-2026-06-03-built md, the issues tracker banner, uiux/all-pages-redesign-spec md §9.

---

## B. Forward plan — remaining work (sequenced)

| # | Work | Owner / gate | Notes |
|---|---|---|---|
| 1 | **Prod deploy** of the wave + redesign | Founder (prod access) | PROD-DEPLOY-OWED. Migrate-then-boot in order **0031 → 0033 → 0034 → 0035 → 0036 → 0037** (release phase, before the jar boots under ddl-auto is validate); each ALTER TYPE ADD VALUE commits before code references it. Post-deploy smoke: free-market assign, negotiation, invitations, analytics range, admin notify. Rebuild frontend image for prod. |
| 2 | **Human visual + screen-reader pass** on the redesign | Founder / a human | The team-review pre-prod gate — a redesign's *look* isn't test-verifiable. Click through Landing, a tutee dashboard, an admin list, a form, on a phone + with Voice Over/NVDA. (Code-level a11y: focus-visible + reduced-motion in place; Data List double-read confirmed a non-issue — display:none toggling.) |
| 3 | **INT-048 root-cause** the corrupt-Postgres-volume pg hba | dba + devops | Inspect the repo backups/ restore path — does it produce a pg hba missing host all all all? Has prod teeth (no down -v escape in prod). INT-047 (Docker build wedge) resolved-by-restart; watch. |
| 4 | **Deferred Tier-D features** (each needs its own spec → team-review) | product + eng | INT-039 extras (counter expiry/decline-terminal product calls), INT-043 force-completion onboarding decision, INT-044 invitation extensions (list-card invite, recommend surfaces), INT-045 metric range decisions, Epic O. |
| 5 | **Standing prod-readiness gaps** (foundations, from status-briefing md §2) | eng + founder | payments (Stripe/Pay Now), automated backups/PITR, continuous integration/continuous deployment, staging env, real email sender domain (off REDACTED email), multi-factor authentication-on for admins, DPO + real Personal Data Protection Act legal copy. Sequence foundations before more features. |
| 6 | **Docs/Postman reconciliation** (this work-plan's sibling deliverables) | tech-writer + bff | frontend-routes/pages/wireframes/modules/ROADMAP/status-briefing/CLAUDE md refresh + 15 missing Postman requests. Tracked alongside this doc. |

**Sequencing recommendation:** §6 (docs/Postman) now; §1 prod + §2 visual pass when the founder is ready
(they gate each other for a real launch); §3 INT-048 soon (prod risk); §5 foundations are the real
launch-blockers; §4 Tier-D after foundations.

---

## C. Cross-links
- Open issues + the wave banner: [issues/README md](issues/README.md) · wave record:
  [issues/BATCH-WAVE-2026-06-03-built md](issues/BATCH-WAVE-2026-06-03-built.md)
- Prod-deploy register + tiers: [ROADMAP md](ROADMAP.md) (PROD-DEPLOY-OWED)
- Prod-readiness gaps: [status-briefing md](status-briefing.md) §2
- Redesign spec + verdict: [uiux/all-pages-redesign-spec md](uiux/all-pages-redesign-spec.md)
