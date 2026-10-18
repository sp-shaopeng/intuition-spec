# Admin Matching Page — Frontend Implementation Spec (M18 §G3)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** The optional-listing assignment amendment is folded into §0a; both queues use tutor User Id.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Status:** SPEC — pending team review  
**Date:** 2026-06-01  
**Author:** Engineering  
**Route:** /admin/matching (replaces the old browse+select two-panel user interface)  
**Drives:** M18 §G3 from assignment-fe-spec md (already team-reviewed + Boss SHIP IT)  
**Source of truth:** assignment-fe-spec md §G3 + assignment-uiux-spec md §G  

---

## 0a. Current free-market assignment contract

product requirements document §20 and [INT-034](int-034-free-market-assign.md) explicitly supersede the old two-post stopgap. Both Selection Request Queue and Interview Queue send the tutor-account and tuition-request identifiers to the shared administration assignment operation. An optional attached tutor Post Id may be included as a listing override after ownership validation; its absence never disables Assign. The canonical tutor profile supplies identity and the existing profile/action eligibility rules apply. Do not tell an eligible tutor to publish a listing to be assigned.

The response contracts expose canonical tutor User Id and nullable tutor Post Id; all consumers must handle the no-listing branch. Verify assignment through each queue with and without a listing, duplicate active-match rejection, and cache invalidation for the closed job/applicants. Historical component completion claims below remain dated evidence.

---

## 0. Context — what already exists and what does not

### Already done (do NOT rebuild)
- Selection Request Queue component (Selection Request Queue implementation reference) ✅
- Close Job Assign Modal component (Close Job Assign Modal implementation reference) ✅
- Stale Commission List component (Stale Commission List implementation reference) ✅
- interview Keys, selection Request Keys, match Keys in query Keys implementation reference ✅
- application programming interface ROUTES admin.selection Requests, .matches, .match Cancel, .commission Pay, .match Activate ✅
- Admin Selection Request Response, Admin Match Item, Admin Commission Item types ✅

### The only file that needs full rewrite
**Matching implementation reference** — the current implementation is an old-style two-panel
browse+select user interface (Tutor Post list ↔ Tutee Post list, click to pick, "Create Match" button).
The M18 spec replaces it with a **lifecycle command centre** organised around:
1. Selection requests (parent choices awaiting admin confirmation → assign)
2. Match management (PENDING/ACTIVE/COMPLETED/CANCELLED matches with lifecycle actions)

The old create Match Mutation (takes tutor Post Id + tutee Post Id) remains for backward compatibility
but the primary assign path from the Selections section uses tutor User Id from the selection
request (same endpoint, same application programming interface ROUTES admin.matches — matching what Interview Queue already does).

---

## 1. Page structure


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2. Data fetching

### 2.1 Selection requests


Implementation example omitted. This public edition retains the written requirements in this section.


Filter client-side to status equals CREATED (the list endpoint already returns only CREATED,
but guard defensively). Sort by created At ascending (oldest first = most urgent).

### 2.2 Matches


Implementation example omitted. This public edition retains the written requirements in this section.


Filter client-side by status based on the active tab. The backend returns all matches; no server-side
filter params are needed (volume is low at current scale).

---

## 3. Mutations

### 3.1 Assign tutor & close job (from selection request)


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.2 Activate match


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.3 Mark PAID (commission)


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.4 Cancel match (reopens the job)


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. user interface states

### Section A — Pending Selections
| State | Render |
|---|---|
| loading | skeleton placeholder rows ×3 |
| empty | A card saying “No pending selections.” |
| data | selection-request queue component (already exists — pass requests + open the assignment dialog for the selected applicant) |
| assign submitting | the assignment dialog while a submission is in progress (spinner on confirm button) |

Reuse Selection Request Queue unchanged. Pass on Assign to set assign Target state (same
pattern as Interview Queue does).

### Section B — Match Lifecycle
| State | Render |
|---|---|
| loading | skeleton placeholder rows ×3 |
| empty (filtered) | Inline: "No tab matches." |
| error | Red alert card + "Refresh" button |
| data | Table (see §5 below) |

**Filter tabs:** All · Pending · Active · Completed · Cancelled (role is tablist +
aria-selected — same pattern as existing admin pages).

---

## 5. Match table layout


The illustration for . match table layout contains the following information:

- Columns: Subject pair Status Commission Created Actions.


| Match status | Visible actions |
|---|---|
| PENDING | Activate → opens fee modal; no cancel yet (or include cancel for flexibility) |
| ACTIVE | Mark PAID (if commission invoice Status  equals  'PENDING') · Cancel |
| COMPLETED | — (read-only) |
| CANCELLED | — (read-only) |

**Subject pair:** the tutor's subject and the tutee's subject (existing Admin Match Item shape).

**Status badge:** use existing status Badge Variant + status Label formatters.

**Commission column:** PENDING matches show "—" (no commission yet); ACTIVE/COMPLETED show
the format sgd operation + invoice status badge.

**Cancel confirm modal** (inline, not a separate component): simple modal component with "Cancel this
match? The job will reopen for new applicants." + Keep match / Cancel match danger
button.

---

## 6. Activate fee modal (inline — no new component)

An inline modal component controlled by activating Match Id state:


Implementation example omitted. This public edition retains the written requirements in this section.


This matches the pattern already used in Commission implementation reference (lines 95–143).

---

## 7. Component reuse checklist

| Need | Reuse |
|---|---|
| Selection requests rendering | selection-request queue component ✅ |
| Assign confirm | assignment confirmation modal ✅ |
| Activate fee | Inline modal component (copy pattern from Commission implementation reference) |
| Cancel confirm | Inline modal component (new, simple) |
| Status badge | a badge styled according to the status ✅ |
| Skeletons | skeleton placeholder ✅ |
| Toasts | toast success / toast error ✅ |
| SGD format | format Sgd ✅ |
| Date format | format Date ✅ |

No new user interface primitives. No new components. Everything reused from existing codebase.

---

## 8. State shape


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 9. Tests to add (Matching test implementation reference)

The old tests (if any) can be replaced. New required cases:

| Test | Assertion |
|---|---|
| renders "Pending Selections" heading | heading present |
| renders Selection Request Queue with loaded selections | mocked requests → at least one row |
| empty selections → "No pending selections." | empty state text |
| Assign click → Close Job Assign Modal opens | modal visible |
| Assign confirm → mutation called with correct tutee Post Id + tutor User Id | the fn operation called |
| 409 → error toast + invalidation | toast content matches |
| match filter tab click → "Active" tab active | aria-selected is yes on tab |
| Activate button → fee modal opens | modal visible |
| Mark PAID click → commission Pay mutation called | mock verify |
| Cancel confirm → cancel Match mutation called | mock verify |
| Loading state → skeleton visible | Skeleton present |

Coverage gate: ≥ 97.5% line / ≥ 88% branch. Run the documented operational check (tsc) to verify no type errors.

---

## 10. Acceptance criteria

1. /admin/matching shows "Pending Selections" section with Selection Request Queue for CREATED selection requests.
2. "Assign tutor & close job" opens Close Job Assign Modal; confirm calls submit operation for admin matches with the tuition-request and tutor-account identifiers.
3. On 409 JOB ALREADY ASSIGNED → error toast + query invalidation.
4. "Match Lifecycle" section shows matches with filter tabs (All / Pending / Active / Completed / Cancelled).
5. PENDING match row has "Activate" button → fee modal → submit operation for admin matches id activate.
6. ACTIVE match row has "Mark PAID" (if commission PENDING) → submit operation for admin commissions id pay.
7. ACTIVE match row has "Cancel" → confirm modal → submit operation for admin matches id cancel → toast "Match cancelled — job reopened."
8. All toasts use the exact copy from §3.
9. the documented operational check passes (≥ 97.5% / 88%); the documented operational check (tsc) green; the documented operational check clean.
10. The old browse+select two-panel user interface is completely replaced.

---

## 11. Files changed

| File | Change |
|---|---|
| Matching implementation reference | Full rewrite — old 249-line browse+select → new lifecycle page |
| Matching test implementation reference | New / replaced tests |
