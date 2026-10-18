# Spec alignment audit — 019–027

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

53 distinct named requirements across nine packages. This is a requirements → spec/design/task alignment review; no implementation or runtime verdicts. Anonymous limited filtering and agency receipt of tutor commission as PAID are retained user decisions.

| Package | Verdict | Requirements | Summary |
|---|---|---:|---|
| 019-admin-money-safety-and-analytics | conflict | 7 | Money confirmation, cohort-label clarity and queue improvements trace through inline design/tasks. The mobile-nav amendment weakens an earlier explicit drawer/a11y acceptance rule; period-diagnosis fixtures do not prove both required metrics. |
| 020-taxonomy-integrity | authority-gap | 3 | Band-level hard blocking and frontend mirror are coherent. Canonical additions remain expressly deferred in the changelog, while later documents describe them as shipped; the exact approved list/approval record is missing from this package. 025 expressly supersedes only finer Secondary scoping. |
| 021-tutee-request-flow | conflict | 3 | PENDING invitation/interview initiation and PENDING editing are explicit founder decisions. Contact-release copy conflicts with the preserved assignment gate, and the new invitation lifecycle does not reconcile acceptance before approval or rejection cascade tests. |
| 022-tutor-profile-and-fe-polish | aligned | 5 | Five small repairs have inline design/tasks and regressions. Student-count verification should be resolved against 009's explicit retained 6-or-more option, not treated as a new product decision. |
| 023-seed-data-variety | conflict | 2 | Seed diversity regresses to unspecified N/K despite 018 already setting measurable thresholds. Its suggested >90-day analytics fixture cannot distinguish Last30 from Last90. |
| 024-postfix-review-fixes | aligned | 6 | Six fixes are explicitly founder-approved; the planning-area decision resolves the earlier choice. Requirement, local design and per-REQ task mapping agree; later 025 owns the deferred finer taxonomy scope. |
| 025-taxonomy-stream-year-scoping | authority-gap | 7 | Seven REQs map well to detailed design/tasks, including the fourth enforcement surface and two mutually exclusive tiers. The package still demands canonical-map/data-model/hard-block approval despite HISTORICALLY MARKED IMPLEMENTED header. There is also a wrong E-Math example and an unsafe rollback description. |
| 026-s3s4-polish-batch | partial | 8 | The approved batch preserves existing flows and contains regressions for all eight items. Compound REQ-008 leaves '2 hours default reviewed' without a resulting behavior or checkable outcome. |
| 027-aws-production-deployment | authority-gap | 12 | Founder approval of lean Amazon Web Services topology is explicit and phase evidence distinguishes authored infrastructure from unapplied/cutover work. The referenced path currently contains an unrelated continuous integration plan; delegated Amazon Web Services architecture/Verify criteria are unavailable from that link. Remaining spec gaps concern drain, stream loss behavior and production acceptance. |

## 019-admin-money-safety-and-analytics

Money confirmation, cohort-label clarity and queue improvements trace through inline design/tasks. The mobile-nav amendment weakens an earlier explicit drawer/a11y acceptance rule; period-diagnosis fixtures do not prove both required metrics.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-001 | aligned | Adds Matching confirmation to the existing Commission confirmation contract; PAID retains user-confirmed meaning agency receipt of tutor commission. Cancellation and no-POST-until-confirm specified. | [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [017-admin-analytics-and-financial-safety/requirements md](../../../017-admin-analytics-and-financial-safety/requirements.md) |
| REQ-002 | partial | Diagnosis-first is explicitly chosen; fixture must include a 31–90-day post AND user plus recent records. >30 days without upper bound and post-only test cannot prove both Posts Created and New Signups. | [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [023-seed-data-variety/spec md](../../../023-seed-data-variety/spec.md) |
| REQ-003 | aligned | Label/display repair inherits the already pinned requested at cohort; no new choice of conversion formula is required. | [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [017-admin-analytics-and-financial-safety/requirements md](../../../017-admin-analytics-and-financial-safety/requirements.md) |
| REQ-004 | aligned | Inline action and populated Job column are named in acceptance, regression and task. | [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md) |
| REQ-005 | aligned | Pagination and status filtering appear in requirement/test/task. Page-size implementation choice is not a product conflict. | [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md) |
| REQ-006 | conflict | A scroll cue alone satisfies this spec but fails 015's explicit no-cramped-scroll-strip drawer/menu and focus/Escape acceptance. No explicit approval to weaken 015 found. | [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [015-mobile-responsive-fixes/requirements md](../../../015-mobile-responsive-fixes/requirements.md) |
| REQ-007 | aligned | Shared friendly level helper and row assertion are consistent with the taxonomy user experience requirement. | [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md) |

Stale bookkeeping / settled amendments:

- HISTORICALLY MARKED IMPLEMENTED frontmatter coexists with unchecked build/reviewer checklists and older draft changelog text. This is stale progress bookkeeping, not proof that work or approval did not occur.

## 020-taxonomy-integrity

Band-level hard blocking and frontend mirror are coherent. Canonical additions remain expressly deferred in the changelog, while later documents describe them as shipped; the exact approved list/approval record is missing from this package. 025 expressly supersedes only finer Secondary scoping.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-001 | aligned | Founder hard-block decision is explicit; N LEVEL widen is later folded into changelog. Read same-band acceptance as the original tier, superseded in part by 025 once its effective mapping is established. | [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md); [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md); [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md); [025-taxonomy-stream-year-scoping/spec md](../../../025-taxonomy-stream-year-scoping/spec.md) |
| REQ-002 | aligned | Picker narrowing, mirrored data and continuous integration drift test trace consistently; 025 extends the mirror to exact Secondary pairs. | [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md); [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md); [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md) |
| REQ-003 | authority-gap | Canonical list is to be founder-approved before seeding, but changelog says deferred and no actual canonical table/approval is attached. Later presence of data is delivery evidence, not approval. | [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md); [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md); [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md); [025-taxonomy-stream-year-scoping/spec md](../../../025-taxonomy-stream-year-scoping/spec.md); [review-2026-06-23-postfix/CONSOLIDATED md](../../../issues/review-2026-06-23-postfix/CONSOLIDATED.md) |

Stale bookkeeping / settled amendments:

- HISTORICALLY MARKED IMPLEMENTED frontmatter coexists with unchecked build/reviewer checklists and older draft changelog text. This is stale progress bookkeeping, not proof that work or approval did not occur.
- Header version 0.1 trails changelog 0.2; record scoped effective version.

## 021-tutee-request-flow

PENDING invitation/interview initiation and PENDING editing are explicit founder decisions. Contact-release copy conflicts with the preserved assignment gate, and the new invitation lifecycle does not reconcile acceptance before approval or rejection cascade tests.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-001 | conflict | Allow PENDING initiation is settled. 'Contact is shared once approved' promises an earlier reveal than M18 assignment. Rejection closure is specified but omitted from named test/task; acceptance while pending is not reconciled with older ACTIVE-only accept. | [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md); [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md); [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md); [specs/product requirements document md](../../../PRD.md); [architecture/int-044-job-invitations md](../../../architecture/int-044-job-invitations.md) |
| REQ-002 | aligned | Availability display reuses existing data transfer object/grid and component assertion; this does not authorize changing saved schedule encoding. | [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md); [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md); [009-tutee-post-taxonomy-and-ux/requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) |
| REQ-003 | aligned | PENDING editing and retaining PENDING/reapproval are explicit and mapped to the private implementation folder guards plus e2e. The older editable-state exclusion is superseded. | [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md); [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md); [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md) |

Stale bookkeeping / settled amendments:

- HISTORICALLY MARKED IMPLEMENTED frontmatter coexists with unchecked build/reviewer checklists and older draft changelog text. This is stale progress bookkeeping, not proof that work or approval did not occur.

## 022-tutor-profile-and-fe-polish

Five small repairs have inline design/tasks and regressions. Student-count verification should be resolved against 009's explicit retained 6-or-more option, not treated as a new product decision.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-001 | aligned | Completeness signals reuse authoritative backend fields, matching 012. Planning areas remain required under 024's explicit decision. | [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md); [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md); [012-tutor-apply-and-profile-ux/requirements md](../../../012-tutor-apply-and-profile-ux/requirements.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md) |
| REQ-002 | aligned | Initial qualification/experience prefill from profile is compatible with canonical-profile identity and independently editable listing overrides; it is not an ongoing sync job. | [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md); [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md); [specs/product requirements document md](../../../PRD.md) |
| REQ-003 | aligned | Human-readable availability label has direct unit acceptance/task. | [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md); [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md) |
| REQ-004 | aligned | Landing must state existing commission model; detailed wording inherits current policy including user-confirmed PAID meaning. A generic line-presence test is weaker than copy-content verification but no conflicting formula is introduced. | [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md); [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md) |
| REQ-005 | aligned | This is a drift check. 009 explicitly keeps 1–5 plus 6-or-more with exact 6–20 reveal; it never removed the 6-or-more dropdown option. | [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md); [022-tutor-profile-and-fe-polish/spec md](../../../022-tutor-profile-and-fe-polish/spec.md); [009-tutee-post-taxonomy-and-ux/requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) |

Stale bookkeeping / settled amendments:

- HISTORICALLY MARKED IMPLEMENTED frontmatter coexists with unchecked build/reviewer checklists and older draft changelog text. This is stale progress bookkeeping, not proof that work or approval did not occur.
- REQ-005's 'confirm intended' framing is settled by explicit 009 founder decision; retained 6-or-more is intentional.

## 023-seed-data-variety

Seed diversity regresses to unspecified N/K despite 018 already setting measurable thresholds. Its suggested >90-day analytics fixture cannot distinguish Last30 from Last90.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-001 | partial | Replace N/K placeholders or reference the retained 018 bar (8 subjects, 5 levels, 4 areas, max tuple share 20%). 'Keep encrypted personal information' is stale for email/phone because a founder directive explicitly changed those fields to plaintext; it is not an open policy question. | [023-seed-data-variety/spec md](../../../023-seed-data-variety/spec.md); [023-seed-data-variety/spec md](../../../023-seed-data-variety/spec.md); [018-flag-hygiene-seeder-and-fe-hygiene/requirements md](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md); [architecture/plaintext-email-phone md](../../../architecture/plaintext-email-phone.md) |
| REQ-002 | conflict | A >90-day row is outside both 30d and 90d windows. It cannot provide the required count difference; post and/or signup also cannot prove both metrics required by 019. | [023-seed-data-variety/spec md](../../../023-seed-data-variety/spec.md); [023-seed-data-variety/spec md](../../../023-seed-data-variety/spec.md); [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md) |

Stale bookkeeping / settled amendments:

- HISTORICALLY MARKED IMPLEMENTED frontmatter coexists with unchecked build/reviewer checklists and older draft changelog text. This is stale progress bookkeeping, not proof that work or approval did not occur.
- Encrypted email/phone wording is superseded by explicit plaintext-email-phone founder directive; do not present encryption as an undecided policy.

## 024-postfix-review-fixes

Six fixes are explicitly founder-approved; the planning-area decision resolves the earlier choice. Requirement, local design and per-REQ task mapping agree; later 025 owns the deferred finer taxonomy scope.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-001 | aligned | Uses public handle/post identity with server-side owner resolution, preserves PENDING initiation, 404 privacy and self-invite guard; e2e drives actual frontend contract. | [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md) |
| REQ-002 | aligned | Clearing hidden exact-count validation upon returning to 1–5 preserves 009's chosen control. | [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [009-tutee-post-taxonomy-and-ux/requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) |
| REQ-003 | aligned | Extends friendly incomplete-profile feedback to list card; broader 012 auth/role feedback remains applicable and is not repealed. | [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [012-tutor-apply-and-profile-ux/requirements md](../../../012-tutor-apply-and-profile-ux/requirements.md) |
| REQ-004 | aligned | Explicit founder REQUIRED ruling aligns completeness and Apply gate; no optional-planning-area decision remains. | [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md) |
| REQ-005 | aligned | Secondary band label correctly encompasses all named streams in internal taxonomy and has direct unit/task trace. | [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md) |
| REQ-006 | aligned | Friendly alias preserves readability of soft-deprecated data without restoring it to picker; requirement/test/task agree. | [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md); [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md) |

Stale bookkeeping / settled amendments:

- HISTORICALLY MARKED IMPLEMENTED frontmatter coexists with unchecked build/reviewer checklists and older draft changelog text. This is stale progress bookkeeping, not proof that work or approval did not occur.

## 025-taxonomy-stream-year-scoping

Seven REQs map well to detailed design/tasks, including the fourth enforcement surface and two mutually exclusive tiers. The package still demands canonical-map/data-model/hard-block approval despite HISTORICALLY MARKED IMPLEMENTED header. There is also a wrong E-Math example and an unsafe rollback description.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-001 | authority-gap | Four-surface hard rejection is fully traced, but approved exact mapping and stronger hard-block decision are still explicitly gated. Recover record rather than infer from HISTORICALLY MARKED IMPLEMENTED. | [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md); [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md); [025-taxonomy-stream-year-scoping/tasks md](../../../025-taxonomy-stream-year-scoping/tasks.md) |
| REQ-002 | conflict | No-orphan set rule matches design/tasks, but negative Gherkin falsely says E-Math S1 is rejected while Appendix A permits it. Overall mixed-set 422 remains correct because A-Math is orphaned. Approved map is also unlinked. | [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md); [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md); [025-taxonomy-stream-year-scoping/tasks md](../../../025-taxonomy-stream-year-scoping/tasks.md) |
| REQ-003 | partial | Requirement says a subject absent from SUBJECT ALLOWED LEVELS yields ALL levels; design uses band fallback for known non-junction subjects. Unknown subject must be distinguished from known band-only subject. Mapping approval also remains unlinked. | [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md); [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md); [025-taxonomy-stream-year-scoping/tasks md](../../../025-taxonomy-stream-year-scoping/tasks.md) |
| REQ-004 | aligned | New pair-keyed parser, non-empty guard, equality and deliberate one-side mutation align in all layers. Exact expected dataset remains subject to package approval gap. | [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md); [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md); [025-taxonomy-stream-year-scoping/tasks md](../../../025-taxonomy-stream-year-scoping/tasks.md) |
| REQ-005 | partial | Read-old/write-valid semantics and N-Level parity map to tests. Rollback drops a table while its described query fallback assumes that table exists but is empty; specify compatible deployment/rollback. | [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md); [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md); [025-taxonomy-stream-year-scoping/tasks md](../../../025-taxonomy-stream-year-scoping/tasks.md) |
| REQ-006 | aligned | Precise rows suppress legacy widening; fallback only for zero-row subjects. Requirement truth table, design and T2/T5 agree. | [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md); [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md); [025-taxonomy-stream-year-scoping/tasks md](../../../025-taxonomy-stream-year-scoping/tasks.md) |
| REQ-007 | aligned | Every Secondary option is scoped or explicitly allowed at band level; failing fixture and rationale-bearing allow-list appear in design and T8. | [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md); [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md); [025-taxonomy-stream-year-scoping/tasks md](../../../025-taxonomy-stream-year-scoping/tasks.md) |

Stale bookkeeping / settled amendments:

- Design still says all 3 surfaces; spec and tasks explicitly correct this to 4 including tutor registration. Use 4. Header/body approval gap is tracked separately because actual canonical mapping approval matters.

## 026-s3s4-polish-batch

The approved batch preserves existing flows and contains regressions for all eight items. Compound REQ-008 leaves '2 hours default reviewed' without a resulting behavior or checkable outcome.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-001 | aligned | Saved View and wizard Review both display duration/frequency; direct render check and mapped design/task. | [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md) |
| REQ-002 | aligned | Stepper follows actual reachable flow; removes misleading indicator without removing the required review/explicit-submit safety behavior. | [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md) |
| REQ-003 | aligned | Back then forward preserves entered fields; scenario directly tests the reported loss. | [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md) |
| REQ-004 | aligned | Display-only removal of PSLE suffix on lower-primary years is clear; factual rationale is not used to change taxonomy validity. | [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md) |
| REQ-005 | aligned | Humanised public handle is an allowed implementation without expanding personal information. The optional display-name alternative must still respect existing public data transfer object policy; no raw private identity authorization inferred. | [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md) |
| REQ-006 | aligned | Suffix or clear retry are explicitly permitted acceptable outcomes; test covers both, so implementation choice is not a missing product decision. | [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md) |
| REQ-007 | aligned | Displayed funnel counts must match pinned cohort rate; diagnosis-first retained. 017 already settles conversion definition, so do not reopen cohort policy. | [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [017-admin-analytics-and-financial-safety/requirements md](../../../017-admin-analytics-and-financial-safety/requirements.md) |
| REQ-008 | partial | Grid overlap, one-step registration and password a11y are concrete; '2 hours default reviewed' gives no resulting actor/viewport/default. Per-item component/responsive test cannot establish that undefined item. | [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md); [002-tutor-profile-and-onboarding-ux/requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) |

Stale bookkeeping / settled amendments:

- HISTORICALLY MARKED IMPLEMENTED frontmatter coexists with unchecked build/reviewer checklists and older draft changelog text. This is stale progress bookkeeping, not proof that work or approval did not occur.

## 027-aws-production-deployment

Founder approval of lean Amazon Web Services topology is explicit and phase evidence distinguishes authored infrastructure from unapplied/cutover work. The referenced path currently contains an unrelated continuous integration plan; delegated Amazon Web Services architecture/Verify criteria are unavailable from that link. Remaining spec gaps concern drain, stream loss behavior and production acceptance.

| ID | Verdict | Assessment | Evidence |
|---|---|---|---|
| REQ-A0-1 | aligned | Readiness excludes showcase while retaining real dependencies; positive and dependency-down acceptance agree with A0 reviewed evidence. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) |
| REQ-A0-2 | partial | Requirement drains in-flight work; test checks only no new polls after shutdown, which does not establish completion/requeue of already claimed messages. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md) |
| REQ-A0-3 | aligned | Secret-free public provenance, build SHA and arm64 are explicit; phase 2/4 evidence carries architecture through deployment. Ellipsis in local evidence is not treated as a proven response leak. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) |
| REQ-A0-4 | aligned | Same-image migration then non-web exit coheres with phase 4 Run Task-before-canary and expand-contract note. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) |
| REQ-A0-5 | aligned | Explicit production one-time passcode-log prohibition supersedes broad older logging for prod only; test plus cutover grep specified. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) |
| REQ-A0-6 | aligned | Trusted sidecar hop and untrusted forwarded-IP rejection preserve rate-limit integrity; tunnel-only infrastructure is consistent. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) |
| REQ-A0-7 | aligned | Production application programming interface docs disabled; explicit local evidence preserves non-prod docs without changing the production requirement. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) |
| REQ-A0-8 | partial | multi-factor authentication-before-admin and password-change bootstrap are clear and match business requirements document. Bootstrap secret deletion is required but no named acceptance/assertion covers deletion and retry/failure outcome. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [specs/business requirements document md](../../../BRD.md) |
| REQ-A0-9 | aligned | Prod connection limits/TLS config and phase 1/2 evidence agree; max-connection computation detail delegated to missing approved plan. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) |
| REQ-A0-10 | partial | Bounding is explicit, but a MAXLEN option can discard undelivered records after outbox SENT; no accepted loss/backpressure/replay policy reconciles bounded growth with at-least-once delivery. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [architecture/transactional-messaging-and-idempotency md](../../../architecture/transactional-messaging-and-idempotency.md); [architecture/transactional-messaging-and-idempotency md](../../../architecture/transactional-messaging-and-idempotency.md) |
| REQ-INFRA | authority-gap | Per-phase Verify table and full approved design are delegated to a source path that currently contains an unrelated continuous integration plan. Phase evidence exists and records actual decisions (14d PITR, manual-only pre-cutover deploy, SNS key fix); do not call authored infrastructure applied. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md); [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) |
| REQ-CUTOVER | partial | Specific happy-path, multi-factor authentication, real email, origin isolation and restore checks exist and remain pending. Parent latency/recovery/retention/notification measurements and cloud integration residency/transfer evidence are not tied to this cutover checklist; recover missing plan before declaring omission final. | [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md); [specs/business requirements document md](../../../BRD.md); [specs/product requirements document md](../../../PRD.md); [specs/product requirements document md](../../../PRD.md) |

Stale bookkeeping / settled amendments:

- Header version 0.1 trails 0.2 changelog; unchecked phase tasks trail recorded A0/0–5 deliverables. infrastructure as code home TBD at spec is resolved to intuition-spec/infra by phase-0 evidence. Manual-only deploy is explicit pre-cutover safety choice, not a contradiction of eventual push-trigger target.
- Phase-5 creates no log filters because Phase-4 owns them; this is not missing REQ-INFRA log filters. SNS key change has explicit reviewed rationale. Overview lists future Multi-AZ/replica options under 'open'; actual launch deferral is settled by spec.

## Paired-source findings

### A019-1 — Mobile-nav acceptance weakens the retained drawer requirement (medium; conflict)

Affected: 019/REQ-006.

- [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md): “Add a scroll affordance/hamburger.”
- [015-mobile-responsive-fixes/requirements md](../../../015-mobile-responsive-fixes/requirements.md): “all 8 admin sections are reachable in a usable drawer/menu (not a cramped scroll strip) ... focus-trapped and Escape-dismissible”

A horizontal strip with a cue can pass 019 but fail 015's explicit acceptance. No scoped approval to remove accessibility behavior was found.

Proposed repair: Carry 015's drawer/equivalent, focus trap and Escape assertions into the 019 regression. This is alignment repair, not a request to choose between known policies.

### A023-1 — Analytics seed is outside both compared windows (high; conflict)

Affected: 019/REQ-002, 023/REQ-002.

- [023-seed-data-variety/spec md](../../../023-seed-data-variety/spec.md): “outside the default analytics window (e g. >90 days ago) ... Last 30 days vs Last 90 days ... counts differ”
- [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md): “Posts Created (and New Signups) differ ... seed one post dated >30d ago + one recent”

A 100-day-old post produces the same contribution (zero) to both windows. A post-only fixture does not make New Signups differ. Treating unchanged counts as a code bug creates a false diagnosis.

Proposed repair: Use fixed-clock fixtures at 15 and 60 days for BOTH posts and users, plus a >90-day exclusion fixture. Assert each metric's exact expected counts.

### A023-2 — Diversity N/K placeholders replace an already measurable parent bar (medium; partial)

Affected: 023/REQ-001.

- [023-seed-data-variety/spec md](../../../023-seed-data-variety/spec.md): “no single card repeated >N times ... ≥K distinct subject+level combinations”
- [018-flag-hygiene-seeder-and-fe-hygiene/requirements md](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md): “≥ 8 distinct subject Code, ≥ 5 distinct level Code, ≥ 4 distinct planning-area ... no single ... tuple exceeds 20%”

023's test cannot be evaluated without N/K and does not prove area/budget variety; 018 already supplies a retained measurable requirement.

Proposed repair: Reference or copy the 018 thresholds and define the budget bucket; do not reopen numeric policy solely because later shorthand omitted it.

### A020-1 — Canonical taxonomy approval is not traceable (high; authority-gap)

Affected: 020/REQ-003, 025/REQ-001, 025/REQ-002, 025/REQ-003.

- [020-taxonomy-integrity/spec md](../../../020-taxonomy-integrity/spec.md): “REQ-003 (canonical SG list) DEFERRED pending founder approval.”
- [025-taxonomy-stream-year-scoping/spec md](../../../025-taxonomy-stream-year-scoping/spec.md): “spec 020 REQ-003 already added the SG set”
- [025-taxonomy-stream-year-scoping/spec md](../../../025-taxonomy-stream-year-scoping/spec.md): “mapping ... and the data-model choice ... PROPOSED, pending MOE / founder approval ... NOT ... approved fact”
- [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md): “Every row must be confirmed or corrected ... before any seed/metadata is written”

Later implementation reports and HISTORICALLY MARKED IMPLEMENTED frontmatter do not identify which data rows/model/rejection choice were approved. The issue is missing approval provenance, not proof that approval never happened.

Proposed repair: Recover exact approval and effective map in a versioned repo decision. Preserve 020's explicit band hard-block; 025 states a separate finer-rule approval gate.

### A021-1 — PENDING invitation copy overpromises contact release (high; conflict)

Affected: 021/REQ-001.

- [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md): “keep contact-reveal gated on approval ... contact is shared once approved”
- [specs/product requirements document md](../../../PRD.md): “personal information contract (binding ...): Child's name + parent's phone are hidden from applicants until the parent commits (assignment).”
- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md): “contact reveal stays a post-match concern.”

An approved but unmatched job does not meet the binding contact-release gate. Founder approval of PENDING initiation does not explicitly authorize earlier disclosure.

Proposed repair: State that invitations/interviews may begin during review and contact is released after assignment; retain participant/status authorization checks.

### A021-2 — New PENDING invitation lifecycle has no acceptance-before-approval contract (medium; partial)

Affected: 021/REQ-001.

- [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md): “PENDING parent may invite ... If the post is later REJECTED ... pending invite/interview ... voided/closed (soft)”
- [architecture/int-044-job-invitations md](../../../architecture/int-044-job-invitations.md): “assert job ACTIVE (422) ... Reuse Job Application Service apply”

Initiation is settled, but receiving a PENDING invitation may still hit the older ACTIVE-only accept/application contract. Rejection cleanup is required without being included in 021's named regression/task.

Proposed repair: Document the recipient's pre-approval state and test it. Proposal: allow response recording while keeping assignment/contact on their retained gates, or clearly defer acceptance until approval; record actual product choice. Add rejection cascade, notification and no-contact assertions.

### A025-1 — E-Math Sec1 example contradicts the proposed map (medium; conflict)

Affected: 025/REQ-002.

- [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md): “E-Math at Sec 1 also rejected per Appendix A”
- [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md): “E-Math (EMATH OLEVEL) | S1–S4 | S1–S5”

The mixed A-Math/E-Math at S1 set is correctly rejected because A-Math has no partner; the explanation wrongly classifies E-Math. A test copied from it could assert the wrong single-subject rule.

Proposed repair: Remove the incorrect E-Math rationale and test E-Math S1 alone against the effective approved map.

### A025-2 — Unknown-subject test conflates absent precise-map entry with absent taxonomy subject (medium; conflict)

Affected: 025/REQ-003.

- [025-taxonomy-stream-year-scoping/requirements md](../../../025-taxonomy-stream-year-scoping/requirements.md): “Given a subject code absent from SUBJECT ALLOWED LEVELS ... picker shows ALL levels”
- [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md): “if the subject is a key ... precise set; else keep the existing level group-filter + N LEVEL FALLBACK GROUP path”

Every known non-Secondary subject intentionally lacks precise-map rows and should retain its band. Requirement scenario instead permits all levels.

Proposed repair: Define unknown as absent from both canonical taxonomy and precise map. Add separate known band-only and completely unknown test cases.

### A025-3 — Rollback drops the relation required by the described fallback (high; conflict)

Affected: 025/REQ-005.

- [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md): “If the subject has ANY subject allowed levels rows ... Else ... existing band-level join”
- [025-taxonomy-stream-year-scoping/design md](../../../025-taxonomy-stream-year-scoping/design.md): “rollback drops subject allowed levels ... no code change required ... no-op on an empty table”
- [025-taxonomy-stream-year-scoping/tasks md](../../../025-taxonomy-stream-year-scoping/tasks.md): “The rollback drops the table.”

An absent database query language relation is not an empty table; the described predicate cannot rely on an EXISTS query against a dropped table. This is a design contradiction, not a runtime observation.

Proposed repair: Specify schema-compatible rollback ordering (old app before dropping relation), or retain an empty relation under an explicit rollback mode. Add a rollback acceptance scenario.

### A026-1 — Compound polish requirement has an untestable review-only item (low; partial)

Affected: 026/REQ-008.

- [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md): “the '2 hours' default reviewed ... Test: per-item component/responsive assertions.”
- [002-tutor-profile-and-onboarding-ux/requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md): “2-hour granularity is the DEFAULT at the @390 mobile breakpoint”

Reviewing a default gives no result to assert. 002 already pins tutor mobile behavior; 026's request-wizard actor is not explicitly distinguished.

Proposed repair: Record actor, viewport, chosen default and unchanged option availability. Do not reopen tutor mobile 2-hour default merely because this later batch says reviewed.

### A027-1 — Referenced Amazon Web Services plan link currently contains an unrelated continuous integration plan (high; authority-gap)

Affected: 027/REQ-INFRA, 027/REQ-CUTOVER.

- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “source: [REDACTED LOCAL PATH] (founder-approved plan, dual-reviewed)”
- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “per-phase verification in the plan's sequencing table ... full architecture ... review traceability”
- [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md): “Fix red continuous integration on backend + frontend (Boss-led) ... separate from ... Amazon Web Services deployment work.”

The referenced path currently contains an unrelated continuous integration plan; delegated Amazon Web Services review premises, canary mechanics and Verify criteria are unavailable from that link.

Proposed repair: Restore the approved Amazon Web Services v3 plan into version control and point 027 at that immutable artifact. Approval of lean topology remains explicit at spec,135.

### A027-2 — Bounded stream option lacks delivery-loss acceptance (high; partial)

Affected: 027/REQ-A0-10.

- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “XADD get a capped MAXLEN ~ N (or a reaper) ... length is bounded ... without consumption”
- [architecture/transactional-messaging-and-idempotency md](../../../architecture/transactional-messaging-and-idempotency.md): “outbox row reaches SENT only after the XADD lands”
- [architecture/transactional-messaging-and-idempotency md](../../../architecture/transactional-messaging-and-idempotency.md): “ACK only after the side-effect succeeds (at-least-once) ... never before ... silent loss”

Length-only trimming can delete queued, unconsumed messages after the durable outbox has ceased retrying them. Memory bounds alone do not establish at-least-once delivery or an explicitly accepted loss policy.

Proposed repair: Specify age/expiry-aware trimming of completed records, backpressure and recoverability for undelivered work. Test saturated queue with consumer outage and recovery; expired one-time passcode may be terminal, but critical nonexpired mail needs an explicit outcome.

### A027-3 — Production cutover checklist lacks a measurable link to retained NFRs (medium; partial)

Affected: 027/REQ-CUTOVER, 027/REQ-INFRA.

- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “end-to-end smoke ... happy-path ... origin unreachable ... one-time passcode log-grep zero”
- [specs/business requirements document md](../../../BRD.md): “RPO ... 24 hours ... RTO ... 4 hours ... Backup retention ... 30 days rolling ... Quarterly ... full restore”
- [specs/product requirements document md](../../../PRD.md): “All user-facing application programming interface endpoints ... <300ms ... <1,500ms ... measured at the server”

A happy-path smoke and PITR drill do not establish workload, latency or recovered data quality. 027 explicitly chooses 7–14d PITR but does not say whether this replaces all 30d backup retention or supplements snapshots. Missing source plan could contain the details.

Proposed repair: Recover the plan first, then attach exact pass criteria for restored data/RPO/RTO, retention by backup class, representative load/latency and message delivery. Preserve explicitly approved Single-AZ/one-node topology; do not demand Multi-AZ by inference.

### A027-4 — Residency assertions are broader than documented integration acceptance (medium; partial)

Affected: 027/REQ-CUTOVER.

- [027-aws-production-deployment/Amazon Web Services-OVERVIEW md](../../../027-aws-production-deployment/AWS-OVERVIEW.md): “Cloudflare ... global ... Amazon Web Services in Singapore (so personal data stays in Singapore)”
- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “Resend custom domain ... real @example com email”
- [specs/business requirements document md](../../../BRD.md): “Cross-border transfer of any data category ... third-party integrations ... requires a written determination ... prior to implementation.”
- [specs/product requirements document md](../../../PRD.md): “Singapore data residency OR ... signed Data Processing Agreement ... delivery tracking”

The documents choose Cloudflare/Resend but do not link internal transfer/provider acceptance. Locating the database in Singapore alone is not the complete internal data-processing requirement. No claim about actual provider legal compliance is made.

Proposed repair: Attach data-flow scope and the existing internal provider/transfer determination if present; include provider delivery-event acceptance in cutover. Do not change the chosen stack solely from this documentation gap.

## Files read

- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — full text
- [[REDACTED LOCAL PATH]) — targeted,1756–1814
- [[REDACTED LOCAL PATH]) — targeted
- [[REDACTED LOCAL PATH]) — targeted
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — targeted
- [[REDACTED LOCAL PATH]) — targeted
- [[REDACTED LOCAL PATH]) — targeted
- [[REDACTED LOCAL PATH]) — targeted
- [[REDACTED LOCAL PATH]) — targeted
- [[REDACTED LOCAL PATH]) — targeted
- [[REDACTED LOCAL PATH]) — targeted; other ACTIVE rules searched
- [[REDACTED LOCAL PATH]) — targeted; guards searched
- [[REDACTED LOCAL PATH]) — targeted,199–214
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full; referenced path currently contains an unrelated continuous integration plan

## Referenced, not read

- [REDACTED LOCAL PATH] — Non-normative rendered companion; Markdown source read.
- [REDACTED LOCAL PATH] — Non-normative rendered companion; Markdown source read.
- [REDACTED LOCAL PATH] — Not visually reviewed; text architecture and phase records reviewed.
- [REDACTED LOCAL PATH] — Not visually reviewed; cost Markdown reviewed.

## Limits

- Approval searches in spec Markdown, source review summaries, CLAUDE and plan found no versioned 020/025 canonical-map approval; not proof no approval occurred elsewhere.
- Current source-code line references inside old specs were treated as historical and not used as current implementation evidence.
- External provider prices, product-plan availability, legal compliance and curriculum accuracy were not independently researched; only internal agreement reviewed.
