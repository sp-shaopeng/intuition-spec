# INT-110/121/122/123/124 — Reliability, security and interface corrections

> [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers in the historical record are AI roles, not employees. Reported test results were not rerun for publication; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

This batch was drafted on 10 June 2026 and reviewed on 11 June. It proposes corrections without a database migration, new stored status or breaking interface change. The following describes required behaviour and checks, not a claim that this publication performed the work.

## INT-110 — Keep deployment access keys outside copied project files

The historical review found a private access key inside a directory copied during deployment. Exclusion rules reduced the chance of copying it but did not remove the risk: a different copy operation or backup could still include the key.

Move the key outside the project and deployment directories. Restrict access to its owner and update the private deployment instructions. Keep the existing exclusions as an extra precaution. The review required rotating this key regardless of whether a prior accidental copy could be proved: install the new public key, verify access with the new key, remove the old authorisation, and remove obsolete copies from the project and backups.

The check must show that a trial deployment would copy no access key, the new key works, the old key is rejected, and the deployment account has only the intended new authorisation. No key, command or private location is included here. This publication does not certify that rotation occurred.

## INT-122c — Supply secrets separately for each environment

The review found signing and encryption values embedded in development configuration that was also copied to another environment. A “development only” label does not prevent accidental reuse.

Supply every required secret through private environment configuration. Example configuration should contain placeholders only. Local development and the deployed environment must use distinct values. Confirm that neither the deployment files nor the copy operation embeds literal secrets, and that each service starts with its intended private configuration. Do not print the values while comparing environments.

The original encryption rationale predates the approved change to plain-text email and phone fields. The [current requirements](../CURRENT-REQUIREMENTS.md) govern contact storage; this historical key-handling task is not evidence that contacts are encrypted today.

## INT-122a — Restrict operational diagnostics

Ordinary tutor and tutee accounts must not read detailed metrics, build information or monitoring data intended for administrators. The public health check remains available. Test anonymous access to health, rejection of ordinary-account access to restricted diagnostics, and authorised administrator access. Also verify that the authorised monitoring collector still works after tightening permissions.

The later production plan permits a separate minimal public version response containing only the deployed revision identifier. That exception does not open the detailed diagnostic endpoints. See [specification 027](../027-aws-production-deployment/spec.md).

## INT-122b — Do not reveal whether another person's match exists

A person who is not a participant must receive the same not-found response for an existing private match as for a nonexistent match. Otherwise, different responses can expose which identifiers exist. Keep the response free of contact information and retain the required access record.

Test both cases for identical status and response shape. An authorised participant must still receive contact through the permitted route. The website's separate ACTIVE/COMPLETED display rule remains governed by the current requirements.

## INT-123 — Keyboard access and shared interface behaviour

### A. Navigation and focus

Use a straightforward expandable navigation section unless the full keyboard behaviour of a menu is implemented. The trigger must expose whether the section is open and identify the content it controls. Closing with Escape or an outside action should restore focus to the trigger. The mobile drawer also needs Escape handling.

Test opening and closing with the keyboard, focus restoration, and the mobile drawer. Do not apply menu-specific accessibility labels to controls that lack the corresponding keyboard behaviour.

### B. Announce newly revealed contact information

Keep the screen-reader announcement container present before contact arrives, then put the authorised contact content inside it. A container that appears already filled may not announce its first update. Test that the same container exists before and after the content change. Change the existing notification component only if its first announcement is actually being lost.

### C. Give every field a unique identifier

Two fields with the same visible label must still have different identifiers. Use an explicitly supplied identifier when present; otherwise generate a unique one. Keep each label, hint and error associated with the correct field.

Test two Password fields, an explicit identifier override, and the association between a field and its error message.

### D. Remove unused assets and files

Confirm that each candidate file is unused before removing it. Remove the unused sample data, stylesheet and image assets identified in the private review. Run the build, style checks and relevant tests afterward. Recheck coverage because adding or removing measured files can change its denominator.

### E. Share account-action behaviour

The tutor and tutee profile pages duplicated the delete-account and change-password dialogs. Move each behaviour into one shared implementation used by both pages. Keep incorrect-password feedback inside the dialog; an expected credential error during reconfirmation must not cause an unrelated global sign-out.

Test both profile pages through the shared behaviour. Future fixes should be made once and apply to both.

## INT-124 — Clear and accurate interface details

| Item | Required behaviour | Regression example |
|---|---|---|
| A. Job status | Derive the badge from the actual post status. An active job is open; a filled job must not also say Open. | A matched job shows Position filled and no Open badge. |
| B. Application links | Link the job title to an accessible detail page. If a closed job has no accessible detail, show plain text. | Following a title reaches its actual detail; no link points to a missing anchor in a public list. |
| C. Optional tutor listing | Read the server's own-listings response as its actual list shape, then offer eligible active listings. | A response with one active listing makes that choice visible and sends its identifier when chosen. |
| D. Wrong-role navigation | Send signed-in users to their own dashboard and explain the restricted area. | A tutor opening a parent area reaches the tutor dashboard and sees “That area is for parents.” Equivalent messages cover tutor and administration areas. |
| E. Empty matches | Explain the choose-and-confirm process. Interviews are optional. | The parent and tutor messages both describe family selection and agency confirmation, without saying an interview is mandatory. |
| F. Other-applicant count | Count other applicants on the selected job, not selections across all jobs. If the count is unavailable, omit the number. | A job with three applicants reports two other applicants even when seven selections exist elsewhere. With a single applicant, there are no other applicants to notify. |

## INT-121 — Consistent server behaviour

### A. Prevent duplicate interviews with the same tutor

Both the listing route and the route without a listing must identify the tutor by their account. A previous interview requested without a listing must still be found when a later request comes through that tutor's listing. Keep the separate three-interview cap per request unchanged.

Test a request without a listing followed by one through a listing owned by the same tutor. The second must receive the existing duplicate-interview error with status 422.

### B. Use one time standard for stored events

Use Coordinated Universal Time for stored payment, match completion, profile update, moderation, post submission/closure, authentication and lock-related timestamps. Prefer an injected clock so tests can use a fixed time. Do not rewrite existing records or alter comparisons without checking their time basis.

With a fixed test clock, recording payment and match completion in the same operation must produce equal times to the second. Audit the remaining stored timestamps for accidental dependence on the machine's local timezone.

### C. Restrict no-show actors to the two participant roles

The no-show action accepts only tutor or tutee as the responsible participant. Use the shared role definition instead of a separately maintained text rule. A valid tutee value remains accepted; an invalid value still returns status 400.

### D. Reject interview times in the past

Staff scheduling must apply the same past-time check as participant proposals before changing the interview to scheduled. Compare the supplied instant with the current clock. A past time receives the existing slot-in-the-past error with status 422; a valid future time succeeds.

### E. Align the commission documentation with the service rule

The historical request documentation allowed zero while the service required at least one Singapore dollar. Align the description with the service floor and the later agreed-plan rules. This wording task does not approve a different fee or rounding policy; [INT-157](int-157-commission-first-month-locked-spec.md) and current requirements govern those decisions.

## Completion checks and work order

Start with the small status, link and message corrections, then access restrictions, shared field/dialog behaviour, interview guards and time handling. Keep secret relocation and rotation as separately verifiable operational work.

Acceptance requires the behaviours above, their negative cases and the relevant build and test checks. The historical coverage targets were 97.5% of measured backend lines and branches, and 97.5% of measured frontend lines and 88% of branches. See [testing and release](../../docs/testing-and-release.md) for the current inventory, exclusions and automation limits. A historical target is not a fresh passing result.
