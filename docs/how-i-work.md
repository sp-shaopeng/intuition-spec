# How I work on InTuition

I am building InTuition with AI assistance. I use it to help turn requirements into designs, implementation tasks, tests and review findings. I remain responsible for the decisions and for checking what has actually been built.

This page describes **how I manage the project**. The [user-flow guide](user-flows.md) separately explains what parents, tutors and staff do in the product. The [project workflow improvement plan](workflow-improvement-plan.md) sets out what I want to improve in planning, AI task management, reviews and the regression test lab.

InTuition is **not live**. Life commitments have delayed launch. The work shown here is a development record, including corrections and unfinished checks, rather than a claim that the service is ready for customers.

## From a problem to a checked change

The project's written workflow connects a user problem to a requirement, a task, a review and evidence. Its purpose is to make it possible to answer a simple question: what observation shows that this change solved the intended problem?

The [step-by-step walkthrough](workflow-walkthrough.md) covers specification approval, bounded AI assignments, tests, independent review and release decisions. This page focuses on what those rules mean in practice and where the evidence is incomplete. A regression test is a check intended to catch a problem if it returns.

The private agent instructions call their coordinator the “Boss” and use specialist AI roles for these perspectives. Those names describe responsibilities in an assisted workflow; they do not represent an independent human engineering team. The public [development workflow](../specs/domain/dev-workflow.md) contains more of the project's history and rules. Where older instructions conflict with the [current requirements](../specs/CURRENT-REQUIREMENTS.md), the current requirements govern the product.

## What spec-driven development means here

I write down the rules before asking AI to build. That is what I mean by spec-driven development. A feature package contains a specification, requirements, design and tasks. It needs examples of a correct result and cases that must be rejected, alongside unresolved decisions and its version history.

The AI's task should point to that approved version and say which part it owns, which files it may change, what it depends on and how the result will be checked. The project instructions call for separate owners when work can proceed independently and one writer where changes share files. Those are working rules, not proof that every historical task followed them.

If a rule changes, the design, task and tests need review together. Keeping an old test green by changing the requirement would hide the disagreement. The guest-search finding is a concrete example: tests currently expect hidden controls even though the current rule permits guests to filter results. [Read the finding](current-file-alignment.md#g01--guests-cannot-use-the-required-search-controls).

AI can investigate a question and propose options. An unanswered question about money, permissions or recovery stays open until the responsible person decides. A confident implementation does not settle that decision.

## The regression lab needs maintenance too

The project has a separate Playwright test repository and backend tests that exercise a running service. Some Playwright checks use the website; others call the server directly. Both are useful, but a successful server request does not show that the corresponding button works.

The lab needs attention whenever the specification, application, fixtures or environment changes. A new acceptance case needs the right test. An intentional rule change needs a reviewed test update. A failure needs investigation before its assertion is removed or weakened. Shared test accounts and skipped prerequisites also need attention, because they can make a passing result misleading.

The inspected combined workflow records application versions and attempts to check the running backend's identity. Source review found incompatible build paths and a version request to a protected endpoint, so its current setup cannot be relied on to reach the tests. It also selects moving application development branches; this does not establish a required test of the exact changes being approved. The [testing guide](testing-and-release.md) explains the current boundaries. The [improvement plan](workflow-improvement-plan.md) covers reproducible versions, owned test data, unreliable tests, lab health and the evidence needed to close a task.

## Example: a review step that submitted too early

Issue #206 described a tuition-request form that reached its final review step and then submitted before the user deliberately chose to submit. [Specification 008](../specs/008-tutee-post-flow-bugfixes/spec.md) split this from a second problem: budget validation could become stale when the related field changed.

The recorded investigation found a browser interaction problem. Advancing the form reused a button in a way that changed its purpose during the same click. The implementation now gives Continue and Submit distinct identities and checks that submission happens from the final step. Its browser regression test checks that arriving at Review creates no request and that deliberate submission creates one.

This was a useful boundary between test types: the simulated browser used for component tests did not reproduce the original click behaviour. A real-browser check was needed. The [requirements](../specs/008-tutee-post-flow-bugfixes/requirements.md) explain the expected behaviour, and the [task record](../specs/008-tutee-post-flow-bugfixes/tasks.md) connects it to verification work.

There is also an explicit limit to the case study. A later correction adds more precise keyboard and input-method requirements. The new file review found that the form still advances on Enter without the required control and composition checks, and the final step still permits implicit submission. The original button fix is present, but it does not satisfy every later acceptance condition. [See the remaining work](current-file-alignment.md).

| Date and record | What it supports | What it does not establish |
|---|---|---|
| 21 June 2026, specification 008's original development record | AI-assisted investigation reported the button-identity cause, a change and browser regression coverage. The founder retains decision authority; the named specialist sign-offs are AI roles. | This publication did not independently reconstruct the original human approval or reopen a complete original run bundle. Its reported result remains historical. |
| 5 September 2026, private specification 008 version 1.3 | The corrected requirement covers deliberate submission, keyboard input and text composition. | An old sign-off or passing test does not approve or verify the added cases. |
| 18 October 2026, source snapshot SRC-2026-10-18-A | Reviewers inspected the identified frontend and browser-test revisions. The button separation and a test for entering Review without submitting are present. The broader keyboard guards are missing. | Reading the test does not execute it. No fresh browser pass is claimed. |
| Current disposition | Keep the narrow historical fix and reopen the missing acceptance cases under G07. The private repair plan remains a draft. | No claim of complete form compliance or launch readiness. |

The private evidence index maps the source label to exact repository revisions and records unchanged tracked source. Source, raw reports and account details stay private. This short trail is deliberately incomplete where original evidence was not independently rechecked; AI did not fill those gaps with an invented result.

## Example: review after tests found more work

The [scheduler work](../specs/007-generic-scheduler-db-scheduler/spec.md) moved scheduled tasks into a database-backed system. Its historical record describes a startup failure found when the parts were run together, followed by review findings involving cancellation during execution and missing failure measurements.

The changes went back through correction and review. That history is more informative than a test-count badge: it shows why a passing isolated test suite is only one kind of evidence. The same specification retains a delivery safeguard that must be addressed before enabling WhatsApp reminders. I do not treat the historical delivery label as proof that this remaining condition is closed.

## How AI output is checked

AI can propose useful changes and also make incorrect claims. A reviewer is expected to open the relevant artifact: the changed implementation, a captured result, a screenshot or a test report. A statement that checks passed is not interchangeable with the report. The workflow also asks for negative cases, such as an unauthorised user being denied, and checks on the stored outcome rather than only the response status.

This is consistent with GitHub's guidance to review and validate AI-generated work. That guidance is used here as an external reference, not as evidence that InTuition uses every GitHub agent feature. [GitHub's responsible-use guidance](https://docs.github.com/en/copilot/responsible-use/agents).

The public collection contains sanitised requirements and decision records. Raw captures, private source, account data and operational details are excluded. [Testing and release](testing-and-release.md) explains what was inspected for this publication and which results remain unverified.

## Evidence behind this page

This account was checked against the private project instructions, shared workflow and evidence rules, the linked specifications, the current request-form implementation and its browser regression test. It describes the project's documented method and concrete examples; it does not assert that every historical task followed every step perfectly.
