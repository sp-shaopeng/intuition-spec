# InTuition requirements and use-case audit

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](corrections.md) and [current requirements](../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

**Date:** 2026-09-05  
**Status:** Diagnostic review; proposed policies below are not approved requirements.  
**Scope:** Current workspace specifications and selected implementation paths, covering visitor, parent/adult learner, tutor, admin and super-admin journeys. No application code or existing requirements were changed.

## Findings at a glance

The product has substantial requirements coverage, but there is no reliable single current baseline. The largest risks are connections between features: approval versus assignment, interest versus acceptance of changed terms, agency payment versus tuition completion, and database state versus actual message delivery.

The [source catalog](catalog.md) indexes **27 numbered specs and 170 distinct namespaced requirement entries**, plus **112 older product requirements document feature rows, 19 original acceptance-criteria headings and 15 business requirements document rules**. Those groups overlap; their sum is not a count of unique current requirements. The catalog also links the module, architecture and 43-flow sequence indexes. It is an index of documented commitments, not proof of implementation or deployment.

Three specialist reviews support this synthesis: [marketplace](marketplace-notes.md), [lifecycle and money](lifecycle-notes.md), and [identity, privacy and operations](identity-operations-notes.md). They contain additional scenarios, precise source references and the reasoning behind each classification.

### Confirmed during this conversation

| ID | Confirmed requirement | Consequence |
|---|---|---|
| C-01 | Anonymous visitors may filter a limited public result set. | Hiding all filters behind login is an implementation mismatch. Do not reopen whether public filtering is allowed. Clarify the exact limit table, rather than invent a search-count quota. |
| C-02 | “Mark commission paid” means the agency received the tutor's commission. | Parent-to-tutor lesson payment is a separate event and cannot itself count as agency revenue or settle commission. |

### How conflicting sources were assessed

The current user's decisions control. Explicit later amendments supersede the affected older clauses, not the entire old product model. Original requirements remain useful where no later change was found. Code establishes current behavior; it does not, by itself, approve a product-policy change. A HISTORICALLY MARKED IMPLEMENTED label is not deployment evidence.

Examples of source drift: [SCOPE](../../SCOPE.md) is dated May 29 and still describes email, applications and refresh tokens as missing; it should be read as a historical snapshot. [requirements traceability matrix](../../RTM.md) lists only specs 001–010 even though 027 exists. Spec 025 says HISTORICALLY MARKED IMPLEMENTED in its frontmatter but [DRAFT and awaiting approval in its body](../../025-taxonomy-stream-year-scoping/spec.md). This is a traceability/approval-record gap, not evidence that approval never occurred.

## 1. Consolidated requirement map

“Baseline” means the best-supported documented behavior after explicit amendments. An entry here does not mean its implementation passed tests.

| Area / actors | Requirement baseline | Important boundary | Main sources |
|---|---|---|---|
| Public discovery / visitor | Public tutor and assignment lists, limited public filters, public rates/content; detail requires login and preserves return intent. | Result limits and contact privacy are separate from authentication to filter. | C-01; [M17](../../architecture/m17-marketplace-ux-overhaul.md) |
| Registration / tutor, tutee | Roles are mutually exclusive; required consent flags and policy versions are recorded; tutor explicitly accepts commission terms. | Optional onboarding details may be omitted; account creation is not profile completeness. | [product requirements document](../../PRD.md); [001](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md); [002](../../002-tutor-profile-and-onboarding-ux/spec.md) |
| Tutor profile / tutor | Canonical identity, public handle, capability pairs, rate, service areas, qualification and commitment; shared completeness result controls applying. | A complete profile is not automatically proof of vetting. A listing is optional. | [product requirements document M18](../../PRD.md); [012](../../012-tutor-apply-and-profile-ux/requirements.md) |
| Parent request / tutee | One subject/level; student count, budget, venue, duration, frequency and schedule; explicit review then submit. Owner can edit PENDING/ACTIVE under reapproval rules. | Edits may invalidate another person's previous interest; MATCHED is not ordinary editable inventory. | [009](../../009-tutee-post-taxonomy-and-ux/requirements.md); [013](../../013-tutee-dashboard-and-invitations/requirements.md); [021](../../021-tutee-request-flow/spec.md) |
| Tutor listing / tutor | Optional multi-subject/level showcase, rates, venue/group sizes, qualifications, exact hourly slots, multiple optional MRT stations. | Listing-specific facts and profile defaults can differ. | [002](../../002-tutor-profile-and-onboarding-ux/requirements.md); [posting](../../modules/posting/spec.md) |
| Draft continuity / both roles | Save drafts, show saved/failed feedback, resume work; internal wizard Back preserves values. | Back navigation inside a wizard is not persistence across reload or leaving for the rate guide. | [posting autosave](../../modules/posting/spec.md); [026](../../026-s3s4-polish-batch/spec.md) |
| Taxonomy / all creators | Shared the private implementation folder subject-level validity; precise Secondary stream/year mappings; validation on new writes, preserving legacy records. | Validating a set is not the same as declaring every subject-level combination within it. | [020](../../020-taxonomy-integrity/spec.md); [025](../../025-taxonomy-stream-year-scoping/requirements.md) |
| Moderation / admin | Review posts, approve or reject, communicate reason, audit contact access; active edits reenter review. | Publication approval does not itself assign a tutor or authorize contact release. | [approval](../../modules/approval/spec.md); [021](../../021-tutee-request-flow/spec.md) |
| Applications / tutor | Profile-complete tutors may apply to ACTIVE jobs without a published listing. Idempotent apply, status-aware reapply; APPLIED and SHORTLISTED may be withdrawn. | Direct-apply quota counts both open statuses; invitation acceptance deliberately bypasses that quota. | [M18](../../PRD.md); [quotas](../../architecture/int-077-marketplace-quota-caps.md); [withdrawal](../../architecture/int-093-batch-state-machine-integrity-spec.md) |
| Favourites / tutee | Save/remove tutors, consistent heart state, own favourites page. | A favourite is not an application, invitation, interview or selection commitment. | [favourite flow](../../sequences/22-favourite-toggle.md) |
| Invitations / parent and tutor | Parent may invite on PENDING or ACTIVE requests, withdraw and inspect sent invitations. Accept folds into the application/selection funnel; cannot overwrite a different chosen tutor. | Sending before approval was added; the recipient's acceptance/waiting journey is incomplete. | [021](../../021-tutee-request-flow/spec.md); [invitations](../../architecture/int-044-job-invitations.md); [selection protection](../../architecture/int-093-batch-state-machine-integrity-spec.md) |
| Parent selection / tutee | Choose a tutor or request an interview; selection is a preference for admin, not a match. | Final assignment requires admin action and confirmation of both parties. | [product requirements document flow](../../PRD.md) |
| Interview negotiation / both roles | Proposed slots, alternating counterproposals, accept one, decline with reason; up to three concurrent live interviews per posting. | Accepted time and admin scheduling are different states. Unanswered negotiation expires after 72 hours. | [negotiation](../../architecture/int-039-interview-negotiation.md); [cap](../../modules/interview/max-interviews-per-posting.md) |
| Interview scheduling / admin | Agreed slot prefilled but editable; admin schedules, supplies/updates meeting link, records outcome; Singapore time and stated 15-minute duration. | Override authority exists; re-consent and confirmed appointment collisions are not fully specified. | [014](../../014-interview-scheduling-ux-and-admin-visibility/requirements.md); [negotiation](../../architecture/int-039-interview-negotiation.md) |
| Matching / admin | One PENDING/ACTIVE match per parent request; tutor may have multiple matches. Atomic assign marks winner MATCHED and other open applications SUPERSEDED. | Both admin entry points must use the same transition. Other interviews need a defined fate. | [product requirements document assignment](../../PRD.md) |
| Contact reveal / participants | Contact is available only to the two participants after assignment, including a PENDING match; denied to unrelated users and after cancellation. | PENDING match means assigned; PENDING post means awaiting publication approval. | [personal information contract](../../PRD.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |
| Commission / tutor and admin | Tutor owes agency a one-time commission, normally 50% of a locked agreed first-month plan; manual off-platform collection. | Amount lock, first-session due trigger, invoice/receivable and receipt are separate concepts. | C-02; [commission](../../modules/commission/spec.md); [locked plan](../../architecture/int-157-commission-first-month-locked-spec.md) |
| First-session confirmation / tutee and admin | Tutee confirmation or a permitted admin override before commission invoicing; reminder/escalation for nonresponse. | Later M18 text explicitly preserves this requirement; absence from the simplified implementation does not remove it. | [AC-05](../../PRD.md); [M18 money path](../../PRD.md); [lock does not change trigger](../../architecture/int-157-commission-first-month-locked-spec.md) |
| Cancellation / admin | PENDING/ACTIVE matches cancel; default reopens demand and applications and cancels pending commission. COMPLETED is terminal in the current model. | The default does not answer all post-lesson debt, paid replacement or no-longer-seeking cases. | [cancel baseline](../../architecture/match-lifecycle-fixes-spec.md) |
| Reviews / participants | One review per participant per COMPLETED match, rating aggregation and public tutor reviews. | Payment-controlled completion can exclude real teaching experiences and open reviews before tuition ends. | [application programming interface](../../reference/backend-api.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |
| Dashboards and analytics / all roles | Role-specific request/listing/application/interview/match/history views; admin queues, date-range metrics and defined conversion denominators. | Source document status and point-in-time counts must not be presented as verified deployment or cohort conversion. | [routes](../../reference/frontend-routes.md); [017](../../017-admin-analytics-and-financial-safety/spec.md); [019](../../019-admin-money-safety-and-analytics/spec.md) |
| Auth and recovery / all users | Password reset, role/session checks, tutor/tutee 30-minute access with rotating refresh, separate stateful admin sessions, immediate block/reset/logout revocation. | Logout during an outage and loss of trusted recovery channels need explicit outcomes. | [003](../../003-password-reset-failure-fix/requirements.md); [004](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md); [011](../../011-auth-robustness/requirements.md) |
| Contact changes / all users | Step-up initiation, old value retained, single-use 30-minute link, explicit confirmation POST, uniqueness checked at commit; new-email verification for email edits, account-email confirmation for phone edits. | Existing verified email and independent old-phone warning are assumed without a complete acquisition/recovery/channel contract. | [012 contact contract](../../012-tutor-apply-and-profile-ux/requirements.md) |
| Admin control / admin, super-admin | Any admin creates ordinary admins; super-admin alone removes ordinary admins; super-admin protected; forced password change, audit and revocation; production multi-factor authentication. | Bootstrap and protection rules do not define compromised-owner or lost-multi-factor authentication recovery. | [admin management](../../modules/admin-management/spec.md); [027 bootstrap](../../027-aws-production-deployment/spec.md) |
| Consent and data lifecycle / users and operators | Required consent/version audit; optional marketing; published access/correction/withdrawal route; soft-delete convention; retention/anonymisation obligations. | Legacy re-consent, scoped withdrawal, responsible adult and conflicting retention deadlines need a current operational contract. | [001](../../001-legal-and-compliance-consent-and-policy-pages/spec.md); [retention](../../BRD.md) |
| Notifications / recipients and operators | In-app inbox; transactional messaging outbox, retries/dedup/failed-message queue; gated WhatsApp schedule confirmations/reminders, both parties when eligible. | WhatsApp OFF is intentional. Queued, provider-accepted and delivered are different milestones. | [inbox](../../sequences/21-notification-bell-flow.md); [006](../../006-whatsapp-interview-reminders/requirements.md); [messaging](../../architecture/transactional-messaging-and-idempotency.md) |
| Platform quality and operations | Responsive 390px flows, accessible controls, actionable errors/trace IDs, latency/SLO targets, migration safety, secure production bootstrap, measured backup/restore. | Workload, business-hours calendar and restored business correctness need release acceptance, not just infrastructure health. | [015](../../015-mobile-responsive-fixes/requirements.md); [016](../../016-error-copy-and-handling/requirements.md); [non-functional requirement](../../PRD.md); [027](../../027-aws-production-deployment/spec.md) |

## 2. Decisions with the greatest business impact

The following are proposed priorities for resolving requirements. They are not instructions to implement speculative policy. Each row needs an owner-approved answer and testable acceptance criteria.

### D-01 — What happens financially when tuition is cancelled?

**Scenario:** Three lessons occurred, but the tutor has not yet paid the agency. Cancelling currently cancels the pending commission. If the tutor paid yesterday, the match is terminal instead. The same teaching outcome receives different treatment based on collection timing.

**Decide:** Before the first qualifying lesson, after lessons start, and after commission receipt: what is owed, waived, credited/refunded, and charged for a replacement? Does a free or unsuccessful trial count as a qualifying session?

**Proposed direction:** Separate cancellation of tuition from cancellation of agency debt. Preserve financial history; require a recorded waiver/credit decision where applicable. Do not promise a refund/free replacement until its eligibility and limits are agreed. [Old invoiced-cancellation rule](../../PRD.md) versus [later cancel policy](../../architecture/match-lifecycle-fixes-spec.md). Detail: lifecycle L1.

### D-02 — How are honest money-entry errors and partial payments handled?

**Scenario:** An admin enters nine sessions instead of eight, marks the wrong tutor paid, or receives S$100 against S$270 due. A confirmation dialog cannot cover correction, installments, payment references or late data entry.

**Decide:** Who can adjust/reverse a record, which evidence and approval are required, and whether actual receipt date may differ from entry date.

**Proposed direction:** Audited adjustments linked to immutable originals, with actor, reason, receipt reference/date and outstanding balance. Keep transcription correction distinct from renegotiating the agreed plan. [Locked snapshot](../../architecture/int-157-commission-first-month-locked-spec.md); [irreversible mark-paid wording](../../019-admin-money-safety-and-analytics/spec.md). Detail: lifecycle L2.

### D-03 — What does “first month” mean?

**Scenario:** S$60/hour × 1.5 hours × twice weekly produces S$360 commission with eight sessions, or S$405 with the current nine-session estimate.

**Decide:** Explicit agreed lesson count, four weeks, actual calendar-month sessions, or rounded frequency × 4.33. This is explicitly still marked a founder decision, not merely an old resolved question.

**Proposed direction:** Make the mutually agreed session count authoritative and label any prefill as an estimate. [product requirements document four weeks](../../PRD.md); [INT-157 unresolved 4.33 choice](../../architecture/int-157-commission-first-month-locked-spec.md). Detail: lifecycle L3.

### D-04 — What exact plan have both parties accepted?

**Scenario:** Parent advertised S$40–50, tutor advertised S$60–80, coordinator negotiates S$55, but activation records S$65. General signup consent does not prove agreement to this particular amount.

**Decide:** Out-of-range negotiated rates, evidence of both parties' acceptance of the exact plan, and override authority. Hard-clamping is explicitly unresolved. Activation is already the documented amount-lock event; align “locked at match” shorthand with that definition.

**Proposed direction:** Permit negotiation with a warning/reason and capture acceptance of the exact rate × sessions × hours before activation. Preserve the documented activation lock. [Open rate policy](../../architecture/int-157-commission-first-month-locked-spec.md); [activation lock](../../architecture/int-157-commission-first-month-locked-spec.md); [both confirm](../../PRD.md). Detail: lifecycle L4.

### D-05 — How are actual tuition end, reviews and replacement tracked after placement completion?

**Scenario:** Tutor pays the agency after week one but teaches for another year. The payment currently completes the match and stamps its end date. Conversely, a real teaching experience may be ineligible for review while commission remains unpaid or the arrangement is cancelled.

**Established:** M18 explicitly defines COMPLETED as commission PAID. That enum meaning is settled. **Decide:** How to record actual tuition end, when reviews are appropriate and how replacement works after this financial milestone.

**Proposed direction:** Preserve COMPLETED as placement completion and track actual teaching start/end separately. Consider review eligibility based on a defined experience milestone, with rules for cancellation, edits and moderation; this would amend the current COMPLETED-only review rule. [Authoritative completion definition](../../PRD.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md). Detail: lifecycle L5.

### D-06 — What happens when request terms change after tutors respond?

**Scenario:** A tutor applies for Maths at S$60/hour; the parent changes the same request to Chemistry at S$35/hour, another venue or a different schedule. Reapproval alone does not establish the tutor's agreement to the new terms.

**Decide:** Which changes are material; what happens to applications, invitations, selections and interviews; who gets notified and must reconfirm.

**Proposed direction:** Version the relevant terms, retain history, notify after reapproval and require renewed acceptance before assignment for material changes. Cosmetic changes need not erase interest. [Request editing](../../013-tutee-dashboard-and-invitations/requirements.md); [PENDING edits](../../021-tutee-request-flow/spec.md). Detail: marketplace #2.

### D-07 — Can a tutor accept an invitation before the request is approved?

**Scenario:** Parent can send an invitation while a request is PENDING, but acceptance still folds into an ACTIVE-only application. The tutor sees an invitation before it is actionable.

**Decide:** Wait-for-approval versus provisional acceptance, including the visible status, approval notification and how the already-required rejection closure is presented and communicated.

**Proposed direction:** Preserve approval before application/selection creation; show “Awaiting request approval” and notify when actionable. A provisional acceptance alternative needs a distinct state, not an accidental widening of all public applications. [PENDING initiation](../../021-tutee-request-flow/spec.md); [acceptance rules](../../architecture/int-044-job-invitations.md). Detail: marketplace #1.

### D-08 — What does “vetted tutor” guarantee, including a tutor with no listing?

**Scenario:** A tutor completes self-entered profile fields and applies without a listing, which is allowed. A parent sees “every tutor vetted.” Post-content review, identity checking and qualification verification are different claims.

**Decide:** Which checks are actually promised, when they must occur (before applicant exposure, interview or assignment), who records the result, and when it expires or must be repeated after profile edits.

**Proposed direction:** Define a small explicit checklist and evidence record for the promised checks. Use precise labels such as “listing reviewed” unless stronger verification has occurred. Preserve open expression of interest; do not silently reintroduce the retired listing gate. [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); [listing-free model](../../PRD.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md). This is a gap in the recorded product assurance, not proof that the team performs no manual checks.

## 3. Remaining unclear requirements and important edge cases

| ID | Scenario / gap | Decision and proposed direction | Evidence / detailed notes |
|---|---|---|---|
| D-09 | Parent cancels because tuition is no longer needed; automatic cancellation relists the request and restores the former tutor. Old applicants may no longer be available. | Keep reopening as the established replacement default, but define “find another tutor” versus “stop searching,” excluded former tutors, renewed interest and quota behavior. | [Reopen rule](../../PRD.md); lifecycle L6 |
| D-10 | Interviews with A and B remain scheduled after A is assigned, or while the parent closes/withdraws demand. | Define job×interview transitions, cancellation reasons, recipient notices and reminder cancellation. Preserve past outcomes; do not automatically revive appointments when reopening. Rejection cleanup is already required and is not a new decision. | [Assignment table](../../PRD.md); [rejection cleanup](../../021-tutee-request-flow/spec.md); lifecycle L7 |
| D-11 | Tomorrow's offered time passes before the 72-hour inactivity timeout; an agreed interview waits indefinitely for admin. | Define slot freshness, minimum lead time, expired-last-slot behavior and agreed-time handoff deadline/escalation. Keep 72 hours as an additional inactivity backstop. | [Offer/accept/expiry](../../architecture/int-039-interview-negotiation.md); lifecycle L8 |
| D-12 | Admin changes a mutually agreed appointment or books a tutor/parent into two confirmed interviews. | Admin override is already allowed. Specify evidence/reason, old/new-time notice, re-acknowledgment and cross-job collision handling for confirmed appointments. | [Override authority](../../architecture/int-039-interview-negotiation.md); [editable prefill](../../014-interview-scheduling-ux-and-admin-visibility/requirements.md); lifecycle L9 |
| D-13 | Different listings carry different rates/areas; later requirements promise profile prefill from prior listing saves despite the no-sync model. | Choose a field ownership/prefill policy. Proposal: profile defaults remain authoritative, listing overrides independent, explicit first-use copy rather than silent bidirectional sync. | [No-sync baseline](../../PRD.md); [broader prefill promise](../../002-tutor-profile-and-onboarding-ux/requirements.md); marketplace #3 |
| D-14 | A listing has two subjects and two levels, but only two of four possible pairs are valid/offered. | Define explicit pairs or valid intersections rather than an ambiguous cross-product. Preserve the separately settled broad Apply eligibility. | [Cross-product meaning](../../architecture/int-153-tutor-subject-level-matrix-spec.md); [new set validation](../../025-taxonomy-stream-year-scoping/requirements.md); marketplace #4 |
| D-15 | Parent selects Tue 19–20; saving/reopening expands this to Tue 18–22. | Decide exact hours versus explicitly coarse preferences. Parent bitmask preservation was deliberate, so exact persistence needs a scoped change; until then the user interface should state its actual precision. | [Parent encoding constraint](../../009-tutee-post-taxonomy-and-ux/requirements.md); [show entered schedule](../../021-tutee-request-flow/spec.md); marketplace #5 |
| D-16 | In-person request can lack a service location; online-only tutor must supply a planning area to pass profile completeness. | Define geography separately for venue, searchable service area and identity. Proposal: coarse location for in-person requests and explicit online capability; do not make users invent an area. Any change to mandatory profile areas needs an explicit policy amendment. | [Posting fields](../../modules/posting/spec.md); [completeness](../../012-tutor-apply-and-profile-ux/requirements.md); marketplace location supplement |
| D-17 | New copy says contact is shared “once approved”; other copy says “never”; backend requires a confirmed assignment. | Post approval is not assignment. Proposal: retain the post-assignment participant-only rule and correct 021's promise, with an actor×state visibility table. No premature leak was demonstrated. | [021 wording](../../021-tutee-request-flow/spec.md); [binding release contract](../../PRD.md); marketplace #10 |
| D-18 | Minor learner or minor tutor ticks the same “18 or verified guardian consent” box and accepts a commitment. | Define self-attestation versus actual guardian verification and the responsible adult/contact for each role. Do not add identity-document collection without a specific need. | [Consent wording](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); identity I-01 |
| D-19 | Legacy user has no recorded policy consent; another user declines new terms or withdraws only marketing consent. | Re-consent is already a prelaunch follow-up. Specify accept/decline/defer, action restrictions, version rollout and support access. Separate marketing/channel withdrawal from account deletion and define manual export/correction case completion. | [Explicit residual](../../001-legal-and-compliance-consent-and-policy-pages/spec.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); identity I-02/I-03 |
| D-20 | Deleted user's personal information is due for anonymisation after 30 days in one spec but retained two years in another; newer outbox/contact-change records are omitted. | Publish a current per-data-class retention/minimisation matrix with exceptions, backup/export handling and verification evidence. Preserve the settled soft-delete convention; do not silently choose a retention period in code. | [Auth 30 days](../../modules/auth/spec.md); [business requirements document matrix](../../BRD.md); identity I-04 |
| D-21 | Phone change assumes an already-verified email, but a new/legacy user never verified it or lost the mailbox. | Keep the selected email-link mechanism; define proof of mailbox ownership, unverified/typo branches, lost-mailbox support recovery and concurrent email/phone changes. Separately resolve the historical onboarding choice of verification before activation versus immediate access; it does not cover these contact-change recovery branches. | [Contact prerequisite](../../012-tutor-apply-and-profile-ux/requirements.md); [historical open verification choice](../../modules/auth/spec.md); identity I-05 |
| D-22 | Requirement promises old email AND old phone warnings with revoke, while WhatsApp is OFF and no other phone channel is defined. | Dual-channel warning is the current requirement. Specify the phone channel and cancel capability/race outcomes, or explicitly amend scope for email-only while phone messaging is OFF. Email containing an old phone number is not notice delivered to that phone. No scope downgrade is approved by this audit. | [Dual-channel requirement](../../012-tutor-apply-and-profile-ux/requirements.md); identity I-06 |
| D-23 | Logout happens offline; page state clears but HttpOnly cookies cannot be cleared by JS. Reload/another tab may restore access later. | Preserve HttpOnly. Define local sign-out versus confirmed server revocation, cross-tab/reload behavior and reconnect retry. The current frontend-cookie instruction is internally impossible as written. | [Contradictory instruction](../../011-auth-robustness/design.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); identity I-07 |
| D-24 | Sole protected super-admin loses multi-factor authentication, leaves the business or is compromised. | Define separately authorized emergency recovery/containment, approvers, identity check, audit, session revocation and multi-factor authentication reenrollment. Bootstrap alone does not cover recovery. | [Protected account](../../modules/admin-management/spec.md); [prod bootstrap](../../027-aws-production-deployment/spec.md); identity I-08 |
| D-25 | Outbox says SENT but message is only in Redis; queue recovery could send expired one-time passcode or old meeting details; one recipient could fail independently. | Define queued/provider-accepted/delivered/failed/expired stages, event-specific deadlines, expiry/supersession and operator retry. Reconcile five-minute interview notice target versus “all other” 30 minutes. | [Interview service-level agreement](../../PRD.md); [general service-level agreement](../../PRD.md); [outbox milestone](../../architecture/transactional-messaging-and-idempotency.md); identity I-09 |
| D-26 | An application programming interface is fast on seed data, or a database restore succeeds while pending assignments/messages are missing; weekend admin downtime counts differently between documents. | Keep the approved budget topology. Specify one SLO/calendar, representative workload/data volume and an end-to-end restore proving business state, not just database availability. | [non-functional requirement targets](../../PRD.md); [business requirements document recovery/calendar](../../BRD.md); [cutover drill](../../027-aws-production-deployment/spec.md); identity I-10 |

Two additional decisions concern authority rather than new functionality:

- **A-01: Approved taxonomy version.** Spec 025's shipped header, draft body and proposed mapping/signoff instructions disagree. Recover and link the actual approval/effective mapping; do not infer approval from code or ask the founder to reconfirm already-approved content without first locating its record. [Conflicting status](../../025-taxonomy-stream-year-scoping/spec.md).
- **A-02: Meaning of limited anonymous search.** Public filtering is confirmed. Record one audience×endpoint table for page size, maximum page, request rate, filter support and result-count visibility. The current anonymous tutor-list application programming interface allows pages 0–4 and caps size at 20; that is a per-query pagination cap, not a cumulative visitor search allowance. [Private supporting material excluded (publication note)](../../../docs/publication-notes.md).

## 4. Clear requirements that need implementation or scope reconciliation

These should not be presented as unanswered feature wishes. Verify/fix them against the existing requirement, or explicitly approve a scope change.

| ID | Established requirement | Observed gap / necessary action | Evidence |
|---|---|---|---|
| G-01 | Public limited filtering (C-01). | Both browse pages hide filter controls when logged out. | [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |
| G-02 | Save/resume drafts for both creation flows. | Current component-local wizard state loses work on navigation/reload; an in-flow rate-guide link leaves the form. Implement the promised persistence or explicitly defer it. | [Autosave requirement](../../modules/posting/spec.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |
| G-03 | Validate forms and surface actionable field errors. | Optional headline, notes and teaching-approach limits can block final submission without corresponding visible errors. | [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |
| G-04 | APPLIED and SHORTLISTED applications may be withdrawn. | Backend follows the newer rule; tutor user interface still offers withdrawal only for APPLIED. | [New rule](../../architecture/int-093-batch-state-machine-integrity-spec.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |
| G-05 | First-session confirmation before commission invoice. | Current activation/payment path does not implement the documented confirmation journey. Reconcile this rather than interpreting three current statuses as policy approval to remove it. Narrow ambiguity remains: business requirements document allows override at any time while product requirements document describes seven-day timeout. Define evidence for early override. | [AC-05](../../PRD.md); [M18 preserves it](../../PRD.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |
| G-06 | Contact changes use supported confirmation emails and step-up protection on all exposed surfaces. | Renderer lacks CONTACT CHANGE CONFIRM/NOTICE; recipient resolution uses current user email rather than the new-email destination; old direct phone endpoint bypasses the pending flow. These are separate from D-21/D-22's missing recovery policies. | [Contract](../../012-tutor-apply-and-profile-ux/requirements.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |
| G-07 | Rejected PENDING posts close/void pending invites and interviews. | Already explicit in 021. Verify the complete rejection cascade; do not ask whether the rule should exist. This audit does not assert all its runtime paths fail. | [Requirement](../../021-tutee-request-flow/spec.md) |
| G-08 | Browser regression suite covers its configured browsers. | continuous integration installs Chromium only although the full project set includes Web Kit. Correct the installation/gate and verify with a fresh runner. | [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) |

Marketing also needs reconciliation: the “never shared, ever” promise is incompatible with participant contact reveal (D-17). The hardcoded “7 days median time to match” should have a dated, traceable actual source before being stated as a measured outcome; it currently has no runtime data source in the component. This is not proof the founder has no supporting data. [Private supporting material excluded (publication note)](../../../docs/publication-notes.md); [honesty constraint](../../005-landing-page-trust-and-social-proof/requirements.md).

## 5. Settled changes and deliberate exclusions

Do not reopen these simply because an older document differs:

- M18 removed the published-listing prerequisite for applying; profile completeness replaced it.
- M17 made lists public and details authenticated; current user clarification resolves public filters.
- Optional onboarding capture does not make all fields mandatory at account creation.
- Subject-level matrix completeness does not impose a per-job eligibility hard gate on Apply.
- PENDING parent requests may be edited and may initiate invitations/interviews.
- Invitation acceptance must not overwrite a different tutor already deliberately chosen by the parent.
- Reopening on cancellation is the decided default; D-09 concerns missing exceptions and stale interest.
- Interview cap is concurrent per parent posting; 72-hour unanswered expiry and editable admin accepted-time prefill are intentional.
- Commission snapshot location, nullable legacy values and mandatory manual FLEXIBLE session count were resolved in INT-157 v2. The commercial multiplier/rate questions were not.
- The commission amount locks at activation. COMPLETED-on-PAID is the authoritative match definition; actual tuition end and post-settlement replacement still need separate handling.
- Tutor/tutee access expiry period is 30 minutes with refresh; older seven-day access-token wording is historical. Admin sessions remain separate.
- Current contact-storage and soft-delete choices have explicit project decisions; this audit does not replace them. Retention timing still needs reconciliation.
- Phone change uses the approved email-link fallback. WhatsApp reminders remain gated OFF until enablement criteria; do not treat OFF as a missing feature.
- db-scheduler superseded the old reminder sweep mechanism. The founder explicitly deferred automatic scheduler alerts in favor of metrics; do not silently restore that requirement.
- Production one-time passcode debug logging is explicitly OFF under spec 027, superseding the earlier broad logging decision.
- Testimonials are deliberately blocked on real consented content. Missing fabricated testimonials are not a product defect.
- Integrated payment processing, direct tutor-parent chat, automated matching, native mobile apps and broader acquisition work are not automatically added to scope by this review.

## 6. Acceptance scenarios to add or reconcile

These are requirements tests to define after decisions, not tests run during this audit. Each should assert actor permission, source/destination states, money effects, data visibility, messages and idempotent replay where relevant.

| Scenario | Expected invariant / decision to encode |
|---|---|
| Visitor filters by subject, level and area without login. | Bounded public results; no contact; detail/action gate preserves intent. C-01/A-02. |
| Tutor signs up with optional details omitted, then tries to apply. | Signup succeeds; apply explains exact missing profile fields; no listing required. |
| Parent follows rate-guide link halfway through a request and returns or reloads. | Draft and step recover, or a clearly approved alternative preserves input. G-02. |
| Optional notes exceed their permitted length. | Error appears on the owning field/step; submission never silently stalls. G-03. |
| Pending request invites tutor; tutor responds before approval, then request is approved/rejected. | Defined actionable/waiting state; correct notifications; rejection closes pending work. D-07/G-07. |
| Parent materially edits a request after tutor interest and before admin assignment. | Latest agreed terms are visible; no assignment based on obsolete acceptance. D-06. |
| Parent changes/withdraws selection while admin is assigning. | One valid outcome wins; no successful assignment to a superseded choice. Requires concurrency acceptance as part of the selection contract. |
| Parent chooses A with future interviews for A and B. | Explicit survival/cancellation policy, correct notices and no stale reminder. D-10. |
| Last proposed time passes before 72 hours; admin misses an agreed slot. | No acceptance/scheduling of unusable past agreement; defined expiry/reproposal/escalation. D-11. |
| Admin moves an agreed time or creates a confirmed overlap across jobs. | Reason, appropriate re-consent and collision warning/block behavior. D-12. |
| Activate at S$60 × 1.5h × twice weekly. | Explicit agreed count controls fee; rate/count/amount shown consistently. D-03/D-04. |
| Parent paid for lessons but tutor has not paid agency. | Agency commission remains unpaid; no agency revenue recognized. C-02. |
| First lesson has not occurred; admin tries invoice progression. | Documented qualifying-session/override gate holds; override has evidence. G-05. Actual advance receipts need an explicit recording procedure under D-02; no ban on recording real receipts is inferred. |
| Cancel before lessons, after lessons/unpaid, after agency paid, and for no-longer-seeking. | Approved debt, replacement, reopening and review outcomes; no implicit debt erasure. D-01/D-05/D-09. |
| Receipt entered against wrong tutor, partial payment, or wrong session count. | Audited correction and outstanding balance without deleting history. D-02. |
| Minor learner/tutor or legacy user without consent attempts a new commitment. | Defined responsible adult and current-consent rules, with support/privacy access preserved. D-18/D-19. |
| Concurrent email and phone changes; lost mailbox; expired link; cancel races with confirm. | Old values remain until valid confirmation; correct recipient/channel; one final outcome and audit. D-21/D-22/G-06. |
| Logout during outage, reload and second tab, then reconnect. | Defined pending revocation behavior without weakening HttpOnly cookies. D-23. |
| Delete account with active lessons and outstanding commission. | Clear effects on demand, counterparties and debt; retention/anonymisation includes later-added records. D-01/D-20. |
| Queue outage then recovery after one-time passcode expiry or meeting cancellation/link change. | Expired/obsolete messages are not treated as successful delivery; safe retry does not duplicate valid messages. D-25. |
| Recover sole super-admin and restore production data from backup. | Authorized recovery and actual business reconciliation, within agreed loss/time bounds. D-24/D-26. |

## 7. Recommended next deliverable

Create a current requirements baseline before another broad build batch:

1. **Decision log:** record C-01/C-02 now; resolve D-01–D-08 first, then remaining cross-feature/security/operations cases. Each approved answer needs date, owner, affected requirement IDs and explicitly superseded wording.
2. **Actor×state×action matrix:** distinguish post PENDING, match PENDING, pending commission and PENDING SCHEDULING. Include permissions, errors, downstream transitions, contact visibility and notifications.
3. **Field and event glossary:** profile defaults versus listing overrides; exact capability pairs; service availability; agreed versus advertised fee; agency receipt versus parent tuition payment; queued versus delivered.
4. **Traceability:** expand requirements traceability matrix to all specs and link requirements to current code, acceptance tests, evidence and release version. Separate APPROVED, implemented, tested, deployed and feature-enabled states.
5. **Bounded implementation backlog:** prioritize G-05/G-06 and user-visible G-01–G-04; build unresolved-policy changes only after the relevant decision. Maintain explicit deferrals instead of letting old P0 commitments disappear.

## Review limits

This was a requirements audit, not a production certification. All numbered spec folders were indexed; domain reviewers examined the relevant requirements, amendments and selected code paths across the product. Historical prose and diagrams were used for traceability, not treated as uniformly current. The local backend was stopped and dependency hydration previously blocked build/lint; no runtime or regression pass is claimed. The new files are review artifacts only. No production configuration, source, existing spec or external issue tracker was modified.
