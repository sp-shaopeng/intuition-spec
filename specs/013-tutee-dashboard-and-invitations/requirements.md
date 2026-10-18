# Requirements — Tutee dashboard + invitations (013)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> EARS + Gherkin per atomic requirement, grounded 2026-06-22. Source: CONSOLIDATED md (#207.1, #207.2, #208,
> sent-invitations finding).

## Scope note (verified)

- Edit: replace operation for tutee posts id exists (Tutee Post Controller implementation reference); GET detail post Detail exists. frontend
  has no general edit/view action (only Resubmit for REJECTED, Dashboard implementation reference).
- Venue: Dashboard implementation reference renders venue Preferences 0 only → truncation by design, not data loss.
- Invitations: create (post Invite) + withdraw exist; Job Invitation persists tutee Post Id, tutor User Id,
  status (soft-delete). **No tutee GET for sent invitations.** Tutor Detail implementation reference invited Post Ids is
  session-only use State.
- Interviews: submit operation for interviews (create) + read operation for tutee interviews (my List) exist and work
  (sequential re-run confirmed tutee interview flow). The dashboard just doesn't surface outbound interviews.

---

## REQ-001 — Tutee can view and edit a post from the dashboard

- **Type:** functional · **Priority:** Must · **Severity:** S2 · **Source:** #207.1
- **Rationale:** A tutee who made a typo or wants to adjust budget/venue/subject must be able to fix it without
  recreating the post. The backend supports it; only the frontend entry point is missing.
- **Behaviour (EARS):** WHEN a tutee views their own dashboard posts, offer View for each post and Edit for
  DRAFT, REJECTED, REVISION REQUESTED, PENDING and ACTIVE. MATCHED, CLOSED and EXPIRED are
  View-only. This contract follows the product requirements document revision requirement and 021's later PENDING-edit amendment;
  historical code-state observations do not remove required editing.
  Pre-fill from the own-post detail and save through the authorized update/revise path. DRAFT/revision saves do
  not silently publish; explicit submit/resubmit returns to PENDING. PENDING edits remain under approval.
  ACTIVE edits warn that saving returns the post to PENDING for admin re-approval. Surface server validation or
  state conflicts. frontend and backend must use the same approved status contract; an unavailable revise route is required
  follow-up work, not a product exception.
- **Owner:** frontend-expert · software-engineer · bff-specialist


- **Acceptance criteria (Gherkin):**


- Scenario: Edit a post from the dashboard.
- Given a tutee with an editable post.
- When they choose Edit.
- Then the form pre-fills from the post detail.
- And saving issues PUT tutee posts id interface operation and shows success.
- Scenario: View a post's full detail.
- Given a tutee post.
- When they choose View.
- Then all fields (including all venues) are shown.
- Scenario: Edit matches the approved five editable states.
- Given posts across all Post Status values.
- Then Edit is offered for DRAFT, REJECTED, REVISION REQUESTED, PENDING and ACTIVE.
- And MATCHED, CLOSED and EXPIRED are View-only.
- And saving revisions does not publish without explicit resubmission and approval.
- Scenario: Editing an ACTIVE post warns it re-enters approval.
- Given an ACTIVE post.
- When the tutee opens Edit.
- Then a notice states that saving re-submits the post for admin re-approval (it leaves ACTIVE).
- Scenario: A 409 INVALID STATE on edit is surfaced (guard).
- Given the backend rejects an edit with 409 INVALID STATE.
- Then the rejection reason is surfaced to the tutee (never silent).


- **Required evidence:** RTL test of edit pre-fill + PUT; live Playwright edit round-trip. Under REQ-001 implementation reference.
- **Dependencies:** product requirements document revision/edit requirements; 021 REQ-003 permits PENDING edits. Verify and complete server revise/resubmit support where required.

## REQ-002 — Dashboard shows all selected venues (not just the first)

- **Type:** functional · **Priority:** Should · **Severity:** S3 · **Source:** #207.2
- **Rationale:** Showing only venue Preferences 0 makes a tutee think their other selections were lost.
- **Behaviour (EARS):** WHEN the dashboard renders a post row, the system SHALL represent ALL selected venue
  preferences (e g. a compact list / "+N more" / full list on the detail view), not only the first. The persisted
  data SHALL be confirmed intact (all venues round-trip through create → list → detail).
- **Owner:** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: All venues are represented.
- Given a post created with 3 venue preferences.
- When the dashboard list and the detail view render.
- And none are silently dropped.


- **Required evidence:** RTL test (3 venues → all represented); live capture. Under REQ-002 implementation reference.
- **Dependencies:** none.

## REQ-003 — Tutee sent-invitations: persistent read endpoint + dashboard surface

- **Type:** functional · **Priority:** Must · **Severity:** S2/S3 · **Source:** #208 + sent-invitations finding
- **Rationale:** Today a sent invitation has no tutee-visible record beyond a session-only pill. A tutee can't see
  whom they invited, the status, or withdraw after a refresh. The data already persists; add the read path.
- **Behaviour (EARS):** The system SHALL provide a tutee-scoped read endpoint
  read operation for tutee invitations (route constant TUTEE INVITATIONS / tutee invitations) listing the invitations
  the authenticated tutee SENT, scoped by Job Invitation created By caller (the inviting tutee, Job Invitation implementation reference),
  paginated newest-first, default ?status is PENDING, ?status is ALL for history, EXCLUDING withdrawn/soft-deleted by
  default. **Response = the existing personal information-safe Job Invitation Response reused as-is** (fields enumerated in design md;
  tutor's **public handle only**, never display Name/real-name; the tutee's own post + own note + status + dates +
  opaque ids). The query SHALL be index-backed by a new idx jobinv createdby status created by, status changeset
  (DBA). WHEN the tutee dashboard renders, the system SHALL surface sent invitations as durable server-backed state
  (replacing the session-only invited Post Ids), so the "Invited" indication survives a refresh and a withdraw action
  operates on the persisted row.
- **Owner:** bff-specialist (contract + personal information; reuse Job Invitation Response) · software-engineer (controller LIST +
  service query, soft-delete-aware) · dba (the created by index changeset) · frontend-expert (surface)
- **Acceptance criteria (Gherkin):**


- Scenario: Sent invitations persist across refresh.
- Given a tutee who invited a tutor to a job.
- When they reload the dashboard / tutor page.
- Then the invitation still shows as sent (from the server, not session state).
- Scenario: Sent-invitations list returns the tutee's own invitations.
- Given GET tutee invitations interface operation as the inviting tutee.
- Then it returns that tutee's PENDING invitations (and history with a status filter).
- And it excludes withdrawn/soft-deleted by default.
- Scenario: A tutee cannot read another tutee's sent invitations (authz guard).
- Given tutee A's token.
- When A lists sent invitations.
- Then only A's invitations are returned (never B's).


- **Required evidence:** backend test (own-only, soft-delete-aware, authz); RTL test (server-backed Invited state +
  withdraw); live capture invite → refresh → still shown. Under REQ-003 implementation reference.
- **Dependencies:** INT-044 (invitation model).

## REQ-004 — Outbound interview requests surface on the tutee dashboard

- **Type:** functional · **Priority:** Should · **Severity:** S2 (part of #208) · **Source:** #208
- **Rationale:** A tutee who requests an interview from a tutor's page expects to see that "active request" on their
  dashboard. The interview persists (/tutee/interviews) but the dashboard doesn't reference it, so it "looks lost".
- **Behaviour (EARS):** WHEN the tutee dashboard renders, the system SHALL surface the tutee's outbound interview
  requests **as a listed sub-group inside the dedicated "Your outbound requests" section** on /tutee/dashboard
  (placement DECIDED in design md — a single dedicated section below "My posts" containing two sub-groups:
  Invitations sent + Interview requests), each row showing the tutor identity, status, and date with a link into
  /tutee/interviews, so a just-created request is discoverable from the dashboard, not only the separate Interviews
  page. The created-then-navigated flow SHALL leave the request visible (the existing interview Keys cache
  invalidation, Tutor Detail implementation reference, is retained).
- **Owner:** frontend-expert · uiux-designer (IA RESOLVED — owns final visual treatment of the decided section)
- **Acceptance criteria (Gherkin):**


- Scenario: A requested interview is discoverable from the dashboard.
- Given a tutee who just requested an interview from a tutor's page.
- When they view the dashboard.
- Then the request appears as a row in Interview requests under Your outbound requests below My posts.
- And the same persisted row remains after refresh.
- And a count or link without listed requests does not satisfy this requirement.


- **Required evidence:** RTL test (dashboard surfaces outbound interviews); live capture request → dashboard shows
  it. Under REQ-004 implementation reference.
- **Dependencies:** INT-039 (interviews), REQ-003 (shared "outbound requests" dashboard area).

---

## Traceability summary

| REQ | One-line | Type | Priority | Severity | Owner |
|---|---|---|---|---|---|
| REQ-001 | frontend edit/view post action (backend PUT/GET already exist) | functional | Must | S2 | frontend-expert · bff-specialist |
| REQ-002 | Dashboard shows all venues, not just 0 | functional | Should | S3 | frontend-expert |
| REQ-003 | Tutee sent-invitations GET endpoint + durable dashboard surface | functional | Must | S2/S3 | bff-specialist · software-engineer · frontend-expert |
| REQ-004 | Outbound interview requests surfaced on the tutee dashboard | functional | Should | S2 | frontend-expert · uiux-designer |

## Resolved decisions (were open questions — closed at design, 2026-06-22)

1. **REQ-001 editable statuses — corrected 2026-09-05:** the earlier code-derived three-state list is
   historical. product requirements document requires REVISION REQUESTED editing, and 021 permits PENDING editing. Effective five-state
   list and approval-preserving transitions appear in REQ-001/design md; missing server support is work to do.

2. **REQ-003/004 dashboard IA — RESOLVED:** one dedicated "Your outbound requests" section on /tutee/dashboard
   (below "My posts"), containing two listed sub-groups — Invitations sent + Interview requests — each with status,
   date, and an action/link. Designed to extend with an "Applications sent" group later (spec 012's surfaces) without
   re-architecting. Full IA in design md "Dashboard IA — #208 placement DECIDED".
3. **REQ-003 response contract — RESOLVED:** reuse the existing personal information-safe Job Invitation Response (public handle only,
   own-job, own-note, status, dates, opaque ids); query by Job Invitation created By; new
   idx jobinv createdby status index (DBA). Field-by-field visibility table in design md REQ-003.

No open founder/Boss questions remain for this spec.
