# M15 — Applications (Tutor → Job "Apply" / Expression of Interest)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The thin-slice listing prerequisite/admin-exclusive applicant model is superseded by [M18](../../PRD.md) and corrected below. [INT-093](../../architecture/int-093-batch-state-machine-integrity-spec.md) defines the current expanded application states; the historical APPLIED/WITHDRAWN-only implementation is not the complete current requirement.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** DEPLOYED (migration 0018) — backend + frontend shipped and team-reviewed (implementation
review + Boss, 2026-05-29). The review returned SEND BACK; all hard blockers were fixed
(scope-creep notification cut, re-apply ratified as reactivate-in-place, the private implementation folder
contract aligned). See [ADR-013](../../ROADMAP.md) and the
[M13/M14/M15 review record](../../architecture/reviews/M13-M14-M15-team-review.md).
The applicant count is **admin-only** (admins see applicants on a tutee job); it is
**not** shown on the public M14a showcase or any counterparty surface.
**Author:** Solution Architect (draft)
**Date:** 2026-05-29
**Driver:** A real supply→demand signal — let tutors express interest in a tutee job.
Today matching is fully admin-driven; tutors cannot express interest in a job. M15
gives admins a true count of distinct tutors whose application to that job is APPLIED.
**Note:** the interview-signal "proxy" count was **CUT** (dishonest); and the public
[M14a showcase](../landing-showcase/spec.md) ultimately ships with **no applicant
count** at all — the count from M15 is **admin-only**, not a public landing metric.

---

## 1. Concept

A **tutor applies to a tutee job** (an ACTIVE tutee post). Each application is a
durable record. This creates the "how many applied" signal and gives admins a
ranked pool to match from, instead of cold admin matching.

## 2. Data model — job applications


Implementation example omitted. This public edition retains the written requirements in this section.


**DBA-mandated constraints/indexes (follow migration 0009 precedent):**
- **Partial unique:** unique tutee post id, tutor id when status WITHDRAWN
  — one *active* application per tutor per job, but **withdraw → re-apply must
  succeed** (a plain unique would 500 on re-apply).
- **Count index:** the job applications operation
  — M14 reads the count across ~20 cards; without it that's 20 scans/page.
- **List index:** idx job applications tutor ON tutor id, created at DESC.
- Index tutor post id when tutor post id has a value.
- status is a **named PG enum** job application status (every other status
  column in this codebase is a named enum — keep the pattern).

The applicant count for a job is the number of distinct tutors with an APPLIED application to that specific job.
This — and **only** this — is what M14's job card displays. There is no proxy.

## 3. Endpoints (thin slice)

| Method | Path | Role | Purpose |
|---|---|---|---|
| POST | tutor applications interface operation | TUTOR | Apply to a tutee job the tuition-request identifier, optional tutor-listing identifier and optional message (idempotent per pair) |
| DELETE | tutor applications id interface operation | TUTOR | Withdraw own application |
| GET | tutor applications interface operation | TUTOR | List own applications (paginated) |
| GET | admin posts tutee Post Id applications interface operation | ADMIN | View applicants for a job (feeds matching) |

Guards: only ACTIVE tutee posts accept applications; applying tutor must hold an
ACTIVE tutor post (mirrors the existing browse-requests guard). Public showcase
exposes only the **count**, never applicant identities.

> **OPERATOR note:** the admin applicant view is gated to ADMIN **only** in the
> thin slice. The OPERATOR role does not exist yet (User Role =
> TUTEE/TUTOR/ADMIN — verified by the M13/M14/M15 review). When M13 introduces
> OPERATOR, this endpoint will additively accept it; no contract change is
> needed today.

## 4. Interactions

- **Matching (M4):** admin can create a match from an applicant — applications
  give the admin a real shortlist. On match, related applications → MATCHED/closed.
- **Notifications (M9):** notify the tutee (new applicant) and tutor (status
  change). Honour existing notification patterns.
- **Landing showcase (M14):** job card applicant Count.

## 5. Resolved decisions

Resolved during finalisation on 2026-05-29 (team review + Boss). Each item below
was an open question in the draft spec; the answers are now binding for the
thin-slice implementation.

1. **Must a tutor hold an ACTIVE tutor post to apply? No (M18 founder amendment).** Any authenticated tutor
   with a complete canonical profile may apply to an ACTIVE job without publishing a listing. Optional
   tutor Post Id does not become a hidden eligibility requirement. Reuse 012's server completeness predicate.
2. **Who sees applicant details?** Admins use the admin endpoint; the owner of the target job also receives the
   M18 personal information-free applicant projection (public handle/profile facts/status, no contact). Anonymous visitors do not
   receive applicant identities. Do not confuse the private parent list with public aggregate-count policy.

3. **Status lifecycle ownership.** The thin slice ships **only the
   APPLIED / WITHDRAWN** transitions, both owned by the tutor (apply /
   withdraw their own application). SHORTLISTED, REJECTED, and MATCHED
   exist in the named enum so the column is forward-compatible, but **no endpoint
   sets them yet**. Admin-driven SHORTLISTED/REJECTED and the M4 match-close
   transition to MATCHED are deferred to a later slice. Duplicate apply is
   **idempotent** (returns the existing active application); **withdraw → re-apply
   succeeds** by design (see the partial unique constraint in §2).

### Deferred (not in the thin slice)

- Admin-set SHORTLISTED / REJECTED transitions and their notifications.
- M4 matching closing related applications to MATCHED on match creation.
- Surfacing applicant Count — added by **M14** once this module ships
  (no count is displayed anywhere until then).

## 6. Sequencing

Concept approved → finalise spec + team review → Liquibase job applications →
apply/withdraw/list endpoints + tutor user interface → wire applicant Count into M14 →
admin applicant view feeding M4 matching → team review → ship.
