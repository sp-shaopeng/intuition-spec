# INT-093 (batch) — Commission / Match / Application state-machine & concurrency integrity

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The resolved application state rules below remain binding: SUPERSEDED/CLOSED may reactivate; REJECTED/MATCHED may not. Money status guards are only part of the workflow. PAID confirms actual agency receipt of tutor commission. Preserve [product requirements document §8.3/§20.3](../PRD.md) first-session confirmation and invoice obligations; this batch does not waive them or authorize parent-payment/owed-money substitutes. Exact exceptional/advance-payment policy remains in [open decisions](../CURRENT-REQUIREMENTS.md).

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** spec — **team-reviewed 2026-06-11 — conditions folded in** → Boss · **Date:** 2026-06-10
**Wave:** B (state-machine & concurrency integrity seams) · **Source:** issues/PROJECT-REVIEW-2026-06-10.md (INT-093, INT-102…107)
**Owners:** software-engineer (lead) + dba · review pair: security-expert (INT-104 FK/role), qa-engineer (race ITs)
**Tracker rows covered:** INT-093 (S2), INT-102 / INT-103 / INT-104 / INT-105 / INT-106 / INT-107 (S3)
**Reserved migration numbers (this wave only):** 0049 (Match @Version), 0050 (selection-request FKs). Leave all other numbers for other waves.

> **Out of scope here (other waves, do NOT touch):** INT-091/092 (the get tutor post operation missing-value failure on free-market matches → the resolve tutor user id operation), INT-094 (deleted At not checked on ACTIVE-guards), INT-108 (favourite hard-delete). These services are read for context only; this wave changes **state-transition guards + concurrency tokens + referential integrity**, nothing else. Where a fix here touches a line a sibling wave also touches (e g. Admin Commission Service pay Commission), the two changes are additive and non-conflicting — sequence per the note at the end.

---

## Theme

The security/authz/personal information core is clean (review Part 2). Every defect in this batch is an **illegal state transition or an unguarded check-then-act** on the commission/match/application machine — a row reaching a status its lifecycle forbids, or two admins racing a load→check→write with no concurrency token. The fix shape is uniform: **(a) a from-status / target-status guard with the right web request protocol code, (b) an optimistic-lock token where the race is concurrent rather than sequential, (c) referential integrity (FK + role check) so a "tutor" id is always a real TUTOR.**

Project rules in force (SHARED-LESSONS): @Version + ddl-auto is validate is **migrate-then-boot** (column present before the jar boots; mirror 0028c/0048); the Object Optimistic Locking Failure Exception→409 handler is **entity-aware** (Global error handler optimistic Lock Entity); a 422 is "you sent a legal request against an illegal state", a 409 is "concurrent/duplicate", a 404 is "no such resource (don't leak)"; soft-delete only; no raw database structure definitions.

---

## INT-093 (S2) — A CANCELLED commission can be marked PAID, resurrecting a CANCELLED match to COMPLETED

### Problem
The money path has exactly one guard (equals PAID). A CANCELLED commission (set by cancel Match) passes that guard, gets flipped to PAID, and the set status operation runs **unconditionally** — so a voided commission on a cancelled match is "completed" while the job may already be re-assigned. Analytics inflate; contact-reveal re-opens (COMPLETED is revealable).

### Current state (verified)
Admin Commission Service implementation reference:


Implementation example omitted. This public edition retains the written requirements in this section.


- Commission Status = PENDING | PAID | CANCELLED (verified). The guard ignores CANCELLED.
- cancel Match sets a PENDING commission → CANCELLED (Admin Match Service implementation reference) and reopens the job MATCHED→ACTIVE (). So a CANCELLED commission with an ACTIVE (re-assignable) or CANCELLED match is a real, reachable state.
- Match Status = PENDING | ACTIVE | COMPLETED | CANCELLED (changeset 0008). Pay is only legal on a match that is **PENDING or ACTIVE** — a payment completes an in-progress placement, never a cancelled one.

### Design / fix (Admin Commission Service pay Commission)
Replace the single equals PAID guard with two explicit guards, evaluated **after load, before any write**:

1. **Commission must be PENDING** — the get invoice status operation. This subsumes the old PAID guard (a PAID commission is not PENDING) and newly rejects CANCELLED. Keep the existing COMMISSION ALREADY PAID code for the PAID case specifically (the private implementation folder contract depends on it): branch — equals PAID → 409 COMMISSION ALREADY PAID;  equals CANCELLED or anything ≠ PENDING → 409 COMMISSION NOT PAYABLE ("This commission has been cancelled and cannot be paid.").
2. **Match must be in a payable status** — after resolving the get match operation, assert the get status operation. Else 409 MATCH NOT PAYABLE ("The match is no longer active; commission cannot be settled."). Only then flip COMPLETED + stamp match Completed At.

Both are **pure status guards** (no schema). Order: commission guard first (cheaper, the primary contract), then match guard. New error constants ERR COMMISSION NOT PAYABLE COMMISSION NOT PAYABLE, ERR MATCH NOT PAYABLE MATCH NOT PAYABLE. (Belt-and-braces with INT-105's @Version: @Version guards the *concurrent* pay-vs-cancel race; these guards block the *sequential* illegal transition the @Version cannot see.)

### Schema / migration
None.

### Test plan (lanes: **[live-lab]** = Abstract Api IT → localhost, excluded from per-pull request continuous integration via ci-no-lab, needs rebuild+redeploy+reseed; **[Testcontainers/per-pull request]** = runs in per-pull request continuous integration, no live stack)
- **[live-lab]** Admin Commission Pay IT pay Cancelled Commission returns409 Not Payable — create match → cancel (commission→CANCELLED, job→ACTIVE) → submit operation for admin commissions id pay → **409 COMMISSION NOT PAYABLE**; assert commission stays CANCELLED and match stays CANCELLED (no resurrection). (Kept as-is — team-review confirmed.)
- **[live-lab]** pay Already Paid Commission returns409 Already Paid — pay once (200) → pay again → **409 COMMISSION ALREADY PAID** (preserve existing contract).
- **[Testcontainers/per-pull request]** pay Commission On Completed Match returns409 Match Not Payable — **REPLACED (team-review 2026-06-11): a unit/Testcontainers FORCED-STATE test, not a live-application programming interface IT.** The state (match COMPLETED + commission PENDING) is not constructible via the live application programming interface — pay is the only completer and it flips the commission PAID first; cancel Match cancels both (Admin Match Service implementation reference). Force the state directly (repository/entity setup), invoke pay Commission, assert **409 MATCH NOT PAYABLE**. The match-status guard stays in the code (defence-in-depth) but is proven by this forced-state test so the JaCoCo branch gate holds.
- **[live-lab]** pay Pending Commission On Active Match succeeds — happy path 200, match→COMPLETED, commission→PAID.

### One-off data audit (team-review 2026-06-11)
Run an audit query for pre-existing PAID-commission-on-CANCELLED-match rows — the analytics-inflation legacy of the missing guard (Commission Repository sum Paid Commission Amount Between filters only invoice Status+paid At, Commission Repository implementation reference, so such rows inflate collected-commission analytics): find paid commissions linked to cancelled matches. Record findings in the tracker; **no automated backfill unless rows exist.**

### Acceptance
A CANCELLED (or any non-PENDING) commission can never be marked PAID; a non-PENDING,ACTIVE match can never be set COMPLETED by a payment; the existing already-paid 409 contract is unchanged; gates green.

---

## INT-105 (S3) — Match has no @Version: activate↔cancel admin race is an unguarded check-then-act

### Problem
activate Match and cancel Match are load→check→write under READ COMMITTED with no optimistic-lock token. Two admins (cancel first, activate second) both read PENDING; cancel commits (job reopened, commission CANCELLED) then activate commits → an ACTIVE match + live commission on a job that is open for re-assignment. Double-activate is backstopped by commissions match id unique; **cancel-vs-activate has no backstop.**

### Current state (verified)
- Match implementation reference — **no @Version field** (verified: fields end at the match completed at repairfragment0 marke rresolve tutor user id operation is the last member). Contrast Tutee Post version (0028c) and Commission version (Commission implementation reference, 0048).
- Admin Match Service activate Match (+) and cancel Match () both find By Id → status-check → set Status → save with no version compare.
- The entity-aware 409 handler already exists: Global error handler handle Optimistic Lock () branches on the optimistic lock entity operation (), which reads the get persistent class name operation with a message-scan fallback, mapping Tutee Post→JOB ALREADY ASSIGNED, Commission→COMMISSION ALREADY PAID, else CONCURRENT MODIFICATION.

### Design / fix
1. **Add @Version to Match** — mirror Commission implementation reference exactly:


Implementation example omitted. This public edition retains the written requirements in this section.


   (@Builder Default is mandatory — Match is @Builder-constructed in create Match; without it the builder writes no value into a NOT-NULL column.)
2. **Extend the entity-aware 409 handler** for Match — in optimistic Lock Entity add classify a conflict involving a match as a match conflict **and order it AFTER the Tutee Post/Commission checks** (substring "Match" would otherwise also match nothing else here, but keep the named entities first for clarity); in handle Optimistic Lock add return a specific match-conflict error telling the administrator to reload and retry. The generic CONCURRENT MODIFICATION fallback stays for any future entity.
3. No service-logic change — the optimistic-lock conflict surfaces at commit on the second writer; the existing sequential status guards (CANCELLABLE STATUSES, the activate status check) remain as the friendly pre-check.

> **Accepted inaccuracy (team-review 2026-06-11):** the entity-aware @Version handler maps a Commission conflict to COMMISSION ALREADY PAID even when a concurrent CANCEL (not a pay) won the race (Global error handler) — accepted; the client refetch self-corrects.

**Ordering rule (SHARED-LESSONS):** @Version + ddl-auto is validate ⇒ the version column MUST exist in the database **before** the new Match jar boots, or startup crash-loops on validate. Therefore migration 0049 (below) is included after 0048 in the master changelog and is applied (migrate-then-boot) before the code referencing version deploys.

### Schema / migration — 0049
File: 0049-match-optimistic-version implementation reference (mirror 0048-commission-optimistic-version implementation reference exactly):
- **Changeset id 0049-match-optimistic-version, author int-105** — add Column table Name is matches, column version type is bigint default Value Numeric is 0 constraints nullable is no; the reviewed database rollback drops the column. Additive, default 0 → online/expand-contract-safe.
- Master include: add  implementation reference immediately after the 0048 include (end of db changelog-master implementation reference), with a comment citing INT-105 and the 0028c/0048 precedent.

### Test plan
- **[Testcontainers/per-pull request]** Match Optimistic Lock IT concurrent Activate And Cancel one Wins409 — **MUST be Testcontainers/per-pull request (team-review 2026-06-11): a live-lab race test will flake or get skipped, shipping the @Version backstop unproven.** Start two threads on one PENDING match (one activate, one cancel); assert exactly one succeeds and the other returns **409** (MATCH CONCURRENTLY MODIFIED or the entity-aware code); assert the final database state is internally consistent (NOT "ACTIVE match + reopened job"). Per SHARED-LESSONS the boundary happy-path test is insufficient — the evidence test is the concurrent burst asserting the final count/state.
- **[Testcontainers/per-pull request]** Unit: Global error handler Test match Optimistic Lock maps409 — feed an Object Optimistic Locking Failure Exception naming Match → assert the MATCH CONCURRENTLY MODIFIED code (and that Tutee Post/Commission mappings still resolve, i e. the new branch didn't shadow them).
- **[live-lab]** Migrate-then-boot smoke: after deploy, confirm that migration 0049 was recorded and that matches have a required whole-number version field with a default of zero.

### Acceptance
A concurrent activate-vs-cancel on the same match resolves to exactly one winner; the loser gets a 409, not a corrupt dual-write; existing single-admin flows unchanged; version column present before boot; gates green.

---

## INT-102 (S3) — Application withdraw has no from-status guard: MATCHED → WITHDRAWN allowed

### Problem
withdraw only short-circuits the already-WITHDRAWN case, then flips any status to WITHDRAWN. The winning tutor can withdraw a **MATCHED** application while the Match stays ACTIVE: the parent's applicant view shows the assigned tutor as withdrawn, match source Application points at a WITHDRAWN row, and a later cancel's reopen Applications (REOPENABLE = SUPERSEDED|MATCHED) no longer reopens it.

### Current state (verified)
Job Application Service implementation reference:


Implementation example omitted. This public edition retains the written requirements in this section.


Job Application Status = APPLIED | WITHDRAWN | SHORTLISTED | REJECTED | MATCHED | SUPERSEDED | CLOSED (verified). Withdraw is the *tutor's* "I'm no longer interested" action — only meaningful from an **open, not-yet-decided** state.

### Design / fix
After the WITHDRAWN no-op early-return, add a from-status allow-list:


Implementation example omitted. This public edition retains the written requirements in this section.


Allowed: APPLIED, SHORTLISTED. No-op (idempotent return): WITHDRAWN. 422 APPLICATION NOT WITHDRAWABLE: MATCHED, SUPERSEDED, REJECTED, CLOSED. (MATCHED → the tutor must go through cancel, which is admin-mediated and reopens correctly; the tutor cannot unilaterally desync a live match.)

### Schema / migration
None.

### Test plan
- **[live-lab]** Job Application Withdraw IT withdraw Matched Application returns422 — assign the tutor (app→MATCHED) → tutor DELETE/POST .../withdraw → **422 APPLICATION NOT WITHDRAWABLE**; assert app stays MATCHED and the match stays ACTIVE.
- **[live-lab]** withdraw Superseded Application returns422 — same for SUPERSEDED.
- **[live-lab]** withdraw Applied Application succeeds + withdraw Already Withdrawn is No Op200 — preserve happy path + idempotency.

### Acceptance
Withdraw succeeds only from APPLIED|SHORTLISTED; WITHDRAWN is an idempotent no-op; every other status 422s without mutating; a MATCHED application can never be desynced from its live match by the tutor.

---

## INT-103 (S3) — Re-apply over a non-APPLIED/non-WITHDRAWN pair row blind-INSERTs → unique-index 500

### Problem
apply/apply Internal handle exactly two existing-row cases — APPLIED (idempotent return) and WITHDRAWN (reactivate). Any other pair status (SHORTLISTED, REJECTED, MATCHED, SUPERSEDED, **CLOSED**) falls through to a fresh append records, which the uq job applications active pair partial-unique (when status WITHDRAWN, 0018) rejects → Data Integrity Violation Exception → **500**. Reachable via the block→unblock cascade: cascade Block Tutor sets the tutor's apps CLOSED while their jobs stay ACTIVE; unblock → re-apply → 500.

### Current state (verified)
Job Application Service implementation reference:
- apply (): APPLIED early-return () → quota → **WITHDRAWN-only** reactivate (the find first by tutee post id and tutor id and status operation, ) → the or else get operation → save. A CLOSED/SHORTLISTED/etc. row is invisible to both lookups → new entity → append records → 500.
- apply Internal (): same shape () — same fall-through.
- The repo has the find first by tutee post id and tutor id and status operation (status-scoped) only; **there is no status-agnostic pair finder** (verified — needs to be added).

### Design / fix
Load the pair row **regardless of status**, then branch on its status — never blind-insert into a constrained pair.

1. **Repo:** add the find first by tutee post id and tutor id operation to Job Application Repository (there is at most one active row per pair by the unique index; for WITHDRAWN multiples, scope the finder Order By Updated At Desc to pick the latest — but in practice WITHDRAWN reactivates in place so only one row exists per pair).
2. **Single decision in a shared private helper** the resolve apply target operation used by both apply and apply Internal, by status:
   - no value (no row) → new Job Application (current behaviour).
   - APPLIED → idempotent return (current early-return preserved — keep it before the quota check so it doesn't 422 a capped tutor).
   - WITHDRAWN → reactivate the row → APPLIED (current behaviour; reactivated is yes).
   - SUPERSEDED | CLOSED → **reactivate to APPLIED** (SETTLED, team-review 2026-06-11: these are *system-imposed* states — an assign superseded the tutor / a block cascade closed them — so re-apply is recovery and must be unblocked; this is the block→unblock recovery path the founder's soft-delete model implies). Reuse the WITHDRAWN reactivation branch (set status APPLIED, update updated At); these are status WITHDRAWN-but-the-only-row cases so reactivating the SAME row keeps the unique index satisfied.
   - REJECTED → **409 APPLICATION REJECTED** (SETTLED, team-review 2026-06-11: a rejection is an *explicit human decision* that a silent re-apply must not undo — friendly 409, no mutation).
   - MATCHED → **409 APPLICATION ALREADY MATCHED** ("You are already matched on this job."). The tutor is the winner; a re-apply is meaningless and must not silently reset the row.
   - SHORTLISTED → **idempotent return** (already an active, favourable state — treat like APPLIED; do not 500, do not duplicate).
3. Net invariant: **the apply path never executes a second append records for an existing pair** — it either returns the existing row, reactivates it, or 4xxs. This kills the uq job applications active pair 500 for every status.

> **DECIDED (team-review 2026-06-11):** **SUPERSEDED/CLOSED = reactivate** (system-imposed states; unblock recovery) but **REJECTED = 409** (an explicit human decision a silent re-apply must not undo). The 500 dies either way; the ITs below assert exactly this split.

### Schema / migration
None (the existing uq job applications active pair is correct; the bug is the service blind-inserting against it).

### Test plan
- **[live-lab]** Job Application Reapply IT reapply Over Closed Row returns200 Reactivated — drive a pair to CLOSED (block→cascade), unblock, re-apply → **200**, one row, status APPLIED (NOT a 500, NOT a duplicate row). Assert the count operation = 1.
- **[live-lab]** reapply Over Superseded Row returns200 Reactivated — assign someone else (this tutor→SUPERSEDED), cancel/reopen, re-apply → 200 reactivated.
- **[live-lab]** reapply Over Rejected Row returns409 — admin rejects the application → re-apply → **409 APPLICATION REJECTED**, row stays REJECTED (pins the settled split).
- **[live-lab]** reapply Over Matched Row returns409 — tutor is MATCHED → re-apply → **409 APPLICATION ALREADY MATCHED**.
- **[live-lab]** reapply Over Shortlisted returns200 Idempotent and the existing APPLIED/WITHDRAWN cases — regression-pin no behaviour drift.

### Acceptance
No apply (direct or invite-accept) can 500 on uq job applications active pair; every existing-pair status maps to a deterministic 200-reactivate / 200-idempotent / 4xx; exactly one row per pair persists.

---

## INT-104 (S3) — choose-tutor / assign never validate the tutor id; tutor selection requests tutor user id / created by have no FK

### Problem
A parent's choose Tutor accepts any tutor Id as-is (own user Id, a TUTEE/ADMIN id, a random unique identifier); create Match resolves the winner via get Reference By Id (no select → nonexistent id 500s at flush on the matches FK; an existing **non-TUTOR** id inserts cleanly → a Match whose "tutor" is a tutee/admin, whose contact-reveal would then expose that user's personal information). Schema 0029 puts FKs only on tutee post id/job application id — tutor user id and created by are unconstrained, so a bad selection row persists silently.

### Current state (verified)
- service/Tutor Selection Service choose Tutor (): loads/owns/ACTIVE-checks the *post*, but tutor Id is used directly (the set tutor user id operation  / builder the tutor user id operation ) with **no existence or role check**. choose Tutor Internal () is the same.
- Contrast Job Invitation Service invite step 5 (): the find by id operation — the correct pattern to mirror.
- service/Admin Match Service create Match (): look up the selected tutor account by its identifier — a lazy reference, never validated; existence/role unchecked.
- 0029-tutor-selection-requests implementation reference: FKs on tutee post id (fk selreq tutee post) and job application id (fk selreq application, ON DELETE SET NULL) **only** — tutor user id and created by have no FK (verified).
- User carries the database query language restriction operation (verified) — find By Id already hides soft-deleted users, so the role-filter pattern also excludes deleted accounts for free.

### Design / fix
1. **Validate the tutor in choose Tutor and choose Tutor Internal** — before create-or-update, mirror invite step 5:


Implementation example omitted. This public edition retains the written requirements in this section.


   Use the **404 TUTOR NOT FOUND don't-leak** code (a non-TUTOR / nonexistent / deleted id are indistinguishable to the caller). Extract to a shared the assert is tutor operation private helper (both methods + reusable by create Match) to keep one source of truth (no-magic-strings; one code).
2. **Validate the winner in Admin Match Service create Match** — replace the blind get Reference By Id () with a validated load: the find by id operation. (422 here, not 404: the admin is assigning from an already-persisted selection request, so a bad id is an integrity failure of the request, not a missing path — surfaces distinctly from the parent-facing 404.) Use this tutor User to build the Match (it's already loaded, so no extra cost vs the reference).
3. **Add the missing FKs (migration 0050)** as the durable backstop — even with the service checks, a future insertion path must not be able to persist an orphan/non-existent tutor user id.

> Decision flagged for team-review: choose Tutor 404 (don't-leak, parent surface) vs create Match 422 (integrity on a stored request, admin surface). Both reject the bad id; the codes differ by surface. Confirm with security-expert that the parent 404 is the intended non-leaking response.

### Schema / migration — 0050
File: 0050-selreq-tutor-fks implementation reference, **changeset id 0050-selreq-tutor-user-fks, author int-104**:
- add Foreign Key Constraint fk selreq tutor user — base Table Name is tutor selection requests base Column Names is tutor user id → referenced Table Name is users referenced Column Names is id (no ON DELETE: users are soft-deleted, never physically removed, so RESTRICT/default is correct).
- add Foreign Key Constraint fk selreq created by — base Column Names is created by → the users operation.
- the reviewed database rollback drops both constraints.
- **Deterministic orphan-clean changeset 0050a (team-review 2026-06-11 — REQUIRED; 0050 must NOT rely on a human pre-flight):** add a preceding changeset **0050a-selreq-orphan-clean** that deterministically cleans orphan rows whose tutor user id/created by is not in users (set-based, soft-delete-aware) — or, equivalently, use require the rule ... NOT VALID then VALIDATE CONSTRAINT — so migrate-then-boot can never crash-loop on existing bad rows. Orphans are plausible: choose Tutor accepts any unique identifier today (Tutor Selection Service/111); the seeder is clean (Dev Data Seeder implementation reference). An add Foreign Key Constraint over violating data fails the migration and crash-loops the boot. Standard migrate-then-boot ordering stays: the FK changeset is included after 0049 in the master, applied before the constrained code path runs.
- Master include:  implementation reference after the 0049 include, comment citing INT-104.

### Test plan
- **[live-lab]** Choose Tutor Validation IT choose Non Tutor User returns404 — parent selects their own user Id / a TUTEE id / a random unique identifier → **404 TUTOR NOT FOUND**; assert no tutor selection requests row was created/updated.
- **[live-lab]** Admin Assign Validation IT assign Non Tutor returns422 — craft a selection request whose tutor user id is non-TUTOR (or assert the FK now prevents it) → assign → **422 INVALID TUTOR**, no Match created.
- **[Testcontainers/per-pull request]** selection Request Fk Rejects Orphan Tutor — a direct insert with a bogus tutor user id is rejected by fk selreq tutor user (proves the backstop, not just the service check — per "a fix is not done until the constraint actually constrains").
- **[live-lab]** Happy path: choose a real TUTOR → 200/201; assign → Match created.

### Acceptance
A non-TUTOR / nonexistent / deleted id cannot be selected (404) or assigned (422); the two FKs reject any orphan tutor user id/created by at the database level; personal information of a non-tutor can never surface via a malformed match; gates green; no pre-existing row violates the new FKs.

---

## INT-106 (S3) — Invitation accept silently re-points the parent's CREATED selection request to a different tutor

### Problem
When a tutor accepts an invitation, choose Tutor Internal UPDATEs an existing CREATED selection request **in place**, overwriting tutor User Id. Scenario: parent invites tutor A, later explicitly selects applicant B (CREATED request now points at B); A accepts the stale invitation → the request silently re-points to A; admin assigns "the parent's choice" — which is no longer the parent's choice. The parent's deliberate selection is overwritten by a counterparty action with no signal.

### Current state (verified)
service/Tutor Selection Service choose Tutor Internal (): identical create-or-update to choose Tutor — the find first by tutee post id and status operation (); if present, the set tutor user id operation () and save (the in-place overwrite); else insert. The ux selreq one open per job partial unique (0029b) enforces ≤1 CREATED request per job, which is exactly why the accept path *reuses* (overwrites) the existing row.

### Design / fix — **DECIDED (team-review 2026-06-11): skip+notify (create-if-absent on the accept path)**
The accept path (choose Tutor Internal) must **not overwrite a CREATED request that points at a different tutor**. Behaviour:
- No CREATED request for the job → create one for the accepting tutor (current else-branch). ✅
- A CREATED request already pointing at **this same** tutor → idempotent no-op return (the parent and the acceptor agree). ✅
- A CREATED request pointing at a **different** tutor → **do NOT overwrite — SETTLED: skip+notify.** Skip the selection-request write entirely (create-if-absent); the accept still folds into the apply→choose funnel (the tutor's application is created via apply Internal, INT-044), so the tutor becomes a selectable applicant — the parent can still choose them explicitly later. **And** emit an after Commit notification to the parent (Notification Type reuse) — "Tutor handle accepted your invitation and is now available to choose" — so the accept is visible without hijacking the existing selection. This preserves parent agency (the founder's admin-mediated model) and loses no information. (Notify-and-overwrite was rejected: it still discards the parent's choice.)
- **PLUS (REQUIRED frontend copy, team-review 2026-06-11):** show one generic accept-success message to the tutor on **ALL** invitation accepts — **"You've accepted — you're now an applicant for this job. The family confirms their final choice with our team."** — so a skipped accept isn't misleading and selection state never leaks to the tutor.
- Implementation: in choose Tutor Internal, when the get tutor user id operation → return the existing request unchanged with created is no (do not mutate), and register an after Commit parent notification (INT-066 rule: the in-tx selection row is untouched, the notification is an external-ish fan-out → after Commit/outbox, mirroring the project's notification pattern). choose Tutor (the *parent's own* explicit action) keeps the in-place overwrite — the parent IS allowed to change their own choice.

> Note the asymmetry is the whole point: **the parent may re-point their own CREATED request (choose Tutor); a tutor's accept may not (choose Tutor Internal).** Today both share one code path and both overwrite — the fix splits the accept-path semantics.

### Schema / migration
None. (If a parent-notification type doesn't exist, reuse an existing Notification Type value with the appropriate link Path — adding a new enum value is an ALTER TYPE ADD VALUE and would need its own run In Transaction is no changeset, so prefer reuse; confirm in team-review.)

### Test plan
- **[live-lab]** Invite Accept Selection IT accept Does Not Overwrite Different Chosen Tutor — parent invites A; parent selects applicant B (CREATED→B); A accepts → assert the CREATED request **still points at B** (not A); assert A now has an APPLIED application (foldable into the funnel); assert a parent notification was emitted.
- **[live-lab]** accept Creates Selection When None Exists — no CREATED request → A accepts → CREATED request created for A (regression of the legit path).
- **[live-lab]** accept Is Idempotent When Same Tutor Chosen — CREATED already → A → A accepts → no-op, still points at A.
- **[Testcontainers/per-pull request]** frontend (Vitest): the invitation-accept success state renders the generic copy ("You've accepted — you're now an applicant…") for both the create-selection and the skipped-accept responses — identical copy, no selection-state leak.

### Acceptance
A tutor's invitation-accept never overwrites a parent's CREATED selection that names a different tutor; the parent is notified; the tutor sees the same generic accept-success copy on every accept (no selection-state leak); the parent's own re-choose (choose Tutor) still works; gates green.

---

## INT-107 (S3) — Tutee can edit a MATCHED or CLOSED job's substance: only PENDING is blocked

### Problem
update rejects only status equals PENDING. ACTIVE correctly re-queues to PENDING (re-approval), but **MATCHED/CLOSED** (and REVISION REQUESTED/EXPIRED) fall through: subject, budget, requirements mutate in place with no status change and no re-approval — the economic context of an ACTIVE match (or a closed/filled job) can be silently rewritten by the tutee.

### Current state (verified)
service/Tutee Post Service update (): only guard is the get status operation (). was Active () → re-queue to PENDING on save (). Every other status (MATCHED, CLOSED, REJECTED, REVISION REQUESTED, DRAFT, EXPIRED) writes fields in place. Post Status = DRAFT | PENDING | ACTIVE | REVISION REQUESTED | REJECTED | MATCHED | CLOSED | EXPIRED (verified).

### Design / fix
Replace the single PENDING-block with an explicit **editable allow-list**: the tutee may edit only a post the marketplace has NOT yet committed —


Implementation example omitted. This public edition retains the written requirements in this section.


- **DRAFT** → free edit (no re-approval; not yet submitted).
- **REJECTED** → edit allowed (the resubmit/revise path the frontend needs — pairs with INT-098). Note: REJECTED edit does NOT auto-resubmit here; the existing submit flow handles that. (REVISION REQUESTED is intentionally excluded for now — it has no revise route today per INT-098; add it to EDITABLE only when that route lands, to avoid promising an unreachable state. Flag in team-review.)
- **ACTIVE** → edit allowed, re-queues to PENDING (current was Active behaviour, unchanged — re-approval).
- **PENDING** → 409 (unchanged: under review).
- **MATCHED | CLOSED | EXPIRED** → **409 INVALID STATE** (new): a committed/terminal job's substance is frozen.

Keep the existing 409 INVALID STATE code (frontend contract) with a status-aware message.

### Schema / migration
None.

### Test plan
- **[live-lab]** Tutee Post Update IT edit Matched Post returns409 — post→MATCHED → owner replace operation for tutee posts id changing budget → **409 INVALID STATE**; assert fields unchanged.
- **[live-lab]** edit Closed Post returns409 — CLOSED → 409.
- **[live-lab]** edit Active Post requeues To Pending and edit Draft Post succeeds — regression of the allowed paths (ACTIVE→PENDING, DRAFT stays DRAFT).
- **[live-lab]** edit Rejected Post succeeds — REJECTED edit allowed (no status flip).

### Acceptance
Tutee edit succeeds only from DRAFT|REJECTED|ACTIVE (ACTIVE→re-approval); MATCHED/CLOSED/EXPIRED/PENDING are 409; a live match's or closed job's economics can't be silently rewritten; gates green.

---

## Cross-cutting rules (apply to every item)

- **@Version ordering (INT-105):** the 0049 matches version column is included after 0048 and applied (migrate-then-boot) BEFORE the @Version-bearing Match jar boots — ddl-auto is validate crash-loops on a missing column. Same rule applied to Commission/0048 and Tutee Post/0028c.
- **FK ordering (INT-104):** the 0050 FKs are included after 0049; the **deterministic 0050a orphan-clean changeset** (team-review 2026-06-11 — never a human pre-flight) runs immediately before them so an FK over dirty data can never fail the migration and crash-loop boot (alternatively NOT VALID → VALIDATE).
- **web request protocol-code discipline:** illegal-state-on-a-legal-request = **422** (withdraw/edit/assign-bad-id); concurrent/duplicate/optimistic-lock = **409** (pay-not-payable, match version conflict, already-matched re-apply); missing/non-leaking resource = **404** (parent choose bad tutor). Never let any of these fall through to the catch-all 500 (INT-075 lesson — a constraint/missing-row must map to a 4xx).
- **No-magic-strings:** every new error code is a shared named error constants constant on the owning service; the role check is a shared the assert is tutor operation helper.
- **Verify-the-wiring:** for each fix, the IT must FAIL without the guard (drive the illegal transition and assert the 4xx) — an existence-only test proves nothing (SHARED-LESSONS "a fix is not done until the call site uses it, asserted by a test"). The @Version/FK backstops get a database-level test (concurrent burst / direct bad insert), not just a service unit test.
- **Test lanes (team-review 2026-06-11):** every test above is tagged **[live-lab]** (Abstract Api IT → localhost; excluded from per-pull request continuous integration via the ci-no-lab profile; needs rebuild+redeploy+reseed first) or **[Testcontainers/per-pull request]** (runs in per-pull request continuous integration, no live stack). Race/forced-state evidence tests MUST be [Testcontainers/per-pull request] so per-pull request continuous integration actually proves them.
- **Observability Do D (team-review 2026-06-11):** add micrometer counters — entity-aware optimistic-lock 409s **tagged by entity** (e g. competing-update conflicts grouped by match, commission or tuition-request record) and commission not payable — emitted in Global error handler, following the INT-087 counter pattern (e g. Outbox Poller implementation reference).

## Sequencing recommendation (do first → last)

1. **INT-093** (S2, money-path, no migration) — highest severity, pure service guard, ship first; unblocks the commission flow correctness independently.
2. **INT-102** + **INT-107** (no-migration status guards on application-withdraw and post-edit) — small, independent, parallelisable with #1 (distinct files/services).
3. **INT-103** (no migration; adds a repo finder + shared helper) — slightly larger; do after #2 so the application-state guards are coherent.
4. **INT-105** (migration 0049 + Match @Version + handler) — schema-bearing; serialise the migration with #5 (one ledger, no number collision). Migrate-then-boot.
5. **INT-104** (migrations 0050a orphan-clean + 0050 FKs, + role checks) — schema-bearing; the deterministic 0050a changeset (not a human pre-flight) clears violating rows before the FKs apply. Lands after 0049 in the master.
6. **INT-106** (no migration; behaviour split on the accept path + parent notification) — last; depends on the selection-request semantics being otherwise stable (touches the same Tutor Selection Service as #4's role check, so do them sequentially in that file, not concurrently — SHARED-LESSONS same-file collision rule).

Migrations land in numeric order on deploy: 0049 then 0050. The two schema items (#4 INT-104, #5 INT-105) and the Tutor Selection Service items (#5 INT-104, #6 INT-106) touch overlapping files (Admin Match Service, Tutor Selection Service, Global error handler, the master changelog) — build them **sequentially within the backend subtree**, never two agents on the same file (concurrency-safety rule). After all six: rebuild + redeploy + reseed dev, then the documented operational check against the live stack (regression ITs hit localhost), then push on sp-dev.
