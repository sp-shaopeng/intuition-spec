# User Flows

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** These early flows inherit the current public-filter, assigned-participant contact and money-event rules. Where an old step or screenshot conflicts, the corresponding numbered flow and current requirements control. See [current requirements](../../CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.
## Singapore Private Tuition Agency — user experience Flow Specification

**Document Version:** 2.1  
**Date:** 2026-05-27  
**Prepared By:** user experience Design  
**Source Documents:** business requirements document v1.1, product requirements document v1.1  

---

## Overview

This document specifies all end-to-end user flows for the three actor types: **Tutee**, **Tutor**, and **Admin**. Each flow step documents:
- The user action and the system response
- The user's likely emotional state at that moment
- Decision points and branch paths
- Error states and recovery paths
- Trust-building moments (anxiety and doubt peaks are called out explicitly)

The highest-risk moments for dropout are highlighted with DROPOUT RISK labels. Trust-building moments are highlighted with TRUST-BUILDING labels.

---

## Flow 1: Tutee — End-to-End Flow

### 1.1 Homepage Visit

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.1.1 | Lands on homepage (organic search, referral, social) | Renders landing page with hero, 3-step explanation, trust signals, browse tutors CTA | Curious but cautious — comparing against Smile Tutor in another tab | First impression: hero must lead with privacy promise, not feature list |
| 1.1.2 | Reads "How It Works" section | Static content renders | Mild scepticism — "every platform claims to be different" | Step 3 of the tutee journey must say "We contact you" — not "browse and choose" |
| 1.1.3 | Reads trust signals block | Static content renders | Slight reassurance if privacy promise is prominent | TRUST-BUILDING The sentence "Your phone number and email are never shared with any tutor — ever" must appear above the fold in the trust signals block |
| 1.1.4 | Clicks "Post a Tuition Request" CTA | Redirects to /register/tutee if not logged in | Motivated enough to act — key moment of intent | CTA copy "Post a Request" converts better than "Sign Up" at this stage |

**Trust-building moment at 1.1.3:** This is the primary objection for parents who have been burned by unsolicited tutor outreach on competing platforms. If this statement is buried, conversion to registration drops. The privacy promise must be in the hero section itself, not only in the trust signals block below.

---

### 1.2 Registration

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.2.1 | Arrives at /register/tutee | Role selection or dedicated tutee register form renders | Slightly impatient — wants to complete quickly | Keep form to 4 fields only: name, email, phone, password |
| 1.2.2 | Enters full name, email, phone, password | Real-time validation on email format and password strength | Neutral — form-filling mode | Show "Your number is seen by our team only — never shared with tutors" beneath phone field |
| 1.2.3 | Submits registration form | System creates account; sends verification email; shows "Check your email" screen | Slight anxiety — "did it work?" | Confirmation screen must say exact email address ("We sent to m***@gmail com") |
| 1.2.4 | Checks email inbox | Verification email arrives (within 5 min per N-01 service-level agreement) | Mild frustration if email is delayed; must check spam | Subject line: "Confirm your email — [Platform Name]" (not generic) |
| 1.2.5 | Clicks verification link | Account activated; redirected to privacy acknowledgement step | Relief — "it worked" | Transition directly to privacy explainer without any marketing interstitial |

**Error states:**
- Email already registered: inline error "An account with this email already exists. Log in instead?" with link to /login
- Weak password: inline feedback beneath field, non-blocking until submit
- Phone format invalid: inline error "Please enter a Singapore mobile number (+65 8XXXXXXX or 9XXXXXXX)"
- Verification link expired (24h): show "This link has expired. Request a new verification email" with resend button

**Recovery paths:**
- Resend verification: button on the "Check your email" screen; rate-limited (1 resend per 5 minutes)
- Lost email: user can re-enter email on forgot-verification page; triggers fresh email

---

### 1.3 Privacy Acknowledgement (First Login — One-Time Standalone Page)

TRUST-BUILDING — CRITICAL This is a standalone page (not a modal, not a checkbox on the form). It must feel like a considered, purposeful moment — not a legal hurdle.

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.3.1 | Lands on privacy acknowledgement screen after email verification | Full-screen page with clean layout — no nav, no sidebar, just the privacy message and one button | Uncertain — reads carefully | The absence of nav is intentional: signals this is important, not skippable |
| 1.3.2 | Reads privacy model explanation | Static content: 3 bullet points on contact privacy, written in plain English | Cautiously reassured | Use specific language ("never to any tutor") not vague ("handled responsibly") |
| 1.3.3 | Clicks "I Understand — Continue to Dashboard" | Acknowledgement logged (user ID + timestamp); redirected to tutee dashboard | Relieved; ready to proceed | Success state must feel like a beginning, not a gate cleared |

**Required copy on this screen:**


The illustration for privacy acknowledgement first login — one-time standalone page contains the following information:

- Your privacy is protected throughout this process.
- • Your phone number and email address are stored securely and only visible.
- to our coordination team — never to any tutor on this platform.
- • Tutors browsing the platform see your request by subject and area only.
- Your name does not appear.
- • Our coordinator will contact YOU directly when a suitable match is found.
- You do not need to chase anyone.
- This is how we are different from every other tuition platform in Singapore.
- [ I Understand — Continue to Dashboard ].


**Trust-building moment at 1.3.2:** This screen exists specifically to address the peak anxiety moment for parents who have been overwhelmed by unsolicited outreach on other platforms. The copy must be specific ("never to any tutor"), concrete, and positioned as a key differentiator — not buried inside a lengthy privacy policy.

---

### 1.4 Dashboard — First Visit (Empty State)

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.4.1 | Arrives at /tutee/dashboard | Empty state dashboard with clear CTA: "Post your first tuition request" | Motivated — clear next step | Empty state must not feel vacant; use encouraging copy and a visual |
| 1.4.2 | Sees dashboard empty state copy | "You have no active requests yet. It takes less than 10 minutes to get started." | Slight hesitation about time investment | 10-minute claim sets a concrete expectation — must be true |
| 1.4.3 | Clicks "Post a Tuition Request" | Navigates to /tutee/posts/new — multi-step wizard begins | Committed | Progress bar must appear immediately at Step 1 |

**Empty state copy:**


The illustration for dashboard — first visit empty state contains the following information:

- Welcome, [first name].
- You haven't posted a tuition request yet.
- Post one now — it takes less than 10 minutes,.
- and our team will review it within 2 business days.
- [ Post Your First Request ].


---

### 1.5 Create Tuition Request Post (4-Step Wizard)

The post creation wizard has 4 steps. A persistent progress indicator shows "Step X of 4" with step labels: Step 1: Subject + Level, Step 2: Schedule, Step 3: Venue + Location, Step 4: Contact Details + Notes. Draft saves automatically at each step transition.

#### Step 1: Subject + Level

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.5.1 | Selects subject from grouped dropdown (PSLE / O-Level / A-Level / IB / Poly-Uni / Adult) | Subject field populated | Focused | Group subjects by level family to reduce cognitive load |
| 1.5.2 | Selects education level (MOE taxonomy) | Level field populated; market rate hint appears | Focused | Rate hint reassures unsure parents about budget norms |
| 1.5.3 | Selects number of students (1 / 2 / 3+) | Field populated | Focused | |
| 1.5.4 | Selects student relationship (Parent's child / Self / Other) | Field populated | Focused | |
| 1.5.5 | Enters budget range (SGD/hr min and max) | Validates min ≤ max; shows market rate hint if budget appears outside norm | Slight uncertainty about appropriate budget | Market rate hint: non-blocking, informational only |
| 1.5.6 | Clicks "Next: Schedule" | Draft saved; navigates to Step 2 | Progressing | |

**Market rate hint copy (non-blocking):**


The illustration for step 1: subject + level contains the following information:

- Typical rates for the selected level the selected subject: SGD [X]–[Y]/hr depending on tutor experience.
- Your budget [is within / is below / is above] this range — tutors will still see your request.


**Error states (Step 1):**
- Budget max less than min: inline error "Your maximum budget must be at least equal to your minimum"
- No subject selected on Next: highlight field, scroll to it

#### Step 2: Schedule

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.5.7 | Taps/clicks availability slots on weekly grid | Selected slots highlighted in primary navy; desktop: 7×3 checkbox grid; mobile: accordion per day with 3 toggle switches | Engaged — this is the most interactive step | Mobile accordion default: Saturday and Sunday expanded (high-demand) |
| 1.5.8 | Selects session duration (1hr / 1.5hr / 2hr / 2.5hr) | Field populated | Focused | |
| 1.5.9 | Selects frequency per week (1× / 2× / 3× / Flexible) | Field populated | Focused | |
| 1.5.10 | Optionally selects preferred start date | Calendar picker renders | Focused | |
| 1.5.11 | Clicks "Next: Location" | Draft saved; navigates to Step 3 | Progressing | |

**Error states (Step 2):**
- No availability slot selected: toast error "Please select at least one availability slot"

#### Step 3: Venue + Location

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.5.12 | Selects preferred venue types (multi-select chips: Student's Home / Tutor's Home / Public Place / Online / Tuition Centre) | Chips toggle on/off | Focused | |
| 1.5.13 | Searches and selects nearest MRT station or planning area via typeahead | Searchable dropdown with all Singapore MRT stations and URA planning areas; broad region derived automatically | Slight friction if unsure of area name | Helper note: "Your home address is never collected" must appear beneath field |
| 1.5.14 | Clicks "Next: Contact" | Draft saved; navigates to Step 4 | Progressing | |

**Note:** Budget is collected in Step 1 (field 1.5.5). Step 3 contains only Venue Types and Location — no budget fields.

**Helper copy beneath location field:**


The illustration for step 3: venue + location contains the following information:

- We use this to match tutors who can travel to your area.
- Your home address is never collected.


#### Step 4: Contact Details + Special Notes + Submit

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.5.15 | Sees contact section pre-populated from registration | Phone and email shown as masked previews: "+65 9XXX XXXX" and "m***@gmail com" | Mild surprise — "oh, it's already there" | Masking confirms data is held without revealing it |
| 1.5.16 | Reviews pre-filled contact details | Read-only display with admin-only visibility note | Reassured | Note must say "Only our coordination team can see these" |
| 1.5.17 | Optionally enters special requirements (free text, 500 chars) | Text area with char count | Engaged if they have specific needs | Placeholder examples: "Prefer female tutor", "Child has dyslexia" |
| 1.5.18 | Clicks "Review and Submit" | Shows full post preview screen | Mild anxiety — "does this look right?" | Preview shows exactly what admin sees |
| 1.5.19 | Reviews post preview | Full post preview; all public fields visible; edit link per section | Careful review mode | |
| 1.5.20 | Clicks "Submit for Review" | Post status → Pending Review; confirmation screen shown; N-01 email sent | Relief mixed with mild impatience | Confirmation screen gives clear "what happens next" |

**Contact section copy:**


The illustration for step 4: contact details + special notes + submit contains the following information:

- Your contact details are kept confidential.
- Only our coordination team can see your phone number and email.
- They will reach out to you directly when a match is found.


**Post submitted confirmation screen copy:**


The illustration for step 4: contact details + special notes + submit contains the following information:

- Your request has been submitted.
- We'll review it within 2 business days and notify you by email.
- While you wait, feel free to browse our tutor listings.
- [ Browse Tutors ] [ Back to Dashboard ].


**Error states (Step 4):**
- Network error on submit: show retry button; draft is preserved
- Validation fails on re-check: scroll to first error, highlight field

---

### 1.6 Pending Review State

DROPOUT RISK — MODERATE This is the first waiting period. Anxiety peaks if the user doesn't hear anything.

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.6.1 | Logs into dashboard after submission | Dashboard shows post with "Under Review" badge in amber | Mild impatience — "is anyone looking at this?" | Badge must include "We aim to review within 2 business days" |
| 1.6.2 | Checks dashboard multiple times | Same state displayed; no changes visible until admin acts | Increasing impatience if no update for >24h | Copy must be actively reassuring, not passive |
| 1.6.3 | Admin approves post (within 2 business days per service-level agreement) | Post status → Active; N-02 email sent | Relief + excitement | N-02 email must arrive within 5 minutes of admin action |

**Dashboard copy while in Pending Review:**


The illustration for pending review state contains the following information:

- Status: UNDER REVIEW.
- Your post is being reviewed by our team.
- We aim to respond within 2 business days.
- You'll receive an email as soon as it's reviewed.
- Submitted: [date/time].


**Email on submission (N-01):**


The illustration for pending review state contains the following information:

- Subject: We've received your tuition request.
- Hi [first name],.
- Thank you for posting your tuition request on [Platform Name].
- Our team will review your request within 2 business days and notify you.
- by email whether it has been approved or if we have any questions.
- You can log in to check the status at any time:.
- [ View Your Request ].
- Our team typically reviews requests Monday to Friday during business hours (Singapore time).
- Warm regards,.
- [Platform Name] Team.


**Path B — Admin requests revision:**

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.6.4 | Receives N-04 email: "Revision needed" | Email contains admin's message; link to edit post | Mild disappointment | N-04 email tone must be helpful, not bureaucratic |
| 1.6.5 | Logs in; sees "Revision Required" badge in red | Post card shows admin's message; "Edit and Resubmit" button | Focused on fixing the issue | Show the specific field that needs attention if possible |
| 1.6.6 | Edits post and clicks "Resubmit for Review" | Post re-enters Pending Review; returns to step 1.6 | More cautious optimism | |

**Path C — Admin rejects post:**

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.6.7 | Receives N-06 email: "Update on your request" | Email contains rejection reason (from enum, mapped to human copy); no harsh language | Disappointment; may feel embarrassed | Never use enum codes in user-facing copy |
| 1.6.8 | Logs in; sees "Rejected" badge | Post card shows reason | Deflated; uncertain whether to try again | Include "You may post a new request" link |
| 1.6.9 | Clicks "Post a New Request" | Opens fresh post wizard | Tentative — second attempt | |

**Rejection reason enum → human copy mapping:**
- INCOMPLETE INFO → "We need a little more information to process your request. Please resubmit with more detail about the subject and schedule."
- UNREALISTIC RATE → "The budget you've specified is significantly below market rates for this level. We'd encourage you to review our pricing guide and resubmit."
- DUPLICATE POST → "We noticed you have an existing active request for a similar subject and level. Please check your dashboard before resubmitting."
- SPAM TEST → "This post appears to be a test submission. Please submit a genuine tuition request."
- INAPPROPRIATE CONTENT → "Your post contains content that does not meet our platform guidelines. Please resubmit with appropriate content."

**Validation note — personal information in free-text fields:**
If admin detects personally identifiable information (personal information) in any free-text field (Special Requirements for tutees; Teaching Approach / Additional Notes for tutors) during the review process, this constitutes grounds for rejection with:
- Rejection reason: INAPPROPRIATE CONTENT
- Admin note (internal, not shown to user): "Personal contact information detected in free-text field — please resubmit without contact details."

The user-facing copy for this rejection is: "Your post contains content that does not meet our platform guidelines. Please resubmit with appropriate content." (same INAPPROPRIATE CONTENT mapping above — do not expose the specific personal information reason in user-facing copy).

---

### 1.7 Approval — Post Goes Live (Email N-02)

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.7.1 | Receives N-02 email: "Your request is now live" | Email confirms approval, explains next steps including Day 0 "what happens next" explainer | Excitement mixed with anticipation | The Day 0 email sets expectations for the waiting period — critical dropout prevention |
| 1.7.2 | Logs into dashboard | Post shows "Active" badge in green | Satisfied | Active badge copy: "ACTIVE — WE'RE LOOKING" (active voice, not passive) |
| 1.7.3 | May browse tutor listings out of curiosity | Can browse /tutors (public); cannot see other tutee requests | Engaged; building expectations | |

**N-02 email copy (Day 0 approval email — includes "what happens next" explainer):**


The illustration for approval — post goes live email n-02 contains the following information:

- Subject: Your tuition request is now live.
- Hi [first name],.
- Great news — your tuition request has been approved and is now active.
- on our platform.
- What happens next?.
- 1. Our coordination team is actively reviewing tutor profiles against.
- your request right now.
- 2. When a suitable tutor is identified, we'll notify you immediately.
- by email and on your dashboard.
- 3. Our coordinator will then contact you directly by phone within.
- 1 business day to facilitate the introduction.
- Our target: to find your match within 7 days of your request going live.
- You don't need to do anything right now. We'll reach out the moment.
- we find the right fit.
- [ View Your Request on Dashboard ].
- Warm regards,.
- [Platform Name] Team.


---

### 1.8 Active Post — Approved-but-Unmatched Waiting Period

DROPOUT RISK — HIGHEST This is the highest dropout risk moment in the entire tutee flow. The user has done everything right but is now waiting passively with no visibility into admin's matching activity. Proactive communications at Day 3 and Day 7 are essential dropout-prevention mechanisms.

**Dashboard copy — Active state (Day 1 onwards):**


The illustration for active post — approved-but-unmatched waiting period contains the following information:

- Status: ACTIVE — WE'RE LOOKING.
- Your request is live and our team is actively reviewing tutor profiles.
- to find the right match for you.
- What to expect:.
- • Our target is to find a match within 7 days of your request going live.
- • You'll receive an email the moment a match is identified.
- • Our coordinator will then contact you directly within 1 business day.
- [ Subject ] [ Level ] Posted [date] [Area].


**Waiting period FAQ accordion (shown on dashboard when post is Active and unmatched):**


The illustration for active post — approved-but-unmatched waiting period contains the following information:

- ▼ How long does matching take?.
- Our team targets a match within 7 days of your request going live.
- Some subjects and levels have more limited tutor availability,.
- and we take care to only propose matches we're confident about.
- ▼ Can I see which tutors are being considered?.
- Our coordination team reviews all tutor profiles internally.
- Tutor names and contact details are not shown during the matching process —.
- this protects both parties and ensures a quality introduction.
- ▼ What if I need to change something in my request?.
- You can update your request at any time from this dashboard.
- Updates re-enter the review queue but are typically reviewed within 24 hours.
- ▼ What happens when a match is found?.
- You'll receive an email immediately, and our coordinator will contact.
- you by phone within 1 business day to facilitate the introduction.


**Update my post link:** Shown beneath the FAQ accordion — "Need to change your schedule or budget? [Update my request]" links to post edit view.

---

#### Approved-but-Unmatched: Day 0 Communications (Approval Email — covered in 1.7 above)

---

#### Approved-but-Unmatched: Day 3 Automated Check-In Email

Sent 3 calendar days after approval if no match has been made:


The illustration for approved-but-unmatched: day 3 automated check-in email contains the following information:

- Subject: Your search is underway.
- Hi [first name],.
- Your tuition request for the selected subject at the selected level went live 3 days ago.
- Our team is actively reviewing our tutor pool to find the right match.
- Here's what to expect:.
- • Median time to match: 7 days.
- • We will notify you the moment we identify a suitable tutor.
- In the meantime, you're welcome to browse our tutor listings to get a.
- sense of the profiles we work with:.
- [ Browse Tutors ].
- If your needs have changed, you can update your request anytime:.
- [ View Dashboard ].
- Warm regards,.
- [Platform Name] Team.


**Day 3 email design notes:**
- Subject line must not imply urgency or concern — conveys momentum
- Phrase "actively reviewing" reassures without implying problems
- Offer to browse tutors creates an engagement hook without false urgency
- Do not offer to close the request — this triggers abandonment

---

#### Approved-but-Unmatched: Day 7 Email — "Still Looking, Here's What Admin Is Doing"

Sent 7 calendar days after approval if no match has been made:


The illustration for approved-but-unmatched: day 7 email — still looking, heres what admin is doing contains the following information:

- Subject: Update on your tuition search.
- Hi [first name],.
- We wanted to check in — your tuition request for the selected subject at the selected level.
- has been live for 7 days.
- What our team is doing:.
- Our coordinators are actively reviewing our tutor pool for your specific.
- requirements. For some subjects and areas, we may need a little more time.
- to find a tutor who is the right fit — we'd rather take the time to find.
- someone good than rush a match that isn't right for you.
- If anything has changed (different schedule, different subject, or budget.
- adjustments), updating your request may help us find a faster match:.
- [ Update Your Request ].
- We appreciate your patience and will be in touch as soon as we have news.
- Warm regards,.
- [Platform Name] Team.


**Day 7 email design notes:**
- Subject line must NOT say "no match found" — this increases dropout
- "What our team is doing" heading converts passive waiting into active expectation management
- Giving the user agency (update request) reduces helplessness
- Do not offer to close the request — this prompts abandonment
- Never use the word "unfortunately" in this email

**Post-Day 7 dashboard copy (if still unmatched after 7 days):**


The illustration for approved-but-unmatched: day 7 email — still looking, heres what admin is doing contains the following information:

- Status: ACTIVE — SEARCH ONGOING.
- We're still working to find the right match.
- Our coordinator is reviewing our tutor pool for your specific requirements.
- Your request has been live for [N] days.
- Updating your requirements or budget may help us find a faster match.
- [ Edit Request ].


---

#### Approved-but-Unmatched: Day 14 Email — "Your search is progressing" (N-new)

Sent 14 calendar days after approval if still no match has been made:


The illustration for approved-but-unmatched: day 14 email — your search is progressing n-new contains the following information:

- Subject: Update on your tuition search – the selected subject for the selected level.
- Hi [first name],.
- Your tuition request for the selected subject at the selected level has been active for 14 days.
- Our coordinator has reviewed your request and is actively shortlisting.
- suitable tutors. We expect to have an update for you within the next.
- few days.
- If your timing or budget has changed, updating your preferences may.
- help us find a faster match:.
- [ Update your preferences ].
- We appreciate your patience and will be in touch as soon as we have news.
- Warm regards,.
- [Platform Name] Team.


**Day 14 email design notes:**
- Subject line: specific (includes subject and level) — signals personalised attention, not an automated blast
- Copy uses "shortlisting" — not "still looking" — to convey active, purposeful progress
- Never say "we're still looking" — this increases anxiety and signals passivity
- "Update your preferences" CTA: use the word "preferences" (not "requirements") — feels lower-friction
- Do not offer to close the request — this triggers abandonment
- Flag: INAPPROPRIATE CONTENT rejection reason copy: "Your post contains content that does not meet our platform guidelines. Please resubmit with appropriate content."

**Admin escalation at Day 21 (N-new-admin):**

If the post remains unmatched at Day 21 after approval, an internal escalation alert is automatically sent to the admin team. This notification does not go to the tutee.

| Notification ID | Trigger | Recipient | Content |
|---|---|---|---|
| N-new-admin | Post unmatched 21 days after approval | Admin (internal) | "Post unmatched 21 days: [Post ID] — the selected subject, the selected level, [Area]. Admin review and action required." |

**Notification list addition:**
- N-new — Day 14 tutee email: "Update on your tuition search – the selected subject for the selected level" (sent to tutee if unmatched at Day 14)
- N-new-admin — Day 21 admin escalation alert: "Post unmatched 21 days" (internal; sent to admin team)

---

**Post auto-expiry at Day 60:**
- Day 55 warning email: "Your request will expire in 5 days. Update it to keep it active."
- Day 60: Post auto-expires; N-18 sent: subject line "Your tuition search — next steps" (never "Your post has expired")
- Expiry email must include prominent "Post a new request" CTA

---

### 1.9 Match Notification (Email N-08)

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.9.1 | Receives N-08 email: "A match has been found!" | Email confirms match; explains admin will contact within 1 business day | High excitement — this is the payoff moment | Subject line "Exciting news — we've found a potential match!" must not be changed |
| 1.9.2 | Logs into dashboard | Post shows "Matched" badge (navy); match card appears | Excitement; scanning for tutor details | Badge copy: "MATCHED — Coordinator will contact you shortly" |
| 1.9.3 | Views match card | Match shows: subject, level, "Coordinator will contact you shortly" — NO tutor contact info shown | Mild impatience — "but who is the tutor?" | TRUST-BUILDING Copy must acknowledge the natural desire to know the tutor |

**N-08 email copy:**


The illustration for match notification email n-08 contains the following information:

- Subject: Exciting news — we've found a potential match!.
- Hi [first name],.
- We're pleased to inform you that our coordination team has identified.
- a tutor who matches your request for the selected subject — the selected level.
- What happens next:.
- • Our coordinator will contact you directly within 1 business day.
- to facilitate the introduction.
- • Please ensure your phone ([masked: 65 9XXX XXXX]) is reachable.
- No action is required from you right now.
- [ View Your Dashboard ].
- Warm regards,.
- [Platform Name] Team.


**Dashboard matched state copy:**


The illustration for match notification email n-08 contains the following information:

- Status: MATCHED.
- Our team has found a potential tutor for your the selected subject request.
- Our coordinator will be in touch with you shortly.
- Please keep your phone reachable.


TRUST-BUILDING at 1.9.3: The user will naturally want to know who the tutor is. The copy must acknowledge this ("Our coordinator will introduce you personally") rather than simply showing an empty panel. The absence of contact info should feel premium and deliberate, not a bug.

---

### 1.10 Admin Contacts Tutee Off-Platform

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.10.1 | Admin calls/emails/Whats Apps tutee using stored contact info | Off-platform event — not tracked on platform dashboard | Relief — real human contact | This human touchpoint is the brand differentiator; quality of admin call determines NPS |
| 1.10.2 | Tutee discusses arrangement with admin | Off-platform | Engaged; building trust with coordinator | |
| 1.10.3 | Arrangement agreed; first session scheduled | Off-platform | Anticipation | |
| 1.10.4 | Admin marks match ACTIVE in admin panel | System sends N-10 email to tutee; commission status → AWAITING CONFIRMATION | Anticipation turning to action | N-10 must arrive within 5 minutes of admin action (commission-critical service-level agreement) |

---

### 1.11 First Session Confirmation

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.11.1 | Receives N-10 email: "Please confirm your first session" | Email has prominent "Confirm First Session" button | Slightly uncertain about the process | Button must be the largest element in the email |
| 1.11.2 | Clicks button; arrives at first-session confirmation for the specified match | Confirmation page loads with context (subject, level, match date) | Willing to help | Page must not feel like a payment or legal step |
| 1.11.3 | Reads confirmation page context | Shows: tutor subject/level, why confirmation matters, 30-second estimate | Informed and ready | Explain commission briefly and honestly — no jargon |
| 1.11.4 | Clicks "Yes, my first session has taken place" | Commission status → CONFIRMED; thank-you page shown; N-14 internal event | Satisfied — flow complete | Thank-you page: green check, simple copy, "Return to Dashboard" |

**N-10 email copy:**


The illustration for first session confirmation contains the following information:

- Subject: Please confirm your first tuition session.
- Hi [first name],.
- Your tuition arrangement for the selected subject at the selected level is now underway.
- To complete the process, please confirm when your first session.
- has taken place by clicking the button below.
- [ Confirm First Session Occurred ].
- Why do we ask this?.
- This confirmation allows us to process the coordinator commission.
- and formally complete your arrangement record.
- If your arrangement has not yet started, no action is needed yet.
- Please return to this page once your first session has taken place.
- If you have any questions, please reply to this email.
- Warm regards,.
- [Platform Name] Team.


**Confirmation page copy:**


The illustration for first session confirmation contains the following information:

- the selected subject: the selected level.
- Status: Arrangement Active.
- Please confirm your first session has taken place.
- This step takes 30 seconds and helps us:.
- • Complete the arrangement record.
- • Ensure our coordinator is paid for their work matching you.
- [ Yes, my first session has taken place ].
- If your first session has not yet happened, please wait until it does.
- and return to this page, or contact us at [support email].
- [ Session hasn't happened yet — remind me later ].


**Branch — no action within 7 days:**

| Day | System Action | Admin Panel State | User Communication |
|---|---|---|---|
| Day 7 (7 calendar days after N-10 sent) | Cron job triggers N-12 to tutee; N-13 alert to admin | Match flagged "Awaiting Tutee Confirmation — OVERDUE" in red | N-12 reminder email sent (commission-critical service-level agreement: within 5 min of trigger) |
| Day 7+ | Admin alerted | Override confirmation prompt appears in match detail | Admin may call tutee off-platform to obtain confirmation |
| Day 14+ | Admin must decide | Escalation indicator in matches list | Admin must: override → CONFIRMED, OR cancel match → WAIVED |

**N-12 reminder email copy:**


The illustration for first session confirmation contains the following information:

- Subject: Reminder: Please confirm your first tuition session.
- Hi [first name],.
- We haven't yet received confirmation that your first tuition session.
- for the selected subject has taken place.
- This is a quick, one-click confirmation:.
- [ Confirm First Session Occurred ].
- If your arrangement hasn't started yet, please let us know by.
- replying to this email so we can keep your record up to date.
- Warm regards,.
- [Platform Name] Team.


---

### 1.12 Commission Tracked — Flow Complete

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 1.12.1 | Sees post status on dashboard | Post remains as Matched / Active during active arrangement | Settled — "this is working" | |
| 1.12.2 | Admin invoices tutor off-platform | No tutee action required | Unaware — positive outcome | Tutee is not involved in commission collection |
| 1.12.3 | Arrangement eventually ends; admin closes match | Post moves to Closed | Satisfied; may return for another request | Closed state: show "Find another tutor?" CTA for repeat post |

**Post-match: thank-you and repeat engagement copy (shown when match is marked Closed):**


The illustration for commission tracked — flow complete contains the following information:

- This arrangement has been completed.
- Thank you for using [Platform Name].
- Did you have a good experience? We'd love to hear from you.
- [ Share Feedback ].
- Looking for another tutor — for a different subject or another child?.
- [ Post a New Request ].


---

## Flow 2: Tutor — End-to-End Flow

### 2.1 Homepage Visit

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.1.1 | Lands on homepage | Landing page renders — both tutee and tutor CTA paths visible | Assessing legitimacy of platform | Experienced tutors (especially ex-MOE) are deeply privacy-conscious |
| 2.1.2 | Reads "How It Works — For Tutors" section | 3-step tutor flow rendered | Comparing to Smile Tutor experience | Step 3 must say "We contact you" — not "parents find you" |
| 2.1.3 | Reads trust signals | Privacy-first and admin-vetting signals | Growing interest — no spam model appeals | TRUST-BUILDING "Your contact details are never shared — we handle all introductions" must be prominent |
| 2.1.4 | Clicks "List as a Tutor" CTA | Navigates to /register/tutor | Intent established | |

---

### 2.2 Registration — Account Creation (Step 1 of 3)

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.2.1 | Arrives at tutor registration | Form: full name, email, phone, password | Ready to proceed | |
| 2.2.2 | Fills in registration form | Real-time validation | Efficient | |
| 2.2.3 | Submits form | Account created; verification email sent | Expecting email | |
| 2.2.4 | Verifies email via link | Account activated; system redirects to Step 2: Commission Disclosure | Slightly uncertain what's next | Progress indicator shows "Step 1 of 3 complete" |

**Error states (tutor registration):** Same as tutee flow, plus:
- Role conflict: "You already have an account registered as a tutee. One account may only hold one role."

---

### 2.3 Commission Disclosure — Standalone Full-Page Mandatory Step (Step 2 of 3)

TRUST-BUILDING — CRITICAL This is the most legally and reputationally important user experience moment in the tutor flow. It is a full standalone page — not a T&C scroll buried in registration, not a modal, not a checkbox appended to the registration form. It must be designed to feel transparent and respectful.

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.3.1 | Redirected to /register/tutor/commission after email verification | Full-page commission disclosure: no main nav, only logo + progress indicator | Surprised if they didn't expect a commission; informed if they did | Standalone page signals importance; nav absence prevents skipping |
| 2.3.2 | Reads commission disclosure content | All required content rendered: rate (50%), calculation method, worked example, trigger, payer, one-time nature | Calculating — working out if this is worth it | Rate and SGD amounts must be the visually largest elements |
| 2.3.3 | Reads worked numeric example | SGD 800/month × 50% = SGD 400 one-time (or specific example: SGD 50/hr × 1.5hr × 8 sessions = SGD 600/month → SGD 300 commission) | Concrete understanding forming | Use realistic Singapore rates; not hypothetical minimum |
| 2.3.4 | Checks acknowledgement checkbox | Checkbox activates; "Continue" button becomes enabled | Deliberate, informed choice | Button disabled state must be obvious (not just opacity) |
| 2.3.5 | Clicks "I Understand — Continue to Create My Listing" | Acknowledgement logged: tutor user ID, timestamp (ISO 8601), terms version (e g. "1.0.0"), IP address, affirmative action type is checkbox checked; navigates to Step 3 | Committed | System must confirm log write before redirect; server error shows retry |

**Commission disclosure page — full required copy:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Design constraints for commission disclosure screen:**
- Standalone page — no nav, no sidebar, no distractions
- Commission rate ("50%") and the worked example SGD amount MUST be visually prominent (larger font or callout box — not a body text line)
- Do NOT label this as "Terms and Conditions" — use "Commission Terms" to signal specific, digestible content
- Button must be disabled until checkbox is explicitly checked
- If the user tries to navigate away without completing, show browser-native "Leave page?" prompt
- Acknowledgement must be logged server-side before redirect proceeds; a server error shows a retry message — do not allow proceeding without successful log write

**Error states:**
- Submit without checkbox: "You must accept the commission terms to continue" — button shake animation
- Server error on acknowledgement log: show retry; do not allow proceeding without successful log write

---

### 2.4 Create Tutor Offering Post (5-Step Wizard, Step 3 of registration flow)

#### Step 1: Subjects (Multi-Select Chips Grouped by Level) + Levels (Multi-Select Chips)

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.4.1 | Selects subjects from multi-select chips grouped by level family (Primary/PSLE, O-Level, A-Level, IB, Poly/Uni, Adult) | Subject chips added; search box above groups for filtering | Engaged — core of their offering | Group by level family reduces overwhelm of 30+ subjects |
| 2.4.2 | Selects levels taught from multi-select chips (MOE taxonomy) | Level chips added; at least 1 required | Careful to match what they actually teach | |
| 2.4.3 | Selects venue types (multi-select chips) | Venue chips toggle | Straightforward | |
| 2.4.4 | Selects group size preference (1-on-1 / Small group 2–4 / Both) | Radio/chip selection | Straightforward | |
| 2.4.5 | Clicks "Next: Availability" | Draft saved; navigates to Step 2 | Progressing | |

#### Step 2: Availability Grid + Group Size

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.4.6 | Interacts with weekly availability grid (Mon–Sun × Morning/Afternoon/Evening) | Desktop: 7×3 checkbox grid with cell size 40px; Mobile: accordion per day with 44px-minimum toggle buttons | Slightly complex — thinking through schedule | Show quick-select buttons: "Select Weekday Evenings", "Select Weekends", "Clear All" |
| 2.4.7 | Adjusts grid until schedule is accurate | Live visual feedback; selected cells show navy fill | Satisfied when grid matches mental model | Mobile: Saturday and Sunday expanded by default |
| 2.4.8 | Clicks "Next: Rates" | Draft saved; navigates to Step 3 | Progressing | |

**Availability grid helper copy:**


Implementation example omitted. This public edition retains the written requirements in this section.


#### Step 3: Rates (Min/Max SGD) + Venue Types

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.4.9 | Enters hourly rate min (SGD) via number input | Validates positive integer | Confident if experienced; uncertain if new | Show market rate benchmark for their level/subject combo |
| 2.4.10 | Enters hourly rate max (SGD) | Validates ≥ min | Straightforward | Non-blocking market rate hint: "Ex-MOE teachers for the selected level the selected subject typically charge SGD X–Y/hr" |
| 2.4.11 | Clicks "Next: Qualifications" | Draft saved; navigates to Step 4 | Progressing | |

#### Step 4: Qualifications + Experience + Teaching Approach + Photo Upload

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.4.12 | Enters qualifications (free text, 500 chars) | Character count shown | Some uncertainty about what to include | Placeholder examples help: "NTU Year 3 Engineering. 2 years tutoring Sec 3–4 A-Maths." |
| 2.4.13 | Enters years of tutoring experience (number, 0–50) | Number input | Straightforward | |
| 2.4.14 | Optionally enters teaching approach (free text, 500 chars) | Character count shown | More senior tutors spend time here | |
| 2.4.15 | Optionally uploads profile photo | File upload; JPEG/PNG/Web P; max 5MB; preview shown | Slight hesitation (privacy-conscious) | TRUST-BUILDING Helper copy: "Shown to admin only — not shown to tutees or on public listings" |
| 2.4.16 | Clicks "Next: Location" | Draft saved; navigates to Step 5 | Progressing | |

#### Step 5: Location + Contact + Submit

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.4.17 | Selects location area (MRT station or planning area via typeahead) | Searchable dropdown with all Singapore MRT stations and planning areas | Straightforward | |
| 2.4.18 | Sees pre-filled contact details from registration | Masked phone and email shown; read-only | Reassured — system remembered | Same masking and privacy note as tutee flow |
| 2.4.19 | Reviews full post preview (all 5 steps summarised) | All fields shown; edit links per section | Thorough review | |
| 2.4.20 | Clicks "Submit for Review" | Post → Pending Review; N-01 email sent | Relieved; anticipating approval | |

---

### 2.5 Pending Review State

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.5.1 | Logs in; sees "Under Review" badge | Dashboard shows post with amber badge | Impatient but understanding | Same copy pattern as tutee: 2 business day service-level agreement visible |
| 2.5.2 | Admin approves (within 2 business days) | N-03 email sent; post goes live | Excitement | N-03 email must include "You can now browse tutee requests" |
| 2.5.3 | Receives N-03 email | "Your listing is now live" with link to browse tutee requests | Ready to engage | |

**Note on commission acknowledgement gate:** If admin cannot approve because commission acknowledgement is missing, the admin review view shows a warning and the Approve button is disabled. The system prevents this by enforcing completion of Step 2.3 before post creation is accessible.

**Path B — Revision requested / Path C — Rejected:** Same pattern as tutee flow. Enum-to-human copy mapping applies.

---

### 2.6 Post Live — Browse Tutee Requests (Anonymised Discovery Index)

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.6.1 | Navigates to /tutor/requests | Anonymised tutee request discovery index renders — Active posts only; no contact info | Curiosity — scanning for relevant requests | Page header must explain the anonymisation model |
| 2.6.2 | Filters by subject and level | Filtered results shown | Focused on relevant matches | Same filter pattern as public tutor listing |
| 2.6.3 | Views tutee request detail | Subject, level, area, budget, schedule, number of students, special requirements — NO tutee name, phone, or email | Informed; assessing fit | |
| 2.6.4 | Sees no "Apply" or "Contact" button | No apply/contact action exists anywhere on tutee request pages | Acceptance — understands the model | TRUST-BUILDING Copy must explain why — not just absent |

**No-contact copy on tutee request detail:**


The illustration for post live — browse tutee requests anonymised discovery index contains the following information:

- Interested in this request?.
- Our coordination team reviews all profiles and will contact you.
- if this assignment is a good fit. No action needed from you.


---

### 2.7 Match Notification (Email N-09)

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.7.1 | Receives N-09 email: "A tutee match has been identified for you" | Email confirms match; coordinator will contact within 1 business day | Excitement; professional anticipation | N-09 tone should feel professional and personalised |
| 2.7.2 | Logs into dashboard | Match card appears in My Matches | Scanning for tutee details | Match card shows: subject, level, area — no tutee contact info |
| 2.7.3 | Views match card | Subject, level, area shown; "Coordinator will contact you" | Slight impatience; trusts the process | |

**N-09 email copy:**


The illustration for match notification email n-09 contains the following information:

- Subject: A tutee match has been identified for you.
- Hi [first name],.
- We've identified a tutee request that matches your profile for.
- the selected subject — the selected level.
- Our coordinator will contact you within 1 business day to facilitate.
- the introduction.
- Please ensure your phone is reachable.
- [ View Your Dashboard ].
- Warm regards,.
- [Platform Name] Team.


---

### 2.8 Admin Contacts Tutor Off-Platform

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.8.1 | Admin contacts tutor via stored phone/email/WhatsApp | Off-platform event | Relief — real human contact validates the platform | Admin call tone and professionalism determine tutor NPS |
| 2.8.2 | Tutor discusses arrangement with admin | Off-platform | Engaged; building rapport | |
| 2.8.3 | Arrangement agreed; match marked ACTIVE | Admin marks match ACTIVE in panel | Tutor receives N-11 email | N-11 must include the commission amount calculated |

**N-11 email copy:**


The illustration for admin contacts tutor off-platform contains the following information:

- Subject: Your tuition arrangement is now active.
- Hi [first name],.
- Your tuition arrangement for the selected subject — the selected level is now confirmed as active.
- Commission will be invoiced after the first session is confirmed by the tutee.
- Commission amount: SGD [calculated amount].
- Due: Within 14 days of invoice.
- You'll receive a formal invoice once the commission status is confirmed.
- If you have any questions, please reply to this email.
- Warm regards,.
- [Platform Name] Team.


---

### 2.9 Trial Session → Commission Invoice Received → Payment

| Step | User Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 2.9.1 | Tutor attends first trial session | Off-platform event | Focused — professional delivery | |
| 2.9.2 | Tutee confirms first session (or admin overrides) | Commission status → CONFIRMED | No notification to tutor at this step | |
| 2.9.3 | Admin generates commission invoice | N-15 invoice email sent to tutor | Slightly anxious about commission payment | TRUST-BUILDING Invoice must exactly match the calculation disclosed at registration |
| 2.9.4 | Tutor views invoice | Invoice PDF/HTML with: subject, level, calculation, amount, due date (14 days), payment instructions | Resigned acceptance if terms were clear upfront | Any discrepancy from disclosed amount causes disputes |
| 2.9.5 | Tutor pays commission via bank transfer / Pay Now | Off-platform | Satisfied if amount matches disclosed calculation | |
| 2.9.6 | Admin marks commission PAID | Commission status → PAID | No tutor notification (admin-side only) | |

---

## Flow 3: Admin — End-to-End Flow

### 3.1 Login with time-based one-time passcode multi-factor authentication

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.1.1 | Navigates to /admin/login | Admin login form renders: email, password fields; "Admin Login" heading | Efficient — work mode | Admin panel must be visually distinct from public site (purple tint) |
| 3.1.2 | Enters email and password | Credentials validated | Routine | |
| 3.1.3 | Prompted for time-based one-time passcode code | time-based one-time passcode input field renders; "Enter the 6-digit code from your authenticator app" | Opens authenticator app | Show time-based one-time passcode refresh countdown (30-second cycle visual) |
| 3.1.4 | Enters time-based one-time passcode code | Code validated; session established; redirected to /admin/dashboard | Ready to work | |

**Error states:**
- Wrong password (up to 5 attempts): "Invalid email or password" (deliberately generic — do not indicate which is wrong)
- Wrong time-based one-time passcode code: "Invalid code — please try again. Codes refresh every 30 seconds." (max 5 attempts before 15-minute lockout)
- multi-factor authentication not enrolled: redirect to /admin/mfa/enroll — cannot access any other admin route
- Account locked (5 failed attempts): "Your account has been temporarily locked. Please try again in 15 minutes."
- Session idle 30 minutes: web request protocol 401; forced re-authentication including full multi-factor authentication step

**multi-factor authentication enrollment (first login of new admin):**


The illustration for login with totp multi-factor authentication contains the following information:

- Secure Your Account.
- All admin accounts require multi-factor authentication.
- You must set up your authenticator app before accessing the admin panel.
- Step 1: Download Google Authenticator or any time-based one-time passcode-compatible app.
- Step 2: Scan the QR code below with your authenticator app.
- Step 3: Enter the 6-digit code to verify setup.
- [ QR Code display ].
- [ 6-digit code input ].
- [ Verify and Complete Setup ].


---

### 3.2 Post Review Queue

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.2.1 | Navigates to /admin/posts/review | Review queue table loads; sorted by submission time ascending (oldest first) | Efficient; prioritising older submissions | Default sort is oldest-first to enforce service-level agreement compliance |
| 3.2.2 | Scans queue table | Each row: post type badge (TUTEE/TUTOR), display name, subject, level, submission timestamp, time-in-queue, service-level agreement indicator | Quick triage | service-level agreement indicator is the rightmost column — draws eye to urgency |
| 3.2.3 | Identifies service-level agreement-at-risk posts | Green dot: <1 business day; Amber dot + "X hrs": 1–2 business days; Red badge "OVERDUE": >2 business days | Slight urgency for red items | Red "OVERDUE" badge must be unmissable |
| 3.2.4 | Clicks a post row | Navigates to per-post review detail | Focused review mode | |

---

### 3.3 Review Post Detail — Contact Info Visible to Admin Only

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.3.1 | Views post review detail page | Full post content rendered left column; contact info panel rendered right column (highlighted admin-only section with purple background) | Attentive; assessing quality | Purple background for contact info section makes admin-only status visually unambiguous |
| 3.3.2 | Views contact info section | System triggers immutable audit log entry: admin user ID, timestamp (Singapore time ISO 8601), record type, record ID, fields accessed; "Access logged" notice visible | Professional; aware of access log | Audit log confirmation notice reassures admin that the trail exists |
| 3.3.3 | Reviews all post fields | Level, subject, availability, rates/budget, location, qualifications, special requirements, photo (if uploaded) | Making approval decision | |
| 3.3.4 | For tutor posts: checks commission acknowledgement log | Commission acknowledgement log entry shown: timestamp, terms version, affirmative action type | Checking compliance — cannot approve without this | Admin Approve button is disabled until commission acknowledgements record exists |
| 3.3.5 | Reviews previous post history (if resubmission) | Prior revisions, rejection reasons, and admin notes shown inline | Contextualised review | |

**Admin-only contact info panel design:**


The illustration for review post detail — contact info visible to admin only contains the following information:

- [ ADMIN ONLY — Contact Information ].
- [ < lock icon > Access to this panel is logged automatically (immutable audit trail) ].
- Full Name: [name].
- Email: [email address].
- Phone: [phone number].
- Commission Acknowledgement (tutor posts only):.
- Acknowledged: [date/time Singapore time].
- Terms version: [version].
- Action: checkbox checked.


---

### 3.4 Approve / Reject / Request Revision

**Approve:**

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.4.1 | Clicks "Approve" button | Confirmation modal: "Approve this post? This will make it live on the platform." | Deliberate | Modal prevents accidental approvals |
| 3.4.2 | Confirms approval | Post status → Active; email N-02/N-03 sent within 5 min; success toast shown | Satisfied; moving to next item | Toast: "Post approved. Notification sent to user." |

**Reject:**

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.4.3 | Clicks "Reject" button | Rejection reason dropdown renders (enum values mapped to human labels); optional free-text note field | Careful — rejection has impact | Rejection action is red (btn-destructive) to signal consequence |
| 3.4.4 | Selects rejection reason from enum | Dropdown selection required; cannot proceed without a reason | Deliberate | |
| 3.4.5 | Optionally adds internal note | Free text; not shown to user | For team handoff | |
| 3.4.6 | Clicks "Confirm Rejection" | Post status → Rejected (terminal); N-06/N-07 sent; toast: "Post rejected. Notification sent." | Done | |

**Request Revision:**

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.4.7 | Clicks "Request Revision" | Text area opens for admin message | Diplomatic — wants to help user improve | Label: "Message to user (they will see this in their dashboard and email)" |
| 3.4.8 | Types revision message | Character count shown; 1000 char limit | Thoughtful | |
| 3.4.9 | Clicks "Send Revision Request" | Post status → Revision Requested; N-04/N-05 sent; toast: "Revision request sent to user." | Efficient | |

---

### 3.5 Matching Dashboard — Two-Panel Layout

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.5.1 | Navigates to /admin/matching | Two-panel layout: Left = Active tutee requests (not currently Matched); Right = Active tutor offerings; shared filter bar at top | Strategic — scanning for compatible pairs | Panel counts shown: "23 requests" / "87 listings" |
| 3.5.2 | Applies subject filter | Both panels update simultaneously | Focused | Filters apply to both panels by default; toggle to pin to one panel |
| 3.5.3 | Applies level filter | Both panels update | Narrowing | |
| 3.5.4 | Applies area/MRT filter | Both panels update; remaining items are geographically compatible | Near-decision | |
| 3.5.5 | Reviews budget/rate overlap | Rate range on tutor cards vs budget range on tutee cards; "Show compatible rates only" toggle | Assessing financial fit | |

**Filter controls (both panels):**
- Subject: single select dropdown (full Singapore taxonomy, grouped)
- Level: single select dropdown (full MOE level list)
- Venue type: multi-select chips
- Area/MRT: text search + select (planning area or MRT station)
- Rate/budget overlap toggle: "Show only compatible rate ranges" (filters tutee posts where budget max ≥ tutor rate min)
- Pin to single panel: toggle icon per filter control

---

### 3.6 Create Match

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.6.1 | Clicks tutee post card in left panel | Card highlighted with selection ring (navy); "1 tutee selected" shown in match bar | Deliberate | |
| 3.6.2 | Clicks tutor post card in right panel | Card highlighted; "Create Match" button activates in match action bar between panels | Confident in the pairing | Button only activates when one card from each panel is selected |
| 3.6.3 | System checks cardinality before showing modal | If tutee post already has pending/active match: error banner "This tutee request already has an active match. Cancel the existing match first." Stays on panel — no modal opened | Error handled; selects different tutee | |
| 3.6.4 | Clicks "Create Match" (cardinality check passed) | Confirmation modal opens with full summary | Verifying before committing | |
| 3.6.5 | Reviews match confirmation modal | Shows: tutee summary (subject, level, area, budget, schedule) + tutor summary (display name, subjects, rate, area, availability); "Confirm Match" button | Final review | No tutee contact info in this modal — it's a content review, not contact disclosure |
| 3.6.6 | Clicks "Confirm Match" | Match record created (status PENDING); commission record created (commissions table, status PENDING); tutee post → Matched (removed from discovery); both parties receive N-08/N-09; toast: "Match created. Both parties have been notified." | Satisfied | |

---

### 3.7 Mark ACTIVE — Trigger First-Session Confirmation

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.7.1 | After off-platform introduction confirmed, navigates to match detail | Match detail shows "Mark as Active" action button; current status PENDING | Ready to update status | |
| 3.7.2 | Clicks "Mark as Active" | Commission calculation form appears: agreed rate input (overrides post rate if different), hours per session, sessions per month; "If frequency = Flexible: must enter agreed sessions per month" | Reviewing calculation | Required fields before ACTIVE transition |
| 3.7.3 | Reviews and confirms commission calculation | agreed rate × hours per session × sessions per month × 0.5 displayed | Satisfied if correct | Commission amount shown in SGD with calculation breakdown |
| 3.7.4 | Confirms "Mark as Active" | Match status → ACTIVE; commission status → AWAITING CONFIRMATION; N-10 sent to tutee (within 5 min, commission-critical service-level agreement); N-11 sent to tutor | Done; awaiting tutee confirmation | |

**7-day monitoring — admin panel states:**

| Day | Admin Panel State | Admin Action Available |
|---|---|---|
| Day 1–7 | Match shows "Awaiting Tutee Confirmation" (blue label) | Override available via button but not prominently prompted |
| Day 7+ (timeout reached — OVERDUE flag set by cron) | Match shows red "Overdue Confirmation" flag; N-13 admin alert already sent | Override confirmation button prominently displayed |
| After 14 days no action | Escalation warning banner on match: "Admin action required" | Must choose: Override → CONFIRMED, OR Cancel match → CANCELLED |

---

### 3.8 Commission Invoice Generation

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.8.1 | Commission ledger shows match with status "CONFIRMED — Ready to Invoice" (amber badge) | Ledger row shows "Generate Invoice" action button | Ready to proceed | |
| 3.8.2 | Navigates to the specified commission accessible to an administrator | Invoice preview renders with all required fields per product requirements document §8.4 | Reviewing for accuracy | Preview allows correction before sending |
| 3.8.3 | Reviews invoice fields | Tutor display name, tutee reference (first name + subject + level), agreed rate, session calculation, monthly fee, commission rate, commission amount, due date (invoice date +14 days), payment instructions, agency contact for disputes | Checking accuracy | Any discrepancy from the N-11 email commission amount is a user experience failure |
| 3.8.4 | Clicks "Generate and Send Invoice" | Invoice finalised; N-15 sent to tutor; commission status → INVOICED; invoice reference number assigned | Satisfied; commission tracking active | |
| 3.8.5 | Receives payment off-platform | — | Monitoring | |
| 3.8.6 | Clicks "Mark as Paid" in commission ledger | Commission status → PAID | Satisfied — flow complete | |

**Commission ledger row status colours:**
- PENDING: grey — "Awaiting arrangement to start"
- AWAITING CONFIRMATION: blue — "Waiting for tutee confirmation"
- CONFIRMED: amber — "Ready to invoice"
- INVOICED: orange — "Invoice sent — due [date]"
- PAID: green — "Paid [date]"
- DISPUTED: red — "Under dispute — notes required"
- WAIVED: grey strikethrough — "Waived — [reason]"

---

### 3.9 Additional Admin Match Actions

**Cancel match:**

| Step | Admin Action | System Response | Emotional State | Trust-Building Notes |
|---|---|---|---|---|
| 3.9.1 | Clicks "Cancel Match" on match detail | Confirmation modal with optional reason field | Deliberate — has downstream effects | Modal must show downstream effect: "Tutee post will return to Active (re-enters discovery)" |
| 3.9.2 | Optionally enters cancellation reason | Free text | Thoughtful | |
| 3.9.3 | Confirms cancellation | Match status → CANCELLED; commission status → WAIVED; tutee post → Active (re-enters discovery); N-16 sent to tutee; N-17 sent to tutor | Administrative closure | |

**Post-INVOICED cancellation warning:**


The illustration for additional admin match actions contains the following information:

- This match has an outstanding commission invoice.
- Cancelling the match does NOT automatically void the invoice.
- You must separately set the commission status to WAIVED or DISPUTED.
- and document the reason in the notes field.
- [ I understand — proceed with cancellation ] [ Cancel ].


After cancellation with outstanding invoice, admin must explicitly choose:
- "Waive Commission" (with required notes field) → commission status → WAIVED
- "Mark as Disputed" (with required notes field) → commission status → DISPUTED

---

### 3.10 Commission Tab — Calculate, Generate Invoice, Track Payment

*(This step summarises the commission lifecycle from the admin perspective, combined view)*

| Admin Task | Navigation | Action |
|---|---|---|
| View all commission records | /admin/commission | Filter by status, date range, tutor; view ledger table |
| Generate invoice | the specified commission accessible to an administrator | Preview and send (only available when commission status = CONFIRMED) |
| Mark PAID | Commission ledger "Mark as Paid" button | Commission status → PAID |
| Override confirmation | Match detail "Override Confirmation" button | Commission status → CONFIRMED (logged with admin user id + timestamp) |
| Mark DISPUTED | Commission ledger / match detail | Requires notes field; commission status → DISPUTED |
| Waive commission | Commission ledger / match detail | Requires notes field; commission status → WAIVED |

---

## Appendix: Error State Summary

| Error | Trigger | User-Facing Copy | Recovery |
|---|---|---|---|
| Email already registered | Registration submit | "An account with this email exists. Log in instead?" | Link to login |
| Phone format invalid | Registration submit | "Please enter a Singapore mobile number (+65 8/9XXXXXXX)" | Correct and retry |
| Verification link expired | Click on expired email link | "This link has expired. Request a new verification email." | Resend button |
| Commission checkbox not checked | Tutor clicks Continue on disclosure | "You must accept the commission terms to continue." | Scroll to checkbox |
| Budget max < min | Post wizard Step 1 | "Your maximum budget must be at least equal to your minimum." | Correct and retry |
| No availability slot | Post wizard Step 2 | "Please select at least one availability slot." | Select slot and retry |
| Tutee post already matched | Admin creates second match | "This tutee request already has an active match. Cancel the existing match first." | Cancel existing or select different tutee |
| time-based one-time passcode code invalid | Admin login | "Invalid code — codes refresh every 30 seconds." | Retry with fresh code |
| Admin session timeout | Admin application programming interface call after 30min idle | web request protocol 401; redirect to login | Full re-authentication |
| Network error on post submit | Submit button | "Something went wrong. Your draft has been saved — please try again." | Retry button |
| Post already confirmed (idempotent) | Tutee re-clicks confirm session | web request protocol 200; "Your confirmation was already received. No further action needed." | No action needed |
| Commission acknowledgement not logged | Admin tries to approve tutor post | Approve button disabled; warning: "Commission acknowledgement not yet completed by this tutor." | Tutor must complete disclosure step |
| Role conflict at registration | Email already has opposite role | "You already have an account registered as a [tutee/tutor]. One account may only hold one role." | Log in with existing account |
| Verification link expired | Click on expired link | "This link has expired. Request a new verification email." | Resend button |
