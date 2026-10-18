# INT-077 — Marketplace Quota Caps — Feature Spec & Design Record

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Issue:** INT-077 · **Type:** FEATURE (frontend + backend) · **Layer:** Full-stack
**Status:** HISTORICALLY MARKED IMPLEMENTED to dev — code-complete, gate-green, Boss SHIP, pushed (backend the recorded revision + the recorded revision, frontend the recorded revision) · **Date:** 2026-06-06
**Migration:** 0046-quota-cap-indexes implementation reference
**Cross-cutting:** applications (M15) · invitations (INT-044) · interviews (M6)

> **Conventions (binding, from CLAUDE md):** config-driven limits (no magic numbers);
> race-safety enforced in the database; typed 422 error codes on both ends; no magic strings.
> This doc is the single design home for the per-tutor application cap, the per-job invite
> cap, and the race-hardening of the pre-existing interview cap. Endpoint-level detail lives
> in [backend-api implementation reference](../reference/backend-api.md) ("INT-077 — Marketplace
> quota caps").

## 1. Founder ask

A tutee may introduce a job to at most **10 tutors** (cancellable); a tutor may hold at most
**10 active applications** (withdrawable); **everyone sees the quota left clearly in the frontend.**
The tutee cap is **per job posting** (founder-confirmed).

## 2. Cap semantics

| Cap | Scope | Active statuses (consume a slot) | Config key (default) | Over-cap |
|---|---|---|---|---|
| Tutor applications | **per tutor** (all jobs) | OPEN STATUSES = APPLIED + SHORTLISTED | quota applications-per-tutor (**10**) | 422 ERR APPLICATION LIMIT REACHED |
| Tutee invitations | **per tutee post** | PENDING | quota invites-per-post (**10**) | 422 ERR INVITE LIMIT REACHED |
| Interviews (pre-existing) | per tutee post | ACTIVE STATUSES | hard **3** (not config) | 422 INTERVIEW LIMIT REACHED |

- **Withdraw frees a slot.** Application withdraw (→WITHDRAWN) and invite withdraw
  (→WITHDRAWN) both reduce the active count; the mutation response returns the refreshed
  quota so the frontend updates without a re-fetch.
- **Accept bypasses the tutor cap.** The invitation-ACCEPT path
  (the internal application path reached when accepting an invitation) passes
  enforce Cap is no. An accept is the highest-intent funnel step — a near-certain match — so
  it is not blocked by the tutor's application cap. It still serializes on the advisory lock.
- Limits are **config-driven** — re-baselining a cap (e g. a soft tutor limit of 15 in a thin
  market) is a deploy, not a code change.

## 3. Race-safety — advisory lock (the core design)

Simply counting existing rows and inserting when below the limit, under PostgreSQL READ COMMITTED (the project default;
no isolation is configured) is a **TOCTOU race**: two concurrent writers both read 9, both
insert, and the 11th lands. A partial-unique index enforces "≤1", not "≤N", so it cannot
prevent over-cap on its own.

**Fix:** inside the existing database transaction, take a transaction-scoped
pg advisory xact lock on a server-derived parent key **before** the count, then
count→check→insert. The lock auto-releases at commit and serializes only writers on the same
parent key.

| Operation | Lock key |
|---|---|
| Apply | jobapp:`<tutor Id>` |
| Invite | jobinv:`<tutee Post Id>` |
| Interview request | interview:`<tutee Post Id>` |

Keys derive from the authenticated principal / path — **never client input**. SERIALIZABLE
was rejected (no 40001-retry handling exists in the codebase).

**Companion fix (interview cap):** the pre-existing interview cap had the same TOCTOU race
and its cap-index idx interviews tutee post active (migration 0021) had **drifted** from the
5-status ACTIVE STATUSES the count uses, so the count was no longer index-covered. INT-077
applied the same advisory lock (interview:`<tutee Post Id>`) and widened the index predicate to
match ACTIVE STATUSES. See
[max-interviews-per-posting implementation reference](../modules/interview/max-interviews-per-posting.md).

## 4. frontend quota contract — Quota Response

The response contains the quota limit and the remaining allowance, with the latter never below zero. It is
returned on every relevant mutation/read so the frontend can render an authoritative "N of limit
left" chip without a separate fetch, and apply optimistic update + on Error rollback:

| Endpoint | Field |
|---|---|
| submit operation for tutor applications | quota on Application Response |
| read operation for tutor applications | quota on each Application Response row |
| Withdraw an application | Confirmation and the refreshed application quota. |
| submit operation for tutee posts id invite | quota on Job Invitation Response (per-job) |
| Withdraw an invitation | Confirmation and the refreshed invitation quota. |
| read operation for tutee posts id invite-quota | Quota Response (NEW — pre-click chip; owner-checked: 403 non-owner, 404 missing) |
| Read the owner’s applicants | A response containing the applicant page and invitation quota, rather than a bare page. |

Typed 422 codes flow through the shared api Error implementation reference mapper; the 0-state guides the user
("withdraw one to free a slot") rather than dead-ending on a raw 422.

## 5. Migration 0046

- idx jobapp tutor active — partial index ON job applications using tutor id when status IN
  ('APPLIED','SHORTLISTED') (backs the per-tutor application count).
- Invite count **reuses** idx jobinv post status (migration 0035) — no new index for invites.

## 6. Known follow-up

- **INT-079 (S4):** the backend limit-reached error enum has a name≠value divergence —
  ERR INVITE LIMIT REACHED's serialised value keeps the ERR prefix, but
  ERR INTERVIEW LIMIT REACHED's value is INTERVIEW LIMIT REACHED (no prefix). The
  divergence already produced a dead frontend mapper during INT-077. Normalize name  equals  value.
