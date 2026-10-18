# Tasks — request-form bug fixes (008)

This is the implementation and verification work record for [requirements 001 and 002](requirements.md). The original June fix was reported complete in development. The September accessible-submission clarification has not been reverified for this public edition; the table does not turn that follow-up into a completed result.

Historical specialist labels refer to AI-assisted work, not independent human reviewers. Private evidence must be reviewed in the working project; it is not included here.

| Task | Requirement | Required outcome |
|---|---|---|
| Reproduce the Continue-to-Review click in an actual browser | REQ-001 | Record the original unintended request and the confirmed reused-button cause |
| Keep Continue and Submit as separate controls | REQ-001 | The click that opens Review cannot become a submit action |
| Guard submission by step and deliberate final-button activation | REQ-001 | Earlier steps and implicit field Enter create no request; valid pointer or focused-button Enter/Space creates one |
| Preserve ordinary keyboard behavior | REQ-001 | Multiline input, composition, pickers, lists, links and other controls retain their expected interaction |
| Revalidate both budget values on either change | REQ-002 | Errors appear and clear immediately as the pair becomes invalid or valid |
| Test preset and manual budget entry | REQ-002 | Both obey the same limits; invalid pairs block progression/submission |
| Add component checks for the safeguards | Both | Exercise invalid, valid, pending and non-final-step behavior |
| Run browser checks and narrow-screen checks | Both | Review stays visible until deliberate submit; keyboard cases pass; budget errors update; the touched flow fits 390 pixels |
| Review evidence against the intended build | Both | Separate dated June results from new September acceptance evidence; do not infer success from an old checked box |

The original gate also called for lint, build and the configured frontend coverage thresholds of 97.5% lines and 88% branches. Passing those checks is necessary within that workflow but does not establish the browser behavior on its own.

Scope remains limited to the request-form fixes. The separate taxonomy and interface improvements are tracked in [specification 009](../009-tutee-post-taxonomy-and-ux/spec.md). No application tests or deployment were performed during this documentation edit.
