# Tech Lead Verification — Sequence Diagrams

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Reviewer: Tech Lead
Date: 2026-05-29
Scope: All 27 sequence diagrams under /intuition-spec/specs/sequences/

## Verdict: APPROVED WITH CHANGES

All 27 diagrams are materially accurate. They correctly describe controller/service control flow, transactional boundaries, failure modes, state machines, batch lookups, and the four documented gaps (Login multi-factor authentication wiring, post-creation admin notify, cancel-match notify, email consumer). Cited file paths are correct. Cited line numbers are right within ±2 lines for every file inspected. There are no false claims about what the code does.

What needs to change before sign-off is a small set of stale source comments that the diagrams faithfully quote — and a few line ranges that are off by 1–3 lines. None of these are blockers; they are cleanup items. Two more substantive observations are worth raising (the stale class Java Doc in Admin Match Service that contradicts the actual code, and the documented inconsistency between after Commit enqueue in Auth Service forgot Password vs. inline enqueue in Admin Post Service). Both are real, the diagrams flag both, and neither blocks approval of the diagrams.

---

## Per-flow findings

### 01-register-tutee md
- Status: OK
- Auth Controller implementation reference (register Tutee block) — correct.
- the issue auth cookie operation cited at line 199 — correct.
- Auth Service implementation reference for register Tutee — correct; after Commit at 121-130 — correct.
- Failure-mode list (429 RATE LIMIT EXCEEDED, 409 EMAIL ALREADY REGISTERED / PHONE ALREADY REGISTERED, 400 validation, 500 Illegal State Exception using Encryption failed during user creation) matches Auth Service build User line 515 exactly.

### 02-register-tutor md
- Status: OK
- Register implementation reference (TUTOR REGISTRATION STASH KEY) — correct.
- Register implementation reference (Tutor Registration Stash shape) — correct.
- Commission implementation reference (COMMISSION TERMS VERSION 2026-01-01) — correct.
- Commission implementation reference (mount guard + beforeunload wipe) — correct; mount guard is 53-66, beforeunload listener is 71-75, so the cited range covers both.
- Auth Controller implementation reference — correct (register Tutor controller method).
- Auth Service implementation reference — correct (register Tutor service).
- Ack check is "first validation (line 137)" — correct (the commission acknowledged operation at line 137).
- Two audit log entries USER REGISTERED / COMMISSION ACKNOWLEDGED at lines 183-195 — correct.

### 03-login-with-mfa md
- Status: OK
- Auth Controller implementation reference — correct (login + verify Mfa methods).
- Auth Service implementation reference for login — correct (boundaries exact).
- Auth Service implementation reference for verify Mfa — correct (boundaries exact).
- LOCK AFTER ATTEMPTS is 5, LOCK DURATION MINUTES is 15, multi-factor authentication SESSION expiry period SECONDS is 300, multi-factor authentication MAX ATTEMPTS is 3 — all match constants at Auth Service implementation reference-52.
- jwt admin.session-ttl-seconds default 1800 — confirmed (line 87).
- Gap note for Login implementation reference — verified: the on Submit always tries res data.user and the confirmation that multi-factor authentication is required shape returned by Auth Service login for ADMIN users does not include user, so admin login falls through the catch and shows "Invalid email or password". Real gap; correctly flagged.

### 04-forgot-reset-password md
- Status: OK
- Auth Controller implementation reference — correct.
- Auth Service implementation reference (forgot Password) and 424-463 (reset Password) — both exact.
- one-time passcode LENGTH is 6, one-time passcode EXPIRY MINUTES is 15, one-time passcode MAX ATTEMPTS is 5, forgot Password Limit is 3 — all match constants at Auth Service implementation reference-50 and Value defaults at 80-84.
- Invalidate-all at line 408 — correct.
- after Commit enqueue at line 420 — correct.
- Admin info-log at line 454 — correct.
- forgot Password silent-200 on rate-limit and unknown email — verified (no throw on rate limit; .if Present on user lookup).

### 05-csrf-bootstrap md
- Status: OK
- Auth Controller implementation reference for the csrf operation returning 204 — correct.
- Security Config implementation reference for the with http only no operation + the set cookie customizer operation — correct.
- Csrf Cookie Filter implementation reference for the get token operation forcing materialisation — correct.
- use Init Auth implementation reference ordering / .catch on /csrf — correct (lines 25-30 in the file).
- api implementation reference axios with Credentials, xsrf Cookie Name, xsrf Header Name — correct.

### 06-tutee-post-creation md
- Status: OK
- Tutee Post Controller implementation reference — correct.
- Tutee Post Service implementation reference for create And Submit — correct.
- validate Session Duration at "lines 248-254" — the method spans 246-259 but the the of operation literal is on lines 249-253, so the cited range is essentially correct.
- Analytics NOT wrapped in after Commit — verified at lines 78-83 (plain analytics Event Publisher publish inside a database transaction that saves all related changes together, no after Commit wrapping).
- Gap "no notification Service emit for admin notify" — verified: notification Service is not even injected into Tutee Post Service.

### 07-tutor-post-creation md
- Status: OK
- Tutor Post Service implementation reference for create And Submit — correct.
- Sets initialised at lines 85-90 (subject Codes / level Codes / venue Types / group Sizes) — correct.
- years Experience default 0 if null at line 81 — correct.
- Gap "no notification Service emit to admins on submission" — verified; notification Service is not injected into Tutor Post Service either.

### 08-admin-approval md
- Status: ISSUES (minor)
- Admin Post Controller implementation reference — correct.
- Admin Post Service implementation reference — correct (full file).
- N+1 fix lines 64-81 — correct (batch profile lookups).
- decrypt Safe at line 381 — correct.
- Audit-log comment "target Entity Id post Id, NOT user Id" said to be at line 411 — the actual comment is on **line 412** (the target entity id operation). Off by 1.
- Audit failures swallowed at lines 416-419 — correct.
- Email body lines 426 and 440 — correct.
- Email enqueue failure at lines 432 and 446 — correct.
- State machine: * → PENDING, PENDING → ACTIVE | REJECTED, ACTIVE → PENDING (edit-while-active), DRAFT → PENDING flagged as not exercised by current frontend, DRAFT/REJECTED → CLOSED via soft Delete — matches Tutee Post Service update / soft Delete (and tutor equivalent).
- Correction required: bump line 411 → 412 in the target Entity Id citation. (Trivial.)

### 09-admin-match-creation md
- Status: ISSUES (stale source comment, not a diagram bug)
- Admin Match Controller implementation reference — controller method is 65-75 (comment 59-64). Practically correct.
- Admin Match Service implementation reference for create Match — correct.
- Both notifications emitted within the same a database transaction that saves all related changes together create Match — confirmed at lines 147-150.
- the now operation — correct (line 137).
- The diagram says "Class-level comment notes that contact details are NOT shared at this stage". The actual class-level Java Doc at Admin Match Service implementation reference reads "Notifications are **intentionally omitted** — the frontend success screen already reads 'No contact details have been shared yet'". This Java Doc is **out of date** — the code at lines 147-150 and 233-236 DOES emit notifications now (M9). The diagram correctly shows the emits in the sequence; the cited Java Doc itself is stale. This is a source-code cleanup item, not a diagram defect, but worth raising because a reader of the Java Doc would be misled.

### 10-admin-match-activation md
- Status: OK
- Admin Match Controller implementation reference for activate — correct.
- Admin Match Service implementation reference for activate Match — correct.
- DEFAULT COMMISSION RATE 0.5 at line 59 — correct.
- the round operation (int rounding) — correct.
- Both ACTIVATED notifications emitted within the same a database transaction that saves all related changes together activate Match (lines 233-236) — correct.
- Match Dashboard Service ADMIN CONTACT EMAIL line 40 — correct.

### 11-admin-mark-commission-paid md
- Status: OK
- Admin Commission Controller implementation reference — correct.
- Admin Commission Service implementation reference — correct.
- Only tutor is notified (line 67) — verified; tutee is intentionally not notified on commission payment.
- Match ACTIVE → COMPLETED transition unlocks Review Service submit Review — correct (Review Service requires Match Status COMPLETED).
- 409 idempotency on re-pay — correct.

### 12-admin-match-cancel-cascade md
- Status: OK
- Admin Match Controller implementation reference — correct (cancel endpoint, including comment).
- Admin Match Service implementation reference for cancel Match — correct.
- CANCELLABLE STATUSES PENDING, ACTIVE cited at line 57 — actual declaration spans lines 56-57. Effectively correct.
- Commission cascade PENDING → CANCELLED at lines 273-277 — correct.
- Gap "no notification on cancel" — verified: cancel Match has no notification Service emit calls (unlike create and activate). Match status machine and Commission status machine both check out against the code.

### 13-tutee-request-interview md
- Status: OK
- Tutor Detail implementation reference — correct (request Mutation defined at 45-55, handlers at 79-90).
- Interview Controller implementation reference — correct.
- Interview Service implementation reference for request Interview — correct.
- safe Title at lines 430-433 — correct.
- Admin fan-out via the find by role operation inside the same a database transaction that saves all related changes together — confirmed at lines 119-126.
- Notification body uses the safe title operation — correct.

### 14-admin-schedule-interview md
- Status: OK
- Interview Controller implementation reference — correct.
- Interview Service implementation reference — correct.
- Singapore time conversion at lines 241-244 — correct.
- Only tutee is notified — correct (no tutor emit in schedule Interview).

### 15-admin-set-meeting-link md
- Status: OK
- Interview Controller implementation reference — correct (Patch Mapping).
- Interview Service implementation reference — correct.
- No notification on link set — verified; set Meeting Link does not call notification Service emit. This matches the diagram's deliberate-decision note.

### 16-interview-completion-paths md
- Status: OK
- Complete: Interview Service implementation reference — correct.
- Admin cancel: Interview Service implementation reference — correct.
- No-show: Interview Service implementation reference — correct.
- Tutee self-cancel: Interview Service implementation reference — correct.
- load Interview Owned By Tutee ownership 404-not-403 at line 410 — correct.
- No-show "notify the OTHER party" semantics at lines 382-396 — correct.
- Interview status machine matches every transition in code.

### 17-tutee-views-matches md
- Status: OK
- Match Dashboard Controller implementation reference — correct.
- Match Dashboard Service implementation reference for list Tutee Matches — correct.
- ADMIN CONTACT EMAIL at line 40, conditional reveal at line 96 — correct.
- Three batch lookups (commissions, tutor display names, reviewed match IDs) at lines 65-83 — correct.
- my Review Submitted boolean drives "Leave review" CTA — correct.

### 18-tutor-views-matches md
- Status: OK
- Match Dashboard Controller implementation reference — correct.
- Match Dashboard Service implementation reference for list Tutor Matches — correct.
- Commission triple (monthly Fee Sgd, commission Amount Sgd, commission Status) exposed in Tutor Match Response — correct at lines 150-154 and 163-166.
- No tutor-display-name lookup on tutee side (tutee Subject + level used instead) — correct.

### 19-tutee-submits-review md
- Status: OK
- Submit Review implementation reference — correct.
- Review Controller implementation reference — correct.
- Review Service implementation reference — correct.
- Pre-check at line 106, Data Integrity Violation Exception translation at lines 121-130 — correct.
- resolve Display Name at lines 211-219 — correct.
- the to response operation at lines 300-337 — correct.
- Class-level the pre authorize operation; participant check in service throws 403 NOT PARTICIPANT — correct.

### 20-public-tutor-detail-reviews md
- Status: OK
- Public Browse Controller implementation reference for get Tutor Post — correct.
- Tutor Post Service find Active Post By Id at lines 270-277 — correct.
- Review Controller implementation reference — correct.
- Review Service list Reviews About Tutor Post at lines 156-173 — correct.
- Security Config implementation reference permit All at lines 129-130 — correct.
- get Ratings For Reviewee Ids at lines 180-193 — correct.
- Note about 400 NOT FOUND from detail (uses the bad request operation at line 275) — correct, this is actually a real source quirk: find Active Post By Id returns web request protocol 400, not 404, on missing-or-non-ACTIVE. Diagram correctly calls this out under failure modes. **[CORRECTION INT-137b, 2026-06-13: this quirk is RESOLVED — find Active Post By Id now uses the not found operation → web request protocol 404, repo-wide. The 404-vs-400 inconsistency no longer exists.]**

### 21-notification-bell-flow md
- Status: OK
- Notification Bell implementation reference (REFETCH INTERVAL MS 60 000) — correct.
- Notification Bell implementation reference (use Query block) — correct.
- Lines 17-22 for role routing — correct.
- Notification Controller implementation reference — correct (class body is 30-74).
- Notification Service implementation reference — correct.
- Cross-user 404 (not 403) at lines 89-92 — correct.
- mark All Read is bulk database query language — verified (single the mark all read for recipient operation repo call at line 106).

### 22-favourite-toggle md
- Status: OK
- Browse Tutors implementation reference (bulk ids Among) — correct.
- Tutor Card implementation reference (optimistic add/remove + snapshot/restore) — correct.
- Favourite Controller implementation reference for add Favourite returning 201 vs 200 — correct.
- Favourite Service implementation reference for add Favourite — correct.
- Favourite Service implementation reference for remove Favourite — correct.
- Favourite Service implementation reference for bulk-probe — correct.
- Race-safe via outer existence check + catch on Data Integrity Violation Exception (lines 78-86) — correct.

### 23-public-browse-with-filters md
- Status: OK
- Browse Tutors implementation reference (use Query) — correct.
- Lines 24-30 for filter state — correct.
- Lines 45-52 for has Active Filter — correct.
- REPEAT ARRAY PARAMS at line 17 — correct.
- Public Browse Controller implementation reference for browse Tutors — correct.
- Tutor Post Service find Public Posts Filtered at lines 224-263 — correct.
- the min operation server-side cap at line 236 — correct.
- N+1 candidate for tutor display name in the to response operation at lines 305-309 — correctly flagged.
- permit All (Security Config 129-130) and cross-site request forgery protection ignore ** interface operation at line 108 — correct.

### 24-rate-limited-endpoint md
- Status: OK
- Rate Limit Service implementation reference Lua script — verified at lines 19-33.
- Keys login Key, register Key, forgot Password Key, change Password Key — verified at lines 53-67.
- Defaults (login 10/3600, register 5/3600, forgot-password 3/3600) — verified against Auth Service implementation reference Value defaults.
- client Ip derivation at Auth Controller implementation reference-246 — correct (CF-Connecting-IP, X-Forwarded-For first hop, get Remote Addr).
- 429 on Lua = 0; silent 200 on forgot-password rate limit — correct.

### 25-csrf-protected-write md
- Status: OK
- api implementation reference — correct.
- Security Config implementation reference (csrf Request Handler + csrf Repo) — correct.
- Security Config implementation reference for ignoring Request Matchers — correct (auth endpoints, public/**, actuator/**).
- the add filter after operation for Csrf Cookie Filter — verified at line 192.
- 403 envelope on header missing/mismatch — correct (default Spring cross-site request forgery protection failure).

### 26-email-fanout-on-event md
- Status: ISSUES (real source inconsistency the diagram correctly flags)
- Email Queue Service implementation reference STREAM KEY line 21 — correct.
- Message map lines 26-32 — correct (template Id, to, enqueued At + caller params).
- Try/catch lines 33-36 (swallows errors) — correct.
- Auth Service implementation reference (after Commit enqueue in forgot Password) — correct.
- Admin Post Service queue Approval Email at lines 422-434 and queue Rejection Email at lines 436-447 do NOT wrap enqueue in after Commit — correct, the diagram flags this as a real audit item.
- No consumer in tree — confirmed via codebase search; no class matching *Consumer* exists in intuition-the private implementation folder
- Producer list (forgot Password N-05 string literal, POST APPROVED, POST REJECTED) — verified complete; no other email Queue Service enqueue callers exist.

### 27-encrypted-pii-write-and-lookup md
- Status: ISSUES (one line range slightly off)
- Encryption Service implementation reference Tink Deterministic Aead + keyed message authentication-SHA-256 — correct.
- AAD an empty binary response on encrypt/decrypt at lines 51, 56 — correct.
- compute Hmac lowercases via Locale ROOT cited at line 67 — correct.
- encryption master-key-keyset and encryption hmac-secret properties — verified at lines 26-30.
- Auth Service build User cited at lines 504-517 for the Illegal State Exception using Encryption failed during user creation — actual build User spans **lines 498-517**, with the throw at line 515. The 504-517 range starts inside the method header (line 504 is just User Role role). Should be 498-517 (or just cite line 515 for the throw).
- decrypt Safe returns null instead of throwing — verified at Admin Post Service implementation reference-389.

---

## Cross-cutting findings

1. **Line-number drift is consistently small (≤2 lines)** across all 27 diagrams. The only meaningful drift cases are flow 27 (build User 504-517 should be 498-517) and flow 08 (target Entity Id comment 411 should be 412). Acceptable, but worth a sweep.

2. **Admin Match Service class Java Doc is stale.** The Java Doc on lines 42-44 says "Notifications are intentionally omitted..." but the actual create Match (lines 147-150), activate Match (lines 233-236), and (deliberately) NOT cancel Match all use notification Service. Diagram 09 quotes that comment as if it were accurate context. The diagrams correctly show the emits in the sequence. Fix the source comment OR fix the diagram's cite — either is OK. Recommend fixing the source comment, since the Java Doc is misleading to anyone reading the class.

3. **Documented gaps are all real.**
   - Flow 03: Login implementation reference does not branch on confirmation that multi-factor authentication is required — verified.
   - Flow 06, 07: no notification Service emit on tutee/tutor post creation — verified (the field isn't even injected into those services).
   - Flow 12: cancel Match lacks notification emits — verified.
   - Flow 26: no email consumer in tree — verified.
   - All gaps are correctly flagged in the diagrams; no false positives, no missed gaps within the surveyed code.

4. **Transactional boundaries are correctly drawn.**
   - a database transaction that saves all related changes together after Commit pattern in Auth Service (register tutee/tutor, login, verify Mfa, logout, forgot Password, reset Password, delete Account) — diagrams correctly show side-effect emits (audit logs, analytics, email enqueue) deferred until after commit where the code does so.
   - Notification emits in matching/interview/review flows happen inside the active a database transaction that saves all related changes together (not after Commit) — diagrams correctly do NOT label these as after Commit.
   - Flow 06 (tutee post creation) correctly notes that analytics events are NOT wrapped in after Commit (potential phantom event if tx rolls back) — real source observation.

5. **Failure modes match Global error handler and App Exception factory methods** for every diagram. Mapping verified:
   - App Exception unauthorized → 401; Access Denied Exception / App Exception forbidden → 403; App Exception not Found → 404; App Exception conflict → 409; App Exception locked → 423 (with retry After); App Exception too Many Requests → 429; App Exception unprocessable Entity → 422.
   - Method Argument Not Valid Exception / Http Message Not Readable Exception / Method Argument Type Mismatch Exception → 400 VALIDATION ERROR.
   - Data Integrity Violation Exception containing "Invalid interview status transition" → 422 INVALID STATUS TRANSITION (interview state machine). Other Data Integrity Violations → 409 CONSTRAINT VIOLATION. Generic Exception → 500 INTERNAL ERROR.
   - All status codes in every diagram's "Failure modes" section line up with the above.

6. **Batch-lookup claims are all verified.**
   - M8 match dashboard: 3 batch loads (commissions, tutor profiles, reviewed match IDs) — verified at Match Dashboard Service 65-83 and 134-140.
   - M10 favourites bulk ids Among: verified at Favourite Service 101-106 and Browse Tutors implementation reference 93-110.
   - M11 ratings aggregate: get Ratings For Reviewee Ids single-pass database query language AVG/COUNT — verified at Review Service 180-193, called from Tutor Post Service load Ratings.
   - M3 admin queue N+1 fix: two batched find All By User Id In calls — verified at Admin Post Service 64-81.
   - Admin matches list (M5): commission batch load — verified at Admin Match Service 177-182.

7. **State machines (08, 12, 16)** match the actual status enum transitions and the service code that performs them. No invalid transitions are drawn; no real transitions are missing.

---

## Required corrections

The following are cleanup items. None block approval, but should be applied for accuracy:

1. **08-admin-approval md** — change the cited line for the target Entity Id comment from line 411 to line 412.

2. **27-encrypted-pii-write-and-lookup md** — change the build User citation from line 504-517 to either lines 498-517 (full method) or just line 515 (the actual an error indicating that the attempted state change is invalid).

3. **09-admin-match-creation md** — soften or remove the claim that the class-level comment "notes that contact details are NOT shared at this stage". The source Java Doc actually claims notifications are intentionally omitted, which contradicts the code. Either:
   - Note in the diagram that the class Java Doc is stale and the code does emit notifications (M9 retrofit), OR
   - Fix the source Java Doc in Admin Match Service implementation reference to reflect current behaviour.

4. **(Source, not diagram)** Optional source-side cleanup the diagrams already flag:
   - Auth Service forgot Password uses string literal N-05 rather than a constant in Email Template Keys.
   - Admin Post Service queue Approval Email / queue Rejection Email do not wrap their enqueue calls in after Commit, unlike Auth Service forgot Password. Real inconsistency.

5. **(Source, not diagram)** Admin Match Service class Java Doc on lines 38-45 is stale and contradicts the current code's notification behaviour. Update to reflect current emit semantics (or call out that Java Doc should be removed).

---

## Sign-off

Subject to the five corrections above (three of which are 1-line line-number / wording fixes inside the docs and two of which are source-side cleanup the diagrams already correctly identify), I approve the 27 sequence diagrams as the engineering reference for these flows.

— Tech Lead
