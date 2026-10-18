# INT-147 — Useful details in invitation notifications

This design was recorded on 17 June 2026 as the follow-up to INT-145. Its review roles were AI assistants. This public prose version explains the intended behaviour and acceptance checks without claiming a fresh implementation or test result.

## Problem

A tutor receiving several invitations needs to distinguish them. A generic message alone does not show which subject and level each invitation concerns. The server should supply structured information; the frontend should turn it into readable labels using the same formatting rules as the rest of the product.

## Server and storage

Migration 0055 adds an optional structured-data field to notification records. Older notifications keep an empty value, and no historical data needs to be invented or backfilled. The application mapping must agree with the database's JSON storage format.

For an invitation notification, the additional information contains the tuition-request identifier, subject code and level code. It contains identifiers and codes rather than a prewritten display sentence. The ordinary message body stays generic and the existing destination link remains unchanged. Other notification types can leave the additional field empty.

The notification response exposes the optional information to the frontend. The original contract represented it as JSON text for the browser to parse; that representation must be documented consistently on both sides.

## Browser behaviour

When an invitation has valid additional information, show a readable line identifying the invitation, subject and level, and keep the destination link. Use the existing subject and level labels; do not show raw internal codes.

When the additional information is absent, belongs to an unsupported notification type or cannot be parsed, show the original message body. A malformed or legacy notification must not crash the notification list.

## Acceptance checks

Server checks must verify the correct request identifier, subject and level in the additional information, while also confirming that the generic body contains no raw code. The guard against raw display codes applies to the visible body, not to the structured fields that deliberately carry codes.

Frontend checks must cover readable invitation details, missing data and malformed data. All cases must retain a usable notification and link. The API description and request examples must reflect the optional field, and database verification must confirm that older rows remain readable after migration 0055.

The original completion plan required relevant backend and frontend checks, contract comparison, review and running-development verification. Those requirements are not a report of a current passing run. See [testing and release](../../docs/testing-and-release.md) for the publication's inspection limits.
