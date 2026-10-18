# Architecture change — store email & phone as PLAINTEXT (remove AES-SIV at rest)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** The founder directive changes storage of user email/phone. [M18](../PRD.md)/[INT-125](int-125-batch-closeout-spec.md) later permit assigned-participant contact reveal with ACTIVE/COMPLETED frontend controls; the older counterparty-never phrase is superseded. This does not authorize public contact exposure. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Status:** SPEC — for implementation + security-led review (founder directive, 2026-05-29).
**Driver:** "In the database, email and phone number not encrypted — I want plain text string."
**Supersedes:** ADR-003 (AES-SIV deterministic encryption of email/phone at rest) for these two fields.

## ⚠️ Security / Personal Data Protection Act implication (founder-accepted, recorded here)
Removing AES-SIV means email + phone are stored **in clear** in Postgres. A single database dump,
backup leak, or read-only database compromise now exposes **every user's contact details** with no
crypto barrier — the exact blast radius ADR-003 existed to prevent. Under Singapore Personal Data Protection Act this
materially raises breach severity/notifiability. This is a deliberate, founder-directed accepted
risk. Mitigations that remain mandatory: database access control, encrypted backups, and (before real
users) the prod-hardening gates already tracked.

## Change
- users: replace email encrypted BYTEA + email hmac TEXT + phone encrypted BYTEA +
  phone lookup hash with a required, unique plain-text email field of at most 255 characters and an optional plain-text phone field of at most 32 characters.
- **Migration 0022:** add the plaintext columns; for existing rows, the app can no longer be
  assumed to decrypt in database query language — since only the seeded super-admin exists in dev, the seeder
  re-provisions it with plaintext (wipe-or-recreate is acceptable in dev). Add a unique index on
  email. Drop the four encrypted/keyed message authentication columns (keep them only if a reviewer wants a one-release
  rollback window; default: drop).
- **Entity User:** email/phone plaintext fields replace the four encrypted/hmac fields.
- **Auth/code:** Auth Service build User Account/register/login and all lookups switch from
  Encryption Service encrypt + compute Hmac + exists By Email Hmac/find By Email Hmac to plain
  email (exists By Email/find By Email, case-insensitive normalised lower). Remove email/phone
  encryption + keyed message authentication entirely. Encryption Service stays in the codebase if used elsewhere; if it
  was only for email/phone, leave it unused (don't delete in this slice).
- **Display surfaces unchanged in behavior:** admin/own-profile/match views read plaintext directly;
  the **public M14a showcase stays personal information-free** (no email/phone) — this change does NOT alter the
  personal information-visibility model (own is visible, public is none; assigned-counterparty reveal follows the later M18/INT-125 contract); it only changes *at-rest
  storage*.
- **Seeders** (Super Admin Seeder, Dev Data Seeder): write plaintext email/phone (no encrypt/keyed message authentication).
  Also FIX the existing Dev Data Seeder bugs surfaced this session:
  - use **real** subject/level codes from the subjects/levels reference tables (e g.
    MATH PSLE, ENG OLEVEL, CHEM ALEVEL; levels P5, S3 EXP, JC1) — NOT ENGLISH/PRI5;
  - make seeding **crash-safe**: a seed failure must NEVER prevent app boot (don't rely on a
    commit that happens after the startup routine has returned; force the database work to finish inside the guarded routine, or use an explicitly controlled transaction and
    catch Exception, log, and return). An Application Runner throwing currently kills startup.

## Acceptance
- A user row shows readable email/phone in the database (no bytea). Login by email works; duplicate
  email → 409. Super-admin reseeds with plaintext (REDACTED email).
- Dev Data Seeder, when enabled, seeds 8+8 users + posts/etc. with **valid** subject/level codes and
  **does not crash boot** on any failure.
- Public showcase still exposes no email/phone (regression test). Coverage gates hold.

## Out of scope
- Passwords stay **bcrypt** (unchanged — hashing, not encryption).
- The personal information-visibility user interface model is unchanged.
