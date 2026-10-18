# Ops-Admin Re-Test — Post-Fix Review (2026-06-23)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

**Persona:** Operations admin (the 1–2 person coordinator team) who lives in the admin console.
**Target:** DEPLOYED dev site http://[REDACTED IP ADDRESS] — admin console.
**Login:** REDACTED email (super-admin) — worked.
**Method:** Playwright MCP browser, desktop (1280×900) + mobile (390×844).
**Console:** No JS errors during admin actions (the only error all session was a benign pre-login 401 auth me interface operation).

---

## TL;DR — the headline fixes

The **money fix is solid**. Mark-PAID on /admin/matching now opens a confirm dialog that names the
**amount + tutor + tutee + "irreversible"**, does **nothing on Cancel**, and only pays on **Confirm**. The
commission amount is **correct (50% of first month)**. Analytics labels are **fixed** ("Across matches with a
paid commission"; interview conversion **9.5%**, ≤100%, sensibly labelled). The interview **Job column is
populated on every row**, and **"Time agreed" rows carry an inline Schedule** action. Review queue
**paginates + filters**. Approve and Reject both confirm, and Reject has a working **reason-guard**.

**One real defect remains:** a TUTEE post renders the **raw enum MATH OLEVEL** instead of a friendly subject
label, both in the review-queue list and on the post detail page (heading + Subject field). S3 — cosmetic but
unprofessional in front of an admin and a sign the label map has a gap.

---

## Findings

| ID | Sev | Screen / URL | What I saw | Why it matters to ops | Screenshot |
|---|---|---|---|---|---|
| **OPS-01** | **S3** | /admin/review + /admin/review/f3631dd6… | A TUTEE post shows the **raw enum MATH OLEVEL** as its subject — in the queue list cell AND as the detail-page H1 heading + "Subject" field. Every other post resolves to a friendly label ("A-Math (O-Level)", "Mathematics (PSLE)", "H2 Chemistry (A-Level)", etc.). Levels are all friendly ("Secondary 4 (Express)" — **no S4 EXP** raw level seen anywhere). | An admin reviewing this post can't tell the subject at a glance; it reads like broken data. The friendly-label resolver has no mapping for the key MATH OLEVEL and falls through to the raw value. Looks unprofessional and risks a wrong approve/reject. | admin-01, admin-10 |
| OPS-02 | S4 | /admin/analytics (Interview Funnel) | Conversion = **9.5%**, labelled "Completed ÷ requested, both in the selected period". But the tiles above read **Requested 1 / Completed 11** — 11÷1 would be 1100%, not 9.5%. The 9.5% is correct vs a larger requested-in-period denominator; the **"Requested 1" tile** appears to count something narrower (e g. currently-in-REQUESTED-state) than the conversion denominator. | The rate is sane and ≤100% (the fix), but a coordinator eyeballing "1 requested, 11 completed, 9.5%" sees three numbers that don't reconcile. Minor trust ding on the analytics page. | admin-05 |
| OPS-03 | S4 | /admin/analytics (date range) | Changing **Last 30 days → Last 7 days** updated the From/To dates (2026-05-24→2026-06-16) and changed **"Matches Created in selected period" 65 → 64**. But **"New Signups in selected period" stayed 904** and **"Posts Created in selected period" stayed 447** across both ranges. | Date range IS working (dates + Matches Created changed). The unchanged 904/447 is *plausibly correct* — the daily table shows ~all signups landed Jun 17–22 (297+408+167 on Jun 20–22 alone), so they fall inside both windows. Flagged only so the team confirms these two tiles actually re-query and aren't all-time values mislabelled "in selected period". | admin-05, admin-06 |

No S1 or S2 issues found. No money error. No irreversible action without a confirm.

---

## ✓ Worked / improved — explicit confirmations of the flagged fixes

### 1. MONEY — Mark-PAID confirm (the headline fix) — **FIXED ✓**
On /admin/matching, the one payable row was **Reg Tutor ↔ Reg Parent, Live, S$200 Unpaid**, basis
**"S$50/hr × 4 sessions × 2h = S$400 → 50% = S$200"**.
- Clicking **Mark PAID** opened a confirm dialog titled **"Mark commission as paid?"** reading:
  *"Confirm you are marking the **S$200** commission for tutor **RGffccde** (tutee **Reg Parent**) as paid.
  **This is irreversible.**"* — names amount + tutor + tutee + "irreversible". ✓ (admin-02)
- **Cancel** → dialog closed, row **stayed Live / Unpaid**, nothing paid. ✓
- **Confirm payment** → row flipped to **Completed / S$200 / Paid**, Mark-PAID + Cancel actions removed. ✓ (admin-03)
- **Amount correct:** S$200 = 50% of the S$400 first-month fee. ✓ The breakdown string is shown on every
  commission row (matching dashboard AND commission ledger), and the Commission page header states
  "50% of the monthly fee".
- **Commission page Mark-Paid confirm:** could **not** be exercised — /admin/commission shows
  **Outstanding S$0** (the only payable was the one I just paid), so no payable row with a Mark-PAID button
  existed to click. The ledger's "Commission (50%)" column and per-row breakdown are correct. Recommend the
  team re-confirm the ledger-side confirm with a fresh Active match. (admin-04)

### 2. Analytics labels — **FIXED ✓**
- **"Avg Commission / Match" = S$234**, sub-label **"Across matches with a paid commission"** — exactly the
  required wording, NOT "active matches". ✓ (admin-05)
- **Interview-funnel conversion = 9.5%**, label **"Completed ÷ requested, both in the selected period"** —
  **≤100%** and sensibly labelled. ✓ (the reconciliation nit is OPS-02 above, S4)
- **Date-range filter** present (quick ranges 7/30/90/YTD + From/To pickers) and **changes period metrics**
  (Matches Created 65→64). ✓ (admin-05, admin-06)

### 3. Interview queue — **FIXED ✓**
- **Job column populated on every row** — every interview shows Subject qualification · Level, e g.
  "H2 Physics (A-Level) · Junior College 1", "Mathematics (PSLE) · Primary 5". **No blank Job cells**,
  including the free-market/Reg Parent ↔ Reg Tutor rows. The only "—" cells are in the *Scheduled* column
  (not-yet-scheduled), not Job. ✓ (admin-07)
- **"Time agreed" rows carry an inline Schedule action** — row 2ef9b50a… (status "Time agreed") shows a
  Schedule button right in the Actions cell. The Requested row also has inline Schedule. ✓ (admin-07)

### 4. Review queue — **FIXED ✓**
- **Pagination present** — "Page 1 of 4", Previous (disabled on p1) / Next. ✓ (admin-01)
- **Type filter works** — All / Tutor / Tutee; clicking **Tutee** narrowed the list to TUTEE rows only. ✓
- **Level labels friendly** — "Secondary 4 (Express)", "Junior College 1", "Primary 5/6" — **no S4 EXP**
  raw level anywhere. (Subject raw-enum gap is OPS-01.)

### 5. Admin nav @ 390px — **OK ✓**
All **8 sections** (Review, Matching, Jobs, Interviews, Commission, Users, Analytics, Admins) present and
reachable in the horizontal nav, with the scroll cue **"Scroll sideways to see more admin sections."** The
review queue itself reflows from a table to a card/list layout at mobile width. ✓ (admin-08)

### 6. Approve + Reject — **OK ✓**
- **Approve** a pending TUTEE post (476120b8) → confirm dialog "Approve this post?" → "Confirm Approval" →
  back to queue, count 76→75. ✓
- **Reject** a pending TUTOR post (1d50049c) → dialog with **"Reason *"** (6 presets + Additional notes);
  **"Confirm Rejection" is DISABLED until a reason is chosen** (reason-guard present ✓). Picked
  "Duplicate listing" → button enabled → Confirm → back to queue, count 75→74. ✓ (admin-09)

---

## Evidence
intuition-spec/specs/issues/review-2026-06-23-postfix/evidence/
- admin-01-review-queue png — pagination + filter + MATH OLEVEL row
- admin-02-markpaid-confirm-dialog png — the irreversible-confirm dialog naming amount/tutor/tutee
- admin-03-markpaid-confirmed-paid png — row flipped to Completed/Paid after Confirm
- admin-04-commission-ledger png — ledger (Outstanding S$0, 50% column)
- admin-05-analytics-30d.png / admin-06-analytics-7d.png — labels + date-range change
- admin-07-interviews-job-column png — Job column populated + inline Schedule
- admin-08-nav-390px.png — all 8 admin sections + scroll cue at 390px
- admin-09-reject-reason-guard png — Reject reason-guard (Confirm disabled)
- admin-10-raw-enum-math-olevel png — raw MATH OLEVEL on the detail page (OPS-01)

**Data mutated (expected on dev):** paid 1 commission (match abd59357), approved 1 post (476120b8),
rejected 1 post (1d50049c, reason "Duplicate listing").
