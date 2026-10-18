# Design — Tutee dashboard + invitations (013)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Owned by bff-specialist + software-engineer + frontend-expert + uiux-designer. Grounded 2026-06-22.

## REQ-001 — view/edit own post; approval lifecycle preserved

Reuse the edit wizard and authenticated own-post GET/PUT. The earlier the historical editable set of draft, rejected and active posts
code excerpt described a historical implementation; it did not waive parent requirements. The effective table
follows [product requirements document §4/§10](../PRD.md) and [021 REQ-003](../021-tutee-request-flow/spec.md):

| Post Status | Edit? | Required behavior |
|---|---|---|
| DRAFT | Yes | Save draft; explicit submit starts approval. |
| REJECTED | Yes | Revise; explicit resubmit returns to PENDING, not silent publication. |
| REVISION REQUESTED | Yes | Show the requested revision, allow corrections and explicit resubmit to PENDING. This remains required even if a revise route needs work. |
| PENDING | Yes | Owner corrections are allowed under 021; saving remains/re-enters approval and does not publish. |
| ACTIVE | Yes | Warn before save that the post leaves the public list and returns to PENDING for re-approval. |
| MATCHED | No | View-only; this edit action does not alter locked assignment economics. |
| CLOSED | No | View-only terminal post. |
| EXPIRED | No | View-only terminal post. |

frontend and server share this approved status contract; source constants must implement it, not define exceptions.
Pre-fill the same supported fields as creation, preserve owner authorization and display validation/conflict
errors. Add the revision/resubmit server work if missing; do not label REQ-001 frontend-only without verifying support.
No edit silently bypasses approval. Tests cover all eight statuses, revision reason, draft/revision save versus
explicit submit, PENDING corrections, ACTIVE re-approval warning, and a concurrent transition to MATCHED.

## REQ-002 — venue display

Dashboard implementation reference change from venue Preferences 0 to a representation of all (join / "+N more"); the View/detail
shows the full list. Confirm Own Tutee Post venue Preferences carries the full array (it does — 0 is indexing a
populated array). No backend change.

## REQ-003 — tutee sent-invitations endpoint + durable surface

### Endpoint

Add read operation for tutee invitations (tutee-scoped, the pre authorize operation) to
Tutee Invitation Controller (the existing tutee invitation controller — it already owns POST invite / GET
invite-quota / DELETE withdraw; this is the missing LIST). Paginated, newest-first, ?status is PENDING default,
?status is ALL for history. Add the route constant TUTEE INVITATIONS to Api Routes implementation reference and tutee invitations
to api Routes implementation reference (named, no magic string) — distinct from the existing tutor-side tutor invitations.

### Query key + index — RESOLVED

Job Invitation carries created By (Job Invitation implementation reference, the column operation), the
**inviting tutee user id** ( equals  the post owner, per the Javadoc), NOT the tutor. So the own-only scan is by
the user id operation. Add to Job Invitation Repository (mirroring the existing tutor-side
find By Tutor User Id And Status Order By Created At Desc / find By Tutor User Id Order By Created At Desc, 50-54):


Implementation example omitted. This public edition retains the written requirements in this section.


**Index (DBA task — this is no longer "frontend-only + one endpoint"):** there is no existing index on created by
(the existing invitation indexes are idx jobinv post status tutee post id, status from 0035c and the
ux jobinv one pending per pair partial-unique on tutee post id, tutor user id from 0035b — neither covers a
created by scan). Add a Liquibase changeset for a composite index idx jobinv createdby status (created by,
status)so the own-only filtered list is index-backed no seq-scan as invitation volume grows . Ownerdba.

### Response shape — RESOLVED (reuse the existing personal information-safe data transfer object)

**Reuse the existing Job Invitation Response as-is** (Job Invitation Response implementation reference) — it is already the
tutee-side data transfer object (used by the POST invite + DELETE withdraw on this same controller) and is already personal information-safe and
contract-tested by Job Invitation Response Pii Contract Test. Its fields and the tutee-visibility verdict:

| Field | Tutee may see? | Why |
|---|---|---|
| id | ✅ | the invitation's own id (needed to withdraw) |
| tutee Post Id | ✅ | the tutee's OWN post the invite targets |
| job Subject, job Level | ✅ | anonymised label of the tutee's own job |
| tutor User Id | ✅ | opaque unique identifier only — never a name/email/phone |
| tutor Public Handle | ✅ | the tutor's **public handle ONLY** — never display Name / real name |
| message | ✅ | the tutee's own optional note (text, personal information-contract-tested) |
| status | ✅ | the invite's lifecycle state |
| job Application Id | ✅ | set on ACCEPTED; an opaque id |
| created At, responded At | ✅ | sent / responded timestamps |
| quota | ✅ (nullable) | two ints (limit + remaining); list responses leave it null |

This is the tutee's OWN outbound invite, so the tutor's **public** identity (handle only) plus status/dates/own-job
is the correct disclosure — and it leaks nothing the invite did not already expose. We do NOT reuse the tutor-side
Tutor Invitation Card Response (that carries coarse Area + the tutee's posted budget Min Sgd/budget Max Sgd shaped for
the tutor's reading — not the projection a tutee-outbound list needs). **No new data transfer object and no new personal information surface** — reuse
Job Invitation Response, and extend Job Invitation Response Pii Contract Test to cover the new list path.

### frontend

Replace the session-only invited Post Ids (Tutor Detail implementation reference) with a query against the new endpoint (client
state mirrors server truth, never re-derives it — the cross-cutting wave rule); surface on the dashboard (see the IA
placement below); withdraw operates on the persisted row (invitation Withdraw already exists).

## REQ-004 — outbound interviews on dashboard

frontend-only: the dashboard reads read operation for tutee interviews (my List) and shows the outbound interview requests in
the shared outbound area defined below. The post-create cache invalidation (Tutor Detail implementation reference) is retained.

## Dashboard IA — #208 placement DECIDED (uiux-designer lens)

This is the actual #208 defect ("created → not visible"), so the placement is part of the spec, not deferred. An
endpoint nobody renders does NOT close #208.

**Decision: one dedicated "Your outbound requests" section on /tutee/dashboard**, rendered directly **below** the
existing "My posts" list (Dashboard implementation reference, the existing dashboard section). A single labelled section, NOT
three scattered ones — the tutee's mental model is "things I sent out and am waiting on", and one consolidated
surface answers "what did I do and what's its status?" without hunting across pages.

Inside the section, two clearly-labelled sub-groups (the two outbound types this spec covers), each a compact list:

1. **Invitations sent** — from read operation for tutee invitations (REQ-003). Each row: tutor public handle, the job
   it targets (subject·level), status badge, sent date, and a Withdraw action while PENDING.
2. **Interview requests** — from read operation for tutee interviews (REQ-004). Each row: tutor identity, status badge,
   requested/scheduled date, and a link into the existing /tutee/interviews detail.

(Applications-sent is NOT in this spec's scope — it lives in spec 012's apply surfaces. The section header copy and
structure are designed so an "Applications sent" sub-group can slot in as a third group later without re-architecting
the IA — extensible, but we ship the two this spec owns.)

**Why a dedicated section, not badges/links scattered on existing rows:** the bug is that an outbound request a tutee
created is invisible because nothing on the dashboard references it. Counts-only or a bare link still leaves the
tutee guessing; a labelled section that LISTS the items with their status is the testable acceptance target
("the just-created invitation/interview-request appears here, with its status, and survives a refresh").

**Empty state:** when a sub-group is empty, show a one-line empty hint ("You haven't invited any tutors yet")
rather than hiding the section — so the surface is discoverable before the first action, and the tutee learns the
dashboard is where these live.

uiux-designer owns the final visual treatment (section vs accordion vs tab) but the IA decision — **a single
dedicated outbound-requests area on /tutee/dashboard that lists invitations-sent and interview-requests with
status** — is fixed here as the REQ-004 acceptance target.

## Test / coverage

- backend gate ≥97.5/97.5 — new sent-invitations endpoint + service query (own-only, soft-delete, authz) tested.
- frontend gate ≥97.5/88 — edit pre-fill+PUT, venue display, server-backed Invited state + withdraw, outbound-interview
  surface.
- Live: edit round-trip; 3-venue display; invite→refresh→persists; interview-request→dashboard shows it.
