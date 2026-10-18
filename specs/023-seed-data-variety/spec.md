> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 023-seed-data-variety
status: HISTORICALLY MARKED IMPLEMENTED
version: 0.2
owners: [dba, data-engineer]
source: specs/issues/review-2026-06-22-dev-personas/CONSOLIDATED md
---

# Spec 023 — Seed data variety

MOE-1/TUT-2 (demo-data monotony) + the out-of-period row that unblocks ADM-02's diagnosis. Data/seeder only —
no product code. Standing rule: regression test per REQ.

## Changelog
- 0.2 (2026-09-05) — Documentation alignment correction authorized by the user: carry018 measurable diversity minimums; correct019 cohort fixtures; retain approved plaintext contact storage and bcrypt. No application implementation or deployment is claimed.
- 0.1 (2026-06-22) — DRAFT.

## Requirements

### REQ-001 — realistic seed variety (S2*, data)
The board/landing must not be dominated by one repeated card. Dev Data Seeder implementation reference over-uses
MATH PSLE/LVL P5/P6 across many posts.
- **Given** a fresh seed, **when** the public assignments board / landing "recent requests" loads, **then** the
  ACTIVE showcase posts have at least8 distinct subject codes,5 distinct level codes and4 distinct planning areas; no (subject, level, budget-bucket, planning-area) tuple exceeds20% of that set, as already required by [018 REQ-003](../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md).
- Design: diversify the seeder's subject/level/area/budget selection (use coherent subject+level pairs per spec
  020/025 so the variety uses approved coherent subject/level pairs). User email/phone follow the explicit [plaintext storage amendment](../architecture/plaintext-email-phone.md); keep bcrypt passwords, public personal information exclusions and soft-delete semantics intact.
- **Regression test (e2e/unit):** inspect the complete ACTIVE showcase fixture set and assert all four thresholds independently. Seed enough rows to satisfy the retained minimums; no small-N relaxation. Include a deliberately overrepresented tuple as a negative test for the20% check.

### REQ-002 — out-of-period dated row for ADM-02 (S2*, data)
Freeze the analytics clock and seed independent posts AND signup users at15,60 and100 days before that instant. These fixtures prove the Last30/Last90 behavior required by019; a >90-day row alone is outside both windows and cannot prove a difference.
- **Given** the isolated fixture set, **when** the same report clock is used for Last30 and Last90, **then** Posts Created AND New Signups each include the 15-day record in both periods, include the 60-day record only in Last90, and exclude the 100-day record from both.
- **Regression test (e2e, analytics):** assert the inclusion/exclusion per row and each metric's Last30 < Last90 using isolated fixture deltas or a fresh dataset. Do not accept a posts-only fixture as proof of signup scoping.

## Design notes
backend: Dev Data Seeder implementation reference. Build **first** (unblocks 019/REQ-002). Must use 020's coherent subject+level
pairs (sequence: 020 taxonomy validation can land alongside, but the seed pairs must be valid).

## Tasks
- [ ] REQ-001 diversify seeder with approved coherent pairs +018's8-subject/5-level/4-area/20%-tuple assertions
- [ ] REQ-002 frozen clock +15/60/100-day posts AND users + independent inclusion/exclusion assertions for both metrics
- [ ] gate backend ≥97.5/97.5; live convergence dev (reseed) → virtual machine

## Review & Sign-Off
- [ ] dba (Morgan)
- [ ] data-engineer (Blake)
- [ ] qa-engineer (Reese)
- [ ] Boss (Rex)
