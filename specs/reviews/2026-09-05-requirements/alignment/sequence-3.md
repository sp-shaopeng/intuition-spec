# Sequence alignment —29–43

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

All15 files read in full. Descriptive/code-derived sequences are checked against product requirements and explicit amendments; no implementation verification.

| Sequence | Verdict | Assessment | Evidence |
|---|---|---|---|
| [29-change-password md](../../../sequences/29-change-password.md) | partial | Password rotation/gate clearing remains correct, but wrong-current-password 400 INVALID PASSWORD is superseded by INT-095's422 INVALID CREDENTIALS with session retained. 'Admin can now operate' applies to the older local no-multi-factor authentication slice; prod027 also requires multi-factor authentication. | [sequences/29-change-password md](../../../sequences/29-change-password.md); [architecture/int-095-batch-auth-session-hardening-spec md](../../../architecture/int-095-batch-auth-session-hardening-spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md) |
| [30-account-deletion md](../../../sequences/30-account-deletion.md) | partial | Soft-delete and current-session invalidation describe a narrow historical path. Wrong-password status is superseded by INT-095422 with preserved session. Refresh-family/other-device invalidation and timed anonymization acceptance require current account-lifecycle linkage; current signed session token denylist alone is not full session-revocation evidence. | [sequences/30-account-deletion md](../../../sequences/30-account-deletion.md); [sequences/30-account-deletion md](../../../sequences/30-account-deletion.md); [architecture/int-095-batch-auth-session-hardening-spec md](../../../architecture/int-095-batch-auth-session-hardening-spec.md); [004-auth-session-hardening-30min-ttl-and-refresh/requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) |
| [31-admin-create-delete-admin md](../../../sequences/31-admin-create-delete-admin.md) | partial | Create-admin temp-password shape follows folded module decision and delete remains super-admin-only. Delete sequence omits mandatory target-admin session purge and audit; prod first-login requires multi-factor authentication after forced password change. It cannot represent the whole current operational lifecycle. | [sequences/31-admin-create-delete-admin md](../../../sequences/31-admin-create-delete-admin.md); [sequences/31-admin-create-delete-admin md](../../../sequences/31-admin-create-delete-admin.md); [admin-management/spec md](../../../modules/admin-management/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md) |
| [32-post-edit-and-close md](../../../sequences/32-post-edit-and-close.md) | partial | DRAFT/REJECTED delete remains distinct from live Close. Sequence is incomplete for founder-approved ACTIVE/PENDING close and cascades. Blanket edit status-unchanged text must be scoped: ACTIVE edit requeues PENDING, PENDING edit is explicitly allowed by021, committed states stay restricted. | [sequences/32-post-edit-and-close md](../../../sequences/32-post-edit-and-close.md); [architecture/int-062-cascade-close-spec md](../../../architecture/int-062-cascade-close-spec.md); [architecture/int-093-batch-state-machine-integrity-spec md](../../../architecture/int-093-batch-state-machine-integrity-spec.md); [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md) |
| [33-profile-edit md](../../../sequences/33-profile-edit.md) | conflict | Authenticated own-profile editing is fine, but the postcondition says display Name flows to the public card/showcase. Founder-approved M18 public identity uses public handle and forbids real/display Name. Later026 permits humanised public display; sequence still needs to identify the sanctioned public field. | [sequences/33-profile-edit md](../../../sequences/33-profile-edit.md); [uiux/assignment-uiux-spec md](../../../uiux/assignment-uiux-spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md) |
| [34-tutor-availability-toggle md](../../../sequences/34-tutor-availability-toggle.md) | conflict | Consent/availability predicate is coherent; its assertion that a personal information-free showcase contains display name conflicts with later handle-only contract (also sequence40). Revocation refresh delay is described but no upper bound is supplied here; it should inherit current consent/cache requirements. | [sequences/34-tutor-availability-toggle md](../../../sequences/34-tutor-availability-toggle.md); [sequences/34-tutor-availability-toggle md](../../../sequences/34-tutor-availability-toggle.md); [uiux/assignment-uiux-spec md](../../../uiux/assignment-uiux-spec.md); [sequences/40-public-showcase-strips md](../../../sequences/40-public-showcase-strips.md) |
| [35-tutee-cancel-interview md](../../../sequences/35-tutee-cancel-interview.md) | partial | Own-interview cancellation and cap release remain correct, but REQUESTED/SCHEDULED-only allow-list and old enum omit negotiation states. INT-039 explicitly widens tutee cancellation to non-terminal states. | [sequences/35-tutee-cancel-interview md](../../../sequences/35-tutee-cancel-interview.md); [sequences/35-tutee-cancel-interview md](../../../sequences/35-tutee-cancel-interview.md); [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md) |
| [36-marketplace-detail-login-gated md](../../../sequences/36-marketplace-detail-login-gated.md) | aligned | Public lists and authenticated detail preserve user's limited anonymous filtering policy; personal information-free tutor detail, safe401/404 and handle-based rendering agree with M17. Historical service names are descriptive only. | [sequences/36-marketplace-detail-login-gated md](../../../sequences/36-marketplace-detail-login-gated.md); [sequences/36-marketplace-detail-login-gated md](../../../sequences/36-marketplace-detail-login-gated.md); [sequences/36-marketplace-detail-login-gated md](../../../sequences/36-marketplace-detail-login-gated.md); [architecture/m17-marketplace-ux-overhaul md](../../../architecture/m17-marketplace-ux-overhaul.md) |
| [37-contact-reveal-post-assignment md](../../../sequences/37-contact-reveal-post-assignment.md) | partial | Participant-only application programming interface reveal at PENDING/ACTIVE/COMPLETED and CANCELLED404 aligns with M18 application programming interface contract. Add later deliberate frontend restriction to ACTIVE/COMPLETED; this is not an application programming interface-policy conflict. AES-SIV/decryption wording and old known500 status are historical/superseded. | [sequences/37-contact-reveal-post-assignment md](../../../sequences/37-contact-reveal-post-assignment.md); [sequences/37-contact-reveal-post-assignment md](../../../sequences/37-contact-reveal-post-assignment.md); [sequences/37-contact-reveal-post-assignment md](../../../sequences/37-contact-reveal-post-assignment.md); [architecture/int-125-batch-closeout-spec md](../../../architecture/int-125-batch-closeout-spec.md); [architecture/plaintext-email-phone md](../../../architecture/plaintext-email-phone.md) |
| [38-choose-tutor-selection md](../../../sequences/38-choose-tutor-selection.md) | partial | Selection is correctly owner-scoped and never directly creates a match. Assign payload still depends on nullable tutor Post Id, omitting the explicitly required listingless tutor User Id/selection-request path in INT-034. Keep the older listing path as an optional alternative. | [sequences/38-choose-tutor-selection md](../../../sequences/38-choose-tutor-selection.md); [sequences/38-choose-tutor-selection md](../../../sequences/38-choose-tutor-selection.md); [architecture/int-034-free-market-assign md](../../../architecture/int-034-free-market-assign.md) |
| [39-admin-applicant-view md](../../../sequences/39-admin-applicant-view.md) | aligned | Admin-specific route, role gate and separation from parent personal information-free cards align with M18 and admin user experience amendments. Add link to global multi-factor authentication/forced-change guards if used as an end-to-end prod recipe; endpoint role requirement itself is not inconsistent. | [sequences/39-admin-applicant-view md](../../../sequences/39-admin-applicant-view.md); [sequences/39-admin-applicant-view md](../../../sequences/39-admin-applicant-view.md); [specs/product requirements document md](../../../PRD.md) |
| [40-public-showcase-strips md](../../../sequences/40-public-showcase-strips.md) | partial | Cache-only public personal information-free strips and bounded response cache align with M17. Unqualified CF-Connecting-IP→XFF precedence lacks027's explicit trusted-hop restriction; update environment scope. Description of full enumerable browse should retain current public result caps. | [sequences/40-public-showcase-strips md](../../../sequences/40-public-showcase-strips.md); [sequences/40-public-showcase-strips md](../../../sequences/40-public-showcase-strips.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [architecture/m17-marketplace-ux-overhaul md](../../../architecture/m17-marketplace-ux-overhaul.md) |
| [41-admin-analytics md](../../../sequences/41-admin-analytics.md) | partial | Admin range endpoint and inverted-range guard remain coherent. Count-stub description predates later cohort requirements; sequence does not specify017's pinned interview numerator/denominator or019 diagnosis. Treat as incomplete descriptive snapshot, not authority to retain old metrics. | [sequences/41-admin-analytics md](../../../sequences/41-admin-analytics.md); [sequences/41-admin-analytics md](../../../sequences/41-admin-analytics.md); [sequences/41-admin-analytics md](../../../sequences/41-admin-analytics.md); [017-admin-analytics-and-financial-safety/requirements md](../../../017-admin-analytics-and-financial-safety/requirements.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md) |
| [42-interview-timeslot-negotiation md](../../../sequences/42-interview-timeslot-negotiation.md) | conflict | Negotiation happy path/cap mostly maps to INT-039, but diagram changes tutee counter back to PROPOSED while normative contract retains COUNTER PROPOSED. It also marks prior slots REJECTED instead of WITHDRAWN and gives different slot-validation errors. Descriptive sequence cannot amend those requirements. | [sequences/42-interview-timeslot-negotiation md](../../../sequences/42-interview-timeslot-negotiation.md); [sequences/42-interview-timeslot-negotiation md](../../../sequences/42-interview-timeslot-negotiation.md); [sequences/42-interview-timeslot-negotiation md](../../../sequences/42-interview-timeslot-negotiation.md); [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md); [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md) |
| [43-tutee-tutor-job-invitation md](../../../sequences/43-tutee-tutor-job-invitation.md) | conflict | Apply+choose reuse and admin-only assign align. ACTIVE-only invite prohibition is superseded by021; request identity updated by024. Background-expiry arrow contradicts explicit no-timeout/cause-bearing CLOSED versus EXPIRED design in INT-062; wrong-state error also differs from INT-044409. | [sequences/43-tutee-tutor-job-invitation md](../../../sequences/43-tutee-tutor-job-invitation.md); [sequences/43-tutee-tutor-job-invitation md](../../../sequences/43-tutee-tutor-job-invitation.md); [sequences/43-tutee-tutor-job-invitation md](../../../sequences/43-tutee-tutor-job-invitation.md); [sequences/43-tutee-tutor-job-invitation md](../../../sequences/43-tutee-tutor-job-invitation.md); [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [architecture/int-062-cascade-close-spec md](../../../architecture/int-062-cascade-close-spec.md); [architecture/int-044-job-invitations md](../../../architecture/int-044-job-invitations.md) |

## Paired-source findings

### SEQ3-01 —Credential re-confirmation error contract is stale (superseded-drift)

- [sequences/29-change-password md](../../../sequences/29-change-password.md): “400 INVALID PASSWORD — current password wrong”
- [sequences/30-account-deletion md](../../../sequences/30-account-deletion.md): “400 /401 — wrong password / unauthenticated”
- [architecture/int-095-batch-auth-session-hardening-spec md](../../../architecture/int-095-batch-auth-session-hardening-spec.md): “credential re-confirmation ...422 INVALID CREDENTIALS”

Later requirement deliberately separates mistyped confirmation from expired login and tests session survival.

Proposed repair: Update both sequence errors to422 INVALID CREDENTIALS and show inline retry with /me still200; reserve401 for genuine session failure.

### SEQ3-02 —Admin removal/production onboarding sequence omits retained security steps (partial)

- [sequences/31-admin-create-delete-admin md](../../../sequences/31-admin-create-delete-admin.md): “guard ... soft-delete target ...204 ...disappears”
- [admin-management/spec md](../../../modules/admin-management/spec.md): “purge target's session:admin:* keys ... Audit ADMIN CREATED/ADMIN REMOVED with actor id + target id”
- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “multi-factor authentication-enrol required before any admin action”

Role-gated create/delete and forced password change are not the whole required current production flow. Omitted steps are explicit requirements, not hypothetical safeguards.

Proposed repair: Show session purge+audit on removal and production multi-factor authentication enrollment/challenge before operating; label local no-multi-factor authentication slice separately.

### SEQ3-03 —Post edit/close sequence omits the later live-close transition (superseded-drift)

- [sequences/32-post-edit-and-close md](../../../sequences/32-post-edit-and-close.md): “Edit ... status unchanged ...Only DRAFT or REJECTED ...owner-deleted”
- [architecture/int-062-cascade-close-spec md](../../../architecture/int-062-cascade-close-spec.md): “POST .../close ... ACTIVE or PENDING ... apps CLOSED ... invitations CLOSED”
- [architecture/int-093-batch-state-machine-integrity-spec md](../../../architecture/int-093-batch-state-machine-integrity-spec.md): “ACTIVE ... re-queues to PENDING”
- [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md): “PENDING editable ... save leaves ... PENDING”

The existing DELETE restriction need not be removed; a separate founder-approved close operation and state-aware edit branch are missing.

Proposed repair: Extend sequence with POST close and its visible terminal cascades; add ACTIVE→PENDING edit and PENDING→PENDING edit. Keep MATCHED/CLOSED edit restriction.

### SEQ3-04 —Profile/showcase sequences use private display Name as public identity (conflict)

- [sequences/33-profile-edit md](../../../sequences/33-profile-edit.md): “qualifications/display Name flow ... public tutor card + showcase”
- [sequences/34-tutor-availability-toggle md](../../../sequences/34-tutor-availability-toggle.md): “personal information-free (display name + subjects/levels only)”
- [uiux/assignment-uiux-spec md](../../../uiux/assignment-uiux-spec.md): “public handle ... NEVER display Name”
- [sequences/40-public-showcase-strips md](../../../sequences/40-public-showcase-strips.md): “public handle only (no real name)”

Contradictory field names can recreate private-identity leakage. A human-readable public handle is compatible with026's polish; raw private display Name is not established by these old sequences.

Proposed repair: Name the approved public identity field and describe private profile/display name separately; link any later scoped identity amendment if one exists.

### SEQ3-05 —Cancel interview sequence still excludes negotiation states (superseded-drift)

- [sequences/35-tutee-cancel-interview md](../../../sequences/35-tutee-cancel-interview.md): “REQUESTED or SCHEDULED”
- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md): “tutee ... cancel ... non-terminal→CANCELLED; widen allowed source statuses”

New PROPOSED/COUNTER PROPOSED/PENDING SCHEDULING states are absent from old flow and enum.

Proposed repair: Show cancellation for current non-terminal states and include cap-release/notification effects.

### SEQ3-06 —Choose-tutor assignment sequence still requires a listing-shaped key (superseded-drift)

- [sequences/38-choose-tutor-selection md](../../../sequences/38-choose-tutor-selection.md): “POST /admin/matches tutor Post Id,tutee Post Id ...nullable tutor Post Id”
- [architecture/int-034-free-market-assign md](../../../architecture/int-034-free-market-assign.md): “Re-key ... tutor user Id ... optional tutor Post ... user Id OR tutor Post Id”

A listingless applicant can be selected but has no required tutor Post Id to send in this diagram; INT-034 exists specifically to fix this contract.

Proposed repair: Document user Id/selection-request assignment as canonical listingless path and retain listing-id path as optional enrichment.

### SEQ3-07 —Public-strip client-IP precedence lacks trusted-proxy condition (superseded-drift)

- [sequences/40-public-showcase-strips md](../../../sequences/40-public-showcase-strips.md): “CF-Connecting-IP → X-Forwarded-For[0] → get Remote Addr operation”
- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “trusted hop ...untrusted source ...not honored for bucketing”

The production amendment requires origin authenticity before accepting header-based client identity; a bare precedence list is insufficient.

Proposed repair: Add trusted sidecar/hop precondition and spoofed-header rejection branch, preserving rate-limit dimensions.

### SEQ3-08 —Tutee counter and prior-round slot states differ from normative negotiation contract (conflict)

- [sequences/42-interview-timeslot-negotiation md](../../../sequences/42-interview-timeslot-negotiation.md): “COUNTER PROPOSED→PROPOSED : tutee counters”
- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md): “COUNTER PROPOSED ... →COUNTER PROPOSED”
- [sequences/42-interview-timeslot-negotiation md](../../../sequences/42-interview-timeslot-negotiation.md): “round1→REJECTED”
- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md): “previous round WITHDRAWN; tutee counter→COUNTER PROPOSED”

Sequence changes state vocabulary without an explicit requirement amendment; role turn is carried by whose Turn/last actor, not a new initial status.

Proposed repair: Align state diagram and round transitions with effective contract, or link a scoped approved amendment before changing acceptance tests.

### SEQ3-09 —Negotiation/invitation error statuses disagree with requirements (conflict)

- [sequences/42-interview-timeslot-negotiation md](../../../sequences/42-interview-timeslot-negotiation.md): “422 INVALID STATUS TRANSITION ...400 VALIDATION ERROR ...empty slots/>3”
- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md): “0 slots/>3→422 SLOT REQUIRED/SLOT TOO MANY ...out of turn→422 NOT YOUR TURN”
- [sequences/43-tutee-tutor-job-invitation md](../../../sequences/43-tutee-tutor-job-invitation.md): “422 accept on non-PENDING”
- [architecture/int-044-job-invitations md](../../../architecture/int-044-job-invitations.md): “409 INVITATION NOT PENDING”

These code-derived descriptions cannot choose between conflicting typed frontend contracts.

Proposed repair: Declare canonical error/status per case and update all affected diagrams/assertions. Do not infer that existing implementation output constitutes approval.

### SEQ3-10 —Invitation sequence retains superseded ACTIVE-only initiation and invented timeout (conflict)

- [sequences/43-tutee-tutor-job-invitation md](../../../sequences/43-tutee-tutor-job-invitation.md): “cannot invite on ...PENDING”
- [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md): “allow invite/interview ...PENDING”
- [sequences/43-tutee-tutor-job-invitation md](../../../sequences/43-tutee-tutor-job-invitation.md): “PENDING→EXPIRED : background expiry sweep”
- [architecture/int-062-cascade-close-spec md](../../../architecture/int-062-cascade-close-spec.md): “invitations have NO timeout ...no scheduled timeout”
- [architecture/int-062-cascade-close-spec md](../../../architecture/int-062-cascade-close-spec.md): “EXPIRED is filled by match; CLOSED is withdrawn”

PENDING initiation is explicitly settled; the separate time-based expiration is neither in the governing invitation spec nor approved by the code-derived sequence.

Proposed repair: Update creation predicate to ACTIVE/PENDING per021, identity per024, and represent job-close/assignment cascades with their cause-bearing states. Leave true expiry period as a separately approved decision if needed.
## Review limits

- Sequences are descriptive and code-derived; they cannot supersede product requirements without an explicit amendment.
- No source-code edits, runtime tests or provider calls.
- Anonymous limited filtering remains allowed. PAID remains agency receipt of tutor commission.
- PENDING application programming interface contact permission and ACTIVE/COMPLETED frontend button permission are deliberately different per INT-125:146; not classified as an unresolved policy conflict.
