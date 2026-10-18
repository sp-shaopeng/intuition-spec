> Supporting decision record: this approved plan is retained as a prose specification. Raw evidence, deployment scripts and operational values are excluded.

# InTuition SG — Amazon Web Services Production Deployment Plan (v3, budget ≤ $200/mo)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Context

InTuition SG (Spring Boot 3.3 / Java 21 application programming interface + React 19 / Vite static SPA) is fully built but has
**never run on a real, internet-facing production environment** — only a local Docker stack + a dev
virtual machine. The founder wants it on **Amazon Web Services**, domain stays in **Cloudflare**, with managed database, containers,
streamed logs + monitoring, well-managed secrets, and a safe canary-style deploy on GitHub push.

**Hard constraint (this revision): ≤ $200/mo, low traffic, pre-revenue.** v1/v2 of this plan (container service
fleet + managed relational database Multi-AZ + Elasti Cache 1+1 + multi-account + on-Amazon Web Services staging + Guard Duty + a 2h
multi-machine canary) costed **~$800–1,000/mo** — its *fixed* always-on plumbing (NAT $48, interface
endpoints $70, application load balancer $24, managed relational database Multi-AZ $250, security baseline $50, a 2nd environment ~$200) blows the
budget regardless of traffic. **v3 keeps the founder's core goals** (managed managed relational database + Elasti Cache,
containers, Cloudflare edge, secrets, logs, automated backups, a light canary) and **eliminates the
fixed-cost killers**: no NAT, no application load balancer, no multi-account, no on-Amazon Web Services staging, no default Guard Duty.

**Estimated cost: ~$80–120/mo** (well under the $200 ceiling, with headroom for optional add-ons).

The backend is **verified safe to run as 2 identical instances** (db-scheduler database-row leader
election, Redis-Streams consumer groups, outbox with a record lock SKIP LOCKED, atomic Redis Lua
rate-limiting, stateless signed session token + Redis admin sessions — no sticky sessions). The frontend is a **pure
static bundle**.

## The cost-killers eliminated (why v3 is cheap)

| v2 fixed cost | v3 |
|---|---|
| NAT gateway ~$48 | **Gone** — Fargate tasks in a **public subnet**; egress direct. Inbound is *closed* (cloudflared dials out). |
| Internal application load balancer ~$24 | **Gone** — a **cloudflared sidecar** in the task connects Cloudflare straight to localhost. No load balancer. |
| VPC interface endpoints ~$70 | **Gone** — public-subnet egress; no Private Link needed at this scale. |
| managed relational database Multi-AZ ~$250 | **managed relational database t4g micro single-AZ + PITR ~$20** — same data durability (point-in-time restore), failover becomes a ~10–15 min manual restore. |
| Elasti Cache 1+1 ~$67 | **t4g micro 1 node ~$12.** |
| Multi-account + staging + Guard Duty ~$300 | **Single account; local docker-compose = staging; Cloud Trail (free) only.** Guard Duty re-addable (~$10–15) later. |

## Target architecture


Implementation example omitted. This public edition retains the written requirements in this section.


- **Single Amazon Web Services account**, region **ap-southeast-1** (Personal Data Protection Act residency).
- **VPC**: public subnets (Fargate tasks, with public IP for egress) + private-data subnets (managed relational database,
  Valkey — reachable only from the task security group). **No NAT, no application load balancer, no interface endpoints.**
- **Ingress = Cloudflare Tunnel only.** The cloudflared sidecar opens an outbound connection to
  Cloudflare and forwards example com → localhost. **The task's security group allows
  no inbound at all** — the origin is unreachable from the internet by construction (this also
  dissolves the v2 origin-locking / IP-allowlist / header-spoof risk class). Cloudflare injects
  CF-Connecting-IP; the backend already prefers it (Client Ip Resolver implementation reference); the only request peer
  the app sees is the loopback sidecar.
- **Frontend = Cloudflare Pages** (static SPA, VITE application programming interface BASE URL is https://example com baked at
  build). Free; atomic deploys + instant rollback. No S3/Cloud Front.
- **TLS**: Cloudflare terminates user TLS; the tunnel leg is encrypted CF↔sidecar. **No ACM cert, no
  public origin cert needed** (no public application load balancer).

## Canary / deployment (tunnel-connector, no application load balancer)

Without an application load balancer there's no listener to shift — so the canary is done at the **Cloudflare Tunnel
connector** level, which is simpler *and* free:

- Normal state: the container service **main service** runs 2 tasks, each with a cloudflared sidecar joined to the
  same named tunnel; Cloudflare load-balances across the healthy connectors.
- **Deploy**: push the new image, then start a **1-task "canary" container service service** (new revision) whose
  sidecar joins the *same* tunnel → Cloudflare immediately routes a share of traffic to it ("the 1
  machine"). The old main service keeps serving the rest.
- **Bake (2h, alarm-gated)**: a tiny **Step Functions** execution waits 2h while polling Cloud Watch
  alarms (5xx via log-metric filter, ERROR-log rate, task crash/stop, managed relational database/Valkey health). Cheap
  (pennies/execution; no idle Fargate). A Cloud Watch **Synthetics** canary (or free Cloudflare Health
  Check) drives a little traffic so the bake has signal. Clean → update the **main service** to the
  new revision (rolling, with the container service **deployment circuit breaker** on) and stop the canary service.
  Alarm trips → stop the canary, main untouched (**auto-rollback**).
- **Sev-1 fast-track**: shorten the Step Functions wait (e g. 10 min) via a deploy variable.
- This honors the founder's "1 instance → watch 2h → then the rest" intent at near-zero added cost.
  (No Code Deploy/native-B/G needed — those assume an application load balancer; the tunnel makes them unnecessary.)

## continuous integration/continuous deployment pipeline

**Backend** (push to sp-dev, after the documented operational check continuous integration green):
1. Build an ARM64 backend image containing the intended source revision. Publish it to the private container registry under an immutable tag, scan it on publication and retain the most recent twenty images.
   Task runtime Platform cpu Architecture is ARM64 (x86 image on arm64 task fails to start).
2. **Pre-deploy migration** = one-off container service Run Task of the same image with
   SPRING LIQUIBASE ENABLED is yes, spring main.web-application-type is none, exit-after-update
   Command Line Runner (a standalone Liquibase CLI is infeasible in the JRE image). Blocks on exit 0;
   single-flight (no concurrent migrations).
3. **Canary deploy** (above) → 2h alarm-gated bake → promote/rollback.
4. Post-promote: deploy-gate asserts the SHA via the public /version endpoint.

**Frontend**: the documented operational check (VITE application programming interface BASE URL baked) → deploy dist/ to **Cloudflare
Pages** (wrangler). No S3 sync / Cloud Front invalidation.

**Auth**: GitHub Actions **OIDC** into a scoped IAM role (trust pinned to aud is sts amazonaws.com +
sub is repo:ORG/REPO:environment:prod); GitHub prod Environment with required reviewers = the human
gate; a separate, narrower container image registry-push role.

**Migration safety — expand-contract mandatory** (mixed versions during the bake): additive-only;
**enum-value writes are two-release** (ALTER TYPE ADD VALUE is safe to add, but the old task's
the enumerated operation + prefer native enum types is yes throws on an unknown label — add label + read
in N, write in N+1); **no new db-scheduler task names** in a canary-gated release; Redis key/field
shapes follow the same rule. A **continuous integration database structure definitions linter + dba sign-off** rejects drop/backfill-in-one, required
w/o default, require/FK w/o NOT VALID, index w/o CONCURRENTLY.

## Secrets management (cost-lean)

Use **SSM Parameter Store Secure text** (free for standard params) instead of Secrets Manager —
saves the per-secret fee at no security loss; encrypted with **one customer-managed key management service key** (~$1/mo).

- **Stored**: database PASSWORD, REDIS PASSWORD/AUTH, signed session token SECRET CURRENT using +PREVIOUS,
  MASTER ENCRYPTION KEYSET (Tink — crown jewel), keyed message authentication SECRET KEY (crown jewel — blind-index
  unguessability), RESEND application programming interface KEY, the cloudflared tunnel token, optional WhatsApp tokens.
- **Injection**: container service task def secrets value From = SSM parameter ARN. Non-secret config
  (one-time passcode DEBUG LOG is no, forward-headers is loopback-only, cross-origin request permissions, COOKIE DOMAIN) in environment.
- **key management service key policy**: deny Schedule Key Deletion/Disable Key except a break-glass principal;
  kms:Decrypt scoped to the execution role via kms:Via Service is ssm ap-southeast-1.
- **IAM least-privilege**: execution role = container image registry pull + Logs + ssm:Get Parameters (scoped to
  /intuition/prod/*) + kms:Decrypt; task role near-empty.
- **Secret-zero / break-glass**: generate fresh on a trusted machine (Tink keyset via Tinkey; others
  openssl rand); **offline plaintext backup of the Tink + keyed message authentication keys** (split-knowledge); written,
  tested restore runbook. Never reuse the committed dev values; never let prod secrets touch git/TF
  state.
- **Rotation = expand-contract**: signed session token PREVIOUS+CURRENT then drop; Tink **add-key-then-promote**
  (never replace the keyset); **keyed message authentication rotation = re-index all blind-index columns** (a migration);
  database/Valkey via a rolling deploy that must not overlap a bake.

## Logging + monitoring (lean)

- **Logs**: container service → Cloud Watch Logs the backend and database-migration log groups, **90-day retention
  (Personal Data Protection Act)**, encrypted with the key management service key, **volume-controlled** (INFO not DEBUG; no request bodies;
  one-time passcode DEBUG LOG is no). Cold groups (migrations) → Logs **IA class**. JSON correlation Id/trace Id
  → Logs Insights. A **billing alarm on Logs-ingest GB** (the line that silently 10×'s with traffic).
- **Metrics/alarms**: [REDACTED AUTHORIZATION] service + managed relational database + Valkey Cloud Watch metrics (free) + **log-metric filters** for
  5xx and ERROR rate (cheaper than scraping). Skip Container Insights and the ADOT/Prometheus sidecar
  at launch (re-add when debugging needs it). Alarms: task crash/stop, managed relational database connections >80% /
  storage / CPU-credit balance (t4g), **Valkey Evictions is 0** (any eviction is an incident),
  ERROR-rate, 5xx-count. → SNS → founder email.
- **Health (R6)**: container service container health check + the canary use a **readiness group**
  /actuator/health/readiness (db,redis,disk Space,ping, **excluding showcase** so a routine
  DEGRADED/WARNING can't flap the deploy). /actuator/health stays for humans.
- **Provenance**: a deliberately-public, secret-free **/version** endpoint exposes only git sha
  (the ADMIN-gated /actuator/info would 403); Dockerfile backend gains ARG/ENV GIT SHA.
- **Uptime**: **Cloudflare Health Checks** (free on their side) for external uptime instead of paid
  Synthetics; keep one short-lived Synthetics canary only to drive bake-time traffic.

## Execution sequencing (Boss-orchestrated, SDD)

| Phase | Deliverables | Verify | Owner |
|---|---|---|---|
| **A0 Backend prod-hardening** (ships first, team-review→Boss; all code/config, free) | readiness group excl. showcase (R6); server shutdown is graceful + consumer @Pre Destroy (R8); public /version + Dockerfile backend GIT SHA + **arm64 build** (R1/A7); migration exit-runner (R3); one-time passcode DEBUG LOG is no + forward-headers loopback-only (R11/R12); springdoc off in prod (R15); prod super-admin runner (R14); Hikari pool sizing + Valkey/database TLS + sslmode is require (R10/R20); bounded streams MAXLEN/reaper (R16) | unit/IT ≥97.5%; boots SPRING PROFILES ACTIVE is prod; /version public, /actuator/info 401; one-time passcode log-grep zero; arm64 image runs | software-engineer, security, dba |
| **0 Foundations** | Single account; TF state (S3+Dynamo database lock); VPC (public + private-data subnets, **no NAT**); 1 key management service key (deny-deletion policy); container image registry (scan, immutable, lifecycle); SSM Secure text params w/ fresh values + Tink/keyed message authentication offline backup; GitHub OIDC roles (scoped, push/deploy split); Cloud Trail (free) | TF plan clean; params exist (no values in git/state); Cloud Trail live | devops, security |
| **1 Data** | managed relational database PG15 **t4g micro single-AZ**, gp3 20GB, **PITR 7–14d**, deletion-protect, prevent destroy, custom param group (rds force ssl is 1, timeouts), task-SG-only; Elasti Cache **Valkey t4g micro 1 node**, noeviction (R16), TLS+AUTH+at-rest key management service, prevent destroy | migrate empty database via db changelog-master implementation reference; TLS connect; noeviction confirmed; PITR restore to scratch | dba |
| **2 Compute + tunnel** | container service Fargate cluster (arm64); task def = backend + **cloudflared sidecar**, public subnet, **SG no inbound**, secrets from SSM, sized ephemeral storage; main service (2 tasks) + autoscale; Cloudflare Tunnel config (example com → localhost) | task healthy; tunnel up; https://example com/version correct SHA; no public inbound reachable | devops, solution-architect |
| **3 Edge** | Cloudflare Pages (SPA, VITE application programming interface BASE URL); Cloudflare DNS (@/www→Pages, api→tunnel); Cloudflare web application firewall managed + rate rules; Cloudflare Health Check | SPA serves; application programming interface via tunnel 200; web application firewall active | devops |
| **4 continuous integration/continuous deployment + canary** | GH Actions (backend buildx→container image registry→migrate→canary; frontend→Pages) via scoped OIDC; deploy lock (concurrency) + GH Environment reviewers; canary container service service + **Step Functions 2h bake** (alarm poll) + circuit breaker | dry-run: canary task joins tunnel → short-bake → promote; force alarm → rollback; deploy-gate SHA passes | devops |
| **5 Observability** | log groups (90d, key management service, IA for cold); log-metric-filter 5xx/ERROR alarms + managed relational database/Valkey alarms; SNS→email; Logs-ingest billing alarm | trip test alarm→email; Logs Insights on correlation Id returns rows | devops |
| **6 Cutover** | prod super-admin (one-off task, multi-factor authentication, bootstrap secret deleted after first change); Resend custom domain (REDACTED email, SPF/DKIM/DMARC in CF) + prod key; secret-backup verify; Cloudflare DNS go-live; rollback runbook | end-to-end smoke (login+multi-factor authentication, tutor/tutee flow, admin action, real @example com email); PITR drill; one-time passcode log-grep zero; origin unreachable except via tunnel | boss/tech-lead; security, dba sign-off |

**Staging** = the existing **local docker-compose** stack (the continuous integration regression lab already uses it); no
on-Amazon Web Services staging at this budget. A cheap throwaway can be stood up later if needed.

## Cost model (ap-southeast-1, single account, USD/mo)

ap-southeast-1 list rates, verified-approximate (re-check in the Pricing Calculator at build):

| Item | Assumption | ~$/mo |
|---|---|---|
| Fargate (arm64) | 2 tasks × ~0.5 vCPU/1 GB (backend + cloudflared sidecar) × 730h | $32–40 |
| managed relational database PostgreSQL | t4g micro single-AZ + gp3 20 GB + PITR/backups | $18–22 |
| Elasti Cache Valkey | t4g micro, 1 node | $12 |
| Cloud Watch Logs | ~15–30 GB (90d, IA cold), volume-controlled | $8–15 |
| Cloud Watch alarms | log-metric filters + ~10 alarms | $2–4 |
| key management service | 1 CMK + requests | $2 |
| SSM Parameter Store | standard Secure text params | $0 (free) |
| container image registry | image storage | $2 |
| S3 (TF state) + Dynamo database lock | trivial | $1 |
| Cloud Trail | first management trail | $0 (free) |
| Step Functions (canary) | 1 execution/deploy | ~$0 |
| Cloudflare Pages + Tunnel + Health Checks | static frontend + ingress + uptime | $0 (free tier) |
| Data transfer | egress via Cloudflare (free path); minor Amazon Web Services | $1–3 |
| **TOTAL** | | **~$80–105/mo** |

Comfortably under the **$200 ceiling**. **Optional add-ons if wanted within budget**: Guard Duty
(~$10–15), a 2nd cache node for HA (~$12), bump managed relational database to t4g small if the micro is tight (~+$15). Even
with all three you stay ~$130–150.

**Levers already applied (don't undo):** arm64 Graviton; **Valkey** (−20%); **no NAT** (public-subnet
egress, −$48); **no application load balancer** (cloudflared sidecar, −$24); **Cloudflare Pages+Tunnel** kill all Amazon Web Services
internet-egress + Cloud Front; **SSM not Secrets Manager** (free); single node/single-AZ; log volume
control + IA; **Cloudflare Health Checks not paid Synthetics**. Aurora Serverless v2 (~$73 floor) and
Elasti Cache Serverless (~$61 floor) are *more* expensive than these fixed micros — fixed wins.

## Top risks (residual)

1. **Single-AZ / single-node = no auto-failover.** An AZ or node failure → downtime + manual PITR
   restore (~10–15 min). Accepted trade for budget; reversible (flip Multi-AZ / add replica) when
   revenue arrives. Data durability is unchanged (PITR + backups).
2. **Expand-contract vs mixed-version bake** — additive-only; enum-writes two-release; database structure definitions linter +
   dba sign-off. Redis key shapes included.
3. **Liquibase databasechangeloglock** — discrete pre-deploy task must exit 0; serialized;
   stale-lock recovery checks pg stat activity before release Locks; CONCURRENTLY invalid-index
   detection in the pipeline.
4. **cloudflared connector is the only ingress** — run ≥2 connectors (one per task); alarm on tunnel
   health.
5. **Crown-jewel key custody** — tested offline restore + key management service deny-deletion; keyed message authentication rotation = re-index.
6. **No Guard Duty/full security baseline at launch** — Cloud Trail (free) stays; Guard Duty is a cheap
   re-addable (~$10–15) — recommend adding once live given personal information/Personal Data Protection Act. Flagged, founder's call.
7. **Single account** — prod isn't account-isolated from tooling; mitigate with tight IAM + a scoped
   deploy role. Multi-account re-addable later.

## Verification (end-to-end)

- **Pipeline**: deploy a no-op → canary task joins the tunnel, short bake, alarm-clean promote; force
  a synthetic 5xx → canary stopped, main untouched; concurrent push queues.
- **Edge/origin**: https://example com serves the SPA (Pages); https://example com/version
  200 via tunnel; the task has **no reachable public inbound** (SG denies; only the outbound tunnel).
- **Provenance**: /version git sha  equals  deployed commit.
- **Data**: PITR restore to a scratch instance; Valkey Evictions is 0.
- **Security**: one-time passcode log-grep zero; /actuator/info+/swagger blocked in prod; Cloud Trail emitting.
- **App smoke (cutover)**: super-admin login + multi-factor authentication; one tutor + one tutee happy-path; an admin action;
  a real email from @example com.

## Folded review conditions (still apply at this budget)

A0 backend hardening carries the bulk of the two prior reviews — all **code/config, $0**: R1 /version
gate · R2 (skip Prometheus scrape at launch; management port optional) · R3 migration exit-runner ·
R6 readiness group · R8 graceful shutdown · R9 db-scheduler task-name rule · R10 Hikari sizing ·
R11 one-time passcode DEBUG LOG is no · R12 forward-headers loopback-only (origin unreachable — risk largely
dissolved by the tunnel) · R13 key management service policy + keyed message authentication custody · R14 prod super-admin · R15 springdoc off ·
R16 Valkey noeviction + bounded streams · R18 enum two-release + database structure definitions linter · R19 lock recovery ·
R20 Redis/database TLS · R21 PITR + prevent destroy. Amazon Web Services-review items kept: A7 arm64; A3 OIDC scoping;
B-edge Cloudflare Pages+Tunnel. **Dropped as incompatible with budget (re-addable, documented):**
multi-account (A2), on-Amazon Web Services staging, Guard Duty/Config/Security Hub default (A4), Multi-AZ/managed relational database-Proxy
(A5/R17), 1+1 cache (E1), Cloud Front/application load balancer, interface endpoints, native blue/green (tunnel canary
instead).

## Critical files

- intuition-the private implementation folder — ARG/ENV GIT SHA + arm64 build
- application-prod implementation reference — pool, Redis/database TLS, forward-headers loopback, one-time passcode off, springdoc off
- application implementation reference — one-time passcode DEBUG LOG default; Hikari; prefer native enum types
- Security Config implementation reference — readiness group, /version permit All, swagger/actuator authz
- Showcase Health Indicator implementation reference — exclude from readiness group
- Email Queue Service implementation reference + stream consumers — MAXLEN/reaper, @Pre Destroy
- Super Admin Seeder implementation reference — separate prod bootstrap runner
- db changelog-master implementation reference — migration entrypoint / expand-contract gate
- api implementation reference — VITE application programming interface BASE URL build-time bake → Cloudflare Pages

## SDD / governance

On approval this becomes a versioned spec under intuition-spec/specs/<NNN-aws-production-deployment>/
(requirements + design + tasks + changelog), Boss-driven phase by phase, specialists dispatched per
phase, evidence under private evidence grouped by requirement. **Phase A0 ships first** (backend code/config the infra
depends on) via team-review→Boss, then the infrastructure as code phases. The deferred premium items (multi-account,
Multi-AZ, staging, Guard Duty) are tracked as a **"scale-up when revenue arrives"** follow-up, each a
one-flag Terraform change.
