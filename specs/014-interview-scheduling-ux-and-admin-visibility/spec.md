> **Current-file check, 18 October 2026:** Date display uses Singapore time, but participant input and staff prefill still use the computer’s timezone. The combined no-show and meeting-link update paths also remain incomplete (G04/G05). See the [alignment review](../../docs/current-file-alignment.md). The requirements remain unchanged.

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 014
slug: interview-scheduling-ux-and-admin-visibility
title: Interview scheduling user experience + admin visibility — list-filter gap, auto-fill slot, clipped No-Show, datetime picker, raw-handle display
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED
owner: requirements-engineer   # build owners: frontend-expert / bff-specialist / uiux-designer / software-engineer
created: 2026-06-22
updated: 2026-09-05
---

# Interview scheduling user experience + admin visibility

> **Documentation correction — 2026-09-05:** Separated admin Singapore time local datetime from proposer Coordinated Universal Time instants; terminal decline remains admin follow-up. Current clauses in requirements/design/tasks supersede earlier conflicting directions. Historical HISTORICALLY MARKED IMPLEMENTED labels, dated changelog entries and evidence are retained as history; this correction makes no new runtime, test or deployment claim.


Fixes the interview admin/scheduling experience from the 2026-06-22 review + GitHub #210/#211
(specs/issues/review-2026-06-22/CONSOLIDATED md):

- **#210.1 (S2)** — an interview created→accepted live does NOT appear in the admin interview list (though the
  notification fires). **Root cause FOUND (frontend, not backend):** the admin list endpoint
  Interview Service list Admin Interviews (457-462) returns find All when no status filter is passed — it is
  correct. The bug is the frontend tab map Interview Queue implementation reference: ACTION REQUIRED filters only REQUESTED,
  SCHEDULED only SCHEDULED, HISTORY only COMPLETED,CANCELLED,NO SHOW. The negotiation/pending
  states PROPOSED, COUNTER PROPOSED, PENDING SCHEDULING, TUTOR DECLINED (Interview Status implementation reference)
  appear in **no tab except ALL** — and so does terminal EXPIRED — so an accepted-but-not-yet-scheduled interview
  (PENDING SCHEDULING) is invisible in the action queue. The fix places ALL 10 Interview Status values
  (EXPIRED → history) and drives the map from the enum (set-equality test).
- **#210.2 (S3)** — when scheduling, the admin should auto-fill the tutor-accepted timeslot rather than re-typing it.
- **#210.3 (S3)** — the "Record No Show" button is clipped / not visible next to "Mark Complete" (user interface overflow).
- **#211 (S3)** — the date/time picker is unpleasant across all roles (the schedule datetime control).
- **raw-handle (S3)** — the tutor shows as a raw handle tutor wei jie tan on Interview/Applicants screens (vs a
  proper display name on Matches) — reads like a leaked id.

This is primarily a frontend interview-lane spec; #210.2 may touch the admin schedule contract. It shares
Interview Service/Interview Queue with no other review spec (T-08's seeder fix is spec 017).

- **Requirements:** [requirements md](requirements.md) · **Design:** [design md](design.md) · **Tasks:** [tasks md](tasks.md)
- **Evidence:** the private evidence directory for the requirement

## Historical delivery/review status

The record below describes the earlier delivery cycle; it is not approval or verification of the 2026-09-05 correction.

DRAFT — author does not self-approve. Boss convenes a scoped team-review (frontend-expert, uiux-designer,
bff-specialist, software-engineer) + audit; flips to APPROVED when sign-off boxes are checked.

## Review & Sign-Off
- [ ] Author — requirements (requirements-engineer)
- [ ] Counterpart — admin-queue tab/status coverage (frontend-expert)
- [ ] Counterpart — datetime picker + auto-fill + No-Show layout user experience (uiux-designer)
- [ ] Counterpart — accepted-slot pre-fill (frontend; data already on Admin Interview Response) (frontend-expert)
- [ ] Counterpart — display-name render (frontend; data already on Admin Interview Response) (frontend-expert)
- [ ] Boss (final sign-off → flips status to APPROVED)

## Changelog

- 2026-09-05 — Documentation alignment correction: Separated admin Singapore time local datetime from proposer Coordinated Universal Time instants; terminal decline remains admin follow-up. No application code or runtime verification in this change. Earlier entries below are historical.
- 1.2 (2026-06-22) **Wave-2 spec-review APPROVED-condition folded (WAVE2-SPEC-REVIEW md §014/REQ-004).** REQ-004
  (datetime picker) pinned a **BLOCKING backend-wire-format-unchanged test**: the picker is frontend-only, so the submitted
  value MUST stay the Local Date Time-shaped ISO-8601-local string (no offset, no Z) the backend
  Schedule Interview Request scheduled At (java time.Local Date Time) already parses — a picker swap cannot silently
  re-shape the payload. No backend change. status stays APPROVED.
- 1.1 (2026-06-22) Wave-1 spec-review SEND-BACK resolved (WAVE1-SPEC-REVIEW md §014). (1) REQ-001 tab union fixed to
  cover ALL 10 Interview Status values — EXPIRED (omitted in v1.0) placed in the history tab; full tab→enum map
  recorded in design md. (2) The union equals enum test made **enum-driven set-equality** (compute the enum at test time;
  fail until every value is placed) — not a frozen snapshot. (3) Dropped the 2 phantom backend tasks: agreed Timeslot Id+
  timeslots (REQ-002) and tutor Display Name (REQ-005) already exist on Admin Interview Response implementation reference,
  so REQ-002/005 are frontend-only and the "confirm/add if absent" backend task is removed — this is now an frontend-only spec. status
  stays DRAFT pending Boss re-gate.
- 1.0 (2026-06-22) initial draft — ADDED REQ-001 (admin queue tabs cover ALL live statuses incl. PENDING SCHEDULING/
  PROPOSED/COUNTER PROPOSED/TUTOR DECLINED — frontend tab-map fix), REQ-002 (auto-fill the accepted slot when scheduling),
  REQ-003 (Record-No-Show button not clipped), REQ-004 (better datetime picker), REQ-005 (show tutor display name,
  not raw handle, on Interview/Applicants). Grounded against Interview Queue implementation reference / Interview Service /
  Interview Status. status DRAFT pending scoped team-review.
