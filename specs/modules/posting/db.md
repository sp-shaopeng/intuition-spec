# InTuition — Posting Module — Database Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

Canonical states are the eight in [posting/spec md](spec.md). database query language/trigger examples below using APPROVED/PAUSED are historical; they must not be copied into new migrations. Real schema changes use Liquibase and current canonical enums. This correction makes no assertion about the current deployed schema.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** posting  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Tables

### 1.1 post status Enum

Canonical enum — all modules must use these exact values. posting/spec md is authoritative.


Implementation example omitted. This public edition retains the written requirements in this section.


**Note:** APPROVED and PAUSED FOR REVIEW are NOT used. The approved state is ACTIVE. Edit-while-active re-enters PENDING (not PAUSED FOR REVIEW). Admin requesting changes uses REVISION REQUESTED.

---

### 1.2 subjects Lookup Table


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.3 levels Lookup Table


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.4 tutor posts

Venue types and group sizes stored in junction tables (not arrays) per posting/spec md §3.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.5 tutee posts


Implementation example omitted. This public edition retains the written requirements in this section.


**Venue preferences** for tutee posts live in the tutee post venue preferences junction table (see §1.8 below).

---

### 1.6 tutor post subjects Junction


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.7 tutor post levels Junction


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.8 tutor post venue types Junction


Implementation example omitted. This public edition retains the written requirements in this section.


### 1.9 tutor post group sizes Junction


Implementation example omitted. This public edition retains the written requirements in this section.


### 1.10 tutee post venue preferences Junction


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.8 post review log

Immutable audit trail of all admin approval/rejection actions. Also defined in the Approval module — reproduced here for completeness.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2. Indexes


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 3. State Machine Trigger


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Updated at Trigger

Apply the set updated at operation function (defined in auth module) to both post tables:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 5. Permissions


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 6. Auto-Expiry Scheduler

Spring a scheduled task (cron = "0 0 2 * * ?") Singapore time — triggers internal expiry job.


Implementation example omitted. This public edition retains the written requirements in this section.


expires at is set to approved at + 60 days when a post transitions to ACTIVE.
