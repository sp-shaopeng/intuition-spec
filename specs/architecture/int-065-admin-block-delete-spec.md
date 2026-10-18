# INT-065 — Admin moderation: block a tutor/tutee (reason dropdown) + delete a job posting

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** The header already settles varchar status and deleted at naming; those older open rows are closed. Retain immediate session purge, public suppression, no automatic reopening and the existing cancel lifecycle for affected commitments. ACTIVE-match/pending-commission disposition must link to an explicit current decision; do not infer a money policy from a block flag. See [CURRENT-REQUIREMENTS](../CURRENT-REQUIREMENTS.md). Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Status:** team-reviewed + **Boss ✅ SHIP IT (2026-06-04)** — build may proceed on this spec with the 4 conditions below · **Date:** 2026-06-04

> **Boss build conditions (fold into the build — verified design, these are the must-dos):**
> 1. **SECURITY CRUX — confirm Jwt Authentication Filter honors the session purge per-request** so a block is
>    effective *immediately*, not at the 30-min token expiry. Sessions ARE server-side-tracked (Redis
>    sessions:by-user:`<id>`, reused from Admin User Service purge Admin Sessions), but the build MUST verify the
>    filter rejects a purged/blocked session each request; if it's stateless-signed session token, **add a blocked/session check to
>    the filter** (mandatory — else a blocked abuser stays active ≤30 min).
> 2. **Match/commission-in-flight on block** — define what happens to an ACTIVE match + pending commission when a
>    party is blocked (cancel/freeze — don't orphan a commission). Pin before build.
> 3. **Blocked user sees WHY** — surface the reason on the login rejection (and/or an appeal path); don't block silently.
> 4. **Admin user-list frontend surface** — confirm it exists; if not, building it is in scope.
> Design LOCKED: varchar block reason (not a named PG enum — INT-062 lesson); posting-delete via tutee posts deleted at
> (NOT a new Post Status value — avoids the ALTER TYPE hazard); block = soft reversible flag + ACCOUNT BLOCKED +
> session purge + audit; can't-block-admin guarded in-service; block cascades live content via INT-062 close Live Job.
**Owners:** software-engineer (lead) + dba + security-expert + frontend-expert + uiux-designer · **Tracker:** INT-065 (S2)

## Problem / founder ask
Admin needs **moderation powers**: (1) **block/suspend a user** (tutor or tutee) with a **reason chosen from a
dropdown** (e g. policy violation), and (2) **delete a job posting**. Blocking must keep the audit trail
(soft, reversible), suppress the user from the marketplace, and stop them logging in.

## Current state (verified in code)
- **No block/suspend concept.** User implementation reference has deleted at (soft-delete via the database query language restriction operation),
  locked until (temporary login lockout), failed login attempts, role, is super admin, password change required,
  mfa *. No "blocked/suspended" state, no block reason.
- **Auth Service login ()** already gates on ACCOUNT LOCKED (locked until) + rate-limit — the block check slots in here.
- The existing administrator-removal action (restricted to a super-administrator, logically deletes the account and purges its sessions) is the
  precedent pattern for an admin moderation action (reuse the session-purge).
- **Audit Log + Admin User Service** exist → block/unblock/delete must be audit-logged.
- **Admin post-delete partly overlaps INT-062**: submit operation for admin posts id close already sets a live posting →
  CLOSED + cascades its applications/invitations to CLOSED. "Delete" must be distinguished from "close" (below).

## Design

### A. Block / unblock a user (tutor or tutee)
- **State (soft, reversible):** add to users: blocked at (timestamptz, null = not blocked), block reason
  (**varchar**, NOT a PG named enum — per the INT-062 lesson, avoid ALTER TYPE crash-loops; the dropdown values
  are frontend-owned), block note (varchar, for OTHER/free-text), blocked by admin id (FK users). Null blocked at
  = active. (Distinct from locked until = temporary auto-lockout, and from deleted at = account deletion.)
- **Reason dropdown (frontend constants/, backend validates against the same set):** POLICY VIOLATION, SPAM,
  ABUSE HARASSMENT, FRAUD, FAKE PROFILE, OTHER (→ requires block note free text).
- **Endpoints (admin-only, in Admin User Controller):**
  - submit operation for admin users id block — requires a reason and accepts an optional note. Records when the block occurred, its reason and the administrator responsible; **purges the
    user's sessions** (reuse remove Admin's purge); audit-logs. Guards: only TUTOR/TUTEE targets; **cannot block an
    ADMIN/super-admin** (or restrict to super-admin if blocking admins is ever allowed — propose: not blockable here).
    409 if already blocked; idempotent.
  - submit operation for admin users id unblock — clears blocked at/reason; audit-logs. (Login restored; does NOT
    auto-reopen any content that was closed on block.)
- **Login effect:** Auth Service login → if blocked at differs from no value, reject with a distinct ACCOUNT BLOCKED (403),
  parallel to ACCOUNT LOCKED. The @database query language Restriction is for deleted, not blocked, so blocked users still load
  (needed to show admin the block state) — login/authz must check the flag explicitly.
- **Marketplace suppression (cascade — the real teeth):** on block, the user is hidden from public browse and their
  live content closed: a blocked **tutee**'s ACTIVE jobs → cascade-close (reuse **INT-062** close Live Job); a
  blocked **tutor**'s ACTIVE listings → hidden/closed + their open applications/invitations withdrawn/closed; their
  tutor card drops from /public/tutors. (Reversible login on unblock; closed content stays closed.)
- **Audit:** every block/unblock writes an Audit Log row (actor admin, target, reason, note).

### B. Delete a job posting (admin moderation removal) — distinct from INT-062 close
- **Distinction:** the INT-062 close action = the job is filled/withdrawn → CLOSED, still
  visible/listed (transparency). **Delete** = admin **removes** an inappropriate/spam posting → it should leave the
  marketplace. Proposal: admin delete = **soft-delete** (deleted at on tutee posts, mirroring the User pattern;
  or a REMOVED post status) + the **same cascade as INT-062** (apps/invitations → CLOSED) + a moderation
  **reason** (same dropdown) + audit. Soft-delete-only — never a hard row removal (preserves audit/analytics).
- **Endpoint:** submit operation for admin posts id delete (or DELETE), requires a reason and accepts an optional note; administrator-only, with an audit record.
  (The existing tutee delete operation for tutee posts id stays DRAFT/REJECTED-only; this is the admin moderation removal.)

### C. Migrations (dba)
- 0040 add users blocked at / block reason (varchar) / block note / blocked by admin id (standard
  add Column — NOT an ALTER TYPE; varchar reason avoids the enum crash-loop).
- 0041 (if admin-delete uses soft-delete) add tutee posts deleted at (+ @database query language Restriction or a REMOVED status).
  Decide in team-review (soft-delete column vs new Post Status value — note Post Status IS a named PG enum, so a new
  value needs ALTER TYPE ... ADD VALUE run In Transaction is no; a deleted at column avoids that).

### D. frontend
- **Admin users surface** (block/unblock): a block action with the reason dropdown + note field (OTHER); blocked
  users shown with a "Blocked — reason" badge; unblock action. (If no admin user-list page exists yet, that's part
  of the build — confirm during team-review.)
- **Admin delete-posting:** a "Delete posting" moderation action (reason dropdown) on the admin post/applicants
  surface, distinct from "Close job"; confirm dialog.
- Reason dropdowns sourced from a shared constants/ list (no magic strings); a11y; confirm dialogs for both
  destructive actions.

## Open decisions (for team-review)
- **Block reason storage:** varchar (proposed, per INT-062 lesson) vs a named PG enum. Proposed: varchar.
- **Block cascade scope:** does blocking a tutee close their ACTIVE jobs (+ a tutor's applications/listings)? Proposed:
  yes (block has teeth) — reuse INT-062 cascade. Confirm reversibility expectations (unblock ≠ auto-reopen).
- **Delete vs close:** distinct admin delete (soft-delete/REMOVED, hidden) vs reuse INT-062 close (CLOSED, listed).
  Proposed: distinct delete (the founder said "delete"). Decide the mechanism (deleted at column vs REMOVED status).
- **Who can block whom:** admins block TUTOR/TUTEE only; admins/super-admins not blockable here (use existing admin
  management). Confirm with security.
- **Is there an admin user-list frontend surface today?** If not, building one is in scope.
- Personal Data Protection Act: blocking/deleting must be audit-logged + reasoned (regulatory + moderation defensibility).

## Acceptance criteria
1. Admin blocks a tutor/tutee with a dropdown reason (+ note when OTHER) → user can't log in (ACCOUNT BLOCKED),
   is suppressed from public browse, their live content cascaded-closed; audit-logged; unblock reverses login.
2. Cannot block an admin/super-admin (403/guard); idempotent; non-admin caller 403.
3. Admin deletes a job posting (with reason) → soft-deleted (not hard-removed), removed from listings, its
   applications/invitations cascaded-closed; audit-logged; distinct from INT-062 "close".
4. Reason dropdowns are shared constants (no magic strings); destructive actions have confirm dialogs.
5. Migrate-then-boot (0040[/0041]); gates green (backend ≥97.5/97.5, frontend ≥97.5/88); soft-delete-only (no hard deletes).

## Rollout (after team-review → Boss)
backend (migrations → User fields + block/unblock + delete-posting endpoints + login guard + cascade + audit + tests) →
deploy+reseed dev → frontend (admin block/unblock + delete user interface + reason dropdowns + badges) → docs + postman → live smoke
(block a seed user → login 403 + suppressed; delete a posting → gone from listings, apps closed) → e2e → push.
Prod stays founder-gated.
