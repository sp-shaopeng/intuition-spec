# Tasks — Landing page trust and social proof (005)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Implementation tasks broken from [design md](design.md). Each task is small, names the **owning specialist**,
> traces to one or more **REQ-NNN**, lists the **exact files** to touch, and a **per-task verification step**.
> The Boss **dispatches the owning specialist** for each — the orchestrator does not build these itself. Check a
> box only when its evidence is captured.

| # | Task | Traces to | Owner (specialist) | Files | Verify | Status |
|---|------|-----------|--------------------|-------|--------|--------|
| 1 | Capture live + code evidence that the #161 stats-band reframe is real (shield glyph + "Always private", no bare 0) | REQ-001 | uiux-designer | Landing implementation reference (read); evidence/REQ-001/ | Playwright @1280+@390 of live stats band + a11y snapshot showing img Always private & no 0; read Privacy Glyph/Stat Item | [x] DONE 2026-06-20 (evidence captured) |
| 2 | Close GitHub #161 with the REQ-001 evidence bundle | REQ-001 | issue-engineer | GitHub #161 | gh issue close 161 with screenshot + code ref in the comment; verify state is closed | [ ] |
| 3 | Record REQ-002 (real adoption metric) as DEFERRED / out-of-scope-for-now in the backlog | REQ-002 | business-analyst | this spec + issue backlog | A backlog note exists linking #161's "honest volume metric" as deferred pending a founder number / non-trivial aggregate | [ ] |
| 4 | Finalise the testimonial-strip DESIGN: placement (after parents How-It-Works, before stats), markup (blockquote/cite), tokens, @1280/@390 responsive, empty state | REQ-003, REQ-009 | uiux-designer (+frontend-expert) | design md; (later) Testimonial Strip implementation reference, Testimonial Card implementation reference | design md section reviewed + approved at team-review; no carousel; states defined | [ ] |
| 5 | Decide the testimonial SOURCE: Option A (admin featured flag + read operation for reviews featured + Liquibase column) vs Option B (founder-approved real review-ID list, no migration) | REQ-005 | bff-specialist + dba + data-engineer | design md ADR; (Option A) Review Controller implementation reference, Review Service implementation reference, Review Repository implementation reference, db/changelog/, Security Config implementation reference, api Routes implementation reference | ADR records the choice; if A, contract + changeset id specified; honesty: every card maps to a real review row | [ ] |
| 6 | **BLOCKER — collect founder content**: real consented parent quotes OR written approval to feature specific real M11 reviews (by id) + attribution style | REQ-007, REQ-008 | business-analyst <- founder | evidence/REQ-007/ | evidence/REQ-007/ holds the quotes/approved-ID list + consent note; security-expert confirms personal information-safe attribution | [ ] **BLOCKED (founder content)** |
| 7 | **BUILD the testimonial strip** with the REAL approved content (Task 6) on the chosen source (Task 5) | REQ-004, REQ-003, REQ-005, REQ-006 | frontend-expert (+ bff-specialist/software-engineer if Option A) | Testimonial Strip implementation reference, Testimonial Card implementation reference, Landing implementation reference, (Option A) backend files from Task 5 | the documented operational check clean; the documented operational check ≥97.5% line/≥88% branch; every card traces to a real source row | [ ] **BLOCKED on Task 6** |
| 8 | a11y + CWV pass on the built strip (blockquote/cite, labelled section, AA contrast, reduced-motion, no carousel dep, CLS ~0) | REQ-006 | frontend-expert + uiux-designer | the strip components; package implementation reference (assert no new carousel dep) | axe = 0 serious/critical; Playwright reduced-motion capture; git diff package implementation reference = no carousel dep | [ ] **BLOCKED on Task 7** |
| 9 | Capture live build evidence of the strip @1280 + @390 + the zero-content state | REQ-003, REQ-009, REQ-004 | qa-engineer / uiux-designer | evidence/REQ-003/, evidence/REQ-009/ | Playwright @1280+@390 of the rendered strip + a capture of the empty/absent state | [ ] **BLOCKED on Task 7** |

## Notes
- Independent tasks run in parallel; dependent ones serialize. **Tasks 1-3 (the #161 lane) are independent of the
  testimonial lane and can complete now.** Tasks 4-5 (design + source decision) can proceed in parallel and are
  NOT blocked. **Tasks 7-9 are HARD-BLOCKED on Task 6 (founder content) — do not dispatch until REQ-007 is met.**
- A task is DONE only when its REQ-NNN evidence is captured under the private evidence directory for the requirement and opened by the reviewer.
- Honesty gate (Boss + business-analyst): no task may render a quote, name, rating, or number that does not trace
  to a real review row or a consented founder quote. A fabricated testimonial fails sign-off.
- On completion of all (unblocked) tasks, update spec md status + version/changelog to reflect what shipped.
  If the testimonial lane stays blocked, ship the #161 lane and leave the testimonial REQs OPEN/BLOCKED.

## 2026-09-05 correction follow-up — implementation not verified

- [ ] **REQ-005/008:** define and verify both source variants and the sanitized public attribution projection; no raw reviewer Display Name binding, no invented stars, no publication without consent.
- [ ] **REQ-007 — founder:** provide actual quote/review IDs, homepage publication permission and approved attribution. Keep build blocked until supplied; transport choice remains recorded at design sign-off.
