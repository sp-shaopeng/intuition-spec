# Durable Side-Effects & Transactional Outbox — Architecture Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** The proposal heading applies to unapproved alternatives, not the later folded INT-066 Option A audit decision. Business audit rows are atomic in-transaction; explicit006/007 outbox requirements are binding in their scope. An earlier accepted after Commit email-loss residual does not authorize weaker WhatsApp scheduling durability.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Status:** Proposal (doc-only). → team-review → Boss → build.
**Authors:** software-engineer (Jordan, lead) · solution-architect (the solution architect) · dba (Morgan)
**Date:** 2026-06-05
**Scope:** the *durable-side-effects family* — every place the system performs a side-effect
(email, audit-row write, in-app notification fan-out) coupled to a business transaction, where
that side-effect can be **lost or emitted-without-its-cause** because it is not bound to the
transaction's commit through a durable mechanism.
**Related issues:** INT-066 (audit after Commit anti-pattern), INT-025 (admin notify fan-out),
INT-050 (system-wide idempotency review).

---

## 0. The invariant (the rule this spec exists to enforce)

> **Any side-effect that is not (a) committed in the same database transaction as its cause, or
> (b) emitted from an after Commit hook AND itself durable & retryable, is PROVISIONAL —
> it may be lost, or worse, performed for a cause that then rolls back.**

Two distinct failure shapes fall out of violating it, and they are *opposites*:

| Shape | Definition | Example in this codebase | Consequence |
|---|---|---|---|
| **Premature** | side-effect fires *before/independent of* commit; cause then rolls back | S1 — approval email enqueued mid-transaction (Admin Post Service) | user gets "approved" email for a post that is **not** approved |
| **Lost** | side-effect is supposed to fire *after* commit but is dropped | INT-066 — audit save in after Commit runs in a closed context (Auth Service, Admin User Service); INT-025 — fan-out dropped on a crash between commit and listener | audit/notification silently never happens |

A correct design must close **both** shapes for each side-effect. The rest of this spec
classifies each instance, then picks the mechanism per class.

---

## 1. Problem inventory (grounded in file:line)

### S1 — Approval/rejection email dual-write (MUST-FIX before prod)

**Where:** Admin Post Service implementation reference
- The recorded approval operation ran in a database transaction and requested approval emails for the affected tutor or tutee.
- The recorded rejection operation ran in a database transaction and requested rejection emails for the affected tutor or tutee.
- Both helpers passed the message to the email queue.
- Email Queue Service enqueue is annotated @Async (Email Queue Service implementation reference).

**Current-state analysis — this is a classic *dual-write* with NO commit binding:**
1. @Async means enqueue runs **on a different thread, immediately**, the moment the helper
   is called — i e. **while the approve/reject transaction is still open and uncommitted.**
2. There is **no** after Commit / Transaction Synchronization registration on this path
   (contrast Tutee Post Service run After Commit 404 and Auth Service after Commit 727,
   which exist but are not used here).
3. The async thread pushes the message onto the email-notifications Redis stream
   (Email Queue Service). The Email Consumer Service poller (81) picks it up within ~2s
   and dispatches to Resend. **None of this is rolled back if the database transaction aborts.**

**The defect (Premature shape):** if approve Post commits the status flip but the transaction
*then* rolls back for any reason after 144/160 — an exception in the response mapper, a
commit-time constraint failure, an optimistic-lock conflict, a connection drop at commit — the
user has **already been emailed "your post is approved"** for a post whose status is still
PENDING (or was never persisted). The email is irreversible. This is exactly the inverse of
the INT-066 lost-write bug: here the side-effect is *too eager*, not too late.

> Note the try/catch in queue Approval Email (424) only swallows enqueue *failures* — it
> does nothing about the ordering bug. A successful enqueue on a transaction that later aborts
> is the failure mode, and it is invisible to that catch.

**Severity:** S1 / must-fix-before-prod. Emailing a user a false state-change is a trust and
(for rejection) a support-load problem, and it is non-recoverable once sent.

### INT-066 — Audit rows saved in after Commit are silently lost (Lost shape)

**Where:**
- Auth Service implementation reference — audit saves wrapped in the after commit operation: 123→ (USER REGISTERED,
  tutee), 182→/ (USER REGISTERED + COMMISSION ACKNOWLEDGED, tutor), plus the same
  pattern at 291, 399, 422, 564, 592. Helper: 727.
- Admin User Service implementation reference — 106→ (ADMIN CREATED), 173→ (ADMIN REMOVED).
  Helper: 198.

**Current-state:** the after commit operation registers a Transaction Synchronization whose
the after commit operation runs the action **after** the outer a database transaction has committed and the
persistence context / Hibernate session is closed. A database save attempted there has
**no active transaction and no open session to flush into** — the append records is silently dropped,
with no error logged.

**Proof this is real, not theoretical:** Admin Moderation Service implementation reference documents it in
a code comment — *"Audit IN-TRANSACTION (atomic with the block) — an after Commit save lands in a
closed persistence context and is silently lost (proven by INT-065 live smoke: zero audit log
rows)."* INT-065's three moderation events were fixed by moving the save in-transaction (see
Admin Moderation Service, 181, 273); Admin Post Service write Audit Log (389-412) is
the *correct* reference pattern — it saves the audit row **inside** the business transaction
(412) and its Javadoc (386-388) explicitly explains why fail-closed-in-tx is right for a
compliance row. INT-066 is the **latent twin** in Auth Service + Admin User Service that was
never migrated.

**Decision rule (the durability dichotomy):**
- **An audit row is part of the business fact.** It must be **atomic with the action** — if the
  block/registration/admin-create commits, the audit row commits; if it rolls back, so does the
  audit. That is a database transaction participation (in-tx), NOT after Commit.
- after Commit is **only** correct for side-effects that (i) must NOT happen if the tx rolls
  back, AND (ii) are themselves non-transactional/external and independently durable+retryable
  (email enqueue to a durable stream; notification fan-out backed by retry). An audit database write
  is neither — it belongs in the transaction.

### INT-025 — Notification fan-out durability gap (Lost shape, narrower)

**Where:** The administrator-notification fan-out starts a separate database transaction,
invoked from Tutee Post Service via the run after commit operation (103→).

**Current-state:** This is *better* than INT-066 — the fan-out write itself runs in its own
committed transaction (REQUIRES NEW at 71), and per-recipient failures are isolated
(try/catch 86). The remaining gap is **the trigger**: the fan-out is launched from an
after Commit hook (103). If the JVM crashes (OOM, kill, deploy/pod-restart) in the window
**between the post transaction committing and the after Commit callback running**, the
notification is **never emitted** — the post exists, the admins were never told. This is the
gap addressed by the outbox: delivery intent survives a crash between the business commit and the later notification work.

**Severity:** S2. The crash window is small and admins also have the approval *queue* as a
backstop (the post still appears in get Pending Queue), so a missed notification is degraded-not-
broken. But it is the canonical case for the outbox and should be designed with it.

### INT-050 — Idempotency review (cross-cutting, S2)

**Current-state:** there is **no system-wide idempotency contract.** Protection is ad-hoc and
per-endpoint:
- **State-guard pattern (good, keep):** Admin Post Service approve Post 409 ALREADY REVIEWED on
  non-PENDING (137, 153); Admin Commission Service pay Commission 409 ALREADY PAID
  (55); Admin Moderation Service 409 ALREADY BLOCKED/NOT BLOCKED/ALREADY DELETED.
- **Optimistic-lock pattern (good):** atomic assign in Admin Match Service uses @Version →
  409 JOB ALREADY ASSIGNED (181, 217); free-market dedupe relies on the 0033f
  partial-unique index.
- **No protection against true double-submit / network-retry** of a POST that has no natural
  state guard (e g. a client retries pay Commission after a timeout while the first is still
  in-flight; two concurrent select-tutor requests). State guards close the *replay-after-
  completion* case but not the *concurrent in-flight duplicate* case.

This spec makes the **idempotency-key decision** for the highest-value money/assignment POSTs
(below, §4), but the full endpoint-by-endpoint audit remains INT-050's own deliverable.

---

## 2. Design options

Two competing mechanisms. They are not mutually exclusive — the recommendation (§3) uses each
where it fits.

### Option A — after Commit for external effects + REQUIRES NEW for must-survive writes

**Premise:** keep side-effects in-process; fix the *binding*, not the infrastructure.

- **External, must-NOT-fire-on-rollback effects (email):** move every enqueue call out of the
  transaction body and into a the after commit operation hook (reuse the
  existing Admin Post Service would gain a run After Commit like Tutee Post Service). The
  email is enqueued **only if the transaction actually committed**, closing the Premature shape.
- **Must-survive-a-business-rollback writes:** run them in a **separate** transaction via a
  separately invoked component that starts a new database transaction — exactly the proven
  Login Attempt Tracker pattern (Login Attempt Tracker implementation reference, 54): a dedicated bean so the
  proxy fires, recording the failed-login/lock even when Auth Service login throws and rolls
  back. (Audit rows do NOT use this — see below.)
- **Audit rows:** simply move **in-transaction** (atomic with the action), mirroring
  Admin Post Service write Audit Log and the INT-065 fix Admin Moderation Service. This is
  the INT-066 fix. Delete the now-dead after Commit helpers if nothing else uses them.

**Pros:**
- Zero new infrastructure, zero new tables, zero new schema migration.
- The patterns already exist and are battle-tested in this repo (Login Attempt Tracker,
  Admin Post Service write Audit Log, Tutee Post Service run After Commit).
- Smallest possible change for the S1 email fix — directly defensible as must-fix-before-prod.

**Cons (the architect's objection):**
- after Commit does **not** close the **Lost** shape for the *trigger*. A crash in the window
  between commit and the hook still drops the email enqueue / the notification fan-out
  (INT-025). It converts Premature→fixed but leaves a (smaller) Lost window.
- No replay/audit trail of *intended* side-effects — if Redis is down at the moment of
  after Commit, the email is silently lost (the enqueue catch at Email Queue Service
  swallows it).

### Option B — Transactional outbox table + poller

**Premise:** make the *intent to perform a side-effect* a durable row written in the **same database
transaction** as the business write; a separate poller publishes it.

- A new outbox table. In the approve/reject/register/admin/notify transaction, append records one
  outbox row (type is EMAIL / NOTIFICATION FANOUT, payload JSON) **alongside** the business
  mutation — one atomic commit. If the transaction rolls back, the outbox row rolls back too:
  **the side-effect intent never exists for a cause that didn't commit** (closes Premature).
- A the scheduled task poller reads PENDING outbox rows, performs the effect (push to the
  email-notifications stream / emit notifications), and marks the row SENT. A crash before
  marking SENT leaves the row PENDING → retried next cycle (closes Lost). This is **modelled
  directly on the existing Email Consumer Service** Redis-Streams consumer-group
  (Email Consumer Service implementation reference): poll-on-fixed Delay (81), dispatch-then-ack (143),
  reclaim-on-idle (110) — the outbox poller is the same shape with Postgres as the
  durable log instead of the Redis PEL.

**Pros:**
- Closes **both** Premature and Lost for every side-effect, including the INT-025 crash window.
- At-least-once with a full audit trail of intent; retriable; observable (a row you can query).
- Aligns with the architect's stated standard (SHARED-LESSONS: outbox for every "write data AND
  emit event"); reusable for all future fan-outs (match-state transitions, commission events).

**Cons:**
- New schema (table + indexes + migration 0042), a new poller bean, new tests — more surface
  to build and gate (≥97.5% line+branch).
- Two hops for email now (database outbox → poller → Redis stream → Email Consumer → Resend). The
  Redis stream already provides at-least-once + reclaim; the outbox adds a *second* durable
  buffer in front of it. For email specifically this is arguably belt-and-braces.
- A poller adds steady database read load; needs SKIP LOCKED + a partial index on status is PENDING
  to avoid scanning the (soft-delete-style growing) table.

---

## 3. Recommendation

**Tiered: Option A now (close the S1 + INT-066 bugs immediately), Option B as the standard for
event fan-out (INT-025 and all future "write + emit").** Rationale: match the mechanism to the
side-effect's nature, don't over-build the must-fix.

1. **S1 email (must-fix-before-prod) → Option A.** Move the queue Approval Email/
   queue Rejection Email calls in Admin Post Service (144,160,187,203) into a
   the run after commit operation hook (clone Tutee Post Service). This closes the Premature bug —
   no email is enqueued unless the approve/reject transaction commits. The downstream
   Email Queue Service→Redis-stream→Email Consumer Service path **already provides at-least-once
   + reclaim** (Email Consumer Service PEL/reclaim 110), so the only remaining Lost window is
   the tiny commit→after Commit gap, and email is the lowest-stakes effect to lose (a missed
   approval email is recoverable; a *false* one is not). **Do not block the S1 fix on building
   the outbox.**

2. **INT-066 audit rows → Option A (in-transaction).** Move the Auth Service (125,184,
   190, 293, 401, 424, 566, 594) and Admin User Service (108, 175)
   audit saves **inside** their a database transaction bodies (atomic with the action), exactly as the
   INT-065 fix. Audit rows are part of the business fact; they are never after Commit. Remove
   the dead after Commit helpers (Auth Service, Admin User Service) if no non-audit
   caller remains; the analytics-publication calls bundled in those hooks
   (Auth Service,197) are non-transactional external effects and **stay** after Commit
   — split the runnable so the audit save moves in-tx and the analytics publish stays post-
   commit.

3. **INT-025 notification fan-out → Option B (outbox).** This is the one with a genuine
   crash-window Lost gap and the canonical "write business data AND emit an event" shape. Build
   the outbox table + poller and route POST SUBMITTED admin fan-out through it: the post
   transaction INSERTs an outbox row atomically; the poller calls
   Notification Service emit To All Admins. Keep emit To All Admins REQUIRES NEW (71) +
   per-recipient isolation (86) as the consumer-side idempotency. Establish the outbox here
   as the **standard for all future fan-outs**, recorded as an ADR.

**Why not Option B for everything:** putting email behind the outbox too adds a second durable
buffer in front of a stream that is already at-least-once — cost without proportional benefit,
and it delays the S1 fix. Put the outbox where the durability gap is real (event fan-out), use
the cheaper after Commit/in-tx fixes where they fully close the bug.

---

## 4. Idempotency-key decision (INT-050 hot-spots)

**Decision: a hybrid, NOT a blanket Idempotency-Key header on every POST.**

- **State-guarded mutations (approve/reject, pay-commission, block/unblock, assign):** the
  existing **state-guard + optimistic-lock** pattern is the primary defence and is sufficient
  for *replay-after-completion* — keep it (Admin Post Service, Admin Commission Service,
  Admin Match Service). Do not add a header where a natural state guard already makes the
  second call a clean 409.
- **Money & assignment POSTs that can be double-submitted in-flight** — pay Commission,
  parent select-tutor / admin assign — adopt an **explicit client Idempotency-Key
  header**, stored with the first result; a replay returns the prior result instead of acting
  again. This protects the window the state guard misses (two concurrent requests before either
  commits) without relying on optimistic locks alone. The key store is naturally an outbox-
  sibling table or a Redis key with expiry period ≥ client-retry budget.
- **Consumers are at-least-once → idempotent by construction:** the outbox poller and
  Email Consumer Service may redeliver; consumers must dedupe. Email dedupe via a natural key
  (template Id+to+cause-id); notification fan-out is naturally idempotent if the poller marks the
  outbox row SENT atomically (so a redelivery is a no-op). This mirrors the existing optimistic-
  lock-409 defensive posture extended to every fan-out path.

The full per-endpoint audit (apply/withdraw, activate/cancel, review-submit, password-reset)
remains INT-050's deliverable; this spec fixes the money/assignment hot-spots and sets the rule.

---

## 5. Schema & migration notes (Option B — outbox; migration 0042)

Current migration head is 0041-tutee-post-soft-delete implementation reference; next free Liquibase number is
0042 in the original plan; verify the migration ledger before assigning or reusing a number.

Proposed outbox table (Liquibase XML changeset only — never raw database structure definitions):

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| Event type | Text, up to 40 characters | NOTIFICATION FANOUT (extensible). **Not** a PG named enum — plain VARCHAR, per the INT-062/INT-065 rule that ALTER TYPE ADD VALUE crash-loops migrate-then-boot. |
| payload | jsonb | the side-effect parameters (recipient set, title/body/link, cause id) |
| Status | Text, up to 20 characters | PENDING / SENT / FAILED; soft-state, never hard-deleted |
| attempts | int default 0 | bounded-retry counter |
| available at | timestamptz | for backoff |
| created at / updated at | timestamptz | mandatory (dba non-negotiable) |
| sent at | timestamptz null | when terminal |

**Index and worker selection:** Index pending records by the time they next become eligible. The historical proposal creates that index without blocking ordinary writes, in a separate migration step that is not wrapped in a transaction. Set explicit time limits for waiting on locks and completing the migration.

The worker selects pending records whose next-attempt time has arrived, in due-time order and in a bounded batch. It locks its selected records and skips those already locked by another worker. This lets multiple instances share the work without selecting the same row concurrently.

Soft-state only: terminal rows flip status (SENT/FAILED), never row-deleted (preserves the
audit/retry trail; align with the soft-delete-only directive).

**Migration is additive/expand-only:** new table + new partial index, no change to existing
tables → no expand-contract dance, no backfill, reversible by drop. The Option A fixes (S1
email, INT-066 audit) require **no migration at all** — code-only.

---

## 6. Acceptance criteria

**S1 — email (Option A, must-fix-before-prod):**
- [ ] Admin Post Service approve/reject enqueues the email from an after Commit hook, never in
      the transaction body. Verified by a test that **forces a post-status-flip-then-rollback**
      (or asserts via a Transaction Synchronization spy that enqueue is registered, not called,
      until commit) and asserts **no message hits the email-notifications stream** when the
      transaction rolls back.
- [ ] On a committing approve/reject, exactly one email is enqueued (unchanged happy path).

**INT-066 — audit (Option A, in-transaction):**
- [ ] ADMIN CREATED / ADMIN REMOVED rows persist after Admin User Service create/remove
      (IT queries audit log and finds the row). Currently absent (INT-066 repro).
- [ ] USER REGISTERED / COMMISSION ACKNOWLEDGED and the other Auth Service audit rows
      persist; a forced business rollback rolls the audit row back too (atomicity proven).
- [ ] Analytics publication remains after commit and does not occur after a rollback.
- [ ] Dead after Commit helpers removed if unused (no orphaned private method).

**INT-025 — notification fan-out (Option B, outbox):**
- [ ] On post submission, an outbox row is INSERTed in the **same transaction** as the post
      (rolls back with it — assert no outbox row on a forced post rollback).
- [ ] The poller emits emit To All Admins and marks the row SENT; a simulated crash
      (poll, kill before mark-SENT) re-emits on the next cycle and the consumer dedupes (no
      duplicate admin notifications). 
- [ ] Poller uses SKIP LOCKED + bounded batch; concurrent-instance test shows no double-send.

**INT-050 — idempotency:**
- [ ] pay Commission + select-tutor/assign accept an Idempotency-Key; a replay with the
      same key returns the prior result and does **not** double-act (test: two identical POSTs →
      one effect, identical response).
- [ ] State-guard 409s unchanged for replay-after-completion.

**Required checks:** backend verification at 97.5% line and branch coverage over the measured scope, the database services needed by integration tests, formatting checks, and shared event and status definitions.

---

## 7. Test notes

- **Rollback proof (S1 + INT-066 + INT-025):** the load-bearing tests are *not* happy-path —
  they must force the transaction to roll back after the side-effect point and assert the
  side-effect did/didn't happen. Use a Transaction Synchronization spy or a Testcontainers IT
  that throws at commit (e g. a deferred constraint) to exercise the abort path.
- **Live-stack regression (per project convention):** Abstract Api IT hits localhost —
  any IT that asserts an email reached the stream or an audit row landed needs the stack
  rebuilt/redeployed (+reseed) before running verification of the fix
  (SHARED-LESSONS: regression-its-live-stack).
- **@Data Jpa Test for the outbox repo** will read Tests run: 0 on a local run by Docker Support
  policy — that is expected, not a hollow test; it executes in continuous integration (SHARED-LESSONS 2026-06-04).
- **Idempotency-key tests** must assert the *concurrent in-flight* case, not just sequential
  replay — fire two requests with the same key before the first commits.
- **PG enum trap:** the outbox status/event type are VARCHAR by design; do **not** introduce
  a PG named enum (the INT-034/INT-062 crash-loop class).

---

## 8. ADRs to record on build

1. **Outbox vs after Commit-listener for event fan-out** — Context (crash window drops events),
   Decision (outbox is the standard for "write + emit"), Consequences (new table + poller; email
   stays on the cheaper after Commit→existing-stream path).
2. **Idempotency: hybrid (state-guard/optimistic-lock + Idempotency-Key for money/assignment),
   not a blanket header** — so the decision survives and isn't relitigated per endpoint.

→ team-review → Boss → build.

## 2026-09-05 current contract and verification

The folded audit decision requires the audit fact in the same transaction as its business change. For reminder intent, follow [007](../007-generic-scheduler-db-scheduler/design.md): durable recipient/generation key and outbox row in one commit; a scheduler success/row deletion or after Commit callback alone is not deduplication. Current baseline C12 requires disabled/unavailable email to remain durable deferred work until success or explicit permanent-failure/failed-message queue disposition; historical after Commit loss tolerance is not an active delivery-success contract.
