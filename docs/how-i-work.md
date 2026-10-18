# How I direct AI on InTuition

I use AI to help deliver a Singapore tuition project, starting with the business need and following changes through requirements, implementation, tests and review. I set the intent, constraints and priorities, and retain responsibility for business decisions. AI performs investigation, drafting, engineering and technical checks within the assigned scope.

The service is not live. Personal commitments delayed launch, and the [current alignment review](current-file-alignment.md) identifies unfinished requirements. This page follows actual work and its evidence. The [walkthrough](workflow-walkthrough.md) explains the process in detail; [user flows](user-flows.md) explain the product.

## What I direct, and what AI does

My part is to explain the problem, choose what matters, set constraints and decide unresolved business questions. AI can propose technical approaches and make choices within an assignment with clear limits. It cannot turn an unanswered commercial or access-policy question into approval by implementing an assumption.

The written method connects a business need to a versioned specification, acceptance examples, an issue, implementation, regression tests, independent review and delivery evidence. A specification package contains the rules, design, tasks and decision history. Independent specification review must approve the version before implementation. An AI assignment identifies that version, owned files, dependencies and required checks. Shared files need one writer; independent work can be delegated separately.

I ask AI reviewers to compare claims with the actual specifications, implementation and results. The reviewer should inspect the evidence and challenge an important failure case. Independent review here means a separate AI execution, not a different human team or organisation. A role name alone does not demonstrate independence, and the author does not approve their own work.

These are operating rules, not proof of universal historical compliance. The [development workflow](../specs/domain/dev-workflow.md) retains conflicting older procedures under W01. The [improvement plan](workflow-improvement-plan.md) proposes a founder-approved reconciliation; it does not silently reduce existing review obligations or claim new automation is installed.

## Example: a review step that submitted too early

Issue #206 described a parent tuition-request form that submitted when the user meant only to reach its final Review step. [Specification 008](../specs/008-tutee-post-flow-bugfixes/spec.md) separated this from a budget-validation problem. Its [REQ-001 acceptance cases](../specs/008-tutee-post-flow-bugfixes/requirements.md) describe when submission is allowed; REQ-002 covers validation when either budget field changes.

The recorded AI-assisted investigation identified a browser interaction: the same button changed from Continue to Submit during the click that advanced the form. The implementation separates those controls and checks the final step. The browser regression test checks that entering Review creates no request and deliberately submitting creates one. The [task record](../specs/008-tutee-post-flow-bugfixes/tasks.md) connects these outcomes to verification work.

The simulated browser used for component tests did not reproduce the original interaction. That made an actual browser test necessary for this defect. Reading its source today shows what it checks; it does not rerun the test.

| Date and record | Evidence and limit |
|---|---|
| 21 June 2026, original specification 008 record | Reports the investigation, button-identity fix and browser regression coverage. Specialist sign-offs are AI roles. This publication did not reconstruct the original human approval or reopen a complete original run bundle, so the reported result remains historical. |
| 5 September 2026, specification 008 version 1.3 | Adds precise deliberate-submission, keyboard and text-composition requirements. Earlier approval and tests do not verify these new cases. |
| 18 October 2026, source snapshot SRC-2026-10-18-A | Reviewers found the separated controls and narrow browser test in the identified application/test revisions. They also found missing keyboard guards and implicit final-step submission. This was source inspection, not a fresh browser pass. |
| Current outcome | Preserve the narrow fix and reopen the missing cases under [G07](current-file-alignment.md#g07--the-request-form-keyboard-follow-up-is-unfinished). The private repair plan remains a draft. |

The private evidence index maps the snapshot label to exact revisions and unchanged tracked source. Private code and raw reports stay private. The records show what was fixed and what remains unfinished: one defect has a recorded fix; later acceptance cases still need implementation and execution evidence.

The same distinction matters elsewhere. Guest-search tests currently expect hidden controls even though the governing requirement allows guest filtering. Passing those tests would preserve the wrong behaviour. [G01](current-file-alignment.md#g01--guests-cannot-use-the-required-search-controls) records that disagreement rather than rewriting the requirement to fit the code.

## Example: a completed documentation delivery

This publication provides a separate, completed example of directing AI through a deliverable. It supports the application story; it is not a substitute for implementing the missing product behaviour.

I set the public boundary: explain the project without application code, secrets, private contacts or company identifiers. I rejected HTML in favour of Markdown. I clarified that "workflow" meant how I manage the project with AI, specifications and a maintained regression lab. I also requested business, product, technical-lead, developer, QA and AI-skeptic perspectives, including independent comparison with the actual files and reported progress.

AI inspected the private sources, drafted the public account, converted documentation, assigned reviews of specific areas and combined duplicate findings. Separate reviewers challenged the first drafts before seeing each other's first-pass verdicts, subject to the independence limits recorded in the [review record](review-record.md#fresh-six-role-review-of-the-project-workflow).

Their corrections changed the substance. An admin-panel uptime target had become a human staffing promise. The high-level billing flow had lost the required fourteen-day escalation. Known billing and expiry conflicts were missing from the controlling decision register. AI authors corrected these points, and reviewers rechecked them. Unresolved policy stayed unresolved; documentation edits did not approve business choices.

The [completed publication at a04514d, 18 October 2026](https://github.com/sp-shaopeng/intuition-spec/commit/a04514d1b224cb8241cd979ecb525bd5ba127ccd) records 362 Markdown documents, including 346 specifications and all 43 detailed sequences. Collection checks covered 7,115 local document/heading links and found no broken links, malformed table widths, non-Markdown files, rendered HTML or code blocks. Comparisons found no matches to 100 private credential candidates or six private company legal/contact values. These checks cover the files and values described here; they do not guarantee that every unknown secret format was detected.

I owned the intent and constraints. AI performed the file comparisons, scans and technical reviews; this account does not attribute every technical choice or artifact inspection to me. The completed outcome is a reviewed public documentation revision. It does not close application gaps, prove a launch or install the proposed project controls.

## Keeping the evidence useful after a change

The regression lab needs maintenance when specifications, application files, test data or environments change. I remain accountable for that recurring work; AI can own a specific maintenance task. Browser actions, direct API requests and external message delivery need distinct evidence. An API is the interface through which software sends requests to another part of the system.

The inspected combined workflow attempts version checks, but incompatible build paths and a protected identity endpoint prevent relying on it to reach testing. It also selects moving branches. The proposed first engineering task is to get the test lab to start and check the application version, followed by tests tied to the exact source and built version. Billing stays a blocked planning row while its decisions remain open.

The [testing guide](testing-and-release.md) records the separate limited backend baseline, the unfinished frontend run and other lab limits. The [improvement plan](workflow-improvement-plan.md) covers owned test data, skipped cases, retries, retained evidence, founder effort and complete-journey checks. These improvements remain proposals until implemented and demonstrated.

This portfolio shows deliverables, corrections and unfinished work. It does not establish measured AI productivity savings or readiness for customers. The evidence, including its limits, is what makes the project inspectable.
