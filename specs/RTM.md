# Requirements Traceability Matrix

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

**Updated: 2026-09-05.** All 27 numbered packages and 170 named requirements are indexed here. Use [CURRENT-REQUIREMENTS md](CURRENT-REQUIREMENTS.md) for reconciled decisions and explicitly pending choices.

The source status column records each package’s declared implementation/approval history. Documentation corrections do not imply a new runtime pass or deployment. Each package’s dated correction note states its current verification limits.

| Spec | Title | Version | Source status | Named REQs | Requirements |
|---|---|---|---|---:|---|
| [001](001-legal-and-compliance-consent-and-policy-pages/spec.md) | Legal and Compliance consent and policy pages | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 15 | [Contract](001-legal-and-compliance-consent-and-policy-pages/requirements.md) |
| [002](002-tutor-profile-and-onboarding-ux/spec.md) | Tutor profile and onboarding user experience | 2.3 | HISTORICALLY MARKED IMPLEMENTED | 14 | [Contract](002-tutor-profile-and-onboarding-ux/requirements.md) |
| [003](003-password-reset-failure-fix/spec.md) | Password reset failure fix | 1.6 | HISTORICALLY MARKED IMPLEMENTED | 5 | [Contract](003-password-reset-failure-fix/requirements.md) |
| [004](004-auth-session-hardening-30min-ttl-and-refresh/spec.md) | Auth session hardening 30min expiry period and refresh | 1.5 | HISTORICALLY MARKED IMPLEMENTED | 11 | [Contract](004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) |
| [005](005-landing-page-trust-and-social-proof/spec.md) | Landing page trust and social proof | 1.3 | APPROVED | 9 | [Contract](005-landing-page-trust-and-social-proof/requirements.md) |
| [006](006-whatsapp-interview-reminders/spec.md) | WhatsApp interview reminders | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 8 | [Contract](006-whatsapp-interview-reminders/requirements.md) |
| [007](007-generic-scheduler-db-scheduler/spec.md) | Generic scheduler platform (adopt db-scheduler) + migrate existing scheduled tasks | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 10 | [Contract](007-generic-scheduler-db-scheduler/requirements.md) |
| [008](008-tutee-post-flow-bugfixes/spec.md) | Tutee-post creation flow bug fixes (Step-4 review + Step-3 budget validation) | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 2 | [Contract](008-tutee-post-flow-bugfixes/requirements.md) |
| [009](009-tutee-post-taxonomy-and-ux/spec.md) | Tutee-post taxonomy expansion + number-of-students + time-slot grid + Rate-Guide discoverability | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 6 | [Contract](009-tutee-post-taxonomy-and-ux/requirements.md) |
| [010](010-traceid-on-every-response/spec.md) | trace Id on every application programming interface response (success AND error) — non-breaking | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 4 | [Contract](010-traceid-on-every-response/requirements.md) |
| [011](011-auth-robustness/spec.md) | Auth robustness — logout-vs-refresh race, autofill suppression, logged-out poll storm, duplicate-phone error | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 4 | [Contract](011-auth-robustness/requirements.md) |
| [012](012-tutor-apply-and-profile-ux/spec.md) | Tutor apply feedback + profile user experience — silent apply-403, completeness criteria parity, sticky-Apply, profile personal information editability | 1.4 | HISTORICALLY MARKED IMPLEMENTED | 4 | [Contract](012-tutor-apply-and-profile-ux/requirements.md) |
| [013](013-tutee-dashboard-and-invitations/spec.md) | Tutee dashboard — edit/view post, venue display, and a sent-invitations / outbound-requests view | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 4 | [Contract](013-tutee-dashboard-and-invitations/requirements.md) |
| [014](014-interview-scheduling-ux-and-admin-visibility/spec.md) | Interview scheduling user experience + admin visibility — list-filter gap, auto-fill slot, clipped No-Show, datetime picker, raw-handle display | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 5 | [Contract](014-interview-scheduling-ux-and-admin-visibility/requirements.md) |
| [015](015-mobile-responsive-fixes/spec.md) | Mobile/responsive fixes — availability-grid page overflow, admin nav at 390px, tutor nav breakpoint consistency | 1.1 | APPROVED | 3 | [Contract](015-mobile-responsive-fixes/requirements.md) |
| [016](016-error-copy-and-handling/spec.md) | Error copy + handling — humanise raw "not found: `<uuid>`", no-handler 500→404, create-admin field naming, tel: escaping | 1.1 | HISTORICALLY MARKED IMPLEMENTED | 4 | [Contract](016-error-copy-and-handling/requirements.md) |
| [017](017-admin-analytics-and-financial-safety/spec.md) | Admin analytics correctness + financial safety — conversion >100% cohort bug, avg-commission calc, Mark-Paid confirm | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 3 | [Contract](017-admin-analytics-and-financial-safety/requirements.md) |
| [018](018-flag-hygiene-seeder-and-fe-hygiene/spec.md) | Flag hygiene, seeder fix, and small frontend hygiene — WhatsApp toast while OFF, seeded interview tutor user id, /dashboard 404, step copy, CTA intent, demo-data variety | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 6 | [Contract](018-flag-hygiene-seeder-and-fe-hygiene/requirements.md) |
| [019](019-admin-money-safety-and-analytics/spec.md) | admin money safety and analytics | 0.2 | HISTORICALLY MARKED IMPLEMENTED | 7 | [Contract](019-admin-money-safety-and-analytics/spec.md) |
| [020](020-taxonomy-integrity/spec.md) | taxonomy integrity | 0.3 | HISTORICALLY MARKED IMPLEMENTED | 3 | [Contract](020-taxonomy-integrity/spec.md) |
| [021](021-tutee-request-flow/spec.md) | tutee request flow | 0.2 | HISTORICALLY MARKED IMPLEMENTED | 3 | [Contract](021-tutee-request-flow/spec.md) |
| [022](022-tutor-profile-and-fe-polish/spec.md) | tutor profile and fe polish | 0.2 | HISTORICALLY MARKED IMPLEMENTED | 5 | [Contract](022-tutor-profile-and-fe-polish/spec.md) |
| [023](023-seed-data-variety/spec.md) | seed data variety | 0.2 | HISTORICALLY MARKED IMPLEMENTED | 2 | [Contract](023-seed-data-variety/spec.md) |
| [024](024-postfix-review-fixes/spec.md) | postfix review fixes | 0.1 | HISTORICALLY MARKED IMPLEMENTED | 6 | [Contract](024-postfix-review-fixes/spec.md) |
| [025](025-taxonomy-stream-year-scoping/spec.md) | taxonomy stream year scoping | 0.3 | HISTORICALLY MARKED IMPLEMENTED | 7 | [Contract](025-taxonomy-stream-year-scoping/requirements.md) |
| [026](026-s3s4-polish-batch/spec.md) | s3s4 polish batch | 0.2 | HISTORICALLY MARKED IMPLEMENTED | 8 | [Contract](026-s3s4-polish-batch/spec.md) |
| [027](027-aws-production-deployment/spec.md) | aws production deployment | 0.3 | APPROVED | 12 | [Contract](027-aws-production-deployment/spec.md) |

## Cross-package amendments

- 007 supersedes006’s reminder mechanism; reminder content, opt-in and consent-copy requirements remain.
- 009’s approved taxonomy amendments replace the old mixed N LEVEL table;020/025 own coherence and finer scoping. Remaining approval-record gaps stay explicit.
- 012’s founder email-link fallback supersedes WhatsApp-only/pending-only phone changes.
- 015 mobile drawer/accessibility requirements remain when019 extends admin usability.
- 021 allows PENDING initiation/editing; acceptance timing is separately unresolved O01.
- 027’s recovered approved plan and current phase amendments govern Amazon Web Services production. The original cost/restore estimates remain historical estimates.

The [pre-correction audit](reviews/2026-09-05-requirements/spec-alignment.md) records why these corrections were needed. Its old line citations refer to the reviewed baseline, not a freshly tested implementation.
