# Design — Secure personal information Store & Operations Console (M13)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** The explicit [plaintext-email-phone](plaintext-email-phone.md) directive supersedes encrypted user-contact assumptions only. The folded review still makes a second directory conditional, requires outbox+repair if retained, and keeps fail-closed audit/preventive bulk limits. This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Status:** DEFERRED behind M15; thin slice only, hard-gated (real 17-agent team
review + Boss, 2026-05-29).
**Author:** Solution Architect
**Date:** 2026-05-29

> **Real team-review outcome (supersedes the earlier informal note).** A full
> 17-specialist review + Tech Lead + Boss audit returned **SEND BACK**. Key
> rulings:
> - **Sequence M13 LAST**, after M15. Build only a **thin slice** first: an
>   **audited operator search + single-person contact card**. (Quinn's MVP; Dana
>   would defer entirely behind launch blockers.)
> - **Hard gate — no ops pii personal information backfill in any shared/prod environment until
>   verified database backup + restore exists** (SRE/Dev Ops/Boss). Decrypting all users
>   into a plaintext table on a backup-less database is an unrecoverable, Personal Data Protection Act-reportable
>   risk.
> - **Direct authorized reads are the default** for the contact card after the plaintext-email-phone amendment (decrypt-on-the-fly was the earlier encrypted baseline); a stored plaintext
>   directory is justified **only if partial/substring search is a proven operator
>   need**. The later plaintext email/phone directive supersedes encrypted-user assumptions; a second directory is still not authorized merely because this older design describes one.
> - If a stored directory survives the partial-search test, it ships only via a
>   **transactional outbox with a repair path** — never the aspirational
>   "single-writer + nightly parity" (which detects drift after prod has drifted).
> - **Audit fail-closed:** an ops personal information read returns **503 if the audit write fails**
>   (SRE). A **preventive bulk-read cap that BLOCKS** (not just alerts) is required
>   before any operator can page through the directory (Security).
> - The exfil-detection **metric must actually be emitted and the alert deployed**
>   (intuition ops pii view total + scrape/Alertmanager) — today it is spec-only,
>   so bulk exfiltration would be invisible (Observability).
> - **OPERATOR role does not exist yet** (User Role is TUTEE/TUTOR/ADMIN — verified
>   by Boss); adding it is its own Liquibase ALTER TYPE ... ADD VALUE changeset
>   (run In Transaction is no) plus the Java enum.
> - Erasure/retention must cascade to any plaintext copy; soft-deleted users must
>   purge their directory row (a cross-schema FK won't do it — DBA).
>
> Full records: [reviews/M13-M14-M15-team-review md](reviews/M13-M14-M15-team-review.md).
**Drivers:** Founder request — "We're a tuition agency; we hold a lot of personal
detail for tutors and tutees/parents. Today that data isn't even visible to the
admin or the post owner. I want a safe table/storage for this important data, and
a secure way for the operator/admin to see all of it."

---

## 1. Problem

InTuition is a tuition agency: the operations team must contact and verify real
people (tutors, tutees, parents/guardians). Today:

- **Email and phone are encrypted** (AES-SIV) on users (email encrypted,
  phone encrypted + keyed message authentication). They are unreadable in the database.
- The **only** place decrypted contact surfaces is the admin *pending-post
  detail* endpoint, and ACTIVE-match counterparty email. There is **no**
  consolidated "see this person's full details" view, **no** user search, and
  **no** operator role.
- Other personal information (names, qualifications, bio, relationship, MRT/area, special
  requirements) is already plaintext but **scattered** across
  tutor profiles, tutee profiles, tutor posts, tutee posts and is not
  aggregated anywhere an operator can use it.

Operators therefore cannot do their job (contact, verify, match, support) without
ad-hoc database query language.

## 2. Goal

1. A **dedicated, access-controlled store** for the full personal details of
   every tutor and tutee/parent.
2. A **secure operations console** (frontend entry point + backend endpoints)
   where an authorised operator/admin can **search and view all details** of any
   person, with every access **audited**.
3. An **operator role/agent** that owns this console.

This is tracked as a new module: **M13 — Operations / People Directory.**

## 3. Decision (as ratified by team review)

The founder asked for a real, stored, viewable table of personal details, plus a
secure way for operators to see everything. Security and the DBA vetoed storing
plaintext as columns on users (it would undo the AES-SIV protection from
ADR-003 — one database dump leaks every contact). The ratified design keeps that intent
but **isolates** the plaintext:

- Store the operator-readable contact personal information as **plaintext in a dedicated table in
  a separate Postgres schema ops pii**, with its own restricted grant — NOT as
  columns on users.
- The encrypted users columns remain the **system-of-record** for auth/login.
- **Why store plaintext at all (the decisive justification):** operators must
  search by partial name/email/phone. The keyed message authentication columns only support *exact*
  match; you cannot LIKE %9123% over ciphertext. Searchability is what forces
  a stored plaintext directory — confirmed in review (Conflict 2).

### 3.1 Data model — ops pii contact directory

A single isolated table aggregating the operator-relevant contact identity, one
row per user.


Implementation example omitted. This public edition retains the written requirements in this section.


- **Single-writer rule:** written ONLY through
  the upsert operation, called transactionally from each
  contact mutation (registration, profile/contact update). No scattered writes
  (review Conflict 3 — drift prevention).
- A one-off **backfill** migration decrypts existing users rows (via
  Encryption Service decrypt) to populate the directory.
- **Reconciliation:** a quality assurance test asserts decrypt using users email encrypted  equals 
  contact directory email for every row; a nightly job re-checks parity.
- Profile/post personal information (quals, bio, locations, special requirements) is **not**
  copied — the console reads it live and joins it into the person view. Only the
  contact identity lives in contact directory.

### 3.2 Compensating controls (mandatory — ratified)

- **Schema isolation:** the table lives in its own ops pii schema with a
  dedicated grant, so a leak/dump of the public schema does not expose it.
- **Row-access only via ops endpoints** — never returned by any public, tutee, or
  tutor endpoint.
- **Mandatory audit:** every read (search hit + detail open) writes an
  audit log row (OPS personal information VIEW) with operator id, target user id, fields
  viewed, timestamp, IP. Append-only.
- **Alerting:** Observability raises an alert on OPS personal information VIEW rate spikes
  (bulk-view = potential exfiltration signal).
- **Encryption at rest** stays on users (unchanged).
- **Least privilege:** only the new OPERATOR role (and ADMIN) can reach it.
- **Compliance gate:** BA/legal Personal Data Protection Act sign-off is required **before** the backfill
  runs in any shared environment; erasure + retention jobs must cover
  ops pii contact directory.

### 3.3 Role model

Add **OPERATOR** to User Role (TUTEE, TUTOR, ADMIN, OPERATOR). OPERATOR is
the "admin of the page" persona: full operations console access. ADMIN is a
superset (keeps existing admin powers + ops). Security Config gains:


Implementation example omitted. This public edition retains the written requirements in this section.


OPERATOR accounts are seed/admin-provisioned (no public registration), mirroring
how ADMIN is created today.

### 3.4 Backend — new endpoints (ops interface operation)

| Method | Path | Purpose |
|---|---|---|
| GET | ops people interface operation | Search/list directory (by name/email/phone/role, paginated). Audited. |
| GET | ops people user Id interface operation | Full person view: contact + profile + posts + matches + interviews aggregated. Audited (OPS personal information VIEW). |

New Ops Controller + Ops Directory Service; reuses existing repositories for the
joined profile/post/match/interview data. All paths constants in the backend
route constants, per the no-magic-strings rule.

### 3.5 Frontend — Operations Console entry point

- New guarded area **/ops** (and /ops/people/:id), Protected Route
  roles=["ADMIN","OPERATOR"].
- **Entry point:** a distinct "Operations" link in Nav implementation reference, shown only to
  ADMIN/OPERATOR, leading to the People Directory.
- Pages: Ops People Directory (search + table) and Ops Person Detail
  (consolidated personal information + activity). New ops group in api Routes implementation reference.
- A visible "personal information — handle per Personal Data Protection Act; access is logged" banner on these pages.

### 3.6 Operator agent

Create an **operator agent** definition (the persona that operates this console):
studies current platform behaviour, triages people/records, verifies tutors,
supports parents. Lives alongside the other team agents; documented so the ops
console and the agent stay in sync.

## 4. Alternatives considered

| Option | Verdict in review |
|---|---|
| **Plaintext columns on users** | Earlier rejection superseded by the explicit founder directive in plaintext-email-phone md; this is the current user-contact storage requirement. |
| **Authorized direct reads** | Default contact-card path; no second directory without the proven partial-search need and scoped approval. |
| **Separate ops pii directory** | Conditional only: proven operator partial-search need, role/backup/audit/cap prerequisites and outbox+repair must pass the folded review gate. |

## 5. Personal Data Protection Act / compliance impact

Storing plaintext personal information raises the blast radius of any database compromise and intersects
the roadmap's Personal Data Protection Act items (export/erasure, retention). Erasure must now also purge
contact directory; retention policy must cover it. Legal/BA sign-off required.

## 6. Rollout / migration

1. First establish whether an additional directory is required for proven partial-search needs; otherwise serve the authorized contact card directly from the approved user-contact fields.
2. If the directory is retained, satisfy backup/restore, role, scope and privacy prerequisites before any backfill. Add schema/roles only through reviewed Liquibase changesets.
3. Backfill from the canonical user-contact fields with reconciliation. Write each subsequent change intent to a transactional outbox in the same business transaction; apply idempotently to the directory with bounded retry, operator-visible failure and a repair/replay path.
4. Nightly parity may supplement detection but cannot replace transactional delivery/repair. Never use an untracked single-writer call as the only directory update.
5. Ship only the scoped ops endpoints/console with fail-closed read audit, preventive bulk cap and a demonstrated alert. Preserve the least-privilege operator role decision gate.

## 7. Resolved decisions (from team review)

1. **Storage:** current user-contact plaintext fields follow the later founder directive; an extra isolated directory is conditional under the folded review above, not an unconditional build decision.
2. **Role:** add OPERATOR; ADMIN is a superset.
3. **Post-owner visibility of counterparty personal information:** **out of scope for M13** —
   operators only.
