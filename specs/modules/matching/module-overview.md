# Module 4 — Admin Matching

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** This is a dated implementation snapshot. [INT-034](../../architecture/int-034-free-market-assign.md) supersedes mandatory paired listings with canonical tutor user id and optional listing; M18/INT-125 add assigned-participant contact. Historical not-yet/not-present statements here are not evidence of a current missing feature. Agency receipt is the PAID event. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


## Purpose and Business Context

The Matching module gives agency administrators a single-screen workflow to manually pair an approved tutor post with an approved tutee request.  A match is the central commercial event in the platform: it signals that the agency has brokered a relationship and triggers the commission lifecycle.

In the current implementation the admin selects one tutor post and one tutee request from side-by-side scrollable lists, reviews the pairing in a summary bar, and clicks **Create Match**.  On success the user interface transitions to a confirmation screen and the matches row is written to the database.  Contact details are not automatically shared; the admin is expected to introduce the parties out-of-band.

---

## Feature List

| Feature | Status |
|---|---|
| Two-column dashboard (tutor posts / tutee requests) | Implemented |
| Subject and level filter dropdowns (re-fetches on change) | Implemented |
| Tutor card — subjects, levels, rate, location | Implemented |
| Tutee card — subject/level, budget, venue, availability | Implemented |
| Toggle-select cards (click again to deselect) | Implemented |
| Keyboard navigation (Enter key selects/deselects) | Implemented |
| Summary bar showing the selected pairing | Implemented |
| Create Match button (disabled until both sides selected) | Implemented |
| Loading skeletons while data fetches | Implemented |
| Empty-state messages when no active posts exist | Implemented |
| Success screen with "Create Another Match" reset | Implemented |
| Toast error on application programming interface failure | Implemented |
| Tan Stack Query cache invalidation on mutation success | Implemented |
| Automatic email notification to both parties | Not yet implemented (Phase 2) |
| Pagination for large post lists (>100 results) | Not yet implemented (Phase 2) |
| Admin-dedicated tutor/tutee endpoints (with personal information) | Not yet implemented (Phase 2) |
| Responsive/mobile layout for the two-column grid | Not yet implemented (Phase 2) |
| Match list / history view for admins | Not yet implemented (Phase 2) |

---

## Key Components

### Frontend

| File | Role |
|---|---|
| Matching implementation reference | Single-page component owning all state and rendering |
| api Routes implementation reference | application programming interface ROUTES public.tutors, application programming interface ROUTES public.requests, application programming interface ROUTES admin.matches |
| index implementation reference | Tutor Post, Tutee Post, Match, Match Status |
| query Keys implementation reference | tutor Keys, request Keys, match Keys — Tan Stack Query key factories |
| pagination implementation reference | a page of matching records, with pagination details generic wrapper |
| the private implementation folder | Loading placeholder cards |
| the private implementation folder | Submit button with loading and disabled states |
| toast Store implementation reference | the error operation for mutation failures |
| format implementation reference | format Sgd Range, format Location, format Availability Short, venue Label |
| dummy implementation reference | SG SUBJECTS, SG LEVELS arrays for filter dropdowns |
| Matching test implementation reference | 30 Vitest + Testing Library tests covering all flows |

### Backend (in progress)

| Component | Status |
|---|---|
| Admin Match Controller | In progress |
| submit operation for admin matches endpoint | In progress |
| matches table + migration | In progress |

---

## Dependencies on Other Modules

| Module | Dependency |
|---|---|
| **Auth (Module 1)** | The Create Match endpoint requires an authenticated session with role ADMIN. The public GET endpoints are unauthenticated, but the matching page itself is behind the admin route guard. |
| **Posting (Module 2)** | Tutor posts and tutee posts must exist in the database. The matching page queries the public listing endpoints that Module 2 populates. |
| **Approval (Module 3)** | Only posts with status ACTIVE appear in the lists and can be matched. A post must have been approved by an admin (Module 3) before it becomes ACTIVE. |
| **Commission (Module 5)** | A successful match is the trigger for commission tracking. The commission module reads matches id as its foreign key. |

---

## Known Limitations

1. **Hardcoded page size of 100.** Both queries use page is 0, size is 100. If there are more than 100 active posts of either type, the overflow will silently not appear.
2. **No server-side filter on status.** The frontend passes no status param; the backend is expected to return only ACTIVE posts from the public endpoints by default. This convention is not explicitly enforced in the frontend query params.
3. **Two-column grid is not responsive.** On viewports narrower than md, the two columns stack in a way that has not been tested or styled for mobile.
4. **Success screen text is aspirational.** The user interface states "Both parties have been notified" — however, the backend does not yet send emails on match creation. This copy must be corrected before Phase 2 release.
5. **No admin-specific post detail.** The matching page uses the public tutor/tutee endpoints, which intentionally omit personal information (contact details). Admins therefore cannot see submitter contact info from this screen.

---

## Phase 2 Roadmap

- Add read operation for admin tutors and read operation for admin requests endpoints that include submitter personal information (email, phone) visible only to admins; switch the matching page to those endpoints.
- Implement server-side email notifications when a match is created.
- Replace the size is 100 cap with proper pagination controls in the card columns.
- Add a match history/list page so admins can review past matches and their statuses.
- Make the two-column layout responsive for tablet and mobile.
- Consider adding a conflict-warning user interface when the selected tutee's subject/level does not align with the selected tutor's subjects/levels (currently no client-side validation).
