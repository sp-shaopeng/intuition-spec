# Email consumer hardening — ten corrective changes

This specification began on 1 June 2026 as ten corrections to the email worker, Resend client and supporting documentation. It records intended behaviour, not a fresh passing test run. The review roles were AI assistants; this was not independent human certification.

The September corrections take precedence over the original proposal. [INT-087](int-087-transactional-messaging-hardening-spec.md) governs durable email origination, bounded retries, failed-message handling and duplicate suppression after successful delivery. [Deployment specification 027](../027-aws-production-deployment/spec.md) governs sender-domain verification and release preparation. InTuition is not live.

## Scope and superseded deferrals

These ten changes strengthen the existing email path rather than introduce new product actions. The original list deferred a failed-message queue and retry limits; those are now governed by INT-087 and must not remain silently deferred on the strength of this older document.

Other historical follow-ups included a circuit breaker, provider delivery-event handling, external template files, rejection-message content and reset-link work. Their current status must be checked in the relevant requirements. A historical backlog entry does not establish whether a later change implemented it.

## Fix 1 — Reclaim the actual pending message

The recorded recovery code attempted to read after a message identifier rather than take ownership of that pending message. The corrective design uses Redis's claim mechanism to reclaim a message that was delivered but never acknowledged.

The original schedule waited five seconds after startup and then checked every sixty seconds. Only messages idle for at least thirty seconds were eligible, reducing the risk of taking a message that another worker was still sending. A message that fails dispatch remains pending for a later attempt, subject to the bounded-retry rules in INT-087.

Tests must present an eligible pending message and verify that it is claimed and dispatched, with acknowledgement only after success. A second case presents a message idle for less than thirty seconds and verifies that it is neither claimed nor sent. Repeated recovery must also respect the later duplicate-suppression rules.

## Fix 2 — Distinguish worker instances

Each worker process needs its own consumer identity, stable for that process's lifetime. The historical design combines a readable host label with a random suffix and falls back to a generated identifier if host lookup fails.

Do not share one fixed consumer name across every instance. Verify that generated identities are non-empty and distinguish instances, while remaining stable within one process. This makes pending-message ownership meaningful when more than one worker runs.

## Fix 3 — Remove embedded credentials

Required credentials must come from the local or hosted secret environment. Committed service definitions should reference supplied values, and example documentation should contain placeholders only. Local files holding credentials must remain outside source control.

The original review also required revocation of a previously exposed provider key and replacement with a new key. This publication neither tests credentials nor certifies that revocation occurred. Any claim that this account action was completed needs its own evidence; removing a literal from the latest file is not proof that the old credential stopped working.

## Fix 4 — Protect template values and links

Treat template values as data, escaping characters that could otherwise change the email's markup. Apply this consistently even when a value, such as a one-time code, is normally numeric.

Dashboard links must be absolute HTTPS links to an approved platform origin. Validate them before queueing and use a safe platform destination if rendering receives an unsafe value. Do not include the rejected sensitive value in a diagnostic log.

Tests must show that markup-like input is displayed as text rather than interpreted as markup, an unsafe link falls back safely, and a valid approved link is preserved.

## Fix 5 — Measure attempts and outcomes

Record successful provider acceptance, failed attempts and the duration of each send attempt, grouped by template where useful. A provider accepting a request is not proof that a person received it in their inbox; delivery and bounce events are separate observations.

Verify that success and failure counters change for the corresponding cases and that timing is recorded for both. Missing configuration or a disabled client must not increment a successful-send counter.

## Fix 6 — Give templates a consistent document structure

Each template should use the same basic email document wrapper, including its document declaration, character encoding and body structure. Keep the existing message content inside that wrapper.

Tests should check the wrapper for each supported template and retain assertions about the message itself. This requirement does not establish rendering quality in every email client or guarantee inbox placement.

## Fix 7 — Missing configuration means deferred delivery

A missing or blank provider key should not prevent the whole application from starting. The client should recognise that delivery is unavailable, emit a safe diagnostic and return an explicit deferred outcome.

The worker must preserve the queued work for retry. It must not acknowledge success, record that the message has already been sent, or increase successful-send metrics. When configuration is restored, the same work must remain eligible for delivery.

The producer's enablement setting and the client's ability to send are separate conditions. Turning either off is not a successful delivery. The durable outbox and consumer contract must define where deferred work remains; a normal return that silently drops it is unacceptable.

Verify startup without a key, no provider call while unavailable, preservation of the queued message, and successful retry after configuration is restored. This describes the acceptance contract, not proof that every historical caller already follows it.

## Fix 8 — Keep documentation and operating checks accurate

The roadmap, status notes, environment examples and runbook should agree on Resend as the selected provider, the templates actually supported, the retry safeguards and the remaining release prerequisites. The original report covered three templates: password reset, post approval and post rejection. Treat that as a historical count rather than today's complete inventory.

Operational checks should inspect worker presence, progress through the queue, the number of pending messages and the age of the oldest pending work. The early runbook used one hundred pending entries and five minutes of idle time as investigation signals, not universal service guarantees.

Do not trim a queue merely because it is large. Current requirements prohibit silently removing unconsumed or unacknowledged work. Investigate the cause and preserve delivery or an explicit permanent-failure record. The older unconditional trimming recipe is not an approved recovery action.

Document credential replacement and verification without exposing keys, recipients, message bodies, one-time codes or reset links. Sender-domain and production checks belong to specification 027. No current hosting or mailbox result was inspected for this rewrite.

## Fix 9 — Describe the recovery schedule correctly

The worker documentation must match its actual schedule. The original correction changed “once shortly after boot” to repeated recovery every sixty seconds. If later scheduler work changes that behaviour, update the documentation and checks together.

## Fix 10 — Explain test-only construction

Document the constructor used to supply controlled dependencies in tests, so it is not mistaken for the normal application setup. Use the project's existing annotation convention or a clear comment; do not add a library solely to label this constructor.

## Completion evidence

The acceptance set is the behaviour described in Fixes 1–10: recovery timing and ownership, distinct worker identities, credential handling, safe template values, metrics, consistent documents, deferred delivery, accurate documentation and clear test setup.

Run the relevant backend checks with the required integration services available and meet the configured 97.5% line and branch thresholds over the measured scope. A skipped database test is not evidence that recovery works with Redis. Retain the failed-then-retried scenarios and inspect the queued state, not just the return status.

Logs may identify a template, an opaque or approved masked recipient reference, and an outcome. They must not contain full contact details, one-time codes, credentials, message bodies or reset links. The [current requirements register](../CURRENT-REQUIREMENTS.md) and [testing overview](../../docs/testing-and-release.md) set the wider context.
