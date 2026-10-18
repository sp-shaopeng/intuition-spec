> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 019-admin-money-safety-and-analytics
status: HISTORICALLY MARKED IMPLEMENTED
version: 0.2
owners: [frontend-expert, software-engineer, bff-specialist]
source: specs/issues/review-2026-06-22-dev-personas/CONSOLIDATED md
---

# Spec 019 — Admin money-safety & analytics

Fixes from the 2026-06-22 dev-persona review (admin/operations lens). Standing rule for **every REQ**: ship the
**regression test that would have caught it** (layer named per REQ). Soft-delete only.

## Changelog
- 0.2 (2026-09-05) — Documentation alignment correction authorized by the user: retain015 mobile drawer/a11y requirements; make both analytics cohorts independently testable; clarify agency receipt. No application implementation or deployment is claimed.
- 0.1 (2026-06-22) — DRAFT. Scoped from CONSOLIDATED (ADM-01..05, review-queue, nav, enum). Team-review folded:
  ADM-02 is diagnose-first (backend already period-scopes), ADM-01 reuses the Commission confirm.

## Requirements

### REQ-001 — ADM-01 Mark-Paid confirm on the Matching page (S2, money)
The highest-consequence financial action must be guarded on **both** surfaces. **Mark PAID means the agency has received the tutor commission**, not that the parent paid for a lesson.
- **Given** an admin on /admin/matching with a payable commission, **when** they click "Mark PAID",
  **then** a confirm dialog appears naming amount + tutor + tutee + "this is irreversible", **and** the pay
  mutation fires **only** from the dialog's confirm button.
- **Guard:** **Given** the confirm dialog is open, **when** the admin clicks Cancel, **then** no mutation fires
  and the commission stays unpaid.
- Design: port Commission implementation reference's Pay Confirm modal (26,,) to Matching implementation reference
  (currently the mutate operation fires on click). frontend-only — backend already has Commission.@Version +
  Idempotency-Key (defence-in-depth intact).
- **Regression test (e2e, matching spec):** click Mark-Paid → assert dialog visible AND no POST .../pay
  network call until confirm is clicked.

### REQ-002 — ADM-02 analytics period metrics (S2, diagnose-first)
- Verified: Analytics Service implementation reference already period-scopes (count By Created At Between ,,,
  ,) and Analytics implementation reference reads the *In Period fields. The persona symptom is almost certainly a
  **data confound** (virtual machine seed all recent → period  equals  all-time).
- **Acceptance (diagnose before any code):** with spec 023's out-of-period seed row present, **when** the admin
  selects "Last 30 days" vs "Last 90 days", **then** Posts Created (and New Signups) differ. **Only if** they do
  NOT differ is there a code bug → then fix the offending query.
- **Regression test (e2e, analytics spec):** freeze the report clock. Seed separate posts AND users created 15, 60 and 100 days before that instant (spec 023). Assert Posts Created AND New Signups each include the 15-day row in both windows, include the 60-day row only in Last90, and exclude the 100-day row from both. Assert each metric's Last30 count is below Last90; use isolated fixture deltas or a fresh seed so unrelated data cannot supply the difference. **Founder-confirm REQ** (diagnose-first; code only if flat).

### REQ-003 — ADM-03/04 analytics label clarity (S3)
The earlier Wave-3/2 code observation does not define the business cohort. Preserve spec 017's decided conversion cohort and make its numerator/denominator clear (e g. "Completed of requested in period"). Average-commission labels must use the exact owner-approved cohort recorded in017; the missing cohort definition remains pending there and must not be inferred from existing code.
- **Regression test (unit/component):** label renders the period-scoped denominator; snapshot.

### REQ-004 — ADM-05 interview row: inline Schedule + Job column (S3)
"Time agreed" interview rows must expose an inline Schedule action (not detail-page only), and the negotiation
rows' "Job" column must be populated (currently blank → 50 rows look identical).
- **Regression test (component/e2e):** a Time-agreed row renders a Schedule button; the Job column shows the
  post title/ref.

### REQ-005 — review-queue pagination/filter (S3)
The 63-item review queue is unpaginated/unfilterable. Add pagination + a status filter.
- **Regression test (component):** queue paginates; filter narrows the list.

### REQ-006 — admin nav at 390px (S3)
Retain [015 REQ-002](../015-mobile-responsive-fixes/requirements.md): below the desktop breakpoint, expose all 8 admin sections in a usable hamburger drawer or equivalent menu, with focus trapping and Escape dismissal. A cramped horizontal scroll strip with an added cue does not meet the requirement.
- **Regression test (component/a11y and responsive e2e):** at 390px, open the menu, reach every section by keyboard, verify focus remains within the open menu, dismiss with Escape, and verify no sections are clipped/hidden. Capture the390px view.

### REQ-007 — humanise raw enum in review queue (S4)
The review queue renders a raw S4 EXP; humanise via the level-label helper.
- **Regression test (unit):** the row shows the friendly level label, not the enum code.

## Design notes
frontend: Matching implementation reference (REQ-001), Analytics implementation reference (REQ-002/003 display), the admin
interview queue + review queue components (REQ-004/005/007), admin nav (REQ-006). backend: none expected (REQ-002
diagnose-first). Build lane: **019 frontend** runs parallel to 022; ADM-02 diagnosis after 023 reseed.

## Tasks
- [ ] REQ-001 port Pay Confirm to Matching + e2e
- [ ] REQ-002 fixed-clock15/60/100-day posts AND users (with023) + both-metric cohort e2e; code only if flat
- [ ] REQ-003 labels + test
- [ ] REQ-004 inline Schedule + Job column + test
- [ ] REQ-005 queue pagination/filter + test
- [ ] REQ-006 mobile drawer/menu + focus-trap/Escape + eight-section responsive test
- [ ] REQ-007 enum humanise + unit test
- [ ] gates backend ≥97.5/97.5, frontend ≥97.5/88; live convergence dev→virtual machine

## Review & Sign-Off
- [ ] frontend-expert (Sam)
- [ ] software-engineer (Jordan)
- [ ] bff-specialist (Riley)
- [ ] security-expert (Casey) — REQ-001 money action
- [ ] qa-engineer (Reese)
- [ ] technical-writer (Chris)
- [ ] Boss (Rex)
