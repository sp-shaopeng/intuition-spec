# InTuition frontend routes

**Current-file check: 18 October 2026.** This page describes the inspected website routes and navigation. The [current requirements](../CURRENT-REQUIREMENTS.md) remain the product contract. The [alignment review](../../docs/current-file-alignment.md) records missing or conflicting behaviour; no browser journeys were executed for this route inventory.

For the framework versions and overall architecture, read [how the application is built](../../docs/how-it-is-built.md).

## Routes

This inventory describes the 56 route entries found in the current frontend on 18 October 2026, including redirects and the fallback. It is based on source inspection, not a fresh browser run. Route availability does not establish that every linked workflow satisfies the [current requirements](../CURRENT-REQUIREMENTS.md).

Public tutor and job lists are accessible without signing in. Their detail pages require sign-in. **Current implementation gap:** both list screens still hide filters and pagination from guests, although C01 requires guest filtering and access to the permitted public result pages. The requirement remains unchanged.

### Public pages and registration

| Path | Access | Current page or behavior |
|---|---|---|
| `/` | Public | Landing page with tutor and tuition-job discovery links and showcase sections. |
| `/login` | Public; signed-in users are redirected | Email/password sign-in. A validated `returnTo` takes priority over the normal role landing. If an administrator requires multi-factor authentication, the current screen reports that this interface cannot complete it; there is no working code-entry screen here. |
| `/register` | Public; signed-in users go to their role landing | Parent/tutor role selection and account details. The `role` query parameter can select the registration role. Parent registration creates the account here; tutor registration continues through the following two pages. |
| `/register/profile` | Registration draft required; no sign-in required | Optional tutor profile capture. The tutor may enter details or skip to commission terms. Missing draft state returns them to `/register?role=tutor`. |
| `/register/commission` | Registration draft required; no sign-in required | Tutor commission disclosure and mandatory acknowledgment. Account creation occurs after acknowledgment; successful registration opens `/tutor/dashboard`. Missing draft state returns to `/register?role=tutor`. |
| `/forgot-password` | Public | Request a password-reset code by email. |
| `/reset-password` | Public | Enter the registered email, six-digit code and new password. An `email` query parameter may prefill the address; the reset code is entered in the form. |
| `/rates` | Public | Tuition rate guide. |
| `/privacy` | Public | Privacy policy. |
| `/terms` | Public | Terms of service. |
| `/about` | Public | About the service. |
| `/faq` | Public | Frequently asked questions. |
| `/contact` | Public | Contact information and an email link. This page does not contain an enquiry-submission form. |
| `/tutors` | Public | Tutor-listing catalogue. Signed-in users can use its filters and pagination; the guest-control gap is noted above. |
| `/assignments` | Public | Tuition-job catalogue. Signed-in users can use filters and pagination; tutors also see application controls. The guest-control gap is noted above. |
| `/account/contact-change/confirm` | Public landing page; valid confirmation token required to commit a change | Reads the confirmation token from the link and presents an explicit Confirm action. Opening the page alone does not apply the contact change. |

Tutor onboarding has an optional profile-capture page before the commission acknowledgment page. Its draft is held in memory, so refreshing or opening a later step without the draft returns to registration. A separate tutor listing is not required to create an account or to apply once the approved profile-completeness rules are satisfied.

### Details that require sign-in

| Path | Access | Current page or behavior |
|---|---|---|
| `/tutors/:id` | Any signed-in account; actions depend on role and eligibility | Tutor-listing detail, reviews, and eligible parent invitation/interview controls. |
| `/assignments/:id` | Any signed-in account; actions depend on role and eligibility | Tuition-request detail and eligible tutor application controls. |

An anonymous visitor is redirected to `/login?returnTo=…` before either detail component mounts. The return path includes the requested pathname and query string. Login validates the return destination before navigating back. Viewing a detail page does not authorize contact disclosure or an action on another person's records.

### Parent or learner area

These pages require the TUTEE role. Anonymous visitors are sent to login with the requested pathname as `returnTo`; a signed-in user with another role is sent to their own role landing. Record ownership and action eligibility remain separate server checks.

| Path | Current page or behavior |
|---|---|
| `/tutee/dashboard` | Own requests, request details and permitted management actions, plus outbound activity. |
| `/tutee/posts/new` | Request wizard; also handles existing-request editing through its query parameters. |
| `/tutee/requests/:jobId/applicants` | Applicants for the parent's request, interview initiation and tutor selection requests. Applicant cards omit dedicated private contact fields. |
| `/tutee/interviews` | Own interview list. |
| `/tutee/interviews/:id` | Interview detail, negotiation and permitted cancellation. |
| `/tutee/profile` | Profile and account settings, with an inline reviews section. |
| `/tutee/matches` | Own matches; contact-reveal controls appear for ACTIVE or COMPLETED matches and remain hidden for PENDING matches. The required first-session confirmation action is not implemented in this screen. |
| `/tutee/favourites` | Saved tutor listings. |
| `/tutee/notifications` | Parent notification centre. |

### Tutor area

These pages require the TUTOR role. Anonymous visitors are sent to login with the requested pathname as `returnTo`; a signed-in user with another role is sent to their own role landing. Record ownership and action eligibility remain separate server checks.

| Path | Current page or behavior |
|---|---|
| `/tutor/dashboard` | Own tutor listings and their management actions. This is the listings dashboard; normal tutor login currently lands on `/assignments`. |
| `/tutor/listings/new` | Create a tutor listing. |
| `/tutor/applications` | Application pipeline and permitted application actions. |
| `/tutor/interviews` | Own interview list. |
| `/tutor/interviews/:id` | Interview detail, proposed-time responses and permitted cancellation. |
| `/tutor/profile` | Tutor profile and account settings, with an inline reviews section. |
| `/tutor/matches` | Own matches; contact-reveal controls appear for ACTIVE or COMPLETED matches and remain hidden for PENDING matches. |
| `/tutor/invitations` | Received invitations and permitted responses. |
| `/tutor/notifications` | Tutor notification centre. |

### Administrator area

These pages require the ADMIN role. Anonymous visitors are sent to `/login`; users with another role are sent to their own role landing. An administrator whose account requires a password change is redirected to `/admin/change-password` before entering the other admin pages. Specific operations have additional server permissions and status checks.

| Path | Current page or behavior |
|---|---|
| `/admin/review` | Post-moderation queue. |
| `/admin/review/:id` | Post detail with approval/rejection controls when the post is eligible for review. |
| `/admin/matching` | Match creation and management. |
| `/admin/posts` | Active tuition-job list. |
| `/admin/posts/:jobId/applicants` | Applicants for a job and permitted assignment, closure or moderation actions. |
| `/admin/commission` | Commission ledger and payment-recording interface. Its presence does not establish the complete corrected confirmation and invoicing workflow. |
| `/admin/interviews` | Interview queue and scheduling/lifecycle controls, with a selection-request view. |
| `/admin/interviews/:id` | Interview detail, negotiation history and eligible scheduling/lifecycle controls. |
| `/admin/analytics` | Platform analytics interface. |
| `/admin/users` | Tutor and parent account moderation. |
| `/admin/admins` | Administrator account management. The interface allows administrators to create accounts and reserves removal controls for the super-admin, with protected-account restrictions. |
| `/admin/change-password` | Required password-change screen. An administrator who no longer requires this step is redirected to `/admin/review`. |
| `/admin/notifications` | Administrator notification centre. |

### Shared review page

| Path | Access | Current page or behavior |
|---|---|---|
| `/matches/:matchId/review` | Sign-in required | Match review form shared by tutors and parents. Submission is subject to checks for participation, completed-match status and duplicate reviews. The route itself is not nested under either role's layout. |

### Redirects and fallback

These are distinct route entries, not additional page implementations. Redirects replace the current client-side history entry.

| Path | Access or routing condition | Destination or behavior |
|---|---|---|
| `/requests` | Public | Redirects to `/assignments`, replacing the retired tutor-only browse page. |
| `/tutee/reviews` | TUTEE role | Redirects to `/tutee/profile`, where reviews are shown inline. |
| `/tutor/reviews` | TUTOR role | Redirects to `/tutor/profile`, where reviews are shown inline. |
| `/admin` | ADMIN role, subject to the required-password-change gate | Redirects to `/admin/review`. |
| `/dashboard` | Waits for session initialization | Redirects to `/login` when signed out; otherwise to `/tutee/dashboard` for a parent, `/assignments` for a tutor, or `/admin/review` for an administrator. |
| `*` | Any unmatched path | Not Found page. |

There is no shared `/notifications` route: each role uses the notification path listed above. Separate `/admin/login`, `/admin/mfa` and `/admin/mfa/enroll` routes are also absent from this route inventory. Older page inventories that list them describe an earlier design.

## Current navigation

The navigation below was inspected on 18 October 2026. Earlier M17 and INT-049 navigation notes describe earlier designs; they should not be presented as the current navigation without checking the later changes.

| Account state | Normal login landing and logo destination | Main navigation | Additional navigation |
|---|---|---|---|
| Signed out | Logo opens `/`; login accepts a validated `returnTo` | Find a Tutor, Find Tuition Jobs, How It Works | Login and registration controls |
| Parent or learner | `/tutee/dashboard` | My Requests, Find Tutors, Interviews, Matches | Favourites, Reviews, Profile |
| Tutor | `/assignments` | Find Work, My Pipeline, Interviews, Matches | My Listings, Invitations, Reviews, Profile |
| Administrator | `/admin/review` | Review, Matching, Jobs, Interviews, Commission | Users, Analytics, Admins |

For parents and tutors, the desktop header shows the main links and a More disclosure for additional links. The mobile drawer contains both groups. Role-layout pages also have a mobile bottom bar for the first four main links; the public browse pages do not mount that bottom bar. The administrator layout displays its main and additional links together in a horizontally scrollable header.

The Reviews links remain in the additional navigation, but their destinations redirect to each role's profile page. Notifications use each role's own notification route. How It Works is a signed-out marketing link.

A safe `returnTo` overrides the normal login landing. The logo and `/dashboard` redirect use the same role landing choices. Successful tutor registration is a separate case: it currently opens `/tutor/dashboard`, even though ordinary tutor login opens `/assignments`.

The current signed-out header and mobile drawer link Post a Request directly to `/register?role=tutee`. This differs from the older M17 rule that generic acquisition controls should lead to login. No approved replacement for that instruction was established by this review. This remains a difference between the retained requirement and the current interface; the button must not be described as linking to `/login`.

## Historical navigation and delivery notes

The remaining notes preserve earlier implementation records. They are not the current route inventory. Their old destination and completion claims are superseded as descriptions of current files by the tables above; this does not itself approve a change to the retained product requirements. Reviewer names in historical records are AI roles.

## Nav role-awareness (2026-06-03 — INT-032/037/038/040, updated by INT-049)

- **Top Nav** is role-honest for logged-in users: a TUTEE sees only "Find a Tutor", a TUTOR only
  "Find Tuition Jobs" (INT-029); a logged-out visitor sees both (marketing). **"How It Works" is
  hidden once logged in** (INT-040). The **InTuition logo** routes to dashboard Path when logged in,
  / when logged out (INT-038).

### Dashboard tabs in the header after login (INT-049)
The redesign **moved the per-role dashboard tabs into the top Nav** — there is no longer a separate
in-page secondary tab bar, and no "click Dashboard first" step:
- For a logged-in **TUTEE/TUTOR**, the header renders that role's dashboard tabs directly (active-tab
  underline + aria-current is page), from nav Tabs implementation reference (tabs For User — role-honest: TUTEE set for
  a tutee, TUTOR set for a tutor, none for ADMIN/logged-out). The first tab is the dashboard landing.
  Tutee Layout / Tutor Layout now keep only their auth guard + <Nav/ + <Outlet/ (Tutor Layout also the
  Profile Completion Banner); the old secondary tab bar is removed. Notification Bell moved into the header.
- **Login lands on the dashboard first tab** (/tutee/dashboard, /tutor/dashboard, /admin/review),
  honouring a validated same-origin return To over it.
- **ADMIN** already rendered its tabs in the (purple) Admin Layout header — unchanged in behaviour;
  modernized to token-sourced admin.* colours by INT-049.
- Mobile: the role tabs appear in the mobile menu (parity).

### INT-053–059 wave (2026-06-04 — nav/IA + admin clickable detail)
- **TUTEE TABS** (nav Tabs implementation reference): added **"Find Tutors" → /tutors** (INT-057, public list); **removed the
  "Reviews About Me" tab** from both TUTEE TABS and TUTOR TABS (INT-059) — reviews now render as an inline section
  in the role **Profile** page; /tutee/reviews + /tutor/reviews are kept as **redirects** to /…/profile.
- **Tutee My Posts / Dashboard:** each posting now shows a **"View applicants" link → /tutee/requests/:job Id/applicants** (INT-058; the view already existed, was just unlinked).
- **Admin Matching + Interviews tables (INT-053):** the match/interview rows now show the **tutor ↔ tutee pair + the
  job posting**, each clickable to an existing detail route — tutor → /admin/review/:tutor Post Id (plain text when
  tutor Post Id is null on free-market matches), tutee → /admin/review/:tutee Post Id, job →
  /admin/posts/:tutee Post Id/applicants. Admin Review Detail (/admin/review/:id) now **gates its Approve/Reject
  actions to PENDING/REVISION REQUESTED** posts (shows a notice otherwise) so following a match link can't approve a
  live MATCHED post.
- **Auth pages (INT-054):** Register form widened (max-w-2xl); Auth Card gained a size prop; arbitrary
  max-w- … removed from Register/Login/Auth Card. **Qualifications (INT-055):** tutor Profile qualifications is now a
  select (QUALIFICATION OPTIONS) + "Other" free-text. **Auth refresh (INT-056):** auth Store seeds
  is Loading:yes so a hard refresh on a guarded route no longer bounces a logged-in user to /login.

### INT-062 (2026-06-04 — withdraw/close a live job + "job no longer available")
- **Tutee** can **close/withdraw a live job** (ACTIVE/PENDING) from My Posts/Dashboard (Action column "Close job"
  button, gated to live posts via is Live Post) → confirm dialog ("notifies the tutors who applied/were invited;
  can't be reopened") → submit operation for tutee posts id close. **Admin** has the same on /admin/posts/:job Id/applicants
  (submit operation for admin posts id close).
- Closing cascades the job's open **applications + pending invitations** to CLOSED; everywhere they appear
  (tutor Applications/Invitations, tutee Applicants, admin) the row **stays listed** with a **"Job no longer available"**
  badge (JOB GONE LABEL in format implementation reference; same phrase also relabels the match-fill states app SUPERSEDED /
  invitation EXPIRED); their action CTAs are disabled. Transparency: nothing is hidden/removed.

### INT-065 (2026-06-04 — admin moderation)
- **New Users page** at **/admin/users** (under Admin Layout): a moderation list of tutors & tutees,
  filterable by **role** (TUTOR/TUTEE), **status** (ACTIVE/BLOCKED), and a free-text **search** box; backed by
  read operation for admin users. Each row shows block status + (if blocked) the reason/note/blocked-at.
- **Block / Unblock dialogs:** Block opens a confirm dialog with a **reason dropdown** (Moderation Reason:
  POLICY VIOLATION / SPAM / ABUSE HARASSMENT / FRAUD / FAKE PROFILE / OTHER) and a **note field that is required
  when reason = OTHER** → submit operation for admin users id block. Unblock is a no-body confirm →
  submit operation for admin users id unblock. A blocked user's existing session is killed immediately (server Redis
  flag); unblock restores login but does **not** auto-reopen content closed on block.
- **Delete-posting moderation action** on **/admin/posts/:job Id/applicants**: an admin "Delete posting"
  control (reason dropdown + note-on-OTHER) → submit operation for admin posts id delete. This is **distinct from the
  INT-062 "Close job"** action also on that page — delete is a moderation removal (soft-delete deleted at,
  dropped from listings), close is the parent-lifecycle close.
- **Login** surfaces **ACCOUNT BLOCKED**: a blocked user attempting submit operation for auth login gets a 403 whose
  message names the block reason, shown inline on Login implementation reference.

## Auth-CTA principle (M17)

Founder rule: **the only path to /register is an explicit "Create account" control** — every other auth CTA defaults to /login.

- Generic acquisition CTAs ("Sign up to apply", "List your profile", "Get matched", showcase CTAs, "Find tuition jobs") → **/login**.
- Only explicit "Create account" / "Register" / "List as a Tutor" controls → **/register** (with role-select).
- Login implementation reference carries a loud, role-aware create-account split ("New here? Create a parent account / Create a tutor account" → /register?role=…) — the one sanctioned register entry.
- Protected-route bounces use /login?return To=…; Login implementation reference reads return To, validates it is a same-origin relative path (leading /, reject //, \, https, encoded tricks), and navigates there post-login, else falls back to role home. (Fixes a pre-existing bug where Login implementation reference ignored return To.)

## Navigation (M17)

- Nav links are labelled by **intent**: **"Find a Tutor"** (/tutors) and **"Find Tuition Jobs"** (/assignments) — not jargon like "Browse Assignments".
- The anonymous "Post a Request" CTA links to /login (no longer hard-links the protected /tutee/posts/new).
- The **mobile drawer reaches parity** with desktop — both intent links present (SG is mobile-first).

## Module Completion Status

| Module | Description | Status |
|---|---|---|
| M1 | Auth — registration, login, multi-factor authentication (time-based one-time passcode), password reset | Complete |
| M2 | Posting — tutee requests, tutor listings, public browse | Complete |
| M3 | Approval — admin review queue, approve/reject workflow | Complete |
| M4 | Matching — admin match creation, activation, cancellation | Complete |
| M5 | Commission — commission payment and match completion | Complete |
| M6 | Interview — interview request, scheduling, no-show tracking | Complete |
| M7 | Profile — tutor/tutee profile management | Complete |
| M8 | Match Dashboard — tutee/tutor views with assigned counterparty contact reveal on ACTIVE/COMPLETED | Complete |
| M9 | Notification Centre — in-app bell + page + 10 notification types | Complete |
| M10 | Search & Favourites — filter sidebar, heart, favourites page | Complete |
| M11 | Review & Rating — post-match stars, aggregate on tutor cards | Complete |
| M12 | Admin Analytics — platform metrics dashboard | Complete (verification pending) |
| M17 | Marketplace user experience overhaul — public /assignments Browse page; /tutors/:id + /assignments/:id login-gated via authentication wrapper; auth CTAs default to login (register only on create-account); friendly subject/level labels; Nav intent labels + mobile parity; landing order Jobs → Tutors | Deployed (no migration) |
