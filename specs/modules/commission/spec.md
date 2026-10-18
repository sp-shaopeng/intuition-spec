> **Current-file check, 18 October 2026:** The application currently supports pending commission and staff recording payment. The first-session confirmation, invoice and due-handling stages described here are not implemented (G02). See the [alignment review](../../../docs/current-file-alignment.md). The requirements remain unchanged.

# InTuition — Commission Module — Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The dispute PAID guard and amount calculation below are corrected to confirmed agency-receipt semantics and [INT-157 v2](../../architecture/int-157-commission-first-month-locked-spec.md). Keep first-session confirmation and invoice obligations. A won-but-unpaid dispute is not PAID; its post-resolution unpaid status, partial payments/refunds and later adjustment rules remain explicitly in [open decisions](../../CURRENT-REQUIREMENTS.md). No received revenue is fabricated or silently erased.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

## 1. Overview

The Commission module manages the revenue stream of the Singapore tuition agency marketplace. The agency earns a one-time commission from tutors when a tutoring arrangement begins. The module covers commission creation, tutee confirmation (BR-010), invoice generation, payment tracking, dispute handling, waiver, and financial reporting.

---

## 2. Business Model

### 2.1 Commission Structure

| Parameter | Value |
|---|---|
| Who pays | Tutor |
| When it applies | First confirmed tutoring arrangement per match |
| Amount basis | Agreed monthly tuition fee (SGD) |
| Default rate | 50% of first month's fee |
| Rate configurability | Admin can change via system config (commission rate default) |
| Currency | Singapore Dollar (SGD) |

**Formula:**


The illustration for commission structure contains the following information:

- commission amount = monthly rate sgd × commission rate.


If an admin override is set (override amount sgd has a value), the override amount is used for invoicing instead of the calculated amount.

### 2.2 Commission Acknowledgement (BR-009)

Tutors must explicitly acknowledge the commission terms during registration before they can be matched with tutees. This acknowledgement is recorded in commission acknowledgements with the terms version, timestamp, and IP address for audit purposes. Tutors who have not acknowledged current terms cannot progress past the PENDING match status.

---

## 3. Commission Lifecycle

### 3.1 When a Commission Record is Created

A commission record is created at match activation. INT-157 v2 locks the agreed first-month plan and whole-SGD
fee/commission snapshot, using the current50% rule with its explicit activation override. A later plan or configuration
change does not retroactively alter that snapshot. First-session confirmation/invoice progression remains required.

### 3.2 State Machine


The illustration for state machine contains the following information:

- PENDING.
- ▼ (admin activates match tutee confirmation email sent — BR-010).
- AWAITING CONFIRMATION.
- ▼ (tutee confirms first session occurred).
- CONFIRMED.
- ▼ (admin generates invoice).
- INVOICED.
- PAID DISPUTED WAIVED.
- ▼ (admin resolves dispute).
- PAID or WAIVED.


#### State Definitions

| State | Meaning |
|---|---|
| PENDING | Commission record created; tutee confirmation email not yet sent |
| AWAITING CONFIRMATION | Tutee confirmation email sent (N-12, via matching module); waiting for tutee to confirm first session |
| CONFIRMED | Tutee has confirmed the first session took place; commission is payable |
| INVOICED | Admin has generated a formal invoice; tutor has been notified (N-14) |
| PAID | Agency has actually received the tutor commission; admin records receipt and tutor is notified |
| DISPUTED | Dispute has been opened (from INVOICED or PAID); admin notified (N-16) |
| WAIVED | Admin has waived the commission with a recorded reason |

#### Valid Transitions

| From | To | Actor | Guard / Trigger |
|---|---|---|---|
| PENDING | AWAITING CONFIRMATION | System | Match activated; tutee confirmation email dispatched |
| AWAITING CONFIRMATION | CONFIRMED | Tutee (via confirmation link) or Admin | Tutee clicks confirmation link OR admin manually confirms |
| CONFIRMED | INVOICED | Admin | Admin clicks "Generate Invoice" action |
| INVOICED | PAID | Admin | Admin marks as paid (after receiving payment) |
| INVOICED | DISPUTED | Admin or Tutor | Dispute raised with mandatory dispute notes |
| INVOICED | WAIVED | Admin | Admin waives with mandatory waiver reason |
| CONFIRMED | WAIVED | Admin | Admin waives before invoicing |
| AWAITING CONFIRMATION | WAIVED | Admin | Admin waives before tutee confirms (e g., match cancelled) |
| PENDING | WAIVED | Admin | Admin waives before confirmation email sent |
| DISPUTED | PAID | Admin | Agency actually received the tutor commission and the admin records that receipt; a favourable ruling alone is insufficient |
| DISPUTED | WAIVED | Admin | Admin resolves dispute in tutor's favour |
| PAID | DISPUTED | Admin | Post-payment dispute (e g., chargeback) |

**Invalid transitions are rejected with web request protocol 422 Unprocessable Entity.**

---

## 4. Commission Calculation

### 4.1 Standard Calculation


The illustration for standard calculation contains the following information:

- calculated amount sgd = monthly rate sgd × commission rate.


INT-157 v2 supersedes the earlier generated/recalculable DECIMAL design: compute the agreed first-month
fee and 50% commission at activation, store whole-SGD integer amounts, and keep the plan inputs immutable
thereafter. hours Per Session retains fractional hours; legacy plan fields may be null. Subsequent lesson-plan
changes do not recalculate the locked first-month commission. The hard-rate clamp and 4.33-versus-4 monthly
constant remain explicitly pending decisions, not an invitation to mutate a snapshot.

### 4.2 Admin Override

The existing activation-time commission override remains subject to INT-157 v2's valid range (1 through the
agreed monthly fee). Display the locked basis and final amount together. This does not authorize editing the
locked amount after activation; later adjustments/refunds need a separate approved policy.

**Display rule:** If override amount sgd has a value, show both amounts in the user interface — "Calculated: SGD X XX | Override: SGD Y YY".

### 4.3 Commission Rate Changes

- The current agreed-plan requirement uses 50%, with the explicit activation override described above.
- A future change to the default commission percentage requires an approved commercial amendment.
- Existing locked commission snapshots remain unchanged; audit any approved future configuration change.

---

## 5. Invoice Generation

### 5.1 Trigger

Admin manually triggers invoice generation from the commission detail page once the commission is in CONFIRMED state.

### 5.2 Invoice Number Format


Implementation example omitted. This public edition retains the written requirements in this section.


- TCA = Tuition Centre Agency (fixed prefix)
- YYYY = four-digit calendar year of generation (Singapore time timezone, Asia/Singapore)
- NNNN = zero-padded sequential number, restarting from 0001 each calendar year

**Examples:** TCA-2024-0001, TCA-2024-0042, TCA-2025-0001

**Sequence guarantees:** Invoice number generation uses a database-level transaction lock to prevent gaps or duplicates under concurrent generation. See db md for implementation.

### 5.3 Invoice Contents

| Field | Source |
|---|---|
| Invoice number | Generated at INVOICED transition |
| Invoice date | invoice generated at (Singapore time) |
| Agency name & address | System config — the agency’s registered legal name (UEN the agency’s registration number), the agency’s registered business address (see company implementation reference) |
| Tutor name & contact | From users + tutor profiles |
| Match reference | match id |
| Tutee subject and level | From match / posting |
| Agreed monthly fee | monthly rate sgd |
| Commission rate | commission rate |
| Commission amount | override amount sgd if set, else calculated amount sgd |
| GST | Not applicable (agency below GST threshold — update if registered) |
| Payment instructions | Pay Now UEN + bank transfer details |
| Payment due date | 14 days from invoice date |
| Late fee clause | Stated in invoice footer |

### 5.4 Payment Methods (Singapore Context)

- **Pay Now** — UEN-linked, instant transfer; preferred
- **Bank transfer** — FAST / GIRO to agency bank account
- Physical cheque not accepted

---

## 6. Payment Tracking

Admin receives payment notification (out-of-band — bank notification, Pay Now receipt) and manually marks the commission as PAID in the admin dashboard. The paid at timestamp is recorded. Notification N-15 is dispatched to the tutor confirming receipt.

**Overdue reminder (N-17):** A scheduled job runs daily at 09:00 Singapore time. Any commission in INVOICED status with invoice generated at older than 30 days triggers N-17 to the tutor. The reminder is sent at most once per 7-day window per commission to avoid spam.

---

## 7. Dispute Handling

### 7.1 What Constitutes a Dispute

- Tutor disputes the commission amount (e g., claims incorrect monthly rate was recorded)
- Tutor disputes that the first session occurred (conflicts with tutee confirmation)
- Post-payment dispute (e g., payment made in error)

### 7.2 Raising a Dispute

- Either admin or tutor can raise a dispute.
- dispute notes is mandatory (minimum 10 characters).
- Dispute can be raised from INVOICED or PAID status.
- Admin receives notification N-16 on dispute creation.
- disputed at is recorded.

### 7.3 Resolution

Admin reviews and resolves by transitioning to:
- PAID only when the agency has actually received the tutor commission. A ruling in the agency's favour
  with money still owed must remain uncollected; record the ruling separately. The exact post-dispute unpaid
  status/recovery process is pending business definition and must not be guessed as PAID.
- WAIVED — tutor's favour (commission cancelled, waiver reason required)

No automated resolution — all disputes require human admin decision.

---

## 8. Waiver

Admin may waive a commission at any pre-PAID status (except from PAID itself):

- Valid from: PENDING, AWAITING CONFIRMATION, CONFIRMED, INVOICED, DISPUTED
- waiver reason is mandatory (minimum 10 characters)
- waived at timestamp is recorded
- Waived commissions are excluded from revenue totals but remain visible in dashboard with a strikethrough badge

---

## 9. Commission Acknowledgement (BR-009)

### 9.1 Requirement

Before a tutor account can be activated for matching, the tutor must accept the commission terms. The terms are versioned by date string (e g., 2024-01-01).

### 9.2 Record

Each acceptance is recorded in commission acknowledgements:
- user id — the tutor
- version — the terms version accepted
- acknowledged at — timestamp
- ip address — for Personal Data Protection Act and audit purposes

### 9.3 Re-acknowledgement

If commission terms change (new version published), tutors must re-acknowledge before their next match is activated. The system checks whether the tutor has acknowledged the current active version (stored in system config as commission terms version).

---

## 10. Commission Rate Configurability

The default commission rate is stored in system config under the key commission rate default as a decimal string (e g., 0.5000). Admin can update this via the System Config page. Changes are audit-logged via updated by and updated at on the system config table. The rate must be between 0.0001 and 1.0000 (exclusive upper bound enforced at application programming interface layer).

---

## 11. Dashboard KPIs

| KPI | Calculation |
|---|---|
| Total Earned (SGD) | Sum of effective amount where status PAID |
| Outstanding Invoices | Count where status INVOICED |
| Overdue Invoices | Count where status INVOICED AND the invoice was generated more than 30 days ago |
| Active Disputes | Count where status DISPUTED |
| Average Days to Payment | AVG using paid at - invoice generated at where status PAID |
| Match Success Rate | (Commissions reaching PAID or INVOICED) / (Total commissions) × 100% |
| Average Commission (SGD) | Agency commission actually received divided by distinct matches contributing those receipts; unpaid INVOICED debt is excluded. Exact date cohort/anchor remains pending in [017 REQ-002](../../017-admin-analytics-and-financial-safety/requirements.md) / CURRENT O02. |
| Monthly Revenue (SGD) | Sum received commission by the month of payment receipt within the current year |

effective amount = the override amount when one is recorded; otherwise, the calculated amount

---

## 12. Personal Data Protection Act Compliance (Singapore)

- Commission records contain financial data linked to identified individuals; stored and processed under Personal Data Protection Act.
- ip address in commission acknowledgements is considered personal data; retained per Personal Data Protection Act retention policy (typically 5 years for financial records).
- Invoice PDFs containing tutor personal data must not be stored in publicly accessible storage; use signed URLs with expiry.
- Dispute notes may contain personal information — access restricted to admin roles only.

---

## 13. Acceptance Criteria

### AC-COMM-01 — Automatic Commission Creation
**Given** a match transitions to ACTIVE status,  
**When** the match activation completes,  
**Then** a commission record is created with status PENDING, commission rate copied from system config commission rate default, and monthly rate sgd set to the agreed fee.

### AC-COMM-02 — Awaiting Confirmation Transition
**Given** a commission is in PENDING status,  
**When** the admin activates the match and the tutee confirmation email is dispatched,  
**Then** the commission transitions to AWAITING CONFIRMATION.

### AC-COMM-03 — Tutee Confirmation (BR-010)
**Given** a commission is in AWAITING CONFIRMATION status,  
**When** the tutee clicks the confirmation link (idempotent),  
**Then** the commission transitions to CONFIRMED and updated at is refreshed.

### AC-COMM-04 — Invoice Generation
**Given** a commission is in CONFIRMED status,  
**When** an admin triggers "Generate Invoice",  
**Then** the commission transitions to INVOICED, invoice number is assigned in an invoice identifier containing TCA, the year and a four-digit sequence number format, invoice generated at is set, and notification N-14 is sent to the tutor.

### AC-COMM-05 — Invoice Number Uniqueness
**Given** two admins simultaneously trigger invoice generation for different commissions in the same year,  
**When** both requests complete,  
**Then** each commission receives a unique, non-duplicate invoice number with no gaps.

### AC-COMM-06 — Mark as Paid
**Given** a commission is in INVOICED status,  
**When** an admin marks it as paid,  
**Then** the commission transitions to PAID, paid at is set, and notification N-15 is sent to the tutor.

### AC-COMM-07 — Dispute Creation
**Given** a commission is in INVOICED or PAID status,  
**When** a dispute is raised with dispute notes provided,  
**Then** the commission transitions to DISPUTED, disputed at is set, and notification N-16 is sent to admin.

### AC-COMM-08 — Dispute Without Notes Rejected
**Given** an attempt to dispute a commission,  
**When** dispute notes is empty or fewer than 10 characters,  
**Then** the application programming interface returns web request protocol 422 with a validation error.

### AC-COMM-09 — Waiver With Reason
**Given** a commission is in any pre-PAID status,  
**When** admin waives it with a waiver reason,  
**Then** the commission transitions to WAIVED and waived at is set.

### AC-COMM-10 — Waiver Without Reason Rejected
**Given** an attempt to waive a commission,  
**When** waiver reason is absent or fewer than 10 characters,  
**Then** the application programming interface returns web request protocol 422 with a validation error.

### AC-COMM-11 — Invalid Transition Rejected
**Given** a commission is in PAID status,  
**When** an admin attempts to generate an invoice,  
**Then** the application programming interface returns web request protocol 422 with message "Invalid status transition".

### AC-COMM-12 — Commission Rate Snapshot
**Given** the default commission rate is changed in system config,  
**When** viewing an existing commission created before the change,  
**Then** the existing commission retains its original commission rate and calculated amount sgd unchanged.

### AC-COMM-13 — Admin Override Amount
**Given** an admin sets override amount sgd on a commission,  
**When** the invoice is generated,  
**Then** the invoice shows the override amount, with the calculated amount shown as reference.

### AC-COMM-14 — Overdue Reminder
**Given** a commission has been in INVOICED status for 30+ days,  
**When** the daily 09:00 Singapore time scheduler runs,  
**Then** notification N-17 is sent to the tutor, and a subsequent run within 7 days does not resend.

### AC-COMM-15 — Commission Acknowledgement Required (BR-009)
**Given** a tutor has not acknowledged the current commission terms version,  
**When** an attempt is made to activate a match for that tutor,  
**Then** the activation is blocked with error "Commission terms acknowledgement required".

### AC-COMM-16 — Tutor Views Own Commissions
**Given** a tutor is authenticated,  
**When** they access /dashboard/commission,  
**Then** they see only their own commission records, with no access to other tutors' data or admin-only fields.

### AC-COMM-17 — One Commission Per Match
**Given** a match has an existing commission record,  
**When** an attempt is made to create another commission for the same match,  
**Then** the database constraint prevents duplicate creation and an error is returned.

### AC-COMM-18 — Invoice PDF Security
**Given** an invoice PDF is requested,  
**When** the URL is generated,  
**Then** it uses a signed URL with maximum 1-hour expiry, not a publicly accessible permanent URL.

### AC-COMM-19 — PAID to DISPUTED Transition
**Given** a commission is in PAID status,  
**When** a post-payment dispute is raised with valid dispute notes,  
**Then** the commission transitions to DISPUTED and admin is notified via N-16.

### AC-COMM-20 — Stats Endpoint Accuracy
**Given** commissions exist in various states,  
**When** the admin commissions stats interface operation endpoint is called,  
**Then** totals exclude WAIVED commissions, and PAID totals use effective amount (override if set, else calculated).
