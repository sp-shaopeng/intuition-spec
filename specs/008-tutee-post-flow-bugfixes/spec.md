> **Current-file check, 18 October 2026:** The original button fix is present. The corrected keyboard and text-composition contract still lacks required guards and complete regression coverage (G07). See the [alignment review](../../docs/current-file-alignment.md). The requirements remain unchanged.

# Request-form bug fixes — review step and budget validation

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).


**Specification:** 008 · **Original report and development fix:** 21 June 2026 · **Requirements clarification:** 5 September 2026

**Current status:** The original button bug and budget-validation bug were reported fixed in development in June. The September keyboard and accessible-submission requirements still need verification. The application has not launched, and no application tests were rerun for this public edition.

Prepared by the founder with AI assistance. The frontend, design, quality and review-lead roles in the historical record were AI assistants, not a staffed human review panel or independent certification.

## Contents

- [What went wrong](#what-went-wrong)
- [Current required behavior](#current-required-behavior)
- [Scope](#scope)
- [Verification still needed](#verification-still-needed)
- [Dated investigation and delivery record](#dated-investigation-and-delivery-record)
- [Related documents and evidence limits](#related-documents-and-evidence-limits)

## What went wrong

The parent request wizard had two reported faults.

First, moving from the budget step to Review could submit the request immediately. The user saw the review screen only briefly before the success screen appeared. The confirmed browser reproduction found that Continue and Submit reused the same button element. During the click that advanced the step, that element changed into a submit button, and the same click could submit the form.

The recorded fix gave Continue and Submit separate identities so the clicked control could not change into Submit during the event. It also prevented submission before the final step. An earlier suspicion about Enter-key submission was an investigation theory, not the confirmed cause of that click failure.

Second, the minimum and maximum budget fields were checked together, but changing one did not reliably refresh the other's error. An error could remain after lowering the minimum to a valid value, or fail to appear after raising the minimum above the maximum. The recorded fix rechecked both fields whenever either changed.

## Current required behavior

### REQ-001 — Deliberate, accessible submission

Reaching Review must not create a request. The screen must remain available for inspection until the user deliberately activates the final Submit Request button, either by pointer or by Enter or Space while that button has focus. A valid activation creates one request and then shows the success state.

The final step alone is not sufficient authorization to submit. Pressing Enter in an ordinary field on Review must not submit implicitly. On earlier steps, Enter in an ordinary single-line field should validate that step and advance only when its fields are valid; it must never create the request.

Keyboard handling must preserve the normal behavior of other controls. A multiline field must accept a newline; a text-input composition session must not be interrupted; pickers, lists, links, checkboxes and buttons must keep their own keyboard behavior. These September requirements go beyond merely checking which step is visible.

### REQ-002 — Budget errors follow both values

The maximum budget must be at least the minimum. Recheck the pair whenever either value changes, without changing validation timing for unrelated fields.

- If maximum is below minimum, show the error and block progression or submission.
- Lowering minimum to maximum or below must clear that error immediately, provided the individual values are otherwise valid.
- Raising minimum above an otherwise valid maximum must show the error immediately.
- Editing maximum must produce the equivalent valid or invalid result.
- Preset budget selections and manually entered values must obey the same rule. Empty or invalid values must retain the relevant field errors.

## Scope

These fixes concern the request-creation interface. They do not require a server contract or database change. The original June scope retained the request data format and success display. Subject/level choices, student count, schedule-grid presentation and rate-guide links were assigned to [specification 009](../009-tutee-post-taxonomy-and-ux/spec.md).

The original four-step screen arrangement is the context of this incident, not a permanent requirement that later wizard designs must keep four steps. Later product changes must still preserve deliberate review and submission.

## Verification still needed

Use an actual browser to reproduce the transition from Continue to Review. Component tests alone did not reproduce the original click behavior.

The verification must show that reaching Review creates no request, and that deliberate pointer, focused-button Enter and focused-button Space each submit once. It must also cover Enter in ordinary inputs on earlier and final steps, multiline input, text composition and picker selection. Budget tests must cover error appearance and clearing in both directions, plus blocked submission while invalid. Check that the affected flow remains usable at a narrow mobile width.

These are acceptance requirements. The June report does not prove the additional September scenarios pass.

## Dated investigation and delivery record

| Date | Recorded event | What it establishes |
|---|---|---|
| 21 June 2026, initial draft | The founder reported unintended submission and stale budget errors. Enter-key submission was a suspected cause while browser reproduction was pending. | An investigation starting point, not a confirmed diagnosis |
| 21 June 2026, AI-assisted design review | The review required browser reproduction, a final-step safeguard and both-direction budget checks; the wider user-interface batch remained separate. | A recorded planning review, not independent human approval |
| 21 June 2026, development report | Browser reproduction identified the reused-button click cause. The report described separate button identities, submission safeguards and budget revalidation, and reported two browser checks passing after previously failing. | A historical local-development result; the private report is not included and was not rerun for publication |
| 5 September 2026 | The design clarified deliberate final-button activation, keyboard exceptions and the correct direction of budget tests. | Corrected requirements; implementation and browser verification remain separate follow-up work |

The original document also reported a larger frontend suite and coverage figures. Those belong to that June revision; they are not fresh results or evidence that the September behavior is complete.

## Related documents and evidence limits

- [Requirements](requirements.md) — the acceptance scenarios and their identifiers.
- [Design](design.md) — the confirmed cause and the intended interaction safeguards.
- [Tasks](tasks.md) — the historical implementation and verification work list.
- [Current requirements register](../CURRENT-REQUIREMENTS.md) — precedence when older wording conflicts.

The original private screenshots, browser traces and test reports are excluded from the public edition. This page describes what the record reported and what still needs verification; it does not substitute general publication notes for test evidence.
