# Requirements — Error copy + handling (016)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> EARS + Gherkin per atomic requirement, grounded 2026-06-22. Source: CONSOLIDATED md (INT-211, bare-GET 500,
> ADM-400, tel: escaping).

## Scope note (verified)

- Service throw-sites concatenate ids: ERR TUTEE POST NOT FOUND, Tutee post not found: + id etc. across
  Interview Service, Job Application Service, Review Service, Admin Match Service. These surface verbatim on
  toasts via the frontend error map.
- Global error handler handles 405 (239) + many others, but NO No Handler Found Exception/
  No Resource Found Exception handler → a missing route hits the catch-all Exception (292) → 500.
- Method Argument Not Valid → field Errors (161); Api Error field Errors populated (api implementation reference). The
  create-admin form doesn't surface them per field.
- Contact Reveal Card implementation reference a telephone link constructed from the assigned participant's phone number — unescaped.

---

## REQ-001 — User-facing error copy is humanised; no entity-name/id leak (trace Id kept)

- **Type:** functional · **Priority:** Should · **Severity:** S3 · **Source:** INT-211
- **Rationale:** "Tutee post not found: 5f2c…-uuid" is confusing to users and leaks an internal identifier. The
  human message should be plain; the trace Id Reference (spec 010) remains for support to locate the request.
- **Behaviour (EARS):** WHEN a user-facing error is rendered (toast/page), the system SHALL show a humanised message
  by error CODE (e g. TUTEE POST NOT FOUND → "That request is no longer available. Please refresh and try
  again.") rather than the raw server message that embeds an entity name + uuid. The frontend SHALL prefer a
  code→friendly-copy map. If no mapping exists, show a safe generic message such as "Something went wrong.
  Please try again."; never render the raw server message as fallback. The trace Id "Reference:" (spec 010)
  remains visible when available. Backend message scrubbing is optional defense in depth under the folded
  Wave-2 review; it is not a dependency for safe frontend copy. No existing error CODE SHALL change.
- **Owner:** frontend-expert (code→copy map) · software-engineer/bff-specialist (backend message hygiene; keep codes)
- **Acceptance criteria (Gherkin):**


- Scenario: A not-found error shows humanised copy, not a raw id.
- Given an action that fails with code TUTEE POST NOT FOUND.
- When the toast renders.
- Then it shows a plain human message (no "not found: <uuid>").
- And it still shows the trace Id Reference (spec 010).
- Scenario: Unmapped codes fall back safely (guard).
- Given an error code with no friendly mapping.
- Then a safe generic message is shown (still no raw uuid), with the Reference.


- **Required evidence:** RTL test (mapped code → friendly copy, no uuid; Reference present); confirm the backend
  not-found messages no longer embed the id (or the frontend never shows it); live capture of a humanised toast. Under
  REQ-001 implementation reference.
- **Dependencies:** spec 010 (trace Id Reference).

## REQ-002 — A request to a nonexistent route returns 404, not 500

- **Type:** functional · **Priority:** Should · **Severity:** S4 · **Source:** bare-GET 500
- **Rationale:** read operation for admin commission (no handler) returns 500 — a server-fault signal for what is a
  client mistake. It should be 404 (no such resource), consistent with the 405 already handled for wrong-method.
- **Behaviour (EARS):** WHEN a request targets a path with no matching handler, the system SHALL return 404 with
  the standard error body (code + message + trace Id), NOT 500. The existing 405 (Http Request Method Not Supported)
  behaviour SHALL be retained. The mapping SHALL preserve spec 010 (trace Id in the error body + X-Correlation-ID
  header). IF needed, throw Exception If No Handler Found (and disabling default static-mapping fall-through) SHALL be
  enabled so the no-handler case reaches the handler.
- **Owner:** software-engineer
- **Acceptance criteria (Gherkin):**


- Scenario: Nonexistent route → 404.
- Given GET admin commission interface operation (no such handler).
- When the request is handled.
- Then the response is 404 with a standard error body (code, message, trace Id).
- And it is NOT 500.
- Scenario: Wrong method still → 405 (regression).
- Given a valid path with an unsupported method.
- Then the response is 405 METHOD NOT ALLOWED (unchanged).


- **Required evidence:** IT/Mock Mvc test (unknown path → 404 with trace Id; 405 still 405); live curl. Under
  REQ-002 implementation reference.
- **Dependencies:** spec 010.

## REQ-003 — Create-admin 400 names the missing/invalid field

- **Type:** functional · **Priority:** Should · **Severity:** S4 · **Source:** ADM-400
- **Rationale:** A 400 that doesn't say which field is wrong forces guessing. The backend already returns field Errors;
  the form just needs to surface them.
- **Behaviour (EARS):** WHEN create-admin submission fails validation (400 VALIDATION ERROR with field Errors),
  the system SHALL display the per-field messages against the relevant inputs (and/or a summary naming the fields),
  not a generic "couldn't create admin". WHEN no field Errors are present, a clear generic message SHALL show.
- **Owner:** frontend-expert (· bff-specialist confirm field Errors populated for every required field)
- **Acceptance criteria (Gherkin):**


- Scenario: Missing field is named.
- Given a create-admin submission missing a required field.
- When the 400 with field Errors returns.
- Then the form shows which field is missing/invalid (per-field or a named summary).
- Scenario: No field Errors → clear generic (guard).
- Given a 400 without field Errors.
- Then a clear generic message is shown.


- **Required evidence:** RTL test (field Errors surfaced per field); confirm backend field Errors cover the required
  fields; live capture. Under REQ-003 implementation reference.
- **Dependencies:** none.

## REQ-004 — tel:/mailto: hrefs are URL-encoded

- **Type:** functional · **Priority:** Should · **Severity:** S4 · **Source:** tel: escaping
- **Rationale:** Unescaped spaces in a telephone link constructed from the phone number break some dialers. Encode the value.
- **Behaviour (EARS):** WHEN the system renders a tel or mailto link from a phone/email, it SHALL URL-encode
  the value (strip/encode spaces and other unsafe characters) so the link is valid for dialers/mail clients, while
  the displayed text remains human-readable.
- **Owner:** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: tel: link is valid for a phone with spaces.
- Given a phone like "+65 1234 5678".
- When the tel: link renders.
- Then the href has no raw spaces (encoded/normalised) and dials correctly.
- And the visible text remains human-readable.
- Scenario: mailto: is encoded similarly.
- Given an email contact.
- Then the mailto: href is properly encoded.


- **Required evidence:** unit/RTL test (href encoded; display text intact); live capture. Under REQ-004 implementation reference.
- **Dependencies:** none.

---

## Traceability summary

| REQ | One-line | Type | Priority | Severity | Owner |
|---|---|---|---|---|---|
| REQ-001 | Humanise "<entity> not found: <uuid>" in user copy; keep code + trace Id | functional | Should | S3 | frontend-expert · software-engineer |
| REQ-002 | Nonexistent route → 404 (not 500); 405 unchanged; spec-010 trace Id preserved | functional | Should | S4 | software-engineer |
| REQ-003 | Create-admin 400 names the missing/invalid field (surface field Errors) | functional | Should | S4 | frontend-expert |
| REQ-004 | tel:/mailto: hrefs URL-encoded | functional | Should | S4 | frontend-expert |

## Open questions for the Boss / founder

1. **REQ-001 fix layer:** humanise at the frontend code→copy map (preferred — no risk to application programming interface contract / no test churn on
   backend messages) vs. also scrub the id from backend messages. Default: frontend map is the user-facing fix; backend id-in-message is
   a logging detail (kept out of user copy). bff-specialist confirms no consumer depends on the raw message text.
2. **REQ-002 scope:** enabling throw Exception If No Handler Found can affect static-resource fall-through — verify no
   route/SPA-fallback regression and that spec 010 header/trace Id still apply. software-engineer validates.
