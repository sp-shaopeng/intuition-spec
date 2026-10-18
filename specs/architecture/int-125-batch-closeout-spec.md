# INT-125/128/129/134/135/136 — Close-out batch remediation spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** **team-reviewed (AMEND-FIRST → both amendments folded) + Boss ✅ SHIP IT 2026-06-12 — BUILD MAY START.**
**Boss rulings ratified:** INT-129 = indefinite ACTIVE+COMPLETED reveal (with the builder fence below); INT-136e = anonymous clamp-to-20, page-cap stays empty.
**Boss build-time notes (fold in, no re-review):** (1) ship THIS spec's network-error copy — "Cannot reach the server — check your connection and try again." — the tracker row's shorter wording is superseded; (2) the INT-136e contract doc must state the /assignments cap applies to ALL callers (it has no authenticated bypass, unlike /tutors). **Date:** 2026-06-12
**Owners:** software-engineer (lead; INT-125/135/134) + frontend-expert (INT-128/129/136a-d) + uiux-designer (consult: INT-128 copy, INT-129 product call, INT-136c symmetry) + qa-engineer (lane-tagged test plans, INT-134 fixture util)
**Tracker:** INT-125 (S3 backend), INT-128 (S3 frontend), INT-129 (S3 frontend product call), INT-134 (S3 test-infra), INT-135 (S3 backend), INT-136 (S4 frontend+backend batch) — issues/README md
**State of the world:** Waves A–E shipped (backend HEAD the recorded revision, frontend HEAD the recorded revision); migrations through **0051**; all in-app notification emits are IN-TX per the INT-132/133 rule (SHARED-LESSONS §4.1 split: database-row notifications in-tx; only external/REQUIRES NEW side-effects defer to after Commit).
**Migrations: NONE expected — verified.** Every change below is code/test/doc only: INT-125 writes notifications rows through the existing Notification Service emit; INT-135 moves two publishes into the existing run After Commit; INT-136e touches one controller branch. No new column, no enum, no index. (0052 remains surrendered per the Boss 2026-06-11 ruling.)

**Verified line-number drift vs the tracker rows (flagged, not blocking):**
- INT-125: tracker cites Admin Match Service implementation reference for the expire — it is now **** (Wave C/E edits shifted the file). Tracker also says "include them in the after-commit fan-out" — that row predates INT-132/133; the fan-out is now **in-tx** (notify Assign In Tx, ) and the new invitee fan-out must be in-tx too.
- INT-134: tracker says "4 sibling copies" — there are **FIVE** (Registration Enumeration IT implementation reference is a fifth, missed by the row). All five migrate.
- INT-136a: tracker cites Applicant Card implementation reference — the raw the applicant’s broad area interpolation is at **** (the  spans 71-74). A **sibling site the row missed**: Invitations implementation reference renders the invitation’s broad area, labelled as a region raw — in scope.
- INT-136e: tracker cites Public Browse Controller implementation reference — that is the **/public/tutors** cap; the **/public/assignments** cap with identical silent-empty behaviour is ****. Both endpoints share the behaviour; both are in scope.
- INT-129: backend permits **PENDING as well as ACTIVE/COMPLETED** (Contact Reveal Service), slightly broader than the row's claim — frontend keeps PENDING as "Awaiting confirmation" (no reveal pre-confirmation; unchanged).

---

## INT-125 (S3, backend) — Assign expires PENDING invitations with NO notification to the invited tutors

### Problem
The atomic assign (Admin Match Service create Match) bulk-expires every PENDING invitation on the job, but the in-tx notification fan-out covers only winner / parent / superseded **applicants**. An invited tutor's invitation silently flips to EXPIRED — they never learn the position was filled.

### Current state (verified in code)
- Expire happens inside the assign tx with **no recipient capture**: Admin Match Service implementation reference expire pending invitations for the job and record how many changed (set-based update records — returns only a count).
- The fan-out notify Assign In Tx (374-399) emits exactly three groups: winner "You've been selected" (N-14), parent "Assignment confirmed" (N-15), superseded applicants "Position filled / A job you applied to is no longer available." (N-16). **No invitee group.**
- Applicant recipients ARE captured before their bulk update records (226-227 find Recipients To Supersede) — the INT-016 capture-before-bulk-write pattern is already half-applied; invitations were missed.
- The needed repository method **already exists**: Job Invitation Repository implementation reference the find pending recipients operation (used by Tutee Post Service close Live Job).
- The neutral invite copy **already exists** in the family: Tutee Post Service implementation reference and Admin Moderation Service implementation reference both use **"A job you were invited to is no longer available."** with link /tutor/invitations.

### Design
In create Match, capture invitee user-ids **BEFORE** the bulk expire (INT-016: the update records returns no rows), beside the existing applicant capture (after 227, before step 9/10):


Implementation example omitted. This public edition retains the written requirements in this section.


Extend the notify assign in tx operation with a fourth loop, **in-tx per INT-132/133** (a notification is a plain database row — atomic with the assign; rolls back together on a @Version loser), **failure-isolated per recipient** via the existing safe Notify (402-415, per-recipient try/catch + notification emit failures total counter):


Implementation example omitted. This public edition retains the written requirements in this section.


- **Copy is the exact existing string** (Tutee Post Service / Admin Moderation Service) — neutral, never "another tutor was chosen", never any moderation/selection detail. Template id N-16b (next in the assign family).
- **Dedup/exclusion rules:** (1) exclude the **winner** (they may hold a still-PENDING invitation while having applied independently — they already get N-14, and "position filled" to the winner is wrong); (2) exclude anyone already in superseded Recipients (applied AND invited → one notification, the applicant copy wins, applied-to link is the stronger CTA). Implement as a Set difference before the loop.

### Test plan
- **[live-lab]** Extend regression/flows/admin/Free Market Assign IT (the assign live-lab IT, free Market select And Assign reveal Contact then Cancel no Listing): add a second tutor who is **invited** (PENDING) but not the winner before the assign; after assign, assert (a) the invitation is EXPIRED, (b) the invited tutor's read operation for notifications contains exactly one row with body A job you were invited to is no longer available. and link /tutor/invitations — a **live delivery assert** (INT-132: mock-verify passes on the dead-context path; only the row proves delivery).
- **[live-lab]** Dedup case: a tutor who both applied AND was invited receives exactly **one** notification (the applicant N-16 copy), and the winner receives **no** "Position filled".
- **[Testcontainers/per-pull request]** Admin Match Service Test unit: capture-before-expire ordering (verify find Pending Recipients is called before expire Pending For Job — In Order), and one recipient's emit throwing does not block the others (failure isolation).

### Acceptance
1. After assign, every non-winner PENDING invitee holds one in-app notification with the exact neutral copy; zero notifications mention selection/choice/moderation detail.
2. Capture precedes the bulk expire; emits are in-tx via safe Notify; on a @Version-loser rollback the notification rows roll back with the tx (no phantom invitee notifications).
3. No duplicate notification for applied-and-invited tutors; no "Position filled" to the winner.

---

## INT-128 (S3, frontend) — Network-level failure renders as "Invalid email or password"

### Problem
A request that never reached the server (cross-origin request permissions-blocked, offline, server down — axios error with no response) is mapped by Login to the wrong-credentials banner, sending users to reset passwords that are correct. This masked INT-127 and cost the proof agent a diagnosis round.

### Current state (verified in code)
- api implementation reference — the response interceptor builds Api Error with status: error response status or otherwise 0 and code: ... or otherwise UNKNOWN ERROR. So a network failure is **deterministically an unknown-error result with no server response status** at every consumer.
- Login implementation reference — the catch checks only ACCOUNT BLOCKED CODE, then falls through to show “Invalid email or password. Please try again.” as a form-level error (99) for **everything**, including status 0.
- The shared error-mapping level **exists**: api Error implementation reference (the centralised code→copy module; extract Error Code handles both the unwrapped Api Error and raw-axios shapes). format Api Error (api implementation reference) is the generic formatter with **3 non-test consumer files / 4 call sites** (review-corrected count: query Client implementation reference — the global query/mutation toasts — plus Commission implementation reference and Register implementation reference and use Init Auth implementation reference reference it in comments only) and currently returns axios's unhelpful Network Error message for status-0 failures. Review-verified: none of the consumers wants different copy — the new constant strictly improves all of them, and Commission/Register's or fallback branches are unreachable (format Api Error always returns a string).

### Design — fix at the shared level so ALL forms benefit
**api Error implementation reference** — add (mirroring the module's existing helper style):


Implementation example omitted. This public edition retains the written requirements in this section.


**api implementation reference format Api Error ()** — check the is network error operation first and return NETWORK ERROR MESSAGE — every format Api Error consumer (3 files / 4 call sites incl. the global query Client toasts) gets the fix for free.

**Login implementation reference ()** — in the catch, before the credentials fallback:


Implementation example omitted. This public edition retains the written requirements in this section.


The wrong-credentials copy is now reachable only when the server actually responded.

### Test plan (all [per-pull request] Vitest)
- test/lib/ — is Network Error: true for an unknown-error result with no server response status and a response-less axios error; false for 401/422/500 Api Errors. format Api Error returns the exact NETWORK ERROR MESSAGE for a status-0 error.
- Login test implementation reference — login mutation rejecting with an unknown-error result with no server response status renders **"Cannot reach the server — check your connection and try again."** and does NOT render "Invalid email or password"; a 401 still renders the credentials copy (pin both directions).
- One sibling-form spot check (e g. Register via format Api Error) asserting the network copy on a status-0 rejection.

### Acceptance
1. Status-0/no-response failures render the exact network copy on Login and every format Api Error consumer; credentials copy only on a real server response.
2. The mapping lives in api Error implementation reference/format Api Error (shared), not per-page; no magic strings (the copy is one exported constant).

---

## INT-129 (S3, frontend — product call) — COMPLETED match drops the contact card

### Problem
The moment admin marks commission paid (match → COMPLETED), both parties' match cards replace the contact-reveal section with the review CTA — contact access is lost exactly when lessons are starting.

### Current state (verified in code)
- **backend permits the reveal on COMPLETED** (and PENDING): Contact Reveal Service implementation reference — revealable iff status ∈ PENDING, ACTIVE, COMPLETED; only CANCELLED is 404-indistinguishable.
- **frontend gates on ACTIVE only:** Matches implementation reference show contact sharing for an active match in this historical design (mutual-visibility microcopy at ); Matches implementation reference show the match’s contact-sharing button only when active in this historical design. COMPLETED renders only the review CTA (tutee /; tutor /).

### Design — DECIDED: keep the reveal available on ACTIVE **and** COMPLETED, both personas
**Rationale (the product call, made here):** COMPLETED means "commission paid", not "relationship over" — lessons are ongoing/starting. The backend already authorises COMPLETED reveals (audited, participant-gated), so the frontend gate is the only thing severing contact; sunsetting contact at payment would punish the paying parties at their moment of highest need. CANCELLED stays hidden (backend 404s it — correct).

**Exact change (both files):** widen the reveal-section condition from equals MATCH STATUS ACTIVE to ACTIVE or COMPLETED via a small shared predicate (no magic strings):


Implementation example omitted. This public edition retains the written requirements in this section.


- Matches implementation reference and Matches implementation reference use the is contact revealable operation. The **mutual-visibility microcopy is unchanged** (tutee ). The review CTA continues to render alongside on COMPLETED (both sections, not either/or).
- **Builder fence (review amendment): ONLY the reveal widens to COMPLETED.** Every other ACTIVE-gated section stays narrow exactly as today — Matches implementation reference (fee section), 42 (admin-contact box), Matches implementation reference (commission badge/copy/admin box et al.). Do not generalise is Contact Revealable onto those.
- **Adjudication ratified at review:** indefinite ACTIVE+COMPLETED reveal is correct — contact was already mutually exchanged during ACTIVE (a user interface sunset can't un-share data); every reveal stays participant-gated + audited (Contact Reveal Service implementation reference); account deletion already severs access via @database query language Restriction → 404. No time-bound window.
- PENDING intentionally stays "Awaiting confirmation" (no reveal button pre-confirmation) even though the backend permits it — the admin confirms first; record this as the deliberate frontend-narrower-than-backend choice.
- Scope check (INT-126 blast-radius rule): inspect every use of the contact-sharing button — verified mounts are exactly these two pages; no other gate to widen.

### Test plan (all [per-pull request] Vitest)
- Matches test implementation reference + Matches test implementation reference (**both personas** — the 2026-06-05 role-walk lesson): COMPLETED match renders BOTH the Contact Reveal Button AND the review CTA; ACTIVE renders the button (regression); PENDING and CANCELLED render no button.
- Unit test for is Contact Revealable truth table.

### Acceptance
1. A COMPLETED match shows the contact-reveal section for tutee AND tutor, with the existing mutual-visibility microcopy; review CTA unaffected.
2. PENDING/CANCELLED behaviour unchanged; the rule lives in one named predicate.

---

## INT-134 (S3, test-infra) — Regression-lab fixture entropy. No product code.

### Problem
Fixture actors mint phones from a 10^7 random pool; after a day of suite runs against the accumulated dev database, birthday-paradox collisions throw PHONE ALREADY REGISTERED/PUBLIC HANDLE TAKEN 409s during setup, failing downstream asserts confusingly.

### Current state (verified in code)
- Canonical copy: Abstract Admin IT implementation reference — the reg phone operation = the d operation.
- **FIVE sibling copies (tracker said 4 — drift):** Account Lifecycle IT implementation reference, Password Reset Journey IT implementation reference, Auth Journey IT implementation reference, Favourite Soft Remove IT implementation reference (inline, no helper), Registration Enumeration IT implementation reference.
- Handles: Abstract Admin IT implementation reference mints a 6-char hex-letters-only handle from a unique identifier — a small pool with the same collision class. Emails use full the random unique identifier operation (collision-proof; keep).

### Design (per the tracker row's durable fix)
1. **One shared generator util** — Unique Fixture Data implementation reference:
   - A a sequence used to generate distinct test identifiers — a **nano Time-seeded monotonic sequence**, so values never repeat within a JVM and restart at a different offset per run (collision now requires two runs whose seeds AND offsets align, vs. a flat random pool).
   - the unique phone operation → the d operation → prefix + base-36 of the get and increment operation (≤64 chars); the unique email operation → keep the unique identifier form, centralised.
2. **Retry-once-on-409 in the register helpers:** in Abstract Admin IT's register paths (and the util's shared register wrapper if extracted), if the register POST returns 409 with PHONE ALREADY REGISTERED/PUBLIC HANDLE TAKEN, mint fresh values and retry **exactly once** (a second 409 fails the test loudly — a real defect, not entropy).
3. **Migrate all FIVE sibling copies** (the four tracker-named + Registration Enumeration IT) to the util; delete the private the reg phone operation copies. Guard against re-forking: a grep in the build's conventions or a comment is enough — no need for a scan test on test code.
4. **Runbook line** — add to intuition-spec/specs/ops/runbook md (lab maintenance): *"Periodic regression-lab reseed: after heavy suite use (or on any fixture-setup 409 burst), the documented operational check to reset the accumulated actor pool, then reseed before the documented operational check (live-lab ITs hit localhost)."*

### Test plan
- **[per-pull request]** Plain JUnit on the util: 10k the unique phone operation calls → zero duplicates, all match a Singapore mobile number with the country prefix and eight digits beginning with 8; handles ≤64 chars, unique.
- **[live-lab]** Existing suite green on a NON-reseeded lab (the actual symptom scenario): run the five migrated ITs twice back-to-back without reseed — zero fixture-setup 409 failures.

### Acceptance
1. One generator home; the next int operation → zero hits.
2. Register helpers retry once on the two entropy 409 codes, fail loud on the second.
3. Runbook carries the reseed line. No product code touched.

---

## INT-135 (S3, backend) — Assign-path analytics publish in-tx through the REQUIRES NEW store on a @Version-guarded path → phantom events on a 409 loser

### Problem
A concurrent-assign loser thread (optimistic-lock 409) rolls back its business writes — but the analytics events it published survive, because the store commits in its own transaction. Phantom job assigned/application superseded rows corrupt funnel metrics.

### Current state (verified in code)
- The two publishes are **in-tx**: Admin Match Service implementation reference the publish operation and 298 the publish operation — inside create Match's a database transaction body, **after** the @Version bump (the set status operation + save at 236-239, the optimistic-lock point) but before commit.
- The store is REQUIRES NEW: Analytics Event Store implementation reference the transactional operation on append — a loser's rollback does NOT claw these rows back.
- The sanctioned pattern exists **in the same file**: activate (631-643) resolves values to finals then defers the publish operation to run After Commit (418-430) — "ONLY the analytics publish (a REQUIRES NEW store that would survive a loser's rollback) stays deferred to after Commit so it fires EXACTLY ONCE on the committed outcome" ( javadoc). Admin Commission Service implementation reference does the same for pay. §4.1 confirms this is the one legitimate after Commit case (external/REQUIRES NEW, loss-tolerant).

### Design
Defer **exactly these two publishes** to run After Commit, copying the activate pattern. Resolve every captured value to a **final local first** (INT-016 discipline — never dereference managed entities inside the lambda):


Implementation example omitted. This public edition retains the written requirements in this section.


Scope guard: ONLY these two move. The in-tx notification emits (notify Assign In Tx) stay in-tx (INT-133 — they roll back with the loser, which is correct); the micrometer counters stay where they are (process-local gauges, not durable rows). Note run After Commit (418-430) falls back to inline when no synchronization is active — existing unit tests keep working.

### Test plan
- **[per-pull request]** Admin Match Service Test unit pinning **loser-thread-no-event**: run create Match inside an active synchronization where the tx is rolled back after the method body (or simulate: synchronization registered, after Commit never invoked) → the verify operation and same for EVENT APPLICATION SUPERSEDED. Positive control: invoking the registered synchronization's after Commit publishes both exactly once with the captured props (this test FAILS on the current in-tx code — the required fails-without-the-fix property, SHARED-LESSONS 2026-06-05).
- **[live-lab]** Free Market Assign IT happy path: after a committed assign, both events exist exactly once (no regression from the deferral).

### Acceptance
1. A rolled-back assign persists **zero** job assigned/application superseded rows; a committed assign persists exactly one of each with unchanged props (TR8 had application included).
2. Notification emits remain in-tx; only the two analytics publishes moved.

---

## INT-136 (S4 batch, frontend+backend) — Wave E micro batch, in the Boss-mandated order

### (d — FIRST, Boss order) DELETE dummy implementation reference
**Verified:** the file is 299 lines, exports DUMMY TUTORS/etc. with name-like handles ("Ms L.") contradicting the personal information pattern; a search for imports of the unused sample-data file returns **zero importers** (only the file itself). **Change:** delete the file (and the the private implementation folder dir if then empty). the documented operational check (tsc) + the documented operational check prove nothing referenced it. *This deletion opens the batch* so the later guard hardening (b) never has to whitelist its name-like fixtures.

### (a) Region humaniser + exp Label null guard on Applicant Card (+ the sweep-found sibling)
**Historical finding:** Applicant and invitation cards displayed raw area codes such as CENTRAL instead of readable region names. An applicant with no experience value could be shown as “null yrs exp”. Use the shared region labels and omit the experience text when no value exists.
**Change:**
- format implementation reference: add the region label operation — look up REGION OPTIONS, **fall back to the raw value** for unknown codes (same safety contract as subject Label, format implementation reference).
- Show the shared readable region label on both the applicant card and the invitation screen.
- Show experience text only when an experience value exists; otherwise omit both the text and its separator.

### (b) Harden the INT-099 guard — honestly scoped
**Verified:** no Raw Enum Labels test implementation reference is a **line-based regex scan** (RAW PATTERNS : the subject code/the level code interpolations + the join operation on the same line) plus 9 positive site pins (). Evasion paths confirmed: a destructured field (a subject code extracted into a local variable and displayed directly) or an intermediate variable (a subject code assigned to another variable and displayed directly) never matches the property-access pattern; regions (coarse Area) are entirely uncovered.
**Honest mechanism call:** full static coverage of destructured/intermediate renders requires real AST data-flow analysis (a TS program walk tracing enum-typed values to JSX) — **impractical for a regression guard test** and brittle to maintain. We do NOT claim it. The durable protection remains the **positive site pins** (each named surface must call its humaniser — pins catch a regression even when the negative scan can't see the render shape) plus the humanisers' raw-value fallback. **Changes:**
1. Extend RAW PATTERNS with the region shape: pattern a check for broad-area codes displayed without a readable label, humaniser /region Label\ / (same line-level contract as the existing four).
2. Add a **cheap destructure tripwire** (best-effort, named as such): flag a check for subject, level or area codes extracted into local variables before display in any  implementation format implementation reference — destructuring an enum-code field on a render surface is the evasion's necessary first step; legitimate uses (passing to a humaniser) are rare enough to allowlist by file if ever needed. This narrows, not closes, the gap — say so in the test's header comment.
3. Add positive site pins: Applicant Card implementation reference → SUBJECT, /region Label\ /, Invitations implementation reference → /region Label\ /.
4. Red-team the hardened guard against the known-good fixed code BEFORE trusting any failure (Wave E lesson 1: a new guard that fails indicts the guard first).

### (c) Withdraw-choice confirm symmetry
**Verified:** choosing a tutor confirms via Choose Tutor Modal (a Modal-based confirm, Choose Tutor Modal implementation reference, mounted at Applicants implementation reference), but **withdraw fires immediately**: Applicants implementation reference withdraw the current tutor choice when the user selects that action (the "Change choice" button on the chosen card) and 205 (the banner's "Change my choice") both mutate with no confirm. INT-116 eliminated window confirm repo-wide (guard: no Window Confirm test implementation reference) — the fix must be a Modal, matching choose.
**Change:** add a confirm Withdraw state to Applicants implementation reference; both trigger sites open a shared confirm Modal (pattern-match Choose Tutor Modal's shape: Modal + Modal Header on Close + body + footer Keep-choice/Withdraw buttons; copy e g. title "Change your choice?", body "We'll withdraw your current choice — you can then choose another tutor, or no one.", destructive-primary "Withdraw choice", ghost "Keep choice"); confirm fires the mutate operation and closes. Disabled-while-pending mirrors 206.
**Test:** Applicants test implementation reference — clicking "Change choice" does NOT call the application programming interface until the modal's confirm; **every close affordance** (footer Keep, header ×, native close) leaves the choice intact — assert via the open attribute, not text removal (INT-046 modal lesson).

### (e) Public-teaser cap: document + clamp size, keep page-cap empty
**Verified:** Public Browse Controller implementation reference — MAX ANONYMOUS PAGE 4 (), MAX ANONYMOUS SIZE 20 (). Anonymous /public/tutors out-of-bounds page/size → **200 with an empty page** (); /public/assignments identical (, plus a per-IP rate limit ). Already cost our own Wave E IT author a failed run (?size is 50).
**RECOMMENDED (decide here): clamp size to 20; keep over-cap page returning the empty page; document both.** Rationale: a too-large size is an innocent client mistake — clamping serves the first 20, which is exactly what page 0 of a compliant request returns, so **the INT-012 anti-enumeration ceiling survives untouched** (still ≤ 5 pages × 20 = 100 rows anonymously reachable; a clamp never serves data an honest client couldn't get). A too-deep page, by contrast, is the enumeration vector itself — clamping it would silently re-serve page-4 content under a lying page number (dishonest pagination); the empty page stays. size 1 / page 0 also stay empty (malformed, not innocent).
**Changes:**
1. Both endpoints: limit a guest’s requested page size to the allowed maximum applied on the anonymous path (tutors) / always (assignments, which is anonymous-only by design); the page-bound branch keeps the empty-page return. Keep the log line, now logging the clamp.
2. **Document the contract** in modules/landing-showcase/spec md (the public browse/teaser contract home — verified it already describes the teaser): anonymous cap = page ≤ 4, size clamped to 20, over-deep page → empty page, rate limit 60/min — with the INT-012 anti-enumeration rationale stated so a future reader doesn't "fix" the empty page into full pagination.
**Test:** **[live-lab]** extend flows/marketplace/Browse Assignments IT (+ Browse Tutors IT anonymous case): ?size is 50 → 200 with **20** items (clamped, not empty); ?page is 9 → 200 empty page; total anonymously-reachable rows still ≤ 100.

### INT-136 acceptance
1. dummy implementation reference gone; build + tests green without it.
2. Zero raw region codes on applicant cards/invitations; "null yrs exp" impossible; guard extended (regions + tripwire + 2 new pins) and red-teamed against the fixed code.
3. Withdraw-choice confirms via Modal symmetrically with choose; no Window Confirm guard still green.
4. Anonymous size>20 returns 20 results; deep pages stay empty; contract documented with the anti-enumeration rationale; rate limit untouched.

---

## Build order, lanes, owners, Do D

**Build order (Boss-mandated):**
1. **INT-136d** — delete dummy implementation reference (the batch OPENS with this).
2. frontend lane (sequential within intuition-frontend): INT-136a → 136b → 136c → INT-128 → INT-129.
3. backend lane (sequential within intuition-backend): INT-125 → INT-135 → INT-136e → INT-134 (test-infra last — its retry helper must not paper over the two backend changes' IT runs while they stabilise).
4. Spec/doc edits (runbook line, landing-showcase contract) — any time, distinct files.

**Parallelism:** the frontend lane and backend lane touch disjoint subtrees — run them **concurrently** (CLAUDE md §A2); never two gate runs on one repo (2026-06-05 lesson). Doc edits parallel-safe.

**Owners:** software-engineer — INT-125, INT-135, INT-136e (controller); qa-engineer — INT-134, all lane-tagged test plans, red-teaming the hardened guard; frontend-expert — INT-128, INT-129, INT-136a/b/c/d; uiux-designer — sign-off on the INT-128 copy, the INT-129 COMPLETED-card layout (reveal + review CTA coexisting), the 136c modal copy; data-engineer (consult) — confirm INT-135's deferral keeps the funnel definitions intact (job assigned exactly-once-on-commit).

**Definition of done (per the standing gates + Boss additions):**
- backend the documented operational check ≥97.5/97.5 (Docker up; rebuild+redeploy+reseed before live-lab ITs reflect the fixes); frontend the documented operational check + the documented operational check + the documented operational check ≥97.5 line / ≥88 branch.
- **No migration shipped** (re-verify at build: the documented operational check shows no changelog edits).
- **Boss proof-hygiene (recorded as a Do D line):** the user interface proof pack contains **one unique evidence capture per user interface check** — no two checks may share a byte-identical screenshot/capture (Wave E's 04/05 were byte-identical; verify with shasum over the proof dir before submitting).
- Lessons captured in the owning agents' SKILL md (+ SHARED-LESSONS if cross-cutting); /team-review → Boss on the completed batch; push only after verify+local-deploy green.
