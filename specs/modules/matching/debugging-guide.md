# Module 4 — Debugging Guide

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The troubleshooting observations below are dated, not present-runtime claims. Cancelled pairs may rematch under the later lifecycle; tutor listings are optional under M18. Current first-session auth, paid completion, outbox and scheduler requirements are in [matching/backend md](backend.md) and the linked parent specifications. Earlier open questions already settled by those amendments must not be reopened.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

Targeted at developers who need to debug or extend the Admin Matching module.  Issues are ordered from most commonly reported to least.

---

## 1. Tutor or Tutee List Not Loading

**Symptoms:** The page shows skeleton cards indefinitely, or both columns remain empty after the skeletons disappear.

**Diagnostic steps:**

1. Open Dev Tools → Network tab.  Filter by tutors or requests.  Confirm two requests fire on mount:
   - read operation for tutors?page is 0&size is 100
   - read operation for requests?page is 0&size is 100

2. Check the response status:
   - 200 with data — the application programming interface is reachable but no active posts exist.  Seed some ACTIVE posts.
   - 200 with data: ... — check Tan Stack Query Dev Tools.  The data may be in cache but data data (the inner array) could be miscased.  Confirm the Page Response envelope: response data.data (first .data is the Axios response body, second .data is the Page Response data array).
   - 4xx or 5xx — the backend is rejecting the request.  Check backend logs.

3. Inspect the Tan Stack Query Dev Tools panel (if available in the dev build).  Locate the the list operation entry.  If it shows error, expand to see the error message.

4. Confirm the api instance (api implementation reference) has the correct base URL.  The public endpoints must not require an Authorization header — confirm no request interceptor is injecting auth headers onto public routes (or confirm the backend accepts and ignores them).

**Common root causes:**

- Backend is not running or is running on a different port.
- cross-origin request permissions not configured for the frontend origin in the backend.
- The backend returns posts with status differs from ACTIVE and the frontend has no client-side status filter — check the backend's default filter on these endpoints.

---

## 2. Filter Not Working (Changing Dropdown Has No Effect)

**Symptoms:** Selecting a subject or level from the dropdown does not narrow the card list.

**Diagnostic steps:**

1. Open Dev Tools → Network tab.  After changing a dropdown, confirm a new request fires with the updated query param:
   - read operation for tutors?subject= REDACTED
   - read operation for requests?level= REDACTED

2. If the request does NOT fire, the query key may not have changed.  Add a console log in the component to print tutor Filters on each render and confirm the value updates when the dropdown changes.

3. If the request DOES fire but the response still returns unfiltered data, the backend does not yet support the subject or level query parameters on these endpoints.  The backend must implement server-side filtering — the frontend correctly passes the params but cannot filter client-side after the fact (it trusts the server).

4. Confirm the param name casing matches what the backend expects.  The frontend sends lowercase subject and level.  Some backend frameworks are case-sensitive on query param names.

**Note:** Filter changes do NOT reset card selections.  If a user selects a tutor card and then changes the filter, the selection state (selected Tutor Id) is preserved even if the selected card is no longer visible.  This is intentional but can cause the Create Match button to appear enabled with a card that is no longer in the visible list.  The backend will still receive the correct IDs.

---

## 3. Create Match Returns 409 Conflict

**Symptoms:** The mutation fires, the backend returns web request protocol 409, the error toast appears: "Failed to create match. Please try again."

**Explanation:**

A unique constraint on the matches table — unique tutor post id, tutee post id — prevents the same pair from being matched more than once.  A 409 means a matches row with this exact tutor Post Id, tutee Post Id combination already exists.

**Diagnostic steps:**

1. Query the matches table directly:


Implementation example omitted. This public edition retains the written requirements in this section.


2. Check the status of the existing match.  If it is CANCELLED, the business rule may intend to allow re-matching — raise this with the product team.  The database constraint must be relaxed (or a soft-delete pattern used) before re-matching is possible.

3. If the match was created by a concurrent admin session, the 409 is expected and correct.  Refresh the match list and verify the record was created.

**Do not retry on 409.**  The frontend toast instructs the user to "Please try again" — this wording is generic.  For 409 specifically, the correct action is to check whether the match already exists, not to retry.

---

## 4. Create Match Returns 422 Unprocessable Entity

**Symptoms:** The mutation fires, the backend returns web request protocol 422, the error toast appears.

**Explanation:**

The backend enforces that both the tutor post and the tutee post must have status ACTIVE at the time the match is created.  A 422 means one or both posts are in a non-matchable state (e g. PENDING, REVISION REQUESTED, REJECTED, MATCHED, CLOSED, or EXPIRED).

**Diagnostic steps:**

1. Determine which post is not ACTIVE.  Fetch the post detail via the admin endpoint:


The illustration for . create match returns 422 unprocessable entity contains the following information:

- GET admin posts interface operation<id>.
- Authorization: Bearer <admin-jwt>.


2. Common reasons a post is not ACTIVE when it appears in the list:
   - The public endpoint is returning posts with status other than ACTIVE (backend bug — it should filter server-side).
   - The post was approved but then its status was manually changed in the database.
   - Another match was just created for this post in a concurrent session, which transitioned its status to MATCHED.

3. If the post is legitimately not yet approved, the admin must navigate to Module 3 (Approval) and approve it before returning to Matching.

---

## 5. Success Screen Shows "Both Parties Have Been Notified" But No Email Was Received

**Symptoms:** The match is created (201 response), the success screen appears, but the tutor and tutee report they received no notification email.

**Explanation:**

This is a **known limitation**, not a bug.  The success screen copy ("Both parties have been notified...") is aspirational text written in advance of the email notification feature.  As of the current implementation, the backend does NOT send any email or notification when a match is created.  The admin must contact both parties manually to arrange the introduction.

**Phase 2 action:** Implement a post-match notification service on the backend.  When submit operation for admin matches succeeds, the backend should trigger emails to the tutor and tutee (using their stored contact details, which are accessible to the backend even though they are not exposed in the public application programming interface responses).  Once this is implemented, update the success screen copy to be factually accurate.

**Immediate workaround:** Admins should use the contact details visible in the admin post detail view (Module 3 approval page) to reach out to both parties manually.

---

## 6. Cards Are Not Selectable (Clicking Has No Effect)

**Symptoms:** Clicking a tutor or tutee card does not highlight it or set aria-pressed is yes.

**Diagnostic steps:**

1. Open Dev Tools → Elements.  Inspect a card element.  Confirm:
   - It has role is button.
   - It has tab Index is 0.
   - It has aria-pressed is no (or yes if already selected).

2. If the card has role is button but the click is not registering, check whether a parent element or an overlay is intercepting pointer events.  Add pointer-events: none debugging or use Dev Tools event listener panel.

3. Confirm on Click is wired directly on the card div, not on a child element only.  The click is handled by the card div, and the user can click anywhere within the card (including child text nodes) — but the test confirms the closest operation receives the click, not just the text.

4. In tests: the test suite uses the click operation.  If tests pass but the browser does not respond, the issue is likely a CSS overlay (pointer-events: none or z-index problem) or a conflicting React event handler added during development.

5. Check that the SG SUBJECTS and SG LEVELS data arrays (dummy implementation reference) are correctly populated.  These drive the filter dropdowns.  If they are empty arrays, the dropdowns render but the filter state never changes from , which is fine — but it is worth checking if the component ever enters an unexpected loading or error branch.

---

## 7. Match Count in Column Header Appears Wrong

**Symptoms:** The column header says "Tutor Posts (47)" but the admin knows there are more than 100 active posts.

**Explanation:**

The matching page hardcodes page is 0, size is 100 for both queries.  The column header displays tutors length — the count of items in the current page, not meta total (the server-side total).

If there are more than 100 active posts, the application programming interface will return only the first 100 and meta total will be a larger number, but the user interface will show the count of the returned items (up to 100), not the true total.

**Impact:** Admins will not see or be able to select posts beyond position 100 in either column.

**Workaround (short-term):** Increase size temporarily (e g. size is 500), but note that very large page sizes may cause backend timeouts.

**Fix (Phase 2):** Implement proper pagination controls in the card columns.  Display meta total as the authoritative count and add next/previous page navigation.  Alternatively, implement infinite scroll backed by Tan Stack Query's use Infinite Query.

---

## Quick Reference — Error Codes and Actions

| web request protocol Status | Where It Can Come From | What To Do |
|---|---|---|
| 200 with empty data | GET tutors / GET requests | Seed ACTIVE posts in the database |
| 401 | POST admin/matches | Session expired — re-login |
| 403 | POST admin/matches | Caller is not ADMIN — check signed session token role claim |
| 404 | POST admin/matches | Post ID not found — IDs may be stale; refresh the page |
| 409 | POST admin/matches | Duplicate pair — match already exists in the database |
| 422 | POST admin/matches | Post not ACTIVE — approve the post first (Module 3) |
| 5xx | Any endpoint | Backend error — check server logs |
