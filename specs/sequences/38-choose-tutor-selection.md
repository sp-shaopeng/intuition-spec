# 38 — Choose-Tutor Selection (parent picks → admin queue → assign)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. The parent chooses a tutor for their own eligible job; persist a selection request only.
2. Admin assigns through the tutor User Id/selection-request path; optional tutor Post Id must not block a listing-less tutor.
3. Assignment atomically updates job/match/winner/competitors and required audit, with in-transaction in-app notification/outbox intent and post-commit external delivery. Contact release remains separately gated.

## Authority

[Current requirements C05](../CURRENT-REQUIREMENTS.md) · [int-034-free-market-assign md](../architecture/int-034-free-market-assign.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 38 — Choose-Tutor Selection (parent picks → admin queue → assign)

## Trigger

A parent (TUTEE) reviews the personal information-free applicant cards on their own job, picks a tutor
("choose"), which raises a **selection request** for an admin to action. The admin works
an OPEN-selection queue and turns a chosen request into a match via the M4 assign path.
Choosing **never** creates the match directly (M18 TR9).

## Participants

- **Parent (TUTEE)** / **Admin**
- **frontend** parent applicants view / admin selection queue
- **Tutee Application Controller** — read operation for tutee posts id applicants,
  submit operation for tutee posts id select (the pre authorize operation)
- **Admin Selection Request Controller** — read operation for admin selection-requests (the has role operation)
- **Job Application Service list Applicant Cards For Owner**, **Tutor Selection Service choose Tutor / list Open Requests**
- **Admin Match Controller** — submit operation for admin matches (the assign step; see flow 09)

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Parent surface: Tutee Application Controller implementation reference — role TUTEE + explicit owner check
  (403 IDOR) in the service. Choose returns **201 on create, 200 on re-choose** (62).
- Admin queue: Admin Selection Request Controller implementation reference — lists **CREATED (open)** requests
  only, newest-first, paginated; personal information-free (public handle + opaque tutor User Id).
- The admin response carries a nullable **tutor Post Id** (batch-loaded from the applicant's
  application) so the frontend can assign via submit operation for admin matches — see
  ROADMAP PDO-3 deploy note; the assign itself is flow 09.
- Choosing is decoupled from matching by design: a selection request is a liaison signal, not a
  state change on the job.

## Failure modes

- 403 — non-TUTEE on the parent endpoints, or a TUTEE who is not the job owner (IDOR).
- 403 — non-ADMIN on the selection queue.
- Validation 400 — select body missing tutor Id.

## Related flows

- [28-tutor-apply-withdraw md](28-tutor-apply-withdraw.md) — how applicants get onto the job
- [39-admin-applicant-view md](39-admin-applicant-view.md) — the admin's view of applicants
- [09-admin-match-creation md](09-admin-match-creation.md) — the assign that turns a selection into a match
- [37-contact-reveal-post-assignment md](37-contact-reveal-post-assignment.md) — what unlocks after assign
