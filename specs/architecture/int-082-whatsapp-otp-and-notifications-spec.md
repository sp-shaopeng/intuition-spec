# INT-082 — WhatsApp Phone Verification (one-time passcode) + WhatsApp Notifications (feature-toggled, default OFF)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Current reminder/phone-change contracts are linked in the dated section; the old narrowed-event delivery snapshot is historical, and OFF remains the default.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Status:** BUILT & shipped OFF to dev (2026-06-06). Spec v2 (11 spec conditions C1–C11 addressed) → built backend+frontend →
code team-review → Boss **SHIP IT (OFF)** with pre-enablement gates **PE-0…PE-4 in §9.1**. Default OFF; inert until those gates clear.
**Area:** backend + frontend + database · auth / notifications / external messaging
**Severity:** S2
**Created:** 2026-06-06
**Owners:** software-engineer (lead) + security-expert + dba + bff-specialist + frontend-expert + uiux-designer

---

## 1. Context & goal

Add **phone-number verification via WhatsApp one-time passcode** and **WhatsApp notifications**, shippable but **toggled OFF for
now** behind a feature switch. Founder decisions (2026-06-06):

| Decision | Choice |
|---|---|
| Provider | **Meta WhatsApp Cloud application programming interface** |
| Verification gating | Required to **reveal contact + parent commitments** — see §6 (apply stays OPEN, founder-confirmed) |
| Notification scope | ~~Curated events (3) + per-user opt-in~~ → **INTERVIEW SCHEDULED only, to BOTH tutor and tutee** (narrowed 2026-06-07, founder: per-message cost; INT-084/INT-085 — see §7) |
| Default state | **OFF** in prod; per-environment toggle |
| Phone change | **Re-verification required** — via a new phone-edit endpoint (§5.3) |
| Password reset | **Email-only, always** (existing forgot password Resend template) — never routed via WhatsApp, unaffected by any WhatsApp toggle (founder, 2026-06-07) |

> **Founder amendment 2026-06-07 (BUILT, deployed dev — INT-084 + INT-085):** WhatsApp messaging is
> pricey, so the delivered notification set is narrowed to **INTERVIEW SCHEDULED only** (plus the
> one-time passcode, which is verification, not a notification), now **config-driven** via
> whatsapp notifications.events (default INTERVIEW SCHEDULED) and filtered at outbox dispatch —
> re-expansion is config-only **and requires founder cost approval + a lockstep frontend consent-copy
> update**. Additionally, INTERVIEW SCHEDULED now notifies **both tutor and tutee** (in-app + WhatsApp
> outbox; was tutee-only — the tutor-no-show trust gap).

### 1.1 The central invariant — neither OFF **nor a misconfigured ON** may block users (C1, C2)
- Single canonical predicate, defined ONCE and implemented as one injected component
  Whats App Toggle (no @Value boolean re-read in three services):


Implementation example omitted. This public edition retains the written requirements in this section.


- **OFF (master or sub false):** the assert phone verified operation is a **no-op**; one-time passcode endpoints return a successful response confirming that the feature is disabled;
  nothing is sent.
- **ON-but-unconfigured (the "brick-on-ON" evil twin the review caught):** if whatsapp enabled is yes but the Cloud
  client the is disabled operation (missing/expired access-token/phone-number-id), the gate **must still no-op** — an one-time passcode
  cannot be sent, so blocking apply/reveal would brick the marketplace. Belt-and-braces: the app **fails its readiness
  probe** when enabled is yes and secrets absent, so it never serves the brick state in the first place.
- The gate bites **only** when master ON **and** sub ON **and** the client is healthy. §8 mandates the 4-quadrant
  truth-table test.

---

## 2. Reuse map (mirror existing patterns — verified anchors)

| New component | Mirrors (file:line) |
|---|---|
| Phone Verification Token entity + repo | Password Reset Token implementation reference ;  Password Reset Token Repository implementation reference |
| one-time passcode generate / hash / verify | the generate numeric otp operation (); bcrypt password Encoder encode () / .matches (); attempt Count max 5 (). **Expiry: this feature uses 10 min deliberately — note the password-reset precedent is one-time passcode EXPIRY MINUTES is 15 at Auth Service** (C11) |
| Whats App Queue Service (enqueue) | Email Queue Service implementation reference |
| Whats App Consumer Service (consume+retry) | Email Consumer Service implementation reference |
| Whats App Cloud Client (external web request protocol, the is disabled operation/the is healthy operation) | Resend Email Client implementation reference |
| Feature toggle component | email delivery.enabled (Email Queue Service implementation reference); auth mfa.enabled (Auth Service implementation reference) |
| Action gate (422) | Job Application Service profile-completeness assert (), unprocessable Entity |
| Curated fan-out | infrastructure/outbox/Outbox Service + Outbox Poller (migration 0042); Notification Service emit () |
| Rate limiting | Rate Limit Service implementation reference (Lua sliding-window); ratelimit forgot-password.*=3/3600 |
| Consent audit event | Audit Log usage pattern (e g. Contact Reveal Service CONTACT REVEALED) |
| Secret/env injection | the email provider key supplied privately through configuration (application implementation reference) |

---

## 3. Feature toggles & config (application implementation reference)


Implementation example omitted. This public edition retains the written requirements in this section.


The Whats App Toggle component computes the gate active operation / the otp active operation / the notifications active operation (master AND sub AND
client health). Startup validation: enabled is yes and access-token|phone-number-id blank ⇒ readiness FAIL (C2).
Secrets live in docker-compose implementation reference + .env implementation reference (never committed).

---

## 4. Data model — Liquibase changeset 0047 (additive only)


Implementation example omitted. This public edition retains the written requirements in this section.


**E.164 vs the existing partial unique idx users phone (C8):** the legacy phone is plaintext, written raw at
registration with **no normalization**, so existing rows are heterogeneous. Intended: normalize new writes to E.164
before the uniqueness check. **AS-BUILT (2026-06-06):** the new replace operation for account phone choke-point normalizes, but
**registration (Auth Service) still stores phone RAW** (pre-dates INT-082; not regressed). ⇒ **Pre-enablement gate
PE-1:** normalize phone in registration to match, OR formally accept the residual format-duplicate risk + a manual
normalize-and-report task — code and this spec must agree before the toggle is flipped ON. **Do NOT inline a backfill
in 0047** (a backfill could collide on duplicate humans and fail the migration). Legacy heterogeneity is documented and
left as a separate, later, manual normalize-and-report task. whatsapp messages (delivery audit) is **deferred** —
reuse the outbox + metrics first.

---

## 5. one-time passcode verification flow (effective only when the otp active operation)

one-time passcode operates on the **signed session token user's own stored phone** (no phone in the start/confirm request body) (C5).

1. **submit operation for account phone verify start** (authed). Rate-limit on **the whats app otp key operation** — server-derived
   user Id, not client phone (C5) — plus the per-user daily cap and the global send ceiling. Mint a 6-digit one-time passcode, bcrypt-store
   one **live** token (the uq pvt one live index + prior-invalidate guarantees exactly one), enqueue the Meta
   **authentication template** via Whats App Queue Service. Uniform response (no enumeration). **OFF / brick-on-ON ⇒
   a successful response confirming that the feature is disabled, nothing sent.**
2. **submit operation for account phone verify confirm** (authed) the one-time passcode. Load the live token (used at IS no value,
   attempt count<5, not expired); password Encoder matches. Success ⇒ phone verified is yes, phone verified at is now,
   single-use the token. Mismatch ⇒ increment attempt count, 422. Expired/exhausted ⇒ 422 + re-request hint.
3. **Phone change + re-verify (C3, founder requirement) — NEW endpoint** replace operation for account phone (authed)
   the phone number. This is the **single atomic choke-point** (the only users phone write path besides registration; note
   Profile Service does **not** write phone today — the prior spec's "profile-service update path" citation was wrong).
   In ONE transaction: normalize to E.164, uniqueness-check, update users phone, **reset phone verified is no,
   clear phone verified at, invalidate any live phone verification tokens**. A no-op write (same normalized number)
   does not reset. When the gate is active, a changed-but-unverified number **re-blocks** the gated actions until
   re-verification; when OFF it only tracks state.

---

## 6. Action gate (effective only when the gate active operation) — founder-confirmed scope

- Pure the assert phone verified operation → 422 ERR PHONE NOT VERIFIED. **No-op when the gate active operation** (master/sub OFF
  OR client unhealthy) — the central invariant (§1.1).
- **Gated-action set (configurable; founder chose "reveal + parent commits", C4):**
  - **MUST gate — contact reveal** (Contact Reveal Service) — the actual personal information-exchange moment.
  - **SHOULD gate — tutee select-tutor + invite** — parent-initiated commitments.
  - **OPEN (not gated) — tutor apply** — top-of-funnel for the scarce supply side; gating it bleeds applications →
    fewer matches (team + both user personas). Trust is delivered at reveal regardless.
- The set is config-driven so apply-gating can be switched on later without code change if the founder revisits.
- frontend 422 disambiguation: apply/reveal can now return distinct 422 codes (ERR TUTOR PROFILE INCOMPLETE vs
  ERR PHONE NOT VERIFIED); the frontend switches on error code and routes to the right remediation (C-Riley).

---

## 7. WhatsApp notifications (effective only when the notifications active operation)

> **Narrowed 2026-06-07 (founder cost decision, INT-084; both-parties INT-085). AS-BUILT, deployed dev.**

- **Delivered set = INTERVIEW SCHEDULED only**, sent to **BOTH tutor and tutee** (each recipient
  independently gated on their own whatsapp opt in + phone verified at dispatch), alongside the
  in-app Notification Service emit to both parties, via the transactional outbox.
  **Per-recipient independence (Jamie):** one party can receive the WhatsApp while the other doesn't
  (opted-out / unverified) — both ALWAYS get the in-app notification; the WhatsApp gating never
  affects the in-app emit.
- **Config-driven:** whatsapp notifications.events is INTERVIEW SCHEDULED (application implementation reference),
  parsed by Whats App Toggle into a typed Enum Set (Whats App Notification Event enum maps event ↔ Meta
  template); Outbox Poller dispatch Whats App drops events outside the set at the DISPATCH layer —
  the MATCH CREATED / CONTACT REVEALED **enqueue sites are kept** (dormant Whats App Template Keys),
  so **re-expansion is config-only — but REQUIRES founder cost approval AND a lockstep frontend
  consent-copy update** (the consent string names the delivered events; over-sending past the
  consented set is the Personal Data Protection Act risk).
  **Before re-expansion (Dana + Casey — the ceremony, atomic):** (1) founder approves the cost
  (decision logged: dated line in this spec's §1 table + the issue tracker); (2) frontend consent copy
  (Phone Settings Section implementation reference) updated to name the new event; (3) the frontend copy-pinning test
  updated; (4) config + copy ship **together** — never bump whatsapp notifications.events alone.
- **Observability of drops (team-review condition):** counter the number of dropped WhatsApp notifications, grouped by reason and event
  at every silent return (notifications disabled / event not enabled / recipient ineligible) +
  DEBUG log (outbox id + template, no personal information); the parsed event set is logged at boot; an invalid config
  token **fails boot loudly** naming the token and key.
- Uses pre-approved Meta templates (free-form text disallowed outside the 24h session window).
- frontend: per-user **opt-in toggle** in profile settings; consent string names the **single delivered
  event** in plain language, role-neutral ("when your interview is scheduled") + a one-tap opt-out;
  the test pins the copy positively AND asserts the dropped events' phrases are absent. Consent +
  opt-out are **audit-logged** (event, not just the column flip) so consent-at-time-of-send is
  reconstructable (C9).

### 7.1 Meta Cloud application programming interface + webhook (C6)
- the send operation → the provider’s message-sending operation for the configured sending account and API version,
  the provider access credential supplied privately in the request header, template payload. one-time passcode = transactional; notifications = opt-in.
- **Webhook** webhooks whatsapp interface operation: GET verify-token handshake (echo hub challenge) + POST delivery
  callbacks. **X-Hub-Signature-256 keyed message authentication verified with Message Digest is Equal (constant-time); the hub verify token
  compared constant-time too.** Reject unsigned/invalid. Handler is **idempotent** (Meta retries). Public-but-signed
  endpoint, allowlisted in security config. E.164 normalize before send + before the uniqueness check.

---

## 8. Security / Personal Data Protection Act + test strategy

**Security (C5, C6, C7):**
- Rate-limit one-time passcode on server-derived user Id, ip + per-user daily cap + a **global send-ceiling sliding rate-limit**
  (429 when breached — bounds spend/abuse per window). **AS-BUILT:** this is rate-limit-only; it does NOT persistently
  auto-flip the toggle. ⇒ **Pre-enablement gate PE-2:** build a real auto-trip (persist a tripped flag / flip
  whatsapp otp.enabled is no on a sustained-ceiling breach, with the kill-switch runbook hand-off), OR accept
  rate-limit-only as sufficient — before the toggle is flipped ON.
- attempt count<5 brute-force guard; one live token per user (uq pvt one live); uniform/enumeration-safe responses.
- **Never log the one-time passcode / template params** (the email path logs params at Email Queue Service implementation reference — must NOT be
  copied for the one-time passcode template); mask phone to last 4 everywhere.
- Webhook signature + verify-token constant-time; reject unsigned. Secrets env-only; account-deletion scrubs phone +
  revokes consent (Personal Data Protection Act retention).
**Personal Data Protection Act:** one-time passcode transactional (no marketing consent); notifications strictly opt-in (consent + timestamp + easy opt-out +
audit event).

**Test strategy (gates: backend ≥97.5/97.5, frontend ≥97.5 line/≥88 branch):**
- **Toggle truth-table (load-bearing, C2):** master × sub × secrets — (OFF,*,*)→no-op; (ON,OFF,*)→no-op;
  (ON,ON,no-secrets)→no-op **and** readiness FAIL; (ON,ON,secrets,healthy)→gate bites.
- **Gate scope:** reveal/select/invite gate when active; **apply NEVER gated**; verified ⇒ passes; distinct 422 codes.
- **Phone-change re-verify (C3):** verified → replace operation for account phone → phone verified is no + tokens invalidated →
  gated action 422 (gate ON) → re-verify → passes; same-number no-op does not reset.
- one-time passcode happy/expired/exhausted/wrong-code; rate-limit on user Id (not phone); daily-cap; **stubbed Whats App Cloud Client**
  (no live Meta calls); webhook signature accept / missing / malformed / **replayed** (idempotent).
- Enumerate the easy-miss branches now (client-disabled no-op, webhook-signature-fail, OFF no-op) — no JaCoCo
  exclusions to close gaps (CLAUDE md).

**Observability/SRE (C10):** metrics one-time passcodes sent through WhatsApp, grouped by result, WhatsApp passcode verification attempts, grouped by result,
failed WhatsApp sends, grouped by reason, valid and invalid WhatsApp webhook signatures, actions blocked by the WhatsApp verification requirement,
whatsapp spend; alert on signature-invalid spikes (attack) + send-failure spikes (Meta outage). Measurement events:
phone verify started/succeeded/failed, blocked actions grouped by action type, passcode sends grouped by result.
**Kill-switch runbook:** symptom (422 storm / otp send failures) ⇒ flip whatsapp enabled is no to restore the
marketplace ⇒ rotate token. Meta access token expiry (~60d) ⇒ rotation reminder + pre-expiry alert.

---

## 9. Meta dependency is de-coupled from the build (C10)
Code ships **behind the OFF toggle independent of Meta approval**. Meta Business verification + template pre-approval is
a **parallel ops track** (owner: founder/ops; started before enablement; lead-time days–weeks, templates can be
rejected). **Enablement — not the build — is gated on Meta approval.** This is the founder's "toggle off for now"
requirement honored at the dependency level.

### 9.1 Pre-enablement checklist (BUILT & shipped OFF 2026-06-06; these gate flipping the toggle ON)
The backend+frontend are built, gate-green, and deployed OFF (team-review → Boss: SHIP IT OFF). Before whatsapp enabled is yes
in ANY environment, ALL must be resolved (Boss conditions):
- **PE-0 (ops):** Meta Business verification + one-time passcode/notification template pre-approval (owner: founder/ops).
- **PE-1 (software-engineer + dba):** registration phone normalization reconciled with §4 (normalize in Auth Service, or formally accept the residual format-duplicate risk + manual normalize-and-report task).
- **PE-2 (software-engineer + security-expert):** the global send-ceiling auto-trip built (§8) or rate-limit-only formally accepted.
- **PE-3 (devops + observability): ⛔ STILL OPEN — blocks enablement.** §8 alerts wired (signature-invalid spike; send-failure spike; token pre-expiry) **+ (Boss condition 2026-06-08) an alert/dashboard on whatsapp notification.dropped and the whatsapp spend metric, and the kill-switch runbook entry — a counter nobody watches is the same silence in a slower form.** **whatsapp spend definition (Blake): NOT YET EMITTED in code** — counting basis when built: one increment per accepted Meta send, tagged the message template (per-recipient, post-Whats App Cloud Client send success); an SGD gauge derived via a per-template cost map is optional. Emitting it is part of PE-3, not a shipped claim. Kill-switch runbook entry: see ops/runbook md §"WhatsApp kill-switch".
- **PE-4:** ✅ **DONE 2026-06-08** — backend continuous integration green on the recorded revision (surefire 1714 + Testcontainers failsafe band incl. the toggle truth-table; lab runs in the post-deploy lane via the ci-no-lab profile). INT-064 backend-lane evidence captured.
- **PE-5 (added 2026-06-07, DONE same session):** code aligned to the narrowed event set — config-driven whatsapp notifications.events, dispatch filter, frontend consent copy names one event, both-parties INTERVIEW SCHEDULED (INT-084/INT-085, team-reviewed + Boss SHIP IT, deployed dev).

## 10. Out of scope
Inbound/two-way WhatsApp chat; marketing broadcasts; persisted delivery-receipt table (deferred); non-WhatsApp SMS
fallback; backfill-normalizing legacy phone rows (separate later task); **MATCH CREATED / CONTACT REVEALED WhatsApp
sends (dormant per INT-084 founder cost decision 2026-06-07 — re-expansion is config-only via
whatsapp notifications.events but requires founder cost approval + frontend consent-copy lockstep)**.

## 11. Build sequencing (later, governed)
database 0047 → backend (Whats App Toggle, entity/repo, Whats App Cloud Client/queue/consumer, one-time passcode start/confirm + PUT
/account/phone, gate on reveal/select/invite, outbox WhatsApp dispatch, signed webhook, rate-limit key + spend
breaker, readiness validation) → frontend (JIT inline verify flow on the gated-action 422 — auto-retry the action after
verify; opt-in toggle + consent copy; phone-change re-verify prompt) → gates → /team-review → Boss → deploy (virtual machine+local)
→ docs/Postman → push. **Ship OFF; prove the truth-table test green.**

## 2026-09-05 current contract and verification

Specs [006](../006-whatsapp-interview-reminders/requirements.md) and [007](../007-generic-scheduler-db-scheduler/requirements.md) authorize and specify scheduling confirmations plus 24-hour reminder intents, with short-notice confirmation only. The required cost approval, delivered-event config, frontend consent copy and pinned tests remain lockstep. Before enabling delivery, copy must cover INTERVIEW SCHEDULED and INTERVIEW REMINDER and the007 durable generation/dedup/reconciliation safeguards and failure alerts must be verified. Global WhatsApp stays OFF; this revision supplies no send or runtime evidence.

Spec [012](../012-tutor-apply-and-profile-ux/spec.md) supersedes the immediate PUT/account/phone recipe: stage a pending contact change with step-up authentication and explicit emailed-link confirmation; retain the live phone until confirmation and clear phone Verified only when the changed phone is applied. Legacy phone mutation payloads below are historical. This correction adds no automatic re-consent policy; it carries the already-required consent-copy/event/test parity and founder cost approval before enablement.
