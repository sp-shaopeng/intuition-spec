# INT-062 — Withdraw/close a live job → cascade-close its applications + invitations to a visible "job no longer available" status

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** The known close-state, fan-out and cleanup decisions are folded; invitation acceptance while the post is PENDING remains separate from initiation eligibility.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Status:** spec v2 (Boss SEND BACK→corrected; needs /team-review → Boss before build) · **Date:** 2026-06-04
**Owners:** software-engineer (lead) + dba + bff-specialist + frontend-expert + uiux-designer
**Tracker:** INT-062 (S2) · **Founder decisions:** (1) a tutee/admin **can withdraw a LIVE job** that already has
applications/invitations; (2) distinct new CLOSED statuses.

## Problem / founder rule
Once a tutor↔job **match** exists, OR a tutee/admin **withdraws (closes) a live job**, **every** application to that
job AND every job invitation/recommendation must move to a terminal **"job no longer available"** status — staying
**visible to everyone** (applicant, invited tutor, admin), never hidden/removed (transparency; soft-delete-only).

## Current state (verified in code) — and the two corrections from the Boss SEND BACK
- **Match already cascades** (Admin Match Service create Match 225/232): apps→SUPERSEDED, invitations→EXPIRED
  (set-based, recipients captured pre-update records, INT-016-safe). frontend shows "Superseded"/"Expired".
- **CORRECTION 1 — there is NO "withdraw a live job" path today.** Tutee Post Service soft Delete (153) only allows
  **DRAFT or REJECTED** posts → CLOSED. Those have **no applications/invitations**, so a cascade there is a no-op.
  The founder wants to close a **live (ACTIVE/PENDING)** job → that path **does not exist and must be built.**
- **CORRECTION 2 — invitations have NO timeout.** Job Invitation Status EXPIRED already means "job closed/assigned"
  (only expire Pending For Job sets it; no scheduled timeout). So for the MATCH case the clarity fix is an **frontend relabel**,
  not a new enum. A new CLOSED is justified ONLY by the new withdraw path (below).
- The recorded application states were APPLIED, WITHDRAWN, SHORTLISTED, REJECTED, MATCHED and SUPERSEDED. Invitation states were PENDING, ACCEPTED, DECLINED, WITHDRAWN and EXPIRED. Post states include ACTIVE, PENDING,
  CLOSED, MATCHED, DRAFT, REJECTED.

## Design

### A. New action — withdraw/close a LIVE job (the real feature)
- **Endpoint:** submit operation for tutee posts id close (tutee, own post) + admin equivalent (submit operation for admin posts id close)
  — admin-mediated marketplace, so admin can close any. (Reuse Api Routes; no magic strings.)
- **Allowed states:** a **live** post — ACTIVE or PENDING (approval-pending). **Reject** MATCHED (a match already
  closed it), already-CLOSED, DRAFT/REJECTED (use the existing delete). 409 on invalid state; idempotent (re-close = no-op/409).
- **Effect:** post status CLOSED with the closure time recorded (logical close — soft-delete-only, never row removal). Then cascade.

### B. Cascade (set-based, idempotent, atomic notification intent + metrics)
- Open applications (OPEN STATUSES) → **Job Application Status CLOSED** (new) via the close open applications operation.
- PENDING invitations → **Job Invitation Status CLOSED** (new) via the close pending invitations operation.
- Capture affected recipients before the bulk database update clears the loaded entity state (INT-016 hazard).
- Persist in-app notifications and required outbox intents in the same transaction as closure; external delivery and analytics run after commit. Neutral copy to each affected applicant/invitee: "a job you applied to / were invited to is no longer available"; metrics applications closed total / invitations closed total; analytics
  events application closed / invitation closed.
- **Match path unchanged** (apps→SUPERSEDED, invites→EXPIRED) — no new enum needed there; only the frontend label changes.

### C. Statuses (distinct, cause-bearing)
- App: SUPERSEDED = a winner was matched; **CLOSED** = job withdrawn/no longer available.
- Invitation: EXPIRED = job filled by a match; **CLOSED** = job withdrawn. (Both, plus SUPERSEDED, render the same
  frontend badge "Job no longer available"; the enum keeps the cause for analytics.)

### D. Migrations (dba; ALTER-TYPE rule) — now justified
0038 ALTER TYPE job application status ADD VALUE CLOSED; 0039 ALTER TYPE job invitation status ADD VALUE CLOSED
— **each run In Transaction is no, applied before code references the label** (0033f lesson). 0032 reserved (M19); head 0037.

### E. frontend
- **Tutee:** a **"Close / Withdraw job"** control on the tutee's own live job (My Posts / job detail), with a confirm
  dialog ("This closes the job and notifies the N tutors who applied/were invited; it can't be reopened"). Admin: a
  parallel close control in the admin post/applicants view.
- **Transparency (all parties, visible + listed, never filtered):** a CLOSED badge → "Job no longer available" on
  tutor Applications implementation reference, tutor Invitations implementation reference, tutee Applicants implementation reference, admin; also relabel the match-case
  SUPERSEDED/EXPIRED to the same human phrase. Disable apply/withdraw/accept CTAs for closed rows. Text badge, a11y.

## Open decisions (for team-review)
- **Resolved by the allowed-state rule above:** closing ACTIVE or PENDING jobs is supported; do not reopen the old ACTIVE-only alternative.
- **Current cleanup obligation (baseline C06/C07):** rejection/removal/closure cleans up associated open intake, including pending arrangements; apply the defined interview terminal/cancellation hooks and reminder cancellation. The old option to leave live intake behind is not current. Invitation acceptance while the parent post is PENDING remains O01, separate from closure cleanup.
- **Resolved by acceptance criterion4:** notify every affected applicant/invitee; preserve batched/idempotent delivery and the applicable durable outbox contract.

## Acceptance criteria
1. The close action on an ACTIVE or PENDING job → post CLOSED; its open applications → CLOSED, PENDING
   invitations → CLOSED; 409 on MATCHED/CLOSED/DRAFT/REJECTED; idempotent.
2. Match path still yields apps→SUPERSEDED, invites→EXPIRED (no regression).
3. Closed apps/invitations stay **visible/listed** to applicant, invited tutor, admin with a "Job no longer available"
   badge; their CTAs disabled; nothing hidden/removed.
4. Affected users have one atomic in-app/outbox intent, with external delivery after commit; cascade is set-based (constant query count, no read-loop), idempotent.
5. Authz: tutee closes only own post; admin any; others 403.
6. Gates green (backend ≥97.5/97.5 migrate-then-boot; frontend ≥97.5/88).

## Rollout / delivery (approved full-delivery plan)
Spec → /team-review→Boss → backend (migrations 0038/0039 → enums → repo close-methods → new close endpoint+service+cascade
→ tests) → deploy+reseed dev → frontend (close control + CLOSED badges + relabels) → docs (backend-api md endpoint+enums,
frontend-routes md) → postman (collection: add examples for the new owner and administrator close actions + postman md note) →
live smoke (apply→withdraw→applicant sees "Job no longer available", still listed; invite→withdraw→invited tutor sees
it) → e2e → push. Each gate re-verified by orchestrator. Prod stays founder-gated.
