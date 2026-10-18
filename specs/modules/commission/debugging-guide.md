# Module 5: Admin Commission Tracking — Debugging Guide

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

This guide covers problems that arise during development, integration testing, and production debugging of the Module 5 commission tracking flows.

---

## Issue: Activating a match returns 422

**Symptom:** submit operation for admin matches id activate returns:


The issue: activating a match returns 422 example describes information including status, error Code, message. Concrete payload values and implementation syntax are omitted.


**Cause:** Admin Match Service activate Match enforces that a match must be in PENDING status before it can be activated. If the match is already ACTIVE (activated previously), COMPLETED (commission paid), or CANCELLED, activation is rejected.

**Resolution steps:**

1. Fetch the current match status: read operation for admin matches and find the row by ID. Check the status field in the response.
2. If status is ACTIVE the match was already activated. A commission record exists for it. You do not need to activate again — proceed to mark it paid with submit operation for admin commissions commission Id pay.
3. If status is COMPLETED or CANCELLED those are terminal states. Activation is not possible. Create a new match if a fresh engagement is needed.
4. If you are sure the match should be PENDING but the application programming interface disagrees, query the database directly:


Implementation example omitted. This public edition retains the written requirements in this section.


**Log to look for** (class the application component Match Service):

The service does not emit a specific log line on this error path. Look for the exception stack trace in the application log tagged with MATCH NOT PENDING.

---

## Issue: Creating a match returns 409

**Symptom:** submit operation for admin matches returns:


The issue: creating a match returns 409 example describes information including status, error Code, message. Concrete payload values and implementation syntax are omitted.


**Cause:** The application-level duplicate guard in Admin Match Service create Match found an existing match with status IN PENDING, ACTIVE for the same tutor Post Id, tutee Post Id pair. This check calls:


Implementation example omitted. This public edition retains the written requirements in this section.


**Important — CANCELLED pairs can be re-matched.** The guard only blocks creation when an _open_ match exists. If a previous match for this pair was cancelled, a new match can be created. This is intentional and enforced by the partial unique index:


Implementation example omitted. This public edition retains the written requirements in this section.


CANCELLED rows are excluded from this index, so the database will not block a new append records for a previously cancelled pair.

**Resolution steps:**

1. Check whether an open match exists:


Implementation example omitted. This public edition retains the written requirements in this section.


2. If a PENDING match exists and you want to proceed: either activate it or cancel it first.
3. If no open match appears in the database but you still get 409, the application-layer check may be out of sync — this should not happen in practice. Check whether the Java persistence mapping repository query is returning stale data from a second-level cache (the project currently uses no L2 cache, but verify if caching was added).
4. If you receive a 500 instead of 409 on a re-match attempt after a cancelled pair and the log shows a PSQL Exception: ERROR: duplicate key value violates unique constraint, the partial unique index migration (0009-matches-partial-unique-index implementation reference) has not been applied. Run the documented operational check or check DATABASECHANGELOG for changesets 0009-drop-unconditional-unique-constraint and 0009-add-partial-unique-index.

---

## Issue: Commission totals are wrong in the ledger

**Symptom:** The "Total Collected" or "Outstanding" summary cards on the Commission Ledger show incorrect amounts, or commission data is missing from match rows.

**Cause — N+1 was not the issue:** The backend uses a deliberate batch load to avoid N+1. Admin Match Service list Matches performs two queries:

1. the find all by order by matched at desc operation — fetches all matches.
2. the find all by match id in operation — fetches all commissions for those matches in a single IN ... query.

It then builds a a collection of named fields and their values and joins the two result sets in memory. If a match has no commission entry, the get operation returns no value and the Admin Match Item Response commission field is no value.

**Possible root causes:**

- A commission row exists in the database but the match's id was not included in match Ids. This would indicate a bug in the stream operation. Verify with:


Implementation example omitted. This public edition retains the written requirements in this section.


  This query should return zero rows. If it returns rows, data integrity has been violated (the FK constraint should prevent this).

- The frontend is computing totals from a stale cache. After marking a commission paid, Commission implementation reference invalidates the lists operation which triggers a re-fetch. If the cache is not invalidated (e g., a mutation's on Success callback did not fire due to an error), the user interface will show stale data. Reload the page to force a fresh fetch.

- The frontend total calculation at lines 76–94 of Commission implementation reference uses m commission invoice Status checks. If invoice Status is an unexpected string value (e g., mixed case), the equals PAID and equals PENDING checks fail silently and the amount is excluded from the total. Verify that invoice status in the database contains exactly PENDING or PAID in uppercase.

**Log to look for:**

There are no specific log lines for the batch load path. Add a debug log in Admin Match Service list Matches if you need to confirm the batch size:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## Issue: Monthly fee above 50,000 SGD returns 400

**Symptom:** submit operation for admin matches id activate with monthly Fee Sgd: 60000 returns:


The issue: monthly fee above 50,000 sgd returns 400 example describes information including status, errors, field, message. Concrete payload values and implementation syntax are omitted.


**Cause:** Activate Match Request declares:


Implementation example omitted. This public edition retains the written requirements in this section.


The the max operation constraint is enforced by Spring's Bean Validation (@Valid on the controller parameter) before the request reaches Admin Match Service. This is a hard ceiling built into the request data transfer object.

**Violation cases:**

| Input                    | Constraint violated    | web request protocol status |
|--------------------------|------------------------|-------------|
| monthly Fee Sgd: no value    | @Not no value             | 400         |
| monthly Fee Sgd: 0       | @Positive            | 400         |
| monthly Fee Sgd: -100    | @Positive            | 400         |
| monthly Fee Sgd: 50001   | the max operation          | 400         |
| monthly Fee Sgd: 50000   | (passes all)           | 200         |
| monthly Fee Sgd: abc   | type mismatch (Integer)| 400         |

**Note:** The frontend input (Commission implementation reference line 108) uses the parse float operation and passes the value as a number in the JSON body. Values like 360.5 are valid floats but will be truncated to whole number by Jackson deserialization. Test with whole-number values. If the fee negotiated is not a whole dollar amount, round it before submitting.

---

## Log messages to look for

The following log statements are emitted by Admin Match Service (SLF4J logger bound via @Slf4j, logger name the application component Match Service) and Admin Commission Service (logger name the application component Commission Service). Neither service currently emits INFO or WARN lines for normal flow — the Javadoc comments serve as inline documentation. Exception messages are the primary log signal.

To enable debug logging for these services in application implementation reference (non-production):


Implementation example omitted. This public edition retains the written requirements in this section.


Setting the application component: DEBUG will log the generated database query language for all repository calls, including the batch find All By Match Id In query, which is useful for diagnosing commission load issues.

To see the Hibernate-generated database query language for the batch commission load:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## How to trace a request using the correlation ID header

If the backend is configured with a correlation ID filter (standard practice: a Once Per Request Filter that reads or generates X-Correlation-Id), every log line for a single request will share the same correlation ID in the logging context.

**On the client side:** Include the header on the outgoing request:


The illustration for how to trace a request using the correlation id header contains the following information:

- X-Correlation-Id: my-debug-run-001.
- POST admin matches interface operation[REDACTED RECORD IDENTIFIER]/activate.


**On the server side:** Filter the log output by the correlation ID:


Implementation example omitted. This public edition retains the written requirements in this section.


If the log pattern includes the request’s correlation identifier in the log entry in the Logback pattern, every line for that request will contain the correlation ID. Example pattern:


Implementation example omitted. This public edition retains the written requirements in this section.


**If no correlation ID filter is configured yet:** Add the correlation ID to every log info or log warn call manually during debugging, or add a request correlation filter bean that populates logging context at request start and clears it at request end.

---

## Issue: Paying a commission returns 409

**Symptom:** submit operation for admin commissions id pay returns:


The issue: paying a commission returns 409 example describes information including status, error Code, message. Concrete payload values and implementation syntax are omitted.


**Cause:** Admin Commission Service pay Commission checks the get invoice status operation and throws a 409 if true. This can happen if:

- The user double-clicked "Mark Paid" and a second request arrived before the frontend disabled the button.
- A previous pay request succeeded but the frontend failed to re-render (network error on the response), so the button remained active.

The "Mark Paid" button in Commission implementation reference is disabled when mark Paid Mutation is Pending is true (line 289), which prevents concurrent clicks. However it does not guard against a re-render with stale state where the button reappears.

**Resolution:** A 409 on pay is idempotent from the business perspective — the commission is already paid. The frontend toast "Failed to mark commission as paid. Please try again." is misleading in this case. In the error handler you can check error response status equals 409 and show a different message or silently invalidate the query cache to update the user interface.

---

## Checking migration status

If a database-level issue is suspected (wrong constraints, missing index), check which changesets have been applied:


Implementation example omitted. This public edition retains the written requirements in this section.


Expected output (all three files present, exectype EXECUTED):


The illustration for checking migration status contains the following information:

- 0008-create-match-status-enum 0008-create-matches implementation reference EXECUTED.
- 0008-create-matches-table 0008-create-matches implementation reference EXECUTED.
- 0008-matches-indexes 0008-create-matches implementation reference EXECUTED.
- 0009-drop-unconditional-unique-constraint 0009-matches-partial-unique-index implementation reference EXECUTED.
- 0009-add-partial-unique-index 0009-matches-partial-unique-index implementation reference EXECUTED.
- 0010-create-commissions-table 0010-create-commissions implementation reference EXECUTED.
- 0010-commissions-indexes 0010-create-commissions implementation reference EXECUTED.


If 0009-add-partial-unique-index is missing and the old uq matches pair constraint is still present, re-matching CANCELLED pairs will fail with a 500 (PSQLException) instead of succeeding with 201.
