# Tasks — WhatsApp interview reminders (006)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Implementation tasks broken from [design md](design.md). Each task is small (≈2–5 min of focused work for an
> "enthusiastic junior engineer"), names the **owning specialist**, traces to one or more **REQ-NNN**, lists the
> **exact files** to touch, and a **per-task verification step**. The Boss **dispatches the owning specialist** for
> each — the orchestrator does not build these itself. Check a box only when its evidence is captured under
> the private evidence directory for the requirement. This is a backend pipeline change with a required frontend consent-copy/test lane before enablement. Builds within the backend lane run sequentially.

## Current delivery contract and historical mechanism boundary (2026-09-05)

Spec 007 is authoritative for one-time scheduling, short-notice confirmation-only behavior, durable deduplication and stale-generation guards. The sweep/Shed Lock/reminder sent schema and task recipes below are **historical implementation records, superseded by spec 007**; do not rebuild them or treat their unchecked boxes as current work. Existing pipeline, parameter ordering, template drafting and OFF/opt-in gates still apply. Required current follow-up is the consent-copy lane and spec 007 safeguards; no runtime changes are certified here.

## Schema lane (dba)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 1 | Create migration 0061-interview-reminder-sent implementation reference: include the field reminder sent boolean required DEFAULT no on interviews + partial index idx interview reminder due ON interviews scheduled at when status is SCHEDULED AND reminder sent is no + the reviewed database rollback; register in the master changelog (no number collision — confirm latest is 0060 at build) | REQ-004 | dba | 0061-interview-reminder-sent implementation reference, master changelog | the documented operational check applies it; \d interviews shows column + partial index; databasechangelog has 0061; pre-existing rows = false | [ ] |
| 2 | Add reminder Sent field to the entity (the column operation, default false) | REQ-004 | dba · software-engineer | Interview implementation reference | compiles; Java persistence mapping insert matches database default | [ ] |

## Pipeline lane (software-engineer · bff-specialist)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 3 | Add nullable an ordered list of message-body values to Whats App Notification Payload (keep recipient User Id/template Name non-null validation; add a recipient User Id, template Name convenience ctor delegating body Params is no value) | REQ-002, REQ-008 | software-engineer · bff-specialist | Whats App Notification Payload implementation reference | unit: JSON round-trips body Params in order; legacy 2-arg payload deserializes with body Params is no value | [ ] |
| 4 | Pass the body params operation through Outbox Poller dispatch Whats App to the queue (gating unchanged — toggle, event-set, opt-in, phone-verified still apply) | REQ-002, REQ-008 | software-engineer | Outbox Poller implementation reference | dispatch IT: params reach the queue; OFF/ineligible still dropped | [ ] |
| 5 | the enqueue notification sync operation — serialize ordered params into the Redis message (round-trip order) | REQ-002 | software-engineer | Whats App Queue Service implementation reference | unit: params serialize+deserialize in order | [ ] |
| 6 | Whats App Consumer Service — for a notification with non-empty params build a Meta body component (mirror otp Components' body half) and pass body Component; empty/null params → the send operation (today's path). Client UNCHANGED | REQ-002, REQ-008 | software-engineer | Whats App Consumer Service implementation reference | unit: params → body component (captured client send arg); null/empty → the of operation | [ ] |
| 7 | Add Whats App Template Keys INTERVIEW REMINDER intuition interview reminder + Whats App Notification Event INTERVIEW REMINDER (no magic strings) | REQ-006 | software-engineer · bff-specialist | Whats App Template Keys implementation reference, Whats App Notification Event implementation reference | unit: the from template name operation → INTERVIEW REMINDER | [ ] |
| 8 | Append INTERVIEW REMINDER to whatsapp notifications.events; add interview reminder.sweep-ms (default 900000) | REQ-006, REQ-003 | software-engineer | application implementation reference | props resolve on boot; the is event enabled operation true | [ ] |
| 9 | Add Shed Lock Config LOCK INTERVIEW REMINDER interview-reminder-sweep (mirror LOCK INTERVIEW EXPIRY) | REQ-003 | software-engineer | Shed Lock Config implementation reference | constant present; used by the sweep @Scheduler Lock | [ ] |
| 10 | Add a pure, unit-tested the format sgt operation util (Asia/Singapore, fixed pattern constant) | REQ-001, REQ-003, REQ-007 | software-engineer | new util (e g. Sgt Date Time Formatter implementation reference) | unit: known Coordinated Universal Time instant → expected Singapore time string | [ ] |

## Service lane (software-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 11 | In schedule Interview: gate the two WhatsApp enqueues on meeting Link present; build the format sgt operation. Leave the in-app notifications as-is | REQ-001 | software-engineer | Interview Service implementation reference | IT: with link → 2 param rows; without link → 0 WhatsApp rows, in-app still fired | [ ] |
| 12 | In schedule Interview: when scheduled At changes, set reminder Sent is no (re-arm); same-time re-save leaves it unchanged | REQ-005 | software-engineer | Interview Service implementation reference | IT: reschedule → false; same-time → unchanged | [ ] |
| 13 | Add repo query the find due reminder candidates operation (SCHEDULED ∧ link≠null ∧ reminder sent is no ∧ scheduled at ∈ (now, now+24h]) returning id+tutee+tutor+scheduled At+link, and a set-based the mark reminder sent operation (the in operation) | REQ-003 | software-engineer · dba | Interview Repository implementation reference | @Data Jpa Test: selects only in-window linked un-reminded rows; mark is flag-guarded | [ ] |
| 14 | New the historical interview-reminder sweep, defaulting to fifteen minutes + the scheduler lock operation the send due interview reminders operation — capture primitives BEFORE mutation, enqueue INTERVIEW REMINDER to both parties with params, then mark Reminder Sent; metric whatsapp interview reminder sent (+ …errors on failure, rethrow). Model on expire Stale Negotiations | REQ-003 | software-engineer | Interview Service implementation reference | sweep IT: first run → 2 rows + flag set; second run → 0 (idempotent) | [ ] |

## Tests + evidence (qa-engineer · software-engineer · Technical Writer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 15 | ITs: schedule WITH link → 2 param rows; schedule WITHOUT link → 0 WhatsApp rows (in-app fired); reschedule re-arms; same-time no re-arm | REQ-001, REQ-005, REQ-008 | qa-engineer · software-engineer | *IT implementation reference | the documented operational check green | [ ] |
| 16 | Sweep ITs: in-window linked un-reminded → reminded once + flag set; **second run → 0 (idempotent)**; out-of-window guard; no-link guard | REQ-003 | qa-engineer · software-engineer | Interview Reminder Sweep IT implementation reference | idempotent: re-run = 0 rows | [ ] |
| 17 | Unit: payload body Params round-trip; consumer body-component build from params; null/empty → the of operation; format Sgt; from Template Name → INTERVIEW REMINDER | REQ-002, REQ-006, REQ-007, REQ-008 | software-engineer | *Test implementation reference | branch-sensitive cases covered (link present/absent, params empty/present, scheduled At changed/same, in/out window) | [ ] |
| 18 | Capture evidence under the private evidence directory for the requirement: psql/IT proof of enqueue-with-params (REQ-001), 0-rows-without-link (REQ-001/008), sweep selection + idempotent re-run (REQ-003), \d interviews + databasechangelog (REQ-004), reschedule re-arm (REQ-005). WhatsApp stays OFF — no live send | REQ-001…REQ-006, REQ-008 | qa-engineer · software-engineer | files committed under the private evidence directory for the requirement | reviewer opens each artifact against the running stack | [ ] |
| 19 | Author the three Meta template definitions (exact bodies, params, categories) + the backend-api/runbook doc note; cross-check body Params order = placeholder order | REQ-007 | Technical Writer · software-engineer | intuition-spec/specs/.../ doc + backend-api/runbook | template doc committed; param-order cross-check note | [ ] |
| 20 | Gate green + local deploy + push: JAVA HOME=…21 run the corresponding backend quality check; redeploy dev stack health-green; migration 0061 applied; then commit + push on the repo's current branch | all | software-engineer · devops-engineer | backend repo | BUILD SUCCESS ≥97.5/97.5; health green; pushed | [ ] |

## Governance gate

| # | Task | Traces to | Owner | Verify | Status |
|---|------|-----------|-------|--------|--------|
| 21 | Scoped /team-review on this DRAFT spec (software-engineer, dba, security-expert, qa-engineer, bff-specialist, Technical Writer) + Boss audit → sign-off boxes checked → status: APPROVED (precedes all build tasks above) | all | Boss | sign-off boxes in spec md checked; status flipped to APPROVED before build | [ ] |
| 22 | After build: /team-review (full panel) → Boss verdict on the built deliverable + evidence; fix SEND-BACK conditions in place; on SHIP set spec md status: HISTORICALLY MARKED IMPLEMENTED + confirm version/changelog | all | Boss | Boss SHIP; status HISTORICALLY MARKED IMPLEMENTED; evidence opened | [ ] |

## Notes

- **Spec-review gate first (Task 21):** the author does NOT self-approve — the Boss convenes the scoped team-review on
  this DRAFT and only flips status: APPROVED once every Review & Sign-Off box is checked. No build task starts before that.
- Schema lane (1–2) and parts of the pipeline lane can begin once approved; the service lane (11–14) depends on the
  payload/queue/consumer changes (3–6) and the column (1–2) and the event/constants/util (7–10).
- A task is DONE only when its REQ-NNN evidence is captured under the private evidence directory for the requirement and opened by the reviewer.
- **No live WhatsApp send is in scope to evidence** — WhatsApp is toggled OFF; evidence is enqueue/params/sweep
  correctness + the template drafts (design "honest scope").
- Do D = verify gate green (the documented operational check, ≥97.5% line+branch) + local deploy health-green + push
  (per project CLAUDE md). On completion update spec md status: HISTORICALLY MARKED IMPLEMENTED and confirm version/changelog.

## 2026-09-05 correction follow-up — implementation not verified

- [ ] **REQ-006/008 — frontend-expert + software-engineer:** update consent copy, its pinned tests and delivered-event config together to include scheduling and reminders; link the existing founder cost approval and verify the global OFF default.
- [ ] **REQ-003/005 — scheduler owner:** use only spec 007 current-generation/durable-dedup requirements; verify short-notice confirmation-only behavior and the enablement gates. Do not reintroduce the historical sweep/flag.
