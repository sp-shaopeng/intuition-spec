# How I work on InTuition

I am building InTuition with AI assistance. I use it to help turn requirements into designs, implementation tasks, tests and review findings. I remain responsible for the decisions and for checking what has actually been built.

InTuition is **not live**. Life commitments have delayed launch. The work shown here is a development record, including corrections and unfinished checks, rather than a claim that the service is ready for customers.

## From a problem to a checked change

The project's written workflow connects a user problem to a requirement, a task, a review and evidence. Its purpose is to make it possible to answer a simple question: what observation shows that this change solved the intended problem?

1. **Describe the problem.** An issue records the expected behaviour, what went wrong and how to reproduce it. A feature starts with the outcome someone needs, including what is outside its scope.
2. **Write the specification.** Requirements receive identifiers so implementation tasks and evidence can point back to them. The specification records the design, acceptance conditions, open questions and version history. Material changes require an update to that history.
3. **Review before building.** Relevant perspectives check the proposal: product, interface, server, data, security and testing. The author does not approve their own specification. Work with dependencies is ordered; independent work can proceed in parallel.
4. **Implement and verify.** For a bug, reproduce the failure before changing it. Add a regression check where the failure can actually be observed, make the change, then run the appropriate checks. A regression check is a test intended to catch the same problem if it returns.
5. **Review the result twice.** First check whether it satisfies the specification. Then examine implementation quality, risks and effects on related flows. A coordinator role checks that the evidence supports the claims before closing the work.
6. **Loop back when something fails.** A failed test, incorrect assumption or valid review finding returns the item to investigation or implementation. After the fix, rerun the affected checks and review the result again. Unresolved conditions stay recorded instead of disappearing into a completion message.

The private agent instructions call their coordinator the “Boss” and use specialist AI roles for these perspectives. Those names describe responsibilities in an assisted workflow; they do not represent an independent human engineering team. The public [development workflow](../specs/domain/dev-workflow.md) contains more of the project's history and rules. Where older instructions conflict with the [current requirements](../specs/CURRENT-REQUIREMENTS.md), the current requirements govern the product.

## Example: a review step that submitted too early

Issue #206 described a tuition-request form that reached its final review step and then submitted before the user deliberately chose to submit. [Specification 008](../specs/008-tutee-post-flow-bugfixes/spec.md) split this from a second problem: budget validation could become stale when the related field changed.

The recorded investigation found a browser interaction problem. Advancing the form reused a button in a way that changed its purpose during the same click. The implementation now gives Continue and Submit distinct identities and checks that submission happens from the final step. Its browser regression test checks that arriving at Review creates no request and that deliberate submission creates one.

This was a useful boundary between test types: the simulated browser used for component tests did not reproduce the original click behaviour. A real-browser check was needed. The [requirements](../specs/008-tutee-post-flow-bugfixes/requirements.md) explain the expected behaviour, and the [task record](../specs/008-tutee-post-flow-bugfixes/tasks.md) connects it to verification work.

There is also an explicit limit to the case study. A later correction adds more precise keyboard and input-method requirements. The new file review found that the form still advances on Enter without the required control and composition checks, and the final step still permits implicit submission. The original button fix is present, but it does not satisfy every later acceptance condition. [See the remaining work](current-file-alignment.md).

## Example: review after tests found more work

The [scheduler work](../specs/007-generic-scheduler-db-scheduler/spec.md) moved scheduled tasks into a database-backed system. Its historical record describes a startup failure found when the parts were run together, followed by review findings involving cancellation during execution and missing failure measurements.

The changes went back through correction and review. That history is more informative than a test-count badge: it shows why a passing isolated test suite is only one kind of evidence. The same specification retains a delivery safeguard that must be addressed before enabling WhatsApp reminders. I do not treat the historical delivery label as proof that this remaining condition is closed.

## How AI output is checked

AI can propose useful changes and also make incorrect claims. A reviewer is expected to open the relevant artifact: the changed implementation, a captured result, a screenshot or a test report. A statement that checks passed is not interchangeable with the report. The workflow also asks for negative cases, such as an unauthorised user being denied, and checks on the stored outcome rather than only the response status.

This is consistent with GitHub's guidance to review and validate AI-generated work. That guidance is used here as an external reference, not as evidence that InTuition uses every GitHub agent feature. [GitHub's responsible-use guidance](https://docs.github.com/en/copilot/responsible-use/agents).

The public collection contains sanitised requirements and decision records. Raw captures, private source, account data and operational details are excluded. [Testing and release](testing-and-release.md) explains what was inspected for this publication and which results remain unverified.

## Evidence behind this page

This account was checked against the private project instructions, shared workflow and evidence rules, the linked specifications, the current request-form implementation and its browser regression test. It describes the project's documented method and concrete examples; it does not assert that every historical task followed every step perfectly.
