# Testing and release

InTuition is **not live**, and life commitments have delayed launch. This page describes the test inventory and automation found in the private repositories on 18 October 2026. **The application test suites were not rerun for this publication.** Counts below are files, not executed test cases or fresh passing results.

## What is present

| Layer | Tracked inventory | Purpose |
|---|---:|---|
| Backend test sources | 182 files: 125 named as tests and 57 as integration tests | Business rules, permission checks, controller behaviour, data access and running-system checks. |
| Backend regression lab | 49 of those 57 integration-test files | Exercises the running API from outside the application. This is a subset of the backend total, not an additional 49 files. |
| Frontend tests | 177 files | Components, forms, state and browser-facing behaviour in a simulated browser environment. |
| Playwright specifications | 57 files mixing browser interaction and direct API checks | Some exercise real screens; others send requests directly to the server and do not prove the corresponding website journey works. |

The backend uses JUnit, Mockito and Testcontainers. Testcontainers starts real services for selected integration tests, allowing database behaviour to be checked rather than replaced with a mock. The frontend uses Vitest 4.1.7 and React Testing Library. The separate browser-test lockfile resolves Playwright 1.60.0.

The inventory was counted from tracked filenames in each repository, without executing the tests. Parameterised tests, generated cases, skipped tests and browser variants can change an execution count, so file totals should not be presented as a number of passing tests.

The later independent review reproduced these file counts. It also checked what the tests actually do: the central matching scenario opens the landing page, then performs its business steps through direct server requests. Its contact check uses an assigned PENDING match, while the website requires ACTIVE or COMPLETED. That test cannot prove the matching screens, the website's narrower contact rule, or the missing first-session billing journey. The request-form case study below does operate the browser form. [Playwright explains this distinction between API requests and page interaction](https://playwright.dev/docs/api-testing).

Some test expectations also preserve behaviour that conflicts with newer requirements: guest filters stay hidden, and interview input uses the computer's local timezone. Passing those tests would not establish compliance. The [current-file alignment review](current-file-alignment.md) records the affected requirements and the checks still needed.

## The checks configured in CI

Continuous integration, or CI, runs automated checks when code changes are submitted. GitHub Actions supports checks on pushes and pull requests and makes their results available during review. [GitHub's CI documentation](https://docs.github.com/en/actions/get-started/continuous-integration).

| Workflow found | Configured checks | Important boundary |
|---|---|---|
| Backend | Java build, formatting check, unit and selected integration tests, then coverage thresholds of 97.5% for both lines and branches. | The running-system regression lab is explicitly excluded from this separate-repository workflow. It belongs in the combined workflow. |
| Frontend | Lint, TypeScript/build checks, tests and coverage thresholds of 97.5% for lines and 88% for branches. | Two cross-repository taxonomy checks depend on the backend checkout being available. They can skip when that prerequisite is absent. |
| Combined application | Starts the frontend, backend and supporting services; checks health and the running backend's build identity; runs short browser and server checks on pull requests in the browser-test repository, with broader runs on other triggers. | It checks out the application repositories' development branches. This does not establish a required combined check of the exact changes in every frontend or backend pull request. Access to those private repositories is required, and no current successful run was established for this publication. |

Coverage measures which instrumented lines and decision branches ran during tests. It does not measure whether requirements are complete, assertions are useful, or the interface works for a person. Both coverage configurations exclude some files or categories, so these thresholds are not percentages of every file in the application.

The combined workflow records the versions under test and checks that the running backend corresponds to the expected build. This addresses a specific risk: tests can appear to pass while exercising an older installation. Failure reports are retained, and the browser configuration retains traces, video and screenshots on failure.

## Limits that remain visible

- **Browser coverage is not established by configuration alone.** The Playwright configuration declares desktop Chromium, WebKit and a mobile-sized Chromium project. The inspected CI setup installs Chromium only. A complete successful run of the declared browser matrix should not be inferred from those files.
- **Some safeguards are conditional.** The frontend taxonomy checks can skip without the sibling repository. A comparison between combined-test settings and local settings also skips when its comparison baseline is unavailable.
- **Earlier contract-testing plans exceed the inspected implementation.** Older strategy documents propose Pact and a contract broker. The inspected dependency manifests and CI workflows do not establish those tools as implemented gates. The browser repository does contain API inventory and contract-comparison utilities; these are a different approach and are not proof of a Pact setup.
- **Old versions and labels need context.** Earlier draft documents mention other React or PostgreSQL versions. Historical “shipped” or “deployed” labels describe development work and must not be read as a current public launch announcement.
- **Later requirement corrections need implementation as well as evidence.** The request form lacks the corrected control-specific Enter and input-composition guards. Its existing browser tests cover only part of the keyboard contract. Some no-show tests also require optional database access that the combined workflow does not enable; they do not establish coverage for both participants missing an interview.
- **Message failure handling needs correction and tests.** The technical review found paths that can treat disabled email delivery or a missing provider key as success. It also found a queue size limit that does not first protect all unread or unfinished work. The current requirements call for pending messages to remain available until delivered or explicitly recorded as permanent failures. Those paths need correction and failure-and-recovery tests before launch.

These limits are retained to keep the showcase useful to a reviewer. The [current requirements](../specs/CURRENT-REQUIREMENTS.md), [development workflow](../specs/domain/dev-workflow.md) and [historical test strategy](../specs/domain/backend/test-strategy.md) should be read together.

## Why more than one kind of test matters

A component test can check form validation quickly. A database test can check that competing writes preserve a rule. A browser test can show whether someone can actually complete a journey. Review can expose a requirement that none of those tests currently covers.

The [request-form case study](how-i-work.md#example-a-review-step-that-submitted-too-early) shows this distinction: a real browser exposed a submission bug that the simulated environment did not reproduce. Playwright's guidance also emphasises visible user behaviour and isolation between tests; these are useful principles for assessing tests rather than relying only on their quantity. [Playwright best practices](https://playwright.dev/docs/best-practices).

When a check fails, the intended loop is to reproduce it, determine whether the cause is implementation, test setup or a requirement mismatch, correct it, and repeat the affected checks. A retry that happens to pass is not by itself an explanation. A review finding returns to the same loop and remains open until its evidence is checked.

## Before a public launch

Manual release workflows and an [infrastructure deployment specification](../specs/027-aws-production-deployment/spec.md) are present. They describe a release process; they do not demonstrate that hosting prerequisites, approvals or operational checks are in place. The release workflows also treat green CI as a prerequisite rather than rerunning every check themselves.

A future launch needs current evidence for the chosen application versions: the relevant automated checks, the critical user journeys, pending requirement corrections and the operating checks in the deployment plan. Features that remain switched off need their own enablement evidence. Nothing on this page reports a fresh release, uptime measurement or customer-facing availability.

## Evidence behind this page

The private files inspected were the backend Maven build and CI workflow, frontend package files, test configuration and CI workflow, and the browser-test package files, configuration and combined workflow. Tracked test filenames supplied the counts. No private source, executable test code, workflow configuration, raw logs or service identifiers are included in this public copy.
