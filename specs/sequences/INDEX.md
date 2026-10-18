# Sequence Diagram Index

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Engineer-facing Mermaid sequence diagrams covering every key flow in InTuition SG.
All diagrams are derived from real controller/service/page source — line references
are provided in each Notable details section.

> **Coverage (2026-06-02, INT-022 close-out):** all 24 production controllers now have a
> flow. Flows 36–41 were added to cover the M14/M17/M18 surfaces that previously lagged —
> marketplace detail, contact reveal, choose-tutor selection, admin applicant view, public
> showcase strips, and admin analytics. The two the profile operation regression-only
> controllers (Regression Auth Controller, Regression Manifest Controller) are intentionally
> out of scope (test harness, not product). Cross-checked against reference/backend-api md.

> **Updated 2026-06-03 (INT-015…047 fix batch):** flows refreshed to the as-shipped behaviour —
> **12** (match-cancel: the INT-016 Sev-1 fix + the M18 reopen + cancel notification), **14** (schedule:
> optional meeting Link set in the same step — INT-035), **32** (REJECTED-post soft-delete now works via
> migration 0031 — INT-015), **39** (admin "View applicants" repointed to an admin route — INT-033).
> The Tier-A nav/user experience fixes (INT-032/036/037/038/040/041/042) are frontend-state and are documented in
> reference/frontend-routes md rather than as new sequence flows.

> **Updated 2026-06-03 (wave INT-034/039/044/045/025):** flows **42** and **43** added to cover
> interview timeslot negotiation (INT-039) and tutee→tutor job invitations (INT-044).
> The controller count is now 26. reference/backend-api md updated for all five wave items.

Diagrams render in any Mermaid live editor (e g. https://mermaid live). Use
sequence Diagram for request/response flows and state Diagram-v2 for status
machines (added where the lifecycle has multiple terminal states).

## Auth

| # | Flow | File |
| - | ---- | ---- |
| 01 | Tutee registration — single-step register + signed session token cookie issuance | [01-register-tutee md](01-register-tutee.md) |
| 02 | Tutor registration — Register → Commission stash → POST | [02-register-tutor md](02-register-tutor.md) |
| 03 | Login (and admin multi-factor authentication time-based one-time passcode second step) | [03-login-with-mfa md](03-login-with-mfa.md) |
| 04 | Forgot-password one-time passcode issue + reset | [04-forgot-reset-password md](04-forgot-reset-password.md) |
| 05 | cross-site request forgery protection bootstrap + /me hydration on app boot | [05-csrf-bootstrap md](05-csrf-bootstrap.md) |
| 29 | Change password (authenticated; clears admin forced-change gate) | [29-change-password md](29-change-password.md) |
| 30 | Account deletion (soft-delete + signed session token denylist; Personal Data Protection Act erasure) | [30-account-deletion md](30-account-deletion.md) |

## Posting + approval

| # | Flow | File |
| - | ---- | ---- |
| 06 | Tutee creates a request → PENDING | [06-tutee-post-creation md](06-tutee-post-creation.md) |
| 07 | Tutor creates a listing → PENDING | [07-tutor-post-creation md](07-tutor-post-creation.md) |
| 08 | Admin review queue → approve / reject + post status machine | [08-admin-approval md](08-admin-approval.md) |
| 32 | Post lifecycle: owner edits (PUT) + closes (soft DELETE → CLOSED) | [32-post-edit-and-close md](32-post-edit-and-close.md) |

## Matching + commission

| # | Flow | File |
| - | ---- | ---- |
| 09 | Admin creates a match (PENDING) | [09-admin-match-creation md](09-admin-match-creation.md) |
| 10 | Admin activates match → Commission row | [10-admin-match-activation md](10-admin-match-activation.md) |
| 11 | Admin marks commission paid → match COMPLETED | [11-admin-mark-commission-paid md](11-admin-mark-commission-paid.md) |
| 12 | Admin cancels match + PENDING commission cascade + match state machine | [12-admin-match-cancel-cascade md](12-admin-match-cancel-cascade.md) |

## Interview module (M6 + INT-039)

| # | Flow | File |
| - | ---- | ---- |
| 13 | Tutee requests interview from Tutor Detail modal (pre-INT-039 reference) | [13-tutee-request-interview md](13-tutee-request-interview.md) |
| 14 | Admin schedules a PENDING SCHEDULING / REQUESTED interview (Singapore time wall-clock) | [14-admin-schedule-interview md](14-admin-schedule-interview.md) |
| 15 | Admin sets meeting link | [15-admin-set-meeting-link md](15-admin-set-meeting-link.md) |
| 16 | Three terminal paths (complete / cancel / no-show) + interview state machine | [16-interview-completion-paths md](16-interview-completion-paths.md) |
| 35 | Tutee cancels their own REQUESTED/SCHEDULED interview | [35-tutee-cancel-interview md](35-tutee-cancel-interview.md) |
| 42 | **INT-039** Interview timeslot negotiation (propose→accept/decline/counter→PENDING SCHEDULING→admin schedule + expiry sweep) | [42-interview-timeslot-negotiation md](42-interview-timeslot-negotiation.md) |

## Match dashboard + reviews

| # | Flow | File |
| - | ---- | ---- |
| 17 | Tutee views matches (admin contact only on ACTIVE) | [17-tutee-views-matches md](17-tutee-views-matches.md) |
| 18 | Tutor views matches (symmetric, with commission) | [18-tutor-views-matches md](18-tutor-views-matches.md) |
| 19 | Tutee submits review → notify reviewee → rating aggregate refresh | [19-tutee-submits-review md](19-tutee-submits-review.md) |
| 20 | Public Tutor Detail + reviews fetch | [20-public-tutor-detail-reviews md](20-public-tutor-detail-reviews.md) |

## Profile + admin management

| # | Flow | File |
| - | ---- | ---- |
| 33 | Profile edit (tutee/tutor GET+PUT; role-scoped) | [33-profile-edit md](33-profile-edit.md) |
| 34 | Tutor availability / showcase consent toggle (PATCH featured-consent) | [34-tutor-availability-toggle md](34-tutor-availability-toggle.md) |
| 31 | Admin create + delete admin (super-admin-gated delete) | [31-admin-create-delete-admin md](31-admin-create-delete-admin.md) |

## Notifications + favourites + search

| # | Flow | File |
| - | ---- | ---- |
| 21 | Notification Bell 60s polling + role-aware navigation + mark read | [21-notification-bell-flow md](21-notification-bell-flow.md) |
| 22 | Favourite toggle: optimistic heart + bulk ids Among lookup | [22-favourite-toggle md](22-favourite-toggle.md) |
| 23 | Public tutor browse with filters (subject, level, region, rate, venue, group size, availability) | [23-public-browse-with-filters md](23-public-browse-with-filters.md) |

## Applications module (M15 + INT-044)

| # | Flow | File |
| - | ---- | ---- |
| 28 | Tutor applies to / withdraws from a job (idempotent apply, withdraw → re-apply) | [28-tutor-apply-withdraw md](28-tutor-apply-withdraw.md) |
| 39 | Admin applicant view (admin-only list backing matching) | [39-admin-applicant-view md](39-admin-applicant-view.md) |
| 43 | **INT-044** Tutee→tutor job invitation (invite→accept folds into apply→choose funnel / decline / withdraw / expire) | [43-tutee-tutor-job-invitation md](43-tutee-tutor-job-invitation.md) |

## Marketplace detail + assignment lifecycle (M17 / M18)

| # | Flow | File |
| - | ---- | ---- |
| 36 | Login-gated tutor / assignment **detail** (M17 SEC-1; lists stay public) | [36-marketplace-detail-login-gated md](36-marketplace-detail-login-gated.md) |
| 38 | Choose-tutor selection → admin selection queue → assign (M18) | [38-choose-tutor-selection md](38-choose-tutor-selection.md) |
| 37 | Post-assignment contact reveal (the only counterparty-personal information surface; INT-016 fixed 2026-06-03) | [37-contact-reveal-post-assignment md](37-contact-reveal-post-assignment.md) |

## Landing showcase (M14a / M14b) + admin analytics (M12)

| # | Flow | File |
| - | ---- | ---- |
| 40 | Public showcase strips — Active Tutors + Recent Requests (cached, rate-limited, personal information-free) | [40-public-showcase-strips md](40-public-showcase-strips.md) |
| 41 | Admin analytics dashboard (count/funnel; ADMIN-only) | [41-admin-analytics md](41-admin-analytics.md) |

## Cross-cutting

| # | Flow | File |
| - | ---- | ---- |
| 24 | Rate-limited login (Lua sliding window) | [24-rate-limited-endpoint md](24-rate-limited-endpoint.md) |
| 25 | cross-site request forgery protection-protected write (double-submit cookie) | [25-csrf-protected-write md](25-csrf-protected-write.md) |
| 26 | Email queue fanout (notification + Redis Streams) | [26-email-fanout-on-event md](26-email-fanout-on-event.md) |
| 27 | Encrypted personal information write + keyed message authentication lookup | [27-encrypted-pii-write-and-lookup md](27-encrypted-pii-write-and-lookup.md) |

## How these were sourced

Every diagram was derived by reading the actual code — controllers in
intuition-the private implementation folder, services in intuition-the private implementation folder,
infrastructure in intuition-the private implementation folder, security in
intuition-the private implementation folder, and frontend pages in
intuition-the private implementation folder Where the spec implied behaviour the code does
not yet deliver, the gap is called out explicitly under Notable details or
Failure modes.

Known gaps surfaced while authoring:

- Login implementation reference now **detects** the confirmation that multi-factor authentication is required response (INT-024, 2026-06-02) and shows an
  accurate message instead of "Invalid email or password". **Remaining gap:** the full time-based one-time passcode entry user interface
  (submit operation for auth login mfa) is not built — an multi-factor authentication-enabled admin still can't complete login via
  the user interface (latent; no seeded admin has multi-factor authentication). See flow 03.
- ~~Tutee Post Service create And Submit and Tutor Post Service create And Submit
  emit analytics events but do **not** emit Notification Type records to admins.~~
  **Resolved (INT-025, 2026-06-03):** POST SUBMITTED enum value added (migration 0036);
  both services call the emit to all admins operation after-commit. See
  reference/backend-api md §M9 INT-025 section.
- ~~The Redis Stream email-notifications has no consumer wired into the backend
  source tree.~~ **Resolved (2026-06-02):** Email Consumer Service implementation reference
  now exists in-tree — a a scheduled task XREADGROUP consumer with XCLAIM crash-recovery
  that dispatches via Resend Email Client (see flow 26). Remaining gaps are
  production-hardening, not wiring: no failed-message queue / retry-counter and no verified sender
  domain (sends from REDACTED email) — see architecture/email-consumer-hardening md.
