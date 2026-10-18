> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 006
slug: whatsapp-interview-reminders
title: WhatsApp interview reminders
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED            # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: software-engineer         # the engineer who owns this change (requirements-engineer for cross-cutting/product specs)
created: 2026-06-21
updated: 2026-09-05
superseded-in-part-by: 007   # REQ-003/004 reminder *mechanism* (rolling sweep + reminder sent / migration 0061) replaced by spec-007's per-instance db-scheduler one-time job (2026-06-21); user-facing behaviour identical. REQ-001/002/006/007/008 still stand.
---

> **2026-09-05 requirements correction (documentation only).** The original sweep/flag/re-arm tasks are historical and superseded by 007. The authorized event expansion carries the required frontend consent-copy/test work before enablement; WhatsApp remains OFF.
> The status and shipment/evidence entries below record the earlier implementation. This revision does not
> certify that the corrected contract is implemented, tested, or deployed; follow-up verification remains open.


# WhatsApp interview reminders

When an admin schedules an interview **with a meeting link**, both the tutor and the tutee should receive a WhatsApp
message carrying the **interview timing (Singapore time) + the join link**, and then a second WhatsApp **reminder 24
hours before** the interview — so neither party forgets nor has to dig the link out of an in-app notification. This
spec governs three things the platform does not do today: (1) a **parameterized** WhatsApp notification (today's
INTERVIEW SCHEDULED send is param-less / static-body and fires regardless of link), (2) **link-gating** the notify
so a scheduling with no meeting link enqueues no WhatsApp, and (3) a new **24h-before reminder** driven by an
idempotent Shed Lock scheduled sweep keyed on a new reminder sent flag. It reuses the existing transactional-outbox →
Redis-stream → consumer pipeline and the interview-expiry-sweep idempotency pattern wholesale; the only new schema is
one additive column (migration **0061**). Founder decisions baked in: reminder = **24h before scheduled At** (rolling,
keyed on reminder sent, not a fixed cron); **WhatsApp only** (no email; existing in-app notifications untouched);
notify + reminder fire **only when a meeting link is present**. The whole feature stays **toggle-gated and dormant**
(WhatsApp is OFF in every environment) — so this spec's evidence is enqueue/params/sweep correctness, NOT a live send.

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC (each REQ-NNN)
- **Design:** [design md](design.md) — architecture, data model, error handling, testing, migration
- **Tasks:** [tasks md](tasks.md) — implementation tasks, each traced to a REQ-NNN + owning specialist
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario

## Status

APPROVED (2026-06-21) — scoped team-review passed; Boss signed off after adopting the recommended defaults for the
review's open rulings (founder may still override any at no cost — feature is dev-branch + toggled-OFF/dormant):
**(R1) <24h bookings = Option A** (a booking made ≤24h out gets the schedule confirmation only; reminder sent is set
true at schedule time so the sweep sends no duplicate — promoted to a testable REQ-003 sub-scenario);
**(R2) Singapore time format** = EEE, d MMM yyyy, h:mm a (e g. Sat, 21 Jun 2026, 3:30 pm);
**(R3) intuition otp** = **(re)register it too** (founder ruled 2026-06-21) — Deliverable 1 / REQ-007 registers all
THREE templates in Meta (one-time passcode + interview-scheduled + reminder), not just the two interview ones.
Two Boss hardening conditions folded into the build: (C1) one-line **deploy-backfill** note — existing SCHEDULED+linked
interviews already inside the 24h window at first deploy must not surprise-fire (acceptable while dormant; optionally a
one-time reminder sent is yes data-fix); (C2) **body Params are never logged** (mirror the one-time passcode never-log invariant).

## Review & Sign-Off

- [x] Author / Counterpart — backend pipeline (software-engineer, lead) — SIGN-OFF / 2026-06-21
- [x] Counterpart — schema / migration 0061 + sweep query (dba) — SIGN-OFF (deploy-backfill note C1) / 2026-06-21
- [x] Counterpart — phone personal information / template content / opt-in gating (security-expert) — SIGN-OFF (body Params never-logged C2) / 2026-06-21
- [x] Counterpart — idempotency + edge cases (qa-engineer) — SIGN-OFF (<24h Option A as testable REQ) / 2026-06-21
- [x] Counterpart — payload / event contract (bff-specialist) — SIGN-OFF / 2026-06-21
- [x] Technical Writer (template copy + docs) — SIGN-OFF (3 templates incl. one-time passcode per R3) / 2026-06-21
- [x] Boss (final sign-off → flips status to APPROVED) — APPROVED / 2026-06-21

## Changelog

- 1.2 (2026-06-21) HISTORICALLY MARKED IMPLEMENTED — built (backend-only), gated green (1973 unit tests, ≥97.5% line+branch, Boss-reverified),
  deployed local, all 6 live-evidence REQs PASS (WhatsApp dormant — evidence = outbox rows + database flags). **SEND-BACK →
  fix → re-prove on REQ-003:** the live convergence audit caught a Sev-1 — Interview Service send Due Interview Reminders
  cast the native-query scheduled at (row 3) to Offset Date Time, but the PG JDBC driver returns timestamptz as
  java time.Instant → Class Cast Exception every sweep tick, so the 24h reminder never fired on real PostgreSQL (the
  green local gate missed it — Testcontainers IT self-skips here, F5, and a mock returns Offset Date Time). Fixed via a
  type-tolerant the to offset date time utc operation (Instant/Offset Date Time/Timestamp→Coordinated Universal Time, fails loud)
  + a unit test exercising the real Instant type (now catchable without Docker); the expiry sweep + other Object
  readers audited clean. **Re-proven live:** sweep → 2 INTERVIEW REMINDER rows + reminder sent is yes, re-run → 0
  (idempotent), 0 Class Cast (default cadence too). Founder rulings R1/R2/R3 + hardening C1/C2 all evidenced. Closes the
  WhatsApp-interview-reminder requirement (activation gated on WhatsApp enablement — dormant today).
- 1.1 (2026-06-21) APPROVED — scoped team-review passed + Boss sign-off. Founder rulings: **R1 <24h=Option A**
  (booking ≤24h out gets confirmation only; set reminder sent is yes at schedule time → no duplicate; MODIFIED REQ-003 to
  add this as a testable sub-scenario), **R2 Singapore time format**=EEE, d MMM yyyy, h:mm a ("Sat, 21 Jun 2026, 3:30 pm"),
  **R3 register all THREE templates** incl. intuition otp (MODIFIED REQ-007 — one-time passcode is no longer confirmatory-only).
  Boss hardening folded into build: C1 deploy-backfill note (no surprise-fire for in-window interviews at first deploy),
  C2 body Params never logged. status DRAFT→APPROVED.
- 1.0 (2026-06-21) initial draft — ADDED REQ-001…REQ-008 (link-gated parameterized WhatsApp schedule-notify;
  parameterized notification pipeline carrying body Params; idempotent 24h-before reminder sweep to both parties;
  reminder sent column / migration 0061; reschedule re-arms the reminder; INTERVIEW REMINDER added to the delivered
  event set; the three Meta template definitions; toggle-gated dormancy + param-less back-compat). software-engineer-
  authored with dba + security-expert + qa-engineer + bff-specialist input. status DRAFT pending scoped team-review.
