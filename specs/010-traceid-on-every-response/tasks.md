# Tasks — trace Id on every application programming interface response (010)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Implementation tasks broken from [design md](design.md). Each task is small, names the **owning specialist**, traces
> to one or more **REQ-NNN**, lists the **exact files**, and a **per-task verification step**. The Boss **dispatches the
> owning specialist** for each — the orchestrator does not build these itself. Check a box only when its evidence is
> captured under the private evidence directory for the requirement. This is a **cross-cutting contract** change: the backend lane (advice + error field +
> constants + strict-test sweep) and the frontend lane (reference surface) are **non-overlapping subtrees** and may run in
> parallel; builds **within** each subtree run sequentially.

## Governance gate (FIRST — precedes all build tasks)

| # | Task | Traces to | Owner | Verify | Status |
|---|------|-----------|-------|--------|--------|
| 1 | Scoped /team-review on this DRAFT spec (software-engineer, bff-specialist, frontend-expert, qa-engineer, devops-engineer/observability, tech-lead, Technical Writer) + Boss audit → Review & Sign-Off boxes checked → status: APPROVED. Conditions of approval: the **additive-not-envelope** contract; the response customization component **NO-OP** rule for array/primitive/empty/binary/Problem Detail; the **strict-JSON test sweep** owned + scoped; instrumentation-added body trace Id  equals  header  equals  log line, with preserved-field collision exception; retain the recorded inner-field/actuator scope decision | all | Boss | sign-off boxes in spec md checked; status flipped to APPROVED before any build task | [ ] |

## Strict-test sweep (qa-engineer — precedes/parallels the backend field landing)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 2 | **Enumerate the test blast-radius BEFORE the field lands:** grep the backend for Json Compare Mode STRICT, strict content implementation reference, exact the content operation, and IT full-body the body operation/whole-JSON equality; grep the frontend for raw-body to Equal/to Match Snapshot. Produce the exact file+line list + count in blast-radius implementation reference; classify each as "convert to json Path" vs "expect trace Id" | REQ-001, REQ-002 | qa-engineer | *Test* implementation reference, e2e/IT, *.test implementation reference | a concrete list (no "should be fine"); each entry has a fix verdict | [x] blast-radius md — Mock Mvc/unit: 0; frontend: 0; IT/lab: **1** (Registration Enumeration IT byte-identical 422 bodies → strip trace Id before compare). Also surfaced at BUILD time: narrow-generic Map bodies (notification and favourite responses whose maps were restricted to numeric values) — fixed by routing Maps through structured response object |

## Backend lane (software-engineer · bff-specialist) — sequential within the lane

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 3 | Add the shared shared trace settings constants holder: HEADER is X-Correlation-ID, logging context KEY is correlation Id, BODY FIELD is trace Id, null-safe the current operation (logging context value or fresh unique identifier). Repoint request correlation filter private literals + Security Config cross-origin request permissions literals to it (behaviour unchanged) | REQ-001, REQ-002, REQ-003 | software-engineer | new shared trace settings implementation reference; request correlation filter implementation reference; Security Config implementation reference | no raw X-Correlation-ID/correlation Id literals remain in logic; build compiles; existing header/cross-origin request permissions behaviour unchanged | [x] shared trace settings + filter/cross-origin request permissions repointed; 0 magic-string literals remain |
| 4 | REQ-001: add the put operation in Global error handler build Error (between message and the optional retry After). Confirm every @error handler still routes through build Error (no ad-hoc body added) | REQ-001 | software-engineer | Global error handler implementation reference () | every GEH error body has the trace identifier in the error response; security entry-point coverage is tracked below | [x] build Error adds the trace identifier in the error response (null-safe) |
| 5 | REQ-002: add Trace Id response customization component (@response customization component implements response customization component with the relevant response information): the supports operation = Jackson converter only; before Body Write adds top-level trace Id to Map + bean/Page (via JSON serializer value To Tree→structured response object), and **NO-OPs** on null/Problem Detail/String/Number/Boolean/Collection/array/Resource/byte/non-JSON content type/non-object tree; idempotent (never clobber an existing trace Id); never throws (catch→return original body) | REQ-002 | software-engineer | new Trace Id response customization component implementation reference | object bodies gain trace Id; the NO-OP set returns the body byte-identical | [x] Trace Id response customization component — Map+bean→structured response object; full NO-OP set; idempotent; never throws |
| 6 | bff-specialist contract confirm: the change is **additive** (no field removed/renamed/reshaped); pagination wrappers gain trace Id with content intact; Problem Detail/array/binary carry the id on the header only. Confirm cross-origin request permissions exposes X-Correlation-ID (REQ-003). Capture the contract delta note in contract implementation reference | REQ-002, REQ-003 | bff-specialist | review of files from #4/#5 + Security Config | contract note: additive, carrier rule documented, cross-origin request permissions confirmed | [x] contract md — additive, carrier rule, cross-origin request permissions confirmed |
| 7 | backend tests: (a) build Error unit — the trace identifier in the error response  equals  logging context value + logging context-empty fallback branch; (b) advice slice/unit — object→added, Map→added, array/list/String/primitive/null/binary/Problem Detail→untouched, already-has-trace Id→not-clobbered, throw→NO-OP; (c) REQ-003 pin — header on success/error/204/binary + Access-Control-Expose-Headers includes it. Then **execute the sweep fixes from Task 2** (strict→partial / expect-trace Id) | REQ-001, REQ-002, REQ-003 | qa-engineer · software-engineer | backend test sources + the files listed in Task 2 | the documented operational check green at ≥97.5/97.5; all strict-JSON tests pass | [x] backend tests added; sweep fix applied; the documented operational check BUILD SUCCESS, all coverage checks met |

## Frontend lane (frontend-expert) — parallel to the backend lane, sequential within

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 8 | REQ-004 capture: add trace Id?: string to Api Error and populate it in the response interceptor — error response data error trace Id or otherwise error response headers x-correlation-id. Mirror the body field name as a single frontend constant (no raw literal in logic) | REQ-004 | frontend-expert | api implementation reference (, ) | a rejected request's Api Error carries trace Id from body, header-fallback when body absent | [x] Api Error trace Id + body/header capture in api implementation reference; mirrored constants |
| 9 | REQ-004 render: toast Store error toast carries optional reference; the global error→toast handler passes api Error trace Id; Toast implementation reference renders a small copyable "Reference: `<trace Id>`" only when present (aria-label is Copy reference); Error Boundary implementation reference shows it if present. Never on success; never when absent | REQ-004 | frontend-expert | toast Store implementation reference, Toast implementation reference, Error Boundary implementation reference | error toast shows reference; success toast does not; network error shows none | [x] toast Store reference; query Client passes extract Trace Id; Toast + Error Boundary copyable "Reference:" |
| 10 | frontend tests (RTL): reference rendered from body trace Id; from x-correlation-id header when body absent; omitted on a network error (status 0); absent on success | REQ-004 | frontend-expert | *.test implementation reference for api implementation reference + Toast/Error Boundary | the documented operational check green at ≥97.5 line / ≥88 branch | [x] RTL tests added; lint and build and test:ci green — Lines 98.11% / Branches 92.68% |

## Evidence + live convergence (qa-engineer · frontend-expert · devops-engineer)

| # | Task | Traces to | Owner | Files | Verify | Status |
|---|------|-----------|-------|-------|--------|--------|
| 11 | **Live curl matrix** on the deployed local stack (the documented operational check): (a) a success **object** endpoint → body trace Id present  equals  X-Correlation-ID header; (b) a paginated endpoint → wrapper trace Id, content intact; (c) an array/list → header only, body intact; (d) a 404 + a 422 error → the trace identifier in the error response  equals  header; (e) a 204/binary → header only, body uncorrupted. For one request, show body trace Id  equals  header  equals  the correlation Id in that request's **log line**. Capture under REQ-001, REQ-002 and REQ-003 | REQ-001, REQ-002, REQ-003 | qa-engineer · devops-engineer | curl output + log excerpt | every captured trio (body/header/log) matches; NO-OP bodies uncorrupted | [ ] |
| 12 | **Live Playwright** capture of the frontend reference on a forced error (toast/page shows "Reference: `<id>`") + confirm the shown id matches the request's X-Correlation-ID in the network log. Capture under REQ-004 implementation reference | REQ-004 | qa-engineer · frontend-expert | screenshot + network-log note | reference visible, copyable, id matches header | [ ] |
| 13 | Technical Writer: application programming interface doc note — instrumented success objects and GEH errors carry top-level/inner trace Id; security JSON errors remain follow-up; approved non-object/actuator exceptions carry the header only; added fields share the header value and preserved collisions use the header; how to quote it in a bug report | all | Technical Writer | intuition-spec/ application programming interface/reference doc | doc note added; cross-links the header + body field | [x] backend-api md "Trace id (spec 010)" note + envelope/pagination updates |
| 14 | Do D: backend the documented operational check (≥97.5/97.5) + frontend lint and build and test:ci (≥97.5/88) green; local deploy + the curl matrix (Task 11) + Playwright (Task 12); e2e green; then virtual machine re-confirm; then commit + push on each repo's current branch (concise message + Co-Authored-By trailer; no envelope regressions) | all | software-engineer · frontend-expert · devops-engineer | backend + frontend repos | both gates green at thresholds; live convergence; pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces to | Owner | Verify | Status |
|---|------|-----------|-------|--------|--------|
| 15 | After build: /team-review (full panel) → Boss verdict on the built deliverable + evidence; fix every SEND-BACK condition in place and re-verify by re-reading/re-running; on SHIP set spec md status: HISTORICALLY MARKED IMPLEMENTED + confirm version/changelog; update requirements traceability matrix md row 010 → HISTORICALLY MARKED IMPLEMENTED | all | Boss | Boss SHIP; status HISTORICALLY MARKED IMPLEMENTED; requirements traceability matrix updated; evidence opened against the running system | [ ] |

## Notes

- **Spec-review gate first (Task 1):** the author does NOT self-approve — the Boss convenes the scoped team-review on this
  DRAFT and only flips status: APPROVED once every Review & Sign-Off box is checked AND open questions 1 (inner vs
  sibling placement) and 3 (actuator/error-page scope) are resolved with the founder. No build task starts before that.
- **Additive, never an envelope.** The done-gate proves nothing was removed/renamed: the curl matrix shows original fields
  intact plus the one added trace Id.
- **The NO-OP rule is load-bearing.** The advice must NEVER corrupt or wrap an array/primitive/empty/binary/Problem Detail
  body — those carry the trace id on the X-Correlation-ID header only. Task 7's advice tests prove byte-identical
  passthrough for the whole NO-OP set.
- **Strict-JSON sweep is a real task (Task 2 + Task 7), not an afterthought** — a cross-cutting field change breaks
  whole-body assertions; enumerate, then convert to json Path/partial or update to expect trace Id.
- A task is DONE only when its REQ-NNN evidence is captured under the private evidence directory for the requirement and opened by the reviewer.
- **No magic strings:** HEADER/logging context KEY/BODY FIELD live once in shared trace settings; frontend mirrors the field/header by value.
- Do D = both gates green + local deploy + live curl matrix + Playwright + e2e green + virtual machine re-confirm + push (per project
  CLAUDE md). On completion set spec md status: HISTORICALLY MARKED IMPLEMENTED, confirm version/changelog, and flip the requirements traceability matrix 010 row.

## Documentation correction follow-up — 2026-09-05

Historical checked entries above retain their original evidence dates; they do not establish the following coverage.

- [ ] REQ-001: cover security-entry-point JSON 401/403 bodies with the trace identifier in the error response; verify unchanged status/code/cross-site request forgery protection behavior and header equality. Header-only is interim for these paths, not a permanent waiver.
- [ ] REQ-002/003: pin the pre-existing top-level collision exception: preserve the field, use header for request tracing, assert equality only for fields added by instrumentation.
- [ ] Read the final approved placement/scope from spec md: inner error field; no alias; actuator/error-page header-only. Earlier resolve-open-question checklist text is historical, not an instruction to reopen these decisions.
