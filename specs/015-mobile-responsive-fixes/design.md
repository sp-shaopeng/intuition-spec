# Design — Mobile/responsive fixes (015)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Owned by frontend-expert + uiux-designer. frontend-only. Grounded 2026-06-22.

## REQ-001 — page overflow at 390px

The grid (Availability Grid implementation reference overflow-x-auto around the min Width table) already scrolls internally;
the page overflows because a CSS-grid ancestor track won't shrink below its content (default min-width: auto).

**Confirmed layout relationship:** The availability table already has an internal horizontal scroll container. Its parent is the availability step's wrapper inside the form section's grid. That wrapper's default minimum width prevented it from shrinking below the table's 376-pixel content width, so the whole page overflowed. The surrounding form and field group were not the source of the width requirement.

**Fix:** add min-w-0 to the **grid item = the Step-1 wrapper  at Create Request implementation reference** (the direct
child of the grid track), so it may shrink and the inner overflow-x-auto engages. This is the minimal,
targeted change — do NOT blanket min-w-0 across Form Layout (it would affect every form and risks unrelated
layout regressions; the review flagged over-broad application). If a structural test shows the Step-0/Step-2 wrappers
need it too, scope each to its own step , not the shared container. Keep the grid markup byte-for-byte.

Verify with a 390px Playwright check body scroll Width is at most body client Width **on the availability step** (see
REQ-001 e2e below).

## REQ-002 — admin nav at 390px

Reuse the Nav implementation reference hamburger-drawer pattern (222, with its Escape + focus handling, 94-106) for the admin
sections, or a collapsing admin sidebar. All 8 sections reachable; focus-trapped; Escape-dismissible. uiux-designer
picks the pattern; frontend-expert implements with the existing a11y scaffolding.

## REQ-003 — breakpoint consistency

Audit where the desktop nav is hidden per route (/assignments, /tutor/*) vs shown (/tutor/dashboard) and the
Mobile Bottom Nav visibility rules. Apply one policy (same Tailwind breakpoint classes) across all authenticated
routes so the layout doesn't depend on the route. No link unreachable at any width.

## Test / coverage

- frontend gate ≥97.5/88 — structural tests for the nav across routes + admin drawer a11y.
- e2e: a 390px page-overflow assertion that **advances the wizard past Step-1 (subject/level) to Step-2 = the
  availability/schedule step** (Create Request implementation reference the availability step, second in the wizard, the 2nd wizard step, index 1) before asserting
  body scroll Width is at most body client Width — the INT-210 gap was that e2e only checked the first step (Subject/Level),
  which has no wide grid, so it never exercised the overflow. The grid itself must still scroll internally at 390px.
  Plus 390px captures for admin nav + the three nav routes.
