# Corrections to owned auxiliary files

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

2026-09-05. User-authorized document corrections; no application changes or runtime verification.

| Finding | Disposition | Result |
|---|---|---|
| AUX3-01 | corrected | Crown-jewel journey now approves the job, completes tutor profile, then applies; frontend reveal follows assignment confirmation/activation. |
| AUX3-02 | corrected | Live SHA checks use public/version and git Sha with fail-closed unknown/absent/mismatch behavior; anonymous actuator/info remains401. |
| AUX3-03 | corrected-contract | Disabled/missing email transport returns explicit DEFERRED and consumer tests require no ACK/seen/sent plus later retry delivery. Credentials were redacted, never tested. |
| AUX3-04 | corrected-with-pending-average | Replaced all-time-only funnel instructions/metric rows with017 requested-at cohort. Removed collected÷ACTIVE average; its exact date cohort remains pending017 owner record. |
| AUX3-05 | corrected | Application database/user interface eligibility now uses shared profile completeness rather than published listing; original M15 lifecycle is explicitly historical. |
| AUX3-06 | corrected | Ops directory rollout now uses authorized direct reads by default; extra directory is conditional and requires durable outbox+repair. Old plaintext rejection is explicitly superseded only for user email/phone. |
| AUX3-07 | unapproved-choice-contained | Preserved product requirements document separate analytics-store requirement and removed same-operational-connection/no-new-boundary prescription. Actual deployment/storage selection remains pending a recorded approved boundary. |
| AUX3-08 | identifier-corrected-design-pending | Reset example now uses commission invoice yearly seq consistently. It is explicitly historical and does not prove gapless allocation; current invoice/allocation scope and concurrency/rollback/rollover evidence remain needed. |
| AUX3-09 | corrected | Both prose and wireframe trust copy now scope privacy before assignment and explain later authorized mutual access. Agency-receipt money language also corrected. |
| AUX3-10 | corrected-scoped-supersession | Completion/cancellation commentary and old trigger terminal branch corrected; canonical listingless identity is explicit, with current compact ER relationship diagram. Full pre-M18 schemas remain clearly labelled migration history. |
| AUX3-11 | redacted | Historical Resend literals replaced with placeholders. Revocation remains an evidence requirement; no credential test, use or revocation claim made. |

## Additional scoped corrections

- Auth module now folds001/002/004/012/M16/027, plaintext contacts, no credential browser storage, server-owned consent evidence, shared completeness, server-driven logout/refresh revocation and scoped owner/assigned-contact visibility. Email-verification onboarding stays pending.
- INT-039/043/044 fold PENDING initiation, shared scheduler, profile/subject/location amendments, public identifier resolution, invitation terminal-state/no-timeout distinction and pending acceptance choice.
- Messaging playbook now directly corrects§2.1/§4.1/anti-pattern9: ordinary database notifications/outbox intent in-tx; independent analytics after commit; external dispatch through durable intent.
- Other affected historical module/user interface/coverage docs contain explicit scoped supersession links; original rich invoice/role/provider/analytics choices are not silently promoted or deleted.

## Changed files

- [architecture/assignment-lifecycle-design md](../../../architecture/assignment-lifecycle-design.md)
- [architecture/e2e-ci-and-test-health-spec md](../../../architecture/e2e-ci-and-test-health-spec.md)
- [architecture/email-consumer-hardening md](../../../architecture/email-consumer-hardening.md)
- [architecture/int-039-interview-negotiation md](../../../architecture/int-039-interview-negotiation.md)
- [architecture/int-043-tutor-onboarding-completion md](../../../architecture/int-043-tutor-onboarding-completion.md)
- [architecture/int-044-job-invitations md](../../../architecture/int-044-job-invitations.md)
- [architecture/int-045-analytics-date-range md](../../../architecture/int-045-analytics-date-range.md)
- [architecture/int-065-admin-block-delete-spec md](../../../architecture/int-065-admin-block-delete-spec.md)
- [architecture/int-148-e2e-full-coverage-spec md](../../../architecture/int-148-e2e-full-coverage-spec.md)
- [architecture/plaintext-email-phone md](../../../architecture/plaintext-email-phone.md)
- [architecture/product-instrumentation-spec md](../../../architecture/product-instrumentation-spec.md)
- [architecture/regression-labs-spec md](../../../architecture/regression-labs-spec.md)
- [architecture/secure-pii-and-ops-console md](../../../architecture/secure-pii-and-ops-console.md)
- [architecture/transactional-messaging-and-idempotency md](../../../architecture/transactional-messaging-and-idempotency.md)
- [modules/applications/db md](../../../modules/applications/db.md)
- [modules/applications/frontend md](../../../modules/applications/frontend.md)
- [modules/approval/debugging-guide md](../../../modules/approval/debugging-guide.md)
- [modules/approval/sequence-diagrams md](../../../modules/approval/sequence-diagrams.md)
- [modules/auth/spec md](../../../modules/auth/spec.md)
- [modules/commission/backend md](../../../modules/commission/backend.md)
- [modules/commission/er-diagram md](../../../modules/commission/er-diagram.md)
- [modules/commission/frontend md](../../../modules/commission/frontend.md)
- [modules/matching/db md](../../../modules/matching/db.md)
- [modules/matching/frontend md](../../../modules/matching/frontend.md)
- [modules/matching/module-overview md](../../../modules/matching/module-overview.md)
- [uiux/assignment-uiux-spec md](../../../uiux/assignment-uiux-spec.md)
- [uiux/int-141-marketplace-card-density-trust-spec md](../../../uiux/int-141-marketplace-card-density-trust-spec.md)
- [uiux/int-165-193-fe-audit-fixes-spec md](../../../uiux/int-165-193-fe-audit-fixes-spec.md)

## Checks and limits

- All newly introduced local file links resolve; added-line whitespace checks pass for this lane.
- Recovered Amazon Web Services source byte hash verified; no obvious Amazon Web Services/Resend credential literal in that archived plan. Historical email key redaction verified.
- Targeted checks confirm removal of the never-contact promise, seven-day user access expiry period, N/K seed placeholders, missing-table fallback claim and blanket after Commit rule.
- Repository-wide the documented operational check currently reports only peer-owned whitespace (INT-062 and domain application programming interface spec); parent notified. No own-file whitespace errors.
- No runtime tests or deployment; the new acceptance criteria remain to be exercised during implementation.
