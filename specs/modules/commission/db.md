# InTuition — Commission Module — Database Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The seven-state database query language/trigger below is a historical workflow/schema sketch, not a current migration or a complete receipt authorization guard. Keep first-session/invoice obligations from [product requirements document §8.3/§20.3](../../PRD.md). Use [INT-157 v2](../../architecture/int-157-commission-first-month-locked-spec.md) immutable activation-time whole-SGD snapshot, fractional session hours and nullable legacy plan fields. Any transition to PAID requires actual agency receipt; favourable dispute adjudication alone must remain uncollected. How confirmation/invoice/dispute stages map to collection status is distinct from the retained obligations.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** commission  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Tables

### 1.1 commission status Enum


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.2 commissions

One commission record per match. Created automatically when match is activated.


Implementation example omitted. This public edition retains the written requirements in this section.


**Key design note:** calculated amount sgd is a GENERATED ALWAYS AS ... STORED computed column — it cannot be manually updated. When admin overrides, override amount sgd is set; application code uses the recorded override amount when present, otherwise the calculated amount to get the effective amount. This ensures audit trail shows both the original calculated amount and the override.

---

### 1.3 commission acknowledgements

See Auth module db md — reproduced here for module completeness:


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.4 system config

Key-value store for runtime configuration. Currently used for default commission rate.


Implementation example omitted. This public edition retains the written requirements in this section.


**Seed data:**


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.5 Invoice Number Sequence


Implementation example omitted. This public edition retains the written requirements in this section.


Application generates: an invoice identifier containing TCA, the year and a four-digit sequence number

Yearly reset (Spring a scheduled task (cron = "0 0 0 1 1 ?") Singapore time):


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2. Indexes


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 3. State Machine Trigger


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Updated at Trigger


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 5. Permissions


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 6. Audit Log Integration

All commission state changes are logged to audit log:

| Action Code | Trigger |
|---|---|
| COMMISSION CREATED | Match activated |
| COMMISSION AMOUNT OVERRIDDEN | Admin overrides amount |
| INVOICE GENERATED | Admin generates invoice |
| COMMISSION PAID | Admin marks paid |
| COMMISSION DISPUTED | Admin opens dispute |
| COMMISSION WAIVED | Admin waives |
| SYSTEM CONFIG UPDATED | Admin changes commission rate |
