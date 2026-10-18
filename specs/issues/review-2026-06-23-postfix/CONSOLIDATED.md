# Consolidated Post-Fix Domain-Expert Review — 2026-06-23 (Boss-verified)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

3 domain-expert personas (MOE curriculum tutor · veteran parent · ops admin) re-tested the deployed dev virtual machine
after the 019–023 + canonical-taxonomy wave, **serialized** (one Playwright-MCP browser → no collisions).
No 500s/render-crashes/personal information-leaks in any run. Sources: moe-tutor md, parent-shopper md, ops-admin md.

## Boss verification of the sharp claims

- **Money (ADM-01) — SOLID, re-confirmed by the ops-admin expert live.** Mark-Paid opens a confirm naming
  amount/tutor/tutee + "irreversible"; Cancel→nothing paid, Confirm→paid; amount correct (50% first month,
  S$200 of S$400). The headline money fix holds.
- **Taxonomy — mostly right (the core ask).** Generic "Mathematics (O-Level)" GONE; Combined Humanities,
  H1 sciences, N using T, Sec-5 N using A, lower-primary all present + correctly banded; band-level subject→level scoping
  works (no cross-band nonsense). Confirmed by all 3.
- **All prior-wave admin fixes CONFIRMED FIXED** (Job column populated incl free-market, review-queue
  pagination+filter, 390px nav, analytics labels, reason-guard) + contact-safety "excellent" + Step-4
  no-auto-submit + min>max + invite/interview-on-PENDING (interview path works end-to-end).

## Real defects (Boss-verified)

| ID | Sev | Area | Finding | Root cause (verified) |
|----|-----|------|---------|----------------------|
| **P1** | **S2** (headline broken) | invite | "Invite tutor to job" always **404s** + no error shown; invitation never created. | **frontend↔backend id contract mismatch:** Tutor Detail implementation reference passes the selected tutor's account identifier (tutor-post/handle id) but Job Invitation Service does the find by id operation (expects USER id) → TUTOR NOT FOUND. Public data transfer object hides user Id by design (). Long-standing; masked by direct-application programming interface checks. **Fix: backend resolves the tutor by public handle / tutor-post id** (or expose a safe invite handle), + surface the frontend error. |
| **P2** | **S2** | wizard | Post wizard Step-1 **silently jams**: pick "6 or more" → revert to "1 student" → Continue dies, no message. Deterministic. | Stale required-state on the now-hidden exact-count field (the #205 "6 or more" reveal). frontend validation cleanup. |
| **MOE-T3** | **S2** | apply (money) | Browse **list-card "Apply" fails silently** (422 TUTOR PROFILE INCOMPLETE, zero user interface feedback). The detail-page apply handles the same gate correctly. | The list-card apply path doesn't surface the typed PROFILE INCOMPLETE feedback (the T-07 fix landed on the detail surface only). |
| **MOE-T4** | **S2** | apply gate | Profile shows **"100% complete"** yet Apply is blocked until a **"(optional)" Planning areas** chip is added. Contradictory. | Completeness predicate vs apply-gate disagree on planning-areas (the MOE-3 reconcile didn't cover this field). **Product call:** make planning-areas count toward completeness, or drop it from the apply gate. |
| **MOE-T2** | S2 (label) | taxonomy user experience | Level-group header reads **"SECONDARY (O-LEVEL)"** but contains the N using A/N using T levels — calling Normal-stream "O-Level" is wrong to any SG educator. | Band label; rename to "Secondary" (the band spans Express/N using A/N using T). |
| **OPS-01** | S3 | label | One tutee post shows the **raw enum MATH OLEVEL** (queue list + detail heading) instead of a friendly label. | Label-map gap for the **deprecated** MATH OLEVEL subject (existing data still references it). Add a label/alias. |
| **MOE-T1** | S2* (deeper) | taxonomy depth | Every Secondary subject offers ALL secondary years incl Sec-1 + N using T (e g. A-Math at Sec-1 / N-Tech, which don't exist). | The level group scoping is **band-level, not stream×year** — the agreed design. Finer stream×year scoping is a larger taxonomy model change. **Recommend: defer** (note as known limitation) or a follow-up spec. |

### Lower (S3/S4 — file, don't block)
P3 raw seed handles ("tutor joanne chua") on browse; P4/P5 + MOE saved-post View drops session duration/frequency
("20 slots"); OPS-02 interview-funnel tiles don't reconcile (Requested 1 / Completed 11 / 9.5%); OPS-03 in-period
signups/posts didn't move 30d vs 7d (likely data-confound, confirm); wizard phantom Step-4 (T5), handle-collision
409 on first save (T6), P1–P4 under "(PSLE)" label (T7), back-nav wipes fields (T8); mobile bottom-nav overlap;
"2 hours" default; password-form a11y; "Step 1 of 2" on a one-step register.

## Recommendation
Fast-fix the S2s — **P1 (invite contract)**, **P2 (wizard revert)**, **MOE-T3 (list-card apply feedback)**,
**MOE-T4 (completeness/planning-areas reconcile — needs a product call)**, **MOE-T2 (band label)** + **OPS-01
(label-map)** — via the Boss loop → dev+virtual machine, each with its regression test. **Defer MOE-T1** (stream×year scoping,
larger). File the S3/S4 batch as GH issues. Money + taxonomy core are solid.
