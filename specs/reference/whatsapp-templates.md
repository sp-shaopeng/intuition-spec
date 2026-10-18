# InTuition — WhatsApp Meta Templates Reference

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 correction:** Template copy/parameter ordering remain applicable.007 owns scheduling/deduplication; INT-082 consent-copy and enablement gates still apply. [Current requirements](../CURRENT-REQUIREMENTS.md). No runtime verification was performed by this documentation correction.

> **Date:** 2026-06-21 — **code- and spec-derived.** Template names read from
> Whats App Template Keys implementation reference +
> whatsapp cloud.otp-template in application implementation reference; bodies + param order from
> [specs/006-whatsapp-interview-reminders/design md](../006-whatsapp-interview-reminders/design.md)
> ("Meta template definitions" table). Where prose and code disagree, the **code wins**.

WhatsApp message content outside the 24-hour customer-service session window must use a
**pre-approved Meta template** — free-form text is rejected by the Cloud application programming interface. This page is the
canonical list of the **three templates to register** in the Meta Business account, the exact body
copy, and the ordered-parameter contract the backend relies on.

The feature ships **OFF in every environment** (whatsapp enabled is no); these templates are a
**parallel ops track** (owner: founder/ops) and are a prerequisite for enablement, not for the build.
See the enablement note in [runbook implementation reference](../ops/runbook.md) § "WhatsApp interview reminders
(spec 006)" and the INT-082 PE-enablement gates in
[int-082-whatsapp-otp-and-notifications-spec implementation reference](../architecture/int-082-whatsapp-otp-and-notifications-spec.md) (§9.1, PE-0).

## Templates to register

All three are **language en**. The two interview templates are **Utility** category; the one-time passcode
template is **Authentication**. The template names below MUST match the constants in
Whats App Template Keys / whatsapp cloud.otp-template exactly — a name mismatch makes the Cloud send
fail at delivery time.

| Template name | Category | Lang | Body text | Ordered params |
|---|---|---|---|---|
| intuition otp | Authentication | en | A message containing the one-time verification code, with a “Copy code” button that copies that same code | template parameter one = one-time passcode |
| intuition interview scheduled | Utility | en | A confirmation containing the scheduled date and time in Singapore time, followed by the meeting link | template parameter one = Singapore time datetime, template parameter two = meeting link |
| intuition interview reminder | Utility | en | A reminder containing the scheduled date and time in Singapore time, followed by the meeting link | template parameter one = Singapore time datetime, template parameter two = meeting link |

- intuition otp — sent on the existing INT-082 one-time passcode path. The one-time passcode template name is **configurable**
  (whatsapp cloud.otp-template, default intuition otp); the two notification names are compile-time
  constants.
- intuition interview scheduled — sent to **both** tutor and tutee when an admin schedules an
  interview **and a meeting link is present** (spec 006, REQ-001).
- intuition interview reminder — sent to **both** parties by the eligible007 per-instance reminder job (content specified by006,
  REQ-003) for a SCHEDULED, linked interview entering the 24h window.

### Parameter formats

- **template parameter one Singapore time datetime** — formatted in Asia/Singapore with the pattern
  EEE, d MMM yyyy, h:mm a (e g. Sat, 21 Jun 2026, 3:30 pm). The reminder timestamp is interpreted under the scheduling endpoint’s contract and
  converted to Singapore time for display (spec 006 § "Singapore time datetime formatting").
- **template parameter two meeting link** — the raw meeting URL; WhatsApp auto-links it in the message body.
- **template parameter one one-time passcode** — the one-time code (never logged).

## Param-order contract (REQ-007 guard)

For **both** Utility templates the backend builds body Params as an **ordered list** and Meta
substitutes them positionally:


Implementation example omitted. This public edition retains the written requirements in this section.


The code that builds body Params (Interview Service) and the registered Meta template **MUST agree
on this order.** A mismatch is **silent** — Meta has no idea which param is "the time" and which is
"the link", so a swapped order ships a message with the time and the link transposed, with no error.
When registering or editing a template, verify the template parameter one/template parameter two placeholders sit in the body in
exactly this order.

## Why the link is body text, not a URL button (ADR-001)

The meeting link is carried as a **body-text parameter** (template parameter two), **not** a Meta dynamic URL button.
A dynamic URL button only substitutes a **suffix** onto a **fixed base domain** registered with the
template, but interview links are arbitrary third-party URLs (Zoom / Google Meet / Teams / etc.) with
varying domains. Body text accepts any domain and is auto-linked by WhatsApp — one template covers all
meeting providers. See
[design implementation reference](../006-whatsapp-interview-reminders/design.md)
ADR-001 for the full rationale.

## Code anchors

| What | Where |
|---|---|
| Template name constants | Whats App Template Keys implementation reference |
| one-time passcode template name (config) | whatsapp cloud.otp-template (application implementation reference) |
| Event → template mapping | Whats App Notification Event implementation reference |
| body Params build (interview) | Interview Service implementation reference (schedule + reminder sweep) |
| Component → Meta body shape | Whats App Consumer Service implementation reference |

> Dormant templates intuition match created (MATCH CREATED) and intuition contact revealed
> (CONTACT REVEALED) still exist as constants but are **not delivered** (not in
> whatsapp notifications.events); they are out of scope for spec 006 and listed here only for
> completeness. Re-enabling them is a config + founder-cost-approval ceremony, not covered by this page.
