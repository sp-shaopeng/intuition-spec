# Feature Wave — Built & Shipped to DEV (2026-06-03)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Implementation of the Boss-approved open-issue backlog, orchestrated as delegated expert-agent builds, each
gate **independently re-verified by the orchestrator** (fresh JaCoCo CSV + clean Vitest v8 summary, with correct
exit-code capture — never the agent's word), per-item Boss sign-off, then this combined Boss review.

## Verdict: ✅ SHIP IT (whole wave, on the dev stack)

## Items (Boss-approved build order)
| Issue | What shipped | Migration | Verified gates (by orchestrator) |
|---|---|---|---|
| INT-046 | frontend coverage debt → green | — | frontend lines ≥97.5% |
| INT-034 | Free-market assign re-keyed on matches tutor user id; fixes contact-reveal/dashboard; 0033f dedupe | 0033 | backend 99.13L/98.91B, BUILD SUCCESS |
| INT-049 Ph0 | Design tokens wired into tailwind config + readability/motion baseline (brand locked) | — | frontend green; page redesigns Phases 1+ pending |
| INT-051 | backend branch debt → 99.0% (Resend Email Client/Public Browse/Auth Service genuine tests) | — | backend branch 96.49%→99.0% |
| INT-043 | Tutor onboarding fields optional at signup + post-login completion banner; apply-gate intact | none | backend 99.28L/99.0B, frontend 97.6L/92.63B |
| INT-045 | Analytics collected In Period Sgd + 6 created at/paid at indexes | 0037 | backend 99.28/99.0, frontend 97.6/92.63 |
| INT-025 | POST SUBMITTED admin fan-out (after-commit, failure-isolated, find By Role) | 0036 | backend 99.07L/98.81B |
| INT-039 | Interview timeslot negotiation (state machine, Slot Picker/Whose Turn/Timeline); 3 Boss conditions met | 0034 | backend 99.13/98.91, frontend 98.18/92.36 |
| INT-044 | Tutee→tutor job invitations; accept folds into apply→choose funnel (no new Match); 5 measurement events | 0035 | backend 98.92/98.13, frontend 98.02/92.27 |

**Migration ledger applied/registered:** 0032=M19 (reserved) · 0033(+f)=INT-034 · 0034=INT-039 · 0035=INT-044 ·
0036=INT-025 · 0037=INT-045. (Builds ran out of numeric order, so 0035's include is appended at the end of the
master changelog; Liquibase runs un-run changesets regardless of position — Testcontainers validates on fresh database.)

## Boss review (Rex) — capability + integrity audit
**Verification stance held:** every "green" was re-checked by the orchestrator against the real files. This caught,
this wave: (1) two agents claiming a coverage gate was "unmeetable locally" — disproven (0 skips, Mockito tests),
the false lesson corrected in SHARED-LESSONS; (2) an agent quoting a **stale** jacoco csv (numbers held on
recompute); (3) an enum-literal crash-loop (MATCHED not a match status) caught + fixed; (4) a **real** frontend
line-coverage gap on INT-039 (97.1% v8) the build agent missed — closed via a focused top-up to 98.18%; (5) two
build agents (INT-039, INT-044) ran 70+ / 52 min and the orchestrator took over their backend gates to get ground truth.
Net: **no false green shipped.**

**Invariants confirmed in code (not just gates):** INT-044 accept delegates to the apply internal operation+the choose tutor internal operation
with no direct Match creation (admin-only matching preserved); INT-039's 3 conditions present
(interview expired Micrometer counter + idempotent set-based expiry sweep; Interview Negotiation IT INT-016
no-500 guard; idx interviews tutee post active recreated over the widened active set); INT-043 empty-string
sentinel keeps the apply-gate blocking; INT-034 0033f partial-unique dedupe present.

**Capability upgrade:** ✅ delegated to the owning experts; lessons captured in SHARED-LESSONS (modal close-paths,
v8-summary-is-the-gate, stale-csv, enum-literal, migration-order, don't-rationalize-a-red-gate). Process nits:
two builds ran very long (opus marathons) and skipped self-gating; the orchestrator's monitoring + takeover covered it.

## Remaining (not done by this wave)
- **frontend container rebuild** for the new bundles (backend was redeployed during the wave; frontend dist built but the nginx
  container should be rebuilt to serve the new user interface on ).
- **Prod deploy** — separately gated; add this wave to PROD-DEPLOY-OWED with the migrate-then-boot order (0033→0037).
- **Doc follow-ups:** backend-api md (new endpoints: assign, interview negotiation ×N, invitations ×5, analytics range),
  sequence diagrams (negotiation, invitation), migration ledger 0033–0037 in database/migrations md.
- **INT-049 page redesigns** (Phases 1+), and deferred Tier-D features. INT-047/048 already filed.
