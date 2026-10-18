# Research used for this publication

The project’s own specifications and inspected implementation files support claims about InTuition. External guidance helped with writing, presentation, testing explanations and safe publication. It does not verify the business model or certify the application.

Reviewed for this publication on 18 October 2026:

| Source | How it helped |
|---|---|
| [Google: voice and tone](https://developers.google.com/style/tone) | Use direct, conversational explanations; avoid unnecessary jargon and sales language. |
| [GOV.UK: layout](https://design-system.service.gov.uk/styles/layout/) | Keep explanations easy to scan. The final publication uses GitHub Markdown rather than a separate website. |
| [GitHub: removing sensitive data](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository) | Start with a clean public history and inspect the exported content. Removing a secret from a copy does not revoke the original secret. |
| [Google: what to look for in a review](https://google.github.io/eng-practices/review/reviewer/looking-for.html) | Compare documentation with behaviour and inspect what tests actually assert. The independent alignment review used local files as evidence for InTuition. |
| [Playwright: API testing](https://playwright.dev/docs/api-testing) | Distinguish requests sent directly to the server from actions performed through a browser page. A test file count does not establish completed website journeys. |
| [Google: small changes](https://google.github.io/eng-practices/review/developer/small-cls.html) | Keep each fix focused enough for a reviewer to understand its purpose and test it. |
| [Playwright: best practices](https://playwright.dev/docs/best-practices) | Check visible user actions with isolated tests and controlled data. |
| [GitHub: protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) | Identify possible review and status-check controls; the plan does not claim they are already configured. |
| [Diátaxis](https://diataxis.fr/) | Separate readable explanations and user journeys from detailed reference documents, and link them together. |
| [Playwright: fixtures](https://playwright.dev/docs/test-fixtures) and [authentication](https://playwright.dev/docs/auth) | Explain owned setup/cleanup and separate accounts when tests change shared server data. |
| [Playwright: retries](https://playwright.dev/docs/test-retries) and [continuous integration](https://playwright.dev/docs/ci) | Keep first failures and retry outcomes visible, and align browser installation with the checks selected. |
| [GitHub: secure use](https://docs.github.com/en/actions/reference/security/secure-use) | Inform proposed permission, dependency and evidence-handling checks for the private regression workflow. |
| [GOV.UK: alpha](https://www.gov.uk/service-manual/agile-delivery/how-the-alpha-phase-works) and [beta](https://www.gov.uk/service-manual/agile-delivery/how-the-beta-phase-works) | Separate early business learning from admitting trial users, and account for the capacity to support them. These are planning references, not proof of demand. |
| [DORA: small batches](https://dora.dev/capabilities/working-in-small-batches/) | Keep the first improvement bounded and choose later work from observed results and effort. |
| [GitHub: action exit codes](https://docs.github.com/en/actions/how-tos/create-and-publish-actions/set-exit-codes) and [deployment environments](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments) | Distinguish a warning from a failed check and a named environment from configured approval rules. Read-only project settings supplied the project-specific findings. |
| [Docker: build paths](https://docs.docker.com/reference/compose-file/build/) | Check whether the lab's expected source directories match its checkout layout. The reported mismatch is a source finding, not a newly executed failure. |
| [OpenAPI specification](https://spec.openapis.org/oas/v3.1.1.html) and [Pact guidance](https://docs.pact.io/) | Explain why a shallow interface comparison cannot establish compatibility for the application using that interface. No new tool adoption is claimed. |
| [AWS: recovery testing](https://docs.aws.amazon.com/wellarchitected/latest/framework/rel_planning_for_recovery_dr_tested.html) | Require observed recovery evidence before launch, while preserving the project's existing recovery targets. |
| [GitHub: responsible use of agents](https://docs.github.com/en/copilot/responsible-use/agents) | Explain why generated work still needs review and validation. It does not establish that this project uses GitHub's agent products. |
| [Anthropic: long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | Inform the explanation of bounded tasks, progress records and checking work across sessions. This is external guidance, not evidence that every proposed control operates in InTuition. |

The final publication is Markdown only. It is read directly on GitHub and includes no website scripts, application source or deployment configuration.

For the sequence-diagram revision, [GitHub's diagram documentation](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams) and [Mermaid's sequence reference](https://mermaid.js.org/syntax/sequenceDiagram.html) inform the Markdown format, participant lanes, alternatives and loops. [Claude Code's current best-practice guide](https://code.claude.com/docs/en/best-practices) supports giving AI concrete verification steps, exploring and planning before coding, and reviewing work separately. These sources inform the explanation; they do not establish that proposed project controls have been installed or that the application passed a new test.
