# INT-143–146 — Follow-up corrections from implementation review

This proposal was recorded on 13 June 2026 after review of the INT-141 changes. The review perspectives were AI assistant roles, not employees or independent human approvals. The requirements below describe the corrections and their intended verification; no application tests were rerun for this public edition. The [current register](../CURRENT-REQUIREMENTS.md) governs later changes.

The batch keeps the existing visual identity, introduces no new dependency, and preserves the rule that the frontend turns subject and level codes into readable labels. INT-143 and INT-144 were the required fixes; INT-145 was recommended, with the lower-priority INT-146 items recorded separately.

## INT-143 — Make the raw-code display guard more precise

The server-side source scan meant to detect unreadable subject and level codes had two weaknesses. It could reject unrelated error, area or postal codes because the matching rule was too broad. It could also miss a subject-and-level label assembled from intermediate variables rather than directly from the original fields.

Narrow the scan to direct combinations of subject or level display codes. A test should deliberately introduce the prohibited form and show that it fails, then use an unrelated error code and show that it is allowed.

Do not claim that this scan catches every occurrence across the application. A check of the match-dashboard mapper protects that mapper only. Use three complementary checks: frontend rules against displaying raw codes, explicit assertions at each mapper that produces a display label, and the server-side scan as an additional limited safeguard. Extending the scan to guess the meaning of arbitrary intermediate variables risks more false alarms.

## INT-144 — Count searches rather than result pages

The recorded implementation emitted another list-view event when someone moved to the next page of the same results. That inflated the number of searches used in the browse-to-detail metric.

Emit one event when the view first opens and another when its search filters change. Include all supported filters, such as subject, level, region, rate, venue and group size, when deciding whether a search has changed. Exclude the page number. Each event also carries the current result total.

Both tutor and assignment browsing should use the same tracking helper so the payload and change-detection rules do not drift. Regression checks must show one additional event for a filter change and none for a page change, including filters beyond subject and level.

Define the outcome metric as the percentage of tutor searches followed by at least one tutor-detail view. Count each qualifying search once. Dividing all detail openings by searches can exceed 100% when someone opens several tutors and is not this conversion metric.

## INT-145 — Keep invitation messages readable

A generic invitation message prevents raw subject and level codes from reaching the visible text, but makes multiple invitations difficult to distinguish. Retain a test that the message contains no raw code and that its destination link is present.

The improvement that makes invitations distinguishable is tracked separately in [INT-147](int-147-structured-notification-data-spec.md): supply structured subject and level information and let the frontend render readable labels. Do not quietly reintroduce server-composed labels as a shortcut.

## INT-146 — Lower-priority follow-ups

- A missing request title must remain a supported response case. The recorded consumers already provided a fallback; preserve a regression note rather than claiming the field is always populated.
- Add a running-system check that a missing assignment or post returns the defined not-found response. An isolated unit test does not prove the assembled request path behaves the same way.
- The tutor's match card identifies the job by subject and level, while the tutee's card leads with the tutor's name. Record this as an intentional difference for the two audiences unless a product decision asks for symmetry.

## Verification and completion

Verify each behaviour above, then run the relevant frontend and backend checks. Preserve the existing frontend display-code rule and narrow-screen checks. This batch itself proposes no new visual surface or dependency.

A review finding returns to implementation and verification before closure. The historical process called for development deployment and source publication after checks; this document is not evidence that those steps completed or that InTuition is publicly live.
