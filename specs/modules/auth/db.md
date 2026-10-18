# InTuition — Auth Module — Database Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

Treat the schema database structure definitions below as the dated baseline, not a migration plan. The explicit [plaintext-email-phone amendment](../../architecture/plaintext-email-phone.md) replaces user email/phone encryption/keyed message authentication fields with normalized plaintext contacts. Retain separately scoped secret protection and pending erasure/retention constraints; add the effective [M18 profile-canonical contract](../../PRD.md), [004 HttpOnly refresh/logout contract](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) and [012 pending contact/revoke contract](../../012-tutor-apply-and-profile-ux/design.md). Omitted legacy columns do not remove later requirements; all actual schema edits still require Liquibase.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** auth  
**Version:** 1.1.0  
**Status:** Draft  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 1. Tables

### 1.1 users

Central identity table: the current contact fields are normalized plaintext email/phone under the founder amendment. The encrypted/keyed message authentication database structure definitions below is the historical baseline, not the current migration plan.


Implementation example omitted. This public edition retains the written requirements in this section.


**Row filtering:** All application queries append when deleted at IS no value unless the query is specifically for the Personal Data Protection Act anonymisation job.

---

### 1.2 tutor profiles

Extended profile data for TUTOR-role users. Created in the same transaction as the users row at registration.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.3 tutee profiles

Extended profile data for TUTEE-role users.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.4 password reset tokens

Short-lived one-time passcode tokens for the forgot-password flow.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.5 data erasure log

Immutable record of every Personal Data Protection Act anonymisation run. Created by the scheduled anonymisation job (T+30 days after users deleted at). Referenced by AC-AUTH-15.


Implementation example omitted. This public edition retains the written requirements in this section.


---

### 1.6 commission acknowledgements

Immutable record of tutor's acceptance of commission terms (BR-009). Required before TUTOR registration completes.


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 2. Indexes


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 3. Constraints


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Triggers

### 4.1 updated at auto-update


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 5. Row-Level Security / App-Level Filtering

No PostgreSQL RLS is used (application handles role-based data filtering at the service layer). The application always appends AND deleted at IS no value to user queries. The one exception is the Personal Data Protection Act anonymisation job which runs with a dedicated database role (anonymisation role) that can read soft-deleted rows.

---

## 6. Permissions


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 7. Redis Session Structure (Admin Idle Timeout)

Admin sessions use Redis in addition to signed session token for idle timeout enforcement.

**Key:** the administrator’s server-managed session record (session Id is embedded in the signed session token)  
**Value:** the user identifier, role and last activity time (JSON string)  
**expiry period:** 1800 seconds (30 minutes), reset on every authenticated request  

When with Auth middleware runs for an ADMIN request:
1. Verify signed session token signature + expiry
2. the administrator’s server-managed session record from Redis
3. If key missing or expired → 401, clear cookie
4. If key exists → the administrator’s server-managed session record (sliding window reset)

---

## 8. Seed database query language (Admin Account)

The ADMIN account is never created through the public registration endpoint. It is seeded once during initial deployment:


Implementation example omitted. This public edition retains the written requirements in this section.


**Post-seed checklist:**
- [ ] Admin logs in, sets strong password
- [ ] Admin sets up time-based one-time passcode via authenticator app
- [ ] mfa enabled confirmed TRUE in database before going live
- [ ] Seed script removed from deployment pipeline
