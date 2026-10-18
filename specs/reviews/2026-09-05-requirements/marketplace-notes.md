# InTuition marketplace requirements audit

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](corrections.md) and [current requirements](../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Scope: onboarding, profiles, parent requests, tutor listings, public discovery, applications, invitations. Read-only source/spec audit; no tests and no repository edits. Paths below are relative to REDACTED local path and line references are from current files. The user's latest rule—anonymous visitors can filter with limited search—is authoritative. Anonymous filtering is therefore not a question to reopen.

## Requirement inventory

| Feature | Current requirement baseline and notable constraints | Evidence |
|---|---|---|
| Account / onboarding | Distinct tutor and parent roles. Tutor account creation includes affirmative commission acknowledgement. Profile capture is offered during onboarding but may be skipped; account and captured profile facts persist atomically. No forced completion or mandatory published listing before exploring. | intuition-spec/specs/product requirements document md; specs/002-tutor-profile-and-onboarding-ux/spec md; its requirements md |
| Tutor identity/profile | Canonical identity and default rate/area live on profile. Public handle is suggested, editable, validated and collision-safe. Seven completeness fields gate applying: handle, commitment, min rate, max rate, planning areas, qualification,  is at least 1 subject-level pair. frontend consumes the same result as backend. Onboarding facts prefill profile; qualification/experience prefill a new listing. | product requirements document md; 012-tutor-apply-and-profile-ux/requirements md; 002-*/requirements md; 022-*/spec md; 026-*/spec md |
| Contact edits | Self-service edits use step-up authentication and single-use 30-minute email-link confirmation; email change goes to new email, phone change to existing verified email. Old value remains until explicit POST confirmation; uniqueness rechecked at commit; revoke/old-contact notice/audit required. This supersedes the WhatsApp-only/deferred phone-change approach. | 012-tutor-apply-and-profile-ux/requirements md |
| Parent requests | One subject and one level; student count 1–20, with 6+ exact-count input; budget, venues, duration and frequency; at least one availability slot. Explicit Review/Submit, field validation, all venues and schedule visible in saved details. DRAFT/REJECTED/ACTIVE/PENDING editable, with ACTIVE/PENDING edits reentering review. MATCHED/CLOSED/EXPIRED immutable. | modules/posting/spec md,125-141; 009-*/requirements md; 008-*/spec md,56-61; 013-*/requirements md,65-79; 021-*/spec md; 026-*/spec md |
| Tutor listings | Optional showcase; multiple subjects/levels, rates, venues, group sizes, qualifications, experience, teaching approach. Exact hourly availability persisted; 2-hour click option and mobile default. Multiple optional MRT stations. Posts require approval and editing active posts reenters approval. Profile capability and listing scope remain distinct. | modules/posting/spec md,74-99,373-384; 002-*/requirements md,248-268; product requirements document md |
| Drafts and continuity | Both creation flows have explicit draft-autosave requirements: step navigation, 30 seconds inactivity, unload; saved/failed indicator, server draft, continue/discard one existing draft. Wizard internal Back must preserve values. | product requirements document md; modules/posting/spec md; architecture/int-052-ux-typing-minimisation md; 026-*/spec md |
| Public discovery | Lists and filters public; details login-gated; intent returns after login. User's current limited-anonymous-filtering rule controls. Assignment browse: subject, level, region, budget, page/size; 60 requests/min/IP,  is at most 5 pages,  is at most 20/page, 100-result ceiling, planning-area precision maximum. Authenticated actors may perform role-appropriate apply/invite actions. | product requirements document md; architecture/m17-marketplace-ux-overhaul md,57-87,141-150,194-197; latest user clarification |
| Taxonomy | Shared subject/level options and the private implementation folder drift checks. Band coherence hard-blocks invalid combinations; 025 narrows Secondary to approved stream/year pairs across request, listing, profile and registration. Listing validation accepts sets with no orphan subject or level. Legacy data is validate-on-write only. Profile pair matrix drives browse capability filters and soft admin eligibility warning; no per-job matrix eligibility gate on Apply. | 020-*/spec md; 025-*/spec md; 025-*/requirements md; architecture/int-153-tutor-subject-level-matrix-spec md,120-140 |
| Applications | Any profile-complete tutor can apply to any ACTIVE job, no active listing required. Optional pitch and attached listing; attached listing overrides applicant-card rate/area. Existing application replay idempotent; withdraw/reactivate in place; status-specific reapply rules. Tutor cap 10 active APPLIED/SHORTLISTED, invitation acceptance bypasses cap. Withdrawal allowed APPLIED/SHORTLISTED per newer state-integrity spec; confirmed match requires admin cancellation. | product requirements document md; architecture/assignment-be-spec md,608-620; architecture/int-077-marketplace-quota-caps md; architecture/int-093-batch-state-machine-integrity-spec md |
| Invitations and owner progress | Parent may invite on PENDING or ACTIVE request. Up to 10 pending invitations per job; withdraw frees slot. Tutor accept creates/reactivates application and may create a selection request; it must not overwrite another tutor already deliberately chosen by the parent. Decline does not withdraw an existing application. Durable sent invitations and interview requests appear in the parent's dashboard, with own-only/history status filtering. | 021-*/spec md; architecture/int-044-job-invitations md; architecture/int-077-marketplace-quota-caps md; architecture/int-093-batch-state-machine-integrity-spec md; 013-*/requirements md |

## Strongest findings and decisions

### 1. Missing edge-case contract: what can a tutor do with an invitation while the job is awaiting approval?

Spec 021 explicitly permits sending invitations for a PENDING request and only states that contact remains gated and rejection voids/closes them (021-tutee-request-flow/spec md). The inherited invitation contract still requires ACTIVE at acceptance and acceptance folds into an ACTIVE-only application (architecture/int-044-job-invitations md; architecture/assignment-be-spec md). Current Job Invitation Service implementation reference still enforces ACTIVE.

This is not proof that acceptance must already work on PENDING. Initiation was explicitly widened, but the response-stage interaction was never specified. A legitimate tutor may receive an actionable-looking invite, press Accept, and be told the job is not active. Approval notification/retry expectations are unstated.

Decision question: Should an invited tutor accept provisionally before approval, or wait for approval?

**Proposal:** Keep application/selection creation gated on approval; render a truthful “Awaiting request approval” invitation state and notify the tutor when actionable. If conversion demands immediate response, define a separate provisional acceptance state and explicit approval/rejection transitions before widening application eligibility.

### 2. Missing edge-case contract: material job edits after people already expressed interest

Owners can edit ACTIVE and PENDING requests, including subject, budget, schedule and venue, and reenter approval (013-tutee-dashboard-and-invitations/requirements md; 021-tutee-request-flow/spec md). Applications and invitations refer to that existing job; acceptance is an expression of interest for its terms (product requirements document md; architecture/int-044-job-invitations md). Current Tutee Post Service implementation reference replaces substantive fields and status without specifying a version/reconfirmation transition for existing applicants or scheduled discussions.

The gap is the promise to an already-interested tutor when a Maths request becomes Chemistry or budget halves. It is not an assertion that every edit must destroy interest. The approval cycle alone does not define whether earlier interest remains current.

Decision question: Which changes require existing applicants/invitees to reconfirm, and what happens to pending interviews/selection?

**Proposal:** Define material fields (subject, level, venue/area, budget, duration/frequency/schedule), retain history, notify interested tutors after reapproval, and require reconfirmation before assignment for materially changed terms. Cosmetic notes need not invalidate interest. Coordinator final agreed terms remain authoritative for the match.

### 3. Genuine specification conflict/unclear authority: listing-to-profile prefill versus no synchronization

product requirements document explicitly chooses “profile-canonical + listing-overrides-when-attached (no sync job)” (product requirements document md); applicant-card sourcing repeats it (architecture/assignment-be-spec md). Later 002 promises profile fields are prefilled from onboarding **or any prior profile/listing save**, so known values are not re-entered (002-tutor-profile-and-onboarding-ux/requirements md). Only one-time legacy backfill is defined in the earlier model. Multiple listings may legitimately carry different rate/area values. 022 narrows its explicit listing-prefill requirement to qualification/experience (022-tutor-profile-and-fe-polish/spec md). There is no rule choosing which listing wins on a new profile, or whether a later listing save changes profile defaults.

Decision question: Is profile the permanent default with independent listing overrides, or should listing edits teach/update profile defaults?

**Proposal:** Preserve profile as authoritative; allow an explicit first-use “Use these values as my profile defaults” action (or seed missing profile fields once). Never silently overwrite a populated profile from a listing. Rewrite 002's broad “any listing save” promise accordingly; prefill listing inputs from profile without making subsequent edits bidirectional.

### 4. Requirements integration gap: exact profile pairs versus independent listing subject/level sets

INT-153 says listings imply a cross-product and coverage requires EVERY subject×level pair to exist in profile (architecture/int-153-tutor-subject-level-matrix-spec md), warning rather than blocking on mismatch. Spec 025's later listing rule accepts a subject/level set whenever each subject and each level has at least one valid partner (025-taxonomy-stream-year-scoping/requirements md). That is an explicit refinement of validation, but the listing's meaning/display is not rewritten: mixed-band or mixed-year sets need not mean every pair is offered. The same legacy product can therefore be accepted while its implied cross-product contains invalid pairs or exceeds profile capability.

Decision question: Does a listing advertise every combination or a set of explicit pairs?

**Proposal:** Use explicit subject-level pairs on listings, or derive/display only valid intersections and declare that semantic contract. Keep the decided broad Apply eligibility separate; do not infer a new per-job Apply hard gate. A good acceptance case is a tutor offering one subject at Primary and a different subject at JC with no cross-band implication.

### 5. Deliberate scope limitation needing a product decision: parent schedule precision

002 explicitly fixes the defect where a tutor free 7–9pm otherwise claims all 6–10pm (002-*/requirements md). Parent request 009 explicitly requires hourly grid controls yet forbids changing build Bitmask/slots From Bitmask (009-*/requirements md); 021 then requires details to show the schedule the parent entered (021-*/spec md). Current Create Request implementation reference saves selected hours as broad periods and reloads every hour in that period.

This is not a missed implementation of 002 for parents: 002 was tutor-specific, and 009 deliberately retained the parent encoding. The unresolved product question is whether parent's exact hour choices are promises or coarse preferences. Choosing Tue 19–20 currently becomes Tue Evening (18–22).

Decision question: Should parent availability be exact hours or explicit coarse preferences?

**Proposal:** Persist parent hourly slots end to end, matching the tutor model. Until that is scoped, make the parent's period-level meaning visible and avoid implying that one selected hour will survive a round trip.

### 6. Real authority/status contradiction: taxonomy approval is both shipped and prohibited pending approval

025-taxonomy-stream-year-scoping/spec md says HISTORICALLY MARKED IMPLEMENTED, but 26-30 says DRAFT, nothing built, mapping not approved; 82-91 says not to auto-default and requires founder decisions on exact mapping/model/block-vs-warn; requirements md still labels the canonical mapping proposed. Also 020's changelog says its canonical additions were DEFERRED (020-*/spec md) while 025's out-of-scope text says 020 already added the set (025-*/spec md). This prevents a future maintainer from knowing which curriculum mapping is approved business policy versus merely today's code.

Decision question: Which canonical mapping and hard-block policy received approval, and what are the effective version/date and legacy-data exceptions?

**Proposal:** Publish one approved taxonomy decision record, link its signoff and effective version, mark old bands superseded, and reconcile status/changelog/task boxes. Do not automatically infer founder approval from HISTORICALLY MARKED IMPLEMENTED or from implemented code. This is documentation authority work, not a claim about current MOE curriculum correctness.

### 7. Clear requirement/implementation gap: draft saving already is a P0 requirement

product requirements document md calls save-as-draft P0. modules/posting/spec md defines server autosave, saved/failed indicator, one draft per role and continue/discard. Later architecture/int-052-ux-typing-minimisation md repeats autosave as a user experience requirement; no explicit removal was found. Current wizard state is component-local (Create Request implementation reference; Create Listing implementation reference); backend Tutee Post Service implementation reference immediately creates PENDING on final submit. Create Request implementation reference links away to the rate guide during entry, making lost work reachable via normal product guidance.

Classification: clear unfulfilled documented behavior, not a new feature suggestion. 026's internal Back-preserves-fields (026-*/spec md) does not supersede persistence across reload/navigation.

Decision question only if scope is being changed: Will the P0 draft commitment ship, or be explicitly deferred?

**Proposal:** Deliver resumable server drafts with sparse validation and clear saved/unsaved feedback; retain one draft per role unless the product deliberately revises that limit. Reconsider technical unload mechanism separately; the outcome requirement is preserving work, not relying on an unreliable synchronous send Beacon interpretation.

### 8. Explicit newer-rule supersession, then frontend defect: SHORTLISTED applications must be withdrawable

Older assignment frontend binding condition says APPLIED only (architecture/assignment-fe-spec md). Newer INT-102 allows APPLIED and SHORTLISTED and explicitly accepts that behavior (architecture/int-093-batch-state-machine-integrity-spec md); quota policy counts both as active and promises withdrawal frees slots (architecture/int-077-marketplace-quota-caps md). Current backend honors both (Job Application Service implementation reference), but Applications implementation reference still cites the old condition and hides withdrawal for SHORTLISTED.

Classification: the temporal requirement change is settled; this is frontend + stale-spec drift, not a founder question. Do not ask whether users should be allowed to withdraw again without a new reason to revisit the decision.

**Proposal:** Expose withdrawal for both open statuses and mark the old frontend condition superseded. A withdrawn application with an outstanding selection/interview needs its own downstream-state handling under the cross-flow lifecycle audit.

### 9. Clear latest-user requirement/implementation gap: anonymous filtering

Current user explicitly says anonymous users should filter with limited search. Earlier product requirements document public filtering (product requirements document md) and public assignment browse (architecture/m17-marketplace-ux-overhaul md,194-197) support that baseline. Current Browse Tutors implementation reference and Browse Assignments implementation reference render filters only when user exists; their pagers have the same restriction at  and  respectively.

Classification: not an unresolved public-vs-private business choice. Adopt latest user rule; retain the decided detail login gate. Concrete assignment caps are already documented. Do not conflate all-auth filtering with anti-enumeration.

**Proposal:** Show public filters with bounded results and a clear sign-in path for deeper access. Record a single per-audience/per-endpoint cap table; tutor teaser limits and assignment's historical 5×20 cap should be reconciled rather than assumed identical. Define whether ceiling applies per query or visitor if anti-enumeration is the aim; a per-query result cap alone does not prevent enumerating many different filters.

### 10. Genuine requirement/copy conflict: post approval is being described as contact-release consent

Spec 021 states contact is gated “until the post is approved” and directs frontend copy “contact is shared once approved” (021-tutee-request-flow/spec md). The binding product requirements document M18 privacy contract gates counterparty identity until **assignment** (product requirements document md), while the flow defines assignment only after both parties confirm and admin acts (product requirements document md). These are different events: admin approves a request before any tutor has even been chosen. “Not until approval” alone can mean a necessary condition; the prescribed “once approved” copy wrongly reads as a sufficient condition, and no explicit privacy-model reversal accompanies 021.

Current Contact Reveal Service implementation reference still authorizes by participation in a post-assignment match (PENDING, ACTIVE or COMPLETED), not a request's publication approval. This is a specification/copy conflict, not evidence of an actual premature contact leak. The premature promise appears literally in current Invite Tutor Modal implementation reference and Tutor Detail implementation reference. Separately, tutor interview copy still says contact is “never shared directly” (Tutor Detail implementation reference), another inconsistent promise against the release feature.

Decision question: Does publication approval only unlock the next marketplace actions, with contact release still requiring confirmed assignment?

**Proposal:** Keep assignment as the contact-release condition. Rewrite 021: “You can invite while your request is being reviewed. Contact details are shared only after both parties confirm and our team assigns the tutor.” Add an actor×state contact-visibility table distinguishing post PENDING from match PENDING. Mark 021's existing phrase erroneous, not a silent founder change to the privacy contract.

## Explicit supersessions and settled decisions not to relitigate

- Public lists / gated details: M17 replaces early module's tutor-only request discovery and public tutor-detail rules (m17-*,77-87). Current user's public filtering rule supersedes any later auth-only-filter user interface decision.
- Published-listing prerequisite: old applications module says required (modules/applications/spec md); product requirements document M18 removes it (product requirements document md). No active listing is required to apply.
- Registration: 002 partially reverses INT-043 by adding OPTIONAL capture, explicitly preserving no forced completion (002-*/spec md,92-98). Registration capture is not an apply eligibility gate.
- Subject matrix does not gate each job: INT-153 v3 explicitly builds nonempty matrix + no per-post Apply gating (architecture/int-153-*), settling its earlier “build blocked” paragraph.
- Parent PENDING edits and invitations: 021's founder decision supersedes 013's earlier PENDING View-only (013-*/requirements md; 021-*/spec md).
- Invitation acceptance never overwrites a different chosen tutor: INT-106 decides create-if-absent + parent notification; no need to reopen INT-044's earlier auto-overwrite proposal (architecture/int-093-*).
- Distinct revision-request moderation: INT-098 explicitly retires unreachable REVISION REQUESTED display and uses rejection/resubmission; enum retained (architecture/int-098-batch-marketplace-funnel-ux-spec md). Do not call absence of a revision route an undiscovered defect against the early product requirements document.
- Quota acceptance bypass: explicit and intentional (architecture/int-077-*), not an over-cap bug. Remaining is clamped  is at least 0; limit is a direct-apply cap, not an absolute row count invariant.
- 025 is an explicit partial successor to 020's coarse-band taxonomy (025-*/spec md,12-24); the unresolved issue is approval evidence/status and interaction semantics, not whether later requirements are allowed to refine earlier ones.

## Consolidation advice

Lead with cross-flow decision seams (#1–#4), then classify known requirement gaps (#7–#10). Avoid presenting every old product requirements document sentence as equally current: SCOPE is dated 2026-05-29 and accurately described that moment; later delivered features do not contradict that historical snapshot. The most valuable artifact is a dated, authoritative actor × state × action table and field ownership table, cross-linked to explicit supersession records.

## Supplemental location observation (lower priority than top ten)

### Field-requirement drift: request location and online-only supply

Posting contract requires parent MRT/planning area (modules/posting/spec md), and tutor location is conditionally required for in-person venues (88-90). Later 002 explicitly makes tutor listing MRT selection optional with zero valid (002-*/requirements md), while current Apply policy unconditionally requires planning areas (012-*/requirements md; Tutor Profile Completeness implementation reference). Parent creation currently permits missing location (Create Tutee Post Request implementation reference; Tutee Post Service implementation reference), and the frontend payload omits location entirely (Create Request implementation reference). No explicit parent-location requirement reversal was found.

Split the classification: parent location is a documented required-field/implementation mismatch unless a later approved removal is located; tutor optional listing MRT is an explicit later rule, while its relationship to required profile service areas is an unresolved product policy for online-only tutors, not necessarily a code bug.

Decision question: Is location a student's session venue, a tutor's service area, or simply mandatory identity data; what does online-only mean for each?

**Proposal:** Collect a coarse per-request service location when any in-person venue is selected; allow online-only requests and tutors without fake local service areas, with an explicit online capability flag. Preserve precise address privacy; avoid asking users to pick an arbitrary district just to pass completeness. If geography is mandatory for all accounts by policy, state why and separate it from service-area search.
