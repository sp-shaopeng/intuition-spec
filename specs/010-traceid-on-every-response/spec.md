> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 010
slug: traceid-on-every-response
title: trace Id on every application programming interface response (success AND error) — non-breaking
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED                # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: requirements-engineer   # cross-cutting contract spec; build owners are software-engineer / bff-specialist / frontend-expert
created: 2026-06-21
updated: 2026-09-05
---

# trace Id on every application programming interface response (success AND error) — non-breaking

> **Documentation correction — 2026-09-05:** Scoped GEH versus security error coverage; preserved-field collision exception; folded structured response object mapping and approved placement. Current clauses in requirements/design/tasks supersede earlier conflicting directions. Historical HISTORICALLY MARKED IMPLEMENTED labels, dated changelog entries and evidence are retained as history; this correction makes no new runtime, test or deployment claim.


Founder requirement: **"add a trace Id to ALL responses (success AND error) so issues are easy to trace."**

When a user (or Hunter, or a founder) reports "it broke", today there is no quick way to tie their report to the exact
request in the logs. The backend *already* generates a per-request correlation id (request correlation filter implementation reference:
reads/validates the inbound X-Correlation-ID, otherwise mints a unique identifier, puts it in the SLF4J **logging context** under
correlation Id so it stamps every log line, and sets it back on **every** response as the X-Correlation-ID header).
What is missing is (1) the same id **inside the response body** so it is visible to a human reading a JSON payload or an
error toast, and (2) any **frontend** consumer surfacing it on errors so a user can quote it.

This spec makes the trace id a first-class, **additive** field on responses **without changing the response shape into
an envelope** — trace Id is added as a top-level field on JSON object bodies (success and error) and is **always**
carried by the existing X-Correlation-ID header (the carrier for bodies that cannot take a top-level field — arrays,
primitives, 204/empty, binary/streaming downloads, Problem Detail). The body trace Id, the header, and the log line
for a given request share the **same** value when the body field is added by instrumentation. A preserved
pre-existing top-level collision is explicitly excluded; use the header for request tracing in that case. Security
401/403 JSON body coverage remains outstanding, separately from approved actuator/error-page header-only scope.

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC (each REQ-NNN)
- **Design:** [design md](design.md) — the response customization component mechanism, the error-body field, the NO-OP rule, constants, the test blast-radius
- **Tasks:** [tasks md](tasks.md) — implementation tasks, each traced to a REQ-NNN + owning specialist
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario

## Historical delivery/review status

The record below describes the earlier delivery cycle; it is not approval or verification of the 2026-09-05 correction.

DRAFT — the author does NOT self-approve. The Boss convenes a scoped team-review on this DRAFT (tasks md Task 1);
only when every Review & Sign-Off box below is checked does the Boss flip status to APPROVED and clear build. This is
a **cross-cutting contract touch** (every endpoint's body grows one field) so the spec-review is mandatory and the
test-blast-radius (strict-JSON assertions) must be agreed before build.

## Review & Sign-Off

- [x] Author — requirements (requirements-engineer) — SIGN-OFF / 2026-06-21
- [x] Counterpart — response customization component + error-body field + constants (software-engineer) — SIGN-OFF (never-throws + idempotent) / 2026-06-21
- [x] Counterpart — application programming interface contract, additive-not-envelope, cross-origin request permissions expose-header (bff-specialist) — SIGN-OFF (NO-OP rule exhaustive) / 2026-06-21
- [x] Counterpart — frontend surface of trace Id on errors (frontend-expert) — SIGN-OFF / 2026-06-21
- [x] Counterpart — edge-case matrix + body is header=log evidence + strict-JSON-test sweep (qa-engineer) — SIGN-OFF (blast-radius grep is a gating task) / 2026-06-21
- [x] Counterpart — observability/trace value, header-as-carrier (devops-engineer / observability) — SIGN-OFF / 2026-06-21
- [x] Technical Writer (docs) — SIGN-OFF / 2026-06-21
- [x] Boss (final sign-off → flips status to APPROVED) — APPROVED / 2026-06-21 (defaults: inner the trace identifier in the error response, no X-Trace-Id alias, header-only for actuator/error-page)

## Changelog

- 2026-09-05 — Documentation alignment correction: Scoped GEH versus security error coverage; preserved-field collision exception; folded structured response object mapping and approved placement. No application code or runtime verification in this change. Earlier entries below are historical.

- 1.2 (2026-06-22) APPROVED→HISTORICALLY MARKED IMPLEMENTED (dev) — scoped spec-review passed (additive-not-envelope, NO-OP rule exhaustive,
  never-throws, blast-radius gated); built backend+frontend, both gates Boss-reverified (backend 2032 tests ≥97.5/97.5; frontend 2460 tests,
  98.11% line / 92.68% branch). backend: Trace Id response customization component adds trace Id to JSON-object success bodies via
  object Mapper value To Tree→structured response object (Map/bean/record/Page Response → gains trace Id, content intact; NO-OP for
  array/primitive/Problem Detail/binary/204; idempotent; never throws). Global error handler build Error adds
  the trace identifier in the error response. X-Correlation-ID header on all (cross-origin request permissions-exposed). frontend: api implementation reference/extract Trace Id capture trace Id (body→
  header fallback) → copyable "Reference: `<trace Id>`" on error toasts + Error Boundary. **Blast-radius: NO existing app
  test broke** (all use json Path); 1 IT (Registration Enumeration IT, regression/excluded) noted. **Live convergence
  PROVEN:** success body trace Id  equals  X-Correlation-ID header (dc5bb979…); GEH 400 error body has the trace identifier in the error response.
  **Known gap (follow-up):** security 401/403 entry-point responses carry the header but not a body trace Id (bypass GEH).
  virtual machine deploy pending. status APPROVED→HISTORICALLY MARKED IMPLEMENTED.
- 1.0 (2026-06-21) initial draft — ADDED REQ-001 (trace Id in every ERROR body at the single Global error handler build Error
  choke point), REQ-002 (trace Id in every JSON-**object** SUCCESS body via a response customization component with the relevant response information that NO-OPs on
  arrays/primitives/empty/binary/Problem Detail), REQ-003 (keep X-Correlation-ID on every response — verified already
  present + cross-origin request permissions-exposed; the body's universal carrier), and REQ-004 (frontend api-client/error handler surfaces the trace Id on
  error toasts/pages as a copyable "Reference: `<trace Id>`"). Grounded against the real code on 2026-06-21
  (request correlation filter, Global error handler build Error, Security Config cross-origin request permissions, api implementation reference / api Error implementation reference).
  Additive, non-breaking (NOT an envelope). status DRAFT pending the scoped team-review (Task 1).
