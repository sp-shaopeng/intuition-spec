# Design — Error copy + handling (016)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Owned by software-engineer + frontend-expert + bff-specialist. Grounded 2026-06-22.

## REQ-001 — humanise not-found copy

Primary fix at the frontend: extend the api Error implementation reference code→friendly-copy map so user-facing toasts render by CODE, never
the raw server message (which embeds the uuid). The not-found codes (TUTEE POST NOT FOUND,
TUTOR POST NOT FOUND, …) map to plain messages. Unknown/unmapped codes use a safe generic message
(e g. "Something went wrong. Please try again."), never the raw server message. The trace Id Reference (spec 010) stays. Optionally (lower
priority) scrub the id out of the backend message at the throw-sites, keeping the id in logs/trace Id — only if a
consumer doesn't rely on the message text (bff confirms). No CODE changes.

## REQ-002 — no-handler → 404

Add a the exception handler operation (and No Resource Found Exception for Spring 6) to
Global error handler returning the build error operation. Ensure the no-handler case reaches it:
set spring mvc.throw-exception-if-no-handler-found is yes and (if needed) spring web.resources add-mappings is no,
and confirm the SPA/static fall-through + spec 010 trace Id/header still hold (the filter runs regardless). Keep the
existing 405 handler (239).

## REQ-003 — create-admin field naming

frontend: the create-admin form reads Api Error field Errors (api implementation reference) and renders per-field messages (and/or a
named summary). bff confirms Create Admin Request validation produces a field Errors entry for each required field.

## REQ-004 — encode tel:/mailto:

A small frontend helper that produces a safe tel/mailto href (encode/normalise the value); use it in
Contact Reveal Card implementation reference and any other tel/mailto site. Display text unchanged.

## Test / coverage

- backend gate ≥97.5/97.5 — the 404 handler (unknown path → 404 + trace Id; 405 regression) tested.
- frontend gate ≥97.5/88 — code→copy map (no uuid, Reference present), field Errors surfacing, tel: encoding.
- Live: curl unknown path → 404; humanised toast; create-admin field error; tel: with spaces.
