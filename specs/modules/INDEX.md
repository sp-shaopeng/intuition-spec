# Modules — Master Index

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

This is the module- and use-case-level map of the InTuition platform. The product
shipped as 12 modules (M1–M12), plus M15 Applications, M14a Active-Tutors showcase,
M14b Recent-Requests showcase, M16 Admin Management, and M17 Marketplace user experience overhaul
— all DEPLOYED (schema now at migration **0031**; M17 carried no schema change; M18 added
0025–0030; **0031 = INT-015 post soft-delete-close, 2026-06-03**). Ten
modules carry dedicated spec folders under specs/modules/ (M1–M6, M15, M16,
landing-showcase, plus profile account-user interface notes); the remaining modules are
documented through use-case **sequence diagrams** under
[sequences implementation reference](../sequences/INDEX.md) plus the endpoint and route references
under [Private supporting material excluded (publication note)](../../docs/publication-notes.md).

> **Fix batch 2026-06-03 (INT-015…042):** M4/M5 match-cancel + contact-reveal 500 FIXED (INT-016, Sev-1);
> M2 REJECTED-post delete 500 FIXED (INT-015, migration 0031); M6 schedule gains an optional meeting Link
> in the same step (INT-035); M15 admin "View applicants" repointed to an admin route (INT-033); plus the
> M17/Nav role-aware user experience batch. See issues/README md, the updated sequence flows (12/14/32/39), and
> BATCH-FIX-REVIEW-2026-06-03.md. Carry-overs: INT-046 (frontend coverage debt), INT-047 (local Docker build).

## Module map

| Module | Name | Status | Module folder | Use-case flows |
|---|---|---|---|---|
| M1 | Auth | Complete | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) | [01](../sequences/01-register-tutee.md), [02](../sequences/02-register-tutor.md), [03](../sequences/03-login-with-mfa.md), [04](../sequences/04-forgot-reset-password.md), [05](../sequences/05-csrf-bootstrap.md), [29](../sequences/29-change-password.md), [30](../sequences/30-account-deletion.md) |
| M2 | Posting | Complete | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) | [06](../sequences/06-tutee-post-creation.md), [07](../sequences/07-tutor-post-creation.md), [32](../sequences/32-post-edit-and-close.md) |
| M3 | Approval | Complete | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) | [08](../sequences/08-admin-approval.md) |
| M4 | Matching | Complete | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) | [09](../sequences/09-admin-match-creation.md), [10](../sequences/10-admin-match-activation.md), [12](../sequences/12-admin-match-cancel-cascade.md) |
| M5 | Commission | Complete | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) | [11](../sequences/11-admin-mark-commission-paid.md), [12](../sequences/12-admin-match-cancel-cascade.md) |
| M6 | Interview (+ max-3-active-per-posting cap, migration 0021) | Complete + deployed | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) ([cap note](interview/max-interviews-per-posting.md)) | [13](../sequences/13-tutee-request-interview.md), [14](../sequences/14-admin-schedule-interview.md), [15](../sequences/15-admin-set-meeting-link.md), [16](../sequences/16-interview-completion-paths.md), [35](../sequences/35-tutee-cancel-interview.md) |
| M7 | Profile | Complete |  using reference only_ | [33](../sequences/33-profile-edit.md) (see also [backend-api](../reference/backend-api.md) §M7, [frontend-routes](../reference/frontend-routes.md)) |
| M8 | Match Dashboard | Complete |  using sequences only_ | [17](../sequences/17-tutee-views-matches.md), [18](../sequences/18-tutor-views-matches.md) |
| M9 | Notification Centre | Complete |  using sequences only_ | [21](../sequences/21-notification-bell-flow.md), [26](../sequences/26-email-fanout-on-event.md) |
| M10 | Search & Favourites | Complete |  using sequences only_ | [22](../sequences/22-favourite-toggle.md), [23](../sequences/23-public-browse-with-filters.md) |
| M11 | Review & Rating | Complete |  using sequences only_ | [19](../sequences/19-tutee-submits-review.md), [20](../sequences/20-public-tutor-detail-reviews.md) |
| M12 | Admin Analytics | Complete (verification pending) |  using reference only_ | — (see [backend-api](../reference/backend-api.md) §M12) |
| M14a | Active Tutors showcase (public pre-login strip; tutor "Available" flag + featured consent; personal information-free; **no applicant count**) | Deployed (migration 0019) | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) | [34](../sequences/34-tutor-availability-toggle.md) |
| M15 | Applications (tutor → job apply/withdraw; applicant count is **admin-only**, not shown publicly) | Deployed (migration 0018) | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) | [28](../sequences/28-tutor-apply-withdraw.md) |
| M14b | Recent Job Requests showcase strip — public pre-login strip of anonymised open jobs (no title/contact/exact date; bucketed budget; coarse start-window; planning-area location); dual-audience landing. Mirrors the M14a Active-Tutors strip 1:1 (same cache / fail-closed / rate-limit / CLS posture; consent gate **waived by founder 2026-05-30**). No applicant count in either showcase slice. | Deployed (migration 0023) | [../architecture/m14b-public-job-postings-showcase md](../architecture/m14b-public-job-postings-showcase.md) | — |
| M16 | Admin Management & Super-Admin (any admin creates admins; super-admin protected; seeded REDACTED email local-only; password change required gate; no multi-factor authentication) | Deployed (migration 0020) | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) | [29](../sequences/29-change-password.md), [31](../sequences/31-admin-create-delete-admin.md) |
| M17 | Marketplace user experience overhaul (+ M14b regression fix) — public Browse Assignments page (/assignments, paginated anonymised sibling of the requests showcase, read operation for assignments); auth CTAs default to /login (register only on explicit create-account); detail pages require login server-side (/tutors/:id moves to authenticated read operation for tutors id, new authenticated read operation for assignments id; /public/tutors/** glob narrowed to LIST, reviews kept public); tutors-showcase Transaction Template fix (BUG-1, self-invocation lazy-init); friendly subject/level labels; Nav intent labels ("Find a Tutor" / "Find Tuition Jobs") + mobile parity; landing order Jobs → Tutors. | Deployed (migration: none) | [../architecture/m17-marketplace-ux-overhaul md](../architecture/m17-marketplace-ux-overhaul.md) | — (related: [20](../sequences/20-public-tutor-detail-reviews.md), [23](../sequences/23-public-browse-with-filters.md)) |
| M18 | Assignment Lifecycle & Free Marketplace — free-market apply (no listing required); tutor profile as canonical identity (handle/commitment/rate/area); atomic assign (winner→MATCHED, others→SUPERSEDED, job→MATCHED, optimistic-lock 409, cancel→reopen); parent applicant view (personal information-free) + choose-tutor selection requests; contact reveal post-assignment. **backend + frontend built**: admin §G2 selection queue/assign, §G3 Matching lifecycle, §G4 stale-commission; tutor/parent profile/cards/apply/applicants/contact. backend Admin Selection Request Response tutor Post Id enrichment for the assign payload. | Deployed (dev), changesets 0025–0031; **INT-016 (cancel/contact-reveal Sev-1) FIXED**. Extended 2026-06-03 by the feature wave — **INT-034** free-market assign real fix /tutor user id (0033), **INT-039** interview timeslot negotiation (0034), **INT-044** job invitations (0035), **INT-025** admin notify (0036), **INT-045** analytics range (0037), **INT-043** onboarding — + **INT-049** all-pages user interface redesign. All deployed to dev; see [../work-plan md](../work-plan.md). Not in prod — ROADMAP PDO-3 | [../architecture/assignment-be-spec md](../architecture/assignment-be-spec.md), [../architecture/assignment-fe-spec md](../architecture/assignment-fe-spec.md), [../architecture/admin-matching-fe-spec md](../architecture/admin-matching-fe-spec.md), [../architecture/int-034-free-market-assign md](../architecture/int-034-free-market-assign.md), [../architecture/int-039-interview-negotiation md](../architecture/int-039-interview-negotiation.md), [../architecture/int-044-job-invitations md](../architecture/int-044-job-invitations.md), [../uiux/all-pages-redesign-spec md](../uiux/all-pages-redesign-spec.md) | [28](../sequences/28-tutor-apply-withdraw.md), [42](../sequences/42-interview-timeslot-negotiation.md), [43](../sequences/43-tutee-tutor-job-invitation.md) |

### Proposed / in design / deferred

| Module | Name | Status | Folder |
|---|---|---|---|
| M13 | Operations / People Directory + Secure personal information | Deferred + hard-gated (thin slice, decrypt-on-the-fly) | [../architecture/secure-pii-and-ops-console md](../architecture/secure-pii-and-ops-console.md) |

Review record: [../architecture/reviews/M13-M14-M15-team-review md](../architecture/reviews/M13-M14-M15-team-review.md)

Cross-cutting platform flows (not a single module): [24 Rate-limited endpoint](../sequences/24-rate-limited-endpoint.md),
[25 cross-site request forgery protection-protected write](../sequences/25-csrf-protected-write.md),
[27 Encrypted personal information write + lookup](../sequences/27-encrypted-pii-write-and-lookup.md).

## Documentation depth per module

Module folders carry between 1 and 10 files. The fullest modules (M3–M5) carry the
complete set; M1/M2/M6/M15 carry the core four (spec, backend, frontend, db); newer
slices (M16, landing-showcase, the M6 cap note, profile account-user interface note) carry a
focused single- or dual-doc set. The standard file set is:

| File | Purpose |
|---|---|
| module-overview md | Module summary, actors, use-case list (M3–M5) |
| spec md | Functional + technical specification (all) |
| backend md | Backend implementation detail (all) |
| frontend md | Frontend implementation detail (all) |
| db md | Schema and migrations for the module (all) |
| api-reference md | Endpoint reference (M3–M5) |
| er-diagram md | Entity-relationship diagram (M3–M5) |
| sequence-diagrams md | Module-local sequence diagrams (M3–M5) |
| debugging-guide md | Common issues and debugging (M3–M5) |

| Module | Files present |
|---|---|
| auth (M1) | spec, backend, frontend, db |
| posting (M2) | spec, backend, frontend, db |
| approval (M3) | full set + reviewer-notes md |
| matching (M4) | full set |
| commission (M5) | full set |
| interview (M6) | spec, backend, frontend, db, [max-interviews-per-posting](interview/max-interviews-per-posting.md) (M6 cap) |
| applications (M15) | spec, backend, frontend, db |
| landing-showcase (M14a/M14b) | [spec](landing-showcase/spec.md), [design](landing-showcase/design.md) |
| admin-management (M16) | [spec](admin-management/spec.md) |
| profile (M7) |  using reference only_ + [account-ui-changes](profile/account-ui-changes.md) |

## See also

- Use-case sequence diagrams: [../sequences/INDEX md](../sequences/INDEX.md)
- Backend endpoint reference (M1–M12): [../reference/backend-api md](../reference/backend-api.md)
- Frontend route reference (39 pages): [../reference/frontend-routes md](../reference/frontend-routes.md)
- Postman collection guide: [../reference/postman md](../reference/postman.md)
- How to run the system: [../ops/runbook md](../ops/runbook.md)
