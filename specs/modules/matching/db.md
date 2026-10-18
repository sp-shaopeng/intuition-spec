# InTuition — Matching Module — Database Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** M18/[INT-034](../../architecture/int-034-free-market-assign.md) make tutor user id canonical and tutor post id optional. Agency commission receipt completes the match; [INT-093](../../architecture/int-093-batch-state-machine-integrity-spec.md) governs payable and pre-completion cancellation states. The old mandatory-listing database query language is a historical schema sketch; use the current Liquibase ledger for changes. This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Module:** matching  
**Version:** 1.2.0
**Status:** Draft  
**Last Updated:** 2026-09-05
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Tables

### 1.1 match status Enum


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.2 matches

**Current identity correction:** tutor user id is the required tutor reference; tutor post id is an optional showcase attachment under INT-034. M18's active-job assignment/cardinality rules govern. The full database query language sketch below is historical; do not execute it or treat its old required listing/index shape as the current migration contract.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.3 Partial Unique Index (Cardinality Enforcement)

This is the most critical index in the matching module. It enforces the business rule that a tutee post can have at most ONE PENDING or ACTIVE match at a time.


Implementation example omitted. This public edition retains the written requirements in this section.


If the create operation attempts to insert a second PENDING/ACTIVE match for the same tutee post, PostgreSQL raises a unique constraint violation, which the service layer catches and converts to a 409 TUTEE ALREADY MATCHED error.

Tutor posts are NOT similarly constrained — a tutor may be matched with multiple tutees simultaneously.

---

### 1.4 session confirmation tokens

Stores hashed tokens for tutee first-session confirmation (BR-010). One token per match activation.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2. Indexes


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 3. State Machine Trigger


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Permissions


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 5. Contact Access Logging

When admin views match detail (which includes decrypted contact info), the service logs to audit log:


Implementation example omitted. This public edition retains the written requirements in this section.


This ensures every contact reveal is traceable to an admin user and timestamp.
