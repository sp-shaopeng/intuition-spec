# Tasks — Generic scheduler platform (db-scheduler) + migrate existing tasks (007)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Implementation tasks broken from [design md](design.md). Each task is small (≈2–5 min of focused work for an
> "enthusiastic junior engineer"), names the **owning specialist**, traces to one or more **REQ-NNN**, lists the
> **exact files** to touch, and a **per-task verification step**. The Boss **dispatches the owning specialist** for
> each — the orchestrator does not build these itself. Check a box only when its evidence is captured under
> the private evidence directory for the requirement. This is a **backend-only** change (no frontend lane). Builds within the backend lane run sequentially;
> single-ownership (REQ-009) means each migrated task's old a framework-managed scheduled method method is deleted in the same change that
> registers its db-scheduler task — never leave both live.

## Current-work boundary (2026-09-05)

The June task tables below are historical implementation records. Their row-gone/once-only claim, interview Id-only identity, status/link-only guard, startup-backfill alternative and unconditional-alert language are superseded by the current design. Do not mark the new safeguards complete using those old checks. Current unverified work is listed at the end.

## Governance gate (FIRST — precedes all build)

| # | Task | Traces to | Owner | Verify | Status |
|---|------|-----------|-------|--------|--------|
| 1 | Scoped /team-review on this DRAFT spec (solution-architect, software-engineer, dba, devops-engineer, qa-engineer, security-expert, Technical Writer) + Boss audit → all Review & Sign-Off boxes checked → status: APPROVED, version 0.1→1.0. The architecture pre-vet was NOT this review (cond. 5). **No build task below starts before this.** | all | Boss | sign-off boxes in spec md checked; status flipped to APPROVED before build | [ ] |

## Dependency + table lane (dba · software-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 2 | Add com github.kagkarlsson:db-scheduler-spring-boot-starter (SB-3.3-compatible 16.x) to pom implementation reference | REQ-001 | software-engineer | pom implementation reference | dependency resolves; app context loads the Scheduler bean | [ ] |
| 3 | Configure db-scheduler in application implementation reference: db-scheduler enabled is yes, db-scheduler db.initializer is no, polling-interval, threads, table-name is scheduled tasks (Open Q1 defaults). Remove interview reminder.sweep-ms; keep interview expiry.sweep-ms + showcase reroll.ms | REQ-001, REQ-007 | software-engineer | application implementation reference | props resolve on boot; library applies no database structure definitions | [ ] |
| 4 | Migration 0062-db-scheduler-tasks implementation reference: Liquibase-wrapped db-scheduler PG scheduled tasks database structure definitions (PK task name, task instance, execution time index) + the reviewed database rollback drop; register in master changelog (confirm latest is 0061, no collision) | REQ-001 | dba | 0062-db-scheduler-tasks implementation reference, db changelog-master implementation reference | the documented operational check applies it; \d scheduled tasks; databasechangelog has 0062; ddl-auto is validate clean | [ ] |

## Registration surface lane (solution-architect · software-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 5 | Create the scheduler package: Scheduled Tasks task-name constants (no magic strings) + a thin Scheduled Job Service over db-scheduler Scheduler Client exposing schedule Once / reschedule / cancel (idempotent cancel) | REQ-002 | software-engineer · solution-architect | new Scheduled Tasks implementation reference, Scheduled Job Service implementation reference | unit: schedule Once/cancel hit the client; cancel-absent is a no-op | [ ] |
| 6 | Declare the three Task a registered application components: One Time Task interview-reminder (instance Id is interview Id, Void data), Recurring Task interview-expiry (the configured interview-expiry sweep interval), Recurring Task showcase-reroll (the configured showcase refresh interval), each delegating to a handler | REQ-002, REQ-003, REQ-007 | software-engineer | new Scheduled Jobs implementation reference (or config/) | context IT: the three task names are registered | [ ] |

## Interview reminder one-time lane (software-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 7 | Reminder **handler**: load interview by id; **RE-GUARD (mandatory)** still SCHEDULED + non-blank meeting link else return (0 sends); build the format sgt operation; enqueue INTERVIEW REMINDER to both parties via Outbox Service (reuse). Metric whatsapp interview reminder sent/…errors | REQ-003, REQ-004 | software-engineer | Interview Service implementation reference (handler method), Scheduled Jobs implementation reference | unit: cancelled→0, link-null→0, valid→2 | [ ] |
| 8 | In schedule Interview (490): the cancel operation then, if has Link and scheduled At now+24h, the schedule once operation; else register nothing (Option A ≤24h / no-link). Remove the old reminder sent Option-A flag logic (526–537) | REQ-003, REQ-005 | software-engineer | Interview Service implementation reference | IT: >24h+link→1 row at −24h; ≤24h→0; no-link→0; reschedule moves the row | [ ] |
| 9 | Wire **cancel** of the reminder job into every remaining lifecycle hook: the cancel operation (983, covers admin/tutee/tutor cancel), complete Interview (642), mark No Show (686), the recurring expiry handler (per expired id), set Meeting Link (626, add→register / clear→cancel) | REQ-005 | software-engineer | Interview Service implementation reference | per-hook IT: each transition leaves 0 reminder rows; idempotent cancel no-op | [ ] |
| 10 | **Delete** the old the old framework-managed interview-reminder method (803–867) + enqueue Reminder helper; remove REMINDER WINDOW/METRIC REMINDER SENT usages no longer needed by the sweep (keep REMINDER WINDOW if reused by the ≤24h decision) | REQ-003, REQ-009 | software-engineer | Interview Service implementation reference | grep: no send Due Interview Reminders; compiles | [ ] |

## Recurring expiry + showcase lane (software-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 11 | Extract the expire Stale Negotiations body (733–787) into a handler-callable the run expiry sweep operation (status-guard idempotency + interview expired/…errors counters preserved); delete the [REDACTED PASSWORD] Lock annotations; the recurring task bean (Task 6) calls it; cancel any reminder job for each expired interview | REQ-007, REQ-005, REQ-009 | software-engineer | Interview Service implementation reference | IT: recurring task expires a stale interview (counter+notifications, idempotent re-run); grep: no a framework-managed scheduled method on it | [ ] |
| 12 | Point the showcase recurring task at the refresh operation (237, already self-invocation-safe); delete [REDACTED PASSWORD] Lock on scheduled Refresh (216–223) | REQ-007, REQ-009 | software-engineer | Showcase Service implementation reference | IT: reroll tick via the recurring task; grep: no a framework-managed scheduled method on it | [ ] |

## Schema cleanup lane (dba · software-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 13 | Migration 0063-drop-interview-reminder-sent implementation reference: drop interviews reminder sent + idx interview reminder due + the reviewed database rollback re-adding both; register in master changelog after 0062 | REQ-006 | dba | 0063-drop-interview-reminder-sent implementation reference, db changelog-master implementation reference | the documented operational check applies; \d interviews shows neither; databasechangelog has 0063; the reviewed database rollback re-adds | [ ] |
| 14 | Remove the reminder Sent field (Interview implementation reference) + find Due Reminder Candidates/mark Reminder Sent (Interview Repository implementation reference); ddl-auto is validate matches post-0063 schema | REQ-006 | software-engineer · dba | Interview implementation reference, Interview Repository implementation reference | compiles; grep: no reminder Sent; clean validate boot | [ ] |

## Shed Lock removal lane (software-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 15 | Delete Shed Lock Config implementation reference; remove every @Scheduler Lock annotation; remove shedlock-spring + shedlock-provider-redis-spring from pom implementation reference (139–146). **Keep @Enable Scheduling** (Intuition Application implementation reference). Confirm no task is both a framework-managed scheduled method and a db-scheduler task | REQ-009 | software-engineer | Shed Lock Config implementation reference (delete), service/*, pom implementation reference | grep: 0 @Scheduler Lock, 0 shedlock-*, Shed Lock Config gone; boots; live no-double-fire | [ ] |

## Observability lane (devops-engineer · software-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 16 | Scheduler Listener metrics shim: per-task success/failure counters + execution-latency timer + due/overdue gauge into Micrometer (business counters preserved) | REQ-008 | software-engineer | new Scheduler Metrics Listener implementation reference, registration in Scheduled Jobs implementation reference | unit/IT: counters + timer emitted (registry assert) | [ ] |
| 17 | Bounded retry / give-up **failure-handler** on the tasks (one-time: backoff→give-up per Open Q3; recurring: log-and-skip-to-next-tick); give-up observable (counter/log/alert) | REQ-008 | devops-engineer · software-engineer | Scheduled Jobs implementation reference | test: retries bounded, give-up recorded (no infinite loop) | [ ] |
| 18 | **Failure alert rule** (Prometheus rule per Open Q2) on sustained per-task failures; commit the rule | REQ-008 | devops-engineer | ops/observability config (e g. intuition-spec/specs/ops/... or alerting config) | alert rule committed; fires on the failure counter | [ ] |

## Tests + evidence (qa-engineer · software-engineer · Technical Writer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 19 | Re-guard tests (NAMED, BLOCKING): handler on CANCELLED→0, SCHEDULED-but-link-null→0, valid→2 | REQ-004 | qa-engineer · software-engineer | *Test implementation reference | the three re-guard cases pass | [ ] |
| 20 | One-time scheduling ITs: >24h+link→1 row at −24h (empty task data); ≤24h→0; no-link→0; reschedule→row moves; cancel/complete/no-show/expiry→0; set Meeting Link add/clear; idempotent-cancel no-op | REQ-003, REQ-005 | qa-engineer · software-engineer | Interview Reminder Job IT implementation reference | each scenario green; psql row asserts | [ ] |
| 21 | Recurring ITs: expiry expires a stale interview (counter+notifications, idempotent re-run); showcase reroll tick; both recurring scheduled tasks rows advance | REQ-007 | qa-engineer · software-engineer | *IT implementation reference | recurring behaviour unchanged | [ ] |
| 22 | Migration ITs: 0062 creates scheduled tasks (+rollback); 0063 drops reminder sent/index (+rollback); databasechangelog has both; ddl-auto is validate clean | REQ-001, REQ-006 | qa-engineer · dba | *IT implementation reference | both migrations applied + reversible | [ ] |
| 23 | Observability + single-ownership tests: metrics emitted; bounded-retry/give-up; grep no @Scheduler Lock/shedlock-*; live no-double-fire (single counter increment / single reroll per interval) | REQ-008, REQ-009 | qa-engineer · software-engineer · devops-engineer | *Test implementation reference + live psql | metrics asserted; no double-fire proven | [ ] |
| 24 | "Register a scheduled job" **reference doc** (the generic surface + the worked one-time AND recurring example; the Scheduled Job Service application programming interface; the no-personal information-in-scheduled tasks contract) | REQ-002 | Technical Writer · software-engineer | intuition-spec/specs/reference/ (or specs/007-.../) doc | doc committed; worked example matches code | [ ] |
| 25 | Capture evidence under the private evidence directory for the requirement: psql/IT proofs per REQ — scheduled tasks row lifecycle (REQ-003/005), re-guard 0-sends (REQ-004), 0062/0063 migrations (REQ-001/006), recurring tasks running (REQ-007), metrics+alert+give-up (REQ-008), Shed Lock-gone + no-double-fire (REQ-009), behaviour-equivalence + dormancy (REQ-010). WhatsApp stays OFF — no live send | REQ-001…REQ-010 | qa-engineer · software-engineer | the private evidence directory for the requirement | reviewer opens each artifact against the running stack | [ ] |
| 26 | Gate green + local deploy + push: JAVA HOME=…21 run the corresponding backend quality check; redeploy dev stack health-green; 0062+0063 applied; **live convergence proofs** (design "Testing"); then commit + push on the repo's current branch | all | software-engineer · devops-engineer | backend repo | BUILD SUCCESS ≥97.5/97.5; health green; convergence proven; pushed | [ ] |

## Deliverable governance gate (LAST)

| # | Task | Traces to | Owner | Verify | Status |
|---|------|-----------|-------|--------|--------|
| 27 | After build: /team-review (full panel) → Boss verdict on the built deliverable + evidence (Boss opens the one-time job fire + Shed Lock removal live before SHIP); fix SEND-BACK conditions in place + re-verify; on SHIP set spec md status: HISTORICALLY MARKED IMPLEMENTED, confirm version/changelog, and **mark spec-006 SUPERSEDED-in-part** (its reminder mechanism) in 006's spec md + requirements traceability matrix | all | Boss | Boss SHIP; status HISTORICALLY MARKED IMPLEMENTED; spec-006 marked; evidence opened | [ ] |

## Notes

- **Spec-review gate first (Task 1):** the author does NOT self-approve — the Boss convenes the scoped team-review on
  this DRAFT and only flips status: APPROVED (and version 0.1→1.0) once every Review & Sign-Off box is checked. No
  build task (2+) starts before that. The architecture pre-vet (2026-06-21) was the pre-build vet, NOT this review.
- **Single-ownership during migration (REQ-009):** never leave a task as both a framework-managed scheduled method+Shed Lock AND a db-scheduler
  task — each migrated task deletes its old method in the same change (Tasks 8/10, 11, 12) before Shed Lock is removed
  (Task 15). Drop Shed Lock only after all three are verified migrated.
- A task is DONE only when its REQ-NNN evidence is captured under the private evidence directory for the requirement and opened by the reviewer.
- **No live WhatsApp send is in scope to evidence** — WhatsApp is toggled OFF; reminder evidence is scheduled tasks
  row lifecycle + outbox-row enqueue/drop + re-guard 0-sends (design "honest scope").
- Do D = verify gate green (the documented operational check, ≥97.5% line+branch) + local deploy health-green + live
  convergence + push (per project CLAUDE md). On completion update spec md status: HISTORICALLY MARKED IMPLEMENTED + confirm version/changelog.

## 2026-09-05 correction follow-up — implementation not verified

- [ ] **REQ-003/005 — dba + software-engineer:** additive new Liquibase allocation for monotonic generation, nullable reminder due at fixed at the lifecycle mutation, durable reconciliation intent and per-recipient reminder-intent unique key; no personal information and no rewrite of0062/0063.
- [ ] **REQ-003/004 — software-engineer:** row-lock generation/status/link/due-window check plus unique-key/outbox enqueue in one transaction. Verify repeat/concurrent execution and crash after commit before scheduler acknowledgement, including rollback and per-recipient behavior.
- [ ] **REQ-005 — software-engineer + quality assurance:** wire every time/link/terminal hook to generation/reconciliation; idempotent distinct-generation registration, restart recovery (including crossing the due time) and picked-old-job race tests. No startup/cutover backfill.
- [ ] **REQ-008/010 — SRE + quality assurance:** four retries/five attempts, metrics and give-up; keep OFF. Before enablement verify failure alerts, scheduler safeguards and spec006 consent-copy/event parity. Produce fresh runtime evidence; this docs correction supplies none.

- [ ] **REQ-005 — reconciler race:** pause G1 after snapshot, finish G2 registration/reconciliation, then resume G1; assert G2 remains scheduled, its intent is not acknowledged by G1, and G1 cleanup cannot touch newer/equal keys.
- [ ] **REQ-001/003/005/010 — rollback:** document and verify a schema/task-format-compatible binary rollback/resume with WhatsApp OFF; retain generation/due, dedup, outbox and pending reconciliation state. No applied0062/0063 rewrite or required-relation drop; incompatible reminder handlers remain paused until a compatible corrected binary resumes.
