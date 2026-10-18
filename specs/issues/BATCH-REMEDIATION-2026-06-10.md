# Batch Remediation Plan — INT-090…INT-124 (2026-06-10)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Source:** the whole-project loophole review [PROJECT-REVIEW-2026-06-10.md](PROJECT-REVIEW-2026-06-10.md)
filed 35 OPEN issues (2×S1, 10×S2, 19×S3, 4 batch rows). This plan groups them into **six fix waves**,
each with a standalone spec in specs/architecture/. Specs were authored by the owning experts against
the real code (every fix carries a verified file:line). **This plan goes to /team-review → Boss before
any build.**

## Wave specs (all written, code-grounded)

| Wave | Spec | Issues | Owners | Migrations |
|------|------|--------|--------|-----------|
| **A — M18 funnel completion (S1 revenue loop)** | [int-090-batch-m18-funnel-completion-spec md](../architecture/int-090-batch-m18-funnel-completion-spec.md) | INT-090, 091, 092, 097 | software-engineer + frontend-expert + bff-specialist | none |
| **B — State-machine & concurrency integrity** | [int-093-batch-state-machine-integrity-spec md](../architecture/int-093-batch-state-machine-integrity-spec.md) | INT-093, 102, 103, 104, 105, 106, 107 | software-engineer + dba | **0049, 0050** |
| **C — Soft-delete & moderation integrity** | [int-094-batch-soft-delete-integrity-spec md](../architecture/int-094-batch-soft-delete-integrity-spec.md) | INT-094, 108 | software-engineer + dba | **0051** |
| **D — Auth/session hardening** | [int-095-batch-auth-session-hardening-spec md](../architecture/int-095-batch-auth-session-hardening-spec.md) | INT-095, 096, 109, 111, 112 | frontend-expert + security-expert + software-engineer | none |
| **E — Marketplace & funnel user experience** | [int-098-batch-marketplace-funnel-ux-spec md](../architecture/int-098-batch-marketplace-funnel-ux-spec.md) | INT-098, 099, 100, 101, 113, 114, 115, 116, 117, 118, 119, 120 | uiux-designer + frontend-expert + bff-specialist | **0052 (conditional — see below)** |
| **F — Hygiene / polish** | [int-121-batch-hygiene-spec md](../architecture/int-121-batch-hygiene-spec.md) | INT-110, 121, 122, 123, 124 | devops + security + frontend-expert + software-engineer | none |

## Two findings that corrected the brief (fold into the build)

1. **INT-090 (Wave A):** the backend already has a purpose-built endpoint
   submit operation for admin selection-requests id assign (Admin Selection Request Controller implementation reference →
   Admin Match Service assign From Selection Request) that resolves tutor User Id server-side and is
   Idempotency-Key guarded. The unwired half is concretely an **frontend route-constant gap** (api Routes implementation reference
   has the list route but no assign). Wire the frontend to that endpoint keyed on selection Request Id —
   do **not** construct winning Tutor Id client-side. (Textbook INT-075 "both ends" signature.)

2. **INT-098 (Wave E):** rejection reason was **already widened to VARCHAR using 500** in migration
   0007 (entity confirms Tutee Post implementation reference). The "9 chars of notes" symptom is a **frontend
   constant** (Review Detail implementation reference), not a database limit. So **migration 0052's only
   possible job** is dropping a stale preset-list require constraint from 0005/0006 *if it is still live
   on the up-path*; the dba must read the up-paths and, if the require is already gone, **surrender 0052
   back to the ledger** rather than file a no-op. The real fix is the frontend cap (50→280) + the owner-data transfer object
   rejection Reason field.

## Migration ledger (latest applied = 0048)

- **0049** — Match.@Version optimistic lock (mirror 0048-commission-optimistic-version implementation reference). Wave B.
- **0050** — FKs on tutor selection requests tutor user id + created by (pre-flight no-violating-rows
  check). Wave B.
- **0051** — favourites removed at timestamptz no value (soft-remove). Wave C.
- **0052** — **SURRENDERED back to the ledger** (team-review 2026-06-11): 0007's up-path already
  dropped the require and widened rejection reason to VARCHAR using 500. No database work in Wave E.
- **0050a** — NEW (team-review condition): deterministic orphan-row cleanup (or NOT VALID→VALIDATE)
  preceding 0050's FK addition, so migrate-then-boot can never crash-loop on pre-existing bad rows.
- **Final db changelog-master implementation reference include order (Boss-pinned, 2026-06-11): 0049 → 0050a → 0050 → 0051.**
  Wave B owns 0049–0050; Wave C anchors 0051 after the 0050 include (NOT after 0048). One ledger,
  serialize the migration commits.

No ALTER TYPE ADD VALUE in this batch (INT-094 reuses the existing SUPERSEDED/CLOSED literals;
INT-104's role filter rides the existing @database query language Restriction). Gaps left for other waves as per the
one-ledger / no-collision rule.

## Decisions settled (team-review 2026-06-11) — DECISION LOG

Reviewed by a 4-lane consolidated team review (the private implementation folder · the private implementation folder · user experience/product/users ·
quality assurance/observability/measurement/docs) covering all 18 seats; every lane verdict was
APPROVE-WITH-CHANGES and the conditions are folded into the wave specs (marked
"team-reviewed 2026-06-11"). Decisions are final; dissent noted inline.

- **INT-103 — SPLIT ruling:** **reactivate** to APPLIED for SUPERSEDED/CLOSED pair rows
  (system-imposed states; unblock recovery), **409** for REJECTED (an explicit human decision a
  silent re-apply must not undo).
- **INT-106 — skip + notify:** create-if-absent only; after Commit parent notification; PLUS generic
  accept-success copy to the tutor on ALL invitation accepts so a skipped accept isn't misleading
  and selection state never leaks.
- **INT-094 — Option (b) close-on-delete**, refined: delete Posting flips status→CLOSED **and sets
  closed At**, supersedes the CREATED selection request, and the applicant/invitee fan-out is
  **required** (neutral copy, never the moderation reason). New named domain predicate
  the is open for intake operation is the single home of the intake rule (assign-path belt calls it).
- **INT-108 — soft-delete column** (migration 0051) confirmed; founder rule absolute; total-unique +
  reactivate-on-re-favourite (a partial unique would permit duplicate removed rows).
- **INT-117 — real withdraw-selection endpoint (option a):** the modal already promises
  changeability — keep the promise; soft-delete status flip on the selection request.
- **INT-101 — UNIFY** /requests into a role-aware /assignments; conditions: reuse existing
  card/filter/pager primitives ONLY, zero visual redesign, delete Browse Requests.
- **REVISION REQUESTED (INT-098) — remove from the frontend display mapping** (status unreachable — no
  admin path sets it; backend enum stays, no ALTER TYPE risk).
- **Migration 0052 — SURRENDERED:** verified 0007's up-path already dropped the rejection-reason
  require and widened to VARCHAR using 500 on both tables. INT-098's fix is frontend-only
  (REASON MAX LENGTH 50→280) + the owner-data transfer object field.
- **INT-095 — hardened design:** backend 422 for credential re-confirmation; the shared
  credential-reentry hook branches **422 → inline wrong-password; 401 → session expiry → logout**
  (closes the false-"Incorrect password" hole on expired sessions); breaking-test inventory in the
  spec.
- **INT-112 — accept double-entry:** don't stash the password; re-prompt on Commission (auto-login
  unaffected — the register POST happens there). Trade-off accepted explicitly.

## Cross-wave sequencing & collision safety (polyrepo — subrepos on sp-dev; serialize within the backend subtree)

- **backend-first within Wave A:** ship INT-091/092 (missing-value failure fixes) *before* INT-090 (frontend assign), so the frontend
  doesn't surface free-market matches that then 500 on pay/review.
- **Same-file collisions to serialize (one backend subtree, sequential builds):** INT-104 & INT-106
  both touch Tutor Selection Service; INT-104 & INT-105 both touch Admin Match Service + the master
  changelog; INT-093 & INT-091 both touch Admin Commission Service pay Commission (INT-093 is additive;
  apply the INT-091 the resolve tutor user id operation line first). Build these sequentially, not in parallel.
- **Team-review additions (2026-06-11):** Wave C's INT-094 also edits Admin Match Service create Match
  (the the is open for intake operation belt at the 183 guard) — the same method Wave B INT-104/105 rewrite →
  **B→C strictly sequential** in the backend subtree. db changelog-master implementation reference is a three-way collision
  (0049/0050+0050a/0051 includes) — serialize the migration commits. Wave F INT-121b (Coordinated Universal Time timestamps)
  edits Admin Commission Service, Admin Moderation Service,266, Tutee Post Service — files
  Waves A/B/C also touch → **INT-121b builds only after A–C land**.
- **frontend vs backend subtrees parallelize** (Wave D frontend-heavy, Wave B/C backend-heavy) — independent lanes are safe
  concurrently; builds within a subtree run sequentially.
- **INT-095 ↔ INT-111 coupling:** the auth login logout-exemption is load-bearing for both — implement
  together, with a negative-control test proving genuine session-expiry still logs out.
- **INT-123e (shared delete-account modal) is the single fix point for INT-095's frontend half** — extract the
  shared component once.

## Recommended build order

1. **Wave A** (S1, revenue path) — highest value, smallest surface; backend NPEs then frontend assign + tutor reveal.
2. **Wave B + Wave C** (backend integrity + migrations 0049–0051) — the money/state correctness core.
3. **Wave D** (auth/session) — touches the global interceptor; mandatory negative-control tests.
4. **Wave E** (user experience) — realizes the Part-3 P1 user experience wave (B1–B4); largest frontend surface.
5. **Wave F** (hygiene) — cheap wins, fold in opportunistically.

Each wave: build → the documented operational check (≥97.5/97.5) and/or the documented operational check (≥97.5/88) → /team-review →
Boss → deploy dev → push (current branch). Per the founder definition-of-done.

## Definition-of-done additions (team-review 2026-06-11)

- **Funnel proof (Wave A):** after Wave A deploys to dev, drive ONE free-market match end-to-end
  **through the user interface** (choose → assign → reveal both sides → pay → review) and record the evidence in
  the tracker. This is the exact INT-075 "both ends" class the batch itself fixes — application programming interface-only proof is
  insufficient.
- **Observability Do D per wave** (folded into each spec): A the number of commission-payment attempts, grouped by result counter ·
  B optimistic-lock-409 (by entity) + commission not payable counters in Global error handler ·
  C moderation selections superseded counter · D the number of registrations rejected because the phone is already in use +
  forced-logout instrumentation · E register the new frontend analytics events (selection withdrawn,
  post resubmitted) · F an IT proving the ADMIN-authenticated /actuator/prometheus scrape still
  works after gating.
- **Post-deploy success metrics:** (1) free-market job assigned → commission marked paid
  conversion (was 0%); (2) 5xx rate on pay/review endpoints → 0; (3) uq job applications active pair
  500s → 0; (4) spurious forced-logout rate → 0; (5) tutor-side contact-reveal count > 0.
- **Doc updates owed at ship time:** ① auth module doc — the 401→422 change on
  /auth/account + /auth/change-password is a BREAKING contract change + new REGISTRATION FAILED
  422; ② specs/modules/matching/spec md + api-reference md are stale (pre-INT-034 — no
  free-market match, no submit operation for admin selection-requests id assign) — rewrite with Wave A;
  ③ owner-data transfer object rejection Reason, interview-data transfer object context fields, withdraw-selection endpoint in module
  docs (Wave E); ④ migration ledger 0049–0051 + the 0052 surrender; ⑤ execute the
  ops/dev-vm-deploy md key-rotation edits (Wave F); ⑥ CLAUDE md module table after deploy.

---

*Team-reviewed 2026-06-11 (4-lane consolidated review covering all 18 seats; every lane
APPROVE-WITH-CHANGES; all conditions folded into the wave specs the same day — see the Decision Log
above). Boss audit follows; build may begin per the build order on Boss SHIP.*
