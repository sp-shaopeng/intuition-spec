# 06 — Tutee Post Creation

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. The parent wizard uses subject/level, availability, financial terms and explicit review/submit under 008/009. Enter in a textarea remains a newline; validation must not destroy the draft.
2. Creation submits to moderation as PENDING. The owner can edit PENDING under 021; an ACTIVE edit requeues moderation rather than silently remaining published.
3. Required audit is atomic with the mutation; in-app administrator notification rows/outbox intents commit with the mutation; external delivery runs after commit. Publication waits for approval.

## Authority

[Current requirements C06](../CURRENT-REQUIREMENTS.md) · [requirements md](../008-tutee-post-flow-bugfixes/requirements.md) · [requirements md](../009-tutee-post-taxonomy-and-ux/requirements.md) · [spec md](../021-tutee-request-flow/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 06 — Tutee Post Creation

## Trigger

Authenticated tutee submits Create Request implementation reference. Post is created and
immediately submitted (DRAFT → PENDING).

## Participants

- **Tutee** — authenticated user with role TUTEE
- **frontend Create Request implementation reference**
- **Tutee Post Controller** — submit operation for tutee posts
- **Tutee Post Service**
- **User Repository, Tutee Post Repository**
- **Analytics Event Publisher**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Tutee Post Controller implementation reference.
- Service: Tutee Post Service implementation reference. Status is set directly to PENDING on
  creation — there is no separate "submit" call.
- Allowed session durations: the of operation —
  Tutee Post Service implementation reference.
- Analytics events are NOT wrapped in after Commit here — they are emitted
  before the transaction returns. If the frontend-bound transaction rolls back after
  the event publish, a phantom analytics event may exist for a non-persisted
  post.
- **Gap vs spec**: the prompt states "notification (admin)" — no
  notification Service emit call exists in create And Submit. Admin discovery
  is via polling the pending queue (read operation for admin posts pending), not via
  notification.
- No audit log is written on tutee post creation — only on
  admin approve/reject (see [08-admin-approval md](08-admin-approval.md)).

## Failure modes

- 401 UNAUTHORIZED — signed session token missing/invalid (filter chain) or user gone.
- 403 FORBIDDEN — role mismatch.
- 400 VALIDATION ERROR — budget inversion or invalid session duration.
- 400 field validation (Jakarta Validation) for missing required fields.
- 403 from cross-site request forgery protection if X-XSRF-TOKEN header missing on POST (this path is NOT in
  the ignoring Request Matchers set).

## Related flows

- [08-admin-approval md](08-admin-approval.md)
- [25-csrf-protected-write md](25-csrf-protected-write.md)
