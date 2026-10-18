> **Current-file check, 18 October 2026:** The contact-change requirements remain approved, but their delivery, destination and revocation paths are incomplete in the inspected application (G03). See the [alignment review](../../docs/current-file-alignment.md). The requirements remain unchanged.

# Requirements — Tutor apply feedback + profile user experience (012)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> EARS + Gherkin per atomic requirement, grounded against the real code 2026-06-22.
> Source: CONSOLIDATED md (T-07, T-06, #209.1, #209.3).

## Scope note (verified)

- **backend gate** Job Application Service assert Profile Complete (235-263): complete ⇔ Handle + commitment +
  hourly Rate Min Sgd + hourly Rate Max Sgd + non-empty location Planning Areas + non-empty qualifications +
  non-empty subject Levels. Failure → 422 TUTOR PROFILE INCOMPLETE, "Complete your tutor profile before applying
  to jobs." (NOT a bare 403 from the service).
- **frontend gate** is Profile Complete (Tutor Apply Control implementation reference): complete ⇔ Handle + commitment +
  rate min/max + location Region + qualifications length is at least 10 + subject Levels. **Mismatch:** frontend uses
  location Region; backend uses location Planning Areas. frontend adds a is at least 10 length floor backend lacks. → frontend can show an enabled
  Apply for a profile the backend rejects.
- The reviewer saw 403 Access denied with zero feedback → indicates either a security-layer role 403 fired
  before the service (e g. an unauthenticated/expired session, or a route authz rule), or the 422 was not surfaced
  in that flow. The build must confirm WHICH and guarantee a friendly outcome.
- **#209.1 ALREADY FIXED** — Create Listing implementation reference + Subject Level Matrix Editor implementation reference import the shared
  SUBJECT OPTIONS (taxonomy implementation reference, spec 009). REQ-003 = confirm-live + close GitHub.

---

## REQ-001 — Apply gate failure ALWAYS surfaces friendly, actionable feedback; frontend↔backend completeness criteria reconciled

- **Type:** functional · **Priority:** Must · **Severity:** S2 (core money action) · **Source:** T-07
- **Rationale:** Applying is the tutor's primary value action. A silent failure on it loses supply. The fix is two
  parts: (1) one source of truth for "profile complete" so the frontend never shows an enabled Apply the backend will reject;
  (2) every rejection (422 or any auth/role 403) renders a clear "complete your profile" or re-auth message with a
  link, never a bare "Access denied" with no user interface change.
- **Behaviour (EARS):**
  - The system SHALL use a SINGLE definition of tutor-profile completeness shared by the frontend pre-gate and the backend
    apply gate (same fields: handle, commitment, rate min+max, the location field the backend actually enforces —
    location Planning Areas — qualifications, ≥1 subject Level). The frontend is Profile Complete SHALL be reconciled to
    the backend definition (or the backend exposes a profile Complete/missing Fields signal the frontend consumes) so the two
    cannot disagree.
  - WHEN a tutor with an incomplete profile attempts to apply, the system SHALL pre-gate (show the
    Apply Gate Banner with a "Complete profile →" link) INSTEAD of an enabled Apply CTA, AND IF the request still
    reaches the backend, the 422 TUTOR PROFILE INCOMPLETE SHALL surface as a friendly toast + the banner (current
    113-122 behaviour, kept) — never a silent no-op.
  - IF the apply call returns a 403 (role/auth), the system SHALL surface a clear message (re-authenticate, or
    "tutors only") rather than a bare unhandled "Access denied" with no user interface feedback. No apply failure path SHALL be
    silent.
- **Owner:** bff-specialist (single completeness contract / missing Fields signal) · software-engineer (403 vs 422
  path) · frontend-expert (banner/CTA + 403 handling)
- **Acceptance criteria (Gherkin):**


- Scenario: Incomplete profile is pre-gated, not silently failed.
- Given a tutor whose profile is missing a backend-required field (e g. no planning area).
- When they open a job detail.
- Then the Apply CTA is replaced by a "complete your profile" banner with a link.
- And no enabled Apply button is shown that the backend would reject.
- Scenario: A 422 gate rejection is surfaced.
- Given a stale frontend profile snapshot let an Apply through.
- When the backend returns 422 TUTOR PROFILE INCOMPLETE.
- Then a friendly toast + the banner appear (with a profile link).
- And the failure is never a silent no-op.
- Scenario: A 403 on apply is never silent (guard).
- Given the apply call returns 403.
- When the error is handled.
- Then a clear message is shown (re-auth / tutors-only), never an unhandled "Access denied".
- Scenario: frontend and backend agree on completeness (no false-enabled CTA).
- Given any tutor profile state.
- Then the frontend pre-gate verdict matches the backend apply-gate verdict for the same profile.


- **Required evidence:** unit test of reconciled completeness (frontend verdict  equals  backend verdict over a field-matrix); RTL
  test: incomplete → banner; 422 → toast+banner; 403 → message; live Playwright: incomplete-profile apply shows the
  banner + a forced 422 shows the toast. Under REQ-001 implementation reference.
- **Dependencies:** none (backend gate + frontend control already exist; this reconciles + hardens).

## REQ-002 — Sticky-Apply rail reflects the applied / gated state (T-06)

- **Type:** functional · **Priority:** Should · **Severity:** S4 · **Source:** T-06
- **Rationale:** The bottom sticky "Apply →" jump-link (Assignment Detail implementation reference) is a separate element from
  the stateful Tutor Apply Control; it persists "Apply →" even after the control shows "✓ Interest sent", offering
  an action that no longer applies.
- **Behaviour (EARS):** WHEN the tutor has already expressed interest on a job (the apply control is in its
  "✓ Interest sent" state), the sticky bottom rail SHALL NOT present an active "Apply →" CTA — it SHALL hide,
  disable, or relabel to match the control's state (e g. "View in My Applications"). WHEN the profile is gated, the
  sticky rail SHALL NOT offer an Apply CTA that bypasses the gate.
- **Owner:** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Sticky rail updates after applying.
- Given a tutor who has already sent interest on this job.
- When the detail page renders.
- Then the sticky bottom rail does NOT show an active "Apply →".
- And it matches the apply control's "✓ Interest sent" state.
- Scenario: Sticky rail respects the gate.
- Given an incomplete-profile tutor on a job detail.
- Then the sticky rail does not present an Apply CTA that bypasses the completeness gate.


- **Required evidence:** RTL test of the sticky rail across states (open / applied / gated); live capture. Under
  REQ-002 implementation reference.
- **Dependencies:** REQ-001 (shares the gate/applied state).

## REQ-003 — Tutor subject taxonomy uses the shared options (CONFIRM + CLOSE #209.1)

- **Type:** functional / regression · **Priority:** Should · **Severity:** S3 · **Source:** #209.1
- **Rationale:** ALREADY FIXED — tutor listing + profile editors consume the shared SUBJECT OPTIONS (spec 009).
  This requirement confirms it live and closes #209.1 on GitHub; no code change expected.
- **Behaviour (EARS):** The tutor subject selector (Create Listing + Profile matrix editor) SHALL offer the full
  shared taxonomy (SUBJECT OPTIONS, identical to the tutee create-post taxonomy), with no divergent hardcoded list.
- **Owner:** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Tutor subject options match the shared taxonomy.
- Given the tutor Create Listing / Profile subject selector.
- Then its options equal SUBJECT OPTIONS from taxonomy implementation reference.
- And there is no separate hardcoded tutor subject list.


- **Required evidence:** a test asserting the selector's options derive from SUBJECT OPTIONS; a live screenshot of
  the expanded list; note to close GH #209.1. Under REQ-003 implementation reference.
- **Dependencies:** spec 009 (taxonomy).

## REQ-004 — Self-edit phone/email via an EMAIL-LINK confirmation flow with anti-account-takeover (FOUNDER OVERRIDE — #209.3)

- **Type:** functional / user experience / security · **Priority:** Must (founder override accepted 2026-06-22) · **Severity:** S3
  · **Source:** #209.3. **Founder decision (2026-06-22, supersedes the v1.2 "phone deferred/pending behind WhatsApp"
  behaviour): editing phone routes a confirmation link to the user's ALREADY-VERIFIED ACCOUNT EMAIL; clicking
  Confirm commits the new phone. Editing email routes the link to the NEW email.** This is the email-link fallback —
  NOT defer, NOT WhatsApp/one-time passcode.
- **Rationale:** The reviewer found phone/email non-editable on the tutor profile and could not self-correct a wrong
  contact value. The founder chose to let the user edit email/phone from the profile. **Routing a phone change
  through the account email makes this account-takeover-sensitive** (a hijacked session, or an attacker who controls
  the email, could re-point the recovery contact). The change is therefore gated by a single-use, short-expiry period,
  channel-bound token; the OLD value stays the contact of record until an explicit Confirm click commits it; both the
  old email AND old phone are notified at initiation; and the whole change is audited. This applies to the tutor
  profile and any other user surface that exposes editable contact fields.
- **Behaviour (EARS):**
  - **Step-up to INITIATE (MUST).** WHEN a user initiates an email or phone change, the system SHALL require a
    step-up re-authentication (current password, or a fresh/recent-session re-auth) BEFORE accepting the change. A
    request without valid step-up SHALL be rejected and SHALL NOT create a pending change.
  - **Pending state held; old value of record.** WHEN a contact change is accepted, the system SHALL store it as a
    PENDING change in the contact change request table (one row per request: user Id, field (EMAIL|PHONE),
    old/new contact snapshots under the plaintext-contact amendment, channel (EMAIL LINK), confirm/revoke token hashes, status, expires at,
    created at, consumed at, revoked at), keep the
    existing verified value as the contact of record, and surface channel-specific pending copy: EMAIL changes say
    "We sent a confirmation link to your new email address — your current email stays active until you confirm";
    PHONE changes say "We sent a confirmation link to your existing account email — your current phone stays active
    until you confirm". Any OTHER
    pending request for the SAME field SHALL be superseded (status-flagged, not row-deleted — soft-delete
    convention).
  - **Token issuance (MUST).** The system SHALL issue a secure-random token, store it **hashed at rest** (never
    plaintext), single-use, with a **short expiry period (30 minutes)**, bound to the user, changed field, requested new value and confirmation channel.
  - **Verify route = landing page + explicit Confirm (MUST — anti-prefetch).** The email link SHALL target a frontend
    landing page that REQUIRES an explicit Confirm click; the commit SHALL be performed by a **POST on confirm**,
    NOT a bare GET that auto-commits (email security scanners pre-fetch links — a bare-GET side-effect would
    auto-commit). The link/landing body SHALL name **the exact phone/email being authorised** (anti-confused-deputy).
  - **Email-edit vs phone-edit channel.** For an EMAIL change the link is sent to the NEW email (proves control of
    the new address). For a PHONE change the confirm link is sent to the user's EXISTING verified ACCOUNT EMAIL (the
    founder's chosen fallback — proves control of the account).
  - **Notify the old contacts at INITIATION (MUST).** WHEN a contact change is initiated, the system SHALL notify
    BOTH the OLD email AND the OLD phone (channel-appropriate) that a change was requested, including a
    "you didn't do this? revoke" affordance, BEFORE any commit — so a legitimate owner can stop it.
    Record each warning intent durably with the pending request and track queued/attempted/delivered/failed
    outcomes separately; enqueue is not proof of delivery. Old destinations are initiation-time snapshots.
    The old-phone transport/fallback while WhatsApp is OFF remains an explicit open decision below.
  - **Owner revocation (MUST).** Issue a separate purpose-bound, hashed, 30-minute revoke token to the OLD
    contact warnings. An explicit POST may change only PENDING → REVOKED, invalidate the confirmation token
    and retain the old contact. Authenticated owners may also revoke with cross-site request forgery protection protection. GET links never mutate.
    Confirm and revoke serialize on the same request row; only the first valid transition wins. A revoke after
    CONSUMED returns a conflict and support/recovery guidance; it cannot silently revert a completed change.
  - **Commit = atomic txn, uniqueness re-checked AT COMMIT (MUST).** WHEN the user confirms with a valid,
    unconsumed, unexpired token, the system SHALL, inside a single database transaction swap: re-check uniqueness of the
    new value against the live users email/users phone unique indexes (NOT only at request time — closing the
    TOCTOU); on a collision → **409** (reusing 011/REQ-004's PHONE ALREADY REGISTERED for phone, the email conflict
    for email) with the old value intact; otherwise write the new value, mark the request consumed at, write an
    audit row (old→new, actor, timestamp), and invalidate any other pending request for the same field. A verify
    REPLAY (token already consumed) SHALL be a no-op rejection.
  - **Unverified / expired = no effect.** An expired or unconfirmed token SHALL be discarded (lazy-expire or sweep)
    and SHALL NEVER mutate users email/users phone; the old value stays the contact of record and is used for all
    auth/contact operations while pending.
- **Owner:** software-engineer (token issuance + verify-then-swap + atomic commit + audit + consume-once) ·
  security-expert (step-up, single-use+expiry period token, old-contact notification, confirm-click landing, anti-takeover
  sign-off) · dba (contact change request table + migration + uniqueness-at-commit) · bff-specialist (initiate +
  verify endpoint contract + profile pending Contact Change fields) · frontend-expert / uiux-designer (edit +
  pending-state + landing/confirm page + phone-via-email-link copy)
- **Acceptance criteria (Gherkin):**


- Scenario: Step-up is required to initiate a contact change.
- Given a logged-in tutor.
- When they submit an email/phone change WITHOUT a valid step-up re-auth.
- Then the request is rejected and no pending change is created.
- Given a tutor with a verified email.
- When they change their email (with step-up).
- Then a single-use, 30-min, hashed-at-rest token is issued bound to user Id, EMAIL, new value.
- And a confirmation link is sent to the NEW email naming the exact email being authorised.
- And the OLD email remains the contact of record until confirmed.
- And BOTH the old email and old phone are notified with a revoke affordance.
- Scenario: Editing phone sends a link to the EXISTING account email (the founder fallback).
- Given a tutor with a verified phone and account email.
- When they change their phone (with step-up).
- Then a confirmation link is sent to the EXISTING verified account email naming the exact phone being authorised.
- And the OLD phone remains the contact of record until confirmed.
- Scenario: An email-scanner prefetch (bare GET) does NOT auto-commit.
- Given a pending contact-change link.
- When the link URL is fetched without an explicit Confirm (a GET prefetch).
- Then no commit occurs and the old value is intact.
- And only an explicit Confirm (POST) commits the change.
- Given a contact-change token already consumed (or past its 30-min expiry period).
- When it is presented to verify.
- Then the verify is a no-op rejection and the contact of record is unchanged.
- Scenario: Confirm commits atomically with uniqueness re-checked AT commit.
- Given a pending phone change whose new value was taken by another account AFTER initiation.
- When the user clicks Confirm.
- Then the commit re-checks uniqueness inside the swap transaction.
- And it returns 409 PHONE ALREADY REGISTERED with the old phone intact.
- Scenario: A verified change commits and is audited.
- Given a valid unconsumed unexpired token.
- When the user clicks Confirm.
- Then the new value becomes the contact of record atomically.
- And the request is marked consumed and an audit row records old→new, actor, timestamp.
- And any other pending request for the same field is invalidated.
- Scenario: Owner revokes before confirmation.
- Given an unexpired PENDING change and its distinct revoke credential.
- When the owner explicitly submits POST revoke.
- Then the request becomes REVOKED and the old contact stays active.
- And the confirm credential can no longer commit.
- And a repeated authorized revoke is a no-op success.
- Scenario: Confirm and revoke race.
- Given a PENDING request.
- When valid confirm and revoke requests arrive concurrently.
- Then exactly one terminal transition commits under the same row lock.
- And the loser cannot swap or revert the contact.


- **Required evidence:** backend tests (step-up required to initiate; token hashed/single-use/30-min-expiry period/channel-bound;
  bare-GET prefetch does not commit — only explicit POST-confirm does; old value used while pending; verify→atomic
  swap + audit row; uniqueness re-checked at commit → 409 with old intact; replay/expired = no-op; old-email+old-phone
  notification fired at initiation); frontend tests (pending banner, landing/confirm page requires a click, phone-via-email
  copy); a live capture of edit→pending→confirm→commit (email and phone). Under REQ-004 implementation reference.
- **Dependencies:** spec 001 (consent/audit patterns + the outbox EMAIL path, Outbox Event Type EMAIL /
  Email Payload); spec 004 (session/step-up re-auth primitives); spec 011 REQ-004 (PHONE ALREADY REGISTERED,
  reused at the commit-time uniqueness check); existing Password Reset Token (Password Reset Token implementation reference) as the
  hashed-single-use-expiry period token pattern to mirror. **Channel note:** the email-link flow is INDEPENDENT of INT-082
  WhatsApp/one-time passcode — phone re-verify here uses EMAIL, not WhatsApp; 018/REQ-001's WhatsApp-copy suppression must NOT
  suppress this email-link copy (see cross-wave note).

---

## Traceability summary

| REQ | One-line | Type | Priority | Severity | Owner |
|---|---|---|---|---|---|
| REQ-001 | Apply gate never silent; frontend↔backend completeness criteria reconciled to one source | functional | Must | S2 | bff-specialist · software-engineer · frontend-expert |
| REQ-002 | Sticky-Apply rail reflects applied/gated state | functional | Should | S4 | frontend-expert |
| REQ-003 | Tutor subject taxonomy = shared options (confirm + close #209.1) | functional/regression | Should | S3 | frontend-expert |
| REQ-004 | Self-edit email/phone via EMAIL-LINK confirm (token hashed+single-use+30-min expiry period; landing+explicit-Confirm POST; step-up; old stays until commit; uniqueness re-checked at commit→409; old email+phone notified; audited) — FOUNDER OVERRIDE | functional/user experience/security | Must | S3 | software-engineer · security-expert · dba · bff-specialist · frontend-expert · uiux-designer |

## Open questions for the Boss / founder

1. **REQ-004 (#209.3): RESOLVED — EMAIL-LINK FALLBACK (founder decision 2026-06-22, v1.3).** The v1.2 open
   infra-dependency is CLOSED: there is NO dependency on INT-082 WhatsApp/one-time passcode. Email change → confirm link to the NEW
   email; phone change → confirm link to the EXISTING verified account email. The link lands a page requiring an
   explicit Confirm (POST), token is hashed/single-use/30-min-expiry period/channel-bound; step-up required to initiate; old
   email AND old phone notified at initiation; uniqueness re-checked inside the atomic commit txn (→409); audited.
   This REQ adds a backend lane (contact change request table + migration) and is the heaviest item in spec 012 — scoped
   as its own Wave-2 slice; security-expert + dba re-gate before build.
2. **REQ-001 completeness source of truth: RESOLVED.** Return profile Complete/missing Fields from the same
   server predicate used by apply; the frontend consumes it. The aligned-constant alternative is superseded.
3. **Old-phone warning transport: PENDING.** Email-link confirmation is confirmed and remains independent of
   WhatsApp. The channel/fallback for the mandatory warning to the OLD phone while WhatsApp is OFF is not
   approved here. See [current open decisions](../CURRENT-REQUIREMENTS.md). Do not silently
   waive that warning, invent a blanket block on phone changes, or claim both-channel acceptance complete.
