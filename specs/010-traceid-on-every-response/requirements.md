# Requirements — trace Id on every application programming interface response (010)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin**
> acceptance criteria and the **exact evidence** that will prove each scenario. Negative/guard behaviours are their own
> scenarios. Source: founder requirement — _"add a trace Id to ALL responses (success AND error) so issues are easy to
> trace."_ Founder decision (binding): the trace Id appears in **every response body** (success + error) **AND** as a
> response header — but **additively**, never as an envelope that changes existing fields or shape.

## Scope note (what already exists — NOT re-specified here)

Verified in the backend on 2026-06-21:
- **request correlation filter implementation reference** already, on **every** request: sanitises the inbound X-Correlation-ID
  (must match a unique identifier regex, else mint the random unique identifier operation), the put operation (so it stamps every log line),
  and the set header operation (), removing the logging context key in a finally. The header and the logging context key
  are private literals on the filter today (CORRELATION ID HEADER X-Correlation-ID, logging context KEY correlation Id).
- Registered in **Security Config** the add filter before operation — runs
  **before** signed session token auth so the logging context is populated for the whole chain.
- **cross-origin request permissions already exposes the header:** Security Config the set exposed headers operation (and
  it is an allowed request header at ) — so the browser/frontend can already read it cross-origin (dev 5173→8080).
- **The single error-body choke point** is Global error handler implementation reference:**297–306** build Error(status, code,
  message, retry After)→error Body = new Hash Map<>(); put using code…; put using message…; [put using retry After…]; return
  Response Entity status using status body(Map of using error, error Body). **Verified: every@error handler in the class
  returns via build Error** (the only other the status operation in the file is *inside* build Error itself) —
  so a single edit there covers **all** error bodies, including VALIDATION ERROR, FORBIDDEN, INTERNAL ERROR,
  MISSING REQUIRED HEADER, UNSUPPORTED MEDIA TYPE, METHOD NOT ALLOWED, CONSTRAINT VIOLATION, etc.
- **Problem Detail** is used in exactly one place — Collector Controller implementation reference
  the for status operation (the analytics-collect 422). RFC-7807 bodies must NOT be
  reshaped → that path relies on the header carrier (REQ-003), not a top-level trace Id field.
- **No response customization component/extra @Controller Advice exists today** other than Global error handler — REQ-002
  introduces the first one.
- **Frontend:** there is **no** consumer of X-Correlation-ID/trace Id today (grep intuition-the private implementation folder = none).
  The api client is api implementation reference (axios; response interceptor at  unwraps the backend
  an error object containing its code, message and field-specific errors envelope into the flat Api Error interface at , rejecting with it at );
  error copy is mapped in api Error implementation reference; the global error display is the toast
  (Toast implementation reference + toast Store implementation reference) and Error Boundary implementation reference.

This spec closes the **four gaps**: (1) error bodies carry no trace Id; (2) success bodies carry no trace Id; (3) the
header carrier and its cross-origin request permissions exposure must be confirmed-and-kept for the bodies that cannot take a field; (4) the frontend shows
the trace id nowhere.

---

## REQ-001 — trace Id in every ERROR response body

- **Type:** functional
- **Priority:** Must
- **Source:** Founder — "trace Id on ALL responses … error".
- **Rationale / value:** A user reporting a failed action sees an error toast; if that toast (and the JSON behind it)
  carries the same id as the log line for the failing request, support/engineering can jump straight to the right log in
  seconds instead of guessing from timestamps. The application exception handlers use one helper. Security entry-point errors bypass it and need
  separate coverage; the header is their current carrier, not proof of complete JSON-error-body coverage.
- **Behaviour (EARS):** WHEN the application programming interface returns an error body via Global error handler build Error, the system SHALL add
  a top-level trace Id field to the **inner** error object, equal to the current request's correlation id
  (the get operation), so the body becomes an error object containing code, message and request reference, with optional retry information and field errors.
  IF the logging context value is absent (the correlation filter did not run — e g. an error thrown by a filter ordered *before* it),
  the system SHALL fall back to a freshly generated unique identifier (never null, never the literal no value), and SHALL NOT remove or
  alter any existing field. **Placement is resolved: inner the trace identifier in the error response**, beside code/message.
  Security 401/403 JSON bodies remain required follow-up coverage (not an approved permanent exception).
  Actuator/error-page responses retain the separately approved header-only exception.
- **Owner (specialist):** software-engineer (the field + null-safe logging context read + constant) · bff-specialist (contract shape)
- **Acceptance criteria (Gherkin):**


- Scenario: A 404 error body carries trace Id.
- Given any request that resolves to a handled error (e g. GET an unknown id → 404 NOT FOUND).
- When the response is returned.
- Then the JSON body has error code and error message AND a non-empty the trace identifier in the error response.
- And the trace identifier in the error response equals the response's X-Correlation-ID header.
- And the trace identifier in the error response equals the correlation Id stamped on that request's log line.
- Scenario: A 422/400/500 error body carries trace Id (all paths through build Error).
- Given an error mapped to 400 VALIDATION ERROR, 422, or 500 INTERNAL ERROR.
- When the response is returned.
- Then the inner error object contains a non-empty trace Id that matches the X-Correlation-ID header.
- Scenario: trace Id is null-safe when logging context is empty (guard).
- Given the correlation id is not present in the logging context for a request.
- When build Error produces the body.
- Then the trace identifier in the error response is a non-empty fallback id (a generated unique identifier), never null and never the string "null".


- **Required evidence (defined up front):**
  - A unit test on build Error (or the handler) asserting the trace identifier in the error response is present and equals the logging context value, plus the
    logging context-empty fallback branch — committed under REQ-001 implementation reference.
  - A **live curl** against the deployed local stack hitting a 404 and a 422, capturing the response body **and** headers,
    showing the trace identifier in the error response  equals  the X-Correlation-ID header  equals  the correlation Id in the matching log line — under
    REQ-001 implementation reference.
- **Dependencies:** REQ-003 (shares the header/logging context constant).

## REQ-002 — trace Id in every JSON-object SUCCESS body (non-breaking, via response customization component)

- **Type:** functional
- **Priority:** Must
- **Source:** Founder — "trace Id on ALL responses … success".
- **Rationale / value:** A human inspecting a successful response (a captured payload in a bug report, a Postman run, a
  saved HAR) should see the same trace id without having to read response headers. This must be **additive** — it adds one
  top-level field and changes **nothing else** — so no consumer breaks and the body never becomes an envelope.
- **Behaviour (EARS):** WHEN a controller returns a **2xx** response whose body serialises to a **JSON object** (a Map,
  or a bean/record that Jackson renders as a JSON object — including pagination wrappers such as Spring Page and the
  project's page data transfer object, which ARE objects), the system SHALL add a **top-level** trace Id field equal to the current
  correlation id, **without** altering, removing, reordering-significantly, or overwriting any existing field. IF the body
  is **not** a top-level JSON object — i e. it is a JSON **array/list**, a JSON **primitive/String**, an **empty body**
  (including status 204 or an explicitly absent body), a **binary / file / streaming** download (e g. byte,
  Resource, Streaming Response Body, application/octet-stream, any non-JSON Content-Type), or a **Problem Detail**
  (RFC-7807, must not be reshaped) — the advice SHALL **NO-OP** (return the body untouched) and the trace id SHALL be
  available **only via the X-Correlation-ID header** (REQ-003). IF a body unexpectedly already contains a top-level
  trace Id, the advice SHALL NOT overwrite it (idempotent / non-clobbering). This is an explicit
  collision exception: the preserved value is not an instrumentation-added request id and need not equal the
  header. X-Correlation-ID remains authoritative for request tracing in that case. New endpoint data transfer objects SHALL
  reserve trace Id for the request correlation id; unrelated business identifiers use another field. The advice SHALL never throw — any failure
  to add the field degrades to a NO-OP (the body is returned unmodified; the header still carries the id).
- **Owner (specialist):** software-engineer (the response customization component + the object/non-object discrimination) ·
  bff-specialist (contract: additive, pagination wrappers included, the non-object carrier rule)
- **Acceptance criteria (Gherkin):**


- Scenario: A JSON-object success body gains a top-level trace Id.
- Given an endpoint returning a JSON object (e g. GET auth me interface operation, or a single-resource GET).
- When it returns 200.
- Then the body has all its original fields unchanged.
- And it has an added top-level trace Id equal to the X-Correlation-ID header.
- Scenario: A pagination wrapper gains trace Id (it is an object).
- Given a paginated list endpoint returning a Page/page-data transfer object object (content[], page metadata).
- When it returns 200.
- Then content[] and the page metadata are unchanged.
- And a top-level trace Id is added equal to the X-Correlation-ID header.
- Scenario: A top-level JSON array body is NOT modified (carrier = header).
- Given an endpoint whose 200 body is a JSON array/list.
- When it returns 200.
- Then the array body is byte-for-byte unchanged (no wrapping, no field).
- And the trace id is still available on the X-Correlation-ID header.
- Scenario: A primitive/String/empty/binary/Problem Detail body is NOT modified (NO-OP, no corruption).
- Given a 204 No Content, OR a String/primitive body, OR a file/binary/streaming download, OR a Problem Detail.
- When the response is returned.
- Then the body is returned untouched (no corruption, no added field, content type preserved).
- And the trace id is available on the X-Correlation-ID header.
- Scenario: A pre-existing top-level trace Id is preserved without misidentifying the request.
- Given a success object already containing an unrelated trace Id.
- When the response is returned.
- Then the existing field is unchanged.
- And X-Correlation-ID is the authoritative request id.
- And the preserved collision is excluded from body/header equality checks.
- Scenario: The advice never corrupts a body on failure (guard).
- Given an internal failure while attempting to add trace Id to an object body.
- When the advice runs.
- Then it returns the original body unmodified (degrade to NO-OP, never throw).


- **Required evidence (defined up front):**
  - Unit/slice tests for the advice's supports/before Body Write covering: object→field-added; Map→field-added;
    array/list→untouched; String/primitive→untouched; null/empty→untouched; binary/non-JSON Content-Type→untouched;
    Problem Detail→untouched; already-has-trace Id→not-overwritten — committed under REQ-002 implementation reference.
  - A **live curl matrix** on the deployed local stack: a success object endpoint (body has trace Id  equals  header), a
    paginated endpoint (wrapper has trace Id, content intact), an array/list endpoint (header only, body intact), and
    a 204/binary path (header only, no corruption) — under REQ-002 implementation reference.
- **Dependencies:** REQ-003 (the header carrier for the NO-OP paths); REQ-001 (shares the header/logging context constant).

## REQ-003 — X-Correlation-ID header on ALL responses (confirm + keep) and cross-origin request permissions-exposed

- **Type:** non-functional / contract (confirm-and-keep, plus a regression guard)
- **Priority:** Must
- **Source:** Founder — "trace Id on ALL responses"; the header is the **universal carrier** for bodies that cannot take a
  top-level field (REQ-002 NO-OP paths) and the cross-check for the body field.
- **Rationale / value:** The header already exists on every response and is already cross-origin request permissions-exposed; this requirement
  **pins** that behaviour with a test so the body-field work (REQ-001/002) can rely on the header as the always-present
  carrier and the frontend can read it cross-origin. Every trace value added by this instrumentation MUST match the header. The preserved pre-existing
  top-level field in REQ-002 is the explicit collision exception; support uses the header for that request.
- **Behaviour (EARS):** The system SHALL set X-Correlation-ID on **every** response (success, error, redirect, binary,
  204) — the existing request correlation filter behaviour, retained. WHILE serving a cross-origin request, the system SHALL
  expose X-Correlation-ID to the browser via Access-Control-Expose-Headers (the existing Security Config setting,
  retained). The value SHALL equal the request's logging context correlation Id and SHALL equal every trace Id added by this instrumentation. A preserved pre-existing
  top-level collision is excluded from that equality assertion (REQ-002).
  An **optional** X-Trace-Id alias header is explicitly **deferred** (see open questions) — default keeps the single
  X-Correlation-ID header to avoid duplicate-carrier confusion.
- **Owner (specialist):** bff-specialist (contract + cross-origin request permissions) · devops-engineer/observability (trace value)
- **Acceptance criteria (Gherkin):**


- Scenario: Every response carries the header.
- Given any request (success, error, 204, or a binary download).
- When the response is returned.
- Then it has a non-empty X-Correlation-ID header.
- Scenario: The header is cross-origin request permissions-exposed to the browser.
- Given a cross-origin request from the SPA origin.
- When the response is returned.
- Then Access-Control-Expose-Headers includes X-Correlation-ID.
- And client JS can read response headers['x-correlation-id'].
- Scenario: Body trace Id matches the header (cross-check).
- Given a success object without a pre-existing trace Id and an application error response.
- When each is returned.
- Then the body's trace Id equals that response's X-Correlation-ID header.


- **Required evidence (defined up front):**
  - A test asserting X-Correlation-ID present on a success, an error, and a 204/binary response, and that
    Access-Control-Expose-Headers includes it — under REQ-003 implementation reference.
  - A live curl i capture (headers) for each of the three response classes — under REQ-003 implementation reference.
- **Dependencies:** none (confirms existing behaviour; provides the carrier REQ-001/002 depend on).

## REQ-004 — frontend surfaces the trace Id on errors (copyable reference)

- **Type:** functional (frontend)
- **Priority:** Must
- **Source:** Founder — "so issues are easy to trace" (the user-facing payoff: a reporter can quote the id).
- **Rationale / value:** The id only helps support if a non-technical reporter can read it back. When an action fails, the
  frontend should show a small, copyable reference (e g. Reference: 5f2c…) on the error toast/page, read from the error body's
  trace Id (REQ-001) with a fallback to the X-Correlation-ID response header (REQ-003). It must be unobtrusive (small,
  secondary), never replace the human-readable error message, and never appear on success.
- **Behaviour (EARS):** WHEN the api client rejects a request, it SHALL capture the trace id — preferring
  response data.the trace identifier in the error response, falling back to the x-correlation-id response header — onto the flat Api Error
  (Api Error trace Id?: string). WHEN a global error surface renders (the error toast, and the Error Boundary/error
  page), it SHALL display the trace id as a small copyable "Reference: `<trace Id>`" element WHEN one is present, and SHALL
  omit it WHEN absent (e g. a pure network error with no response). The id SHALL NOT be shown on success toasts.
- **Owner (specialist):** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: An error toast shows a copyable reference.
- Given a mutation/query rejects with a backend error carrying trace Id.
- When the global error toast renders.
- Then it shows the human error message.
- And it shows a small "Reference: `<trace Id>`" that can be copied.
- Scenario: trace Id falls back to the response header.
- Given an error response whose body has no trace Id but whose X-Correlation-ID header is set.
- When the api client rejects.
- Then Api Error trace Id is taken from the x-correlation-id header.
- And the toast shows that reference.
- Scenario: No reference is shown when none is available (guard).
- Given a pure network failure (status 0, no response, no header).
- When the error toast renders.
- Then it shows the network message and NO "Reference:" line.
- Scenario: Success toasts never show a reference.
- Given a successful action with a success toast.
- Then no "Reference:" line is shown.


- **Required evidence (defined up front):**
  - RTL tests: error toast renders "Reference: `<trace Id>`" from the body; from the header (body absent); omits it on a
    network error; success toast has none — under REQ-004 implementation reference.
  - A **live Playwright screenshot** of a forced error showing the reference on the toast/page, and confirmation the shown
    id matches the request's X-Correlation-ID header in the network log — under REQ-004 implementation reference.
- **Dependencies:** REQ-001 (body field), REQ-003 (header fallback + cross-origin request permissions exposure).

---

## Traceability summary

| REQ | One-line | Type | Priority | Owner |
|---|---|---|---|---|
| REQ-001 | trace Id added to every ERROR body at the single Global error handler build Error choke point (logging context value, null-safe fallback) | functional | Must | software-engineer · bff-specialist |
| REQ-002 | trace Id added to every JSON-**object** SUCCESS body via a response customization component with the relevant response information that NO-OPs on array/primitive/empty/binary/Problem Detail (additive, never an envelope, never corrupts) | functional | Must | software-engineer · bff-specialist |
| REQ-003 | X-Correlation-ID on ALL responses (confirm+keep) + cross-origin request permissions-exposed; the universal carrier; body value  equals  header value | non-functional/contract | Must | bff-specialist · devops-engineer |
| REQ-004 | frontend api-client/error surfaces a copyable "Reference: `<trace Id>`" from the error body (header fallback); omitted on network errors; never on success | functional (frontend) | Must | frontend-expert |

## Constraints (recorded for the build + the done-gate)

- **Additive, NON-breaking — NOT an envelope.** No existing field is removed, renamed, or reshaped; success bodies gain
  one top-level field, error inner objects gain one field. Bodies that cannot take a top-level field (array/primitive/
  empty/binary/Problem Detail) are untouched and rely on the header carrier.
- **No magic strings.** The header name (X-Correlation-ID), the logging context key (correlation Id), and the body field name
  (trace Id) become shared constants (today they are private literals on request correlation filter and a literal in
  Security Config). The error path, the advice, the filter, and the cross-origin request permissions config all reference the constants. frontend mirrors
  the body field name + header by value (no raw literals in logic).
- **Determinism / null-safety.** The trace value is the per-request logging context correlation Id (already deterministic per
  request); every read is null-safe with a generated-unique identifier fallback so a body field is never null/no value.
- **Test blast-radius (cross-cutting):** controller/IT tests that assert a **strict/full** JSON body (content .json .. implementation reference
  with strict is yes, full-equality / exact-map assertions, or snapshot/@Json Test exact matches) will now see an extra
  trace Id and break; tests asserting specific fields via **json Path** are unaffected. The build MUST first enumerate
  every strict/full-body assertion (backend unit + IT, and any frontend snapshot of a raw body) and convert them to partial/json Path
  assertions OR update them to expect trace Id. This sweep is a first-class build task (tasks md), not an afterthought.
- **Gates:** backend the documented operational check at **≥97.5% line AND ≥97.5% branch** (JaCoCo); frontend run the corresponding frontend quality check  and 
  run the corresponding frontend quality check  and  run the corresponding frontend quality check at **≥97.5% line / ≥88% branch** (Vitest). The new advice + the frontend reference element
  need their own coverage.
- **Live convergence is mandatory before SHIP:** curl a success **object** endpoint → body has trace Id + header present
  and equal; curl an error (404/422) → error body has trace Id  equals  header; curl an array/204/binary → header only, body
  intact; and confirm the body trace Id  equals  the X-Correlation-ID header  equals  the correlation Id in that request's log
  line. Then a live Playwright capture of the frontend reference on an error.

## Open questions for the Boss / founder (flagged, not guessed)

1. **trace Id placement in the ERROR body — inner the trace identifier in the error response vs. a sibling top-level trace Id.** Default chosen:
   **inner** (an error object containing code, message and request reference) so it sits next to code/message and the frontend reads it from the
   same object it already parses. A sibling top-level trace Id (a request reference incorrectly placed outside the error object) would make success and error
   bodies share the **same** top-level field name (more uniform). Confirm the inner default, or switch to sibling.
2. **Optional X-Trace-Id alias header.** Default: **do NOT add** an alias — keep the single existing X-Correlation-ID
   to avoid two carriers for one value. Add the alias only if the founder wants a vendor-neutral name; if added it must be
   the same value and also cross-origin request permissions-exposed.
3. **Actuator and Spring error-page (/error) responses.** Actuator endpoints (/actuator/**) and the Spring default
   error page are served outside the app controllers; the **header** still applies (the filter runs for them), but the
   response customization component may not (actuator uses its own message converters; the /error page may render Problem Detail/an
   HTML page). Default: **do not special-case** actuator//error for the body field — they get the header only. Confirm
   that is acceptable (the founder ask is about application programming interface responses, not actuator).
4. **response customization component object mechanism — Map/structured response object route.** Design recommends operating on Map bodies
   directly and wrapping bean/record bodies into a Jackson structured response object via the configured JSON serializer to add the
   field, returning the original body untouched for everything else. Confirm there is no preference for a global envelope
   (explicitly rejected here as breaking) or a per-data transfer object base-class field (rejected — invasive, misses Map responses).
