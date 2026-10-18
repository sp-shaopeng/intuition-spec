# DEV Sample-Data Seeder

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 correction:** User-contact fixtures use plaintext fields and current shared taxonomy. Historical sample counts/password examples apply only to their documented dev scope;018/023 own acceptance. [Current requirements](../CURRENT-REQUIREMENTS.md). No runtime verification was performed by this documentation correction.

A flag-gated, DEV-only seeder that populates a local stack with realistic dummy data
(tutors, tutees/parents, posts, applications, interviews, matches, and reviews) so the
showcase, browse, admin queue, and dashboards look alive without manual data entry.

- Source: Dev Data Seeder implementation reference
- Runs as an Application Runner on backend startup.

## Why the application seeder owns account fixtures

The current user-contact model uses normalized plaintext email and nullable phone under the explicit storage amendment. Passwords still require the application’s password encoder. Seed accounts and their object graph through the guarded application seeder so normalization, hashes, defaults and business relationships remain consistent.

Reference/taxonomy data belongs in Liquibase migrations. Sample accounts are local-only; production uses027’s separate idempotent admin bootstrap. The actor counts below describe the older fixture snapshot; current minimum diversity and date-window scenarios are defined by018/023, including recent and31–90-day records for both users and posts. Do not reproduce encrypted-contact/keyed message authentication fixtures.

## How to enable

The seeder is gated on **two** conditions, both required:

1. The Spring local profile is active (the Docker backend runs local; prod never does — there
   is no dev profile).
2. The flag seed sample-data enabled is yes. **It defaults to no** and must be turned on
   explicitly. (This is distinct from the super-admin seeder, which is always on in local.)

Toggle at deploy via either:


Implementation example omitted. This public edition retains the written requirements in this section.


It is **inert in prod** — the local profile never activates there, so the bean is never created.

## What it creates

| Entity | Count | Notes |
|---|---|---|
| Tutors (TUTOR) | 8 | each with a Tutor Profile (featured consent yes) + 1 **ACTIVE** Tutor Post (varied subject, level, rate range, region, available for students yes) |
| Tutees / Parents (TUTEE) | 8 | each with a Tutee Profile + 1 **ACTIVE** Tutee Post (varied subject, level, budget, region, frequency) |
| Job applications | 5 | tutors applying to tutee posts; mix of APPLIED / SHORTLISTED |
| Interviews | 2 | both carry tutee post id; one REQUESTED, one SCHEDULED; kept well under the ≤3-per-posting cap |
| Matches | 2 | one COMPLETED, one ACTIVE |
| Reviews | 2 | both directions on the COMPLETED match (5-star) |

Seed user emails follow REDACTED email … REDACTED email and
REDACTED email … REDACTED email. All seeded users have password change required no.

A single summary line with **counts only** is logged on seed. Passwords are **never** logged.

## Shared dev login password

All seeded sample users share one known-weak dev password:


The illustration for shared dev login password contains the following information:

- [REDACTED PASSWORD].


(Defined as the SEED PASSWORD constant in Dev Data Seeder implementation reference.) These are throwaway dev
credentials — never use them outside a local stack.

## Idempotency & safety

- **Idempotent:** keyed on the sentinel tutor REDACTED email's email hmac. If that user
  already exists, the entire seed is a no-op (safe across restarts).
- The seed is additionally wrapped to swallow Data Integrity Violation Exception, so a race,
  restart, or a soft-deleted seed row (invisible to the lookup but still holding the unique
  email hmac) degrades to a no-op rather than crashing startup.
- **Never for prod.** the profile operation + the default-off flag guarantee it cannot run in a
  production deployment.
