# Wave 2 (S3 set) — build scope, overlap map, lane split

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> Author: requirements-engineer. Date: 2026-06-22. Inputs: FIX-PLAN md, CONSOLIDATED md, WAVE1-SPEC-REVIEW md,
> specs 011–018 (grounded against the real source — file:line cited). Wave 1 = S2 / money / safety (012/013/014-R1/
> 015-R1/017-R3/018-R2). **Wave 2 = the S3 robustness/user experience set** below. S4 / cosmetic items are **Wave 3 — defer**.
>
> All Wave-2 spec slices stay **status: APPROVED-pending** — the Boss re-gates each slice (scoped team-review + audit)
> before build. The two founder overrides (011/REQ-004, 012/REQ-004) are folded into their specs (v1.1 / v1.2).

---

## 1. Wave-2 S3 REQ list (spec/REQ → finding → owner → the BLOCKING test)

| Spec/REQ | Sev | Finding (grounded) | Lane | Owner using s | BLOCKING test |
|---|---|---|---|---|---|
| **011/REQ-001** | S3* | Logout races the Spec-004 silent-refresh; an in-flight the refresh session operation (api implementation reference) can re-mint cookies after logout → "logout may not stick". backend revocation correct. | frontend | software-engineer · security-expert | RTL: logout cancels in-flight refresh + rotated cookies do NOT re-establish session; live: refresh→logout→old refresh cookie rejected (REFRESH REUSED/INVALID) |
| **011/REQ-002** | S3/S4 | Auth forms don't resist stale autofill; a forced password-change auto-submitted from autofill. | frontend | frontend-expert | Component: correct autocomplete per field (current-password/new-password); NO auto-submit on programmatic value-set |
| **011/REQ-003** | S4 | Logged-out unread-count poll storm — **ALREADY gated** (INT-166: Notification Bell implementation reference, auth Redirect implementation reference). | frontend | frontend-expert | RTL regression-pin: user is no value → 0 unread-count fetches; with user → enabled. (Confirm+pin, no re-impl.) |
| **011/REQ-004** | S3 | **FOUNDER OVERRIDE — MAKE IT SPECIFIC.** Duplicate-phone → specific "phone already registered" message (today generic 422, INT-109, Auth Service implementation reference) WITH retained per-IP rate-limit (121-125,176-177,233-234) + NEW constant-time/consistent-latency guard so it isn't a fast oracle. | **backend** (+small frontend error-map) | bff-specialist · software-engineer · security-expert · frontend-expert | backend: specific PHONE ALREADY REGISTERED code/message; rate-limit still rejects over-limit incl. dup-phone path; timing-consistency assertion (dup-phone branch not a fast path, security-expert sign-off); registration failed reason is phone conflict metric preserved |
| **012/REQ-004** | S3 | **FOUNDER OVERRIDE — BUILD SELF-EDIT + RE-VERIFY.** User edits email/phone on profile; change is PENDING + re-verified (email link/one-time passcode; phone via INT-082 WhatsApp/one-time passcode if enabled, **else pending-only — INT-082 is OFF**); old value stays until verified; audited; uniqueness at commit. | **backend-heavy** (+frontend, +migration) | software-engineer · security-expert · bff-specialist · uiux-designer · dba | backend: pending held; old-value-until-verify; verify→atomic swap + audit row; unverified/expired discarded; uniqueness at commit; phone stays pending while WhatsApp OFF (no silent adoption); live: email edit→pending→verify→commit |
| **014/REQ-002** | S3 | Scheduling should auto-fill the accepted timeslot. Data already on payload (Admin Interview Response implementation reference agreed Timeslot Id+timeslots) — **frontend-only pre-fill**, no backend task. | frontend | frontend-expert | RTL: schedule control pre-fills the accepted slot when agreed Timeslot Id present, empty otherwise |
| **014/REQ-003** | S3 | "Record No Show" clipped next to Mark Complete (Interview Detail implementation reference). user interface overflow. | frontend | frontend-expert · uiux-designer | Structural/visual: all action buttons fully visible at desktop AND 390px (none clipped) |
| **014/REQ-004** | S3 | Date/time picker unpleasant for all roles (#211). user experience of the schedule datetime control. | frontend | frontend-expert · uiux-designer | RTL/interaction on the new picker; live capture across roles |
| **014/REQ-005** | S3 | Raw handle tutor wei jie tan shown on Interview/Applicants (reads like a leaked id). tutor Display Name already on payload (Admin Interview Response implementation reference) — **frontend-only**. | frontend | frontend-expert | RTL: renders tutor Display Name, not the raw handle, on Interview + Applicants screens |
| **016/REQ-001** | S3 | Error toast shows raw Tutee post not found: `<uuid>` (entity+id leak). Service sites concatenate the uuid; frontend shows it verbatim. Humanise; keep trace Id Reference. | frontend + backend-copy | frontend-expert · software-engineer · bff-specialist | backend: not-found message carries no entity-name/uuid; frontend: api Error implementation reference maps to a humanised string; trace Id Reference retained |
| **017/REQ-001** | S3 | Interview conversion >100% (cohort mismatch, Analytics Service compute Conversion Rate). | backend | data-engineer · business-analyst · software-engineer | backend: conversion never exceeds 100%; cohort-consistent numerator⊆denominator over a fixture; BA-confirmed definition |
| **018/REQ-004** | S3 | /dashboard (no role prefix) → hard 404 (T-01). Redirect to role dashboard or login. | frontend | frontend-expert | RTL/route: /dashboard redirects by role (or to login when unauthenticated); no 404 |
| **018/REQ-006** | S3 | Generic /register + "Post a request" CTAs bounce to /login (CTA-intent). | frontend | uiux-designer · frontend-expert | RTL: unauthenticated primary CTAs route to login-first intent (not a register→login bounce) |

**Wave-2 totals:** 13 S3-class REQs across 6 specs (011, 012, 014, 016, 017, 018). Two are founder overrides
(011/REQ-004, 012/REQ-004) carrying net-new backend work; the rest are mostly frontend with two the private implementation folder items.

### Corrections vs the expected S3 set (grounded against the specs)
- **016/REQ-002 (no-handler 500→404) is S4, not S3** (016/requirements md). It is **Wave 3 — defer**, unless the
  Boss explicitly pulls it forward to ride 016/REQ-001 (same Global error handler implementation reference file — cheap to bundle).
  The Wave-2 016 slice is **REQ-001 only**.
- **017/REQ-002 (avg-commission calc) is S4, not S3** (017/requirements md). The Wave-2 017 slice is **REQ-001
  only** (conversion >100%). REQ-002 → **Wave 3 — defer** (or bundle into 017 if the Boss wants one analytics pass).
- **017/REQ-003 (Mark-Paid confirm) shipped in Wave 1** (financial safety, APPROVED at the Wave-1 gate). Not Wave 2.
- 014/REQ-002 and REQ-005 are **frontend-only** (data already on Admin Interview Response) — do NOT dispatch a backend agent
  (the WAVE1-SPEC-REVIEW "phantom backend task" caution).
- 011/REQ-003 is a **confirm-and-pin** (already fixed, INT-166), not a re-implementation.

### Wave 3 — defer (S4 / cosmetic / hygiene)
T-06 sticky-Apply (012/REQ-002, S4) · T-03 step copy (018/REQ-005, S4) · T-05 WhatsApp toast OFF (018/REQ-001, S4) ·
tel: escaping (016/REQ-004, S4) · ADM-400 create-admin field naming (016/REQ-003, S4) · 016/REQ-002 no-handler 404
(S4) · 017/REQ-002 avg-commission (S4) · 018/REQ-003 demo-data variety (Could; shipped-or-Wave-1 per FIX-PLAN) ·
012/REQ-003 + 011/REQ-003 confirm-and-close (near-zero build).

---

## 2. File-overlap serialization map

This repo gives agents NO worktree isolation — same-file edits collide. Parallelize only non-overlapping files;
serialize anything that shares a file (with each other OR with shipped Wave-1 code).

### Intra-Wave-2 overlaps (serialize these)
| File | Wave-2 REQs touching it | Rule |
|---|---|---|
| api implementation reference + auth Store implementation reference | **011/REQ-001** | 011-internal; ALSO shared with **012's** api-client reads → coordinate one edit window (FIX-PLAN §overlaps). In Wave 2, 012/REQ-004 adds new endpoints to api Routes implementation reference/api client — **serialize 011/REQ-001 then 012/REQ-004 on the frontend api client.** |
| api Error implementation reference (frontend error map) | **016/REQ-001** (humanise) + **011/REQ-004** (add PHONE ALREADY REGISTERED entry) | Both edit the frontend error map → **serialize 011/REQ-004's frontend entry and 016/REQ-001** on api Error implementation reference. |
| Auth Service implementation reference | **011/REQ-004** only (within Wave 2) | No other Wave-2 REQ edits it → safe, but it is a Wave-1-adjacent auth hot file: one backend build at a time. |
| Interview Detail implementation reference | **014/REQ-002, REQ-003, REQ-004** (and Wave-1 014/REQ-001 was Interview Queue implementation reference) | All three Wave-2 014 REQs touch Interview Detail implementation reference → **sequential within 014** (one agent, or serialized). |
| Job Applicants implementation reference | **014/REQ-005** | 014-only. |
| Analytics implementation reference / Analytics Service implementation reference | **017/REQ-001** | 017-only (REQ-002/003 not in Wave 2). |
| Profile implementation reference | **012/REQ-004** (self-edit email/phone) | 012-internal in Wave 2. **Cross-wave collision: Wave-3 018/REQ-001 (WhatsApp toast OFF) also edits Profile implementation reference** — if 018/REQ-001 is ever pulled forward, serialize with 012/REQ-004. |
| App implementation reference (routing) | **018/REQ-004** (/dashboard redirect) + **018/REQ-006** (CTA intent may touch route/nav) | Both 018 → sequential within 018. |
| Liquibase db/changelog/** | **012/REQ-004** (pending-contact migration) | One ledger — single new changeset; no number collision with other Wave-2 work (no other Wave-2 REQ adds a migration). |

### Overlap with HISTORICALLY MARKED IMPLEMENTED Wave-1 code (re-verify, serialize the build window)
- **Auth Service implementation reference** — Wave-1 did not edit it (Wave-1 auth was frontend); 011/REQ-004 is the first Wave-2 toucher. Safe.
- **Interview Queue implementation reference** — shipped in Wave 1 (014/REQ-001 tab-map). Wave-2 014 edits the sibling
  Interview Detail implementation reference, NOT Interview Queue implementation reference → disjoint, but run after the Wave-1 014 build is merged.
- **Commission implementation reference** — shipped in Wave 1 (017/REQ-003 Mark-Paid confirm). Wave-2 017/REQ-001 edits
  Analytics implementation reference/Analytics Service implementation reference, NOT Commission implementation reference → disjoint.
- **Dev Data Seeder implementation reference** — shipped in Wave 1 (018/REQ-002). No Wave-2 REQ edits it → no collision.
- **api implementation reference/auth Store implementation reference** — Wave-1 012 (apply 403 handling) read the api client; Wave-2 011/REQ-001 + 012/REQ-004
  both edit/extend it → **serialize on the api client across both waves; rebase Wave-2 on the merged Wave-1 api implementation reference.**

---

## 3. backend-vs-frontend lane split (parallelize across subtrees, serialize builds within a subtree)

Per CLAUDE md: parallelize **non-overlapping subtrees** (a backend lane vs a frontend lane); **builds within one subtree run
sequentially** (one the documented operational check, one npm build at a time). At most one backend build + one frontend build concurrently.

**backend lane (intuition-backend — run sequentially within it):**
- **011/REQ-004** — Auth Service implementation reference specific phone-conflict + rate-limit + constant-time guard + tests.
- **012/REQ-004** — pending-contact model: Liquibase migration + edit/verify endpoints + atomic-swap + audit + tests
  (the heaviest backend item; candidate for its own slice; phone half gated on INT-082 / fallback decision).
- **016/REQ-001 (backend-copy half)** — strip entity-name/uuid from the not-found message at the service sites.
- **017/REQ-001** — Analytics Service compute Conversion Rate cohort fix + tests.

**frontend lane (intuition-frontend — run sequentially within it; serialize the shared-file pairs above):**
- **011/REQ-001** (api implementation reference/auth Store — serialize with 012/REQ-004) · **011/REQ-002** (auth forms) · **011/REQ-003**
  (poll-pin) · **011/REQ-004 frontend error-map entry** (serialize with 016/REQ-001 on api Error implementation reference).
- **012/REQ-004 frontend** (Profile edit + pending banner).
- **014/REQ-002, REQ-003, REQ-004, REQ-005** (all Interview Detail implementation reference/Applicants — sequential within 014).
- **016/REQ-001 frontend** (api Error implementation reference humanise — serialize with 011/REQ-004's entry).
- **018/REQ-004** (/dashboard redirect) + **018/REQ-006** (CTA intent) — sequential within 018 (App implementation reference).

**Suggested concurrency:** run the backend lane (011/REQ-004 → 017/REQ-001 → 016-copy → 012/REQ-004) sequentially in the
backend subtree while the frontend lane runs sequentially in the frontend subtree, observing the api implementation reference and api Error implementation reference
serialization pairs. 012/REQ-004 spans both subtrees + a migration — schedule it as its own slice with the backend part
landing before the frontend part.

---

## 4. Founder-decisions / infra-dependencies surfaced (for Ask User Question before the relevant REQ builds)

1. **011/REQ-004 (RESOLVED — make-it-specific).** Founder accepted the residual phone-enumeration + Personal Data Protection Act-disclosure
   risk; the override is conditioned on security-expert keeping the path non-trivial to abuse (retain rate-limit +
   add a constant-time/consistent-latency guard — **net-new, no timing guard exists today**). Remaining design
   choice for bff-specialist: status code 409 (to match EMAIL ALREADY REGISTERED, Auth Service implementation reference) vs
   422, and the code name (PHONE ALREADY REGISTERED). Security-expert signs off the guard at the gate.
2. **012/REQ-004 (RESOLVED — build self-edit + re-verify) — OPEN INFRA-DEPENDENCY.** **INT-082 WhatsApp/one-time passcode is OFF.**
   The phone re-verification has no live channel. **Founder/ops must choose the WhatsApp-OFF phone fallback:**
   (a) pending-only state (default — new phone never adopted until INT-082 is enabled), (b) an email-link
   confirmation of the phone change, or (c) admin-assisted. The **email half is fully buildable now**; the Boss may
   scope 012/REQ-004 as its own Wave-2 slice and ship the email path while deferring the phone path behind INT-082.
3. **017/REQ-001 conversion definition.** business-analyst/data-engineer must confirm the business-meaningful cohort
   (numerator ⊆ denominator) before build so the fix targets the right metric, not just clamps to ≤100%.
4. **Wave-2 scope confirmation.** Confirm the 13-REQ S3 set above is the Wave-2 cut, and that the two S4 items the
   expected set mis-tagged (016/REQ-002, 017/REQ-002) stay in Wave 3 unless deliberately bundled.

---

## Summary

- **Two founder overrides folded in:** 011 → v1.1 (REQ-004 = specific duplicate-phone message + retained rate-limit +
  new constant-time guard; founder accepts residual enumeration/Personal Data Protection Act risk). 012 → v1.2 (REQ-004 = self-edit email/
  phone with re-verification; old value stays until verified; audited; phone pending-only while WhatsApp OFF). Both
  specs flipped DRAFT/APPROVED → **APPROVED-pending** (Boss re-gates the Wave-2 slices).
- **Wave-2 (S3) = 13 REQs** across 011, 012, 014, 016, 017, 018 — table §1. Wave-3 (S4/cosmetic) deferred, incl. two
  items the expected set over-rated to S3 (016/REQ-002, 017/REQ-002).
- **New founder-decision / infra-dependency:** phone re-verification (012/REQ-004) has no channel — **INT-082
  WhatsApp/one-time passcode is OFF**; the email half ships now, the phone half is pending-only / deferred pending the founder's
  fallback choice. 011/REQ-004's constant-time latency guard is net-new work (no timing equalisation exists today).
