# FIX-PLAN — review-2026-06-22 → grouped SDD specs (master plan for the Boss)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> Input: CONSOLIDATED md (Boss-verified S2/S3/S4 + sequential-re-run findings + founder GitHub #207–211).
> Founder decision: **fix EVERYTHING** (S2+S3+S4+T-02 + #207–211). Every spec below is **status: DRAFT** — the Boss
> runs the scoped spec-review gate + founder go before any build. Next free spec numbers were verified: **011–018**.

## Spec list (NNN-slug → findings → owner → severity → REQ count)

| Spec | Findings folded in | Lead owners | Sev mix | REQ count |
|------|--------------------|-------------|---------|-----------|
| **011-auth-robustness** | T-02 logout-vs-refresh race; autofill; logged-out poll storm (already fixed, pin); #209.2 duplicate-phone (founder decision) | software-engineer, security-expert, frontend-expert, bff-specialist | 1×S3*, 1×S3/S4, 1×S4, 1×(founder) | 4 |
| **012-tutor-apply-and-profile-ux** | T-07 silent apply gate; T-06 sticky-Apply; #209.1 taxonomy (already fixed, confirm+close); #209.3 phone/email editability (founder decision) | bff-specialist, software-engineer, frontend-expert, uiux-designer | **1×S2**, 1×S4, 1×S3, 1×S3 | 4 |
| **013-tutee-dashboard-and-invitations** | #207.1 edit/view post; #207.2 venue display; #208 invite/interview not in active requests; sent-invitations list | bff-specialist, software-engineer, frontend-expert, uiux-designer | **2×S2**, 1×S3, 1×S2 | 4 |
| **014-interview-scheduling-ux-and-admin-visibility** | #210.1 admin list filter gap; #210.2 auto-fill slot; #210.3 clipped No-Show; #211 datetime picker; raw-handle display | frontend-expert, uiux-designer, bff-specialist, software-engineer | **1×S2**, 4×S3 | 5 |
| **015-mobile-responsive-fixes** | INT-210 grid page overflow; ADM-nav 390px; T-04 nav breakpoint | frontend-expert, uiux-designer | **1×S2**, 2×S3 | 3 |
| **016-error-copy-and-handling** | INT-211 raw "not found: <uuid>"; bare-path 500→404; ADM-400 field naming; tel: escaping | software-engineer, frontend-expert, bff-specialist | 1×S3, 3×S4 | 4 |
| **017-admin-analytics-and-financial-safety** | conversion >100% cohort bug; avg-commission calc; **Mark-Paid confirm (financial safety)** | data-engineer, business-analyst, software-engineer, frontend-expert | 2×S3, 1×S4 | 3 |
| **018-flag-hygiene-seeder-and-fe-hygiene** | T-05 WhatsApp toast OFF; T-08 seeder tutor user id; demo-data variety; T-01 /dashboard 404; T-03 step copy; CTA-intent | frontend-expert, software-engineer, uiux-designer | 2×S3, 3×S4, 1×low | 6 |

**Total: 8 specs, 33 REQs.** Severity across all: **5×S2**, ~12×S3, ~12×S4, + 1 low + 3 founder-decision items.

## Already-fixed / not-a-defect (found during grounding — confirm-and-close, don't rebuild)

- **#209.1 tutor subject taxonomy** — ALREADY uses the shared SUBJECT OPTIONS (spec 009): Create Listing implementation reference
  + Subject Level Matrix Editor implementation reference. → 012/REQ-003 = confirm live + close GH #209.1. **No build.**
- **Logged-out unread-count poll storm** — ALREADY gated (INT-166): Notification Bell implementation reference +
  auth Redirect implementation reference. → 011/REQ-003 = add a regression test only. **No re-implementation.**
- **#209.2 duplicate-phone generic message** — INTENTIONAL anti-enumeration/Personal Data Protection Act control (INT-109,
  Auth Service implementation reference). The reviewer's "make it specific" **conflicts** with security. → 011/REQ-004 =
  **default NO CHANGE**, founder decision to override.
- **405 wrong-method** — ALREADY handled (Global error handler implementation reference). Only the **no-handler→404** gap is real
  (016/REQ-002).
- **Admin "S1 refresh-logout" / "tutor hijack admin"** — refuted by the Boss (shared-browser artifacts). Not specced.

## Root-causes confirmed during grounding (so build doesn't re-investigate from zero)

- **#210.1 (admin interview list):** backend is correct (list Admin Interviews find All when no filter). Bug = frontend tab map
  Interview Queue implementation reference — PENDING SCHEDULING/PROPOSED/COUNTER PROPOSED/TUTOR DECLINED are in no tab but
  ALL. **frontend-only.** (014/REQ-001)
- **T-08 (tutor interviews empty):** Dev Data Seeder save Interview never sets tutor User → NULL
  tutor user id; list Tutor Interviews uses find By Tutor User Id. **Seeder-only.** (018/REQ-002)
- **T-07 (silent apply):** frontend↔backend completeness criteria MISMATCH — frontend checks location Region+qualifications is at least 10
  (Tutor Apply Control implementation reference); backend checks location Planning Areas+commitment+rate+handle+subject Levels
  (Job Application Service implementation reference). frontend can green-light an apply the backend rejects. (012/REQ-001)
- **INT-210 (grid overflow):** grid's own overflow-x-auto is correct; a flex/grid ancestor lacks min-w-0. (015/REQ-001)
- **INT-211 (raw not-found):** service sites concatenate the uuid into the message; frontend shows it verbatim. (016/REQ-001)
- **Conversion 400% / avg-commission:** Analytics Service cohort mismatches (compute Conversion Rate,
  compute Avg Commission). (017/REQ-001,002)
- **#207.2 venue truncation:** Dashboard implementation reference renders venue Preferences 0 only — display, not data loss. (013/REQ-002)

## Shared-file overlaps → serialize, don't parallelize

These are the collision hot-spots (this repo gives agents no worktree isolation; same-file edits collide):

- **api implementation reference / auth Store implementation reference** — 011 (logout/refresh) AND 012 (apply 403 handling
  reads the api client). → **serialize 011 then 012** on the api client, or coordinate a single edit window.
- **Tutor Apply Control implementation reference** — 012/REQ-001 + 012/REQ-002 (same spec, sequential within).
- **Dashboard implementation reference** — 013/REQ-001,002,003,004 all touch it → sequential within 013.
- **Interview Detail implementation reference / Interview Queue implementation reference** — 014 only (no cross-spec overlap).
- **Analytics Service implementation reference** — 017 only.
- **Dev Data Seeder implementation reference** — 018 only (REQ-002 + REQ-003).
- **Global error handler implementation reference** — 016 only.
- **Create Request implementation reference / Availability Grid implementation reference** — 015 (grid ancestor) + 018 (step copy) touch Create Request's
  tree; 015 edits the grid/layout ancestor, 018 edits step-header copy — **mostly disjoint, but serialize the two
  edits to Create Request implementation reference** to be safe.

## backend vs frontend lanes (parallelize across subtrees, serialize builds within a subtree)

- **Pure-frontend specs** (no/▵minimal backend): **011, 015** (and 012/013/014/016/018 have small backend parts).
- **backend-heavy specs:** **016** (no-handler 404), **017** (analytics calc), **018** (seeder), **013** (sent-invitations
  endpoint), **012** (profile Complete signal), **014** (maybe a response field).
- Per CLAUDE md: parallelize **non-overlapping subtrees** (a backend lane vs a frontend lane), but **builds within
  one subtree run sequentially** (one the documented operational check, one npm build at a time). So within a wave, run **at most one
  backend build and one frontend build concurrently**, and serialize specs that share a file (see overlaps above).

## Recommended sequencing (severity-ordered, dependency-aware)

1. **S2 / money / safety first:** 012/REQ-001 (silent apply, S2 money), 013 (#207.1+#208, 2×S2), 014/REQ-001 (admin
   interview invisible, S2), 015/REQ-001 (grid overflow, S2), 017/REQ-003 (Mark-Paid confirm, financial safety).
2. **Then S3** correctness/user experience: 011/REQ-001 (logout race), 014/REQ-002..005, 016/REQ-001+002, 017/REQ-001+002,
   015/REQ-002+003, 018/REQ-004+006.
3. **Then S4/cosmetic + confirm-and-close:** 011/REQ-002+003, 012/REQ-002+003, 016/REQ-003+004, 018/REQ-001+002+
   003+005.
4. **Founder-decision items** (gate the relevant REQ, don't block the rest of the spec): 011/REQ-004 (#209.2),
   012/REQ-004 (#209.3).

## Recommended wave split (so the Boss can offer the founder scope options)

**Wave 1 — "stop the bleeding" (S2 + financial-safety + the real founder bugs).** Highest business value, smallest
set the founder is most likely to greenlight immediately:
- 012-tutor-apply (REQ-001 silent apply S2; + REQ-002 sticky-rail rides along)
- 013-tutee-dashboard-and-invitations (#207.1, #207.2, #208, sent-invitations — 2×S2)
- 014-interview-scheduling (REQ-001 admin-list-gap S2; the rest of 014 can follow in Wave 2 if scoped tight)
- 015-mobile-responsive (REQ-001 grid overflow S2; REQ-002/003 ride along — cheap)
- 017-admin-analytics-and-financial-safety (REQ-003 Mark-Paid confirm = financial safety; REQ-001/002 correctness)
- 018/REQ-002 (seeder tutor user id — unblocks T-08 visibility, trivial) + 018/REQ-003 (demo variety — sales-facing)

**Wave 2 — S3 robustness/user experience.**
- 011-auth-robustness (REQ-001 logout race, REQ-002 autofill, REQ-003 poll-pin)
- 014 remaining (REQ-002 auto-fill slot, REQ-003 No-Show clip, REQ-004 datetime picker, REQ-005 display-name)
- 016-error-copy-and-handling (REQ-001 humanise, REQ-002 no-handler 404)
- 018 remaining (REQ-004 /dashboard, REQ-006 CTA intent)

**Wave 3 — S4 / cosmetic / hygiene + founder-decision items.**
- 011/REQ-004 (#209.2 — only if founder overrides INT-109), 012/REQ-004 (#209.3 editability decision)
- 016/REQ-003 (create-admin fields), 016/REQ-004 (tel: encoding)
- 018/REQ-001 (WhatsApp toast OFF), 018/REQ-005 (step copy)
- confirm-and-close: 012/REQ-003 (#209.1) + 011/REQ-003 pin (both near-zero build)

## Open founder-decisions (surface via Ask User Question before the relevant REQ builds)

1. **#209.2 duplicate-phone error (011/REQ-004)** — keep the generic 422 (anti-enumeration/Personal Data Protection Act, INT-109 — the
   default) OR make it specific and accept the phone-enumeration + Personal Data Protection Act-disclosure risk. **Conflicts with an existing
   security control; founder must explicitly choose.**
2. **#209.3 tutor phone/email editability (012/REQ-004)** — immutable-by-design (explain in user interface, default) OR editable
   via a verified-change flow (follow-up spec).
3. **017/REQ-001 conversion cohort + REQ-002 avg-commission definition** — business-analyst proposes; founder/BA
   confirm the business-meaningful metric definitions before build.
4. **Wave scope** — confirm Wave 1 is the immediate scope, or pull more forward.

## Governance

Every spec stays **DRAFT**. Per CLAUDE md SDD: the Boss convenes a **scoped team-review** on each DRAFT (the named
counterparts + Boss audit) → flips status: APPROVED → dispatches the owning specialists (orchestrator does NOT
build) → evidence under private evidence grouped by specification and requirement → /team-review (full panel) → Boss SHIP →
status: HISTORICALLY MARKED IMPLEMENTED + requirements traceability matrix update + push. Founder-decision REQs are resolved before their REQ enters build (the rest of
the spec proceeds).
