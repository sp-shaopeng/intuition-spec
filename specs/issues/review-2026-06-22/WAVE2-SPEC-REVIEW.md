# Wave 2 (S3 set) — scoped SDD spec-review (pre-build gate)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> Mode: team-review **Spec Review** (pre-build; review-only, no production code).
> Convened by the Boss for the Wave-2 S3 slices in WAVE2-SCOPE md. Scoped seats per REQ-domain
> (security-expert, software-engineer, bff-specialist, dba, frontend-expert, data-engineer, business-analyst,
> uiux-designer, qa-engineer, tech-lead) + Boss sign-off. Grounded against the real source on 2026-06-22
> (file:line cited). Founder decisions on 011/REQ-004 and 012/REQ-004 folded in.
>
> **Date:** 2026-06-22 · **Reviewer of record:** Boss-orchestrated scoped panel.

---

## 0. Scope confirmation

The Wave-2 cut in WAVE2-SCOPE md §1 is **confirmed** as the 13-REQ S3 set across 6 specs (011, 012, 014, 016,
017, 018). The two mis-tagged S4 items (016/REQ-002 no-handler 404, 017/REQ-002 avg-commission) stay **Wave 3**;
016/REQ-001 and 017/REQ-001 are the only slices pulled from those specs. 012/REQ-004 is scoped as **its own
Wave-2 slice** (heaviest item, spans backend+frontend+migration). No scope correction needed.

**Source grounding (verified this session):**
- Auth Service implementation reference (tutee) / 244-245-region (tutor) — email → 409 EMAIL ALREADY REGISTERED
  (183-185); phone → the registration phone conflict operation generic 422 REGISTRATION FAILED (INT-109 fold,
  comment block 63-81). Rate-limit @Value ratelimit register.limit (121-125), enforced at both
  register entrypoints. **exists By Email/exists By Phone run as early-return guards BEFORE build User/password
  hashing — so today the dup-phone branch IS a fast path; the timing guard is genuinely net-new.**
- Admin Interview Response implementation reference — carries agreed Timeslot Id, timeslots, tutor Display Name, tutor Public Handle
  (verified present). **014/REQ-002 + REQ-005 are frontend-only — confirmed, no backend field task.**

---

## 1. Per-REQ verdicts

### 011/REQ-001 — Logout vs silent-refresh race → **APPROVED with conditions**

**Lenses:** software-engineer (correctness), security-expert (session-resurrection), frontend-expert (frontend revoke), qa-engineer.

The spec correctly grounds the race (api implementation reference single-flight the refresh session operation, auth Store implementation reference the logout operation,
no coordination today). The EARS (abort in-flight + block new refresh + await submit operation for auth logout before nav +
discard rotated cookies + fail-closed) is complete and the design names a concrete mechanism (logout-generation flag,
optional Abort Controller). The three Gherkin scenarios cover the happy path, the revocation-ordering path, AND the
fail-closed network-error guard.

- **Software-engineer (Jordan):** the design's "logout-generation flag checked in the refresh session operation + the
  interceptor" is the right pattern, but the spec must make ONE thing non-optional that it currently lists as
  "Optionally": the in-flight refresh's **rotated-cookie effect must be discarded even if the refresh web request protocol response
  already arrived** (cookies are set by the browser from Set-Cookie before JS sees the promise resolve). A
  generation-flag that only prevents *replay* does not un-set cookies the browser already wrote. **Condition C1:**
  the design must state how already-written rotated cookies are neutralised (the subsequent submit operation for auth logout
  revokes the family server-side AND clears cookies — so the ordering "await logout AFTER the in-flight refresh
  settles, or abort the refresh before its response is processed" must be explicit, not "optionally abort").
- **Security-expert (Casey):** acceptable — backend revocation is the real guarantee; frontend ordering is defence in
  depth. No SEND-BACK.

**Conditions:**
- [ ] **C1** — design states the deterministic cookie-neutralisation: either (a) Abort Controller cancels the
  in-flight /auth/refresh before its Set-Cookie lands, OR (b) the logout operation awaits the in-flight refresh to settle
  THEN runs submit operation for auth logout (which clears+revokes), so rotated cookies cannot outlive logout. "Logout wins"
  must be proven by the live test, not assumed.
- [ ] **C2 (BLOCKING test):** the live Playwright capture must assert the **old refresh cookie is rejected
  (REFRESH REUSED/REFRESH INVALID)** after logout — the RTL test alone (jsdom) cannot prove the server revoked
  the family. This is already named in Required Evidence; pin it as blocking.

**Boss verdict:** ✅ SHIP (build) on C1+C2.

---

### 011/REQ-002 — Autofill suppression → **APPROVED**

**Lenses:** frontend-expert, qa-engineer.

Correct per-field autocomplete semantics (username/email, current-password on login, new-password on
register/reset/forced-change), no auto-submit on programmatic value-set, no blanket autocomplete is off. Scenarios
cover the forced-change auto-submit (the actual reported defect) and the new-vs-current-password distinction.
BLOCKING test (component asserts attribute per field + no auto-submit on programmatic set) is named and adequate.

- **Frontend-expert (Sam):** one addition — the design lists the form set but does not name the **forced
  password-change form's file path**. The audit must enumerate every form (login, register tutee, register tutor,
  reset, forced-change) by path so none is silently missed. Non-blocking (covered by "Audit the auth forms").

**Boss verdict:** ✅ SHIP (build). No conditions.

---

### 011/REQ-003 — Poll-pin (confirm + pin) → **APPROVED**

**Lenses:** frontend-expert, qa-engineer.

Correctly scoped as a **confirm-and-pin**, not a re-implementation — Notification Bell implementation reference and
auth Redirect implementation reference already gate it (INT-166). The BLOCKING regression test (user is no value →
0 fetches; with user → enabled) is the right artifact. No risk.

**Boss verdict:** ✅ SHIP (build). No conditions.

---

### 011/REQ-004 — Duplicate-phone SPECIFIC message + anti-enumeration hardening (FOUNDER OVERRIDE) → **APPROVED with conditions**

**Lenses:** security-expert (THE owner — timing guard + residual-risk sign-off), bff-specialist (contract), software-engineer, frontend-expert, qa-engineer.

The founder override is faithfully folded (v1.1 changelog, REQ-004 rewrite, design §REQ-004). The spec correctly
identifies that **no timing guard exists today** and that the existing exists By Phone early-return IS a fast path —
this review **independently verified that against Auth Service implementation reference** (the phone check fires before
build User/password-hash, so the dup-phone branch returns measurably faster than success today). The spec is the
strongest of the override pair. But the highest-risk element — the constant-time guard — is under-specified to be
buildable and testable as written.

- **Security-expert (Casey) — the gate owner here:** "consistent latency … e g. always run the same
  hashing/validation work, or a constant floor" is a **menu, not a design.** A timing-invariance acceptance
  criterion that says latencies are "statistically indistinguishable" is **untestable without a defined method,
  threshold, and sample protocol.** I will NOT sign off the residual-risk acceptance until the design pins the
  concrete mechanism. **Condition C1.** Note also: the early-return on exists By Email (also a fast path) means the
  email branch and the phone branch and the success branch are THREE different latency classes today — equalising
  only phone-vs-success still leaves email distinguishable; the guard must cover the whole pre-insert decision, or
  the spec must explicitly accept that email membership stays enumerable (it already is specific, so that is
  consistent — but say so).
- **bff-specialist (Riley):** the open contract choice (409 vs 422, code PHONE ALREADY REGISTERED) **must be
  decided in the spec before build, not "at design."** I rule **409 PHONE ALREADY REGISTERED** to mirror the
  existing 409 EMAIL ALREADY REGISTERED (Auth Service implementation reference) — symmetry is the contract value here, and
  422 is for validation-shape failures, not a resource conflict. **Condition C2:** pin 409 + the exact code/message
  in design md and add the frontend api Error implementation reference entry (serialize with 016/REQ-001 per the overlap map).
- **Data-engineer (Blake):** keep the registration failed reason is phone conflict metric (79-81) — the spec does.
  Good. One add: the constant-time floor must NOT be so long it becomes a Do S amplifier (a 500ms floor × the
  rate-limit = cheap to tie up a thread). The floor value is a design parameter to bound. Non-blocking note.

**Conditions:**
- [ ] **C1 (security-expert, BLOCKING)** — design md must specify the timing-equalisation mechanism concretely:
  WHICH approach (constant time-floor on the handler via a deadline-sleep, OR always-run-the-same-work by performing
  a dummy password-hash on the reject path), the **target floor/budget**, WHICH branches it covers
  (phone-conflict, email-conflict, success — or an explicit statement that email stays specific), and the
  **test method**: a deterministic assertion (e g. inject a clock / assert the same code path executes the same
  cost), NOT a flaky wall-clock "statistically indistinguishable" timing measurement in continuous integration. Security-expert signs
  off the residual-risk box only after this is in the spec.
- [ ] **C2 (bff-specialist)** — pin **409 PHONE ALREADY REGISTERED** + the human message in design md; frontend error-map
  entry serialized with 016/REQ-001 on api Error implementation reference.
- [ ] **C3 (BLOCKING test)** — dup-phone timing-invariance test per C1's chosen method; rate-limit test that the
  over-limit path rejects with RATE LIMIT EXCEEDED **before** any phone membership is revealed (ordering
  assertion — the rate-limit gate runs first, 176-177/233-234 confirm it does); preserved
  registration failed reason is phone conflict metric.

**Boss verdict:** ⚠️ SEND-BACK on the design until C1 lands (the timing guard is the founder's explicit override
condition and is currently a menu, not a spec); ✅ SHIP once C1+C2 are written into design md. The REQUIREMENT is
approved; the DESIGN is incomplete on the one net-new, security-critical element.

---

### 012/REQ-001 — Apply-gate never silent + frontend↔backend completeness reconciled → **APPROVED** (Wave-1-carried)

**Lenses:** bff-specialist, software-engineer, frontend-expert, qa-engineer.

Already APPROVED at the Wave-1 gate; the design's "backend returns profile Complete+missing Fields from ONE shared
predicate; delete the frontend is Profile Complete and its qualifications is at least 10 floor" is the correct single-source-of-
truth fix (grounded: frontend checks location Region, backend checks location Planning Areas — real drift). BLOCKING parity
test + "403 never silent" are pinned. No change for Wave 2. Listed here only because the frontend api-client edit
**serializes with 012/REQ-004 and 011/REQ-001** per the overlap map.

**Boss verdict:** ✅ SHIP (already approved).

---

### 012/REQ-002 — Sticky-Apply rail state → **APPROVED**

**Lenses:** frontend-expert.

S4, well-scoped (Assignment Detail implementation reference jump-link must mirror Tutor Apply Control state). Depends on
REQ-001's lifted state. RTL across open/applied/gated states is the right test. No risk.

**Boss verdict:** ✅ SHIP (build).

---

### 012/REQ-003 — Tutor taxonomy confirm + close #209.1 → **APPROVED**

**Lenses:** frontend-expert.

Confirm-and-close; already consumes shared SUBJECT OPTIONS (spec 009). Test asserts options derive from the
shared constant. No risk.

**Boss verdict:** ✅ SHIP (build).

---

### 012/REQ-004 — Self-edit phone/email + re-verification (FOUNDER OVERRIDE) → 🚨 **SEND-BACK** (spec is SUPERSEDED by the founder's email-link decision)

**Lenses:** security-expert (THE highest-risk item — account-takeover via contact change), software-engineer (token/verify flow), bff-specialist (endpoints), dba (pending-contact migration + uniqueness-at-commit), uiux-designer, qa-engineer.

**This is a hard SEND-BACK.** The spec as written (requirements md, design md) still says the
phone path is **"INT-082 WhatsApp/one-time passcode if enabled, else a pending-verification state … the founder/ops decide
whether an alternate fallback channel … is offered"** and lists the fallback as an **open infra-dependency**
(requirements md, design md, spec md changelog 1.2, Open-question #1). **The founder has now
DECIDED that open question:** editing phone → a confirmation link to the user's **already-verified account email**
→ clicking commits the new phone (email-link fallback for PHONE re-verify; NOT defer, NOT WhatsApp). The spec must
be rewritten to mandate the email-link-fallback design; "deferred / pending-only behind WhatsApp" is no longer the
behaviour.

What the rewritten 012/REQ-004 (→ v1.3) MUST specify before it can be approved:

- **dba (Morgan) — pending-contact model + uniqueness-at-commit (BLOCKING):** the design offers "columns OR a
  contact change request table" as a choice. **Pick the table** — it cleanly holds user Id, field (email|phone),
  old value, new value, channel (EMAIL LINK|WHATSAPP), token hash, status, expires at, created at, consumed at,
  audit and supports both field types + a clean audit trail. Decide and spec: (a) Liquibase changeset number (one
  ledger, leave the reserved-gap convention); (b) the token is stored **hashed** (never plaintext); (c)
  **uniqueness enforced at COMMIT** against the live users email/users phone unique indexes inside the swap
  transaction (a pending row colliding with a now-taken value must 409 at verify, reusing 011/REQ-004's
  PHONE ALREADY REGISTERED / the email conflict) — the spec says "uniqueness at commit" but must name HOW
  (re-check inside the a database transaction swap, not at request time, to close the TOCTOU). Soft-delete conventions:
  superseded/expired requests are status-flagged, not row-deleted.
- **security-expert (Casey) — anti-account-takeover (THE risk, BLOCKING):** the email-link-fallback for PHONE
  re-verify creates a specific takeover chain: **a hijacked session changes the phone via a link sent to the
  account email — if the attacker also controls the email, or if the link can be replayed, the recovery contact is
  re-pointed silently.** The spec MUST require: (1) **step-up re-auth** (current password or fresh-session
  re-auth) to *initiate* any contact change — the design mentions this but it must be a hard REQ, not a "should";
  (2) the email-link is **single-use + short expiry** (spec the expiry period, e g. 30 min, and single-use enforced via
  consumed at); (3) **notify the OLD email AND old phone** that a change was requested, with a revoke/"this
  wasn't me" affordance, BEFORE commit — so the legitimate owner can stop it; (4) the email-link for a PHONE change
  must clearly state in its body **which phone it is authorising** (anti-confused-deputy); (5) an unverified/expired
  request never mutates users phone. Without all five spec'd, I SEND-BACK.
- **software-engineer (Jordan) — token + verify flow:** spec the issuance (secure-random token, hashed at rest,
  bound to user Id, field, new value), the verify route's atomic swap (a database transaction: re-check uniqueness →
  write new value → mark request consumed → audit row → invalidate any other pending request for the same field),
  and the lazy-expire / sweep for stale requests. The "atomic swap + audit" is asserted but the transaction
  boundary + the consume-once ordering must be in the design (a verify replay must be a no-op).
- **bff-specialist (Riley):** name the two endpoints in api Routes implementation reference (no magic strings) — a POST to
  initiate a contact change (step-up-guarded) returning the pending state, and a verify route the email link
  targets (e g. GET/POST account contact-change verify?token= interface operation[REDACTED]). Spec the profile GET's new
  pending Email/pending Phone (or pending Contact Change) fields so the frontend can render "verification pending".
  Decide: is the verify link a GET (clickable from an email) that then commits, or a GET that lands a page which
  POSTs? (Prefer a landing page + explicit confirm click to avoid email-scanner prefetch auto-committing — a real
  bug class; spec it.)
- **uiux-designer (Jamie):** the pending-state banner ("current email/phone stays active until you confirm"), the
  email-link landing/confirm page, and — importantly — the **phone-via-email-link copy** ("we sent a confirmation
  link to your account email to authorise your new phone number") must be designed; this indirection is confusing
  if unexplained.
- **qa-engineer (Reese) — BLOCKING tests:** (1) email-link/token **single-use** (second use is rejected) +
  **expiry** (expired token rejected, old value intact); (2) **email-scanner prefetch does not auto-commit** (the
  verify must require an explicit action, not a bare GET side-effect); (3) old value used for all auth/contact ops
  while pending; (4) verify → atomic swap + audit row; (5) **uniqueness at commit** (a value taken between
  initiate and verify → 409, old value intact); (6) step-up required to initiate (no-password → rejected);
  (7) phone change commits via the email link (the new fallback) — NOT held pending-only.

**Boss verdict:** 🚨 SEND-BACK. The spec encodes the SUPERSEDED "phone deferred/pending-only behind WhatsApp"
behaviour and lists the fallback as an unresolved open question; the founder has decided **email-link fallback for
phone re-verify**. requirements-engineer rewrites 012/REQ-004 → **v1.3** mandating the email-link flow (token
issuance, single-use + expiry period, the verify landing route, the contact change request migration, uniqueness-at-commit,
step-up + old-contact notification + anti-takeover) with the dba/security-expert conditions above as hard REQs.
This is the heaviest and highest-risk Wave-2 item — it does not enter build until v1.3 passes a re-gate.

---

### 014/REQ-002 — Auto-fill accepted slot (frontend-only) → **APPROVED**

**Lenses:** frontend-expert.

Verified frontend-only: agreed Timeslot Id + timeslots ARE on Admin Interview Response (resolve the slot by id). No backend
task (the WAVE1 "phantom backend task" caution holds). RTL pre-fill + empty-guard is correct. Depends on REQ-004's
control. No risk.

**Boss verdict:** ✅ SHIP (build).

---

### 014/REQ-003 — No-Show clipped user interface → **APPROVED**

**Lenses:** frontend-expert, uiux-designer.

Interview Detail implementation reference 5-button row overflow; wrap/stack/group so none clips. BLOCKING test (all actions
visible at desktop AND 390px) carries the standing mobile-web gate. No risk.

**Boss verdict:** ✅ SHIP (build).

---

### 014/REQ-004 — Datetime-picker user experience → **APPROVED with a note**

**Lenses:** uiux-designer, frontend-expert, qa-engineer.

EARS is sound (usable, Singapore time defaults, mobile-friendly, keyboard-accessible, past-datetime rejected, **wire format
unchanged** — the critical non-regression). The picker choice (native date-and-time input vs library)
is a non-blocking open question with a sensible default lean (native, zero-dep, a11y, mobile). The one hard
condition: the BLOCKING test must assert the **backend wire format is byte-unchanged** (the spec says so — pin it as the
gate so a picker swap can't silently change the payload).

- [ ] **Note/condition:** unchanged-wire-format assertion is the blocking guard; past-datetime guard + a11y/keyboard
  test included.

**Boss verdict:** ✅ SHIP (build); pin the wire-format-unchanged test.

---

### 014/REQ-005 — Raw-handle display (frontend-only) → **APPROVED**

**Lenses:** frontend-expert.

Verified frontend-only: tutor Display Name IS on Admin Interview Response. Render the display name, fall back to a
human-friendly label (never the raw tutor … slug) on Interview + Applicants. The fallback guard scenario is
present and correct. No risk.

**Boss verdict:** ✅ SHIP (build).

---

### 016/REQ-001 — Humanise raw "not found: `<uuid>`" → **APPROVED**

**Lenses:** frontend-expert (code→copy map), software-engineer/bff-specialist (backend message hygiene), qa-engineer.

Grounded: service sites concatenate the id (Tutee post not found: + id); frontend shows it verbatim. The design
(frontend code→friendly-copy map, fall back to server message only when unmapped, **keep trace Id Reference**, no CODE
changes) is correct. The open question (frontend-map-only vs also scrub backend message) is resolved sensibly: the frontend map is
the user-facing fix, backend id-in-message is a logging detail — bff confirms no consumer depends on the raw message
text. BLOCKING test (mapped code → friendly copy, no uuid, Reference present; unmapped → safe generic) is adequate.

- **bff-specialist (Riley):** approve, with the recorded check that the frontend error map edit on api Error implementation reference
  **serializes with 011/REQ-004's PHONE ALREADY REGISTERED entry** (same file, overlap map §intra-Wave-2).

**Boss verdict:** ✅ SHIP (build).

---

### 017/REQ-001 — Conversion >100% cohort fix → **APPROVED with a founder/BA condition**

**Lenses:** data-engineer, business-analyst (definition — the open call), software-engineer, qa-engineer.

Grounded: the compute conversion rate operation = completed/requested over a
date window where completed and requested are different cohorts → >100% is real. The fix is correct in spirit
(numerator ⊆ denominator, ratio in [0,100%], zero-requested → 0% guard, no Na N).

- **Data-engineer (Blake) + Business-analyst (Dana) — THE open item:** the **cohort definition is left unpinned**
  (requirements md "IF the chosen definition keeps a fixed-window numerator …"; Open-question #1 lists
  THREE candidates: completed-of-requested-in-period (true funnel, lags) / all-time funnel / clamp+relabel). These
  are **not equivalent** — they answer different business questions and produce different numbers. The
  spec must NOT enter build with three live options. **This needs a founder/BA decision before build.** Our
  recommendation: **"of the interviews REQUESTED in the period, the fraction that have reached COMPLETED (as of
  now)"** — a true cohort funnel, numerator strictly ⊆ denominator, naturally ≤100%, and the most honest "did the
  interviews we started convert?" KPI. Clamp-and-relabel is a band-aid that hides the cohort bug and should be
  rejected. **Condition C1.**
- **qa-engineer (Reese):** the BLOCKING test must include **the exact prior 400% fixture now proven ≤100%** plus
  the zero-requested guard — pin it against whatever definition C1 lands.

**Conditions:**
- [ ] **C1 (founder/BA — needs a call):** pin ONE cohort definition (recommended: completed-of-requested-in-period,
  numerator ⊆ denominator) in requirements md/design md; reject clamp-only. The denominator is the decision the
  founder/BA must confirm.
- [ ] **C2 (BLOCKING test):** the prior 400% scenario → ≤100% under the chosen definition; zero-requested → 0% (no
  div-by-zero / Na N); numerator ⊆ denominator asserted over a fixture.

**Boss verdict:** ✅ SHIP (build) once C1 (the cohort definition) is pinned by the founder/BA. Until then the
calc target is ambiguous — do not let it enter build clamp-first.

---

### 018/REQ-004 — /dashboard 404 → role redirect → **APPROVED**

**Lenses:** frontend-expert.

Grounded: App implementation reference, role dashboards at 108/123, no /dashboard route. Redirect by role
when authed, to /login (preserving return) when not. Both scenarios present. No risk.

**Boss verdict:** ✅ SHIP (build).

---

### 018/REQ-006 — CTA-intent /register vs /login → **APPROVED with a scoping note**

**Lenses:** uiux-designer, frontend-expert.

Generic auth-required CTAs → login by default (register reachable from login), consistent with M17; the explicit
create-account CTA still → register (regression guard present). Sound.

- **uiux-designer (Jamie):** the one gap — Open-question #2 ("enumerate which CTAs are in scope") must be closed in
  the spec so the change is consistent and doesn't catch the explicit create-account CTA. **Non-blocking condition:**
  the design must list the in-scope CTAs (audit "Post a request" + generic register CTAs) before build so the RTL
  test enumerates them.

**Boss verdict:** ✅ SHIP (build); enumerate the in-scope CTA list in the design.

---

## 2. Cross-cutting risks

1. **Account-takeover via contact-change (012/REQ-004) — HIGHEST RISK.** The founder's email-link fallback for
   phone re-verify routes a phone change through the account email. If a session is hijacked, or the email link is
   replayable / auto-fetchable by an email scanner, the recovery contact is silently re-pointed. **Mandatory
   controls (all must be spec'd in v1.3):** step-up re-auth to initiate, single-use + short-expiry period token, notify BOTH
   old email and old phone with a revoke affordance before commit, a confirm-click landing page (no bare-GET
   side-effect), and the link body naming the phone it authorises. This is the gate's primary blocker.

2. **Analytics conversion cohort definition (017/REQ-001) — NEEDS A FOUNDER/BA CALL.** Three non-equivalent
   definitions are live in the spec. Build must not start until one denominator is pinned. Recommended:
   completed-of-requested-in-period (numerator ⊆ denominator, naturally ≤100%); reject clamp-and-relabel.

3. **Timing-equalisation guard (011/REQ-004) is net-new and under-specified.** Verified against source that the
   dup-phone branch is a real fast path today. The override's whole security premise rests on a guard the design
   describes only as a menu. It must be pinned to a concrete, deterministically-testable mechanism before
   security-expert signs the residual-risk box — otherwise the founder's override condition isn't met.

4. **Shared-file serialization (process risk, not spec defect).** api Error implementation reference (011/REQ-004 + 016/REQ-001) and the
   frontend api-client (api implementation reference/auth Store implementation reference: 011/REQ-001 + 012/REQ-001 + 012/REQ-004) are edited by multiple Wave-2
   REQs. No worktree isolation → serialize those edit windows per WAVE2-SCOPE md §2. Flagged so the build
   sequencing honours it.

5. **Cross-wave conflict: 018/REQ-001 (Wave-3) vs 012/REQ-004 (Wave-2).** 018/REQ-001 says "suppress ALL WhatsApp
   verification messaging while OFF" on phone save; 012/REQ-004 now introduces an **email-link** verification on
   phone change that must NOT be suppressed by 018/REQ-001's blanket rule. When 018/REQ-001 is built (Wave 3), its
   suppression must be scoped to the WhatsApp/one-time passcode channel only, leaving the email-link flow intact. **Note for the
   012/REQ-004 v1.3 author and the Wave-3 018/REQ-001 author** — not a Wave-2 blocker.

---

## 3. BLOCKING tests (consolidated, per the gate)

- **011/REQ-001:** RTL logout-cancels-in-flight-refresh + cookies-not-re-applied; **live: refresh→logout→old refresh
  cookie rejected (REFRESH REUSED/INVALID)** (server-revocation proof, not jsdom).
- **011/REQ-002:** component asserts correct autocomplete per field + NO auto-submit on programmatic value-set.
- **011/REQ-003:** RTL regression-pin — user is no value → 0 unread-count fetches; with user → enabled.
- **011/REQ-004:** dup-phone timing-invariance (per the pinned method, deterministic not wall-clock); rate-limit
  rejects over-limit **before** membership revealed (ordering); registration failed reason is phone conflict preserved.
- **012/REQ-001:** frontend↔backend completeness parity over a field matrix; "403 never silent" (typed message, no bare
  "Access denied").
- **012/REQ-004 (v1.3):** email-link **single-use** + **expiry**; **email-scanner prefetch does not auto-commit**;
  old value used while pending; verify → atomic swap + audit row; **uniqueness-at-commit** (value taken mid-flight
  → 409, old intact); step-up required to initiate; **phone commits via email link (not held pending-only)**.
- **014/REQ-001 (Wave-1):** enum-driven set-equality (all 10 Interview Status placed, EXPIRED in history).
- **014/REQ-002:** RTL pre-fill the accepted slot when agreed Timeslot Id present, empty otherwise.
- **014/REQ-003:** all action buttons fully visible at desktop AND 390px.
- **014/REQ-004:** picker a11y/keyboard + past-datetime guard + **backend wire-format unchanged**.
- **014/REQ-005:** renders tutor Display Name, not the raw handle (+ graceful fallback).
- **016/REQ-001:** mapped code → friendly copy (no uuid) + trace Id Reference; unmapped → safe generic.
- **017/REQ-001:** prior 400% fixture now ≤100% under the pinned cohort; zero-requested → 0%; numerator ⊆ denominator.
- **018/REQ-004:** /dashboard redirects by role / to login; no 404.
- **018/REQ-006:** generic CTA → login; explicit create-account → register (regression).

---

## 4. Boss verdict summary (per REQ)

| REQ | Verdict | Gate |
|---|---|---|
| 011/REQ-001 | ✅ SHIP w/ conditions | C1 cookie-neutralisation explicit; C2 live old-cookie-rejected |
| 011/REQ-002 | ✅ SHIP | — |
| 011/REQ-003 | ✅ SHIP | — |
| 011/REQ-004 | ⚠️ SEND-BACK (design) → ✅ on C1 | C1 timing-guard concrete+testable; C2 pin 409 PHONE ALREADY REGISTERED; C3 tests |
| 012/REQ-001 | ✅ SHIP | (Wave-1-approved; serialize api-client) |
| 012/REQ-002 | ✅ SHIP | — |
| 012/REQ-003 | ✅ SHIP | — |
| 012/REQ-004 | 🚨 **SEND-BACK** | Rewrite → v1.3 email-link fallback (token/expiry period/single-use, verify landing route, contact change request migration, uniqueness-at-commit, step-up + old-contact notify + anti-takeover) |
| 014/REQ-002 | ✅ SHIP | frontend-only confirmed |
| 014/REQ-003 | ✅ SHIP | — |
| 014/REQ-004 | ✅ SHIP | pin wire-format-unchanged test |
| 014/REQ-005 | ✅ SHIP | frontend-only confirmed |
| 016/REQ-001 | ✅ SHIP | serialize api Error implementation reference w/ 011/REQ-004 |
| 017/REQ-001 | ✅ SHIP w/ condition | C1 founder/BA pins the cohort denominator (needs a call); C2 tests |
| 018/REQ-004 | ✅ SHIP | — |
| 018/REQ-006 | ✅ SHIP | enumerate in-scope CTAs in design |

**Items needing a founder/owner call before their build:**
1. **017/REQ-001 conversion denominator** — pick the cohort (recommended: completed-of-requested-in-period;
   numerator ⊆ denominator; reject clamp-only). **Founder/BA decision required.**
2. **012/REQ-004** — confirm the email-link-fallback rewrite scope (the founder already decided the approach; the
   v1.3 spec must capture it with the security/dba conditions). **requirements-engineer rewrites; security-expert +
   dba re-gate.**
3. **011/REQ-004** — security-expert signs the residual-risk box only after the concrete timing guard is in
   design md (and bff pins 409 + the code name).

**Sign-off:** 14 of 16 Wave-2 REQs APPROVED (most with named, non-fatal conditions). **2 SEND-BACKs:**
012/REQ-004 (hard — spec is SUPERSEDED by the founder's email-link decision) and 011/REQ-004 (design-only — the
net-new timing guard must be made concrete + testable). 017/REQ-001 is approved but blocked-on-a-founder-call for
the cohort denominator. Boxes in each spec's ## Review & Sign-Off stay unchecked for 011 and 012 until their
conditions land; the Boss flips status to APPROVED per-slice only then.
