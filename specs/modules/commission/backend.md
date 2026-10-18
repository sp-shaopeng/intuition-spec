# InTuition — Commission Module — Backend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Agency receipt of tutor commission is the PAID event; [017](../../017-admin-analytics-and-financial-safety/requirements.md)/[019](../../019-admin-money-safety-and-analytics/spec.md) require confirmation. Commission inputs lock at activation. Rich invoice/dispute/waiver states remain explicitly unreconciled proposals where no approval/supersession record exists; do not infer their removal from a code snapshot. This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Module:** commission  
**Version:** 1.2.0
**Status:** Draft  
**Last Updated:** 2026-09-05
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. application programming interface Endpoints

All admin commissions * interface operation require ADMIN role. Tutor dashboard endpoints require TUTOR role.

### 1.1 List Commissions (Admin)


The illustration for list commissions admin contains the following information:

- GET admin commissions interface operation.
- Authorization: ADMIN.


**Query Parameters:**


The illustration for list commissions admin contains the following information:

- status commission status filter by status.
- from ISO8601 match created after.
- to ISO8601 match created before.
- cursor string.
- limit integer default 20, max 100.


**Response 200:**


The list commissions admin example describes information including commissions, next Cursor, stats, total Earned Sgd, outstanding Count, outstanding Sgd. Concrete payload values and implementation syntax are omitted.


**Commission Summary data transfer object:**


The list commissions admin example describes information including id, invoice Number, match Id, tutor Name, subject, level, monthly Rate Sgd, commission Rate, calculated Amount Sgd, override Amount Sgd, effective Amount Sgd, status, matched At, invoice Generated At. Concrete payload values and implementation syntax are omitted.


---

### 1.2 Get Commission Detail


The illustration for get commission detail contains the following information:

- GET admin commissions :id interface operation.
- Authorization: ADMIN.


Returns full commission with match details, tutor contact info, status history (from audit log).

---

### 1.3 Override Commission Amount


The illustration for override commission amount contains the following information:

- PATCH admin commissions :id override-amount interface operation.
- Authorization: ADMIN.


**Request Body:**


The override commission amount example describes information including override Amount Sgd, reason. Concrete payload values and implementation syntax are omitted.


**Guards:** Cannot override after PAID status.

**Processing:** Sets override amount sgd, logs COMMISSION AMOUNT OVERRIDDEN to audit log.

**Response 200:** Updated commission data transfer object.

---

### 1.4 Generate Invoice


The illustration for generate invoice contains the following information:

- POST admin commissions :id generate-invoice interface operation.
- Authorization: ADMIN.


**Guards:** Commission must be CONFIRMED status.

**Processing (in a database transaction):**
1. Lock commission row (read fields ... with a record lock)
2. Generate invoice number = an invoice identifier containing TCA, the year and a four-digit sequence number (using sequence, see §4)
3. Update status INVOICED, invoice number, record the current invoice-generation time
4. Log INVOICE GENERATED to audit log
5. Queue N-14 (invoice notification → tutor with amount + payment instructions) to InTuition async email queue (Spring @Async + Redis)

**Response 200:**


The generate invoice example describes information including status, invoice Number, invoice Generated At. Concrete payload values and implementation syntax are omitted.


**Errors:** 409 (invalid state), 409 (already invoiced — idempotency check)

---

### 1.5 Get Invoice (Printable HTML)


The illustration for get invoice printable html contains the following information:

- GET admin commissions :id invoice interface operation.
- Authorization: ADMIN.
- Content-Type: text/html.


Returns rendered HTML invoice (not PDF — client uses browser print-to-PDF). Fetches commission + match + tutor profile data, renders invoice template with all fields.

Also accessible to TUTOR (own commissions only) for downloading paid invoices.

---

### 1.6 Mark as Paid


The illustration for mark as paid contains the following information:

- POST admin commissions :id mark-paid interface operation.
- Authorization: ADMIN.


**Request Body:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Guards:** Commission must be INVOICED or DISPUTED.

**Processing:**
1. Update status PAID, paid at
2. Log COMMISSION PAID to audit log
3. Queue N-15 (payment received confirmation → tutor) to InTuition async email queue (Spring @Async + Redis)

**Response 200:** the paid state and its recorded time

---

### 1.7 Open Dispute


The illustration for open dispute contains the following information:

- POST admin commissions :id dispute interface operation.
- Authorization: ADMIN.


**Request Body:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Guards:** Commission must be INVOICED or PAID.

**Processing:**
1. Update status DISPUTED, record the current dispute-opening time, dispute notes
2. Log COMMISSION DISPUTED to audit log
3. Queue N-16 (dispute opened → admin internal notification) to InTuition async email queue (Spring @Async + Redis)

**Response 200:** the disputed state

---

### 1.8 Waive Commission


The illustration for waive commission contains the following information:

- POST admin commissions :id waive interface operation.
- Authorization: ADMIN.


**Request Body:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Guards:** Commission must NOT be PAID.

**Processing:**
1. Update status WAIVED, record the current waiver time, waiver reason
2. Log COMMISSION WAIVED to audit log

**Response 200:** the waived state

---

### 1.9 Commission Stats (Admin KPIs)


The illustration for commission stats admin kp is contains the following information:

- GET admin commissions stats interface operation.
- Authorization: ADMIN.


**Query Parameters:** from, to (date range for filtering).

**Response 200:**


The commission stats admin kp is example describes information including total Earned Sgd, total Invoiced Sgd, outstanding Count, avg Days To Payment, by Month, by Status, PENDING, AWAITING CONFIRMATION, CONFIRMED, INVOICED, PAID, DISPUTED, WAIVED. Concrete payload values and implementation syntax are omitted.


---

### 1.10 System Config


The illustration for system config contains the following information:

- GET admin system config interface operation.
- PATCH admin system config interface operation.
- Authorization: ADMIN.


GET returns all config key-value pairs.  
PATCH accepts the commission-rate setting and a proposed value of 45% — validates rate is between 0.01 and 1.0.

Changes are logged to audit log with action SYSTEM CONFIG UPDATED.

---

### 1.11 Tutor Commission History


The illustration for tutor commission history contains the following information:

- GET dashboard commissions interface operation.
- Authorization: TUTOR.


Returns commissions for the authenticated tutor's matches. No admin fields. Response includes invoice Number, status, effective Amount Sgd, matched At, invoice Generated At, paid At.

---

## 2. Service Layer


Implementation example omitted. This public edition retains the written requirements in this section.


State transitions are validated before each operation. All writes go to audit log.

---

## 3. Invoice Number Generation

The retained draft requirement is sequential invoice numbers per calendar year with no gaps or duplicates. The historical sequence sketch below now uses one consistent name; it does not itself prove the gapless promise. Before implementing invoice allocation, record the effective invoice workflow and demonstrate concurrent allocation, failed-transaction/retry and Singapore-year rollover behavior without duplicates/gaps. Schema changes use Liquibase; the database query language below is illustrative.


Implementation example omitted. This public edition retains the written requirements in this section.


Application-level generation inside a transaction:


Implementation example omitted. This public edition retains the written requirements in this section.


Historical reset sketch (scheduler registration follows007, not a new raw scheduler): a Spring the scheduled operation job resets the sequence with an idempotency guard to prevent double-reset if the scheduler fires the task twice:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Notifications

| Event | ID | Recipient | Queue |
|---|---|---|---|
| Invoice generated | N-14 | Tutor | InTuition async email queue (Spring @Async + Redis) |
| Payment received | N-15 | Tutor | InTuition async email queue (Spring @Async + Redis) |
| Dispute opened | N-16 | Admin (internal) | InTuition async email queue (Spring @Async + Redis) |
| Overdue reminder (30 days) | N-17 | Tutor | InTuition async email queue (Spring @Async + Redis) — delayed job |

**N-17 scheduling:** On generate-invoice, schedule delayed job an overdue-invoice reminder tied to the commission with 30-day delay via InTuition async email queue (Spring @Async + Redis). On mark-paid, cancel this job.

---

## 5. Commission Rate Lifecycle

The commission rate stored on each commissions row at creation time is the rate that applies forever to that commission — changes to system config commission rate default only affect future commissions. This prevents retroactive rate changes affecting existing invoices.
