# Postman Collection

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 correction:** Contract amendments in004/010/012/014/017/021/027 take precedence over older request examples. Guest filters remain public, oversized anonymous sizes clamp to20, and revoke/provenance examples must follow the corrected contracts. Collection completeness was not reverified by this docs change. [Current requirements](../CURRENT-REQUIREMENTS.md). No runtime verification was performed by this documentation correction.

A Postman collection for the InTuition SG backend lives at the repo root under
[Private supporting material excluded (publication note)](../../docs/publication-notes.md) (kept outside intuition-spec/ because it is
JSON tooling, not documentation). It covers **every production endpoint across
modules M1–M18 + INT-082** (24 production controllers; the two the profile operation
regression-only controllers are intentionally excluded). For the prose endpoint
reference see [backend-api md](backend-api.md). **Coverage brought to the M18
baseline 2026-06-02 (INT-022 close-out):** added folders M14 (Landing Showcase),
M16 (Admin Management), M17 (Marketplace Detail), M18 (Assignment Lifecycle).

> **Synced to the 2026-06-03 feature wave (97 requests; was 82).** Added the **15** endpoints shipped that pass:
> **M6 — Interview (8)** INT-039 timeslot negotiation — tutee accept/counter, tutor get/accept/decline/counter/
> cancel, admin force-pending (M6 folder now 20 = all Interview Controller endpoints); **M18 (6)** INT-044
> invitations (parent invite/withdraw, tutor list/accept/decline) and INT-034 administrator
> assignment from a tutor-selection request, plus changing a tutor listing’s availability. The documented request fields match the corresponding operations.
> (the selected time-slot identifier, the proposed interview time slots, the reason for declining and any optional alternative time slots,
> the selected time-slot identifier if one has been agreed, the tutor’s identifier and invitation message, whether the tutor is available).
> New id variables used: interview Id, timeslot Id, invitation Id, selection Request Id, job Id, post Id,
> tutor User Id. INT-025 (POST SUBMITTED) added no endpoint. Every controller endpoint (incl. the 20 interview
> endpoints) is now represented; the two the profile operation regression-only controllers remain excluded.

> **INT-077 sync (2026-06-06) — marketplace quota caps.** Added **1** request to the M18 folder:
> read operation for tutee posts id invite-quota ("Get Invite Quota for my job (tutee)"). Updated the descriptions
> of Apply / List Applications / Withdraw Application (tutor) and Invite / Withdraw Invitation / Applicant cards
> (tutee) to document the new quota / application Quota / invite Quota (the allowed limit and remaining allowance) fields, the
> Owner Applicants Response envelope on the applicants list, and the 422 cap codes
> APPLICATION LIMIT REACHED / INVITE LIMIT REACHED. The collection holds no saved response examples,
> so the quota contract is documented inline in each request's description. **INT-078** (removed tutee
> account-level relationship) needed no collection change — the Register Tutee body never included it.

> **INT-082 sync (2026-06-06) — WhatsApp / phone verification (DEFAULT OFF).** Added a new folder
> **"INT-082 — WhatsApp / Phone Verification" (6 requests)** + a whatsapp Verify Token collection variable:
> submit operation for account phone verify start, submit operation for account phone verify confirm (the one-time passcode), replace operation for account phone
> (the new phone number), replace operation for account whatsapp-consent (the user’s choice to opt in or opt out) — all authed via the existing cookie/cross-site request forgery protection scripts — plus
> the public-but-signed Meta webhook read operation for webhooks whatsapp (verify-token handshake) and submit operation for webhooks whatsapp
> (signed delivery callback; placeholder X-Hub-Signature-256). Every request's description notes the **default-OFF**
> behaviour and the an indication of whether phone verification is enabled (disabled in this case) start response. This brings the production controller set to **24**
> (Account Controller + Whats App Webhook Controller added) and the collection total to **110 requests**.

> **INT-084/085 sync (2026-06-08).** No endpoint changes (collection stays 110 requests). The
> consent request's description updated: the delivered event set is **config-driven**
> (whatsapp notifications.events, default **INTERVIEW SCHEDULED only** — founder cost decision
> 2026-06-07), sent to **BOTH tutor and tutee** (INT-085); MATCH CREATED / contact-revealed are
> **dormant** (enqueued, dropped at dispatch).
>
> **⚠️ DEPLOYMENT GATE — expanding whatsapp notifications.events (Casey, team review 2026-06-08):**
> the consent string names the delivered events, so over-sending past it is a Personal Data Protection Act violation. To add
> an event: (1) founder approves the cost; (2) the frontend consent copy (Phone Settings Section implementation reference)
> is updated to name the new event; (3) the frontend test pinning the copy is updated; (4) config + copy
> ship **atomically** — never bump the config alone.

> **Spec 003 sync (2026-06-20) — password-reset error accuracy + one-time passcode debug logging.** No new endpoints
> (collection request count unchanged). The **M1 — Auth → Reset Password** request
> (submit operation for auth reset-password) is now the **first request in the collection to carry saved
> response examples**: the happy path **200** a success confirmation plus the five distinct error branches the frontend
> now surfaces separately — **400 INVALID one-time passcode** (wrong/expired one-time passcode, *byte-identical* to the unknown-account
> case for anti-enumeration), **400 VALIDATION ERROR** (malformed one-time passcode → "one-time passcode must be a 6-digit code"; weak
> new password → [REDACTED CREDENTIAL]), **429 one-time passcode ATTEMPTS EXCEEDED** (per-token attempt cap),
> and **429 RATE LIMIT EXCEEDED** (per-(ip,account) endpoint cap). The request description documents the
> anti-enumeration equivalence and the two dev-only ways to fetch the one-time passcode for testing: the the profile operation
> regression endpoint submit operation for regression password-reset issue-otp and the backend
> otp debug-log enabled flag (D8) that logs the one-time passcode to the application logs in dev. All example bodies/messages
> mirror Auth Service reset Password + the VALIDATION ERROR envelope from Global error handler.

> **Specs 001 / 002 / 004 sync (2026-06-21) — consent · availability slots + multi-MRT · session refresh.**
> **Collection edits to fold in** (the collection is request-only — no saved response examples — so document the new
> contract inline in each request's description):
> - **spec 001 — register consent (M1 — Auth → Register Tutee / Register Tutor):** add the three **required** consent
>   booleans to BOTH register bodies — terms Accepted, pdpa Consent, age Confirmed (each must be yes or the
>   request fails **400 VALIDATION ERROR** — @Assert yes) — plus the optional marketing Opt In (bool). The Tutor
>   body also keeps commission Acknowledged/commission Terms Version.
> - **spec 002 — tutor onboarding-capture (M1 — Auth → Register Tutor):** add the OPTIONAL onboarding fields to the
>   Register Tutor body — hourly Rate Min Sgd/hourly Rate Max Sgd (1–1000), location Planning Areas (string[]),
>   location Region (Location Region), commitment (Tutor Commitment), subject Levels
>   (pairs of subject and level codes). All optional; omit them to keep the existing happy path.
> - **spec 002 — availability slots + multi-MRT (M2 — Posting (Tutor) → Create / Update Tutor Post):** add
>   availability Slots (availability slots with a weekday numbered 0–6, a start hour numbered 0–23 and a duration of 60 or 120 minutes, the new canonical form) and
>   location Mrt Stations (string[], multi-select). The legacy availability Bitmask and single location Mrt Station
>   are still accepted (deprecated); note in the description that a new client sends the slot/array forms and the
>   server derives/keeps the legacy fields in sync. GET/own-post responses now echo availability Slots +
>   location Mrt Stations on single-post paths. The **Get Tutor Profile** response gains suggested Public Handle.
> - **spec 004 — session refresh (NEW request, M1 — Auth → Refresh Session):** submit operation for auth refresh,
>   **no body**, cross-site request forgery protection + the existing cookie scripts. Relies on the host-only path-scoped refresh token cookie set
>   by login/register — Postman's cookie jar persists it. On success re-mints the auth cookie (**now Max-Age is 1800
>   / 30 min**, updated from the prior 86400). Document the typed failures in the description:
>   **401 REFRESH INVALID/REFRESH EXPIRED/REFRESH REUSED**, **403 ACCOUNT BLOCKED**, **429 RATE LIMITED**.
>   ADMIN sessions get NO refresh cookie (admin refresh → 401 REFRESH INVALID). Logout / delete operation for auth account
>   now also revoke the refresh family server-side. _(In Tuition postman collection implementation reference is large binary-ish JSON;
>   apply these via the Postman app or a scripted patch, not a hand-edit, then re-export.)_

## Files

| File | Purpose |
|---|---|
| In Tuition postman collection implementation reference | All endpoints, foldered by module (M1–M18 + Ops) |
| In Tuition postman environment implementation reference | base Url + id variables (defaults to http://localhost) |

## Import

1. Postman → Import → drop both JSON files.
2. Select the **InTuition — Local** environment (top-right).
3. Make sure the backend is running — see [../ops/runbook md](../ops/runbook.md).

## Auth & cross-site request forgery protection

The application programming interface authenticates with a signed session token in an **HttpOnly cookie** (auth), set by the
login/register requests. Postman's cookie jar persists it automatically, so once
you run **M1 — Auth → Login**, every protected request is authenticated.

State-changing requests (POST/PUT/PATCH/DELETE) are **cross-site request forgery protection-protected** with a
double-submit cookie:

1. Run **M1 — Auth → Bootstrap cross-site request forgery protection** once. It sets the XSRF-TOKEN cookie.
2. A collection-level pre-request script copies that cookie into the
   X-XSRF-TOKEN header on every mutating request automatically — you don't set
   it by hand.

## Typical flow

1. Bootstrap cross-site request forgery protection → Register Tutor / Register Tutee → Login.
2. Create a post (M2); put its id into the post Id / tutor Post Id /
   tutee Post Id variables.
3. As an admin: approve (M3), create + activate a match (M4), pay commission (M5).
4. Exercise interviews (M6), profile (M7), dashboards (M8), notifications (M9),
   favourites (M10), reviews (M11), analytics (M12).

## Variables

base Url plus resource ids (post Id, tutor Post Id, tutee Post Id, match Id,
commission Id, interview Id, notification Id, reset Token, and — for the M16/M18
folders — admin Id, tutor User Id). Fill the ids in after creating the matching
resource, or add test scripts to capture them from responses.

## Maintenance

When you add or change an endpoint, update both this collection and
[backend-api md](backend-api.md). The collection's request bodies mirror the
backend request data transfer objects (the application component*) and enum values
(the application component*) as of the last sync.

## Specs 019–023 sync (2026-06-23)

Annotations + saved examples only — **no request added/removed/renumbered** (collection stays a
superset; 113 requests, unchanged). python3 -m json tool passes.

- **Taxonomy coherence 422 (Spec 022):** "Create Tutee Post", "Create Tutor Post", and "Update Tutor
  Profile" now document the coherence rule — subject and level must share the same level group, with
  the N LEVEL→O LEVEL widen (an N-Level subject is treated as O-Level). An incoherent pair (e g.
  A MATH (O-Level) + PRIMARY 5 (Primary)) → **422 UNKNOWN SUBJECT OR LEVEL**; a coherent pair →
  201/200. Canonical codes cited: HUMANITIES OLEVEL, PHYS H1 ALEVEL, S1 NT. Each got a coherent
  success example + an incoherent **422** example; the prior Create-Tutee 422 example body was switched
  to an actually-incoherent pair (was MATH/PRIMARY 5). "Update Tutor Profile" + "Create Tutor Post"
  bodies now include subject Levels / coherent subjects+levels to exercise the rule.
- **Admin interview list (ADM-05):** "List Interviews (admin)" gained a 200 saved example whose row
  carries the new **tutee Post Subject / tutee Post Level** fields on Admin Interview Response (null-safe;
  lets the admin "Job" column render for free-market interviews where tutor Post is null).
- **Invite + request-interview on PENDING (Spec 021):** "Request Interview (tutee)" and "Invite Tutor to
  Job (tutee)" now note the referenced tutee post may be **PENDING** (awaiting approval) as well as ACTIVE
  → both 2xx; contact details stay gated until post-assignment.
- **PENDING editable (Spec 021):** "Update Tutee Post" (replace operation for tutee posts id) notes the update works on
  a PENDING post (200, stays PENDING; an edit does not re-route or flip status).

Validate after editing: no value implementation reference.

## Specs 008/009/010 sync (2026-06-22)

**Spec 010 (tracing) — collection-wide additions:**
- The collection-level **description** now documents that every response carries a trace Id (top-level on JSON-object success bodies; inside error on error bodies) plus the X-Correlation-ID response header (present on **every** response, cross-origin request permissions-exposed,  equals  body trace Id  equals  log-line id). Known gap: 401/403 entry-point responses are header-only.
- Added a collection-level **test script** (event listen is test) that asserts X-Correlation-ID is present on every response and that JSON-object bodies carry trace Id (skips arrays / 204 / non-JSON, which are header-only by design).

**Spec 009 (taxonomy + validation) — "Create Tutee Post" (M2 Posting Tutee):**
- Request **description** notes num Students is bounded **1–20** (the max operation; over → 400 VALIDATION ERROR) and that an unknown subject Code/level Code now returns **422 UNKNOWN SUBJECT OR LEVEL** (was a 500). Taxonomy expanded to 34 subjects / 10 levels (incl. distinct N LEVEL) — see migration 0064 seed; not hand-duplicated here.
- Added two **saved example responses**: a 201 Created showing the top-level trace Id, and a 422 UNKNOWN SUBJECT OR LEVEL showing the new an error containing its code, message and request trace identifier error shape. (This is the first request in the collection to carry saved response examples.)

**Spec 008** — frontend-only wizard bug fixes, **no application programming interface/data transfer object/path change** → no collection edits.

Validate after editing: no value implementation reference.

## INT-062 sync (2026-06-04)

**Two NEW endpoints to add to the collection** (both POST, no body, cross-site request forgery protection + auth like other mutations):
- **M2 — Posting (Tutee):** closing the specified post as the parent ("Close Tutee Post") —
  withdraw a LIVE job; cascade-closes its applications + invitations to CLOSED. 409 on non-live state; 403 non-owner.
- **M3 — Admin Post Approval:** closing the specified post as an administrator ("Close Job (Admin)") — same
  for any admin.
Responses: Application Response status / invitation status now include **CLOSED** (job withdrawn) alongside
SUPERSEDED/EXPIRED (match-filled). No other request/path changes. See backend-api md.

## INT-065 sync (2026-06-04)

Added an **"Admin Moderation (INT-065)"** folder (4 requests) + a moderation User Id collection variable: read operation for admin users (role/status/q/page/size params), submit operation for admin users id block (Moderation Action Request), submit operation for admin users id unblock (no body), submit operation for admin posts id delete (Moderation Action Request) — all ADMIN-only, cross-site request forgery protection+auth via the existing collection scripts. Blocked-user login returns 403 ACCOUNT BLOCKED. See backend-api md.

## INT-053–059 wave sync (2026-06-04)

**No collection edits needed — all response-only or frontend-only.** The collection is request-only (stores no response
examples), and this wave added **no endpoints and no request-body/param changes**:

- **read operation for admin matches + read operation for admin interviews (INT-053):** the **response** data transfer objects gained fields
  (Admin Match Item Response +tutor User Id/tutor Display Name/tutor Public Handle/tutor Post Id/tutee User Id/
  tutee Post Id/tutee Name/job Title; Admin Interview Response +tutee Post Id/tutor User Id/tutor Display Name/
  tutor Public Handle). Request path/params/body unchanged → the existing "List Matches" / interview requests are
  current; re-run to see the richer payload. See backend-api md.
- **INT-054/055/056/057/058/059:** frontend-only (auth-card width, qualifications picker, refresh-session fix, tutee
  Find-Tutors nav, applicants link, reviews→Profile) — **no application programming interface surface change**, nothing to update here.

## INT-015…042 batch sync (2026-06-03)

One **contract change** to fold into the M6 folder; everything else is behavioural (no body/path edits):

- **submit operation for admin interviews id schedule (INT-035) — CHANGED:** the body gains an OPTIONAL
  meeting Link (≤500). New shape: the scheduled date and time and the optional meeting link.
  meeting Link may be omitted (link can still be set later via PATCH …/meeting-link). Update the M6
  "Schedule interview" request to include the optional field.
- **read operation for admin posts tutee Post Id applications (INT-033):** endpoint unchanged — the fix was an
  frontend route. Already in the M15/admin folder; no edit needed.
- **submit operation for admin matches id cancel (INT-016):** unchanged contract; now returns **200** (was a
  500 bug) and additionally reopens the job + notifies the former tutor. Re-run to confirm green.
- **delete operation for tutor posts id (INT-015):** unchanged contract; deleting a **REJECTED** post now
  returns 200/204 (was 500). Re-run against a rejected post.
- **GET tutee interviews interface operation & tutor interviews interface operation (INT-036):** unchanged — paginated
  Page Response (items under data); the frontend bug was reading it as an array. Collection params stay.

## INT-001…019 batch sync (2026-06-02)

The INT bug-fix batch made **no endpoint-contract changes**, so the collection's request set is
current. Behavioural notes (no body/path edits needed):

- **submit operation for auth forgot-password + submit operation for auth reset-password (INT-006):** unchanged
  request bodies. The forgot-password email now contains a **reset deep-link** to
  the password-reset page with the account email supplied as a link parameter (the frontend prefills the email; the 6-digit one-time passcode is typed,
  never in the link). Reset body remains the account email, one-time passcode and new password.
- **read operation for tutors (INT-012):** unchanged params. Anonymous callers are now capped
  (page ≤ 4, size ≤ 20 → empty page beyond that); authenticated callers page freely. The
  collection's default ?page is 0&size is 20 stays within the anonymous cap.
- **Session (INT-013):** the auth cookie now has Max-Age is 1800 (30 min); Postman's cookie jar
  still persists it. After 30 min, re-run **Login**.
- Seed credentials for local manual testing: all Dev Data Seeder sample users share password
  REDACTED password (e g. tutor REDACTED email, tutee REDACTED email) — see INT-018.
