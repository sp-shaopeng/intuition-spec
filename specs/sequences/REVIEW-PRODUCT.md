# Product Verification — Sequence Diagrams

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Reviewer:** Product Manager / Business Analyst (Priya, demand-side advocate)
**Scope:** All 27 sequence diagrams under specs/sequences/
**Cross-checked against:** product requirements document md v1.3 (2026-05-27), business requirements document v1.2
**Date:** 2026-05-29

---

## Verdict

**CONDITIONAL PASS.** The diagrams are exceptional at engineering accuracy — every flow ties back to a controller line number, every gap is honestly called out. That is praiseworthy.

**But as a product story, they fail in three load-bearing ways:**

1. **The privacy promise — the #1 marketing claim in the product requirements document — is never visualised.** No diagram contains an explicit "Note: phone and email are NOT in this response" annotation. The product requirements document says "contact info never shared" is the brand's USP (product requirements document §1.1, BR-001, AC-01, AC-06). The diagrams quietly *omit* email/phone from response payloads but never *show* the omission. A reader who has not read the product requirements document will not realise that the tutee discovery flow (implicit in flow 23) and the tutor browse flow are honouring a load-bearing privacy guarantee. **This is a missed opportunity to make the product's defining feature visible in the architecture.**

2. **The 15-minute interview promise (BR-012, N-22, N-23) appears NOWHERE in flows 13, 14, 15, or 16.** This is a stated USP and a key trust signal. The notification body text in the diagrams is paraphrased to "Your interview for X is scheduled for Y" without the "15-minute" language the product requirements document mandates.

3. **Three diagrams describe silent failures that will mislead users:**
   - **Flow 03 (admin multi-factor authentication)** — admin login currently surfaces "Invalid email or password" because frontend is not wired to the multi-factor authentication branch. An admin will believe their password is wrong. The diagram calls this out, but no remediation flow is shown.
   - **Flow 12 (match cancel)** — neither party receives a notification. The dashboard will eventually update, but the tutee whose match has been cancelled has no idea their post is live again and ready for new matches. They will think they are still matched.
   - **Flow 26 (email fanout)** — the consumer for the email-notifications Redis stream does not exist in-tree. **Every email in the system is being enqueued into a queue that nothing reads.** This includes the forgot-password one-time passcode — i e., **password reset is silently broken at the product level**, even though every diagram from flow 04 onwards shows the one-time passcode being delivered.

The diagrams are technically truthful about these gaps. But the product story they tell is: "users will perceive failure as success." That has to be fixed before launch.

---

## Per-flow product feedback

### Flow 01 — Tutee Registration

- **Trigger clarity:** Good — "User selects 'I need a tutor'" is plain-English.
- **User-facing outcome:** Lands on /tutee/dashboard. Good.
- **Privacy moment missing:** product requirements document §4.1 step [2] says "On first login, tutee shown one-page privacy explainer… 'Your contact details are never shared with any tutor.'" **This step is not in the diagram and not in the code path.** A non-engineer reading this diagram would not know the platform makes that promise to the user at this moment.
- **Compliance user experience missing:** Personal Data Protection Act consent is mentioned only inside the zod validate line ("pdpa Consent"). Where is the consent text? Is it logged? product requirements document §3.10 demands "Personal Data Protection Act-compliant consent language at registration" with audit trail — the diagram does not show consent being logged as an audit event (only USER REGISTERED).
- **Trust-building moment missing:** product requirements document §9 N-01 says "We've received your post — under review." There is no welcome email on registration. The note "no welcome email is queued today (gap)" is honest but the product story is: **a brand-new user gets zero confirmation in their inbox that their account exists.** That hurts trust on Day 0.

### Flow 02 — Tutor Registration (two-step)

- **Trigger clarity:** Good.
- **Commission disclosure moment:** Strong. The diagram correctly shows the commission acknowledgement is logged before the auth cookie is issued. This is the BR-009 / AC-03 promise made visible. Praise here.
- **Privacy moment missing:** Same as flow 01 — no first-login privacy explainer.
- **session Storage stash:** OK from an engineering view, but as a product story, an event like "tab closed mid-flow" loses the user's qualifications + years experience. product requirements document does NOT mandate persistence here, so this is acceptable, but the failure mode "tab closes → stash wiped → user must re-fill" should be acknowledged.
- **Missing user-facing outcome:** Tutor lands on /tutor/dashboard without seeing their listing yet (because listing is a separate step in product requirements document §4.2 [3]). Diagram does not make it clear that the *next* expected action is "create a listing" — the user might think registration alone made them findable.

### Flow 03 — Login with multi-factor authentication

- **Trigger clarity:** Good.
- **CRITICAL gap is explicitly called out:** the frontend does not yet handle confirmation that multi-factor authentication is required. **An admin login currently fails with "Invalid email or password."** This is a launch-blocking bug for the admin role.
- **Account lockout messaging:** Diagram shows 423 ACCOUNT LOCKED with Retry-After. What does the user actually see? "Account locked, try again in 15 minutes" or just an web request protocol code? product requirements document does not specify the user-facing copy. Flag for user experience.
- **multi-factor authentication attempt counter messaging:** After 3 wrong TOTPs, user sees 429 multi-factor authentication ATTEMPTS EXCEEDED. The user-facing copy is not specified. Will admin be told they must restart the login flow, or just get a generic error?

### Flow 04 — Forgot/Reset Password

- **Trigger clarity:** Good.
- **No-enumeration design is correct** — forgot always returns 200. Good security user experience.
- **CRITICAL silent failure:** The one-time passcode email is enqueued to a Redis stream with **no consumer in the tree** (per flow 26). **The user will never receive the one-time passcode.** They will see "If your email exists, you'll receive a code" and wait forever. From a product POV: **password reset is broken.**
- **Reset success outcome:** Diagram shows the navigate operation after success. There is no success toast specified — does the user see "Password reset, please log in"? Or just land on a fresh login screen? Recommend an explicit toast.

### Flow 05 — cross-site request forgery protection Bootstrap + /me Hydration

- **Trigger clarity:** Good engineering note. From product POV, the user perceives nothing here — they just see the app load. That is correct and acceptable.
- **Hidden user impact:** The 401 → /login?return To is ... flow IS the user-facing moment. The diagram shows it. Good.

### Flow 06 — Tutee Post Creation

- **Trigger clarity:** Good.
- **User-facing success:** "success toast, redirect to dashboard / detail." Acceptable but vague. product requirements document §9 N-01 mandates "Your post has been received. We will notify you within 2 business days." — is that the toast text? Or is it an email? **The diagram says no email is queued, no notification to admin is emitted, and no audit log is written.** The tutee creates a post and sees… a toast. That is thin for a user about to wait 2 business days.
- **Trust-building moment missing:** product requirements document §4.1 step [3] says dashboard should show "Post Under Review" badge. The diagram doesn't show this badge state transition.
- **service-level agreement promise missing:** "We will notify you within 2 business days" (product requirements document N-01) is the trust signal. It must be visible at this step or in the post-submission email.

### Flow 07 — Tutor Post Creation

- Symmetric to flow 06. **Same gaps:** no email confirmation, no admin notification, no service-level agreement message to user.
- **Tutor-specific concern:** product requirements document §4.2 [3] says "Email: 'Your listing has been received.'" — this email does not exist.

### Flow 08 — Admin Approval

- **Trigger clarity:** Good.
- **User-facing outcome on approval:** Email is enqueued via POST APPROVED template. The diagram says "Email body intentionally contains only a dashboard URL — never personal information or contact details." **Good — but the body is so minimal it may not feel celebratory.** product requirements document N-02 mandates "Your post has been approved and is now active. Tutors can view your request (anonymised)." This subject + body must be in the template.
- **Rejection messaging:** The diagram shows the rejection reason flowing through. **But does the email render the human-readable label** (e g., "Incomplete information") or the enum code (INCOMPLETE INFO)? product requirements document §6.4 says "Email: 'Your post could not be approved. Reason: [selected reason].'" — clearly the *label*, not the code. **Flag for quality assurance: confirm email template substitutes label, not enum.**
- **Trust-building moment:** product requirements document says "Path B: Revision Requested" exists. The diagram does NOT show the revision-requested action — only approve and reject. This is a gap vs product requirements document §6.3 and §3.4. **Either the feature is missing from the code, or the diagram is incomplete.**

### Flow 09 — Admin Match Creation

- **Trigger clarity:** Good.
- **Critical product moment captured:** "Show success ('No contact details have been shared yet')" — this is the privacy promise made visible in the user interface. **Excellent.** This is the kind of annotation that should appear in every privacy-sensitive flow.
- **User-facing outcome:** Tutee and tutor get an in-app notification via Notification Service emit. **No email.** product requirements document N-08 and N-09 explicitly call for email. **Match creation — the most exciting moment in the whole funnel — produces only a bell ping.** That is a serious gap. product requirements document says "Exciting news — a match has been found!" via email.
- **Conversion funnel concern:** This is the climax of the funnel (post → match). Producing only a bell ping (which the user sees only if they're logged in within 60s of activation) loses the re-engagement loop. If the tutee is not on the app, they will not know a match exists.

### Flow 10 — Admin Match Activation

- **Trigger clarity:** Good.
- **User-facing outcome:** Bell notification only, no email. Same problem as flow 09. product requirements document N-10 and N-11 mandate email here. **N-10 is the first-session confirmation prompt that gates the entire commission funnel.** Sending it only as a bell ping means most tutees will never confirm, breaking the revenue model.
- **Tutee first-session confirmation step missing entirely:** product requirements document §4.1 [8] says the tutee should receive an email with a "Confirm First Session Occurred" button linking to first-session confirmation for the specified match. **Neither the email nor the confirmation page nor the 7-day timeout job appears in the diagrams.** This is a P0 feature (AC-05). The 7-day timeout job (product requirements document §7.7.1) is missing from the sequences.
- **admin Contact Email revealed:** Good — the diagram correctly notes that REDACTED email is revealed on ACTIVE. But what does the tutee actually *see*? A new card on their dashboard? A modal? No user-perception step is shown.
- **Trust moment captured (good):** "Your match is active. You can now begin lessons." — this is what the bell notification says. That is the right message.

### Flow 11 — Admin Marks Commission Paid

- **Trigger clarity:** Good.
- **Tutee is silent:** Diagram explicitly notes "Only the tutor is notified — the tutee is not notified on commission payment." From a product POV, this is fine. The tutee doesn't care about admin's collection. **But:** match transitions to COMPLETED, which unlocks reviews. The tutee should be told "Your tuition is now complete — leave a review!" That re-engagement loop is missing. product requirements document §3.9 P1 in-platform inbox is the right place.
- **Tutor message:** "Your commission of SGD `<amount>` has been marked as paid." Clear and good.

### Flow 12 — Admin Match Cancel + Cascade

- **CRITICAL silent failure:** The "Gap: no notification is emitted on cancel" note is candid but the product impact is severe. **The tutee whose match was cancelled has no idea.** They believe they are still matched, will not look for tutoring elsewhere, and will think the platform is unresponsive. The tutor is in the same state.
- product requirements document N-16 and N-17 mandate emails for cancellation. **This is a launch-blocking gap.**
- **Tutee post returning to discovery:** Diagram does not show that the tutee post should return to the discovery index after cancel (product requirements document §3.7 — "Re-entry of tutee post into discovery index on match cancellation"). **Does the code do this?** The match-status machine shows ACTIVE → CANCELLED but the post-status side-effect is not shown.

### Flow 13 — Tutee Requests Interview

- **Trigger clarity:** Good. The login gate ("anonymous → navigate using /login?return To is ...) is correctly shown.
- **15-minute promise MISSING:** product requirements document BR-012 and N-22 mandate "15-minute duration" in the notification copy. The notification body in this diagram is "A tutee has requested an interview for your listing '`<safe title>`'." — no mention of 15 minutes. **This is the USP.** The user should see "15-minute interview" at the moment of intent.
- **Concurrency limit (max 3) NOT enforced in this diagram.** product requirements document §3.12 says "max 3 open requests per tutee at any time" — but the diagram only checks DUPLICATE INTERVIEW for the same tutor post. **Where is the global 3-concurrent limit?** Either the code lacks it or the diagram omits it. Flag for Tech Lead.
- **Cooldown (30 days) NOT shown.** product requirements document §3.12 says "no new request for same tutor post within 30 days of completed/no-show." The diagram only checks REQUESTED/SCHEDULED. **30-day cooldown is missing.**
- **Contact info promise NOT shown.** The interview flow is precisely the moment where the platform must reassure: "your contact info will not be shared with the tutor." A privacy annotation belongs here.
- **Admin in the loop is visible:** Good — the loop "for each admin in user Repository find By Role using ADMIN" makes admin coordination explicit.
- **Trust moment captured:** Toast + navigate to /tutee/interviews is the success perception. Good.

### Flow 14 — Admin Schedules Interview

- **Trigger clarity:** Good. Singapore time wall-clock note is helpful.
- **15-minute promise MISSING:** Same problem as flow 13. The notification body is "Your interview for '`<title>`' is scheduled for `<scheduled At>`." Missing the "15-minute" duration line product requirements document mandates.
- **Tutor not notified:** Diagram notes "Only the tutee is notified — not the tutor." But product requirements document N-23 explicitly addresses the tutor: "Interview request scheduled — join details inside" with date/time, 15-min notice, meeting link, **and tutee request summary**. **This notification is missing from the flow.** The tutor is left in the dark until they hit /tutor/interviews.
- **Meeting link not yet set when scheduling:** Flow 15 sets the link separately. So tutee receives "Interview scheduled" before there is even a link to give them. That is a confusing user experience — the tutee will see the scheduled time but the link will appear later with no notification. **The two should be combined or the user should be told "meeting link to follow."**

### Flow 15 — Admin Sets Meeting Link

- **Trigger clarity:** Good.
- **Silent change:** "No notification is emitted on link set — the link is visible the next time the tutee opens the specified interview accessible to the parent. This is a deliberate decision." This rationalisation is weak from a product perspective. **The tutee got an email/notification saying "interview scheduled" with NO link, and then must remember to refresh to find the link.** The product story is broken. Either notify on link-set, or do not notify on schedule until link is also set.
- Recommend: add N-22 (or equivalent) to fire on the *link-set* event if it has not yet fired.

### Flow 16 — Interview Terminal Paths

- **Trigger clarity:** Good — four sub-flows clearly labelled.
- **Complete path:** Only the tutee is notified. product requirements document N-25 says "follow up with tutee after interview" goes to admin. **The tutor receives no completion notification.** From the tutor's POV, the interview just disappears. Is that intentional?
- **Cancel path:** Both parties notified. Good.
- **No-show path:** Only the non-no-show party is notified. **Good logic.** product requirements document doesn't dictate this but it's the right product call.
- **Tutee self-cancel path:** Tutor is notified. Good.
- **No conversion-to-match prompt visible:** product requirements document §3.12 says "Interview analytics events (... conversion-to-match)" — the conversion funnel from interview-complete to match-created is the whole point of the feature. Diagram doesn't show any admin-prompt to convert. **Where does the admin get prompted: "this interview is complete — should we create a match?"** This is a critical product moment that should be in the flow.

### Flow 17 — Tutee Views Matches

- **Trigger clarity:** Good.
- **Privacy promise visible:** The diagram explicitly shows admin Contact Email is no value for PENDING/CANCELLED/COMPLETED matches and only REDACTED email for ACTIVE. **This is good.** The diagram should also explicitly note: "tutor contact info (email/phone) is NEVER in this response." That privacy annotation should be unmissable.
- **"Leave review" CTA:** Good — shows the re-engagement loop.
- **Missing user-facing flag:** "First session confirmation pending" prompt should appear for ACTIVE matches in AWAITING CONFIRMATION state. product requirements document §4.1 [8] and §11.1 say this should be a top-priority element on the tutee dashboard.

### Flow 18 — Tutor Views Matches

- **Privacy promise visible:** Diagram correctly notes "Tutor display name lookup is not done — the tutee's identity is the tuition subject/level summary, not a display name (privacy)." **Excellent.**
- **Commission triple shown:** Good — tutor sees what they're owed.
- **Same gap:** No notification that the match has progressed (e g., AWAITING CONFIRMATION → CONFIRMED → INVOICED). The tutor's dashboard is the only source of truth. That hurts re-engagement.

### Flow 19 — Tutee Submits Review

- **Trigger clarity:** Good.
- **User-facing outcome:** "toast 'Thanks for your review!'; navigate using /tutee/matches" — clear and good.
- **Reviewee is notified via bell:** "A match participant has left you a new review." Generic. Could be "Your tutee has left you a 5-star review" but I understand the bell body is intentionally minimal.
- **Aggregate refresh:** Good — recomputed server-side. Tutor's public profile updates. Public trust signal works.

### Flow 20 — Public Tutor Detail + Reviews

- **Trigger clarity:** Good.
- **Privacy promise NOT explicitly shown:** This is the public tutor detail page. **The most important place to annotate "no email/phone/name in response."** Diagram does not. AC-01 mandates this.
- **404 vs 400 inconsistency:** "Returns 400 NOT FOUND if the post is missing OR not ACTIVE (prevents enumeration)" — security-correct, but a 400 for a not-found resource is unusual. The user sees what? A "Tutor not found" page? Or a generic error? **[RESOLVED INT-137b, 2026-06-13: now 404 NOT FOUND (still 404 for non-ACTIVE too, so no enumeration); status matches the body code.]**
- **Login gate on Request Interview:** Good — anonymous → /login?return To is . Tutee → opens modal. Correctly captures the role split.
- **Where is the rate guide CTA?** product requirements document §3.5a says the tutor detail should link to /rates. Not shown.

### Flow 21 — Notification Bell

- **Trigger clarity:** Good — 60s polling.
- **User-facing perception:** Badge with count, "9+" cap. Good.
- **Role-aware navigation:** Good.
- **Critical product concern:** This bell is the ONLY user-visible signal for match created (flow 09), match activated (flow 10), commission paid (flow 11), interview events (flows 13-16), and reviews (flow 19). **Email is not in the loop for any of these.** Users who close the tab or are not active in the SPA will miss all of these critical lifecycle events. The 60s polling is reasonable, but a user who logs in once a week will not see "Match Created" until they log in. product requirements document strongly relies on email for these (N-08, N-09, N-10, N-11, N-22, N-23). **Bell + email should be the model; bell-only breaks re-engagement.**
- **Mark all read:** Good user experience touch.
- **No grouping:** A user with 50 unread notifications gets a flat list. product requirements document does not mandate grouping, so this is acceptable for MVP.

### Flow 22 — Favourite Toggle

- **Trigger clarity:** Good.
- **Optimistic update:** Excellent user experience pattern. Heart flips instantly; rolls back on error.
- **Error message:** "Failed to add favourite. Please try again." — clear, actionable. Good.
- **Bulk membership:** Good performance pattern.
- **No favourites count visible:** Tutee favorites a tutor but there's no badge on their account saying "You have 3 favourites." Minor product gap.

### Flow 23 — Public Browse with Filters

- **Trigger clarity:** Good — filter changes are React Query refetches.
- **Privacy promise NOT explicitly shown:** Same as flow 20. **The headline marketing claim is "contact info never shown" — this is the moment to annotate that on the wire.**
- **Filter clarity:** Good — all product requirements document §3.5 filters are present (subject, level, region, rate, venue, group size, availability).
- **Pagination concern:** "Page is hard-coded to 0 and size to 50; no user interface paginator." **product requirements document §3.5 mandates pagination "default 20 per page."** No paginator means users can never see page 2. **Either the diagram is wrong or the frontend is missing pagination.** Likely a product bug.
- **No rate guide link:** product requirements document §10.2 implies the browse page should link to /rates. Not shown.

### Flow 24 — Rate Limited Endpoint

- **Trigger clarity:** Good — login is the worked example.
- **User-facing message:** "Too many login attempts. Try again later." Clear, actionable.
- **Forgot-password exception:** Good — silent 200, avoids enumeration.
- **No retry-after header in the user message?** "Try again later" without "in X minutes" is friction. Recommend including the window.

### Flow 25 — cross-site request forgery protection-Protected Write

- **Trigger clarity:** Good engineering doc. From product POV, user perceives nothing — correct.
- **403 message:** "Invalid cross-site request forgery protection token" — what does the user actually see? A red toast saying "Session expired, please refresh"? Or a raw Spring error? product requirements document does not specify frontend handling. Recommend mapping 403 to a friendly toast.

### Flow 26 — Email Queue Fanout

- **THE BIGGEST PRODUCT-LEVEL ISSUE in the entire diagram set.**
- "No consumer in this monorepo. The email-notifications stream is produced into but never read."
- **Translation for the PM:** every email in the system — one-time passcode reset codes, post approval/rejection notifications, and (when implemented) match creation, match activation, first-session confirmation, interview schedules — **is enqueued into a dead-letter stream and nothing is sending those emails.**
- **Users will:** request a password reset and never get an one-time passcode; submit a post and never get the approval email; be matched and never know.
- **Dev Ops spec must clarify** whether the consumer exists out-of-tree, when it will be added, and how the absence is being monitored in staging. Until then, this is a hard launch blocker.
- **Producer/consumer reliability:** No failed-message queue, no retry, no backpressure. Acceptable for MVP if consumer is healthy, dangerous if consumer is down.

### Flow 27 — Encrypted personal information Write + keyed message authentication Lookup

- **Trigger clarity:** Good engineering doc.
- **Privacy promise IS captured here:** This is the only diagram that visibly shows email/phone are stored encrypted and looked up via keyed message authentication. **This is the cryptographic backbone of BR-001.** It should be referenced from every personal information-touching flow.
- **decrypt Safe pattern:** Good — admin sees null rather than 500. Resilient user experience.

---

## Missing user moments

These are flows where the user perceives nothing and shouldn't:

1. **Flow 01/02 — First-login privacy explainer** (product requirements document §4.1 [2] / §4.2 [2]). Tutee/tutor should see a one-page "Your contact details are never shared" modal on first login. Neither diagram shows this; code likely lacks it.
2. **Flow 06/07 — Post submission confirmation email** (product requirements document N-01). User submits a post and gets only a toast. The "We've received your post; we will notify within 2 business days" email is missing.
3. **Flow 09 — Match created email** (product requirements document N-08, N-09). The most exciting moment of the funnel only fires a bell ping. Email is mandated.
4. **Flow 10 — Match activated email + first-session confirmation email** (product requirements document N-10, N-11). Bell-only. Commission funnel depends on this email.
5. **Flow 10 — First-session confirmation page** (product requirements document §4.1 [8], AC-05). The first-session confirmation for the specified match page and the 7-day timeout job (product requirements document §7.7.1) are entirely missing from the diagrams. **This is a P0 acceptance criterion that has no flow.**
6. **Flow 11 — Tutee re-engagement after match completed.** Tutee should be prompted to leave a review. No notification fires.
7. **Flow 12 — Match cancel notifications** (product requirements document N-16, N-17). Both parties are silently dropped.
8. **Flow 12 — Tutee post return to discovery index.** Code path unclear; not visualised.
9. **Flow 14 — Tutor "interview scheduled" notification** (product requirements document N-23). Only tutee notified.
10. **Flow 14/15 — "Meeting link to follow"** — gap between schedule and link-set leaves tutee with no link.
11. **Flow 16 — Conversion-to-match prompt after COMPLETED interview.** Admin gets no prompt; product requirements document §3.12 expects this.
12. **Flow 17 — First-session confirmation CTA** on tutee match dashboard.
13. **Flow 20/23 — Rate guide link** (product requirements document §3.5a). Not in any browse flow.
14. **Welcome email on registration** (product requirements document §3.9 implies; no explicit N- code). Currently nothing.

---

## Misleading silent-failure paths

These flows currently tell the user "it worked" when it didn't:

1. **Flow 04 + Flow 26 — Password reset.** User sees "If your email exists, you'll receive a code." Stream consumer does not exist. **User waits forever.** This is the worst kind of silent failure — security-conscious by design, broken in practice.

2. **Flow 03 — Admin login.** When multi-factor authentication is required, frontend shows "Invalid email or password" because the multi-factor authentication branch is not wired. The admin will reset their password, then their time-based one-time passcode, before realising the bug.

3. **Flow 08 — Post approval/rejection email.** Email is enqueued; no consumer reads it. User believes their post is still pending; admin believes the user was notified. **Spreadsheet drift between admin and user perception** — admin will follow up wondering why the user hasn't responded.

4. **Flow 09, 10, 11, 13, 14, 16, 19 — Notifications via bell only.** The product requirements document treats these as email-critical. Users not actively in the SPA will miss the entire match/interview/review lifecycle.

5. **Flow 12 — Match cancel.** Tutee continues to believe they have an active match. Tutor too. Both will eventually log in and discover the cancellation as a fait accompli. **This breaks the platform's trust promise of "we keep you informed."**

6. **Flow 06/07 — Post creation analytics emit BEFORE commit.** "If the frontend-bound transaction rolls back after the event publish, a phantom analytics event may exist for a non-persisted post." From a product POV: **the BI dashboard will overcount post submissions.** This breaks BR / KPI reporting.

7. **Flow 13 — Interview concurrency limit (max 3) and cooldown (30 days) appear absent from the code.** Tutees can spam-request interviews. Product story breaks: "concierge curation" turns into "tutee floods the admin queue."

8. **Flow 23 — No pagination user interface.** Browse page shows first 50 tutors only. As inventory grows past 50, tutors below the 50-mark are silently invisible. Tutors below #50 will never get matched and will churn.

---

## Suggested additions

These are diagrams/annotations that should be added to make the product story coherent:

### New diagrams needed

1. **Flow 04b — First-login privacy explainer + acknowledgement.** product requirements document §4.1 [2] / §4.2 [2]. Currently missing entirely.

2. **Flow 10b — First-session confirmation by tutee.** P0 acceptance criterion AC-05. Should show:
   - Email sent on activation (N-10)
   - Tutee clicks link → lands on first-session confirmation for the specified match
   - Click "Yes, my first session has taken place"
   - Commission → CONFIRMED
   - Idempotent on re-submit (returns current status)

3. **Flow 10c — 7-day timeout cron job** (product requirements document §7.7.1). Should show:
   - Scheduled cron every 6h
   - Query matches in AWAITING CONFIRMATION + 7 days elapsed
   - Send N-12 to tutee + N-13 to admin
   - Mark confirmation overdue yes

4. **Flow 11b — Tutee "leave a review" re-engagement.** Triggers on commission PAID / match COMPLETED.

5. **Flow 16b — Admin "convert interview to match" prompt.** Triggers on COMPLETED interview. product requirements document §3.12 analytics conversion event.

6. **Flow 23b — Tutee discovery index (the tutor view of tutee requests).** This is the symmetric flow to the public tutor browse. product requirements document §3.6 dictates it exists. **It is not in the 27 sequences.** Approved tutors browse anonymised tutee requests — that flow has zero diagram coverage.

7. **Flow 28 — Post auto-expiry cron job** (product requirements document §7.7.2). Daily 02:00 Singapore time, expire posts >60 days, send N-18.

8. **Flow 29 — Account deletion / right-to-erasure** (product requirements document §3.10, P0 per §1.6). Personal Data Protection Act legal requirement. No diagram.

### Annotations to add to existing diagrams

1. **Every personal information-touching flow:** Add an explicit note: "Response payload does NOT include email, phone, or full name of any user other than the requester (BR-001 / AC-01)." Specifically: flows 17, 18, 20, 23, plus the implicit tutee-discovery flow.

2. **Flows 13, 14, 16, 26 (N-22, N-23):** Notification body must include "**15 minutes**" duration. Update body text in diagrams.

3. **Flow 09:** Add "EMAIL N-08 to tutee" and "EMAIL N-09 to tutor" to the par-notify block, not just bell.

4. **Flow 10:** Same — add N-10 and N-11 emails.

5. **Flow 12:** Add N-16 + N-17 emails to both parties. Add "Tutee post → ACTIVE again, re-enters discovery index."

6. **Flow 14:** Add N-23 to tutor.

7. **Flow 06/07:** Add N-01 email + "Post Under Review" dashboard badge state.

8. **Flow 08:** Confirm rejection email renders the human label, not the enum. Add explicit "Request Revision" sub-flow (product requirements document §6.3 path B).

9. **Flow 23:** Note the pagination concern — size is 50 is the hard cap. product requirements document wants 20 default with paginator.

10. **Flow 13:** Add the max-3-concurrent and 30-day-cooldown checks. If they don't exist in code, that is a P0 gap.

11. **Flow 26:** Until the consumer is confirmed, every flow that calls Email Queue Service enqueue should annotate: "**EMAIL DELIVERY IS UNVERIFIED IN THIS BUILD — see flow 26.**"

12. **Flow 20:** Add the rate-guide link from product requirements document §3.5a.

### Product copy that should be in the diagrams

These exact phrases should appear in notification body text in the diagrams (because they are the trust-building moments the user remembers):

- "**Your contact details have not been shared.**" — on every match/interview lifecycle notification
- "**15-minute interview**" — on every interview notification (BR-012)
- "**Our coordinator will contact you within 1 business day.**" — on match created notifications (product requirements document §4.1 [6])
- "**We will notify you within 2 business days.**" — on post submission confirmation (product requirements document §3.9 N-01)
- "**Your post is now active again and eligible for a new match.**" — on match cancel notification to tutee (product requirements document N-16)

---

## Compliance-user experience checklist

| product requirements document requirement | Diagram coverage | Status |
|---|---|---|
| Personal Data Protection Act consent at registration (logged) | Flow 01, 02 mention pdpa Consent but no audit log entry | **GAP** |
| Commission disclosure (logged, P0 per AC-03) | Flow 02 — clearly shown | PASS |
| Contact info never in non-admin response (AC-01) | No flow explicitly annotates this | **GAP** |
| Audit log on admin contact-field access (AC-01 second clause) | Flow 08 mentions decrypt Safe but no audit entry for the view itself | **GAP** |
| Admin multi-factor authentication enforcement (AC-07) | Flow 03 — shown, but frontend broken | PASS (backend) / FAIL (frontend) |
| First-session confirmation (AC-05) | **No flow** | **GAP** |
| Tutee discovery — anonymised (AC-06) | **No flow** | **GAP** |
| Account deletion / right-to-erasure (product requirements document §3.10 P0) | **No flow** | **GAP** |
| Email delivery service-level agreement (5 min for critical, 30 min for others) | Flow 26 — no consumer | **HARD BLOCKER** |
| 15-min interview promise (BR-012) | Not in body text | **GAP** |

---

## Re-engagement loops audit

The conversion funnel is: **register → post → approve → match → activate → confirm session → review → re-engage.**

Each transition should have a re-engagement moment (email + bell + dashboard state). Current coverage:

| Transition | Email | Bell | Dashboard | product requirements document ref | Status |
|---|---|---|---|---|---|
| Register → first login | NO | N/A | landing page | §3.1 | Welcome email missing |
| Post submitted → user feedback | NO | NO | toast only | N-01 | **GAP** |
| Post approved → user celebrates | enqueued (no consumer) | NO | status flips on next fetch | N-02/N-03 | **BROKEN** |
| Post rejected → user understands | enqueued (no consumer) | NO | status on next fetch | N-06/N-07 | **BROKEN** |
| Match created → user excited | NO | YES | yes | N-08/N-09 | **PARTIAL** |
| Match activated → user starts | NO | YES | yes | N-10/N-11 | **PARTIAL** |
| Tutee confirms first session | N/A (no flow exists) | — | — | AC-05 | **MISSING ENTIRELY** |
| Commission paid → match complete | tutor only (bell) | YES (tutor) | yes | N-15 | **PARTIAL** |
| Match completed → tutee prompted to review | NO | NO | "Leave review" CTA on next visit | — | **GAP** |
| Match cancelled → user informed | NO | NO | dashboard on next fetch | N-16/N-17 | **BROKEN** |
| Interview requested → tutor + admin alerted | NO | YES (both) | yes | N-21 | **PARTIAL** |
| Interview scheduled → tutee informed | NO | YES (tutee only) | yes | N-22 | **PARTIAL** |
| Interview scheduled → tutor informed | NO | NO | dashboard on next fetch | N-23 | **GAP** |
| Meeting link set → tutee + tutor informed | NO | NO | dashboard on next fetch | — | **GAP** |
| Interview complete → admin prompted to match | NO | NO | — | §3.12 | **GAP** |
| Post auto-expired → user told | NO | NO | dashboard on next fetch | N-18 | **MISSING FLOW** |

**Summary:** out of 16 transitions, 0 are fully covered by email + bell. The entire platform is a bell-only re-engagement loop with a broken email backend. **This is the single biggest product risk.**

---

## Final product call

**This diagram set is engineering-truthful but product-incomplete.** The team has accurately documented what the code does — including what it doesn't do. The gaps surfaced above are not authoring gaps in the diagrams; they are **implementation gaps that the diagrams have honestly exposed.**

Before any go-live decision is made, the following must be resolved in this order:

1. **Email consumer must exist and be verified.** Flow 26's gap is a hard blocker for password reset, post approval/rejection notifications, and (when added) every lifecycle email.
2. **Admin multi-factor authentication frontend wiring** (flow 03 gap). Admin login is broken today.
3. **Match cancel notification** (flow 12 gap). Users currently silently dropped.
4. **First-session confirmation flow** (missing — AC-05). P0 acceptance criterion has no diagram and no code path. This is the commission funnel.
5. **Interview tutor notification on schedule** (flow 14 gap — N-23). Tutor is left in the dark.
6. **15-minute interview promise** in N-22/N-23 body copy. USP must be in the message.
7. **Tutee discovery flow diagram** (missing). product requirements document §3.6 P0 feature has no sequence.
8. **Account deletion / right-to-erasure flow diagram** (missing). Personal Data Protection Act legal requirement, P0.
9. **Pagination on browse** (flow 23). At >50 tutors, tutors below #50 are invisible.
10. **Interview concurrency + cooldown enforcement** (flow 13). Currently absent.

These are not "P2 nice-to-haves." Items 1–4 are launch blockers. Items 5–10 are launch-quality issues that will erode trust within 30 days of go-live.

The diagrams are an honest mirror. The mirror is showing a product that is not yet ready to ship.
