# InTuition SG — Amazon Web Services Cost Analysis (current Terraform setup)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-06-25 · **Spec:** 027-aws-production-deployment · **Region:** ap-southeast-1 (Singapore)
**Basis:** the actual committed Terraform defaults (not the planning estimate). Always-on, low/pre-revenue traffic.
**Prices:** current Amazon Web Services list, **approximate** — confirm in the Amazon Web Services Pricing Calculator before committing to spend.

---

## 1. Configuration being priced (from the Terraform)

| Resource | Setting (Terraform default) |
|---|---|
| Compute | container service Fargate, **2 tasks** (desired count is 2, autoscale 2→4), each **512 CPU (0.5 vCPU) / 1024 MB**, **arm64**, public IP, + cloudflared sidecar in-task |
| Database | managed relational database **db t4g.micro**, **single-AZ**, gp3 **20 GB** (autoscale→50), **14-day PITR**, encrypted |
| Cache | Elasti Cache **Valkey 7.2**, **cache t4g.micro**, **1 node**, TLS+AUTH, noeviction |
| Edge | Cloudflare **Tunnel** (no application load balancer/NAT) + **Pages** (static SPA) + DNS |
| Secrets | SSM Parameter Store Secure text + 1 key management service CMK |
| Observability | Cloud Watch logs (90-day), 15 alarms, 1 dashboard, SNS (alias/aws/sns) |
| continuous integration/continuous deployment | GitHub Actions OIDC → container image registry → Step Functions canary |

---

## 2. Itemized monthly cost — Amazon Web Services

| Item | Assumption | Low | Expected |
|---|---|---:|---:|
| Fargate (arm64) | 2 × 0.5 vCPU / 1 GB, 24×7 (730h) | $30 | $32 |
| **Public IPv4** | 2 task ENIs × $0.005/h (Amazon Web Services charges since 2024) | $7 | $8 |
| managed relational database db t4g.micro single-AZ | instance ~$0.026/h + gp3 20 GB + PITR | $21 | $26 |
| Elasti Cache Valkey t4g micro | 1 node ~$0.017/h | $12 | $13 |
| Cloud Watch Logs | ~5–20 GB ingest (90d; one-time passcode off, INFO level) | $7 | $14 |
| Cloud Watch alarms | 15 × $0.10 + 2 custom metric filters | $2 | $2 |
| key management service | 1 CMK + decrypt requests | $2 | $2 |
| Data transfer | tunnel egress to Cloudflare + inter-AZ | $2 | $5 |
| container image registry | ~8 GB images | $1 | $2 |
| Cloud Trail + S3 (TF state) + Dynamo database lock | 1st trail free; storage | $2 | $3 |
| SSM Parameter Store | Standard Secure text | $0 | $0 (free) |
| SNS (alias/aws/sns) | <1,000 alarm emails/mo | $0 | $0 (free) |
| Step Functions | ~15 transitions/deploy, few deploys/mo | $0 | <$1 |
| **Amazon Web Services subtotal** | | **~$86** | **~$108** |

---

## 3. Cloudflare (separate bill)

| Plan | Cost | What you get / lose |
|---|---|---|
| **Free** | **$0** | Pages, Tunnel, DNS, [REDACTED AUTHORIZATION] rule. **Lose** managed web application firewall (OWASP), standalone Health Checks, region-pinned checks → set enable managed waf is no. |
| **Pro** | **$20/mo per domain** | Adds managed web application firewall ruleset, Health Checks, analytics. |

---

## 4. Total scenarios

| Scenario | Monthly all-in |
|---|---|
| **Lean** — Cloudflare Free, tasks as-authored (1 GB) | **~$86–108** |
| **+ Cloudflare Pro** (managed web application firewall) | **~$106–128** |
| **Realistic** — Pro + Java memory bump to 2 GB (see §5) | **~$140–160** |
| **Hard floor** — 1 task, Free CF, 1 GB (no HA) | **~$60–75** |

All scenarios sit under the **$200/mo** ceiling.

---

## 5. Two items the raw config under-prices (important)

1. **Java memory — the biggest variable.** 0.5 vCPU / **1 GB** per task is *tight* for Spring Boot 3.3
   (Tink encryption, Redis-stream consumers, db-scheduler, Java persistence mapping) **plus** the cloudflared sidecar (~128 MB),
   leaving the JVM ~800 MB — a real **OOM risk under load**. The likely production setting is
   **1 vCPU / 2 GB** (task cpu is 1024, task memory is 2048), which **~doubles Fargate to ~$64/mo** and lifts the
   total to ~$140–160. Recommendation: budget for 2 GB; validate heap under load at cutover before trusting 1 GB.
2. **Public IPv4 (~$8/mo)** is inherent to the no-NAT public-subnet design — but still far cheaper than the
   NAT gateway (~$32/mo + data processing) it replaces. The design choice holds.

---

## 6. What scales the bill as traffic grows

- **Traffic** → Fargate autoscales 2→4 tasks (~$64–128) + more public IPs + more egress.
- **Log volume** → Cloud Watch ingestion is the classic silent 10×; the **log-ingest billing alarm** (Phase 5) catches it.
- **database growth** → storage autoscales to 50 GB; move to db t4g.small if CPU credits throttle (the **CPU Credit Balance alarm** warns first).
- **Cache pressure** → any Evictions (alarm fires on ≥1) means bump the node or add memory.

---

## 7. Levers

**Lower now (pre-revenue):**
- Run **1 task** (desired count is 1): −$16–32 + −$4 IPv4 → ~$60–75, but loses tunnel-connector HA (single point of failure). *Not recommended.*
- **Cloudflare Free** (skip managed web application firewall): −$20.
- **30-day log retention** for non-audit logs (if Personal Data Protection Act allows): small now, grows with traffic.

**Lower once spend is stable (post-revenue, ~30% off the big lines):**
- **managed relational database Reserved Instance** (1-yr): ≈ −$7–9/mo
- **Elasti Cache Reserved Node** (1-yr): ≈ −$4/mo
- **Fargate Compute Savings Plan** (1-yr): ≈ −$6–13/mo
- Defer all three until the sizes are proven (don't reserve something you'll resize).

**Already applied (structurally cheap — don't undo):** arm64 Graviton (−~20%); Valkey over Redis (−~20%);
**no NAT** (cloudflared sidecar in a public subnet, −~$32/mo); **no application load balancer** (tunnel, −~$24/mo);
**Cloudflare Pages + Tunnel** eliminate Cloud Front and most Amazon Web Services egress; **SSM** instead of Secrets Manager (free);
single-AZ + single cache node; alias/aws/sns (free); S3 gateway endpoint avoided entirely (no interface endpoints).

---

## 8. Bottom line

- **Realistic steady-state: ~$110–160/mo all-in.** The spread is two decisions: **task memory** (1 GB risks OOM →
  likely 2 GB, +$32) and **Cloudflare Pro** for managed web application firewall (+$20).
- **Hard floor ~$60–75/mo**; **ceiling stays under $200** even at 2 GB + Pro + headroom.
- **Deferred HA upgrades** (each a one-flag Terraform change when revenue justifies): managed relational database Multi-AZ (+~$25/mo for
  this instance class — auto-failover), a 2nd Valkey node (+~$12), Guard Duty/Config/Security Hub (+~$25–50),
  Container Insights (+~$10–20).

> Caveat: every figure is current-list **approximate** for ap-southeast-1 and assumes low traffic. Re-cost in the
> Amazon Web Services Pricing Calculator with your expected request/log volumes before committing to Reserved/Savings purchases.
