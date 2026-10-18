# Batch Review — Open-Issue Specs + user interface/user experience Redesign (2026-06-03)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Spec-and-review phase for the open-issue backlog. 8 specs authored by the owning expert agents
(concurrently), one combined 18-agent /team-review, then the mandatory Boss round.

**Verdict: ✅ SHIP IT** (specs approved to proceed to phased implementation) — Rex, with conditions
folded into the specs. No spec rejected.

## Specs in this batch
| Issue | Spec | Migration | Owning experts |
|---|---|---|---|
| INT-049 | uiux/redesign-modern-readable-spec md | — (tailwind config) | uiux-designer + frontend-designer + frontend-expert |
| INT-034 | architecture/int-034-free-market-assign md | 0033 | software-engineer + dba + bff |
| INT-039 | architecture/int-039-interview-negotiation md | 0034 | solution-architect + swe + dba + bff + uiux |
| INT-043 | architecture/int-043-tutor-onboarding-completion md | none | product-designer + frontend-expert + swe + uiux |
| INT-044 | architecture/int-044-job-invitations md | 0035 | solution-architect + bff + dba + uiux |
| INT-045 | architecture/int-045-analytics-date-range md | 0037 | data-engineer + bff + frontend-expert + swe |
| INT-046 | architecture/int-046-fe-coverage-closeout md | — | qa-engineer + frontend-expert |
| INT-025 | architecture/int-025-admin-submit-notification md | 0036 | software-engineer + bff + product-designer |

**DBA migration ledger:** 0032 = M19 OAuth (reserved) · 0033 = INT-034 · 0034 = INT-039 ·
0035 = INT-044 · 0036 = INT-025 · 0037 = INT-045. (INT-045 reassigned from 0032 after a
collision was caught in review.)

## Boss-approved build order (enforced)
**INT-046 → INT-034 → INT-049 Phase 0 → INT-043 → INT-045 → INT-025 → INT-039 → INT-044.**
Rationale: INT-046 is a merge gate (coverage is red — nothing merges green until fixed); INT-034 is the
only spec fixing a broken core flow (free-market assign impossible today = lost commission); INT-049
Phase 0 (token wiring) precedes INT-046's snapshot-sensitive admin tests; INT-043 sets the gate
contract INT-044's accept path depends on; INT-039/044 are highest-coupling new surface — built last,
sequentially, not in parallel.

## Conditions (folded into the specs)
- **INT-034:** add 0033f partial-unique tutor user id, tutee post id when status active — the
  legacy uq matches pair is a no-op for free-market matches (both tutor post id NULL).
- **INT-039:** expiry-job metric+alert+idempotency; IT asserting INT-016 lazy-init safety; recreate the
  0021 cap index for the widened active set.
- **INT-044:** add a Measurement section (invite created/accepted/declined/expired); finalize the
  accept-error (422 TUTOR PROFILE INCOMPLETE) contract after INT-043.
- **INT-045:** confirm no new date-lib dependency.
- **INT-049:** personal information non-regression test on cards/invitation surface; Phase 0 before INT-046 admin tests;
  one named owner for the matching/applications domain across INT-034/039/044.

## Deploy discipline (all schema specs)
Migrate-then-boot in ledger order; each ALTER TYPE ADD VALUE is its own run In Transaction is no
changeset and must commit before the jar referencing the new label boots (the 0025 SUPERSEDED lesson).
Verify INT-034 backfill post-migrate (the count operation = 0) as a release gate.
ITs run against the live stack — rebuild/redeploy (+reseed) before the documented operational check.

## Next milestone
INT-046 green (frontend coverage ≥97.5%) + INT-034 implemented with the 0033f guard, verified on the live
dev stack.
