# M16 — Admin Management & Super-Admin

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** §8 local-only seeding and centralized password-change mutation guards are folded into the opening rule. Human dev credentials remain separate from regression identities.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Status:** BUILT + DEPLOYED (migration 0020) — controllers Admin User Controller (admins) shipped; founder directive 2026-05-29.
**Driver:** "I want the admin to be able to create more admin accounts. There is a default
(super) admin. Seed the database with [REDACTED EMAIL] / admin for the time being."

---

## 1. Access model (ratified by founder)
- **Any ADMIN can CREATE other admin accounts.**
- **Only the SUPER-ADMIN can delete/demote/disable an admin**, and the super-admin row is
  **protected** — it cannot be deleted, demoted, or disabled by anyone (no self-delete either).
- Super-admin is modelled as a **flag** users is super admin BOOLEAN required DEFAULT no,
  NOT a new role — a super-admin is still role ADMIN with the protection + management powers.
  This avoids touching the role enum / every the has role operation check.

## 2. Default super-admin seed (DEV ONLY)
- A **bootstrap seeder** (Spring Command Line Runner/Application Runner) runs **only under the
  exact local profile with seed admin.enabled is yes** (never prod). It is **idempotent**: if a super-admin already exists, do
  nothing.
- Seeds REDACTED email / admin as role is ADMIN, is super admin is yes.
- Email uses the current plaintext-contact amendment with canonical normalization/uniqueness (baseline C13); the password remains BCrypt-hashed through the shared creation path. Legacy AES-SIV/keyed message authentication contact instructions are superseded, not a new migration request.
- password change required yes — the account is forced to change the password before it can
  perform admin actions (see §5). **No multi-factor authentication** (founder: "no multi-factor authentication right now").
- The seeder is gated exactly by the profile operation AND seed admin.enabled; it does not run in test or prod. Use the shared creation path and race-safe canonical-email dedup from §8.
- **Hard guardrail:** these credentials are a known-weak dev bootstrap. The seeder MUST be inert in
  prod; a separate, secure super-admin provisioning path is required before any real deployment
  (flagged for the foundations phase). Acceptable "for the time being" in dev only.

## 3. Data model (migration 0020)
- users is super admin BOOLEAN required DEFAULT no
- users password change required BOOLEAN required DEFAULT no
- Reuse existing deleted at (soft-delete) for disable/remove; reuse existing multi-factor authentication columns.

## 4. Endpoints (admin admins interface operation)
| Method | Path | Who | Purpose |
|---|---|---|---|
| POST | admin admins interface operation | any ADMIN | Create a new admin the account email and temporary password. New admin: role is ADMIN, is super admin is no, password change required is yes. Reuses current registration normalization/password hashing; rejects duplicate canonical email. |
| GET | admin admins interface operation | any ADMIN | List admin accounts (id, display Name, email, is Super Admin, created At, last Login At). |
| DELETE | admin admins id interface operation | **SUPER-ADMIN only** | Soft-delete/demote an admin. **Guards:** cannot target a super-admin; cannot target self; 403 for non-super-admin callers. |

- All under the existing admin ** interface operation → the has role operation matcher; the DELETE adds a
  service-layer super-admin check (and an audit-log entry: ADMIN CREATED / ADMIN REMOVED).
- cross-site request forgery protection applies (mutating). Rate-limit the create endpoint.

## 5. First-login hardening for seeded / newly-created admins
- password change required is yes → login succeeds but the account is gated: the only action
  allowed is change-password (reuse the existing change-password flow); admin application programming interfaces return a
  PASSWORD CHANGE REQUIRED signal until it's cleared.
- **No multi-factor authentication** in this slice (founder directive). Admin login is email + password only. (Any existing
  multi-factor authentication code paths stay feature-flagged off; this feature does not add or require multi-factor authentication.)

## 6. Frontend
- An **Admin → Admins** management page (ADMIN-gated): list admins, a "Create admin" form
  (email, display name, temp password), and a delete/disable action **shown only to the super-admin**.
- A forced **change-password** screen when password change required is set (blocks the rest of
  the admin user interface until done).
- No magic strings: application programming interface ROUTES admin.admins*.

## 8. Review #1 resolutions (binding — fold into the build)

**Seeder gating (Security+SWE):** gate exactly the profile operation (there is NO dev profile; !prod would wrongly fire under test and pollute the coverage build). Docker backend runs local (compose verified) so it DOES populate the founder's stack. Add a second guard: a seed admin.enabled property (default true in local) so it can be switched off. Idempotent: look up the seed by its current canonical email key; wrap the insert and **catch Data Integrity Violation Exception** as a no-op (survives races/restarts and the the database query language restriction operation soft-delete blind spot). Never log the seed/temp password.

**Shared creation path (SWE):** extract a shared method from the private the build user operation that both create-admin and the seeder call. It MUST handle a **null phone** (admins have none; the current plaintext phone column is nullable) — current build User NPEs on null phone. Drop display Name from the admin model (admins have no profile table; the get display name operation returns "Admin"); identify admins by email. Create payload is the account email and temporary password.

**password change required gate (Security+SWE):** a **centralized service-layer guard** (NOT a servlet filter — protects branch coverage) checked per-request against current database state at the top of every admin ** interface operation mutation EXCEPT change-password; returns PASSWORD CHANGE REQUIRED. It must block submit operation for admins too. The existing change-password flow (submit operation for auth change-password) MUST be updated to **clear password change required** on success (today it does not, so the gate would never lift).

**is super admin immutability (Security):** write-once by the seeder. NO application programming interface may set or modify it — no PATCH/PUT on admins, no mass-assignment. This is the only thing preventing privilege escalation.

**Session revocation on remove (Security — real hole):** admin sessions are **stateful in Redis** (session:admin:<session Id>, ~30-min expiry period); soft-delete does NOT revoke them. At admin login, maintain a sessions:by-user:<user Id> set; on DELETE/demote, **purge the target's session:admin:* keys** so a removed admin can't keep acting (and re-mint themselves). Audit ADMIN CREATED/ADMIN REMOVED with **actor id + target id**.

**Rate-limit (Security):** create-admin limited per **acting-admin user Id** (e g. 10/hour), reusing Rate Limit Service.

**last Login At (SWE):** wire it on successful admin login (currently never written) so the list isn't always null; frontend renders null → "Never".

**Frontend contract (the private implementation folder experience):**
- Add is Super Admin and password Change Required to the /auth/me payload AND the login user shape AND the Auth User type — the caller flag is required to conditionally show Delete; row-level is Super Admin from the list disables Delete on protected rows.
- Forced change-password is a **new standalone gated route** (none exists today) wrapping the existing change-password form; intercept at **Admin Layout/hydration** (use Init Auth//auth/me), NOT login on Submit (so refresh/deep-link is gated). After success, re-fetch /auth/me to release the gate.
- **Temp-password delivery:** the creating admin types the temp password; on success the user interface shows it back with copy "Share this with the new admin; they must change it on first login." (System-generated is a future option.)
- application programming interface ROUTES admin.admins = admin admins interface operation (list+create); the admin delete operation = admin admins id interface operation. Duplicate email → a field-level inline error (define code/status). Delete needs a confirm modal; hide/disable Delete on self and on super-admin rows.

**Before-prod gate (Security):** seeder provably inert in prod (allowlist + flag + a startup test); secure super-admin provisioning (random out-of-band password); re-enable multi-factor authentication on admin login. Tracked for the foundations phase — NOT in this dev slice.

## 7. Out of scope / guardrails
- The dev seed credentials must never ship to prod; secure super-admin provisioning is a
  foundations-phase item.
- Current plaintext email/phone storage follows baseline C13; preserve password hashing and audit every admin create/remove.
- Coverage gates apply (backend 97.5% line+branch; frontend 97.5%/88%).
