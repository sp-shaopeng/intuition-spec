# InTuition test strategy — proposed checks and inspected automation

This document began as a draft on 27 May 2026. It now distinguishes that proposal from the implementation and automation inspected on 18 October 2026. The [current requirements](../../CURRENT-REQUIREMENTS.md) govern expected behaviour, including the September corrections. **No application tests were rerun for this publication, and InTuition is not live.**

## What was proposed and what was found

| Area | Original proposal | Inspected state and limit |
|---|---|---|
| Backend tests | JUnit, Mockito, real-database integration tests and security checks. | These tools and test sources are present. Their presence is not a fresh passing result. |
| Database version | PostgreSQL 16 in illustrative test setup. | The inspected development and combined-test definitions use PostgreSQL 15. The old illustration is not evidence of the current version. |
| Frontend | An early React 18 description with component and browser tests. | The inspected application uses React 19, Vitest and React Testing Library; a separate repository contains Playwright tests. |
| API contract checks | Pact tests on both sides of the browser-to-server boundary, backed by a contract broker. | The inspected manifests and workflows do not establish Pact or a broker as implemented checks. API inventory and comparison utilities are present but do not prove a Pact setup. |
| Coverage | Minimum line and branch coverage. | Configured backend thresholds are 97.5% for lines and branches; frontend thresholds are 97.5% for lines and 88% for branches. Both configurations exclude some files or categories. |
| Release enforcement | All five proposed gates required before merging or shipping. | The workflow files establish configured jobs, not proof that all five proposals or repository merge restrictions are enforced. Current successful executions were not verified here. |

See [testing and release](../../../docs/testing-and-release.md) for the tracked test-file inventory, configured workflows and known conditional checks.

## 1. Test behaviour where it can be observed

A unit test checks a small part of the application with controlled substitutes for its dependencies. An integration test checks parts working together, such as an application query against PostgreSQL. A browser test checks the experience through a real browser. Each answers a different question.

Business-rule tests should cover successful actions, refused actions and invalid state changes. Database tests should exercise queries and constraints against the real database technology. Security tests should use the actual permission checks rather than replacing them with a mock that always permits access. Interface tests should make explicit assertions about behaviour instead of relying on stored snapshots alone.

A passing response status is insufficient when the requirement concerns stored state. Check that the intended record changed, that unrelated records did not change, and that a rejected action left the system intact.

## 2. Tools present and their roles

| Test type | Tooling found | Intended observation |
|---|---|---|
| Backend unit tests | JUnit and Mockito | Business decisions, validation and failure handling in isolation. |
| Database integration tests | Spring testing support and Testcontainers | Queries, migrations, constraints and competing writes with real services. |
| Server request and security tests | Spring testing support and MockMvc | Responses and permission decisions through the application's request handling. |
| Frontend component tests | Vitest and React Testing Library | Forms, rendering and state changes in a simulated browser. |
| Running-system regression tests | API tests and Playwright in separate test layers | Complete journeys across the application and its dependencies. |

These tools do not establish the completeness of the test cases. A named test can also skip, fail to exercise its target or assert too little.

## 3. Five review gates from the original strategy

The gate numbers are retained for traceability. Gates 1–4 describe behaviours to check. Gate 5 records a tool-specific proposal that was not established as implemented during inspection. This section must not be read as a report that every gate passed.

### Gate 1 — Business-rule coverage

Tests should exercise both sides of each important decision: valid and invalid inputs, allowed and refused actions, and legal and illegal state transitions. Interview limits and cooldowns are examples where checking only the allowed case would miss the rule.

The configured backend threshold is 97.5% line coverage and 97.5% branch coverage over the measured scope. Coverage records execution, not the quality of an assertion. The configuration excludes categories such as generated code, some configuration and data-transfer classes, regression helpers and selected repository implementation classes. It must not be described as covering every source file.

### Gate 2 — Session and permission checks

The original matrix defines these scenarios. Their expected outcomes need tests through the actual security layer:

| Scenario | Expected outcome |
|---|---|
| A tutee with a valid session uses an operation intended for tutees. | Access is permitted if its other business conditions hold. |
| A tutor uses an operation reserved for tutees. | Access is forbidden. |
| A session has expired or has been revoked. | The request is treated as unauthenticated. |
| An administrator has a valid session and the required active server-side session record. | Access is permitted if its other conditions hold. |
| The required administrator session record is missing or expired. | The request is treated as unauthenticated. |
| A request has no session cookie for a protected operation. | Authentication is required. |
| A tutee attempts an administrator-only operation. | Access is forbidden. |

Tests must also follow the current session and refresh requirements rather than assuming the original matrix covers every later change. The [session-hardening specification](../../004-auth-session-hardening-30min-ttl-and-refresh/spec.md) provides that context.

### Gate 3 — Contact privacy at the response boundary

The current account model stores email and phone in plain text under an explicit amendment. The privacy expectation is enforced through permitted access and restricted responses; this test must not claim that contact fields are encrypted.

Use synthetic accounts with known contact details. Request public listings and pre-assignment views, then inspect the full responses. They must contain neither counterparty contact fields nor the known contact values embedded in other text. Apply the same checks to relevant interview and application views, not only the first list screen.

Test the authorised exception separately. Assigned participants may use the controlled contact-reveal path when the required match state and ownership conditions hold. The current interface shows contacts for ACTIVE or COMPLETED matches and hides them for a PENDING match. Another user must not gain access merely by learning the match identifier. Administrator access has its own permission and audit requirements.

A blanket assertion that no non-admin response ever contains contact details would reject legitimate participant access. The test must distinguish public privacy from authorised sharing.

### Gate 4 — Competing interview requests

Prepare a database state in which two concurrent attempts would create the same prohibited active interview pair. Start both attempts together rather than running one after the other. After both finish, exactly one permitted record should exist: one attempt succeeds and the competing duplicate is rejected by the enforced rule.

Check the stored rows as well as both outcomes. Do not count an unrelated connection failure or an invalid fixture as proof that the duplicate guard worked.

This duplicate-pair check is different from a limit on the total number of interviews for a posting. A separate concurrency case must exercise the posting limit with different otherwise valid participants. Passing the duplicate test does not prove the wider limit cannot be exceeded.

### Gate 5 — API agreement checks; Pact remains a proposal

An API contract describes what the browser sends, what the server returns and how errors are represented. Agreement tests should cover successful sign-in, invalid credentials, locked or blocked accounts, registration outcomes, session lookup, logout and rate limits. Use the current operation contract for exact fields and statuses; the May draft is not authority for later changes.

The original Pact proposal had five steps: the browser side would define expected exchanges; tests would generate a contract record; that record would be shared through a broker; the server would verify it; and a mismatch would fail the build. The inspected files do not establish this sequence as an operating gate.

The same limit applies to the old cursor-format proposal. Pagination checks must use the current endpoint's documented contract and result bounds, rather than assuming one historical cursor design is used everywhere.

## 4. Database verification

Use a real PostgreSQL service for PostgreSQL-specific constraints, queries and competing-write tests. An in-memory substitute cannot prove those behaviours. Testcontainers provides temporary services for selected tests; the separate running-system lab exercises an assembled application.

The inspected development and combined-test definitions use PostgreSQL 15. Tests for a future release must use the version selected for that release and apply the reviewed migrations before asserting application behaviour. A database test should fail clearly when its required service cannot start; an unnoticed skip is not verification.

## 5. Analytics privacy checks

For each relevant business action, verify that the intended event is emitted with the permitted fields. The original strategy specifically excludes meeting links from analytics properties and excludes the tutee identifier from the interview-requested event. Check the complete event payload, including failure and retry paths where relevant, rather than only its event name.

These are test requirements. This document does not claim a fresh audit of every analytics event.

## 6. Configured automation and its boundaries

The backend workflow runs build, formatting, selected integration tests and coverage checks. Its separate-repository run explicitly excludes the running-system regression lab. A combined workflow starts the application parts together and runs a smaller regression set on pull requests, with broader runs on other triggers.

The frontend workflow runs lint, build and coverage checks. Its cross-repository taxonomy checks depend on the sibling backend checkout and can skip without that prerequisite. The browser configuration declares a wider browser matrix than the inspected CI installation step supplies, which installs Chromium only. A complete passing browser matrix is therefore not established by those files.

Configured jobs, observed results and enforced merge restrictions are different facts. The [testing and release overview](../../../docs/testing-and-release.md) records the inspection scope and further limits. No statement here substitutes for current reports on the exact versions proposed for release.

## 7. Frontend verification and failure handling

Component tests should check rendered forms and observable behaviour, including validation, keyboard actions and error recovery. Use a real browser for interactions that the simulated environment cannot reproduce. The [request-form case](../../008-tutee-post-flow-bugfixes/spec.md) demonstrates why both layers are needed.

The configured frontend threshold is 97.5% line coverage and 88% branch coverage over its measured scope. Exclusions include entry-point and type declarations, selected API and query infrastructure, and the navigation component. Those exclusions must remain visible beside any coverage claim.

When a check fails, identify whether the cause is application behaviour, a test assumption, unavailable infrastructure or a changed requirement. Correct the cause and repeat the affected checks. An unexplained successful retry is not proof that the defect has gone away. Review findings return to the same loop, with unresolved conditions kept open until their evidence has been checked.
