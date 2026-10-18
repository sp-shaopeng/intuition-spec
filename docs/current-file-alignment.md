# How the specifications compare with the current files

**Reviewed 18 October 2026. The application does not yet meet every current requirement.** Three new AI reviewers independently checked the backend, frontend and browser-test files, and the original specifications. They did not use the earlier publication reviews as evidence. The author then corrected the public documentation and returned the changes for review.

This review separates three things: an agreed requirement, behaviour supported by the current files, and evidence that the behaviour works when run. A missing feature stays a missing feature; rewriting a requirement to match the older application would hide the problem. InTuition remains unlaunched.

The [workflow improvement plan](workflow-improvement-plan.md) sets out the proposed repair order and the evidence each fix needs. The [user-flow guide](user-flows.md) connects these findings to 21 main journeys. Those plans do not close any finding below; closure needs checked implementation and test evidence.

## What was compared

- The current specification working copy, including its uncommitted corrections and additional documents.
- The current frontend and backend source, selected tests, dependency files and build checks.
- The separate Playwright repository, including which checks use a browser and which call the server directly.
- The public Markdown collection, its guides and both specification indexes.

All **346 eligible specification documents** have public counterparts. The test-file counts were independently reproduced: **125 backend files named as tests, 57 named as integration tests, 177 frontend test files, and 57 Playwright specifications**. These are inventory counts, not passing-test counts. Full application suites, live delivery, hosting and recovery exercises were **not run** for this review.

Private evidence records identify source files and lines. They remain private because the publication excludes application code, operating details and private records. The table below covers all 17 settled requirements; “consistent in inspected paths” is deliberately narrower than certification of the whole feature.

## Requirement-by-requirement result

The identifiers refer to the [current requirements register](../specs/CURRENT-REQUIREMENTS.md).

| Requirement | Result from current files | Boundary or remaining work |
|---|---|---|
| C01 — guest browsing | Gap found | Public list pages exist, but the website hides guest filters and pagination. See G01. |
| C02 — tutor pays the agency | Consistent in inspected recording paths | Staff record agency receipt; revenue totals use paid commissions. This is a manual assertion of receipt, not bank reconciliation. |
| C03 — activation and billing | Partly implemented | The first-month calculation, locked plan and completion on payment exist. First-session confirmation, invoices and due handling do not. See G02. |
| C04 — contact access | Core website rule consistent; misleading text remains | Match screens reveal contact for ACTIVE or COMPLETED matches. The server separately allows authorized assigned participants in PENDING matches. Two earlier screen messages wrongly promise contact on post approval. See G06. |
| C05 — profile and optional listing | Partly checked | Applying without a listing and reading profile completeness are supported. Applicant cards use the public handle and coarse location. Whether every permitted listing override can be blank remains unverified; the creation form still requires some values. |
| C06 — pending requests | Initiation and editing supported in inspected paths | Pending requests can start invitations/interviews. Invitation acceptance still requires an active post while O01 is unresolved. This review does not establish every rejection/removal cascade. |
| C07 — interview lifecycle | Partly implemented | Negotiation and scheduling exist. The website cannot record both participants as absent; updating the meeting link does not send the required new-link notice or write the required audit record. See G05. |
| C08 — contact changes | Significant gaps found | Confirmation templates, the new-address destination, old-phone warning and owner revocation are incomplete or incorrect. See G03. |
| C09 — sessions | Main mechanisms present; exceptions remain | Thirty-minute sessions, protected cookies, renewal and server logout are present. A filter interaction may break the unchanged-cookie rule on a rate-limited renewal. The login screen cannot complete an MFA challenge. See G10 and G11. |
| C10 — Singapore time | Gap found | Display uses Singapore time, but participant input and staff prefill depend on the computer's local timezone. See G04. |
| C11 — WhatsApp and reminders | Disabled as required; safeguards incomplete | The database scheduler is present. Duplicate prevention, schedule-version checks and recovery from conflicting rescheduling remain incomplete. See G09. |
| C12 — durable delivery | Partly implemented; confirmed loss paths | Saved delivery requests and retry workers exist, but disabled/missing-key delivery can be marked complete and queue trimming can remove unfinished work. See G08. |
| C13 — readable contacts and soft deletion | Consistent in sampled paths | The account model stores readable contact fields; inspected account/post deletion marks records deleted. This does not settle the separate retention and erasure policy in O08. |
| C14 — database changes | File structure consistent; execution not verified | Numbered XML migrations and schema validation are present. Approval recovery, release ordering and rollback rehearsal were not established. Removing a table before the application stops using it remains unsafe. |
| C15 — tests and sample data | Configured thresholds and file counts confirmed | Backend thresholds are 97.5% lines and branches; frontend thresholds are 97.5% lines and 88% branches, with exclusions. Achieved coverage and all sample-data/date-range requirements were not reverified. Some tests assert older behaviour. |
| C16 — production arrangement | Selected mechanisms present; launch acceptance unverified | The public version response is minimal and detailed status routes require staff access. Authored deployment work does not prove recovery, retention or a successful release. |
| C17 — service and recovery targets | Requirements retained; measured compliance not established | No fresh evidence establishes delivery deadlines, admin-panel or public uptime, allowed data loss, restore time or backup retention. The clocks and permission questions in O12–O14 remain unresolved. |

## Application gaps that remain open

These findings explain why an overall “all aligned” verdict would be wrong. They are source-based findings; where a path depends on a race or a combination of requests, the needed execution check is stated explicitly.

### G01 — Guests cannot use the required search controls

Both public browse screens hide their filters and page controls until login. Existing component tests expect that restriction, so a green test run would preserve the mismatch. The requirement still allows guest filtering and the first five result pages. Completion needs updated controls and tests that check both anonymous access and its page limits.

### G02 — The billing journey stops short of the agreed process

The current application can activate a match, create a pending commission and let staff record it paid. It has no first-session confirmation action, invoice stage or payment-due workflow. The required seven-day follow-up therefore has no complete implementation either. These stages must be built and checked together; unresolved advance-payment, dispute, partial-receipt and refund rules must stay explicit rather than being inferred from the existing payment action.

### G03 — Contact changes are not complete

The service creates confirmation and warning requests, but the email renderer does not support their templates. An email-change confirmation is addressed through the account's current email rather than the proposed new address. The warning also lacks a fixed record of its original destination. Merely adding the missing templates would not correct those destination errors.

There is no owner revocation action or separate revocation link, and placing the old phone number inside an email does not deliver a warning to that phone. The phone-warning transport remains an open decision. Completion needs delivery through the full path, proof of control of the new email, warnings to the correct old destinations, and safe handling when confirmation and revocation compete.

### G04 — Interview input can shift the agreed time

The participant picker labels its input as Singapore time but interprets it in the browser's timezone. Staff prefill has the same dependency. For example, 14:00 Singapore time should represent 06:00 UTC; a browser set to UTC currently interprets that input as 14:00 UTC. Display formatting is correct, which can make the discrepancy harder to notice. Completion needs explicit Singapore-time conversion and independently calculated expectations under more than one browser timezone.

### G05 — Interview follow-up is incomplete

Staff can mark either the tutor or the parent absent, but there is no website action for both being absent. Changing a scheduled meeting link saves it and updates reminder scheduling, but does not immediately notify both participants or record the specified audit event. A reminder is not a substitute, especially when the change occurs less than 24 hours before the interview. Completion needs both-participant absence handling and a checked link-change notification/audit path.

### G06 — Some contact-sharing text contradicts the actual gate

The invitation dialog and tutor-detail request flow promise contact details once a post is approved. The match screens actually require ACTIVE or COMPLETED. The inspected screen paths do not show a contact leak here; they show a misleading promise. The messages need to explain assignment and activation consistently. The server's authorized access for an already assigned PENDING match is a separate rule and should not be confused with the website display rule.

### G07 — The request-form keyboard follow-up is unfinished

The original Continue/Submit button fix is present. However, Enter on an earlier step still advances without the required checks for the active control and text composition, and the final step still allows implicit submission. The existing browser test covers only part of the corrected keyboard contract. Completion needs the missing guards and cases for text areas, composition, focused controls and deliberate final submission.

### G08 — Delivery can be recorded as complete without reaching the recipient

When the email delivery switch is off, the queue handoff returns without adding the message, while the outbox marks the saved request sent. A missing provider key instead pauses the email worker before it reads new or unfinished messages; it does not acknowledge them. With the delivery switch on, messages can remain queued and undelivered even though the outbox handoff is marked sent. Queue trimming does not first protect all unfinished work. These are source findings, not observed runtime failures. G08 remains open. Completion needs failure-and-recovery checks that inspect saved work, including restarts and a full queue.

### G09 — Reminder recovery needs more than a scheduler

The current reminder identifies an interview but does not verify the expected version of its schedule or save a durable per-recipient duplicate guard. A rescheduling attempt that meets an already-running job can be ignored without a later repair step. This source review did not execute the race or crash scenarios. WhatsApp must remain disabled until the corrected duplicate, outdated-reminder and recovery requirements are implemented and tested.

### G10 — A rate-limited renewal has an untested cookie edge case

The renewal handler rejects excessive requests before writing its own cookies. An earlier authentication filter can nevertheless clear an access cookie for a blocked account or a token made stale by a password reset, before the handler returns 429. The inspected test does not send that combination or assert the complete response-cookie boundary. This is a source-path finding requiring an integration test. Resolve it without silently weakening blocked-account protection or the approved unchanged-cookie requirement.

### G11 — The login screen cannot finish an MFA challenge

The backend has multi-factor authentication support, but the current login screen shows an explanatory error when a challenge is required; it has no challenge-entry step. That is a missing website capability for environments that require MFA, not evidence that MFA is absent from the backend. Completion needs the permitted challenge and recovery journeys checked in the environment where they are required.

### G12 — The guest request button differs from the retained navigation rule

The current signed-out Post a Request button opens parent registration directly. The earlier founder instruction says that general acquisition buttons should open login, with registration reached through an explicit account-creation choice. This review found the mismatch but did not establish an approved replacement decision. The route reference now states the actual destination; that documentation correction does not approve the behaviour change.

## Corrections to this publication

The business guide now distinguishes guest-search and billing requirements from implemented behaviour. It describes moderation as the available approve/reject actions rather than inventing a separate request-changes action. Settled delivery obligations are kept separate from unresolved transport and timeout decisions.

The [route reference](../specs/reference/frontend-routes.md) is checked against the current route definitions, including redirects, role-specific notifications, onboarding and contact confirmation. The [testing guide](testing-and-release.md) identifies mixed browser/API coverage and tests that preserve older behaviour. Neither an API-only matching flow nor a file total proves the complete website journey.

The source comparison also found business conditions damaged by conversion to prose, including date thresholds and financial grouping. Those conditions are restored as readable rules. Conflicting historical expiry or retention proposals remain labelled; the conversion must not silently choose a new policy. Nonessential company registration and address values have been replaced with descriptions of the required legal fields.

## What would close this review

All three reviewers accepted the corrected documentation within their separate scopes after rechecking it. The source reviewer verified 66 restored explanations across 25 documents; the frontend reviewer checked all 56 current route entries; the backend reviewer checked that the gap descriptions match the inspected implementation. Final publication checks found 358 Markdown files, complete specification indexes, no broken local links, and no credential-candidate or retained company-identity matches. None of those results closes the application gaps above.

The documentation can accurately describe an incomplete application. Full product alignment requires implementing the remaining agreed behaviour, resolving the genuinely open decisions, and collecting fresh evidence for the affected versions. The original six publication reviews and this later comparison do not replace those checks. The inspected private application files were not changed during this documentation review.

The source comparison also found a timing disagreement across the retained requirements: AC-05 starts the seven-day confirmation period when the email is sent, §7.7.1 and AC-11 start it at match activation, and BRD BR-010 says when the notification is received. No later controlling decision settles that difference. It remains open separately from O12, which concerns the reminder delivery deadline and detection schedule. The BRD also permits staff to override confirmation at their discretion before timeout, while other clauses describe override after seven days. The current register now tracks the starting event as O13 and earlier override permission as O14; O15 separately records the submission-versus-approval post-expiry conflict. None has been decided.

[Back to the README](../README.md) · [Current requirements](../specs/CURRENT-REQUIREMENTS.md) · [Testing and release](testing-and-release.md)
