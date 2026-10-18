# InTuition

### A tuition business, from the first idea to the checks behind each change.

I am building InTuition for Singapore families, learners and tutors. The idea is to make finding a tutor easier without leaving families to manage every introduction, interview and follow-up themselves.

This repository explains the business, the product and how I work with AI to build it. It contains specifications and reading guides. It does not contain the application’s source code.

**InTuition is not yet live. I have put the launch on hold because I have been busy with life.** Development work and test records exist, but they are not evidence of a public launch, paying customers or reliable service under real customer traffic.

**The specifications are ahead of the current application in several areas.** A fresh independent comparison found gaps in guest filtering, contact changes, interview times and the first-session billing flow. The [current-file alignment review](docs/current-file-alignment.md) separates the agreed requirements from what the inspected files actually support.

**[Business and product](docs/business-and-product.md)** · [User flows](docs/user-flows.md) · [Workflow improvement plan](docs/workflow-improvement-plan.md) · [Testing](docs/testing-and-release.md) · [All specifications](docs/specification-index.md)

## Why I want to build it

A family needs more than a list of tutor profiles. They need someone who fits the subject, level, budget, location and schedule. They also need to understand what happens after they express interest: who arranges the interview, who confirms the assignment and when contact details can be shared.

A tutor needs a clear request, a fair way to apply and an explanation of the agency fee before accepting work. The agency needs a shared record of decisions so that a missed message or a change of staff does not leave the arrangement unclear.

InTuition brings those steps into one service. People can browse, post requests, apply and express a preference. Agency staff review listings and confirm assignments. The aim is a useful online service with human responsibility at the points where a decision matters.

That is the product vision. Customer demand, willingness to pay and the amount of staff time needed for each successful arrangement still need to be tested after launch. The first customer group and how to recruit enough suitable parents and tutors are also open decisions. I am not claiming that the business model has already been proven.

## Who it is for

| Person | What they need | What InTuition is designed to help them do |
|---|---|---|
| Parent or learner | Find a suitable tutor and understand the next step | Post a request, browse tutors, arrange an interview and choose a preferred applicant |
| Tutor | Find suitable work and understand the terms | Maintain a profile, apply to requests, respond to invitations and see the agency commission |
| Agency staff | Keep decisions and follow-ups consistent | Review posts, handle interviews, confirm assignments, record commission receipts and manage exceptions |

The product is specific to tuition: subject and education level, teaching area, lesson length, frequency, tutor availability and budget all affect a useful match. A general directory would not answer these questions on its own.

## How the business is intended to earn money

The documented model is a **one-time agency commission paid by the tutor**, normally **50% of the agreed first month’s tuition fee**. Parents paying tutors for lessons is a separate transaction. The application records agency commission; it does not establish that money has been collected merely because a parent paid for a lesson.

The first-month fee comes from the agreed hourly rate, the agreed number of sessions in that month and the length of each session. For example:

| Agreed item | Illustrative amount |
|---|---:|
| Hourly tuition rate | S$60 |
| Sessions in the first month | 4 |
| Hours per session | 1.5 |
| Agreed first-month fee | S$360 |
| Agency commission at 50% | S$180 |

This is an example of the rule, not a price quote or revenue result. When staff activate the arrangement, the agreed plan and commission amount are fixed. Later changes in lesson frequency do not silently recalculate that amount. Confirmation of the first lesson is required before the invoice/due process. The commission is marked paid when the agency receives it.

“Match completed” means that the agency’s matching and commission process has completed. It does **not** mean that all tuition lessons have finished.

The current requirements still leave some commercial decisions open, including the suggested weeks-per-month figure, restrictions on agreed rates, and treatment of refunds, partial receipts and early termination. These need an explicit decision before the affected flows can be signed off. [Read the business rules and open questions](docs/business-and-product.md).

## What the intended product journey looks like

1. **Discover.** A visitor can browse the public information and a limited set of listings. Personal contact details are not part of public results.
2. **Describe the need.** A parent or learner creates a tuition request. A tutor completes a profile and can apply to relevant requests without needing a published tutor listing.
3. **Review.** Agency staff check posts before public publication. Being registered or having a reviewed post is not a blanket promise that a tutor has passed every possible background check.
4. **Talk and choose.** Participants can use the interview and invitation flows. A parent can express a preferred tutor. Interest, an interview or a selection request does not itself create the final assignment.
5. **Confirm the arrangement.** Staff assign the tutor and activate the agreed plan. Only authorized participants in the required match state can see the other person’s contact details.
6. **Follow through.** The first lesson is confirmed, the commission process proceeds and staff record the agency’s receipt of payment. Cancellations and changes must leave related records in a consistent state.

[Read the 21 main user flows](docs/user-flows.md) · [Walk through the business](docs/business-and-product.md#the-main-product-journey) · [Read the current decisions](specs/CURRENT-REQUIREMENTS.md)

## How the frontend and backend fit together

The **frontend** is the website people use. It is built with React and TypeScript, with Vite used during development. It presents the parent, tutor and staff screens and helps people understand what they can do next.

The **backend** is the part that checks requests and applies the rules. It uses Java 21 and Spring Boot. A hidden button is not enough protection: the backend must check who is acting, which record they own and whether the requested action is allowed at that point.

PostgreSQL holds the business records. Liquibase keeps a reviewed history of changes to the database structure. Redis supports short-lived state and background work. Email delivery uses Resend; WhatsApp has documented support but remains off until its enablement conditions are met. Planned AWS deployment is described separately from the development setup and from proof of production readiness.

The current requirements say contact fields are stored in plain text, replacing an earlier encryption proposal. Access restrictions and access records still apply. This repository does not claim that the original encryption proposal is the implemented design, or that the project has received a security or legal certification.

[Read the architecture and its limits](docs/how-it-is-built.md).

## How I use AI without handing it the decisions

AI helps with writing specifications, implementing changes, examining the work and finding test cases. It does not make an unresolved business decision become approved. It also does not make a change correct by saying that the work is finished.

The working process is:

**Business need → specification and pass/fail examples → independent specification approval → tracked issue → failing check → implementation and tests → independent result review → release decision**

| Step | What I expect before moving on |
|---|---|
| Define the need | A clear user problem and a reason to change the product |
| Write the specification | Business rules, limits and examples of what must and must not happen |
| Approve the specification | Reviewers other than the author approve the corrected plan and examples before building; unresolved business decisions block dependent work |
| Track the issue | A bounded task connected to those requirements, with dependencies visible |
| Implement and test | Reproduce the defect or add the acceptance check, make the change, and check both the new behaviour and existing behaviour it might break |
| Review the result | Reviewers inspect the change and test evidence from the business, product and engineering perspectives, including failure cases |
| Close and release | Findings resolved, the relevant checks recorded and the documentation updated |

When a review or test fails, the task goes back to the relevant step. A requirements change can require changes to the design and tests as well as the implementation. A passing old test does not prove a newly clarified requirement.

The issue IDs, review records and requirement lists in the specification collection show how that process developed. They also show inconsistencies and follow-up work. They are not all equivalent to a freshly verified result. Named review roles in those working records refer to AI-assisted roles; they should not be read as a claim that I employ a large business or engineering team.

[See the workflow and examples](docs/how-i-work.md), then read the [improvement plan](docs/workflow-improvement-plan.md) for linking each rule to its change, test evidence and independent review.

## How I check that changes have not broken the product

There are several kinds of checks because they catch different mistakes:

- **Backend checks** exercise business rules, permissions and interactions with stored records.
- **Frontend checks** exercise what the screens show and how they respond to user actions.
- **Playwright checks** combine browser interactions with direct API workflow tests. They do not establish the complete website journey or the required first-session billing flow.
- **Agreement checks between repositories** look for differences between the backend’s documented requests and the expectations used by other parts of the project.

The configured coverage targets are 97.5% of lines and branches for the backend, and 97.5% of lines and 88% of branches for the frontend. Coverage measures how much of the measured code the tests exercise. It does not prove that every business rule is right, that every file is included or that the service is ready to launch.

The inspected source inventory contains **182 backend test files, 177 frontend test files and 57 Playwright test files**. These are file counts, not counts of passing test cases. A later backend baseline passed 2,222 tests across its main and integration phases, but excluded the live regression lab. The frontend baseline did not finish. [Read the exact limits of that check](docs/testing-and-release.md#follow-up-baseline-check).

Useful regression questions are concrete: can a second click create a duplicate assignment; can an unassigned person see a phone number; can an old interview reminder still fire after rescheduling; can an unpaid commission be counted as received; can a failed message be retried without creating a second business action?

[Read the checks, known limits and release work still needed](docs/testing-and-release.md).

## Where the project stands

The private implementation and development records contain substantial work. The public specification set also includes proposals, old decisions and later corrections. The September 2026 [current requirements register](specs/CURRENT-REQUIREMENTS.md) explains which decisions take priority and which questions remain open.

Before a public launch, the changed requirements need to be checked against the implementation, the full relevant tests need to pass, delivery and recovery need to be verified, and the remaining business and operating decisions need to be settled. My personal reason for delaying launch is that life has been busy; that is separate from those readiness checks.

## How to read this repository

| Start here | Then go deeper |
|---|---|
| [Workflow walkthrough](docs/workflow-walkthrough.md) | Each step, the record it needs and what happens when a check fails |
| [Workflow improvement plan](docs/workflow-improvement-plan.md) | How to improve project planning, AI tasks, spec-driven development, reviews and maintenance of the regression lab |
| [User flows](docs/user-flows.md) | 21 main journeys, their exceptions and current limits, linked to all 43 detailed sequences |
| [Business and product](docs/business-and-product.md) | Vision, customers, agency responsibilities and commercial rules |
| [How it is built](docs/how-it-is-built.md) | Frontend, backend, storage, messages and deployment plans |
| [How I work](docs/how-i-work.md) | Specifications, issues, AI assistance and reviews |
| [Testing and release](docs/testing-and-release.md) | Test layers, real limits and release checks |
| [Specification index](docs/specification-index.md) | The complete public document collection |
| [Publication notes](docs/publication-notes.md) | What was removed or rewritten before publication |
| [Review record](docs/review-record.md) | Six independent AI reviews, findings and changes |
| [Research notes](docs/research-notes.md) | External guidance used to prepare this publication |
| [Earlier design directions](docs/design-directions.md) | Markdown record of two historical design alternatives, with their unapproved claims clearly labelled |
| [Cover concept](docs/cover-concept.md) | Markdown record of the earlier document-cover concept |

This is a public record of the thinking and the work. The application source, credentials, raw account evidence and original private Git history are not part of it.
