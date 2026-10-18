# Monitoring & Observability Spec — Intuition Platform

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 correction:** Retain application SLOs and observability requirements while mapping deployment/provider instructions to027. Prom QL/dashboard examples are reference designs, not evidence that production monitoring is wired. [Current requirements](../CURRENT-REQUIREMENTS.md). No runtime verification was performed by this documentation correction.

## 1. Overview

This document defines the monitoring strategy, alert rules (Prom QL), runbook sections, and SLOs for the Intuition tuition-agency platform. It covers three tiers:

- **Application** (Spring Boot / React frontend)
- **Infrastructure** (the approved027 Amazon Web Services production topology; local/OCI dev are separate)
- **Business** (post volume, match rate, commission collected)

---

## 2. Observability Stack

| Layer | Tool | Notes |
|-------|------|-------|
| Metrics | Prometheus + Spring Boot Actuator (/actuator/prometheus) | management endpoint.prometheus enabled is yes; scrape interval 15 s |
| Dashboards | Production dashboard destination from027’s current deployment plan; legacy Grafana examples remain illustrative | |
| Alerting | Prometheus Alertmanager → Pager Duty / email | |
| Logs | 027 production log destination with the retained structured logging/retention contract | Structured JSON; logging context correlation Id on every request |
| Tracing | Open Telemetry (future Phase 2) | OTLP export to Grafana Tempo |
| Uptime | Uptime Robot (public health check) | Checks /actuator/health every 1 min |

### 2.1 Prometheus Scrape Configuration

/actuator/prometheus requires ROLE ADMIN (Spring Security). The Prometheus scraper must authenticate. Use a long-lived service-account signed session token issued offline and stored in Prometheus secrets:


Implementation example omitted. This public edition retains the written requirements in this section.


> **Note:** The application: intuition-backend label is injected by the scrape job, not by Spring Boot auto-configuration. All Prom QL alert rules in Section 4 filter on application is intuition-backend — if this label is absent, no alerts will fire.
>
> The Bearer token must be a signed session token with role is ADMIN claim. Issue it via the admin seed script (seed-admin implementation reference) and rotate every 90 days.

---

## 3. Golden Signals

| Signal | Metric | Alert Threshold |
|--------|--------|-----------------|
| Latency | P95 web request protocol response time | > 500 ms for 5 min |
| Traffic | Requests/sec | < 0.01 req/s for 10 min (dead service) |
| Errors | 5xx error rate | > 1% for 3 min |
| Saturation | JVM heap usage | > 80% for 5 min |

---

## 4. Prom QL Alert Rules

### 4.1 application programming interface Latency — P95


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.2 Interview Scheduling P95 Alarm (business requirements document §9.5)


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.3 web request protocol 5xx Error Rate


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.4 JVM Heap Saturation


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.5 Redis Connection Pool Exhaustion


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.6 Hikari CP Connection Pool


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.7 Service Unavailability (Dead Man's Switch)


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.8 Transactional Outbox — backlog, failures & poller stall (INT-067 class)

> The durable fan-out (write business data + emit an event in the SAME tx, drained by a
> the scheduled task poller) is invisible to counters when it *stops incrementing* — a stall is the
> **absence** of progress, which a raw error-rate alert can never catch. These four rules make a
> silent feature death (cf. INT-067, where a the scheduled task sweep threw every run and silently never
> drained) page-able. Metric names assume the Micrometer gauge outbox pending depth (backlog depth) and
> the counters outbox events sent total / outbox events failed total, plus the interview-expiry
> sweep counter interview expired errors total.


Implementation example omitted. This public edition retains the written requirements in this section.


> **Why a gauge AND counters:** outbox pending depth (gauge) answers "how deep is the backlog right
> now"; outbox events sent total / failed total (counters) answer "is it draining and is it
> clean". Rule (c) needs BOTH — backlog depth alone can't distinguish "busy but progressing" from
> "stuck", and the sent-counter alone can't tell idle-because-empty from idle-because-wedged. Ship
> the gauge + these alerts in the SAME change as any new poller/sweep (INT-075 lesson: new async
> machinery ships with its backlog gauge + alert, never after).

---

## 5. Business Metrics (Custom) — Phase 2 / Not Yet Implemented

> **Phase 2 deliverable — not part of M5 Dev Ops baseline.** These metrics are not instrumented in the current codebase. They are defined here so implementation is unambiguous when Phase 2 begins.

Register these as Micrometer Counter / Gauge beans in Analytics Event Publisher:

| Metric | Type | Description |
|--------|------|-------------|
| the number of submitted posts, grouped by tutor or tutee type | Counter | Posts created per type |
| the number of approved posts, grouped by type | Counter | Admin approvals |
| the number of rejected posts, grouped by type | Counter | Admin rejections |
| intuition matches created total | Counter | New matches by admin |
| intuition commissions invoiced total | Counter | Invoices generated |
| intuition commissions paid total | Counter | Payments marked paid |
| intuition pending posts gauge | Gauge | Current PENDING queue depth |

### 5.1 Durable fan-out (transactional outbox + sweeps) — instrumented

> Unlike the Phase-2 business metrics above, these are wired alongside the outbox/sweep machinery
> (INT-067 class) and back the alert rules in §4.8. A new poller/sweep MUST ship its backlog gauge
> in the same change — a stall is the absence of progress and is otherwise invisible.

| Metric | Type | Description |
|--------|------|-------------|
| outbox pending depth | Gauge | Current count of PENDING rows in the transactional outbox (backlog depth) |
| outbox events sent total | Counter | Outbox events successfully drained by the poller |
| outbox events failed total | Counter | Outbox events that failed delivery (downstream/sink error) |
| interview expired errors total | Counter | Errors thrown by the idempotent interview-expiry sweep (INT-067) |
| interview expired | Counter | Interviews swept to EXPIRED (INT-039 — successful expiry count) |

---

## 6. Structured Log Format

Every request log must include:


The . structured log format example describes information including timestamp, level, correlation Id, user Id, method, uri, status, duration Ms, service. Concrete payload values and implementation syntax are omitted.


**personal information rules for logs:**
- user Id — unique identifier only, never email or name
- Phone/email — NEVER in logs; plaintext database storage does not waive log redaction
- special Requirements — never logged (free-text personal information sink)
- Meeting links — never logged (violates Personal Data Protection Act context)

**Log retention policy (Personal Data Protection Act Singapore compliance):**
- Production application-log retention: **maximum90 days**, configured for the current027 logging destination
- After 90 days, logs must be purged or anonymized (replace user Id unique identifier with a salted hash, remove correlation Id)
- Rationale: correlating correlation Id sequences over time constitutes metadata that can de-anonymize users in a breach scenario; 90-day cap limits exposure window
- Verify the027 log-group/sink retention setting is within this cap; retain an explicit approved exception for longer audit retention
- Archive policy: if longer retention is required for audit, export to encrypted cold storage (e g., Amazon Web Services S3 Glacier) after stripping all identifiers

---

## 7. Runbook Sections

### 7.1 High Latency (high-latency)

1. Identify the slow URI from the alert label
2. Check slow-query log: EXPLAIN ANALYZE <query> on the Railway Postgres console
3. Check missing indexes: specs/domain/database/indexes md
4. If caused by N+1: add @Entity Graph or a JOIN FETCH in the repository
5. Rollback: if a recent deploy caused the regression, use Railway's instant rollback

### 7.2 5xx Errors (5xx-errors)

1. Pull structured logs from Logtail filtered by level is ERROR + the alert time window
2. Look for correlation Id to trace the full request chain
3. If no value Pointer Exception or Data Integrity Violation Exception: check recent Liquibase migrations
4. If Jwt Exception: check key rotation — jwt secret.previous may need updating
5. Escalate if error rate stays > 5% for 10 min → Pager Duty P1

### 7.3 JVM Heap (jvm-heap)

1. Capture a memory diagnostic using the authorised process or hosting tools
2. Analyse with Eclipse MAT for leak suspects
3. Common culprit: holding large Result Sets in memory — use pagination
4. Short-term mitigation: restart/recover the affected027 service using its deployment runbook; verify recovery rather than assume a30-second RTO

### 7.4 database Pool Exhaustion (db-pool)

1. Inspect database sessions that are idle while holding a transaction
2. Kill long-running idle transactions: the pg terminate backend operation
3. Check connection count against the configured maximum number of database connections
4. If pool size is the bottleneck, increase spring datasource.hikari maximum-pool-size (default 10)
5. Permanent fix: review how long service operations hold database transactions — avoid holding transactions during slow external calls

### 7.5 Interview Scheduling Latency (interview-latency)

1. Query is likely joining interviews → matches → tutor posts → tutee posts
2. Check index on interviews match id and interviews status
3. If background email queue is blocking: verify Email Queue Service async executor thread pool is not saturated
4. business requirements document §9.5 requires interview scheduling to complete within 2 s P95 in steady state

### 7.6 Service Down (service-down)

1. Check Railway dashboard for deployment errors or OOM kills
2. Verify environment variables are set (JWT SECRET CURRENT, DB URL, REDIS HOST)
3. If crash-loop: check application logs for startup failure (Liquibase, Data Source connection)
4. Incident restart objectives are separate from disaster recovery. Retain business requirements document RTO≤4hours and RPO≤24hours; verify the relevant failure mode under027

### 7.7 Outbox Backlog / Poller Stall (outbox-backlog)

Triggered by Outbox Backlog High, Outbox Events Failing, Outbox Poller Stalled, or
Interview Expiry Sweep Errors (§4.8).

1. Read the gauge + counters: outbox pending depth (how deep), the rate operation
   (is it draining), the rate operation (is it clean).
2. **Backlog high but sends advancing** → producers are outrunning the poller; transient spike.
   Confirm outbox pending depth is trending DOWN; if it plateaus, treat as a stall (step 4).
3. **Failures > 0** → events claimed but not delivered. Inspect the failed rows and the downstream
   sink (Redis stream / email). Check backend ERROR logs around the alert window by correlation Id.


Implementation example omitted. This public edition retains the written requirements in this section.


4. **POLLER STALLED (the INT-067 signature: backlog > 0 AND zero sends for 10 min)** — the
   the scheduled task drain is throwing every run and swallowing it.
   - Pull the backend log for a repeating swallowed exception on the drain/claim query.
   - INT-067 root cause to rule out first: a named-PG-enum JPQL cast renders
     PROPOSED Interview Status → PG folds to lowercase → type "interviewstatus" does not
     exist → the query throws EVERY run. Fix is a NATIVE cast to the snake case PG enum type
     (the cast operation), not a @Enumerated JPQL literal.
   - Mitigation while fixing: a backend restart re-arms the scheduler (does not fix a query bug).
5. Verify recovery: outbox pending depth drains toward 0 and outbox events sent total advances;
   the alert clears.

> **Verify on real PG, never H2:** an outbox/sweep regression IT MUST run on Testcontainers PG —
> H2 has no PG enum types and hides the INT-067 cast bug entirely.

---

### 7.8 Redis Down — money POSTs fail CLOSED by design (redis-down)

The money/assign POSTs (pay-commission, admin-assign, tutee-select) require an Idempotency-Key
guard backed by Redis (SETNX sentinel + replay store). **By design, if Redis is unreachable these
endpoints FAIL CLOSED — they return 500 rather than risk a double-charge / double-assign without
the dedup guard.** A burst of 500s ONLY on the money/assign endpoints, with Redis alerts firing
(see §4.5 Redis Connection Pool Exhausted), is this condition — not an application logic bug.

1. Confirm Redis is the cause: the documented operational check (expect
   PONG); check lettuce command completion seconds count is flat (§4.5).
2. **Restore Redis** — this is the fix, not a code change. Restart/recover the Redis service; the
   money/assign endpoints recover automatically once the dedup store is reachable again.
3. Do NOT "fix" the 500 by removing the guard — failing closed is the correct, intended behaviour;
   the alternative (process money without idempotency) risks double-charges.
4. Email/notification fan-out also rides Redis (streams); see §7.7 and the Email Consumer runbook
   for the PEL — those queue and drain on recovery (do not fail closed; they retry).

---

## 8. SLO Summary

| SLO | Target | Measurement Window |
|-----|--------|--------------------|
| Public availability | 99.5% | Calendar month, per business requirements document/product requirements document |
| Admin availability | 99.9% | 08:00–22:00 Singapore time, seven days/week, per retained business requirements document window |
| application programming interface latency P95 | < 500 ms | 24-hour rolling |
| Interview scheduling P95 | < 1 s | 24-hour rolling |
| Error budget (5xx) | < 0.5% of requests | 7-day rolling |
| Admin queue review time | < 24 h (median) | Business KPI, not technical SLO |

---

## 9. Alert Routing


Implementation example omitted. This public edition retains the written requirements in this section.


---

*Last updated: 2026-06-05 — adds §4.8 transactional-outbox / poller-stall Prom QL alerts
(outbox pending depth, outbox events failed total, interview expired errors total, INT-067 class),
§5.1 durable-fan-out metrics, §7.7 outbox-backlog runbook, and §7.8 Redis-down (money POSTs fail
closed by design) triage. Previously: 2026-05-28 — backend Modules 1–5, Dev Ops baseline, business requirements document §9.5
interview latency SLO.*
