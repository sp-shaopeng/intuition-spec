# Fix Spec — Issues INT-001 … INT-005

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** SPEC — pending team review
**Date:** 2026-06-01
**Author:** Engineering
**Tracker:** [README implementation reference](../issues/README.md)
**Governance note:** Per founder — an issue is OPEN until the **founder** verifies it. Engineering
"FIXED-DEV" is a claim, not closure. INT-002 is already implemented + dev-verified but remains
OPEN until founder sign-off; it is included here for completeness and re-review.

---

## Scope

| Issue | Sev | One-line fix |
|---|---|---|
| INT-001 | S2 | Sign-up reachability — verify (likely an INT-002 symptom); no source dead-end found |
| INT-002 | S1 | signed session token filter: invalid/stale token → anonymous, never hard-fail the chain (DONE-DEV) |
| INT-003 | S2 | Drop tutor real name from the unauthenticated /public/tutors data transfer object + personal information contract test |
| INT-004 | S2 | Global error→toast fallback for unhandled 4xx/5xx (Tan Stack Query cache handlers) |
| INT-005 | S3 | Kill Java persistence mapping/database query language log noise (config) + consistent business logging (who/what/why) |

---

## INT-001 — Sign-up reachability

**Finding:** the sign-up path is wired correctly in source:
- Nav implementation reference logged-out cluster: desktop **"Sign up" → /register** (primary), mobile
  **"Register" → /register**.
- Login implementation reference "New to InTuition?" → **Create parent/tutor account** → /register?role is tutee|tutor.
- /register route exists (App implementation reference); Register implementation reference reads ?role and pre-selects.

The reported "sign up required me to sign in first" matches the **INT-002** stale-cookie 401
(register/login 401'd before the handler ran), making sign-up *appear* broken.

**Action:**
1. No code change to the sign-up wiring (it is correct).
2. **Verify** after INT-002: from a browser with a stale auth cookie, "Sign up" / "New to
   InTuition?" now reaches /register and registration succeeds (200/201).
3. If a genuine dead-end is found during verification, file a follow-up with the exact surface.

**Acceptance:** founder reaches /register from both Nav "Sign up" and Login "New to InTuition?"
on the running stack, and can complete registration.

---

## INT-002 — Stale/invalid auth cookie → 401 on public endpoints (DONE-DEV, re-review)

**Change (implemented):** Jwt Authentication Filter no longer calls the send unauthorized operation on a
token problem. On invalid/expired token, missing admin session, or revoked jti, it now
**clears the security context and continues the chain as anonymous** (the continue anonymous operation).
Authorization is enforced downstream by the any request operation + unauthorized Entry Point
(same 401 JSON). send Unauthorized and the now-unused Http Status import are removed.

**Why this is correct & safe:** standard stateless-signed session token filter contract — authenticate-if-valid,
else anonymous; never hard-fail. Public endpoints proceed; protected endpoints still 401.

**Tests (added, green):** Jwt Authentication Filter Test — 3 new cases on a PUBLIC endpoint
(/public/tutors) with (a) invalid signed session token, (b) admin session-gone, (c) revoked tutee → all **200**.
Existing protected-endpoint 401 cases unchanged (still 401). 12/12 pass.

**Live verification (dev):** forgot-password + stale cookie → 200; login + stale cookie + bad
creds → 401 INVALID CREDENTIALS (handler ran); /auth/me + stale cookie → 401 (still protected).

**Prod-impact:** real auth bug (not dev-only) → on founder sign-off, also add to the ROADMAP
**PROD-DEPLOY-OWED** register (plain app deploy, no migration).

---

## INT-003 — /public/tutors leaks tutor real name

**Problem:** read operation for tutors (unauthenticated) returns tutor Display Name (real name,
e g. "Joanne Chua") in each row, violating the M14a/M18 "public card = handle only" personal information decision.
The handle (tutor Public Handle) is already present in the data transfer object.

**Change:**
1. Remove tutor Display Name from the public browse projection Tutor Post Response (the data transfer object served
   by Public Browse Controller GET /public/tutors). If the same data transfer object is reused by an authenticated
   surface that legitimately needs the name, split it — the **public** projection must not carry
   the real name. (Verify frontend browse card leads with tutor Public Handle (M18 §B1) and does not read
   tutor Display Name; remove any such read.)
2. Add a **personal information contract test** mirroring Showcase Tutor Response Pii Contract Test: reflect over the
   public tutor browse data transfer object's fields; assert an allow-list (handle, subjects, levels, rate, coarse
   area, YOE, commitment, rating, …) and a deny-list (display Name, tutor Display Name, name,
   email*, phone*, contact, address). Fails the build if the real name re-enters.

**Acceptance:** the documented operational check (no auth) returns rows with tutor Public Handle and
**no** tutor Display Name/real name; personal information contract test green; frontend browse still renders cards
(handle-led). No frontend regression in the documented operational check / tutor browse tests.

---

## INT-004 — Silent 4xx/5xx (no global error toast)

**Problem:** query Client implementation reference global mutations on Error only console errors; there is no
queries on Error and no Query Cache/Mutation Cache handler. Any query/mutation without its own
on Error toast fails silently.

**Change (query Client implementation reference):** construct the Query Client with
shared handling of failures when reading server data and shared handling of failures when submitting changes.
The shared handler is a **fallback**, not a duplicate:


Implementation example omitted. This public edition retains the written requirements in this section.


- Keep existing tailored on Error toasts at call sites (they still fire; the global one only
  covers the unhandled rest — for mutations, set meta suppress Error Toast on call sites that
  already toast, to avoid double toasts; document this convention).
- Forms that render field errors inline (login/register 400/422) set
  suppress the shared error message when the screen already handles it.

**Tests (Vitest):** unhandled mutation error → global toast shown; meta suppress Error Toast →
no global toast; 401 → no toast. Plus a render test that the toast text uses the server message.

**Acceptance:** a forced 4xx/5xx on any unhandled call shows a toast with the server message
(fallback "Something went wrong — please try again"); no double-toast on already-handled errors;
401 stays silent (redirect path). the documented operational check + the documented operational check green.

---

## INT-005 — Log noise + business logging

### Part A — kill Java persistence mapping/database query language noise (config, immediate)

application-local implementation reference currently:


The illustration for part a — kill java persistence mapping/database query language noise config, immediate contains the following information:

- logging level com intuition is DEBUG.
- logging level org hibernate database query language is DEBUG ← remove.
- logging level org hibernate type descriptor implementation reference Basic Binder is TRACE ← remove.


**Change:** remove the two Hibernate lines from the default local profile; keep
com intuition is DEBUG (so business logs surface). For the rare need to see database query language, add an opt-in
application-debug-sql implementation reference (activated via SPRING PROFILES ACTIVE is local,debug-sql) that
re-enables org hibernate.database query language is DEBUG. Confirm the **prod** profile logs neither database query language nor binds
(volume + personal information in bind params).

### Part B — consistent business logging (who/what/why)

Pattern (login already does this): domain action.outcome key is val …, IDs/roles only (never raw
email/phone/token), with the request reference included in logs already in the console pattern.
- **Already present:** Auth Service login (auth login.failed reason=…, auth login.success user Id=…).
- **Add the same pattern at these service boundaries:**
  - Auth Service: registration outcomes by tutee or tutor role, including failure reasons, auth forgot Password requested,
    password-reset outcomes including failure reasons.
  - Admin Post Service: post approval and rejection outcomes with the acting administrator and post identifiers.
  - Admin Match Service: match creation, assignment, activation and cancellation outcomes with failure reasons and the relevant actor, request and tutor-listing identifiers (assign includes the supersede outcome).
  - Admin Commission Service: commission-payment recording outcomes with actor and commission identifiers.
  - Interview Service: interview request, scheduling, no-show, cancellation and completion outcomes with failure reasons.
  - Tutor Selection Service: tutor-selection creation, update and failure outcomes.
- Level: info for success, warn for business failures (denials/conflicts), error only for
  unexpected exceptions.
- This is the **runtime log** surface (operator visibility), distinct from the persistent
  audit log table — both are kept.

**Acceptance:** with database query language noise off, a login attempt + an admin assign produce readable
domain action.outcome … correlation Id=… lines and no raw database query language; a failed action logs a reason;
no personal information (email/phone/token) in any log line. Backend tests stay green; coverage gate holds.

---

## Sequencing & deploy

1. INT-005 Part A (config) — trivial, do first so the rest is observable.
2. INT-003 (backend data transfer object + personal information test), INT-005 Part B (backend business logs) — backend, one rebuild.
3. INT-004 (frontend global toast + tests) — frontend.
4. INT-002 — already in; re-verified.
5. INT-001 — verification only.
6. the documented operational check (backend) green + the documented operational check (frontend, Node 20) green + the documented operational check.
7. the documented operational check; live smoke: stale-cookie sign-up works (INT-002/001),
   /public/tutors nameless (INT-003), forced error toasts (INT-004), clean business logs (INT-005).
8. On founder verification, move prod-affecting items (INT-002, and INT-003 if deemed prod-bug)
   to the ROADMAP PROD-DEPLOY-OWED register.

## Coverage / gates
backend JaCoCo ≥97.5% line+branch; frontend Vitest ≥97.5% line / ≥88% branch; the documented operational check (tsc) green;
ESLint + Spotless clean. New tests: INT-002 filter (done), INT-003 personal information contract, INT-004 toast cases.
