# Module 5: Admin Commission Tracking — Sequence Diagrams

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** PAID records agency receipt of tutor commission, after the existing first-session confirmation/collection sequence; it is not tutor payout or parent tuition payment. Unpaid cancellation now follows the current assignment lifecycle; exceptional reversals remain open.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

All diagrams use Plant UML syntax and can be rendered at https://www plantuml.com/plantuml/uml/ or via any Plant UML-capable IDE plugin.

---

## 1. Admin Activates a Match

The admin provides a monthly fee. The backend creates a Commission record and transitions the match from PENDING to ACTIVE in a single transaction.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2. Admin Cancels a Match

Cancellation is permitted from the current nonterminal assignment states. No new commission is created. An existing unpaid commission transitions to CANCELLED in the same transaction, and the job/application intake states reopen under the assignment lifecycle. Paid/terminal reversal policy remains explicitly unresolved; do not apply this unpaid path to it.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 3. Admin Marks Commission as Paid

Paying a commission transitions it from PENDING to PAID and atomically sets the match to COMPLETED.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Admin Loads Commission Ledger

The frontend fetches all matches and their commissions in a single web request protocol request. The backend avoids N+1 queries by batch-loading all commissions for the returned match IDs in one query.


Implementation example omitted. This public edition retains the written requirements in this section.


## 2026-09-05 current contract and verification

Before marking PAID, confirm agency receipt from the tutor; commit commission PENDING→PAID and match COMPLETED atomically. First-session confirmation alone does not complete the match. Verify cancellation updates an existing unpaid commission and restores eligible intake state; it cannot leave an active invoice stranded. See [current open decisions](../../CURRENT-REQUIREMENTS.md) for paid/terminal reversal rather than inventing it from these code-derived diagrams.
