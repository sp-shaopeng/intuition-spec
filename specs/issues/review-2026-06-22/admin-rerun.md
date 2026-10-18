# Admin / Operator Deep-Flow Re-test — 2026-06-22

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

External reviewer, Admin/operator lens. Driven locally via Playwright MCP. frontend http://localhost, application programming interface http://localhost. Logged in fresh as REDACTED email (super-admin; forced-change gate already cleared → straight to /admin).

Session hygiene note: on first load a **stale TUTOR session** (Mr Tan Wei Jie) was present from a prior run (frontend bootstraps auth from the HttpOnly cookie via read operation for auth me). Cleared it with a cross-site request forgery protection-tokened submit operation for auth logout, then logged in clean as admin. Confirmed identity = ADMIN / is Super Admin true before testing.

---

## 1. Deep flows reached + WORKED / broken

| # | Flow | Result | Evidence |
|---|------|--------|----------|
| 1 | **Commission payment → match completion (KEY)** | ✅ WORKED | 01, 02 |
| 2 | **Interview scheduling (Spec 006)** | ✅ WORKED | 03, 04, 05 |
| 3 | **Match oversight — activate / cancel→reopen** | ✅ WORKED | 06, 07 |
| 3b | **Choose-tutor → selection-request → assign-winner** | ✅ WORKED | 08, 09 |
| 4 | **Approval queue approve + reject** | ✅ WORKED | (verified via application programming interface) |
| 4b | **Analytics date-range** | ✅ WORKED | 10 |
| 5 | **390px on commission / interviews / matching** | ⚠️ nav issue re-confirmed | 11, 12, 13 |

### Flow 1 — Commission payment (the key flow)
- /admin/commission ("Commission Ledger") renders from read operation for admin matches. Before: Total Collected **S$936**, Outstanding **S$120**, 25 matches.
- Target match 57710e7b… (Mr Tan Wei Jie ↔ Mrs Lim) shown **Unpaid**, Monthly Fee S$240/mo, Commission **S$120**, Basis line printed in full: S 40/hr × 4 sessions × 1.5h S 240 → 50% S 120. 50%-of-first-month logic is explicit and correct.
- Clicked **Mark Paid** → immediate flip to **Paid**; Outstanding **S$120 → S$0**, Total Collected **S$936 → S$1,056** (+120). Buttons removed.
- application programming interface verified: match the recorded revision → status: COMPLETED, commission invoice Status: PAID, paid At timestamp set, commission Amount Sgd: 120. **Payment → match completion confirmed end to end; state/audit clean.**

### Flow 2 — Interview scheduling
- Target interview c6320215… (Mdm Hidayah ↔ Mr Tan Wei Jie) found in **"Time agreed"** state (tutor accepted a slot). Detail page shows the full **negotiation timeline**: Round 1 proposed by Tutee, agreed slot "Wed, 24 Jun, 9:00–9:15 (Singapore time) · Agreed", other two "Not chosen".
- **Schedule** dialog: Date & Time (required, datetime-local) + Meeting link (optional); Confirm disabled until date set. Filled 2026-06-24 09:00 + a Google Meet link, confirmed.
- Result: status **"Time agreed" → "Scheduled"**, "24 Jun 2026, 09:00 am Singapore time" + clickable Meeting Link rendered, success toast. submit operation for admin interviews id schedule → 200. New action set appeared (Set Meeting Link / Mark Complete / No Show ×2 / Cancel).
- **WhatsApp OFF**: no client-side send observed (sends would be server-side regardless); nothing errored.

### Flow 3 — Match lifecycle (activate / cancel→reopen)
- Used a non-seeded **Pending Review** match (ff6beda1…, Crown Tutor ↔ Crown Parent) — left the seeded match alone.
- **Activate** opened a proper confirm dialog with editable hourly rate / sessions / hours-per-session + live commission preview (S 45 × 9 × 1.5 S 608 → 50% S 304) and optional override. Confirmed → status **Pending Review → Live**, commission **S$304 (Unpaid)** created, actions became Mark PAID / Cancel. POST …/activate → 200.
- **Cancel** opened confirm: "This will cancel the match. The job will reopen for new applicants." Confirmed → **Live → Cancelled**, commission Cancelled. POST …/cancel → 200.
- application programming interface verified the reopen: match the recorded revision status: CANCELLED, commission CANCELLED, and the underlying job 673690e7… is back to **ACTIVE**. **Cancel→reopen confirmed.**

### Flow 3b — Choose-tutor → assign-winner
- /admin/matching "Pending Selections" region lists parent choose-tutor requests with **Assign tutor & close job** + View applicants.
- Used selection for job the recorded revision (Science PSLE, parent chose tutor wei jie tan). The confirm dialog spelled out the contract: mark tutor as winner, set job "Position filled", notify other applicants their applications are no longer available, undo-by-cancel. Confirmed.
- submit operation for admin selection-requests 96dcac03 assign → **201 Created**. application programming interface verified: new match the recorded revision (PENDING), job the recorded revision → **MATCHED**. **Selection → assign-winner funnel confirmed.**

### Flow 4 — Approval queue
- Approve: opened pending TUTEE post the recorded revision, detail shows full post + admin-only contact personal information (name/email/phone — correct: admin sees all). Inline confirm "post will go live and owner notified by email" → confirmed → application programming interface: post **ACTIVE**.
- Reject: post the recorded revision, structured reason picker (6 canned reasons + optional notes; Confirm disabled until reason chosen). Chose "Missing required information" → confirmed → application programming interface: post **REJECTED**, rejection Reason stored.

### Flow 4b — Analytics date-range
- /admin/analytics renders quick-range buttons (7/30/90/YTD) + From/To date inputs. Default 30-day window 2026-05-24 → 2026-06-22. "Collected in Period" = S$1,056 (reflects my just-paid commission).
- Clicked **Last 7 days** → inputs updated to 2026-06-16 → 2026-06-22 and a fresh read operation for admin analytics?from is 2026-06-16&to is 2026-06-22 → 200 fired. Date-range re-query confirmed.

---

## 2. ISSUES

### INT-RR1 · Interview Funnel "Conversion Rate 400.0%" · **S3**
- **Repro:** /admin/analytics, default 30-day range, Interview Funnel card.
- **Saw:** Requested 1, Scheduled 2, Completed 4, No Show 3, Cancelled 9, **Conversion Rate = 400.0% (Requested → Completed)**.
- **Expected:** A conversion rate is a proportion and should be ≤ 100%. The metric divides period-Completed by period-Requested, but Completed/Requested are counted on different lifecycle timestamps within the window, so the ratio exceeds 100% and reads as nonsense to an operator. Either define conversion on a consistent cohort (e g. of interviews *requested in period*, how many reached Completed) or clamp/relabel.
- **Evidence:** evidence/admin-rerun-10-analytics-daterange png

### INT-RR2 · "Avg Commission / Match" = total ÷ active-matches, misleading · **S4**
- **Repro:** /admin/analytics Revenue card.
- **Saw:** Total Collected S$1,056, Active Matches **1**, **Avg Commission / Match S$1,056** ("Across active matches").
- **Expected:** Average commission per match should divide collected commission by the number of *commissionable/paid* matches (6 completed-paid here), not by the 1 currently-active match — otherwise the average equals the grand total whenever exactly one match is active. Label vs computation mismatch.
- **Evidence:** evidence/admin-rerun-10-analytics-daterange png

### INT-RR3 · Admin section nav is a hidden horizontal-scroll strip at 390px · **S3**
- **Repro:** any /admin/* at 390px width.
- **Saw:** the 8-item admin nav (nav aria-label is Admin sections) sits in a 154px container with overflow-x:auto and scroll Width ≈ 661px. Only "Review" + part of "Matching" are visible; **Jobs, Interviews, Commission, Users, Analytics, Admins are off-screen** (x ≥ 292 > viewport). No hamburger/collapse and no visible scrollbar affordance on mobile.
- **Expected:** A mobile admin should be able to discover/reach all 8 sections (hamburger menu, wrap, or a clear scroll affordance). Re-confirms pass-1's cramped-nav flag — still unresolved.
- **Evidence:** evidence/admin-rerun-11-390-commission png, 12-390-interviews png, 13-390-matching png

### INT-RR4 · "Mark Paid" mutates finance state with NO confirmation · **S3**
- **Repro:** /admin/commission, an Unpaid match → click **Mark Paid**.
- **Saw:** the payment is recorded instantly (Unpaid→Paid, match→COMPLETED) with **no confirm dialog**. Every other state-changing admin action tested has a confirm step (Activate, Cancel, Approve, Reject, Assign-winner, Schedule all gate behind a dialog).
- **Expected:** Recording a commission payment is the highest-consequence, hardest-to-reverse financial action (it COMPLETES the match). It should have at least the same confirm gate as Cancel/Activate. Inconsistent and risky for a misclick.
- **Evidence:** evidence/admin-rerun-01-commission-before png (button), 02-commission-after-paid png (instant result, no dialog)

### INT-RR5 · read operation for admin commission returns 500 instead of 404/405 · **S4** (cosmetic / application programming interface-hygiene)
- **Repro:** the documented operational check/fetch GET http://localhost:8080admin commission interface operation while authed as admin.
- **Saw:** 500 INTERNAL ERROR (generic). Several other bare paths probed the same way (/users/me, /me, /auth/session) also 500.
- **Expected:** an unmapped/var-mismatched collection route should be 404/405, not a 500 swallowed into the generic handler. **Not user-facing** — the Commission page itself is built from read operation for admin matches (200) and renders cleanly with zero console errors — but the 500-on-unknown-route pattern is worth a look (could mask a real missing-value failure/path-variable bug and pollutes error metrics).

---

## 3. Still-blocked / not-covered
- **WhatsApp delivery**: feature is OFF by design; could not observe an actual interview-scheduled WhatsApp send (server-side + disabled). Noted, not testable from the admin user interface in this config.
- **No destructive testing** of the originally-seeded ACTIVE match — it was the commission key-flow target and is now legitimately COMPLETED/PAID. All lifecycle (activate/cancel/assign) testing used other, non-seeded matches/posts.
- Did **not** change the seed admin password (forced-change gate intentionally off for this run).

---

## Coverage summary
All five requested deep-flow areas reached and exercised against the live stack:
1. Commission payment → completion — **WORKED** (S$120, 50% logic, COMPLETED+PAID confirmed via application programming interface).
2. Interview schedule from "Time agreed" — **WORKED** (Scheduled + meeting link; POST 200).
3. Match activate / cancel→reopen — **WORKED** (job reopened to ACTIVE confirmed via application programming interface).
3b. Selection-request assign-winner — **WORKED** (201; job→MATCHED, new PENDING match).
4. Approve + Reject — **WORKED** (post→ACTIVE / REJECTED+reason via application programming interface).
4b. Analytics date-range — **WORKED** (re-queried with new from/to).
5. 390px — nav issue re-confirmed.

**Issues by severity:** S3 ×3 (INT-RR1 conversion>100%, INT-RR3 mobile admin nav, INT-RR4 no-confirm Mark-Paid), S4 ×2 (INT-RR2 avg-commission label/calc, INT-RR5 500-on-unknown-route). No S1/S2 — all operator flows are functional and data-clean.
