# INT-094 + INT-108 — Soft-delete & moderation-integrity remediation (Wave C)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Spec 021 expands only invite/interview eligibility to non-deleted PENDING jobs; the deletion guard now follows each action instead of relying on ACTIVE-only intake.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Status:** **team-reviewed 2026-06-11 — conditions folded in** → Boss. **Date:** 2026-06-10
**Owners:** software-engineer (lead, INT-094 guards) + dba (INT-108 migration 0051, FK/index) + security-expert (consult: moderation-deleted job must not stay a live intake target)
**Tracker:** INT-094 (S2), INT-108 (S3) — from issues/PROJECT-REVIEW-2026-06-10.md
**Reserved migration number (this wave):** **0051** (latest in tree is 0048 ; 0049/0050a/0050 reserved by the **Wave B** spec (int-093) — leave the gap; final master-changelog include order: 0049 → 0050a → 0050 → 0051. Boss-corrected 2026-06-11).
**Scope rule:** soft-delete only (founder directive, SHARED-LESSONS md "Always soft-delete, never hard-delete"). No raw database structure definitions — one Liquibase changeset.

---

## INT-094 (S2) — A moderation-deleted job stays status is ACTIVE and still accepts NEW intake

### Problem
Admin Moderation Service delete Posting removes a job by setting **only** tutee posts deleted at; it never changes status. Every downstream lifecycle guard gates on status equals Post Status ACTIVE and **ignores deleted At**. So a job an admin removed for fraud/spam is still a fully live target for **new** applies, invitations, interviews, parent selections, and admin assign — reachable by direct application programming interface even after the one-time close cascade has terminated the *existing* intake. A removed-for-fraud job can still be matched and reach contact-reveal (personal information crossing).

### Current state (verified in code)

**delete Posting writes only deleted At, never status** — Admin Moderation Service implementation reference:


Implementation example omitted. This public edition retains the written requirements in this section.


The cascade closes **existing** open applications + pending invitations. It does **not** touch status, and it does **not** supersede an open (CREATED) tutor selection requests row for the post.

**Tutee Post has NO @database query language Restriction/@Where** — Tutee Post implementation reference (deliberate, with a comment):


Implementation example omitted. This public edition retains the written requirements in this section.


So **why do the guards still see a deleted post?** Because the deleted At IS no value filter lives **only in the named query methods** (Tutee Post Repository implementation reference — owner-list, public browse, admin queue, detail), while every lifecycle guard loads the post via **plain the find by id operation**, which has no such predicate. find By Id returns the deleted row unchanged → the status equals ACTIVE check passes → intake proceeds.

**The five (six) intake guards that gate on status equals ACTIVE only — each loads via plain find By Id:**

| # | Site | find By Id line | status equals ACTIVE guard line |
|---|------|-----------------|------------------------------|
| 1 | Job Application Service apply (public entry) | 99 the find by id operation | 105 the get status operation |
| 2 | Job Invitation Service invite | 123 the find by id operation | 133 the get status operation |
| 3 | Interview Service propose Interview | 150 the find by id operation | 161 the get status operation |
| 4 | Tutor Selection Service choose Tutor | 69 the find by id operation | 77 the get status operation |
| 5 | Admin Match Service create Match (assign) | 174 the find by id operation | 183 the get status operation |
| 6 | Job Invitation Service accept (folds into apply) | 254 the find by id operation / 385 re-check | 385 the get status operation |

(Site 6 is the invitation-accept path — it re-checks ACTIVE at 385 before folding into the apply→choose funnel; it must be covered by whichever option we pick, otherwise a removed job is still acceptable-into.)

Also leaked: an open CREATED tutor selection requests row for the deleted post survives delete Posting, so the admin selection-request queue (/admin/selection-requests) can still surface and assign the parent's choice on a removed job. Tutor Selection Request Status already has a SUPERSEDED value (Tutor Selection Request Status implementation reference).

### Design — **DECIDED (team-review 2026-06-11): Option (b)** — make delete Posting flip status → CLOSED and supersede the open selection request

Two options were on the table:

- **(a) — defensive: every status equals ACTIVE guard also requires deleted At equals no value.** Touch all six guard sites; add the get deleted at operation (return the existing * NOT ACTIVE 422, or a dedicated POST REMOVED).
- **(b) — corrective: delete Posting puts the post into a terminal state.** Set status CLOSED (mirroring the existing INT-062 close Live Job semantics) in addition to deleted At, and supersede any open CREATED selection request — so the *single existing* status equals ACTIVE guards already reject it with no per-site change.

**RECOMMENDED: (b)**, for these reasons:

1. **One write vs six guards.** (b) is a 3-line change at the single source of truth (delete Posting) plus the existing cascade helpers; (a) edits six call sites and creates a standing "did we add deleted At to the NEW guard too?" obligation for every future intake path (the exact required-contract-needs-both-ends / INT-082 drift class — a future seventh intake guard will silently forget it).
2. **State honesty.** A removed job sitting at status is ACTIVE is itself a data-integrity lie: ACTIVE means "open for intake," which is false. (b) makes status reflect reality, so analytics, the admin queue counts, and any future status-driven logic are correct for free.
3. **Mirrors an existing, reviewed pattern.** INT-062 already closes a live job to CLOSED and cascades apps/invitations — delete Posting already reuses those exact cascade queries (close Open Applications / close Pending Invitations at 271-272). Flipping status is the missing half of that mirror.

**Keep a thin (a)-style backstop where it is nearly free:** the most security-sensitive of the six — the **admin assign** path (Admin Match Service create Match) — additionally asserts deleted At equals no value as defence-in-depth, because assign is the step that creates a Match and unlocks contact-reveal (personal information). Belt (status flipped) + braces (assign re-checks deleted At). The other five rely on the now-correct status.

#### Exact changes

**1. Admin Moderation Service delete Posting (Admin Moderation Service implementation reference, around 266):**


Implementation example omitted. This public edition retains the written requirements in this section.


- the set status operation is guarded by equals ACTIVE so a delete of a DRAFT/PENDING/REJECTED/MATCHED/CLOSED post does not illegally rewrite its status (DRAFT/REJECTED/MATCHED/CLOSED remain closed to intake, while PENDING permits only the invite/interview actions explicitly added by spec 021). The post stays deleted at differs from no value regardless, so it is hidden from all listings via the existing query filters.
- **supersede Open For Post** — a new set-based bulk update records on Tutor Selection Request Repository flipping the single CREATED row for the post → SUPERSEDED (mirror close Open Applications's set-based shape; @Modifying @Query). Verify the existing CREATED-only partial-unique index (one open request per job) is honoured — SUPERSEDED is outside it, so no conflict.
- **Applicant/invitee notification fan-out is REQUIRED, not optional (team-review 2026-06-11):** notify the affected applicants and invitees, reusing the neutral close Live Job copy — **"A job you applied to is no longer available."** — **NEVER** the moderation/fraud reason. Capture the recipient ids BEFORE the bulk UPDATEs (INT-016 capture-before-clear discipline), persist in-app notification rows and outbox intents in the business transaction; external delivery/analytics run after commit (INT-125).
- Audit row (EVENT POSTING DELETED) and the structured log already exist (274-287); extend the log line with superseded Selections, and add a micrometer counter **moderation selections superseded** beside the existing closed-counters (Observability Do D, team-review 2026-06-11).

**2. Admin Match Service create Match defence-in-depth (Admin Match Service implementation reference, at the 183 guard):**


Implementation example omitted. This public edition retains the written requirements in this section.


**2a. Domain-predicate refinement (agreed, team-review 2026-06-11):** introduce action-specific named predicates, each requiring deleted At equals no value: application/assignment eligibility retains ACTIVE; invite/interview initiation accepts ACTIVE or PENDING under spec 021. Do not use one widened predicate for every action. Every action checks the soft-delete condition directly, including deleted PENDING rows whose status is intentionally preserved.

**3. Propagate the soft-delete guard to every action.** ACTIVE deletion closes status, but PENDING deletion keeps PENDING while setting deleted At. Invite/interview paths widened by021 must therefore check deleted At explicitly; status alone is insufficient. Verify apply, invite, accept, interview, selection and assign against both removed ACTIVE and removed PENDING posts. Keep each action's current error contract.

### Schema / migration (INT-094)
**None.** tutee posts deleted at, status, version all already exist (INT-065 migration + 0028c). The SUPERSEDED literal already exists in the tutor selection request status enum — **no ALTER TYPE ADD VALUE** needed (grep-confirm the enum changeset before writing the bulk update records, per the INT-034 "copy the literals, don't trust prose" lesson). All of INT-094 is code + one new repository method. (Migration 0051 below is INT-108's only.)

### Test plan (INT-094) — lanes: **[live-lab]** = Abstract Api IT → localhost (excluded from per-pull request continuous integration via ci-no-lab; rebuild+redeploy+reseed first); **[Testcontainers/per-pull request]** = runs in per-pull request continuous integration
- **[live-lab]** **Moderation Delete Closes Intake IT** — seed an ACTIVE job → submit operation for admin posts id delete → assert status is CLOSED AND deleted at differs from no value AND closed at differs from no value in database → then, against the SAME job:
  - tutor submit operation for assignments id apply → **422** TUTEE POST NOT ACTIVE (not 200).
  - tutee submit operation for tutee posts id invitations → **422** JOB NOT ACTIVE.
  - tutee POST .../interviews (propose) → **422** TUTEE POST NOT ACTIVE.
  - tutee submit operation for tutee posts id select → **422** TUTEE POST NOT ACTIVE.
  - admin submit operation for admin matches (assign) → **422** (caught by the new deleted At belt at create Match, and by CLOSED status).
  - invite-accept fold (Job Invitation Service accept) on the removed job → **422** (site 6).
- **[live-lab]** **Moderation Delete Supersedes Selection IT** — parent has a CREATED selection request → admin deletes the post → assert the selection request is SUPERSEDED and no longer appears in read operation for admin selection-requests; a subsequent assign on it → 4xx (not a Match).
- **[live-lab]** **Notification fan-out (REQUIRED)** — applicants/invitees of the deleted job each receive the neutral "A job you applied to is no longer available." notification; assert the moderation reason appears NOWHERE in the payload.
- **[live-lab]** **Non-regression guard** — deleting a DRAFT/PENDING post does NOT change its status (only deleted at set); assert status unchanged AND every now-eligible invite/interview action still rejects the deleted PENDING row. Existing open invitations/selection requests are closed/superseded without leaking moderation reasons; unresolved interview closure policy is tracked in CURRENT-REQUIREMENTS md#open-decisions.
- Any database-level/enum verification runs on **Testcontainers PG** (named enum + cascade), not H2 (INT-067 lesson) — i e. **[Testcontainers/per-pull request]**.

### Acceptance (INT-094)
1. After submit operation for admin posts id delete on an ACTIVE job: the post is deleted at differs from no value AND status is CLOSED AND closed at differs from no value; every new apply/invite/interview/select/assign on it returns 4xx (422/the existing not-active code, never 200, never a 500).
2. Any open CREATED selection request for the post is SUPERSEDED and disappears from the admin queue; applicants/invitees are notified with the neutral copy (never the moderation reason).
3. Admin assign additionally rejects a deleted At differs from no value post even if some other code path were to re-open status (defence-in-depth).
4. No new migration; no ALTER TYPE; soft-delete preserved (the row is never hard-removed).

---

## INT-108 (S3) — Favourites are physically deleted (founder soft-delete-rule violation)

### Problem
Favourite Service remove Favourite calls a Java persistence mapping **hard delete** — the only delete* in the private implementation folder — violating the absolute founder rule ("every delete is logical: status/flag, never a physical row removal — preserves audit/analytics/undo").

### Current state (verified in code)
- Favourite Service implementation reference:


Implementation example omitted. This public edition retains the written requirements in this section.


- Favourite Tutor Post Repository implementation reference — the delete by tutee id and tutor post id operation (physical DELETE).
- Favourite Tutor Post implementation reference comment is explicit: *"Soft-delete is not used — the row is removed when the tutee un-favourites."* No removed at/status column exists.
- Read paths that must learn to filter removed rows: exists By Tutee Id And Tutor Post Id (55, add-idempotency probe), find By Tutee Id Order By Created At Desc (36, the favourites list), find Tutor Post Ids By Tutee Id And Tutor Post Id In (47-51, the browse "is-favourited" membership probe), find By Tutee Id And Tutor Post Id (38).

### Design — **DECIDED (team-review 2026-06-11): soft-delete column removed at (migration 0051) CONFIRMED** — founder rule absolute; un-favourite churn is demand signal

**DECIDED: add favourite tutor posts removed at and soft-remove.** Rationale: this is the only Java persistence mapping hard delete left in the codebase; closing it makes the soft-delete rule absolute (no "favourites are special" carve-out a future engineer has to remember), and favourite/un-favourite toggling is exactly the signal a data-engineer would want for tutor-demand analytics (which tutors get bookmarked then dropped) — **un-favourite churn is demand signal**. A re-favourite then reactivates the prior row instead of inserting a new one — cleaner than the current insert-after-delete. Keep total-unique + reactivate-on-re-favourite (verified right: **uq fav tutee post from 0014 is total**; a partial unique would permit duplicate removed rows).

**Alternative (recorded, REJECTED in team-review 2026-06-11):** a founder exemption — declare a favourite an *ephemeral bookmark* with no audit/undo value and keep the hard delete. **Tradeoff:** saves one column + the filter wiring, but (i) breaks the "every delete is logical" invariant the Boss audits, (ii) discards the un-favourite signal forever, (iii) leaves the single inconsistency that a future review re-files. The cost of the column is tiny; recommend against the exemption.

#### Exact changes (soft-delete path)

**Model** — Favourite Tutor Post implementation reference: add


Implementation example omitted. This public edition retains the written requirements in this section.


and replace the "Soft-delete is not used" class comment with the soft-remove contract.

**Repository** — Favourite Tutor Post Repository implementation reference:
- **Delete the hard-delete method** delete By Tutee Id And Tutor Post Id (40-42).
- Add a set-based soft-remove:


Implementation example omitted. This public edition retains the written requirements in this section.


- Add f removed At IS no value to **every read** that means "currently favourited":
  - exists By Tutee Id And Tutor Post Id → exists By Tutee Id And Tutor Post Id And Removed At Is no value (used by add Favourite idempotency probe).
  - find By Tutee Id Order By Created At Desc → add when f removed At IS no value (the list, 36; convert to @Query keeping the @Entity Graph).
  - find Tutor Post Ids By Tutee Id And Tutor Post Id In (47-51) → add AND f removed At IS no value.
  - find By Tutee Id And Tutor Post Id (38) → keep returning the row regardless (needed to **reactivate** on re-favourite), but callers must check removed At.

**Service** — Favourite Service implementation reference:
- remove Favourite (91): mark that tutee’s saved tutor entry as removed at the current time — still idempotent (the removed At IS no value predicate makes a double-remove a 0-row no-op).
- add Favourite (54): re-favouriting a previously-removed pair must **reactivate** (set removed At no value) rather than append records a second row — otherwise the existing unique tutee id, tutor post id constraint 500s on the second favourite. Load via find By Tutee Id And Tutor Post Id: if a row exists with removed At differs from no value → clear it (reactivate, return yes); if removed At equals no value → idempotent no; if absent → insert as today. This preserves the current idempotency contract and the unique constraint.

### Schema / migration — **0051**
**Changeset id:** 0051-favourites-soft-remove (author int-108), included in db changelog-master implementation reference after the **0050** include (Boss-corrected anchor 2026-06-11 — final order 0049 → 0050a → 0050 → 0051; Wave B owns 0049–0050).
**database structure definitions intent (additive, online-safe):**
- add Column favourite tutor posts removed at timestamptz no value (null = active; nullable, no default — mirrors tutee posts deleted at / users deleted at).
- Optionally add an index to find a tutee’s active favourites in newest-first order. Keep the existing uniqueness rule across all records for each tutee–tutor-listing pair, including removed records. Saving a removed favourite again reactivates its existing record; a uniqueness rule covering only active records would permit unwanted duplicate history.
- the rollback that removes the added field.
- Plain add Column — **no ALTER TYPE** (this is a timestamp column, not an enum). Migrate-then-boot: the column must exist before the removed At-mapped entity jar boots under ddl-auto is validate (mirror the 0048 migrate-then-boot note).

### Test plan (INT-108) — lanes as INT-094
- **[live-lab]** **Favourite Soft Remove IT** — favourite → un-favourite → assert the row **still exists** with removed at differs from no value (NOT physically gone); read operation for favourites does not list it; the browse membership probe reports not-favourited.
- **[live-lab]** **Reactivate** — favourite → remove → favourite again → assert exactly **one** row (no unique-constraint 500), removed at cleared, add Favourite returned yes.
- **[Testcontainers/per-pull request]** **Idempotent remove** (service/unit) — double remove Favourite → second call is a 0-row no-op (no error).
- **[Testcontainers/per-pull request]** **List filter** (service/unit) — a removed favourite never appears in list My Favourites.

### Acceptance (INT-108)
1. remove Favourite performs NO physical delete — the row persists with removed at set (grep the private implementation folder for delete* Java persistence mapping calls → zero in Favourite Service/Repository).
2. Re-favouriting a removed pair reactivates the existing row (one row, no constraint violation).
3. All "is-favourited" reads filter removed at IS no value; removed favourites are invisible to list + browse probe.
4. Migration 0051 applied additively; gates green (backend ≥97.5/97.5).

---

## Owners + risk note

- **software-engineer (lead):** INT-094 delete Posting status-flip + selection-supersede + the assign deleted At belt; INT-108 service/repository soft-remove + reactivate.
- **dba:** migration **0051** (favourites removed at + optional partial index; confirm the existing unique constraint name and that it stays total); review the INT-094 supersede Open For Post bulk update records against the CREATED-only partial-unique index.
- **security-expert (consult):** sign off that a moderation-deleted job can no longer reach assign/contact-reveal (the personal information-crossing risk this issue creates).

**Risk notes:**
1. **INT-094 status semantics.** Flipping delete Posting to set CLOSED changes the post's status field — verify nothing keys off "deleted posts keep their pre-delete status" (none found: all reads filter deleted at IS no value so they never surface a deleted post's status anyway). The change is guarded by equals ACTIVE so non-ACTIVE deletes are untouched.
2. **Future-guard drift.** Option (b) intentionally does NOT add a deleted At check to all six guards — the invariant is upheld upstream. A future *seventh* intake path that loads a post by find By Id and gates on status equals ACTIVE is automatically safe **only because** delete Posting flips status; document this coupling in a code comment at delete Posting and at the assign belt so it is not silently broken. (This is the residual we accept for the 6-guards-vs-1-write tradeoff.)
3. **INT-108 reactivate-vs-unique.** The unique tutee id, tutor post id must stay **total** (not partial on removed at) and re-favourite must reactivate the existing row — otherwise a re-favourite append records 500s on the constraint. Pin with the reactivate IT.
4. **Migrate-then-boot.** 0051 must be in the master changelog and applied before the removed At-mapped entity boots (ddl-auto is validate) — reseed the dev stack after deploy before regression ITs (live-stack lesson).
5. **Coverage denominator.** Adding the new repository methods + branches (reactivate path, removed at filters) shifts the JaCoCo denominator — cover the reactivate and idempotent-no-op branches explicitly (modal/secondary-path coverage lesson) to hold ≥97.5/97.5.
