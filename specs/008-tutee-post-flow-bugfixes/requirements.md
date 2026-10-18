# Requirements — request-form bug fixes (008)

The June 2026 report concerned unintended submission on reaching Review and budget errors that did not update. The original fix was reported in development; the September accessible-submission clarification still needs verification. See the [dated record](spec.md). The requirements below state the corrected behavior, not a fresh test result.

## Scope

The incident involved the parent request wizard, with subject/level, schedule, venue/budget and review steps. The fix does not change the information sent to the server, the success display, or the database. Subject lists, student count, schedule presentation and rate-guide links belong to [specification 009](../009-tutee-post-taxonomy-and-ux/spec.md).

## REQ-001 — Review requires deliberate submission

**Priority:** Must. The person creating a request must be able to inspect it before creating a post.

| Setup and action | Required result |
|---|---|
| Complete the earlier steps and click Continue into Review | Review stays visible; no request is created |
| On Review, deliberately activate Submit Request with a pointer | Validate the form, create exactly one request when valid, then show success |
| Focus Submit Request and press Enter or Space | The same valid submission behavior is available without a pointer |
| Press Enter in an ordinary single-line field on an earlier step | Validate that step and advance only when valid; never submit the request |
| Press Enter in an ordinary field on Review | Remain on Review without implicitly submitting |
| Invoke submission while an earlier step is visible | Reject the submission without creating a request |
| Use a multiline field, text composition, a picker, a list, a link or another interactive control | Preserve the control's own keyboard behavior; do not turn it into wizard navigation |
| Try to submit an invalid form or activate again while submission is pending | Do not send an additional request through that invalid or pending action |

The confirmed original cause was the same button changing from Continue into Submit during the click that advanced the step. Separate control identities and a final-step safeguard are required, but they do not replace the deliberate-activation and keyboard rules above.

**Evidence required:** An actual browser test of Continue-to-Review, followed by deliberate pointer and focused-button keyboard submission. Also test field Enter on earlier and final steps, multiline input, text composition and picker selection. Component tests should cover the submission safeguards, but cannot replace the browser reproduction of the original click behavior.

## REQ-002 — Budget validation follows both fields

**Priority:** Must. The maximum budget must be greater than or equal to the minimum, and the visible error must reflect the current values.

| Setup and action | Required result |
|---|---|
| Enter a maximum below the minimum | Show the budget-order error and block progression/submission |
| With that error visible, lower the minimum to the maximum or below | Clear the order error as soon as the pair is valid |
| Start with a valid pair, then raise the minimum above the maximum | Show the order error immediately |
| Change maximum to make the pair valid or invalid | Recheck the pair and update the error in the same way |
| Choose a preset budget band | Validate both values consistently with manual editing |
| Leave a value empty, nonnumeric or below its allowed minimum | Preserve the relevant field-validation error; do not submit |

Recheck both budget fields whenever either changes. Keep unrelated fields' existing validation timing. Do not make the whole form noisier merely to repair this two-field dependency.

**Evidence required:** Tests for both directions of invalid-to-valid and valid-to-invalid editing, preset bands and blocked progression/submission while invalid. A browser check should show the error clearing and appearing in the visible form.

## Verification boundary

The historical frontend gate required 97.5% line coverage and 88% branch coverage over its configured coverage set, plus formatting/lint, build and a 390-pixel usability check. This page does not claim those checks were rerun, or that coverage proves the interaction requirements. Review the actual browser and behavior evidence separately.

Private screenshots, traces and test logs are not included. A completed task requires reviewable evidence in the working project; this public document is not that evidence.
