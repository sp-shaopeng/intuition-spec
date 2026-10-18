# Tutor (Supply-Side) Reviewer — InTuition SG — 2026-06-22

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

External reviewer driving the LOCAL app (frontend  / application programming interface ) via Playwright as a real, price-sensitive,
time-poor tutor. Goal: is it obvious how to GET matches and GET PAID, and is the apply→interview→match→pay
funnel smooth?

Accounts:
- Fresh tutor registered this session: REDACTED email / REDACTED password
- Seed populated tutor: REDACTED email / REDACTED password

---

## 1. Overall impression

As a price-sensitive, time-poor tutor, the supply-side product is genuinely appealing ONCE you're in: the
"get matches without cold calls + we never share your contact" promise is consistent everywhere, the
apply→pipeline flow is smooth, and — crucially for a tutor deciding whether to bother — the commission is
stated plainly on the Rate Guide: "50% of the first month's tuition, one-time, no ongoing platform fee, fees
paid directly family↔tutor." That feels fair and clear. The profile/listing builders are well-designed
(quick-set rate bands, grouped chips, a genuinely usable mobile availability grid with a sticky day column +
swipe hint at 390px). The profile-completeness meter ("Add your area to start applying") is a nice nudge.

BUT the experience is undermined by a serious auth defect (T-02): once any prior session's refresh token is in
the browser, the app silently resurrects that identity and hard-redirects you out of whatever you're doing —
this repeatedly destroyed my in-progress tutor REGISTRATION and even flipped a freshly-logged-in tutor back to
an admin mid-action. A real first-time tutor on a shared/kiosk machine, or anyone who logs out to make a second
account, could be silently logged in as someone else. That's both a broken funnel AND a privacy/authz problem.
The second friction point is T-07: clicking Apply with an incomplete profile fails with a bare 403 and ZERO
feedback — the most important money-making button silently does nothing, with no hint to finish your profile.

Net: strong, trustworthy supply-side design with a clear/fair commission story, but the session-revival bug is
a ship-blocker for the registration funnel and the silent apply-403 will quietly lose applications.

---

## 2. Journey notes

### Pre-flight / session hygiene
- On first load the browser carried a STALE authenticated session (first a tutee "Aravind Kumar", later an
  ADMIN REDACTED email with password Change Required is yes). Logging out via the app cleared it. This is a
  test-harness artifact, not necessarily a product bug, but see ISSUE about /dashboard 404 below which is real.

### Registration (tutor, 4-step wizard: Account → Profile → Terms → First listing)
- Register page (/register?role is tutor) Step 1 = Account: name, email, +65 phone, password (with rules),
  confirm, and 3 required consents (Terms, Personal Data Protection Act, Age) + 1 optional marketing. Good: phone help text
  "seen by our team only — never shared with tutors." Consent user experience is clear and Personal Data Protection Act-itemised.
- Step 1 form validates well; phone normalised ([REDACTED PHONE] → [REDACTED PHONE]). Continue advances to /register/profile.
- Step 2 = Profile, headed "Tell us about yourself — All optional — add what you can now and finish the rest
  on your profile later. This is what parents see." GOOD onboarding-optional user experience (INT-043). Fields: hourly-rate
  quick-bands (S$20-30 / 30-40 / 40-60 / 60+) + min/max numeric; Commitment (Part/Full-time); Area
  (Central/North/South/East/West) + Planning-area chips incl. "Island-wide" (30 areas); Subjects & levels
  (marked * required — see user experience nit T-03); Highest qualification; Years of experience. "Skip for now" present.
- BLOCKER: could not complete tutor registration end-to-end because of the silent-refresh session-revival bug
  (T-02). Mid-form on Step 2, a stale admin session revived and hard-redirected me to /admin/review/`<id>`,
  destroying the in-progress registration. Reproduced 4+ times. See T-02. This is the #1 supply-side blocker.

#### user experience nit (Step 2): heading says "All optional" but Subjects & levels is * required
- Minor copy/validation inconsistency — the section header promises everything optional yet one field is starred.

### Tutor profile / onboarding completion (banner) — via seed account REDACTED email
- Pivoted to the seed tutor to get past T-02. Tutor dashboard ("My Listings") shows the INT-043 profile-
  completion banner: "Complete your profile to start applying — Still needed: your area." with
  "Complete profile →" and a Dismiss (×). VERIFIED + good.
- Dashboard: 1 Live listing (PSLE & Lower-Sec Math, S$28-45/hr); a "Public showcase" featured-consent switch
  (ON) and per-listing "Currently accepting students" switch (OFF) with clear copy on what's needed to appear
  in the public showcase. Privacy reassurance throughout ("contact information is kept private").
- Profile page /tutor/profile: tabs Profile / Security / Reviews / Account. Shows "Profile completeness 83%"
  with a progress bar + "Add your area to start applying." Setting Area is East → "✓ Profile saved", completeness
  100%. Public-handle field with good privacy hint; rate range; multi-MRT planning-area chips (Island-wide
  option); per-subject level chips. Strong, well-guided.
- Note: saving the profile also fired a "Sending your verification code…" toast (WhatsApp one-time passcode, INT-082) even
  though WhatsApp is meant to be OFF — see user experience nit T-05.

### Browse assignments + apply/withdraw (free-market) — WORKS
- /assignments: 78 results, paginated (4 pages), filters Subject/Level/Region, personal information-free cards, per-card Apply.
- Detail page: budget band, frequency, session length, area, venue (Online), strong "Contact details are never
  shown" reassurance, "Apply / Express interest".
- Apply dialog "Express interest": optional pitch (0/1000 counter), "Attach a listing" dropdown (None is use
  profile, or a specific listing — explains it swaps rate/area shown to parent), clear "Applying does not book
  the job". Sent → button becomes "✓ Interest sent". Clean.
- My Pipeline /tutor/applications: shows applications with pitch + applied date + status; **application quota
  "7 of 10 applications left"** (a cap — good to surface). Withdraw → confirm dialog ("You can re-apply later")
  → status flips to "Withdrawn", quota restored 7→8. Apply→withdraw cycle works end-to-end.
- One seed application shows "Interview scheduled" + "View interview →" — used to reach the interview flow next.

#### user experience nit T-04 · Desktop nav collapses to a hamburger; main links only in a bottom "Primary" nav
- On /assignments, /tutor/* at desktop width the top nav shows only logo + "Toggle menu" (hamburger); the
  Find Work / My Pipeline / Interviews / Matches links live in a bottom mobile-style nav. The dashboard
  (/tutor/dashboard) DOES show the full top nav — inconsistent. Looks like a responsive breakpoint bug on
  the assignments/profile layouts. Severity S3.

#### user experience nit T-05 · Profile save triggers a WhatsApp "Sending your verification code…" toast while WhatsApp is OFF
- Confusing for the tutor (no WhatsApp expected). Likely the one-time passcode feature-flag isn't gating the frontend toast.
  Severity S4 (cosmetic/trust).

#### user experience nit T-06 · Apply-rail bottom "Apply →" button persists after applying
- After "✓ Interest sent", a secondary bottom-rail "Apply →" button still shows — mild confusion. S4.

---

## 3. ISSUES

### ISSUE T-01 · Navigating to /dashboard (no role prefix) renders a 404 "Page not found"
- Severity: S3 (broken internal route / likely a redirect target)
- Repro: While authenticated, navigate to http://localhost/dashboard.
- Expected: redirect to the role-appropriate dashboard (e g. /tutor/dashboard) or to login.
- Saw: hard 404 "Page not found". If any post-auth redirect ever points at /dashboard, users hit a dead end.
- Evidence: review-2026-06-22/evidence/tutor-01-dashboard-404.png

### ISSUE T-02 · Stale refresh token silently revives a logged-out session and HIJACKS the current flow (S1)
- Severity: S1 (blocks registration + onboarding; data-integrity/security concern — wrong user resurrected)
- Repro:
  1. Be logged in as user A (e g. admin), then log out via the app's "Log Out" button.
  2. auth/me flips to 401 momentarily and the logged-out nav shows — BUT the host-only refresh token
     cookie is NOT reliably revoked by the frontend logout click (it races an in-flight silent refresh).
  3. Navigate to /register?role is tutor (or /login, or start any auth flow). Within ~1s the app's silent
     token-refresh redeems the surviving refresh token, auth/me flips back to 200 as user A (or an even
     older session, e g. a tutee), and the router hard-redirects you to that user's home
     (/admin/review, /tutee/dashboard, /tutee/posts/new).
  4. This fires MID-FORM: while filling tutor registration Step 2 (Profile), the revived admin session yanked
     the page to /admin/review/`<uuid>`, destroying all entered data.
- Expected: after logout, the refresh family is revoked; navigating to an auth page shows the form and lets a
  new user register/log in. No silent resurrection of a previous identity.
- Saw: previous sessions repeatedly resurrected; registration/onboarding could not be completed through the user interface.
- Notes / root-cause signal: calling submit operation for auth logout directly WITH the X-XSRF-TOKEN header DOES
  revoke the family (subsequent submit operation for auth refresh → 401, auth/me → 401). So the backend revocation works;
  the defect is that the frontend "Log Out" button does not reliably complete revocation before the next silent
  refresh fires (a logout-vs-refresh race), leaving a live refresh token that auto-resurrects the session.
- Impact: a shared/kiosk browser, or simply a user who logs out and then tries to register a second account,
  can be silently logged back in as the PREVIOUS user — a real privacy/authz problem on top of the broken flow.
- Evidence: review-2026-06-22/evidence/tutor-02-hijack-admin-review png (mid-registration redirect to admin)
- Deeper repro detail (confirmed): with a SINGLE auth cookie in the jar, a fresh login as Wei Jie
  (/me = "Mr Tan Wei Jie") spontaneously flipped to "Admin" 2.5s later with NO user action — the SPA's
  background silent-refresh loop redeemed the surviving admin refresh token and overwrote the auth cookie.
  Only a full cookie-jar clear + SPA teardown (about:blank) + reload produced a stable clean session.
  Net: once any stale refresh token exists in the browser, it keeps resurrecting that identity over the
  current one. This is the supply-side test's single biggest impediment and a real authz/privacy risk.

### ISSUE T-07 · Applying with an incomplete profile fails with a generic 403 and NO user feedback (S2)
- Severity: S2 (silent failure on the core money-making action)
- Repro: as a tutor whose profile is <100% (missing area), click "Apply" on a job card on /assignments.
- Expected: a clear inline message like "Complete your profile (add your area) to apply" — ideally the Apply
  button is disabled/links to the profile when the profile is incomplete.
- Saw: submit operation for tutor applications → an access-denied response with status 403; the
  list card just resets, NO toast, NO error, nothing happens. The tutor has no idea why their apply vanished.
  (Once area was added → profile 100% → the same apply succeeded.)
- Notes: the gate is correct (profile-completeness), but the frontend shows the Apply button as active and gives a
  raw 403 with no explanation. The error is also a generic "Access denied" rather than a domain reason.
- Evidence: review-2026-06-22/evidence/tutor-03-dashboard png (banner "Complete your profile to start applying")

### ISSUE T-08 · "Interview scheduled" badge in My Pipeline, but Interviews page + application programming interface are empty (S3)
- Severity: S3 (data/user experience inconsistency; tutor told an interview exists but can't find it)
- Repro: as REDACTED email, open My Pipeline (/tutor/applications). The "P6 Math tutor needed
  (Tampines)" application shows status "Interview scheduled" and a "View interview →" link to /tutor/interviews.
- Saw: /tutor/interviews renders empty state "No interviews scheduled"; read operation for tutor interviews
  returns an empty result list with a total count of zero. The two views disagree.
- Expected: if an application is "Interview scheduled", the interview should appear on the Interviews page;
  otherwise the badge/link should not claim a scheduled interview.
- Evidence: review-2026-06-22/evidence/tutor-05-interview-inconsistency png

---

## 4. NEEDS ENGINEER

The seed tutor REDACTED email has NO active match and NO live interview, so these supply-side stages
could not be exercised. Please set them up and send me back:

1. **A live interview for the tutor** in a state I can act on — so I can test accept / propose-timeslot /
   counter / decline-with-reason / the negotiation→PENDING SCHEDULING→scheduled flow (INT-039). Today
   read operation for tutor interviews returns empty for Wei Jie even though an application is badged "Interview
   scheduled" (see T-08), so the interview-action user interface is unreachable.
2. **A tutee→tutor job invitation** addressed to Wei Jie (INT-044) so I can test invite accept/decline and that
   it folds into the apply→choose funnel.
3. **An ACTIVE match** for Wei Jie (admin creates/activates a match) so I can test the Tutor Match dashboard,
   and the **post-assignment contact reveal** (read operation for matches id contact).
4. **A commission to pay** on that match so I can walk **commission payment → match completion** and confirm
   what the tutor sees/owes (and that the "50% of first month" amount is shown).
5. **A completed match + a tutee review** so I can verify **reviews received** in the profile Reviews tab and
   on the **public tutor detail** page.

Also for a clean re-test of REGISTRATION (T-02): I need the browser session fully cleared OR T-02 fixed —
a fresh tutor registration could not be completed end-to-end because the stale-session revival kept hijacking
the wizard. (Backend revocation works when logout is called with the XSRF header; the frontend logout races a silent
refresh.) A fresh tutor account REDACTED email was NOT successfully created.

---

## 5. What worked well (positives)
- Commission clarity + fairness: Rate Guide states 50%-of-first-month one-time, no ongoing fee — answers the
  tutor's "what's the catch" directly. Rate tables by level × tutor-type are useful for pricing.
- Apply/withdraw free-market flow is smooth: express-interest dialog with optional pitch + attach-listing,
  "✓ Interest sent", My Pipeline with an application quota (10), withdraw-with-confirm + re-apply allowed.
- Profile/listing builders: rate quick-bands, grouped subject/level chips, multi-MRT planning areas, and a
  mobile-usable time×day availability grid (sticky day column, swipe hint, 1h/2h modes, quick-set shortcuts).
- Profile completeness meter + "Add your area to start applying" nudge; INT-043 completion banner present.
- Consistent privacy/trust messaging ("contact details are never shown", admin-coordinated) across every
  supply-side screen. personal information-free job cards. 390px mobile rendering is clean on assignments + profile + grid.

## 6. Coverage map
- DONE: login (seed tutor), profile-completion banner + profile edit (rate/area/multi-MRT/subjects) + save,
  browse assignments (+filters/pagination), assignment detail, apply (express-interest + pitch), My Pipeline +
  quota, withdraw + re-apply, listing-creation Step 1-2 incl. availability grid, Reviews tab (empty), Rate
  Guide + commission, 390px mobile pass, public-showcase/featured + accepting-students toggles (viewed).
- BLOCKED/NEEDS-ENGINEER: fresh registration end-to-end (T-02), interview accept/propose/counter/decline,
  invitation accept/decline, match dashboard with a real match, commission pay → completion, contact reveal,
  reviews received, public tutor detail with reviews.
