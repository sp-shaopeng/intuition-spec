# Business and product

InTuition is a Singapore tuition marketplace built around a service: help a parent or learner find a suitable tutor, agree an arrangement, and understand what happens next. The website supports that service from the first search through applications, interviews, assignment and commission tracking.

**InTuition is not yet live. Launch is on hold because the founder has been busy with life.** This is a record of the product idea, development work and remaining decisions. There are no public-launch results, customer-growth figures, collected-revenue claims or proven business outcomes behind this showcase. Development examples are demonstrations, not evidence of marketplace activity.

## The problem the product is meant to solve

Finding a tutor involves more than finding a name. A parent needs someone who teaches the right subject and level, can travel to the right area, is available at suitable times, and charges an affordable rate. They also need enough information to decide whether that person is a good fit. Tutors face the other side of the same problem: requests may not match their subjects, schedule, location or rates, and an enquiry does not necessarily become a lesson.

InTuition's product thesis is that clearer information, a short conversation before commitment, and a coordinator who follows an arrangement through can reduce this uncertainty. It is a hypothesis to test with real users. The project does not claim that its interviews improve learning results, that its matching is better than competitors, or that moderation guarantees teaching quality.

Tuition provides a concrete setting for this work. A request has identifiable requirements—subject, school level, area, budget, lesson length and frequency—and a successful introduction has an observable next step: the first lesson. The operational work around that introduction gives the software a purpose beyond publishing advertisements. The original [business requirements](../specs/BRD.md) and [product requirements](../specs/PRD.md) describe this rationale; their market-size estimates and sales targets are planning assumptions, not independently established results.

## Who it serves

| Person | What they need to achieve | How the product is intended to help |
|---|---|---|
| Parent arranging tuition | Find a suitable tutor for a child's subject, level, schedule and budget | A structured request, relevant tutor profiles, applicant comparison, an optional interview and a clear assignment process |
| Adult learner | Find help that fits a particular learning need and available time | The same request and matching process, with the learner acting for themselves |
| Part-time tutor | Find suitable work around other commitments | A reusable profile, subject and area preferences, availability, applications and clear status updates |
| Full-time or specialist tutor | Build a sustainable teaching schedule | Discover relevant requests, show experience and teaching scope, and manage several arrangements |
| Agency coordinator | Turn interest into a workable arrangement and follow it through | Moderation queues, interview scheduling, assignment controls, contact-access rules and a commission ledger |

These are intended users and needs drawn from the specification. They are not claims that each group has already adopted the service. In the detailed documents, **tutee** means the account requesting tuition, usually a parent or adult learner. **Admin** means an authorized agency coordinator.

## A marketplace with an agency service

InTuition has two sides: people requesting tuition and tutors offering it. Tutors can discover requests and apply; parents can review applicants, invite a tutor or express a preference. This is the marketplace part.

The agency still has a distinct job. It reviews posts before public publication, coordinates the introduction, and confirms the assignment. A parent's choice is a request to proceed; it does not itself create a binding match. Only the coordinator creates the assignment. This keeps one person responsible for checking that both parties understand the arrangement.

The design evolved from entirely staff-driven matching to this combination of user choice and agency confirmation. A tutor now uses a complete profile as their main identity and may apply without publishing a separate listing. A listing is an optional way to present an offering. The change is recorded in [the assignment lifecycle](../specs/PRD.md#20-assignment-lifecycle--free-marketplace-m18) and the [current decision register](../specs/CURRENT-REQUIREMENTS.md).

This model creates a business tradeoff. Personal coordination may make an unfamiliar service easier to trust, but it costs staff time. More applications do not automatically mean more successful introductions, and more introductions do not automatically mean more collected commission. The project needs to measure each step before assuming it can grow economically.

## The main product journey

The [service sequence](user-flows.md#service-overview) shows the parent, tutor and coordinator handoffs. The separate [billing sequence](sequence-diagrams.md#billing-and-agency-receipt) shows first-session confirmation, invoicing and who pays whom; [cancellation](sequence-diagrams.md#cancellation-and-financial-records) has its own financial safeguards. These are required journeys with visible implementation gaps, not records of a live service.

### 1. Understand the available choices

Visitors can browse a limited set of public tutor and tuition-request results. The agreed requirement also allows guest filtering and pagination, but the current website still hides those controls until login. This is an implementation gap, not a change to the public-access rule. More detailed views and actions have separate sign-in requirements. Public cards use permitted profile information and general location rather than exposing contact details.

The landing showcase is meant to help a visitor understand the type of supply and demand the service supports. In a future live service, any activity or availability claim must come from appropriate real records. In this project showcase, sample records are demonstrations. A busy-looking demonstration is not evidence of real demand.

### 2. Describe the tuition need or teaching offering

A parent supplies the subject and level, general location, preferred availability, budget, lesson duration and frequency. A tutor supplies their teaching scope, qualifications, rates, areas and availability through their profile and any optional listing.

The intended onboarding flow explains the one-time agency fee and requires tutors to acknowledge it before account creation. It includes the calculation, who pays and when the fee becomes due. Profile capture during signup is optional and may be completed later; applying still requires the approved profile information.

Posts enter a moderation process. The current coordinator screen offers approval or rejection with a reason. Owners can edit and resubmit where the post's state allows it; there is no separate “request changes” action in the inspected screen. This is a review of the submitted information, not a universal certification of the tutor. The meaning and evidence required for a broader “verified tutor” badge remain an open decision.

The current requirements also allow a parent to edit their own request and initiate an invitation or interview while the request is awaiting approval. Public publication and assignment still require approval. Whether a tutor may accept an invitation before that approval is complete remains unresolved. See [the request flow](../specs/021-tutee-request-flow/spec.md) and [open decisions](../specs/CURRENT-REQUIREMENTS.md#open-decisions).

### 3. Express interest and compare applicants

A tutor can apply to an active request when their profile meets the required completeness rules. The parent can review applicants for their own request without receiving private contact details. They can request an interview or choose a tutor for the coordinator to follow up with.

One tuition request can have at most one pending or active match. A tutor can take on several arrangements. When an assignment is made, the request leaves the open discovery pool and the other applications are updated so those tutors are not left believing the job is still available. A permitted cancellation can reopen the request. The [applications](../specs/modules/applications/spec.md) and [matching](../specs/modules/matching/spec.md) documents explain those rules.

### 4. Have a short conversation before commitment

The interview feature gives the parent and tutor a way to discuss suitability before committing to an arrangement. The intended conversation is up to 15 minutes. The product supports proposals and counter-proposals for times, an agreed slot, a meeting link, and outcomes such as completion, cancellation or no-show. After the participants agree on a time, the coordinator schedules the interview and provides the meeting link.

An interview does not by itself release contact details or create an assignment. Scheduling and rescheduling also need consistent updates and reminders so neither participant follows an obsolete time or link. The [interview specification](../specs/modules/interview/spec.md) preserves the original idea and points to the later negotiation design.

### 5. Confirm the arrangement

The coordinator checks that both parties wish to proceed, creates the assignment, and activates the arrangement. Activation records the agreed lesson plan used for commission: hourly rate, sessions in the first month, and hours per session.

Contact access depends on both participation and match state. The current interface rule reveals contact only to authorized participants in active or completed matches; it hides it while a match is pending. Approval, an invitation, an interview, or choosing a tutor is not enough. The coordinator's access is separately authorized and audited.

### 6. Confirm the first lesson and collect the agency fee

The required journey asks the parent or learner to confirm that the first session occurred before invoicing and deciding when commission becomes due. **These confirmation, invoice and due-handling stages are absent from the current frontend and backend.** The inspected application instead supports activation followed by staff recording commission as paid. That shorter flow does not satisfy the full requirement.

The retained rules require a seven-day follow-up for missing confirmation, but disagree on when that period starts: activation in PRD §7.7.1 and AC-11, sending the confirmation email in AC-05, or receiving the notification in BRD BR-010. That starting event is tracked as O13 in the [decision register](../specs/CURRENT-REQUIREMENTS.md#open-decisions). Staff must follow up, with confirmation, an authorized override, or cancellation as possible outcomes. The source also disagrees on whether staff may override confirmation before timeout; this permission remains unresolved as O14. Separately, open decision O12 concerns the reminder’s delivery deadline and the six-hour detection schedule. A timeout is not proof that a lesson occurred or that money arrived. See the [alignment review](current-file-alignment.md) for this and the other confirmed implementation gaps.

The tutor pays the agency's commission. Staff record payment only after the agency receives it. Recording that receipt completes the platform's match workflow; it does not mean all teaching between the tutor and learner has ended. Reviews, notifications and the participants' dashboards provide supporting follow-up around the arrangement.

These steps describe the current intended contract. The delivery status of each rule is a separate question, addressed below.

## How the business would earn revenue

The current default is **one commission per match, paid by the tutor to the agency, equal to 50% of the agreed first month's tuition fee**. It is not a recurring percentage of every later month's lessons. Parent-to-tutor lesson payments are a separate transaction.

The amount is locked when the coordinator activates the arrangement:

| Calculation | Current rule |
|---|---|
| Agreed first-month tuition fee | Round the agreed hourly rate × agreed sessions in that month × hours per session to whole Singapore dollars |
| Default agency commission | Round the locked first-month fee × 50% to whole Singapore dollars |
| Activation override | A permitted coordinator override can set an amount from S$1 up to the locked first-month fee; it must be represented clearly as an override |
| Later lesson-plan changes | They do not automatically recalculate the locked commission |

For example, **S$60 per hour × 4 agreed sessions × 1.5 hours = S$360** for the first month. The default commission is **S$180**, owed by the tutor to the agency. This is an illustrative calculation, not a reported sale.

Three events must stay distinct:

| Event | What it establishes |
|---|---|
| Arrangement activation | The agreed plan and amount are fixed |
| First-session confirmation, or the authorized exception process | The prerequisite for invoicing and due handling has been satisfied |
| Agency receipt of the tutor's commission | The commission may be recorded as paid and the match workflow completed |

A parent paying for a lesson does not prove that the agency has received its commission. Winning a dispute does not turn unpaid debt into cash received. The amount lock also does not answer every later financial question: partial payments, unsolicited advance payments, refunds, early teaching termination and post-dispute unpaid states still need explicit rules.

Two smaller commercial choices are also open: whether a weekly schedule should initially suggest four or 4.33 weeks when deriving a month, and whether an agreed hourly rate must stay inside the advertised range. The final session count is confirmed at activation; a flexible schedule requires a positive count entered by the coordinator, with no invented default. The example above uses an explicitly agreed count so it does not depend on that unresolved multiplier.

These rules come from [the current decision register](../specs/CURRENT-REQUIREMENTS.md), [the locked first-month commission design](../specs/architecture/int-157-commission-first-month-locked-spec.md), and [the commission specification](../specs/modules/commission/spec.md). Older documents contain different shorthand and examples; they should not override the later decisions.

## What has been built, and what the record does not prove

The repository contains more than a proposal. It records successive implementations, testing, reviews and refinements of the product. At the same time, the record contains dated snapshots. The September requirements reconciliation explicitly says that corrected wording is not fresh proof that every corresponding behavior has been implemented or deployed.

| Area | What the development record supports | Boundary to keep in mind |
|---|---|---|
| Core service | Registration, profiles, requests and listings, moderation, staff matching, interviews, match dashboards, in-app notifications, favourites, reviews and commission recording were reported as built in development | A development delivery does not establish a public launch or customer adoption |
| User-led discovery and selection | Later records describe tutor applications, parent selection, admin assignment and controlled contact reveal as an end-to-end development flow | Older documents describing staff as the only source of interest are superseded |
| Product refinement | Later specification packages cover onboarding, more precise availability, parent invitations, request editing, account recovery, subject/level choices and mobile interface fixes | Historical “shipped” labels retain their original scope; later corrections require separate verification |
| Current commercial contract | The tutor-to-agency payment rule and activation lock are represented in the inspected implementation | First-session confirmation, invoice generation and the due-handling workflow are absent; they need implementation and tests before the full contract is met |
| Payment collection | The documented core can record commission received by staff | Manual recording is not an integrated payment processor or automated bank reconciliation |
| Email and messaging | Later work covers delivery and reminder mechanisms, beyond the early notes saying email was missing | Delivery design is not proof of current production delivery; WhatsApp remains off until its enablement conditions are satisfied |
| Public launch | A production deployment plan and infrastructure work exist | The project is not yet live; authored infrastructure is not evidence of a running production service |
| Testimonials and wider verification claims | The content requirements are defined | Approved source content, consent and a precise meaning for any verification badge remain dependencies |

The [historical scope](../specs/SCOPE.md), [business update](../specs/business-update.md) and [business case](../specs/business-case-next.md) explain the development sequence. The [current requirements](../specs/CURRENT-REQUIREMENTS.md) resolve policy conflicts. The [production specification](../specs/027-aws-production-deployment/spec.md) separates infrastructure authoring from production readiness. Readers should use those distinctions rather than treating every old “live” or “production” label as a current launch claim.

## Economics to validate after work resumes

The proposed 50% commission is a pricing assumption. The repository does not demonstrate that tutors will accept it, that enough parents will complete a match, or that the service can collect it at an acceptable cost. The next business work is to test those assumptions in a small, properly prepared trial—not to treat a forecast as revenue.

The useful measures would follow the actual service:

| Question | Evidence to collect |
|---|---|
| Can a parent find a suitable tutor? | Time from approved request to a suitable applicant and confirmed assignment |
| Do both parties follow through? | Applications, interviews, assignments and first lessons, counted separately |
| Will tutors pay the quoted fee? | Acceptance of the disclosed terms, objections to price, invoices and actual agency receipts |
| Does the service earn enough for the work involved? | Collected commission per match alongside coordinator time, payment-chasing time and direct delivery costs |
| Can it grow without increasing manual work at the same rate? | Staff minutes per completed introduction, rework, cancellations and disputes |
| Does the public experience help people get started? | Visits leading to registration, usable profiles and genuine requests, rather than traffic alone |

For planning, collected commission depends on the number of arrangements that reach a payable stage, the agreed fee for each, and how much the agency actually receives. The money left to support the business must then cover coordination, support, acquiring users, messaging, infrastructure and any payment costs. The founder's time matters even before it becomes a payroll expense.

The original documents contain ambitious inventory, match and monthly-revenue targets. They also contain cost and time estimates written before later changes. Those figures describe intentions at a point in time. This showcase does not present them as traction, validated forecasts or current operating costs.

A future trial should check both sides of the service: whether parents find the process useful and whether tutors accept and pay the agency fee. Its success criteria and stopping conditions should be agreed before it begins. It must also use the correct payer—the tutor—rather than repeat an older trial note that mistakenly described parents paying commission.

The first group of customers and the way to recruit them are also open decisions. A trial needs a specific learning need, subject or area, and enough tutors whose availability fits it. Recruitment effort, cost and requests that never find a suitable tutor need to be recorded for both sides. The existing documents do not establish a tested recruitment channel or a launch commitment.

## Product decisions still worth making

The remaining work is partly implementation and partly deciding what the service promises. Before restarting launch, the most consequential product questions are:

- Whether the fee and amount of human coordination create enough value for tutors to pay sustainably.
- How cancellations, early endings, disputes, refunds and partial receipts change the financial record without silently rewriting an agreed amount.
- What tutor moderation actually checks, and which statements the service can support with evidence.
- How long different records must be retained and how a person's erasure request is handled.
- Whether an invitation may be accepted while its underlying request is still awaiting approval.
- How to deliver the required old-phone warning, how to measure the first-session timeout deadline, and who follows up on delivery failures. The critical-message categories and duty to preserve unfinished work are already specified.

These remaining decisions do not make the settled requirements optional. Critical-message timing and preserving failed deliveries are already required; the old-phone warning transport and the first-session timeout clock still need decisions. The [decision register](../specs/CURRENT-REQUIREMENTS.md#open-decisions) keeps those boundaries visible.

## How this explanation was prepared

Project facts come from the specification archive linked throughout this page, interpreted using the latest decision register and the founder's statement that launch is paused. This is a synthesis of the record, not a fresh runtime audit.

The structure follows the [U.S. Small Business Administration's business-plan guidance](https://legacy.sba.gov/business-guide/plan-your-business/write-your-business-plan): explain the customer, value offered, operating work, costs and revenue. The distinction between intended needs and demonstrated evidence follows [GOV.UK's guidance on understanding user needs](https://www.gov.uk/service-manual/user-research/start-by-learning-user-needs), which treats opinions not grounded in user research as assumptions to test. These external sources informed the writing; they do not validate InTuition's market demand or business model.
