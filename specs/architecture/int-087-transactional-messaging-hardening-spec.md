# INT-087 — Transactional-Messaging & Idempotency Hardening (implementation spec)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** BUILT + team-reviewed (Boss SHIP) + code-reviewed (the review caught and fixed a dedup-ordering bug — the
consume-side seen sentinel was set BEFORE the send, suppressing the retry of a failed send; now set-after-success).
All 5 phases (F1–F5) implemented; F6 deferred to INT-082 PE-3. Gate **green: 1713 tests, 98.84% line / 97.71% branch**.
Awaiting deploy (dev). Phased; built phase-by-phase, each with its own gate + team review.
**Created:** 2026-06-09. **Owners:** software-engineer (lead) + solution-architect + dba + bff-specialist + qa-engineer.
**Source:** the INT-087 audit (issues/README md) against the playbook
[transactional-messaging-and-idempotency md](transactional-messaging-and-idempotency.md).
**Severity:** the gaps are S2 (F1/F2) → S4 (F5); none is a live outage (WhatsApp is OFF; email's gap is a
Redis-down/crash window). This is reliability hardening of an already-mature layer, not a firefight.

---

## 0. Scope & non-goals

**In scope (this spec):** F1 emails-on-the-durable-outbox, F2 stream failed-message queue + bounded retry, F3 consume-side dedup,
F4 Commission optimistic lock, F5 idempotency request-fingerprint.
**Deferred (NOT specced here):** F6 — WhatsApp Meta client idempotency key — folded into INT-082 **PE-3** (the
feature is OFF; no value until enablement).
**Non-goals:** no change to the WhatsApp narrowing (INT-084/085) or the OFF toggle; no rewrite of the
outbox/idempotency-key/notification-dedup layers (they are textbook — see the audit's "what is textbook" list).

**Phasing (independent, shippable in order):** Phase 1 + Phase 2 (+ Phase 3, which falls out of touching the same
files) form the headline "make email as reliable as notifications" bundle and should land first. Phase 4 is an
independent one-column money-path hardening. Phase 5 is polish.

**Schema:** only Phase 4 adds a migration (0048, additive column with default — safe online). Phases 1–3 + 5 are
code-only.

---

## 1. Phase 1 (S2) — Emails ride the durable outbox (F1) + synchronous stream handoff

### 1.1 Problem
forgot password, post approved, post rejected are enqueued via after Commit → @Async Email Queue Service enqueue
→ XADD Auth Service implementation reference Admin Post Service implementation reference,487). The direction is correct (won't fire on rollback),
but the enqueue is **not durable**: the business tx has committed, and if Redis is down or the pod crashes in the
commit→async-XADD gap, the email is **lost with no recovery row**. Password reset — our highest-stakes transactional
message — has the *weakest* delivery guarantee, while in-app notifications (lower stakes) already ride the durable
outbox (INT-025). Principle D / Principle A of the playbook.

### 1.2 Design — route email origination through the outbox (mirror the WhatsApp path)
1. **Outbox Event Type EMAIL** — add to Outbox Event Type implementation reference (after WHATSAPP NOTIFICATION). The dispatch
   dispatch selection covers every event type, so this is a compile error until the handler exists — no silent
   gap.
2. **Email Payload record** (new, in infrastructure/outbox/), mirroring Whats App Notification Payload:
   the email payload operation with a compact-constructor
   null/blank guard.
   - **DECISION — carry recipient User Id, resolve the get email operation at dispatch** (NOT the raw email address). This
     mirrors dispatch Whats App resolving the phone from the user, and **keeps personal information (the email address) out of the
     outbox events JSONB** at rest — the Personal Data Protection Act-minimizing choice (Casey). Every email recipient is a user:
     forgot-password runs inside the find by email operation so a user is in hand; approval/rejection
     target the post owner. params carries only the non-personal information template vars (e g. the one-time passcode for forgot-password — note
     the one-time passcode IS sensitive; see 1.4).
3. Recording an email must join the caller’s existing database transaction and fail if there is no such transaction. Reuse the outbox’s established structured-payload storage path.
4. **the dispatch email operation** — mirrors dispatch Whats App (149-180): deserialize → resolve the
   user (user Repository find By Id) → if absent, the dropped operation + return (marked SENT, same
   as the WhatsApp drop pattern) → else hand off to the email stream (1.3). Add the dispatch email operation
   to the switch.
5. Record durable email work inside the business transaction at each of the three existing call sites: password reset, post approval and post rejection. Use the account identifier already available there, not the raw email address. The outbox worker performs the queue handoff; these callers must not depend on an in-memory after-commit callback as their only delivery record.

### 1.3 DECISION — the poller's stream handoff must be SYNCHRONOUS
Email Queue Service enqueue (29 @Async) and Whats App Queue Service enqueue Notification (61 @Async) are
fire-and-forget. If the poller calls an @Async XADD, it marks the outbox row SENT **before the XADD actually
lands** — re-introducing a (narrower) lost-message window. **Fix:** the poller path must XADD synchronously, so an
outbox row reaches SENT only once the message is durably in the Redis stream and a throw reschedules the row.
Options (spec picks **A**):
- **(A, chosen)** Add a package-private synchronous the enqueue sync operation to Email Queue Service / Whats App Queue Service
  that does the the ops for stream operation without @Async; the poller calls those; the existing @Async public
  methods stay for any non-poller caller. Smallest blast radius, explicit intent.
- (B) Drop @Async from the poller-invoked method — rejected: changes behaviour for other callers.

**This also retro-hardens the existing WhatsApp outbox path** (same latent "SENT before XADD" gap today). Call it out
as a bonus fix, not scope creep.

### 1.4 Security/personal information note (for Casey)
The forgot-password params carries the **one-time passcode** (already true today, in the email stream message). It now also sits
in outbox events payload (JSONB) until the poller marks the row SENT (seconds). That is a new at-rest location for a
short-lived secret. **Mitigations to decide in review:** (a) accept it — the one-time passcode is single-use, 10-min expiry period, the database is
the same trust boundary as the email stream and the password reset tokens table that stores its bcrypt hash; (b)
store only the one-time passcode *token id* in the payload and re-issue/look-up at dispatch — heavier. Recommend (a) with a note,
since the one-time passcode plaintext already transits the (volatile) email stream and the outbox row is deleted/short-lived;
do **not** log the payload (the poller must never log an EMAIL payload body — mirror the WhatsApp "never log params"
rule, Whats App Queue Service).

### 1.5 Acceptance criteria
- An outbox EMAIL row is INSERTed in the SAME tx as the business write; a rolled-back business tx leaves no email
  row (test: force rollback → assert no outbox events EMAIL row, no XADD).
- Redis down during dispatch → the row stays PENDING and retries with backoff (not lost); after MAX ATTEMPTS → FAILED
  (operator-visible), never silently dropped.
- The poller marks SENT only after a synchronous XADD; a thrown XADD reschedules the row.
- forgot-password / post-approved / post-rejected still deliver end-to-end on the happy path (live-stack IT).
- No email address appears in outbox events payload; the payload is never logged.

---

## 2. Phase 2 (S2) — failed-message queue + bounded retry for the stream consumers (F2)

### 2.1 Problem
Email Consumer Service + Whats App Consumer Service (identical structure) have **no retry cap and no dead-letter**: a
send-failure is not ACKed → XCLAIM-reclaimed every 30s **forever**; a malformed message is silently ACK-dropped with
only a log warn (Email Consumer Service implementation reference). A poison message (permanently-bouncing address, Resend 4xx,
malformed payload) is an infinite loop that also occupies the bounded reclaim batch. The outbox got bounded-retry→
FAILED; these older stream consumers never did. Principle E.

### 2.2 Design — delivery-count cap → dead-letter stream (both consumers)
1. **Read the redelivery count.** Spring Data Redis Pending Message exposes only
   the get elapsed time since last delivery operation, NOT the redelivery count. The count IS in the native XPENDING reply. **Use a
   raw Redis Callback / the stream commands operation** in the reclaim pending operation to read the per-message
   delivery count.
2. **Cap.** Constant failed-message queue MAX DELIVERIES 5 (config-overridable, mirror the outbox MAX ATTEMPTS style). When a
   pending message's delivery count is at least failed-message queue MAX DELIVERIES: XADD a record to a dead-letter stream
   (email-dead-letter / whatsapp-dead-letter) carrying original Message Id, reason, delivery Count, <message
   fields, MINUS any secret/one-time passcode>, then **XACK the poison off the PEL** so the reclaim loop stops.
3. **Malformed drop** (Email Consumer Service, and the WhatsApp equivalent): keep ACKing it (it can never
   succeed) but ALSO failed-message queue it + increment a counter — make the silent drop visible.
4. **Metrics.** These consumers have **no Meter Registry** today; inject it (mirror Outbox Poller). Add
   separate email and WhatsApp failed-message counters, grouped by reasons such as exhausted retries or malformed content
   and a PEL-depth gauge. **Wire the alert into INT-082 PE-3's observability bucket** (the dropped/spend/dead-letter
   dashboard that already gates WhatsApp enablement).
5. ACK-after-send ordering is already correct — do not touch it.

### 2.3 Acceptance criteria
- A message that fails dispatch failed-message queue MAX DELIVERIES times is moved to the dead-letter stream, ACKed off the PEL, and
  increments the failed-message counter for exhausted retries (test: a controlled client that always fails must demonstrate that the message
  lands in the failed-message queue stream after the cap and is gone from XPENDING).
- A malformed message is failed-message queue'd + counted, not just warn-logged.
- No secret/one-time passcode field is copied into the dead-letter record.
- A healthy message still dispatches + ACKs on the first try (no regression).

---

## 3. Phase 3 (S3) — Consume-side idempotency dedup (F3)

### 3.1 Problem
At-least-once + XCLAIM reclaim means a message dispatched-but-not-ACKed (consumer crashed after the Resend/Meta send,
before XACK) is **re-sent** on reclaim → duplicate email / duplicate one-time passcode. Notifications got source Event Id dedup; the
stream consume paths did not. Principle B.

### 3.2 Current design — record success after delivery

Before sending, check whether this stream message already has a successful-delivery record. If it does, acknowledge the repeat without sending again and record a suppressed duplicate. Otherwise attempt delivery, and create the success record only after a successful outcome, before acknowledging the message. The historical retention period for that record is twenty-four hours.

The original draft proposed creating the record before sending. That order is superseded: a failed first attempt would then be mistaken for an already-delivered message and lost on retry. A failed or deferred attempt must remain eligible for retry and must not set the success record.

This ordering still leaves a crash window between external delivery and recording its success. Do not claim exactly-once delivery or that every possible duplicate is prevented. Provider-supported replay protection and the outstanding delivery requirements need their own evidence.

### 3.3 Acceptance criteria
- Re-delivering an already-dispatched message (simulate: dispatch once, then re-invoke dispatch And Ack with the same
  record id) sends ZERO additional emails and ACKs (test asserts the stubbed client send is called once across two
  deliveries).
- A successful first attempt sends once. A failed or deferred attempt remains retryable and creates no success record; restoring the sender permits the same message to be delivered.

---

## 4. Phase 4 (S3) — Commission optimistic lock (F4)

### 4.1 Problem
Admin Commission Service pay Commission guards with invoice Status equals PAID → 409 COMMISSION ALREADY PAID
(58-59). The INT-075 Idempotency-Key dedupes **same-key** retries. But two **distinct-key** concurrent submits
(an admin double-clicks across two tabs that each minted a key; or two admins) are a check-then-write **TOCTOU under
READ COMMITTED** — both read PENDING, both write PAID, both emit the tutor notification + analytics. Same class as
INT-077, on a money path. Commission has no @Version (grep confirms).

### 4.2 Design — mirror the existing Tutee Post.@Version precedent
1. Add a required whole-number version to each commission record, initially zero. The database update must detect when another transaction has changed that version since it was read.
2. **Migration 0048-commission-optimistic-version implementation reference** mirroring 0028c:
   add Column commissions version bigint default Value Numeric is 0 not no value + a rollback drop Column + the
   include entry in db changelog-master implementation reference after the 0047 line (81). Schema head is 0047.
3. **DECISION — the 409 error code.** Global error handler already maps
   Object Optimistic Locking Failure Exception → 409 but **hardcodes JOB ALREADY ASSIGNED / "This job has already been
   assigned."** — wrong copy for a commission. Two options:
   - **(A, chosen)** Make the handler **entity/context-aware**: branch on the get persistent class name operation (or message)
     to distinguish a job already assigned, a commission already paid and a generic concurrent modification, with matching explanations;
     default to a generic CONCURRENT MODIFICATION ("This record was modified concurrently; please retry."). Keeps a
     single handler, correct per-resource copy, and a safe generic fallback for any future @Version entity.
   - (B) Introduce one blanket CONCURRENT MODIFICATION for all — rejected: regresses the precise
     JOB ALREADY ASSIGNED contract the the private implementation folder already depend on.
4. The existing invoice Status equals PAID state guard **stays** (defence-in-depth: it catches the sequential
   replay-after-commit; @Version catches the concurrent race). The Idempotency-Key stays (catches same-key retry).
   Three layers, each for a distinct failure mode — none redundant.

### 4.3 Acceptance criteria
- Two concurrent pay Commission calls with DISTINCT idempotency keys on the same PENDING commission → exactly one
  succeeds (200 + PAID + one tutor notification + one analytics event); the other gets 409 (the new
  COMMISSION ALREADY PAID/CONCURRENT MODIFICATION), and the commission is paid exactly once. (Concurrent-burst IT,
  same shape as Quota Concurrency IT — runs in continuous integration.)
- Sequential replay after the key expiry period still 409s on the state guard.
- JOB ALREADY ASSIGNED behaviour for tutee posts is unchanged (the handler branch preserves it).

---

## 5. Phase 5 (S4) — Idempotency request-fingerprint (F5)

### 5.1 Problem
Stripe errors if a reused Idempotency-Key carries a different request body; Idempotency Service returns the FIRST
stored body silently (81-91). A client bug that reuses a key for a different operation gets a wrong-but-200
response. Latent correctness trap.

### 5.2 Design
1. Extend Idempotency Service Stored Result from int status, text body to
   int status, text body, text request Fingerprint.
2. request Fingerprint SHA-256 scope + : + canonical Request Body — compute on first execution, store alongside
   (96-102); on replay (81-91) recompute and compare → on mismatch throw **422
   IDEMPOTENCY PARAMS DIFFER** ("Request body differs from the original request with this Idempotency-Key.").
3. **DECISION — how the body reaches the service.** The three call sites are
   execute Once/the execute once dynamic operation — they don't currently pass the request body. Pick:
   - **(A, chosen)** add a text request Fingerprint parameter the **controller** computes (it has the
     @Request Body/path) and passes in — explicit, no reflection, no new interceptor; the 3 controllers already build
     the call. Backward-compatible default: a null fingerprint skips the check (so non-fingerprinted callers still
     work during rollout).
   - (B) a servlet filter that caches the body and injects it — rejected for now: body-caching wrappers add
     complexity for 3 endpoints.
4. Idempotency Contract gains ERR PARAMS DIFFER IDEMPOTENCY PARAMS DIFFER. Global error handler already has
   a 422/unprocessable Entity path (used by the profile-completeness gate) to reuse.

### 5.3 Acceptance criteria
- Same key + same body → replay returns the stored result (unchanged behaviour).
- Same key + DIFFERENT body → 422 IDEMPOTENCY PARAMS DIFFER, action NOT re-run.
- A null/absent fingerprint (rollout-compat) → behaves exactly as today.

---

## 6. Cross-cutting

- **No behaviour change** to the WhatsApp narrowing (INT-084/085) or the OFF toggle; Phase 2/3 touch the WhatsApp
  consumer but only add failed-message queue/dedup around the existing dispatch.
- **Gates:** each phase carries the backend gate (backend verification at 97.5% line and branch coverage, no new JaCoCo exclusions; the concurrency ITs in
  Phase 2/4 run in continuous integration via the ci-no-lab Testcontainers band) and its own /team-review + Boss at build time.
- **Migrations:** only Phase 4 (0048). Expand/contract-safe (additive column, default 0). Migrate-then-boot ordering
  per CLAUDE md; ddl-auto is validate requires the column present before the @Version jar boots.
- **Observability:** Phase 2's dead-letter + the existing whatsapp notification.dropped counters feed the INT-082
  **PE-3** dashboard/alert (still OPEN) — this spec adds metrics; wiring the alert remains PE-3's job.
- **personal information/Personal Data Protection Act:** Phase 1 keeps email addresses out of outbox events; the one-time passcode-in-payload note (1.4) is the one item
  for Casey to rule on.

## 7. Build sequencing (later, governed)
**Bundle 1 (S2, headline):** Phase 1 + Phase 2 + Phase 3 — "email as reliable as notifications + no poison loops + no
duplicate sends." One /team-review + Boss; deploy; live-stack IT for forgot-password.
**Bundle 2 (S3, money path):** Phase 4 — migration 0048 + @Version + the handler branch + the concurrent-burst IT.
**Bundle 3 (S4, polish):** Phase 5 — fingerprint.
Each bundle: build → gates → /team-review → Boss → deploy (virtual machine + local) → docs/Postman → push.

## 8. Out of scope / deferred
F6 (WhatsApp Meta client idempotency key) → INT-082 **PE-3** (feature OFF). A full inbox-table per consumer (vs the
lightweight SETNX seen dedup) is deferred unless a stream's volume makes the Redis-key approach insufficient.
