# Design — Generic scheduler platform (db-scheduler) + migrate existing tasks (007)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> How the requirements in [requirements md](requirements.md) will be built. Owned by **solution-architect** (lead) +
> **software-engineer** (registry + per-instance/recurring wiring + Shed Lock removal), with **dba** (migrations
> 0062/0063), **devops-engineer** (observability + alert + give-up policy), **qa-engineer** (re-guard + live-convergence),
> **security-expert** (new dependency + no-personal information-in-scheduled tasks contract), and the **Technical Writer** ("register a
> scheduled job" reference doc). Grounded against the real code read on 2026-06-21 (file:line citations throughout).
> **Honest scope:** this is an internal mechanism swap — the verifiable surface is scheduled tasks row correctness,
> handler re-guard, single-ownership (no double-fire), and metrics; WhatsApp stays OFF so reminder evidence is outbox
> rows, not a live send. No frontend change, no new web request protocol endpoint.

## Current architecture contract — corrected 2026-09-05

The original June implementation/evidence is historical. This section defines the required design for REQ-001–010, including safeguards that remain to be implemented and verified before WhatsApp enablement.

### Registration, generation and transaction boundaries (REQ-002–005)

Keep the generic Scheduled Job Service wrapper and registered task beans; business services do not depend on db-scheduler types. Reminder task identity is interview-reminder, interview Id:generation, with no serialized personal information or meeting link. Recurring interview expiry and showcase reroll retain their existing business handlers; outbox/stream pollers retain their independent scheduling/locking.

Under an interview-row lock, every relevant lifecycle mutation updates business state and a monotonic reminder generation, and writes a durable schedule-reconciliation intent in the same transaction. Time/link changes or terminal transitions invalidate the old generation; identical saves do not. At that mutation, persist nullable reminder due at for the new generation: scheduled At−24h only when linked/SCHEDULED and strictly >24h away at mutation time, otherwise null. The intent identifies only the interview and generation. It is not an external after Commit callback: a crash after business commit must not lose registration.

An idempotent reconciler reads the current generation, registers its distinct one-time key at the persisted reminder due at when the current generation remains SCHEDULED/linked with a non-null due time and the interview has not begun, cancels only obsolete unpicked reminder keys whose generation is strictly LESS than its captured snapshot generation (never equal/newer keys, and never “all other tasks for this interview”), and marks only that same generation’s intent reconciled (compare-and-set) after registration/cancellation is confirmed; it must never acknowledge a newer generation’s intent. Failure leaves it retryable. After registration, re-read current generation before acknowledging the intent. If it advanced, cancel only the stale worker’s exact own task key and mark only its own old intent superseded; an executing old row may finish as a guarded zero-intent no-op. Never cancel or acknowledge a newer generation from an older snapshot. A picked old row may complete independently; generation checks make it harmless. Reconcile new lifecycle events after restart, but do not invent a startup backfill of pre-cutover interviews (founder R2). A mutation made at or within24h authorizes confirmation only (null due time). A recovered originally eligible generation with an overdue due time runs as soon as possible before scheduled At; restart delay must not silently discard it.

### Durable reminder intent and stale-task guard (REQ-003/004)

The handler transaction locks the interview row and verifies current generation, SCHEDULED status, current nonblank link, non-null reminder due at and due window (reminder due at is at most now scheduled At). A mismatch is a normal zero-intent return. Under that lock, for each intended recipient insert a durable reminder-intent key unique on interview id, generation, recipient user id, event type and enqueue the corresponding existing transactional outbox row **in the same commit**. Use append records ... when a duplicate is found leave the existing record unchanged returning the resulting fields (or equivalent non-aborting atomic insert-if-absent), and enqueue only when this transaction inserted the key. An existing key is an already-enqueued no-op for that recipient; do not catch a uniqueness exception inside an aborted PostgreSQL transaction. any enqueue failure rolls back its key and the transaction. Capture the current timing/link only inside that guarded transaction and never log the payload.

The durable key survives scheduler acknowledgement/removal, outbox cleanup and retry. Do not remove it while the generation can be replayed; retention follows the business/audit retention policy, not scheduler row lifetime. This guarantees once-only committed outbox intent per recipient/generation, not exactly-once provider delivery. Crash after commit/before scheduler acknowledgement is safe because retry sees the key. Scheduler row locking alone and swallowed Task Instance Currently Executing Exception do not meet this guarantee.

Use the same interview lock in handler and lifecycle updates. If cancellation/reschedule commits first, the stale handler emits zero intents; if the handler commits first its prior-generation intent is already authorized and cannot be recalled. Test both serialized outcomes; do not claim cancellation retracts an already-committed send.

### Data model and migration (REQ-001/006)

Historical applied migrations remain 0062 (scheduled tasks, initializer disabled) and 0063 (remove reminder sent/index). Do not rewrite them or restore the old flag. Add reminder generation, nullable reminder due at, durable reconciliation intent, and recipient reminder-intent uniqueness state through **new additive Liquibase changesets using the next unallocated IDs**. Preserve existing row data and soft-delete conventions; index pending reconciliation and the unique reminder identity. The intent/key tables contain IDs/state only, no phone/link/name. Backfill generation defaults and null reminder due at safely for existing rows without scheduling old interviews. Applied database structure definitions and runtime behavior are not certified here.

### Observability, failure and enablement (REQ-008/010)

Preserve per-task success/failure counters, execution-latency timer, due/overdue gauge, interview expired* and reminder counters. A reminder “sent” counter must be labelled as enqueue success if measured at the outbox boundary. Four retries after the initial attempt (five attempts total), then a give-up counter and structured log; keep recipients/meeting links out of logs. Recurring jobs continue on their next tick. Polling interval 10s, five threads. Founder-approved current rollout is **no startup backfill; metrics while OFF; Prometheus failure alerts deferred until before enablement**.

Global WhatsApp OFF and the event/opt-in/verified-phone gates remain unchanged. Before enablement require verified durable dedup, generation guards, reconciliation recovery, alerting and spec006's consent-copy/test/event parity. This documentation revision does not enable notifications or claim those new checks pass.

### Required verification (all pending for this corrected contract)

1. Migrations apply via Liquibase; no library database structure definitions; generic/recurring tasks retained; Shed Lock not reintroduced.
2. >24h linked schedule → unique current-generation job at due time;  is at most 24h/no link → none; no startup backfill.
3. Two concurrent executions, duplicate delivery and crash-after-outbox-commit → one durable intent/outbox row per recipient/generation, with rollback on enqueue failure.
4. Picked-task race with reschedule, link replacement/clear and every terminal transition: both lock orders and stale/future/due window guards.
5. Business-commit/registration crash and restart, including crossing the due time: durable reconciliation restores the correct new task without duplicate intent or loss; a genuinely short-notice mutation remains unregistered and an identical save does not re-arm.
6. Reconciler interleaving: pause G1 after snapshot; commit/register/acknowledge G2; resume G1. G1 cannot cancel G2 or mark G2 reconciled, and G2 remains scheduled at its due time. G1 cleanup touches only its exact stale key/older keys; any stale execution emits zero intents.
7. Metrics/five-attempt cap/give-up; OFF and all eligibility gates; consent copy/event parity and failure alert before any enablement.
8. Application rollback preserves current generation/due eligibility, pending reconciliation and dedup/outbox rows; a compatible corrected binary resumes pending work without a missing reminder intent or duplicate enqueue.

## Current rollout and rollback contract (REQ-001/003/005/006/010)

- **Rollout:** keep WhatsApp OFF; apply new additive Liquibase migrations first, then deploy a binary that understands generation-qualified task keys, due eligibility and durable intent tables. Keep historical0062/0063 unchanged. Run the crash/race/reconciliation checks above and the existing consent/alert gates before any separate enablement approval.
- **Rollback:** turn/keep WhatsApp OFF and pause reminder execution/reconciliation plus the affected WhatsApp outbox dispatch while switching binaries, so the usual OFF-drop branch cannot discard the rollback fixture’s pending work. Preserve scheduled tasks, interview generation/due state, durable reconciliation intents, recipient dedup keys and pending outbox rows; no drop/truncate/reset-to-zero and no acknowledgement merely because delivery is disabled. The former0063→0062 table-drop recipe is historical and is not a compatible rollback for this contract.
- A rollback target must support the current schema and generation-qualified task/intent formats. If the older binary cannot, keep the reminder producer/reconciler paused and WhatsApp OFF until a compatible corrected binary resumes; do not run an incompatible handler on the new task keys. Other business operations must preserve generation/intent writes or remain paused for this affected flow. Reconcile pending work on resumption using persisted original eligibility, including overdue-but-not-started eligible interviews.
- **Required proof:** prepare an eligible pending generation, a committed dedup/outbox intent and an unacknowledged reconciliation item; exercise the documented compatible application rollback/resume; verify every durable record survives, pending work resumes, and no duplicate recipient intent is created. Any schema rollback that drops a required relation is forbidden while a runnable binary still queries it. This is an implementation acceptance requirement, not an executed rollback claim.

## Historical June decisions (ADR log)

The mechanism arguments below explain the original migration. Where they describe interview Id-only task identity, scheduler-row-only idempotency, or lifecycle hooks without durable generation reconciliation, they are superseded by the current design above. They are not instructions for rebuilding the unsafe assumptions.

### ADR-001 — Adopt db-scheduler (vs bespoke sweeps / a hand-rolled scheduled jobs table / Quartz) (2026-06-21)
- **Context:** Three time-based jobs are bespoke a framework-managed scheduled method + Redis-Shed Lock methods embedded in services; a new
  time-based use case means another hand-written sweep + another Shed Lock name. The founder wants a generic, persistent,
  cluster-safe scheduler so future tasks just *register* a handler. Options considered:
  - **(a) Keep bespoke a framework-managed scheduled method + Shed Lock sweeps.** Lowest immediate effort, zero new dependency. Rejected: it is
    exactly the coupling we are removing — every new job re-writes leader election + idempotency + a sweep loop, mixing
    "when to run" with domain logic; no per-instance precise scheduling (only rolling sweeps); doesn't scale.
  - **(b) Hand-roll our own scheduled jobs table + a poller.** Full control, no third-party dep. Rejected: we would
    re-implement (and have to test/operate) leader election, locking, retries, heartbeats, and failure handling — a
    well-solved problem with subtle concurrency bugs; high build + maintenance cost for no differentiation.
  - **(c) Quartz.** Mature and feature-rich. Rejected: heavyweight for our need (clustering config, Job Store tables,
    its own serialization model, a larger operational surface); its persistent Job Store is more machinery than three
    jobs + occasional one-time jobs warrant; steeper learning curve.
  - **(d) db-scheduler (com github.kagkarlsson:db-scheduler-spring-boot-starter, 16.x).** **CHOSEN.** Lightweight,
    PostgreSQL-backed (one scheduled tasks table — reuses our existing database, no new infra), database-row leader election
    (cluster-safe **without** Shed Lock), first-class **one-time** and **recurring** tasks, a Spring Boot starter that
    auto-registers Task beans (the clean "register a handler" surface we want), and a small footprint. SB-3.3 / Java
    21 compatible.
- **Decision:** Adopt **db-scheduler** (d). It directly satisfies "register a handler", gives persistent + cluster-safe
  execution on our existing PostgreSQL, and makes Shed Lock redundant (one election mechanism, not two).
- **Consequences:** (+) New jobs are one Task a registered application component + a handler; Shed Lock and its Redis dependency for scheduling go
  away; per-instance one-time scheduling becomes trivial; survives restarts (database-backed). (−) A new third-party
  dependency to track/patch; we own its table via Liquibase (db initializer is no); db-scheduler emits no Micrometer
  metrics by default, so we add an observability shim (REQ-008). The trade-off is accepted: a small dep + a metrics
  shim vs. hand-rolling a scheduler or carrying two election mechanisms.

### ADR-002 — Interview reminder as a per-instance ONE-TIME job (vs a recurring reminder sent sweep) (2026-06-21)
- **Context:** "24h before" can be (i) a rolling recurring sweep selecting interviews that have entered the 24h window,
  keyed on a reminder sent flag (spec-006's mechanism, migration 0061), or (ii) a per-interview **one-time** job
  scheduled precisely at scheduled At−24h. db-scheduler makes (ii) first-class.
- **Decision:** Use a **per-instance one-time job** (ii): instance Id interview Id, **no serialized payload**,
  registered at schedule time when has Link and scheduled At now+24h, with a **mandatory handler re-guard** (still
  SCHEDULED + linked) as the safety net and **every lifecycle hook** cancelling/rescheduling the row. Interview
  **expiry** and **showcase reroll** stay (iii) **recurring** tasks — they are periodic scans with no per-instance
  trigger, so a recurring task is the natural fit; their existing sweep bodies move unchanged.
- **Consequences:** (+) Precise timing (fires at scheduled At−24h, not within a sweep interval); no reminder sent
  flag or partial index needed (the scheduled tasks row IS the state — drop them via 0063); no rolling sweep to
  operate for the reminder; the row is naturally dedup'd by name, instance Id. (−) Correctness now depends on
  **every** lifecycle transition cancelling/rescheduling the job (REQ-005) — mitigated by the mandatory, tested
  re-guard (REQ-004) which makes a missed hook fail safe (0 sends), and by idempotent cancel. The recurring expiry/
  showcase tasks retain the proven status-guard idempotency. Net: precise, lower-state, well-guarded — accepted.

### Supersession note (spec-006)
This spec **supersedes spec-006's reminder *mechanism*** — the rolling reminder sent-keyed sweep
(Interview Service send Due Interview Reminders) and migration 0061's reminder sent column + idx interview reminder due
index — replacing it with the per-instance one-time job (ADR-002) and dropping the column/index (migration 0063, REQ-006).
**User-facing behaviour is preserved byte-for-byte** (24h-before, both parties, link-gated, ≤24h → none, same template /
copy / Singapore time format / opt-in gating — REQ-010). The INTERVIEW REMINDER event/template, Outbox Service, and
Sgt Date Time Formatter are reused unchanged. Spec-006's other REQs (REQ-001 link-gated schedule-notify, REQ-002 the
parameterized pipeline, REQ-006 the event, REQ-007 the templates, REQ-008 dormancy) are **NOT** superseded — only its
reminder delivery mechanism. The Boss marks spec-006 **SUPERSEDED-in-part** at 007's ship (it is not this author's to
flip; recorded here + in 007's changelog).
