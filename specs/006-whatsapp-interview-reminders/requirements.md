# Requirements — WhatsApp interview reminders (006)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin**
> acceptance criteria and the **exact evidence** that will prove each scenario. Negative/guard behaviours are their
> own scenarios. Source: founder requirement (this session) — _"when an admin sets an interview meeting link, WhatsApp
> the timing + link to BOTH parties; 24h before, WhatsApp a reminder to both; draft the one-time passcode / scheduled / reminder Meta
> templates"_ — plus the Boss's pre-spec synthesis plan (twinkly-percolating-glade md). Founder bindings: reminder =
> **24h before scheduled At** (rolling, keyed on reminder sent); **WhatsApp only**; notify + reminder **only when a
> meeting link is present**.

## Scope note (what already exists — NOT re-specified here)

Verified in code on 2026-06-21:
- Interview already has scheduled at + meeting link (migration 0011) and a SCHEDULED status
  (Interview implementation reference).
- submit operation for admin interviews id schedule already saves meeting Link + scheduled At, flips status →
  SCHEDULED, emits **in-app** notifications to both parties, and **already enqueues a WhatsApp INTERVIEW SCHEDULED
  outbox row to BOTH parties** (Interview Service schedule Interview ~507–548). Those in-app notifications are **out of
  scope** (founder: leave them as-is).
- The WhatsApp pipeline is built and toggle-gated/dormant: transactional outbox → Outbox Poller dispatch Whats App
  (171–204, gates on whatsapp notifications toggle + event-in-config-set + recipient opt-in + phone-verified) →
  Redis stream → Whats App Consumer Service → the send operation. The consumer is
  data-driven; only the one-time passcode path passes template params today (the otp components operation), and notifications are sent with an
  **empty components list** (the send operation).
- The idempotent scheduled-sweep pattern to copy: the expire stale negotiations operation (692–753) —
  the scheduled operation + @Scheduler Lock (Redis Shed Lock), primitives-captured-before-mutation, status-guarded
  set-based update records, re-run = 0 rows.

This spec closes the **gap**: today's INTERVIEW SCHEDULED send is **param-less** (no timing/link in the body) and
fires **regardless of meeting link**, and there is **no 24h reminder**.

---

## REQ-001 — Link-gated, parameterized WhatsApp schedule-notify to both parties

- **Type:** functional
- **Priority:** Must
- **Source:** Founder requirement ("set the meeting link → WhatsApp timing + link to both parties"); plan Deliverable 2C.
- **Rationale / value:** The current INTERVIEW SCHEDULED WhatsApp send is param-less and link-blind, so a recipient
  gets a static "an interview was scheduled" with no time and no way to join. The notify must carry the **formatted Singapore time
  timing + the meeting link** and must fire **only when a meeting link is present** — a scheduling without a link gives
  the parties nothing to join, so no WhatsApp should be enqueued (the in-app notification is unaffected).
- **Owner (specialist):** software-engineer (backend)
- **Acceptance criteria (Gherkin):**


- Scenario: Scheduling WITH a meeting link enqueues a parameterized WhatsApp to both parties.
- Given an interview in PENDING SCHEDULING with a tutor and a tutee.
- When an admin POSTs admin interviews id schedule interface operation with a scheduled At AND a non-blank meeting Link.
- Then exactly two WhatsApp INTERVIEW SCHEDULED outbox rows are enqueued (one per party).
- And each row's payload carries body Params = [<formatted Singapore time datetime of scheduled At>, <the meeting link>].
- Scenario: Scheduling WITHOUT a meeting link enqueues NO WhatsApp (link-gated).
- Given an interview in PENDING SCHEDULING with a tutor and a tutee.
- When an admin POSTs /schedule with a scheduled At but a null/blank meeting Link.
- Then zero WhatsApp outbox rows are enqueued.
- And the existing in-app INTERVIEW SCHEDULED notifications to both parties are still emitted.


- **Required evidence (defined up front):** test-run log of a service/controller IT proving (a) schedule-with-link → 2
  whatsapp outbox rows whose JSON payload contains body Params= <sgt>, ; (b) schedule-without-link → 0 WhatsApp
  outbox rows while the 2 in-app notifications still fire. database/psql query output of outbox events (type WHATSAPP)
  showing the param payload for the with-link case and absence for the without-link case.
- **Dependencies:** REQ-002 (payload must carry params), REQ-004 (reschedule-reset shares the same save path)

## REQ-002 — Parameterized WhatsApp notification pipeline (payload carries body Params; consumer builds the body component)

- **Type:** functional
- **Priority:** Must
- **Source:** Plan Deliverable 2B; gap analysis (notifications are sent with empty components).
- **Rationale / value:** Today only the one-time passcode path passes template parameters; curated notifications send an **empty
  components list**, so a Meta template with the first template parameter/the second template parameter placeholders cannot be populated. To deliver timing + link
  (REQ-001) and the reminder (REQ-003), the payload must carry an ordered body Params list end-to-end
  (payload → outbox row → poller → Redis message → consumer), and the consumer must build a Meta **body component**
  from those params (mirroring the existing otp Components shape) instead of the of operation.
- **Owner (specialist):** software-engineer (backend) · bff-specialist (payload/event contract)
- **Acceptance criteria (Gherkin):**


- Scenario: A notification payload round-trips its body params through the outbox.
- Given a Whats App Notification Payload built with body Params = ["A", "B"].
- When it is Jackson-serialized into an outbox events payload jsonb column and read back.
- Then the deserialized payload exposes body Params = ["A", "B"] in order.
- Scenario: The consumer builds a Meta body component from the params.
- Given a queued WhatsApp notification message carrying ordered params ["A", "B"].
- When the consumer dispatches it.
- Then it calls the cloud client with a components list containing a "body" component.
- And that body component's parameters are [type:text,text:"A", type:text,text:"B"] in order.


- **Required evidence (defined up front):** test-run log: unit test asserting Whats App Notification Payload JSON
  round-trips body Params in order; unit test asserting the consumer builds the body component (captured/mocked
  the send operation argument) from the params, mirroring otp Components.
- **Dependencies:** none

## REQ-003 — A linked interview more than 24 hours away receives one reminder intent per party at 24 hours before

- **Type:** functional; **Priority:** Must.
- **Current mechanism:** [spec 007 REQ-003/004/005](../007-generic-scheduler-db-scheduler/requirements.md) supersedes the original rolling sweep. Register a one-time job only when scheduled At is strictly more than 24 hours away and a link exists. An interview scheduled/rescheduled at or within 24 hours receives its linked scheduling confirmation only; no catch-up or immediate reminder is registered (founder Option A).
- Both recipients use the existing opt-in/verified-phone/event/toggle gates and ordered Singapore time timing/link parameters. Current-generation durable deduplication and stale-generation guards from spec 007 are required before enablement. WhatsApp remains OFF.
- **Acceptance/evidence:** >24h linked job → due reminder intents once;  is at most 24h → no reminder job; missing link → none; stale generation or terminal interview → none; crash retry does not enqueue duplicates. Historical sweep evidence is not proof of this current contract.

## REQ-004 — Historical reminder sent storage is superseded

- **Type:** data; **Priority:** Must for historical traceability.
- Migration 0061 introduced interviews reminder sent and its due index. Spec 007 explicitly replaced that mechanism and migration 0063 removed both. Do not recreate the old flag or edit applied migrations. Current scheduling and durable reminder-intent state follow spec 007; any required new schema uses a new Liquibase allocation.
- Historical schema evidence remains under this REQ; current evidence must show the spec 007 contract.

## REQ-005 — Rescheduling advances reminder generation under spec 007

- **Type:** functional; **Priority:** Must.
- The original reset-reminder sent recipe is superseded by spec 007 REQ-005. A changed time/link or terminal transition invalidates the prior reminder generation; identical saves do not re-arm an already-enqueued generation. Register the new generation only when linked, SCHEDULED, and >24h away. A  is at most 24h reschedule gets the scheduling confirmation only.
- **Acceptance/evidence:** verify changed time, changed link, identical save, terminal transition, in-flight stale task and short-notice reschedule. Historical flag-reset evidence remains history, not an instruction to restore the flag.

## REQ-006 — INTERVIEW REMINDER is a delivered WhatsApp notification event

- **Type:** functional
- **Priority:** Must
- **Source:** Plan Deliverable 2B/2D; the dispatch layer only delivers events in whatsapp notifications.events.
- **Rationale / value:** Outbox Poller dispatch Whats App drops any event NOT in the configured
  whatsapp notifications.events set. For the reminder to be delivered when WhatsApp is ON, a new
  Whats App Notification Event INTERVIEW REMINDER (mapped to a new Whats App Template Keys INTERVIEW REMINDER =
  "intuition interview reminder"constant — no magic strings must exist ANDINTERVIEW REMINDER must be added to the
  delivered-events config. Without the config entry the reminder would be enqueued but silently dropped.
- **Owner (specialist):** software-engineer (backend) · bff-specialist (event contract)
- **Acceptance criteria (Gherkin):**


- Scenario: The reminder event resolves and is in the delivered set.
- Given the new constant Whats App Template Keys INTERVIEW REMINDER = "intuition interview reminder".
- And whatsapp notifications events includes INTERVIEW REMINDER.
- When Whats App Notification Event from Template Name using intuition interview reminder is resolved.
- Then it returns INTERVIEW REMINDER.
- And Whats App Toggle is Event Enabled using INTERVIEW REMINDER is true.
- Scenario: An INTERVIEW REMINDER outbox row is NOT dropped at dispatch as an unknown event (guard).
- Given the config delivers INTERVIEW SCHEDULED and INTERVIEW REMINDER.
- When the poller dispatches an INTERVIEW REMINDER row (WhatsApp toggle ON in the test).
- Then it is not dropped with reason "event not enabled".


- **Required evidence (defined up front):** test-run log: unit asserting from Template Name resolves the new event and
  is Event Enabled is true when configured; the whatsapp notifications.events property value showing
  INTERVIEW SCHEDULED,INTERVIEW REMINDER; a dispatch unit/IT asserting the reminder event is not dropped as
  event not enabled.
- **Dependencies:** REQ-002

## REQ-007 — Three Meta WhatsApp template definitions (one-time passcode, scheduled-with-params, reminder) are drafted for registration

- **Type:** constraint
- **Priority:** Must
- **Source:** Founder requirement ("draft the WhatsApp message templates for Meta registration"); plan Deliverable 1.
- **Rationale / value:** Meta requires every WhatsApp business-initiated template to be pre-approved with exact body
  text and parameter placeholders before it can be sent. The build references three template names; their definitions
  (name, category, language, body text with numbered placeholders, button) must be drafted precisely so the founder can
  register them and so the param **order/count** matches what the code populates. The meeting link is carried in the
  **body text** (the second template parameter), NOT a dynamic URL button — Meta URL-button base domains are fixed and cannot host arbitrary
  meeting URLs (see design ADR-001).
- **Owner (specialist):** Technical Writer (copy) · software-engineer (param-order alignment)
- **Acceptance criteria (Gherkin):**


- Scenario: The three template definitions are specified with exact text + params.
- Given the spec design section "Meta template definitions".
- Then intuition otp (Authentication) is defined: body "1 is your InTuition verification code." + copy-code button, param 1=one-time passcode.
- And intuition interview scheduled (Utility) is defined: body "Your InTuition interview is confirmed for 1 (Singapore time). Join the meeting here: 2", 1=Singapore time datetime, 2=meeting link.
- And intuition interview reminder (Utility) is defined: body "Reminder: your InTuition interview is on 1 (Singapore time). Join here: 2", 1=Singapore time datetime, 2=meeting link.
- Scenario: Each template's placeholder order matches the code's body Params order (guard).
- Given the schedule-notify / reminder build the body Params list.
- Then body Params[0] maps to 1 (Singapore time datetime) and body Params[1] maps to 2 (meeting link) for both Utility templates.


- **Required evidence (defined up front):** the committed template-definition document (design section + a runbook/
  backend-api doc the Technical Writer authors) with the three exact bodies; a cross-check note/test confirming code
  body Params order = template placeholder order. (Live Meta-registration approval is an ops task gated on WhatsApp
  enablement — out of this spec's build evidence.)
- **Dependencies:** REQ-002, REQ-006

## REQ-008 — The whole feature stays toggle-gated/dormant and back-compatible (no behaviour change when WhatsApp OFF)

- **Type:** constraint
- **Priority:** Must
- **Source:** Founder + project posture (WhatsApp ships OFF, INT-082 enablement gates); plan §"built dormant".
- **Rationale / value:** WhatsApp is OFF in every environment (no phone-number-id / sandbox). The feature must add NO
  observable behaviour while OFF: enqueued WhatsApp rows are dropped at dispatch exactly as today, the existing in-app
  notifications are unchanged, and the **param-less notification path used by other events stays byte-for-byte
  compatible** (a payload with null/empty body Params produces today's empty-components send). The new sweep must be
  inert beyond writing reminder sent is yes (the outbox rows it creates are dropped while OFF). This is the dormancy +
  back-compat guard that lets the spec ship without a live send.
- **Owner (specialist):** software-engineer (backend) · security-expert (gating)
- **Acceptance criteria (Gherkin):**


- Scenario: With WhatsApp OFF, the schedule-notify and reminder produce no send (NEGATIVE / guard).
- Given whatsapp enabled is no (every environment).
- When an interview with a link is scheduled and later swept into the 24h window.
- Then WhatsApp outbox rows are enqueued but dropped at dispatch (notifications disabled), no message is sent.
- And the in-app INTERVIEW SCHEDULED notifications are emitted exactly as before.
- Scenario: A param-less notification (other events) is unchanged (back-compat guard).
- Given a Whats App Notification Payload with null or empty body Params (e g. a legacy/other event).
- When the consumer dispatches it.
- Then it calls client send using template, to, List of operation — the same empty-components behaviour as today.
- And no body component is added.


- **Required evidence (defined up front):** test-run log: a dispatch test with the toggle OFF showing the WhatsApp row
  dropped (notifications disabled) and no client send; a consumer unit asserting a null/empty-body Params payload
  yields the send operation (today's path) with no body component; confirmation the in-app
  notifications still fire. Config showing whatsapp enabled is no.
- **Dependencies:** REQ-001, REQ-002, REQ-003

---

## Traceability summary

| REQ | One-line | Type | Priority | Owner |
|---|---|---|---|---|
| REQ-001 | Link-gated parameterized WhatsApp schedule-notify (timing+link) to both parties | functional | Must | software-engineer |
| REQ-002 | Parameterized notification pipeline — payload carries body Params, consumer builds body component | functional | Must | software-engineer · bff-specialist |
| REQ-003 | Idempotent 24h-before reminder sweep Whats Apps both parties, marks reminder sent | functional | Must | software-engineer · dba |
| REQ-004 | reminder sent column + partial index, migration 0061 (additive/back-compat) | data | Must | dba |
| REQ-005 | Reschedule resets reminder sent (re-arms the reminder) | functional | Must | software-engineer |
| REQ-006 | INTERVIEW REMINDER event + template constant added to delivered events | functional | Must | software-engineer · bff-specialist |
| REQ-007 | Three Meta template definitions (one-time passcode, scheduled-with-params, reminder) drafted | constraint | Must | Technical Writer · software-engineer |
| REQ-008 | Feature stays toggle-gated/dormant + param-less path back-compatible | constraint | Must | software-engineer · security-expert |

## Open questions for the Boss / founder (flagged, not guessed)

1. **Singapore time datetime format string.** The plan specifies "formatted Singapore time datetime" but not the exact pattern. Proposed
   default: EEE, d MMM yyyy, h:mm a in Asia/Singapore (e g. Sat, 21 Jun 2026, 3:30 pm). Confirm the format and
   whether "(Singapore time)" in the template body is sufficient or the offset should be shown. (Design pins a default;
   founder may override.)
2. **intuition otp registration status.** The one-time passcode template is referenced as already used by Phone Verification Service
   but the plan asks to "confirm/register" it. Is it already approved in Meta, or does this spec's Deliverable 1 cover
   its (re)registration too? Affects whether REQ-007 lists it as new or confirmatory. (Treated as confirmatory here.)
3. **Reminder when WhatsApp is later enabled but a party never opted in.** Per the existing poller gating, a party
   without opt-in/phone-verified simply gets no WhatsApp (in-app unaffected). Confirm that "best-effort, per-recipient"
   delivery (no all-or-nothing) is the intended behaviour for the reminder too. (Assumed yes — mirrors the existing
   schedule-notify.)
4. **Interview booked ≤24h out — confirmation vs. duplicate reminder (founder ruling, affects REQ-003 + REQ-005).**
   If an admin schedules an interview less than 24h away (with a link), the schedule confirmation fires immediately AND
   the rolling sweep would also send a reminder on its next tick (the interview is already inside the 24h window) — so
   both parties would get **two near-identical WhatsApp messages within minutes**, and the reminder copy ("…is on
   date") reads oddly for something hours away. Options:
   - **A (recommended, encoded as the default):** the confirmation IS the reminder — at schedule time, when
     scheduled At ≤ now+24h, also set reminder sent yes, so the sweep sends **no duplicate**. A separate reminder
     only fires for interviews booked **>24h** ahead. (Build task 11/12 encodes A.)
   - **B:** send the reminder anyway (~1 sweep after scheduling) — accepts the redundant double message.
   - **C:** send a reminder only if ≥N hours of buffer remain (e g. skip if <2h out).
   The spec encodes **A** to unblock the team-review; the Boss/founder confirms (or picks B/C) at the APPROVED gate.

## 2026-09-05 consent-copy propagation (REQ-006/008)

The already-authorized schedule/reminder expansion retains INT-082's lockstep condition: before enabling delivery, founder cost approval is recorded and frontend WhatsApp consent copy plus copy-pinning tests name INTERVIEW SCHEDULED and INTERVIEW REMINDER alongside any other delivered events. Event configuration and that copy/test change ship together; backend-only expansion is insufficient. This carries the existing consent-copy/cost-approval condition and adds no automatic re-consent policy. Global WhatsApp OFF remains unchanged.
