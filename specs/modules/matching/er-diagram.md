# Module 4 — Entity Relationship Diagram

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Canonical tutor user id and nullable tutor post id are required by INT034; partial active-match deduplication replaces the historical unconditional two-post uniqueness model. Remaining simplified ERD fields are illustrative, not migration database query language.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

Plant UML ER diagram covering the tables touched by the Matching module.  Render with the Plant UML VSCode extension or the online server at https://www plantuml.com/plantuml.

---

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Key Design Notes

### unique constraint on the matches operation

The database enforces that the same tutor post cannot be matched to the same tutee post more than once.  If the admin accidentally clicks "Create Match" twice for the same pair, or if two admins submit concurrently, the second insert will violate this constraint and the backend returns **409 Conflict**.

### Foreign keys

| Column | References | On Delete |
|---|---|---|
| tutor posts tutor id | users id | RESTRICT (user cannot be deleted while posts exist) |
| tutee posts tutee id | users id | RESTRICT |
| matches tutor post id | tutor posts id | RESTRICT |
| matches tutee post id | tutee posts id | RESTRICT |
| matches created by | users id | SET NULL (admin account deletion does not cascade) |

### Status flow precondition

Only posts with status ACTIVE may be included in a match.  A post reaches ACTIVE only after admin approval (Module 3 — Approval).  The backend enforces this; any attempt to match a non-ACTIVE post returns **422 Unprocessable Entity**.

### Cardinality

- One tutor post can appear in many matches rows (e g., a tutor matched with multiple tutees over time) — the unique constraint prevents only duplicate *pairs*, not all reuse.
- One tutee post can similarly appear in many matches rows.
- In practice, once a match is confirmed the post statuses are expected to transition to MATCHED, making them invisible from the public listing endpoints and preventing further matching — but this transition is a backend responsibility, not enforced by the frontend.
