# From a request to a checked change

I use AI to help with writing, implementation, testing and review. The purpose of this process is to keep the important decisions visible and to check the work before treating it as finished.

These are responsibilities in an AI-assisted workflow, not a claim that I employ a large team. The project records describe this process; individual historical tasks still need their own evidence. InTuition is not yet live, and life commitments have delayed launch.

**Need → specification → independent specification review → issue → implementation → result review → tests → updated records → release decision**

## 1. Start with the need

**Who checks it:** business and product perspectives.

Describe who is affected and what they need to do. Record the expected value and any business decisions still needed. A feature request is not yet a complete rule.

**Before moving on:** the change has a clear purpose. If the payer, permission or desired outcome is unclear, resolve that question first.

## 2. Write down the rules

**Who prepares it:** product and technical perspectives.

Write the requirement, explain the design and give examples of what should pass or fail. Include permissions, limits, failure cases and what should remain unchanged.

**Before moving on:** another reader can tell whether a proposed change meets the need. If two documents disagree, identify which decision controls and update the affected documents.

## 3. Review the specification before building

**Who checks it:** reviewers other than the author, chosen for the affected business, product, technical and testing questions.

Challenge the plan before implementation starts. Look for missing cases, contradictions, unclear ownership and decisions that have been assumed rather than approved. A specification author should not approve their own proposal.

**Before moving on:** reviewers other than the author must approve the corrected specification and its pass/fail examples before implementation starts. Resolve blocking findings first. An unresolved business decision blocks the work that depends on it; the implementation must not choose that policy by accident.

## 4. Make the issue small enough to check

**Who checks it:** the technical lead and developer perspectives.

Link the issue to its requirements. Name the affected parts of the system, dependencies and the checks required before closing the task.

**Before moving on:** the task has a clear finish. If the scope grows, update the issue and specification instead of silently adding unrelated changes.

## 5. Use AI to implement the change

**Who does the work:** the developer role, with business decisions retained by the founder.

Give the AI the approved specification, relevant rules, existing structure and limits. Inspect the changes it makes. Add or update checks for the behaviour being changed.

**Before moving on:** there is a reviewable change, not just a completion message. If implementation reveals an unresolved product choice, return to the specification.

## 6. Review the result

**Who checks it:** reviewers separate from the author, covering the affected responsibilities.

First check whether the change satisfies the specification. Then examine its quality, failure cases and effects on related journeys. Reviewers should explain what is wrong and which observation supports the finding.

**Before moving on:** blocking findings have been fixed and the corrections checked. A reply to a comment is not itself proof that the problem is fixed.

## 7. Run the relevant tests

**Who checks it:** developer and QA, the testing perspective.

Check the new behaviour and existing journeys it might break. A screen test and a backend test answer different questions; use both where a rule crosses the two.

**Before moving on:** record the actual results and any checks that could not run. A failure goes back to the implementation, test setup or requirement. A retry that happens to pass does not explain the original failure.

## 8. Update the records together

**Who checks it:** product and technical lead perspectives.

Keep the requirement, design, issue, test expectations and decision record aligned. Date older evidence so it cannot be mistaken for verification of a later correction.

**Before moving on:** a reader can follow why the change happened and what was checked. Inconsistent records need a correction or an explicit follow-up.

## 9. Decide whether it is ready to release

**Who decides:** the founder, with technical and operating checks.

Passing tests are one input. Delivery, recovery, permissions, commercial exceptions and the ability to run the service also need attention. A written deployment plan is not a successful recovery exercise.

**Before moving on:** there is current evidence for the affected requirements and the chosen release. InTuition has not reached a public launch; this repository does not claim that its release checks are complete.

## What the loop looks like in practice

A parent request form once submitted while the user was moving to the review step. The recorded fix separated Continue from Submit and added a browser check that entering Review did not create a request. A later requirement added keyboard and input-method cases. A fresh source review found missing guards for those cases, so they need implementation changes and their own verification. The [alignment review](current-file-alignment.md) records that distinction.

That example matters because an existing test does not automatically cover a newly clarified requirement. [Read the case study and its limits](how-i-work.md#example-a-review-step-that-submitted-too-early).

For this public repository, a sixth reviewer takes the skeptical reader’s view: is the claim supported, does the wording sound natural, and does the document admit what has not been checked? [Read the publication review record](review-record.md).

[Back to the README](../README.md) · [Testing and release](testing-and-release.md) · [Current decisions](../specs/CURRENT-REQUIREMENTS.md)
