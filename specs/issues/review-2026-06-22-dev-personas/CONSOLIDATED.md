# Consolidated Dev-Env Persona Review — 2026-06-22 (Boss-verified, de-noised)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

3 experienced personas tested the **deployed dev virtual machine ([REDACTED IP ADDRESS])**, run **serialized** (one at a time —
the no-concurrent-Playwright-MCP lesson; no shared-browser collisions this time). Engineer (Boss) verified the
sharp claims against source/live before listing. No 500s, no render crashes, no personal information leaks in any of the 3 runs.

Personas: **MOE teacher** (supply/tutor, curriculum depth) · **experienced tutee** (demand, comparison-shopper) ·
**admin** (operations). Source files: moe-teacher md, tutee-shopper md, admin-ops md (+ evidence/).

## Real defects (Boss-verified)

| ID | Sev | Area | Finding | Verify status |
|----|-----|------|---------|---------------|
| ADM-01 | **S2** | admin/money | **Matching-page "Mark PAID" fires on one click with NO confirm**; the Commission page has a gold-standard irreversible-action confirm. Highest-consequence financial mutation, unguarded on one of its two surfaces. | Live-observed; Matching implementation reference has confirms for activate/cancel but apparently not the pay action — **code-confirm needed, real candidate** |
| MOE-2 | **S2** | taxonomy | **Subject↔level not scoped**: A-Math (O-Level) offers Primary 1–6 / Nursery / Poly / University / Adult levels. Curriculum-nonsense a teacher spots instantly; lets tutors/tutees create incoherent subject+level combos. | Confirmed by-design — the taxonomy is a flat cross-product, no subject→valid-levels constraint |
| TUT-1 | **S2** | tutee/flow | After posting a request that's **Pending Review**, "Request Interview" + "Invite to job" on a tutor are blocked with copy **"You don't have an active request yet"** — factually wrong (one exists, pending) and pushes a duplicate post at peak intent. | Real — the invite/interview gate requires an ACTIVE (approved) post; copy + timing are the defect |
| ADM-02 | **S2** | admin/analytics | Date-range filter changes the dates but **"in selected period" metrics don't recompute** (New Signups stays =Total Users; Posts Created identical across 30d vs 7d). Some metrics (conversion) ARE period-aware, so it's partial. | Real candidate — needs which-metrics-are-period-scoped audit |
| MOE-1 / TUT-2 | S2* | data | Job board + landing "recent requests" are dominated by the **same Primary-5 Maths card repeated** (215 results / 7× on landing) → reads as fake seed data, kills the "real jobs now" social proof. | **KNOWN** (demo-data variety — dba/seeder, NOT code). *Already on the backlog (review-2026-06-22 "demo-data").* |
| MOE-3 | S3 | tutor/profile | Dashboard banner "still needed: public handle" contradicts the profile showing 100% complete / apply-gate satisfied. | Real — completeness-signal mismatch between dashboard banner and the gate |
| TUT-3 | S3 | tutee/user experience | "Request details" dialog omits the **schedule/availability** (the grid the parent spent the most effort on); shows only budget/venue/notes. | Real — display gap |
| TUT-4 | S3 | tutee/user experience | **No Edit on a posted request** when it's Pending (only View / View applicants / Close). #207 edit-post covers DRAFT/REJECTED/ACTIVE — PENDING is excluded → forces close→repost→re-approval. | By-design gap (PENDING not in the editable set) — reconsider |
| ADM-05 | S3 | admin/user experience | Interview "Time agreed" rows have no **inline Schedule** button (only via detail page); negotiation rows render a **blank "Job" column** → 50 rows look identical. | Real — workflow friction |
| Tax-dupes | S3 | taxonomy | "Mathematics (O-Level)" duplicates E-Math/A-Math (no generic O-Level Maths subject); no lower-primary (P1–P4) subjects though P1–P4 levels exist; missing N using T stream, Combined Humanities, H1 sciences. | Real — taxonomy content gaps/dupes |
| ADM-03 | S3 | admin/analytics | "Avg Commission/Match S$252" looked like grand-total ÷ active. | **Code fix IS live** (the count by invoice status operation, Analytics Service implementation reference) — the number reflects the **data** (few PAID commissions on the virtual machine). De-escalated; label could be clearer. |
| ADM-04 | S3 | admin/analytics | Conversion **9.6% (≤100%) — the 400% bug is GONE** ✓, but the label "Completed of requested (in period)" with displayed Requested is 1/Completed is 11 reads inconsistently. | Wave-2 fix holds; label/denominator-display clarity only |
| Polish | S4 | frontend | "Mon Evening" availability chip (no space); raw enum S4 EXP shown in the review queue; only 1/14 tutors has a review; 63-post review queue unpaginated/unfilterable; admin 8-section nav at 390px shows 2 items, swipe-reachable but no scroll cue/hamburger (ADM-nav, known); commission absent from landing (only FAQ/signup); listing wizard re-asks qualification already on profile; "6 or more" still in students dropdown (check Spec-009 drift). | Mixed — mostly real minor polish |

## Verified WORKING (regression-positive — Wave 1/2/3 fixes hold live on the virtual machine)
- **Step-4 Review does NOT auto-submit** (Spec-008 holds); **budget min>max blocks** correctly.
- **Contact-safety best-in-class** (tutee persona): email/phone self-edit requires password + email-confirm link, current details stay active until confirmed; WhatsApp opt-in with clear consent; soft-delete. (Spec 012, Wave 2.)
- **Interview agreed-slot auto-fill** + **No-Show reachable** (Wave 2 #210.2/.3); **conversion ≤100%** (Wave 2, the 400% bug gone); **create-admin field-named validation** (Wave 3 ADM-400); **super-admin Remove disabled**; reject-reason guard; assign-tutor + activate (live 50% commission preview) + cancel→reopen confirms; analytics inverted-range guard; Commission-page Mark-Paid confirm.
- Mobile @390px solid on the tutee/tutor flows (hamburger + bottom tabs, sticky day column, swipe hint).

## To confirm (low-priority)
- Admin notification bell **563 unread** — likely seed noise, but confirm POST SUBMITTED fan-out isn't unbounded.

## Note
The admin re-run **mutated virtual machine data** (approved/rejected posts, a match marked paid→completed, an interview scheduled).
virtual machine admin password reset to Admin per founder. The mutated state is fine for dev; reseed before a clean next pass.
