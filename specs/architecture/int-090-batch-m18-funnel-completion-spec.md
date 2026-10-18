# INT-090…097 (Wave A) — M18 free-market funnel completion: close the revenue loop

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Boss audit 2026-06-11 — SEND-BACK conditions folded in:** Wave A now covers **BOTH** admin assign
> surfaces (Matching implementation reference AND Interview Queue implementation reference — see Design step 3b, filed as INT-126) via one
> shared assign hook with a stable Idempotency-Key; acceptance criterion 5 added (grep admin matches
> blast-radius check). Build may start.

**Status:** spec for build — **team-reviewed 2026-06-11 — conditions folded in** → Boss · **Date:** 2026-06-10
**Owners:** software-engineer (lead) · frontend-expert · bff-specialist · (review: qa-engineer, security-expert)
**Tracker:** INT-090 (S1, frontend) · INT-091 (S1, backend) · INT-092 (S2, backend) · INT-097 (S2, frontend)
**Evidence record:** [issues/PROJECT-REVIEW-2026-06-10.md](../issues/PROJECT-REVIEW-2026-06-10.md) (Part 1, S1/S2)

## Why this is one wave

These four defects are the **revenue-loop closers** for M18's *default* path (apply-without-listing →
parent chooses → admin assigns → activate → commission → review → contact). The INT-034 re-key
(matches keyed on tutor user id, migration 0033) made free-market matches *creatable* by the application programming interface,
but three consumers were never migrated off the legacy tutor Post join and the frontend assign user interface was never
wired to the listing-less contract. So the crown-jewel e2e (which drives the application programming interface) passes while the
**shipped product dead-ends**: the admin can't click "assign", and if assigned via application programming interface, pay/review 500.
This is the same class as **INT-075** ("a required cross-service contract needs both ends") and the
INT-082 "drift on the non-entry-point path" lesson — *the feature's own files are correct; the
unchanged neighbours are not.*

**No migrations in this wave.** Every change is application code only — the schema (0033 re-key,
Match tutor User, the resolve tutor user id operation) already exists. State this explicitly so no number is
allocated and migrate-then-boot is untouched.

---

## INT-090 — Admin user interface cannot assign a free-market (listing-less) chosen tutor (S1, frontend)

### Problem
The parent→choose→assign funnel dead-ends at the admin Matching screen. When the chosen tutor applied
without a listing (tutor Post Id equals no value — the M18 *default*), the Assign button is disabled and the
revenue loop cannot be closed from the user interface.

### Current state (verified in code)
- Selection Request Queue implementation reference — the INT-034 stopgap
  disable assignment when no tutor listing is attached plus an explanatory line (63-67) "assignment isn't available yet".
- Matching implementation reference — Close Job Assign Modal on Confirm guards
  if assign Target tutor Post Id and posts only the tutor-listing and tuition-request identifiers to
  application programming interface ROUTES admin.matches. A null tutor Post Id confirm is a silent no-op.
- A search for the chosen tutor-account identifier in the frontend → **zero hits** (verified). The frontend never sends the
  free-market target by any name.
- **backend is fully ready** (verified):
  - Create Match Request implementation reference accepts exactly-one of tutor Post Id | winning Tutor Id
    (@Assert yes cross-field, 400 on both-null/both-set); Admin Match Controller implementation reference
    passes winning Tutor Id + source Selection Request Id through.
  - **Better still, a purpose-built endpoint already exists:** POST
    admin selection-requests id Admin interface operation Selection Request Controller implementation reference-77
    → the assign from selection request operation,
    Admin Match Service implementation reference). It reads the chosen tutor User Id **server-side** from the
    selection request (so the frontend never has to handle the opaque tutor id), is **Idempotency-Key
    guarded** (Idempotency Contract SCOPE ASSIGN, INT-087 F5 fingerprint), and returns 201 Match Response
    with the same 404/422/409 semantics as submit operation for admin matches.
  - Admin Selection Request Response implementation reference carries selection Request Id + tutor User Id
    (opaque, personal information-free) — everything the frontend needs.
- **The unwired half:** api Routes implementation reference has selection Requests (list) but
  **no assign route constant** (verified). The backend constant Api Routes ADMIN SELECTION REQUESTS ASSIGN
  (Api Routes implementation reference) has no frontend counterpart — the textbook INT-075 signature.

### Design / fix (frontend only — use the selection-request-assign endpoint, NOT raw winning Tutor Id)
1. **Add the route constant** (no magic strings): in api Routes implementation reference, under admin, add
   the selection-assignment operation for the requested record.
2. **Selection Request Queue implementation reference** — delete the stopgap: remove disable assignment when no tutor listing is attached (line 54),
   the conditional title (55-59), and the show this explanation when no tutor listing is attached explainer block (63-67).
   The button always calls the on assign operation. Keep the existing comment header but rewrite it to state
   that assign now keys on the server-side tutor User Id via the selection-request-assign endpoint
   (kill the "cannot be assigned yet" lie).
3. **Matching implementation reference** — change assign Mutation (99-130) to call the selection-request-assign
   endpoint, keyed on assign Target selection Request Id (NOT tutor Post Id):


Implementation example omitted. This public edition retains the written requirements in this section.


   Update Close Job Assign Modal on Confirm (280-287): drop the if assign Target tutor Post Id guard;
   call the mutate operation whenever assign Target is non-null.
   Keep the existing JOB ALREADY ASSIGNED 409 on Error handling (this endpoint returns the same code).
   - **Keep the stable idempotency key per operation** (one the new idempotency key operation minted per assign,
     reused on a cross-site request forgery protection self-heal/react-query retry — the §INT-075/INT-087 rule; the backend rejects a reused
     key for a *different* selection request with 422 IDEMPOTENCY PARAMS DIFFER, which is correct).
3b. **Interview Queue implementation reference — the SECOND mount of this assign surface (Boss audit 2026-06-11, INT-126).**
   Interview Queue implementation reference posts the tutor-listing and tuition-request identifiers to application programming interface ROUTES admin.matches with
   **no explicit Idempotency-Key** (it rides the interceptor autofill, so a react-query retry mints a
   fresh key and the double-submit collapse is lost — violating the P0 stable-key rule documented at
   Matching implementation reference), and 250-266 renders the same shared Selection Request Queue +
   Close Job Assign Modal with its own if assign Target tutor Post Id silent-no-op guard (256-262).
   Deleting the stopgap inside the shared component WITHOUT fixing this surface would turn the
   Interview Queue's Assign into an **enabled button that silently does nothing** for free-market rows
   — the exact dead-click class this wave exists to kill. **REQUIRED:** extract ONE shared assign
   mutation/hook (e g. the use selection request assign operation — mints one stable the new idempotency key operation per
   operation, reused on retry) consumed by BOTH Matching implementation reference and Interview Queue implementation reference; remove the
   tutor Post Id guard at Interview Queue implementation reference; both surfaces call
   submit operation for admin selection-requests id assign; both invalidate the same query keys.
4. **Modal copy must reflect a listing-less assign.** Close Job Assign Modal implementation reference currently says
   "Mark handle as assigned (winner)." — keep, but ensure the surrounding bullets do not assume a
   listing. The handle (tutor Public Handle) is personal information-free and present for both paths, so the copy already
   works; **the load-bearing fix is the false applicant count below.**
5. **Fix the false "other applicants" number (folds in INT-124f).** Matching implementation reference passes
   the max operation — that is *pending selections across ALL jobs*, not the
   applicants on THIS job — into Close Job Assign Modal's "Notify the other N applicants" (about this
   job). Either (a) source the real count for *this* tutee Post Id (preferred — if the queue row can
   carry it), or (b) drop the specific number and use generic copy ("Notify the other applicants their
   applications are no longer available"). **Do not ship a confidently-wrong number on a destructive
   confirm.** If (a) needs a backend field it is out of this wave's scope → use (b) now, file the count as a
   follow-up.

### Cross-check (INT-090 = the unwired frontend half of INT-034)
This is the **same defect class as INT-075**: the backend shipped a required contract
(ADMIN SELECTION REQUESTS ASSIGN / exactly-one tutor target) and the frontend was never wired to call it, so
a careful backend-side review and the application programming interface-driving e2e both pass while the **clicked product is broken**.
Per the INT-075 lesson: a contract change across the backend↔frontend boundary needs *both ends* + a user interface-level test
that actually clicks Assign. Confirm the wiring by grepping the frontend for the new route constant AND adding
the Vitest below — existence of the endpoint is not use.

### Test plan (lanes, team-review 2026-06-11: **[live-lab]** = Abstract Api IT → localhost, excluded from per-pull request continuous integration via ci-no-lab, needs rebuild+redeploy+reseed; **[Testcontainers/per-pull request]** = runs in per-pull request continuous integration, incl. Vitest)
- **[Testcontainers/per-pull request]** Matching test implementation reference: a selection request with tutor Post Id: no value →
  render → the Assign button is **enabled** (regression for the stopgap) → click → confirm in
  Close Job Assign Modal → assert api post was called with
  admin selection-requests <id> assign interface operation and an Idempotency-Key header. A second test asserts
  the **stable key** is reused across a retry of the same assign (mirror the existing mark Paid/assign
  idempotency assertions in this suite).
- **[Testcontainers/per-pull request]** Selection Request Queue test implementation reference: assert the button is
  enabled for a tutor Post Id: no value row and that on Assign fires (regression for the removed disabled).
- **[Testcontainers/per-pull request]** Interview Queue test implementation reference (Boss condition, INT-126): a
  selection request with tutor Post Id: no value assigns successfully **from the Interview Queue page** —
  click Assign → confirm in Close Job Assign Modal → assert the selection-request-assign endpoint is
  called with a stable Idempotency-Key that is **reused on a retry** of the same assign.
- **[Testcontainers/per-pull request]** JOB ALREADY ASSIGNED 409 path: assert the toast + cache invalidation still
  fire (unchanged behaviour).

### Acceptance criteria
1. With a chosen free-market tutor (tutor Post Id equals no value), the admin can click **Assign tutor & close
   job** and the job is assigned (match created), the queue + match list refresh, and a success toast shows.
2. The frontend calls submit operation for admin selection-requests id assign (via an application programming interface ROUTES constant) with a
   stable Idempotency-Key reused on retry; no raw path literal; no winning Tutor Id constructed on the frontend.
3. The stopgap disabled + "isn't available yet" copy are gone; the confirm modal shows no false
   applicant count.
4. the documented operational check + the documented operational check (tsc) + the documented operational check green (≥97.5% line / ≥88% branch).
5. **Blast-radius check (Boss condition):** BOTH admin assign surfaces are wired —
   a search of the administration matching requests shows **no remaining assign POST**
   (or every remaining hit carries an inline justification); the shared assign hook is the only assign
   path; the Interview Queue implementation reference guard is gone.

---

## INT-091 — Commission pay 500s (missing-value failure) on every free-market match (S1, backend)

### Problem
submit operation for admin commissions id pay throws an missing-value failure for any free-market match (tutor Post equals no value). The missing-value failure
is thrown *after* the state writes, so the tx rolls back: **commission stays PENDING, match never
COMPLETED — the admin cannot collect commission on any default-path match.** The money path is broken.

### Current state (verified in code)
- Admin Commission Service implementation reference —
  read the tutor's identifier through the match's tutor listing. For a free-market match
  Match tutor Post is **null** (INT-034 / 0033) → missing-value failure. The state writes at 64-72 (commission→PAID,
  match→COMPLETED) precede it, so the whole a database transaction method rolls back on the throw.
- Admin Commission Service does **not** inject User Repository (verified — imports only
  Commission Repository, Match Repository). tutor User Id here is only used as the
  the emit operation recipient id, computed into a final local for the after Commit lambda.
- **The canonical resolve pattern** (verified) is the resolve tutor user id operation (Match implementation reference
  — prefers tutor User, falls back to the legacy tutor Post owner, never NPEs): used by
  Contact Reveal Service implementation reference, Admin Match Service implementation reference,
  Match Dashboard Service implementation reference.

### Design / fix
Replace line 80 with the resolve helper — it returns the user Id directly, and this site only needs the
id (recipient of the COMMISSION PAID notification), so **no User Repository injection is needed**:


Implementation example omitted. This public edition retains the written requirements in this section.


Keep it computed into the final local *before* the after Commit lambda (it already is — the INT-087
Principle-D / INT-016 detach discipline is preserved; the resolve tutor user id operation touches tutor User which is
loaded in-tx). the resolve tutor user id operation returns null only for a malformed row (neither tutor User nor
tutor Post) — not reachable for a valid match, and a null recipient would simply skip the notification,
not 500.

### Test plan
- **[live-lab]** **IT (the load-bearing one):** drive a **null-tutor Post (free-market) match** through
  create→activate→**pay** and assert 200, commission PAID, match COMPLETED. Add to the existing
  Admin Commission IT implementation reference (live-stack Abstract Api IT) — it currently only
  exercises the legacy listing-backed match, which is exactly why this missing-value failure shipped. The free-market match
  must be produced via the real assign path (selection-request-assign) so the fixture has tutor User set,
  tutor Post null. (Remember: ITs hit the LIVE stack at localhost — rebuild+redeploy+reseed before
  the documented operational check reflects the fix; and the documented operational check so the strengthened IT bytecode is current — the
  2026-06-10 stale-IT-class lesson.)
- **[Testcontainers/per-pull request]** **Unit:** Admin Commission Service Test implementation reference — a mock match with
  tutor Post equals no value and tutor User set → pay Commission returns without missing-value failure and the notification targets
  the resolve tutor user id operation. Add a test that FAILS against the old line (a test that passes whether or not
  the fix is wired proves nothing — INT-075 lesson).

### Acceptance criteria
1. Paying commission on a free-market match returns 200; commission → PAID, match → COMPLETED; the
   COMMISSION PAID notification fires to the resolved tutor.
2. No the get tutor post operation deref remains in Admin Commission Service.
3. the documented operational check green (≥97.5% line + branch) with the new free-market pay IT.

---

## INT-092 — Review submission 500s (missing-value failure) on free-market matches (S2, backend)

### Problem
A completed free-market match → either party submits a review → missing-value failure → 500. Review & Rating is dead for
listing-less matches (the default M18 shape).

### Current state (verified in code)
- Review Service implementation reference — read the tutor account through the match's tutor listing → missing-value failure when
  tutor Post equals no value. Unlike INT-091, this site needs the **full User entity**, not just the id: the
  tutor becomes reviewer/reviewee (the builder operation, 111-119) and the
  the emit operation recipient (132-137).
- Review Service does **not** inject User Repository (verified).

### Design / fix (same shape as INT-091, but resolve to the entity)
Inject User Repository into Review Service and resolve the tutor by id:


Implementation example omitted. This public edition retains the written requirements in this section.


Mirror the Contact Reveal Service implementation reference pattern (resolve id → user Repository find By Id →
404 on the absent edge). tutee (the get tutee post operation, 87) is unchanged — the tutee
side never went through the re-key. Use the existing ERR MATCH NOT FOUND code for the (unreachable)
absent-user edge so the frontend handler is unchanged.

### Test plan
- **[live-lab]** **IT:** extend Review Submit Journey IT implementation reference (or Dashboard Reviews IT) with a
  free-market completed match (null tutor Post) → tutee submits a review → 200, review persisted,
  REVIEW RECEIVED notification to the resolved tutor; then the tutor submits → 200. Live-stack +
  test-compile caveats as INT-091. **The IT must reuse the Free Market Assign IT fixture path (team-review
  2026-06-11):** the seed has NO free-market match — Dev Data Seeder implementation reference both set tutor Post —
  so the fixture must create one via the real assign application programming interface; Free Market Assign IT implementation reference is the pattern.
- **[Testcontainers/per-pull request]** **Unit:** a Review Service test with tutor Post equals no value, tutor User set →
  submit Review resolves the tutor via user Repository find By Id (Mockito verify) and does not missing-value failure.

### Acceptance criteria
1. Both parties can submit a review on a completed free-market match (200); review persisted; the
   reviewee notification fires.
2. No the get tutor post operation deref remains in Review Service submit Review.
3. the documented operational check green (≥97.5% line + branch).

> **Audited-accepted (team-review 2026-06-11, no change):** the tutor user id operation derefs
> the get tutor post operation un-guarded ONLY on the legacy listing branch — tutor User null ⇒ tutor Post
> non-null by construction; resolve Tutor User Id Safe is the safe twin. Audited, accepted, no change.

---

## INT-097 — Tutor has no contact-reveal on an ACTIVE match (one-sided personal information crossing) (S2, frontend)

### Problem
On an ACTIVE match the tutee can reveal the tutor's real details, but the tutor — who travels to the
first session and pays the commission — cannot see the family's contact. The persona asymmetry is a
funnel gap, not a security gap.

### Current state (verified in code)
- Contact Reveal Button is mounted ONLY in Matches implementation reference
  (grep over the private implementation folder → single mount, verified). Matches implementation reference renders commission
  badges + the admin-email box (64-73) but **no reveal control**.
- **The endpoint is symmetric and ready.** Contact Reveal Service implementation reference:
  the assert phone verified operation (54) keys the INT-082 phone gate on
  the **caller**, not on a role — so it applies identically to the tutor side. Participant gating
  (63-69) computes is Tutor/is Tutee from the resolve tutor user id operation and the tutee id and **403s a
  non-participant for either side** → read operation for matches id contact is already correct for a tutor caller.
  Returns the counterparty (counterparty User Id is Tutor ? tutee User Id : tutor User Id, 83) — so a
  tutor reveal returns the *tutee's* details automatically.
- Contact Reveal Button is **persona-agnostic** (Contact Reveal Button implementation reference —
  props are the match identifier only; the INT-082 Phone Verify Modal + use Phone Verify Gate are role-neutral).
  Reusable on the tutor card with zero new props and **zero backend change**.

### Design / fix (frontend only)
Mount the contact-sharing button for the selected match on the tutor ACTIVE-match card in
Matches implementation reference, gated on match status equals MATCH STATUS ACTIVE — mirror the tutee placement
(Matches implementation reference), inside render Card after the commission badges (54-62). Keep the
existing admin-email box (it serves a different purpose — admin coordination) or place the reveal
directly beneath it. No backend change; the phone-verify gate already generalises (INT-082 — and stays
dormant while the feature is OFF, so this adds a button that just works today).

**REQUIRED (team-review 2026-06-11) — tutee-side mutual-visibility microcopy on the ACTIVE match card.**
The reveal writes an audit row (Contact Reveal Service) but emits NO counterparty notification — so the
family must be told visibility is mutual BEFORE it happens. Add to the tutee ACTIVE-match card: **"Once your
match is active, you and your tutor can both see each other's contact details — shared only after our team
confirms the match."**

### Test plan
- **[Testcontainers/per-pull request]** Matches test implementation reference: an ACTIVE tutor match → assert the
  "Reveal contact details" button renders; click → read operation for matches <id> contact is called and the
  Contact Reveal Card renders the counterparty (tutee). A non-ACTIVE match → no button. Mirror the existing
  Matches test implementation reference reveal assertions (the component already has its own
  Contact Reveal Button test implementation reference for the gate paths — no need to re-test those).
- **[Testcontainers/per-pull request]** Matches test implementation reference: the mutual-visibility microcopy renders
  on the tutee ACTIVE-match card (exact string above).

### Acceptance criteria
1. On an ACTIVE match the tutor sees and can use **Reveal contact details**; it returns the family's
   contact via the participant-gated endpoint (and routes the INT-082 verify flow when the gate is ON).
2. No backend change; the reveal is symmetric with the tutee side.
3. The tutee ACTIVE-match card carries the mutual-visibility microcopy (the family is told visibility is
   mutual before it happens).
4. the documented operational check + the documented operational check + the documented operational check green.

---

## Observability Do D (team-review 2026-06-11)

Add a **commission-payment counts grouped by success, not-payable, conflict or error** micrometer counter in the pay path —
Wave A's missing-value failure class was invisible in metrics; a per-endpoint money-path counter is the minimal fix.

## Follow-up filed — INT-125 (team-review 2026-06-11, out of Wave A scope)

Assign expires PENDING invitations with **NO notification to the invited tutors**
(Admin Match Service implementation reference — no recipient capture; the after-commit fan-out covers winner/parent/
superseded applicants only). Out of Wave A's frontend-only scope for INT-090 — filed as **INT-125**.

## No migrations

**This wave requires no Liquibase changeset.** All four fixes are application code only — the schema
they depend on (matches tutor user id re-key, Match tutor User, the resolve tutor user id operation, migration
0033) already exists and is deployed. No migration number is allocated; migrate-then-boot is untouched.

## Owners

| Issue | Lane | Owner | Reviewer |
|---|---|---|---|
| INT-090 | frontend (admin Matching + queue + route constant) | frontend-expert + bff-specialist | qa-engineer |
| INT-091 | backend (Admin Commission Service) | software-engineer | qa-engineer |
| INT-092 | backend (Review Service) | software-engineer | qa-engineer |
| INT-097 | frontend (tutor/Matches) | frontend-expert | uiux-designer (persona symmetry) |

INT-091 and INT-092 are the **same fix shape** (the get tutor post operation → the resolve tutor user id operation),
on **distinct files**, so they parallelise within the intuition-backend lane only if edited
sequentially (one the documented operational check per repo — the concurrent-gate-corruption lesson). INT-090 + INT-097 are the
intuition-frontend lane (distinct files, safe to parallelise from the backend lane).

## Risk / rollout note (this touches the LIVE revenue path)

- **Blast radius:** INT-091/092 sit on the money + reputation path. The fix is a one-line resolve swap to
  a helper already used in 6 sites, so risk is low — but **prove it with a free-market-match IT**, not a
  unit-only change, because the legacy listing-backed fixtures are exactly what hid the missing-value failure. A unit test
  that mocks the get tutor post operation non-null would pass against the bug.
- **Sequencing:** ship **backend first** (INT-091/092) so that by the time INT-090 lets admins assign
  free-market matches from the user interface, the downstream pay/review steps already work — otherwise the frontend fix
  surfaces more matches that then 500 on pay. Order: backend (091, 092) → deploy + reseed dev (so the live-stack
  ITs reflect the fix — regression-its-live-stack lesson) → frontend (090, 097) → the documented operational check + test:ci.
- **Definition of done:** verify gates yourself (the documented operational check ≥97.5/97.5; the documented operational check AND
  test:ci ≥97.5/88 — never trust a "green" claim), confirm the stack deploys + health-green locally,
  then push per-subrepo on the current branch (sp-dev) with a concise message + the Co-Authored-By
  trailer. Stage real source/tests only (no coverage/, no .env implementation reference).
- **Verify the wiring, not just the code (INT-075):** before declaring INT-090 done, grep the frontend for
  the new selection Request Assign route constant at the call site and run the Matching click-test —
  the endpoint existing is not the endpoint being called. This is the exact trap that shipped INT-034's
  unwired half.
