# Identity, privacy, notifications and operations requirements audit

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](corrections.md) and [current requirements](../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Read-only audit, 2026-09-05. No source edits, dependency runs, tests, network research or legal conclusions. This checks internal requirements and acceptance criteria. Paths below are relative to REDACTED local path unless another repository is named. Recommendations are proposals, not approved product changes.

Latest confirmed business meaning: commission mark Paid means the agency received the tutor's commission payment. Nothing in this audit reopens that definition or assumes payment processing is required.

For consolidation into eight decision areas: combine I-02/I-03 as the consent lifecycle and I-05/I-06 as contact-change/recovery; keep I-01, I-04 and I-07 through I-10 as separate areas.

## Requirement inventory

| Area | Current requirements / main source |
|---|---|
| Roles and registration | TUTEE/TUTOR public registration, exclusive roles, explicit tutor commission acknowledgement; three required consent flags and optional marketing flag, timestamps and policy versions. Auth module is historical; 001 requirements, 91–166 and 351–367 are the later consent contract. |
| Anonymous access | Preserve current user baseline: anonymous users may browse and filter a limited public result set; auth is required for protected detail/action flows. reference/backend-api md documents assignments cap page≤4, size≤20 and 60/min; no new anonymous-search restriction proposed. |
| Password recovery | Email one-time passcode; valid reset consumes token, clears forced-password-change state, old password rejected; wrong/expired/unknown-account neutral; actionable validation/rate-limit errors. 003 requirements. Production one-time passcode logging is OFF under 027:50–54. |
| Sessions | 30-minute access tokens; TUTOR/TUTEE rotating refresh with 7-day idle / 30-day absolute cap, 30-second rotation grace; hashed refresh secret, cross-site request forgery protection, rate limit, block/reset/logout revocation. ADMIN remains Redis stateful with sliding 1800s and receives no refresh. 004 requirements, 204–251, 254–340; design. |
| Browser auth robustness | Logout wins an in-flight refresh race; clears caches; stale autofill does not auto-submit; logged-out unread poll disabled; duplicate-phone error specifically 409 PHONE ALREADY REGISTERED with limiter-before-membership and dummy bcrypt. 011 requirements, 113–210; architecture/int-095-batch-auth-session-hardening-spec md. |
| Contact self-edit | Step-up at initiation; pending old value remains active; hashed single-use 30-minute link; explicit Confirm POST; new-email confirmation for EMAIL, existing-account-email confirmation for PHONE; uniqueness at commit; other same-field requests superseded; audit and old-channel notices. 012 requirements. |
| Admin management | Any ADMIN may create admins. Only super-admin may remove ordinary admins; super-admin protected and flag immutable through application programming interface. Forced password change, mutation gate, removal revokes Redis sessions, actor+target audit. Dev multi-factor authentication OFF is an explicit choice; production multi-factor authentication and secure bootstrap required. modules/admin-management/spec md, 49–85; 027:65–70. |
| Moderation | Block TUTOR/TUTEE, reason+note, immediate auth rejection, marketplace/content suppression and audit; unblock restores login but does not reopen closed content. Posting removal logical and distinct from closure. architecture/int-065-admin-block-delete-spec md, 97–104. |
| Consent/privacy | Required consent capture and published Terms/Privacy; marketing master AND WhatsApp channel gate for marketing; transactional WhatsApp separate; public policy offers access/correction/withdrawal by contacting support. 001 requirements, 229–267, 329–367; 001 spec. |
| Notification inbox | Per-recipient list, unread counter, individual/read-all idempotent actions, cross-user 404, 60-second bell polling only while authenticated; structured invite context humanized by frontend and safe legacy fallback. sequences/21-notification-bell-flow md, 54–87; architecture/int-147-structured-notification-data-spec md. |
| WhatsApp interviews | OFF until enablement gates. Schedule confirmation to both parties when linked; reminder ~24h before if originally >24h away; ≤24h gets confirmation only. Opt-in+phone-verified eligibility. OFF rows drop, existing in-app behavior unchanged. 006 requirements, 253–282; 007 requirements, 386–412. |
| Scheduler/messaging | db-scheduler persistent one-time reminders and recurring expiry/showcase jobs; lifecycle cancel/re-guard; bounded retries; outbox intent atomic with business data; consumers dedupe, terminal failures/failed-message queue, secret suppression. 007 requirements and spec; architecture/transactional-messaging-and-idempotency md, 135–155; ops/runbook md. |
| Error/support user experience | Consistent typed human errors, no raw entity id in public copy; 404 for unknown route; field-level create-admin validation; copyable trace Id on errors with header fallback, none for pure network errors; tel/mailto encoding. 010 requirements; 016 requirements. |
| Responsive/non-functional requirement | Core 390px flows and internal grid scroll; admin mobile menu reachability, focus and Escape handling; consistent breakpoints. Availability, delivery service-level agreement, latency, backup/restore requirements plus approved lean production plan. 015 requirements; product requirements document; business requirements document; 027:26–99. |
| Production | Dedicated readiness, graceful shutdown/drain, SHA provenance, migrate-before-boot, prod one-time passcode logging OFF, trusted forwarded IP, no prod swagger, secure admin+multi-factor authentication, database/Redis TLS, bounded streams; Amazon Web Services/Cloudflare lean plan and cutover smoke/PITR drill. 027:28–99. Cutover/apply remain explicitly blocked in spec; not claiming deployment. |

## Ten requirement gaps / decisions

### I-01 — Define what the minor/guardian assertion means operationally

Evidence: 001 requirements requires the exact checkbox “at least 18 … OR … verified parent/guardian consent”; persistence at  is one age Confirmed boolean+timestamp. Terms implementation reference permits under-18 users with parent/guardian consent. There is no age band, guardian identity, verification actor or guardian-account journey in this contract.

Scenario: a 16-year-old creates a TUTOR account and accepts commission terms, or creates a TUTEE account to arrange lessons directly. The same boolean cannot tell which eligibility branch was used or who is responsible for confirmations/contact sharing.

Decision: Is the claim deliberately a self-attestation for either role, or does “verified” imply an agency/guardian action? Who is the contracting/contact person for a minor learner/tutor?

Proposed default: preserve the currently allowed self-attestation at signup; explicitly label it as such internally and define the responsible adult/contact before a minor's first assignment commitment. Do not invent an identity-document collection requirement. Add separate adult, guardian-attested learner, guardian-attested tutor and missing-attestation ACs. This is a product definition question, not a legal conclusion.

### I-02 — Re-consent is an explicit prelaunch residual without an end-to-end acceptance flow

Evidence: 001 spec explicitly leaves “pre-existing-user re-consent path before public launch” as a follow-up. 001 requirements makes old consent columns default FALSE;  only records versions on registration. Current Privacy page changes clause (Privacy implementation reference) promises updated publication, not a user acceptance journey.

Scenario: a pre-001 user logs in with a live match, or policies change while an existing user is logged in. Requirements do not define when acceptance is requested, which operations remain possible, how refusal works, or which version is used during staggered the private implementation folder rollout.

Decision: What actions require current required-consent versions, and what must remain accessible when consent is declined?

Proposed default: re-consent at next login before new marketplace commitments; retain access to existing arrangement status, support, privacy requests and logout. Do not retroactively set old flags true. Define versioned acceptance audit, decline/defer behavior, direct-link/refresh behavior and old-the private implementation folder tests. This is a missing acceptance specification for an already-required residual, not a new policy dispute.

### I-03 — Withdrawal/access requests have a public promise but no operational completion contract

Evidence: published Privacy page Privacy implementation reference offers access, correction and consent withdrawal through contact. 001 design treats withdrawal history as possible future work. business requirements document names admin export/correction/deletion processes but gives no response/fulfilment criteria. 001 requirements establishes only signup-time marketing state.

Scenario: a user asks support to stop marketing but retain tuition services, or requests a data export/correction while unable to log in. There is no defined identity check, case owner, acknowledgment, result artifact or recorded completion; marketing opt-out is not the same decision as deleting the account.

Decision: Which consent scopes can be withdrawn independently, who processes the request, and what user-visible completion and service consequences are promised?

Proposed default: separate marketing opt-out, WhatsApp channel opt-out, service-consent withdrawal and account deletion. Support can fulfill manually initially, but use a tracked case, documented identity check, timestamped decision, confirmation and an internal response target. For marketing, stop future eligible sends as soon as withdrawal is recorded, including queued sends. Pin that behavior with a queued-before/revoked-before-send acceptance scenario. No externally mandated deadline is asserted here.

### I-04 — Retention periods and anonymisation coverage need one current matrix

Evidence: auth module spec promises immediate soft-delete followed by personal information anonymisation at T+30 days; business requirements document lists user personal information for two years, closed posts one year, matches five years, commissions seven years, and older hard-delete wording; product requirements document promises photo deletion within 24 hours. product requirements document excludes immutable audit from analytics anonymisation; business requirements document simultaneously says “five years” and “no deletion.” Current project instruction requires soft-delete only, so reopening physical row deletion is unnecessary.

Scenario: account deletion occurs during an active arrangement; at day 30 the profile is anonymised, but raw old/new contact values, consent records, notifications, outbox payloads, exports and backups may remain identifiable. The rules do not reconcile the deadlines or enumerate the later-added data classes.

Decision: What is the authoritative retention/anonymisation deadline and exception for each data class, and how is a completed request verified across replicas/exports/backups?

Proposed default: keep logical records/FKs per current soft-delete rule; adopt the more specific auth-module T+30-day account personal information anonymisation as the proposed operational baseline, with a separately approved minimized financial/audit exception. Inventory contact-change history, consent records, notification/outbox/failed-message queue payloads and analytics, and define an artifact proving identifiers were removed without destroying required business linkage. The settled soft-delete method is documentation drift; the timing/data-class reconciliation remains the requirement gap.

### I-05 — Contact-change flow assumes an already-verified account email without defining acquisition or loss

Evidence: 012 requirements and  require PHONE confirmation through an EXISTING verified account email; scenarios  begin with verified email/phone. Historical auth module spec explicitly leaves email verification open, and registration at  immediately issues a session. Current User implementation reference has phone Verified () but no email Verified field; the reviewed contact-change initiation contract has no unverified-email branch.

Scenario: the user mistyped the email at signup, no longer has mailbox access, or holds a legacy account with no recorded verification. They need to correct the phone but cannot satisfy the presumed precondition.

Decision: Does completing the email-link confirmation itself prove sufficient control, or must email verification preexist? How can a user recover when that mailbox is unavailable? Separately reconcile the historical onboarding choice of verification before activation versus immediate access; that choice does not specify legacy/unverified/lost-mailbox recovery.

Proposed default: keep the founder-selected email-link channel. Treat a successful link action as proof of current mailbox possession, and remove any false assumption that signup had previously verified it; if a prior verified-email requirement is retained, define its acquisition/gate explicitly. Provide a separately authenticated support recovery path for a lost mailbox, with audit and session/token revocation. Add new-user, legacy-unverified, typo, lost-mailbox and simultaneous-email/phone-change ACs.

### I-06 — “Notify both old contacts with revoke” has no independent phone/revocation contract

Evidence: 012 requirements and  require BOTH old email AND old phone to receive an initiation notice with a revoke affordance before commit;  says the flow is independent of WhatsApp. Its design only specifies initiate and verify routes. Current Contact Change Service sends one email template containing an old Phone parameter; its comment says that stands in for a phone notice while WhatsApp is OFF. Listing a phone number in an email does not reach the old-phone holder independently. INT-082 spec places non-WhatsApp SMS out of scope.

Scenario: an attacker controls the mailbox and password but the real owner still controls the old phone. The independent warning/revoke protection promised by the requirement has no usable channel. There is also no specified unauthenticated revoke capability, expiry/race behavior or post-commit recovery outcome.

Decision: Independent dual-channel notice is already the current requirement. Which phone channel and revoke mechanism will satisfy it, or is an explicit scope amendment needed while phone messaging is unavailable? What exactly may the old-contact holder revoke, before and after confirmation?

Proposed direction: specify and enable the independent phone channel before claiming the current requirement is satisfied. Email-only notice is a possible explicit scope amendment, not an approved substitute or dual-channel protection. Keep the email-link confirmation choice intact. Define a separate single-use cancel capability plus support recovery for a completed unauthorized change; a cancel/confirm race must have one winner and both attempts audited.

### I-07 — Offline logout acceptance contradicts the HttpOnly credential contract

Evidence: 011 requirements says the frontend clears auth itself, including on network failure; design calls auth JS-readable and requires local cookie expiration. 004 design says auth is host-only HttpOnly. Current Auth Controller confirms the http only operation for auth, as well as refresh at .

Scenario: the user logs out during an outage on a shared browser, then reloads or opens a second tab after connectivity returns. Clearing React state is not server revocation, and JS cannot satisfy the stated HttpOnly-cookie removal requirement.

Decision: Is failed-network logout “local screen locked pending server revocation” or a completed account-session termination? How must reload/new-tab/reconnect behave?

Proposed default: retain HttpOnly. Distinguish local sign-out from confirmed server revocation; block automatic session restoration while logout is pending, persist/broadcast only the minimal pending-logout marker, retry server revocation on reconnect, then remove the marker. Define acceptance for outage+reload, second tab, in-flight refresh, success, and failed revocation notification. Do not weaken cookies to make the old frontend instruction executable.

### I-08 — Protected super-admin and mandatory prod multi-factor authentication lack a recovery/compromise use case

Evidence: modules/admin-management/spec md protects the super-admin from disable/delete/demotion by anyone;  makes its flag write-once through the seeder;  requires multi-factor authentication before production. 027:65–70 specifies secure first bootstrap, requires multi-factor authentication enrollment, then deletes the bootstrap secret after first password change. There is no loss-of-authenticator, unavailable-owner, or compromised-super-admin acceptance flow in these sources or the reviewed ops runbook.

Scenario: the only super-admin loses its authenticator, leaves the business, or is compromised. A regular admin can create more regular admins but cannot contain or replace the protected principal through the described management model.

Decision: Who can recover/replace/revoke the super-admin and multi-factor authentication, by which separately authorized emergency process, and how are all sessions invalidated?

Proposed default: no public bypass. Document an out-of-band, two-person controlled recovery runbook with identity checks, time-bounded credentials, mandatory multi-factor authentication re-enrollment, immediate revocation of existing sessions and immutable actor/approver/result audit. Drill loss and compromise scenarios before production. Bootstrap idempotence alone is not recovery acceptance.

### I-09 — Define delivery milestones, delayed-message expiry and operational recovery

Evidence: product requirements document gives interview schedule N-22/N-23 a five-minute service-level agreement, but product requirements document says all non-commission events get 30 minutes. 006 requirements and 007:155–162 prove outbox enqueue, not provider/recipient delivery. Messaging playbook states outbox SENT occurs after Redis XADD. Yet ops/runbook md calls outbox SENT a password-reset “delivered-rate.” The same runbook  defines terminal DLQs and inspect-on-demand metrics but no redrive/recipient-resolution procedure. The 007 handler re-guard () is before enqueue; the allowed behavior of an already-queued reminder/link that becomes obsolete is unspecified.

Scenario to verify: a queue/provider outage delays a 15-minute one-time passcode or a reminder; the interview is cancelled or its link changes before the queue recovers. A support person could see SENT even though the message only reached Redis, or could redrive obsolete instructions. No obsolete-message runtime reproduction is claimed by this audit.

Decision: Does the service-level agreement end at durable enqueue, provider acceptance or confirmed delivery? Which messages expire/supersede, who owns failed delivery, and how is a safe manual retry authorized?

Proposed default: distinguish queued, provider-accepted, delivered using if available, permanently-failed and expired/superseded. Preserve five minutes for schedule notifications (the more specific product requirements document rule), separately measure enqueue latency, and do not call XADD “delivered.” Drop expired OTPs, recheck event currency before replaying schedule/link messages, require fresh one-time passcode requests, and define a named operator action with cause id/dedup/audit for valid redrives. Add outage→recovery, changed-link, cancelled-interview and one-recipient-failed ACs. WhatsApp stays OFF under its already-settled enablement gates.

### I-10 — non-functional requirement targets lack a shared measurement and recovery acceptance profile

Evidence: business requirements document gives RPO24h/RTO4h, daily backups, 30-day retention, quarterly restore drills, and admin service-level agreement 99.9% 08:00–22:00 seven days/week; product requirements document says admin service-level agreement Monday–Friday. product requirements document specifies P50/P99 latency but no data size, load, duration or tested workload. The later approved lean deployment explicitly selects single-AZ/no failover with ~10–15min manual restore at 027:108–110 and cutover PITR drill at . Budget/topology is settled; there is no need to ask again whether to purchase Multi-AZ.

Scenario: a restore comes back in 20 minutes but pending notifications, refresh/session state or the latest confirmed assignment is missing; or application programming interfaces meet latency on seed data while exceeding it at launch load. It is unclear which success criterion the release should pass. Friday-night/Saturday downtime also counts differently in the two service-level agreement documents.

Decision: What exact launch workload, observation window, business-hour calendar, maximum lost business data and restored use cases define non-functional requirement acceptance on the chosen topology?

Proposed default: keep the approved budget architecture; pin a single product SLO table, use product requirements document's later weekday admin calendar unless explicitly changed, specify representative user/data/traffic sizes, and prove one measured end-to-end restore including login, current assignments/commissions, pending outbox/scheduled work and encryption/secrets. Record measured RPO/RTO plus reconciliation steps, not just a green database restore. Treat old backup-retention/topology differences as documentation reconciliation against 027, not an undecided infrastructure purchase.

## Superseded or settled items deliberately excluded from “open decisions”

- Historical 7-day access token → 004 30-minute access + 7-day idle/30-day absolute refresh.
- Historical admin seed-only/multi-factor authentication-every-dev-login → M16 any-admin-create/no-dev-multi-factor authentication; 027 production bootstrap+multi-factor authentication restores a prod requirement.
- Historical AES-SIV email/phone → explicit founder override in architecture/plaintext-email-phone md. Do not reopen encryption as an undecided choice.
- 003 prod one-time passcode debug default ON → 027:50–54 explicitly requires OFF for public production.
- INT-109 generic duplicate phone → 011 specific 409 with documented founder acceptance and compensating controls.
- 012 phone pending behind WhatsApp → v1.3 email-link fallback; 018 suppression is WhatsApp-only, not all phone verification.
- 006 reminder sweep/reminder sent → 007 per-instance scheduler; ≤24h confirmation-only Option A settled.
- 007 initial mandatory alert language → spec and  founder R4 metrics-only/deferred alert. Do not present that as an unapproved deferral. Delivery-done dedup before WhatsApp ON is an explicit residual at .
- Older outbox proposal says email after Commit is enough → authoritative later transactional-messaging-and-idempotency md records email on durable outbox.
- New production plan is authored/approved with cutover blocked, not evidence of a live production release; stale unchecked task lists do not override explicit changelog status.
