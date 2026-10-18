# Owned auxiliary corrections

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

Corrections confined to original38 auxiliary file manifest;29 changed,9 left unchanged after targeted alignment review. Older code/database query language/runtime evidence remains historical. No source-code/runtime actions.

Date-05. Specification changes only; no new HISTORICALLY MARKED IMPLEMENTED/runtime/test claim.

## Finding disposition

### A2-F1 — corrected

PAID only after agency receives tutor commission; a won-but-unpaid dispute remains uncollected; pending unpaid resolution/status policy labeled.

[commission/spec md L208](../../../modules/commission/spec.md)

### A2-F2 — corrected to parent requirement

Authenticated matched-tutee first-session confirmation; public token alone insufficient; GET no mutation; override confirms first session rather than completing match.

[matching/backend md L232](../../../modules/matching/backend.md)

### A2-F3 — explicitly pending

Only rate hard-clamp and4.33-versus4 remain provisional; FLEXIBLE no-prefill, storage and activation lock are settled.

[architecture/int-157-commission-first-month-locked-spec md L115](../../../architecture/int-157-commission-first-month-locked-spec.md)

### A2-F4 — corrected

Reject guard uses canonical PENDING; PAUSED FOR REVIEW removed from current clause.

[approval/backend md L199](../../../modules/approval/backend.md)

### A2-F5 — corrected

Removed mandatory-listing guard from application requirements and B4 regression assertion; link M18 and012 canonical gate.

[applications/backend md L62](../../../modules/applications/backend.md); [architecture/regression-branch-closure-spec md L3](../../../architecture/regression-branch-closure-spec.md)

### A2-F6 — corrected

Public/pre-match privacy plus explicit ACTIVE mutual-participant contact exception; no never-share-ever promise.

[profile/account-ui-changes md L3](../../../modules/profile/account-ui-changes.md); [architecture/int-090-batch-m18-funnel-completion-spec md L3](../../../architecture/int-090-batch-m18-funnel-completion-spec.md)

### A2-F7 — corrected specification coverage

Simplified module summaries explicitly retain first-session confirmation/invoice obligations and receipt-based completion; no implementation waiver inferred.

[commission/module-overview md L3](../../../modules/commission/module-overview.md); [architecture/int-157-commission-first-month-locked-spec md L3](../../../architecture/int-157-commission-first-month-locked-spec.md)

### A2-F8 — unsafe blanket recommendation removed; criterion pending

No universal verified badge or unsupported all-tutors-vetted claim. Any badge requires an approved meaning/eligibility criterion and evidence.

[uiux/frontend-design-review-2026-06-13.md L74](../../../uiux/frontend-design-review-2026-06-13.md)

## Changed files

- [architecture/assignment-be-spec md](../../../architecture/assignment-be-spec.md)
- [architecture/assignment-fe-spec md](../../../architecture/assignment-fe-spec.md)
- [architecture/int-093-batch-state-machine-integrity-spec md](../../../architecture/int-093-batch-state-machine-integrity-spec.md)
- [architecture/int-095-batch-auth-session-hardening-spec md](../../../architecture/int-095-batch-auth-session-hardening-spec.md)
- [architecture/int-157-commission-first-month-locked-spec md](../../../architecture/int-157-commission-first-month-locked-spec.md)
- [architecture/match-lifecycle-fixes-spec md](../../../architecture/match-lifecycle-fixes-spec.md)
- [architecture/regression-branch-closure-spec md](../../../architecture/regression-branch-closure-spec.md)
- [architecture/regression-password-reset-fixtures-spec md](../../../architecture/regression-password-reset-fixtures-spec.md)
- [modules/applications/backend md](../../../modules/applications/backend.md)
- [modules/applications/spec md](../../../modules/applications/spec.md)
- [modules/approval/backend md](../../../modules/approval/backend.md)
- [modules/approval/er-diagram md](../../../modules/approval/er-diagram.md)
- [modules/approval/spec md](../../../modules/approval/spec.md)
- [modules/auth/db md](../../../modules/auth/db.md)
- [modules/commission/api-reference md](../../../modules/commission/api-reference.md)
- [modules/commission/db md](../../../modules/commission/db.md)
- [modules/commission/module-overview md](../../../modules/commission/module-overview.md)
- [modules/commission/spec md](../../../modules/commission/spec.md)
- [modules/interview/max-interviews-per-posting md](../../../modules/interview/max-interviews-per-posting.md)
- [modules/interview/spec md](../../../modules/interview/spec.md)
- [modules/landing-showcase/spec md](../../../modules/landing-showcase/spec.md)
- [modules/matching/backend md](../../../modules/matching/backend.md)
- [modules/matching/debugging-guide md](../../../modules/matching/debugging-guide.md)
- [modules/posting/backend md](../../../modules/posting/backend.md)
- [modules/posting/db md](../../../modules/posting/db.md)
- [modules/posting/spec md](../../../modules/posting/spec.md)
- [modules/profile/account-ui-changes md](../../../modules/profile/account-ui-changes.md)
- [uiux/all-pages-redesign-spec md](../../../uiux/all-pages-redesign-spec.md)
- [uiux/frontend-design-review-2026-06-13.md](../../../uiux/frontend-design-review-2026-06-13.md)

## Checks

- The whitespace check on the edited files reported no errors.
- New local link targets verified: 52.
- Markdown code fences balanced; numbered requirement IDs retained.
- No runtime/application tests executed.
- Diff artifact: [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md).

## Additional corrected clauses

- Commission average excludes unpaid INVOICED balances;017/O02 time cohort remains pending
- Explicit plaintext-email-phone amendment supersedes contact encryption/keyed message authentication assumptions in auth/db and profile notes
- INT-125/129 extends frontend participant reveal to ACTIVE+COMPLETED, retains hidden match-PENDING and leaves unrelated ACTIVE sections unchanged

## All assigned files

- [architecture/admin-identity-linking-spec md](../../../architecture/admin-identity-linking-spec.md): unchanged; no settled correction identified in targeted review.
- [architecture/assignment-be-spec md](../../../architecture/assignment-be-spec.md): corrected / supersession clarified.
- [architecture/assignment-fe-spec md](../../../architecture/assignment-fe-spec.md): corrected / supersession clarified.
- [architecture/backend-correctness-security-spec md](../../../architecture/backend-correctness-security-spec.md): unchanged; no settled correction identified in targeted review.
- [architecture/int-025-admin-submit-notification md](../../../architecture/int-025-admin-submit-notification.md): unchanged; no settled correction identified in targeted review.
- [architecture/int-046-fe-coverage-closeout md](../../../architecture/int-046-fe-coverage-closeout.md): unchanged; no settled correction identified in targeted review.
- [architecture/int-090-batch-m18-funnel-completion-spec md](../../../architecture/int-090-batch-m18-funnel-completion-spec.md): unchanged; no settled correction identified in targeted review.
- [architecture/int-093-batch-state-machine-integrity-spec md](../../../architecture/int-093-batch-state-machine-integrity-spec.md): corrected / supersession clarified.
- [architecture/int-095-batch-auth-session-hardening-spec md](../../../architecture/int-095-batch-auth-session-hardening-spec.md): corrected / supersession clarified.
- [architecture/int-138-int137-codereview-fastfollow-spec md](../../../architecture/int-138-int137-codereview-fastfollow-spec.md): unchanged; no settled correction identified in targeted review.
- [architecture/int-157-commission-first-month-locked-spec md](../../../architecture/int-157-commission-first-month-locked-spec.md): corrected / supersession clarified.
- [architecture/m14b-public-job-postings-showcase md](../../../architecture/m14b-public-job-postings-showcase.md): unchanged; no settled correction identified in targeted review.
- [architecture/match-lifecycle-fixes-spec md](../../../architecture/match-lifecycle-fixes-spec.md): corrected / supersession clarified.
- [architecture/regression-branch-closure-spec md](../../../architecture/regression-branch-closure-spec.md): corrected / supersession clarified.
- [architecture/regression-password-reset-fixtures-spec md](../../../architecture/regression-password-reset-fixtures-spec.md): corrected / supersession clarified.
- [modules/applications/backend md](../../../modules/applications/backend.md): corrected / supersession clarified.
- [modules/applications/spec md](../../../modules/applications/spec.md): corrected / supersession clarified.
- [modules/approval/backend md](../../../modules/approval/backend.md): corrected / supersession clarified.
- [modules/approval/er-diagram md](../../../modules/approval/er-diagram.md): corrected / supersession clarified.
- [modules/approval/frontend md](../../../modules/approval/frontend.md): unchanged; no settled correction identified in targeted review.
- [modules/approval/spec md](../../../modules/approval/spec.md): corrected / supersession clarified.
- [modules/auth/db md](../../../modules/auth/db.md): corrected / supersession clarified.
- [modules/commission/api-reference md](../../../modules/commission/api-reference.md): corrected / supersession clarified.
- [modules/commission/db md](../../../modules/commission/db.md): corrected / supersession clarified.
- [modules/commission/debugging-guide md](../../../modules/commission/debugging-guide.md): unchanged; no settled correction identified in targeted review.
- [modules/commission/module-overview md](../../../modules/commission/module-overview.md): corrected / supersession clarified.
- [modules/commission/spec md](../../../modules/commission/spec.md): corrected / supersession clarified.
- [modules/interview/max-interviews-per-posting md](../../../modules/interview/max-interviews-per-posting.md): corrected / supersession clarified.
- [modules/interview/spec md](../../../modules/interview/spec.md): corrected / supersession clarified.
- [modules/landing-showcase/spec md](../../../modules/landing-showcase/spec.md): corrected / supersession clarified.
- [modules/matching/backend md](../../../modules/matching/backend.md): corrected / supersession clarified.
- [modules/matching/debugging-guide md](../../../modules/matching/debugging-guide.md): corrected / supersession clarified.
- [modules/posting/backend md](../../../modules/posting/backend.md): corrected / supersession clarified.
- [modules/posting/db md](../../../modules/posting/db.md): corrected / supersession clarified.
- [modules/posting/spec md](../../../modules/posting/spec.md): corrected / supersession clarified.
- [modules/profile/account-ui-changes md](../../../modules/profile/account-ui-changes.md): corrected / supersession clarified.
- [uiux/all-pages-redesign-spec md](../../../uiux/all-pages-redesign-spec.md): corrected / supersession clarified.
- [uiux/frontend-design-review-2026-06-13.md](../../../uiux/frontend-design-review-2026-06-13.md): corrected / supersession clarified.
