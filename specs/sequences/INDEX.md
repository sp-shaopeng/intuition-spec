# Detailed sequence records

This index links **43 detailed flow records**. They contain written requirements and retained historical implementation descriptions. They do not each contain a rendered diagram. Start with the [high-level sequence diagrams and coverage map](../../docs/sequence-diagrams.md) for the visual handoffs.

The [current requirements](../CURRENT-REQUIREMENTS.md) take priority over older examples. A corrected contract states what must happen; a historical source description states what was observed at an earlier time. Neither is a fresh application test. Named reviewers are AI roles, not employees. See [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Start with a visual overview

| Reader question | Diagram |
|---|---|
| Who requests, applies, chooses and assigns? | [Service overview](../../docs/user-flows.md#service-overview) |
| What does the browser send, and what does the server save or reject? | [Browser, server and saved result](../../docs/how-it-is-built.md#browser-server-and-saved-result) |
| How are interviews agreed and changed? | [Interviews](../../docs/sequence-diagrams.md#interviews) |
| Who confirms the first lesson and pays the agency? | [Billing](../../docs/sequence-diagrams.md#billing-and-agency-receipt) |
| What happens to a cancelled match and its debt? | [Cancellation](../../docs/sequence-diagrams.md#cancellation-and-financial-records) |
| How are access and contact changes protected? | [Accounts](../../docs/sequence-diagrams.md#account-access-and-recovery) and [contact changes](../../docs/sequence-diagrams.md#contact-changes) |
| What if a notification fails? | [Delivery recovery](../../docs/sequence-diagrams.md#notifications-and-delivery-recovery) |
| How does the founder direct AI coding and review? | [AI coding workflow](../../docs/workflow-walkthrough.md) |

The [coverage map](../../docs/sequence-diagrams.md#coverage-of-the-43-detailed-records) accounts for every record below, including deliberate text-only treatment of small settings, display rules and metric definitions. It is navigation coverage, not a claim that all acceptance cases pass.

## Accounts and access

| # | Flow | Record |
|---|---|---|
| 01 | Parent registration, required consent and session setup | [Parent registration](01-register-tutee.md) |
| 02 | Tutor registration, required commission acknowledgement and optional profile capture | [Tutor registration](02-register-tutor.md) |
| 03 | Sign-in and environment-specific administrator MFA | [Login](03-login-with-mfa.md) |
| 04 | Emailed recovery code and password reset | [Password recovery](04-forgot-reset-password.md) |
| 05 | Initial session check and protection for website requests | [Session bootstrap](05-csrf-bootstrap.md) |
| 29 | Authenticated password change and required session handling | [Password change](29-change-password.md) |
| 30 | Logical account deletion, session revocation and retained records | [Account deletion](30-account-deletion.md) |
| 31 | Permitted administrator creation and super-admin removal | [Administrator management](31-admin-create-delete-admin.md) |

## Posting, profiles and moderation

| # | Flow | Record |
|---|---|---|
| 06 | Parent submits a request for moderation | [Parent request](06-tutee-post-creation.md) |
| 07 | Tutor submits an optional listing for moderation | [Tutor listing](07-tutor-post-creation.md) |
| 08 | Staff approve or reject submitted content | [Moderation](08-admin-approval.md) |
| 32 | Permitted post edits, live closure and logical removal | [Post lifecycle](32-post-edit-and-close.md) |
| 33 | Role-scoped profile editing; separate protected contact changes | [Profile editing](33-profile-edit.md) |
| 34 | Tutor availability and public-showcase consent | [Availability and consent](34-tutor-availability-toggle.md) |

## Applications, assignment and commission

| # | Flow | Record |
|---|---|---|
| 28 | Tutor applies, withdraws or reapplies when eligible | [Applications](28-tutor-apply-withdraw.md) |
| 43 | Parent invitation and tutor response, with O01 still open | [Invitations](43-tutee-tutor-job-invitation.md) |
| 38 | Parent selection request followed by staff assignment | [Choose a tutor](38-choose-tutor-selection.md) |
| 39 | Staff-only applicant view | [Staff applicant view](39-admin-applicant-view.md) |
| 09 | Authorized staff create the assignment | [Match creation](09-admin-match-creation.md) |
| 10 | Activation locks the agreed first-month plan | [Activation](10-admin-match-activation.md) |
| 37 | Assigned participant contact access with separate website state gates | [Contact access](37-contact-reveal-post-assignment.md) |
| 11 | Actual tutor-to-agency receipt marks commission PAID and match COMPLETED | [Agency receipt](11-admin-mark-commission-paid.md) |
| 12 | Eligible cancellation, commission consequences and permitted reopening | [Cancellation](12-admin-match-cancel-cascade.md) |

First-session confirmation, invoicing and due handling remain required even though the inspected application only supports a shorter payment path. See [G02](../../docs/current-file-alignment.md#g02--the-billing-journey-stops-short-of-the-agreed-process) and the [billing diagram](../../docs/sequence-diagrams.md#billing-and-agency-receipt). Invitation initiation is allowed on a PENDING request; acceptance before approval remains O01. There is no newly invented background invitation timeout.

## Interviews

| # | Flow | Record |
|---|---|---|
| 13 | Parent requests an interview; current rules precede the older direct-request example | [Interview request](13-tutee-request-interview.md) |
| 42 | Time proposals, counter-proposals, agreement and staff handoff | [Time negotiation](42-interview-timeslot-negotiation.md) |
| 14 | Staff schedule the agreed Singapore time and meeting arrangements | [Scheduling](14-admin-schedule-interview.md) |
| 15 | Staff change a link and notify both participants | [Meeting-link changes](15-admin-set-meeting-link.md) |
| 16 | Completion, cancellation and either/both-person no-show outcomes | [Interview outcomes](16-interview-completion-paths.md) |
| 35 | Parent cancels an owned interview in eligible non-terminal states | [Parent cancellation](35-tutee-cancel-interview.md) |

Time input, link-change notice/audit, BOTH-no-show and reminder safeguards remain incomplete. The [interview diagram](../../docs/sequence-diagrams.md#interviews) labels those required behaviours; it does not close G04/G05/G09.

## Discovery, dashboards and follow-up

| # | Flow | Record |
|---|---|---|
| 23 | Public filters and bounded guest results | [Public browsing](23-public-browse-with-filters.md) |
| 36 | Login-gated tutor/assignment detail, distinct from public lists | [Protected detail](36-marketplace-detail-login-gated.md) |
| 40 | Bounded public showcase summaries and consent rules | [Public showcases](40-public-showcase-strips.md) |
| 17 | Parent match dashboard and permitted participant contact | [Parent matches](17-tutee-views-matches.md) |
| 18 | Tutor match dashboard, permitted contact and commission | [Tutor matches](18-tutor-views-matches.md) |
| 19 | Eligible review submission and duplicate prevention | [Submit a review](19-tutee-submits-review.md) |
| 20 | Protected tutor detail with separately public reviews | [Detail and reviews](20-public-tutor-detail-reviews.md) |
| 22 | Save or remove a favourite | [Favourites](22-favourite-toggle.md) |
| 21 | Notification centre, read state and permitted navigation | [Notifications](21-notification-bell-flow.md) |
| 41 | Staff analytics with defined dates, cohorts and receipt accounting | [Analytics](41-admin-analytics.md) |

## Shared technical boundaries

| # | Flow | Record |
|---|---|---|
| 24 | Request limits, their scope and trusted client-address handling | [Rate limits](24-rate-limited-endpoint.md) |
| 25 | Request-forgery protection for website changes | [Protected writes](25-csrf-protected-write.md) |
| 26 | Saved delivery intent, external dispatch and failure recovery | [Email delivery](26-email-fanout-on-event.md) |
| 27 | Readable email/phone storage with restricted disclosure | [Contact storage](27-encrypted-pii-write-and-lookup.md) |

Record 27 retains its historical filename, but **C13 requires readable contact storage**. The older encrypted-contact design is superseded in that scope; password hashing is separate. Record 26's earlier missing-consumer and missing-failed-queue observations are also historical. The inspected application has delivery workers, retry handling and a failed-message queue. Its remaining **G08/G09** delivery-loss and reminder-recovery gaps are described in [the architecture guide](../../docs/how-it-is-built.md#security-and-stability-mechanisms) and [current-file review](../../docs/current-file-alignment.md).

## History and evidence limits

The original June 2026 authoring records counted 24 production controllers on 2 June, then 26 after records 42–43 were added on 3 June. Those are dated inventory claims, not today's controller count or proof of complete journeys. The original descriptions drew on private implementation files; private source and usable source-line extracts are not published here.

Later contract corrections are identified in the numbered records, principally the 5 September 2026 reconciliation. The 18 October source comparison supplies the current disclosed implementation limits. Read each record's current contract first, then its historical description as history. The new high-level diagrams summarize those requirements without restoring private implementation examples.

[All high-level user flows](../../docs/user-flows.md) · [Current requirements and open decisions](../CURRENT-REQUIREMENTS.md) · [Diagram coverage](../../docs/sequence-diagrams.md)
