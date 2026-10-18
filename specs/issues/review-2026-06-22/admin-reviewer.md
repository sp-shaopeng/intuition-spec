# Admin / Operator Review — InTuition SG (LOCAL) — 2026-06-22

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

External reviewer driving the LOCAL app as an Admin / platform operator (super-admin REDACTED email + normal admin).
Tooling: Playwright MCP. frontend http://localhost, application programming interface http://localhost.

## 1. Overall impression
The admin BACK-END (the actual operator capabilities) is in good shape: the approval queue, approve/reject with
mandatory-reason validation, analytics with a working date-range + inverted-range guard, admins list, and the
super-admin protections all behave correctly and respond fast when probed directly. The information design of the
console is clean and operator-friendly (clear nav, friendly subject/level labels, service-level agreement "On track" chips, a useful
analytics payload).

BUT the console is currently **frustrating-to-unusable to actually drive in a browser** because the admin session
keeps getting dropped on full-page navigation (S1), compounded by the login form being hijacked by stale browser
autofill (S4). As a 1–2 person ops team, the single most damaging thing is "I refreshed the analytics page and got
kicked to login" — that happened to me on nearly every hard navigation. The underlying functions work; the
session/navigation layer does not hold together. Fix S1 (and the /auth/refresh recovery) and this becomes a
genuinely efficient console. As-is, I could approve a post in one second via the application programming interface, but could not reliably keep a
session long enough to click through approve → match → commission in the user interface.

Could a real coordinator run the marketplace efficiently? On capability: yes. On the current browser experience:
not reliably — they'd be re-logging-in constantly. The seeded data also has zero matches/interviews/commissions,
so the matching → commission → interview oversight flows could not be exercised end-to-end (NEEDS ENGINEER).

## 2. Journey notes

### Login + forced password change
- Super-admin login application programming interface works (200, role: ADMIN, is Super Admin: yes, password Change Required: yes).
- **user interface login was flaky due to browser autofill clobbering the password field** — saved credentials from a previous
  register session repeatedly overrode my typed email/password and pushed me to /register. Had to set values via JS.
  This is partly environmental (browser saved creds) but worth noting: the login form does not use
  autocomplete is off/distinct field handling that resists stale autofill. Minor.
- password Change Required gate works correctly: visiting /admin redirected to /admin/change-password with a clear
  "You must change your password before you can continue." screen (evidence admin-01). Changed admin → REDACTED password.
  **NOTE for engineer: seeded super-admin password is now REDACTED password locally.**
- After change: success toast "Password changed successfully" and auto-landed on /admin/review. Smooth.
- /dashboard is a 404 (admins land on /admin/*; not a bug, just no admin dashboard at that path).

### Approval / Review queue (/admin/review)
- Clean, fast table: Type / Post (subject + friendly level) / Rate / Submitted (Singapore time + "On track" service-level agreement chip) / Status / Action.
- 25 posts awaiting review; both TUTEE and TUTOR posts in one queue. Friendly labels (e g. "Mathematics (PSLE)", "H2 Chemistry (A-Level)").
- Notifications bell shows "184 unread" (9+ badge) — POST SUBMITTED admin fan-out is clearly firing.
- Nav is clear: Review / Matching / Jobs / Interviews / Commission / Users / Analytics / Admins.

### Approval actions (probed via application programming interface in a stable single-context session)
- read operation for admin posts pending → 200, 26 pending posts (queue source is healthy).
- read operation for admin posts id → 200 returns full post incl. submitter name/email/phone (admin sees personal information — expected for ops).
- **Approve happy path:** submit operation for admin posts id approve → 200, status ACTIVE, and the post correctly LEFT the
  pending queue afterward. Works.
- **Reject guard:** submit operation for admin posts id reject with an empty reason → **400 VALIDATION ERROR**; with no body →
  **400**. Mandatory rejection reason is enforced (good — destructive action requires justification). Soft-delete
  expectation: rejected posts move to a rejected state (analytics shows rejected: 6), not physically removed.

### Matching / selection-requests / interviews (probed via application programming interface)
- read operation for admin matches → 200, read operation for admin matches?status is MATCHED → 200, read operation for admin selection-requests → 200,
  read operation for admin interviews → 200 — all endpoints healthy but the seed set returns EMPTY (no created matches /
  selection-requests / interviews). Could not exercise create-match → activate → cancel/reopen, choose-tutor →
  assign-winner, interview scheduling/no-show/meeting-link, or commission completion end-to-end. (NEEDS ENGINEER seed.)

### Analytics (/admin/analytics, probed via application programming interface)
- 200 with a genuinely useful operator payload: users total 301, tutees 155, tutors 134, posting funnel
  tutee 127, tutor 29, pending Review 25, active 92, rejected 6, matches funnel total 23, pending 12, active 1,
  completed 4, cancelled 6, revenue total Collected S$600, outstanding S$0, avg Commission/match S$600,
  collected In Period Sgd. period from,to,days framing present.
- **Date-range filter works:** ?from=&to= → 200. **Inverted range** (from to) → **400 INVALID DATE RANGE**
  with a clear message "Query parameter 'from' must be on or before 'to'." Good edge handling.

### Admin management & super-admin (probed via application programming interface)
- read operation for admin admins → 200, 12 admins; fields id, email, is Super Admin, created At, last Login At.
- **Self-protection:** delete operation for admin admins own Super Id → **403 CANNOT REMOVE SELF** ("You cannot remove your own account.").
- **Super-admin protection:** delete operation for admin admins other Super Id → **403 CANNOT REMOVE SUPER ADMIN**
  ("The super-admin account is protected and cannot be removed."). Both protections solid.
- Create-admin contract: submit operation for admin admins and email, password and display name both → 400
  "must not be blank" — there is another required field I didn't supply, and the error does NOT name which field
  (see S4-b minor). Could not complete a create via application programming interface blind; user interface form would label the field.

### Notifications (POST SUBMITTED fan-out)
- Bell showed 184 unread on first login, then **185** after a tutee post was submitted during the session —
  confirming the admin POST SUBMITTED after-commit fan-out fires. Did not open the notifications drawer (session
  drops blocked deeper user interface), but the count behaviour is correct.

### Mobile 390px (admin surfaces)
- Review-queue content is responsive at 390px (no page-level horizontal overflow; table collapses to a list).
- The top admin nav (8 sections) is crammed into a ~154px-wide **horizontally-scrolling strip** with no hamburger
  — all 8 links present but the operator must side-scroll a tiny bar to reach Analytics/Admins. See S3.

### Opening a post to review (the administrator's review page for the selected record)
- Clicking "Review →" on a queue row repeatedly **logged the admin out** and bounced to
  sign-in followed by a return to that administration review page. Console showed 401 on read operation for admin posts id, then 401 on
  submit operation for auth refresh, then redirect to login. See S1 issue below — root cause is the auth cookie being
  dropped on SPA navigation (Same Site is Strict + cross-origin →).
- The underlying application programming interfaces are healthy when the cookie IS present: read operation for admin posts pending → 200 (queue),
  read operation for admin posts id → 200 (detail), read operation for auth me → 200. So this is a session-transport defect, not a
  data/endpoint defect.

## 3. ISSUES

### S1 — Admin session does not survive a full-page navigation (auth cookie = Same Site is Strict, the private implementation folder programming interface cross-origin) → admin bounced to /login
- **Severity: S1** (blocks the core operator loop — a full page load / deep-link / refresh on any /admin/* route
  silently ejects the admin to login mid-task).
- **Repro (LOCAL, frontend  / application programming interface ), isolated from autofill:**
  1. Authenticate as admin via submit operation for auth login (cookie set: a session cookie scoped to the site, expiring after 30 minutes, hidden from browser scripts and restricted to same-site requests).
     Immediately after, in the same JS context, read operation for auth me → **200 ADMIN super** (cookie works for XHR/fetch).
  2. Do a top-level browser navigate to the administrator's review page for the selected record (a full document load, = deep-link / refresh / paste-URL).
  3. The SPA's bootstrap read operation for auth me now returns **401** — the auth cookie was NOT sent on the navigation —
     and the route guard redirects to /login. submit operation for auth refresh also 401s (does not recover).
- **Expected:** an authenticated admin who refreshes or deep-links an /admin/* URL stays logged in.
- **Saw:** session dropped on full-page navigation; ejected to /login. Reproduced repeatedly, with a clean
  single admin session (no competing session).
- **Root cause (evidence):** Set-Cookie value (withheld). Chromium treats localhost →
  localhost as cross-site for SameSite, so on a fresh top-level document load the Strict cookie is withheld
  from the bootstrap /auth/me; /auth/refresh then 401s instead of re-establishing. In-page fetch
  (no navigation) keeps the cookie fine, which is why client-side SPA transitions sometimes work but a
  refresh/deep-link/hard-load does not.
- **Impact / caveat:** a true single-domain production deploy (frontend and application programming interface same origin) would likely NOT hit the
  cross-site withholding, so this may be partly a dev-topology artifact. BUT: (a) the local dev experience is
  badly broken for any admin who refreshes a page, and (b) the /auth/refresh returning 401 rather than recovering
  the session is a real robustness defect that would also surface on legitimate access-token expiry.
  **NEEDS ENGINEER: confirm under prod single-origin, and fix refresh-on-401 to actually rotate/recover.**
- **Evidence:** admin-03-logged-out-on-review-click png; console (401 on /auth/me after nav, 401 on /auth/refresh).

### S3 — Admin top-nav (8 sections) is a cramped ~154px horizontal-scroll strip at 390px (no hamburger)
- **Severity: S3.** At 390px the 8 admin sections (Review/Matching/Jobs/Interviews/Commission[REDACTED LOCAL PATH])
  are squeezed into a ~154px-wide horizontally-scrolling container (scroll Width client Width). No hamburger /
  collapse. An operator on a phone must side-scroll a tiny bar to reach Analytics/Admins — slow and easy to miss
  sections. Content area itself is fine (no page overflow). Recommend a hamburger or a wrapped/2-row nav on mobile.
- **Evidence:** admin-04-review-queue-390px.png.

### S4-b — Create-admin validation error doesn't name the missing field
- **Severity: S4 (minor).** submit operation for admin admins with email and password and email, password and display name both
  return 400 VALIDATION ERROR "must not be blank" — a required field is missing but the message doesn't say which,
  so an operator can't tell what to fill. Field-level error messaging would save support time.

### S4 — Login form vulnerable to stale browser autofill silently overriding typed credentials
- **Severity: S4 (user experience/minor).** Saved register-flow credentials from a prior session repeatedly overrode the
  email/password I typed into the login form, producing "Invalid email or password" and even pushing the browser
  to /register. Had to set field values programmatically. The login inputs don't appear to guard against stale
  autofill (e g. distinct field semantics). Low impact for real users but worth a glance.

## 4. NEEDS ENGINEER
- **Confirm S1 under a single-domain / prod-like setup** (frontend and application programming interface same origin) — is the Same Site is Strict
  session stable there? And fix the /auth/refresh path so a 401 attempts re-auth rather than dead-ending to login.
- Because S1 keeps ejecting me, several deeper flows could only be exercised via direct application programming interface probing in a single JS
  context, not the full clickable user interface. Functions VERIFIED via application programming interface (work): approve, reject-reason guard, analytics +
  date-range + inverted-range guard, admins list, self/super-admin delete protection, queue source.
- **NEEDS ENGINEER — seed data to user interface-verify the rest** (the matching/commission/interview oversight flows): the
  local seed has ZERO matches, ZERO selection-requests, ZERO interviews, and no payable commission. To review:
  - **Matching:** at least one APPROVED tutee post + applicants so I can create a match, activate it, then
    cancel/reopen it; plus a free-market post with multiple applicants to exercise choose-tutor → assign-winner
    (the supersede-others atomic flow).
  - **Commission/completion:** a match in a state where commission is payable, so I can mark commission paid and
    complete the match.
  - **Interview oversight:** at least one interview request (ideally several incl. a PENDING SCHEDULING) so I can
    schedule, set a meeting link, and mark a no-show.
- **NEEDS ENGINEER — create-admin contract:** the required field beyond email/password isn't obvious from the application programming interface
  error; confirm the exact body the user interface form sends so create-admin can be exercised.
