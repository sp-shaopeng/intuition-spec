# Requirements/spec alignment — 010–018

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Numbered specs 010–018; requirements→design→tasks and parent/later amendment alignment. Documentation-only review; no code/runtime verification.

Status frontmatter, DRAFT prose and unchecked task/signoff tables often disagree. Folded dated review decisions/changelogs are considered over stale boilerplate. HISTORICALLY MARKED IMPLEMENTED here is a document label, not independently verified delivery. No absence of evidence is treated as proof of missing implementation.

User decisions preserved: Anonymous visitors may filter limited public results. PAID means the agency received tutor commission; separate from parent lesson payment.

## 010 — conflict

Additive trace carriers and frontend references are coherently traced, but the body/header invariant conflicts with preservation of an existing unrelated trace Id. Security error-body coverage remains an acknowledged follow-up.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | partial | [010-traceid-on-every-response/requirements md L43–85](../../../010-traceid-on-every-response/requirements.md); [010-traceid-on-every-response/design md L69–90](../../../010-traceid-on-every-response/design.md); [010-traceid-on-every-response/tasks md L27](../../../010-traceid-on-every-response/tasks.md) | Every GEH error is designed and tasked; 'all error bodies' is broader than the acknowledged 401/403 security-entrypoint follow-up. Header fallback remains intentional. |
| REQ-002 | conflict | [010-traceid-on-every-response/requirements md L87–146](../../../010-traceid-on-every-response/requirements.md); [010-traceid-on-every-response/design md L92–167](../../../010-traceid-on-every-response/design.md); [010-traceid-on-every-response/tasks md L28–30](../../../010-traceid-on-every-response/tasks.md) | Object/no-object rules trace through design and tasks. A pre-existing body trace Id must be preserved, without any rule requiring it to equal this request's correlation id. |
| REQ-003 | conflict | [010-traceid-on-every-response/requirements md L148–186](../../../010-traceid-on-every-response/requirements.md); [010-traceid-on-every-response/design md L169–174](../../../010-traceid-on-every-response/design.md); [010-traceid-on-every-response/tasks md L26–30](../../../010-traceid-on-every-response/tasks.md) | Universal header/cross-origin request permissions carriage is traced; unconditional body is header equality cannot hold for the preserved-collision case in REQ-002. |
| REQ-004 | aligned | [010-traceid-on-every-response/requirements md L188–231](../../../010-traceid-on-every-response/requirements.md); [010-traceid-on-every-response/design md L176–192](../../../010-traceid-on-every-response/design.md); [010-traceid-on-every-response/tasks md L36–38](../../../010-traceid-on-every-response/tasks.md) | Body-first/header-fallback capture, copyable errors-only reference and no-response omission agree across requirements, design and tasks. |

### 010-F1 — The broad every-error-body promise is narrowed only as an acknowledged follow-up (P2, partial)

- [010-traceid-on-every-response/requirements md L43–56](../../../010-traceid-on-every-response/requirements.md): trace Id in every ERROR response body
- [010-traceid-on-every-response/spec md L65–67](../../../010-traceid-on-every-response/spec.md): Known gap (follow-up): security 401/403 entry-point responses carry the header but not a body trace Id (bypass GEH).

REQ-001's actual EARS is GEH-scoped. Track the security path as explicit unfinished coverage or record an approved header-only exception; do not claim every JSON error is covered.

### 010-F2 — Existing trace Id preservation contradicts universal body/header equality (P2, conflict)

- [010-traceid-on-every-response/requirements md L103–105](../../../010-traceid-on-every-response/requirements.md): IF a body unexpectedly already contains a top-level trace Id, the advice SHALL NOT overwrite it
- [010-traceid-on-every-response/requirements md L156–160](../../../010-traceid-on-every-response/requirements.md): The value SHALL equal the request's logging context correlation Id and SHALL equal any trace Id placed in the body.

Scenario: endpoint supplies an unrelated trace Id. Preserve it and equality fails. Reserve the field for the request id, or explicitly document a collision exception and frontend precedence.

## 011 — conflict

Auth behavior is well scoped and the duplicate-phone override is explicitly settled. Logout design contradicts the retained HttpOnly cookie contract; its task checklist also misses folded technical details.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | conflict | [011-auth-robustness/requirements md L33–79](../../../011-auth-robustness/requirements.md); [011-auth-robustness/design md L6–42](../../../011-auth-robustness/design.md); [011-auth-robustness/tasks md L19–20](../../../011-auth-robustness/tasks.md) | Logout ordering/revocation goal agrees with 004, but 011 assumes the auth cookie is JS-readable and mandates client-side expiry; 004 preserves HttpOnly. This is a design-contract inconsistency, not evidence of runtime failure. |
| REQ-002 | aligned | [011-auth-robustness/requirements md L81–111](../../../011-auth-robustness/requirements.md); [011-auth-robustness/design md L44–49](../../../011-auth-robustness/design.md); [011-auth-robustness/tasks md L21](../../../011-auth-robustness/tasks.md) | Specific autocomplete tokens, explicit submit and form-set audit agree. Forced-change path enumeration remains a nonblocking review note. |
| REQ-003 | aligned | [011-auth-robustness/requirements md L113–139](../../../011-auth-robustness/requirements.md); [011-auth-robustness/design md L51–55](../../../011-auth-robustness/design.md); [011-auth-robustness/tasks md L22](../../../011-auth-robustness/tasks.md) | Confirmed existing notification poll/auth Redirect guards are regression requirements, not new implementation demands. |
| REQ-004 | partial | [011-auth-robustness/requirements md L141–211](../../../011-auth-robustness/requirements.md); [011-auth-robustness/design md L57–130](../../../011-auth-robustness/design.md); [011-auth-robustness/tasks md L23–25](../../../011-auth-robustness/tasks.md) | Founder override settles 409 PHONE ALREADY REGISTERED, one bcrypt per pre-insert branch, rate-limit-before-membership and metric retention. Task 6b still asks for indistinguishable latency instead of the deterministic spy test. |

### 011-F1 — Logout design requires JavaScript to clear an HttpOnly auth cookie (P1, conflict)

- [011-auth-robustness/design md L29–34](../../../011-auth-robustness/design.md): The frontend-side clear covers the JS-readable auth cookie.
- [004-auth-session-hardening-30min-ttl-and-refresh/requirements md L10–12](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md): the auth cookie is already HttpOnly ✓ ... Those four are out of scope.

Preserve HttpOnly; express cookie neutralization through awaited server Set-Cookie/revocation. WAVE2 review requires ordering, not removal of HttpOnly. Document the failure-of-logout guarantee separately rather than claiming local JS clears the server credential.

### 011-F2 — Registration hardening task still requests the superseded timing measurement (P2, partial)

- [011-auth-robustness/requirements md L165–172](../../../011-auth-robustness/requirements.md): This is asserted by a DETERMINISTIC spy test ... NOT a wall-clock timing measurement.
- [011-auth-robustness/tasks md L24](../../../011-auth-robustness/tasks.md): rate-limit holds; latency indistinguishable; sign-off recorded

Fold v1.2 into task 6b: one bcrypt on each conflict/success branch, rate-limit-before-membership, deterministic spies. Policy is settled.

## 012 — conflict

Apply-gate and taxonomy amendments are coherent once folded decisions are followed. Contact-change requirements/design use the founder's email fallback, while the actionable task plan still directs the superseded WhatsApp/pending-only behavior.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | aligned | [012-tutor-apply-and-profile-ux/requirements md L24–72](../../../012-tutor-apply-and-profile-ux/requirements.md); [012-tutor-apply-and-profile-ux/design md L5–36](../../../012-tutor-apply-and-profile-ux/design.md); [012-tutor-apply-and-profile-ux/tasks md L16–24](../../../012-tutor-apply-and-profile-ux/tasks.md) | Effective requirement is a server-computed completeness/missing Fields signal from one predicate; v1.1 expressly rejects the alternatives still mentioned in requirements. 024 reinforces planning areas and extends list-card feedback. |
| REQ-002 | aligned | [012-tutor-apply-and-profile-ux/requirements md L74–99](../../../012-tutor-apply-and-profile-ux/requirements.md); [012-tutor-apply-and-profile-ux/design md L38–43](../../../012-tutor-apply-and-profile-ux/design.md); [012-tutor-apply-and-profile-ux/tasks md L25](../../../012-tutor-apply-and-profile-ux/tasks.md) | Sticky rail mirrors applied/gated/inactive state. Mechanism is a routine user interface decision, not an unresolved product policy. |
| REQ-003 | aligned | [012-tutor-apply-and-profile-ux/requirements md L101–118](../../../012-tutor-apply-and-profile-ux/requirements.md); [012-tutor-apply-and-profile-ux/design md L45–47](../../../012-tutor-apply-and-profile-ux/design.md); [012-tutor-apply-and-profile-ux/tasks md L26](../../../012-tutor-apply-and-profile-ux/tasks.md) | Shared taxonomy confirmation is traced; later 024/025/026 taxonomy amendments should be consumed through the shared options. |
| REQ-004 | conflict | [012-tutor-apply-and-profile-ux/requirements md L120–227](../../../012-tutor-apply-and-profile-ux/requirements.md); [012-tutor-apply-and-profile-ux/design md L49–142](../../../012-tutor-apply-and-profile-ux/design.md); [012-tutor-apply-and-profile-ux/tasks md L27–37](../../../012-tutor-apply-and-profile-ux/tasks.md) | v1.3 explicitly chooses email-link confirmation and old-value preservation. Tasks 8b–8d and live evidence still use WhatsApp or pending-only. Revocation promised in requirements is not given a route/state transition in design. |

### 012-F1 — Contact-change tasks instruct the explicitly superseded WhatsApp/pending-only flow (P1, conflict)

- [012-tutor-apply-and-profile-ux/requirements md L123–126](../../../012-tutor-apply-and-profile-ux/requirements.md): editing phone routes a confirmation link to the user's ALREADY-VERIFIED ACCOUNT EMAIL ... NOT defer, NOT WhatsApp/one-time passcode.
- [012-tutor-apply-and-profile-ux/tasks md L28–30](../../../012-tutor-apply-and-profile-ux/tasks.md): phone via INT-082 if enabled, else pending-only

Rewrite tasks 1,8b,8c,8d,10 around the founder email-link decision, landing-page explicit POST and required safeguards. This is a documentation correction, not a new founder question.

### 012-F2 — The promised revoke affordance lacks a revocation contract (P1, partial)

- [012-tutor-apply-and-profile-ux/requirements md L154–156](../../../012-tutor-apply-and-profile-ux/requirements.md): notify BOTH the OLD email AND the OLD phone ... including a 'you didn't do this? revoke' affordance, BEFORE any commit
- [012-tutor-apply-and-profile-ux/design md L124–127](../../../012-tutor-apply-and-profile-ux/design.md): Two named routes ... an initiate route ... and a verify route

Design lists PENDING/CONSUMED/EXPIRED/SUPERSEDED (line73) but no revoke operation, state mapping, permission/token rule or revoke-vs-confirm race. Specify these so the mandatory owner-stop behavior is independently buildable/testable; no claim about current code.

## 013 — partial

Durable outbound views and venue display are well traced. PENDING editing is explicitly amended by 021; Revision Requested still lacks a reconciled product scope. The outbound interview acceptance clause is weaker than its decided list design.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | authority-gap | [013-tutee-dashboard-and-invitations/requirements md L19–63](../../../013-tutee-dashboard-and-invitations/requirements.md); [013-tutee-dashboard-and-invitations/design md L5–45](../../../013-tutee-dashboard-and-invitations/design.md); [013-tutee-dashboard-and-invitations/tasks md L22–30](../../../013-tutee-dashboard-and-invitations/tasks.md) | 021 expressly supersedes the old PENDING view-only rule, so that is settled. But product requirements document requires editing REVISION REQUESTED posts and 013 excludes it solely 'until INT-098' based on code state; no product waiver appears here. |
| REQ-002 | aligned | [013-tutee-dashboard-and-invitations/requirements md L65–82](../../../013-tutee-dashboard-and-invitations/requirements.md); [013-tutee-dashboard-and-invitations/design md L47–51](../../../013-tutee-dashboard-and-invitations/design.md); [013-tutee-dashboard-and-invitations/tasks md L31](../../../013-tutee-dashboard-and-invitations/tasks.md) | All venues represented in summary and fully shown in detail; later 021/026 add schedule, duration/frequency detail requirements. |
| REQ-003 | aligned | [013-tutee-dashboard-and-invitations/requirements md L84–120](../../../013-tutee-dashboard-and-invitations/requirements.md); [013-tutee-dashboard-and-invitations/design md L53–110](../../../013-tutee-dashboard-and-invitations/design.md); [013-tutee-dashboard-and-invitations/tasks md L18–32](../../../013-tutee-dashboard-and-invitations/tasks.md) | Own-only server list, status/history, persisted withdraw and personal information-safe public-handle projection are traced. 024 changes creation identity resolution, not this list projection. |
| REQ-004 | partial | [013-tutee-dashboard-and-invitations/requirements md L122–145](../../../013-tutee-dashboard-and-invitations/requirements.md); [013-tutee-dashboard-and-invitations/design md L112–149](../../../013-tutee-dashboard-and-invitations/design.md); [013-tutee-dashboard-and-invitations/tasks md L33](../../../013-tutee-dashboard-and-invitations/tasks.md) | The decided design/tasks require listed invitation and interview rows in one dedicated section. Gherkin still allows only a link/count, which would not prove the decided requirement. |

### 013-F1 — Revision-requested editing has no explicit requirements waiver (P2, authority-gap)

- [specs/product requirements document md L229–230](../../../PRD.md): Tutee: edit own post (when in Draft or Revision Requested state)
- [013-tutee-dashboard-and-invitations/design md L33](../../../013-tutee-dashboard-and-invitations/design.md): REVISION REQUESTED ... intentionally excluded until INT-098 lands its revise route

The code-state explanation is a temporary delivery gap, not product supersession. Record the supported revision path/release boundary. PENDING editing is separately and explicitly settled by 021:41–47.

### 013-F2 — Outbound request acceptance still permits a count/link despite the decided listed-row requirement (P2, partial)

- [013-tutee-dashboard-and-invitations/requirements md L137–141](../../../013-tutee-dashboard-and-invitations/requirements.md): Then the outbound interview request is visible or linked (count/section)
- [013-tutee-dashboard-and-invitations/design md L138–141](../../../013-tutee-dashboard-and-invitations/design.md): Counts-only or a bare link still leaves the tutee guessing; a labelled section that LISTS the items with their status is the testable acceptance target

Tighten Gherkin to require the two groups and identifiable rows/status/date/link; the decision itself is already settled and tasks agree.

## 014 — conflict

Admin queue, accepted-slot prefill, responsive actions and admin-only identity are coherent. The all-role picker design incorrectly generalizes the admin's Local Date Time wire format to participant proposal application programming interfaces.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | aligned | [014-interview-scheduling-ux-and-admin-visibility/requirements md L21–57](../../../014-interview-scheduling-ux-and-admin-visibility/requirements.md); [014-interview-scheduling-ux-and-admin-visibility/design md L5–24](../../../014-interview-scheduling-ux-and-admin-visibility/design.md); [014-interview-scheduling-ux-and-admin-visibility/tasks md L17](../../../014-interview-scheduling-ux-and-admin-visibility/tasks.md) | Every current enum is placed; later 019 adds inline scheduling/job labels. TUTOR DECLINED remains terminal despite the design table calling the action-tab group non-terminal; tab placement itself is explicitly approved. |
| REQ-002 | aligned | [014-interview-scheduling-ux-and-admin-visibility/requirements md L59–84](../../../014-interview-scheduling-ux-and-admin-visibility/requirements.md); [014-interview-scheduling-ux-and-admin-visibility/design md L26–32](../../../014-interview-scheduling-ux-and-admin-visibility/design.md); [014-interview-scheduling-ux-and-admin-visibility/tasks md L18](../../../014-interview-scheduling-ux-and-admin-visibility/tasks.md) | Accepted-slot prefill/empty guard is frontend-only, with editable override consistent with INT-039. 'Or add it' in required-evidence prose is obsolete, not new backend scope. |
| REQ-003 | aligned | [014-interview-scheduling-ux-and-admin-visibility/requirements md L86–103](../../../014-interview-scheduling-ux-and-admin-visibility/requirements.md); [014-interview-scheduling-ux-and-admin-visibility/design md L34–37](../../../014-interview-scheduling-ux-and-admin-visibility/design.md); [014-interview-scheduling-ux-and-admin-visibility/tasks md L19](../../../014-interview-scheduling-ux-and-admin-visibility/tasks.md) | All interview actions remain reachable at desktop and 390px; consistent strengthening of the business requirements document admin minimum viewport. |
| REQ-004 | conflict | [014-interview-scheduling-ux-and-admin-visibility/requirements md L105–130](../../../014-interview-scheduling-ux-and-admin-visibility/requirements.md); [014-interview-scheduling-ux-and-admin-visibility/design md L39–52](../../../014-interview-scheduling-ux-and-admin-visibility/design.md); [014-interview-scheduling-ux-and-admin-visibility/tasks md L20](../../../014-interview-scheduling-ux-and-admin-visibility/tasks.md) | Requirement preserves each endpoint's wire format. Design asserts Local Date Time/no-offset/no-Z then applies it to admin and proposers, whereas INT-039 proposal slots are ISO instants with Z and Coordinated Universal Time grid validation. |
| REQ-005 | aligned | [014-interview-scheduling-ux-and-admin-visibility/requirements md L132–159](../../../014-interview-scheduling-ux-and-admin-visibility/requirements.md); [014-interview-scheduling-ux-and-admin-visibility/design md L54–65](../../../014-interview-scheduling-ux-and-admin-visibility/design.md); [014-interview-scheduling-ux-and-admin-visibility/tasks md L21](../../../014-interview-scheduling-ux-and-admin-visibility/tasks.md) | Design explicitly confines display-name rendering to admin screens and excludes the tutee personal information surface; do not misread the broad heading as permission to reveal names to parents. |

### 014-F1 — All-role picker design conflates two existing datetime wire contracts (P1, conflict)

- [014-interview-scheduling-ux-and-admin-visibility/design md L41–51](../../../014-interview-scheduling-ux-and-admin-visibility/design.md): ISO-8601-LOCAL string with NO timezone offset and NO Z suffix ... Applies to admin schedule + tutee/tutor slot proposal controls.
- [architecture/int-039-interview-negotiation md L371–392](../../../architecture/int-039-interview-negotiation.md): "slots": [ "2026-06-05T14:00:00Z" ] ... each on a 15-min Coordinated Universal Time boundary

Keep admin scheduled At Local Date Time; separately preserve offset/instant serialization for propose/counter. Requirement014:113 says wire formats must stay unchanged; pin a payload example/test for each route family.

## 015 — aligned

All three responsive requirements have matching designs/tasks; the exact grid ancestor and actual availability-step check are folded. Later mobile requirements extend coverage rather than revoke these goals.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | aligned | [015-mobile-responsive-fixes/requirements md L17–45](../../../015-mobile-responsive-fixes/requirements.md); [015-mobile-responsive-fixes/design md L5–29](../../../015-mobile-responsive-fixes/design.md); [015-mobile-responsive-fixes/tasks md L15–18](../../../015-mobile-responsive-fixes/tasks.md) | 390px page containment with internal grid scrolling, unchanged bitmask/sticky column, and correct wizard-step proof are traced. This satisfies the scoped part of business requirements document mobile usability. |
| REQ-002 | aligned | [015-mobile-responsive-fixes/requirements md L47–66](../../../015-mobile-responsive-fixes/requirements.md); [015-mobile-responsive-fixes/design md L31–35](../../../015-mobile-responsive-fixes/design.md); [015-mobile-responsive-fixes/tasks md L16](../../../015-mobile-responsive-fixes/tasks.md) | Reachable admin sections, focus handling and Escape are consistent. Drawer vs collapsing sidebar is a permitted implementation choice; 019 later permits a scroll affordance/hamburger while preserving reachability. |
| REQ-003 | aligned | [015-mobile-responsive-fixes/requirements md L68–88](../../../015-mobile-responsive-fixes/requirements.md); [015-mobile-responsive-fixes/design md L37–41](../../../015-mobile-responsive-fixes/design.md); [015-mobile-responsive-fixes/tasks md L17](../../../015-mobile-responsive-fixes/tasks.md) | Single route-independent breakpoint policy and reachable navigation are traced; exact breakpoint is an implementation choice, not a missing business decision. |

## 016 — conflict

404 handling, field errors and contact-link normalization are traced. Error fallback policy is internally inconsistent: the EARS/review allow server-message fallback while the guard requires a safe generic message.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | conflict | [016-error-copy-and-handling/requirements md L19–46](../../../016-error-copy-and-handling/requirements.md); [016-error-copy-and-handling/design md L5–11](../../../016-error-copy-and-handling/design.md); [016-error-copy-and-handling/tasks md L16–24](../../../016-error-copy-and-handling/tasks.md) | Mapped friendly copy and trace reference are settled. Unmapped behavior conflicts: server message fallback vs safe generic/no raw unique identifier. backend scrubbing is explicitly optional in the folded Wave-2 review, leaving the SHALL-scrub requirement stale rather than an open product decision. |
| REQ-002 | aligned | [016-error-copy-and-handling/requirements md L48–73](../../../016-error-copy-and-handling/requirements.md); [016-error-copy-and-handling/design md L13–19](../../../016-error-copy-and-handling/design.md); [016-error-copy-and-handling/tasks md L15–18](../../../016-error-copy-and-handling/tasks.md) | Missing handler→404, preserved 405/trace/header and SPA/static guard are traced; no runtime behavior inferred. |
| REQ-003 | aligned | [016-error-copy-and-handling/requirements md L75–97](../../../016-error-copy-and-handling/requirements.md); [016-error-copy-and-handling/design md L21–24](../../../016-error-copy-and-handling/design.md); [016-error-copy-and-handling/tasks md L17–25](../../../016-error-copy-and-handling/tasks.md) | Field messages and no-field Errors generic fallback agree; tasks concentrate on per-field rendering, design supplies fallback context. |
| REQ-004 | aligned | [016-error-copy-and-handling/requirements md L99–120](../../../016-error-copy-and-handling/requirements.md); [016-error-copy-and-handling/design md L26–29](../../../016-error-copy-and-handling/design.md); [016-error-copy-and-handling/tasks md L26](../../../016-error-copy-and-handling/tasks.md) | Shared safe href normalization/encoding with unchanged readable text is consistent and does not authorize extra contact disclosure. |

### 016-F1 — Unmapped error fallback has two incompatible acceptance rules (P2, conflict)

- [015-mobile-responsive-fixes/requirements md L24–29](../../../015-mobile-responsive-fixes/requirements.md): only fall back to the server message when no mapping exists
- [015-mobile-responsive-fixes/requirements md L39–41](../../../015-mobile-responsive-fixes/requirements.md): Given an error code with no friendly mapping ... a safe generic message is shown (still no raw uuid)

Specify safe generic fallback or a defined sanitization contract. Wave2 review301–314 explicitly settles optional backend scrubbing, but repeats the server-fallback/safe-generic inconsistency; do not reopen frontend-vs-backend ownership as if unsettled.

## 017 — authority-gap

Conversion cohort and mark-paid confirmation are explicitly settled, including later Matching-page and label extensions. Average commission still lacks a recorded exact time-cohort definition despite a downstream assertion that code is correct.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | aligned | [017-admin-analytics-and-financial-safety/requirements md L24–70](../../../017-admin-analytics-and-financial-safety/requirements.md); [017-admin-analytics-and-financial-safety/design md L6–27](../../../017-admin-analytics-and-financial-safety/design.md); [017-admin-analytics-and-financial-safety/tasks md L15–18](../../../017-admin-analytics-and-financial-safety/tasks.md) | Founder/BA v1.2 fixes completed-of-requested-in-period; it supersedes INT-045's old point-in-time conversion and the old alternatives. 026 further requires displayed tiles to reconcile. |
| REQ-002 | authority-gap | [017-admin-analytics-and-financial-safety/requirements md L72–98](../../../017-admin-analytics-and-financial-safety/requirements.md); [017-admin-analytics-and-financial-safety/design md L29–34](../../../017-admin-analytics-and-financial-safety/design.md); [017-admin-analytics-and-financial-safety/tasks md L16–25](../../../017-admin-analytics-and-financial-safety/tasks.md) | business requirements document specifies mean per collected match and the acceptance uses collected/matches-with-collected-commission. The selected time window and date anchor remain delegated to BA; 019 saying 'correct in code' does not record an approved formula. |
| REQ-003 | aligned | [017-admin-analytics-and-financial-safety/requirements md L100–131](../../../017-admin-analytics-and-financial-safety/requirements.md); [017-admin-analytics-and-financial-safety/design md L36–52](../../../017-admin-analytics-and-financial-safety/design.md); [017-admin-analytics-and-financial-safety/tasks md L24](../../../017-admin-analytics-and-financial-safety/tasks.md) | Explicit amount+match/tutor confirmation at all three Commission call sites, cancel no-op and endpoint guard/idempotency retention agree. 019 extends to Matching. User confirms PAID means agency receipt of tutor commission. |

### 017-F1 — Average commission time cohort is still not pinned as an approved metric (P2, authority-gap)

- [017-admin-analytics-and-financial-safety/requirements md L149–151](../../../017-admin-analytics-and-financial-safety/requirements.md): what 'avg commission per match' should mean ... for which window — business-analyst decides
- [019-admin-money-safety-and-analytics/spec md L43–46](../../../019-admin-money-safety-and-analytics/spec.md): Avg-commission and conversion are correct in code (Wave-3/2). Clarify the labels

BRD554 already requires a mean per collected match. Specify whether selected-period paid at or lifetime defines the population, distinct-match denominator, zero behavior and label. User confirms collected means agency receipt of tutor commission. Code-correctness claims do not settle the commercial metric.

## 018 — conflict

Routing, CTA intent, seeder identity and step copy are coherent. Phone-save wording retains a superseded immediate-update path alongside the email fallback; seed variety design also contains an unapproved small-N relaxation.

| Requirement | Verdict | Evidence | Assessment |
|---|---|---|---|
| REQ-001 | conflict | [018-flag-hygiene-seeder-and-fe-hygiene/requirements md L20–50](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md); [018-flag-hygiene-seeder-and-fe-hygiene/design md L5–16](../../../018-flag-hygiene-seeder-and-fe-hygiene/design.md); [018-flag-hygiene-seeder-and-fe-hygiene/tasks md L15–27](../../../018-flag-hygiene-seeder-and-fe-hygiene/tasks.md) | Channel-scoped WhatsApp suppression is explicitly settled and must preserve email-link verification. 'Phone updates/phone updated' plus the old silent-update question and blanket evidence line conflict with 012 old-value-until-confirm and 018's own channel note. |
| REQ-002 | aligned | [018-flag-hygiene-seeder-and-fe-hygiene/requirements md L52–78](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md); [018-flag-hygiene-seeder-and-fe-hygiene/design md L18–25](../../../018-flag-hygiene-seeder-and-fe-hygiene/design.md); [018-flag-hygiene-seeder-and-fe-hygiene/tasks md L16](../../../018-flag-hygiene-seeder-and-fe-hygiene/tasks.md) | Seeder tutor identity invariant is stated and asserted consistently; it is a dev-data requirement, not production schema work. |
| REQ-003 | conflict | [018-flag-hygiene-seeder-and-fe-hygiene/requirements md L80–104](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md); [018-flag-hygiene-seeder-and-fe-hygiene/design md L27–42](../../../018-flag-hygiene-seeder-and-fe-hygiene/design.md); [018-flag-hygiene-seeder-and-fe-hygiene/tasks md L17](../../../018-flag-hygiene-seeder-and-fe-hygiene/tasks.md) | Requirements/tasks mandate ≥8 subjects, ≥5 levels, ≥4 areas. Design alone allows scaling to min using threshold,N, weakening the measurable acceptance threshold without a folded decision. |
| REQ-004 | aligned | [018-flag-hygiene-seeder-and-fe-hygiene/requirements md L106–128](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md); [018-flag-hygiene-seeder-and-fe-hygiene/design md L44–48](../../../018-flag-hygiene-seeder-and-fe-hygiene/design.md); [018-flag-hygiene-seeder-and-fe-hygiene/tasks md L18](../../../018-flag-hygiene-seeder-and-fe-hygiene/tasks.md) | Role dashboard / unauthenticated login with return preservation is traced and explicitly Wave-2 approved. |
| REQ-005 | aligned | [018-flag-hygiene-seeder-and-fe-hygiene/requirements md L130–147](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md); [018-flag-hygiene-seeder-and-fe-hygiene/design md L50–53](../../../018-flag-hygiene-seeder-and-fe-hygiene/design.md); [018-flag-hygiene-seeder-and-fe-hygiene/tasks md L19](../../../018-flag-hygiene-seeder-and-fe-hygiene/tasks.md) | Step copy must reflect actual required fields; later 024/026 add specific wizard fixes without changing this rule. |
| REQ-006 | aligned | [018-flag-hygiene-seeder-and-fe-hygiene/requirements md L149–189](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md); [018-flag-hygiene-seeder-and-fe-hygiene/design md L55–63](../../../018-flag-hygiene-seeder-and-fe-hygiene/design.md); [018-flag-hygiene-seeder-and-fe-hygiene/tasks md L20](../../../018-flag-hygiene-seeder-and-fe-hygiene/tasks.md) | Generic login wall vs explicit/role-qualified registration CTAs are enumerated and mirrored in design; the scope question is resolved. Public limited filtering is independent and user-authorized. |

### 018-F1 — Phone-save copy still promises an immediate update alongside the settled email-confirm flow (P1, conflict)

- [018-flag-hygiene-seeder-and-fe-hygiene/requirements md L25–34](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md): the phone updates ... neutral 'phone updated' copy ... MUST still show its 'we sent a confirmation link to your account email' copy
- [012-tutor-apply-and-profile-ux/requirements md L138–153](../../../012-tutor-apply-and-profile-ux/requirements.md): keep the existing verified value as the contact of record ... For a PHONE change the confirm link is sent to the user's EXISTING verified ACCOUNT EMAIL

Remove silent-update defaults at018:206–207 and align tasks/evidence with channel-scoped suppression and pending email confirmation. The email fallback itself is explicitly settled.

### 018-F2 — Seed variety design weakens the fixed acceptance threshold (P3, conflict)

- [018-flag-hygiene-seeder-and-fe-hygiene/requirements md L85–88](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md): ≥ 8 distinct subject Code, ≥ 5 distinct level Code, ≥ 4 distinct planning-area
- [018-flag-hygiene-seeder-and-fe-hygiene/design md L41–42](../../../018-flag-hygiene-seeder-and-fe-hygiene/design.md): If the seeder's current open-post count is small, scale the distinct counts to '≥ min using 8, N subjects' etc.

Tasks use the fixed bar. Remove the design-only exception or explicitly amend the acceptance dataset. The remaining 20% rule is also impossible for a positive N<5; define minimum fixture size instead.

## Read vs referenced

All 36 owned spec/requirements/design/tasks documents were read in full. Related documents and evidence were read to the coverage listed below; code-file mentions are historical source claims, not fresh implementation evidence.

- [SCOPE md](../../../SCOPE.md) — full
- [product requirements document md](../../../PRD.md) — targeted excerpts,221–235,670–691,1177–1194,1231–1267,2003–2025,2086–2095
- [business requirements document md](../../../BRD.md) — targeted excerpts,548–577,640–653,787–799
- [004-auth-session-hardening-30min-ttl-and-refresh/requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) — targeted excerpts,159–180
- [issues/review-2026-06-22/WAVE1-SPEC-REVIEW md](../../../issues/review-2026-06-22/WAVE1-SPEC-REVIEW.md) — targeted excerpts
- [issues/review-2026-06-22/WAVE2-SPEC-REVIEW md](../../../issues/review-2026-06-22/WAVE2-SPEC-REVIEW.md) — targeted excerpts,182–247,249–404; heading/decision index
- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md) — targeted excerpts,367–399,458–470
- [architecture/int-045-analytics-date-range md](../../../architecture/int-045-analytics-date-range.md) — targeted normative scoping/table and decisions; output truncation excluded unviewed example middle
- [019-admin-money-safety-and-analytics/spec md](../../../019-admin-money-safety-and-analytics/spec.md) — full
- [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md) — full
- [024-postfix-review-fixes/spec md](../../../024-postfix-review-fixes/spec.md) — full
- [026-s3s4-polish-batch/spec md](../../../026-s3s4-polish-batch/spec.md) — full
- [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) — full
- [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) — full
- [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) — full
- [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) — full
- [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) — full

Referenced or searched only:

- [REDACTED LOCAL PATH] — Referenced by source or only located/searched; not fully reviewed for this numbered matrix.
- [REDACTED LOCAL PATH] — Referenced by source or only located/searched; not fully reviewed for this numbered matrix.
- [REDACTED LOCAL PATH] — Referenced by source or only located/searched; not fully reviewed for this numbered matrix.
- [REDACTED LOCAL PATH] — Referenced by source or only located/searched; not fully reviewed for this numbered matrix.
