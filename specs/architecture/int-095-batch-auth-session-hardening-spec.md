# INT-095 batch — Auth / session hardening (Wave D)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The INT-109 neutral duplicate-phone policy below is historical and explicitly superseded by [011 REQ-004](../011-auth-robustness/requirements.md): 409 PHONE ALREADY REGISTERED, rate-limit-before-membership, exactly one bcrypt on each phone-conflict/email-conflict/success branch, deterministic spies and preserved conflict metrics. Logout retains HttpOnly credentials and uses 011 await-then-server-revoke; no frontend cookie deletion guarantee. Other session/cache protections remain applicable.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** spec — **team-reviewed 2026-06-11 — conditions folded in** → Boss · **Date:** 2026-06-10 · **Tracker:** INT-095 (S2), INT-096 (S2),
INT-111 (S3), INT-112 (S3), INT-109 (S3·backend)
**Owners:** frontend-expert (lead) + security-expert + software-engineer (+ dba for INT-109 throttle, optional)
**Source review:** [issues/PROJECT-REVIEW-2026-06-10.md](../issues/PROJECT-REVIEW-2026-06-10.md) §S2/§S3

> **Why one spec:** four of these five defects all live on **the global axios auth interceptor**
> (api implementation reference) — the single most dangerous frontend surface, because a wrong change logs every user out
> or, worse, *fails to* log out a genuinely-expired session. They must be designed together so the
> exemption list, the backend status-code change and the stale-closure fix stay consistent and don't fight each
> other. INT-109 is the backend-only sibling (enumeration oracle) bundled because it touches the same auth files.

---

## Cross-cutting decision — what counts as a "real" 401 (read first)

Today the frontend rule is **"any non-exempt 401 ⇒ force logout + redirect to /login"**
(api implementation reference → auth Redirect should Trigger Logout, exempting only auth me, auth csrf, and the two
showcase routes). The bug is that the backend reuses **401 INVALID CREDENTIALS** for THREE semantically different
situations, and only one of them is a real session expiry:

| Call site | backend code (verified) | Status | Is the session actually invalid? |
|---|---|---|---|
| Auth Service login bad creds | INVALID CREDENTIALS | 401 (Auth Service implementation reference) | **No** — there is no session yet; the user is *trying* to get one |
| delete Account wrong confirm-password | INVALID CREDENTIALS | 401 (Auth Service implementation reference) | **No** — the bearer session is valid; only the re-typed password is wrong |
| change Password wrong current-password | INVALID CREDENTIALS | 401 (Profile Service implementation reference) | **No** — same; session valid |
| signed session token filter rejects expired/blocked token | (entry point 401) | 401 | **Yes** — the real case the logout rule exists for |

**Authoritative rule adopted by this spec (per INT-075 "wire BOTH ends"):** a 401 should mean *"your session is
gone"* and nothing else. We fix it on **both** ends, with each end authoritative for a different reason:

1. **backend is authoritative for the status code** — credential *re-confirmation* failures (delete-account,
   change-password) are **not** authentication failures of the session, so they return **422
   INVALID CREDENTIALS** (Unprocessable Entity: the request was authenticated and well-formed, the supplied
   field value was wrong). login bad-creds **stays 401** (there genuinely is no valid session) — but the frontend
   exempts the login route from the logout trigger (see INT-095/INT-111 below), because logging an
   anonymous-login-attempt "out" and clobbering its return To is the INT-111 bug.
2. **frontend is authoritative for "which 401s are not session-expiry"** — the exemption list in auth Redirect implementation reference
   is the belt-and-braces backstop so that even if a backend handler is missed (or a future endpoint reuses 401 for
   a field error), the named auth routes never force-logout.

This **belt-and-braces** posture is deliberate: the backend fix alone (422) would leave the frontend rule correct only as
long as every re-confirmation endpoint remembers to not-use-401; the frontend exemption alone would leave a 401 still
flowing through other interceptor consumers. We do both. Crucially, **neither change weakens genuine
session-expiry logout** — the signed session token-filter 401 on an expired/blocked token hits no exempt route and carries no
INVALID CREDENTIALS field semantics, so it still triggers logout exactly as today.

---

## INT-095 (S2) — Wrong password on delete-account / change-password force-logs-out the user

### Problem
A logged-in user opens "Delete account" or "Change password", mistypes their current password, and is
**instantly logged out client-side** while their server session is still fully valid. The inline error handlers
that are supposed to render "Password is incorrect" — all **FIVE** of them (Profile implementation reference,
Profile implementation reference, Force Change Password implementation reference; an earlier draft listed only 3) —
are **dead code in practice** — the global interceptor fires first.

### Current state (verified)
- backend: Auth Service delete Account throws the unauthorized operation → **401**
  (Auth Service implementation reference; helper App Exception unauthorized = Http Status UNAUTHORIZED,
  App Exception implementation reference).
- backend: Profile Service change Password throws the same 401 (Profile Service implementation reference).
- frontend: response interceptor calls the get state operation + dispatches auth:unauthorized whenever
  the should trigger logout operation is true (api implementation reference).
- frontend: should Trigger Logout returns no only for auth me, auth csrf, public showcase Tutors,
  public showcase Requests; **every other 401 ⇒ logout** (auth Redirect implementation reference, verified).
- Route constants (verified — **CORRECTED, team-review 2026-06-11**: the change-password/delete-account
  constants live in the **profile** namespace, not auth — application programming interface ROUTES auth.change Password /
  application programming interface ROUTES auth.delete Account **do not exist**): API ROUTES profile.change Password =
  [REDACTED CREDENTIAL]API ROUTES profile.delete Account = 'auth account interface operation'
  (api Routes implementation reference); application programming interface ROUTES auth.login auth login. interface operation

### Design / fix — touch BOTH ends (flagged)

**backend (authoritative status code):**
- Auth Service implementation reference — replace the unauthorized operation with
  the unprocessable entity operation
  → **422** (App Exception implementation reference already exists).
- Profile Service implementation reference — same change: App Exception unprocessable Entity("INVALID CREDENTIALS",
  "Current password is incorrect.") → **422**.
- **Do NOT touch Auth Service implementation reference (login).** Login bad-creds stays **401 INVALID CREDENTIALS** — a
  failed login is the absence of a session, 401 is semantically correct there, and the frontend-exemption (below)
  handles the "don't logout an anonymous login attempt" concern.
- **Why 422 not 400:** the request is authenticated and structurally valid; one field's *value* is rejected by
  business logic — that is exactly Unprocessable Entity, matching the project's existing use of 422 for
  business-rule field failures (TUTOR PROFILE INCOMPLETE, quota caps). 400 would imply a malformed request.

**frontend (belt-and-braces exemption):** add the two re-confirmation routes **and** the login route to the
no-logout set, so even a stray 401 on these routes never triggers a global logout.
- auth Redirect implementation reference — add a new exemption group (keep AUTH PROBE ROUTES semantically separate):


Implementation example omitted. This public edition retains the written requirements in this section.


  and in should Trigger Logout, after the existing checks:
  do not automatically retry routes that require the user to enter credentials again
  Keep the **exact-path** match (query stripped) — never a substring match — as the existing code already does.

**Reconciliation — CORRECTED (team-review 2026-06-11; the earlier claim that "a real session-expiry 401 hits
no exempt route" was FALSE):** a session CAN expire while the user is mid-flow on change-password or
delete-account — and after the 401→422 change, a 401 on /auth/change-password or /auth/account can ONLY
mean session expiry. But the exemption above suppresses the global logout on exactly those routes, and all
FIVE inline handlers map status equals 401 → "Incorrect password"/generic (Profile implementation reference,
Profile implementation reference, Force Change Password implementation reference) — i e. an expired session would be misreported as a
wrong password. **REQUIRED design change:** the shared credential-reentry hook (the INT-123e extraction) must
branch on status — **422 → inline wrong-password error; 401 → treat as session expiry (dispatch
auth:unauthorized / perform Logout)** — with a test for each branch. The route exemption keeps the *global
interceptor* from firing on these routes; the *hook* owns the 401-means-expiry handling locally.

### Test plan
- **Vitest** auth Redirect test implementation reference: assert the should trigger logout operation,
  401, auth change-password interface operation equals no, 401, auth login interface operation equals no,
  422, auth change-password interface operation equals no; and a **negative control**:
  401, matches interface operation equals yes and 401, notifications interface operation equals yes (genuine expiry still
  logs out). Add 401, auth account-foo interface operation equals yes to prove no substring leak.
- **Vitest** Profile test implementation reference (both tutee + tutor) and Force Change Password test implementation reference: mock the application programming interface to reject
  with a validation response reporting invalid credentials with status 422; assert the inline error message renders AND that
  the get state operation is still non-null (no logout) AND auth:unauthorized was NOT dispatched.
- **backend IT** (Account Lifecycle IT / change-password IT, live stack): authenticated delete-account with a wrong
  confirm Password → **422** INVALID CREDENTIALS, and the session cookie is still valid on a follow-up
  read operation for auth me (200). Same for change-password with wrong current Password. Guard against the suite-shared-IP
  429 (clear ratelimit value (withheld) before the run — SHARED-LESSONS 2026-06-03).
- **backend IT** regression: login with a wrong password still returns **401** INVALID CREDENTIALS (unchanged).
- **Vitest** shared credential-reentry hook (INT-123e extraction) — one test per branch (team-review
  2026-06-11): a **422** rejection renders the inline wrong-password error (no logout); a **401** rejection is
  treated as session expiry (auth:unauthorized dispatched / perform Logout called), never rendered as
  "Incorrect password".
- **Breaking-test inventory (verified — these existing tests assert the old 401 contract and must be
  retargeted to 422):** backend Account Lifecycle IT implementation reference +  implementation reference;
  Profile Service Test implementation reference. frontend
  Profile test implementation reference;
  Force Change Password test implementation reference. e2e: no hits.

### Acceptance
1. Mistyping the password on delete-account or change-password renders an inline error and **does not** log
   the user out; the server session stays valid.
2. The two re-confirm endpoints return **422** INVALID CREDENTIALS; login bad-creds stays **401**.
3. A genuine session-expiry 401 on any protected endpoint still triggers logout + /login redirect — and a
   401 on the exempted re-confirm routes is handled as session expiry by the shared hook (never shown as
   "Incorrect password").

---

## INT-096 (S2) — Logout never clears the Tan Stack Query cache (cross-account personal information bleed)

### Problem
On a shared device: user A logs out, user B logs in as a same-role account within stale Time (30s) → A's
cached notifications / matches / **admin user directory (real names, emails, phones)** render for B from cache
with **no refetch**; even after 30s the stale data flashes before the background refetch lands.

### Current state (verified)
- perform Logout (auth implementation reference) calls only the get state operation.
- The interceptor logout branch (api implementation reference) calls only the get state operation
  (= clear the signed-in user, auth Store implementation reference).
- grep query Client clear over src/ → **zero hits** (verified).
- No query key in query Keys implementation reference is user-scoped (tutor Keys, request Keys, match Keys, … keyed by
  resource/filters only, verified); default Options queries.stale Time 30 000 (query Client implementation reference).

### Design / fix
Clear the entire query cache on **every** logout path so no resolved query data survives a user switch.
- auth implementation reference perform Logout: import query Client and call the clear operation **after**
  the get state operation, **before** the navigate operation. Order: stop showing chrome (store logout) →
  drop cached data (the clear operation) → navigate.
- Interceptor/session-expiry path — **import-cycle constraint (team-review 2026-06-11):** api implementation reference must
  NOT import query Client directly — query Client implementation reference already imports format Api Error from api implementation reference, so
  api implementation reference closes a cycle (an earlier draft flagged the cycle only for auth Store).
  **DECIDED:** clear the cache via the existing **auth:unauthorized listener in use Init Auth implementation reference**
  (recommended) — call the clear operation there, before the navigate. (The alternative — extracting
  format Api Error to its own module so api implementation reference could import query Client — is recorded, not taken.) The
  expired-session shared-device threat stays covered: every interceptor-driven logout dispatches
  auth:unauthorized, and the listener now clears.
- **set User user-change guard (defence in depth):** in auth Store set User, when the **user id changes**
  (login as a different account without an explicit logout in between — e g. token swap), also clear. Because
  the store cannot import query Client without a cycle risk, implement this as a tiny subscription in the app
  bootstrap rather than inside the store: in use Init Auth implementation reference (or a dedicated use Auth Store subscribe in
  main implementation reference) track the previous user id; on a transition to a *different* non-null id, call
  the clear operation. Keep the store free of the Query Client dependency (avoids an import cycle
  auth Store → query Client → … → auth Store).

**Race with in-flight queries:** the clear operation removes all queries and **cancels** their retries, but a
network request already dispatched will still resolve; its result is written into a *fresh* (post-clear) cache
entry only if an observer re-subscribes. Because logout immediately navigates to / (public) and unmounts the
authenticated route tree, the observers are gone, so a late-resolving A-scoped request has no mounted consumer
to render into. To be safe and explicit, call the cancel queries operation is **not** required (clear is
sufficient post-unmount), but DO ensure the clear operation runs **before** the navigate operation in perform Logout so the
cache is empty at the moment the public tree mounts. Document this ordering in a code comment.

### Test plan
- **Vitest** auth test implementation reference: spy on query Client clear; call the perform logout operation; assert the clear operation was
  called exactly once AND after use Auth Store logout AND before navigate.
- **Vitest** api interceptor.test implementation reference + use Init Auth test implementation reference: drive a 401 on a protected route → assert
  auth:unauthorized dispatched and the use Init Auth listener calls the clear operation exactly once
  (the cycle-safe path — api implementation reference itself never imports query Client).
- **Vitest** integration-ish: seed query Client with a fake the list operation entry holding a name/email;
  call perform Logout; assert the get query data operation is undefined.
- **Vitest** user-switch guard: render the bootstrap subscriber; sign in the first test user; seed a cache entry;
  sign in a different test user; assert the clear operation fired and the entry is gone; assert set User with the **same** id
  (a /me refresh) does **not** clear (so a token refresh doesn't nuke a warm cache needlessly).

### Acceptance
1. After logout (manual or session-expiry), the get query data operation for any previously-fetched
   authenticated key returns undefined; logging in as another user shows no stale data and triggers fresh
   fetches.
2. the clear operation runs on all three paths: perform Logout, the interceptor 401 branch (via the
   auth:unauthorized listener in use Init Auth — never an api implementation reference import), and a
   different-user set User transition.

---

## INT-111 (S3) — Stale-closure location in use Init Auth builds return To from the first-mounted page

### Problem
handle Unauthorized is registered in a -dependency effect, freezing location at first render. Any later
401 (session expiry, or — today — a failed *login* POST) navigates to
/login?return To=`<the path the app first mounted on>`, not the page the user was actually on. A failed login
POST additionally **clobbers a legitimate return To** the user arrived with.

### Current state (verified)
use Init Auth implementation reference:


Implementation example omitted. This public edition retains the written requirements in this section.


location comes from the use location operation captured once; the effect deps are  (eslint-disabled, verified).

### Design / fix
- Read the **live** path inside the handler from window location, not the captured location:


Implementation example omitted. This public edition retains the written requirements in this section.


  This also drops the now-unused the use location operation import (lint will flag it — remove it).
- **Pairs with INT-095:** with auth login exempted in auth Redirect implementation reference (INT-095 frontend change), a failed login
  POST no longer dispatches auth:unauthorized at all, so handle Unauthorized won't fire on the login page in
  the first place. The on Login guard here is the second line of defence (a 401 that somehow reaches the
  handler while already on /login must not self-redirect or clobber return To). Keep both — the exemption
  prevents the dispatch; the guard prevents a bad redirect if a dispatch ever leaks.
- Use window location.search so query-bearing deep links round-trip correctly (the current code drops the
  query entirely). safe Return To validation on the consuming side (Login) is unchanged and still applies.

### Test plan
- **Vitest** use Init Auth test implementation reference: mount on route /tutee/matches?tab is active; dispatch
  auth:unauthorized; assert navigate called with
  /login?return To= REDACTED password (live path, including search), **not** the initial
  mount path.
- **Vitest**: mount on /login?return To= REDACTED password; dispatch auth:unauthorized; assert
  navigate to sign-in while replacing the current history entry with **no** return To override (the on Login guard preserves the
  existing one / does not clobber).

### Acceptance
1. A session-expiry redirect carries the **current** path+search as return To, not the first-mounted page.
2. A 401 while on /login does not self-redirect or overwrite an existing return To.

---

## INT-112 (S3) — Plaintext password lingers in session Storage on SPA-nav abandonment

### Problem
Register implementation reference stashes email, phone, password and display name (plaintext password) in session Storage
for the two-step tutor signup (Register → Commission). Commission implementation reference wipes it only on beforeunload and on
submit — **SPA navigations** (clicking the logo, About, Login) fire neither, so the credential survives in
session Storage for the rest of the tab session.

### Current state (verified)
- Stash write: Register implementation reference (Tutor Registration Stash includes password value (withheld)).
- Wipe sites in Commission implementation reference: beforeunload listener (78) and on successful submit (110) — verified;
  no route-leave cleanup.

### Design / fix — recommendation: **route-leave wipe + don't stash the password**

Two layers, recommend shipping **both**:

1. **Primary (recommended): never stash the password.** Stash only the non-secret fields
   (email, phone and display name, without the password) and **re-prompt for the password on the Commission step** (one password
   field, validated client-side, before the final register POST). This removes the plaintext-credential-at-rest
   problem entirely and is the most defensible per Personal Data Protection Act / least-data principle. The minor user experience cost (re-typing the
   password once) is acceptable for a one-time signup and is a security-positive (confirms the user remembers
   their password before account creation). This aligns with the founder soft-delete/data-minimisation posture
   and the B9 roadmap note ("only the password re-entered").
2. **Backstop (cheap, ship anyway): wipe on route-leave.** Add an effect *cleanup* that removes the stash key
   when the Commission component unmounts (covers SPA nav away), in addition to the existing beforeunload:


Implementation example omitted. This public edition retains the written requirements in this section.


   **Caveat:** the cleanup also runs on the *successful submit → navigate away* path, which is fine (the stash
   is consumed before navigation). But it must run **after** on Submit has read the stash — it does, because
   unmount happens on navigation which is post-submit. Verify ordering in the test.

> **DECIDED (team-review 2026-06-11):** adopt #1 (don't stash the password) + keep #2 as defence-in-depth
> for the remaining non-secret stash. Verified facts: the register POST happens at Commission implementation reference
> and auto-login uses that response (112-115), so re-prompting on the Commission step does **NOT** break
> auto-login; but Register implementation reference already collects the password, so this recommendation means the user
> types it twice. **Decision: accept the double-entry** — explicit trade-off: no plaintext credential at rest
> beats one extra field on one step.

### Test plan
- **Vitest** Commission test implementation reference: render Commission with a stash present; unmount the component (simulate SPA
  nav); assert the get item operation is no value.
- **Vitest** (#1 path): submit the Register tutor step; assert the written stash JSON has **no** password
  key (the expect operation); on Commission, assert a password field
  renders and the final register POST payload carries the re-entered password.
- **Vitest**: successful Commission submit still works (stash consumed before the unmount wipe).

### Acceptance
1. The plaintext password is never persisted in session Storage (recommended fix), OR at minimum is wiped the
   instant the user navigates away from the Commission step by any means (SPA link, refresh, tab close).
2. The happy-path two-step signup still completes.

---

## INT-109 (S3, backend) — Historical neutral-duplicate proposal, superseded by 011

The following problem/design/test record explains the earlier decision only. It is not an instruction to
restore generic registration errors; the effective specific-409 and deterministic-work contract is in 011.

### Problem
Registration returns **distinct** 409 codes for an existing email vs an existing phone, turning signup into an
account-membership oracle. The **phone** oracle is the sharp one: the SG mobile space is small (8 digits), so an
attacker who has someone's number can confirm InTuition membership. forgot-password is correctly silent; only
registration leaks.

### Current state (verified)
Auth Service implementation reference:


Implementation example omitted. This public edition retains the written requirements in this section.


Rate limit on registration is **5/hr/IP** (the register key operation, Auth Service implementation reference) — but IPs
rotate, so the limiter is not a sufficient control against a determined enumerator.

### Design / fix
- **Keep the email 409 (EMAIL ALREADY REGISTERED).** This is a deliberate user experience tradeoff: email collision is the
  common honest case ("you already have an account — log in / reset password"), and email is already
  semi-public; the value of a clear message outweighs the (low) marginal leak vs. a generic error. (Record this
  as an accepted residual risk.)
- **Fold the phone collision into a non-distinguishing failure.** Replace the distinct
  PHONE ALREADY REGISTERED 409 with the **same generic, non-membership-revealing failure** the form already
  shows for a generic validation problem — recommend returning **422 REGISTRATION FAILED** with a neutral
  message ("We couldn't complete registration with these details. Check your information and try again, or log
  in if you already have an account.") that is **identical whether the phone exists or not** for the phone
  branch. Critically, the response must be **indistinguishable** from the success-adjacent generic-failure
  case — same status, same code, same message, same timing class — so an attacker cannot tell "phone taken"
  from "some other rejection".
  - **Implementation note (avoid a NEW oracle):** do the phone-existence check and the generic failure in a way
    that does not leak via *timing* or via *which field the form highlights*. The simplest correct shape: keep
    the exists By Phone check, but on hit throw App Exception unprocessable Entity("REGISTRATION FAILED",
    `<neutral msg>`)with **no field attribution tophone**. Do not return afield Errors phone entry (that
    would re-create the oracle on the frontend). The frontend renders the neutral banner, not a phone-specific error.
  - **Unique-constraint backstop:** save User Or Conflict already maps the database unique violation
    (Auth Service implementation reference); ensure the phone-unique violation also maps to the **same** generic
    REGISTRATION FAILED (not a distinct code), so a TOCTOU race between the check and the insert can't leak
    the distinction either.
- **Personal Data Protection Act angle:** a phone-membership oracle is a personal-data disclosure surface — confirming that a specific
  mobile number belongs to a registered InTuition user is "personal data about an identifiable individual"
  exposed to an unauthenticated party. Closing the distinct-code oracle is a Personal Data Protection Act data-protection improvement;
  note it in the change log for the privacy reviewer (mirrors the INT-122/contact-reveal audit-trail posture).
- **Consider a per-account throttle (optional, dba-assisted):** the IP limiter is evadable; a secondary
  **per-phone** attempt counter (Redis key ratelimit:register-phone:`<normalized Phone>`, low cap e g. 3/day)
  blunts targeted enumeration even across rotating IPs. This is a *defence-in-depth* extra, not required for
  the oracle fix — flag for the team to decide (it adds a tiny write per registration attempt). Normalize the
  phone to E.164 first (INT-082 normalization) so the key is canonical.

### Test plan
- **backend IT** Registration Enumeration IT (live stack): register a user; attempt to register a **second** user
  with the **same phone, different email** → assert the response is **422 REGISTRATION FAILED** with the
  neutral message and **no field Errors phone**. Attempt with a **fresh phone but otherwise invalid** input
  that also yields REGISTRATION FAILED and assert the two responses are **byte-identical** (status + code +
  message) — the oracle is closed.
- **backend IT**: same-**email** collision still returns **409 EMAIL ALREADY REGISTERED** (unchanged user experience).
- **backend unit** (Auth Service Test): mock exists By Phone → yes; assert the thrown exception is 422
  REGISTRATION FAILED, not 409 PHONE ALREADY REGISTERED.
- **(If throttle adopted)** IT: 4th registration attempt with the same phone within the window → 429
  RATE LIMIT EXCEEDED (a neutral limiter response, still non-distinguishing about membership).
- **frontend Vitest** Register test implementation reference: a REGISTRATION FAILED 422 renders the neutral banner and does **not**
  highlight the phone field.

### Acceptance
1. A duplicate **phone** at registration returns a generic, non-distinguishing failure indistinguishable from a
   generic registration rejection (no distinct code, no phone field error).
2. A duplicate **email** still returns the clear EMAIL ALREADY REGISTERED 409 (accepted residual).
3. (If adopted) a per-phone throttle caps targeted enumeration across rotating IPs.

---

## Observability Do D (team-review 2026-06-11)

- **Server-side counter the number of registrations rejected because the phone is already in use** — INT-109's oracle-fold makes a phone
  collision externally indistinguishable by design, which would otherwise make phone collisions
  unmeasurable; the counter keeps the signal internal-only (never surfaced in the response).
- **Instrument the forced-logout path** (the interceptor/auth:unauthorized logout) so the INT-095 fix's
  impact is verifiable: the spurious-logout rate must trend to **0** after deploy.

## Owners

| Issue | Lead | Support |
|---|---|---|
| INT-095 | frontend-expert | security-expert + software-engineer (backend 422 change) |
| INT-096 | frontend-expert | security-expert (personal information bleed threat) |
| INT-111 | frontend-expert | — |
| INT-112 | frontend-expert | security-expert (credential-at-rest) |
| INT-109 | security-expert + software-engineer | dba (optional per-phone throttle) |

Shared frontend fix point (INT-095 modal duplication, INT-123e): the delete-account/change-password modal logic is
byte-identical between Profile implementation reference and Profile implementation reference. **Extract it to
one shared component as part of INT-095** so the inline-422-error handling has a single home — otherwise the
INT-095 frontend behaviour must be implemented (and tested) twice and can drift.

## Regression-risk note — the global auth interceptor is the most dangerous surface

Four of these five changes touch the **global axios auth interceptor** (api implementation reference) and its decision
helper (auth Redirect implementation reference) — the one place that decides, for *every* request in the app, whether to log the
user out. Get it wrong in one direction and **every user is force-logged-out on a benign error**; get it wrong
in the other and **a genuinely-expired session is never logged out** (a security regression that is invisible
in the happy path). Mandatory guards for this batch:

1. **Keep the exact-path (query-stripped) match** in should Trigger Logout — never substring. Add the
   auth account-foo interface operation ⇒ yes negative-control test so a future route can't silently inherit an
   exemption (the helper's own JSDoc already warns about this).
2. **Negative-control tests are mandatory, not optional:** every change that *adds* an exemption must ship a
   paired test asserting a genuine session-expiry 401 (matches interface operation, notifications interface operation) **still**
   logs out. The danger is silently broadening the exemption.
3. **backend↔frontend both-ends rule (INT-075 lesson):** the INT-095 status-code change (401→422) and the frontend exemption
   must land **together**; grep the frontend for the new exemption routes before declaring done, and verify the backend
   422 with a live IT (not just a unit test) — a code-review of one side cannot reveal an unwired other side.
4. **the clear operation must run on ALL logout paths** (manual + interceptor-401 + user-switch). A clear on
   only perform Logout leaves the session-expiry path leaking cache — exactly the shared-device threat
   INT-096 is about. Test all three paths.
5. **Don't let the INT-109 fix create a new oracle** (timing, field-attribution, or a distinct
   unique-violation code on the TOCTOU race) — the byte-identical-response IT is the load-bearing test.
6. **Live-stack ITs** (Abstract Api IT → localhost) require rebuild+redeploy using +reseed before the documented operational check
   reflects the backend 422/oracle changes; clear ratelimit:* scoped keys before a full-suite auth IT run to
   avoid the suite-shared-IP 429 flake (SHARED-LESSONS 2026-06-03).

## Rollout (after team-review → Boss)
backend (INT-095 422 + INT-109 generic failure + optional throttle + ITs) → deploy+reseed dev →
frontend (INT-095 exemption + shared modal extraction, INT-096 the clear operation on all paths, INT-111 live-path handler,
INT-112 no-stash + route-leave wipe + Vitest) → gates green (backend ≥97.5/97.5, frontend ≥97.5 line / ≥88 branch) →
live smoke (mistype password on delete-account → inline error, still logged in; logout→login-as-B → no stale
personal information; duplicate phone register → neutral failure) → e2e → push on sp-dev (per-subrepo). Prod stays
founder-gated.
