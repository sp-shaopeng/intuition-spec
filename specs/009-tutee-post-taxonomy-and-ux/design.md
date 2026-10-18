# Design — subjects, levels and the tuition-request form

This is the public prose edition of specification 009. It records a June 2026 design with later corrections. The [current requirements](../CURRENT-REQUIREMENTS.md) take priority. It is not a fresh test result or a launch announcement.

## Architecture context

The database stores the recognised subjects and education levels. The browser keeps a matching list for its pickers. At the time of this design, there was no shared service that supplied that list to the browser. Both copies therefore needed an explicit comparison test. Unknown submitted values must receive a clear validation error before the request is saved.

## REQ-001 and REQ-002 — add the approved subjects and levels

Add only the founder-approved entries in [the specification](spec.md). The database change must be safe to repeat and must leave existing entries unchanged. The browser must use the same codes, labels and education groups, in a clear order; empty groups should not be displayed.

The approved correction gives N-Level its own group. Add that group before any new subject refers to it. Previously applied database changes remain unchanged; corrections use a new migration record. The comparison test must check the complete sets, labels and group mappings. Equal counts or a few matching examples are insufficient.

## REQ-003 — reject unknown subjects or levels

Before creating or updating a request, the server must confirm that both selected values are recognised. An unknown subject or level returns status 422 and the existing unknown-subject-or-level error. It must not produce a database failure or save a partial request. Use the same rule on every reachable editing path.

## REQ-004 — support an exact count above five students

Keep choices for one through five students. Choosing “6 or more” reveals a required whole-number field from 6 to 20. The review screen and saved request must contain the exact count, such as “8 students”. Editing a request above five must restore that choice and the saved number.

The server independently requires a whole number from 1 to 20. Browser validation alone is insufficient. Test below-six and above-twenty values in the revealed field, fractional values, missing values, and the ordinary one-to-five choices.

## REQ-005 — change the availability grid without changing saved availability

Show days as rows and time slots as columns. A selected day and time must still refer to exactly the same stored slot after the layout changes. Keep selection, deselection, quick selection and the conversion to and from stored availability unchanged.

Put the instructions beside the one-hour/two-hour controls. Keep validation errors beside the field. At a 390-pixel screen width, the grid may scroll horizontally inside its container, but the whole page must not. Keep the day column visible while scrolling, preserve the swipe hint, and keep touch targets at least 44 by 44 pixels.

The blocking regression test compares the stored availability for the same selections before and after the layout change. A visually correct grid that saves different times fails this requirement.

## REQ-006 — make the Rate Guide easy to find

Add links to the existing Rate Guide from the relevant FAQ answer, the footer and step three of the request form. The FAQ also supplies structured information to search engines: preserve its plain-text answer there while adding the visible link. Test that all three links open the Rate Guide and that the search-engine information remains valid.

## Required checks

| Requirement | Setup and action | Expected result |
|---|---|---|
| REQ-001/002 | Apply the approved additions and compare both complete lists. Select the new options in a real browser. | Database and browser agree on every code, label and group; valid requests save successfully. |
| REQ-003 | Create and edit requests with unknown subject or level values, then with valid values. | Invalid values return 422 before saving; valid values succeed. |
| REQ-004 | Exercise all count choices and the exact-count field, including invalid numbers. | Only permitted whole numbers save; review and edit screens show the exact count. |
| REQ-005 | Repeat a range of selections before and after the grid change; check a narrow screen. | Identical stored availability, usable touch controls and no sideways page scrolling. |
| REQ-006 | Follow each new Rate Guide link and validate the FAQ's search-engine information. | The links reach the intended page and the structured information remains valid. |

Run the affected server and browser checks against the actual candidate build. The [testing guide](../../docs/testing-and-release.md) explains the configured coverage targets, exclusions and limitations of the combined workflow. No application tests were rerun for this public edition.

## Risks and recovery

The main risk is saving different times after changing the grid. The before-and-after comparison is required before accepting that change. A mismatch between the browser and database lists can also hide an option or reject a valid-looking choice, so the full-list comparison must fail on any difference.

Adding a database education group and using it in the same unsuitable migration can fail; make the group available first. Keep the FAQ's visible links separate from its search-engine text. Enforce the student-count ceiling on both sides so a direct request cannot bypass it.
