# Module 4 — Sequence Diagrams

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** The two-post browse/selection diagrams are historical user interface examples. Current assignment starts from tutor User Id and tutee Post Id with optional tutor Post Id, using INT034 atomic job/application/selection closure and active-match dedup. Listing absence is never an assignment rejection.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

All diagrams use Plant UML syntax.  Render with the Plant UML VSCode extension, the online server at https://www plantuml.com/plantuml, or any continuous integration Plant UML runner.

---

## 1. Admin Loads the Matching Dashboard

Two independent Tan Stack Query fetches fire in parallel on mount.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2. Admin Filters by Subject and Level

Changing either dropdown updates local state, which changes the Tan Stack Query key, causing a background re-fetch.  Previously selected cards are NOT reset by the filter change (selection state is kept in local use State).


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 3. Admin Selects Tutor Card + Tutee Card + Clicks Create Match

Card selection is purely local state.  No network call occurs until the button is clicked.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Success Path — 201 Created


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 5. Error Path — 4xx Response


Implementation example omitted. This public edition retains the written requirements in this section.


## 2026-09-05 current contract and verification

Current sequence: authorized admin chooses a canonical tutor for a live eligible job → validates optional listing ownership and existing eligibility → locks/checks active arrangement → writes Match and closes/supersedes related intake atomically with audit/outbox intent → returns current response → invalidates job/applicant/selection/match views. Verify both with-listing and without-listing paths and concurrent duplicate rejection. The diagram below/above is not a current requirement to select two posts.
