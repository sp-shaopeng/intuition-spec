# Sequence15–28 alignment review

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Full text read; compare the described behavior to current requirements. This does not certify runtime behavior.

# Sequence alignment

Sequence15–21 only: full text/diagrams read; relevant current requirements targeted

These are code-derived descriptive flows, not independent policy authority. No source-derived deliberate-decision or GAP statement is assumed to supersede parent requirements without explicit authority.

## Per-file matrix

| File | Verdict | Assessment |
|---|---|---|
| [15-admin-set-meeting-link md](../../../sequences/15-admin-set-meeting-link.md) | authority-gap | SCHEDULED-only meeting-link update and validation align. The claimed deliberate no-notification choice contradicts product requirements document AC17 resend-to-both; no explicit policy waiver was found.007 also requires reminder registration on late link, so this old save-only flow is incomplete.006/007 WhatsApp OFF remains preserved. |
| [16-interview-completion-paths md](../../../sequences/16-interview-completion-paths.md) | partial | Pre-negotiation admin/tutee cancellation states and five-state diagram are superseded by INT039.007 adds reminder cleanup on every terminal path. Two-party no-show actor domain excludes product requirements document BOTH; INT121 labels its typed-enum change contract-unchanged, so it does not explicitly authorize dropping BOTH. |
| [17-tutee-views-matches md](../../../sequences/17-tutee-views-matches.md) | partial | Authenticated, user-scoped match list and review CTA align. Tutor subject still unconditionally dereferences a listing, contrary to INT034’s explicit optional-listing fallback. The admin-contact-email branch is a legacy field rule and must not be mistaken for current counterparty contact-reveal eligibility. |
| [18-tutor-views-matches md](../../../sequences/18-tutor-views-matches.md) | conflict | Tutor commission wording says the tutor is owed money, reversing BRD431 and current user definition: the tutor owes the agency and PAID means agency receipt. The listing-joined dashboard query is also explicitly superseded by INT034 tutor-user identity. INT137a humanised subject/level fold aligns. |
| [19-tutee-submits-review md](../../../sequences/19-tutee-submits-review.md) | aligned | Participant-only, completed-match review, duplicate uniqueness, rating validation, notification and aggregate-refresh flow are consistent at the checked level. Title/diagram deliberately follows tutee while endpoint accepts either participant; that is not a wrong-role navigation claim. No global review-anonymity requirement is inferred from005’s narrower testimonial attribution rule. |
| [20-public-tutor-detail-reviews md](../../../sequences/20-public-tutor-detail-reviews.md) | aligned | M17 fold at3–9 explicitly makes detail authenticated and labels the anonymous detail diagram history; reviews stay public. This is compatible with guest limited filtering. Do not flag the retained historical route as a current access contradiction or reopen the settled detail gate. |
| [21-notification-bell-flow md](../../../sequences/21-notification-bell-flow.md) | partial | Authenticated recipient scoping,404 on another user’s notification, idempotent read, bulk-read and count refresh align. INT147’s structured invitation data/humanised rendering and malformed-data fallback are omitted from this older inbox flow; these are supplemental coverage, not evidence current rendering fails. |

## Source-paired findings

### SEQ2a-F01 — Tutor commission is described as money owed to the tutor

**current-user-conflict**. Current user confirms PAID means the agency received the tutor commission. business requirements document independently says the tutor owes the commission. The sequence’s explanation reverses payer/payee; this is a direct wording conflict, not a monetary implementation claim.

[sequences/18-tutor-views-matches md](../../../sequences/18-tutor-views-matches.md)


The illustration for seq2a-f01 — tutor commission is described as money owed to the tutor contains the following information:

- Unlike the tutee view, the tutor view exposes the **commission triple**.
- (monthly Fee Sgd, commission Amount Sgd, commission Status) because the.
- tutor needs to see how much they're owed and whether it has been paid.


[business requirements document md](../../../BRD.md)


The illustration for seq2a-f01 — tutor commission is described as money owed to the tutor contains the following information:

- ### BR-004: Commission Calculation.
- Commission = (agreed monthly tuition fee) × (commission rate).
- Default commission rate = **50%** of first month's tuition fee.
- Commission is charged **once per match** that results in an active arrangement.
- There are no recurring platform fees, subscription charges, or per-session fees.
- The commission is owed by the **tutor** (Singapore industry standard).
- Commission is tracked and collected off-platform in MVP (no in-platform payment).


[sequences/12-admin-match-cancel-cascade md](../../../sequences/12-admin-match-cancel-cascade.md)


The illustration for seq2a-f01 — tutor commission is described as money owed to the tutor contains the following information:

- Service: Admin Match Service implementation reference-280. The comment block (lines 241-254).
- documents the design intent of the cascade — uncollectable amounts must not.
- silently inflate outstanding revenue.
- CANCELLABLE STATUSES PENDING, ACTIVE (line 57).
- Commission PAID is intentionally NOT touched — agency already received payment.


**Proposed default:** State how much the tutor owes the agency and whether the agency has received payment.

### SEQ2a-F02 — No-notification meeting-link update lacks authority against explicit resend requirement

**authority-gap**. The source-derived sequence calls silence deliberate, but product requirements document explicitly requires updated-link notification to both parties.007 adds late-link reminder registration; this does not explicitly waive immediate updated-link notifications.006’s scheduling/WhatsApp changes likewise do not settle this cross-channel requirement by implication.

[sequences/15-admin-set-meeting-link md](../../../sequences/15-admin-set-meeting-link.md)


The illustration for seq2a-f02 — no-notification meeting-link update lacks authority against explicit resend requirement contains the following information:

- Svc leads to >Svc: interview meeting Link = req meeting Link.
- Svc leads to >IRepo: save.
- Svc- leads to >Ctrl: Admin Interview Response.
- Ctrl- leads to >frontend: 200 OK.


## Notable details

- Controller: Interview Controller implementation reference — @Patch Mapping.
- Service: Interview Service implementation reference.
- No notification is emitted on link set — the link is visible the next time
  the tutee opens the tutee's interview-detail page. This is a deliberate decision (the
  interview is already scheduled; the link is a follow-up admin detail).


The illustration for notable details contains the following information:

- [product requirements document md](../../../PRD.md).


GIVEN an interview is in SCHEDULED status
WHEN admin updates the meeting link before the scheduled time
THEN the interview record is updated with the new link AND notifications are resent to both tutee and tutor with the updated link


The illustration for notable details contains the following information:

- [007-generic-scheduler-db-scheduler/requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md).


  **(f) set Meeting Link** (626) — adding a link to a >24h SCHEDULED interview registers (or re-registers) the job;
  clearing it cancels. Cancellation is **idempotent** (cancelling a non-existent job is a no-op) so any hook firing
  twice or for a ≤24h interview that never registered is safe.


Implementation example omitted. This public edition retains the written requirements in this section.


    alt status not in [REQUESTED, SCHEDULED] → 422 INVALID STATUS TRANSITION


The illustration for notable details contains the following information:

- [sequences/16-interview-completion-paths md](../../../sequences/16-interview-completion-paths.md).


    Tutee leads to >Ctrl: POST tutee interviews id cancel interface operation
    Ctrl leads to >Svc: cancel My Interview using tutee Id, id
    Svc leads to >IRepo: find By Id; assert ownership (else 404 INTERVIEW NOT FOUND)
    alt status not in [REQUESTED, SCHEDULED] → 422 INVALID STATUS TRANSITION
    end
    Svc leads to >IRepo: status is CANCELLED, save
    Svc leads to >NS: emit using tutor User Id, INTERVIEW CANCELLED, Interview cancelled, The tutee has cancelled their interview request for \<title>\., /tutor/interviews


The illustration for notable details contains the following information:

- [sequences/16-interview-completion-paths md](../../../sequences/16-interview-completion-paths.md).


The lifecycle diagram describes these state changes:

- Start leads to REQUESTED after tutee request Interview.
- REQUESTED leads to SCHEDULED after admin schedule Interview.
- REQUESTED leads to CANCELLED after tutee cancel My Interview / admin admin Cancel Interview.
- SCHEDULED leads to COMPLETED after admin complete Interview.
- SCHEDULED leads to CANCELLED after tutee cancel My Interview / admin admin Cancel Interview.
- SCHEDULED leads to NO SHOW.


[architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[007-generic-scheduler-db-scheduler/requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md)


Implementation example omitted. This public edition retains the written requirements in this section.


**Proposed default:** Rebase the state machine and show all three cancellation actors plus idempotent reminder cleanup.

### SEQ2a-F04 — Both-parties no-show was dropped without an explicit policy amendment

**authority-gap**. product requirements document requires TUTEE/TUTOR/BOTH. The sequence only accepts the first two. INT121 later reuses the two-party Timeslot Proposer enum but explicitly says contract unchanged, so its implementation-hygiene change cannot itself establish authority to remove BOTH.

[sequences/16-interview-completion-paths md](../../../sequences/16-interview-completion-paths.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[sequences/16-interview-completion-paths md](../../../sequences/16-interview-completion-paths.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[product requirements document md](../../../PRD.md)


The illustration for seq2a-f04 — both-parties no-show was dropped without an explicit policy amendment contains the following information:

- GIVEN an interview was scheduled but one or both parties did not join.
- WHEN admin marks NO SHOW.
- THEN the interview status transitions to NO SHOW and the no show actor field is set (TUTEE / TUTOR / BOTH).


[architecture/int-121-batch-hygiene-spec md](../../../architecture/int-121-batch-hygiene-spec.md)


Implementation example omitted. This public edition retains the written requirements in this section.


**Proposed default:** Retain a dedicated no-show domain including BOTH, or explicitly amend product requirements document and define how simultaneous absence is recorded; do not infer a policy change from enum reuse.

### SEQ2a-F05 — Match dashboard diagrams still depend on a mandatory tutor listing

**explicit-supersession**. INT034 specifically replaces these two read paths with tutor-user identity and safe listing-less subject/title fallback. The old sequence is obsolete documentation, not evidence of current runtime failure.

[sequences/17-tutee-views-matches md](../../../sequences/17-tutee-views-matches.md)


The illustration for seq2a-f05 — match dashboard diagrams still depend on a mandatory tutor listing contains the following information:

- loop for each match.
- Svc leads to >Svc: tutor Subject = tutor Post subject Codes first.
- If a commission record exists, the service reads its stored first-month fee.


[sequences/18-tutor-views-matches md](../../../sequences/18-tutor-views-matches.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[architecture/int-034-free-market-assign md](../../../architecture/int-034-free-market-assign.md)


Implementation example omitted. This public edition retains the written requirements in this section.


**Proposed default:** Use tutor-user query and explicit optional-listing fallback in both dashboard diagrams.

### SEQ2a-F06 — Public detail history is explicitly labelled and already superseded

**settled-historical-alternative**. Header explicitly governs current detail authentication and public review exception. The retained anonymous diagram is labelled pre-M17 history. User permission for anonymous filtering does not itself remove detail authentication.

[sequences/20-public-tutor-detail-reviews md](../../../sequences/20-public-tutor-detail-reviews.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[sequences/20-public-tutor-detail-reviews md](../../../sequences/20-public-tutor-detail-reviews.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[sequences/20-public-tutor-detail-reviews md](../../../sequences/20-public-tutor-detail-reviews.md)


Implementation example omitted. This public edition retains the written requirements in this section.


**Proposed default:** Keep the explicit history label; optionally move the obsolete detail branch out of the live review-fetch diagram for easier reading. No policy clarification needed.

### SEQ2a-F07 — Inbox sequence omits the structured invitation rendering contract

**partial-coverage**. Core recipient authorization/read-state flow aligns. The older inbox presentation stops at generic Notification Response; INT147 adds invitation code data, humanised labels and malformed/legacy fallback. This is missing descriptive coverage, not a current code failure.

[sequences/21-notification-bell-flow md](../../../sequences/21-notification-bell-flow.md)


Implementation example omitted. This public edition retains the written requirements in this section.


[architecture/int-147-structured-notification-data-spec md](../../../architecture/int-147-structured-notification-data-spec.md)


Implementation example omitted. This public edition retains the written requirements in this section.


**Proposed default:** Add a rendering branch for invitation data with humanised labels and safe fallback, leaving other notification types unchanged.

## Limitations

- No existing source/spec edits or runtime tests.
- No anonymous filtering conflict found within15–21;20 has an explicit historical-detail banner.
- Parent approval-versus-assignment contact rule is handled in root audit. Admin contact email and counterparty reveal are separate fields/surfaces.
- Related amendments read selectively; review completion/uniqueness assessed at the documented flow level, not implementation behavior.


---

# Sequence alignment22–28

All seven sequence files read in full. Supporting requirements/amendments were reviewed in targeted excerpts. These are document-alignment findings, not runtime bug confirmations.

| Flow | Verdict | Result |
|---|---|---|
| [22](../../../sequences/22-favourite-toggle.md) | partial | Optimistic toggle, membership lookup and idempotent responses remain useful; persistence still points at physical deletion superseded by approved soft-remove/reactivation. |
| [23](../../../sequences/23-public-browse-with-filters.md) | conflict | Guest filter hiding conflicts with current user confirmation. The oversized-size empty result is explicitly superseded by clamp-to20; retained pagination caps remain applicable. |
| [24](../../../sequences/24-rate-limited-endpoint.md) | conflict | Atomic limiter mechanics are descriptive. Its IP+email bucket does not prove the retained per-IP login ceiling; production forwarded-header trust must incorporate027. |
| [25](../../../sequences/25-csrf-protected-write.md) | aligned | The worked protected-write flow consistently uses a readable XSRF cookie/header while keeping the auth token HttpOnly. Same Site is Strict and double-submit remain compatible with004; no material mismatch found in this narrow flow. Global response-trace/security-error details should link to010, whose security-filter body coverage is an acknowledged separate follow-up. |
| [26](../../../sequences/26-email-fanout-on-event.md) | partial | Explicitly historical email producer/consumer snapshot is superseded by durable outbox, bounded consumer retry/failed-message queue and the approved Resend production path. It cannot establish current delivery failure. |
| [27](../../../sequences/27-encrypted-pii-write-and-lookup.md) | partial | Entire encrypted user-email/phone flow is historical under the explicit plaintext-contact amendment. That amendment changes storage, not public contact visibility. |
| [28](../../../sequences/28-tutor-apply-withdraw.md) | partial | The labelled M15 slice predates free-market apply, completeness feedback, parent applicant access and expanded application-state rules. Existing idempotency/row retention remain relevant; obsolete listing and state gates need replacement. |

## Paired-source findings

### SEQ2B-01 — Favourite removal still describes the superseded physical-delete path (superseded-drift)

- [sequences/22-favourite-toggle md](../../../sequences/22-favourite-toggle.md): “delete By Tutee Id And Tutor Post Id (no-op if absent)”
- [architecture/int-094-batch-soft-delete-integrity-spec md](../../../architecture/int-094-batch-soft-delete-integrity-spec.md): “DECIDED: add favourite tutor posts removed at and soft-remove; re-favourite reactivates the prior row”
- [architecture/int-094-batch-soft-delete-integrity-spec md](../../../architecture/int-094-batch-soft-delete-integrity-spec.md): “Every current-favourite read filters removed At IS NULL; remove uses soft Remove; add reactivates”

The settled amendment preserves optimistic user experience/idempotency while changing persistence and membership probes. An web request protocol DELETE endpoint does not itself mean physical deletion; the obsolete repository method is the conflicting clause.

Proposed correction: Replace the repository operation with soft-remove, show removed-row filtering for membership/list queries and reactivate the same row when favourited again.

### SEQ2B-02 — Guest-filter ban conflicts with the current user decision (settled-user-decision)

- [sequences/23-public-browse-with-filters md](../../../sequences/23-public-browse-with-filters.md): “filter bar renders only for logged-in users; anonymous visitors get a capped teaser sample (no filter user interface)”
- [sequences/23-public-browse-with-filters md](../../../sequences/23-public-browse-with-filters.md): “Marketplace Filter Bar renders only when user is set; Browse Assignments gates its filter bar the same way”
- [reviews/2026-09-05-requirements/audit md](../audit.md): “C-01: Anonymous visitors may filter a limited public result set”

Current user confirmation controls; this is not an unresolved choice about guest filtering. Result caps and detail/action authentication remain distinct.

Proposed correction: Show guest filter interaction and bounded result queries. Keep the documented pagination/rate caps; do not turn five result pages into a lifetime five-search quota.

### SEQ2B-03 — Oversized anonymous page requests still return the superseded empty result (superseded-drift)

- [sequences/23-public-browse-with-filters md](../../../sequences/23-public-browse-with-filters.md): “if principal equals null AND (page>4 OR size>20) → return empty page”
- [architecture/int-125-batch-closeout-spec md](../../../architecture/int-125-batch-closeout-spec.md): “Boss rulings ratified: INT-136e = anonymous clamp-to-20, page-cap stays empty”
- [architecture/int-125-batch-closeout-spec md](../../../architecture/int-125-batch-closeout-spec.md): “size>20 returns20 results; deep pages stay empty; malformed size/page stays empty”

The explicit amendment changes the size behavior only. Tutor anonymous cap and assignments endpoint cap have intentionally different authenticated bypass behavior.

Proposed correction: Show valid oversized size clamped to20; retain page>4 empty, malformed-input handling, and endpoint-specific audience rules. Update the contradictory hardcoded-size note at sequence23:58–59.

### SEQ2B-04 — Composite login bucket does not establish the retained per-IP limit (authority-gap)

- [sequences/24-rate-limited-endpoint md](../../../sequences/24-rate-limited-endpoint.md): “login is strictest (10 / hour per IP+email Hmac)”
- [sequences/24-rate-limited-endpoint md](../../../sequences/24-rate-limited-endpoint.md): “login Key using ip,email Hmac,10,3600”
- [modules/auth/spec md](../../../modules/auth/spec.md): “AC-AUTH-19:10 attempts/hour/IP; AC-AUTH-20:3 password reset requests/hour/email”

A bucket keyed by both IP and email limits each pair, not the total attempts from an IP. No explicit amendment authorizing replacement of the retained per-IP ceiling was found in this targeted pass. This is a contract mismatch, not a claim about the running limiter.

Proposed correction: Record whether the composite bucket supplements or replaces the per-IP ceiling, with the approval trace for any replacement; show a test varying email within one IP. Keep forgot-password and reset-password endpoint distinctions explicit.

### SEQ2B-05 — Forwarded client-IP derivation omits the approved trusted-hop boundary (superseded-drift)

- [sequences/24-rate-limited-endpoint md](../../../sequences/24-rate-limited-endpoint.md): “prefers CF-Connecting-IP, falls back to X-Forwarded-For first hop, then remote Addr”
- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “trusted hop (loopback/cloudflared sidecar only); spoofed CF-Connecting-IP/XFF from untrusted source is not honored”

Header preference alone is incomplete for the approved production deployment. The old sequence has no source-trust condition.

Proposed correction: Show trusted-peer validation before using forwarded headers, and the untrusted-source rejection/fallback behavior required by027.

### SEQ2B-06 — Email flow remains an obsolete fire-and-forget/missing-consumer snapshot (superseded-drift)

- [sequences/26-email-fanout-on-event md](../../../sequences/26-email-fanout-on-event.md): “Email Queue Service @Async; Email consumer GAP: not present”
- [sequences/26-email-fanout-on-event md](../../../sequences/26-email-fanout-on-event.md): “after Commit → @Async → XADD; consumer/SES assumed”
- [sequences/26-email-fanout-on-event md](../../../sequences/26-email-fanout-on-event.md): “failure logged; no retry/back-off; No consumer in this monorepo”
- [architecture/int-087-transactional-messaging-hardening-spec md](../../../architecture/int-087-transactional-messaging-hardening-spec.md): “Email Payload recipient User Id; enqueue Email MANDATORY in business transaction; Outbox Poller dispatch”
- [architecture/transactional-messaging-and-idempotency md](../../../architecture/transactional-messaging-and-idempotency.md): “Email outbox SENT only after XADD; former after Commit → @Async → XADD sites repointed”
- [architecture/transactional-messaging-and-idempotency md](../../../architecture/transactional-messaging-and-idempotency.md): “bounded retry, Email Consumer Service failed-message queue, dedup only after successful send; ACK after side-effect succeeds”
- [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md): “Resend custom domain and prod key; real email in cutover smoke”

Later approved design explicitly replaces the volatile producer path and defines a consumer. The old claim that email has never been delivered and the internal after Commit/inline contradiction at26:57–67 are historical observations, not current runtime findings.

Proposed correction: Redraw business transaction→durable outbox→synchronous XADD→consumer/Resend→ACK-or-retry/failed-message queue. Attribute implementation/deployment evidence to its source; remove present-tense missing-consumer and assumed-SES assertions.

### SEQ2B-07 — Encrypted user contact storage and keyed message authentication lookup are explicitly superseded (superseded-drift)

- [sequences/27-encrypted-pii-write-and-lookup md](../../../sequences/27-encrypted-pii-write-and-lookup.md): “email Encrypted/phone Encrypted columns; keyed message authentication lookup columns”
- [sequences/27-encrypted-pii-write-and-lookup md](../../../sequences/27-encrypted-pii-write-and-lookup.md): “Plaintext email/phone is never indexed or stored”
- [architecture/plaintext-email-phone md](../../../architecture/plaintext-email-phone.md): “founder directive; Supersedes ADR-003 for email/phone”
- [architecture/plaintext-email-phone md](../../../architecture/plaintext-email-phone.md): “plaintext users email/users phone; normalised exists By Email/find By Email; remove email/phone encryption/keyed message authentication”

The at-rest change is explicit and settled. It does not authorize public contact exposure or remove password hashing, encrypted backups, or unrelated encryption uses.

Proposed correction: Mark this encrypted-storage sequence historical and document normalized plaintext user-contact persistence/lookups under the approved directive; retain current visibility authorization separately.

### SEQ2B-08 — Application sequence retains the removed active-listing prerequisite (superseded-drift)

- [sequences/28-tutor-apply-withdraw md](../../../sequences/28-tutor-apply-withdraw.md): “no active listing →403 TUTOR NO ACTIVE POST”
- [sequences/28-tutor-apply-withdraw md](../../../sequences/28-tutor-apply-withdraw.md): “403 TUTOR NO ACTIVE POST”
- [product requirements document md](../../../PRD.md): “old TUTOR NO ACTIVE POST403 guard is removed; profile-completeness gate; tutor Post Id stays optional”
- [012-tutor-apply-and-profile-ux/requirements md](../../../012-tutor-apply-and-profile-ux/requirements.md): “single the private implementation folder completeness definition; incomplete profile pre-gate;422 friendly toast/banner and403 actionable feedback”

M18 explicitly opens free-market applications to tutors without a listing. Later012/024 clarify completeness and feedback without restoring the listing gate.

Proposed correction: Replace listing-count gate with the current shared completeness gate and its visible failure outcomes; retain optional tutor Post Id and the ACTIVE-job guard.

### SEQ2B-09 — M15-only application state handling omits settled reapply and withdraw rules (superseded-drift)

- [sequences/28-tutor-apply-withdraw md](../../../sequences/28-tutor-apply-withdraw.md): “Only APPLIED/WITHDRAWN transitions exist”
- [sequences/28-tutor-apply-withdraw md](../../../sequences/28-tutor-apply-withdraw.md): “find APPLIED then WITHDRAWN; otherwise save new application”
- [sequences/28-tutor-apply-withdraw md](../../../sequences/28-tutor-apply-withdraw.md): “WITHDRAWN no-op; otherwise update status is WITHDRAWN”
- [product requirements document md](../../../PRD.md): “wire SHORTLISTED/MATCHED; add SUPERSEDED; retain REJECTED reserved”
- [architecture/int-093-batch-state-machine-integrity-spec md](../../../architecture/int-093-batch-state-machine-integrity-spec.md): “withdraw APPLIED/SHORTLISTED only; WITHDRAWN no-op; other states422 APPLICATION NOT WITHDRAWABLE”
- [architecture/int-093-batch-state-machine-integrity-spec md](../../../architecture/int-093-batch-state-machine-integrity-spec.md): “load pair regardless of status; SUPERSEDED/CLOSED reactivate; REJECTED409; MATCHED409; SHORTLISTED idempotent”

The explicit M15 snapshot predates M18 and the approved state-machine amendment. It cannot serve as the complete current lifecycle; especially, a matched application must not be withdrawn while its match remains active.

Proposed correction: Keep historical M15 context labelled and add the full current state branches, preserving one durable row and all explicit error/no-mutation cases. Link the parent applicant view in product requirements document; its addition does not revoke admin-only access to the separate admin endpoint.

## Documents read

- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — full
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file
- [[REDACTED LOCAL PATH]) — targeted normative/acceptance/amendment clauses; not full file

## Review limits

- All seven assigned sequence files read in full; supporting sources targeted to relevant requirements and amendments.
- Sequences describe historical code paths; they cannot approve policy changes. HISTORICALLY MARKED IMPLEMENTED/code existence is not independently treated as approval.
- No runtime tests, source-code edits, provider calls or external standards research.
- Current user decisions preserved: anonymous limited filtering allowed; PAID means agency receipt of tutor commission. None of these seven flows changes the commission event.
- Security error body coverage under010 is not inferred to be implemented from the global trace Id goal. No current absence of email delivery or consumer is asserted.
