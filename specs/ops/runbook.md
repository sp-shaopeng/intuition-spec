# Operations Runbook — Booting & Running InTuition

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 correction:** Provenance, outage/failed-message queue and reminder procedures are reconciled with007/027. Local and OCI dev setup remain environment-specific; Amazon Web Services production uses027. [Current requirements](../CURRENT-REQUIREMENTS.md). No runtime verification was performed by this documentation correction.

Authoritative, step-by-step guide to **boot, verify, and troubleshoot** the
InTuition stack. For a deeper Docker-Compose-only reference see
[local-stack md](local-stack.md); for alerting/observability see
[monitoring md](monitoring.md).

There are two ways to run the system:

- **Mode A — Full Docker stack.** One command brings up Postgres, Redis, backend,
  and frontend. Best for "just run it" and for verifying production-like wiring.
- **Mode B — Native dev loop.** Postgres + Redis in Docker; backend and frontend
  run natively for hot reload. Best for active development.

---

## 0. Prerequisites

| Requirement | Version / Detail | Check |
|---|---|---|
| Docker Desktop | running, ≥ 4 GB RAM, Compose v2 | the documented operational check |
| Java (native backend only) | **21 — required** | check the installed Java version |
| Maven (native backend only) | 3.9+ | the documented operational check |
| Node + npm (native frontend only) | Node 20+, npm 10+ | check the installed Node version |
| Free host ports | 5432, 6379, 8080, 5173 | confirm each required port is available |

### Java 21 is mandatory (native backend)

The system default JDK (e g. Java 26) **breaks Lombok annotation processing and
JaCoCo instrumentation**. Always pin Java 21 via JAVA HOME before any the documented operational check
command:


Implementation example omitted. This public edition retains the written requirements in this section.


If openjdk@21 is not installed: brew install openjdk@21.

### Start Docker if it is not running


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Service & port reference

| Service | Image / Build | Host → Container | Purpose |
|---|---|---|---|
| postgres | postgres-alpine | 5432 → 5432 | Primary datastore |
| redis | redis-alpine | 6379 → 6379 | Sessions, cache, email queue |
| backend | ./intuition-backend (Dockerfile backend.dev) | 8080 → 8080 | Spring Boot application programming interface |
| frontend | ./intuition-frontend (Dockerfile, nginx) | 5173 → 80 | React SPA |

Default credentials (dev only, from docker-compose implementation reference / .env implementation reference):
Postgres intuition / REDACTED password, Redis password REDACTED password.

---

## Mode A — Full Docker stack


Implementation example omitted. This public edition retains the written requirements in this section.


Subsequent starts (no code change) are near-instant:


Implementation example omitted. This public edition retains the written requirements in this section.


Verify (see [§ Health verification](#health-verification)). Then open
**http://localhost**.

---

## Mode B — Native dev loop

### 1. Start only the data services in Docker


Implementation example omitted. This public edition retains the written requirements in this section.


### 2. Run the backend natively (Java 21, local profile)


Implementation example omitted. This public edition retains the written requirements in this section.


The local profile (application-local implementation reference) points
the datasource and Redis at localhost, enables Liquibase (migrations run on
startup), and sets dev cross-origin request permissions. Wait for the log line:


The illustration for . run the backend natively java 21, local profile contains the following information:

- Started Intuition Backend Application in N seconds.


### 3. Run the frontend natively (new shell)


Implementation example omitted. This public edition retains the written requirements in this section.


Vite proxies /api → VITE application programming interface BASE URL (default http://localhost), so
the SPA talks to the native backend with no cross-origin request permissions friction. Open
**http://localhost**.

---

## Health verification

Works for both modes.


Implementation example omitted. This public edition retains the written requirements in this section.


Quick port sweep:


Implementation example omitted. This public edition retains the written requirements in this section.


### Deploy gate: verify build provenance

Use the minimal unauthenticated read operation for version endpoint from027, returning the deployed revision identifier. Compare its value to the commit actually built/deployed; empty or mismatched SHA fails verification. /actuator/info remains protected and is not an anonymous deployment probe.


Implementation example omitted. This public edition retains the written requirements in this section.


Run against the intended environment and document the comparison. Historical /actuator/info success notes do not override027.

## Teardown


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| the documented operational check fails / compose hangs | Docker Desktop not running | open -a Docker, wait, retry |
| Port already in use (5432/6379/8080/5173) | Another process owns the port | Stop it, or lsof -ti :PORT \| xargs kill |
| the documented operational check build fails on Lombok / JaCoCo | Wrong JDK (e g. Java 26) | Export Java 21 JAVA HOME (see §0) |
| Backend unhealthy at first boot | Postgres not ready yet | Wait; it retries. the documented operational check |
| Backend fails: Liquibase changelog lock | Unclean shutdown left a stuck lock | Release the lock (below), restart |
| Frontend started on **5174** not 5173 | 5173 already taken (e g. Docker frontend already up, or a stray vite) | Don't run two frontends; kill the stray: pkill -f vite then lsof -ti  |
| 401 on every application programming interface call | Stale/invalid auth cookie | Clear site cookies, re-login |
| cross-origin request permissions error in browser | Origin not allowed | In local profile any localhost/REDACTED ip address port is allowed; for other envs add it to cross-origin request permissions ALLOWED ORIGINS and restart |

### Release a stuck Liquibase changelog lock

If the backend fails to start citing a Liquibase lock after an unclean shutdown:


Implementation example omitted. This public edition retains the written requirements in this section.


Then restart the backend. (Inspect first with
the migration lock's identifier, locked state and owner.)

### Kill a stray native process holding a port


Implementation example omitted. This public edition retains the written requirements in this section.


---

---

## Email Consumer (Resend / Redis Streams)

### New developer setup


Implementation example omitted. This public edition retains the written requirements in this section.


If RESEND application programming interface KEY is not set, the app starts normally but logs a WARN on startup and skips all email dispatch. Delivery is deferred. Record pending work; do not report success or ACK solely because the transport is disabled. Verify replay on re-enable before claiming no data loss.

### Verify the consumer is running


Implementation example omitted. This public edition retains the written requirements in this section.


### Check pending entries (PEL)


Implementation example omitted. This public edition retains the written requirements in this section.


### Disable email without restarting

Remove or blank RESEND application programming interface KEY from your environment and restart the backend.
Email Consumer Service will idle; no messages are lost (they stay in the stream/PEL).

### Key rotation (Resend application programming interface key)

1. Generate a new key in the Resend dashboard → application programming interface Keys.
2. Update RESEND application programming interface KEY in your hosting platform's secret store (Railway variables, etc.).
3. Redeploy the backend — the new key is picked up at startup.
4. Revoke the old key in the Resend dashboard.
5. Smoke test: trigger a forgot-password flow and confirm the email arrives.

### Resend outage or disabled transport

1. Inspect outbox backlog, stream pending entries, retry age and failed-message queue state. Delivery-disabled or missing credentials is not a successful dispatch.
2. Preserve replayable work. Pause outbound attempts using the documented consumer control; do not disable durable business-event capture or mark unsent work delivered.
3. Retry transient failures with the bounded policy. Permanent failures move to the documented failed-message queue with a reason and explicit operator replay/disposition; see “Dead-letter queues” below.
4. After recovery, re-enable the transport, drain/reclaim pending work, and verify outcomes and duplicate prevention.

### Stream bounds and retention

Follow027’s delivery-preserving bounded-stream contract. Do not use unconditional XTRIM MAXLEN on a stream that may contain unconsumed/unacknowledged entries. Bound memory using the specified acknowledged-entry retention/backpressure/durable replay strategy; alarm before capacity exhaustion. A stream length test alone does not prove delivery preservation.


Implementation example omitted. This public edition retains the written requirements in this section.


Use the environment’s configured authenticated/TLS Redis connection. These commands inspect state; they are not a deletion/replay procedure.

## Notes

- Compose pins container names with the intuition prefix
  (intuition-postgres, intuition-redis, intuition-backend,
  intuition-frontend) for predictable the documented operational check targets.
- Liquibase migrations run automatically on backend startup in every mode.
- Do **not** run the native frontend (Mode B) while the Docker frontend (Mode A)
  is up — they both want port 5173; the second one silently falls back to 5174.
- Env var reference: see .env implementation reference at the repo root and
  [local-stack md](local-stack.md) § Environment variables.

---

## WhatsApp kill-switch (INT-082/084 — feature is OFF by default; this applies once enabled)

**Symptom:** 422 storm on submit operation for account phone verify start, otp send/whatsapp send.failed
spiking, unexpected Meta spend, or "user says no WhatsApp arrived" reports.

**Action (restores the marketplace immediately — the gate no-ops when OFF):**
1. Set whatsapp enabled is no (env/compose override) and restart the backend container.
2. Verify: boot log prints the WhatsApp config line (enabled is no …); one-time passcode start returns
   a successful response confirming that the feature is disabled; gated actions (contact-reveal / select / invite) work without verification.
3. If the trigger was a compromised/expired token: rotate WHATSAPP CLOUD ACCESS TOKEN (Meta tokens
   expire ~60d) before re-enabling.

**Diagnosing silent non-delivery (feature ON):** check the number of dropped WhatsApp notifications, grouped by reason and event —
notifications disabled (sub-toggle off), event not enabled (event outside
whatsapp notifications.events — default INTERVIEW SCHEDULED only, founder cost decision
2026-06-07), recipient ineligible (not opted-in / phone unverified). The parsed event set is logged
at boot; an invalid event token in the config **fails boot loudly** naming the token.

**Expanding the delivered event set:** see the DEPLOYMENT GATE in
[postman implementation reference](../reference/postman.md) — founder cost approval + lockstep frontend
consent-copy update, shipped atomically. Never bump the config alone.

---

## WhatsApp interview reminders —007 current scheduler

WhatsApp remains OFF until the recorded founder/ops enablement conditions are met.007 supersedes the old15-minute reminder sent sweep with one-time interview-reminder jobs keyed to interview identity and current schedule generation.

Before enablement:

1. Confirm required toggles, credentials and approved templates under006/INT-082. Keep template time/link parameter ordering.
2. Update the consent copy and its test to include the delivered reminder event in the same change as event configuration. Validate the intended recipients and opt-in conditions.
3. Verify durable delivery deduplication, reschedule generation checks, cancellation/link-removal behavior and shutdown/retry semantics from007. A deleted scheduler row alone is not proof of once-only delivery.
4. Wire required metrics/alerts and run the current acceptance scenarios. Do not reuse the old sweep’s timing, metrics or migration0061 rollback as current instructions.

Diagnose scheduler task backlog/failures separately from outbox, stream, delivery and failed-message queue state. Disable outbound delivery through the kill-switch in an incident while preserving the documented durable state. Rollback follows007’s schema/application compatibility plan; no ad hoc scheduler-row or stream deletion.

The template reference covers message content. The scheduler specification owns timing and delivery safety.

## Dead-letter queues (INT-087)

Both Redis-stream consumers (Email Consumer Service, Whats App Consumer Service) park poison
messages in a dedicated dead-letter stream instead of XCLAIM-looping a bad message forever. The
streams are **email-dead-letter** and **whatsapp-dead-letter**.

**What dead-letters a message** (the reason tag):
- max redeliveries — a PEL message reclaimed ≥ dlq max-deliveries times (default **5**, config-
  overridable) without a successful send is treated as poison (permanently-bouncing address,
  downstream rejecting it). The reclaim loop dead-letters it and ACKs it off the PEL.
- malformed — the message is missing required fields (email: template Id/to; WhatsApp:
  kind/to, or a notification missing template). It can never succeed, so it is dead-lettered
  + ACKed on first sight.

The one-time passcode field is **never** copied into the dead-letter record (the failed-message queue must not become a new at-rest
location for a secret), so one-time passcode messages dead-letter without their code.

**Inspect a dead-letter stream:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Metrics to watch** (Micrometer):
- the number of email messages moved to the failed-message queue, grouped by reason / the number of WhatsApp messages moved to the failed-message queue, grouped by reason — counter, incremented per
  dead-letter (tagged max redeliveries / malformed).
- email pel.depth / whatsapp pel.depth — gauge of the consumer's pending-entries-list depth
  (a stall is the *absence* of progress, which a counter alone can't catch — watch this for a
  growing backlog).
- email consumed.deduped / whatsapp consumed.deduped — counter, incremented when a redelivered
  message is recognised as already-sent and ACKed without a second send (the consume-side dedup
  working as intended).

> **ALERT WIRING IS STILL OWED.** No alert is currently fired on a rising *.deadlettered,
> *.pel depth, or failed-message queue XLEN — wiring these alerts is tracked under **INT-082 PE-3 (still open)**.
> Until then these are inspect-on-demand only.

---

## Success metrics (Dana)

- **Password-reset delivery progression** — outbox SENT measures handoff to the stream, not email delivery. Track consumer/provider acceptance, retry/PEL and permanent failure/failed-message queue (email deadlettered / failed-message queue XLEN) separately. Claim recipient delivery only where the provider supplies corresponding evidence; an outbox handoff cannot satisfy a delivery service-level agreement.
- **Commission-pay distinct-key race count** — the count of COMMISSION ALREADY PAID 409s (the
  Commission optimistic-lock losers): how often two concurrent pays race one PENDING commission
  (it is still paid exactly once).
