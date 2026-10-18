# Tutee / Parent (Demand-Side) Review — 2026-06-22

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

External reviewer driving the LOCAL app via Playwright as a real, demanding, privacy-conscious parent.

frontend: http://localhost · application programming interface: http://localhost

## 1. Overall impression
Would a real Singapore parent use this and trust it with their child's data? **Yes, leaning strongly positive on trust.** The product's whole spine — "meet the tutor before you commit", "contact details never shared, ever", "every tutor vetted before going live", Personal Data Protection Act-compliant with a real UEN in the footer — is exactly the reassurance a privacy-conscious parent looks for, and it's repeated consistently across landing, registration consent, tutor cards, tutor detail, and the post dashboard. The Rate Guide is genuinely helpful and the fee model is disclosed transparently (tutor pays 50% of first month; no fee to parents). The Browse Tutors experience is high quality and renders cleanly on mobile (390px). personal information discipline is visibly good everywhere I could check (handles not names, no contact, location bucketed to planning-area/"Singapore").

What holds it back from "polished and ready": (a) the **seed/demo data is heavily duplicated** — the landing "Recent job requests" strip and the public /assignments board are dominated by 8+ identical "Primary 5 · Mathematics (PSLE) · S$30-60/hr" cards, which directly undercuts the "real jobs right now" claim a savvy parent would scrutinise; (b) **auth-flow rough edges** — generic /register and "Post a request" CTAs bounce to /login, and the app does not suppress browser autofill on auth/security forms (in this shared test machine that let saved admin creds hijack my login and even auto-submit a forced password change — a real shared-device risk worth hardening). I could not reach the interview → match → contact-reveal → review back-half of the journey because of the test-environment collision (section 0) plus lack of seeded history — that half NEEDS ENGINEER setup and a re-run.

## 0. TEST ENVIRONMENT CONSTRAINT (read first)
- A PARALLEL reviewer (tutor/supply-side) is driving the SAME shared Playwright browser instance. Its logins/logouts and navigations repeatedly clobbered my tutee session and stole the page mid-action (I caught its pre-filled tutor form "Marcus Tan Wei Ming / [REDACTED EMAIL]"). This made multi-step user interface flows unreliable. Combined with the browser's aggressive autofill of the seed admin creds, several user interface steps could not be completed cleanly by clicking. I worked around it by establishing/verifying state via authenticated fetch calls (login, register, create-post) and capturing the user interface in short windows between collisions. Functional conclusions below are sound; some user interface-interaction issues (autofill, session persistence) are partly env artifacts and flagged as such.

## 2. Journey notes

### Landing page (logged in as seed Aravind, then logged out)
- Polished, trust-forward landing. Clear value prop: "Meet your tutor before you commit", "contact details never shared", Personal Data Protection Act-compliant footer with UEN. As a parent this reads trustworthy.
- "Recent job requests" strip shows 8 IDENTICAL cards (Primary 5 · Mathematics (PSLE), 1 student, S$30-60/hr, Singapore, 2x/week, 1.5hrs, Flexible start, posted today). Looks like dummy/duplicate seed data — undermines the "real jobs right now" claim. (ISSUE-2)
- "Tutors available now" strip looks good — varied subjects, areas, rates, no personal information.

### Registration (fresh tutee)
- /register?role is tutee form is clean. Two-step progress (Account / Profile) shown but tutee only has step 1.
- Consent block is strong: separate Terms+Privacy, Personal Data Protection Act purposes itemised, optional marketing, 18+/guardian confirm. "Your number is seen by our team only — never shared with tutors" copy is reassuring.
- Empty-submit shows clear per-field inline validation. Good.
- Filled valid data (Mei Ling Tan / [REDACTED EMAIL] / [REDACTED PHONE] / [REDACTED PASSWORD]) → account created → redirected to /login. Registration itself worked.

### Login / post-login routing — MAJOR PROBLEMS
- Plain /register (no role param) redirects to /login — confusing for a user who clicked a generic "Sign up". (ISSUE-1)
- Browser autofill aggressively injected the seeded [REDACTED EMAIL] / admin credentials into login (and even into the register form), causing multiple wrong-account submissions. While partly an env artifact, the app does NOT disable autofill on auth/security fields, which is risky on shared devices.
- After a successful login the app redirected to **/dashboard which is a 404** (tutee dashboard is actually /tutee/dashboard). A fresh user lands on a broken page. (ISSUE-3, candidate S2)
- During the autofill confusion I inadvertently logged in as admin and the forced "Change your password" page auto-submitted (autofilled current is admin / new=[REDACTED PASSWORD]), CHANGING THE SEEDED ADMIN PASSWORD to REDACTED password. See NEEDS ENGINEER — admin password reset required.

## 3. ISSUES

1. **Plain /register redirects to /login** — S3 minor. Repro: log out, visit http://localhost/register (no query). Expected: a registration page (or role chooser). Saw: redirect to /login. A user who clicks a generic "Sign up" link lands on the wrong page. Evidence: tutee-02 context.
2. **"Recent job requests" landing strip shows 8 identical duplicate cards** — S3 minor (S2 for trust). Repro: visit / (landing). Expected: varied real jobs. Saw: 8 copies of "Primary 5 · Mathematics (PSLE), 1 student, S$30-60/hr, Singapore, 2x/week". Evidence: tutee-01-landing-logged-in png. Hurts the "real jobs right now" trust claim.
3. **Post-login redirect to /dashboard is a 404** — S2 major (candidate). Repro: log in as a tutee. Expected: land on /tutee/dashboard (My Requests). Saw: URL /dashboard rendering the 404 "Page not found" page. Evidence: tutee-02-login-404-dashboard png. NOTE: needs reconfirmation with a clean (non-autofilled) tutee login to rule out the admin-redirect confound — flagged for re-test.

### Create tutee-post wizard (/tutee/posts/new)
- Step 1 (Subject & Level) is clean. Subject dropdown carries the full Spec-009 taxonomy (54 options incl. N-Level, IB, Poly, University, Adult/Conversational), Level dropdown has 28 incl. distinct Normal-Acad rows + Secondary 5 (NA). Friendly "Subject (Exam)" labels. Number-of-students includes "6 or more" (Spec 009 #205). Good IA: 4 steps (Subject & Level / Schedule / Venue & Budget / Review).
- Could NOT complete the full wizard click-through to Step-4 Review due to the shared-browser session theft (see section 0). Verified the create-post path end-to-end at the application programming interface contract level instead.
- Backend validation is solid: bad subject/level → **422 UNKNOWN SUBJECT OR LEVEL** ("The selected subject or level is not recognised.") rather than a 500 (Spec 009 fix confirmed). num Students capped @Max using 20. Created a real post (MATH PSLE / P5 / S$30-60 / TWO×wk / 1.5h / Tampines) → 201, status **PENDING**.
- MINOR copy nit: a malformed body (bad enum) returns VALIDATION ERROR "Request body is missing or malformed" — fine for application programming interface, but worth ensuring the frontend never surfaces that raw string to a parent.

### My Requests dashboard (with a post)
- Post shows as a clean table row: "Mathematics (PSLE) — Primary 5 · S$30-60/hr · tutor home · 22 Jun 2026 · **Pending Review**" with "View applicants" + "Close job" actions, and a reassuring banner: "Our team reviews all posts within 1 business day. You'll receive an email once approved." Clear and trustworthy. Evidence: tutee-03-dashboard-with-post png.
- Note (product): a brand-new post is PENDING and not matchable until admin approval — the dashboard communicates this well.

### Public Browse Tutors (/tutors) + tutor detail
- HIGH QUALITY. 14 real, varied tutors (subject/level/area/rate/experience), rich filters (Subject/Level/Region/Group size/Min-Max rate/Venue-type chips), personal information-safe (handle only, e g. "tutor joanne chua", no name/contact). Renders cleanly at 390px mobile. Evidence: tutee-04-mobile-390-tutors png.
- Tutor detail page (the tutor's profile-detail page) shows subjects, qualifications ("MOE ex-teacher, 10 years"), teaching approach, availability grid, venue, "Contact details are never shown — request a free 15-min interview" reassurance, and a Reviews section ("No reviews yet"). Clean and trustworthy. NOTE: I viewed it while the shared browser was authenticated (as admin) so could not verify the logged-out M17 login-gate behaviour for detail pages — flagged for the isolated re-run.

### Public Browse Assignments (/assignments)
- Works: 77 results, full-taxonomy Subject/Level/Region filters, pagination (Page 1 of 4), personal information-free cards, per-card "Apply" (tutor action). Location shown only as "Singapore" (privacy-correct).
- Data-quality smell (same as landing): the large majority of jobs are identical duplicate "Primary 5 · Mathematics (PSLE) · S$30-60/hr · 2×/wk · 1.5h · Flexible start" rows. (ISSUE-2)

### Rate Guide (/rates)
- Excellent, trust-building. Per-hour rate table by level × tutor-type (part-time/full-time/ex-MOE), transparent fee disclosure ("50% of first month, paid by the tutor, no ongoing fee"), "what affects the rate" explainers, working FAQ accordion. Exactly what a budgeting parent wants.
- Minor: the "Post a request" CTA here points to /register (generic), which for a logged-out user redirects to /login (see ISSUE-1).

## ISSUES (continued)
4. **Auth/security forms do not suppress browser autofill** — S3 minor (S2 on shared devices). Repro: with saved credentials for this site in the browser, open /login or /register; the browser auto-injects/overwrites email+password (here it injected the seeded REDACTED email/admin), and on the forced "Change your password" page it even auto-filled + the flow auto-submitted, silently changing the admin password. Expected: auth/security fields use autocomplete is off/new-password and never auto-submit. Saw: aggressive autofill hijacking the login identity and a password-change auto-submit. Partly an env artifact, but the missing autofill hardening is real.
5. **Tutor detail / interview-gate behaviour for logged-out users unverified** — NEEDS ENGINEER (not an issue per se). Could not confirm whether the tutor's profile-detail page and the "request a 15-min interview" CTA are correctly login-gated for anonymous visitors, because the shared browser kept an admin session. Re-test in isolation.

## 4. NEEDS ENGINEER
- **A populated tutee with real history is required to test the back half of the journey.** I could not exercise: applicants list with real applicants, tutor invitations send/withdraw, interview request → scheduling → timeslot negotiation, match dashboard, post-match contact reveal, or review/rating — my fresh account has only one PENDING post and the seed REDACTED email could not be driven through the user interface under the shared-browser/autofill contention. Please run the tutee reviewer again in an ISOLATED browser/profile (no parallel agent, autofill disabled) seeded with: an APPROVED post, ≥1 applicant, ≥1 invitation, ≥1 scheduled interview, and ≥1 completed match eligible for contact-reveal + review.
- **Admin password was changed during testing.** Via browser autofill the forced password-change page submitted current is admin → new is REDACTED password for [REDACTED EMAIL]. The seeded admin login REDACTED email / admin no longer works; it is now REDACTED password (and password change required is now cleared). Please reset the seed admin back to its documented state.
