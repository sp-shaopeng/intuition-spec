# 07 — Tutor Post (Listing) Creation

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Tutor listing creation remains optional relative to free-market Apply. Prefill from the canonical profile; listing overrides do not synchronize back into that profile.
2. Validate shared taxonomy, venue/location and allowed subject/level pairs on every covered surface. Submit the listing for moderation.
3. Persist required audit atomically and record in-app administrator notifications/outbox intents atomically and dispatch externally after commit. Listing submission alone is not approval, identity verification or contact-release authority.

## Authority

[Current requirements C05](../CURRENT-REQUIREMENTS.md) · [requirements md](../002-tutor-profile-and-onboarding-ux/requirements.md) · [requirements md](../025-taxonomy-stream-year-scoping/requirements.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 07 — Tutor Post (Listing) Creation

## Trigger

Authenticated tutor submits Create Listing implementation reference. Listing is created and
immediately submitted (DRAFT → PENDING).

## Participants

- **Tutor** — authenticated user with role TUTOR
- **frontend Create Listing implementation reference**
- **Tutor Post Controller** — submit operation for tutor posts
- **Tutor Post Service**
- **User Repository, Tutor Post Repository**
- **Analytics Event Publisher**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: post handler not shown here; pattern matches
  Tutee Post Controller. Service: Tutor Post Service implementation reference.
- subject Codes, level Codes, venue Types, group Sizes are sets — populated
  via the requested subjects with duplicate values removed (lines 85-90).
- years Experience default 0 if null.
- Same analytics-before-commit pattern as tutee post — see flow 06.
- load Ratings is called inside to Response (via find Own Posts /
  the to response operation); for a brand-new tutor with no reviews this returns an
  empty map and average Rating/review Count come through as null.
- **Gap vs spec**: no notification Service emit to admins on submission.

## Failure modes

- 401 UNAUTHORIZED, 403 FORBIDDEN — auth/role.
- 400 VALIDATION ERROR — rate inversion or Jakarta Validation.
- cross-site request forgery protection 403 if header missing.

## Related flows

- [06-tutee-post-creation md](06-tutee-post-creation.md) — symmetric tutee version
- [08-admin-approval md](08-admin-approval.md) — what happens next
