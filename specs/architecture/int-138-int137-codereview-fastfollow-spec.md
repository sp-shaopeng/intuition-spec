# INT-138/139/140 — INT-137 Code-Review Fast-Follow

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** SPEC v2 — team-reviewed + Boss SEND-BACK (2026-06-13), conditions folded; ready to build
**Date:** 2026-06-13
**Authors:** frontend-expert (F1/F2/F3/F4) · uiux-designer (F1) · qa-engineer (F2/F6) · devops-engineer (F2 continuous integration)
**Tracker:** [README implementation reference](../issues/README.md) — INT-138 (S3), INT-139 (S3), INT-140 (S4 batch)
**Migration:** none · **Scope:** frontend + e2e/continuous integration (no backend)
**Next milestone:** build → frontend gate + 390px mobile gate → redeploy → push

> **Review history.** v1 → team-review (relevant seats + Tech Lead) → Boss **SEND BACK** (2026-06-13), 2 conditions folded into v2:
> (1) **INT-139 must ENFORCE the @mobile gate in continuous integration** (wire into the the private implementation folder per-pull request lane, ref INT-064) — not just a one-time manual run, or the skip recurs (Sam/Taylor); (2) the v1 "no team-review needed" line was a procedure violation (every deliverable → team-review → Boss) — removed; this review is the record. F1 title-first kept as a deliberate **consistency** decision with Applications (Jamie), not a user experience-optimal claim.
>
> **Improvement pass (founder-requested 2026-06-13): fix the CLASSES structurally, not the instances.** v2 upgrades two findings: **F1 → a shared Job Identity component** (one identity-rendering path for every job card, so hierarchy can't drift again) and **F6 → automated frontend+backend raw-code scan guards** (a build gate, so the raw-code class — 4 recurrences — fails continuous integration on the 5th). Both pair with INT-139's mobile-gate continuous integration enforcement: turn standing review points into automated gates, not manual habits. Deliberate scope expansion (S3/S4 → small structural), justified by a 4×-recurring defect class.

---

## 1. Problem

INT-137 (raw-code humanisation + NOT FOUND→404) passed its gates and deployed to dev, but a **detailed
post-build code review of the implemented diff** (backend the recorded revision, frontend the recorded revision) — the review the prior
team-review covered only at spec level — found five follow-ups. None is a correctness bug; the backend is
clean (404 semantics verified: wrong-owner→403, missing→404; nullable tutee Post Title consumers
null-safe). All five are the private implementation folder polish, grouped here.

| ID | Sev | Finding |
|----|-----|---------|
| INT-138 | S3 | F1 — Matches card hierarchy inconsistent with the sibling Applications card |
| INT-139 | S3 | F2 — INT-137 frontend shipped without running the standing 390px mobile-overflow gate |
| INT-140 | S4 | F3 (aria raw codes) + F4 (helper duplication) + F6 (no card-level humanisation test) |

---

## 2. INT-138 (F1) — align the Matches card hierarchy with Applications

**Defect.** The two tutor surfaces invert their heading hierarchy:
- Matches implementation reference — **heading** = the subject level label operation, **caption** = the match’s tuition-request title.
- Applications implementation reference — **heading** = job Title (app tutee Post Title or otherwise Tutee request), **caption** = subject Level.

Same persona, two screens, opposite hierarchy — reads as inconsistent.

**Fix — IMPROVED: extract ONE shared identity component (structural, not per-site alignment).**
Aligning Matches to Applications by hand fixes today's two screens but leaves the *next* job-card
surface free to drift again (Invitations already renders its own identity line). The higher-leverage
fix is a single presentational component that every job-card surface consumes, so the hierarchy is
consistent by construction:


Implementation example omitted. This public edition retains the written requirements in this section.


Consume it in the three surfaces, deleting their bespoke identity markup:
- Matches implementation reference → a readable job label from its title, subject and level
- Applications screen → the same readable job label, retaining the link to that assignment
- Invitations implementation reference → a readable job label derived from the invitation’s subject and level when no title exists

One component, one hierarchy, one humanisation path — the inconsistency class is gone, not just this
instance. Unit-test Job Identity directly (title present → title heading + label caption; title null →
label heading; code input → humanised, never raw). Net LOC ≈ flat or negative once all three migrate
(promotion-pays-once rule — migrate all three in this change, don't leave a twin).

**Avoid the THIRD twin (iteration-2 catch).** A humanised subject·level renderer ALREADY exists:
Identity Link implementation reference's the secondary label operation (58-63, INT-061), plus
format subject Level Label and format interview Context Title. Do NOT add a 4th copy. Consolidate onto
**one canonical helper** (format subject Level Label) and make every site delegate to it:
Identity Link secondary Label, interview Context Title, and Job Identity all call subject Level Label
(this absorbs F4 — see §4). **Placement + hygiene:** Job Identity lives with the INT-049 shared
primitives in the private implementation folder, **component-only export** (react-refresh: no value/const
exports in the file — the same rule Identity Link implementation reference already follows), consumes design tokens, no
arbitrary values. **application programming interface:** drop the unused as prop; a single optional to switches text↔Link. If
Identity Link's shape can absorb the tutor surfaces without admin-only baggage, prefer generalising it
over a new component — decide by reading Identity Link first (don't guess).

## 3. INT-139 (F2) — run the 390px mobile-overflow gate (standing frontend review point)

**Defect.** INT-137's humanised labels ("A-Math (O-Level) · Secondary 4 (Express)") are materially
wider than the raw codes, but the frontend shipped without the mandatory 390px overflow check
(frontend-expert SKILL standing point, added the same session). The gate exists:
mobile-overflow spec implementation reference, Playwright mobile-chrome project (390×844,
playwright config implementation reference).

**Fix — two parts (Boss condition #1):**
1. **Run now:** the @mobile project against the redeployed dev stack; confirm the tutor Matches /
   Applications / Invitations cards (and the new longer labels) keep
   document scrolling Element scroll Width is at most client Width. If a card overflows, the label container
   needs min-w-0 + truncation (truncate/break-words) — the INT-074 pattern. Record the green
   result in the INT-139 row.
2. **Enforce forward (so the skip can't recur):** wire the @mobile overflow project into the per-pull request
   continuous integration lane (the frontend ci implementation reference or the e2e regression implementation reference lane — ref INT-064) so a future frontend change that
   overflows at 390px fails the build, not a manual ritual. Without this, INT-139 only fixes the
   symptom; the standing-point miss was *caused* by the gate not being enforced.

   **Dependency (iteration-2 honesty):** the @mobile overflow spec drives the *running app* (frontend+backend
   served), so it can only run in the **composed multi-repo e2e lane** — which is **INT-064**, whose
   verification is currently access-gated (no gh/Actions visibility here). So enforcement splits:
   the **run-now** (part 1) is achievable today against the local redeployed stack and is INT-139's
   shippable core; the **continuous integration-enforce** (part 2) is a tracked sub-task **blocked on INT-064** landing —
   record it as such, don't mark INT-139 fully done until the gate actually runs in continuous integration. The frontend
   per-repo ci implementation reference alone cannot host it (it has no served backend), so do not fake-enforce there.

This is a process fix as much as a code one: the standing mobile-web review point must be **executed
and enforced**, not just declared.

## 4. INT-140 (F3 + F4 + F6) — a11y, DRY, test depth (S4 batch)

**F3 — aria-labels emit raw codes.** Invitations implementation reference interpolate raw
the invited job's subject and level into the Accept/Decline button aria-labels — screen readers
announce "AMATH OLEVEL" while sighted users see the humanised label. Fix: route through
the subject level label operation in the aria-label too. (Update the
Invitations test implementation reference button-name regexes if they match the raw form.)

**F4 — IMPROVED: collapse ALL subject·level humanisers onto one helper (3 copies today).** The
the filter operation shape exists in **three** places: format subject Level Label
(182-194), format interview Context Title (163-174), and admin/Identity Link secondary Label
(58-63). Make subject Level Label the single source and have the other two delegate:
interview Context Title → the subject level label operation;
Identity Link secondary Label → the subject level label operation. Behaviour identical;
keep both existing test suites green. One label path means the scan guard (F6) has exactly one allowed
sink to whitelist.

**F6 — IMPROVED: an automated repo-wide raw-code GUARD, not one more fixture.** A single
Matches test fixture only guards one surface; the raw-code class has now recurred four times
(INT-068 → 099 → 136a → 137a) precisely because each fix was per-site. Realise the INT-137a
SHARED-LESSON structural rule as an executable guard so the 5th recurrence fails continuous integration automatically:
- **frontend guard — ESLint AST rule, NOT a text-regex scan (iteration-2 fix).** A source-text regex over
   implementation format implementation reference is brittle (false positives on already-humanised vars, false negatives on renamed fields).
  Use the existing **lint gate** instead: an ESLint no-restricted-syntax selector (no plugin needed)
  that flags a JSX Expression Container whose expression is a member access ending in
  Subject/Level/Subject Code/Level Code and is NOT an argument to a humaniser
  (subject Label/level Label/subject Level Label/interview Context Title) — i e. raw code fields
  reaching JSX. AST = precise; runs in the documented operational check (already a gate); per-line // eslint-disable-next-line …
  is the audited opt-out. If no-restricted-syntax can't express the "not inside a humaniser call"
  negative cleanly, fall back to a tiny local rule in an inline ESLint plugin. (The render-based
  Showcase Strip Pii Regression pattern is the model for *personal information output*, but it's per-component; for a
  repo-wide field-shape rule, lint/AST is the right tool.)
- **backend scan test** (Mapper No Code Concat Test): scan the private implementation folder**/service/** + **/dto/** for
  the code operation / the code operation display concatenation (the exact Match Dashboard Service
  defect). Fail with file:line; opt-out via a named constant for any genuine non-display use.

This turns "remember to humanise" into a build gate. Keep the Job Identity unit test (F1) and add one
Matches/Applications code-fixture render assertion as a fast smoke, but the scan is the durable
prevention. (Pairs with INT-139's mobile-gate enforcement — both make a standing review point a continuous integration
gate rather than a manual habit.)

---

## 5. Verification

Frontend (Node 20):


Implementation example omitted. This public edition retains the written requirements in this section.


- INT-138: Job Identity unit test green (title-present → title heading; title-null → label heading; code input → humanised, never raw); all three surfaces (Matches/Applications/Invitations) consume it; their bespoke identity markup deleted.
- INT-140: the ESLint raw-code rule fails the documented operational check on any un-humanised *Subject/*Level JSX render (green now = none); backend Mapper No Code Concat Test green — zero the code operation display concatenation; Invitations aria humanised; the 3 humanisers collapsed onto subject Level Label with all existing tests green.
- **Guard proof (non-vacuous):** temporarily re-introduce a raw render (the tutee's subject) → the documented operational check RED; a backend the get level code operation → Mapper No Code Concat Test RED; revert both → green. A guard that can't fail is a false green.

Mobile gate (INT-139) — against the redeployed stack:


Implementation example omitted. This public edition retains the written requirements in this section.


- All tutor card routes: scroll Width is at most client Width at 390px. Paste the green result into INT-139.

### Acceptance criteria (measurable)
1. Inspect rendered subject and level values; only explicitly reviewed exceptions may bypass the readable-label rule; lint green.
2. Job Identity is the sole identity-block in Matches/Applications/Invitations (their bespoke markup deleted); Identity Link secondary Label + interview Context Title delegate to subject Level Label (one label path).
3. frontend gate green (≥97.5 line / ≥88 branch); the two guards proven RED-on-break.
4. 390px mobile gate green on Matches/Applications/Invitations (run-now); continuous integration-enforce filed as the INT-064-blocked sub-task.
5. No backend behaviour change beyond the Mapper No Code Concat Test guard (137b/137a described as implemented in the historical record).

## 5b. Scope, sequencing & guard caveats (iteration-3)

**Mo S Co W — so the visible fix isn't held hostage to tooling:**
- **MUST (the actual findings):** Job Identity extraction + migrate all 3 surfaces (F1/INT-138);
  consolidate the 3 humanisers onto subject Level Label (F4); humanise the Invitations aria-label
  (F3); run-now 390px mobile gate (INT-139 part 1). These are correctness/consistency and depend on
  nothing external.
- **SHOULD (defense-in-depth, same wave if smooth):** the ESLint raw-code rule + backend source-scan guard
  (F6). High value (4×-recurring class) but heuristic — they back up the canonical-helper fix, they
  are not the fix. If either proves fiddly, ship MUST and file the guard as a tracked follow-up rather
  than block the visible correctness fix.
- **BLOCKED:** mobile-gate continuous integration enforcement (INT-139 part 2) — gated on INT-064.

**Guard caveats (don't oversell them):**
- The ESLint rule is a **field-name heuristic** (*Subject/*Level member access reaching JSX). It
  can false-positive (humanised value stored in a *Subject-named var) and false-negative (raw code
  in an off-name var); scope it to the known data transfer object code-field names to cut noise, and treat it as
  defense-in-depth, not proof. The primary prevention is the single subject Level Label sink +
  Job Identity.
- backend guard: a **dependency-free source-scan test** (read service/+dto/ sources, regex the
  the code operation display-concat shape) — do **NOT** add Arch Unit/new deps for this (no-new-deps
  rule); the pattern is narrow enough for a targeted scan.

## 6. Ship

Primarily frontend (Job Identity + 3 surface migrations + consolidation + ESLint rule); **one small backend
test-only change** (the Mapper No Code Concat Test guard — no backend main-source change, 137a/137b already
shipped). Build → frontend gate (lint+build+test:ci) + backend the documented operational check (guard test) + run-now mobile gate →
redeploy frontend → push on sp-dev (frontend + backend-test + spec) → update INT-138/139/140 rows to FIXED-DEV with
evidence; file the INT-139 continuous integration-enforce sub-task as INT-064-blocked. Team-reviewed + Boss conditions
folded (see Review history); a Boss spot-check on the built change closes it.
