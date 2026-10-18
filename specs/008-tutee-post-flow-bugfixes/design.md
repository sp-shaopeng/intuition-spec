# Design — request-form bug fixes (008)

This design explains the corrected behavior in [the requirements](requirements.md). The original browser fault was reported fixed in June 2026; September keyboard clarifications still need implementation verification. The [specification](spec.md) keeps the dates separate.

## REQ-001 — Keep navigation distinct from submission

The confirmed browser reproduction showed Continue and Submit sharing one rendered button. While the Continue click was still being processed, advancing the step changed that element into a submit control. The same click could then create a request.

Give those controls separate stable identities so the clicked element cannot become Submit mid-event. Reject any submission before the final step. On Review, also require deliberate activation of the final submit control: reaching that step is not enough.

The ordinary single-line input handler must prevent Enter from implicitly submitting the form. On earlier steps it validates and advances when valid; on Review it stays in the form. This handler must exclude multiline fields, text composition, controls that own their keyboard interaction, checkboxes, radios, links and buttons. Enter and Space on the focused final button must keep their native activation behavior.

Do not rely only on the browser's reported submitter. An implicit Enter in a field can be attributed to the default button, so the input-key guard and deliberate-final-control check serve different purposes.

## REQ-002 — Revalidate the budget pair

Retain the shared rule that maximum must be at least minimum. After either budget value changes, store the new value and validate both values together. That refreshes the dependent error immediately, whether the edit makes the pair valid or invalid.

Preset budget choices already request validation for both values and must continue to do so. Other form fields should retain their existing validation timing.

The alternatives considered were changing validation timing for the entire form or rewriting the shared validation rule. Neither alone addresses the stale partner-field error as directly, and changing every field would unnecessarily widen the interaction change.

## Failure handling

- Empty, nonnumeric and out-of-range values retain the individual field errors. The cross-field fix does not remove those checks.
- The entire request must be valid before submission, even when the final button was deliberately activated.
- While creation is pending, the interface must prevent another user activation from sending a duplicate. Server-side duplicate handling is a separate concern.
- A failed request must remain a failure in the interface rather than show the success panel.

## Verification

Use a real browser to reproduce Continue-to-Review and assert that no creation request is sent. Then activate the final button deliberately and assert one request. Cover pointer, focused-button Enter and focused-button Space; ordinary-input Enter before and on Review; multiline input; text composition; and picker interaction.

For budget, start with maximum below minimum and lower minimum until valid; the error must clear. Start with a valid pair and raise minimum above maximum; the error must appear. Repeat equivalent edits to maximum and check that invalid pairs cannot advance or submit.

Component tests can check the guards and dependency validation. They did not reproduce the original browser click transformation and therefore do not replace that browser test. The changed steps must also be usable at 390 pixels without page-level horizontal overflow.

## Rollout and evidence

The original change was limited to the interface and required no database migration. Verify the corrected behavior on the intended build before release. Keep the June reported results separate from the September requirements; no current passing result is established by this public edit.

The original private reproduction and test reports are omitted. This design describes what the checks must show, not an independently reproduced result.
