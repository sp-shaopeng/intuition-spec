# INT-039 — Interview Timeslot Negotiation — Architecture Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** [021](../021-tutee-request-flow/spec.md) permits PENDING-post invitation/interview initiation, preserves post-match contact, and requires pending-record cleanup on rejection. [007](../007-generic-scheduler-db-scheduler/spec.md) owns scheduler registration/lifecycle; its mechanism supersedes raw scheduling details here without deleting the72-hour negotiation-expiry rule. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Issue:** INT-039 · **Epic:** M6 Interview lifecycle redesign · **Size:** L
**Status:** DRAFT for /team-review (debate → Boss verdict) before build
**Date:** 2026-06-03
**Lead:** Solution Architect · **Contributors:** Software Engineer, DBA, frontend-facing service Specialist, user interface/user experience Designer
**Liquibase changeset:** **0034** (assigned; 0031=M19/Epic-O reserved per CLAUDE md, 0032 reserved M19, 0033 reserve buffer)
**Ties to:** INT-035 (admin sets concrete time + meeting link in the same step — described as implemented in the historical record, the agreed interview time and meeting link)

> **Conventions (binding, from CLAUDE md):** Java 21; Liquibase for ALL schema (no raw database structure definitions); no magic
> strings (frontend paths → application programming interface ROUTES in api Routes implementation reference; backend method-level path
> constants in the controller as today; enum values → typed enums); coverage gate **≥97.5% line AND
> branch (backend)**, **≥97.5% line / ≥88% branch (frontend)**; error envelope nested a structured error containing its error code
> raised via App Exception factories (bad Request 400 / forbidden 403 / not Found 404 /
> conflict 409 / unprocessable Entity 422); soft-delete only (no hard delete of negotiation rows).

> **Ground truth read before writing (CONFIRMED against live code 2026-06-03):**
> - Interview (Interview implementation reference): unique identifier PK; tutor Post **nullable**, tutor User, tutee,
>   tutee Post **nullable**, note, status (PG named enum interview status via
>   the jdbc type code operation), scheduled At, meeting Link, requested At, no Show Actor.
> - Interview Status (Interview Status implementation reference) = requested, scheduled, completed, cancelled or no-show.
>   PG enum created in 0011-create-interviews implementation reference: Allowed values for interview status: ....
> - Interview Service request Interview enforces: tutee post owned and ACTIVE/PENDING for initiation (021); tutor identity by
>   tutor Post Id (listing) OR tutor User Id (free-market applicant, must have open application);
>   duplicate guard; **cap** MAX INTERVIEWS PER POSTING 3 counting REQUESTED, SCHEDULED
>   (count-then-insert, TOCTOU, best-effort — documented in max-interviews-per-posting md).
> - Admin actions today: schedule (REQUESTED→SCHEDULED, sets scheduled At from Singapore time wall-clock +
>   optional INT-035 link), set Meeting Link (SCHEDULED only), complete, admin Cancel, mark No Show.
> - Notifications via the emit operation. Existing
>   types: INTERVIEW REQUESTED, INTERVIEW SCHEDULED, INTERVIEW CANCELLED, INTERVIEW COMPLETED,
>   INTERVIEW NO SHOWPG enumnotification type ; extended through ALTER TYPE ... ADD VALUE e g. 0013).
> - Enum-extension pattern: ALTER TYPE `<t>` ADD VALUE when not already defined `<v>` in its **own changeset**
>   with **run In Transaction is no** (PG forbids using a new label in the same tx — see 0025).
> - Changelog head = 0031. frontend routes registry interviews:  create, my List, my Detail, my Cancel,
>   tutor List, admin List, admin Detail, admin Schedule, admin Meeting Link, admin Complete, admin Cancel,
>   admin No Show .

---

## 1. Problem

Today the interview flow is a **unilateral admin schedule**: REQUESTED → admin SCHEDULED (admin
picks the time out of thin air). The tutor has **no say** in timing, the tutee proposes **no times**,
and the admin must guess a slot that suits two people they cannot see the calendars of. This causes
no-shows, back-channel rescheduling over email/phone (personal information leakage risk, INT lessons), and admin churn.

INT-039 introduces a **two-party timeslot negotiation** that happens **before** the admin step:

1. Tutee requests an interview and **proposes up to 3 timeslots** (within the next 7 days, each 15 min).
2. Tutor **accepts** one slot, or **declines** with a typed reason (NOT INTERESTED | CANT MAKE IT).
3. On CANT MAKE IT, the tutor may **counter-propose** his own up-to-3 slots.
4. The tutee may then **accept** a countered slot, **counter again**, or **cancel**.
5. The ping-pong continues until **mutual agreement** → status **PENDING SCHEDULING**.
6. Only then does the **admin** take over: sets the concrete time + meeting link (INT-035 reuse).
7. The **admin has full read visibility throughout and can override/step in at any state**.

The negotiation must reuse the existing entity, cap, notification, and admin-scheduling machinery
rather than fork a parallel system.

### Scope deltas vs the issue sketch (decisions taken in this spec)
- **NOT INTERESTED is terminal** → TUTOR DECLINED and **frees the cap slot** (it is not an active
  interview). It is a hard "no, not me" — there is nothing to counter. (Open decision D1 confirmed.)
- **CANT MAKE IT is non-terminal** → moves to COUNTER PROPOSED and **stays active** (consumes a slot).
- **AGREED is a transient agreement marker that immediately yields PENDING SCHEDULING** in the same
  transaction — we model the handoff as a single status PENDING SCHEDULING with an agreed slot id
  rather than two rows of state, to avoid a state nobody acts on (D4). See §3 state machine.
- **Expiry:** a PROPOSED / COUNTER PROPOSED interview with no counterparty response **auto-expires
  after 72h** of inactivity → EXPIRED (frees the slot). Scheduled job, not a request-path check (D2).

---

## 2. Design

### 2.1 Reuse, don't fork
The negotiation lives **on the existing interviews row** (same id, same tutee post id cap, same
admin views). We add a **child table** interview timeslots for the proposed slots and **extend the
interview status enum** with the negotiation states. The existing REQUESTED and SCHEDULED states
are **retained for backward compatibility** with legacy rows and the listing-less path, but the new
request flow starts at PROPOSED (see §3 migration note for how legacy rows coexist).

### 2.2 New Interview Status values
Added to the Java enum and the PG interview status type:

| Value | Meaning | Active? (consumes cap slot) |
|---|---|---|
| PROPOSED | Tutee has proposed 1–3 slots; awaiting tutor. | **Yes** |
| COUNTER PROPOSED | A party declined CANT MAKE IT and proposed new slots; awaiting the other party. | **Yes** |
| TUTOR DECLINED | Tutor declined with NOT INTERESTED. **Terminal.** | No |
| PENDING SCHEDULING | Both parties agreed on a slot; awaiting admin to set concrete time + link. | **Yes** |
| EXPIRED | No counterparty response within the inactivity window. **Terminal.** | No |

> AGREED from the sketch is **not** a persisted status — agreement transitions directly to
> PENDING SCHEDULING in one tx (see §1 scope delta). The agreed slot is captured on
> interviews agreed timeslot id.

decline reason is a **separate PG enum** decline reasons: not interested or cannot make the proposed time,
nullable column on interviews (set when a decline happens; cleared on a fresh counter round).

### 2.3 Slot ownership & states
Each proposed slot is a row in interview timeslots:
- proposed by ∈ tutee or tutor (PG enum timeslot proposer).
- slot status ∈ proposed, withdrawn, accepted or rejected (PG enum interview timeslot status).
- A **round** is a batch of up to 3 slots proposed by one party. When the other party counters, the
  previous round's still-PROPOSED slots are flipped to WITHDRAWN (soft, audit-preserving) and a
  new round inserted. Exactly **one** slot ever reaches ACCEPTED (the agreed one); on agreement all
  sibling slots in that round flip to REJECTED.

### 2.4 State machine (text diagram)


Implementation example omitted. This public edition retains the written requirements in this section.


**Cap interaction (MAX INTERVIEWS PER POSTING 3):** the "active" set used by
count By Tutee Post Id And Status In is **extended** to
proposed, counter-proposed, awaiting staff scheduling or scheduled plus legacy REQUESTED. Terminal states
(TUTOR DECLINED, EXPIRED, CANCELLED, COMPLETED, NO SHOW) do **not** consume a slot. This
preserves the founder rule "max 3 interviews per posting" across the new lifecycle. The count-then-
insert TOCTOU caveat from max-interviews-per-posting md still applies; unchanged.

**Symmetric ping-pong:** counter and the decline operation are modelled by a single
service operation that (a) validates it is the requester's turn, (b) withdraws the current open round,
(c) inserts the new round, (d) sets proposed by and toggles whose turn it is via the resulting
status + the round's proposed by. "Whose turn" is derived: the party who did **not** propose the
current open round is the one allowed to accept/decline/counter.

---

## 3. Schema — Liquibase changeset 0034

File: 0034-interview-timeslot-negotiation implementation reference
Add  implementation reference to
db changelog-master implementation reference after the 0031 line.

> **Enum-extension rule (PG):** each ALTER TYPE ... ADD VALUE must be its **own changeset** with
> **run In Transaction is no** and when not already defined, because PG forbids using a freshly added label
> in the same transaction (precedent: 0013, 0025). New types created with CREATE TYPE are fine
> in-transaction.

### 3.1 Changesets (in order)


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.2 DBA notes
- automatically remove dependent records on interview timeslots interview id is correct because slots are pure children
  with no independent meaning. But per the founder **soft-delete** rule, the app **never** deletes an
  interview — terminal states are reached via status, so cascade only matters for test teardown.
- agreed timeslot id FK is added in a **separate changeset (e)** after both objects exist (a column
  can reference a table created earlier in the same changelog, but splitting keeps each changeset
  independently rollback-able).
- All timestamps are TIMESTAMPTZ. App stores Coordinated Universal Time; the **15-min grid is enforced in Coordinated Universal Time**, and the
  admin's Singapore time wall-clock conversion (Asia/Singapore) remains the existing schedule Interview concern.
- The ck timeslot quarter hour check is defence-in-depth; primary validation is in the service so
  the user gets a typed App Exception code, not a constraint-violation 500.
- interviews updated at is set by the app on every negotiation write (it backs the expiry sweep).
  No database trigger (the codebase uses app-managed timestamps).

### 3.3 Entity changes (Interview implementation reference)
- Add the enumerated operation (column Definition
  interview decline reason), nullable.
- Add the one to many operation.
- Add unique identifier agreed Timeslot Id, Timeslot Proposer last Actor, Offset Date Time updated At.
- New entity Interview Timeslot mirroring the table; new enums Decline Reason, Timeslot Proposer,
  Timeslot Status in the application component.

---

## 4. application programming interface contracts

### 4.1 No magic strings
- **frontend:** extend the interviews block in api Routes implementation reference (existing
  registry, line 66). New keys below.
- **backend:** add path-constant fields in Interview Controller alongside TUTEE BASE/TUTOR BASE/...
  (the controller's existing pattern), and any shared route to the application component Routes if a
  test/security matcher needs it. Error codes are typed constants in Interview Service (existing
  ERR * pattern).


Implementation example omitted. This public edition retains the written requirements in this section.


### 4.2 Endpoints

| Method & path | Role | Body | Allowed from → to | Notes |
|---|---|---|---|---|
| submit operation for interviews | TUTEE | Propose Interview Request (below) | (new) → PROPOSED | **Replaces** the old REQUESTED-creating call: now requires 1–3 slots. |
| submit operation for tutor interviews id accept | TUTOR | the chosen time-slot identifier | PROPOSED/COUNTER PROPOSED using tutee round → PENDING SCHEDULING | Picks one open tutee slot; ACCEPTED, siblings REJECTED. |
| submit operation for tutor interviews id decline | TUTOR | the reason (+ optional slots when CANT MAKE IT) | PROPOSED/COUNTER PROPOSED → TUTOR DECLINED (NOT INTERESTED) **or** COUNTER PROPOSED (CANT MAKE IT + slots) | NOT INTERESTED terminal+frees slot; CANT MAKE IT with slots = decline+counter in one call. |
| submit operation for tutor interviews id counter | TUTOR | one to three proposed start times | COUNTER PROPOSED/PROPOSED (tutee round) → COUNTER PROPOSED | Pure counter (when tutor just wants to propose alternatives without a decline reason — folded into decline for CANT MAKE IT; kept for symmetry/user experience). |
| submit operation for tutee interviews id accept | TUTEE | the chosen time-slot identifier | COUNTER PROPOSED (tutor round) → PENDING SCHEDULING | Picks one open tutor slot. |
| submit operation for tutee interviews id counter | TUTEE | one to three proposed start times | COUNTER PROPOSED (tutor round) → COUNTER PROPOSED | Tutee proposes a fresh round. |
| submit operation for tutee interviews id cancel | TUTEE | — | non-terminal → CANCELLED | **Existing** endpoint; widen allowed source statuses (§5). |
| submit operation for tutor interviews id cancel | TUTOR | — | non-terminal → CANCELLED | **New** — tutor may bail out of an in-flight negotiation. |
| read operation for tutee interviews id, read operation for tutor interviews id | owner | — | — | Return negotiation detail incl. timeslots + whose Turn. (Tutor detail is **new**.) |
| submit operation for admin interviews id schedule | ADMIN | the agreed interview time and meeting link | PENDING SCHEDULING (and legacy REQUESTED) → SCHEDULED | **Widen** source to include PENDING SCHEDULING (INT-035 reuse). |
| submit operation for admin interviews id force-pending | ADMIN | an optional chosen time-slot identifier | any non-terminal → PENDING SCHEDULING | Admin override: pick a proposed slot (or just advance) when parties stall. |
| submit operation for admin interviews id cancel | ADMIN | — | non-terminal → CANCELLED | **Existing**; widen source statuses. |
| read operation for admin interviews, read operation for admin interviews id | ADMIN | — | — | **Existing**; response gains negotiation fields + timeslots (full visibility). |

### 4.3 Request data transfer objects (validation)


The request dt os validation example describes information including tutor Post Id, tutor User Id, tutee Post Id, note, slots. Concrete payload values and implementation syntax are omitted.


**Server-side slot validation (every propose/counter):**
- 1–3 slots; each on a 15-min Coordinated Universal Time boundary (SLOT NOT ON GRID).
- Each start at within **the now operation … now operation+7 days** (SLOT OUT OF WINDOW); must be in the future
  (SLOT IN PAST).
- No duplicate start at within the round (SLOT DUPLICATE).
- Slot duration is fixed 15 min (implicit end start + 15m); not client-supplied.

**New error codes** (typed in Interview Service): SLOT REQUIRED, SLOT TOO MANY,
SLOT NOT ON GRID, SLOT OUT OF WINDOW, SLOT IN PAST, SLOT DUPLICATE, TIMESLOT NOT FOUND,
TIMESLOT NOT OPEN, NOT YOUR TURN, plus reuse of existing INVALID STATUS TRANSITION,
INTERVIEW NOT FOUND, INTERVIEW LIMIT REACHED, INTERVIEW NOT FOUND (owner mismatch → 404).

### 4.4 Response shape
Interview Response / Admin Interview Response gain:


The response shape example describes information including status, decline Reason, whose Turn, agreed Timeslot Id, timeslots, start At. Concrete payload values and implementation syntax are omitted.


frontend-facing service note: **single round-trip** — the detail GET returns the full negotiation (all rounds + whose
turn) so the frontend never has to stitch slot calls. Admin list stays paginated and does **not** embed
slots (avoid over-fetch); admin **detail** embeds them.

### 4.5 Notifications (both-party, per transition)
Reuse Notification Service emit. After-commit, failure-isolated per existing pattern. Mapping:

| Transition | Notify | Type | Link |
|---|---|---|---|
| propose (→PROPOSED) | tutor + all admins | INTERVIEW PROPOSED | tutor: the tutor's interview-detail page |
| tutor/tutee counter (→COUNTER PROPOSED) | the other party + admins | INTERVIEW COUNTERED | other party's detail |
| decline NOT INTERESTED (→TUTOR DECLINED) | tutee + admins | INTERVIEW DECLINED | the tutee's interview-detail page |
| decline CANT MAKE IT (+counter) | tutee + admins | INTERVIEW DECLINED then INTERVIEW COUNTERED (or one combined body) | tutee detail |
| accept (→PENDING SCHEDULING) | **both** parties + admins | INTERVIEW AGREED | each party's detail |
| admin schedule (→SCHEDULED) | tutee (+tutor) | INTERVIEW SCHEDULED (existing) | existing |
| expiry (→EXPIRED) | both parties + admins | INTERVIEW EXPIRED | each party's detail |
| cancel | other party + admins | INTERVIEW CANCELLED (existing) | existing |

All bodies must use the existing the safe title operation truncation before interpolating titles into the
varchar using 500 body (lesson: long title rolls back the parent tx).

---

## 5. Backend behaviour changes (service)

1. **request Interview → propose Interview:** all existing guards retained (tutee post owned and ACTIVE/PENDING for initiation (021),
   tutor identity, duplicate, cap). New: validate slots (§4.3), persist Interview with
   status is PROPOSED, last Actor is TUTEE, updated At is now, and insert round-1 interview timeslots
   (proposed by is TUTEE). Notify per §4.5.
2. **Cap set widened** (§2.4) in count By Tutee Post Id And Status In and the partial index
   idx interviews tutee post active predicate → must include the new active states. **This means the
   0021 partial index predicate must be recreated** in 0034:


Implementation example omitted. This public edition retains the written requirements in this section.


   (Add as a changeset in 0034, after the enum ADD VALUEs — uses the new labels, so it sits in its own
   tx **after** the ADD VALUE changesets committed; placement guarantees the labels exist.)
3. **Turn enforcement:** every accept/decline/counter checks the whose turn operation
   else 422 NOT YOUR TURN. whose Turn = the party who did **not** propose the current open round
   (derived from the latest round's proposed by / last Actor).
4. **Accept:** validate timeslot Id belongs to this interview, is PROPOSED, and is owned by the
   counterparty; set it ACCEPTED, sibling open slots in its round REJECTED, set
   agreed timeslot id, status is PENDING SCHEDULING. (No AGREED persisted.)
5. **Counter / decline-CANT MAKE IT:** withdraw current open round (→WITHDRAWN), insert new round
   (round max+1, proposed by caller), status is COUNTER PROPOSED, flip last Actor.
6. **Decline-NOT INTERESTED:** status is TUTOR DECLINED, set decline Reason, withdraw open slots.
   Terminal; frees cap slot.
7. **schedule Interview:** widen the source-status guard from REQUESTED only to
   awaiting staff scheduling or the earlier requested state (legacy). When agreed, default scheduled At to the agreed slot's
   start at (admin may override). INT-035 link path unchanged.
8. **cancel My Interview / admin Cancel Interview:** widen allowed source statuses from
   requested or scheduled to requested, scheduled, proposed, counter-proposed or awaiting staff scheduling.
9. **Expiry sweep:** a recurring db-scheduler task (007) every15 min: find interviews in
   proposed or counter-proposed with a last-update timestamp more than 72 hours in the past, set EXPIRED, withdraw open slots,
   notify both parties. Idempotent; uses the same updated At. (Devops: one in-process scheduler is
   the historical single-instance assumption does not safely extend to several instances. Multiple workers must claim eligible records with locks and skip records another worker has claimed. This was recorded as open decision D3.)

> **Concurrency lesson (MEMORY: Java persistence mapping clear Automatically detaches / INT-016):** do **not** use
> the modifying operation bulk updates that evict the context mid-tx while we still hold
> lazy interview timeslots / tutee references for the notification bodies. Capture all data needed
> for notifications **before** any bulk status flip, or do row-by-row entity updates. Reproduce any
> Sev bug before patching.

---

## 6. user experience flow (user interface/user experience)

**Tutee — request → negotiate:**
1. From a tutor detail / application, "Request interview" opens a form: posting selector (existing cap
   nudge "2 of 3 used"), optional note, and a **slot picker**: a 7-day calendar in **Singapore time**, 15-min
   grid, add **up to 3** slots (chips with remove). Submit disabled until ≥1 slot and cap not reached.
2. After submit: status **"Waiting for tutor"** with the 3 proposed times listed. Clear "what happens
   next" microcopy.
3. If tutor declines NOT INTERESTED: a calm terminal state "Tutor isn't available for this request"
   — offer "request another tutor" (cap slot is freed).
4. If tutor counters (CANT MAKE IT): banner "Tutor suggested new times" → tutee can **Accept a time**,
   **Suggest other times** (re-open picker, ≤3), or **Cancel**. A visible "It's your turn" pill.
5. On agreement: "Time agreed — our team will confirm the meeting link shortly" (PENDING SCHEDULING).
6. On schedule: existing scheduled view with concrete time + meeting link.

**Tutor — respond:**
- New tutor interview **detail** page (today tutor only has a list). Shows proposed slots as
  selectable cards. Primary actions: **Accept** (radio-select one slot), **Can't make it →** (reason
  modal: pick CANT MAKE IT then add ≤3 alternative times, or NOT INTERESTED = decline outright),
  **Cancel**. "It's your turn" / "Waiting for tutee" state pill mirrors the tutee side.

**Cross-cutting user experience rules:**
- **One source of truth for "whose turn"** = the whose Turn field; both sides render the same pill so
  nobody waits on the other (a real INT-class confusion bug class).
- All times shown in **Singapore time** with explicit "(Singapore time)" label; 15-min slots shown as 2:00–2:15 PM.
- Map every error code to inline copy (SLOT OUT OF WINDOW → "Pick a time within the next 7 days",
  NOT YOUR TURN → "Waiting for the other person to respond", INTERVIEW LIMIT REACHED → existing).
- Empty/limit states reuse the existing cap nudges. No personal information exchanged in the negotiation user interface (no
  email/phone) — contact reveal stays a post-match concern.
- a11y: slot picker is keyboard-navigable, radios labelled, status pills are not colour-only.

**Admin (Ops Console):** existing admin interview list + detail gains a read-only **negotiation
timeline** (rounds, who proposed what, reasons, timestamps) and override buttons: **Force to
scheduling** (optionally pick a slot), **Schedule** (concrete time + link, pre-filled with agreed
slot), **Cancel**. Admin sees everything at every state.

---

## 7. Acceptance criteria

1. Tutee can create an interview proposing **1–3** future slots, each on a 15-min boundary within
   **7 days**; 0 slots or >3 → 422 (SLOT REQUIRED/SLOT TOO MANY); off-grid → SLOT NOT ON GRID;
   out of window → SLOT OUT OF WINDOW. New interview is PROPOSED.
2. Tutor accept of a valid open tutee slot → PENDING SCHEDULING, agreed timeslot id set, that slot
   ACCEPTED, siblings REJECTED; **both** parties + admins notified (INTERVIEW AGREED).
3. Tutor decline NOT INTERESTED → TUTOR DECLINED (terminal), decline Reason persisted, tutee
   notified, and the **cap slot is freed** (a new request for that posting then succeeds even if 3
   were previously in flight).
4. Tutor decline CANT MAKE IT + 1–3 counter slots → COUNTER PROPOSED, previous round WITHDRAWN,
   whose Turn is TUTEE; tutee notified.
5. Tutee may then accept (→PENDING SCHEDULING), counter again (→COUNTER PROPOSED, whose Turn is TUTOR),
   or cancel (→CANCELLED); ping-pong can repeat ≥3 rounds without error.
6. **Turn enforcement:** acting out of turn → 422 NOT YOUR TURN (e g. tutee accepts while it's the
   tutor's turn).
7. **Cap:** active states requested, proposed, counter-proposed, awaiting staff scheduling or scheduled count
   toward the 3-per-posting limit; terminal states do not. A 4th active proposal → INTERVIEW LIMIT REACHED.
8. Admin sees the full negotiation timeline at every state; admin **force-pending** and **schedule**
   work from any non-terminal state; admin **cancel** works from any non-terminal state.
9. Admin schedule from PENDING SCHEDULING sets concrete time (defaulting to the agreed slot) +
   optional meeting link (INT-035), → SCHEDULED; existing complete/no-show flow unchanged.
10. Expiry: a PROPOSED/COUNTER PROPOSED interview untouched for 72h → EXPIRED, slots withdrawn,
    both parties notified, cap slot freed.
11. Ownership: a tutee/tutor acting on an interview they don't own → 404 (INTERVIEW NOT FOUND),
    never a 403 leak of existence (existing pattern).
12. No magic strings (frontend application programming interface ROUTES, backend constants); Spotless + ESLint clean; coverage gates hold
    (backend ≥97.5/97.5, frontend ≥97.5/88). Liquibase 0034 applies cleanly on a fresh database and is in master.

---

## 8. Test plan

**Backend unit/service (Interview Service Test):**
- propose: slot count bounds (0/1/3/4), grid/window/past/duplicate validation, cap interaction,
  duplicate-tutor guard still fires, listing vs listing-less identity both work.
- accept (tutor & tutee): valid slot, slot not open, slot belongs to wrong party, wrong-turn,
  status not eligible → each typed code.
- decline NOT INTERESTED (terminal + slot freed) and CANT MAKE IT (+counter) paths.
- counter: round increments, previous round withdrawn, last Actor/whose Turn flip; repeated rounds.
- cancel from each non-terminal state; admin cancel/force-pending/schedule from each eligible state.
- schedule from PENDING SCHEDULING defaults to agreed slot; from REQUESTED legacy still works.
- expiry sweep: boundary at exactly 72h, idempotency, notification fan-out.
- notification emission asserted for **both** parties + admins per transition; safe Title truncation.
- **Branch-coverage focus** (97.5% gate): every INVALID STATUS TRANSITION and NOT YOUR TURN branch.

**Backend integration (Interviews IT, regression — runs against live stack localhost; reseed +
rebuild/redeploy first, MEMORY lesson):** full happy-path negotiation (propose→counter→counter→accept→
admin schedule), decline-terminal path, cap-freeing after decline, admin override path,
optimistic/concurrent accept (two requests, one wins → second gets a clean transition error).

**Frontend (Vitest):** slot picker (add/remove, ≤3, grid, 7-day window, Singapore time rendering), whose-turn
pill on both roles, error-code → copy mapping, action enable/disable by state, accept/counter/decline
flows mocked, admin timeline render. Branch gate ≥88%.

**Manual / bug-founder (post-deploy):** drive a real two-account negotiation end to end; verify no personal information
leaks in negotiation payloads; verify expiry by clock-skewing a seeded row; attempt out-of-turn and
cross-owner actions.

---

## 9. Open decisions (for /team-review)

- **D1 — NOT INTERESTED terminates?** Spec says **yes** (terminal TUTOR DECLINED, frees cap slot).
  Alternative: keep it non-terminal so admin can still force a match — rejected as user-hostile.
- **D2 — Expiry window.** Spec proposes **72h** inactivity → EXPIRED. Confirm value (24h? 7-day
  hard stop regardless of activity since all slots are within 7 days anyway?).
- **D3 — Expiry mechanism.** In-process the scheduled task (single-node) vs with a record lock SKIP LOCKED claim
  (multi-node-safe) vs an admin-triggered sweep. Devops to weigh against current deploy topology.
- **D4 — Persist AGREED?** Spec collapses agreement straight to PENDING SCHEDULING (no dangling
  AGREED state). Confirm no consumer needs a distinct AGREED.
- **D5 — Tutee "decline"?** Spec gives the tutee **cancel** + **counter** but no "decline reason"
  (the tutee initiated; declining their own request = cancel). Confirm.
- **D6 — Round cap?** Should we bound ping-pong (e g. max 6 rounds) before forcing admin involvement,
  to prevent infinite negotiation? Proposed: no hard cap; expiry + admin override are the backstops.
- **D7 — Slot overlap across the parent posting?** Should a tutee be blocked from proposing the same
  start at across two different interviews on the same posting? Proposed: allow (they may interview
  multiple tutors and pick one) — admin schedules concretely later.
- **D8 — Cap index recreation** (§5.2) changes a deployed index predicate; confirm online
  support efficient lookup with an index impact is negligible at current row counts (it is).

---

## 10. Out of scope
- Calendar integrations (Google/ICS), tutor availability calendars, timezone selection (Singapore time-only).
- Meeting-link generation/automation (admin still pastes the link; INT-035 unchanged).
- Reminders/escalation beyond the single 72h expiry sweep.
- Changing the post-match contact-reveal flow.

---

## Boss-review conditions (2026-06-03)
1. **Expiry sweep observability + idempotency.** The 72h the scheduled task expiry job must be idempotent
   (set-based update records guarded by current status) and emit a Micrometer counter (interview expired) +
   an alert if it errors — otherwise an outage is invisible (Nova/Seb).
2. **INT-016 lazy-init IT.** Add an IT asserting the negotiation transitions that build notification
   bodies do NOT throw Lazy Initialization Exception — capture ids/subjects into primitives BEFORE any
   the modifying operation/post-commit access (see [[jpa-clear-automatically-detaches]]).
3. **Recreate the 0021 cap index.** The max-3 partial index predicate must be recreated in 0034 to
   cover the widened active set requested, proposed, counter-proposed, awaiting staff scheduling or scheduled,
   or the cap silently under/over-counts.
