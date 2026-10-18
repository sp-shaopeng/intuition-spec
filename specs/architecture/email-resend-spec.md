# Email delivery through Resend — integration design

Resend was selected as the email provider on 31 May 2026. This is the historical integration design, updated to make the later delivery requirements explicit. It does not claim that a new email was sent or that InTuition is live. The [current requirements](../CURRENT-REQUIREMENTS.md), [email hardening](email-consumer-hardening.md) and [INT-087](int-087-transactional-messaging-hardening-spec.md) govern later corrections.

## The original missing piece

The investigation recorded a producer that put password-reset and post-moderation messages into a Redis stream, but no worker that read the stream and delivered them. Enabling the producer or pointing to an email-capture tool could not fix the absence of a sender.

The affected callers were password reset, post approval and post rejection. The original proposal completed the consumer side without changing those user-facing actions. Later INT-087 work changes how email intent is recorded durably; the older instruction to leave every producer unchanged must not override that requirement.

## Delivery path

A committed business action records durable email work. A worker obtains the queued message, selects the template, renders the permitted values and submits it through the Resend client. Only a confirmed successful outcome permits the relevant acknowledgement and duplicate-delivery record.

On a retryable failure, preserve the work and apply the bounded retry policy. Repeated or permanent failures need an explicit failed-message disposition and an observable reason. Missing configuration or a disabled sender is deferred work, not success.

Keeping delivery outside the user's main request avoids making that request wait on the external provider. The queue also provides a place to retain work for recovery. It does not by itself guarantee exactly-once delivery: retries, crashes and duplicate suppression still need the safeguards and tests in INT-087.

## Components and configuration

The design uses the official Resend Java client, a stream consumer and a template renderer. The provider client reads its key and sender settings from supplied configuration. Real credentials must never appear in committed files, logs or errors. Development and release environments need appropriately scoped, separately managed credentials.

Automated tests use a controlled sender substitute so they can exercise successful and failed deliveries without calling a live provider. Turning off real delivery for tests must still preserve the distinction between deferred work and successful delivery.

Release preparation requires a verified sender domain and the checks in [specification 027](../027-aws-production-deployment/spec.md). This document does not assert that any domain is verified, that a credential has a particular provider restriction, or that a sandbox email will reach a particular mailbox.

## Templates and visible behaviour

The initial scope was password-reset codes and the existing post-approval and post-rejection notifications. The broader notification catalogue also includes commission and assignment events; each template needs its own approved content and acceptance checks.

The renderer maps a template identifier and permitted values to a subject and email body. Escape inserted values, validate destination links and preserve a readable fallback where the contract allows it. Do not expose raw internal codes as user-facing labels.

The original integration itself required no new public API or database schema. Later durable outbox changes have their own migration and transaction requirements; they are not made optional by that historical scope statement.

## Privacy and observability

Log only the template, an opaque or approved masked recipient reference, and an outcome. Do not log full email addresses, one-time codes, credentials, rendered message bodies or password-reset links.

Measure successful provider submissions, failed attempts and queue backlog or pending-message age. A password-reset delivery problem must be visible as a failed or deferred operation rather than disappearing behind a successful request response. Provider submission metrics must not be described as proof of inbox delivery.

## Acceptance checks

Template tests should verify the required content and safe rendering. Provider-client tests must cover success, failure and unavailable configuration. A queue-to-consumer integration check must verify that the intended template and recipient are passed to the controlled sender, with acknowledgement and duplicate suppression occurring only at the correct outcome.

Recovery checks must exercise a failed attempt followed by a retry, a worker interruption and an unavailable sender. Confirm the retained queue or outbox state so a normal-looking return cannot hide dropped work.

A separately authorised development check may exercise password reset through actual email receipt and sign-in with the new password. No such check was performed for this publication. Relevant backend build, formatting and coverage checks remain required, with the configured coverage exclusions made explicit in the [testing overview](../../docs/testing-and-release.md).

## Historical open questions

The May proposal left consumer-group details, failed-message handling and template copy open. Later hardening and INT-087 specify the retry and duplicate-handling rules; they must be consulted before treating these as new choices. Provider-library versions should be checked when an implementation change is made, rather than treating the original example version as permanent guidance.

The design and review used AI assistance. Neither the original role labels nor this public rewrite constitute independent human assurance or a current release result.
