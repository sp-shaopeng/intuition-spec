# InTuition SG — Next-Phase Product Roadmap

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-05-29 (Singapore time)
**Author:** Quinn (Product Designer)
**Audience:** Product, Business, Engineering — roadmap decision-makers
**Pairs with:** business-case-next md (Business owns ROI/financials; this doc owns product scope)
**Grounded in:** status-briefing md (verified code read), ROADMAP md (ADRs), M13-M14-M15-team-review md

> Read the briefing first. This roadmap is consistent with the briefing's verified facts.
> Where the briefing offered engineering's cost/value input, I make the product call.
> (Revised 2026-05-29 per Boss SEND BACK + the 16-person team review — see the change-bars
> in §2 and the new Epics M and N.)
>
> **⚠️ Status update (2026-05-30) — supersedes the M14b framing below.** The founder
> **waived** the M14b (Recent Requests) consent/legal gate; **M14b is now DEPLOYED
> (migration 0023)** alongside **M17 Marketplace user experience overhaul (deployed, no schema
> change)**. Item H's "M14b deferred-gated" wording and the §5 "Not doing → M14b" row
> are retained as the point-in-time product call but no longer reflect shipped reality —
> see ROADMAP md (baseline + ADR-012 update) and modules/INDEX md.

---

> ## 🔎 Status update (2026-06-02, Quinn) — post bug-fix batch + Hunter run
>
> A founder-reported defect batch (INT-001…014) shipped to the **dev** stack and was team-reviewed,
> tested, and live-smoked. **Product wins:** first-time visitors now reach sign-up from the login
> prompt; forgot-password emails contain a real "reset your password" link (the #1 confusion);
> tutor-detail pages no longer crash; the nav drops the dead "About" and gives admins a real entry;
> **pre-login browse now hides filters and shows only a capped teaser** (the founder's "no filters
> before login" call), with the filters unlocking after sign-in; sessions are a tidy 30 minutes.
>
> **Honest caveats this changes nothing about** — still dev-only, still no real email delivery beyond
> the sandbox, still no payments/prod hardening (see business-update md and the Tier-1 blockers).
>
> **New product-visible defects found (now on the eng plan, NOT yet fixed — issues/BATCH3-REMEDIATION-PLAN md):**
> - 🔴 **Admins can't cancel a match or reveal contact details without a 500** (INT-016) — breaks the
>   core concierge loop; it is the top fix.
> - 🟠 **A tutor whose listing was rejected can't delete it** (INT-015).
> - 🟠 **A tutor can be blocked from applying to a job** by the profile-completeness gate (INT-017) —
>   needs a product call on whether onboarding must force profile completion first.
>
> Product gate: the **free-market apply + admin-assign loop is not demoable end-to-end** until
> INT-016/INT-017 clear — treat that as the bar for any concierge-cohort trial.

---

> ## 🔎 Status update (2026-06-03, Quinn — PD) — fix batch shipped + new founder asks
>
> **The product gate above is CLEARED.** The INT-015…042 batch was team-reviewed (+ Boss SHIP),
> code-reviewed, and **deployed to dev**: **INT-016** (cancel/contact-reveal 500) and **INT-017** (seeded
> tutor can't apply) are FIXED and verified green — so the **free-market apply → choose-tutor → admin-assign
> → contact-reveal loop is now demoable end-to-end**, which is the bar we set for a concierge-cohort trial.
> Also shipped: **INT-015** (delete rejected post), and a marketplace user experience pass — role-honest nav, working
> interview/applicant lists, schedule-with-meeting-link, register polish.
>
> **New founder-requested features — added to the backlog as planned items (specced, not built):**
> - **INT-039 — Interview timeslot negotiation (M6 redesign):** tutee proposes ≤3 slots (15-min, ≤7 days)
>   → tutor accept / decline-with-reason → counter-propose → mutual agreement → admin finalises. Raises
>   booking conversion + reduces admin scheduling load. **Size L; own spec + team review before build.**
> - **INT-043 — Tutor onboarding / profile completion:** make qualification + experience optional at
>   sign-up; prompt profile completion after login (pairs with INT-017's apply-gate). Lowers sign-up
>   friction. **Size M.**
> - **INT-044 — Tutee invites a job to a specific tutor:** directed invitation (the inverse of choose-tutor),
>   a new demand→supply channel. **Size M; own spec.**
> - **INT-045 — Admin dashboard date selector / range for stats:** range-scoped metrics (7/30/90/custom)
>   vs all-time only — the analytics robustness the founder asked for. **Size M.**
> - **Epic O — WhatsApp-first notifications + phone verification** (already in this doc): the primary-channel
>   shift; blocked on a provider decision (Twilio vs Meta) + Personal Data Protection Act consent.
>
> **Sequencing call (PD):** these are **Tier-2 enhancements** — they do NOT jump the foundations queue
> (Epics 1–4) or the prod-readiness blockers in business-case-next md. Recommended order once foundations
> hold: INT-043 (unblocks onboarding/apply) → INT-039 (booking conversion) → INT-045 (ops insight) →
> INT-044 → Epic O (after a provider/Personal Data Protection Act decision).
>
> **Carry-overs (not product scope, but track):** **INT-047** — local Docker build wedged (deploy via a
> commit workaround; needs a Docker reboot); **INT-046** — frontend coverage at 95.82% (debt surfaced by fixing
> the test-runner deadlock; all tests pass).
>
> **PD/BD review of this update — Quinn (Product) + Dana (Business):** ✅ agreed. *Quinn:* the four new asks
> are real user-value, correctly logged as backlog with sizes and a sequence; none is greenlit to build
> without its own spec — no scope creep into this phase. *Dana:* none changes the launch-readiness math or
> the runway/commission hypotheses in business-case-next md; INT-045 (range stats) is a prerequisite for
> *measuring* the concierge experiment, so it earns priority once foundations hold. **No SEND BACK.**

## 1. Guiding Principle for This Phase

> **🚩 Scale posture (founder directive, 2026-05-29): we are building for many users, not an MVP.** This *reinforces* foundations-first — it does not soften it. "Only bites at scale" is no longer a reason to defer: the admin N+1 (I) and showcase the random operation are now real, rate-limiting (F) is mandatory not optional, backups mean **PITR/WAL (RPO minutes)** not just nightly dumps, staging/prod must be **horizontally scalable** (not a single host), and a **load/capacity baseline** is required before launch. The concierge cohort still validates willingness-to-pay, but manual ops is a learning bridge, not the architecture. See status-briefing md banner and project scale target memory.

**By the end of this phase, a real Singaporean can sign up, recover their password, apply to a job, and be notified — safely, on a floor that won't collapse — and that floor is built to hold many of them, not a handful.**

That outcome *is* user-facing value — it turns the product from a demo into something a real person can log into, recover, act in, and hear back from. Delivering it honestly requires fixing the foundation underneath it first, because the briefing's hard truth is that **our biggest risks are operational, not feature gaps**: a single disk failure wipes the company (no backups), every release is a hand-build on one laptop (no continuous integration/continuous deployment), the first time new code meets real data is in front of customers (no staging), we are blind to outages (no monitoring deployed), and a user who clicks "forgot password" today receives *nothing* (no email sender). So the phase pairs the foundation (backups, continuous integration/continuous deployment, staging, observability) with the two things a user actually feels — **email turned on** (the channel both our tutor and parent personas ranked #1) and **M15 shipped** (self-serve apply) — and explicitly defers payments, the ops console, and the landing showcase until the floor holds them up. The thirteenth built module, tracked as **M15**, is part of the payoff, not a sacrifice. We are not spending a phase on invisible plumbing; we are making the product *real and safe to use* — and the foundation is what lets it be both.

> **Cut-line / release valve.** This is five-plus epics with two external clocks (SES ~7d, Stripe 5–10d) and a heavy per-deliverable review tax — it can slip. If it does, drop in this order: **J → I → F** (the Could/Should fast-follows) and the M14-prep, *protecting the Musts (A, B, C, D, M, N, G) and the 5a deploy*. We do not cut backups, observability, instrumentation capture, or the Personal Data Protection Act erasure rule to hit a date.

---

## 2. Mo S Co W Table — Next-Phase Candidates

Every item is mapped to the user/business job it serves. "Must Have" is reserved for things the product **literally cannot launch safely without** — not things that would be nice.

> **Revised per Boss SEND BACK (2026-05-29).** Three changes from the first draft:
> (1) **M15 (E) demoted Must → Should** — by this table's own definition a Must is a
> launch-safety floor; M15 is a feature (the platform runs on admin matching today).
> (2) **Personal Data Protection Act retention/erasure (G) promoted Should → Must** — it is a regulatory
> obligation and gates the M15 deploy. (3) Two epics the roadmap's own decisions
> depend on were **added** as Musts: **Observability** (M) and **Instrumentation** (N).

| # | Candidate | Mo S Co W | One-line justification |
|---|-----------|--------|------------------------|
| A | **database backups + restore drill** | **Must** | Catastrophic, irreversible data-loss risk; cheapest item relative to blast radius. Nothing touches real user data until this exists. |
| B | **continuous integration/continuous deployment pipeline** | **Must** | A manual laptop build *will* ship a regression within weeks of launch; also the literal unblock for M15's coverage gate (the Docker-socket gate runs green in continuous integration). Force-multiplier. |
| C | **Staging environment** | **Must** | continuous deployment has nowhere to deploy and there is no safe place to test against real-shaped data; hard prerequisite for the restore drill and any live-mode payment smoke test later. |
| D | **Email delivery (build the Email Queue *consumer* → real sender)** | **Must** | Password reset is *silently broken* for real users today — a launch blocker. **There is no consumer today** (the queue is producer-only); this is net-new, not "wiring." |
| M | **Observability baseline (deploy monitoring + alerting)** | **Must** | The phase's whole thesis is "operate safely" — you cannot operate what you cannot see. The monitoring md spec exists but **nothing is deployed**. Hard-gates M15 prod writes. |
| E | **Ship M15 (Applications deploy)** | **Should** *(highest-value)* | Already built, reviewed, blockers fixed. A feature, not a launch-safety floor — the platform matches via admins today. The *only* honest source of the future applicant count. |
| N | **Instrumentation slice (persist the funnel events)** | **Must** | Analytics Event Publisher logs-and-loses events; M15's application submitted data is dropped at source. M14's trigger and the parent-count decision both depend on a funnel this phase must capture from day one. Cheap: one analytics events table + swap the stub body. |
| F | **M15 fast-follow: rate-limit on apply** | **Should** | Abuse/spam protection on a public write endpoint before real tutors hit it. Genuine integrity guard once self-serve apply is live. |
| G | **Personal Data Protection Act retention/erasure for applications (+ analytics events)** | **Must** | Regulatory (Personal Data Protection Act) obligation the moment we store real application records of real people; also gates the M15 deploy. Covers the new analytics events table too. |
| H | **M14 Landing Showcase — split: M14a Active Tutors (building) / M14b Recent Requests (deferred-gated)** | **Should** | Split by team review + Boss (2026-05-29). **M14a (Active Tutors)** is buildable now: tutor-SET "Available" flag (no surveillance), personal information-free, cached, **no applicant count**, ships with the anon→signup funnel (N) as a hard co-dependency. **M14b (Recent Requests)** is **DEFERRED, gated** on opt-in consent + privacy-notice update + legal sign-off. No count is surfaced in either slice. |
| I | **M15 fast-follow: admin applicant-list N+1 fix** | **Could** | Real but only bites at scale; admin-only surface with low volume at launch. Fix when applicant counts justify it. |
| J | **M15 fast-follow: "my applications" status timeline** | **Could** | user experience polish on the tutor side. The list works without it; iteration 2 with usage data. |
| K | **Payments (Stripe/Pay Now)** | **Won't (this phase)** — *start paperwork now* | Large build, gated on staging + 5–10 day Stripe verification. Admin "mark paid" works at low volume **but races today — see the idempotency fix below before the build lands**. **Action this phase: file Stripe verification day one.** |
| L | **M13 Ops Console + Secure personal information** | **Won't (this phase)** | Hard-gated by review + Boss: needs backups, fail-closed audit, blocking bulk-read cap, and the OPERATOR role — none of which exist. Most sensitive surface; sequenced last by design. |

**Secrets management** (vault for continuous integration/SES/backup-encryption/Stripe keys; rotation for the signed session token signing key and the Tink encryption keyset) is a **Must** that runs across Epics 2–4 — plaintext secrets sit in docker-compose implementation reference today and there is no production secret store.

**Pre-payments code fix (file now, ship before the Stripe build):** the pay commission operation has no optimistic lock / idempotency key — it races today and Stripe webhooks are at-least-once. Add a database-level guard with the Stripe paperwork, not after the integration.

**Feature-creep flags I am raising now:**
- *"While we ship M15, let's also add the status timeline / N+1 fix / notifications."* No. The briefing already records that an out-of-scope notification was cut during M15 review. Ship the thin slice; fast-follows F–J are separately prioritized, not bundled.
- *"M14 gives us social proof, let's build it alongside M15."* The applicant-count proxy is dead (both Platform Users called it "a lie") and **no count is shown in M14 at all**. The team review + Boss split M14: **M14a (Active Tutors)** is buildable independently of M15 (it shows no count) but must ship with the anon→signup funnel (N) to be measurable; **M14b (Recent Requests)** is deferred until opt-in consent + privacy notice + legal sign-off clear. Do not build M14b alongside M15.
- *"Payments is the revenue model, it must be a Must."* Revenue depends on it; *launching safely* does not. We have a working manual path. This is the classic "important business goal" vs. "launch blocker" confusion. Won't-this-phase, paperwork now.

---

## 3. Sequenced Epics — Phase Next (in order)

Five epics. The first three are the foundation and run as much in parallel as the team allows; epics 4–5 depend on them. Sizes use ROADMAP convention (S ≤ 2d, M ≤ 1wk, L ≤ 2wk).

### Epic 1 — Don't Lose The Company: database Backups + Restore Drill
- **Goal:** No single failure (disk, fat-fingered command) can permanently destroy user data.
- **Size:** S (backups) + the drill rides on Epic 3's staging.
- **Dependencies:** None to start backups; restore *drill* needs Epic 3 (staging) to restore into.
- **Done when:** A nightly encrypted backup lands off-box (encrypted with a key held **independently of the app's column-encryption keyset**), a **dead-man's-switch alerts if a fresh backup doesn't land**, and a **proven, recurring, integrity-asserted restore** succeeds under the stated RTO — "Postgres started" is not success; the restore must pass row-count reconciliation, FK-orphan probes, a Liquibase-checksum match, and a canary business-fact query, run on a schedule (monthly minimum, in continuous integration) and logged. (Maps to ROADMAP #34; RPO ≤ 24h for closed beta — **PITR/WAL before paid launch**, see §5; RTO ≤ 4h.)
- **personal information / Personal Data Protection Act (mandatory):** the restore-**verification** drill runs into a **locked, ephemeral, prod-trust instance that is destroyed after**, NOT the shared developer staging. The drill does **not** seed staging with real personal information (see Epic 3).

### Epic 2 — Stop Shipping Blind: continuous integration/continuous deployment Pipeline
- **Goal:** Every change is automatically tested before merge; merges deploy without a human running a laptop build. Also unblocks M15's coverage gate that couldn't run locally.
- **Size:** M.
- **Dependencies:** Epic 3 (continuous deployment needs somewhere to deploy); continuous integration half can land first.
- **Done when:** A pull request with a failing test or sub-threshold coverage cannot merge (proven by an actual blocked red pull request), **and** the **Testcontainers integration suite spins up Postgres+Redis and the JaCoCo 97.5/97.5 gate passes in the continuous integration runner** (Docker-in-continuous integration, Java 21 pinned) — this specific green run, not "a red pull request is blocked," is what retires M15's locally-unrunnable gate. Includes a **rollback path** (previous-image redeploy) and an **expand/contract migration rule** enforced as a merge gate. (Maps to ROADMAP #6.)

### Epic 3 — A Place To Be Wrong Safely: Staging Environment
- **Goal:** A production-shaped, production-isolated environment to deploy and test against before customers do.
- **Size:** M.
- **Dependencies:** Cloud provider account + DNS (external, file early).
- **Done when:** staging example.com/actuator/health returns 200 with all components UP; the staging database's isolation is **proven by an active negative test** (a connection attempt from staging→prod database and prod→staging database is refused/times out, logged) — asserted isolation is not tested isolation; **staging is seeded with synthetic or irreversibly-masked data, never a raw prod personal information dump**; staging email/payments run in **SES sandbox / Stripe test mode only** (a test asserts prod-SES is unreachable from the staging profile); and a merge to main auto-deploys within 10 minutes. (Maps to ROADMAP #6a.)

### Epic 4 — Actually Reach Users: Email Delivery
- **Goal:** Build the missing email **consumer** and a real sender so transactional email (starting with password reset) actually arrives.
- **Size:** **L-heavy** — this is **net-new, not "wiring."** Email Queue Service is **producer-only**; there is no consumer in the codebase. Scope: a Redis Streams **consumer group** with explicit ACK, a **failed-message queue** with redelivery, **idempotency** (password-reset dedupe under at-least-once delivery), templates, SES integration, and **bounce/complaint handling**.
- **Dependencies:** Epic 3 (test in staging/SES sandbox first), Epic 2 (deploy safely), Epic M (observability before real sends); external — Amazon Web Services SES production access (~7d lead, **file day one**) + SPF/DKIM/DMARC DNS before first send.
- **Email is the #1 user-ask:** both platform users (tutor + parent) independently ranked notification *above* every feature — today the platform cannot reach anyone. Sequence email as early as the foundation allows; surface SMS/WhatsApp as a future-horizon item.
- **Done when:** "forgot password" delivers to a real inbox within 30s; **a redelivered message sends exactly once** (idempotent); restarting the consumer loses no un-acked messages (consumer-group ACK proven); the launch-critical event emails (match activated, interview scheduled/rescheduled, commission due) send while honouring opt-out — **and security mail like password reset is NOT suppressed by a marketing opt-out**; and **delivery/bounce/complaint/failed-message queue-depth metrics are emitted with alerts at 60–70% of SES suspension thresholds** (email observability is part of *this* epic). (Maps to ROADMAP #1 + #23.)

### Epic M — See What We Operate: Observability Baseline
- **Goal:** Deploy the monitoring that today only exists as a spec, so an outage pages a human instead of waiting for a user complaint.
- **Size:** M. Rides with Epic 3 (no point scraping before staging exists).
- **Dependencies:** Epic 3 (a target to scrape).
- **Done when:** Prometheus + Alertmanager + Grafana are deployed; the app **self-tags the application label via a Meter Registry Customizer** (so a scrape-config typo can't silently disable every alert); an external uptime check + dead-man's-switch run on **independent channels** (not the same SES we're monitoring); and **at least one alert is proven to fire end-to-end in staging** (rule → Alertmanager → a human inbox, with the runbook link resolving to real content), including the two traps (expired scrape signed session token, missing application label). **Hard-gates Epic 5: M15 does not take prod writes until its apply endpoint has a dashboard + a 5xx alert proven to fire.**

### Epic N — Capture What Happened: Instrumentation Slice
- **Goal:** Stop dropping the funnel. Persist the analytics events the stub currently logs-and-loses, so M15/email success is measurable and M14's decision is data-driven.
- **Size:** S. Rides with Epic 5 (capture apply data from day one of real traffic — it can never be recovered retroactively).
- **Dependencies:** none blocking; pairs with Epic 5.
- **Done when:** a single append-only analytics events Postgres table exists (Liquibase changeset); Analytics Event Publisher publish writes to it instead of log info; match activated and commission marked paid events are emitted (the funnel's missing middle and revenue terminus); email send/delivered/bounced events are emitted in Epic 4; and the table is covered by the Personal Data Protection Act erasure rule (G). An ADR fixes the publisher's **async + failure contract** now so the 11 existing call sites don't churn later. **Do NOT** stand up Segment/Amplitude/a warehouse this phase — the Postgres table is the MVP.

### Epic 5a — Ship What We Built: M15 Applications Deploy
- **Goal:** Tutors can apply to/withdraw from jobs in production; admins see real applicants. The door to self-serve matching and the **sole** source of any future applicant count.
- **Size:** S — the code is built and reviewed; this is genuinely "press go" once the foundation holds.
- **Dependencies:** Epics 1, 2, 3, M, N (backups before real data, continuous integration green gate, staging, observability gate, instrumentation to capture day-one apply data).
- **Done when:** migration 0018 is applied to the live database via the expand/contract pipeline with a **rehearsed rollback**; and a **post-deploy smoke test against the live database passes** — apply, withdraw, and **re-apply-after-withdraw** (the partial-unique/reactivate path that never ran on the live schema), plus a concurrent-double-apply check (one wins, the other a clean conflict, not a 500), and the admin applicant list renders. No applicant *count* is surfaced to parents here — that decision lives in §4.

### Epic 5b — M15 Integrity Fast-Follows (net-new code, separate estimate)
- **Goal:** The integrity/compliance guards that real applicant data requires.
- **Size:** M — these are **net-new code**, each carrying the full team-review + 97.5% coverage tax, not part of the 5a deploy.
- **Done when:** a rate limit protects the apply endpoint — **N applies/window → 429, verified by test** (F, Should); and an **application retention/erasure rule** is enforced — aged withdrawn/rejected rows purged by a job, and a user-erasure request cascades to their applications **and analytics events**, verified by a seeded-aged-row test (G, **Must** — Personal Data Protection Act). 5b gates "M15 carries real applications," even though the 5a deploy can land first.

> **M14 (Landing Showcase) was split by team review + Boss (2026-05-29).** **M14a
> (Active Tutors)** does **not** show an applicant count and does **not** depend on M15;
> it is implementation-ready (see modules/landing-showcase/) and its only hard
> co-dependency is the anon→signup funnel (Epic N) shipping alongside it, plus durable
> Redis. **M14b (Recent Requests)** is **deferred-gated** on opt-in consent + privacy
> notice + legal sign-off. The earlier "real count from M15" framing is dropped — no
> count is surfaced in either slice. Triggers in §5.

---

## 4. Recommendations on the Open Product Decisions

The briefing raises five forks. Business will pressure-test these — here are my decisive answers with rationale.

### Decision 1 — Should a parent see "N tutors applied" to their job? To whom, and when?
**Recommendation: Not in this phase. When we do show it, show it only to the parent who owns the job, only above a floor (e g. "3+ interested"), and never a precise low number — and never a proxy.**
- *Rationale:* It is settled (ADR-013) that any count must come from real M15 applications, and M15 only just ships this phase — there is no meaningful volume yet, so showing "1 tutor applied" or "0" is discouraging, not reassuring. A platform-user (Wei) said high counts discourage applicants; the parent persona (Priya) said a *real* count reassures her. Both are true, which means the count is a **calibrated reassurance signal, not a leaderboard.** Defer the surfacing decision until M15 has produced real data, then ship it to the job owner as a coarse, floored band. The honest interim answer is: collect the data now (Epic N), show nothing yet. **Quantified trigger to flip it on:** median applications-per-job ≥ 3 across ≥ 20 live jobs — then A/B the floored band ("3+ interested") shown only to the job owner. That distribution is itself unknowable without the instrumentation slice (Epic N), which is why N is a Must this phase.

### Decision 2 — When do we turn on real email?
**Recommendation: This phase — it is a launch blocker, the highest-priority feature work after backups.**
- *Rationale:* "Forgot password" returning silence is not a missing feature, it is a *broken core flow*. Self-serve onboarding is impossible without it. This is not a "should we" — it is Epic 4, Must. The only sequencing nuance is that it rides on staging (Epic 3) so we don't test SES against production first.

### Decision 3 — When do we build payments?
**Recommendation: Not this phase — but file Stripe business verification on day one of this phase.**
- *Rationale:* Payments is large and externally gated (staging prerequisite + 5–10 business-day Stripe verification). We have a *working* manual workaround: admins mark commission paid, which is fine at the low volume of a freshly-launched marketplace. Building payments now would consume the whole phase and still leave us unable to operate safely. The one thing that *is* this-phase is the **paperwork** — verification lead time is the real bottleneck, so start the clock now and let the build land next phase on top of a real staging environment. This separates "we're not building it" from "we're not preparing for it."
- *Interim control (required):* manual "mark paid" is an audit/trust hole today (a staffer can mark paid what was never collected, and the pay commission operation has no idempotency guard). For the concierge bridge: add a **logged note + two-person check on manual Mark-Paid**, and file the pay commission operation database-level idempotency fix with the Stripe paperwork so the eventual webhook lands on a race-safe path.

### Decision 4 — Foundations (backups, continuous integration/continuous deployment, staging) before more features?
**Recommendation: Yes, unambiguously. This is the spine of the whole phase.**
- *Rationale:* Engineering recommended it; the briefing's own headline is that the three highest-risk gaps are among the cheapest to build. The velocity cost is real and I am owning it openly in the guiding principle — but the alternative is launching onto a floor that can drop the company. A product with great features and no backups is a liability, not a product. Foundations first.

### Decision 5 — How admin-driven should matching stay?
**Recommendation: Keep curated admin matching as the spine; let M15 add tutor-initiated *applications* as a signal into that flow — not yet full self-serve auto-matching.**
- *Rationale:* Curated matching is a marketplace-quality moat early on (it's why the platform demos trustworthy). M15 is correctly scoped as "tutor applies, admin still decides" — applications feed the admin, they don't bypass them. That's the right amount of self-serve for this phase: we open the door without abandoning the quality control that differentiates us. Pushing to full self-serve auto-matching is a future bet to make *after* we see application volume and quality, not a commitment to make now.

---

## 5. What We Are Deliberately NOT Doing This Phase

Each "no" has a written trigger that flips it to a "yes."

| Not doing | Why not now | Trigger that changes it |
|-----------|-------------|-------------------------|
| **Payments integration (Stripe/Pay Now, ROADMAP #4a–c)** | Large; staging-gated; manual "mark paid" works at launch volume. | Staging is live (Epic 3 done) **and** Stripe verification is approved **and** monthly commission volume makes manual marking a real ops burden. Paperwork starts *this* phase regardless. |
| **M13 Ops Console + Secure personal information (ROADMAP/ADR-011)** | Hard-gated: needs backups, fail-closed audit, blocking bulk-read cap, and a non-existent OPERATOR role. Most sensitive surface on the platform. | Backups verified (Epic 1) **and** the ops team has a demonstrated, repeated need to look up real contact details that the existing admin tools can't safely serve. Then build the thin decrypt-on-the-fly slice per ADR-011, last. |
| **M14b Recent Requests strip (ROADMAP/ADR-012)** | SEND BACK by team review + Boss (2026-05-29): publicly broadcasting tutee request data with no opt-in consent, no privacy-notice update, and no legal sign-off is not acceptable. | Opt-in consent (tutee posts show on landing, default FALSE, no retroactive publication) **and** re-consent path **and** privacy-notice update at point of posting **and** legal/founder sign-off all clear. Then build the stripped-down (region + subject + level + day-granularity recency only) personal information-free card. **(M14a Active Tutors is NOT deferred — it is building; see below.)** |
| **M15 fast-follows I (admin N+1) and J (status timeline)** | Real but non-blocking; one is scale-only, one is polish. | Applicant volume per job makes the admin list slow (I), or tutor feedback/usage data shows the flat list confuses applicants (J). Then pull into the next phase. |
| **Net-new Tier 2+ features (calendar sync, chat, badges, Post GIS, subscriptions, etc.)** | The whole point of this phase is *not* building more features until the floor exists. | Foundations (Epics 1–4) are done and M15 is live and stable. Then re-prioritize Tier 2 against real usage — with data, not excitement. |

---

## Cross-reference

ROI, cost, revenue timing, and the business case for accepting reduced feature velocity this phase live in **business-case-next md** (Business). This document deliberately stops at product scope and sequencing. Where the two disagree on payments timing or the velocity trade, that is a Product/Business decision to resolve jointly — the recommended position here is the foundations-first sequence in §1 and §3.

---

## Epic O — WhatsApp-first notifications + phone verification (founder directive, 2026-06-03)

**Founder intent:** the **primary** notification channel to users should be the **phone** (WhatsApp, falling back to SMS) — Singapore users live on WhatsApp. (1) Add a **phone-verification** step (most likely a WhatsApp one-time passcode). (2) Put the whole thing behind a **switch (feature flag)**. (3) Eventually route **all** notifications via WhatsApp **instead of email — EXCEPT the password-reset flow**, which stays email-only (security/deliverability; password resets must not depend on a WhatsApp template approval or a wrong number).

**Fits existing infra (low architectural risk):** email today is async — Email Queue Service (Redis Streams) → Email Consumer Service → Resend Email Client. A WhatsApp/SMS channel mirrors this exactly: a messaging queue → consumer → Whats App Client/Sms Client. User phone already exists (plaintext, nullable); we add phone verified + a verification-code table (Liquibase — coordinate the migration number with M19 OAuth's 0031 reservation).

**Phased plan (each phase independently shippable):**
1. **Phase O1 — Phone verification only.** phone verified BOOLEAN, one-time passcode send/verify endpoints, rate-limited, code hashed + expiry period. Provider sends the one-time passcode. No notification change yet. The switch defaults OFF.
2. **Phase O2 — Channel abstraction + switch.** A Notification Channel strategy (EMAIL | WHATSAPP) and a global flag notifications channel (+ optional per-user preference). Password-reset is hard-pinned to EMAIL in code regardless of the flag.
3. **Phase O3 — Cut over.** With the flag ON, transactional notifications go WhatsApp-first (SMS fallback on undelivered), email retained as fallback for unverified phones. Per-channel delivery metrics.

**Decisions the founder must make (block implementation — provider account, cost, compliance):**
- **Provider:** Meta WhatsApp Cloud application programming interface (official; template pre-approval; cheapest at scale) vs **Twilio** (WhatsApp **and** SMS unified, fastest to integrate, pricier) vs Message Bird/Vonage.
- **one-time passcode channel:** WhatsApp one-time passcode (needs an approved template, slower to greenlight) vs **SMS one-time passcode** (simplest, universal, no template gate) for the *verification* step specifically.
- **Compliance (Personal Data Protection Act, SG):** WhatsApp business-initiated messages need opt-in + approved templates; consent copy at registration + an opt-out. Transactional vs marketing categorization affects cost and rules.
- **Cost:** per-message pricing × volume — Business owns the ROI line (add to business-case-next md).

**Why not implement blind now:** external paid provider, application programming interface secrets, and a SG-Personal Data Protection Act consent/legal surface. Recommend building **Phase O1 (verification) behind the OFF switch first** once a provider is chosen, prove deliverability, then O2/O3. Mandatory /team-review + Boss before the O3 cutover, since it changes how every user is contacted.  using Recorded from founder directive pending provider decision._

> **⚠️ Status update (2026-06-08) — Epic O largely SUPERSEDED by INT-082/084/085 (built, OFF, dev).**
> O1 (phone verification via WhatsApp one-time passcode) is **BUILT** (INT-082, Meta Cloud application programming interface chosen, migration 0047,
> toggled OFF behind PE gates). The **O3 "route ALL notifications via WhatsApp" cutover is DROPPED** —
> founder cost decision 2026-06-07: WhatsApp messaging is pricey, so the delivered set is narrowed to
> **INTERVIEW SCHEDULED only (both parties — INT-085)** + the one-time passcode, config-driven
> (whatsapp notifications.events); re-expansion is config-only but requires founder cost approval +
> consent-copy lockstep. **Password reset stays email-only, permanently carved out** (as O always said).
> SMS fallback remains out of scope. Enablement still gated on PE-0…PE-3 (Meta approval, phone
> normalization, send-ceiling decision, dropped/spend alerting + runbook).
