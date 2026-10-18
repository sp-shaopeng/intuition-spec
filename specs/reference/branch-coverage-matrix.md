# Branch Coverage Matrix — service business logic vs regression lab

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-05-31 · **Owner:** Tech Lead + Docs + Regression Eng
**Source of truth:** the service code (intuition-the private implementation folder).
This maps **every business branch** (each thrown error code + each status transition, by source
line) to whether the **regression lab** exercises it. A branch with no test is BOTH a doc gap and a
coverage gap.

**Method:** extracted every App Exception.* CODE / an application error carrying the required web response status throw site and
every the set status operation transition from the 14 services — **117 throw sites + 19 status transitions =
136 business branches** (verified by grep over service/, all ERR * constants resolved to their
string codes). The lab asserts by web request protocol status + scenario (it references only 2 error codes by name),
so "tested" is mapped per scenario to the owning IT method.

**Legend:** ✓ = a regression IT exercises this branch · ⚠️ = exercised only indirectly (happy-path
precursor, not asserted as its own case) · ✗ = **GAP** (no test) · 🚫 = excluded by design (documented).

---

## Admin Commission Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 50 | COMMISSION NOT FOUND | 404 | pay unknown id | ✓ Admin Commission IT pay Non Existent Commission is404 |
| 57 | COMMISSION ALREADY PAID | 409 | pay an already-PAID commission | ✓ Admin Commission IT double Pay second Is Conflict |
| 62 | → match COMPLETED | — | pay activates completion | ✓ Admin Commission IT pay Commission + Review Submit Journey IT (relies on it) |

## Admin Guard Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 42 | UNAUTHORIZED | 401 | no/ә invalid admin principal | ✓ Admin Authz Matrix IT all Admin Reads anonymous are401 |
| 49 | PASSWORD CHANGE REQUIRED | 403 | gated admin attempts a mutation | ✓ Admin Bootstrap IT new Admin is Gate Blocked On Mutation… |

## Admin Match Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 96 | TUTOR POST NOT FOUND | 404 | create match, bad tutor Post Id | ✓ Admin Matching IT create Match unknown Tutor Post is404 |
| 104 | TUTOR POST NOT ACTIVE | 422 | tutor post not ACTIVE | ✓ Admin Matching IT match with Pending Tutor Post is422 |
| 112 | TUTEE POST NOT FOUND | 404 | create match, bad tutee Post Id | ✓ Admin Matching IT create Match unknown Tutee Post is404 |
| 120 | TUTEE POST NOT ACTIVE | 422 | tutee post not ACTIVE | ✓ Admin Matching IT match with Pending Tutee Post… |
| 139 | DUPLICATE MATCH | 409 | pair already matched | ✓ Admin Matching IT duplicate Match for Same Pair is409 |
| 211 | MATCH NOT FOUND | 404 | activate unknown match | ✓ Admin Matching IT activate Unknown Match is404 |
| 215 | MATCH NOT PENDING | 422 | activate non-PENDING | ✓ Admin Matching IT double Activate second Is422 |
| 263 | MATCH NOT FOUND | 404 | cancel unknown match | ✓ Admin Matching IT cancel Unknown Match is404 |
| 267 | MATCH NOT CANCELLABLE | 422 | cancel a COMPLETED/CANCELLED match | ✓ Admin Matching IT cancel Completed Match… |
| 225 | → ACTIVE | — | activate | ✓ Admin Matching IT match create activate cancel |
| 269 | → CANCELLED | — | cancel | ✓ same + cancel Match cascades Commission To Cancelled |

## Admin Post Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 137/180 | ALREADY REVIEWED (tutor post) | 409 | approve/reject a non-PENDING tutor post | ✓ Admin Post Approval IT double Approve second Is Conflict |
| 153/196 | ALREADY REVIEWED (tutee post) | 409 | approve/reject a non-PENDING tutee post | ✓ Admin Post Approval IT double Approve Tutee Post… |
| 165/208 | NOT FOUND | 404 | approve/reject unknown id | ✓ Admin Post Approval IT approve non Existent Post is404 |
| 139/157 | → ACTIVE | — | approve | ✓ pending Post appears In Queue then Approve… |
| 182/201 | → REJECTED | — | reject | ✓ reject Pending with Reason succeeds |

## Admin User Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 104 | EMAIL ALREADY REGISTERED | 409 | create admin, dup email | ✓ Admin Bootstrap IT create Admin duplicate Email is409 |
| 151 | NOT SUPER ADMIN | 403 | non-super deletes | ✓ Admin User Mgmt IT non Super Admin cannot Delete… |
| 155 | CANNOT REMOVE SELF | 403 | super deletes self | ✓ Admin User Mgmt IT super Admin cannot Delete Self |
| 160 | ADMIN NOT FOUND | 404 | super deletes unknown | ✓ Admin User Mgmt IT super Admin delete Non Existent is404 |
| 165 | CANNOT REMOVE SUPER ADMIN | 403 | delete a super-admin target | ✓ Admin User Mgmt IT super Admin cannot Delete Another Super Admin is403 target Untouched |
| 174 | RATE LIMIT EXCEEDED | 429 | create-admin rate limit | 🚫 rate-limit excluded in dev lab |

## Auth Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 96/148/486 | RATE LIMIT EXCEEDED | 429 | register/forgot rate limit | 🚫 excluded (limiter raised in dev) |
| 101/153 | EMAIL ALREADY REGISTERED | 409 | register dup email | ✓ Auth Journey IT register duplicate Email is409 and Does Not Create Second Account |
| 106/158 | PHONE ALREADY REGISTERED | 409 | register dup phone | ✓ Auth Journey IT register duplicate Phone is409 and Does Not Create Second Account |
| 144/619 | COMMISSION ACK REQUIRED | 400 | tutor register without ack | ✓ Auth Journey IT register Tutor without Commission Ack is400 no Account Created (promise asserted at 400; wire code is VALIDATION ERROR — the data transfer object @Assert yes shadows the service COMMISSION ACK REQUIRED branch, which is unreachable via web request protocol) |
| 223/243 | ACCOUNT LOCKED | 423 | login while locked / lock on Nth fail | ✓ Password Reset Journey IT reset clears Account Lockout (asserts 423) |
| 250/288 | INVALID CREDENTIALS | 401 | wrong password | ✓ Auth Journey IT login wrong Password is401 |
| 305 | USER NOT FOUND (login) | 404 | — | ⚠️ login path returns INVALID CREDENTIALS to client; 404 internal |
| 314/566 | ACCOUNT DELETION REQUESTED | 401 | login/delete on a deleted acct | ✓ Account Lifecycle IT delete Account then Login Denied |
| 340 | INVALID MFA SESSION | 401 | bad multi-factor authentication session token | 🚫 multi-factor authentication deferred (#03) |
| 348 | MFA ATTEMPTS EXCEEDED | 429 | too many time-based one-time passcode tries | 🚫 multi-factor authentication deferred |
| 363 | MFA NOT CONFIGURED | 400 | multi-factor authentication step, no secret | 🚫 multi-factor authentication deferred |
| 372 | INVALID time-based one-time passcode | 401 | wrong time-based one-time passcode | 🚫 multi-factor authentication deferred |
| 524/529/540 | INVALID OTP | 400 | reset, wrong/expired one-time passcode | ✓ Password Reset Journey IT wrong Otp… + otp is Single Use |
| 591 | OTP ATTEMPTS EXCEEDED | 429 | reset, too many one-time passcode tries | ✓ **INT-088 FIXED** (Auth Service reset Password value (withheld)). The cap is now read from a Redis counter otp-attempts:`<token Id>` incremented OUTSIDE the rolled-back tx (the wrong-one-time passcode branch at 600 does the increment operation then throw INVALID one-time passcode at 605; the Redis bump survives the rollback, unlike the database increment Attempt Count which still rolls back). The cap guard (591) is now reachable. Proof: Password Reset Journey IT reset exceeding Otp Attempt Cap is429 and Password Unchanged (@Disabled removed, green — 5 wrong then correct one-time passcode → 429, password unchanged) + units tests for excessive reset attempts and wrong-passcode attempt tracking. (Line refs shift +interim-rate-limit block; pre-fix the cap was the unreachable 562.) |
| 498 | REGRESSION EMAIL REQUIRED | 400 | local one-time passcode fixture, non-band email | ✓ Password Reset Journey IT issue Otp Fixture refuses… |
| 562 | INVALID CREDENTIALS (delete acct) | 401 | delete account, wrong password | ✓ Account Lifecycle IT delete Account wrong Password… |

## Favourite Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 64 | TUTOR POST NOT FOUND | 404 | favourite unknown post | ✓ Favourites IT favourite Unknown Tutor Post is404 |
| 72 | TUTOR POST NOT ACTIVE | 422 | favourite non-ACTIVE post | ✓ Guard Negatives IT favourite Non Active Post… |
| 84 | TUTEE NOT FOUND | 404 | — | ✗ **GAP — Tranche C deferred (defensive/unreachable):** the the find by id operation after the post lookup cannot fail for an authenticated TUTEE (every registered tutee has a User row + the endpoint is TUTEE-gated), so this branch is unreachable through the black-box harness. Not a closeable GAP. |
| — | add/list/remove happy path | 200/201/204 | round-trip | ✓ Favourites IT tutee favourite Round Trip… |

## Interview Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 99 | TUTEE POST NOT OWNED | 403 | request with someone else's tutee post | ✓ Interviews IT propose with Someone Elses Tutee Post is403 not Owned |
| 108 | TUTEE POST NOT ACTIVE | 422 | request, tutee post not ACTIVE | ✓ Interviews IT propose with Own Pending Tutee Post is422 not Active |
| 139 | TUTOR POST NOT FOUND | 404 | request, bad tutor post | ✓ Interviews IT propose with Unknown Tutor Post is404 |
| 147 | TUTOR POST NOT ACTIVE | 422 | request, tutor post not ACTIVE | ✓ Interviews IT propose with Pending Tutor Post is422 tutor Not Active |
| 160 | DUPLICATE INTERVIEW | 409 | duplicate request same pair | ✓ Interview Limit IT duplicate Interview same Pair… |
| 172 | **INTERVIEW LIMIT REACHED** | 422 | >3 active interviews per posting (M6 cap) | ✓ Interview Limit IT fourth Active Interview… |
| 191/225/263 | INTERVIEW NOT FOUND | 404 | get/cancel/admin-op unknown | ✓ Admin Interview Ops IT schedule non Existent is404 (partial) |
| 217 | tutee cancel invalid status | 422 | cancel terminal (CANCELLED) | ✓ Interviews IT tutee Cancel on Terminal Interview is422 |
| 283/289 | schedule invalid status | 422 | schedule non-REQUESTED | ⚠️ implied by lifecycle, not a negative |
| 318 | meeting-link invalid status | 422 | link a non-SCHEDULED | ✓ Interviews IT admin Set Meeting Link on Proposed Interview is422 |
| 340 | complete invalid status | 422 | complete non-SCHEDULED | ✓ Interviews IT admin Complete on Proposed Interview is422 |
| 411/417/435 | no-show invalid status / actor | 422 | no-show non-SCHEDULED | ✓ Admin Interview Ops IT no Show on Unscheduled Interview is422 |
| 146/213/285/333/369/434 | → REQUESTED/CANCELLED/SCHEDULED/COMPLETED/NO SHOW | — | lifecycle | ✓ Admin Interview Ops IT (schedule/complete/no-show/cancel) + 35-tutee-cancel |

## Job Application Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 96 | TUTEE POST NOT FOUND | 404 | apply to unknown job | ✓ Guard Negatives IT apply To Unknown Job is404 |
| 103 | TUTEE POST NOT ACTIVE | 422 | apply to non-ACTIVE job | ✓ Guard Negatives IT apply To Non Active Job… |
| — | ~~TUTOR NO ACTIVE POST~~ (guard removed) | n/a | — | 🚫 **GUARD REPLACED (M18)** — the active-listing guard was deleted (Job Application Service implementation reference) in favour of a profile-completeness check; this branch no longer exists in the SUT, so it is **not a closeable GAP**. |
| 139 | TUTOR POST INVALID | 400 | pinned tutor post not found | ✓ Guard Negatives IT apply With Unknown Tutor Post is400 tutor Post Invalid |
| 163 | APPLICATION NOT FOUND | 404 | withdraw unknown/not-owned | ✓ Guard Negatives IT withdraw Unknown Application is404 |
| — | apply/withdraw + reactivate | 200 | happy + idempotent + re-apply | ✓ Applications IT tutor apply To Job then Appears then Withdraw |

## Profile Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 146 | INVALID CREDENTIALS | 401 | change-password, wrong current | ✓ Account Lifecycle IT change Password wrong Current… |
| 151 | PROFILE NOT FOUND | 404 | — | ✗ **GAP — Tranche C deferred (defensive/unreachable):** every registered tutor/tutee gets its matching profile row at registration, and the profile endpoints are role-gated, so an own-profile lookup can never miss through the black-box harness. Not a closeable GAP. |
| — | cross-role profile access | 403 | tutor↔tutee namespace | ✓ a test that prevents tutors from accessing the parent-only profile area |
| — | featured-consent set | 200 | toggle | ⚠️ asserted via Showcase IT outcome, not the PATCH directly |

## Review Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| 78 | MATCH NOT FOUND | 404 | review unknown match | ✓ Review Submit Journey IT review on Unknown Match is404 |
| 84 | INVALID STATUS | 422 | review a non-COMPLETED match | ✓ Review Submit Journey IT review on Non Completed Match… |
| 103 | NOT PARTICIPANT | 403 | non-participant reviews | ✓ Review Submit Journey IT review by Non Participant… |
| 108/129 | DUPLICATE REVIEW | 422 | same reviewer twice | ✓ Review Submit Journey IT review duplicate By Same Reviewer… |
| — | submit (tutee→tutor) | 201 | happy | ✓ Review Submit Journey IT tutee Participant submits Review… |
| — | submit (tutor→tutee) | 201 | the other reviewer path | ✓ Review Submit Journey IT review by Tutor Participant… |

## Tutee Post Service / Tutor Post Service
| Line | Branch | Status | Trigger | Lab |
|---|---|---|---|---|
| Tutee 115 / Tutor 147 | FORBIDDEN | 403 | edit/delete not-owner | ✓ Post Lifecycle IT delete Post by Non Owner… |
| Tutee 143 | INVALID STATE | 409 | delete a non-DRAFT/REJECTED post | ✓ Post Lifecycle IT delete Active Post by Owner… |
| Tutee Post 39 / Tutor Post 60 | create → PENDING | 201 | create+submit | ⚠️ precursor in admin chains, not asserted as own contract |
| 127/159 and 143/179 | PENDING or CLOSED | — | create / soft-delete | ✓ Post Lifecycle IT edit Active Post… / delete Rejected Post by Owner… |
| Tutor Post 165 | set Availability | 200 | availability toggle | ✓ Post Lifecycle IT tutor Availability Toggle by Owner flips Both Ways non Owner Forbidden (also surfaced + fixed **INT-089**: every toggle 500'd on a Lazy Initialization Exception serializing the un-warmed @Element Collection sets — set Availability now warms them in-tx, mirroring find Own Post) |

---

## Summary

**Single authoritative ledger = the per-service tables above** (reconciled 2026-06-10). The marks in
those tables are now the live state; the "CLOSED this pass" block below is retained only as the
**changelog** of which IT closed each row, not as a competing count. Counting the **mapped table rows**:

| | count (reconciled) |
|---|---|
| Mapped table rows (scenario granularity) | **81** |
| ✓ Tested (asserted by an IT) | **68** (INT-088 one-time passcode cap flipped ✗→✓) |
| ⚠️ Indirect only (happy-path precursor, not asserted) | **4** |
| 🚫 Excluded by design (4 multi-factor authentication + 2 rate-limit + 1 guard-removed) | **7** |
| ✗ **GAP (open)** | **2** (both defensive/unreachable — INT-088 one-time passcode cap is now FIXED + tested, see below) |

### Phase R1 closure (2026-06-10, software-engineer Jordan)

The 24 open ✗ GAP rows were worked in TIER order. **21 closed ✗→✓** with one negative-path/status-transition
regression IT each, into the existing flow IT classes (no new harness). INT-088 (the one-time passcode cap, below) was
subsequently **fixed and its test enabled** (Bundle A C1/C6), so **22 are now ✓**; the remaining **2 stay ✗
honestly** (both unreachable defensive 404s):

- **OTP ATTEMPTS EXCEEDED 429 (Auth Service) — INT-088 FIXED (Bundle A C1).** Was a live account-takeover:
  the brute-force cap was DEAD because the reset password operation is a database transaction that saves all related changes together and the wrong-one-time passcode branch did
  the increment attempt count operation **then** throw INVALID one-time passcode, so the database increment rolled back with the tx →
  attempt count never accumulated → the cap guard was unreachable. **Fix:** the attempt counter now
  lives in Redis (otp-attempts:`<token Id>`), incremented OUTSIDE the rolled-back tx at 600 (it survives
  the throw, mirroring the multi-factor authentication counter and the login-lockout precedent), so the is at least one-time passcode MAX ATTEMPTS cap (591)
  fires.
  **Proof:** Password Reset Journey IT reset exceeding Otp Attempt Cap is429 and Password Unchanged (@Disabled
  removed → green) + tests for excessive reset attempts and wrong-passcode attempt tracking.
- **TUTEE NOT FOUND 404 (Favourite Service) — Tranche C deferred (unreachable):** the find by id operation
  can't miss for an authenticated TUTEE on a TUTEE-gated endpoint.
- **PROFILE NOT FOUND 404 (Profile Service) — Tranche C deferred (unreachable):** every registered user
  has its matching profile row; an own-profile lookup never misses via the role-gated endpoints.

**Bonus source bug fixed in-pass: INT-089 (Sev-2).** Closing sequence #34 (set Availability) revealed every
tutor availability toggle returned **web request protocol 500** — a Lazy Initialization Exception while Jackson serialized the
un-warmed @Element Collection sets after the session closed (the M17 detail-path Lazy Init class, recurring).
Fixed by warming the lazy sets in-tx (mirroring find Own Post); the feature was completely broken and untested,
which is exactly why the matrix flagged it as a GAP. Rebuilt + redeployed the live stack before re-verifying.

> **R1 KPI met:** open ✗ on money / personal information / authz → the Tier-1 security/authz guards
> (EMAIL/PHONE ALREADY REGISTERED, COMMISSION ACK REQUIRED promise, CANNOT REMOVE SUPER ADMIN, interview
> ownership/active guards) are all CLOSED. The former remaining security ✗ (one-time passcode cap, INT-088) has since been
> **fixed and its test enabled** (Bundle A) — there are now **no open security ✗ rows**; the only 2 remaining
> ✗ are unreachable defensive 404s.

> **Two granularities, stated once to avoid a third floating number.** The header counts **136 business
> branches** at code granularity (117 throws + 19 transitions); the tables map those to **81 scenario
> rows** (some rows bundle a * NOT FOUND/* NOT ACTIVE twin pair). The **authoritative open-GAP figure
> is 24 ✗ closeable rows** — this supersedes the earlier "~62", which double-counted rows the closure pass
> had already moved to ✓ in the changelog but not yet in the tables. The 24 are mostly lower-value
> symmetric * NOT FOUND 404 / * NOT ACTIVE 422 twins plus a handful of high-value guards
> (COMMISSION ACK REQUIRED, CANNOT REMOVE SUPER ADMIN, interview-request ownership/active guards);
> multi-factor authentication + rate-limit + the M18-removed guard stay 🚫 (not closeable).

### CLOSED this pass (branch → owning IT, all green ×3)

| Branch | web request protocol | Owning IT |
|---|---|---|
| INTERVIEW LIMIT REACHED | 422 | Interview Limit IT fourth Active Interview… |
| DUPLICATE INTERVIEW | 422 | Interview Limit IT duplicate Interview same Pair… |
| review INVALID STATUS (non-COMPLETED) | 422 | Review Submit Journey IT review on Non Completed Match… |
| DUPLICATE REVIEW | 422 | Review Submit Journey IT review duplicate By Same Reviewer… |
| tutor-as-reviewer (other participant) | 201 | Review Submit Journey IT review by Tutor Participant… |
| match TUTEE POST NOT ACTIVE | 422 | Admin Matching IT match with Pending Tutee Post… |
| MATCH NOT CANCELLABLE (COMPLETED) | 422 | Admin Matching IT cancel Completed Match… |
| post non-owner delete FORBIDDEN | 403 | Post Lifecycle IT delete Post by Non Owner… |
| post delete-ACTIVE INVALID STATE | 409 | Post Lifecycle IT delete Active Post by Owner… |
| post edit-ACTIVE happy + edit-PENDING lock | 200 / 409 | Post Lifecycle IT edit Active Post… / edit Pending Post…409 |
| post delete-REJECTED happy | 204 | Post Lifecycle IT delete Rejected Post by Owner… |
| change-password wrong-current INVALID CREDENTIALS | 401 | Account Lifecycle IT change Password wrong Current… |
| change-password happy + old-rejected | 200/401 | Account Lifecycle IT change Password happy… |
| account deletion + login-denied after | 200/401 | Account Lifecycle IT delete Account then Login Denied |
| delete-account wrong password | 401 | Account Lifecycle IT delete Account wrong Password… |
| tutee-post ALREADY REVIEWED (twin) | 409 | Admin Post Approval IT double Approve Tutee Post… |
| apply TUTEE POST NOT ACTIVE | 422 | Guard Negatives IT apply To Non Active Job… |
| favourite TUTOR POST NOT ACTIVE | 422 | Guard Negatives IT favourite Non Active Post… |

> **Retired (do not reinstate):** the former apply TUTOR NO ACTIVE POST (403) row and its
> Guard Negatives IT apply Without Active Listing… test were removed when **M18** deleted the active-listing
> guard (Job Application Service implementation reference, replaced by a profile-completeness check). The branch no longer
> exists in the SUT, so it is reclassified 🚫 in the Job Application Service table — **not** a closeable GAP.

This block is the **changelog** of what the branch-closure pass moved ✗→✓; as of the 2026-06-10
reconciliation the per-service tables above carry the live ✓/⚠️/✗/🚫 marks (no longer double-booked) and
are the single source of truth. A row here must also read ✓ in its table.

## The 24 open ✗ GAP rows — tiered by class (close order, highest consequence first)

The earlier high-value tranche (INTERVIEW LIMIT REACHED, review integrity, match-cancellable, post
ownership, account-recovery) is **CLOSED** — see the changelog block above. What remains, tagged by class:

**Tier 1 — security / authz / money / account-integrity (close first, NOT trivial 404s):**
1. COMMISSION ACK REQUIRED 400 (Auth Service/619) — tutor registers without the commission ack; a
   money/legal-consent guard with zero test.
2. CANNOT REMOVE SUPER ADMIN 403 (Admin User Service) — privilege-escalation guard, untested.
3. EMAIL ALREADY REGISTERED 409 / PHONE ALREADY REGISTERED 409 (Auth Service/106) — dup-identity
   account-integrity guards (register is only ever used as a precursor today).
4. one-time passcode ATTEMPTS EXCEEDED 429 (Auth Service) — reset-flow brute-force throttle. **CLOSED (INT-088 fix,
   Bundle A): Redis attempt counter outside the rolled-back tx; test enabled + green.**
5. Interview-request ownership/active guards — TUTEE POST NOT OWNED 403, TUTEE POST NOT ACTIVE 422,
   TUTOR POST NOT ACTIVE 422 (Interview Service/108/147) — authz + state on the booking entry point.

**Tier 2 — genuinely uncovered sequence (no IT today):**
6. Tutor Post Service set Availability 200 — sequence **#34 tutor-availability-toggle** has no test at all.

**Tier 3 — lower-value symmetric * NOT FOUND / * NOT ACTIVE / invalid-status twins** (one side already
tested, the mirror not):
- match: TUTOR POST NOT FOUND 404 (96), TUTEE POST NOT FOUND 404 (112), MATCH NOT FOUND 404 on
  activate (211) + cancel (263).
- interview: TUTOR POST NOT FOUND 404 (139); invalid-status cancel/meeting-link/complete 422
  (217/318/340).
- apply: TUTEE POST NOT FOUND 404 (96), TUTOR POST INVALID 400 (139), APPLICATION NOT FOUND 404
  (163).
- favourite: TUTOR POST NOT FOUND 404 (64), TUTEE NOT FOUND 404 (84).
- profile: PROFILE NOT FOUND 404 (151).
- review: MATCH NOT FOUND 404 (78).

**Excluded by design (🚫, NOT counted as closeable):** all multi-factor authentication branches (#03 deferred), all rate-limit 429s
(limiter raised in the dev lab), and the M18-removed TUTOR NO ACTIVE POST guard.

**The honest headline:** the lab covers the **high-consequence happy paths, the primary
double-submit/auth negatives, and (post-closure) the documented business-rule guards** well. The **24 open
✗** are 5 Tier-1 security/authz/money guards + 1 genuinely-uncovered sequence (#34) + ~18 lower-value
symmetric * NOT FOUND/invalid-status twins. The Tier-1 set is the R1 success KPI: **open ✗ on
money/personal information/authz → 0.**

This matrix is the canonical branch inventory; each ✗ is a candidate regression test. See
architecture/regression-labs-spec md §13 for the flow-level coverage map.
