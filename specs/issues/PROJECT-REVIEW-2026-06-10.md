# Whole-Project Loophole Review — 2026-06-10 (INT-090 … INT-124)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Founder asked: *"review the entire project, see if there is any loophole, record all issues as open
issues. also evaluate can uiux be further improve."*

**Method:** four parallel read-only specialist lanes — Security (security-expert), Backend
business-logic + data (software-engineer + dba), Frontend engineering (frontend-expert), and user interface/user experience
journeys (uiux-designer) — each reading the real source, deduping against the tracker
(INT-001..089). The orchestrator independently re-verified every S1/S2 claim against the code
before filing (and in doing so found one the agents missed: the commission-pay missing-value failure, INT-091).
Every finding below is code-backed with file:line. Issues are filed as OPEN rows in
[README md](README.md); this document is the full evidence record.

**Headline:** no S1/S2 authz bypass, IDOR, personal information leak, or injection was found — the security core
(deny-by-default authz, ownership checks on every the requested record's identifier, personal information-free public data transfer objects, signed session token hardening,
advisory-locked quotas, the INT-016/087 transaction discipline) is genuinely solid. The defect
cluster lives at **the seams between modules**: the INT-034 free-market re-key and the INT-065
deleted at soft-delete each left consumers un-migrated, the frontend half of the INT-034 assign
contract was never wired, and the 401-means-logout frontend rule is over-broad.

---

## Part 1 — Findings (filed as INT-090 … INT-124)

### S1 — broken core flows

**INT-090 — Admin user interface cannot assign a free-market (listing-less) chosen tutor — frontend never sends winning Tutor Id.**
Selection Request Queue implementation reference still carries the INT-034
stopgap disable assignment when no tutor listing is attached ("assignment isn't available yet");
Matching implementation reference guards if assign Target tutor Post Id and posts only
the tutor-listing and tuition-request identifiers. the search for the chosen tutor-account identifier in the frontend → **zero hits** —
verified. Yet the backend shipped the real contract (Create Match Request implementation reference accepts exactly-one
of tutor Post Id | winning Tutor Id; Admin Match Controller implementation reference), and
Admin Selection Request Response already carries tutor User Id. M18's *default* path is
apply-without-listing, so the parent→choose→assign revenue funnel **dead-ends at the admin screen**
(the crown-jewel e2e passes because it drives the application programming interface, not this screen). Same class as INT-075
"required contract needs both ends". Fix: send winning Tutor Id when tutor Post Id is null; delete
the stopgap.

**INT-091 — Commission pay 500s (missing-value failure) on every free-market match — money path broken for the default match shape.**
Admin Commission Service implementation reference — read the tutor's identifier through the match's tutor listing
(verified). Match tutor Post is **null** for free-market matches (INT-034, migration 0033). The
missing-value failure is thrown *after* the state writes, so the tx rolls back: commission stays PENDING, match never
COMPLETED — the admin **cannot complete payment on any free-market match**. Found by the
orchestrator while verifying INT-092 (the agents' review missed this sibling). Fix: use
the resolve tutor user id operation like Contact Reveal Service/Admin Match Service/Match Dashboard Service
already do; add an IT with a null-tutor Post match through activate→pay.

### S2 — major

**INT-092 — Review submission 500s (missing-value failure) on free-market matches.**
Review Service implementation reference read the tutor account through the match's tutor listing (verified). Free-market
match completes → either party submits a review → missing-value failure → 500. Review & Rating is dead for
listing-less matches. Same fix shape as INT-091.

**INT-093 — A CANCELLED commission can be marked PAID — and resurrects a CANCELLED match to COMPLETED.**
Admin Commission Service implementation reference (verified): the only guard is equals PAID; CANCELLED (set by
Admin Match Service cancel Match, ) passes, then the set status operation is
unconditional. Scenario: cancel match (job reopens ACTIVE) → later POST the administration action for recording commission payment
→ 200: voided commission PAID, cancelled match COMPLETED while the job is live (possibly
re-assigned); analytics inflated; contact-reveal re-enabled on the cancelled match (COMPLETED is
revealable, Contact Reveal Service implementation reference). The 0048 @Version guards races, not this
sequential illegal transition. Fix: require invoice Status equals PENDING (409) and assert match
status before flipping to COMPLETED.

**INT-094 — Moderation-deleted job stays status is ACTIVE — still accepts NEW applies, invites, interviews, selections, and admin assign.**
Admin Moderation Service delete Posting sets only deleted At (line 266, verified); every lifecycle
guard checks status equals ACTIVE only, never deleted At: Job Application Service apply,
Job Invitation Service invite, Interview Service propose Interview,
Tutor Selection Service choose Tutor, Admin Match Service create Match. The one-time close
cascade closes *existing* intake; the post remains an ACTIVE target for *new* intake via direct
application programming interface — a removed-for-fraud job can still be matched. Fix: every ACTIVE guard also requires
deleted At equals no value (or delete Posting flips status → CLOSED and supersedes any CREATED selection
request, which it currently also leaves open).

**INT-095 — Wrong password on delete-account / change-password force-logs-out the user (401 interceptor over-broad).**
backend returns **401** INVALID CREDENTIALS for a wrong password confirmation on *authenticated*
endpoints (Auth Service implementation reference delete-account; Profile Service implementation reference change-password). frontend
logs out on every non-exempt 401 (api implementation reference); auth Redirect implementation reference exempts only
auth me, auth csrf, and the two showcase routes (verified). The inline handlers that expect to
render the 401 (Profile implementation reference, Profile implementation reference,
Force Change Password implementation reference) are dead code in practice: mistype your current password
→ instant client-side logout while the server session is still valid. Fix (pick one end, per
INT-075 lesson wire BOTH): backend returns 400/422 for credential re-confirmation failures (preferred —
the session IS valid), or frontend exempts auth delete Account/auth change Password/auth login from
the logout trigger.

**INT-096 — Logout never clears the Tan Stack Query cache — personal information persists and can bleed across accounts.**
auth implementation reference (perform Logout) and the interceptor logout branch only call
the logout operation (= clear the signed-in user). grep query Client clear → zero hits (verified). No
query key in query Keys implementation reference is user-scoped; stale Time: 30 000. On a shared device: logout →
login as another same-role user within 30s → the previous user's notifications/matches/admin user
directory (real names/emails/phones) render from cache with **no refetch**; after 30s they still
flash. Fix: the clear operation in perform Logout + the interceptor branch.

**INT-097 — Tutor has no contact-reveal on an ACTIVE match (one-sided personal information crossing).**
Contact Reveal Button is mounted only in Matches implementation reference (verified — only mount in
the private implementation folder). Matches implementation reference renders commission badges + admin-email box, no reveal —
yet read operation for matches id contact is participant-gated for both sides. The tutor (who travels to the
first session and pays the commission) cannot see the family's contact. Fix: mount the button on
the tutor ACTIVE-match card (the INT-082 phone-verify gate already generalises).

**INT-098 — Rejection reason never reaches the post owner; REJECTED / "Revision Needed" are dead-end states.**
Tutee Post frontend type has no rejection Reason (index implementation reference; only admin-side has it).
The REJECTED row's only action is "Resubmit" → /tutee/posts/new, a **blank form**
(Dashboard implementation reference). REVISION REQUESTED ("Revision Needed",
format implementation reference) has **no edit/revise route anywhere** (App implementation reference). Compounding: the admin's
reason field is capped at 50 chars *including* the preset (Review Detail implementation reference
— preset "Contact details in post content (personal information)" leaves **9 chars** of notes). The user is told
their post failed, never told why in-app, and must re-type everything — or for "Revision Needed"
literally cannot act. Fix: return rejection Reason on the owner data transfer object + render it; pre-fill the
wizard on Resubmit; add an edit path (or remove the unreachable status); widen the reason column
(Liquibase) to ~280.

**INT-099 — Raw subject/level enum codes rendered across all tutor-listing surfaces.**
Tutor Card implementation reference (public browse cards render tutor subjects/levels
raw — verified), Tutor Detail implementation reference, Dashboard implementation reference,
Dashboard implementation reference (parent sees CHN PSLE — PRI 6),
Invite Tutor Modal implementation reference, Review Detail implementation reference. The
humanisers exist (format implementation reference) and ARE used on the jobs side. The highest-traffic
public conversion surface speaks database enums. Fix: route every render through
the subject label operation/the level label operation; add a regression grep-test (this class already regressed once —
INT-068 fixed the admin ledger only).

**INT-100 — Browse CTAs send *logged-in* users to the login form; /login & /register have no already-authenticated redirect.**
Browse Tutors implementation reference "Post a Request" → /login unconditionally (this is a
tutee primary tab); Browse Assignments implementation reference "Are you a tutor? … Log In" →
/login unconditionally; Login implementation reference never checks use Auth Store user. A logged-in parent
clicking the platform's main conversion CTA lands on a login form. Fix: role-aware CTAs (tutee →
/tutee/posts/new; hide cross-role CTAs) + redirect authenticated visitors off /login//register
to login Landing For Role.

**INT-101 — Authed marketplace surfaces hard-capped at the first 50, no filters/pagination; copy lies about subject matching.**
Tutor's **login landing** /requests (Browse Requests implementation reference): fixed
the first page of up to 50 records, no Marketplace Filter Bar, no pager — and the empty state says "No open
requests *matching your subjects*" when no matching exists. Also Browse Tutors implementation reference
(authed: 50, no pager, while total Results shows a bigger count) and
Invitations implementation reference. The anonymous teaser /assignments has *more* capability
(filters) than the supply side's #1 revenue screen. Job/tutor #51+ is unreachable. Fix: reuse
Marketplace Filter Bar + the Favourites implementation reference pager pattern (or unify /requests into
/assignments role-aware); fix the copy.

### S3 — moderate

**INT-102 — Application withdraw lacks a from-status guard — MATCHED → WITHDRAWN allowed.**
Job Application Service withdraw (, verified): only guard is the WITHDRAWN no-op; the
winning tutor can flip their MATCHED application to WITHDRAWN while the Match stays ACTIVE — the
parent's applicant view shows the assigned tutor as withdrawn; match source Application points at
a WITHDRAWN row; later cancel's reopen Applications (REOPENABLE = SUPERSEDED|MATCHED) no longer
reopens it. Fix: allow withdraw only from APPLIED|SHORTLISTED (422 otherwise).

**INT-103 — Re-apply against an existing SHORTLISTED/CLOSED/REJECTED/MATCHED/SUPERSEDED row → unique-index 500.**
the apply operation/the apply internal operation handle only APPLIED (idempotent return) and WITHDRAWN (reactivate)
(Job Application Service implementation reference); any other pair status falls through to append records →
uq job applications active pair (0018, when status WITHDRAWN) →
Data Integrity Violation Exception 500. Reachable: block tutor (cascade Block Tutor sets apps
CLOSED, jobs stay ACTIVE) → unblock → re-apply → 500. Fix: load the pair row regardless of status;
409/422 (or reactivate per product call) on the other statuses — never blind-insert.

**INT-104 — Free-market assign & parent choose-tutor never validate the tutor id; tutor selection requests tutor user id has no FK.**
Tutor Selection Service choose Tutor uses tutor Id as-is (no existence/role check — contrast
Job Invitation Service invite step 5's role equals TUTOR filter); Admin Match Service create Match
uses get Reference By Id (nonexistent id → FK 500 at flush; an existing non-TUTOR id inserts
cleanly). Schema 0029: FKs only on tutee post id/job application id; tutor user id and
created by unconstrained. A parent can select their own user Id / a random unique identifier; admin assign then
500s or creates a Match whose "tutor" is a tutee/admin — contact reveal would expose that user's
personal information. Fix: validate existence + TUTOR role (404/422) and add the missing FKs in a new changeset.

**INT-105 — Match has no @Version — activate↔cancel admin race is an unguarded check-then-act.**
Match implementation reference has no @Version (Tutee Post has one per 0028c, Commission per 0048).
activate Match () and cancel Match () are load→check→write under READ COMMITTED.
Cancel commits first (job reopened), activate commits second → ACTIVE match + live commission on a
job open for re-assignment — the exact class INT-087 F4 fixed for Commission. Double-activate is
backstopped by commissions match id unique; cancel-vs-activate has no backstop. Fix: @Version
on Match (mirror 0048) + the existing entity-aware 409 handler.

**INT-106 — Invitation accept silently re-points the parent's open selection request to a different tutor.**
Job Invitation Service accept → Tutor Selection Service choose Tutor Internal: an existing
CREATED request is UPDATEd in place (the set tutor user id operation). Parent invites tutor A,
later explicitly selects applicant B; A accepts the stale invitation → the request silently
re-points to A; admin assigns "the parent's choice" which isn't. Fix: create-if-absent only (don't
overwrite a CREATED request pointing at a different tutor), or notify the parent of the overwrite.

**INT-107 — Tutee can edit a MATCHED or CLOSED job's substance — only PENDING is blocked.**
Tutee Post Service update rejects only status equals PENDING; ACTIVE re-queues to PENDING,
but MATCHED/CLOSED fields (subject, budget, requirements) mutate in place with no status change and
no re-approval — the economic context of an ACTIVE match can be rewritten. Fix: restrict update to
DRAFT|REJECTED|ACTIVE (409 otherwise).

**INT-108 — Favourites are physically deleted — founder soft-delete rule violation.**
Favourite Service implementation reference the delete by tutee id and tutor post id operation (verified) —
the only Java persistence mapping hard delete in the private implementation folder Violates the absolute rule ("every delete is logical").
Fix: removed at/status flag via migration, or record an explicit founder exemption for ephemeral
bookmark rows.

**INT-109 — Registration is an account/personal information enumeration oracle (email AND phone).**
Auth Service implementation reference: distinct 409 EMAIL ALREADY REGISTERED / 409 PHONE ALREADY REGISTERED.
The phone oracle is the sharp one — the 8-digit SG space is small; an attacker with someone's
number can confirm platform membership (rate-limited 5/hr/IP but IPs rotate). forgot-password is
correctly silent. Fix: keep the email 409 (user experience tradeoff) but fold the phone collision into a
non-distinguishing failure; consider per-account throttling.

**INT-110 — secure remote access private key lives in the working tree (dev Key/) inside the virtual machine rsync path.**
REDACTED key path is a plaintext RSA private key at the wrapper root; the virtual machine deploy
copies this project directory to the development machine, removing destination files absent from the source (SHARED-LESSONS 2026-06-06). Any rsync
without an explicit exclude copies the virtual machine's own access key onto the virtual machine (and into backups/). Fix:
move to REDACTED local path, add rsync excludes (dev Key/, .DS Store), rotate the key if it has ever
synced.

**INT-111 — Stale-closure location in use Init Auth — session-expiry redirect builds return To from the page the app first mounted on.**
use Init Auth implementation reference: handle Unauthorized registered in a  effect freezes location at
first render; any later 401 navigates to /login?return To=<initial-mount-path> — wrong post-login
destination, and a failed login POST clobbers a legitimate return To. Fix: read
window location.pathname + search inside the handler; exempt auth login from the logout
trigger (pairs with INT-095).

**INT-112 — Plaintext password lingers in session Storage when the tutor abandons the commission step via SPA navigation.**
Register implementation reference stashes email, phone, password and display name;
Commission implementation reference wipes only on beforeunload and on submit — SPA links (logo, About,
Login) fire neither. The credential survives in session Storage for the tab session. Fix: wipe in
the route-leave cleanup too, or stash everything except the password and re-prompt.

**INT-113 — Notifications page fetches only page 1, no pager.**
Notifications implementation reference — Page Response consumed with no page params and no pager user interface.
Older notifications are permanently unreachable ("Mark all as read" works, reading doesn't). Fix:
the Favourites implementation reference pager pattern.

**INT-114 — Creating an interview never invalidates the interview caches.**
Applicants implementation reference and Tutor Detail implementation reference POST
interviews create with on Success that only toasts — no the invalidate queries operation
(every other interview mutation invalidates). With stale Time: 30 000 a tutee who requests an
interview and opens "My Interviews" sees a cached list without it — looks like silent failure.

**INT-115 — Interview cards lack counterpart/job context — every card can show the same title (or none).**
The Interview data transfer object carries only tutor Post* fields (index implementation reference) — no job
subject/level, no anonymised counterpart handle. Interviews implementation reference titles each card
with the tutor's *own listing title* (3 negotiations = 3 identical cards);
Interviews implementation reference renders tutor Post Title with no fallback — blank for free-market
interviews. INT-039 negotiation becomes guesswork. Fix: add job subject/level + counterpart handle
to the data transfer object and cards.

**INT-116 — Interview cancel fires on first click with no confirmation; three inconsistent destructive-confirm patterns.**
Interview Detail implementation reference + Interview Detail implementation reference cancel
immediately (even SCHEDULED). Compare: close-job = design-system Modal, decline-invitation = Modal,
withdraw-application = window confirm (Applications implementation reference). Fix: one shared confirm
pattern; add to interview cancel; replace the window confirm.

**INT-117 — "Change my choice" / "Change choice" don't change anything; the modal's promise has no un-choose path.**
Applicants implementation reference "Change my choice" only the set chosen handle operation (panel dismiss; the
selection request stays CREATED); Applicant Card implementation reference "Change choice" re-opens the modal for
the *same* tutor; Choose Tutor Modal implementation reference promises "You can change your choice until our team
confirms" but no withdraw-selection endpoint exists. You can pick a *different* tutor, never return
to "no one chosen". Fix: wire a real withdraw-selection (backend+frontend) or relabel + soften the promise.

**INT-118 — Tutor's "Commission pending" is a money dead end — no how/when/where to pay.**
Matches implementation reference: a debt-looking badge + amount with no explanation; payment exists
only as the admin's mark-paid. Fix: one explanatory sentence + admin contact under the badge
("We'll contact you to collect the one-time placement fee after your first session — nothing to do
yet").

**INT-119 — Apply fallback failure is silent.**
Tutor Apply Control implementation reference — on a server TUTOR PROFILE INCOMPLETE
422 the sheet closes and the handler returns with no toast/banner. Stale cached profile → Apply →
sheet vanishes → nothing said (INT-004 class). Fix: show the Apply Gate Banner/toast routing to
/tutor/profile.

**INT-120 — Tutor dashboard "Complete portfolio" routes to the new-listing wizard → duplicate listings; competing onboarding prompts.**
Dashboard implementation reference → /tutor/listings/new (no listing-edit route exists, so following
it creates a duplicate listing), while Tutor Layout implementation reference (Profile Completion Banner) and the
apply gate both point at /tutor/profile. Two simultaneous prompts, different fields, different
destinations. Fix: point at editing the existing listing or fold into the single
profile-completeness model.

### S4 batches

**INT-121 — Backend polish batch (S4).**
(a) Interview duplicate-guard bypassable across branches: listing branch checks
tutee Id+tutor Post Id (Interview Service implementation reference), free-market branch checks
tutee Id+tutor User Id () — a prior free-market interview is invisible to the listing-path
check → two ACTIVE interviews with the same tutor (still capped 3/posting). Fix: both branches
check by tutor User Id. (b) Mixed the now operation (system TZ) vs the now operation on
persisted lifecycle timestamps — pay Commission sets paid At system-TZ two lines above a Coordinated Universal Time
match Completed At; also Commission created At default, Admin Post Service approve/reject,
Tutee Post Service, Admin Moderation Service. Standardize on Coordinated Universal Time / a Clock bean. (c) No-show actor is
a raw String (No Show Request the pattern operation, compared the equals operation at
Interview Service implementation reference) — the Timeslot Proposer enum exists; type it end-to-end (no-magic-
strings rule). (d) Admin schedule Interview accepts a past scheduled At
(Interview Service implementation reference) while tutee/tutor slots enforce ERR SLOT IN PAST (). (e)
Activate Match Request javadoc says override "≥ 0" vs the service's ≥ 1.

**INT-122 — Security hygiene batch (S4).**
(a) /actuator/metrics + /actuator/info readable by ANY authenticated user (only health is
permit All and prometheus is ADMIN in Security Config implementation reference; metrics/info fall through to
the any request operation ) — leaks GIT SHA, JVM/datasource/request internals to normal
users; gate as ADMIN like prometheus. (b) Contact-reveal 403-vs-404 existence oracle:
Contact Reveal Service implementation reference returns 403 for a non-participant on an existing match vs 404
for missing — inconsistent with the deliberate 404-everything hardening two branches below
(); return 404 for non-participants too. (c) A real Tink Aes Siv Key keyset + signed session token/keyed message authentication dev
secrets sit in docker-compose implementation reference in the rsync-to-virtual machine tree — dev-only by design, but ensure
the virtual machine uses a distinct keyset and the compose secrets never serve a shared env.

**INT-123 — Frontend a11y/code-hygiene batch (S4).**
(a) Nav "More" menu uses role is menu/menuitem without the APG keyboard contract; Escape/outside
close doesn't restore focus to the trigger; mobile drawer has no Escape (Nav implementation reference)
— drop to a plain disclosure or implement the full pattern. (b) Live regions mounted together with
their content (Contact Reveal Card implementation reference, Toast implementation reference) — first announcement may be dropped;
keep containers permanently mounted. (c) Input derives id from the label
(Input implementation reference) — two same-labelled inputs collide; the use id operation fallback. (d) Dead
code: dummy implementation reference (290 lines, zero imports), the private implementation folder (unimported), unreferenced svg
assets — delete. (e) Delete-account/change-password modal logic duplicated byte-identical between
Profile implementation reference and Profile implementation reference — extract (single fix point
for INT-095).

**INT-124 — user experience micro-defect batch (S3/S4).**
(a) Assignment Detail implementation reference hardcodes an "Open" badge that contradicts the page's
own "Position filled" (,202-211) — derive from post status. (b) Application "job title" link
→ /requests#<id> dangling hash (Applications implementation reference; no card ids, CLOSED jobs not
in the list). (c) Apply-with-listing selector can never appear: Tutor Apply Control implementation reference
treats read operation for tutor posts as Page Response and reads .data, but the endpoint returns a plain
array → listings always  — silently-dead feature. (d) Wrong-role navigation bounces silently
to / with no message (Tutee Layout implementation reference + mirrors). (e) Matches empty-states teach
the wrong funnel ("Complete an interview to get matched" — Matches implementation reference,
Matches implementation reference; matching comes from choose/assign, interviews are optional). (f)
Assign-confirm modal shows a false "other applicants" number: Matching implementation reference passes
selection Requests length - 1 (pending selections across ALL jobs) into
Close Job Assign Modal implementation reference's "Notify the other N applicants" (about THIS job);
Interview Queue implementation reference same.

---

## Part 2 — Areas verified clean (evidence in lane reports)

- **Authz core:** deny-by-default; no /public/** blanket permit All (exact GET paths only); dual-layer admin gating; ownership re-checked server-side on every the requested record's identifier mutation across all services. No IDOR found.
- **personal information:** public data transfer objects are records with no name/email/phone/user Id; contact crosses only at the participant-gated, audited reveal endpoint. No mass assignment (no data transfer object binds role/status/is Super Admin).
- **signed session token/cross-site request forgery protection/cross-origin request permissions/cookies:** iss+aud+exp+signature enforced; block-flag + password-epoch kill-switches; double-submit cross-site request forgery protection Same Site is Strict; explicit cross-origin request permissions origin list; 401-vs-403 handlers correct (INT-029/030).
- **Seeded admin + regression fixtures:** the profile operation only.
- **database query language injection:** all 8 native queries bind-parameterised; nothing string-concatenated.
- **Rate limiting:** all auth surfaces + collector + public assignments; trusted-proxy-aware IP resolution.
- **State machines:** approval strictly PENDING-gated; the three quota caps (interview 3/posting, apply 10/tutor, invite 10/job) all advisory-locked with partial-index backstops; atomic assign lock-ordering + uq matches tutor user pair backstop solid.
- **Transactions:** INT-016/087 discipline consistently applied (after Commit resolves primitives first; outbox enqueues in-tx; capture-before-detach on clear Automatically).
- **Liquibase:** all ALTER TYPE ADD VALUE in own run In Transaction is no changesets; no raw database structure definitions; numbering gaps deliberate.
- **frontend:** zero raw application programming interface paths outside API ROUTES; route guards + safe Return To correct; safe Uuid guarded everywhere (INT-081); Singapore time formatting consistent; invitations optimistic-update textbook-correct; global error-toast policy in place.

---

## Part 3 — user interface/user experience: can it be further improved? (founder question)

**Verdict: yes — meaningfully, and the headroom is NOT visual.** INT-049 did its job: tokens,
layout primitives, the four states, a11y and form craft are strong (Login/Register, the
choose-tutor flow, negotiation surfaces and quota chips are best-in-class for this codebase). What
the redesign could not fix is **narrative and symmetry across the funnel**: users are repeatedly
left in correct-looking states with no explanation of what happens next (rejection reasons,
commission pending, selection pending); one persona gets polish the other doesn't (parent reveals
contact, tutor can't; parent gets Success Panels, tutor gets badges); and the admin user interface was never
re-wired to the INT-034 assign contract, so the core parent-chooses-tutor journey dead-ends
(INT-090). The biggest win is the P1 cluster below — roughly one focused wave.

| # | Improvement | Why | Where | Effort | Priority |
|---|---|---|---|---|---|
| B1 | **Status-driven "what happens next" timeline on every artifact** — reuse Status Timeline/Success Panel on dashboard rows/detail (post → review → live → applicants → chosen → matched, + the tutor mirror) | The single biggest anxiety in an admin-mediated marketplace is waiting blind; INT-098/118 and the empty-state defects are all instances of one missing pattern: state without narrative | tutee/Dashboard, tutor/Applications, both Matches pages; Status Timeline exists | M | **P1** |
| B2 | **Close the M18 loop in the user interface** — INT-090 (assign) + INT-097 (tutor contact reveal) as one package | Completes the platform's revenue loop end-to-end for the default path; highest value per engineering hour right now | Selection Request Queue, Matching implementation reference, Matches implementation reference | S | **P1** |
| B3 | **Unify tutor jobs-browse**: one /assignments surface, role-aware (filters + pagination + apply control), retire Browse Requests | Fixes INT-101 structurally; tutors get search on their primary tab; one card system to maintain | Browse Requests implementation reference → Browse Assignments implementation reference + Tutor Apply Control | M | **P1** |
| B4 | **Label hygiene + "no raw enum" regression test** (INT-099) | Marketplace credibility on the first screen a parent sees; cheap; prevents the third regression of this class | Tutor Card, Tutor Detail, dashboards, Invite Tutor Modal | S | **P1** |
| B5 | **Notification deep-link coverage**: every counterparty-caused state change (superseded, invite declined, counter-proposed, approved/rejected **with reason**) lands as an in-app notification with a working link Path | The communication loop IS the product in an admin-mediated marketplace; INT-025 covered admins only; pair with INT-113 | Notifications implementation reference, backend Notification Type fan-outs | M | P2 |
| B6 | **Trust signals at decision points**: verified-tutor badge (admin-vetted is already true — show it), review aggregate on job/assignment cards, "admin-mediated, Personal Data Protection Act" microcopy beside choose/apply CTAs | Conversion psychology: trust where users hesitate (choose modal, apply sheet, contact reveal) | Applicant Card, Choose Tutor Modal, Apply Interest Sheet, Tutor Card | S | P2 |
| B7 | **Mobile bottom-nav continuity**: "Find Tutors" (a bottom tab) navigates to a Public Layout route and the bar vanishes | Disorienting on the role's own primary tab; complements INT-074 | nav Tabs implementation reference, Mobile Bottom Nav, Public Layout | S | P2 |
| B8 | **Onboarding consolidation**: one profile/portfolio completeness model, one prompt (fixes INT-120), first-week checklist on empty dashboards | Goal-gradient: tutors who complete profile + 3 applies in week 1 are the supply you keep; today three competing prompts dilute it | Tutor Dashboard, Profile Completion Banner, Profile Completeness Bar | M | P2 |
| B9 | **Saved-state recovery on register→commission wizard** (non-secret fields survive refresh; only the password re-entered) | The beforeunload wipe makes any step-2 refresh restart the funnel — measurable signup drop-off; pairs with INT-112 | Register implementation reference, Commission implementation reference | S | P3 |
| B10 | **Admin queue badges**: pending-review / pending-selections / unpaid-commission counts in the admin nav tabs | Operator efficiency — the operator persona currently polls five tabs | Admin Layout, nav Tabs implementation reference | M | P3 |
| B11 | **Empty-state truthfulness pass**: every empty state names the actual next step in the current funnel (fixes INT-124e and siblings) | Several empty states teach an outdated interview-mandatory model | all Empty State call sites | S | P3 |

B1–B4 ≈ one focused wave; B-items are enhancements (ROADMAP candidates, founder call), the
INT-090…124 rows are defects and live in the tracker.

---

*Lane agents: security-expert, software-engineer+dba, frontend-expert, uiux-designer (4 parallel
read-only general-purpose agents). Orchestrator verification pass: all S1/S2 evidence re-checked
against source before filing; INT-091 found during that pass.*
