# Running the project with AI, specifications and a maintained test lab

Draft proposal, revised 18 October 2026. This plan is about how the founder runs the project: choosing work, assigning AI tasks, reviewing specifications, maintaining tests and deciding when a change is finished. The [user-flow guide](user-flows.md) separately explains the product journeys.

The founder owns priorities, business decisions and release approval. AI helps investigate, write specifications, implement bounded changes, test and review. The role names below describe responsibilities, not employed staff. This proposal does not claim new automation is installed or waive existing review requirements.

## What exists today

Project instructions already require specifications before code, independent review, test-first development, a durable task board and evidence before completion. They also require lessons from failures. These are documented operating rules; they do not prove that every historical task followed them.

The inspected end-to-end lab has browser and API tests, shared setup helpers, a seed-manifest check, failure traces, one CI retry and a combined application workflow. That workflow records application revisions and checks backend build identity. However, it selects moving application branches; some tests need optional database access; the declared browser matrix exceeds the browsers installed by CI. The main matching scenario performs its business steps through API calls. [Testing and release](testing-and-release.md) describes these limits and a separate limited backend baseline. This plan's source audit did not execute tests.

The [alignment review](current-file-alignment.md) also found tests preserving older behaviour. The improvement is to make the existing process dependable, with explicit owners and evidence at each handoff.

## 1. Choose a small amount of work

Proposed rule: put each request, defect or review finding on one private board. Record the affected person, expected outcome, severity, requirement link, dependencies and next action. The founder chooses the order: protect money, access and saved data first; then unblock agreed user journeys; then improve convenience.

Keep the application repair backlog separate from process maintenance. Start with at most two independent implementation tasks, plus review or lab maintenance. This is a proposed starting limit, not a claim about current staffing. Finish or unblock work before starting more. Split a large request into changes that can each be reviewed and verified; Google's guidance explains why small, focused changes are easier to review. [Small changes](https://google.github.io/eng-practices/review/developer/small-cls.html).

## 2. Make the specification ready before asking AI to build

Use [current requirements](../specs/CURRENT-REQUIREMENTS.md) to resolve authority, then link the detailed governing package. Keep requirement IDs stable, including their package prefix. One current rule controls; older descriptions retain their date and a link to the replacement.

A task is ready when it has an owner, scope, written rules, a design, a task list, observable pass/fail examples, dependencies and the evidence needed to finish. Include roles, failure cases and effects on related journeys. An independent specification reviewer challenges those examples before implementation. The author cannot approve their own specification.

Unresolved business decisions stay explicit. AI may research options and recommend one, but existing code does not supply approval. A changed decision updates the specification, examples, affected tests and decision history together. Reopen dependent work and invalidate affected approvals. Preserve the existing local review and final-audit rules; this plan does not replace them with a lighter review.

## 3. Give each AI task clear boundaries

The founder or coordinating AI gives the implementing AI a short handoff: issue and requirement IDs, approved specification version, allowed files, interface contracts, starting versions, acceptance examples, required checks and known limits. Name one accountable owner and a reviewer who did not author the change.

Assign one writer to each shared area. Parallel tasks need separate file ownership and agreed contracts; sequence dependent changes and shared builds. If an AI discovers a contract change or another owner's file needs editing, it reports the dependency and updates the plan before expanding its scope. Follow the authorized repository/branch rules and preserve unrelated edits.

Ask for reviewable artifacts: a diff, a test result, a reproduced failure or a specification change. "Finished" in a message is insufficient. Keep private code, credentials, patches and raw test traces in private repositories or approved private evidence storage.

## 4. Track progress and blockers through artifacts

Use the existing board states consistently:

| State | Evidence or next action |
|---|---|
| TODO | Scope and owner recorded; readiness gaps identified |
| IN-PROGRESS | Owner has a current artifact and a next checkpoint |
| BLOCKED | Specific dependency, unblock owner and next action recorded |
| IN-REVIEW | Reviewable change and required results available |
| DONE | Acceptance and required review checked; release obligations remain explicit |

At each checkpoint, inspect the actual diff, report or running job before updating status. Investigate a stalled process rather than repeatedly waiting or restarting it. A blocker stops its dependent task, not unrelated work. At session end, leave the current revisions, unfinished steps, findings and next actions on the board so another AI can continue without reconstructing the conversation.

## 5. Develop against meaningful regression tests

For a defect, reproduce the failure and identify its cause. Add a focused test that fails for that cause, implement the fix, and show the test passes. For new behaviour, derive the checks from the approved acceptance examples. Test the user's outcome and important rejection paths, not just a successful response or the implementation's internal shape.

A reviewer should check that the test would detect the original defect. Update the relevant existing journey when a feature changes; creating another isolated test is insufficient if the maintained journey still assumes the old rule. Playwright recommends testing visible behaviour with stable, user-facing locators. [Playwright best practices](https://playwright.dev/docs/best-practices).

## 6. Maintain the end-to-end lab as part of the product

Give lab maintenance an owner and reserve work for it alongside features. Each feature task includes its regression impact; the testing owner reviews shared helpers, test-data changes and coverage gaps. Fixtures are the accounts and records a test needs; a seed is a known starting set of that data. A smoke check is a short check of essential behaviour. A gate is a check that must pass before the work moves on.

### Own the fixtures and reset safely

Keep reusable setup in shared helpers. Give each test its own synthetic users and records; own their creation, cleanup and lifecycle. Check the expected seed version and data before running dependent tests. Changes to shared seeds need a reviewed list of affected tests. The existing frozen backend seed is separately staged and default-off; do not assume all suites use it.

Use isolated test environments where available. On a shared development stack, reset only the identified test records or limiter keys that the run owns, after confirming the target and coordinating with other users. Never flush a shared cache or reset unrelated data to make a test pass. Session disposal alone is not database cleanup. Playwright fixtures can keep setup and teardown together. [Fixture guidance](https://playwright.dev/docs/test-fixtures).

### Separate browser, API and delivery evidence

Mark each scenario by its test layer. API setup is useful, but a website claim needs the relevant action through the browser and its visible result. A direct assignment request does not prove the staff assignment screen works. A mocked provider does not prove external delivery. Include mobile, keyboard and timezone cases where the requirement depends on them.

### Reproduce the whole application version

Record the specification, frontend, backend and test revisions, migration state, seed version, browser/toolchain versions and relevant feature settings. Include an identifiable private snapshot of uncommitted changes. Rebuild changed components and verify the running versions before the lab starts. Replace moving-branch assumptions with explicitly selected combined revisions as a proposed improvement.

Confirm that the website and API point to the same intended test environment. The mutation-heavy development suite is not a production smoke check; define a separate, non-mutating production check. Check stack health, browser installation and required fixture access first. Missing prerequisites should produce a clear incomplete run, not a successful empty result. Use the locked dependencies consistently; investigate installation failures rather than silently accepting a different dependency set.

### Treat failures, skips and flaky tests honestly

Separate product defects, requirement mismatches and lab/environment failures. Preserve the first failure and diagnose it. Follow the approved retry rules; a pass on retry remains a flake to investigate, not a clean first-pass result. Playwright reports these outcomes separately. [Retry guidance](https://playwright.dev/docs/test-retries).

Record failed, skipped, aborted and retried cases with reasons. A required journey skipped for missing database access remains unverified. Any temporary quarantine needs an owner, issue, expiry, missing-coverage statement and the required approval; it cannot silently remove a release requirement. The lab README describes quarantine, but inspected configuration does not establish that exclusion as enforced. Reconcile policy and configuration before relying on it.

### Review changes to tests and keep the gates healthy

Review deleted tests, weakened assertions, new skips and refreshed contract snapshots against the requirement they protect. Record a replacement check or an approved retirement of the rule. Do not lower thresholds or add exclusions merely to restore green.

Retain backend coverage of 97.5% lines and branches, and frontend coverage of 97.5% lines and 88% branches, alongside required formatting, lint, build and integration checks. The lab's route/API inventory is useful for finding missing tests, but a referenced endpoint is not a passed journey. Keep static inventory, executed results and requirement acceptance separate.

The inspected combined workflow does not invoke the available lab lint, inventory, contract-comparison or coverage scripts. Reconcile those omissions with the governing requirements before calling them enforced gates. Make the small smoke set dependable, then run the required broader suite and browser matrix. Review toolchain upgrades separately and confirm all declared browsers actually run. Required CI checks and branch protection are proposed enforcement work until their settings and a deliberately blocked failure have been verified. GitHub supports those mechanisms; its documentation does not establish this project's settings. [Protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches).

## 7. Review, release and reopen when evidence changes

The independent reviewer starts from the rule and actual change, opens the evidence, and challenges a likely failure path. Review specification compliance before implementation quality. Google also asks reviewers whether tests are useful and would fail when behaviour breaks. [Review guidance](https://google.github.io/eng-practices/review/reviewer/looking-for.html).

Before publishing documentation, check local links, preserved business meaning, current versus historical claims and disclosure risks. The public edition remains Markdown-only; application code, private contacts, company identifiers and raw evidence stay out.

Keep the chain traceable: requirement → issue → approved specification → implementation commits → acceptance test → evidence → review decision. Record the states separately: implemented means the files contain the change; verified names passed acceptance evidence; task-done adds the required review and records; released names deployment and post-release evidence. Passing tests in development does not establish production readiness.

The founder approves deployment of the reviewed version after the required tests, migration preparation and recovery checks. Mark it released only after deployment and the required checks of the running version pass. A change to code, fixtures or a material requirement reopens affected verification and review. Reproduce a regression, link it to the missed rule/test and return it to the board. Keep remaining delivery obligations visible.

## 8. Learn and introduce the changes gradually

After a failure or review correction, record the cause, the missing check and a reusable prevention rule. Update the owning guidance or lab helper where appropriate. Avoid lessons that merely say "be careful".

| Proposed rollout | Owner | Acceptance before moving on |
|---|---|---|
| First two small changes | Founder/coordinating AI | Use the ready-task handoff, board and evidence chain; another reader can reconstruct both results |
| Next lab-maintenance pass | Lab owner/testing reviewer | Assign fixture/seed owners, reconcile browser and skip policies, reproduce one combined version and preserve failure evidence |
| Following release cycle | Founder/release reviewer | Verify required checks actually block failure; review measures and adjust the work limit |

Collect baselines during that first cycle: time waiting for review/blockers, required journeys skipped, first-pass versus retry results, defects reopened after completion, and changes with complete evidence links. Report the run count and period; do not invent current rates or targets. Use the findings to choose the next improvement, while retaining the governing requirements and review obligations.
