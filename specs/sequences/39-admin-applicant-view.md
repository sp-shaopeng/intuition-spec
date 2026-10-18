# 39 — Admin Applicant View

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Trigger

An admin opens a tutee job to see who has applied — the applicant list that feeds the
M4 matching / M18 assign workflow.

## Participants

- **Admin**
- **frontend** admin job/applicants view
- **Admin Application Controller** — read operation for admin posts tutee Post Id applications
  (the pre authorize operation)
- **Job Application Service list Applicants For Job**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Admin Application Controller implementation reference; path constant
  the applicant list for the specified post.
- **2026-06-03 (INT-033 — FIXED):** the admin "View applicants" link in Selection Request Queue had
  pointed at the **TUTEE** route the applicant list for the specified requests (under Tutee Layout, which redirects
  any non-TUTEE to /) — so an admin clicking it bounced to the landing page. Fixed by adding an
  admin-namespaced frontend route /admin/posts/:job Id/applicants (Job Applicants implementation reference, under
  Admin Layout) that calls this endpoint and repointing the link. The endpoint itself was unchanged.
- ADMIN-only (the OPERATOR role does not exist yet — see status-briefing md §3).
- Distinct from the **parent** personal information-free applicant cards (flow 38): this admin view backs the
  matching workflow. Distinct from the **tutor's own** applications list (flow 28).

## Failure modes

- 403 — non-ADMIN caller.
- Empty page when the job has no applicants.

## Related flows

- [28-tutor-apply-withdraw md](28-tutor-apply-withdraw.md) — how applications are created
- [38-choose-tutor-selection md](38-choose-tutor-selection.md) — the parent-driven counterpart
- [09-admin-match-creation md](09-admin-match-creation.md) — the match this view feeds
