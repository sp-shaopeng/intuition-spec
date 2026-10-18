# Design — WhatsApp interview reminders (006)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> How the requirements in [requirements md](requirements.md) will be built. Owned by **software-engineer** (lead)
> with **dba** (migration 0061 + sweep query), **bff-specialist** (payload/event contract), **security-expert**
> (phone personal information / gating), **qa-engineer** (idempotency tests), and the **Technical Writer** (template copy). Grounded
> against the real code read on 2026-06-21. **Honest scope:** WhatsApp is toggled OFF in every environment, so the live
> Cloud send is **dormant** — this design's verifiable surface is **enqueue correctness, param round-trip, and sweep
> idempotency**, NOT a live WhatsApp delivery.

## Current delivery contract and historical mechanism boundary (2026-09-05)

Spec 007 is authoritative for one-time scheduling, short-notice confirmation-only behavior, durable deduplication and stale-generation guards. The sweep/Shed Lock/reminder sent schema and task recipes below are **historical implementation records, superseded by spec 007**; do not rebuild them or treat their unchecked boxes as current work. Existing pipeline, parameter ordering, template drafting and OFF/opt-in gates still apply. Required current follow-up is the consent-copy lane and spec 007 safeguards; no runtime changes are certified here.

## Architecture

Reuse the existing transactional-outbox → Redis-stream → consumer pipeline and the interview-expiry-sweep idempotency
pattern; add only a body Params channel through that pipeline and one new scheduled method. **No new web request protocol endpoint**
(the admin schedule endpoint already exists) and **frontend consent-copy and copy-pinning test propagation required before enablement**.


Implementation example omitted. This public edition retains the written requirements in this section.


Reused patterns (do not reinvent): expire Stale Negotiations Shed Lock+a framework-managed scheduled method idempotent set-based sweep
(Interview Service ~692–753); transactional Outbox Service enqueue Whats App Notification; the one-time passcode body/component shape
(Whats App Consumer Service otp Components ~256); both-parties resolution from the get tutee operation /
the tutor user id operation.

## Data model

**Migration 0061-interview-reminder-sent implementation reference** (Liquibase XML changeset; latest existing changeset = 0060, verify at
build — leave no number collision):

| Element | Spec | Serves |
|---|---|---|
| change the record structure for interviews include the field reminder sent boolean required DEFAULT no | column, additive, existing rows → false | REQ-004 |
| partial index idx interview reminder due ON interviews scheduled at when status SCHEDULED AND reminder sent no | keeps the recurring sweep query cheap (only un-reminded scheduled rows) | REQ-003, REQ-004 |
| the reviewed database rollback dropping the index + column | back-out | REQ-004 |

Interview implementation reference gains a flag recording whether a reminder had been sent (@Column(name = "reminder sent", nullable =
false)), default false so Java persistence mapping inserts match the database default. No soft-delete concern (a new column, not a deletion). No
other table touched.

## application programming interface / contracts

**No new web request protocol endpoint.** The contract change is the **outbox payload** and the **Redis message**, owned with
bff-specialist:

- Whats App Notification Payload (record persisted in outbox events payload jsonb) gains an ordered, **nullable**
  an ordered list of message-body values:


Implementation example omitted. This public edition retains the written requirements in this section.


  - **Back-compat (REQ-008):** body Params is nullable/optional. A null/empty list reproduces today's param-less send
    (the send operation). Existing serialized rows (no body Params field) deserialize with
    body Params no value — Jackson tolerates the missing field. Keep the non-null validation on recipient User Id +
    template Name; do NOT require body Params.
  - A convenience constructor recipient User Id, template Name is retained (delegates with body Params no value) so the
    dormant MATCH CREATED / CONTACT REVEALED call sites need no change.
- the enqueue whats app notification operation accepts the params (or the payload carrying them).
- Outbox Poller dispatch Whats App passes the body params operation to enqueue Notification Sync(template Name, phone,
  body Params) — **gating unchanged** (toggle, event-set, opt-in, phone-verified all still apply; REQ-008).
- the enqueue notification sync operation serializes the ordered params into the
  Redis stream message (e g. a JSON array field, or indexed p0,p1,… fields — implementer's choice, must round-trip
  order).
- Whats App Consumer Service builds, for a notification with non-empty params, a Meta **body component**
  a message body with an ordered list of text parameters (mirroring otp Components'
  body half, minus the one-time passcode button) and passes body Component to client send; with empty/null params it passes
  the of operation (today's path). The cloud **client is unchanged**.

**New constants / enum values (no magic strings, both the template name and the event):**
- Whats App Template Keys INTERVIEW REMINDER intuition interview reminder (REQ-006).
- the interview reminder operation (REQ-006).
- Shed Lock Config LOCK INTERVIEW REMINDER interview-reminder-sweep (new lock; mirrors LOCK INTERVIEW EXPIRY).
- Config: interview reminder.sweep-ms (default 900000) and whatsapp notifications.events is INTERVIEW SCHEDULED,INTERVIEW REMINDER.

## The 24h reminder sweep — query + idempotency (REQ-003)


Implementation example omitted. This public edition retains the written requirements in this section.


Idempotency mirrors expire Stale Negotiations:
1. **Capture primitives BEFORE mutation** (INT-016 lazy-init lesson): pull id, tutee User Id, tutor User Id,
   scheduled At, meeting Link into a local list — notification/param building never touches lazy entity state.
2. **Enqueue** the two INTERVIEW REMINDER rows per interview with the format sgt operation.
3. **Set-based, flag-guarded update records** the mark reminder sent operation: update records interviews SET reminder sent = true when id IN
   (:ids) AND reminder sent = false — re-running selects nothing (the partial index no longer matches), so a second
   sweep enqueues 0 rows (REQ-003 idempotency). the scheduler lock operation prevents two instances
   sweeping concurrently; a database transaction makes enqueue+mark atomic per sweep.
4. **Metric** whatsapp interview reminder sent incremented by the number reminded; a failed sweep increments an
   …errors counter and rethrows (alert-able), matching the expiry sweep.

**Reschedule reset (REQ-005):** in schedule Interview, compare the incoming scheduled At against the persisted value;
if it changed, the set reminder sent operation before save, so a moved interview re-arms. A re-save with the same
time leaves reminder sent untouched (no spurious duplicate reminder).

## Singapore time datetime formatting

A small util the format sgt operation formats in the of operation with a fixed pattern
(default proposed: EEE, d MMM yyyy, h:mm a, e g. Sat, 21 Jun 2026, 3:30 pm). It is **pure + unit-tested** (the
coverage gate sees it directly). scheduled at is stored in Coordinated Universal Time (Offset Date Time), so the formatter converts to Singapore time
for display. The exact pattern is an **open question for the founder** (requirements md Open Q1); the design pins a
default so build is unblocked, and the pattern is a single constant (no magic string scattered).

## Meta template definitions (Deliverable 1 — content, REQ-007)

Language en. The meeting link is in **body text** (the second template parameter), NOT a dynamic URL button (ADR-001).

| Template name | Category | Body | Params |
|---|---|---|---|
| intuition otp | Authentication | a verification message containing the generated code + "Copy code" URL button (button repeats the first template parameter) | the first template parameter = one-time passcode (existing one-time passcode path; confirmatory — Open Q2) |
| intuition interview scheduled (v2, **with params**) | Utility | an interview confirmation containing the Singapore-time appointment and meeting link | the first template parameter = Singapore time datetime, the second template parameter = meeting link |
| intuition interview reminder (new) | Utility | an interview reminder containing the Singapore-time appointment and meeting link | the first template parameter = Singapore time datetime, the second template parameter = meeting link |

**Param-order contract (REQ-007 guard):** for both Utility templates the first body value mapped to the first template parameter (Singapore time datetime),
the second body value mapped to the second template parameter (meeting link). The code building body Params and the registered templates MUST agree on this
order; a mismatch silently swaps time/link in the message.

## Error handling

- **Link-gating (REQ-001/003):** absence of a meeting link is a normal branch, not an error — simply enqueue nothing
  (in-app notify still fires). No exception, no user-visible failure.
- **Sweep failure:** the a framework-managed scheduled method method wraps its body in try/catch, increments
  whatsapp interview reminder errors, logs, and rethrows so the failure is observable (matches expiry sweep). The
  flag-guarded update records means a retried sweep cannot double-send.
- **WhatsApp OFF (REQ-008):** enqueued rows are dropped at Outbox Poller dispatch Whats App (notifications disabled /
  event not enabled / recipient ineligible) exactly as today; no message is sent and no error surfaces.
- **Recipient ineligible (opt-in/phone):** dropped per-recipient at dispatch (recipient ineligible) — best-effort,
  no all-or-nothing (Open Q3); the in-app notification already covered the recipient.

## Testing strategy (backend gate: the documented operational check ≥97.5% line + branch)

| REQ | Test type | Asserts |
|---|---|---|
| REQ-001 | service/controller IT | schedule WITH link → 2 WhatsApp outbox rows carrying body Params= sgt,link; schedule WITHOUT link → 0 WhatsApp rows, in-app still emitted |
| REQ-002 | unit ×2 | Whats App Notification Payload JSON round-trips body Params in order; consumer builds the body component from params (captured client send arg) |
| REQ-003 | @Data Jpa Test/sweep IT | query selects only SCHEDULED+linked+un-reminded+in-window; first run → 2 reminder rows + reminder sent is yes; **second run → 0 (idempotent)**; out-of-window + no-link guards select nothing |
| REQ-004 | migration IT | \d interviews shows column + partial index; databasechangelog has 0061; pre-existing rows default false |
| REQ-005 | service IT | reschedule to new time → reminder sent is no (re-armed); same-time re-save → stays true |
| REQ-006 | unit + dispatch IT | the from template name operation → INTERVIEW REMINDER; is Event Enabled true when configured; reminder row not dropped event not enabled |
| REQ-007 | doc + cross-check | three template bodies committed; param order = placeholder order |
| REQ-008 | dispatch + consumer unit | toggle OFF → WhatsApp row dropped (notifications disabled), no client send; null/empty body Params → the send operation (today's path) |

**Branch-gate watch (≥97.5% branch):** the link-present vs link-absent branch (REQ-001/003), the
scheduled At-changed vs same branch (REQ-005), the body Params empty-vs-present branch in the consumer (REQ-002/008), and
the in-window vs out-of-window query selection (REQ-003) are the branch-sensitive cases and each needs an explicit test.
Live regression ITs run against the live stack (Abstract Api IT → localhost) — rebuild/redeploy before the documented operational check
reflects the change. **No live WhatsApp send is asserted** (toggle OFF; ADR-002): evidence is enqueue/params/sweep
correctness + the template drafts.

## Migration / rollout

- **Migration:** one additive Liquibase changeset (0061), applied by the documented operational check via Testcontainers; back-compatible
  (existing rows default false). the reviewed database rollback provided.
- **Config:** add interview reminder.sweep-ms (default 900000) and append INTERVIEW REMINDER to
  whatsapp notifications.events. Adding the event delivers it only when WhatsApp is ON.
- **Dormant rollout:** ships with WhatsApp OFF; enqueued rows are dropped at dispatch, so the only live effect is
  reminder sent bookkeeping. Activation rides the INT-082 PE-enablement (phone-number-id + secrets + verified sender
  + whatsapp enabled is yes) + the three registered Meta templates — out of this spec's build.
- **Rollback:** revert the changeset (drops column+index); remove INTERVIEW REMINDER from the config set; the param
  channel is additive and inert when unused. No data loss (additive column only).

## Decisions (ADR log)

### ADR-001 — Meeting link in template body text, NOT a dynamic URL button (2026-06-21)
- **Context:** Meta WhatsApp templates can carry a URL button, but a dynamic URL button only substitutes a **suffix**
  onto a **fixed base domain** registered with the template. Interview meeting links are arbitrary third-party URLs
  (Zoom / Google Meet / Teams / etc.) with varying domains.
- **Decision:** Carry the meeting link as a **body-text parameter** (the second template parameter) in the intuition interview scheduled and
  intuition interview reminder Utility templates, not as a dynamic URL button.
- **Consequences:** (+) Any meeting domain works; one template covers all providers; param order is simple
  (datetime, link). (−) The link renders as in-message text (WhatsApp auto-links it) rather than a styled button —
  acceptable and standard for multi-domain links.

### ADR-002 — 24h reminder via a rolling reminder sent-keyed sweep, NOT a tight per-interview cron (2026-06-21)
- **Context:** "24h before" could be a precisely-timed per-interview scheduled job, or a frequent sweep selecting any
  interview that has entered the 24h window. The codebase already runs the interview-expiry sweep on this exact pattern.
- **Decision:** Use a **rolling Shed Lock sweep** (default every 15 min) selecting SCHEDULED ∧ linked ∧ reminder sent is no
  ∧ scheduled at ∈ (now, now+24h]then a flag-guarded set-basedreminder sent is yes. The flag — not a timer — is the
  idempotency key (founder binding).
- **Consequences:** (+) Reuses a proven, lock-guarded, idempotent, re-run-safe pattern; no per-interview job
  scheduling/cancellation to manage on reschedule (just reset the flag, REQ-005); survives restarts (state is the database
  flag). (−) The reminder fires within one sweep interval of the 24h boundary (≤15 min late), not to-the-second — an
  acceptable tolerance for a courtesy reminder, and tunable via interview reminder.sweep-ms.

### ADR-003 — Additive body Params on the payload (nullable), reusing the one-time passcode component shape (2026-06-21)
- **Context:** Curated notifications send empty components; only one-time passcode passes params via otp Components. We need params
  for two notification templates without breaking the dormant param-less events or existing serialized outbox rows.
- **Decision:** Add a **nullable** an ordered list of message-body values to Whats App Notification Payload; the consumer builds a
  Meta body component from it (mirroring otp Components' body half) when present, else sends the of operation (today's
  path). Keep a recipient User Id, template Name convenience constructor for the param-less call sites.
- **Consequences:** (+) Fully back-compatible (null = today); one code path serves both; reuses the proven one-time passcode component
  shape; the cloud client is untouched. (−) The Redis message gains an optional params field that must round-trip order
  — covered by the REQ-002 round-trip tests.
