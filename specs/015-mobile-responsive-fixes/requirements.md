# Requirements — Mobile/responsive fixes (015)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> EARS + Gherkin per atomic requirement, grounded 2026-06-22. Source: CONSOLIDATED md (INT-210, ADM-nav, T-04).

## Scope note (verified)

- Availability Grid implementation reference has overflow-x-auto around a min Width  (34). The intra-grid scroll
  is correct; the page overflows because an ancestor flex/grid track isn't allowed to shrink below content
  (min-w-0 missing). Classic flexbox min-content overflow.
- Nav implementation reference has hidden md:flex desktop nav (141,) + a mobile hamburger drawer (222). Admin's 8 sections
  don't get the hamburger treatment → cramped scroll strip at 390px.
- Mobile Bottom Nav implementation reference carries route links on mobile; on /assignments + /tutor/* the top desktop nav is hidden
  and links live only in the bottom nav, but /tutor/dashboard shows the full nav — inconsistent.

---

## REQ-001 — The availability grid scrolls within itself; the page does not overflow at ≤390px

- **Type:** functional / responsive · **Priority:** Must · **Severity:** S2 (regression) · **Source:** INT-210
- **Rationale:** A page that scrolls sideways at 390px is broken on the most common device class; the wizard step is
  unusable. The grid is designed to scroll internally — the page must not.
- **Behaviour (EARS):** WHEN the availability grid renders inside its form at ≤390px, the system SHALL constrain the
  grid's flex/grid ancestor using s so they cannot exceed the viewport width (e g. min-w-0 on the shrinkable track),
  so the grid's own overflow-x-auto engages and ONLY the grid scrolls horizontally — the page body SHALL NOT
  scroll horizontally. The grid's bitmask encoding and sticky day-column (spec 009) SHALL be unchanged.
- **Owner:** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: No page-level horizontal scroll at 390px.
- Given the tutee create-post wizard availability step at 390px width.
- When the page renders.
- Then document horizontal scroll width does not exceed the viewport width.
- And the availability grid itself scrolls horizontally within its container.
- Scenario: Grid behaviour unchanged ≥ sm.
- Given a viewport ≥ sm.
- Then the grid fits without the scroll affordance (as before).
- And the encoded availability is unchanged.


- **Required evidence:** an e2e/Playwright check at 390px that **advances the wizard from Step-1 (Subject/Level) to
  the availability/schedule step** (the availability step, second in the wizard, the 2nd step) and asserts body scroll Width is at most body client Width
  there (the gap e2e missed by only checking Step-1); a structural test that min-w-0 is on the Step-1 wrapper
   (Create Request implementation reference), not the shared Form Layout grid container; before/after 390px screenshots.
  Under REQ-001 implementation reference.
- **Dependencies:** spec 009 (grid).

## REQ-002 — The admin nav is usable at 390px (hamburger/drawer)

- **Type:** functional / responsive · **Priority:** Should · **Severity:** S3 · **Source:** ADM-nav
- **Rationale:** A ~154px cramped horizontal scroll strip for 8 admin sections is hard to use on mobile; the public
  nav already solves this with a hamburger.
- **Behaviour (EARS):** WHEN the admin views the app below the desktop nav breakpoint, the system SHALL present the
  admin sections via a usable mobile pattern (hamburger drawer or equivalent), consistent with the existing
  Nav implementation reference mobile drawer, with all 8 sections reachable, focus-trapped, and Escape-dismissible (mirroring the
  existing drawer's a11y).
- **Owner:** uiux-designer · frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Admin sections via a mobile drawer at 390px.
- Given an admin at 390px.
- When they open the nav.
- Then all 8 admin sections are reachable in a usable drawer/menu (not a cramped scroll strip).
- And the menu is focus-trapped and Escape-dismissible.


- **Required evidence:** RTL/a11y test of the admin drawer; live 390px capture. Under REQ-002 implementation reference.
- **Dependencies:** none.

## REQ-003 — Consistent nav breakpoint across routes

- **Type:** functional / responsive · **Priority:** Should · **Severity:** S3 · **Source:** T-04
- **Rationale:** The same viewport shows the full nav on /tutor/dashboard but collapses to bottom-only on
  /assignments and /tutor/* — confusing and makes some links unreachable depending on route.
- **Behaviour (EARS):** WHEN any authenticated route renders at a given viewport, the system SHALL apply ONE
  consistent nav breakpoint policy — the desktop nav and the mobile bottom nav SHALL appear under the same
  width rules on /assignments, /tutor/*, and /tutor/dashboard (no route-specific collapse). All primary
  navigation SHALL remain reachable at every viewport.
- **Owner:** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Same breakpoint across routes.
- Given the same viewport width.
- When navigating /tutor/dashboard, /assignments, and a /tutor/* page.
- Then the nav layout (desktop vs bottom-mobile) is consistent across all three.
- And no primary nav link is unreachable on any of them.


- **Required evidence:** RTL/structural test of the nav across the three routes at a fixed width; live captures.
  Under REQ-003 implementation reference.
- **Dependencies:** none.

---

## Traceability summary

| REQ | One-line | Type | Priority | Severity | Owner |
|---|---|---|---|---|---|
| REQ-001 | Availability grid scrolls internally; no page overflow at ≤390px (min-w-0 on the ancestor) | functional/responsive | Must | S2 | frontend-expert |
| REQ-002 | Admin nav usable at 390px (hamburger drawer, a11y) | functional/responsive | Should | S3 | uiux-designer · frontend-expert |
| REQ-003 | Consistent nav breakpoint across /assignments, /tutor/*, /tutor/dashboard | functional/responsive | Should | S3 | frontend-expert |

## Open questions for the Boss / founder

1. **REQ-002 admin nav pattern:** reuse the public Nav implementation reference hamburger drawer for admin, or a dedicated admin
   sidebar that collapses. uiux-designer recommends at design.
2. **e2e coverage gap:** the existing e2e only checks Step-1 of the wizard, which is why INT-210 slipped. Add a
   390px page-overflow assertion to the availability step (and consider a general 390px overflow sweep) — scope
   with qa-engineer.
