# INT-153 — Tutor subject and level capabilities

A tutor may teach one subject at some levels and another subject at different levels. Independent subject and level lists cannot express those distinctions accurately. This design records individual subject-and-level pairs on the tutor's profile.

The proposal and review history date from 17 June 2026. Later requirements, including specification 025, take precedence where noted. The historical implementation account below was not rerun for this publication. Review roles were AI assistants, not independent human approvals. Refer to the [current requirements](../CURRENT-REQUIREMENTS.md) for unresolved decisions.

## Profile data and validation

Each stored pair links one tutor to one subject and one level. The same pair must not be duplicated for that tutor. Migration 0056 creates the association records and an index that supports finding tutors for a subject and level. Existing tutors initially have an empty collection; no capabilities are inferred for them.

The reviewed design requires database references to the existing subject and level tables. Codes are limited to 50 characters, matching those tables. This corrects the original draft's suggestion of 64 characters and its assumption that reference constraints were unavailable. Application validation must reject unknown codes with a clear validation response before a database constraint becomes the last safeguard.

A profile update may contain up to 200 pairs. Each supplied code must be non-blank and within the length limit. If the pair collection is absent, leave the existing values unchanged. If a collection is supplied, it replaces the previous collection, so submitting a subset removes omitted pairs. The profile response returns the saved pairs.

A full table rollback was part of the original migration plan. Any actual rollback still needs a compatibility review against the application version that would run afterward. This document does not establish that a rollback was executed.

## Browsing, application eligibility and administration

Tutor browsing supports a subject-only filter or a combined subject-and-level filter. A combined filter matches the exact pair; it must not treat two independent lists as proof of that pair.

The profile must contain at least one pair before the tutor can apply. The recorded profile-completeness check already had six fields, so adding this collection made it the seventh. The interface's progress display and missing-field explanation must agree with that rule.

The historical implementation proceeded without a per-post restriction requiring the tutor's pair collection to cover the particular job before applying. A stricter restriction would change the marketplace policy and requires a separate decision. Administration can put apparently eligible tutors first and warn about a mismatch, but the recorded design does not hard-block administrator assignment on that basis.

Tutor listings retain their own subject and level selections. The profile's exact pairs and a listing's independently selected subjects and levels are different representations. Specification 025 requires that each selected subject and level have support among the allowed pairs; it does not require every possible combination of the two lists. A stronger profile-coverage warning remains advisory and must not silently become a new create-or-apply restriction.

## Interface and response behaviour

The profile editor allows a tutor to select levels for each subject and shows readable subject-level summaries. Display labels are composed by the frontend; raw codes are not shown as the user-facing description.

The historical review found a gap between the profile response and the tutor-detail response: the detail page expected the pair collection, but its server response did not include it. The recorded fix supplied the owning profile's pairs on individual detail, own-listing, create and update responses.

List responses omit that collection where the cards do not display it. This avoids loading an additional collection separately for every row. Where a page does need many tutors' pairs, use a joined or batched read and verify the number of database queries instead of issuing one extra query per tutor.

The small public showcase strip intentionally does not include the full pair collection. Its purpose is a limited preview; the more detailed capability display belongs on the authenticated tutor-detail page. This is a recorded scope choice, not a claim that the missing display passed an acceptance test.

## Acceptance checks

Tests must demonstrate replacement of the saved collection, removal when a smaller collection is submitted, unknown-code rejection and the database reference safeguard. Check the empty-collection application rule using a tutor whose other required profile fields are complete, so another missing field cannot make the test pass for the wrong reason.

Browsing checks must cover subject-only filtering and exact-pair filtering. Include a query-count check to detect an extra database read for every returned tutor. The full journey should set capabilities, find the tutor through the appropriate filters and allow an otherwise eligible application; the empty-capability case must be refused.

Response checks must verify that individual tutor details include the profile pairs while list paths that omit the collection continue to avoid unnecessary reads. Interface checks must confirm readable labels, editor updates and the seventh profile-completeness field. Keep the API description and examples consistent with these responses.

These are required observations and a historical account of changes, not a fresh passing test report. The [testing and release overview](../../docs/testing-and-release.md) explains the present evidence boundary.
