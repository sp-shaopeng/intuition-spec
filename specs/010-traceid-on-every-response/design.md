# Design 010 — Trace references in responses

A trace reference lets someone reporting a problem give support staff an identifier that can be matched to the server's logs. This design adds that reference to supported responses and to error messages in the browser, while preserving the existing response content.

**Status:** The June 2026 record describes the original implementation. The September correction clarifies exceptions and leaves some security-response coverage outstanding. This public rewrite does not establish that every corrected requirement is implemented. InTuition is not live, and application tests were not rerun for this publication. Read the [requirements](requirements.md) and [current requirements register](../CURRENT-REQUIREMENTS.md) alongside this design.

## How the reference travels

The server establishes a reference for each request and attaches it to the request's log entries. It also returns the reference in a response header named X-Correlation-ID. A header is information sent alongside the response body, so it can carry a reference even when the response has no body or contains a file.

The design uses that same request reference in supported response bodies. The browser reads it when handling an error and can show a small, copyable reference beneath the error message. Support staff can then use the reference to find the relevant logs.

The header name, body-field name and method for obtaining the current request reference must be shared across the server components. This avoids one component silently using a different name. If the current reference is unavailable, the fallback must be non-empty; verification must still check agreement between the header, added body field and request log.

## REQ-001 — Application errors and outstanding security coverage

Errors handled by the application's common error handler gain a trace reference inside their existing error information, beside the error code and message. Existing validation details, retry information and response status must stay intact. The response must not be wrapped in a new outer structure.

Some authentication and permission failures are produced by the security layer before the common application error handler runs. The September correction requires equivalent body-reference coverage for these responses, without changing permission checks or request-forgery protection. That acceptance condition remains outstanding until separately verified. The response header is the interim reference carrier for these paths.

Operational health responses and framework-generated error pages are approved header-only exceptions. Their existing body formats must not be changed merely to insert a reference.

## REQ-002 — Successful responses that contain an object

A JSON object is a response with named fields. The design adds a trace-reference field beside the existing fields of supported successful JSON objects. An object containing a page of results also qualifies: its records and pagination information remain intact, with the reference added to the surrounding object.

Before adding the field, the response is converted using the application's configured JSON serializer. A serializer controls how application values become the response sent to the browser. The design uses an object representation that can safely hold both the existing values and the new textual reference. This avoids the earlier proposal's risk of treating a response as if all its values had the same type.

The original private contract note supporting that correction is not included in this public edition. Its existence is not a fresh passing-test claim.

Responses that are not suitable objects must remain unchanged. This includes lists, single values, empty responses, downloads, streamed content and non-JSON formats. Standard problem-detail responses also keep their defined structure. Their trace reference remains available through the header.

If an object already contains a field with the trace-reference name, preserve it rather than overwriting application data. In this collision case, the header is authoritative for the request. Only fields added by this tracing feature are required to match the header.

If the response customisation itself fails, return the original body and retain the header. Adding diagnostic information must not turn an otherwise valid response into a new error.

## REQ-003 — Preserve the response header

Keep the correlation header on successful responses, application errors, security errors, empty responses and downloads. The browser must be permitted to read that header when the frontend and server use different origins. An origin is the combination of a site's scheme, host and port.

Checks must cover both header presence and browser access to it. The proposal for a second header name was explicitly deferred; adding an alias is not part of this design.

## REQ-004 — Show an optional reference in the browser

When handling an application error, the browser first reads the reference inside the error information. If that is absent, it tries the correlation header. The reference is optional because a network failure may occur before any server response arrives.

An error notification or error page should show a small “Reference” line only when a reference is available. The user must be able to copy it with an accessible control. It supplements the useful error message; it does not replace the explanation of what went wrong. Success notifications do not show an error reference, and missing references do not produce an empty label.

## Behaviour to verify

The following table states expected behaviour, not recorded passing results.

| Response or condition | Body treatment | Reference available from |
|---|---|---|
| Successful JSON object without a conflicting field | Add the reference beside existing fields. | Body and header. |
| Paginated JSON object | Add the reference to the wrapper; preserve records and pagination. | Body and header. |
| List or array | Leave unchanged. | Header. |
| Single text, number or boolean value | Leave unchanged. | Header. |
| Empty response | Keep the body empty. | Header. |
| Download, binary response or stream | Leave the content unchanged. | Header. |
| Non-JSON response or standard problem-detail response | Preserve the established format. | Header. |
| Error from the common application error handler | Add the reference inside the existing error information. | Body and header. |
| Security-layer authentication or permission error | Body coverage remains an outstanding requirement. | Header in the meantime. |
| Existing field with the same name | Preserve its value; do not mistake it for this request's reference. | Header is authoritative. |
| Failure while adding diagnostic information | Return the original body. | Header. |

## Compatibility, cost and checks

Adding a field is intended to preserve compatibility, but it still requires checking consumers that compare complete responses. Tests must keep meaningful assertions about existing fields and add the expected reference where appropriate. They must not discard important checks simply to make a changed response pass.

Verify the added reference against the response header and corresponding log entry. Include the exceptional response types above, the existing-field collision, a missing request context and a forced failure in response customisation. Browser checks must cover references read from the body, the header fallback, copying the reference, and omission when no reference is available.

The object conversion adds work before the response is written. Measure that cost if a frequently used operation needs optimisation. Preserve the existing serialization rules and response values when making such changes.

No database schema change or new library is required by this design. Verification should include the server and frontend checks, followed by requests against the running development application and a browser check of the visible reference. The [task record](tasks.md) describes the work; the [testing and release overview](../../docs/testing-and-release.md) distinguishes configured checks from results observed for this publication.
