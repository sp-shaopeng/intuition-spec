> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 016
slug: error-copy-and-handling
title: Error copy + handling — humanise raw "not found: <uuid>", no-handler 500→404, create-admin field naming, tel: escaping
version: 1.1
status: HISTORICALLY MARKED IMPLEMENTED
owner: requirements-engineer   # build owners: software-engineer / frontend-expert / bff-specialist
created: 2026-06-22
updated: 2026-09-05
---

# Error copy + handling

> **Documentation correction — 2026-09-05:** Safe generic unmapped-error fallback; backend scrub remains optional. Current clauses in requirements/design/tasks supersede earlier conflicting directions. Historical HISTORICALLY MARKED IMPLEMENTED labels, dated changelog entries and evidence are retained as history; this correction makes no new runtime, test or deployment claim.


Hardens error messaging + a few error-handling edges from the 2026-06-22 review
(specs/issues/review-2026-06-22/CONSOLIDATED md). Complements spec 010 (trace Id Reference stays on the toast):

- **INT-211 (S3)** — error toasts show raw Tutee post not found: <uuid> (entity + id leak). Many service sites
  concatenate the id into the message (Interview Service implementation reference, Job Application Service implementation reference,220,451,
  560,686 Review Service implementation reference Admin Match Service implementation reference,198, …). Humanise the user-facing copy; keep the
  trace Id Reference for support.
- **bare GET 500 (S4)** — read operation for admin commission (singular; the real route is /admin/commissions) returns
  **500, not 404/405**. The Global error handler handles Http Request Method Not Supported (405, 239) but has
  **no handler for a missing route** (No Handler Found Exception/No Resource Found Exception) → it falls through to
  the catch-all Exception (292) → 500. A nonexistent path is a client error (404), not a server fault.
- **ADM-400 (S4)** — create-admin 400 doesn't name the missing field. The backend already returns field Errors
  (Create Admin Request is @Not Blank @Email …; GEH Method Argument Not Valid → field Errors, 161), and the frontend
  Api Error carries field Errors (api implementation reference) — the gap is the create-admin form not surfacing them per field.
- **tel: escaping (S4)** — Contact Reveal Card implementation reference builds a telephone link constructed from the phone number with unescaped spaces → some dialers
  fail. Encode the phone in tel/mailto hrefs.

Mixed backend+frontend error-hygiene lane. The backend no-handler→404 mapping touches Global error handler + config; the copy +
field surfacing + tel: are frontend (api Error implementation reference / forms / Contact Reveal Card).

- **Requirements:** [requirements md](requirements.md) · **Design:** [design md](design.md) · **Tasks:** [tasks md](tasks.md)
- **Evidence:** the private evidence directory for the requirement

## Historical delivery/review status

The record below describes the earlier delivery cycle; it is not approval or verification of the 2026-09-05 correction.

DRAFT — author does not self-approve. Boss convenes a scoped team-review (software-engineer, frontend-expert,
bff-specialist) + audit; flips to APPROVED when sign-off boxes are checked. The no-handler→404 mapping must be
verified not to regress the spec 010 trace Id/header behaviour.

## Review & Sign-Off
- [ ] Author — requirements (requirements-engineer)
- [ ] Counterpart — no-handler 404 mapping + message-id hygiene (software-engineer)
- [ ] Counterpart — humanised error-copy map + field surfacing + tel: encoding (frontend-expert)
- [ ] Counterpart — error contract (no personal information/id in user copy; keep code+trace Id) (bff-specialist)
- [ ] Boss (final sign-off → flips status to APPROVED)

## Changelog

- 2026-09-05 — Documentation alignment correction: Safe generic unmapped-error fallback; backend scrub remains optional. No application code or runtime verification in this change. Earlier entries below are historical.
- 1.0 (2026-06-22) initial draft — ADDED REQ-001 (humanise "<entity> not found: <uuid>" in user-facing copy, keep
  trace Id), REQ-002 (no-handler 500→404), REQ-003 (create-admin 400 names the missing field via field Errors),
  REQ-004 (encode tel:/mailto: hrefs). Grounded against the service throw-sites, Global error handler,
  Create Admin Request, Contact Reveal Card. status DRAFT pending scoped team-review.
