# Auxiliary specification alignment: group1

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

37 auxiliary architecture/module/user interface files; normative targeted review, not implementation verification

Current user decisions: guest filtering allowed within limited results; PAID means agency receipt of tutor commission.

## Per-file matrix

| File | Verdict | Review | Assessment |
|---|---|---|---|
| [architecture/admin-matching-fe-spec md](../../../architecture/admin-matching-fe-spec.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Legacy assignment stopgap is superseded: §0a disables assignment without a listing, while §0b already uses tutor User Id. PRD20.1–20.4 and INT034 explicitly make listing optional; preserve the rest of the admin lifecycle user interface only after that rebase. |
| [architecture/durable-side-effects-outbox-spec md](../../../architecture/durable-side-effects-outbox-spec.md) | authority-gap | targeted normative extraction; implementation code/examples not exhaustively read | The document is explicitly a proposal, not a settled global durability policy. It honestly accepts the small commit→after Commit email-loss window while recommending an outbox for fan-out. Later006/007 use transactional outbox for WhatsApp; this does not silently approve every proposal or make historical email loss tolerance a current promise. |
| [architecture/email-resend-spec md](../../../architecture/email-resend-spec.md) | partial | full | Founder Resend decision supersedes SES/Mail Hog delivery choices; durable queue and secret-isolation intentions align. Recipient logging language needs the privacy logging rule applied, and historical delivery/status statements are not current implementation evidence. |
| [architecture/int-034-free-market-assign md](../../../architecture/int-034-free-market-assign.md) | aligned | targeted normative extraction; implementation code/examples not exhaustively read | Free-market assignment without a Tutor Post aligns with PRD20.1. Folded Boss condition at452–467 overrides the earlier no-index recommendation. Treat route options and migration database query language as historical proposals, not authority to reinstate a listing prerequisite. |
| [architecture/int-052-ux-typing-minimisation md](../../../architecture/int-052-ux-typing-minimisation.md) | aligned | full | Boss-approved selection-first typing reduction retains the brand and explicitly cuts sliders; onboarding ownership is moved to INT043. Later INT152/002 supersede single-area/MRT cardinality where scoped, while post planning area remains scalar. |
| [architecture/int-062-cascade-close-spec md](../../../architecture/int-062-cascade-close-spec.md) | partial | full | Founder close-live-post intent and application/invitation cascade align. The spec explicitly leaves interview cascade policy open and later021 introduces PENDING interview/invitation intake; closure/rejection/deletion handling must be reconciled across those paths, not inferred from built status. |
| [architecture/int-077-marketplace-quota-caps md](../../../architecture/int-077-marketplace-quota-caps.md) | aligned | full | Quota model is explicit, including the deliberate exception that accepting an invitation bypasses the application cap. Lock/count and remaining-quota rules are consistent with that exception; do not report an unconditional cap10 as the requirement. |
| [architecture/int-082-whatsapp-otp-and-notifications-spec md](../../../architecture/int-082-whatsapp-otp-and-notifications-spec.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | WhatsApp OFF/readiness fail-open and opt-in constraints align.012 expressly replaces immediate phone editing with pending email-link confirmation.006 expressly adds reminders, but its backend-only design does not carry forward INT082’s required consent-copy update; this remains a spec alignment gap before enablement. |
| [architecture/int-094-batch-soft-delete-integrity-spec md](../../../architecture/int-094-batch-soft-delete-integrity-spec.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Folded soft-delete and soft-favourite choices are clear. The single ACTIVE-and-not-deleted intake predicate relies on PENDING not being intake;021 explicitly widens invite/interview intake to PENDING, so this predicate and deletion acceptance coverage need a scoped amendment. This is a design propagation gap, not a code vulnerability claim. |
| [architecture/int-098-batch-marketplace-funnel-ux-spec md](../../../architecture/int-098-batch-marketplace-funnel-ux-spec.md) | aligned | targeted normative extraction; implementation code/examples not exhaustively read | Funnel decisions are explicit: REJECTED edit/resubmit, no reachable REVISION REQUESTED user interface, jobs route /assignments, real selection-withdraw endpoint, and progressive profile completion. These supersede older module prose; column500 versus FE280 is intentionally a stricter client cap, not a contradiction. |
| [architecture/int-121-batch-hygiene-spec md](../../../architecture/int-121-batch-hygiene-spec.md) | aligned | targeted normative extraction; implementation code/examples not exhaustively read | Security, application programming interface hygiene, typed financial input, accessible feedback and reusable user interface rules align with the stated scope. Historical code observations and rollout completion claims were not treated as independent requirements or re-verified. |
| [architecture/int-147-structured-notification-data-spec md](../../../architecture/int-147-structured-notification-data-spec.md) | aligned | full | Structured invitation notification JSON carries codes, optional labels and safe fallback; no new personal information or broad event migration is promised. Later notification vocabulary expansion can extend this explicit initial invitation scope. |
| [architecture/int-149-164-vm-review-batch-spec md](../../../architecture/int-149-164-vm-review-batch-spec.md) | aligned | full | Batch scope deliberately leaves matrix/commission to their own decisions, drops student Relationship system-wide, and makes profile areas multiple while keeping post area scalar. Later INT153/019/002 extend those separately owned slices; they do not make all location fields arrays. |
| [architecture/int-153-tutor-subject-level-matrix-spec md](../../../architecture/int-153-tutor-subject-level-matrix-spec.md) | partial | full | v2 folded matrix decisions and v3 detail-only display waiver are clear. The no-per-post apply gate is supported by PRD20.1 free-market scope, not merely the v3 BUILT label. Listing cross-product profile-alignment wording remains different from025 no-orphan validation (known taxonomy issue); preserve that distinction. |
| [architecture/issues-INT-001-005-fix-spec md](../../../architecture/issues-INT-001-005-fix-spec.md) | aligned | targeted normative extraction; implementation code/examples not exhaustively read | Stale auth becomes anonymous on public endpoints, public tutor cards are handle-only, unhandled errors surface, and business logs forbid raw contact/token values. Founder-verification status is historical governance, not evidence these rules remain unimplemented. |
| [architecture/m18-google-oauth-signin md](../../../architecture/m18-google-oauth-signin.md) | aligned | targeted normative extraction; implementation code/examples not exhaustively read | LOCKED §11 expressly overrides the earlier auto-linking and signup-wall alternatives: prove existing-account control, progressive tutor activation, encrypted stateless OAuth state, ADMIN exclusion, and backend flag. Renumbering is explicit; do not reopen these settled questions. |
| [architecture/regression-admin-flows-spec md](../../../architecture/regression-admin-flows-spec.md) | aligned | targeted normative extraction; implementation code/examples not exhaustively read | LOCKED §8 expressly supersedes shared-dev-super-admin password mutation and self-healing hacks with separate local-only regression cohort and independent operator bootstrap. Remaining open-question prose repeats decisions already settled and is stale, not a behavioral conflict. |
| [architecture/regression-improvement-spec md](../../../architecture/regression-improvement-spec.md) | authority-gap | targeted normative extraction; implementation code/examples not exhaustively read | A phased, pending-review improvement proposal with bounded exit criteria and advisory-only linter. It is aligned in intent with the approved regression lab, but not approval to build every phase or convert descriptive current GAP counts into permanent requirements.026/other later features can invalidate the historical guard list. |
| [architecture/regression-labs-proposal md](../../../architecture/regression-labs-proposal.md) | aligned | full | Explicit pre-deep-dive proposal, subsequently superseded by founder-approved regression-labs-spec and its LOCKED §12. Tool/model options, ephemeral-stack proposal and hard-promotion-gate wording are historical alternatives, not unresolved current product choices. |
| [architecture/solution-architecture md](../../../architecture/solution-architecture.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Broad original architecture remains mixed with explicit later amendments: M16 defers admin multi-factor authentication in dev and permits admin creation; M19/modern auth require cross-site request forgery protection;004 adds refresh sessions; founder Resend replaces SES;027 changes production infrastructure; M18 changes marketplace/contact/lifecycle. Base invariants such as explicit data transfer object allowlists and Liquibase ownership survive. Do not use the 2026-05-27 Approved heading to override those later decisions. |
| [modules/admin-management/spec md](../../../modules/admin-management/spec.md) | aligned | full | Founder access model and binding review resolve seemingly inconsistent seed profiles, display Name payload and password-change gate. Canonical rules are local-only seeder+flag, any admin creates, super-admin removes, current-database mutation gate, session revocation. Production multi-factor authentication is a separate before-prod requirement, not silently waived by dev-only No multi-factor authentication. |
| [modules/approval/api-reference md](../../../modules/approval/api-reference.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Developer application programming interface reference explicitly records unimplemented pagination/status guards as historical observations. Those observations must not override approval’s PENDING-only contract, later moderation constraints or reason-length updates (INT098). No present runtime defect is inferred. |
| [modules/approval/db md](../../../modules/approval/db.md) | partial | full | Immutable review log and required rejection reason are clear. Older PAUSED FOR REVIEW/drip scheduling state and reason enum require rebase to the current posting/moderation amendments; database structure definitions/code examples were checked as design intent, not assumed deployed schema. |
| [modules/approval/module-overview md](../../../modules/approval/module-overview.md) | partial | full | Developer overview mixes approval requirements with dated implementation observations. Its fail-open audit rule conflicts with the later explicit in-transaction audit invariant; historical encryption/provider, REJECTED-terminal and REVISION REQUESTED descriptions have later amendments. Keep admin approval as publication gate. |
| [modules/auth/backend md](../../../modules/auth/backend.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | New012 contact-change section is folded correctly, but the older auth body still describes mandatory admin multi-factor authentication, disabled cross-site request forgery protection and pre-refresh/old registration contracts. M16, M19,003,004,012 are the supersession path; the appended wave2 section does not re-authorize its stale opening sections. |
| [modules/auth/frontend md](../../../modules/auth/frontend.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Public tutor detail, request browsing, three-step signup, multi-factor authentication and server-side cookie middleware prose are legacy baselines. M17/PRD20, progressive registration,003 password reset,004 refresh and012 contact editing govern current flows. Password-complexity wording differs from old backend and must use the unified later auth policy. |
| [modules/commission/sequence-diagrams md](../../../modules/commission/sequence-diagrams.md) | partial | full | Activation and PAID bookkeeping diagrams describe the old admin workflow, not payer authority. Current user confirms PAID means agency receipt from tutor. Cancellation-as-is is superseded by assignment lifecycle commission cancellation; first-session confirmation/receipt order must follow the current parent requirements, not be inferred from the diagram’s missing steps. |
| [modules/interview/backend md](../../../modules/interview/backend.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | 021 PENDING eligibility was appended, but most of the body remains the pre-negotiation REQUESTED/admin-only flow. INT039 expressly replaces request creation and widens cancellation/scheduling states;006/007 own reminder delivery. The contact-until-approved phrase also inherits the known021 versus assignment privacy conflict. |
| [modules/interview/db md](../../../modules/interview/db.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Original five-state enum, scheduling link constraint and post-bound interview data model need rebase to INT039 negotiation and free-market identity. Retention/no-hard-delete intent survives; erasure pseudocode does not by itself supply a viable FK-compatible anonymization design. |
| [modules/interview/frontend md](../../../modules/interview/frontend.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | The original user interface and guards are pre-negotiation. INT039 provides proposal/counter/accept/decline and participant cancellation;021 allows PENDING parent request intake. Preserve mobile/a11y and Singapore time requirements, but refresh scheduling validation and contact copy from the current contracts. |
| [modules/landing-showcase/design md](../../../modules/landing-showcase/design.md) | aligned | targeted normative extraction; implementation code/examples not exhaustively read | The header and §B explicitly say the founder waived M14b consent/legal gate, and point to the replacement design. Cached showcase/available flag/personal information allowlist remain the baseline; retained deferred-gate text is labelled history and must not be resurrected as a blocker. |
| [modules/matching/api-reference md](../../../modules/matching/api-reference.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Old application programming interface reference carries public real-name sample,7-bit availability and listing-required pair matching. INT003 enforces handle-only public identity;009 fixes21-bit availability; PRD20/INT034 make profile identity and optional listing canonical. These source-derived examples cannot override the later rules. |
| [modules/matching/er-diagram md](../../../modules/matching/er-diagram.md) | partial | full | Historical ERD omits tutor user id and requires tutor post id and globally unique post pairs. INT034’s folded migration/index requirements supersede those relationships; duplicate active-match protection remains required. Treat simplified user/table diagrams as historical, not current schema approval. |
| [modules/matching/sequence-diagrams md](../../../modules/matching/sequence-diagrams.md) | partial | full | Historical two-post matching flow is superseded by PRD20 atomic assignment and INT034 optional listing. Retain deliberate user-selection/confirmation and error feedback intent; its preserved cached selection and no-post invalidation are descriptive examples, not a new current contract. |
| [modules/matching/spec md](../../../modules/matching/spec.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Original administrator-only, never-share-contact, first-session→COMPLETED model is explicitly revised by PRD20 and assignment lifecycle/INT034. Current user settles payer direction. The old APPROVED enum vocabulary and open questions are stale; one active arrangement per tutee remains valid. |
| [modules/posting/frontend md](../../../modules/posting/frontend.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Historical posting wizard/public routing/status vocabulary has later amendments (INT052/098,002,009,013,021,025/026). The file also contradicts itself about schedule-step placement: original rates→venue→availability order versus final mandatory schedule-before-budget rule. Preserve draft-persistence and accessible selection intentions without treating older wire examples as final route/field policy. |
| [uiux/redesign-modern-readable-spec md](../../../uiux/redesign-modern-readable-spec.md) | partial | targeted normative extraction; implementation code/examples not exhaustively read | Brand-preserving and accessibility/readability scope is clear, but its locked never-ever contact promise conflicts with the later assignment-stage disclosure rule. The original brand lock remains authoritative for this slice; later005 testimonial/content decisions can supersede old landing examples. Current anonymous-filter rule overrides any login gate suggested by historical restyles. |

## Source-paired findings

### AUX1-F01 — Reminder scope expansion does not carry forward its explicit consent-copy condition

Classification: **partial-coverage**. 006 clearly authorizes interview reminders and preserves OFF. The unresolved alignment is the prior lockstep frontend consent requirement, not founder approval of reminder cost; no present delivery or implementation violation is asserted.

[architecture/int-082-whatsapp-otp-and-notifications-spec md](../../../architecture/int-082-whatsapp-otp-and-notifications-spec.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[006-whatsapp-interview-reminders/design md](../../../006-whatsapp-interview-reminders/design.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[006-whatsapp-interview-reminders/requirements md](../../../006-whatsapp-interview-reminders/requirements.md)


Implementation example omitted. This public edition retains the written requirements in this section.


**Proposed default:** Carry the reminder event into consent wording and its copy-pinning acceptance check before enabling WhatsApp; explicitly waive or retain any re-consent requirement instead of inferring it.

### AUX1-F02 — PENDING intake expansion needs the soft-delete predicate updated

Classification: **partial-coverage**. INT094’s rationale excludes PENDING from intake;021 deliberately expands only invite/interview intake to PENDING. This is a state-predicate/acceptance-coverage amendment, not proof deleted posts are currently actionable.

[architecture/int-094-batch-soft-delete-integrity-spec md](../../../architecture/int-094-batch-soft-delete-integrity-spec.md)


Implementation example omitted. This public edition retains the written requirements in this section.


- the set status operation is guarded by equals ACTIVE so a delete of a DRAFT/PENDING/REJECTED/MATCHED/CLOSED post does not illegally rewrite its status (those are not live-intake states anyway; only ACTIVE accepts new intake). The post stays deleted at differs from no value regardless, so it is hidden from all listings via the existing query filters.
- **supersede Open For Post** — a new set-based bulk update records on Tutor Selection Request Repository flipping the single CREATED row for the post → SUPERSEDED (mirror close Open Applications's set-based shape; @Modifying @Query). Verify the existing CREATED-only partial-unique index (one open request per job) is honoured — SUPERSEDED is outside it, so no conflict.
- **Applicant/invitee notification fan-out is REQUIRED, not optional (team-review 2026-06-11):** notify the affected applicants and invitees, reusing the neutral close Live Job copy — **"A job you applied to is no longer available."** — **NEVER** the moderation/fraud reason. Capture the recipient ids BEFORE the bulk UPDATEs (INT-016 capture-before-clear discipline), after Commit fan-out.
- Audit row (EVENT POSTING DELETED) and the structured log already exist (274-287); extend the log line with superseded Selections, and add a micrometer counter **moderation selections superseded** beside the existing closed-counters (Observability Do D, team-review 2026-06-11).

**2. Admin Match Service create Match defence-in-depth (Admin Match Service implementation reference, at the 183 guard):**


Implementation example omitted. This public edition retains the written requirements in this section.


**2a. Domain-predicate refinement (agreed, team-review 2026-06-11):** introduce **the is open for intake operation** — status equals Post Status ACTIVE and deleted At equals no value — as the **single named home of the intake rule**. The assign-path belt above (and any future intake guard) calls it instead of inlining the check, so the rule has one source of truth.

**3. No change to the other five guards** — once status is CLOSED, their existing status differs from ACTIVE checks reject the deleted post (returning their current * NOT ACTIVE 422). Add a one-line code comment at each pointing to this spec so a future reader knows the deleted At invariant is upheld upstream by delete Posting.


The illustration for aux1-f02 — pending intake expansion needs the soft-delete predicate updated contains the following information:

- [architecture/int-094-batch-soft-delete-integrity-spec md](../../../architecture/int-094-batch-soft-delete-integrity-spec.md).


  - admin submit operation for admin matches (assign) → **422** (caught by the new deleted At belt at create Match, and by CLOSED status).
  - invite-accept fold (Job Invitation Service accept) on the removed job → **422** (site 6).
- **[live-lab]** **Moderation Delete Supersedes Selection IT** — parent has a CREATED selection request → admin deletes the post → assert the selection request is SUPERSEDED and no longer appears in read operation for admin selection-requests; a subsequent assign on it → 4xx (not a Match).
- **[live-lab]** **Notification fan-out (REQUIRED)** — applicants/invitees of the deleted job each receive the neutral "A job you applied to is no longer available." notification; assert the moderation reason appears NOWHERE in the payload.
- **[live-lab]** **Non-regression guard** — deleting a DRAFT/PENDING post does NOT change its status (only deleted at set); assert status unchanged.


The illustration for aux1-f02 — pending intake expansion needs the soft-delete predicate updated contains the following information:

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


### Step 2 — Rate & Experience

**Fields:**
- **Hourly Rate Range** (required): Two SGD currency inputs — Min rate and Max rate. Display as S XX – S XX/hr.
  - Min: 1–500 SGD
  - Max: must be ≥ min, max 1000 SGD


The illustration for step 2 — rate & experience contains the following information:

- [modules/posting/frontend md](../../../modules/posting/frontend.md).


### Step 3 — Venue & Location

**Fields:**
- **Venue Types** (multi-select, required, 1+):
  - Student's Home
  - Tutor's Home
  - Public Place (library/café)
  - Online (Zoom / Google Meet)
  - Tuition Centre
- **Location** (optional): District picker (D01–D28) or MRT line + station. Toggle between district and MRT mode.

**MRT Station Picker:** Searchable dropdown of all Singapore MRT stations, grouped by line (NSL, EWL, CCL, DTL, TEL, NEL).

---

### Step 4 — Availability

**Availability Grid Component:**


The illustration for step 4 — availability contains the following information:

- [modules/posting/frontend md](../../../modules/posting/frontend.md).


### Availability grid — step placement

The schedule step (availability grid + session duration + frequency) **must come before** the budget/venue step. Users should anchor their availability commitment before deciding on financial commitment. See wireframe Screen 6, Step 2.


Implementation example omitted. This public edition retains the written requirements in this section.


| **BR-Approval-1 (Start state)** | Admin can only approve or reject a post in PENDING status. Attempting to act on any other status results in a 404 (the current service implementation does not validate status before acting — see Debugging Guide §4 for implications). |
| **BR-Approval-2 (Rejection reason required)** | A rejection requires a non-blank reason field in the request body, validated by @Not Blank at the controller and enforced by a database trigger on post review log. |
| **BR-Approval-3 (Audit required)** | Every approve and reject action writes an entry to audit log (table: audit log, event types: ADMIN POST APPROVE, ADMIN POST REJECT). Audit failure is logged but never blocks the action. |
| **BR-Approval-4 (Email notification)** | Every approve and reject action queues an async email to the post owner via Redis Streams key email-notifications. Email failures are logged but never block the action. |
| **BR-Approval-5 (No double-review guard in current impl)** | The current Admin Post Service does **not** check current post status before updating. A 409 idempotency guard (specified in the backend spec) is not yet implemented in the running code. See Debugging Guide §4. |


The illustration for availability grid — step placement contains the following information:

- [architecture/durable-side-effects-outbox-spec md](../../../architecture/durable-side-effects-outbox-spec.md).


2. **INT-066 audit rows → Option A (in-transaction).** Move the Auth Service (125,184,
   190, 293, 401, 424, 566, 594) and Admin User Service (108, 175)
   audit saves **inside** their a database transaction bodies (atomic with the action), exactly as the
   INT-065 fix. Audit rows are part of the business fact; they are never after Commit. Remove
   the dead after Commit helpers (Auth Service, Admin User Service) if no non-audit
   caller remains; the the publish operation calls bundled in those hooks


Implementation example omitted. This public edition retains the written requirements in this section.


**frontend payload:** the tutor-listing and tuition-request identifiers (same as existing create Match Mutation). When tutor Post Id is null (tutor applied without a listing), disable the Assign button with helper text "This tutor applied without a listing — ask them to complete their profile first."

**Also fix Interview Queue implementation reference:** The existing assign Mutation sends the tuition-request and tutor-account identifiers which is incorrect. Change to the selected tutor-listing and tuition-request identifiers. Add null-guard: disable assignment when the selected record has no tutor listing on the Confirm button of Close Job Assign Modal when tutor Post Id is null.


The illustration for availability grid — step placement contains the following information:

- [product requirements document md](../../../PRD.md).


### 20.1 The model — free marketplace (founder)

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

- [architecture/int-034-free-market-assign md](../../../architecture/int-034-free-market-assign.md).


## Boss-review condition (2026-06-03) — 0033f duplicate-match guard
Team review (Morgan/DBA) found the unique operation becomes a no-op
for free-market matches (both tutor post id NULL) → no database-level dedupe under concurrent admins.
**Required addition** to changeset 0033:


Implementation example omitted. This public edition retains the written requirements in this section.


Keep an app-level pre-check for a friendly 409 (MATCH ALREADY EXISTS); the partial-unique is the
integrity backstop. Mirror the active-status set used elsewhere if it differs.


Implementation example omitted. This public edition retains the written requirements in this section.


**Guards:**
1. Interview must exist → 404
2. Interview must be in REQUESTED status → 422 INVALID STATUS TRANSITION
3. scheduled At must be a future datetime (> NOW operation + 30 minutes) → 422 SCHEDULED AT MUST backend FUTURE
4. meeting Link must be a valid URL (http/https scheme, valid hostname) → 422 INVALID MEETING LINK


The illustration for boss-review condition 2026-06-03 — 0033f duplicate-match guard contains the following information:

- [modules/interview/backend md](../../../modules/interview/backend.md).


1. Ownership check → 403
2. Status must be REQUESTED → 422 (cannot cancel after admin has scheduled; contact admin)

**Processing:**
1. Update interviews: status CANCELLED, the now operation, cancellation reason CANCELLED BY TUTEE
2. Queue N-21 cancellation notice to admin (variant: "Interview request cancelled by tutee") to InTuition async email queue (Spring @Async + Redis)
3. If interview was in SCHEDULED status: Queue N-24 cancellation notice to tutor (the tutor already received the meeting link in N-23 and must be informed)


The illustration for boss-review condition 2026-06-03 — 0033f duplicate-match guard contains the following information:

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


The illustration for boss-review condition 2026-06-03 — 0033f duplicate-match guard contains the following information:

- [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md).


### REQ-001 — TUT-1 allow invite/interview on a PENDING post (S2, FOUNDER-CONFIRMED)
A tutee with a PENDING (awaiting-approval) request can invite a tutor / request an interview; the tutor's
**contact stays gated until the post is approved**. The old false copy ("you don't have an active request") is gone.
- **Given** a tutee whose only request is PENDING, **when** they open a tutor and click Invite / Request Interview,
  **then** the action is allowed (not blocked), **and** no contact is revealed until approval.
- **Guard:** **Given** the post is later REJECTED, **then** the pending invite/interview does not leak contact and
  is voided/closed cleanly (soft).


Implementation example omitted. This public edition retains the written requirements in this section.


**Non-negotiables preserved verbatim:**
1. Hero headline "Meet your tutor before you commit." (Landing implementation reference).
2. The privacy promise — "your contact details are never shared with any tutor,
   ever" and every personal information-free public surface.
3. Admin purple separation (#7 C3 AED shell) — untouched; admin is the LAST phase.
4. Commission **50% / SGD** prominence on the tutor registration commission page.


The illustration for req-001 — tut-1 allow invite/interview on a pending post s2, founder-confirmed contains the following information:

- [modules/matching/spec md](../../../modules/matching/spec.md).


- **Admin is the gatekeeper.** No party can initiate, accept, or decline a match. Only admin staff create matches.
- **Contact information stays private.** Neither party sees the other's phone number or email address at any stage. Admin sees contact info in the matching dashboard to perform manual outreach off-platform.
- **One active arrangement per tutee post.** A tutee post may only be in one PENDING or ACTIVE match at a time. This prevents double-booking a tutee and enforces the agency's curated matching promise.
- **Tutors can scale.** A tutor post may participate in multiple simultaneous matches, reflecting that professional tutors often serve multiple students.
- **Commission follows first session confirmation.** The commission lifecycle is tracked separately on the commissions table. Match status (PENDING/ACTIVE/COMPLETED/CANCELLED) and commission status are independent dimensions.


The illustration for req-001 — tut-1 allow invite/interview on a pending post s2, founder-confirmed contains the following information:

- [product requirements document md](../../../PRD.md).


### 20.6 personal information contract (binding — extends BR-001)

- **Parent applicant view is a DISTINCT personal information-free data transfer object** (NOT Admin Application Response) with its own
  **contract test**: handle, subjects, levels, rate, **coarse** area, YOE, PT/FT, rating — **NO real
  name, email, phone, or exact location**.
- **Child's name + parent's phone are hidden from applicants until the parent commits** (assignment).
- **Owner-authorization** on the applicant-list + choose-tutor endpoints (IDOR → 403).
- The public **assignment record** on a tutor profile shows subject / level / period / status only —
  **never the counterparty's identity**.


The illustration for personal information contract binding — extends br-001 contains the following information:

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


## 2. Admin Cancels a Match

Cancellation is permitted from PENDING or ACTIVE. No commission is created. If a commission already existed (cancelling an ACTIVE match), it is loaded and included in the response but its status does not change.


The illustration for . admin cancels a match contains the following information:

- [modules/commission/sequence-diagrams md](../../../modules/commission/sequence-diagrams.md).


The administration service looks up any commission attached to the match. If one exists, it returns that record without changing its invoice status; otherwise it returns no commission record.

The illustration for . admin cancels a match contains the following information:

- [architecture/assignment-lifecycle-design md](../../../architecture/assignment-lifecycle-design.md).


| admin activates | MATCHED | MATCHED | SUPERSEDED | PENDING → **ACTIVE** (commission PENDING) |
| **parent pays after 1st session → admin marks PAID** | MATCHED | MATCHED | SUPERSEDED | ACTIVE → **COMPLETED** (commission **PAID**) |
| admin cancels (pre-complete) | MATCHED → **ACTIVE** | MATCHED → **APPLIED** | SUPERSEDED → **APPLIED** | → **CANCELLED** (commission CANCELLED) |


The illustration for . admin cancels a match contains the following information:

- **Proposed default:** Document PAID as agency-received tutor commission, preserve first-session/collection ordering from the chosen current policy, and replace the old cancellation diagram.
- ### AUX1-F09 — Admin bootstrap and OAuth alternatives are settled by explicit folded decisions.
- Classification: **explicit-supersession**. These are examples of stale opening prose, not genuine unresolved requirements: M16 binds local-only seeding/mutation gating, M19 overrides auto-linking and inline activation, regression-admin forbids touching the human dev login.
- [modules/admin-management/spec md](../../../modules/admin-management/spec.md).


## 2. Default super-admin seed (DEV ONLY)
- A **bootstrap seeder** (Spring Command Line Runner/Application Runner) runs **only under the
  local/dev profile** (never prod). It is **idempotent**: if a super-admin already exists, do
  nothing.
- Seeds REDACTED email / admin as role is ADMIN, is super admin is yes.
- Email is AES-SIV encrypted + keyed message authentication and the password BCrypt-hashed **via the existing
  Encryption Service + Password Encoder** (cannot be done in plain Liquibase database query language — that's why a
  seeder, not a changeset).
- password change required yes — the account is forced to change the password before it can
  perform admin actions (see §5). **No multi-factor authentication** (founder: "no multi-factor authentication right now").
- The seeder runs in any environment **except prod** (so it populates the dev Docker stack the
  founder actually uses), and is idempotent.


The illustration for . default super-admin seed dev only contains the following information:

- [modules/admin-management/spec md](../../../modules/admin-management/spec.md).


## 8. Review #1 resolutions (binding — fold into the build)

**Seeder gating (Security+SWE):** gate exactly the profile operation (there is NO dev profile; !prod would wrongly fire under test and pollute the coverage build). Docker backend runs local (compose verified) so it DOES populate the founder's stack. Add a second guard: a seed admin.enabled property (default true in local) so it can be switched off. Idempotent: look up the seed by its email hmac; wrap the insert and **catch Data Integrity Violation Exception** as a no-op (survives races/restarts and the the database query language restriction operation soft-delete blind spot). Never log the seed/temp password.

**Shared creation path (SWE):** extract a shared method from the private the build user operation that both create-admin and the seeder call. It MUST handle a **null phone** (admins have none; phone encrypted/phone hmac are nullable) — current build User NPEs on null phone. Drop display Name from the admin model (admins have no profile table; the get display name operation returns "Admin"); identify admins by email. Create payload is email and a temporary password.

**password change required gate (Security+SWE):** a **centralized service-layer guard** (NOT a servlet filter — protects branch coverage) checked per-request against current database state at the top of every admin ** interface operation mutation EXCEPT change-password; returns PASSWORD CHANGE REQUIRED. It must block submit operation for admins too. The existing change-password flow (submit operation for auth change-password) MUST be updated to **clear password change required** on success (today it does not, so the gate would never lift).


The illustration for . review #1 resolutions binding — fold into the build contains the following information:

- [architecture/m18-google-oauth-signin md](../../../architecture/m18-google-oauth-signin.md).


## 11. Team Review Decisions (LOCKED — panel + Boss, 2026-05-30)

Cleared to implement; these override §1–§9 where they differ. Implementation Boss
review will verify every item.

1. **Account linking = Option B, FINAL.** No silent auto-link, ever. If a Google
   sign-in's verified email matches an existing account that has a usable password, the
   user must **prove control** (enter the existing password OR click an emailed
   one-time link) before google sub is bound. Auto-link permitted ONLY when the
   existing account has **no usable password AND no prior successful login**. Linking is
   always explicit, authenticated, and **audited**. sub is the permanent join key —
   **never re-link by email** after the initial bind. Errors are generic (no account
   enumeration).
2. **Tutor activation = SPLIT, FINAL.** After Google, the user answers ONE question
   (parent or tutor) → a usable account is created immediately (can browse). Commission
   + qualifications gate ONLY *posting a listing / applying to a job* (progressive
   onboarding — no inline paperwork wall at signup). Parent = one tap + role = done.


The illustration for . team review decisions locked — panel + boss, 2026-05-30 contains the following information:

- [architecture/regression-admin-flows-spec md](../../../architecture/regression-admin-flows-spec.md).


1. **Q2 — SEPARATE regression super-admin. FINAL.** Add a the profile operation seeder for
   REDACTED email (role is ADMIN, is Super Admin is yes, a known reg password, NOT
   password change required — or with it, cleared once in bootstrap). The suite **NEVER touches
   REDACTED email** — the human dev login stays REDACTED email /admin, untouched. This KILLS
   D2a (self-healing two-password hack) and D4 (shared-credential mutation side effect). Both are
   struck from the spec.

2. **Q3 — dedicated REGRESSION SEED-BAND, REQUIRED (not just disposable rows).** A the profile operation
   regression cohort the Dev Data Seeder/demo data NEVER populates: emails *@regression local,
   and a reserved marker for posts/matches/commissions (a regression tag column OR a reserved ID
   band — DBA to pick). All admin MUTATIONS (approve/reject/match/activate/cancel/pay/delete) operate
   ONLY inside this band. Demo data stays presentable; re-runs stay green; no assert-and-restore
   (non-atomic, untrustworthy). Provide a reset/re-seed hook for the band, pre-suite.

3. **D1 — idempotent the admin bootstrap operation in a shared base @Before All, memoized per JVM.** JUnit
   @Order is within-class only and Failsafe gives no cross-class ordering → every admin IT extends
   a base whose @Before All ensures the gate-cleared ops-admin exists (idempotent). Admin Bootstrap IT
   ASSERTS the journey; it does NOT provide the fixture for the others. Bootstrap **fails LOUD** if it
   cannot establish the operator — never silent-skip the admin suite.


Implementation example omitted. This public edition retains the written requirements in this section.


### ADR-005: cross-site request forgery protection — Same Site is Strict Sufficient, Csrf Filter Disabled

**Status:** Accepted

**Decision:** Disable Spring Security's Csrf Filter. Do not generate or validate cross-site request forgery protection tokens.

**Justification:** cross-site request forgery protection attacks require the browser to send the auth cookie to a cross-origin attacker-controlled endpoint. Same Site is Strict prevents the browser from sending the auth cookie on any cross-origin request, including form POST and the fetch operation calls from other origins. Combined with same-origin delivery via the Cloudflare proxy (ADR-003) and encrypted web request protocol-only, cross-origin cookie delivery is structurally impossible.

**Precondition:** This decision is valid **only** as long as ADR-003 (Cloudflare proxy) is in place. If the Spring Boot application programming interface is ever deployed directly under a different domain (not proxied via Cloudflare under example com), cross-site request forgery protection protection must be re-evaluated and re-enabled before the first user request.

---

### ADR-006: signed session token Token Lifetime — 24h TUTEE/TUTOR, 30-min Idle ADMIN

**Status:** Accepted

**Decision:**
- TUTEE / TUTOR: 24-hour signed session token with jti denylist on logout (Redis key the session-revocation record for the token identifier, expiry period = remaining token expiry). No refresh token in MVP.
- ADMIN: 30-minute idle expiry period in Redis jti allowlist. Forced re-login after idle. Each request resets the expiry period (sliding window).


The illustration for adr-006: signed session token token lifetime — 24h tutee/tutor, 30-min idle admin contains the following information:

- [architecture/m18-google-oauth-signin md](../../../architecture/m18-google-oauth-signin.md).


6. **Statelessness + cross-site request forgery protection:** use a cookie-based Authorization Request Repository that is
   **encrypted** (it carries the PKCE code verifier), HttpOnly, Secure, Same Site is Lax,
   expiry period ≤10 min, deleted on callback. Preserve Session Creation Policy STATELESS
   (oauth2Login's default Http Session repo must NOT ship). The success handler issues
   **both** our signed session token cookie AND a fresh cross-site request forgery protection cookie (else the first post-login mutation
   403s).


The illustration for adr-006: signed session token token lifetime — 24h tutee/tutor, 30-min idle admin contains the following information:

- [architecture/email-resend-spec md](../../../architecture/email-resend-spec.md).


**Owner:** Backend Tech Lead · **Source of truth:** the service code.
**Supersedes:** assignment-be-spec md §9 (Mailhog) — see "Why this replaces Mailhog" below.
**Founder decision (2026-05-31):** integrate **Resend** for email delivery; a **dev-only** application programming interface key is
provided.


The illustration for adr-006: signed session token token lifetime — 24h tutee/tutor, 30-min idle admin contains the following information:

- [modules/admin-management/spec md](../../../modules/admin-management/spec.md).


## 1. Access model (ratified by founder)
- **Any ADMIN can CREATE other admin accounts.**
- **Only the SUPER-ADMIN can delete/demote/disable an admin**, and the super-admin row is
  **protected** — it cannot be deleted, demoted, or disabled by anyone (no self-delete either).
- Super-admin is modelled as a **flag** users is super admin BOOLEAN required DEFAULT no,
  NOT a new role — a super-admin is still role ADMIN with the protection + management powers.


The illustration for . access model ratified by founder contains the following information:

- **Proposed default:** Publish an amendment map or rebase architecture text, preserving enduring data transfer object/ownership/data invariants and clearly scoped dev-versus-prod differences.
- ### AUX1-F11 — Old public matching samples conflict with current identity and availability contracts.
- Classification: **explicit-supersession**. Public real-name output and7-bit interpretation are historical examples, not valid requirements. Public handle-only identity and21-bit grid are explicitly specified elsewhere.
- [modules/matching/api-reference md](../../../modules/matching/api-reference.md).


Returns a page of tutor-listing results.  All fields are safe for public display — no personal information is exposed.


The . access model ratified by founder example describes information including data, id, tutor Display Name. Concrete payload values and implementation syntax are omitted.


[modules/matching/api-reference md](../../../modules/matching/api-reference.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[architecture/issues-INT-001-005-fix-spec md](../../../architecture/issues-INT-001-005-fix-spec.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[architecture/solution-architecture md](../../../architecture/solution-architecture.md)


The illustration for . access model ratified by founder contains the following information:

- The domain has a well-defined relational structure: users → posts → matches → commissions, with explicit foreign key relationships and cardinality constraints enforced by partial unique indexes.
- Spring Data Java persistence mapping Hibernate provides type-safe repositories, JPQL fetch joins for N 1 prevention, and lifecycle callbacks for audit logging.
- Liquibase owns all schema changes. spring jpa hibernate ddl-auto is validate in production — Hibernate validates but never modifies the schema.
- Partial indexes (match cardinality enforcement) are managed exclusively in Liquibase changesets.
- The 21-bit availability bitmask is stored as a PostgreSQL INTEGER, allowing bitwise queries in the admin matching dashboard filter.


**Proposed default:** Replace or label sample data transfer objects and bitmask notes historical; use current handle-only data transfer object allowlist and21-bit availability specification.

### AUX1-F12 — Matching completion on first-session confirmation differs from later payment completion

Classification: **explicit-supersession**. The matching module says first-session confirmation itself completes the match. PRD20’s explicit reconciliation says confirmation precedes collection and admin PAID, which completes it. Current user further clarifies receipt is tutor commission paid to agency; root handles that payer wording drift.

[modules/matching/spec md](../../../modules/matching/spec.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[product requirements document md](../../../PRD.md)


Implementation example omitted. This public edition retains the written requirements in this section.


**Proposed default:** Rebase the matching state-transition table to the current first-session→commission-receipt→PAID sequence, without introducing a second payment gate.

## Coverage and limitations

- All37 named files were opened through line-preserving extracts; full versus targeted coverage and reviewed ranges are recorded per file.
- Supporting implementation code/examples were treated as historical and not verified against source or runtime.
- Referenced later clauses were read selectively;001–009 full primary-document audit is in separate artifacts.
- Aligned means normative clauses agree after explicit folds/supersessions; it does not certify implementation or delivery status.

The JSON includes precise reviewed line ranges and the document reference list. No source, test or runtime changes were made.
