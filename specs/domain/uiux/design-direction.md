# InTuition — user experience Flow-Restructure Design Direction (Wave 0)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** Choosing a tutor and requesting an interview are separate paths. The choose-success message promises assignment follow-up; only the interview path promises interview scheduling. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.

**Status:** ✅ **ALL 7 WAVES HISTORICALLY MARKED IMPLEMENTED to dev 2026-06-05** (frontend the recorded revision→the recorded revision, each team-reviewed → Boss SHIP, gates held ≥97.5 line / ≥88 branch, 1768 tests; live-smoked). Remaining follow-ups: measurement instrumentation (apply-funnel / choose→match) + a More Menu outside-click unit test.  using Originally the Wave 0 founder-approval-gate deliverable._
**Date:** 2026-06-04 · **Scope:** Keep the locked INT-049 brand/visual language ([design-tokens md](design-tokens.md)); redesign information architecture, screen hierarchy, conversion funnels, and state coverage. frontend-only, application programming interface contract frozen.
**Grounding:** team-reviewed (all lenses + Wei/Priya) → Boss SHIP-after-conditions; conditions folded in. user experience laws applied are named per change.

---

## 1. Why (the problem this fixes)
The app looks good (INT-049 tokens) but the *flows* carry debt the visual layer hides:
- **State coverage is inconsistent.** The purpose-built Data List (owns loading/empty/error/success; cards <md / table ≥md) is used in only **3 of ~30 list pages**; the rest hand-roll states → mixed spinners-vs-skeletons (CLS, jittery <400ms feel), inconsistent error recovery. Form Layout is used in **zero** pages; the two revenue wizards copy-paste divergent step machinery.
- **Choice overload.** 6–7 flat nav tabs per role (Hick/Miller); the #1 revenue screen (Tutor Detail) competes two CTAs and doesn't collapse responsively (mobile money-CTA below the fold).
- **Trust gaps at the moments that matter.** No standard post-submit success state; ~~some surfaces leak raw enum codes~~ (resolved INT-099/137a/140 — all surfaces humanise via subject Label/level Label/Job Identity, enforced by an ESLint + backend scan guard); the parent's privacy guarantee isn't stated where she feels the risk.

## 2. Design principles (every wave is audited against these)
| # | Principle | Law / heuristic | What it means concretely |
|---|---|---|---|
| 1 | **One primary action per view** | Von Restorff, Fitts | Exactly one clay/primary button; everything else secondary/ghost/link. Destructive actions separated + confirmed. |
| 2 | **Every list → Data List; every form → Form Layout** | Nielsen #1, #9 | Pages never hand-roll loading/empty/error/success/degraded or step machinery. The primitive guarantees the 5 states. |
| 3 | **Grouped, capped nav (≤5 primary)** | Hick, Miller | Rest in a "More"/account overflow; one nav source (nav Tabs implementation reference) for all roles incl. admin; mobile bottom nav on dashboards only. |
| 4 | **Progress always visible in multi-step arcs** | Goal-Gradient, Zeigarnik | Shared Wizard Stepper showing the denominator ("Step 2 of 4"), incl. across the onboarding arc. |
| 5 | **Human language, not enums** | Nielsen #2 | All user-facing strings via utils/format (never a data transfer object field). |
| 6 | **Feedback < 400ms** | Doherty | Optimistic user interface on mutations; skeletons, not spinners. |
| 7 | **WCAG 2.2 AA is the floor** | — | Focus order, modal trap, AA contrast on new state strips, 44px targets, prefers-reduced-motion (already global). |

## 3. The 5 states standard (the contract Data List/Form Layout enforce)
Every data surface must define all five — they are where trust is won/lost:
1. **Loading** — skeleton matching final layout (no CLS), never a bare spinner.
2. **Empty** — Empty State: what goes here + a CTA; never a blank void.
3. **Error** — plain language, blames the system, says how to recover, **preserves user input**, never color-only.
4. **Success** — explicit acknowledgement within 400ms; for submissions, a real success *panel* (Peak-End), not just a toast-and-redirect.
5. **Degraded** — stale-while-revalidate: show cached rows + a subtle "refreshing…" strip rather than blanking.

## 4. Information architecture (grouped by job-to-be-done, capped at 5)
Today: TUTEE 6 / TUTOR 7 / Admin 7 flat tabs. Proposed grouping (labels indicative, finalised in Wave 2+7):


Implementation example omitted. This public edition retains the written requirements in this section.


Mobile: a **bottom tab bar (≤4)** on dashboards/pipeline screens only — **not** on public browse (it would fight the apply CTA). The primary action stays reachable on every screen.

---

## 5. Flagship mockups (the 3 flows approval unlocks)

### Flow 1 — Auth + onboarding  *(Wei: "commission honesty early, or I quit and badmouth you")*
A persistent stepper spans the whole arc so a tutor never feels lost; commission is disclosed **before** account creation, in plain S$.


The illustration for flow 1 — auth + onboarding * wei: commission honesty early, or i quit and badmouth you * contains the following information:

- TUTOR ONBOARDING — desktop mobile (≤md): same, sticky footer CTA.
- InTuition [ Log out ] InTuition [log out].
- ● ○ ○ ○ ● ○ ○ ○ Step 1 of 4.
- Account Terms Profile 1st listing.
- Step 1 of 4 [Auth Card].
- Your details.
- [Auth Card · md] Name ____.
- Your details Form Layout Email ____.
- Name ____ Email ____ .Section ① Mobile 65 ____.
- Mobile 65 ____ Password ____ [▢▢▢ strength].
- Password .Section ②.
- ____ [strength ▮▮▯] [ Continue → ] (sticky).
- ▸ A note on fees: InTuition charges tutors 50% (commission line shown here too,.
- of the FIRST month's fee per successful match. before account is created).
- No upfront cost. You'll confirm this next.
- ……………………………………………………………… (sticky footer ↓).
- [ Continue to commission terms → ] ← ONE primary.


Reuse: Auth Card, Form Layout Section, existing Password Strength + relationship radios, Profile Completion Banner (INT-043) on the first-run dashboard as the Goal-Gradient nudge toward steps 3–4. New: shared Wizard Stepper reading a derived step from the existing session Storage stash (no new persistence).

### Flow 2 — Marketplace apply funnel  *(revenue path; Von Restorff + Fitts)*
One loud CTA; responsive collapse; login-gate signalled before the click.


Implementation example omitted. This public edition retains the written requirements in this section.


Mirrored on Assignment Detail for the tutor-applies direction (primary = "Apply"). Loading Spinner→Skeleton Card; "not found"→Empty State.

### Flow 3 — Posts → applicants → choose → interview → match → contact reveal  *(Priya: trust)*
The applicant list routes through Data List (card mode); the **post-choose confirmation is the single most important screen.**


Implementation example omitted. This public edition retains the written requirements in this section.


Contact details are **never prefetched** — authorized contact details for the specified match is called only at the authorized reveal (personal information invariant). Reuse: existing aria-live region + optimistic set Query Data (Applicants implementation reference), Contact Reveal Card, Pii Notice.

---

### 7a. a11y note — Form Layout Section titles are <legend>, not headings (intentional)
Form Layout Section renders each section title as a <legend> inside a <fieldset> — this is correct fieldset semantics (the legend names the group of controls for screen readers), **not** a heading regression. A flat form does not need a heading outline; the multi-step orientation ("Step 2 of 4") is carried by the shared Wizard Stepper, not by per-section <h* elements. This is a deliberate accessibility choice recorded here so a future audit does not "fix" it back into headings. (Recorded in the Wave-6 lesson; reconfirmed in Wave 2+7.)

## 6. What this is NOT (guardrails)
- Not a visual rebrand — brand tokens, type scale, component skins **unchanged**.
- Not a backend change — **application programming interface contract frozen**; enum humanization is frontend-only via utils/format.
- Not new primitives where extension suffices — Data List gains degraded + render Card on the **same** state-machine; we add only Wizard Stepper + promote already-duplicated controls (net-negative LOC).

## 7. Rollout (post-approval)
Sequence **0 → 1 (primitives + both success states) → 4 (auth) → 6 (tutor) → 3 (browse) → 5 (tutee) → 2+7 (nav/IA + admin, batched, one review)**. Each wave: team-review (uiux+frontend+product+qa) → Boss → frontend gate (≥97.5 line / ≥88 branch, re-run by orchestrator) → live smoke on dev → push (sp-dev). Measurement (apply-funnel, choose→match, onboarding completion) instrumented to prove the redesign, not assert it.

## 8. Approval asks (founder)
1. The **IA grouping + labels** in §4 (My Requests / Find Tutors …; Find Work / My Pipeline …; admin Review/Matching/Interviews/Money/People).
2. The **three flagship layouts** in §5 and the **verbatim trust/commission copy**.
3. The **rollout sequence** in §7 (MUST waves 1/4/6 first; admin batched last).
Anything you redline here, I fold in before Wave 1 touches code.
