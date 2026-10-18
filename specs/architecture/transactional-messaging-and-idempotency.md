# Transactional Messaging & Idempotency — Engineering Playbook

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** [INT-125](int-125-batch-closeout-spec.md) explicitly carries the INT-132/133 in-tx notification correction; analytics that commits independently remains after-commit. [027](../027-aws-production-deployment/spec.md) requires delivery-preserving stream capacity, drain/reclaim and durable retry. A disabled transport is unavailable, not successful delivery. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Status:** Authoritative reference (cross-cutting). Owners: solution-architect + software-engineer + bff-specialist + dba.
**Created:** 2026-06-08. **Audience:** every engineer touching a write that has a side-effect (notification, email, WhatsApp, payment, analytics, external application programming interface).

> **Why this exists.** Any time a single user action must (a) change our database AND (b) cause a side-effect
> outside that transaction — send an email, emit a notification, call Meta/Stripe, publish an analytics event —
> you are exposed to **two failure modes that silently corrupt the system**:
> 1. **The dual-write problem** — database commit succeeds but the side-effect is lost (or vice-versa). A user is
>    charged but no receipt; a post is approved but no email; an interview is scheduled but nobody is told.
> 2. **Duplicate delivery** — a retry, a redelivery, or a double-click fires the side-effect twice. Two charges,
>    two OTPs, two "you've been matched" messages.
>
> These are not edge cases — at scale they are **certainties**. This playbook is the project's standard answer.

---

## 0. The one-paragraph version

**Never do a database write and an external side-effect as two independent operations.** Make the *intent* to do the
side-effect part of the same database transaction (write a durable row), then let a separate worker perform the side-effect
**at-least-once**, and make every consumer/handler **idempotent** so duplicates are harmless. For synchronous client
mutations that move money or commit state, require a client **Idempotency-Key** and store the first result so a retry
is a no-op. **Exactly-once delivery does not exist** across heterogeneous systems — you get exactly-once *effect* by
combining at-least-once *delivery* with idempotent *processing*.

---

## 1. Principle A — Transactional Outbox (kill the dual write)

**Rule:** the decision to emit a side-effect is written to a durable outbox events row **inside the same
transaction** as the business change. It commits or rolls back atomically with the cause. A separate poller publishes
it afterward. There is never a code path where the business fact is committed and the side-effect intent is not.

**Why:** if you instead call the broker/queue inline (or in after Commit against a volatile queue — see §4), a crash
or a broker outage in the gap loses the message with no record to recover from.

**InTuition reference implementation (textbook — copy this):**
- Recording delivery work must join the caller’s existing database transaction and must fail if no such transaction exists —
  Outbox Service implementation reference (enqueue Notification Fanout, enqueue Whats App Notification). MANDATORY is the load-bearing
  detail: it throws if there is no active tx, making an accidental dual-write a startup/test failure, not a prod silent loss.
- The poller claims a bounded batch with with a record lock SKIP LOCKED (so N pollers never double-publish) ordered by
  available at, over a partial index when status is PENDING — Outbox Poller implementation reference, Outbox Event Repository implementation reference,
  migration 0042b.
- **Bounded** retry with exponential backoff (30s → 1h, MAX ATTEMPTS is 10) and a real terminal FAILED state surfaced
  to operators — Outbox Poller implementation reference. A poison message escalates; it does not retry forever.
- The example to follow when adding any new side-effect: the create and submit operation (the outbox row
  is INSERTed in the post's own tx).

**Checklist when you add a side-effect:**
- [ ] Is the intent persisted in the same tx as the business write? (outbox row, MANDATORY propagation)
- [ ] Does the poller claim with SKIP LOCKED and a PENDING partial index?
- [ ] Is retry **bounded** with backoff and a terminal failed state (not infinite)?
- [ ] Is the dispatch idempotent or the consumer deduped? (Principle B)

---

## 2. Principle B — Idempotent consumer / Inbox dedup (make duplicates harmless)

**Rule:** because delivery is **at-least-once**, every consumer must be safe to run on the same message twice. Carry a
stable unique id on the message and **dedupe on consume** — either a database unique constraint (cheapest, race-proof) or an
"inbox" table the handler checks-and-inserts in one tx.

**Why:** at-least-once is not a weakness to apologize for — it is the *correct* delivery guarantee. The cost of "send
twice" is paid by an idempotent consumer, which is cheap; the cost of "send zero times" (the exactly-once fantasy's
failure mode) is unrecoverable.

**InTuition reference implementation (correct):**
- Notification fan-out carries the outbox event id as source event id; a **partial unique index**
  source event id, recipient user id when source event id has a value (migration 0044) makes a second delivery a
  no-op even if the in-tx exists By… check races — Notification Service implementation reference. Belt (fast-path check) AND braces
  (database constraint). Per-recipient try/catch so one duplicate-violation doesn't block the batch.
- Direct, run-once emits pass source Event Id is no value and are intentionally NOT deduped (they fire exactly once from an
  after Commit hook) — the partial index excludes them.

**The decision rule:** a multi-recipient or at-least-once-delivered notification needs a dedup key; a synchronous
run-once emit does not. Pick the unique key as the *(event, recipient)* pair, not just the event.

**Checklist:**
- [ ] Does the message carry a stable unique id (event id, not a fresh unique identifier per attempt)?
- [ ] Is there a database-level unique guard, not just an application if-exists check (which is a TOCTOU under READ COMMITTED)?
- [ ] For a fan-out, is the key *(event, recipient)* so each recipient is independently exactly-once?

### 2.1 Repeating a sweep can preserve database state while still duplicating notifications
**Rule:** a scheduled sweep that mutates via a status-guarded bulk update records ... when status IN ... is correctly
idempotent for the database (a second run touches 0 rows). But if it then **reads a snapshot list and fans out notifications
in a loop**, two instances (≥2 pods, or an overlapping re-fire) each read the *same* due rows before either commits and
each emit the fan-out → **duplicate notifications**, even though only one bulk-update "won". The write is idempotent; the
side-effect is not.
- **Decision rule:** (a) make the notification carry a source event id keyed on the swept row id and dedupe on the
  source event id, recipient unique index (Principle B) — so a second pod's emit is a no-op; AND/OR (b) ensure the
  sweep is single-flight (a leader lock / pg try advisory lock around the job, or a single scheduler instance). (a)
  alone is sufficient and is the durable answer; (b) alone still duplicates if the lock isn't held. Ordinary notification rows stay in the same transaction with the sweep under §4.1; external intent uses the outbox. Do not defer plain REQUIRED notification writes into after Commit.
- **Live lead (carried from Cycle 1):** Interview Service interview-expiry sweep emits INTERVIEW EXPIRED inline in a
  loop with no source event id dedup → multi-pod duplicate. (Single-instance scheduling masks it today.)

---

## 3. Principle C — Idempotency keys for synchronous mutations (Stripe-shaped)

**Rule:** any client-facing POST that moves money or commits irreversible state requires a client-supplied
Idempotency-Key header. The server stores the **first** outcome (status + body) under that key and returns it verbatim
on any replay, so a network retry or a double-click is a no-op — not a second charge.

**InTuition reference implementation (Stripe-shaped — correct):**
- Idempotency Service execute Once/execute Once Dynamic — Redis SETNX an IN PROGRESS sentinel (24h expiry period); a
  concurrent duplicate while in-flight gets 409 ERR IN PROGRESS; a completed key replays the stored Stored Result
  (status + JSON body); a failure **DELETEs the claim** so the client may retry the same key —
  Idempotency Service implementation reference, Idempotency Contract implementation reference.
- Applied to the three state-committing POSTs: pay-commission, select-tutor, assign — each the request header operation
  **required**, missing → 400 MISSING REQUIRED HEADER (not 500), Global error handler implementation reference. **The contract
  needs BOTH ends** (header required server-side AND minted client-side) — see [[required-contract-needs-both-ends]].
- Defence in depth: the handler ALSO state-guards (PENDING→PAID → 409 ALREADY PAID) so even after the 24h key expiry period
  expires, a late replay can't double-pay — Admin Commission Service implementation reference.

**Stripe parity gaps — both now CLOSED:**
- ✅ **Implemented (INT-087 Phase 5).** A reused Idempotency-Key with a **different request body** now errors
  422 IDEMPOTENCY PARAMS DIFFER (a request-fingerprint of the first call is stored and compared) — the action is NOT
  re-run, rather than silently returning the first body — Idempotency Contract implementation reference.
- ✅ **Implemented (INT-087 Phase 4).** Commission now carries @Version (migration 0048), so two *distinct* keys
  racing the same PENDING commission can't both win — the loser gets 409 COMMISSION ALREADY PAID via an entity-aware
  Object Optimistic Locking Failure Exception handler (generic 409 CONCURRENT MODIFICATION fallback for other entities) —
  Commission implementation reference, Global error handler implementation reference. The key + state-guard now pair with a @Version as Stripe
  prescribes.

**Checklist:**
- [ ] Does the mutation move money or commit irreversible state? → require Idempotency-Key.
- [ ] Is the first (status, body) stored and replayed verbatim, with a sentinel for in-flight concurrency?
- [ ] Is there a second-line state guard for after the key expiry period expires?
- [ ] Client minted with the safe uuid operation and sent on EVERY such request? (the [[secure-context-apis-break-on-http-vm]] trap)

---

## 4. Principle D — after Commit is for *triggering* durable work, never for *being* the durable record

**Rule:** Transaction Synchronization after Commit correctly ensures a side-effect does NOT fire on rollback. But the
work it triggers must itself be durable. **after Commit → call a volatile queue is still a lost-message window**: the
tx has committed, and if the async enqueue then fails (broker down, pod crash in the gap), the side-effect is lost with
no record to retry from — the dual-write problem moved by one line, not solved.

**InTuition state:**
- ✅ Notifications use the durable outbox (after Commit only triggers the poller indirectly via a committed row).
- ✅ **Implemented (INT-087 Phase 1).** **Emails** (forgot password, post approved, post rejected) now ride the
  durable outbox: a new Outbox Event Type EMAIL + Email Payload (carries the recipient **user Id**, not the raw address —
  personal information stays out of outbox events payload) enqueued via Outbox Service enqueue Email (MANDATORY propagation, in the
  business tx). The poller's dispatch Email resolves user→the get email operation and hands off **synchronously** to the
  email-notifications stream, so the outbox row reaches SENT only after the XADD lands — Outbox Poller implementation reference,
  Email Payload implementation reference, Outbox Service implementation reference. The three former after Commit → @Async → XADD sites are repointed to
  this in-tx outbox path. The highest-stakes transactional message (password reset) now has the **strongest** guarantee,
  not the weakest. The poller NEVER logs the EMAIL payload body (the forgot-password params carries the one-time passcode).

**Decision rule:** after Commit is the right place to *kick off* a side-effect whose intent is ALREADY durably recorded
(outbox row, or the business row itself). It is the WRONG place to *originate* a side-effect whose only record is the
volatile call you're about to make. Route originating side-effects through the outbox.

**Also (correctness of the hook itself):** an after Commit body must not throw in a way that surprises the caller — the
tx is already committed; wrap in try/catch, log, never rethrow into the synchronization (a thrown after Commit can mask
as a failed request while the data is committed).

### 4.1 Optimistic concurrency: classify the side-effect by transaction ownership

The later [INT-125 design](int-125-batch-closeout-spec.md) explicitly applies the INT-132/133 correction: an ordinary in-app notification is a database row in the business transaction. It rolls back with a losing optimistic transaction, just like the state mutation. Keep those rows in-tx, with the recorded recipient isolation/dedup policy; do not move a plain REQUIRED write into a dead after Commit context.

- External email/WhatsApp intent is a durable outbox row in that same transaction. Broker/network dispatch happens after commit through the outbox poller, with synchronous handoff before SENT and consumer ACK only after success or a completed durable failed-message queue handoff.
- Analytics that writes through a separately committed/REQUIRES NEW store must originate only after successful business commit, or through an explicitly durable event intent. A losing optimistic transaction must not persist a phantom event. Capture immutable values before deferring; do not touch lazy entities after commit.
- A database write deliberately executed after commit needs a fresh transaction; plain REQUIRED may join the already-committed context and lose its write. That is not a reason to defer ordinary in-app rows.
- Acceptance: race competing transitions and assert exactly the winning business outcome, its committed notification/outbox rows, and no persisted analytics for losers. Mock call counts alone cannot disprove correct rollback behavior. Inject rollback and confirm no recipient delivery escapes it; open the actual persisted notifications/event records.

The earlier blanket instruction to move every notification out of an optimistic transaction is superseded. The boundary is whether the effect commits/rolls back with the business transaction, not merely whether a method calls a notification service.

---

## 5. Principle E — At-least-once everywhere; bound the retries; failed-message queue the poison

**Rule:** every at-least-once consumer needs (1) a retry cap, (2) a backoff, and (3) a **dead-letter** destination for
messages that exceed the cap. A message that fails forever must escalate to a human, not loop silently.

**InTuition state:**
- ✅ The **outbox** poller does this right: MAX ATTEMPTS is 10 → terminal FAILED, backoff, operator-visible.
- ✅ **Implemented (INT-087 Phases 2 + 3).** The **Redis-stream consumers** (Email Consumer Service,
  Whats App Consumer Service) now have a **failed-message queue**: past dlq max-deliveries (default **5**, read from the XPENDING
  delivery count) a poison message is dead-lettered to email-dead-letter / whatsapp-dead-letter (reason is max redeliveries)
  and ACKed off the PEL so the reclaim loop stops; a malformed message is dead-lettered on first sight
  (reason is malformed). The one-time passcode field is never copied into the failed-message queue record. They also gained **consume-side dedup** (a
  seen key per message id, set **only after a successful send** — see the new anti-pattern in §8) and metrics
  (counts of messages sent to the failed-message queue by reason, pending-message depth, and suppressed duplicate deliveries) — Email Consumer Service implementation reference,
  Whats App Consumer Service implementation reference. (Alert wiring on these is owed under INT-082 PE-3.)

**Checklist for any queue/stream consumer:**
- [ ] Retry cap (e g. 10) with backoff?
- [ ] Past the cap → dead-letter stream/table + a metric + an alert?
- [ ] ACK only **after** the side-effect succeeds (at-least-once), never before (that's at-most-once / silent loss)?
- [ ] Pod-unique consumer name + idle-based XCLAIM reclaim so a dead pod's PEL is recovered?

---

## 6. Principle F — Webhook handlers are idempotent and signature-verified

**Rule:** inbound webhooks (Meta, Stripe) are retried by the sender and can be replayed by an attacker. Verify the
signature in **constant time**, then make the handler idempotent (dedupe on the provider's event id, or keep it
side-effect-free).

**InTuition reference (correct):** Whats App Webhook Controller — X-Hub-Signature-256 keyed message authentication + verify-token both compared
with Message Digest is Equal (constant-time), handler is side-effect-free (logging + metric), so Meta's retries are
harmless by construction. When the Stripe webhook lands (roadmap #4a), it MUST dedupe on the Stripe event id (inbox
pattern) before transitioning commission state — replay of checkout session.completed must be a no-op.

---

## 7. When to use which (decision matrix)

| Your write has a side-effect that is… | Use |
|---|---|
| An in-app notification / fan-out (at-least-once OK, must survive crash) | **Outbox** (Principle A) + event,recipient dedup (B) |
| An email / SMS / WhatsApp / external application programming interface call originated by a database change | **Outbox** (A) → queue → idempotent consumer (B, E). *Not* after Commit → volatile enqueue.* |
| A synchronous client POST that moves money / commits state | **Idempotency-Key** store (C) + state guard + @Version |
| An analytics/measurement event (loss-tolerant, must not fire on rollback) | after Commit (D) is acceptable — it's best-effort by definition |
| An inbound provider webhook | Constant-time signature verify + idempotent handler (F) |
| A pure read, or a side-effect that is part of the business fact (audit row) | In-transaction, atomic with the change (no deferral) |

---

## 8. Anti-patterns (reject these in review)

1. **Calling the message broker inside the database transaction** — dual write; broker slowness also holds the database lock.
2. **Sending from an in-memory callback after commit, with no durable delivery record** — lost-message window (Principle D).
3. **if !exists insert with no database unique constraint** — TOCTOU under READ COMMITTED (same class as INT-077); add the
   unique index.
4. **Unbounded consumer retry / no failed-message queue** — a poison message loops forever and starves the queue (Principle E).
5. **Marking a message SENT/ACKed before the side-effect succeeds** — silent loss (at-most-once wearing at-least-once
   clothes).
6. **A required Idempotency-Key wired on only one end** — 400/500 storms; the contract needs both ends.
7. **Trusting a provider for exactly-once** — design for at-least-once + idempotent processing regardless.
8. **A dedup/idempotency sentinel guarding a fallible side-effect set BEFORE the side-effect** — this converts
   at-least-once into at-most-once on failure (the seen-key survives a failed send → the retry is suppressed → silent
   loss). Set the sentinel **only on SUCCESS** (set-after); review by tracing the failure-then-retry path. *(A real bug
   caught in code-review on the INT-087 consume-side dedup — the seen key was originally SETNX'd before the send, so
   a message whose first send threw was deduped+ACKed on reclaim and never sent. Fixed to set-after-success in
   Email Consumer Service implementation reference / Whats App Consumer Service implementation reference.)*
9. **An independently committed/external effect escaping an optimistic loser** — retain ordinary notification/outbox rows in-tx; defer independent analytics until commit and dispatch external work through durable intent (§4.1).

---

## 9. Sources

- [Microservices io — Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html) ·
  [Amazon Web Services Prescriptive Guidance — Outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html) ·
  [Event-Driven io — Outbox/Inbox & delivery guarantees](https://event-driven.io/en/outbox_inbox_patterns_and_delivery_guarantees_explained/)
- [Stripe — Designing robust application programming interfaces with idempotency](https://stripe.com/blog/idempotency) ·
  [Stripe application programming interface — Idempotent requests](https://docs.stripe.com/api/idempotent_requests) ·
  [brandur org — Implementing Stripe-like idempotency keys in Postgres](https://brandur.org/idempotency-keys)
- [Redis — Build a Redis-backed job queue with retries and dead-letter handling](https://redis.io/tutorials/redis-backed-job-queue-for-background-workers/) ·
  [Redis antirez — Streams consumer group patterns](https://redis.antirez.com/fundamental/streams-consumer-patterns.html)
