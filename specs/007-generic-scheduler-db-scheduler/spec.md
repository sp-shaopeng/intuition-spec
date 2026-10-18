> **Current-file check, 18 October 2026:** The scheduler is present, but durable duplicate prevention, schedule-version checks and repair after a conflicting reschedule remain incomplete (G09). See the [alignment review](../../docs/current-file-alignment.md). The requirements remain unchanged.

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 007
slug: generic-scheduler-db-scheduler
title: Generic scheduler platform (adopt db-scheduler) + migrate existing scheduled tasks
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED            # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: solution-architect    # platform/architecture change — co-authored with requirements-engineer
created: 2026-06-21
updated: 2026-09-05
supersedes-in-part: 006   # supersedes 006's reminder *mechanism* (sweep + reminder sent / migration 0061), identical user-facing behaviour
---

> **2026-09-05 requirements correction (documentation only).** The once-only promise now means durable deduplicated outbox intent, with schedule generations, row-lock guards and durable registration reconciliation required before enablement. Founder no-backfill/metrics-first choices are folded; WhatsApp stays OFF.
> The status and shipment/evidence entries below record the earlier implementation. This revision does not
> certify that the corrected contract is implemented, tested, or deployed; follow-up verification remains open.


# Generic scheduler platform (adopt db-scheduler) + migrate existing scheduled tasks

Scheduled work today is **coupled to business services**: each time-based task is a bespoke a framework-managed scheduled method +
@Scheduler Lock method living inside a service, with a hand-picked Redis Shed Lock name. There are exactly three such
tasks — Interview Service expire Stale Negotiations (Interview Service implementation reference),
Interview Service send Due Interview Reminders (803–867, the spec-006 24h reminder), and
Showcase Service scheduled Refresh (Showcase Service implementation reference). Adding a new time-based use case means a new
hand-written sweep, a new Shed Lock name, and more "when to run" logic mixed into domain code — it doesn't scale.

This spec adopts **db-scheduler** (com github.kagkarlsson:db-scheduler-spring-boot-starter) as a **generic,
persistent, cluster-safe** scheduler so a future time-based use case just **registers a Task a registered application component + a handler** —
no service-embedded a framework-managed scheduled method. It then **migrates all three existing tasks onto db-scheduler** and **removes Shed Lock
entirely** (a task must never run under both mechanisms). The continuous 2-second pollers
(Outbox Poller poll, Email Consumer Service/Whats App Consumer Service poll/reclaim Pending) are **out of scope** —
they have their own locking (with a record lock SKIP LOCKED / Redis-Streams PEL) and **keep a framework-managed scheduled method, so
@Enable Scheduling STAYS** (Intuition Application implementation reference).

The headline modelling decision (founder-bound, team-pre-vetted): the **interview 24h reminder becomes a per-instance
one-time job** — instance Id interview Id, **no serialized payload**, handler **re-guards** (still SCHEDULED + has
meeting link) before sending — registered only when a link is present AND scheduled At now + 24h; a booking ≤24h
out registers nothing (the schedule confirmation is the reminder). Interview **expiry** and **showcase reroll** stay
periodic and become **recurring** registered tasks. This **supersedes spec-006's reminder mechanism** (the rolling
reminder sent-keyed sweep + migration 0061 column/index) while preserving **byte-for-byte identical user-facing
behaviour** (24h-before, both parties, link-gated, ≤24h → none). User-facing reminder content is unchanged.

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC (each REQ-NNN)
- **Design:** [design md](design.md) — architecture, the registration surface, data model, observability, ADRs
- **Tasks:** [tasks md](tasks.md) — implementation tasks, each traced to a REQ-NNN + owning specialist
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario

## Status

DRAFT — authored by solution-architect + requirements-engineer from the approved, **team-pre-vetted** plan
(twinkly-percolating-glade md; architecture vet 2026-06-21, verdict ✅ sound). **The author does NOT self-approve.**
The plan's architecture pre-vet folded **five conditions** into this DRAFT — they are baked in as requirements / AC and
must be honoured at build:

1. **Interview reminder = per-instance one-time job; interview Id is the task INSTANCE id, NO serialized payload**
   (REQ-003). 2. **Handler re-guard MANDATORY + explicitly tested** — fire on a cancelled / rescheduled interview →
   **0 sends** (REQ-004). 3. **Enumerate EVERY interview lifecycle hook** that cancels/reschedules the job — schedule,
   reschedule, cancel, complete, no-show, expiry — miss none (REQ-005). 4. **Observability + bounded retry IN-SCOPE,
   not deferred** — metrics + failure alert + give-up policy (REQ-008). 5. **The formal spec-review on THIS DRAFT still
   gates build** — the architecture pre-vet was not a replacement (Task 1, governance).

The formal **scoped team-review on this DRAFT still runs** (Task 1) before any build — that review checks the five
boxes below and the Boss flips the status to APPROVED. **No build task starts until then.**

## Review & Sign-Off

- [x] Author / Counterpart — scheduler architecture + registration surface (solution-architect, lead) — SIGN-OFF / 2026-06-21
- [x] Counterpart — generic registry + per-instance/recurring wiring + Shed Lock removal (software-engineer) — SIGN-OFF / 2026-06-21
- [x] Counterpart — migrations 0062 (scheduled tasks) + 0063 (drop reminder sent), db initializer is no (dba) — SIGN-OFF (database structure definitions pinned to db-scheduler 16.x + boot-validate) / 2026-06-21
- [x] Counterpart — observability shim + failure alert + bounded-retry/give-up policy (devops-engineer) — SIGN-OFF (metrics-only per founder R4; alert rule deferred) / 2026-06-21
- [x] Counterpart — single-ownership, re-guard tests, live-convergence proofs (qa-engineer) — SIGN-OFF / 2026-06-21
- [x] Counterpart — new dependency / handler-data contract (no personal information in instance Id or task data) (security-expert) — SIGN-OFF / 2026-06-21
- [x] Technical Writer — "register a scheduled job" reference doc — SIGN-OFF / 2026-06-21
- [x] Boss (final sign-off → flips status to APPROVED) — APPROVED / 2026-06-21

## Changelog

- 1.2 (2026-06-21) post-ship **detailed team + Boss review** (founder-requested: cleanliness/missing-value failure/idempotency/back-compat).
  3 lanes — quality assurance: SHIP (fire-once is atomic via a database transaction+MANDATORY enqueue; every SCHEDULED-exit hook wired);
  DBA+frontend-facing service: PASS (0063 drop safe+reversible, no lingering reminder sent/sweep refs, no application programming interface/wire change, conflict-free
  deps); SW-Eng/missing-value failure: minor-fixes (no missing-value failure). **Fixes applied (2 passes, gate re-green 2004 tests ≥97.5%):** (a) catch
  Task Instance Currently Executing Exception in Scheduled Job Service cancel/schedule Once — closes a real 500-race
  (cancel/complete hook firing during the reminder's execution rolled back the status change); (b) restored
  whatsapp interview reminder errors + new …gave up counter (observability regression); (c) atomic
  schedule If Not Exists replaces the get-then-act TOCTOU; (d) failure-handler give-up now behaviour-tested (guards the
  4-retry→remove invariant); (e) null-scheduled At re-guard; (f) [L] dead the reschedule operation removed, @Slf4j,
  set Meeting Link trim, Javadoc tidy. **Tracked condition (NOT fixed — gated behind WhatsApp OFF):** a crash in the
  sub-second window between the handler's outbox-commit and db-scheduler's row-delete can re-fire the one-time job →
  duplicate reminder rows (the row-delete is not a send-consumed durable marker). **Must add a delivery-done dedup key
  before WhatsApp enablement (PE phase).** version 1.1→1.2 (status stays HISTORICALLY MARKED IMPLEMENTED).
- 1.1 (2026-06-21) HISTORICALLY MARKED IMPLEMENTED — built (db-scheduler 16.12.0; scheduler/Scheduled Jobs registry + Scheduled Job Service
  + Scheduler Listener; reminder→one-time job w/ instance Id+re-guard+all lifecycle hooks; expiry+showcase→recurring;
  Shed Lock Config deleted + shedlock deps removed; migrations 0062/0063). Gate green: **1999 tests, LINE 0.988 /
  BRANCH 0.975** (Boss-reverified). **0062 database structure definitions verified column-for-column vs the decompiled db-scheduler 16.12.0 schema
  AND boot-validated.** **Live convergence (deploy local) — all REQs PASS:** one-time reminder fires once → 2 rows to
  both parties Singapore time datetime, link (proven by injection AND by the real admin-schedule application programming interface in the passing
  interview-lifecycle e2e: schedule Once task is interview-reminder run At is scheduled At−24h); re-guard on CANCELLED → 0
  sends; 0063 dropped reminder sent; recurring interview-expiry+showcase-reroll scheduled; Shed Lock gone from the
  runtime jar (0 classes); scheduler metrics shim registered. **e2e 109/109 green on a clean stack** (first run's 4
  failures were environmental deterministic-handle data-collisions — REGISTRATION FAILED/PUBLIC HANDLE TAKEN —
  reproduced as local-database-state and cleared by a reseed; not spec-007). Founder rulings R1–R4 honored.
  **Supersedes-in-part spec-006**'s reminder mechanism (sweep + reminder sent/0061) — identical user-facing behaviour.
  **Live convergence caught a Sev defect the green gate missed (3rd this session):** the as-built code FAILED first boot
  with a Spring **circular dependency** (interview Service → scheduled Job Service → Scheduler → interview Reminder Task(
  Interview Service) → interview Service— the Task @Beans took the services as eager ctor params while Interview Service
  depends on the Scheduler. Fixed by injecting the services **lazily via Object Provider**, resolved at fire time in the
  handler lambdas (Scheduled Jobs implementation reference + test); clean boot after, gate re-verified green. (Note: a blank
  meeting-link is rejected 400 by request validation, so the set Meeting Link-clear→cancel branch is test-covered only, not
  application programming interface-reachable — not a defect.)
  Delivery note: three build sub-agents hit stream-watchdog "stalls" while their edits kept landing via the workspace's
  iCloud/Dropbox sync lag; the Boss took over inline, re-verified the disk/build, and drove the convergence by hand.
  status APPROVED→HISTORICALLY MARKED IMPLEMENTED.
- 1.0 (2026-06-21) APPROVED — scoped formal spec-review passed + Boss sign-off. **Founder rulings on the 4 open
  questions:** (Q4 cut-over) **accept pre-cutover misses — NO startup backfill** (low-stakes, dormant while WhatsApp OFF;
  MODIFIED design "Migration/rollout" — drop the backfill); (Q3 retry) **4 retries (exp backoff) → give up + emit failure
  metric** (REQ-008); (Q1) db-scheduler **10s poll / 5 threads**; (Q2 alert) **metrics-only for now — DEFER the Prometheus
  alert rule** (counters/gauges emitted + dashboard-visible; MODIFIED REQ-008 — alert rule deferred until WhatsApp
  enablement). **Boss build condition:** the 0062 scheduled tasks database structure definitions MUST be pinned to db-scheduler 16.x's official PG
  schema and **boot-validated** (schema mismatch = boot failure — the #1 build risk). status DRAFT→APPROVED.
- 0.1 (2026-06-21) initial draft — ADDED REQ-001…REQ-010: db-scheduler dependency + scheduled tasks table
  (migration 0062, db initializer is no); the generic Task a registered application component registration surface (REQ-002, with a worked
  example); the interview 24h reminder as a per-instance one-time job keyed on interview Id with NO serialized payload
  (REQ-003) + a mandatory, tested handler re-guard (REQ-004) + every lifecycle hook cancelling/rescheduling the job
  (REQ-005); migration 0063 dropping reminder sent + idx interview reminder due (REQ-006, supersedes spec-006);
  interview expiry + showcase reroll as recurring registered tasks (REQ-007); observability (metrics shim + failure
  alert) + bounded-retry/give-up policy (REQ-008); Shed Lock fully removed with single-ownership during migration
  (REQ-009); user-facing reminder behaviour identical / back-compat (REQ-010). Records ADR-001 (adopt db-scheduler vs
  bespoke sweeps / hand-rolled table / Quartz) and ADR-002 (per-instance one-time vs recurring sweep for the reminder)
  in design md. **Supersedes-in-part spec-006**'s reminder *mechanism* (the Boss marks 006 SUPERSEDED-in-part at ship).
  solution-architect + requirements-engineer authored, grounded against the real code on 2026-06-21. status DRAFT
  pending the scoped team-review (Task 1).
