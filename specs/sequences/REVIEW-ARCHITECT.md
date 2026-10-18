# Architect Verification — Sequence Diagrams

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Reviewer: Solution Architect (Mira)
Scope: All 27 diagrams under intuition-spec/specs/sequences/ against the live
backend code under intuition-the private implementation folder
Lens: cross-cutting concerns, transaction boundaries, async vs sync,
race/TOCTOU, failure paths, internal consistency of the diagram set.

## Verdict

**Conditionally accepted.** The set is unusually honest — it surfaces real
gaps (no multi-factor authentication wiring in Login implementation reference, no admin notification on post submission,
no email-consumer in the tree, queue Approval Email not wrapped in
after Commit). That's the bar I want from a spec.

But it is **not architecturally complete**. Six cross-cutting concerns are
shown inconsistently or not at all across flows that all share the same
filter chain, the same transaction semantics, and the same notification
substrate. If a reader picks any single diagram in isolation, they will draw
the wrong conclusions about what protects that endpoint and what fails open vs
closed. Specific gaps below.

Most importantly, there is a **real bug surfaced by reading the code in
parallel with the diagrams** — Auth Service login writes lock Until /
increment Failed Attempts and then throws App Exception locked, which
inherits Runtime Exception. Spring's default rollback rules will undo the
lockout write. None of the diagrams 03 / 24 flag this. (See Required
additions, item 1.)

---

## Cross-cutting findings

### 1. cross-site request forgery protection coverage is implicit, not visible

Flow 25 documents the double-submit mechanism correctly. But every
authenticated write flow (06, 07, 09–22) silently assumes the header is
present. Only flow 06 explicitly notes "cross-site request forgery protection 403 if header missing" under
failure modes. Others list 403 FORBIDDEN as "role mismatch" without
distinguishing it from "cross-site request forgery protection token mismatch" — these are two completely
different production failures with different remediation paths.

**Decision implied but not documented:** Security Config ignoring Request Matchers
covers login, login/mfa, register/**, forgot-password, reset-password,
** interface operation, /actuator/**. Everything else is cross-site request forgery protection-protected. The
diagram set has no canonical place that lists this — it's inferred by
reading flow 25's "Notable details" and the Security Config file.

### 2. Rate limit coverage is incomplete

Flow 24 only profiles the login endpoint. The actual rate-limited surface in
Auth Service + Profile Service is:

| Endpoint | Key | Default |
|---|---|---|
| submit operation for auth login | the login key operation | 10 / 3600s |
| submit operation for auth register tutee | the register key operation | 5 / 3600s |
| submit operation for auth register tutor | the register key operation | 5 / 3600s |
| submit operation for auth forgot-password | the forgot password key operation | 3 / 3600s |
| submit operation for auth change-password | the change password key operation | 5 / 900s |

Flows 01, 02, 04 show their gate. Login flow 03 shows it. change-password
has **no diagram at all** in this set even though Profile Controller implementation reference
exists — that's an explicit omission worth flagging if the M7 Profile module
is still incomplete (which the CLAUDE md says it is). The rate limit applies
even though no flow visualises it.

Flow 24 should be reframed as "Rate-limit substrate (any endpoint)" with a
tagged list of which flows are gated, instead of dressing itself up as a
login-specific drilldown.

### 3. Audit log emission is silent in flows that DO emit

Flow 01 and 02 correctly show an audit entry recording user registered and
an audit entry recording commission acknowledged. But these audit-log events also fire:

- **Flow 03 — login**: auth login.success analytics is shown but the
  ADMIN LOGIN audit (Auth Service implementation reference-356) is mentioned only in a
  trailing Note on the multi-factor authentication branch. The non-admin login does NOT write an
  audit log — that's worth surfacing as a deliberate decision (do we audit
  successful tutee logins? answer: no).
- **Flow 04 — reset-password**: shows PASSWORD RESET audit in the diagram.
  Good.
- **Flow 08 — approve/reject**: shows ADMIN POST APPROVE /
  ADMIN POST REJECT audit. Good.
- **Flow 09, 10, 11, 12 — match operations**: do NOT write audit logs. Is
  that intentional? Cancelling a match (financial impact, commission cascade)
  with zero audit trail is a compliance smell. None of these diagrams call
  this out as a deliberate decision OR as a gap.

The diagram set is internally inconsistent on what gets audited — flows 01,
02, 04, 08 show audit; flows 09–16, 19, 22 do not even though they perform
state changes that an auditor would want to trace.

### 4. Transaction boundary semantics are inconsistent across diagrams

Three patterns coexist in the code and the diagrams handle them
inconsistently:

**Pattern A — after Commit for side effects:** flows 01, 02, 04 show
after Commit for audit log writes and email enqueue. Correct and idiomatic.

**Pattern B — in-transaction side effects:** flows 06, 07 explicitly call
out that analytics Event Publisher publish runs inside the transaction
(Notable details paragraph 4 of flow 06). That's an architectural decision
worth a Note over Svc in the diagram body, not buried in prose. A phantom
analytics event on a rolled-back post is a real failure mode.

**Pattern C — in-transaction notification emit:** flows 09, 10, 11, 13, 14,
16, 19 all call notification Service emit inside the same a database transaction that saves all related changes together
boundary. The diagrams use par … and to show two emits — but none flags
that if the *second* emit fails, the *first* notification is rolled back too
because they share a transaction. That's actually the desired semantics
(consistency) but flow 13 in particular has a per-admin loop (lines 119-125
of Interview Service) — if the 7th admin's emit fails, the first 6
notifications and the interview record all roll back. Diagrams should show
the boundary explicitly via a Note over Svc,I Repo: same a database transaction that saves all related changes together.

**Pattern D — email enqueue inside transaction without after Commit:** Flow
26's audit note is correct: Admin Post Service queue Approval Email /
queue Rejection Email are called inline inside a database transaction that saves all related changes together approve Post,
not after commit. If the post status save succeeds but a downstream operation
(none today, but a future hook?) fails and rolls back, the approval email
still went to the user because @Async already handed it off. The architect
view: the email should be deferred to after Commit for symmetry with
Auth Service forgot Password. Flow 08 does not mention this — it just shows
the enqueue happening. Flow 26 is the only place this inconsistency is
called out, and only as a small audit paragraph.

### 5. Encryption / keyed message authentication lookup invisible in flows that perform it

Flow 27 is solid. But the only flow that explicitly shows compute Hmac in
its diagram body is flow 01. Flow 03 (login) hides it behind
the find by email hmac operation without showing the keyed message authentication computation. Flow 04
(forgot-password) shows the compute hmac operation — good. Flow 22 hides the fact
that no email/phone is used (it's all UUIDs) and so is fine.

The architectural concern: **any flow that returns email/phone to the wire
performs a decrypt step.** Flow 05 (/me) shows the decrypt field operation
explicitly — good. Flow 08 (admin post detail) shows decrypt Safe — good.
But the consequence — that key rotation strategy must cover both encryption
key AND keyed message authentication secret — is documented nowhere outside flow 27. No diagram in
the set covers a key-rotation flow, and no flow's failure modes mention
"keyed message authentication secret rotated → all logins fail with 401."

### 6. Notification emit policy is inconsistent and partly broken

Cross-checking flows against notification Service emit calls in actual
services:

| Flow | State change | Notification emitted in code? | Diagram says? |
|---|---|---|---|
| 06 tutee post create | DRAFT→PENDING | No (admin not notified) | Correctly flagged as gap |
| 07 tutor post create | DRAFT→PENDING | No (admin not notified) | Correctly flagged as gap |
| 08 admin approve/reject | PENDING→ACTIVE/REJECTED | No (email only) | Implies email is enough — should explicitly contrast with notification absence |
| 09 match create | →PENDING | Yes (both parties) | Correct |
| 10 match activate | →ACTIVE | Yes (both parties) | Correct |
| 11 commission paid | →COMPLETED | Yes (tutor only) | Correct — tutee gap is documented |
| 12 match cancel | →CANCELLED | **No** | Correctly flagged as gap |
| 13 interview request | →REQUESTED | Yes (tutor + all admins) | Correct |
| 14 interview schedule | →SCHEDULED | Yes (tutee only) | Correct — tutor gap is documented |
| 15 meeting link set | (link change) | No | Correctly documented as deliberate |
| 16 complete | →COMPLETED | Yes (tutee only) | Correct |
| 16 admin cancel | →CANCELLED | Yes (both) | Correct |
| 16 no-show | →NO SHOW | Yes (other party) | Correct |
| 16 tutee self-cancel | →CANCELLED | Yes (tutor only) | Correct |
| 19 review submit | new review | Yes (reviewee) | Correct |
| Any password change / account delete | — | No | Not visualised; deliberate |

**Architectural smell:** the policy is inconsistent across modules. Match
cancel notifies nobody (flow 12 gap), but interview admin-cancel notifies
both parties (flow 16). The same admin action — "I am cancelling this thing"
— has different communication consequences depending on which entity. That's
a user experience-level decision dressed up as an engineering decision. No diagram surfaces
the question.

Additionally, the email-fanout for those notifications **does not exist**.
Flow 26's gap (no consumer in tree) means every notification-only flow is
actually "we wrote a row to the notifications table; the bell will eventually
pick it up via polling (flow 21); no email was sent." That's a critical
architectural property — none of flows 09–19 mention it in their failure
modes. A user with notifications disabled in the bell user interface (or who simply
doesn't log in) will never know their match was cancelled.

### 7. Observability is shown only once

Flow 24 mentions request correlation filter and logging context populate. No other flow
references X-Correlation-ID despite every flow going through that filter
(Security Config implementation reference). Flow 21 (polling bell at 60s) is the highest-traffic
flow in the entire system and doesn't mention metrics or rate of polling
load on /notifications/unread-count. If the admin user count grows or the
bell polling interval is tuned, no diagram captures the operational impact.

---

## Concerns by flow

- **01 Register tutee** — Good. Shows after Commit, rate limit, encryption.
  Missing: which audit fields are populated; the signed session token cookie attributes are
  precise (good).
- **02 Register tutor** — Good. Stash mechanism documented. Missing: that
  the stash holds plaintext password in session Storage for 1 click — a real
  security-architecture decision worth a Note (it is mentioned in prose,
  not in the diagram).
- **03 Login + multi-factor authentication** — Diagram is correct on multi-factor authentication mechanics. **Missing
  critical:** lockout-write rollback bug (see Required Additions §1). multi-factor authentication
  attempt counter mutates Redis even though it's outside the Java persistence mapping transaction
  — that's actually correct, but the diagram doesn't make the
  Redis-state-survives-rollback property explicit.
- **04 Forgot/reset password** — Good. Best example of after Commit in the
  set. Worth duplicating its pattern (Note over Svc: after Commit) into
  other flows.
- **05 cross-site request forgery protection bootstrap** — Good. The /csrf → /me ordering rationale is
  surfaced. Missing: explicit interaction with Jwt Authentication Filter on
  /me — the diagram shows "Jwt Authentication Filter validates auth cookie"
  but doesn't show the JTI blocklist check (delete Account writes JTI to
  Redis). Probably fine — JTI is M1 deletion, not bootstrap.
- **06 Tutee post create** — Good gap call-out on analytics-before-commit
  and admin-notify absence. Missing: the in-transaction notification policy
  contrast.
- **07 Tutor post create** — Same as 06. Symmetric.
- **08 Admin approval** — Has audit log. Email enqueue inline (not
  after Commit). **Missing:** explicit acknowledgement that this is the
  inconsistency flow 26 flags. The diagram should Note: enqueue is inline,
  not after Commit — flow 26 §Audit.
- **09 Match create** — Correct on the dual notification fan-out. Missing:
  audit log. A match creation is a billable financial event and should be
  audited. Architect's recommendation: add an audit entry recording match created.
- **10 Match activate** — Correct on commission row creation. Missing:
  audit log. The 50% commission rate hard-coded at the model level (M5) is
  invisible in the diagram — DEFAULT COMMISSION RATE 0.5 is a business
  rule baked into code, worth a Note.
- **11 Commission paid** — Correct, including the tutee-not-notified gap.
  Missing: audit log for "marked paid" — this is the strongest single
  financial event in the system.
- **12 Match cancel cascade** — State machine is correct. Two gaps
  surfaced (no notification, no audit). Excellent. Missing: explicit
  callout that the commission row update happens in the same transaction —
  but flow 12 does say "save both" which is fine.
- **13 Interview request** — Correctly shows admin fan-out loop. Missing:
  the loop is **inside** the Java persistence mapping transaction. For an admin team of 50, that's
  50 INSERTs synchronously on the request path. Worth flagging as a scaling
  concern. Also, the safe title operation truncation is mentioned in prose — should be
  in the diagram body as a Note because it's an architectural defence
  (otherwise the body column overflows and the parent tx rolls back).
- **14 Schedule interview** — Correct on Singapore time zone handling. Missing: the
  state guard fires on status differs from REQUESTED (422) — this should be tagged
  on the state machine in flow 16, not just here.
- **15 Set meeting link** — Correct. Missing: personal information-leak risk. The
  meeting Link is admin-supplied; if it contains a tracking URL or personal information, it
  flows to the tutee unmodified. The diagram doesn't show a URL sanitisation
  step because there is none. Worth surfacing as either "deliberately
  unfiltered, admin is trusted" or "TODO sanitise."
- **16 Interview terminal paths** — State machine is correct. Guards are
  shown on the right transitions. Good. Missing: the contrast that
  tutee-self-cancel uses 404-on-not-owned (existence not leaked) — this is
  shown in flow 16 prose but should be in the sequence body as a
  Note over Svc: ownership check returns 404, NOT 403.
- **17 Tutee match dashboard** — Good on N+1 batch lookups. Missing: the
  ADMIN CONTACT EMAIL constant is a static REDACTED email — that's
  the only contact channel exposed and it's hardcoded. Architect's note: this
  should be a configurable property, not a constant. Diagram correctly notes
  it but should flag the architecture decision (single hardcoded admin email
  for the entire platform contact-disclosure surface).
- **18 Tutor match dashboard** — Symmetric. Same concerns as 17.
- **19 Tutee submits review** — Race handling correctly shown. Missing:
  that the notification Service emit is **inside** the transaction, AFTER
  the catch on Data Integrity Violation Exception. If the duplicate path is
  taken, no notification is sent — that's correct. But the diagram doesn't
  show that the unique-constraint guarantee is a Liquibase migration in
  changelog/ — it's an inferred database property.
- **20 Public tutor detail** — Good on permit All + parallel queries. The
  400 NOT FOUND (not 404) for missing-or-not-active is intentional
  (prevent enumeration of approved-then-closed posts) and is documented.
  **[CORRECTION INT-137b, 2026-06-13: now web request protocol 404 with the same NOT FOUND code —
  enumeration is still prevented (404 for both missing AND non-ACTIVE); the status was
  aligned to the body code repo-wide.]**
  Good.
- **21 Notification bell** — 60s polling. **Missing:** scaling discussion.
  N active users × 1 request / 60s = N/60 req/s permanent baseline load on
  /notifications/unread-count even when nothing is happening. For 10k
  users that's ~166 req/s baseline forever. No sequence diagram is the place
  for capacity planning but a one-line Note would be honest. Also:
  this endpoint is NOT rate-limited. A malicious authenticated user could
  poll it at high rate. The diagram should call that out.
- **22 Favourite toggle** — Race handling correctly shown. Optimistic
  update flow is excellent. Missing: the 201 vs 200 distinction (controller
  returns 201 on new row, 200 on idempotent hit) is documented in prose;
  worth surfacing in the diagram body because it's an architectural
  decision (informative status codes vs strict REST).
- **23 Public browse with filters** — Honest gap: per-post
  tutor Profile Repository find By User Id is an N+1 (line 305-309 of
  Tutor Post Service). This is a known smell, called out. The bitmask filter
  on availability is mentioned but the database query language is not described. The
  architecture decision (bitwise AND in the when clause vs application-side
  filter) is invisible.
- **24 Rate-limited endpoint** — Correct on Lua sliding window. **Missing:**
  the diagram does not show the IP-derivation order (CF-Connecting-IP →
  X-Forwarded-For first hop → remote Addr). This matters because if
  Cloudflare is bypassed, the limiter falls back to remote Addr which is the
  reverse proxy IP — all attackers share a single bucket. That's a real
  architectural caveat. Also no mention of forgot-password returning silent
  200 instead of 429 (this IS in failure modes — good).
- **25 cross-site request forgery protection protected write** — Good, with the right level of detail.
  **Missing:** no example of a request WITHOUT the cookie failing. The
  diagram shows the success path and one branch (header missing / cookie
  mismatch) but doesn't show the Same Site is Strict enforcement which is the
  actual primary defence — the cookie just isn't sent on a cross-origin POST.
- **26 Email fanout** — The most honest diagram in the set. Surfaces the
  no-consumer-in-tree gap and the producer-inconsistency
  (Auth Service does after Commit, Admin Post Service doesn't). This is
  the model for how the others should be written.
- **27 Encrypted personal information** — Good. **Missing:** key-rotation flow. If the
  keyed message authentication secret rotates, every existing user becomes unfindable on login
  (their stored email hmac was computed with the old secret). No diagram
  captures this operational concern. Architect's recommendation: a
  separate flow (28) for key rotation OR a Note in flow 27 listing the
  rotation strategy (e g. dual-key read, single-key write).

---

## Required additions to diagrams

### 1. Lockout write rollback bug (flow 03 / new flow 03b)

Auth Service login at lines 229-247 increments failed Login Attempts and on
the 5th failure also calls the lock until operation. It then throws
the locked operation. App Exception extends Runtime Exception
(App Exception implementation reference), and the method is a database transaction that saves all related changes together with no
no Rollback For override. Spring's default rollback rules will
**undo the lockout write**. The same applies to increment Failed Attempts.

If this code is exercised in production, the lockout is non-persistent — the
counter and lock both reset every request. Either:

- The failed Login Attempts is being modified through a @Modifying @Query
  that issues a direct update records outside the Java persistence mapping persistence context (in which
  case it survives rollback because it's a separate database query language statement) — need
  to verify with User Repository. Or:
- The code has a latent bug that the integration test suite is not catching.

**Action:** flow 03 should add a Note over Svc, database: WARNING — lock Until /
increment Failed Attempts writes are inside a database transaction that saves all related changes together and App Exception
triggers rollback. Verify @Modifying @Query semantics. This is a
must-investigate finding.

### 2. cross-site request forgery protection check on every protected write

Every flow 06–22 that POSTs/PUTs/PATCHes/DELETEs to a non-ignored path needs a
Note over Ctrl: cross-site request forgery protection check — flow 25 placed near the controller entry. As
written, only flow 06 mentions it in failure modes. A reader of flow 13 has
no clue that the interview-request POST is cross-site request forgery protection-checked.

### 3. Rate-limit tags on flows that are gated

Each rate-limited flow should explicitly tag its limit in the diagram body:

- 01: register limit 5/3600s per IP
- 02: same
- 03: login limit 10/3600s per IP, email Hmac
- 04: forgot-password limit 3/3600s per IP, email Hmac
- (new) change-password: [REDACTED CREDENTIAL]

This makes the limit visible without diving into prose.

### 4. Audit log calls in match + interview flows

Flows 09, 10, 11, 12 perform financially material state changes. Add
Svc leads to >Audit: ADMIN MATCH CREATED / ADMIN MATCH ACTIVATED /
ADMIN COMMISSION PAID / ADMIN MATCH CANCELLED IF the team decides to add
them. If the team decides NOT to add them, that's a deliberate decision and
should be documented in each flow's "Architectural decisions" section
(currently absent — see §Architectural decisions below).

### 5. Email-consumer-gap propagation

Every flow that "emits a notification but does NOT enqueue email"
(09, 10, 11, 12, 13, 14, 15, 16, 19) should add a single Note:

> Email is NOT sent for this event. User discovery is via Notification Bell
> polling (flow 21). See gap in flow 26.

Without this note, readers will assume the in-app notification implies an
email — a real expectation mismatch for users.

### 6. Transaction boundary clearer in fan-out flows

Flow 13 has the most fan-out (1 tutor + N admins, all synchronous, all in
one transaction). Add Note over Svc, NS: All emits + interview save share
one a database transaction that saves all related changes together boundary — any failure rolls back all. For 10–50 admins
this is acceptable; at 1000 admins this becomes a latency problem.

### 7. Race conditions explicit in idempotency flows

Flows 19 and 22 correctly handle Data Integrity Violation Exception. Flow 09
has a similar pre-check race
(exists By Tutor Post Id And Tutee Post Id And Status In) but does NOT have a unique
constraint backstop documented. Two admins clicking "Create match" on the
same pair simultaneously could both pass the existence check and both insert
— is there a database unique constraint on tutor post id, tutee post id, status?
If yes, show it. If no, the race is open.

### 8. Key rotation strategy in flow 27

A separate Diagram — key rotation block showing dual-secret read during
rollover, or an explicit note that rotation is operationally manual and
requires a maintenance window with re-encryption.

### 9. State machine guard error codes on flow 16

The state diagram in flow 16 doesn't show the error code on each guarded
transition. Add invalid 422 INVALID STATUS TRANSITION labels on the
transitions where the guard could fire. Same for flow 08 (post status) and
flow 12 (match status — 422 MATCH NOT CANCELLABLE etc.).

---

## Architectural decisions surfaced (good or bad)

### Good

1. **Encryption + keyed message authentication separation (flow 27).** Deterministic AEAD + keyed message authentication
   index column is the right pattern for lookup-by-personal information at rest. The empty
   AAD is a deliberate trade-off (would be stronger as user id but
   complicates lookup). Documented.
2. **after Commit for audit + email enqueue in auth (flow 01, 02, 04).**
   Idiomatic Spring, prevents stray emails on rollback. Pattern should be
   propagated.
3. **Idempotency via unique constraint + caught Data Integrity Violation Exception
   (flows 19, 22).** Correct race-safe pattern. Two-phase: pre-check for
   common path, exception catch for the race.
4. **N+1 fix in admin queue (flow 08).** Batch find All By User Id In instead
   of per-row lookup. Worth replicating to flow 23 (public browse).
5. **Defence-in-depth on admin (flow 08 onwards).** Both URL-level
   the has role operation AND class-level @Pre Authorize. If a controller
   accidentally drops one, the other catches it.
6. **cross-site request forgery protection cookie filter ordering (flow 25).** add Filter After(Csrf Cookie Filter,
   [REDACTED AUTHORIZATION] Filter) is the correct position. Documented.
7. **Lua sliding window (flow 24).** Atomic, no TOCTOU. EVALSHA caching.
   Production-grade.

### Bad / questionable

1. **Lockout writes inside rollback path (flow 03).** Already discussed. The
   single most serious finding from this review.
2. **No email consumer in the tree (flow 26).** Producer is firing
   XADD email-notifications and nothing reads it. Every email pathway is
   dark. If a consumer exists out-of-tree, this monorepo cannot be deployed
   alone — that's a missing operational contract, not just a code gap.
3. **Inconsistent after Commit discipline (flow 26 audit).**
   Auth Service forgot Password defers email; Admin Post Service approve Post
   does not. Same operation class (queue email after state change), two
   different transactional contracts.
4. **Admin loop inside transaction (flow 13).** Per-admin notification Service emit
   inside a synchronous tx. N is 1 today, doesn't scale.
5. **Hardcoded REDACTED email (flows 17, 18).** Single platform contact,
   compiled into the JAR. Cannot be changed without redeploy.
6. **No audit trail on financial state changes (flows 09–12).** Match
   create/activate/cancel/commission-paid are the audit-worthy actions in
   this system. Today they leave no audit log row. M5 is "Complete" per
   CLAUDE md status table. Compliance gap.
7. **No notification on match cancel (flow 12) but yes on interview cancel
   (flow 16).** Inconsistent user experience policy.
8. **Analytics events before commit (flows 06, 07).** Phantom events on
   rolled-back posts. Should be moved into after Commit.
9. **/notifications/unread-count not rate-limited (flow 21).** Trivial Do S
   vector for an authenticated attacker; high baseline load regardless.
10. **meeting Link not URL-sanitised (flow 15).** Admin-supplied URL flows
    to tutee dashboard. Trust boundary unstated.
11. **change-password flow not diagrammed.** It exists, it's rate-limited
    differently (5/900s vs 5/3600s for register), and the M7 Profile module
    is the next milestone — yet the spec set has no diagram for it.
12. **Key rotation flow absent (flow 27).** Encryption secret + keyed message authentication secret
    rotation is a recurring operational task (compliance requirements,
    suspected compromise). No documented procedure.

---

## Recommendation to Tech Lead

Accept this diagram set as the authoritative reference for M1–M6 with the
following must-fix additions before sign-off:

- Required additions §1 (lockout rollback investigation) — investigate code,
  amend or add diagram note.
- Required additions §5 (email-gap propagation) — one-line Note to every
  notification-emitting flow.
- Required additions §3 (rate-limit tags on gated flows) — small annotations.
- Architectural decisions §6 (audit log on match/commission flows) — make a
  deliberate decision (add OR document non-decision).

Everything else is "would improve, not blocking." Ship the set, iterate.

---

*— Mira, Solution Architect*
