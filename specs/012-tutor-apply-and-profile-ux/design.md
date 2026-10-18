# Design — Tutor apply feedback + profile user experience (012)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Owned by bff-specialist + software-engineer + frontend-expert + uiux-designer. Grounded 2026-06-22.

## REQ-001 — single completeness source + non-silent gate

**Root cause:** frontend is Profile Complete (Tutor Apply Control implementation reference) and backend assert Profile Complete
(Job Application Service implementation reference) check DIFFERENT fields (frontend location Region + qualifications is at least 10; backend
location Planning Areas + qualifications non-empty). frontend can green-light an apply the backend rejects.

**DECISION (gate-binding — single source of truth, the aligned-constant fallback is REJECTED):** the GET
tutor-profile / profile-me response exposes a server-computed profile Complete: boolean and
missing Fields: string, both derived from the SAME predicate the apply-gate uses
(Job Application Service assert Profile Complete To Apply, 242-263). The 7 gate fields (verified in source) are:
Handle, commitment, hourly Rate Min Sgd, hourly Rate Max Sgd, location Planning Areas (non-empty),
qualifications (non-empty, **NO is at least 10 length floor — that floor is a frontend-only invention and must be deleted, not
ported to the backend**), and subject Levels (non-empty, INT-153). The completeness logic must be **extracted into one
shared method/component** so the apply-gate (422) and the profile Complete flag cannot diverge — one predicate,
two callers.

The frontend consumes profile Complete directly and **deletes its own is Profile Complete predicate**
(Tutor Apply Control implementation reference) — the predicate that checks location Region (backend checks location Planning Areas) and
the trim operation (backend has no floor) is the exact source of the drift bug. The banner lists
missing Fields from the server. Client state mirrors server truth; it never re-derives it (cross-cutting wave rule).

**Why not the aligned-constant fallback:** re-aligning the frontend constant to today's backend fields re-creates the same two
predicates that drifted in the first place; the next backend gate change rots it again. The gate rejects it unless
bff-specialist shows a concrete blocker to returning the signal on the profile payload (and records it here).

**403 vs 422:** confirm at build whether the reviewer's bare 403 Access denied came from the security layer
(role/auth) rather than the service 422. Ensure Tutor Apply Control on Error (113) has a default branch that
surfaces a clear message for 403 too (today the default toast is "Couldn't send your interest. Try again." — make
the 403/auth case explicit). No apply error path may be a silent no-op.

**Files:** backend Job Application Service implementation reference + the tutor-profile response data transfer object/controller (if adding the
profile Complete/missing Fields signal); frontend Tutor Apply Control implementation reference, Profile Completeness Bar/Apply Gate Banner.

## REQ-002 — sticky rail state

Assignment Detail implementation reference renders a sticky jump-link separate from Tutor Apply Control. Lift the apply/gate
state (applied? gated? job inactive?) so the sticky rail mirrors it — hide/relabel when applied ("View in My
Applications"), suppress the Apply CTA when gated/inactive. Simplest: have Tutor Apply Control own/report its state
upward, or compute the same existing/gate checks in the detail page.

## REQ-003 — confirm only

No code change. Test asserts the selector options derive from SUBJECT OPTIONS; live screenshot; close GH #209.1.

## REQ-004 — self-edit email/phone via the EMAIL-LINK confirmation flow (FOUNDER OVERRIDE → v1.3)

Founder chose the **email-link fallback** (supersedes the v1.2 "phone pending behind WhatsApp"): editing phone routes
a confirm link to the user's already-verified ACCOUNT EMAIL; editing email routes the link to the NEW email. Clicking
**Confirm** on a landing page (a POST) commits. This routes a phone change through the account email — treat as
**account-takeover-sensitive**; the full anti-takeover set below is MANDATORY, not optional.

### Data model (dba) — contact change request table + migration

A dedicated **contact change request** table (not pending-columns-on-user — the table cleanly holds both field
types + an audit trail). Liquibase XML changeset (no raw database structure definitions); pick the **next free number** in the single ledger,
honouring the reserved-gap convention (the highest changeset on disk today is **0065** 0065-jobinv-createdby-index implementation reference,
so the next free is **0066** 0066-contact-change-request implementation reference — verify against db/changelog/ at build time, no
number collision). Columns:

| column | type | notes |
|---|---|---|
| id | uuid PK | |
| user id | uuid FK → users | not null |
| field | varchar | EMAIL \| PHONE (VARCHAR, not a PG enum — avoids the ALTER TYPE crash-loop class) + require |
| old value | varchar | initiation-time contact snapshot under the plaintext-email-phone amendment; authorized use only |
| new value | varchar | pending contact under the same storage amendment; not the live value until confirmed |
| channel | varchar | EMAIL LINK (only channel; VARCHAR + require; leaves room for a future channel) |
| token hash | varchar | **hashed at rest** (mirror Password Reset Token otp hash — password Encoder encode); never plaintext |
| status | varchar | PENDING \| CONSUMED \| EXPIRED \| SUPERSEDED \| REVOKED (soft-delete — superseded/expired are status-flagged, never row-deleted) + require |
| expires at | timestamptz | not null; expiry period = 30 min from created at |
| created at | timestamptz | not null |
| consumed at | timestamptz | null until commit (single-use enforcement) |
| revoke token hash | varchar | independent purpose-bound revoke token; never accepts a confirmation token |
| revoked at | timestamptz | set only by successful PENDING → REVOKED; soft-retained audit history |

The user's CURRENT verified users email/users phone columns are UNCHANGED until commit. **Uniqueness is enforced
AT COMMIT** against the live users email/users phone unique indexes inside the swap transaction (re-check inside
a database transaction, NOT at request time) → closes the TOCTOU; a collision returns 409. Model the entity on the existing
Password Reset Token (Password Reset Token implementation reference — hashed token, expires at, used at/consumed at,
the is expired operation/the is used operation helpers).

### Flow (software-engineer)

1. **Initiate** (POST initiate-contact-change): require step-up (below); validate the new value's shape; supersede
   any existing PENDING request for the same field (status → SUPERSEDED); create the PENDING row with a
   secure-random token (hashed at rest, 30-min expires at, bound to the user, changed field, requested new value and confirmation channel); enqueue
   the confirm email via the durable **outbox** (Outbox Event Type EMAIL + Email Payload — recipient USER id, NOT
   the raw address; for a PHONE change the recipient is the account-email owner = the same user) AND enqueue the
   old-email + old-phone "a change was requested — revoke?" warning intents; return the pending state.
   Each purpose resolves an initiation-time destination from the request: NEW EMAIL for email
   confirmation, ACCOUNT EMAIL for phone confirmation, OLD EMAIL/OLD PHONE for warnings. Queue payloads use
   request/user ids plus recipient purpose, never raw contact values or tokens in logs. Do not resolve an old
   warning destination from a later-mutated user record. Persist delivery-attempt outcomes separately from queue acceptance.
2. **Verify/commit** (POST confirm, triggered by the landing page's explicit Confirm — see below): use the
   request id to load the row; require the authenticated owning account with cross-site request forgery protection protection and verify the
   supplied token against its stored hash; reject if not PENDING / expired / consumed (replay → no-op rejection); inside one database transaction:
   re-check uniqueness of new value against the live unique index → on collision throw **409**
   (PHONE ALREADY REGISTERED for phone, the email conflict for email — reuse 011/REQ-004); else write
   the requested account contact field to its confirmed new value, set consumed at, status → CONSUMED, write an audit row (old→new, actor,
   timestamp), and invalidate any other pending request for that field. The OLD value is used for all auth/contact
   ops until commit. A lazy-expire (or a swept recurring task) flips overdue PENDING rows to EXPIRED.

### Revoke contract (2026-09-05 correction)

- At initiation mint an independent random revoke token, bound to request id, owner, field and purpose REVOKE,
  with the same 30-minute expiry. Store only its hash. Confirmation and revoke credentials are not interchangeable.
- Warning links open /account/contact-change/revoke?request Id=…&token=[REDACTED]] GET only renders a confirmation page;
  no revoke, swap, or token consumption on link preview. Do not leak tokens to referrers, analytics or logs.
- submit operation for account contact-change revoke accepts either the authenticated owning user with cross-site request forgery protection protection
   or the dedicated valid revoke credential. Reject other users using the normal no-existence-leak response.
   The token authorizes cancellation only; it cannot reveal contact details or authorize another account action.
- Both forms identify the row explicitly: the authenticated profile action sends the contact-change request identifier; the warning-link
  action sends the contact-change request identifier and its separate revocation token (map the link's token query value to revoke Token). Successful revoke
  and a valid idempotent repeat return **204 with no response body**; no contact value is disclosed by this route.
- Lock the same request row for verify and revoke. PENDING → REVOKED sets revoked at, invalidates the verify
  token, records actor/authentication method/time, and leaves the current contact unchanged. A repeat valid
  revoke of the same REVOKED request returns success without another mutation. EXPIRED/SUPERSEDED cannot commit;
  CONSUMED returns 409 with recovery/support guidance, not an automatic rollback. Verify loses with no mutation
  if revoke won. If verify won, revoke cannot undo it.
- The authenticated profile can revoke its pending change without waiting for an email. Remove the pending
  banner after an authoritative terminal response, not optimistically before a successful transition.

### Old-contact warning delivery (required; phone transport decision pending)

The request transaction records both warning intents through the durable outbox. Dispatchers resolve the old
destinations from request snapshots, record attempts/outcomes, retry transient failure through the
existing delivery mechanism and surface exhausted/unavailable delivery to operators without raw personal information in logs.
Queued, attempted and delivered are separate facts. The OLD-phone transport/fallback while WhatsApp is OFF is
not decided by this spec; see [open decisions](../CURRENT-REQUIREMENTS.md). The approved
email-link confirmation flow remains in force. This correction neither invents a new blanket block on changes
nor waives the old-phone warning. Both-channel completion cannot be signed off until its delivery path and
failure behavior have an approved contract and evidence.

### Verify route — landing page + explicit Confirm (anti-prefetch, MANDATORY)

The email link targets a **frontend landing page** (/account/contact-change/confirm?request Id=…&token=[REDACTED]]) that REQUIRES an explicit
Confirm click; the commit is a **POST on confirm**, NOT a bare GET side-effect — email security scanners pre-fetch
links, and a bare-GET auto-commit would let a scanner (or a prefetcher) silently commit the change. The verify backend
endpoint MUST therefore be a POST (or otherwise require a body/cross-site request forgery protection token the landing page supplies), never a
GET that mutates. The link/landing body NAMES the exact phone/email being authorised (anti-confused-deputy).

### Security (security-expert) — anti-account-takeover set (ALL MANDATORY)

1. **Step-up to initiate** — current password (or fresh-/recent-session re-auth) is REQUIRED to start any contact
   change; a hijacked session cannot silently re-point the recovery contact. (Hard REQ, not a "should".)
2. **Token single-use + short expiry period** — secure-random, hashed at rest, 30-min expires at, single-use via
   consumed at.
3. **Notify the OLD email AND old phone at INITIATION** — both, with a "you didn't do this? revoke" affordance,
   BEFORE commit, so the legitimate owner can stop it.
4. **Confirm-click landing page** — no bare-GET auto-commit (anti-scanner-prefetch, above).
5. **Link body names the exact value being authorised** — anti-confused-deputy.
6. An unverified/expired token NEVER mutates users email/users phone.

security-expert signs off the residual takeover surface once all six are in the build.

### Contract (bff-specialist)

Three named routes in api Routes implementation reference (no magic strings): an initiate route (POST account contact-change interface operation,
step-up-guarded, returns the pending state) and a verify route the landing page POSTs to
(submit operation for account contact-change verify, authenticated owner + cross-site request forgery protection, body the contact-change request identifier and confirmation token), and the revoke route described above
(submit operation for account contact-change revoke, body and 204 success contract above). Return request Id, field,
masked new value and expires At in the initiate response, pending-change GET and profile pending Contact Change
so the frontend can render "verification pending" and revoke the selected request. Neither response exposes either token.

### user experience (frontend-expert / uiux-designer)

The edit affordance; the pending banner ("your current email/phone stays active until you confirm"); the
landing/confirm page (explicit Confirm button → POST); and — importantly — the **phone-via-email-link copy** ("we
sent a confirmation link to your account email to authorise your new phone number — click it to confirm"), since this
indirection is confusing if unexplained.

**Files:** backend — account/profile service + contact-change initiate/verify endpoints + data transfer objects + the entity + the
Liquibase migration; the outbox email enqueue; frontend — tutor Profile edit form + pending-state user interface + the
landing/confirm page + api Routes implementation reference. This is the heaviest REQ in the spec and is scoped as its own Wave-2 slice.

**Cross-wave note:** 018/REQ-001 (Wave-3) suppresses WhatsApp verification copy while WhatsApp is OFF. That
suppression MUST be channel-scoped to WhatsApp/one-time passcode only and MUST NOT suppress this REQ's email-link phone-verification
copy (the email-link flow is independent of INT-082).

## Test / coverage

- backend gate ≥97.5/97.5 — the shared completeness predicate + profile Complete/missing Fields on the profile payload
  tested over a field matrix (each of the 7 gate fields missing → profile Complete is no + that field in
  missing Fields); the 422 apply path stays tested (Job Application Service Test) and MUST be driven by the SAME
  shared predicate (one source).
- frontend gate ≥97.5/88 — **BLOCKING parity test (non-negotiable):** over a field matrix, the frontend gate renders iff
  profile Complete equals no — the frontend verdict cannot diverge from the backend signal (the frontend no longer has its own
  predicate to diverge). RTL: incomplete → banner listing missing Fields (not an enabled CTA); 422 → toast+banner;
  **403 → an explicit message, never an unhandled bare "Access denied"** (the on Error default branch must surface
  the auth/403 case, not the generic "Try again" toast). Plus sticky-rail states + taxonomy assertion.
- Live: Playwright incomplete-profile apply (banner + missing fields) + forced 422 (toast) + the reviewer's bare-403
  path identified and shown to surface a friendly message + sticky-rail-after-applied.
- **REQ-004 BLOCKING tests** (backend ≥97.5/97.5; frontend ≥97.5/88): (1) token **single-use** (second confirm rejected) +
  **expiry** (expired token rejected, old value intact); (2) **email-scanner prefetch does NOT auto-commit** — a
  bare GET on the verify URL performs no mutation; only the explicit POST-confirm commits; (3) the OLD value is used
  for all auth/contact ops while pending; (4) verify → **atomic swap + audit row**; (5) **uniqueness re-checked AT
  commit** — a value taken between initiate and verify → **409** (PHONE ALREADY REGISTERED / email conflict), old
  value intact; (6) **step-up required to initiate** — no-step-up request rejected, no pending row; (7) phone change
  commits via the EMAIL link (the founder fallback) — NOT held pending-only; (8) old-email + old-phone notification
  intents durably recorded at initiation and delivery outcomes distinguished; old-phone channel sign-off remains
  pending. Add (9) revoke-before-confirm, replay, wrong-purpose/owner/expired credentials, GET no-op and both
  confirm-versus-revoke lock orderings. Live capture: email and phone edit→pending→confirm→commit, plus revoke.

**Storage authority:** [plaintext-email-phone](../architecture/plaintext-email-phone.md) supersedes user email/phone
encryption/keyed message authentication assumptions. Contact snapshots follow that explicit amendment; confirm/revoke credentials remain
hashed at rest. This correction does not extend the contact-storage amendment to passwords or tokens.
