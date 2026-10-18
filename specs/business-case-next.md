# Business case — readiness, pricing and the next phase

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).


**Prepared by:** The founder with AI assistance. Historical business, product and review-lead roles were AI assistants, not employees or independent human reviewers.

**Record dates:** Original proposal, 29 May 2026; development updates, 2–3 June 2026; current-rule reconciliation, September 2026; public editorial clarification, October 2026.

**Current project position:** InTuition is not yet live. Launch is paused because the founder has been busy with life. No revenue, adoption or market-validation result is claimed here. The early descriptions of missing infrastructure and delivery services are historical snapshots, not an October inventory. See the [build overview](../docs/how-it-is-built.md), [testing and release](../docs/testing-and-release.md), and [current requirements](CURRENT-REQUIREMENTS.md) for the later distinctions.

## Contents

- [What this business case asks](#what-this-business-case-asks)
- [Historical development sequence](#historical-development-sequence)
- [The current commercial rules](#the-current-commercial-rules)
- [The proposed willingness-to-pay trial](#the-proposed-willingness-to-pay-trial)
- [Cost and capacity assumptions](#cost-and-capacity-assumptions)
- [The path from interest to collected commission](#the-path-from-interest-to-collected-commission)
- [Readiness before a real-user trial](#readiness-before-a-real-user-trial)
- [How to prioritize the next work](#how-to-prioritize-the-next-work)
- [What to measure](#what-to-measure)
- [Open product choices](#open-product-choices)
- [Historical limits and sources](#historical-limits-and-sources)

## What this business case asks

The question is whether the service can help a parent and tutor agree a workable arrangement, collect its disclosed fee, and do so at a cost the business can sustain.

A working demonstration answers only part of that question. It can show a product flow, but it does not establish that users want it, will pay for it, or can be supported reliably. The original proposal had no production traffic, acquisition-cost data, conversion rates or observed staff cost per match. Those absences make the pricing and operating model assumptions to test.

The intended service combines user choice with staff confirmation. Tutors can apply to active requests after completing the required profile fields, without publishing a listing. A parent can review applicants and choose a preferred tutor. Only the coordinator creates the assignment. Guests can browse bounded public results; contact remains private until the authorized participant and match-state conditions allow its display.

## Historical development sequence

| Date | What the record said at the time | Limit of the claim |
|---|---|---|
| 29 May 2026 | Core signup, posting, moderation, matching, interviews and manual commission recording could be demonstrated. The proposal identified missing delivery, recovery and payment work. | A development snapshot, not a public launch or a current list of gaps |
| 2 June 2026 | A development fix batch improved several user flows. Testing reported faults in match cancellation/contact access and other core behavior. | These were then-current findings; they should not be described as still present without a fresh check |
| 3 June 2026 | The record reported that tutor application, parent preference, admin assignment and controlled contact reveal worked together in development after fixes. | Evidence of a reported development flow, not a completed paid trial |
| September 2026 | The decision register clarified commission receipt, activation locking, contact gates and other requirements. | Documentation correction did not certify every implementation or deployment |

Later product requests included interview negotiation, onboarding improvements, parent invitations and date-range reporting. Their business value depends on the same underlying questions about usefulness, price and staff effort. More features do not by themselves validate the model.

## The current commercial rules

The **tutor pays the agency**. The parent's payment to the tutor for lessons is separate. An earlier trial proposal mistakenly referred to parents paying commission; that payer assumption is superseded throughout this public version.

At activation, the coordinator records the agreed hourly rate, first-month session count and hours per session. Multiplying those values and rounding to whole Singapore dollars gives the locked first-month fee. The default commission is 50% of that fee, also rounded to whole Singapore dollars. The permitted activation override is from S$1 up to the locked fee. Later changes to the teaching plan do not automatically rewrite that amount.

An example agreed plan of S$60 per hour, four sessions and 1.5 hours per session produces a S$360 fee and S$180 default commission. This is arithmetic, not a sale. The four-versus-4.33-week suggestion, whether a negotiated rate must stay within the posted range, and rules for partial receipts, refunds and early termination remain open.

Activation fixes the amount. First-session confirmation, or the authorized exception process after the seven-day timeout, gates invoicing and due handling. Only actual agency receipt allows the commission to be marked paid. An unpaid invoice or an agency-favour dispute decision is not collected revenue. Paid commission completes the platform's match workflow, not necessarily the teaching relationship.

See [INT-157](architecture/int-157-commission-first-month-locked-spec.md), [the commission specification](modules/commission/spec.md) and [current decisions C02–C03](CURRENT-REQUIREMENTS.md).

## The proposed willingness-to-pay trial

The original May proposal suggested a small trial in which staff would manually help users and follow up on payment. Its purpose was to learn whether the service and fee made sense before investing in fully automated collection. No such completed trial or result is claimed.

The historical draft proposed at least ten activated matches over six weeks, with a target of at least half paying commission within fourteen days and a stop-and-reconsider threshold below one quarter or after repeated payment chasing. These were planning choices, not statistically validated sample sizes or approved current billing terms. A future trial must define its denominator and start the payment observation window only at the appropriate invoicing/due stage, rather than treating activation as proof that commission is due.

Before running it, agree:

- Who is eligible, what service they receive, and exactly which fee and terms each tutor accepts.
- Which milestone starts each measurement window, including first-session confirmation and invoice delivery.
- How staff will verify receipts and distinguish unpaid, disputed, waived and received amounts.
- What feedback to collect from parents about usefulness and from tutors about price and value.
- How much staff time to record for matching, interviews, support, collection and reconciliation.
- What results justify continuation, revision or stopping, including how incomplete cases are handled.

The original suggestion to explore a lower commission band if acceptance was weak remains an experiment idea, not a silent change to the current 50% rule. Any alternative must be expressly agreed and disclosed before a participant commits.

## Cost and capacity assumptions

The original proposal estimated an eight-to-twelve-week phase and gave example runway thresholds of five and nine months. It also assumed small monthly infrastructure costs before a larger-scale design. Those estimates depended on an imagined work schedule, provider approvals and infrastructure choices that changed later. They are not a current delivery commitment or a budget grounded in the founder's actual available time.

The useful framework remains:

- Estimate the work using current scope and available founder time.
- Separate one-off build and setup effort from recurring operating cost.
- Include staff coordination, customer acquisition, support and payment chasing, even when the founder performs that work without a salary.
- Budget for delivery services, hosting, backups, monitoring and payment costs using current quotes before launch.
- Compare actual agency receipts with the cost of completing the introductions that produced them.

A single average commission is not enough to establish profitability. Collection rates, case complexity, cancellations and repeat support can change how much each arrangement costs. The founder's budget and willingness to continue funding the project were not established by the original AI-written forecast.

## The path from interest to collected commission

| Stage | What must happen | What it does not prove |
|---|---|---|
| Request and profile | A parent describes a need; a tutor completes the required profile | That the two are suitable for each other |
| Moderation and discovery | Staff approve public posts; permitted users can find relevant results | That post approval certifies teaching quality |
| Interest and preference | Tutor applies; parent reviews applicants and may request an interview or choose a preference | That a binding assignment exists |
| Assignment and activation | Staff check agreement, create the match and lock the agreed plan | That the first lesson happened or the fee is already received |
| First-session confirmation | The required confirmation or authorized exception process is completed | That an invoice was delivered or paid |
| Invoice and collection | The tutor receives the payment request and pays the agency | That a staff member's payment flag alone verifies settlement |
| Receipt and reconciliation | Staff or a verified integration checks receipt against the financial record | That teaching has ended or every later refund/dispute question is resolved |

Manual collection can be part of a deliberate small trial, but it still needs an auditable receipt check. A future payment integration should reduce effort and mistakes; it does not replace clear commercial rules. This record does not claim an integrated payment processor or automatic bank reconciliation is operating.

## Readiness before a real-user trial

The May proposal identified three pressing dependencies: recoverable data, a dependable way to deliver essential messages, and a release process that catches regressions before affecting users. The specific implementation has evolved since then. The outcome requirements remain useful:

| Area | Evidence needed before relying on it |
|---|---|
| Recovery | A documented restore that meets the agreed loss and recovery limits, not merely a backup configuration |
| Delivery | Essential messages are either delivered or visibly pending/failed and recoverable; disabled delivery is not reported as success |
| Release control | A reviewed change can be tested, deployed, identified and rolled back with compatible data |
| Access and privacy | Public and applicant views exclude private contact; participant and staff access checks work as required |
| Money handling | Fee terms are understood, first-session obligations are retained, and payment status is backed by agency receipt |
| User support | Someone can handle account recovery, failed introductions, disputes and requests concerning personal information |
| Measurement | Events and reports have a clear population, time period and meaning; test fixtures are not counted as customers |

These are project readiness requirements, not a legal-compliance certification. The privacy notice, consent record, retention and erasure policy, third-party data flows and incident-response process need appropriate review. Current decisions require logical business deletion and leave detailed retention timing open; the older proposal for a separate staff contact directory must not be treated as authorization for another unreviewed personal-data store.

## How to prioritize the next work

The enduring recommendation is to remove the obstacles to a small, measurable and supportable service before adding more features.

1. **Recheck current core behavior and recovery.** Resolve gaps using current evidence, not the old issue list. Include failed delivery and the ability to unwind an assignment correctly.
2. **Finish the commercial and privacy decisions that affect a trial.** In particular, clarify receipts, disputes and retention rather than leave staff to improvise.
3. **Prepare the trial and its measurement.** Recruit appropriate participants, disclose terms, assign support responsibility and define stopping criteria.
4. **Learn from service and pricing feedback.** Test whether coordination solves a real problem and whether tutors value it enough to pay.
5. **Automate where measured effort or mistakes justify it.** Payment collection, reporting and operational tools should follow the needs observed in the trial.

The original May document ranked backups, release infrastructure and email ahead of a larger operations console and landing-page enhancements. That was a response to the development state at that time. It is retained as the reasoning behind the sequence, not as proof those items are all still missing.

## What to measure

| Question | Useful measure |
|---|---|
| Can people find suitable choices? | Time from approved request to a suitable applicant and confirmed assignment |
| Does interest lead to teaching? | Separate counts for applications, interviews, assignments and confirmed first sessions |
| Will tutors pay? | Disclosed fee acceptance, objections, invoices, actual agency receipts and time to payment |
| Is the service economical? | Collected commission minus direct costs, alongside staff minutes per case and payment-chasing effort |
| Are users completing the intended journey? | Visits to signup, usable profiles, real requests and subsequent assignments, with defined cohorts |
| Is operating work reliable? | Failed/replayed messages, unresolved support cases, recovery exercises and reconciliation differences |

Targets such as delivery percentages, conversion rates and staff-time limits must be labelled as targets until observed. No improvement should be attributed to a landing-page change or an interview feature without evidence that supports that explanation. Small trials can reveal problems and help form better questions; they should not be presented as proof of market-wide behavior.

## Open product choices

**Applicant counts.** A high number might reassure some users and discourage others; zero may have a different effect. The historical idea to substitute interview interest for applications was rejected. If a count is shown, it must represent the actual thing named and comply with the approved visibility rules. Whether showing it helps is a research question.

**Staff confirmation.** User applications and parent preferences can reduce discovery work without removing staff responsibility for assignment. Fully self-service matching is a possible future model change, not the current rule.

**Interviews.** The current flow allows negotiation of times and staff scheduling of an accepted slot. It does not reveal private contact or guarantee a successful match. Measure the benefit and coordination cost.

**Payment automation.** Compare a properly reconciled manual trial with the cost and requirements of integration. Old provider-approval lead-time estimates should be rechecked when the project resumes; they are not promises from those providers.

**Public trust claims.** Use accurate moderation states. Do not invent testimonials, adoption numbers or a verified-tutor guarantee. Approved testimonial content and the meaning of wider verification remain open dependencies.

## Historical limits and sources

This is an editorial synthesis of the May–June business argument with September commercial and access rules made explicit. Private implementation logs and trial data are not included; no fresh application test, provider check, market study or customer experiment was performed for this edit.

Sources: [historical scope](SCOPE.md), [business update](business-update.md), [roadmap](roadmap-next.md), [current requirements](CURRENT-REQUIREMENTS.md), [commission specification](modules/commission/spec.md), and [the business and product overview](../docs/business-and-product.md). Historical AI review comments are planning inputs, not independent assurance or proof of demand.
