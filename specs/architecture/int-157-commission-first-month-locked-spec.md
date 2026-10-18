# INT-157 — Lock the agreed first-month commission at activation

**Current requirement:** When an administrator activates a match, record the agreed first-month lesson plan and lock the commission calculated from it. Later changes to lesson arrangements must not silently recalculate that commission. The tutor owes this commission to the agency; it is separate from lesson payments between the parent and tutor.

The original proposal was written on 17 June 2026 and clarified on 5 September 2026. This page preserves the settled rules and identifies the remaining commercial decisions. It is a design record, not proof of a completed release or a new test run. The [current requirements register](../CURRENT-REQUIREMENTS.md) takes precedence.

## Why record the plan

The historical implementation already stored a fixed first-month fee and commission amount. However, an administrator entered the fee directly, without a structured record of the agreed lessons behind it. That made the basis of the calculation difficult to inspect.

The agreed plan supplies that basis. It contains three positive inputs:

- The agreed hourly rate in Singapore dollars.
- The agreed number of sessions in the first month, as a whole number.
- The agreed hours per session, allowing a fractional duration such as one and a half hours.

Activation records these inputs together with the resulting first-month fee, commission rate and commission amount. A stored copy of values at that moment is often called a snapshot.

## Fee and commission calculation

Multiply the agreed hourly rate by the first-month session count and the hours per session. Round the result to the nearest whole Singapore dollar to obtain the locked first-month fee.

The standard agency commission is 50% of that rounded first-month fee, again rounded to the nearest whole Singapore dollar. These are two rounding steps: first calculate the fee, then calculate the commission from that fee.

For example, S$60 per hour for four sessions of one and a half hours gives a first-month fee of S$360. The standard commission is S$180.

An authorised administrator may use the existing commission-amount override for the supported exception cases. The override must be at least S$1 and no greater than the locked first-month fee. The actual stored amount must be shown, so an overridden amount is not incorrectly presented as the standard 50% calculation.

Each plan input must be greater than zero. The rounded first-month fee must be at least S$1; a positive but tiny input must not create a zero-dollar fee.

## Suggested values and administrator confirmation

Hours per session can initially be suggested from the tuition request's session duration. The agreed hourly rate is entered or confirmed by the administrator. A suggested rate is a convenience, not proof that the parties have agreed to it.

For fixed weekly frequencies, the historical proposal used 4.33 weeks per month to suggest a session count. Under that provisional convention, one, two and three sessions per week suggest four, nine and thirteen sessions in the first month after rounding. The administrator must confirm or change the suggested count.

**FLEXIBLE frequency has no prefilled session count.** It does not represent a number that can be multiplied. The administrator must enter a positive whole session count. This rule is settled and is not reopened by the unresolved choice of monthly multiplier.

## What is stored and displayed

The agreed-plan values belong beside the locked fee in the commission record. The design identifies additive migration 0057 for these fields. “Additive” means adding the new information without deleting the old fee records. Hours must retain fractional values rather than being restricted to whole hours.

Legacy records may have no agreed-plan values. In that case, display the existing fee and commission without inventing a plan. Preserve the earlier internal activation paths needed for compatibility, while the new agreed-plan path calculates the fee and uses the existing commission-recording behaviour.

The administration form should show the three inputs and a fee-and-commission preview before activation. The resulting match or commission detail should show the saved basis and actual amount. The response needs to supply the numerical values separately so the interface can explain the calculation, including an override, without parsing a prewritten sentence.

Once activation has recorded the plan, ordinary changes to a request, profile or lesson arrangement must not alter those saved values. A future explicit renegotiation process would need a separate requirement and is outside INT-157.

## Amount, due date and receipt are separate

Activation fixes the amount. It does not establish that the first lesson has happened or that money has been received. First-session confirmation continues to gate invoicing and due handling under the current requirements.

Marking agency commission as paid means the agency has received the tutor's commission. A parent paying for lessons, or a dispute being decided in the agency's favour without receipt, is not that event. The [current register](../CURRENT-REQUIREMENTS.md) records separate unresolved treatment of advance or partial receipts and later refunds.

## Acceptance checks still need evidence

The tests must show that activation calculates and stores both rounding steps, validates every input and the minimum fee, and correctly handles the allowed override range. They must also show that a legacy record remains readable without plan values.

To check the lock, activate a match with a known plan, inspect the saved fee and commission, then change the underlying lesson arrangement through supported actions. Inspect the stored record again: the original plan and commission must remain unchanged. A statement that the values are immutable is not sufficient evidence.

The interface checks must show the preview before activation and the saved basis afterward, including fractional hours, FLEXIBLE frequency without a default, and an override that differs from the standard commission. Database checks must confirm that the additional fields can be introduced without losing older records. The API description, request examples and interface expectations must agree on the new inputs and returned values.

No fresh passing result for these checks is asserted here. Private test reports and operational captures are not included in this public edition.

## Review history and remaining decisions

The June review used AI assistant roles for architecture, data, API and testing perspectives. These were not employees or independent human approvals. The recorded review corrected the flexible-frequency default, preserved older activation paths, chose commission records for the saved plan, retained fractional hours and clarified legacy handling.

Two commercial choices remain open in the [current requirements register](../CURRENT-REQUIREMENTS.md):

1. Whether the agreed hourly rate must remain inside the originally posted budget or rate range. A hard restriction could prevent a legitimate renegotiated rate. The historical proposal allowed a suggested value without imposing that restriction.
2. Whether fixed weekly frequency should suggest a first-month session count using 4.33 weeks or four weeks. The historical default was provisional; it must not be presented as a settled commercial rule.

These open choices do not change the settled requirements to lock the confirmed plan at activation, require a manually entered count for FLEXIBLE frequency, tolerate missing plan values on legacy records, and retain the first-session due trigger.
