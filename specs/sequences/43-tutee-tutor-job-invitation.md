# 43 — Tutee→Tutor Job Invitation (INT-044)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. The parent may initiate an invitation on their own live PENDING or ACTIVE post; use the sanctioned tutor identifier.
2. Invitation acceptance timing is O01 pending explicit resolution. Acceptance reuses Apply+Choose and does not create a Match; publication/assignment still require approval.
3. Withdrawal, job close/rejection/removal and assignment drive explicit cause-preserving transitions. There is no invented background invitation timeout; use INT-062’s CLOSED versus assignment EXPIRED rules.
4. Use INT-044 error/status semantics and logical row retention. Contact remains restricted to assigned participants and frontend match-state gates.

## Authority

[Current requirements C06](../CURRENT-REQUIREMENTS.md) · [spec md](../021-tutee-request-flow/spec.md) · [int-062-cascade-close-spec md](../architecture/int-062-cascade-close-spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 43 — Tutee→Tutor Job Invitation (INT-044)

## Trigger

A tutee has an ACTIVE job posting and wants to proactively invite a specific tutor
to apply, rather than waiting for the tutor to discover and apply organically.
Accepting an invitation folds into the existing apply + choose-tutor funnel — no
Match is created directly (AC-04 holds; admin-only matching is preserved).

## Participants

- **Tutee (TUTEE)** — sends and manages invitations
- **Tutor (TUTOR)** — receives and responds to invitations
- **Admin** — actions the selection-request queue after the tutor accepts
- **frontend** — My Jobs / Tutor Invitations pages
- **Tutee Invitation Controller** — submit operation for tutee posts id invite, delete operation for tutee invitations id
- **Tutor Invitation Controller** — read operation for tutor invitations, submit operation for tutor invitations id accept, submit operation for tutor invitations id decline
- **Job Invitation Service** — state machine + delegating to the apply internal operation + the choose tutor internal operation on accept
- **database** — job invitations table (migration 0035)

## Invitation Status State Machine


The lifecycle diagram describes these state changes:

- Start leads to PENDING.
- PENDING leads to ACCEPTED.
- PENDING leads to DECLINED after tutor declines.
- PENDING leads to WITHDRAWN.
- PENDING leads to EXPIRED after background expiry sweep.


> One PENDING invitation per (job, tutor) is enforced by partial-unique index
> ux jobinv one pending per pair. Re-invite after a terminal state succeeds
> (a new PENDING row is created).

## Diagram — Happy path: invite → accept → choose-tutor → admin assign


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — Tutee withdraws a pending invitation


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — Tutor declines an invitation


Implementation example omitted. This public edition retains the written requirements in this section.


## Diagram — Re-invite after a terminal state


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- **Accept delegates, never matches directly:** the accept operation calls
  the apply internal operation (creates/reactivates the job application) then
  the choose tutor internal operation (creates a CREATED selection request on the tutee's
  behalf). No Match entity is created — the admin-only matching invariant holds.
  Source: Job Invitation Service implementation reference accept method.
- **personal information contract:** Tutor Invitation Card Response is enforced by
  Tutor Invitation Card Response Pii Contract Test; Job Invitation Response by
  Job Invitation Response Pii Contract Test. Neither data transfer object exposes a real name, email,
  phone, MRT station, or exact planning area.
- **status is VARCHAR using 16**, not a PG enum — avoids ALTER TYPE ADD VALUE
  friction for future lifecycle states (migration 0035 design note).
- **Idempotent invite:** submit operation for invite returns **201** on first create and
  **200** on re-invite while a PENDING row already exists (the existing row is
  returned unchanged).
- **Wrong-owner / wrong-recipient → 404**, not 403 (existence not leaked).
  Tutee IDOR on withdraw: non-inviter gets 404. Tutor IDOR on accept/decline:
  non-recipient gets 404.
- **already Applied hint** in Tutor Invitation Card Response lets the frontend show
  "Already applied" on the card without a second round-trip.
- **5 measurement events** are emitted by Job Invitation Service to the analytics
  stream (invite created, accepted, declined, withdrawn, expired).
- Source: Tutee Invitation Controller implementation reference, Tutor Invitation Controller implementation reference,
  Job Invitation Service implementation reference, Api Routes implementation reference, migration 0035-job-invitations implementation reference.

## Failure modes

- 404 — tutee post not found, or not owned by the caller (IDOR on invite).
- 404 — invitation not found, or caller is not the inviter/recipient (IDOR on
  accept/decline/withdraw).
- 422 — tutee's job is not ACTIVE (cannot invite on DRAFT/PENDING/MATCHED/etc.).
- 422 — accept on a non-PENDING invitation.
- 409 — database partial-unique violation if a concurrent request creates a second
  PENDING for the same (job, tutor) pair (rare; application-level idempotency
  should return 200 first).
- 403 — wrong role on controller (TUTEE-only vs TUTOR-only; Security Config +
  @Pre Authorize).

## Related flows

- [38-choose-tutor-selection md](38-choose-tutor-selection.md) — the choose-tutor flow that accept feeds into
- [28-tutor-apply-withdraw md](28-tutor-apply-withdraw.md) — organic apply (invitation accept folds into this)
- [42-interview-timeslot-negotiation md](42-interview-timeslot-negotiation.md) — separate pre-match communication channel
- [39-admin-applicant-view md](39-admin-applicant-view.md) — admin views the applicant list (includes accepted invitees)
