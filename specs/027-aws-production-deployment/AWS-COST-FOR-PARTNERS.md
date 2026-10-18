> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

% InTuition SG — What the Platform Costs to Run
% A plain-English summary for the founding team
% 2026-06-25

# The big picture

InTuition runs on two well-known services: **Cloudflare** (the "front door" that shows our website
to customers and keeps attackers out) and **Amazon Amazon Web Services in Singapore** (the "engine room" that runs
the app and safely stores all our data — kept in Singapore to meet local privacy law).

We deliberately built it **lean**: a small, modern setup that runs the real product reliably while
we're still pre-revenue, and that we can scale up later as paying customers arrive — without
rebuilding anything.

**The whole thing costs roughly US$90–145 a month** to run — comfortably under our US$200 budget.

\newpage

# The picture (and the price of each part)

[Private supporting material excluded (publication note)](../../docs/publication-notes.md) width is 100% 

\newpage

# What we pay for each month — and what it buys us

| Part of the system | What it does for the business | Monthly cost |
|---|---|---|
| **Cloudflare — front door & website** | Shows our website to customers fast, anywhere in the world; blocks attacks and spam. | **$0** (or **+$20** to switch on the full firewall) |
| **The application** | The actual product — logins, posting jobs, matching tutors. We run **two copies** so it stays up if one has a problem. | **$32–64** |
| **The database** | Stores every account, post, match and review — with **daily backups** we can restore from. | **$25** |
| **Fast memory (cache)** | Makes the app feel quick and handles behind-the-scenes message queues (emails, notifications). | **$12** |
| **Safety & visibility** | Backups, encryption keys that protect personal data, and monitoring that emails us if something breaks. | **$15** |
| | **Total to run the platform** | **≈ $90–145 / month** |

Everything is **pay-as-you-go** — no upfront payment, no long-term contract. We only pay for what we use.

\newpage

# The two choices that affect the bill

Almost everything above is fixed and small. Just **two decisions** move the number, and both are easy
to change later:

### 1. How much memory the app gets — *the main one*
- We can run the app on **less memory ($32/mo)** or **more memory ($64/mo)**.
- More memory is **safer** — the app is built in Java, which needs breathing room, or it can crash
  when lots of users are on at once.
- **Recommendation: pay the extra ~$32/mo for the headroom.** Cheap insurance against downtime.

### 2. Cloudflare's full firewall — *optional*
- The free plan already blocks basic abuse. The **$20/mo** plan adds a proper security firewall
  (recommended once we have real users, since we hold personal data).
- **Recommendation: turn it on at launch (+$20/mo).**

With both recommendations, we're at about **$140–160/month**.

\newpage

# What grows the bill later (and what doesn't)

- **Most of the cost is fixed** — it does **not** go up just because time passes.
- The bill **only grows as customer traffic grows** — more customers → the app runs more copies. That's
  a good problem (it means revenue), and it scales smoothly.
- When we're earning steadily, we can **pre-pay for a year** on the big pieces and **save ~$15–25/month** —
  but we'll only do that once usage is stable, so we never lock in the wrong size.

# Three ways we could run it

| Option | What you get | Monthly |
|---|---|---|
| **Bare minimum** | One copy of the app, free Cloudflare, no spare database | **$60–75** |
| **Recommended** | Two copies + extra memory + full firewall (stable, secure) | **$140–160** |
| **Recommended + extra security tools** | Adds deeper threat monitoring (sensible soon after launch) | **$165–195** |

All three stay **under our $200/month budget.**

# Bottom line

- To launch properly and sleep at night: **about $140–160 a month.**
- We can run as low as **~$60–75** if we ever need to tighten the belt (with less safety margin).
- No surprises, no lock-in, and the cost only climbs as customers — and revenue — do.

> All figures are current Amazon/Cloudflare pricing for Singapore at low traffic, and are approximate.
> We'll confirm exact numbers once real usage is flowing.
