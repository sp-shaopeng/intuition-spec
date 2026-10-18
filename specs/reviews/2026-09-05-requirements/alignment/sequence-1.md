# Sequence alignment

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Sequence01–14: full text and diagrams read; later-requirement comparison targeted

Code-derived descriptive flows, not independent authority. Old GAP/current-code notes were not runtime-verified or promoted to policy.

Current user rules: anonymous filtering within limited results; PAID means agency receipt from tutor.

## Per-file matrix

| File | Verdict | Assessment |
|---|---|---|
| [01-register-tutee md](../../../sequences/01-register-tutee.md) | partial | Older credential-only register request and audit-after-commit omit001 required consent flags, server timestamps, policy versions and atomic audit.004 owns the later session contract. This is stale descriptive coverage, not a current application programming interface defect. |
| [02-register-tutor md](../../../sequences/02-register-tutor.md) | partial | Older two-step session Storage/password diagram is superseded by001 in-memory consent draft and002 founder-approved optional profile capture page. The new capture remains skippable; no signup completion wall is authorized. |
| [03-login-with-mfa md](../../../sequences/03-login-with-mfa.md) | partial | Unconditional ADMIN multi-factor authentication trigger differs from founder M16 dev deferral.004 adds non-admin30-minute access and refresh sessions. Preserve production multi-factor authentication as a separate gate; the old frontend Gap is not a current implementation finding. |
| [04-forgot-reset-password md](../../../sequences/04-forgot-reset-password.md) | partial | one-time passcode email/deep-link and non-enumeration intent align. Attempt counting, audit timing, session survival and reset cleanup are older than003;004 additionally invalidates pre-reset refresh families. |
| [05-csrf-bootstrap md](../../../sequences/05-csrf-bootstrap.md) | partial | cross-site request forgery protection-before-hydration and explicit public-route exception align with guest browsing. The diagram oversimplifies invalid-signed session token handling relative to004 silent refresh and auth-probe handling; distinguish anonymous hydration from recoverable expiry. |
| [06-tutee-post-creation md](../../../sequences/06-tutee-post-creation.md) | partial | Create-and-submit/PENDING and financial validation align at a high level.009/021 extend form and edit behavior. The old no-admin-notification and analytics-before-commit observations are dated, not current requirements or defects. |
| [07-tutor-post-creation md](../../../sequences/07-tutor-post-creation.md) | partial | Listing creation stays approval-gated and optional relative to free-market apply.002 canonical profile/multi-MRT and025 listing-pair validation are absent from the old high-level flow; old notification Gap prose is descriptive only. |
| [08-admin-approval md](../../../sequences/08-admin-approval.md) | partial | PENDING-only approve/reject aligns.019 adds queue pagination/filter and021 PENDING owner edit/rejection cascade. Fail-open audit and no-live-close observations are superseded by the later atomic-audit and INT062 close rules. |
| [09-admin-match-creation md](../../../sequences/09-admin-match-creation.md) | partial | Old two-ACTIVE-post matching is superseded by PRD20 atomic assignment and INT034 tutor-user identity with optional listing. Rebase winner/supersede/job transitions and one active match per tutee. |
| [10-admin-match-activation md](../../../sequences/10-admin-match-activation.md) | partial | Activation and50% commission creation align with the later lifecycle. Legacy admin-email disclosure diagram is incomplete coverage of current contact disclosure/first-session follow-up; no new payer direction is defined. |
| [11-admin-mark-commission-paid md](../../../sequences/11-admin-mark-commission-paid.md) | partial | PAID bookkeeping and match completion are compatible with agency receipt from tutor.019 explicitly requires confirmation before the pay mutation on both admin surfaces, whereas the sequence POSTs after initial click; remaining payable-state guards are not fully illustrated. |
| [12-admin-match-cancel-cascade md](../../../sequences/12-admin-match-cancel-cascade.md) | aligned | Dated INT016 fold and diagram reopen job/applications and notify former tutor after commit. PENDING commission cancellation and preserving PAID align with agency receipt (line97 explicitly says it). Older no-notification Gap98–101 is stale prose, already superseded within the file. |
| [13-tutee-request-interview md](../../../sequences/13-tutee-request-interview.md) | partial | Pre-negotiation note-only REQUESTED flow is superseded by INT039 proposal/accept/counter and expanded active-state cap, plus021 PENDING parent-job eligibility. Anonymous mutation gate is valid and says nothing against guest filtering. |
| [14-admin-schedule-interview md](../../../sequences/14-admin-schedule-interview.md) | partial | Optional link and Singapore time convention remain. INT039 adds PENDING SCHEDULING source; founder INT082 explicitly replaces tutee-only with both-party scheduling notification.006/007 own gated WhatsApp/reminder delivery. |

## Source-paired findings

### SEQ1-F01 — Registration diagrams omit required consent and atomic audit

**explicit-supersession**. 001 is explicit current consent policy. Missing sequence coverage does not prove current signup drops consent.

[sequences/01-register-tutee md](../../../sequences/01-register-tutee.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[sequences/01-register-tutee md](../../../sequences/01-register-tutee.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[001-legal-and-compliance-consent-and-policy-pages/requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md)


Implementation example omitted. This public edition retains the written requirements in this section.


  Scenario: Tutee register with all required consents true persists them
    Given a valid tutee registration payload with terms Accepted is yes, pdpa Consent is yes, age Confirmed is yes
    When POST auth register tutee interface operation is called
    Then the response is success (account created)
    And the persisted user has the three consent flags true with timestamps set by the server


The illustration for seq1-f01 — registration diagrams omit required consent and atomic audit contains the following information:

- **Required evidence:** curl log of POST auth register tutee interface operation with the consent fields → success, followed by the database row showing the three flags timestamps. (Tutor path is REQ-006.).


[001-legal-and-compliance-consent-and-policy-pages/requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md)


The illustration for seq1-f01 — registration diagrams omit required consent and atomic audit contains the following information:

- Scenario: A CONSENT GIVEN audit row is written on successful registration.
- Given a successful tutee or tutor registration with the three required consents true.
- When the registration transaction commits.
- Then an audit log row exists with event type = "CONSENT GIVEN" for that user id.
- And its details JSON records terms Accepted, pdpa Consent, marketing Opt In, age Confirmed and the terms version privacy version consented to.
- And it is saved in the same transaction as USER REGISTERED (consistent with Auth Service implementation reference INT-066 note).


- **Required evidence:** a private inspection of the user’s consent audit record, including the event, account identifier, consent choices and policy versions; confirmation the row is committed atomically with USER REGISTERED.


Implementation example omitted. This public edition retains the written requirements in this section.


User selects "I am a tutor" on Register implementation reference. The first form does **not** call
the application programming interface — it stashes the form values in session Storage under
intuition tutor Registration and navigates to /register/commission. The
commission page collects the binding acknowledgement and posts the full payload
to auth register tutor interface operation.


The illustration for seq1-f01 — registration diagrams omit required consent and atomic audit contains the following information:

- [sequences/02-register-tutor md](../../../sequences/02-register-tutor.md).


    User leads to >FE1: Select "I am a tutor", fill form
    FE1 leads to >SS: set Item using stash email, +65phone, password, display Name
    FE1 leads to >User: navigate using /register/commission
    Note over FE2: Mount guard — if stash missing → toast + bounce to /register?role is tutor
    User leads to >FE2: Read commission terms, enter qualifications + years, tick acknowledge
    FE2 leads to >SS: get Item using stash — abort if absent/invalid
    FE2 leads to >Ctrl: POST auth register tutor interface operation stash+qualifications+years Experience+commission Acknowledged:true+commission Terms Version:"2026-01-01"


The illustration for seq1-f01 — registration diagrams omit required consent and atomic audit contains the following information:

- [001-legal-and-compliance-consent-and-policy-pages/requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md).


  Scenario: The tutor draft carries the four consent booleans (AC-a)
    Given a visitor on Register implementation reference Step 1 with role is tutor who ticked the three required consents (and optionally marketing)
    When they advance to the commission step
    Then the in-memory tutor Registration Draft (tutor Registration Draft implementation reference) carries terms Accepted, pdpa Consent, marketing Opt In, and age Confirmed alongside email/phone/display Name/password

  Scenario: Step 1 blocks advancing to commission when a required consent is unticked (AC-b)
    Given a visitor on Register implementation reference Step 1 with role is tutor who left a required consent box (terms / Personal Data Protection Act / age) unticked
    When they click "Continue to commission terms →"
    Then the form does not advance to the commission step (no draft is set / no navigation)
    And an inline validation message is shown on the unticked required box

  Scenario: Commission implementation reference includes the four consent booleans in the register Tutor payload (AC-c)
    Given a complete tutor Registration Draft carrying the four consent booleans
    When Commission implementation reference submits to API ROUTES auth.register Tutor (~Commission implementation reference)
    Then the POST body built at ~Commission implementation reference-87 includes terms Accepted, pdpa Consent, marketing Opt In, and age Confirmed


The illustration for seq1-f01 — registration diagrams omit required consent and atomic audit contains the following information:

- [002-tutor-profile-and-onboarding-ux/requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md).


- **Source:** GH #197.2; founder decision 2026-06-20 (Option b — expand onboarding)
- **Rationale / value:** So the profile page can pre-populate the **full** REQ-008 set, onboarding must add a
  **new capture page** to today's 2-page arc (Register → [NEW capture] → Commission → POST register Tutor) that
  COLLECTS and PERSISTS onto Tutor Profile at registration: **hourly Rate Min/Max, planning-areas, region,
  subject-levels, commitment, AND qualifications + years Experience.** The last two are explicitly in scope
  because REQ-008's premise ("prefill qualifications/years from onboarding") has **no source today** — the frontend
  draft + Commission payload omit them (see grounding), so without capturing them REQ-008 cannot meet its AC.
  All fields stay **OPTIONAL** (INT-043's intent — never re-block signup is preserved): a tutor who provides
  them prefills the profile; a tutor who skips them registers exactly as today and leans on the completion
  banner. **No new schema** — every target column already exists on Tutor Profile (rate columns,
  planning-areas + subject-levels junctions, location region, commitment, and the scalar
  qualifications/years experience).
- **Scope is NOT a small edit (honest cost):** this requires (1) a **new onboarding-capture page + route**
  inserted between Register and Commission, (2) an **extended Tutor Registration Draft** shape carrying the new
  fields (today only 4), (3) an **extended Commission/register payload** carrying them into
  application programming interface ROUTES auth.register Tutor, (4) the **new Register Tutor Request data transfer object fields**, and (5) the
  **persistence wiring** in Auth Service register Tutor. Costed as tasks 18a–18e (not one task).


Implementation example omitted. This public edition retains the written requirements in this section.


User submits the Login form on Login implementation reference. For TUTEE / TUTOR the flow ends
with a signed session token cookie; for ADMIN the backend returns a short-lived mfa session
cookie and the frontend must POST a 6-digit time-based one-time passcode to auth login mfa interface operation to
exchange it for the final signed session token.


The illustration for seq1-f01 — registration diagrams omit required consent and atomic audit contains the following information:

- [modules/admin-management/spec md](../../../modules/admin-management/spec.md).


## 5. First-login hardening for seeded / newly-created admins
- password change required is yes → login succeeds but the account is gated: the only action
  allowed is change-password (reuse the existing change-password flow); admin application programming interfaces return a
  PASSWORD CHANGE REQUIRED signal until it's cleared.
- **No multi-factor authentication** in this slice (founder directive). Admin login is email + password only. (Any existing
  multi-factor authentication code paths stay feature-flagged off; this feature does not add or require multi-factor authentication.)


The illustration for . first-login hardening for seeded / newly-created admins contains the following information:

- [modules/admin-management/spec md](../../../modules/admin-management/spec.md).


**Before-prod gate (Security):** seeder provably inert in prod (allowlist + flag + a startup test); secure super-admin provisioning (random out-of-band password); re-enable multi-factor authentication on admin login. Tracked for the foundations phase — NOT in this dev slice.


The illustration for . first-login hardening for seeded / newly-created admins contains the following information:

- [004-auth-session-hardening-30min-ttl-and-refresh/requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md).


## REQ-009 — frontend single-flight silent-refresh-then-retry on a 401, else redirect to login

- **Type:** user interface
- **Priority:** Must
- **Source:** INT-013 "refresh-or-relogin on expiry"; frontend-expert.
- **Rationale / value:** When a protected call returns 401 because the access token just expired, the SPA must call
  /auth/refresh **once**, and on success transparently retry the original request — the user never sees a logout.
  If refresh fails, THEN the SPA logs out and redirects to login. The retry must be one-shot (no loop), mirroring the
  existing csrf Retried one-shot pattern in api implementation reference.
- **Single-flight (the frontend half of the concurrency self-Do S, REQ-003):** when several protected calls 401 at the same
  moment (e g. a dashboard fan-out), they must NOT each submit operation for auth refresh — concurrent refreshes from the same
  client would race the backend rotation and trip reuse-detection. Instead the interceptor holds **one shared in-flight
  refresh promise**: the first 401 starts the refresh; every concurrent 401 **awaits the same promise** and, on its
  resolution, replays its own original request. The shared promise is cleared once settled (so a later, genuinely-new
  expiry can refresh again).
- **Interceptor ordering + no recursion (grounded in api implementation reference):** today api implementation reference (~lines 99-102) calls
  the should trigger logout operation and logs out immediately on any non-probe 401. The silent-refresh attempt MUST run
  **BEFORE** that should Trigger Logout branch — only if the single-flight refresh itself fails does control fall
  through to logout/redirect. A 401 returned **by /auth/refresh itself** must BYPASS the refresh attempt (no


Implementation example omitted. This public edition retains the written requirements in this section.


    alt token attempt Count  is at least  5
        Svc- leads to >Ctrl: 429 OTP ATTEMPTS EXCEEDED
    end
    Svc leads to >Pwd: matches using otp, otp Hash
    alt mismatch
        Svc leads to >Repo: increment Attempt Count using token Id
        Svc- leads to >Ctrl: 400 INVALID OTP
    end
    Svc leads to >Repo: token used At is now; save
    Svc leads to >Pwd: encode using new Password
    Svc leads to >Repo: user Repository update Password Hash using user Id, new Hash
    Note over Svc: after Commit: Audit LogPASSWORD RESET, user Id


The illustration for req-009 — frontend single-flight silent-refresh-then-retry on a 401, else redirect to login contains the following information:

- [sequences/04-forgot-reset-password md](../../../sequences/04-forgot-reset-password.md).


- Admin password reset does NOT invalidate active Redis admin sessions — a
  warning is logged (line 454). They expire naturally.


The illustration for req-009 — frontend single-flight silent-refresh-then-retry on a 401, else redirect to login contains the following information:

- [003-password-reset-failure-fix/design md](../../../003-password-reset-failure-fix/design.md).


- **Backend** — Auth Controller (submit operation for auth forgot-password ~L162, submit operation for auth reset-password
  ~L169) → Auth Service forgot Password (~L496) / Auth Service reset Password (~L582). forgot Password issues a
  6-digit one-time passcode via issue Reset Token (~L541): invalidates prior tokens, stores a **bcrypt hash** of the one-time passcode in
  password reset tokens with a **15-min** expiry (uq pvt one live = at most one live token per user), and
  enqueues the one-time passcode email on the durable outbox. reset Password (~L582): per-(ip,account) rate-limit (429
  RATE LIMIT EXCEEDED) → resolve user (else 400 INVALID one-time passcode) → latest valid token (else 400 INVALID one-time passcode) →
  **Redis** one-time passcode-attempt cap (429 one-time passcode ATTEMPTS EXCEEDED) → the matches operation (else bump
  Redis counter + 400 INVALID one-time passcode) → mark token used → update Password Hash And Clear Change Required (clears the
  hash, failed login attempts, locked until, AND password change required) → audit row in-tx →
  after Commit session revocation.


The illustration for req-009 — frontend single-flight silent-refresh-then-retry on a 401, else redirect to login contains the following information:

- [004-auth-session-hardening-30min-ttl-and-refresh/requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md).


  Scenario: A refresh family created at-or-before a password reset is rejected
    Given a TUTEE reset their password (pwreset:after:<id> epoch written)
    And the user holds a refresh family whose created At Epoch is  is at most  that reset epoch
    When the user POSTs auth refresh interface operation
    Then the response is 401 (Auth Error Code REFRESH INVALID) and no new access cookie is issued
    And both cookies are cleared


Implementation example omitted. This public edition retains the written requirements in this section.


    [*] - leads to  PENDING : tutee/tutor create And Submit
    PENDING - leads to  ACTIVE : admin Post Service approve Post
    PENDING - leads to  REJECTED : admin Post Service reject Post
    ACTIVE - leads to  PENDING : owner update (edit-while-active)
    DRAFT - leads to  PENDING : owner submit (DRAFT not exercised by current frontend; create-and-submit is the only path)
    DRAFT - leads to  CLOSED : owner soft Delete
    REJECTED - leads to  CLOSED : owner soft Delete
    ACTIVE - leads to  CLOSED : (no frontend path today)


The illustration for req-009 — frontend single-flight silent-refresh-then-retry on a 401, else redirect to login contains the following information:

- [sequences/08-admin-approval md](../../../sequences/08-admin-approval.md).


- Audit failures are swallowed (lines 416-419) so audit infra issues never
  block the admin operation.


The illustration for req-009 — frontend single-flight silent-refresh-then-retry on a 401, else redirect to login contains the following information:

- [architecture/durable-side-effects-outbox-spec md](../../../architecture/durable-side-effects-outbox-spec.md).


2. **INT-066 audit rows → Option A (in-transaction).** Move the Auth Service (125,184,
   190, 293, 401, 424, 566, 594) and Admin User Service (108, 175)
   audit saves **inside** their a database transaction bodies (atomic with the action), exactly as the
   INT-065 fix. Audit rows are part of the business fact; they are never after Commit. Remove
   the dead after Commit helpers (Auth Service, Admin User Service) if no non-audit
   caller remains; the the publish operation calls bundled in those hooks


The illustration for req-009 — frontend single-flight silent-refresh-then-retry on a 401, else redirect to login contains the following information:

- [architecture/int-062-cascade-close-spec md](../../../architecture/int-062-cascade-close-spec.md).


**Tracker:** INT-062 (S2) · **Founder decisions:** (1) a tutee/admin **can withdraw a LIVE job** that already has
applications/invitations; (2) distinct new CLOSED statuses.

## Problem / founder rule
Once a tutor↔job **match** exists, OR a tutee/admin **withdraws (closes) a live job**, **every** application to that
job AND every job invitation/recommendation must move to a terminal **"job no longer available"** status — staying
**visible to everyone** (applicant, invited tutor, admin), never hidden/removed (transparency; soft-delete-only).


The illustration for problem / founder rule contains the following information:

- **Proposed default:** Use current atomic audit and owner-close transition, labelling previous observations historical.
- ### SEQ1-F06 — Two-listing creation omits atomic assignment.
- **explicit-supersession**. PRD20 and INT034 explicitly remove mandatory-listing identity and add winner/supersede/job transitions.
- [sequences/09-admin-match-creation md](../../../sequences/09-admin-match-creation.md).


    Admin leads to >frontend: Pick tutor Post Id + tutee Post Id, click Create
    frontend leads to >Ctrl: POST admin matches interface operation tutor Post Id, tutee Post Id
    Note over Ctrl: @Pre Authorize has Role using ADMIN (class) + URL has Role using ADMIN
    Ctrl leads to >Svc: create Match using tutor Post Id, tutee Post Id, admin User Id
    Svc leads to >TPRepo: find By Id using tutor Post Id
    alt missing
        Svc- leads to >Ctrl: 404 TUTOR POST NOT FOUND
    end
    alt tutor Post status  differs from  ACTIVE
        Svc- leads to >Ctrl: 422 TUTOR POST NOT ACTIVE


The illustration for problem / founder rule contains the following information:

- [sequences/09-admin-match-creation md](../../../sequences/09-admin-match-creation.md).


    Svc leads to >MRepo: exists By Tutor Post Id And Tutee Post Id And Status In using PENDING, ACTIVE
    alt open match exists
        Svc- leads to >Ctrl: 409 DUPLICATE MATCH
    end
    Svc leads to >MRepo: save using Match status is PENDING, created By is admin User Id, matched At is now Coordinated Universal Time
    par Notify both parties (M9)
        Svc leads to >NS: emit using tutee User Id, MATCH CREATED, Match created, body, /tutee/matches
        and
        Svc leads to >NS: emit using tutor User Id, MATCH CREATED, Match created, body, /tutor/matches
    end
    Svc- leads to >Ctrl: Match Responseid, tutor Post Id, tutee Post Id, PENDING, matched At
    Ctrl- leads to >frontend: 201 Created


The illustration for problem / founder rule contains the following information:

- [product requirements document md](../../../PRD.md).


Any tutor can express interest / **apply to any ACTIVE job with NO published listing required**.
Applying is open expression of interest. The old TUTOR NO ACTIVE POST 403 guard
(Job Application Service) is **removed**; the only remaining apply guard is "job must be ACTIVE"
(TUTEE POST NOT ACTIVE 422). Drive-by spam is throttled by a **profile-completeness gate** (rate +
area + commitment + qualifications) instead. tutor Post Id on an application stays **optional**.

### 20.2 Canonical identity = tutor profile (not listing)

Because applicants may have no listing, the **profile** carries the public identity: rate range,
area, YOE, **Part-time/Full-time**, rating. Listings become optional showcases. Model =
**profile-canonical + listing-overrides-when-attached** (no sync job). tutor profiles gains
commitment, hourly rate min/max sgd, location planning area, location region (none exist
today) + a **backfill** from each tutor's most-recent post. See §5.1a.


The illustration for canonical identity tutor profile not listing contains the following information:

- [product requirements document md](../../../PRD.md).


### 20.4 The atomic "assign" transition (one a database transaction)

Extends Admin Match Service create Match (inject Job Application Repository):
1. Chosen tutor's application → **MATCHED**; link match source application id.
2. Job (Tutee Post) ACTIVE → MATCHED (**reuse existing Post Status MATCHED** — no new enum;
   drops from public browse, blocks new applies).
3. **All other APPLIED/SHORTLISTED applications for that job → SUPERSEDED** (NEW status) via a
   **single set-based update records** (never a read-loop; index on tutee post id, status).
4. **Optimistic-version check** on the Tutee Post so two admins can't double-assign (2nd → 409).
5. Notifications fan-out (superseded / matched / parent) is **after-commit + failure-isolated** — a
   notification error must not roll back the assignment.
6. On **cancel** (pre-complete): reopen — job MATCHED→ACTIVE, superseded apps →APPLIED (Q1).

**Both entry points converge on ONE transition (the solution architect):** the existing admin 2-panel Match Creation
(§7.3) and the new parent-selection-driven assign both call the *same* extended
Admin Match Service create Match — there is **no second match path**. The parent selection request
(§20.3) is just a pre-step that tees up which (tutee, tutor) the admin confirms.


The illustration for the atomic assign transition one @transactional contains the following information:

- **Proposed default:** Rebase to tutor-user/optional-listing assignment including job, applications, cardinality guard and fan-out.
- ### SEQ1-F07 — Financial mutation sequence skips required confirmation.
- **explicit-supersession**. 019 requires confirmation on both admin surfaces. PAID direction is settled by user and corroborated by sequence12:97.
- [sequences/11-admin-mark-commission-paid md](../../../sequences/11-admin-mark-commission-paid.md).


    Admin leads to >frontend: Click Mark Paid on commission id
    frontend leads to >Ctrl: POST admin commissions id pay interface operation
    Ctrl leads to >Svc: pay Commission using id
    Svc leads to >CRepo: find By Id using id
    alt missing
        Svc- leads to >Ctrl: 404 COMMISSION NOT FOUND
    end
    alt commission invoice Status  equals  PAID
        Svc- leads to >Ctrl: 409 COMMISSION ALREADY PAID
    end
    Svc leads to >CRepo: set Invoice Status using PAID, set Paid At using now, save
    Svc leads to >MRepo: match set Status using COMPLETED, save
    Svc leads to >NS: emit using tutor User Id, COMMISSION PAID, Commission marked paid, Your commission of SGD <amount> has been marked as paid., /tutor/matches


The illustration for the atomic assign transition one @transactional contains the following information:

- [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md).


### REQ-001 — ADM-01 Mark-Paid confirm on the Matching page (S2, money)
The highest-consequence financial action must be guarded on **both** surfaces.
- **Given** an admin on /admin/matching with a payable commission, **when** they click "Mark PAID",
  **then** a confirm dialog appears naming amount + tutor + tutee + "this is irreversible", **and** the pay
  mutation fires **only** from the dialog's confirm button.
- **Guard:** **Given** the confirm dialog is open, **when** the admin clicks Cancel, **then** no mutation fires
  and the commission stays unpaid.
- Design: port Commission implementation reference's Pay Confirm modal (26,,) to Matching implementation reference
  (currently the mutate operation fires on click). frontend-only — backend already has Commission.@Version +
  Idempotency-Key (defence-in-depth intact).
- **Regression test (e2e, matching spec):** click Mark-Paid → assert dialog visible AND no POST .../pay
  network call until confirm is clicked.


The illustration for req-001 — adm-01 mark-paid confirm on the matching page s2, money contains the following information:

- [sequences/12-admin-match-cancel-cascade md](../../../sequences/12-admin-match-cancel-cascade.md).


- Service: Admin Match Service implementation reference. The comment block (lines 241-254)
  documents the design intent of the cascade — uncollectable amounts must not
  silently inflate outstanding revenue.
- CANCELLABLE STATUSES PENDING, ACTIVE (line 57).
- Commission PAID is intentionally NOT touched — agency already received payment.


Implementation example omitted. This public edition retains the written requirements in this section.


> **2026-06-03 (INT-016, Sev-1 — FIXED):** cancel previously **500'd** with a
> Lazy Initialization Exception. Root cause was NOT open-in-view: the M18 cancel→reopen step calls
> the reopen applications operation, a the modifying operation query that
> **detaches the persistence context mid-transaction**; the subsequent the get tutor operation/the get subject codes operation
> on the now-detached Match/Tutor Post proxies threw. Fixed by capturing the former-winner id + the
> response's subject codes **before** the clearing query and building the response from primitives.
> Cancel now also **reopens the job** (MATCHED→ACTIVE + winner/superseded apps→APPLIED) and emits an
> after-commit, failure-isolated "match cancelled / job reopened" notification to the former tutor
> (Admin Match Service cancel Match).


The illustration for req-001 — adm-01 mark-paid confirm on the matching page s2, money contains the following information:

- [sequences/12-admin-match-cancel-cascade md](../../../sequences/12-admin-match-cancel-cascade.md).


    Note over Svc: after commit — notify former tutor "match cancelled / job reopened"; (failure-isolated; former Winner Id resolved pre-detach)


The illustration for req-001 — adm-01 mark-paid confirm on the matching page s2, money contains the following information:

- [sequences/12-admin-match-cancel-cascade md](../../../sequences/12-admin-match-cancel-cascade.md).


- **Gap**: no notification is emitted on cancel — cancel Match lacks
  notification Service emit calls (unlike create and activate). The match
  dashboard reflects the new status on next fetch, but neither party gets a
  bell ping.


Implementation example omitted. This public edition retains the written requirements in this section.


        frontend leads to >frontend: Open modal, capture note (optional)
        Tutee leads to >frontend: Submit
        frontend leads to >Ctrl: POST interviews interface operation tutor Post Id, note
        Note over Ctrl: Security Config has Role using TUTEE on POST interviews interface operation
        Ctrl leads to >Svc: request Interview using tutee Id, req
        Svc leads to >TPRepo: find By Id using req tutor Post Id
        alt missing
            Svc- leads to >Ctrl: 404 TUTOR POST NOT FOUND
        end
        alt tutor Post status  differs from  ACTIVE
            Svc- leads to >Ctrl: 422 TUTOR POST NOT ACTIVE
        end
        Svc leads to >IRepo: exists By Tutee Id And Tutor Post Id And Status In using REQUESTED, SCHEDULED
        alt duplicate
            Svc- leads to >Ctrl: 422 DUPLICATE INTERVIEW
        end
        Svc leads to >URepo: find By Id using tutee Id
        Svc leads to >IRepo: save using Interview status is REQUESTED, requested At is now Coordinated Universal Time, note


The illustration for req-001 — adm-01 mark-paid confirm on the matching page s2, money contains the following information:

- [sequences/14-admin-schedule-interview md](../../../sequences/14-admin-schedule-interview.md).


    alt status  differs from  REQUESTED
        Svc- leads to >Ctrl: 422 INVALID STATUS TRANSITION
    end
    Svc leads to >Svc: interview status = SCHEDULED
    Svc leads to >Svc: scheduled At = req scheduled At at Zone(Zone Id of using Asia/Singapore).to Offset Date Time operation
    opt INT-035 — meeting Link present & non-blank
        Svc leads to >Svc: interview meeting Link = req meeting Link trim operation
    end
    Svc leads to >IRepo: save using interview
    Svc leads to >NS: emit using tutee Id, INTERVIEW SCHEDULED, Interview scheduled, Your interview for \<title>\ is scheduled for <scheduled At>., /tutee/interviews/ id


The illustration for req-001 — adm-01 mark-paid confirm on the matching page s2, money contains the following information:

- [sequences/14-admin-schedule-interview md](../../../sequences/14-admin-schedule-interview.md).


- Only the tutee is notified — not the tutor. (Tutor was already notified at
  the request stage in flow 13.)


The illustration for req-001 — adm-01 mark-paid confirm on the matching page s2, money contains the following information:

- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md).


| submit operation for interviews | TUTEE | Propose Interview Request (below) | (new) → PROPOSED | **Replaces** the old REQUESTED-creating call: now requires 1–3 slots. |
| submit operation for tutor interviews id accept | TUTOR | the chosen time-slot identifier | PROPOSED/COUNTER PROPOSED using tutee round → PENDING SCHEDULING | Picks one open tutee slot; ACCEPTED, siblings REJECTED. |
| submit operation for tutor interviews id decline | TUTOR | the reason (+ optional slots when CANT MAKE IT) | PROPOSED/COUNTER PROPOSED → TUTOR DECLINED (NOT INTERESTED) **or** COUNTER PROPOSED (CANT MAKE IT + slots) | NOT INTERESTED terminal+frees slot; CANT MAKE IT with slots = decline+counter in one call. |
| submit operation for tutor interviews id counter | TUTOR | one to three proposed start times | COUNTER PROPOSED/PROPOSED (tutee round) → COUNTER PROPOSED | Pure counter (when tutor just wants to propose alternatives without a decline reason — folded into decline for CANT MAKE IT; kept for symmetry/user experience). |
| submit operation for tutee interviews id accept | TUTEE | the chosen time-slot identifier | COUNTER PROPOSED (tutor round) → PENDING SCHEDULING | Picks one open tutor slot. |
| submit operation for tutee interviews id counter | TUTEE | one to three proposed start times | COUNTER PROPOSED (tutor round) → COUNTER PROPOSED | Tutee proposes a fresh round. |
| submit operation for tutee interviews id cancel | TUTEE | — | non-terminal → CANCELLED | **Existing** endpoint; widen allowed source statuses (§5). |
| submit operation for tutor interviews id cancel | TUTOR | — | non-terminal → CANCELLED | **New** — tutor may bail out of an in-flight negotiation. |
| read operation for tutee interviews id, read operation for tutor interviews id | owner | — | — | Return negotiation detail incl. timeslots + whose Turn. (Tutor detail is **new**.) |
| submit operation for admin interviews id schedule | ADMIN | the agreed interview time and meeting link | PENDING SCHEDULING (and legacy REQUESTED) → SCHEDULED | **Widen** source to include PENDING SCHEDULING (INT-035 reuse). |
| submit operation for admin interviews id force-pending | ADMIN | an optional chosen time-slot identifier | any non-terminal → PENDING SCHEDULING | Admin override: pick a proposed slot (or just advance) when parties stall. |
| submit operation for admin interviews id cancel | ADMIN | — | non-terminal → CANCELLED | **Existing**; widen source statuses. |


The illustration for req-001 — adm-01 mark-paid confirm on the matching page s2, money contains the following information:

- [architecture/int-082-whatsapp-otp-and-notifications-spec md](../../../architecture/int-082-whatsapp-otp-and-notifications-spec.md).


> **Founder amendment 2026-06-07 (BUILT, deployed dev — INT-084 + INT-085):** WhatsApp messaging is
> pricey, so the delivered notification set is narrowed to **INTERVIEW SCHEDULED only** (plus the
> one-time passcode, which is verification, not a notification), now **config-driven** via
> whatsapp notifications.events (default INTERVIEW SCHEDULED) and filtered at outbox dispatch —
> re-expansion is config-only **and requires founder cost approval + a lockstep frontend consent-copy
> update**. Additionally, INTERVIEW SCHEDULED now notifies **both tutor and tutee** (in-app + WhatsApp
> outbox; was tutee-only — the tutor-no-show trust gap).


The illustration for req-001 — adm-01 mark-paid confirm on the matching page s2, money contains the following information:

- [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md).


### REQ-001 — TUT-1 allow invite/interview on a PENDING post (S2, FOUNDER-CONFIRMED)
A tutee with a PENDING (awaiting-approval) request can invite a tutor / request an interview; the tutor's
**contact stays gated until the post is approved**. The old false copy ("you don't have an active request") is gone.
- **Given** a tutee whose only request is PENDING, **when** they open a tutor and click Invite / Request Interview,
  **then** the action is allowed (not blocked), **and** no contact is revealed until approval.
- **Guard:** **Given** the post is later REJECTED, **then** the pending invite/interview does not leak contact and
  is voided/closed cleanly (soft).
- Design: **backend change** — Job Invitation Service implementation reference (ERR JOB NOT ACTIVE) and Interview Service implementation reference
  (ERR TUTEE POST NOT ACTIVE) currently hard-block non-ACTIVE; widen to accept PENDING (keep contact-reveal
  gated on approval). frontend — widen the gate at Tutor Detail implementation reference + Invite Tutor Modal implementation reference
  and fix copy to "your request is awaiting approval; you can invite now — contact is shared once approved".


Implementation example omitted. This public edition retains the written requirements in this section.
