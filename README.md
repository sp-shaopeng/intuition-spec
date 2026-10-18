# InTuition

## How I use AI to deliver a software project

InTuition is my record of directing AI through a real project: understanding a business idea, writing specifications, giving AI tasks with clear limits, implementing changes, maintaining regression tests and reviewing evidence. I own the intent, priorities and business decisions. AI carries out much of the investigation, drafting, engineering and technical checking.

The product is a Singapore tuition marketplace for parents, learners, tutors and agency staff. This public repository contains its specifications and the evidence behind selected work. Application code and private operational records stay private.

**InTuition is not live. Personal commitments have delayed launch.** The application has development work and historical test results, but also known gaps. This portfolio distinguishes completed documentation, code reviewed and behaviour demonstrated by an actual run.

## Follow one change through the work

The intended chain is:

**Business need → specification and acceptance examples → independent specification approval → task with clear limits → implementation and regression tests → independent review → verified delivery**

A regression test checks whether a problem returns. Verification needs an identified version and an observed result; an AI message saying "done" is not that result.

| My responsibility | Work I delegate to AI | Evidence to examine |
|---|---|---|
| Define the problem and constraints | Investigate sources and propose requirements | Current rules, open decisions and pass/fail examples |
| Choose scope and priorities | Break approved work into owned tasks | Issue, specification version, allowed files and dependencies |
| Keep responsibility for the outcome | Implement, test and investigate failures | Change, test result and the version actually checked |
| Decide business tradeoffs and release intent | Independently challenge claims and recheck corrections | Findings, responses, remaining gaps and release evidence |

These are responsibilities, not a claim that I manually inspected every artifact or that every historical task followed the process perfectly. Review role names identify AI sessions, not employees.

## An application example: a form submitted too early

Issue #206 concerned a tuition-request form that submitted when the user only meant to enter Review. [Specification 008](specs/008-tutee-post-flow-bugfixes/spec.md) connected the problem to explicit acceptance cases. The recorded investigation traced it to a button changing from Continue to Submit during the same click. The implementation separates those controls, and a browser test checks that entering Review creates no request while deliberate submission creates one.

Later requirements added keyboard and text-composition cases. The October source review found those guards incomplete. The narrow fix remains present; the expanded requirement remains unfinished. No fresh browser pass is claimed.

**[Read the dated requirement, implementation, test and review trail](docs/how-i-work.md#example-a-review-step-that-submitted-too-early).** It shows both what AI helped produce and why the work was reopened.

## A separate completed delivery: this documentation

I asked for a public account with no application code or private values, chose Markdown over HTML, clarified that the workflow should explain project management, and requested six independent AI perspectives. AI inspected, drafted, delegated reviews and corrected findings, including an uptime requirement mistakenly described as staffing and an omitted fourteen-day billing escalation.

The [18 October publication at a04514d](https://github.com/sp-shaopeng/intuition-spec/commit/a04514d1b224cb8241cd979ecb525bd5ba127ccd) completed that documentation delivery after rechecks. Its inventory covered 362 Markdown documents, 346 specifications, 43 detailed sequences and 7,115 local links. [The documentation case](docs/how-i-work.md#example-a-completed-documentation-delivery) explains the format and privacy checks and their limits. This result does not establish software readiness.

## The business and application behind the work

Families describe their needs; tutors apply; staff review posts, coordinate interviews and confirm assignments. Contact access depends on assignment and match state.

The intended fee is a one-time **50% of the agreed first month's tuition fee, paid by the tutor to the agency**. First-session confirmation precedes invoicing; payment means the agency actually receives it. A parent's lesson payment is separate. Demand, recruitment, operating effort and several financial exceptions remain unproven or undecided.

The website uses React and TypeScript; the backend uses Java and Spring Boot, with PostgreSQL and reviewed Liquibase migrations. [Architecture](docs/how-it-is-built.md), [business rules](docs/business-and-product.md) and [21 user flows](docs/user-flows.md) explain the details.

## What to read next

- [How I work](docs/how-i-work.md): founder decisions, AI assignments and the two evidence trails.
- [Testing and release](docs/testing-and-release.md): the limited backend pass, unfinished frontend baseline and combined-lab blockers.
- [Current alignment](docs/current-file-alignment.md): missing behaviour, including billing, contact changes, guest controls and interview timing.
- [Improvement plan](docs/workflow-improvement-plan.md): proposed controls and lab maintenance, not rules already enforced by tools.
- [Specification index](docs/specification-index.md), [review record](docs/review-record.md) and [publication notes](docs/publication-notes.md): sources, corrections and publication boundaries.
