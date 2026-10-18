# Auxiliary specification alignment — lane 2

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Auxiliary spec alignment lane 2: all 38 assigned files represented; requirements/design review only, no code inspection, edits, runtime tests or external actions in this pass.

Every assigned file received a targeted normative/headings scan. Selected clauses and folded decisions were read in depth. This is not a full line-by-line review of approximately 12,000 lines; examples and implementation observations remain historical/descriptive. An aligned verdict means no conflict found in inspected normative material, not exhaustive certification.

Confirmed decisions: Anonymous visitors may filter limited public results. Commission PAID means agency actually received the tutor's commission; parent lesson payments are separate.

## File inventory

### aux2-01 — architecture/admin-identity-linking-spec md — aligned

Admin-only identity linking, phone visibility and close-job discovery agree with later admin-persona work. Public/applicant personal information expansion is expressly excluded. Early destination/data transfer object choices are implementation choices, not an unresolved disclosure policy.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/admin-identity-linking-spec md L164–182](../../../architecture/admin-identity-linking-spec.md); [014-interview-scheduling-ux-and-admin-visibility/design md L54–65](../../../014-interview-scheduling-ux-and-admin-visibility/design.md).

### aux2-02 — architecture/assignment-be-spec md — partial

M18 free-market assignment and atomic lifecycle are the effective baseline. The final founder-decision section expressly supersedes the earlier OPEN table, including removal of listing gates. Later contact-change, phone-verification and locked-plan specs must be linked; old examples are not independent authority.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/assignment-be-spec md L843–875](../../../architecture/assignment-be-spec.md); [product requirements document md L2011–2025](../../../PRD.md); [architecture/int-157-commission-first-month-locked-spec md L73–109](../../../architecture/int-157-commission-first-month-locked-spec.md).

### aux2-03 — architecture/assignment-fe-spec md — partial

The document explicitly resolves all six OPEN using frontend decisions and folds Round-2 review. Preserve applicant/assignment user experience, but use 012's later server-computed profile predicate rather than treating earlier frontend computations as a second canonical gate.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/assignment-fe-spec md L1–6](../../../architecture/assignment-fe-spec.md); [architecture/assignment-fe-spec md L420–459](../../../architecture/assignment-fe-spec.md); [012-tutor-apply-and-profile-ux/design md L11–28](../../../012-tutor-apply-and-profile-ux/design.md).

### aux2-04 — architecture/backend-correctness-security-spec md — aligned

Targeted pagination, trusted-proxy IP, reset/session hardening and singleton scheduler intentions are consistent with subsequent security work. Proposed library/mechanism choices are not commercial policy; 007 is the later scheduler architecture. No current implementation claim is made.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/backend-correctness-security-spec md L19–29](../../../architecture/backend-correctness-security-spec.md); [architecture/backend-correctness-security-spec md L167–179](../../../architecture/backend-correctness-security-spec.md); [007-generic-scheduler-db-scheduler/spec md L1–24](../../../007-generic-scheduler-db-scheduler/spec.md).

### aux2-05 — architecture/int-025-admin-submit-notification md — aligned

The folded decision mandates immediate per-submit notifications to all admins; batching/opt-out only if volume warrants later review. DRAFT status and optional after-commit refactoring are not evidence that the audience or delivery decision is unapproved.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/int-025-admin-submit-notification md L253–254](../../../architecture/int-025-admin-submit-notification.md); [architecture/int-025-admin-submit-notification md L368–376](../../../architecture/int-025-admin-submit-notification.md).

### aux2-06 — architecture/int-046-fe-coverage-closeout md — aligned

A historical coverage-closeout specification, not a product amendment. The final recommendation retains the 97.5% line gate and rejects lowering/excluding merely to pass. Earlier option analysis must not be mistaken for the approved plan.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/int-046-fe-coverage-closeout md L406–454](../../../architecture/int-046-fe-coverage-closeout.md).

### aux2-07 — architecture/int-090-batch-m18-funnel-completion-spec md — aligned

Folded contact-reveal requirements explicitly require symmetric ACTIVE-match participant access and advance mutual-visibility copy. This settles old 'counterparty never' statements; the dormant original phone gate is superseded by later 012 channel decisions.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/int-090-batch-m18-funnel-completion-spec md L271–299](../../../architecture/int-090-batch-m18-funnel-completion-spec.md); [012-tutor-apply-and-profile-ux/requirements md L123–126](../../../012-tutor-apply-and-profile-ux/requirements.md).

### aux2-08 — architecture/int-093-batch-state-machine-integrity-spec md — partial

Application reapply/withdraw rules are explicitly resolved: SUPERSEDED/CLOSED can reactivate; REJECTED/MATCHED cannot. Payment guards and COMPLETED updates are a service-correctness slice, not permission to omit the parent product requirements document first-session obligation or redefine received commission.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/int-093-batch-state-machine-integrity-spec md L44–45](../../../architecture/int-093-batch-state-machine-integrity-spec.md); [architecture/int-093-batch-state-machine-integrity-spec md L135–175](../../../architecture/int-093-batch-state-machine-integrity-spec.md); [product requirements document md L2034–2037](../../../PRD.md).

### aux2-09 — architecture/int-095-batch-auth-session-hardening-spec md — partial

Session/cache cleanup and safe logout requirements remain consistent. The old generic registration-duplicate policy in INT-109 is explicitly superseded by 011's founder-approved phone-specific 409; do not reopen that decision or use this historical batch to override 011.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/int-095-batch-auth-session-hardening-spec md L335–400](../../../architecture/int-095-batch-auth-session-hardening-spec.md); [011-auth-robustness/spec md L1–24](../../../011-auth-robustness/spec.md).

### aux2-10 — architecture/int-138-int137-codereview-fastfollow-spec md — aligned

Folded review preserves required shared identity/humanization, mobile and accessibility improvements. The later conditional follow-up for a structural scan is engineering sequencing; boilerplate SPEC status is not itself a missing product decision.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/int-138-int137-codereview-fastfollow-spec md L1–13](../../../architecture/int-138-int137-codereview-fastfollow-spec.md); [architecture/int-138-int137-codereview-fastfollow-spec md L183–189](../../../architecture/int-138-int137-codereview-fastfollow-spec.md).

### aux2-11 — architecture/int-157-commission-first-month-locked-spec md — authority-gap

Agreed-plan commission, activation lock, integer SGD storage, nullable legacy fields and FLEXIBLE requiring admin input are resolved. Only hourly-rate hard-clamping and commercial 4.33-versus-4 multiplier remain explicitly founder decisions; provisional build defaults are recorded, not ratification.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/int-157-commission-first-month-locked-spec md L76–109](../../../architecture/int-157-commission-first-month-locked-spec.md).

### aux2-12 — architecture/m14b-public-job-postings-showcase md — aligned

The deployed public showcase explicitly records founder waiver of consent gating and the permitted limited personal information-free public slice. Preserve that settled amendment; current user confirmation also allows filtering limited guest results, without granting full-detail access.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/m14b-public-job-postings-showcase md L1–5](../../../architecture/m14b-public-job-postings-showcase.md); [architecture/m14b-public-job-postings-showcase md L37–54](../../../architecture/m14b-public-job-postings-showcase.md); [architecture/m14b-public-job-postings-showcase md L121–145](../../../architecture/m14b-public-job-postings-showcase.md).

### aux2-13 — architecture/match-lifecycle-fixes-spec md — partial

Cancellation cascade and paid-commission preservation are explicit. Older commission-basis discussion is superseded by INT-157's locked agreed plan. Excluding refunds from this fix is a scope boundary, not a complete cancellation/refund business policy.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/match-lifecycle-fixes-spec md L111–116](../../../architecture/match-lifecycle-fixes-spec.md); [architecture/match-lifecycle-fixes-spec md L157–161](../../../architecture/match-lifecycle-fixes-spec.md); [architecture/match-lifecycle-fixes-spec md L338–342](../../../architecture/match-lifecycle-fixes-spec.md); [architecture/int-157-commission-first-month-locked-spec md L91–102](../../../architecture/int-157-commission-first-month-locked-spec.md).

### aux2-14 — architecture/regression-branch-closure-spec md — partial

Historical regression branch list includes an obsolete no-listing 403 assertion. M18 explicitly removes that guard. Retain valid authorization/duplicate/terminal-state coverage but do not restore historical assertions contrary to founder-approved free-market requirements.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/regression-branch-closure-spec md L43–43](../../../architecture/regression-branch-closure-spec.md); [product requirements document md L2013–2017](../../../PRD.md).

### aux2-15 — architecture/regression-password-reset-fixtures-spec md — partial

Historical fixture closeout explicitly folds stronger real password-change evidence into its header. Read this as a dated test-fixture record; older /me-only acceptance or open fixture questions should not override the folded mutation proof.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [architecture/regression-password-reset-fixtures-spec md L1–6](../../../architecture/regression-password-reset-fixtures-spec.md); [architecture/regression-password-reset-fixtures-spec md L98–120](../../../architecture/regression-password-reset-fixtures-spec.md).

### aux2-16 — modules/applications/backend md — partial

M15 thin-slice requirements for a mandatory ACTIVE tutor listing and admin-only applicant detail are superseded by M18 listing-optional applications and parent personal information-free applicant access. The older endpoint/code description should be labeled historical.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/applications/backend md L48–54](../../../modules/applications/backend.md); [modules/applications/backend md L198–229](../../../modules/applications/backend.md); [product requirements document md L2013–2017](../../../PRD.md); [product requirements document md L2086–2095](../../../PRD.md).

### aux2-17 — modules/applications/spec md — partial

The early application slice retains listing prerequisites and admin-only applicant access that M18 explicitly replaces. Durable application identity and withdrawal semantics remain useful; two-state historical descriptions are expanded by INT-093.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/applications/spec md L64–96](../../../modules/applications/spec.md); [product requirements document md L2013–2017](../../../PRD.md); [architecture/int-093-batch-state-machine-integrity-spec md L135–175](../../../architecture/int-093-batch-state-machine-integrity-spec.md).

### aux2-18 — modules/approval/backend md — partial

Request-revision/email owner flow is specified, but reject guards still accept nonexistent PAUSED FOR REVIEW while the same file explicitly eliminates that state. This is settled canonical-state documentation drift, not a new founder question.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/approval/backend md L148–168](../../../modules/approval/backend.md); [modules/approval/backend md L187–188](../../../modules/approval/backend.md); [modules/approval/backend md L205–217](../../../modules/approval/backend.md).

### aux2-19 — modules/approval/er-diagram md — partial

Targeted ER/transition material omits REVISION REQUESTED from the abbreviated status example while the module's current canonical workflow includes it. Historical implementation/audit-log observations are descriptive and cannot waive the required revision history.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/approval/er-diagram md L190–241](../../../modules/approval/er-diagram.md); [modules/approval/er-diagram md L253–260](../../../modules/approval/er-diagram.md); [modules/approval/spec md L33–65](../../../modules/approval/spec.md).

### aux2-20 — modules/approval/frontend md — aligned

Admin request-revision requires an owner-facing message, review-log persistence and notification; force-pending is ACTIVE→PENDING with a reason. These targeted normative flows match the canonical approval module. This does not prove the owner edit user interface satisfies its separate requirement.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/approval/frontend md L161–173](../../../modules/approval/frontend.md); [modules/approval/spec md L58–61](../../../modules/approval/spec.md).

### aux2-21 — modules/approval/spec md — partial

The canonical eight-state approval workflow and owner revision/resubmission are clear. Old drip examples still use APPROVED rather than canonical ACTIVE; treat those examples and previous notification mechanics as historical, not an alternate status model.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/approval/spec md L33–65](../../../modules/approval/spec.md); [modules/approval/spec md L252–260](../../../modules/approval/spec.md).

### aux2-22 — modules/auth/db md — partial

Encryption/keyed message authentication, soft deletion, erasure and role restrictions remain consistent in targeted sections. This is an older schema snapshot; M18 profile additions and later refresh/contact-change records require links rather than interpreting omitted columns as approved feature removal.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/auth/db md L12–50](../../../modules/auth/db.md); [modules/auth/db md L206–224](../../../modules/auth/db.md); [product requirements document md L2019–2025](../../../PRD.md); [004-auth-session-hardening-30min-ttl-and-refresh/requirements md L1–44](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md).

### aux2-23 — modules/commission/api-reference md — partial

The application programming interface reference predates agreed-plan activation and later cookie/refresh authentication. Its monthly Fee-only activation and signed session token-Bearer-only description need explicit version/supersession links. No full first-session application programming interface coverage is established by this abbreviated reference.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/commission/api-reference md L1–5](../../../modules/commission/api-reference.md); [modules/commission/api-reference md L123–132](../../../modules/commission/api-reference.md); [architecture/int-157-commission-first-month-locked-spec md L80–98](../../../architecture/int-157-commission-first-month-locked-spec.md); [004-auth-session-hardening-30min-ttl-and-refresh/requirements md L10–12](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md).

### aux2-24 — modules/commission/db md — partial

The seven-state confirmation/invoice/dispute model must be reconciled with later simplified collection status without deleting first-session requirements. Legacy monetary definitions are superseded by INT-157's explicit immutable integer-SGD storage; database query language examples are historical.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/commission/db md L14–32](../../../modules/commission/db.md); [modules/commission/db md L173–183](../../../modules/commission/db.md); [architecture/int-157-commission-first-month-locked-spec md L86–102](../../../architecture/int-157-commission-first-month-locked-spec.md); [product requirements document md L2034–2037](../../../PRD.md).

### aux2-25 — modules/commission/debugging-guide md — aligned

Targeted troubleshooting describes duplicate-open-match protection and permits rematching after cancellation, consistent with the module overview. It is a historical operational guide, not evidence of present behavior or approval to restore listing prerequisites.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/commission/debugging-guide md L49–56](../../../modules/commission/debugging-guide.md); [modules/commission/module-overview md L34–39](../../../modules/commission/module-overview.md).

### aux2-26 — modules/commission/module-overview md — partial

The simplified PENDING→ACTIVE→COMPLETED/commission summary omits the still-required first-session confirmation progression and predates agreed-plan inputs/listing-optional assignment. Lack of detail is a requirements-coverage gap, not proof of a code failure or product waiver.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/commission/module-overview md L34–53](../../../modules/commission/module-overview.md); [modules/commission/module-overview md L90–90](../../../modules/commission/module-overview.md); [product requirements document md L1058–1072](../../../PRD.md); [product requirements document md L2034–2037](../../../PRD.md); [architecture/int-157-commission-first-month-locked-spec md L80–102](../../../architecture/int-157-commission-first-month-locked-spec.md).

### aux2-27 — modules/commission/spec md — conflict

A won but unpaid dispute is explicitly labeled PAID, contradicting both the receipt guard in the same module and the user's confirmed meaning of PAID. The older generated/recalculable DECIMAL commission definition is separately superseded by INT-157's immutable integer snapshot.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/commission/spec md L81–89](../../../modules/commission/spec.md); [modules/commission/spec md L189–195](../../../modules/commission/spec.md); [modules/commission/spec md L103–107](../../../modules/commission/spec.md); [architecture/int-157-commission-first-month-locked-spec md L86–102](../../../architecture/int-157-commission-first-month-locked-spec.md).

### aux2-28 — modules/interview/max-interviews-per-posting md — partial

Max three concurrent interviews is founder-ratified; cancellation frees capacity. The original ACTIVE-only posting eligibility and REQUESTED/SCHEDULED active-set description are expanded by later PENDING-post permission and negotiation states, without changing the cap.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/interview/max-interviews-per-posting md L1–10](../../../modules/interview/max-interviews-per-posting.md); [modules/interview/max-interviews-per-posting md L30–34](../../../modules/interview/max-interviews-per-posting.md); [architecture/int-039-interview-negotiation md L60–90](../../../architecture/int-039-interview-negotiation.md); [021-tutee-request-flow/spec md L19–31](../../../021-tutee-request-flow/spec.md).

### aux2-29 — modules/interview/spec md — partial

The original five-state interview/listing flow and REQUESTED-only self-cancel requirement are historical. INT-039 explicitly adds negotiation, active-cap states and cancellation routes; 021 allows PENDING posts. Legacy flows coexist, but they are not the entire current state machine.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/interview/spec md L96–96](../../../modules/interview/spec.md); [modules/interview/spec md L229–229](../../../modules/interview/spec.md); [architecture/int-039-interview-negotiation md L50–94](../../../architecture/int-039-interview-negotiation.md); [021-tutee-request-flow/spec md L19–31](../../../021-tutee-request-flow/spec.md).

### aux2-30 — modules/landing-showcase/spec md — partial

The header explicitly marks early consent narrative historical and points to founder-approved M14b. Old listing/name examples need M18 profile/handle authority linked. Later capped browse must incorporate the confirmed allowance for limited anonymous filtering; that does not authorize unlimited public detail.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/landing-showcase/spec md L3–6](../../../modules/landing-showcase/spec.md); [modules/landing-showcase/spec md L97–98](../../../modules/landing-showcase/spec.md); [modules/landing-showcase/spec md L123–131](../../../modules/landing-showcase/spec.md); [modules/landing-showcase/spec md L408–435](../../../modules/landing-showcase/spec.md); [product requirements document md L1177–1194](../../../PRD.md).

### aux2-31 — modules/matching/backend md — authority-gap

Legacy matching/activation code examples are superseded in several details, but the public bearer-token first-session endpoint directly differs from the parent product requirements document's authenticated tutee-only flow, which M18 explicitly reuses. Reconcile the access contract while retaining the first-session obligation.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/matching/backend md L231–247](../../../modules/matching/backend.md); [product requirements document md L1060–1068](../../../PRD.md); [product requirements document md L2034–2037](../../../PRD.md).

### aux2-32 — modules/matching/debugging-guide md — partial

Historical troubleshooting contains open questions/code observations about rematching and notifications. Later lifecycle fixes/commission overview resolve rematching; M18 removes mandatory tutor listings. These old observations cannot become new product requirements.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/matching/debugging-guide md L71–85](../../../modules/matching/debugging-guide.md); [modules/matching/debugging-guide md L110–110](../../../modules/matching/debugging-guide.md); [modules/commission/module-overview md L34–39](../../../modules/commission/module-overview.md); [product requirements document md L2013–2017](../../../PRD.md).

### aux2-33 — modules/posting/backend md — partial

The document folds later permission to edit PENDING tutee posts, yet older browse/listing guards and APPROVED/PAUSED aliases persist elsewhere. Use 021 and M18 as effective amendments; do not confuse historical backend examples with approved exceptions.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/posting/backend md L135–140](../../../modules/posting/backend.md); [modules/posting/backend md L154–154](../../../modules/posting/backend.md); [modules/posting/backend md L220–220](../../../modules/posting/backend.md); [product requirements document md L2013–2017](../../../PRD.md); [021-tutee-request-flow/spec md L41–47](../../../021-tutee-request-flow/spec.md).

### aux2-34 — modules/posting/db md — partial

The top explicitly points to the canonical eight states, while older state-trigger/expiry database query language examples retain APPROVED/PAUSED. This is historical example drift, not evidence of a current schema defect. Refresh references to the canonical posting state model.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/posting/db md L14–14](../../../modules/posting/db.md); [modules/posting/db md L306–357](../../../modules/posting/db.md); [modules/posting/db md L397–407](../../../modules/posting/db.md); [modules/posting/spec md L230–236](../../../modules/posting/spec.md).

### aux2-35 — modules/posting/spec md — partial

Canonical revision/resubmission and ACTIVE→PENDING semantics are explicit. The old active-tutor-listing browse restriction is superseded by M18 and the assignment backend founder table. Availability encoding was only targeted-scanned; no new alignment conclusion is claimed on that encoding.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/posting/spec md L230–236](../../../modules/posting/spec.md); [modules/posting/spec md L320–320](../../../modules/posting/spec.md); [architecture/assignment-be-spec md L866–867](../../../architecture/assignment-be-spec.md); [product requirements document md L2013–2017](../../../PRD.md).

### aux2-36 — modules/profile/account-ui-changes md — partial

The same document allows authorized active-match contact while promising 'counterparty never'. The later symmetric ACTIVE-match requirement explicitly settles this: retain private public/pre-match surfaces and describe the approved participant exception.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [modules/profile/account-ui-changes md L3–12](../../../modules/profile/account-ui-changes.md); [modules/profile/account-ui-changes md L28–38](../../../modules/profile/account-ui-changes.md); [architecture/int-090-batch-m18-funnel-completion-spec md L271–299](../../../architecture/int-090-batch-m18-funnel-completion-spec.md).

### aux2-37 — uiux/all-pages-redesign-spec md — partial

Locked design tokens/humanization and folded SHIP decisions remain the design baseline; stale OPEN status is boilerplate. Broad never-share/contact wording and old login CTA generalizations need the M18 participant exception and 018 role-intent amendment linked.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [uiux/all-pages-redesign-spec md L24–50](../../../uiux/all-pages-redesign-spec.md); [uiux/all-pages-redesign-spec md L314–314](../../../uiux/all-pages-redesign-spec.md); [uiux/all-pages-redesign-spec md L592–604](../../../uiux/all-pages-redesign-spec.md); [architecture/int-090-batch-m18-funnel-completion-spec md L279–299](../../../architecture/int-090-batch-m18-funnel-completion-spec.md); [018-flag-hygiene-seeder-and-fe-hygiene/spec md L1–24](../../../018-flag-hygiene-seeder-and-fe-hygiene/spec.md).

### aux2-38 — uiux/frontend-design-review-2026-06-13.md — authority-gap

This is an advisory visual review, not an approved product contract. Its recommendation to mark every tutor 'Verified by InTuition' assumes a universal vetting guarantee absent from the free-market profile-completeness requirement. Require a defined verifiable eligibility criterion before treating the badge as mandatory.

Coverage: targeted normative/headings scan and selected relevant excerpts; not exhaustive full-text review.

Evidence: [uiux/frontend-design-review-2026-06-13.md L63–68](../../../uiux/frontend-design-review-2026-06-13.md); [uiux/frontend-design-review-2026-06-13.md L90–96](../../../uiux/frontend-design-review-2026-06-13.md); [product requirements document md L2013–2025](../../../PRD.md).

## Paired findings

### A2-F1 — PAID must not mean merely winning a dispute (P1, conflict)

- [modules/commission/spec md L81–81](../../../modules/commission/spec.md): INVOICED → PAID: after receiving payment
- [modules/commission/spec md L189–193](../../../modules/commission/spec.md): PAID — agency's favour (tutor must pay or has already paid)

CONFIRMED user decision: PAID means agency received the tutor's commission.

Scenario: An admin rules in the agency's favour but the tutor has not transferred money; the old resolution clause would falsely record received revenue.

Recommendation: Keep debt/adjudication separate from received payment; update the dispute transition text. This is a documentation conflict, not an implementation finding.

### A2-F2 — First-session access model differs between product requirements document and module endpoint (P1, authority-gap)

- [product requirements document md L1060–1068](../../../PRD.md): /tutee/matches/match id/confirm-session (authenticated, tutee-only)
- [modules/matching/backend md L231–247](../../../modules/matching/backend.md): POST matches confirm-session :token interface operation; Authorization: None (token-authenticated)
- [product requirements document md L2034–2037](../../../PRD.md): §20 reuses §8.3 verbatim

Scenario: A forwarded confirmation email can be acted on without the tutee account under the module contract, while the parent contract requires the tutee session.

Recommendation: Use the parent authenticated tutee-only requirement unless an explicit approved amendment establishes token-only access. Retain required first-session confirmation; missing simplified coverage does not supersede it.

### A2-F3 — Two commercial agreed-plan defaults remain explicitly provisional (P2, authority-gap)

- [architecture/int-157-commission-first-month-locked-spec md L104–109](../../../architecture/int-157-commission-first-month-locked-spec.md): NEEDS FOUNDER DECISION: hard clamping; multiplier 4.33 vs 4; build proceeds with provisional defaults
- [architecture/int-157-commission-first-month-locked-spec md L76–102](../../../architecture/int-157-commission-first-month-locked-spec.md): FLEXIBLE admin-enters, storage and activation immutability resolved; first-session due trigger unchanged

Scenario: A twice-weekly plan defaults to 9 monthly sessions under 4.33, versus 8 under 4; a renegotiated rate outside the posting range may or may not be accepted.

Recommendation: Ratify rate-clamp policy and monthly multiplier. Do not reopen FLEXIBLE prefill, storage placement/type or activation lock; those are resolved.

### A2-F4 — Approval guard uses a state explicitly removed by the same document (P2, partial)

- [modules/approval/backend md L187–189](../../../modules/approval/backend.md): Post must be PENDING or PAUSED FOR REVIEW
- [modules/approval/backend md L205–217](../../../modules/approval/backend.md): ACTIVE → PENDING; there is no separate PAUSED FOR REVIEW state

Scenario: A reader implementing reject eligibility from the endpoint guard invents a state the canonical workflow disallows.

Recommendation: Replace the stale guard with canonical PENDING eligibility. This is settled document drift; no founder decision is needed.

### A2-F5 — Old application requirements and regression assertions reintroduce a removed listing gate (P1, partial)

- [architecture/regression-branch-closure-spec md L43–43](../../../architecture/regression-branch-closure-spec.md): B4 requires TUTOR NO ACTIVE POST 403 without an ACTIVE tutor listing
- [modules/applications/backend md L51–54](../../../modules/applications/backend.md): Calling tutor must have an ACTIVE listing
- [product requirements document md L2013–2017](../../../PRD.md): NO published listing required; old TUTOR NO ACTIVE POST 403 guard removed; profile-completeness gate instead

Scenario: An implementer treats the old regression list as authoritative and blocks an otherwise complete tutor from applying.

Recommendation: Mark these pre-M18 assertions superseded and link the free-market requirement plus 012's canonical completeness predicate. No policy question remains about listing necessity.

### A2-F6 — Old never-share profile wording omits the approved matched-participant exception (P1, partial)

- [modules/profile/account-ui-changes md L28–30](../../../modules/profile/account-ui-changes.md): any counterparty view still expose no email/phone
- [architecture/int-090-batch-m18-funnel-completion-spec md L271–299](../../../architecture/int-090-batch-m18-funnel-completion-spec.md): ACTIVE tutor match reveals family contact; required mutual-visibility microcopy before it happens

Scenario: A user reads an absolute never-share assurance and then the assigned tutor receives the family's contact under the approved workflow.

Recommendation: Link the explicit ACTIVE-match participant exception and use the mandated mutual-visibility disclosure. Keep public/pre-match privacy; this is settled supersession, not a new disclosure proposal.

### A2-F7 — Simplified commission summaries must not erase first-session confirmation (P1, partial)

- [modules/commission/module-overview md L34–53](../../../modules/commission/module-overview.md): PENDING/ACTIVE/COMPLETED matching and fixed commission summary
- [product requirements document md L1058–1072](../../../PRD.md): Activation sends tutee confirmation, logs confirmation and escalates after seven days
- [product requirements document md L2034–2037](../../../PRD.md): Money path is EXISTING §8.3/AC-05/BR-010; reused verbatim
- [architecture/int-157-commission-first-month-locked-spec md L100–102](../../../architecture/int-157-commission-first-month-locked-spec.md): due-after-first-session trigger is unchanged (confirmed)

Scenario: A team reads the shortened module overview as the complete payment workflow and leaves out the tutee action and overdue handling.

Recommendation: Explicitly cross-reference the retained obligation and reconcile collection status versus confirmation/invoice state. This is partial specification coverage, not an inferred code defect or an open question about whether confirmation is required.

### A2-F8 — An advisory design review cannot establish universal tutor verification (P2, authority-gap)

- [uiux/frontend-design-review-2026-06-13.md L63–68](../../../uiux/frontend-design-review-2026-06-13.md): tutors are admin-vetted; recommend Verified by InTuition on every tutor card
- [product requirements document md L2013–2025](../../../PRD.md): Any tutor can apply without a published listing; identity is profile-canonical and completeness-gated
- [uiux/frontend-design-review-2026-06-13.md L90–96](../../../uiux/frontend-design-review-2026-06-13.md): Recommendations for quality/conversion, not correctness defects

Scenario: Every complete listing-less tutor receives a trust badge even though no verification eligibility/state is defined by this amendment.

Recommendation: Define what the badge certifies and its qualifying evidence before adopting it. The free-market model does not itself prove every tutor is vetted; this is an authority gap in an advisory proposal, not proof of false live copy.

## Read versus referenced coverage

All 38 inventory files were targeted-read as stated above. Parent and later-amendment documents below were read only at the relevant clauses cited, not exhaustively. No separate uninspected source is used as evidence.

- [014-interview-scheduling-ux-and-admin-visibility/design md](../../../014-interview-scheduling-ux-and-admin-visibility/design.md): selected relevant clauses, not full document
- [product requirements document md](../../../PRD.md): selected relevant clauses, not full document
- [012-tutor-apply-and-profile-ux/design md](../../../012-tutor-apply-and-profile-ux/design.md): selected relevant clauses, not full document
- [007-generic-scheduler-db-scheduler/spec md](../../../007-generic-scheduler-db-scheduler/spec.md): selected relevant clauses, not full document
- [012-tutor-apply-and-profile-ux/requirements md](../../../012-tutor-apply-and-profile-ux/requirements.md): selected relevant clauses, not full document
- [011-auth-robustness/spec md](../../../011-auth-robustness/spec.md): selected relevant clauses, not full document
- [004-auth-session-hardening-30min-ttl-and-refresh/requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md): selected relevant clauses, not full document
- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md): selected relevant clauses, not full document
- [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md): selected relevant clauses, not full document
- [018-flag-hygiene-seeder-and-fe-hygiene/spec md](../../../018-flag-hygiene-seeder-and-fe-hygiene/spec.md): selected relevant clauses, not full document
