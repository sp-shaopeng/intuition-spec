# Requirements — Generic scheduler platform (db-scheduler) + migrate existing tasks (007)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin**
> acceptance criteria and the **exact evidence** that will prove each scenario. Negative/guard behaviours are their own
> scenarios. Source: the approved, team-pre-vetted plan (twinkly-percolating-glade md) — founder decisions: adopt
> **db-scheduler** (not hand-rolled); model the 24h interview reminder as a **per-instance one-time job**; **migrate all
> three time-based tasks and remove Shed Lock entirely**. The plan's architecture pre-vet (2026-06-21) folded five
> conditions into these requirements (see spec md "Status"). All file:line citations were verified on 2026-06-21.

## Scope note (what exists — verified in code, NOT re-specified here)

- **Three time-based a framework-managed scheduled method + @Scheduler Lock tasks** (the migration targets):
  - Interview Service expire Stale Negotiations — Interview Service implementation reference, lock interview-expiry-sweep
    (Shed Lock Config LOCK INTERVIEW EXPIRY), the interview-expiry sweep interval, defaulting to fifteen minutes (15 min),
    status-guarded idempotent set-based update records, metric interview expired (+ …errors).
  - Interview Service send Due Interview Reminders — Interview Service implementation reference, lock interview-reminder-sweep
    (LOCK INTERVIEW REMINDER), 15 min, reminder sent-flag idempotent, metric whatsapp interview reminder sent
    (+ …errors). (The spec-006 reminder — this requirement set **supersedes its mechanism**.)
  - Showcase Service scheduled Refresh — Showcase Service implementation reference, lock showcase-reroll
    (LOCK SHOWCASE REFRESH), the showcase refresh interval, defaulting to ninety seconds (90 s), stateless reroll of both pools.
- **Shed Lock** = Redis provider (Shed Lock Config implementation reference, @Enable Scheduler Lock, Redis Lock Provider;
  shedlock-spring + shedlock-provider-redis-spring.16.0, pom implementation reference). Only those three locks exist.
  db-scheduler does its **own database-row-based** leader election → Shed Lock becomes removable once all three move.
- **Do NOT migrate** (continuous pollers with their own locking, NOT scheduled jobs): Outbox Poller poll
  (with a record lock SKIP LOCKED), Email Consumer Service/Whats App Consumer Service poll/reclaim Pending
  (Redis-Streams PEL). They keep a framework-managed scheduled method, so **@Enable Scheduling STAYS** (Intuition Application implementation reference).
- **Build:** Spring Boot **3.3.0**, Java 21, Maven (pom implementation reference). No existing scheduler/Quartz dep. PostgreSQL
  driver present; Liquibase ledger latest = **0061** (db changelog-master implementation reference).
- **Interview lifecycle hooks** (verified — every state-change path the reminder job must track):
  schedule Interview (490), set Meeting Link (626), complete Interview (642), admin Cancel Interview
  (663), mark No Show (686), cancel My Interview (326), cancel Tutor Interview (419), the shared
  the cancel operation helper (983), and the expiry sweep flipping PROPOSED/COUNTER PROPOSED → EXPIRED (733).
- **Reuse (unchanged):** the enqueue whats app notification operation
  (Outbox Service implementation reference), Whats App Template Keys INTERVIEW REMINDER, util/Sgt Date Time Formatter (incl. the
  the to offset date time utc operation Instant fix), the INTERVIEW REMINDER event/template. The reminder sent column +
  idx interview reminder due (model Interview implementation reference; repo find Due Reminder Candidates/mark Reminder Sent
  Interview Repository implementation reference; migration 0061) are **removed** by this spec (REQ-006).

---

## REQ-001 — db-scheduler dependency + the persistent scheduled tasks table (migration 0062, Liquibase-owned database structure definitions)

- **Type:** infrastructure / data
- **Priority:** Must
- **Source:** Plan §A. Founder decision: adopt db-scheduler (not hand-rolled).
- **Rationale / value:** A generic, persistent, cluster-safe scheduler needs (a) the library on the classpath and
  configured, and (b) its backing table. The project runs ddl-auto is validate and **never auto-creates** schema — so
  db-scheduler's own database structure definitions initializer must be **disabled** (db-scheduler db.initializer is no) and the
  scheduled tasks table must ship as a **Liquibase changeset** (migration 0062) with a the reviewed database rollback, exactly like
  every other schema change.
- **Owner (specialist):** dba (migration) · software-engineer (dependency + config)
- **Acceptance criteria (Gherkin):**


- Scenario: The db-scheduler starter is on the classpath and the app boots.
- Given pom implementation reference declares com github kagkarlsson:db-scheduler-spring-boot-starter (SB-3.3-compatible, 16.x).
- And application implementation reference sets db-scheduler enabled is yes and db-scheduler db initializer is no.
- When the application starts.
- Then the db-scheduler Scheduler bean is created and polling, and no database structure definitions is auto-applied by the library.
- Scenario: Migration 0062 creates the scheduled tasks table via Liquibase (NOT the library).
- Given the Liquibase changelog is applied (mvn verify / Testcontainers).
- When the database is inspected.
- Then a scheduled tasks table exists with db-scheduler's columns (task name, task instance, task data,.
- And databasechangelog records changeset 0062.
- And ddl-auto is validate boots clean against it (no Hibernate validation error).
- Scenario: 0062 is reversible (guard).
- Given changeset 0062 has applied.
- When the reviewed database rollback is applied.
- Then the scheduled tasks table is dropped.


- **Required evidence:** pom implementation reference diff showing the starter + version; application implementation reference showing
  db-scheduler enabled, db initializer is no, polling/threads/table-name; \d scheduled tasks output;
  confirm the migration history includes the required 0062 entries; a boot log showing the Scheduler started with no
  library-side database structure definitions; the changeset id 0062-db-scheduler-tasks implementation reference.
- **Dependencies:** none (latest changeset at authoring = 0061; confirm no number collision at build)

## REQ-002 — Generic registration surface: a scheduled job = one Task a registered application component + a handler (first-class)

- **Type:** functional / architecture
- **Priority:** Must
- **Source:** Plan §B. Founder goal: "future tasks just *register* a handler."
- **Rationale / value:** The whole point of adopting db-scheduler is that adding a new time-based use case must be a
  **bounded, repeatable, low-coupling** act — declare a db-scheduler Task<T> as a registered application component (one-time or
  recurring) and supply a handler; **no new service-embedded a framework-managed scheduled method, no new Shed Lock name, no new bespoke sweep
  loop.** This must be a real, documented surface (a config/scheduler package holding the Task beans, an optional
  thin Scheduled Job Service wrapper over Scheduler Client for schedule Once / reschedule / cancel so callers
  never touch db-scheduler types directly) — mirroring the existing "register a handler by type" ergonomics of the
  transactional outbox dispatch. It ships with a **worked example** in the reference doc so "register for other use
  cases" is demonstrably real, not aspirational.
- **Owner (specialist):** solution-architect (surface design) · software-engineer (impl) · Technical Writer (ref doc)
- **Acceptance criteria (Gherkin):**


- Scenario: Each migrated job has one registered scheduler task and handler; the older framework scheduling mechanism no longer runs that job.
- Given the scheduler package exposes the migrated jobs as db-scheduler Task @Beans.
- When the application context starts.
- And none of the three migrated jobs still runs through the older framework-managed scheduled method.
- Scenario: A new use case is added by declaring ONE bean + a handler (worked example, guard).
- Given the "register a scheduled job" reference doc's worked example.
- When a developer follows it to add a hypothetical new one-time or recurring job.
- Then the new job needs one registered task and handler, plus explicit scheduling for a one-time job; it does not require a new framework-scheduled method or lock configuration.
- Scenario: Callers schedule/reschedule/cancel via the wrapper, not raw db-scheduler types (guard).
- Given the thin Scheduled Job Service over Scheduler Client.
- When a caller schedules, reschedules, or cancels a one-time job.
- Then it calls schedule Once using task Instance Id, run At / reschedule / cancel and does not import db-scheduler.


- **Required evidence:** the scheduler/config package source listing the Task a registered application components; a context/IT log
  enumerating the registered task names; a grep proving the three migrated jobs carry no a framework-managed scheduled method; the committed
  "register a scheduled job" reference doc with the worked example; the Scheduled Job Service wrapper source.
- **Dependencies:** REQ-001

## REQ-003 — One-time reminder intent with durable deduplication

- **Type:** functional/data; **Priority:** Must; **Source:** original once-only reminder requirement, clarified against the recorded crash gap on 2026-09-05.
- A SCHEDULED interview with a nonblank meeting link and scheduled At strictly > now+24h registers a one-time job for scheduled At−24h. At or within 24h, the scheduling confirmation is the only reminder; no catch-up job. Payload is only interview ID plus non-sensitive schedule generation, never contact/link data.
- At each relevant mutation, persist whether that generation qualified (>24h with link), represented by nullable reminder due at scheduled At−24h; null means no reminder was authorized for that generation. Reconciliation after a restart must not reclassify an originally eligible generation as short-notice merely because time elapsed. Each relevant schedule/link change advances a durable monotonic reminder generation; the task key includes interview ID and generation. Repeated identical saves preserve generation. Terminal transitions invalidate it. A distinct new-generation key prevents a picked old task from blocking the replacement.
- **Durable once-only boundary:** at most one committed reminder **outbox intent** per interview/generation/recipient. A durable uniqueness record and corresponding outbox enqueue commit in the same transaction. A scheduler-row deletion or successful library execution alone does not prove this. Provider delivery remains at-least-once/retry-dependent; do not call this exactly-once WhatsApp delivery.
- **Acceptance/evidence:** eligible job at the correct due time;  is at most 24h/no-link register none; task payload has no personal information; repeated/concurrent executions and a crash after business/outbox commit but before scheduler acknowledgement create no second outbox intent. Verify both recipients independently, including partial retry. These safeguards are mandatory before WhatsApp enablement, not claimed implemented by the historical shipment evidence.

## REQ-004 — Generation, status and due-time guards serialize with lifecycle changes

- **Type:** functional/concurrency; **Priority:** Must.
- At fire time, start a transaction and lock the current interview row; require task generation  equals  current reminder generation, status SCHEDULED, nonblank current link, non-null current-generation reminder due at, and reminder due at  is at most  now < scheduled At. A failing guard returns normally with zero reminder intents. Read parameters only after this guard under the same transaction.
- Lifecycle mutations use the same row lock. Check/insert dedup state and enqueue outbox intents in that transaction. Thus a mutation that commits first makes the old handler stale; a handler that commits first legitimately records the prior generation before the mutation. Already-committed external delivery cannot be recalled and is not promised cancelable.
- **Acceptance/evidence:** canceled, completed, no-show, expired, link-cleared, future-replaced and stale-generation jobs each produce zero intents. Force both race orders for a picked job versus reschedule/cancel/link replacement. A still-valid, due, current generation produces one intent for each recipient; a second run produces none.
- **Dependencies:** REQ-001/002/003/005 and the existing transactional outbox.

## REQ-005 — EVERY interview lifecycle transition cancels-or-reschedules the reminder job (enumerated, none missed)

- **Type:** functional
- **Priority:** Must
- **Source:** Plan §C; architecture pre-vet condition 3.
- **Rationale / value:** Correctness needs all hooks (the re-guard backstops a gap but is not a substitute). Every
  interview state-change path that invalidates or moves the reminder must **cancel or re-register** the one-time job so
  scheduled tasks stays in sync with interview state. The enumerated, verified hooks (file:line above):
  **(a) reschedule** (schedule Interview 490 to a new scheduled At) → cancel + (if >24h & link) re-register at the
  new scheduled At−24h; **(b) cancel** — admin Cancel Interview (663), cancel My Interview (326),
  cancel Tutor Interview (419), all via the cancel operation (983) → cancel the job; **(c) complete**
  (complete Interview 642) → cancel; **(d) no-show** (mark No Show 686) → cancel; **(e) expiry**
  (expire Stale Negotiations 733, the recurring task) → cancel any job for an expired interview;
  **(f) set Meeting Link** (626) — adding a link to a >24h SCHEDULED interview registers (or re-registers) the job;
  clearing it cancels. Cancellation is **idempotent** (cancelling a non-existent job is a no-op) so any hook firing
  twice or for a ≤24h interview that never registered is safe.
- **Owner (specialist):** software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: Reschedule cancels the old job and registers a new one at the new time.
- Given a SCHEDULED, linked interview >24h out with a registered interview-reminder job.
- When an admin reschedules it to a different scheduled At still >24h out.
- Then the current generation advances and durable reconciliation registers its distinct task at new Scheduled At - 24h.
- And an already-picked old task may remain until acknowledgement but its generation guard emits zero intents.
- Scenario: Cancel / complete / no-show / expiry each remove the reminder job (guard, parametrised).
- Given a SCHEDULED, linked interview >24h out with a registered interview-reminder job.
- When the interview transitions via cancel | complete | no-show | expiry.
- Then no current eligible reminder remains for that interview Id.
- Scenario: Cancelling a job that was never registered is a no-op (idempotency guard).
- Given a  is at most 24h interview that never registered a reminder job.
- When any lifecycle hook attempts to cancel its reminder job.
- Then no error is raised and no row is affected.


- **Required evidence:** per-hook IT/psql proof — for each of reschedule / cancel (×3 entrypoints) / complete /
  no-show / expiry / set Meeting Link using add & clear: the scheduled tasks row before/after; the idempotent-cancel
  no-op proof. A traceability note mapping each hook (file:line) → its cancel/reschedule call.
- **Dependencies:** REQ-003, REQ-004

## REQ-006 — Migration 0063 drops reminder sent + idx interview reminder due (supersedes spec-006 mechanism)

- **Type:** data
- **Priority:** Must
- **Source:** Plan §A/§C. Supersedes spec-006's REQ-003/REQ-004 mechanism.
- **Rationale / value:** The per-instance one-time model needs **neither** the reminder sent flag **nor** the partial
  index that spec-006's rolling sweep relied on (the current dedup state is the generation/recipient reminder-intent key in REQ-003, not the scheduler row alone). Migration **0063** drops both; its the reviewed database rollback re-adds them. This is safe — spec-006 is on sp-dev, never
  in prod. The entity field (Interview reminder Sent, Interview implementation reference) and the repo query/mark
  (find Due Reminder Candidates/mark Reminder Sent, Interview Repository implementation reference) are removed with it.
- **Owner (specialist):** dba · software-engineer (entity/repo removal)
- **Acceptance criteria (Gherkin):**


- Scenario: Migration 0063 drops the reminder sent column and its partial index.
- Given the Liquibase changelog is applied (mvn verify / Testcontainers).
- When the interviews table is inspected.
- Then no column reminder sent exists.
- And no index idx interview reminder due exists.
- And databasechangelog records changeset 0063.
- Scenario: 0063 is reversible (guard).
- Given changeset 0063 has applied.
- When the reviewed database rollback is applied.
- Then reminder sent (boolean required DEFAULT false) and idx interview reminder due are re-created.
- Scenario: The entity and repo no longer reference reminder sent (guard).
- Given the build compiles.
- Then Interview has no reminder Sent field.
- And Interview Repository has no find Due Reminder Candidates / mark Reminder Sent methods.
- And ddl-auto is validate boots clean (entity matches the post-0063 schema).


- **Required evidence:** \d interviews showing no reminder sent / no idx interview reminder due;
  confirm the migration history includes the required 0063 entries; a grep proving the entity/repo references are gone; a clean
  ddl-auto is validate boot; the changeset id 0063-drop-interview-reminder-sent implementation reference.
- **Dependencies:** REQ-003 (the one-time model must be live before the flag is dropped)

## REQ-007 — Interview expiry + showcase reroll become RECURRING registered tasks (logic unchanged)

- **Type:** functional
- **Priority:** Must
- **Source:** Plan §D.
- **Rationale / value:** Expiry and showcase reroll are periodic scans with no per-instance trigger → model each as a
  **recurring** db-scheduler Task (e g. Schedules fixed Delay) whose handler calls the **existing** logic. The
  existing method bodies are **moved, not rewritten** — extract them into handler-callable methods, preserving the
  status-guard idempotency and the business metric counters (interview expired / …errors; showcase per-pool
  isolation). The intervals stay config-driven (interview expiry.sweep-ms, showcase reroll.ms). The old
  a framework-managed scheduled method + @Scheduler Lock methods are **deleted in the same change** (single-ownership, REQ-009).
- **Owner (specialist):** software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: Interview expiry runs as a recurring db-scheduler task, behaviour unchanged.
- Given the recurring interview-expiry task is registered with fixed Delay = interview expiry sweep-ms.
- When a >72h-stale PROPOSED/COUNTER PROPOSED interview exists and the task runs.
- Then the interview is flipped to EXPIRED, open slots withdrawn, the interview expired counter increments,.
- And re-running the task is a no-op (status-guarded idempotency preserved).
- Scenario: Showcase reroll runs as a recurring db-scheduler task, behaviour unchanged.
- Given the recurring showcase-reroll task is registered with fixed Delay = showcase reroll ms.
- When the task runs.
- Then both showcase pools are rerolled with the existing per-pool isolation (a failing pool never starves the other).
- Scenario: Each migrated job has a single scheduling owner; its older scheduled entry point is removed.
- Given the build compiles.
- Then interview expiry and showcase refresh are no longer invoked by the older framework scheduling and lock annotations.


- **Required evidence:** IT log + psql: a stale interview expired by the recurring task (counter + notifications);
  a showcase reroll tick; scheduled tasks rows for both recurring tasks present and advancing execution time;
  a grep proving no a framework-managed scheduled method remains on the two migrated methods; existing expiry/showcase unit tests still green.
- **Dependencies:** REQ-001, REQ-002, REQ-009

## REQ-008 — Observability + bounded retry/give-up are IN-SCOPE (metrics shim, failure alert, give-up policy)

- **Type:** non-functional / operability
- **Priority:** Must
- **Source:** Plan §E; architecture pre-vet condition 4 (SRE/Obs blocking — not deferred).
- **Rationale / value:** db-scheduler emits **no Micrometer metrics by default**. Operating a scheduler blind is
  unacceptable: a silently-dead reminder job is invisible until users complain. This spec **must** ship: (a) a
  Scheduler Listener/metrics shim emitting **per-task success / failure / execution-latency** and a **due/overdue**
  gauge into Micrometer (alongside the preserved business counters interview expired*, whatsapp interview reminder*);
  (b) metrics and structured failure/give-up logs now, with a **failure alert required before WhatsApp enablement** (founder R1–R4 defer alert activation while OFF); (c) a **bounded retry / give-up
  (failure-handler) policy** so a poisoned job retries a finite number of times then gives up (does not retry forever)
  and is surfaced (counter + structured log; alert before enablement), not silently looping.
- **Owner (specialist):** devops-engineer (alert + give-up policy) · software-engineer (metrics shim)
- **Acceptance criteria (Gherkin):**


- Scenario: A scheduled-job execution emits success/latency metrics.
- Given the Scheduler Listener metrics shim is wired.
- When a registered task executes successfully.
- Then a per-task success counter increments and an execution-latency timer records the duration (visible in.
- Scenario: A failing scheduled job emits a failure metric and is alert-able.
- Given a handler that throws.
- When the task executes.
- Then a per-task failure counter increments.
- And failure/give-up metrics and structured logs are observable while WhatsApp is OFF.
- And a sustained-failure alert is configured and verified before WhatsApp reminder delivery is enabled.
- Scenario: A poisoned job is bounded and gives up (no infinite retry).
- Given a handler that always throws and a configured failure-handler with a max-retry/give-up policy.
- When the task fails repeatedly up to the limit.
- Then db-scheduler stops retrying it (gives up) per the policy.
- And the give-up is observable (failure counter / log / alert), not a silent infinite loop.


- **Required evidence:** the metrics-shim source + a unit/IT asserting success/failure counters & latency timer are
  emitted; the registered Micrometer metric names; the recorded alert deferral while OFF and a tested alert rule before enablement; the **failure-handler / give-up policy** config + a test proving retries are bounded and the
  give-up is recorded.
- **Dependencies:** REQ-001, REQ-002

## REQ-009 — Shed Lock fully removed, with single-ownership during migration (no task runs under both mechanisms)

- **Type:** functional / cleanup
- **Priority:** Must
- **Source:** Plan §E; architecture pre-vet condition (don't drop Shed Lock until all three are verified migrated).
- **Rationale / value:** Once all three tasks run on db-scheduler (which has its own database-row leader election), Shed Lock is
  dead weight. Remove Shed Lock Config implementation reference, every @Scheduler Lock annotation, and the shedlock-spring +
  shedlock-provider-redis-spring pom implementation reference deps. **Critical invariant (single-ownership):** during migration a task
  must **never** exist as BOTH a framework-managed scheduled method+Shed Lock method AND a db-scheduler task — that would double-fire (duplicate
  interview expired increments, duplicate notifications, racing rerolls). Each migrated task therefore **deletes the
  old method in the same change** as it registers the db-scheduler task. @Enable Scheduling **stays** (the
  outbox/stream pollers still use a framework-managed scheduled method).
- **Owner (specialist):** software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: Shed Lock is fully removed from the codebase and build.
- Given the migration is complete.
- Then Shed Lock Config implementation reference is deleted.
- And no @Scheduler Lock annotation remains anywhere.
- And pom implementation reference declares no shedlock-* dependency.
- And the app still boots (no missing-bean error).
- Scenario: @Enable Scheduling stays for the continuous pollers (guard).
- Given Shed Lock is removed.
- Then Intuition Application still carries @Enable Scheduling.
- And the delivery pollers continue using their existing framework-managed schedule; those pollers are outside this migration.
- Scenario: No task runs under both mechanisms (single-ownership guard).
- Given any migrated task.
- Then each job runs through either the database scheduler or its retained poller schedule, never both.
- And a single execution per due interval is observed (no double-fire: no duplicate interview expired increments,.


- **Required evidence:** a grep proving Shed Lock Config deleted, zero @Scheduler Lock, zero shedlock-* in pom implementation reference;
  Intuition Application still @Enable Scheduling; a live-stack proof of **no double-fire** for expiry/showcase
  (single counter increment / single reroll per interval) across the migration window.
- **Dependencies:** REQ-003, REQ-007 (all three migrated first)

## REQ-010 — User-facing reminder behaviour is identical; the migration is back-compatible (no observable change while WhatsApp OFF)

- **Type:** constraint
- **Priority:** Must
- **Source:** Plan §C/§supersession; project posture (WhatsApp ships OFF).
- **Rationale / value:** This is an internal mechanism swap. The **user-visible** outcome must be unchanged: a reminder
  fires ~24h before a SCHEDULED, linked interview, to both parties, with the format sgt operation; a
  booking ≤24h out gets none (Option A); a no-link booking gets none. The WhatsApp template (INTERVIEW REMINDER),
  copy, Singapore time formatting, opt-in/phone-verified gating, and dormant-while-OFF behaviour are **all unchanged** (the handler
  enqueues to the same outbox, dropped at dispatch while WhatsApp is OFF). No frontend change.
- **Owner (specialist):** software-engineer (backend) · qa-engineer (back-compat)
- **Acceptance criteria (Gherkin):**


- Scenario: The reminder content and timing match spec-006 exactly (back-compat).
- Given a SCHEDULED, linked interview >24h out under the new one-time model.
- When the reminder fires (WhatsApp ON in the test).
- Then both parties receive INTERVIEW REMINDER with body Params=[format Sgt using scheduled At, meeting Link],.
- Scenario: With WhatsApp OFF the migration produces no observable change (dormancy guard).
- Given whatsapp enabled is no (every environment).
- When a reminder job fires.
- Then the enqueued outbox rows are dropped at dispatch (no message), exactly as before.
- And no in-app notification behaviour changes.


- **Required evidence:** a behaviour-equivalence note + test proving identical params/timing/gating vs spec-006; a
  dormancy proof (job fires → outbox rows dropped at dispatch, WhatsApp OFF); confirmation of required spec006 consent-copy/test propagation before enablement.
- **Dependencies:** REQ-003, REQ-005

---

## Traceability summary

| REQ | One-line | Type | Priority | Owner |
|---|---|---|---|---|
| REQ-001 | db-scheduler dependency + scheduled tasks table (migration 0062, Liquibase database structure definitions, db initializer is no) | infra/data | Must | dba · software-engineer |
| REQ-002 | Generic registration surface — a job = one Task a registered application component + handler (worked example; thin Scheduled Job Service) | functional | Must | solution-architect · software-engineer · Tech Writer |
| REQ-003 | Interview 24h reminder = per-instance one-time job, instance Id is interview Id:generation, NO serialized personal information; >24h+link registers, ≤24h/no-link → none | functional | Must | software-engineer · solution-architect |
| REQ-004 | Reminder handler re-guard MANDATORY + tested (cancelled/rescheduled → 0 sends) | functional/safety | Must | software-engineer · qa-engineer |
| REQ-005 | Every lifecycle hook cancels/reschedules the job (schedule/reschedule/cancel×3/complete/no-show/expiry/set Meeting Link), idempotent cancel | functional | Must | software-engineer |
| REQ-006 | Migration 0063 drops reminder sent + idx interview reminder due + entity/repo refs (supersedes 006) | data | Must | dba · software-engineer |
| REQ-007 | Interview expiry + showcase reroll → recurring registered tasks, logic moved-not-rewritten | functional | Must | software-engineer |
| REQ-008 | Observability shim (success/failure/latency + due gauge) + failure alert + bounded retry/give-up | non-functional | Must | devops-engineer · software-engineer |
| REQ-009 | Shed Lock fully removed, single-ownership (no double-fire), @Enable Scheduling stays | functional | Must | software-engineer |
| REQ-010 | User-facing reminder behaviour identical + back-compat/dormant while WhatsApp OFF | constraint | Must | software-engineer · qa-engineer |

## Resolved founder choices (folded 2026-09-05)

The original Q1–Q4 are settled by spec md v1.0: **10s polling / five threads; four retries with exponential backoff then give-up metric; metrics-only while OFF, failure alert deferred until enablement; no startup/cutover backfill**. Do not reopen the discarded 3–5-attempt or backfill alternatives. Durable reconciliation of new lifecycle events is required for correctness and is distinct from cutover backfill.

## 2026-09-05 lifecycle, rollout and observability clarification (REQ-005/008/010)

Every schedule/reschedule, link add/change/clear, cancellation entry point, completion, no-show and expiry advances or preserves generation as REQ-003 specifies and updates a durable reconciliation intent in the same business transaction. An idempotent reconciler ensures the current eligible generation has its task and cancels obsolete unpicked tasks; catch-and-ignore of Task Instance Currently Executing Exception is not sufficient. Recovery after a commit/registration crash must reconcile from durable intent, not lose the new job. This reconciles new lifecycle changes; it does not backfill pre-cutover interviews.

Founder R1–R4 are settled: no startup/cutover backfill; polling 10s/five threads; bounded failure handling with four retries after the initial attempt (five attempts total), give-up counter plus structured log; metrics now, Prometheus alert activation deferred while WhatsApp is OFF and required before enabling reminder delivery. Keep the success/failure/latency/due metrics and all existing notification eligibility checks. REQ-010 also carries spec 006's frontend consent-copy/test prerequisite. Before enablement, verify durable dedup, stale-generation races, reconciliation crash recovery, consent copy/event parity and alerting. No enablement or code verification occurs in this docs correction.


## 2026-09-05 reconciler race and rollback acceptance (REQ-003/005/010)

A reconciler snapshot may cancel only task generations strictly older than that snapshot; it must never cancel a newer task or acknowledge a newer generation's intent. If G1 resumes after G2 has already registered and reconciled, G2 must remain present; G1 may clean only its own exact stale key/older keys. Re-check generation before acknowledging and test this explicit interleaving independently of handler-versus-reschedule races.

Rollout/rollback must retain generation/due state, dedup keys, pending outbox and reconciliation rows, with WhatsApp OFF until a compatible corrected binary resumes. Do not apply the historical scheduler-table-drop rollback or run an older generation-unaware handler against new keys. Prove compatible binary rollback/resume with pending work and no duplicate enqueue before enablement; see design md's current rollout/rollback section.
