> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

% InTuition SG — Amazon Web Services Deployment: Plain-English Overview, Architecture & Cost
% Spec 027 · ap-southeast-1 (Singapore) · 2026-06-25

# Document status

This is the dated2026-06-25 overview, corrected for authority/residency wording on2026-09-05. Costs and vendor plan entitlements are historical estimates, not current quotes. The controlling source is [approved-decisions md](approved-decisions.md), including manual-only pre-cutover deployment, later canary gates and measured recovery acceptance. Existing DOCX/PNG exports are historical illustrations of this earlier overview.

# What we're building, in one paragraph

Your tuition marketplace runs as a small, modern cloud setup. The **website** (the React app your
users see) is served straight from **Cloudflare** — fast, free, and global. The **brains** (the
Spring Boot application programming interface, the database, the cache) run on **Amazon Web Services in Singapore**. Provider processing/residency and cross-border records for Cloudflare/Resend still need the acceptance evidence in [approved-decisions md](approved-decisions.md); choosing an Amazon Web Services region alone does not establish them. Nothing on Amazon Web Services is exposed to the open internet: the application programming interface can only be reached
through a secure **Cloudflare tunnel**. We chose every piece to stay cheap and simple while you're
pre-revenue, and almost everything can be scaled up later by flipping one switch.

\newpage

# The architecture (picture)

[Private supporting material excluded (publication note)](../../docs/publication-notes.md) width is 100% 

\newpage

# How traffic flows (the important part)

There are **two separate journeys** a request can take. They never touch the same servers.

## Journey 1 — Loading the website (the React app)

1. A user opens **https://example com** in their browser.
2. Cloudflare answers directly and serves the **static React app from Cloudflare Pages** (HTML/JS/CSS).
3. **Amazon Web Services is not involved at all** for this step — the app files live on Cloudflare's global edge, so
   the page loads fast anywhere.

## Journey 2 — The app talking to the application programming interface (logins, posts, dashboards, etc.)

1. Once the React app is running in the browser, it makes application programming interface calls to **https://example com**.
2. The request hits **Cloudflare** first — which applies **TLS (encrypted web request protocol), DNS, and screening** (a
   per-IP rate-limit always; the full managed **web application firewall firewall** when on the Cloudflare **Pro** plan).
3. Cloudflare passes it down the **secure tunnel** to a small **cloudflared connector** that runs
   *inside* each of our Amazon Web Services containers.
4. The connector hands the request to the **Spring Boot application programming interface** running right next to it (on
   localhost). The Amazon Web Services containers do have a public IP for *outbound* internet access (to pull
   images, send email, reach the database) — but **no inbound port is open** (the firewall/security
   group denies all inbound), so that address is **unreachable from the internet**. The tunnel is the
   only way in.
5. The application programming interface does its work, reading/writing the **PostgreSQL database** and the **Valkey cache**,
   which live in a private network with **no internet access** (only our app can reach them).
6. The response travels back out the same tunnel to Cloudflare and on to the user.

**Why this matters:** because the connector "dials out" to Cloudflare (instead of us opening a door
in), the Amazon Web Services servers keep **every inbound port closed**. An attacker on the internet cannot reach the
database, the cache, or even the application programming interface directly — the only path in is through Cloudflare, which screens
requests first. This also let us delete two of the usual (and pricey) Amazon Web Services pieces — a public load
balancer and a NAT gateway — saving roughly **$56/month**.

*Two honest caveats:* (1) "no inbound" is enforced by a firewall rule on the containers, not by
physical isolation — it's an operational guardrail we should protect (a future automated check can
reject any change that opens a port). (2) *Outbound* is open (the app needs it), so this protects
against break-ins, not against a compromised app phoning out — standard for this kind of setup.

## Journey 3 — Shipping a new version (deployment)

1. You push code to GitHub. **GitHub Actions** builds the app image and stores it in **container image registry**.
2. It runs any **database migrations** first, then starts a **single "canary" container** alongside
   the live ones — Cloudflare sends it a slice of real traffic.
3. We **watch it for ~2 hours** (errors, health). If anything looks wrong it is **automatically
   rolled back** and the old version keeps serving. If it's clean, the new version is **promoted to
   the rest**. (You can shorten the 2 hours for an emergency fix.)
4. A new **website** version is just uploaded to Cloudflare Pages (instant, with instant rollback).

\newpage

# Decisions we've made (and why) — with cost

| What | We chose | Why (plain English) | $/month |
|---|---|---|---|
| Where it lives | Amazon Web Services Singapore + Cloudflare | Regional Amazon Web Services hosting; provider processing/transfer acceptance is recorded separately | — |
| App servers | 2 small containers (Fargate, ARM) | No servers to manage; ARM chips are ~20% cheaper; 2 = no single point of failure | ~$32 (at 1 GB) |
| Front door | Cloudflare Tunnel (no public load balancer, no NAT) | Nothing on Amazon Web Services is exposed; also removes two costly Amazon Web Services pieces | **saves ~$56** |
| Website hosting | Cloudflare Pages | Free, fast, global; instant rollback | ~$0 |
| Database | Managed PostgreSQL (smallest size, single copy) + daily point-in-time backups | Real managed database with recovery, at the cheapest viable size | ~$23–26 |
| Cache | Managed Valkey (one small node) | Speeds up the app + handles queues; one node fits the budget | ~$12–13 |
| Passwords & keys | Amazon Web Services SSM + one encryption key | Securely stored, free tier; the encryption key is locked against deletion | ~$2 |
| Monitoring | Cloud Watch logs + alarms + email | Know when something breaks; lean setup | ~$10–18 |
| Safe deploys | 2-hour canary with auto-rollback | New versions can't quietly break production | <$1 |

**Running cost of what's decided: about $86–108/month.**

\newpage

# Decisions still open (your call) — with cost

| Decision | Options | Recommended | Cost effect |
|---|---|---|---|
| **App memory** | 1 GB (current) vs **2 GB** per container | **2 GB** — 1 GB is tight for Java and risks crashes under load | +$32/mo for 2 GB |
| **Cloudflare plan** | Free vs **Pro** | **Pro** — turns on the managed firewall (web application firewall) + health checks | +$20/mo for Pro |
| **Database failover** | Single copy now vs **Multi-AZ** | Single now; add Multi-AZ when you have paying users | +~$25/mo later |
| **Cache failover** | 1 node now vs add a 2nd | 1 node now | +~$12/mo later |
| **Deeper security tooling** | Cloud Trail only vs add Guard Duty/Config/Security Hub | Add **soon after launch** (you hold personal data) | +~$25–50/mo |
| **1-year commitments** | Pay-as-you-go vs reserved | Reserve **after** spend is stable (~2–3 months) | saves ~$15–25/mo |
| **To deploy at all** | — | Provide Amazon Web Services login, a Cloudflare application programming interface token, GitHub settings, an alert email | no cost (blocking) |

\newpage

# Bottom line on cost

| Setup | Monthly (all-in) |
|---|---|
| Bare minimum (1 container, Free Cloudflare, no failover) | ~$60–75 |
| As-built lean | ~$86–108 |
| **Recommended launch** (2 GB app + Cloudflare Pro) | **~$140–160** |
| Recommended + full security tooling | ~$165–195 |

**Everything stays under your $200/month ceiling.** The two choices that move the number most are
**app memory** (1 GB → 2 GB, +$32) and **Cloudflare Pro** (+$20).

> Figures are approximate for Singapore at low traffic. We'll re-check in the Amazon Web Services Pricing Calculator
> with real usage before buying any 1-year reservations.
