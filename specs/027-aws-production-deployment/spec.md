> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 027-aws-production-deployment
status: APPROVED
version: 0.3
owners: [devops-engineer, solution-architect, security-expert, dba, software-engineer]
source: approved-decisions md (restored founder-approved v3 source plus later recorded amendments)
github: null
---

# Spec 027 — Amazon Web Services production deployment (budget ≤ $200/mo)

First real, internet-facing production deployment of InTuition SG to **Amazon Web Services** (ap-southeast-1,
Personal Data Protection Act residency), domain in **Cloudflare**, on a **≤ $200/mo, low-traffic, pre-revenue** budget
(est. ~$80–105/mo). Managed managed relational database + Elasti Cache, container service Fargate containers, Cloudflare Pages + Tunnel
edge, SSM/key management service secrets, Cloud Watch logs/alarms, automated backups (PITR), GitHub-push continuous integration/continuous deployment with a
light tunnel-connector canary (1 task → 2h alarm-gated bake → promote/rollback).

Architecture chosen after two review rounds (domain team-review R1–R21; senior Amazon Web Services principal review
A1–A10/B1–B7) and a Fin Ops pass that re-scoped from the ~$800–1,000/mo premium topology to this lean
design. The recovered [original v3 plan](evidence/approved-plan-v3.md) preserves the full design, historical cost model and review traceability. [Current approved decisions and verification](approved-decisions.md) apply later phase amendments and retained product/non-functional requirement criteria; the foreign local-plan path is no longer authoritative.

## Changelog
- 0.3 (2026-09-05) — User-authorized documentation correction: recover original Amazon Web Services v3 with provenance; apply later phase decisions; specify drain, delivery-preserving stream capacity, bootstrap cleanup and retained non-functional requirement acceptance. No new infrastructure apply or runtime verification is claimed.
- 0.2 (2026-06-25) — Phase A0 HISTORICALLY MARKED IMPLEMENTED (gate-verified, Boss SHIP IT, local-boot ✓, pushed). Phases 0–5 Terraform AUTHORED + Boss-reviewed + committed (Boss SEND-BACK on Phase 4 [canary crash-loop gate] and Phase 5 [SNS/CMK silent-drop] — both fixed + re-verified). Remaining: Phase 6 cutover + terraform apply — BLOCKED on founder Amazon Web Services creds + Cloudflare token + GitHub secrets.
- 0.1 (2026-06-24) — APPROVED. Founder-approved plan (v3, budget ≤$200/mo). Dual-reviewed; Fin Ops re-scope. Phase A0 ships first.

## Requirements

### REQ-A0-1 — readiness health group excludes showcase (R6)
A dedicated management endpoint.health group.readiness (db,redis,disk Space,ping, **excluding**
showcase) at /actuator/health/readiness, used by container service/canary health checks, so a routine
Showcase Health Indicator DEGRADED/WARNING can't flap deploys.
- Test: readiness returns 200 when showcase is DEGRADED but db/redis UP; flips to 503 when db down.

### REQ-A0-2 — graceful shutdown + consumer drain (R8)
server shutdown is graceful + spring lifecycle.timeout-per-shutdown-phase; @Pre Destroy/Smart Lifecycle
on the Redis-stream consumers + outbox poller so SIGTERM drains in-flight work.
- Tests: begin a real in-flight send/dispatch, initiate context close, and assert no new claims/polls start. Work that completes within the configured grace period must record success before ACK/SENT; work interrupted at timeout remains durably recoverable (outbox retry/operator replay or unacknowledged stream entry reclaimed after restart). Force termination before successful send/XADD and prove restart retries it rather than losing it. Document grace/stop timeouts and prove they cover the configured shutdown budget. At-least-once duplicates remain handled by the existing dedup contract; shutdown is not an exactly-once claim.

### REQ-A0-3 — public /version provenance endpoint + GIT SHA + arm64 (R1/A7)
A permit All, secret-free read operation for version returning only the deployed revision identifier; Dockerfile backend gains
ARG/ENV GIT SHA; the image builds **arm64** (Graviton Fargate).
- Test: /version returns the build SHA unauthenticated; /actuator/info still 401 for anonymous.

### REQ-A0-4 — migration exit-runner (R3)
A guarded path to run Liquibase then exit (same image; SPRING LIQUIBASE ENABLED is yes +
spring main.web-application-type is none + a Command Line Runner that applies db changelog-master implementation reference
and the exit operation), for the pre-deploy container service Run Task (prod keeps liquibase enabled is no at boot).
- Test: the runner applies pending changesets and exits 0; no web server starts.

### REQ-A0-5 — one-time passcode debug logging OFF in prod (R11) — SECURITY BLOCKER
one-time passcode DEBUG LOG must be no in the prod profile/task def (default is TRUE in base config →
plaintext one-time passcode + reset-link + personal information in Cloud Watch = account-takeover + Personal Data Protection Act breach). Blocking cutover
gate + post-deploy log-grep proving zero one-time passcode-DEBUG lines.
- Test: with the prod profile, an one-time passcode flow logs no plaintext one-time passcode/recipient line.

### REQ-A0-6 — forward-headers / rate-limit integrity (R12)
Constrain Spring forward-headers-strategy to the trusted hop (loopback/cloudflared sidecar only),
and ensure rate-limit client-IP derives from CF-Connecting-IP via the (now sole) trusted peer.
- Test: a spoofed CF-Connecting-IP/XFF from an untrusted source is not honored for bucketing.

### REQ-A0-7 — springdoc/swagger disabled in prod (R15)
springdoc api-docs enabled is no + springdoc swagger-ui enabled is no in the prod profile.
- Test: /v3/api-docs and /swagger-ui are not served under the prod profile.

### REQ-A0-8 — prod super-admin bootstrap runner (R14)
A **separate** the profile operation idempotent one-off runner (NOT the local seeder; the local seeder's
hard-coded Gmails/weak passwords must never activate in prod): creates the super-admin with a password
from SSM, password Change Required is yes, multi-factor authentication-enrol required before any admin action; the bootstrap
secret is deleted after first password change.
- Tests: runner is idempotent (exists By Email short-circuit); no pre-multi-factor authentication admin action possible. After first successful password change, verify the bootstrap SSM secret is deleted and not logged. Inject deletion failure: the password change remains successful, the cleanup stays durably pending/retryable with an operator-visible failure, and a retry removes the secret without recreating/resetting the admin. Do not report bootstrap complete until cleanup succeeds.

### REQ-A0-9 — connection + TLS hardening (R10/R20)
Pin spring datasource.hikari maximum-pool-size (computed vs managed relational database max connections); enable
client-side Redis/Valkey TLS (spring data.redis ssl.enabled is yes) and database sslmode is require in prod
to match Elasti Cache/managed relational database in-transit encryption.
- Test: prod profile config asserts pool size set + Redis SSL on + database sslmode require.

### REQ-A0-10 — bounded Redis streams (R16)
Email/WhatsApp streams have a configured positive capacity and cleanup policy, paired with noeviction, while retaining the [at-least-once outbox/consumer contract](../architecture/transactional-messaging-and-idempotency.md). Never trim an unread/unacknowledged message solely because it is old or the stream is full. Reap only entries whose required delivery/failed-message queue handoff has completed. If there is no safe space, defer publication: retain the durable outbox intent in retryable or operator-replayable failure state and do not mark it SENT.
- Tests: fill the configured capacity with unread/pending messages, continue producing with consumers stopped, and assert stream storage stays within the documented capacity bound while every unaccepted publish remains durably recoverable. Restart consumers/replay retained outbox work and account for every event as delivered, deduped after prior success, or durably dead-lettered. Exercise full-capacity, pending-entry reclaim, publish failure and consumer restart; no unconsumed event disappears through trimming. Record capacity/cleanup thresholds and saturation alerts. A bounded Redis stream does not claim that the durable outbox can grow without monitoring or capacity planning.

### REQ-INFRA — Terraform infrastructure (Phases 0–6)
Terraform (Amazon Web Services + Cloudflare providers, state in S3+Dynamo database) under intuition-spec/infra/terraform/, per [approved-decisions md](approved-decisions.md):
VPC (public + private-data, no NAT); 1 key management service CMK (deny-deletion policy); container image registry (scan, immutable,
lifecycle); SSM Secure text params; GitHub OIDC roles (scoped, push/deploy split); Cloud Trail; managed relational database
PG15 t4g micro single-AZ (gp3, PITR 14d selected by Phase 1, force ssl, prevent destroy); Elasti Cache Valkey t4g micro
1 node (noeviction, TLS+AUTH, prevent destroy); container service Fargate (arm64, backend + cloudflared sidecar,
public subnet, SG no-inbound, SSM secrets); Cloudflare Tunnel (example com→localhost);
Cloudflare Pages (SPA) + DNS + web application firewall + Health Check; continuous integration/continuous deployment (GH Actions OIDC → container image registry → migration Run Task →
tunnel-connector canary → Step Functions 2h bake → promote/rollback); Cloud Watch logs (90d, key management service,
IA-cold) + log-metric-filter alarms + SNS email + Logs-ingest billing alarm.
- Tests: the recovered [original Verify table](evidence/approved-plan-v3.md), updated by the [current phase acceptance table](approved-decisions.md). Preserve Phase 4's manual-only pre-cutover deployment constraint and both canary liveness gates; Phase 4 owns5xx/ERROR filters, Phase 5 SNS uses its reviewed alias/aws/sns amendment.

### REQ-CUTOVER — go-live (Phase 6)
Prod super-admin provisioned (multi-factor authentication); Resend custom domain REDACTED email (SPF/DKIM/DMARC in
Cloudflare) + prod key; Cloudflare DNS go-live; rollback runbook; PITR restore drill.
- Test: end-to-end smoke (super-admin login+multi-factor authentication, one tutor + one tutee happy-path, an admin action, a real
  @example com email); origin unreachable except via tunnel; one-time passcode log-grep zero. Also capture [retained recovery, retention, availability, latency and provider acceptance](approved-decisions.md); a happy-path smoke is not evidence for those independent requirements.

## Design notes

Use [approved-decisions md](approved-decisions.md) for current scope and the [recovered source](evidence/approved-plan-v3.md) for the original architecture/canary/secrets/cost/review record. Historical claims and estimates are attributed to that dated source; current acceptance evidence remains required.

- **Edge**: Cloudflare Pages (frontend) + Cloudflare Tunnel → cloudflared sidecar → localhost (no
  application load balancer, no NAT, no public inbound — origin unreachable from the internet by construction).
- **Data**: managed relational database PG15 t4g micro single-AZ + PITR; Elasti Cache Valkey t4g micro 1 node, noeviction.
  Single-AZ/single-node has no auto-failover. Measure recovery against the retained business requirements document objectives; the old10–15-minute manual-restore estimate is not a guarantee. Multi-AZ/replica remains a deferred scale-up choice.
- **Migrations**: pre-deploy container service Run Task (REQ-A0-4); **expand-contract mandatory** during the bake
  (additive-only; enum-value writes two-release; no new db-scheduler task names; Redis key shapes);
  continuous integration **database structure definitions linter** + dba sign-off.
- **Secrets**: SSM Secure text (free) + 1 customer-managed key management service key (deny-deletion policy); Tink
  keyset + keyed message authentication are crown-jewels with offline split-knowledge backup + tested restore runbook.
- **Repos**: intuition-backend (A0 code/deploy workflow), intuition-frontend (Pages build/deploy workflow), and intuition-spec/infra/terraform/ (infrastructure as code), as settled in Phase 0 evidence.

**Deferred (re-addable, documented as "scale-up when revenue arrives"):** multi-account, Multi-AZ,
on-Amazon Web Services staging, Guard Duty/Config/Security Hub default, Cloud Front/application load balancer, interface endpoints, native
blue/green. Guard Duty (~$10–15) recommended soon after go-live given personal information/Personal Data Protection Act.

## Tasks
- [ ] Phase A0 backend hardening (REQ-A0-1..10) + tests, including in-flight drain/recovery, bootstrap-secret cleanup retry and delivery-preserving stream capacity; gate backend ≥97.5/97.5; prod profile boots
- [ ] Phase 0 Foundations (Terraform: state, VPC, key management service, container image registry, SSM, OIDC, Cloud Trail)
- [ ] Phase 1 Data (managed relational database + Elasti Cache Valkey; migrate empty database; PITR drill)
- [ ] Phase 2 Compute + Cloudflare Tunnel (Fargate + cloudflared sidecar; /version live)
- [ ] Phase 3 Edge (Cloudflare Pages + DNS + web application firewall + Health Check)
- [ ] Phase 4 continuous integration/continuous deployment + canary (GH Actions OIDC → container image registry → migrate → tunnel canary → Step Functions bake)
- [ ] Phase 5 Observability (log groups, alarms, SNS, billing alarm)
- [ ] Phase 6 Cutover (super-admin/bootstrap cleanup, Resend domain, DNS go-live, rollback runbook, smoke) + retained non-functional requirement/provider/retention evidence in approved-decisions md
- [ ] /team-review after each phase deliverable + Boss verdict; private evidence grouped by requirement

## Review & Sign-Off
- [x] Founder — approved plan v3 (budget ≤$200/mo)
- [x] Scoped team-review (architect/security/dba/devops) — R1–R21 folded in
- [x] Amazon Web Services principal review — A1–A10/B1–B7 folded in (then re-scoped to budget)
- [ ] Boss (Rex) — per-phase verdicts as phases ship
