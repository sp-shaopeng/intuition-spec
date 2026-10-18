# InTuition — Commission Module — Frontend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Mark PAID confirms agency receipt of tutor commission, not parent lesson payment, and must use the amount/parties confirmation and cancel-no-mutation guards from [017](../../017-admin-analytics-and-financial-safety/requirements.md)/[019](../../019-admin-money-safety-and-analytics/spec.md). Existing snapshot/lock point is activation. INVOICED/CONFIRMED/DISPUTED/WAIVED flows below remain draft scope awaiting reconciliation; no HISTORICALLY MARKED IMPLEMENTED label elsewhere silently approves or removes them. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Module:** commission  
**Version:** 1.2.0
**Status:** Draft  
**Last Updated:** 2026-09-05
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Routes Overview

| Route | Auth | Description |
|---|---|---|
| /admin/commissions | ADMIN | Commission dashboard (KPI + list) |
| /admin/commissions/ id | ADMIN | Commission detail + action management |
| /admin/commissions/ id /invoice | ADMIN | Printable invoice |
| /admin/system/config | ADMIN | System config (commission rate) |
| /dashboard/commission | TUTOR | Tutor's own commission history |

---

## 2. Commission Dashboard (/admin/commissions)

### 2.1 KPI Cards Row


The illustration for kpi cards row contains the following information:

- Total Earned Outstanding Avg. Days to Pay This Month.
- S$12,450 3 invoices 8 days S$2,100.
- All time S$1,350 pending last 90 days May 2025.


### 2.2 Filters

Status filter tabs: All | Awaiting Confirmation | Confirmed | Invoiced | Paid | Disputed | Waived

Date range picker (matched at or invoice generated at).

### 2.3 Commission Table


The illustration for commission table contains the following information:

- Invoice # Tutor Level/Subject Monthly Rate Commission Status Action.
- TCA-2025-012 John T. Phys • S4 Exp S$800/mo S$400 INVOICED [View].
- TCA-2025-011 Sarah L. Chinese • P6 S$600/mo S$300 PAID [View].
- — David C. English • P5 S$500/mo S$250 AWAITING CONF [View].
- — Amy W. Math • S3 Exp S$700/mo S$350 CONFIRMED [View].


**Status Badges:**
- PENDING — gray
- AWAITING CONFIRMATION — amber "Awaiting Confirmation"
- CONFIRMED — blue "Ready to Invoice"
- INVOICED — orange "Invoice Sent"
- PAID — green "Paid"
- DISPUTED — red "Disputed"
- WAIVED — gray strikethrough "Waived"

---

## 3. Commission Detail Page (/admin/commissions/ id)

### 3.1 Header


The illustration for header contains the following information:

- Commission: TCA-2025-012 [INVOICED].
- Match: John T. → Physics (Sec 4) Matched: 22 May 2025.
- Monthly Rate: S$800/hr Commission Rate: 50%.
- Commission Amount: S$400.00 Invoice: TCA-2025-012.


### 3.2 Status Timeline


The illustration for status timeline contains the following information:

- ● Commission Created 22 May 2025 (match activated).
- ● Confirmation Email Sent 22 May 2025 (to tutee).
- ● Session Confirmed 24 May 2025 (by tutee).
- ● Invoice Generated 27 May 2025 (by Admin A).
- ○ Payment Received Pending.


### 3.3 Action Buttons (context-sensitive)

**AWAITING CONFIRMATION:**
- Resend Confirmation Email — re-queue N-12 to tutee
- Override: Mark Confirmed — admin manually confirms (with reason field), bypass tutee confirmation

**CONFIRMED:**
- Generate Invoice → opens invoice preview, then Send Invoice
- Override Amount — admin can enter custom commission amount (with reason)
- Waive Commission → modal with reason field

**INVOICED:**
- Mark as Paid → date picker (payment date)
- Mark as Disputed → modal with dispute notes field
- Waive Commission → modal

**PAID:** Read-only. Show payment date. Download Invoice.

**DISPUTED:**
- Resolve Dispute: Mark Paid
- Resolve Dispute: Waive

**WAIVED:** Read-only. Show waiver reason and admin.

### 3.4 Override Amount Form


The illustration for override amount form contains the following information:

- Override Commission Amount.
- Calculated amount: S$400.00 (50% of S$800/mo).
- Override amount: [S$_____].
- Reason (required): [________________________].
- [Cancel] [Apply Override].


---

## 4. Invoice Page (/admin/commissions/ id /invoice)

Printable HTML. Print-friendly CSS (@media print). Designed to print as A4.


Implementation example omitted. This public edition retains the written requirements in this section.


**Print button:** Print Invoice / Download PDF (browser print-to-PDF).

---

## 5. System Config (/admin/system/config)

Simple key-value config editor for admin.


The illustration for . system config /admin/system/config contains the following information:

- System Configuration.
- Commission Rate (default).
- Current: 50% [Edit].
- Note: Changing this rate affects new commissions only.
- Existing commissions use the rate at time of creation.


Edit opens an inline form with a number input (percentage) and save/cancel buttons.

---

## 6. Tutor Commission Dashboard (/dashboard/commission)

Tutor's own view — no admin fields, no other tutors' data.


The illustration for . tutor commission dashboard /dashboard/commission contains the following information:

- My Commission History.
- Invoice # Subject/Level Amount Status Date.
- TCA-2025-012 Physics • S4 Exp S$400 INVOICED 27 May.
- TCA-2025-008 Math • S3 Exp S$350 PAID 10 Apr.


Click invoice row → show invoice detail (same layout as admin invoice, but without edit controls).

Download Invoice button on paid invoices.

**Outstanding banner** (shown when INVOICED commission exists):


The illustration for . tutor commission dashboard /dashboard/commission contains the following information:

- ⚠️ You have an outstanding invoice of S$400.00 (TCA-2025-012).
- Please pay by 10 Jun 2025 to remain in good standing.
- [View Invoice].
