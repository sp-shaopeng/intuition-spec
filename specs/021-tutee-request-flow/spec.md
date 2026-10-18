> **Current-file check, 18 October 2026:** Pending initiation/editing is present in inspected paths, but two website messages still promise contact after approval instead of explaining assignment and activation (G06). See the [alignment review](../../docs/current-file-alignment.md). The requirements remain unchanged.

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 021-tutee-request-flow
status: HISTORICALLY MARKED IMPLEMENTED
version: 0.2
owners: [frontend-expert, bff-specialist, uiux-designer]
source: specs/issues/review-2026-06-22-dev-personas/CONSOLIDATED md
---

# Spec 021 — Tutee request flow

TUT-1/3/4 from the 2026-06-22 review. Founder decisions: allow invite/interview on PENDING; PENDING editable.
Standing rule: regression test per REQ. Soft-delete only.

## Changelog
- 0.2 (2026-09-05) — Documentation alignment correction authorized by the user: retain approved PENDING initiation/editing; correct contact boundary and specify rejection cleanup; acceptance-before-approval decision remains pending. No application implementation or deployment is claimed.
- 0.1 (2026-06-22) — DRAFT. Team-review confirmed TUT-1 is a real backend change (gates hard-block non-ACTIVE).

## Requirements

### REQ-001 — TUT-1 allow invite/interview on a PENDING post (S2, FOUNDER-CONFIRMED)
A tutee with a PENDING (awaiting-approval) request can invite a tutor / request an interview. Moderation approval alone does not release contact. Contact remains limited to the assigned participants under the [M18 personal information contract](../PRD.md) and [INT-125 reveal decision](../architecture/int-125-batch-closeout-spec.md): the frontend offers reveal on ACTIVE/COMPLETED matches; match-PENDING remains awaiting confirmation even where the participant-authorized application programming interface permits it.
- **Given** a tutee whose only request is PENDING, **when** they click Invite / Request Interview for a tutor, **then** initiation is allowed and no contact is revealed.
- **Given** that post is approved but no assignment exists, **then** invite/interview views still reveal no phone/email. Approval must not create a match or bypass participant authorization.
- **Guard:** **Given** the PENDING post has outstanding invitations/interviews, **when** an admin rejects it, **then** those pending records are logically closed/voided in the same rejection outcome, remain available in the relevant history, and subsequent accept/propose actions cannot reactivate them or reveal contact. Repeating cleanup is idempotent. Assert the final state of BOTH invitation and interview records and each relevant recipient/owner view; no physical delete.
- Design: widen initiation guards in the backend invitation/interview services and frontend tutor-detail/invite controls to accept PENDING. Retain the separate contact-reveal service and match-state user interface gates. Use existing domain terminal statuses for rejection cleanup and trace them in the implementation; do not introduce an invitation timeout.
- Copy: "Your request is awaiting approval. You can invite now; contact is available after assignment and confirmation."
- **Regression tests:** PENDING invitation AND interview initiation each succeed and appear in their outbound lists without personal information; approval without assignment still has no contact; rejection closes BOTH pending record types and prevents follow-up actions/personal information; authorized ACTIVE/COMPLETED match views show reveal while match-PENDING stays hidden.
- **Pending decision:** whether a tutor may ACCEPT an invitation before post approval is awaiting the user's answer. Initiation permission is settled; it does not itself amend the older ACTIVE-only acceptance contract. Until the decision is recorded, retain that acceptance guard and do not claim pre-approval acceptance is supported.

### REQ-002 — TUT-3 show the schedule in request-details (S3)
The request-details dialog must show the availability/schedule the tutee entered.
- **Given** a posted request with availability, **when** the tutee opens its details dialog, **then** the schedule
  grid is shown (alongside budget/venue/notes).
- Design: Dashboard implementation reference — add a schedule block (reuse the availability-grid display util; data already
  on the post data transfer object).
- **Regression test (component):** the details dialog renders the schedule for a post with availability.

### REQ-003 — TUT-4 edit a PENDING post (S3)
A PENDING post is editable; editing re-enters approval (same as ACTIVE-edit).
- **Given** a PENDING post, **when** the tutee clicks Edit and saves, **then** the post updates and re-enters the
  approval queue (no destructive close→repost).
- Design: add PENDING to EDITABLE POST STATUSES (Dashboard implementation reference) **and** backend Tutee Post Service implementation reference
  EDITABLEbackend blocks PENDING at); confirm edit re-enters approval.
- **Regression test (e2e):** edit a PENDING post → saved + status returns to PENDING (re-approval); not closed.

## Design notes
frontend: Tutor Detail implementation reference, Invite Tutor Modal implementation reference, Dashboard implementation reference,
Create Request implementation reference (edit). backend: Job Invitation Service implementation reference, Interview Service implementation reference,
Tutee Post Service implementation reference. Build **021 backend first** (gate widen), then frontend. Depends on no other spec.

## Tasks
- [ ] REQ-001 widen PENDING initiation only + assigned-participant/match-state contact gates + rejection cascade + all listed regressions; keep pre-approval acceptance decision pending
- [ ] REQ-002 schedule in details dialog + component test
- [ ] REQ-003 PENDING editable (frontend+backend EDITABLE sets) + re-approval + e2e
- [ ] gates backend ≥97.5/97.5, frontend ≥97.5/88; live convergence dev→virtual machine

## Review & Sign-Off
- [ ] frontend-expert (Sam)
- [ ] bff-specialist (Riley) — gate widen contract
- [ ] uiux-designer (Jamie) — copy/flow
- [ ] security-expert (Casey) — contact stays gated on PENDING
- [ ] qa-engineer (Reese)
- [ ] technical-writer (Chris)
- [ ] Boss (Rex)
