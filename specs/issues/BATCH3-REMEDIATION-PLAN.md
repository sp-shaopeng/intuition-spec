# Batch 3 — Remediation Plan (Hunter findings → team review)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-06-02. **Author:** Hunter (bug-founder) findings, triaged by the team (the technical lead/Rex).
**Scope:** pre-existing defects surfaced when the full backend regression-lab suite + frontend suite were run
green-to-completion for the first time in a while (the suite had been globally red from INT-018 +
INT-019, both now fixed). None were introduced by the INT-001…014 batch.

## How these were found
The reseed + rebuild let the regression-lab ITs (which exercise the **live stack**) run end to end.
Two infra defects (INT-018 seed-password drift, INT-019 enum-literal cast) were blocking the whole
suite; fixing them dropped failures 16 → 5 and exposed three genuine product/persistence bugs plus
two test-infra issues.

## Verdict (Rex)
- **FIX-NOW, already done & verified:** INT-018, INT-019 (unblocked the suite; low-risk, isolated).
- **PLAN, do NOT blind-fix at 2am:** INT-015, INT-016, INT-017 — core admin/lifecycle flows; each
  needs a scoped change + a regression test + re-run against the live stack. Risk of destabilising
  matching/lifecycle is real; they go through normal review.
- **TEST-INFRA, low severity:** INT-020 (runner deadlock), INT-021 (test/component drift).

## Prioritised plan

### P1 — INT-016 (S1): Lazy Initialization Exception on match-cancel & contact-reveal
- **Why first:** S1, two core flows 500 (admin cancel, contact reveal).
- **Fix:** within the cancel / contact-reveal transactions, fetch-join the Tutor Post (and any
  Tutor) associations the response/notification path reads, or map to a data transfer object before the tx closes.
  open-in-view is no, so audit every the get tutor operation deref on these paths.
- **Verify:** Admin Matching IT match create activate cancel lifecycle,
  cancel Match cascades Commission To Cancelled, Assignment Lifecycle IT contact Reveal four States → green.
- **Owner:** software-engineer + dba review.

### P2 — INT-015 (S2): owner cannot delete a REJECTED post (terminal-status trigger)
- **Fix:** for a REJECTED (terminal) post, hard-delete the row (no deleted at column exists) rather
  than moving it to CLOSED; or relax the database trigger to permit REJECTED→CLOSED as the logical-delete
  sentinel. Check FK fan-out (applications/interviews) before choosing hard-delete.
- **Verify:** Post Lifecycle IT delete Rejected Post by Owner succeeds → 200/204.
- **Owner:** dba (trigger) + software-engineer (Tutor Post Service soft Delete).

### P3 — INT-017 (S2): seeded tutor blocked from applying (profile-incomplete 422)
- **Decide (product):** is a freshly-onboarded real tutor expected to complete their profile before
  applying? If yes, the IT must complete the profile as a setup step; if the *sample* tutors should be
  apply-ready, Dev Data Seeder must populate commitment + rate min/max + planning area + qualifications.
- **Verify:** Applications IT tutor apply To Job then Appears then Withdraw → 200.
- **Owner:** product-designer (decision) + data/seed owner.

### P4 — INT-021 (S3): Applicants choose-flow test drift
- **Fix:** scope the queries (the within operation / the get all by role operation) — BUT first confirm the page
  isn't *duplicating* the "assigned" banner (a real render bug would also produce "found multiple").
- **Owner:** qa-engineer.

### P5 — INT-020 (S3): host vitest full-suite deadlock
- **Note:** subsets + the Docker the documented operational check pass; this is a host tinypool/worker stall at ~88
  files, not a code failure. **Investigate:** vitest pool/isolate config, or run the gate in continuous integration
  (Linux) where it is green. Workaround today: chunked runs.
- **Owner:** devops-engineer.

## Status of the INT-001…014 batch (for contrast — SHIP)
Implemented, reviewed (team + code), and verified: unit tests green; the recovered regression-lab ITs
(auth/profile/interview/review/commission/favourite/showcase) green; frontend subsets + new tests green; frontend
Docker build green on Node 20; stack reseeded + deployed. Conditions carried forward: set
APP FRONTEND BASE URL for any non-local deploy (INT-006); refresh-token deferred to ROADMAP #3b.
