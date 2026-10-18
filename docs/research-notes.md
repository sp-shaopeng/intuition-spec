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

The final publication is Markdown only. It is read directly on GitHub and includes no website scripts, application source or deployment configuration.
