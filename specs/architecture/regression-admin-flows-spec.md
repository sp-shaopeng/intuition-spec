# M-Reg-Admin — Admin Flow Regression Coverage (SPEC)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** The final regression decisions supersede earlier shared-credential/self-healing examples: dedicated @regression local admin/cohort, idempotent per-class bootstrap; never mutate the human [REDACTED EMAIL] login. Original exploratory branches remain historical.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

Status: **SPEC — team-reviewed + Boss-cleared, conditions binding (§8 LOCKED)** · Author: Regression
Eng (Robin) + Architect · Date: 2026-05-30
See §8 for the LOCKED decisions (panel + Boss); they OVERRIDE §3–§7 where they differ.
Boss verdict: SEND BACK → conditions folded into §8 → **v2 RE-ADJUDICATED ✅ CLEARED TO BUILD**
(2026-05-30, all 6 conditions verified MET). Build sequence: (1) the profile operation seeders —
reg super-admin + regression seed-band + reset hook; (2) the admin bootstrap operation base +
the change password operation; (3) Admin Bootstrap IT (create-admin journey, @smoke) — prove the
operator first; (4) Admin Authz Matrix IT + security trio (stale-signed session token, elevation, gate negatives);
(5) flow ITs M3→M4→M5→M16→M12→M6→Admin Application; (6) green ×2 consecutive → Boss exit re-audit.
Scope: extend the backend regression lab to cover ALL admin-gated user flows, driven by an admin account
created at runtime by the seeded super-admin. **Spec only — no code in this document.**

---

## 1. Problem / founder directive

The regression lab covers all non-admin flows (56/56 green) but **zero admin flows**, because the
seeded super-admin (REDACTED email) ships with password change required is yes and we hadn't
wired a login path for it. Founder directive (2026-05-30):

1. Regression cases must be **user-flow specific** — implement ALL remaining flows.
2. The super-admin → **create admin account** journey must itself be tested.
3. That created admin account must **continue operating** through the rest of the admin regression
   cases (it's the operator for M3/M4/M5/M12/M16/admin-interviews).
4. Ensure **all cases covered**. Review.

## 2. Grounded facts (verified live against the running stack — NOT assumptions)

- **Super-admin login works today:** submit operation for auth login → **200**,
  body an administrator account that is a super administrator and must change its password.
- **The password-change gate is per-MUTATION, service-layer** (Admin Guard Service
  .require Password Change Cleared→ 403PASSWORD_CHANGE_REQUIRED), invoked at the top of every
  /admin/** mutation **except** change-password. It re-reads the live database flag (a stale signed session token can't
  bypass it).
- **Reads are NOT gated by the flag:** with password Change Required is yes, read operation for admin admins
  returned **200**. So a freshly-seeded super-admin can READ admin data but cannot WRITE until it
  clears the gate.
- **Change-password:** submit operation for auth change-password →
  a successful result. new Password must match ^ ? .* A-Z ? .*\d ? .* REDACTED password .+, 8–128 chars.
- **Create admin:** submit operation for admin admins → **201**
  the account identifier, email and temporary password. Created admin: role is ADMIN, is Super Admin is no,
  password change required is yes. temp Password must satisfy the same complexity regex.
- **Delete admin:** delete operation for admin admins id → **204**, **SUPER-ADMIN only** (service-layer
  flag check, not a role); cannot delete a super-admin or self.
- **Admin controllers in scope:**
  - Admin Post Controller (M3): read operation for admin posts pending, read operation for admin posts id,
    submit operation for admin posts id approve, submit operation for admin posts id reject
  - Admin Match Controller (M4): GET/POST /admin/matches, submit operation for id activate, submit operation for id cancel
  - Admin Commission Controller (M5): submit operation for admin commissions id pay
  - Admin User Controller (M16): POST/GET /admin/admins, delete operation for id
  - Analytics Controller (M12): read operation for admin analytics
  - Interview Controller admin ops (M6): read operation for admin interviews, the selected record, submit operation for id schedule,
    update operation for id meeting-link, submit operation for id complete,cancel,no-show
  - Admin Application Controller: GET `<applicants path>`

## 3. The operator strategy (founder's core ask)

A single shared **operator-admin** created at the start of the admin suite, reused by all admin
cases — exactly the real production bootstrap path:


Implementation example omitted. This public edition retains the written requirements in this section.


**Design decisions (for review):**
- **D1 — Operator lifecycle:** the ops-admin is created ONCE per suite run and REUSED across admin
  cases (a JUnit fixture / ordered bootstrap test), not recreated per test. Rationale: matches the
  real "one operator runs the console" model, keeps the suite fast, and the create-admin journey is
  tested exactly once as its own case.
- **D2 — Idempotency / re-runnability:** the suite runs repeatedly against the shared dev stack.
  - The super-admin's password must be deterministic across runs. **Problem:** once run #1 changes
    admin→REDACTED password, run #2's step 1 (login admin/admin) fails. **Options (pick in review):**
    - **D2a (recommended): make the bootstrap self-healing** — try the login operation; on 401, try
      the login operation. Whichever works, ensure the gate is cleared, then proceed. No
      database reset needed; survives re-runs.
    - D2b: reset the super-admin via a regression-only reset (re-run Super Admin Seeder / database query language) before
      each suite — heavier, touches data.
  - The ops-admin email must be stable; creating it twice → 409 EMAIL ALREADY REGISTERED. The
    bootstrap tolerates "already exists" and just logs in (self-healing, like D2a).
- **D3 — Cleanup:** ops-admin is a regression artifact on the dev stack. Leave it (idempotent, reused
  next run) rather than delete — deleting it would force recreation churn and the super-admin
  password is already mutated. Document that REDACTED email is a known regression user.
- **D4 — Super-admin password mutation is a SIDE EFFECT on shared dev data.** After the first run,
  REDACTED email's password is [REDACTED CREDENTIAL], not admin. This must be DOCUMENTED (devs
  logging in manually need to know) and the dev-login docs/README updated. Acceptable for dev;
  flagged for review.

## 4. Flows to cover (user-flow specific, one IT class per journey)

Each is a complete journey driven by the gate-cleared ops-admin (except where super-admin is
required). Assertions go beyond status codes (body/state), per the established lab standard.

| IT class | Module | Journey |
|----------|--------|---------|
| Admin Bootstrap IT | M16 | The §3 bootstrap: super-admin login → gate-blocked mutation (403) → change-password → create ops-admin (201) → ops-admin change-password → operational. **This is the create-admin journey.** |
| Admin Post Approval IT | M3 | ops-admin lists pending posts → views one → **approve** a pending post (becomes ACTIVE) and **reject** another (with reason); verify status transitions + that a non-pending post can't be approved. |
| Admin Matching IT | M4 | ops-admin creates a match (tutor+tutee post) → activate → cancel; verify lifecycle + that contact reveal appears on ACTIVE (M8 cross-check). |
| Admin Commission IT | M5 | ops-admin marks a commission paid → verify status; invalid/again → proper error. |
| Admin User Mgmt IT | M16 | ops-admin lists admins (200); ops-admin (non-super) **DELETE → 403** (super-only); super-admin DELETE a throwaway admin → 204; cannot delete super-admin/self. |
| Admin Analytics IT | M12 | ops-admin reads /admin/analytics → 200 with the metric shape; close the M12 "verification pending" gap. |
| Admin Interview Ops IT | M6 | ops-admin lists interviews → schedule → set meeting-link → complete / no-show / cancel; verify transitions. |
| Admin Authz Matrix IT | cross | every /admin/** endpoint: anonymous → 401, TUTEE/TUTOR → 403/401 (the cross-role boundary, using assert Denied Cross Role). |

**Negative/edge cases to include (founder: "all cases covered"):**
- Gate enforced: mutation before change-password → 403 PASSWORD CHANGE REQUIRED (both super + ops).
- Create-admin duplicate email → 409; weak temp password → 400 validation.
- Delete by non-super-admin → 403; delete super-admin → 403; delete self → 403.
- Approve a non-PENDING post → conflict; pay an already-paid commission → conflict.
- Wrong-role on every admin endpoint → denied.

## 5. Constraints & infra (reuse what M-Reg already established)
- BLACK-BOX against the running dev stack (no fresh database — founder decision stands).
- Api Client already manages auth+cross-site request forgery protection cookies explicitly; add a the change password operation helper and an
  the admin bootstrap operation fixture. No new auth machinery.
- Dev login rate-limit already raised (RATELIMIT LOGIN LIMIT is 100000) — the admin suite adds ~10
  more logins; within budget.
- Tagged @smoke (the bootstrap + read paths) vs @full (full mutation lifecycles).
- Each mutation test SELF-CLEANS or operates on disposable seed rows so re-runs stay green.

## 6. Acceptance criteria
- The super-admin→create-admin→operate journey passes end-to-end (the founder's named flow).
- All 8 admin IT classes green; every /admin/** endpoint exercised at least once with a real
  gate-cleared admin.
- Suite is **re-runnable** (self-healing bootstrap, no manual database reset) — proven by 2 consecutive
  green runs.
- The shared-data side effect (super-admin password mutation, persistent ops-admin) is documented.
- Full backend regression suite (non-admin 56 + admin) green, BUILD SUCCESS.

## 8. LOCKED decisions (panel + Boss, 2026-05-30) — override §3–§7

Boss SEND-BACK → these are binding for the build. Re-adjudication required before merge.

1. **Q2 — SEPARATE regression super-admin. FINAL.** Add a the profile operation seeder for
   REDACTED email (role is ADMIN, is Super Admin is yes, a known reg password, NOT
   password change required — or with it, cleared once in bootstrap). The suite **NEVER touches
   REDACTED email** — the human dev login stays REDACTED email /admin, untouched. This KILLS
   D2a (self-healing two-password hack) and D4 (shared-credential mutation side effect). Both are
   struck from the spec.

2. **Q3 — dedicated REGRESSION SEED-BAND, REQUIRED (not just disposable rows).** A the profile operation
   regression cohort the Dev Data Seeder/demo data NEVER populates: emails *@regression local,
   and a reserved marker for posts/matches/commissions (a regression tag column OR a reserved ID
   band — DBA to pick). All admin MUTATIONS (approve/reject/match/activate/cancel/pay/delete) operate
   ONLY inside this band. Demo data stays presentable; re-runs stay green; no assert-and-restore
   (non-atomic, untrustworthy). Provide a reset/re-seed hook for the band, pre-suite.

3. **D1 — idempotent the admin bootstrap operation in a shared base @Before All, memoized per JVM.** JUnit
   @Order is within-class only and Failsafe gives no cross-class ordering → every admin IT extends
   a base whose @Before All ensures the gate-cleared ops-admin exists (idempotent). Admin Bootstrap IT
   ASSERTS the journey; it does NOT provide the fixture for the others. Bootstrap **fails LOUD** if it
   cannot establish the operator — never silent-skip the admin suite.

4. **Security tests — BLOCKING (Casey, corrected by Boss):**
   - **Stale-signed session token can't bypass the gate:** capture the pre-change-password signed session token, change password,
     replay the OLD token on a mutation → still 403 PASSWORD CHANGE REQUIRED (gate re-reads live database).
   - **Privilege-elevation denial:** a non-super (ops) admin cannot mint or elevate a super-admin —
     create-admin always yields is Super Admin is no; assert no path sets it true.
   - **Profile guard = the profile operation, NOT "ENV is dev.** There is NO dev profile in this
     codebase (verified). The reg seeder + any reg-only endpoint are the profile operation only; the
     bootstrap may additionally assert the base URL is localhost before mutating.
   - **delete-nonexistent admin:** super-admin → **404**; non-super → **403** (existence not leaked —
     the super-check fires before the target lookup; verified in source).

5. **New seeder the profile operation only** (M-Reg §12 consistency — never leaks to prod profile).
   Hardcoded reg passwords are synthetic + dev-only + private-repo; acceptable.

6. **Coverage completeness (Reese, blocking):** per-VERB gate negatives (not one generic check);
   double-submit/idempotency conflicts (double-approve, double-activate, double-pay → conflict);
   admin LIST pagination + empty-state assertions; an IT class for the orphaned
   **Admin Application Controller**; prove super-only-delete is FLAG-driven (test a 2nd non-super admin,
   not just identity). Admin Bootstrap IT is @smoke and fail-fast.

## 9. Open questions (remaining, non-blocking)
- **Q1 (D2):** self-healing bootstrap (D2a) vs per-run super-admin reset (D2b)?
- **Q2 (D4):** is mutating the shared dev super-admin password acceptable, or should the regression
  suite use a SEPARATE seeded super-admin (e g. REDACTED email) so the human
  REDACTED email /admin dev login stays untouched? (Recommended: separate reg super-admin — keeps
  dev login stable AND makes the suite hermetic. Needs a seeder addition.)
- **Q3:** mutation tests on shared data (approve/match/pay) — operate on dedicated disposable seed
  rows, or assert-and-restore? How to keep the seeded demo data presentable after a run.
- **Q4:** should admin flows also get frontend Playwright coverage, or backend-only for now?
- **Q5:** the cross-role 401-vs-403 quirk (tutor→tutee endpoint returns 401) — fix the app to 403, or
  keep the assert denied cross role operation in the lab and log a backlog item?
