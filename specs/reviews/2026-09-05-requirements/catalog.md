# InTuition requirements source catalog

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](corrections.md) and [current requirements](../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

**Review date:** 2026-09-05. Read alongside [the audit](audit.md).

This is a source index, not a new approved product requirements document or a claim that every listed requirement is implemented. It indexes all 27 numbered spec folders currently present: 170 distinct namespaced requirement headings. It also indexes the 112 feature rows, 19 acceptance-criteria headings, and 15 business rules in the older product requirements document/business requirements document. These groups overlap and must not be summed into a count of unique current requirements. Detailed clauses remain in the linked source documents.

HISTORICALLY MARKED IMPLEMENTED and APPROVED below are the source documents' declarations, not verified deployment status. Older requirements can be superseded by later decisions; see the audit for known reconciliations. Requirements with the same REQ-001 identifier are unique only when combined with their spec folder.

## Numbered spec coverage

| Spec | Version | Declared status | Named entries |
|---|---|---|---:|
| [001-legal-and-compliance-consent-and-policy-pages](../../001-legal-and-compliance-consent-and-policy-pages/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 15 |
| [002-tutor-profile-and-onboarding-ux](../../002-tutor-profile-and-onboarding-ux/spec.md) | 2.2 | HISTORICALLY MARKED IMPLEMENTED | 14 |
| [003-password-reset-failure-fix](../../003-password-reset-failure-fix/spec.md) | 1.5 | HISTORICALLY MARKED IMPLEMENTED | 5 |
| [004-auth-session-hardening-30min-ttl-and-refresh](../../004-auth-session-hardening-30min-ttl-and-refresh/spec.md) | 1.4 | HISTORICALLY MARKED IMPLEMENTED | 11 |
| [005-landing-page-trust-and-social-proof](../../005-landing-page-trust-and-social-proof/spec.md) | 1.2 | APPROVED | 9 |
| [006-whatsapp-interview-reminders](../../006-whatsapp-interview-reminders/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 8 |
| [007-generic-scheduler-db-scheduler](../../007-generic-scheduler-db-scheduler/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 10 |
| [008-tutee-post-flow-bugfixes](../../008-tutee-post-flow-bugfixes/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 2 |
| [009-tutee-post-taxonomy-and-ux](../../009-tutee-post-taxonomy-and-ux/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 6 |
| [010-traceid-on-every-response](../../010-traceid-on-every-response/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 4 |
| [011-auth-robustness](../../011-auth-robustness/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 4 |
| [012-tutor-apply-and-profile-ux](../../012-tutor-apply-and-profile-ux/spec.md) | 1.3 | HISTORICALLY MARKED IMPLEMENTED | 4 |
| [013-tutee-dashboard-and-invitations](../../013-tutee-dashboard-and-invitations/spec.md) | 1.1 | HISTORICALLY MARKED IMPLEMENTED | 4 |
| [014-interview-scheduling-ux-and-admin-visibility](../../014-interview-scheduling-ux-and-admin-visibility/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 5 |
| [015-mobile-responsive-fixes](../../015-mobile-responsive-fixes/spec.md) | 1.1 | APPROVED | 3 |
| [016-error-copy-and-handling](../../016-error-copy-and-handling/spec.md) | 1.0 | HISTORICALLY MARKED IMPLEMENTED | 4 |
| [017-admin-analytics-and-financial-safety](../../017-admin-analytics-and-financial-safety/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 3 |
| [018-flag-hygiene-seeder-and-fe-hygiene](../../018-flag-hygiene-seeder-and-fe-hygiene/spec.md) | 1.2 | HISTORICALLY MARKED IMPLEMENTED | 6 |
| [019-admin-money-safety-and-analytics](../../019-admin-money-safety-and-analytics/spec.md) | 0.1 | HISTORICALLY MARKED IMPLEMENTED | 7 |
| [020-taxonomy-integrity](../../020-taxonomy-integrity/spec.md) | 0.1 | HISTORICALLY MARKED IMPLEMENTED | 3 |
| [021-tutee-request-flow](../../021-tutee-request-flow/spec.md) | 0.1 | HISTORICALLY MARKED IMPLEMENTED | 3 |
| [022-tutor-profile-and-fe-polish](../../022-tutor-profile-and-fe-polish/spec.md) | 0.1 | HISTORICALLY MARKED IMPLEMENTED | 5 |
| [023-seed-data-variety](../../023-seed-data-variety/spec.md) | 0.1 | HISTORICALLY MARKED IMPLEMENTED | 2 |
| [024-postfix-review-fixes](../../024-postfix-review-fixes/spec.md) | 0.1 | HISTORICALLY MARKED IMPLEMENTED | 6 |
| [025-taxonomy-stream-year-scoping](../../025-taxonomy-stream-year-scoping/spec.md) | 0.2 | HISTORICALLY MARKED IMPLEMENTED | 7 |
| [026-s3s4-polish-batch](../../026-s3s4-polish-batch/spec.md) | 0.1 | HISTORICALLY MARKED IMPLEMENTED | 8 |
| [027-aws-production-deployment](../../027-aws-production-deployment/spec.md) | 0.1 | APPROVED | 12 |

## Named requirement entries

### 001-legal-and-compliance-consent-and-policy-pages

| Requirement | Source heading |
|---|---|
| [001/REQ-001](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When a user submits the registration form with all required consents ticked, the system shall persist the Terms-and-Privacy agreement, the Personal Data Protection Act data-use consent, and the age/guardian confirmation each with its own audit timestamp. |
| [001/REQ-002](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When a user submits registration with the marketing opt-in ticked, the system shall persist marketing opt in = TRUE with marketing opt in at set; when it is left unticked, the system shall persist marketing opt in = FALSE with marketing opt in at NULL, and registration shall still succeed. |
| [001/REQ-003](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | While the registration form has the Terms-and-Privacy agreement checkbox unticked, the system shall block submission and surface a validation message. |
| [001/REQ-004](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When the registration form is rendered, the system shall display all four consent checkboxes with the exact wording from GH #198, with the Terms-of-Use and Privacy-Notice references as links to /terms and /privacy. |
| [001/REQ-005](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When the tutee-registration application programming interface receives a request, the system shall require the three required consent flags (terms accepted, pdpa consent, age confirmed) to be true and persist them with server-set timestamps. |
| [001/REQ-006](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When the tutor-registration application programming interface receives a request, the system shall require the three required consent flags and persist them with server-set timestamps, alongside the existing commission acknowledgement. |
| [001/REQ-007](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | The system shall add the four consent column pairs PLUS the two policy-version columns to the users table via a Liquibase changeset (0058), each boolean defaulting FALSE and guarded by a require that the flag cannot be true without its timestamp, with the ALTERs bounded by SET LOCAL lock timeout is 3s. |
| [001/REQ-008](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | If a registration request reaches the application programming interface without all three required consent flags set true, then the system shall reject it with a 4xx validation error and create no account. |
| [001/REQ-009](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When a visitor opens /terms, the system shall display the full Terms & Conditions copy from GH #200, including the company name + UEN and a contactable business address. |
| [001/REQ-010](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When a visitor opens /privacy, the system shall display the full Privacy Policy copy from GH #199, including the DPO contact block with the retained business address. |
| [001/REQ-011](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | The system shall remove the company street address from the public Footer. |
| [001/REQ-012](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | The system shall remove the company street address from the public Contact page, while keeping the legal name and UEN. |
| [001/REQ-013](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | While the address is removed from the Footer and Contact page, the system shall continue to display the company street address on the /privacy and /terms pages. |
| [001/REQ-014](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When a registration succeeds with the required consents, the system shall write a CONSENT GIVEN audit-log row recording the consent set and the policy versions, alongside the existing USER REGISTERED (and, for tutors, COMMISSION ACKNOWLEDGED) rows. |
| [001/REQ-015](../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) | When a registration succeeds, the system shall record the Terms and Privacy policy versions the user consented to (terms version, privacy version on users), populated from a single source-of-truth constant. |

### 002-tutor-profile-and-onboarding-ux

| Requirement | Source heading |
|---|---|
| [002/REQ-001](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Tutor selects availability in fine-grained 1-hour blocks |
| [002/REQ-002](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Tutor selects availability in 2-hour blocks via a granularity control |
| [002/REQ-003](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Backend persists tutor-post availability at 1-hour granularity (junction table) |
| [002/REQ-004](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Availability application programming interface accepts and returns the fine-grained slot list |
| [002/REQ-005](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Canonical 1-hour slot vocabulary is a single shared source of truth |
| [002/REQ-006](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Existing 21-bit availability is migrated forward to fine-grained slots (no data loss) |
| [002/REQ-007](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Step 3 "Preferred MRT station / area" accepts MULTIPLE selections |
| [002/REQ-008b](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Tutor onboarding captures the full prefill set (rate, area, region, subjects, commitment, qualifications, years) and persists them at registration |
| [002/REQ-008c](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Onboarding registration succeeds when the new optional fields are omitted (INT-043 backward compat) |
| [002/REQ-008](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Tutor profile pre-populates ALL onboarding-derived fields (rate, subjects, qualification, years, area) |
| [002/REQ-009](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Invalid availability slot input is rejected |
| [002/REQ-010](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Public handle is auto-generated on first load, then editable |
| [002/REQ-011](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Reject an edited public handle that violates the handle rules |
| [002/REQ-012](../../002-tutor-profile-and-onboarding-ux/requirements.md) | Verify the profile user interface already supports adding more than one subject |

### 003-password-reset-failure-fix

| Requirement | Source heading |
|---|---|
| [003/REQ-001](../../003-password-reset-failure-fix/requirements.md) | Reproduce and capture the exact reset failure before any fix is designed |
| [003/REQ-002](../../003-password-reset-failure-fix/requirements.md) | A valid one-time passcode + compliant new password completes the reset and the user logs in with the new password, NOT bounced to forced-change |
| [003/REQ-003](../../003-password-reset-failure-fix/requirements.md) | Reset error messages are accurate and non-misleading, without leaking account existence |
| [003/REQ-004](../../003-password-reset-failure-fix/requirements.md) | A regression test reproduces the defect (FAILS before the fix, PASSES after) in the gated test layer |
| [003/REQ-005](../../003-password-reset-failure-fix/requirements.md) | one-time passcode debug logging (founder-approved, all OTPs, recipient + validity, all profiles incl. prod) |

### 004-auth-session-hardening-30min-ttl-and-refresh

| Requirement | Source heading |
|---|---|
| [004/REQ-001](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | 30-minute access-token expiry period for TUTOR/TUTEE |
| [004/REQ-002](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Refresh endpoint mints a new access token |
| [004/REQ-003](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Refresh-token rotation with reuse detection (concurrency-safe) |
| [004/REQ-004](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Refresh respects the existing revocation/reset/block kill-switches |
| [004/REQ-005](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Logout clears AND revokes the refresh token |
| [004/REQ-006](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Login/register/multi-factor authentication issue both an access cookie and a refresh cookie |
| [004/REQ-007](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Sliding window with an absolute-lifetime cap |
| [004/REQ-008](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Expired refresh token (idle expiry period elapsed) is rejected |
| [004/REQ-009](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | frontend single-flight silent-refresh-then-retry on a 401, else redirect to login |
| [004/REQ-010](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Admin stateful sessions are unchanged (no regression) |
| [004/REQ-011](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) | Rate-limit on the refresh endpoint (refresh-grinding defence) |

### 005-landing-page-trust-and-social-proof

| Requirement | Source heading |
|---|---|
| [005/REQ-001](../../005-landing-page-trust-and-social-proof/requirements.md) | VERIFY the #161 stats-band reframe is live and correct, then close #161 |
| [005/REQ-002](../../005-landing-page-trust-and-social-proof/requirements.md) | Real adoption/volume metric stays DEFERRED (out of scope for now) |
| [005/REQ-003](../../005-landing-page-trust-and-social-proof/requirements.md) | Testimonial strip DESIGN (placement, responsive, a11y) on the landing page |
| [005/REQ-004](../../005-landing-page-trust-and-social-proof/requirements.md) | Testimonial strip BUILD — BLOCKED on founder content (REQ-007) |
| [005/REQ-005](../../005-landing-page-trust-and-social-proof/requirements.md) | The strip pulls REAL M11 reviews via a defined, non-fabricated source |
| [005/REQ-006](../../005-landing-page-trust-and-social-proof/requirements.md) | Testimonial strip meets a11y + CWV budget (no LCP-tanking carousel) |
| [005/REQ-007](../../005-landing-page-trust-and-social-proof/requirements.md) | BLOCKER: founder supplies real testimonial content OR approval to curate real M11 reviews |
| [005/REQ-008](../../005-landing-page-trust-and-social-proof/requirements.md) | Attribution respects consent and the personal information model |
| [005/REQ-009](../../005-landing-page-trust-and-social-proof/requirements.md) | Designed empty / "coming-soon" state while content is pending (no blank void) |

### 006-whatsapp-interview-reminders

| Requirement | Source heading |
|---|---|
| [006/REQ-001](../../006-whatsapp-interview-reminders/requirements.md) | Link-gated, parameterized WhatsApp schedule-notify to both parties |
| [006/REQ-002](../../006-whatsapp-interview-reminders/requirements.md) | Parameterized WhatsApp notification pipeline (payload carries body Params; consumer builds the body component) |
| [006/REQ-003](../../006-whatsapp-interview-reminders/requirements.md) | Idempotent 24h-before reminder sweep Whats Apps both parties |
| [006/REQ-004](../../006-whatsapp-interview-reminders/requirements.md) | reminder sent column on interviews (migration 0061, additive/back-compat) |
| [006/REQ-005](../../006-whatsapp-interview-reminders/requirements.md) | Rescheduling an interview resets reminder sent (re-arms the reminder) |
| [006/REQ-006](../../006-whatsapp-interview-reminders/requirements.md) | INTERVIEW REMINDER is a delivered WhatsApp notification event |
| [006/REQ-007](../../006-whatsapp-interview-reminders/requirements.md) | Three Meta WhatsApp template definitions (one-time passcode, scheduled-with-params, reminder) are drafted for registration |
| [006/REQ-008](../../006-whatsapp-interview-reminders/requirements.md) | The whole feature stays toggle-gated/dormant and back-compatible (no behaviour change when WhatsApp OFF) |

### 007-generic-scheduler-db-scheduler

| Requirement | Source heading |
|---|---|
| [007/REQ-001](../../007-generic-scheduler-db-scheduler/requirements.md) | db-scheduler dependency + the persistent scheduled tasks table (migration 0062, Liquibase-owned database structure definitions) |
| [007/REQ-002](../../007-generic-scheduler-db-scheduler/requirements.md) | Generic registration surface: a scheduled job = one Task a registered application component + a handler (first-class) |
| [007/REQ-003](../../007-generic-scheduler-db-scheduler/requirements.md) | Interview 24h reminder as a PER-INSTANCE one-time job (instance Id = interview Id, NO serialized payload) |
| [007/REQ-004](../../007-generic-scheduler-db-scheduler/requirements.md) | Reminder handler RE-GUARD is mandatory and explicitly tested (fire on cancelled/rescheduled → 0 sends) |
| [007/REQ-005](../../007-generic-scheduler-db-scheduler/requirements.md) | EVERY interview lifecycle transition cancels-or-reschedules the reminder job (enumerated, none missed) |
| [007/REQ-006](../../007-generic-scheduler-db-scheduler/requirements.md) | Migration 0063 drops reminder sent + idx interview reminder due (supersedes spec-006 mechanism) |
| [007/REQ-007](../../007-generic-scheduler-db-scheduler/requirements.md) | Interview expiry + showcase reroll become RECURRING registered tasks (logic unchanged) |
| [007/REQ-008](../../007-generic-scheduler-db-scheduler/requirements.md) | Observability + bounded retry/give-up are IN-SCOPE (metrics shim, failure alert, give-up policy) |
| [007/REQ-009](../../007-generic-scheduler-db-scheduler/requirements.md) | Shed Lock fully removed, with single-ownership during migration (no task runs under both mechanisms) |
| [007/REQ-010](../../007-generic-scheduler-db-scheduler/requirements.md) | User-facing reminder behaviour is identical; the migration is back-compatible (no observable change while WhatsApp OFF) |

### 008-tutee-post-flow-bugfixes

| Requirement | Source heading |
|---|---|
| [008/REQ-001](../../008-tutee-post-flow-bugfixes/requirements.md) | Step-4 "Review" must not submit before the explicit "Submit Request" click |
| [008/REQ-002](../../008-tutee-post-flow-bugfixes/requirements.md) | Step-3 budget cross-field validation re-fires the partner field on change |

### 009-tutee-post-taxonomy-and-ux

| Requirement | Source heading |
|---|---|
| [009/REQ-001](../../009-tutee-post-taxonomy-and-ux/requirements.md) | Subject taxonomy expansion (#203) |
| [009/REQ-002](../../009-tutee-post-taxonomy-and-ux/requirements.md) | Level taxonomy expansion (#204) |
| [009/REQ-003](../../009-tutee-post-taxonomy-and-ux/requirements.md) | Taxonomy validation: unknown subject/level returns 422, not 500 (bug-prevention) |
| [009/REQ-004](../../009-tutee-post-taxonomy-and-ux/requirements.md) | Number of students: "1–5 + 6 or more" with an exact-count reveal (#205) |
| [009/REQ-005](../../009-tutee-post-taxonomy-and-ux/requirements.md) | Time-slot grid transpose to time×day + helper-text move (#206 user experience) — bitmask byte-for-byte unchanged (BLOCKING) |
| [009/REQ-006](../../009-tutee-post-taxonomy-and-ux/requirements.md) | Rate-Guide discoverability: three links to the existing /rates page (#202 + #206-3) |

### 010-traceid-on-every-response

| Requirement | Source heading |
|---|---|
| [010/REQ-001](../../010-traceid-on-every-response/requirements.md) | trace Id in every ERROR response body |
| [010/REQ-002](../../010-traceid-on-every-response/requirements.md) | trace Id in every JSON-object SUCCESS body (non-breaking, via response customization component) |
| [010/REQ-003](../../010-traceid-on-every-response/requirements.md) | X-Correlation-ID header on ALL responses (confirm + keep) and cross-origin request permissions-exposed |
| [010/REQ-004](../../010-traceid-on-every-response/requirements.md) | frontend surfaces the trace Id on errors (copyable reference) |

### 011-auth-robustness

| Requirement | Source heading |
|---|---|
| [011/REQ-001](../../011-auth-robustness/requirements.md) | Logout reliably revokes and halts the silent-refresh loop before navigation |
| [011/REQ-002](../../011-auth-robustness/requirements.md) | Auth forms resist stale browser autofill |
| [011/REQ-003](../../011-auth-robustness/requirements.md) | The notification unread-count poll is gated on an authenticated session (confirm + pin) |
| [011/REQ-004](../../011-auth-robustness/requirements.md) | Duplicate-phone registration returns a SPECIFIC message, with anti-enumeration hardening (FOUNDER OVERRIDE of INT-109) |

### 012-tutor-apply-and-profile-ux

| Requirement | Source heading |
|---|---|
| [012/REQ-001](../../012-tutor-apply-and-profile-ux/requirements.md) | Apply gate failure ALWAYS surfaces friendly, actionable feedback; frontend↔backend completeness criteria reconciled |
| [012/REQ-002](../../012-tutor-apply-and-profile-ux/requirements.md) | Sticky-Apply rail reflects the applied / gated state (T-06) |
| [012/REQ-003](../../012-tutor-apply-and-profile-ux/requirements.md) | Tutor subject taxonomy uses the shared options (CONFIRM + CLOSE #209.1) |
| [012/REQ-004](../../012-tutor-apply-and-profile-ux/requirements.md) | Self-edit phone/email via an EMAIL-LINK confirmation flow with anti-account-takeover (FOUNDER OVERRIDE — #209.3) |

### 013-tutee-dashboard-and-invitations

| Requirement | Source heading |
|---|---|
| [013/REQ-001](../../013-tutee-dashboard-and-invitations/requirements.md) | Tutee can view and edit a post from the dashboard |
| [013/REQ-002](../../013-tutee-dashboard-and-invitations/requirements.md) | Dashboard shows all selected venues (not just the first) |
| [013/REQ-003](../../013-tutee-dashboard-and-invitations/requirements.md) | Tutee sent-invitations: persistent read endpoint + dashboard surface |
| [013/REQ-004](../../013-tutee-dashboard-and-invitations/requirements.md) | Outbound interview requests surface on the tutee dashboard |

### 014-interview-scheduling-ux-and-admin-visibility

| Requirement | Source heading |
|---|---|
| [014/REQ-001](../../014-interview-scheduling-ux-and-admin-visibility/requirements.md) | Admin interview queue surfaces ALL live statuses (no invisible interviews) |
| [014/REQ-002](../../014-interview-scheduling-ux-and-admin-visibility/requirements.md) | Scheduling auto-fills the accepted timeslot |
| [014/REQ-003](../../014-interview-scheduling-ux-and-admin-visibility/requirements.md) | Record-No-Show (and all interview actions) are fully visible, not clipped |
| [014/REQ-004](../../014-interview-scheduling-ux-and-admin-visibility/requirements.md) | A usable date/time picker for scheduling (all roles) |
| [014/REQ-005](../../014-interview-scheduling-ux-and-admin-visibility/requirements.md) | Show the tutor's display name, not the raw handle, on Interview/Applicants screens |

### 015-mobile-responsive-fixes

| Requirement | Source heading |
|---|---|
| [015/REQ-001](../../015-mobile-responsive-fixes/requirements.md) | The availability grid scrolls within itself; the page does not overflow at ≤390px |
| [015/REQ-002](../../015-mobile-responsive-fixes/requirements.md) | The admin nav is usable at 390px (hamburger/drawer) |
| [015/REQ-003](../../015-mobile-responsive-fixes/requirements.md) | Consistent nav breakpoint across routes |

### 016-error-copy-and-handling

| Requirement | Source heading |
|---|---|
| [016/REQ-001](../../016-error-copy-and-handling/requirements.md) | User-facing error copy is humanised; no entity-name/id leak (trace Id kept) |
| [016/REQ-002](../../016-error-copy-and-handling/requirements.md) | A request to a nonexistent route returns 404, not 500 |
| [016/REQ-003](../../016-error-copy-and-handling/requirements.md) | Create-admin 400 names the missing/invalid field |
| [016/REQ-004](../../016-error-copy-and-handling/requirements.md) | tel:/mailto: hrefs are URL-encoded |

### 017-admin-analytics-and-financial-safety

| Requirement | Source heading |
|---|---|
| [017/REQ-001](../../017-admin-analytics-and-financial-safety/requirements.md) | Interview conversion rate is cohort-consistent and never exceeds 100% |
| [017/REQ-002](../../017-admin-analytics-and-financial-safety/requirements.md) | Avg-commission calc and label agree on one cohort/denominator |
| [017/REQ-003](../../017-admin-analytics-and-financial-safety/requirements.md) | Mark-Paid requires explicit confirmation (financial safety) |

### 018-flag-hygiene-seeder-and-fe-hygiene

| Requirement | Source heading |
|---|---|
| [018/REQ-001](../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md) | No WhatsApp verification messaging while the feature is OFF (T-05) |
| [018/REQ-002](../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md) | Seeder sets tutor user id on seeded interviews (T-08) |
| [018/REQ-003](../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md) | Seed-data variety on landing / assignments (demo-data) |
| [018/REQ-004](../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md) | /dashboard redirects to the role dashboard (T-01) |
| [018/REQ-005](../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md) | Step copy matches the actual required fields (T-03) |
| [018/REQ-006](../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md) | Unauthenticated CTAs default to login (CTA-intent) |

### 019-admin-money-safety-and-analytics

| Requirement | Source heading |
|---|---|
| [019/REQ-001](../../019-admin-money-safety-and-analytics/spec.md) | ADM-01 Mark-Paid confirm on the Matching page (S2, money) |
| [019/REQ-002](../../019-admin-money-safety-and-analytics/spec.md) | ADM-02 analytics period metrics (S2, diagnose-first) |
| [019/REQ-003](../../019-admin-money-safety-and-analytics/spec.md) | ADM-03/04 analytics label clarity (S3) |
| [019/REQ-004](../../019-admin-money-safety-and-analytics/spec.md) | ADM-05 interview row: inline Schedule + Job column (S3) |
| [019/REQ-005](../../019-admin-money-safety-and-analytics/spec.md) | review-queue pagination/filter (S3) |
| [019/REQ-006](../../019-admin-money-safety-and-analytics/spec.md) | admin nav at 390px (S3) |
| [019/REQ-007](../../019-admin-money-safety-and-analytics/spec.md) | humanise raw enum in review queue (S4) |

### 020-taxonomy-integrity

| Requirement | Source heading |
|---|---|
| [020/REQ-001](../../020-taxonomy-integrity/spec.md) | subject→level scoping, server (S2, HARD BLOCK) |
| [020/REQ-002](../../020-taxonomy-integrity/spec.md) | subject→level scoping, frontend picker (S2) |
| [020/REQ-003](../../020-taxonomy-integrity/spec.md) | canonical SG subject×level list (S3, FOUNDER DECISION) |

### 021-tutee-request-flow

| Requirement | Source heading |
|---|---|
| [021/REQ-001](../../021-tutee-request-flow/spec.md) | TUT-1 allow invite/interview on a PENDING post (S2, FOUNDER-CONFIRMED) |
| [021/REQ-002](../../021-tutee-request-flow/spec.md) | TUT-3 show the schedule in request-details (S3) |
| [021/REQ-003](../../021-tutee-request-flow/spec.md) | TUT-4 edit a PENDING post (S3) |

### 022-tutor-profile-and-fe-polish

| Requirement | Source heading |
|---|---|
| [022/REQ-001](../../022-tutor-profile-and-fe-polish/spec.md) | MOE-3 reconcile completeness banner (S3) |
| [022/REQ-002](../../022-tutor-profile-and-fe-polish/spec.md) | listing wizard pre-fill (S3) |
| [022/REQ-003](../../022-tutor-profile-and-fe-polish/spec.md) | availability chip spacing (S4) |
| [022/REQ-004](../../022-tutor-profile-and-fe-polish/spec.md) | commission line on landing (S4) |
| [022/REQ-005](../../022-tutor-profile-and-fe-polish/spec.md) | "6 or more" students drift check (S4, FOUNDER-FLAG) |

### 023-seed-data-variety

| Requirement | Source heading |
|---|---|
| [023/REQ-001](../../023-seed-data-variety/spec.md) | realistic seed variety (S2*, data) |
| [023/REQ-002](../../023-seed-data-variety/spec.md) | out-of-period dated row for ADM-02 (S2*, data) |

### 024-postfix-review-fixes

| Requirement | Source heading |
|---|---|
| [024/REQ-001](../../024-postfix-review-fixes/spec.md) | P1 invite-tutor 404 (S2, broken feature) |
| [024/REQ-002](../../024-postfix-review-fixes/spec.md) | P2 wizard Step-1 jam (S2) |
| [024/REQ-003](../../024-postfix-review-fixes/spec.md) | MOE-T3 list-card silent Apply (S2, money) |
| [024/REQ-004](../../024-postfix-review-fixes/spec.md) | MOE-T4 planning-areas required (S2, founder: REQUIRED) |
| [024/REQ-005](../../024-postfix-review-fixes/spec.md) | MOE-T2 band label (S2, label) |
| [024/REQ-006](../../024-postfix-review-fixes/spec.md) | OPS-01 deprecated MATH OLEVEL label (S3) |

### 025-taxonomy-stream-year-scoping

| Requirement | Source heading |
|---|---|
| [025/REQ-001](../../025-taxonomy-stream-year-scoping/requirements.md) | server-side stream×year scoping, single-pair surfaces (tutee-post + tutor profile matrix + tutor registration) |
| [025/REQ-002](../../025-taxonomy-stream-year-scoping/requirements.md) | server-side stream×year scoping, tutor-listing surface (multi-subject × multi-level set) |
| [025/REQ-003](../../025-taxonomy-stream-year-scoping/requirements.md) | frontend picker cascades to the precise valid levels per subject |
| [025/REQ-004](../../025-taxonomy-stream-year-scoping/requirements.md) | frontend-mirror drift check extended to the precise (subject, level) set |
| [025/REQ-005](../../025-taxonomy-stream-year-scoping/requirements.md) | backward-compatibility: validate-on-write only, no retro-invalidation |
| [025/REQ-006](../../025-taxonomy-stream-year-scoping/requirements.md) | no double-widen: a junction-scoped subject is validated by its precise rows ONLY |
| [025/REQ-007](../../025-taxonomy-stream-year-scoping/requirements.md) | fail-open completeness check: no Secondary subject silently falls back to permissive band-level |

### 026-s3s4-polish-batch

| Requirement | Source heading |
|---|---|
| [026/REQ-001](../../026-s3s4-polish-batch/spec.md) | show session duration + frequency (S3) |
| [026/REQ-002](../../026-s3s4-polish-batch/spec.md) | wizard phantom Step-4 (S4) |
| [026/REQ-003](../../026-s3s4-polish-batch/spec.md) | wizard back-nav preserves fields (S3) |
| [026/REQ-004](../../026-s3s4-polish-batch/spec.md) | P1–P4 level label (S4) |
| [026/REQ-005](../../026-s3s4-polish-batch/spec.md) | tutor handle display on browse (S3) |
| [026/REQ-006](../../026-s3s4-polish-batch/spec.md) | handle-collision on first profile save (S3) |
| [026/REQ-007](../../026-s3s4-polish-batch/spec.md) | analytics funnel tile reconciliation (S3) |
| [026/REQ-008](../../026-s3s4-polish-batch/spec.md) | minor polish (S4) |

### 027-aws-production-deployment

| Requirement | Source heading |
|---|---|
| [027/REQ-A0-1](../../027-aws-production-deployment/spec.md) | readiness health group excludes showcase (R6) |
| [027/REQ-A0-2](../../027-aws-production-deployment/spec.md) | graceful shutdown + consumer drain (R8) |
| [027/REQ-A0-3](../../027-aws-production-deployment/spec.md) | public /version provenance endpoint + GIT SHA + arm64 (R1/A7) |
| [027/REQ-A0-4](../../027-aws-production-deployment/spec.md) | migration exit-runner (R3) |
| [027/REQ-A0-5](../../027-aws-production-deployment/spec.md) | one-time passcode debug logging OFF in prod (R11) — SECURITY BLOCKER |
| [027/REQ-A0-6](../../027-aws-production-deployment/spec.md) | forward-headers / rate-limit integrity (R12) |
| [027/REQ-A0-7](../../027-aws-production-deployment/spec.md) | springdoc/swagger disabled in prod (R15) |
| [027/REQ-A0-8](../../027-aws-production-deployment/spec.md) | prod super-admin bootstrap runner (R14) |
| [027/REQ-A0-9](../../027-aws-production-deployment/spec.md) | connection + TLS hardening (R10/R20) |
| [027/REQ-A0-10](../../027-aws-production-deployment/spec.md) | bounded Redis streams (R16) |
| [027/REQ-INFRA](../../027-aws-production-deployment/spec.md) | Terraform infrastructure (Phases 0–6) |
| [027/REQ-CUTOVER](../../027-aws-production-deployment/spec.md) | go-live (Phase 6) |

## Original business rules

These preserve the business requirements document labels for traceability. Their titles are historical wording; for example BR-001 has been extended by the later post-assignment contact contract.

| Rule | Source |
|---|---|
| BR-001: Contact Information Visibility | [business requirements document](../../BRD.md) |
| BR-002: Post Approval Gate | [business requirements document](../../BRD.md) |
| BR-003: Admin-Only Matching | [business requirements document](../../BRD.md) |
| BR-004: Commission Calculation | [business requirements document](../../BRD.md) |
| BR-005: One Commission Per Match | [business requirements document](../../BRD.md) |
| BR-006: No Direct Platform Communication | [business requirements document](../../BRD.md) |
| BR-007: Registration Required to Post | [business requirements document](../../BRD.md) |
| BR-008: Post Lifecycle | [business requirements document](../../BRD.md) |
| BR-009: Commission Disclosure at Registration | [business requirements document](../../BRD.md) |
| BR-010: First-Session Confirmation Before Commission Invoice | [business requirements document](../../BRD.md) |
| BR-011: Interview Request Eligibility | [business requirements document](../../BRD.md) |
| BR-012: Interview Duration Limit | [business requirements document](../../BRD.md) |
| BR-013: Admin-Mediated Interview Scheduling | [business requirements document](../../BRD.md) |
| BR-014: Meeting Link Requirement | [business requirements document](../../BRD.md) |
| BR-015: Contact Privacy During Interview | [business requirements document](../../BRD.md) |

## Original feature inventory

product requirements document-Fnnn below is an audit-generated row label, not an identifier defined by the source. Priorities are the original product requirements document priorities, not a newly approved launch backlog. Later feature specs control explicit changes.

| Audit row | Area | Feature | Original priority | Source |
|---|---|---|---|---|
| product requirements document-F001 | 3.1 Authentication & Account Management | Tutee registration with email verification | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F002 | 3.1 Authentication & Account Management | Tutor registration with email verification | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F003 | 3.1 Authentication & Account Management | Role enforcement: mutually exclusive tutee/tutor roles | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F004 | 3.1 Authentication & Account Management | Admin login with time-based one-time passcode multi-factor authentication enforcement | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F005 | 3.1 Authentication & Account Management | Password reset flow | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F006 | 3.1 Authentication & Account Management | Account profile edit (own details, non-contact fields) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F007 | 3.1 Authentication & Account Management | Contact field edit (own email/phone, admin-only visible) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F008 | 3.1 Authentication & Account Management | Account deletion / data subject withdrawal | **P0** | [product requirements document](../../PRD.md) |
| product requirements document-F009 | 3.1 Authentication & Account Management | Admin user management (suspend, deactivate, ban) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F010 | 3.2 Tutor Commission Disclosure | Standalone commission disclosure screen at tutor registration (mandatory step) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F011 | 3.2 Tutor Commission Disclosure | Affirmative acknowledgement (checkbox) logged as system event | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F012 | 3.2 Tutor Commission Disclosure | Disclosure must include: rate, calculation method, worked example, trigger, payer | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F013 | 3.2 Tutor Commission Disclosure | Tutor profile not approvable by admin until acknowledgement is logged | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F014 | 3.3 Post Creation & Management | Tutee: create tuition request post (all fields per §5.2) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F015 | 3.3 Post Creation & Management | Tutor: create tuition offering post (all fields per §5.1) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F016 | 3.3 Post Creation & Management | Post save as draft before submission | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F017 | 3.3 Post Creation & Management | Submit post for admin review | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F018 | 3.3 Post Creation & Management | Tutee: edit own post (when in Draft or Revision Requested state) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F019 | 3.3 Post Creation & Management | Tutor: edit own post (when in Draft or Revision Requested state) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F020 | 3.3 Post Creation & Management | Tutee/Tutor: close own active post | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F021 | 3.3 Post Creation & Management | Tutee/Tutor: view own post status and history | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F022 | 3.3 Post Creation & Management | Auto-expiry of unmatched active posts after configurable period (default 60 days) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F023 | 3.3 Post Creation & Management | System notification on auto-expiry | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F024 | 3.3 Post Creation & Management | Multiple active posts per tutor (tutor may hold several active offering posts) | P1 | [product requirements document](../../PRD.md) |
| product requirements document-F025 | 3.4 Admin Post Moderation | Admin review queue (all pending posts, sorted by submission time) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F026 | 3.4 Admin Post Moderation | Per-post view including full contact info (admin only) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F027 | 3.4 Admin Post Moderation | Approve action | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F028 | 3.4 Admin Post Moderation | Reject action with mandatory reason from enum | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F029 | 3.4 Admin Post Moderation | Request Revision action with message | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F030 | 3.4 Admin Post Moderation | Immutable audit log entry on every contact field access | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F031 | 3.4 Admin Post Moderation | Post revision re-enters approval queue | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F032 | 3.4 Admin Post Moderation | Internal notes per post (admin comments for handoff) | P1 | [product requirements document](../../PRD.md) |
| product requirements document-F033 | 3.4 Admin Post Moderation | Bulk action on review queue (approve/reject multiple) | P2 | [product requirements document](../../PRD.md) |
| product requirements document-F034 | 3.5 Public Tutor Listing (Unauthenticated Browse) | Publicly browseable approved tutor listings (no contact info) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F035 | 3.5 Public Tutor Listing (Unauthenticated Browse) | Filter by subject, level, venue type, location area | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F036 | 3.5 Public Tutor Listing (Unauthenticated Browse) | Individual tutor post detail page (no contact info shown) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F037 | 3.5 Public Tutor Listing (Unauthenticated Browse) | Pagination of listing results | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F038 | 3.5a Tuition Rate Guide (Public Reference Page) | Rate guide page at /rates (publicly accessible) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F039 | 3.5a Tuition Rate Guide (Public Reference Page) | Rate table by education level: Pre-school, Lower Primary, Upper Primary, Lower Sec, Upper Sec, JC, IB, Poly/Uni | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F040 | 3.5a Tuition Rate Guide (Public Reference Page) | Rate breakdown by tutor type: Part-time (undergrad/student), Full-time freelancer, Ex-MOE/school teacher | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F041 | 3.5a Tuition Rate Guide (Public Reference Page) | Reference data from market (Smile Tutor, Mind Flex benchmarks) with "actual rates may vary" disclaimer | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F042 | 3.5a Tuition Rate Guide (Public Reference Page) | CTA row: "Browse Tutors" (public) and "Post a Request" (login-gated) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F043 | 3.5a Tuition Rate Guide (Public Reference Page) | FAQ section: 3–5 common rate questions (why rates vary, group vs 1-on-1, ex-MOE premium) | P1 | [product requirements document](../../PRD.md) |
| product requirements document-F044 | 3.5a Tuition Rate Guide (Public Reference Page) | Link from Homepage stats section and from tutor/tutee post creation forms | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F045 | 3.6 Tutee Request Discovery (Authenticated Tutors) | Approved tutors (logged in) can browse anonymised tutee request posts | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F046 | 3.6 Tutee Request Discovery (Authenticated Tutors) | Tutee posts shown in discovery index only when in Active state (not Matched, Closed, etc.) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F047 | 3.6 Tutee Request Discovery (Authenticated Tutors) | No contact info shown on tutee discovery posts (name shown as pseudonym only) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F048 | 3.6 Tutee Request Discovery (Authenticated Tutors) | Filter tutee requests by subject, level, venue, area, budget range | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F049 | 3.6 Tutee Request Discovery (Authenticated Tutors) | Free-market apply — tutor applies to any ACTIVE job, NO published listing required | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F050 | 3.6 Tutee Request Discovery (Authenticated Tutors) | Apply gated only by profile completeness (rate/area/PT-FT/qualifications) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F051 | 3.7 Admin Matching Dashboard | Two-panel matching dashboard: tutee requests (left) + tutor offerings (right) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F052 | 3.7 Admin Matching Dashboard | Filter both panels by subject, level, venue type, area/MRT, budget/rate overlap | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F053 | 3.7 Admin Matching Dashboard | Create Match action linking one tutee post to one tutor post | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F054 | 3.7 Admin Matching Dashboard | Cardinality enforcement: tutee post → max 1 pending/active match | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F055 | 3.7 Admin Matching Dashboard | Tutor post → multiple simultaneous matches permitted | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F056 | 3.7 Admin Matching Dashboard | Match lifecycle status tracking: PENDING → ACTIVE → CLOSED/CANCELLED | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F057 | 3.7 Admin Matching Dashboard | Admin cancel match action | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F058 | 3.7 Admin Matching Dashboard | Matched tutee post removed from tutor discovery index | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F059 | 3.7 Admin Matching Dashboard | Re-entry of tutee post into discovery index on match cancellation | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F060 | 3.7 Admin Matching Dashboard | Admin internal notes per match | P1 | [product requirements document](../../PRD.md) |
| product requirements document-F061 | 3.7 Admin Matching Dashboard | Parent-facing applicant view (personal information-free) — parent sees who applied to their own job | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F062 | 3.7 Admin Matching Dashboard | Parent "choose tutor" selection request → admin queue | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F063 | 3.7 Admin Matching Dashboard | Admin "close job" = atomic assign (winner→MATCHED, others→SUPERSEDED, job→MATCHED) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F064 | 3.7 Admin Matching Dashboard | Reopen on cancel (job→ACTIVE, superseded apps→APPLIED) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F065 | 3.7 Admin Matching Dashboard | Stuck-PENDING-commission admin view + age alert | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F066 | 3.8 Commission Workflow | Commission calculation displayed when admin marks match ACTIVE | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F067 | 3.8 Commission Workflow | Tutee first-session confirmation email with action link | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F068 | 3.8 Commission Workflow | 7-day timeout notification and admin alert if tutee does not confirm | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F069 | 3.8 Commission Workflow | Admin override to advance commission to CONFIRMED | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F070 | 3.8 Commission Workflow | Commission invoice generation (admin-triggered after CONFIRMED) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F071 | 3.8 Commission Workflow | Invoice fields: tutor display name, tutee initials, subject/level, monthly fee, rate, amount, due date | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F072 | 3.8 Commission Workflow | Admin marks commission PAID manually | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F073 | 3.8 Commission Workflow | Commission status: PENDING / AWAITING CONFIRMATION / CONFIRMED / INVOICED / PAID / DISPUTED / WAIVED | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F074 | 3.8 Commission Workflow | Commission status visible to tutor (amount + status only, per Appendix D) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F075 | 3.8 Commission Workflow | Commission DISPUTED and WAIVED status workflow user interface | **P1** | [product requirements document](../../PRD.md) |
| product requirements document-F076 | 3.8 Commission Workflow | In-platform payment collection | — | [product requirements document](../../PRD.md) |
| product requirements document-F077 | 3.9 Notifications | Email notification on all 15+ trigger events (see §9 Notification Plan) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F078 | 3.9 Notifications | In-platform notification inbox (bell icon) | P1 | [product requirements document](../../PRD.md) |
| product requirements document-F079 | 3.9 Notifications | SMS notifications | P2 | [product requirements document](../../PRD.md) |
| product requirements document-F080 | 3.10 Security & Compliance | personal information encryption at rest (email, phone — deterministic AES-SIV or equivalent) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F081 | 3.10 Security & Compliance | Contact fields excluded from all public application programming interface responses | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F082 | 3.10 Security & Compliance | Personal Data Protection Act-compliant consent language at registration | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F083 | 3.10 Security & Compliance | Data residency: all data in Singapore region | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F084 | 3.10 Security & Compliance | role-based access control enforcement server-side | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F085 | 3.10 Security & Compliance | Immutable audit log for contact field access (separate store, 5-year retention) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F086 | 3.10 Security & Compliance | Commission acknowledgement log (write-once, 5-year retention) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F087 | 3.10 Security & Compliance | Mobile-first user interface (tutee and tutor flows, 390px primary viewport) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F088 | 3.10 Security & Compliance | Account deletion / right to erasure flow | **P0** | [product requirements document](../../PRD.md) |
| product requirements document-F089 | 3.10 Security & Compliance | Data export on request (Personal Data Protection Act right of access) | P1 | [product requirements document](../../PRD.md) |
| product requirements document-F090 | 3.10 Security & Compliance | DPO sign-off on data governance before go-live | P0 (gate) | [product requirements document](../../PRD.md) |
| product requirements document-F091 | 3.11 Public Content Pages | Landing page with value proposition and CTAs | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F092 | 3.11 Public Content Pages | How It Works pages (tutee flow, tutor flow) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F093 | 3.11 Public Content Pages | About page | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F094 | 3.11 Public Content Pages | FAQ page | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F095 | 3.11 Public Content Pages | Contact page | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F096 | 3.11 Public Content Pages | Privacy Policy page | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F097 | 3.11 Public Content Pages | Terms & Conditions page | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F098 | 3.12 Interview Sessions (USP) | Tutee can request interview from approved tutor post detail page | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F099 | 3.12 Interview Sessions (USP) | Interview concurrency limit: max 3 open requests per tutee at any time | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F100 | 3.12 Interview Sessions (USP) | Interview cooldown: no new request for same tutor post within 30 days of completed/no-show | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F101 | 3.12 Interview Sessions (USP) | Admin interview queue: all REQUESTED interviews sorted by submission time | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F102 | 3.12 Interview Sessions (USP) | Admin schedules interview: sets date/time and meeting link (any URL) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F103 | 3.12 Interview Sessions (USP) | Meeting link storage and delivery to both tutee and tutor via platform notification | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F104 | 3.12 Interview Sessions (USP) | Interview duration display: 15 minutes stated in notification copy (not technically enforced) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F105 | 3.12 Interview Sessions (USP) | Admin updates meeting link before scheduled time | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F106 | 3.12 Interview Sessions (USP) | Admin marks outcome: COMPLETED / CANCELLED / NO SHOW | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F107 | 3.12 Interview Sessions (USP) | Tutee: my interviews list (status badge, meeting link for SCHEDULED) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F108 | 3.12 Interview Sessions (USP) | Tutor: my interview requests list (date/time, meeting link, tutee request summary — no contact info) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F109 | 3.12 Interview Sessions (USP) | No contact information shared through interview workflow | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F110 | 3.12 Interview Sessions (USP) | Interview analytics events (request, schedule, complete, no-show, conversion-to-match) | P0 | [product requirements document](../../PRD.md) |
| product requirements document-F111 | 3.12 Interview Sessions (USP) | Admin: update interview record (reschedule link, change time) | P1 | [product requirements document](../../PRD.md) |
| product requirements document-F112 | 3.12 Interview Sessions (USP) | Automated reminder to both parties 24h before scheduled interview | P1 | [product requirements document](../../PRD.md) |

## Original acceptance-criteria headings

| Acceptance criterion | Source |
|---|---|
| AC-01: Contact Information Never Exposed to Non-Admin Users (BR-001) | [product requirements document](../../PRD.md) |
| AC-02: Post Approval Gate (BR-002) | [product requirements document](../../PRD.md) |
| AC-03: Commission Disclosure at Tutor Registration (BR-009) | [product requirements document](../../PRD.md) |
| AC-04: Admin-Only Matching (BR-003) | [product requirements document](../../PRD.md) |
| AC-05: First-Session Confirmation Before Commission Invoice (BR-010) | [product requirements document](../../PRD.md) |
| AC-06: Tutor Discovery Index — Anonymisation (BR-001, BR-003) | [product requirements document](../../PRD.md) |
| AC-07: Admin multi-factor authentication Enrollment Enforcement | [product requirements document](../../PRD.md) |
| AC-08: Audit Log Immutability on Contact Field Access | [product requirements document](../../PRD.md) |
| AC-09: Admin Session Idle Timeout | [product requirements document](../../PRD.md) |
| AC-10: Terms Version Schema for Commission Acknowledgement | [product requirements document](../../PRD.md) |
| AC-11: Reminder Email Timing (N-12) | [product requirements document](../../PRD.md) |
| AC-12: Tutee Pseudonym Format on Invoice | [product requirements document](../../PRD.md) |
| AC-13: First-Session Confirmation Endpoint Idempotency | [product requirements document](../../PRD.md) |
| AC-14: One Commission Per Match — BR-005 | [product requirements document](../../PRD.md) |
| AC-15: No Direct Messaging — BR-006 | [product requirements document](../../PRD.md) |
| AC-16: Interview Request — Eligibility Guards (BR-011) | [product requirements document](../../PRD.md) |
| AC-17: Interview Scheduling — Meeting Link Required (BR-013, BR-014) | [product requirements document](../../PRD.md) |
| AC-18: Interview — No Contact Info Shared (BR-015) | [product requirements document](../../PRD.md) |
| AC-19: Interview Outcome Tracking (BR-013) | [product requirements document](../../PRD.md) |

## Additional requirement sources

- [product requirements document non-functional requirements](../../PRD.md): availability, notification timing, storage, latency, and rate limiting.
- [product requirements document M18 extension](../../PRD.md): free-market applications, parent selection, atomic assignment, contact reveal, and operational acceptance.
- [Module index](../../modules/INDEX.md): auth, posting, approval, matching, commission, interviews, profiles, dashboards, notifications, favourites, reviews, analytics, showcases, applications, admin management.
- [Sequence index](../../sequences/INDEX.md): 43 indexed flows; historical diagrams must be reconciled with later feature specs.
- [Private supporting material excluded (publication note)](../../../docs/publication-notes.md): assignment, invitations, negotiation, quotas, profile matrix, fixed commission, security, messaging, and infrastructure amendments.
- [business requirements document retention and role matrices](../../BRD.md): data lifecycle and permissions.

The catalog intentionally links to the source rather than silently rewriting old requirements into new rules.
