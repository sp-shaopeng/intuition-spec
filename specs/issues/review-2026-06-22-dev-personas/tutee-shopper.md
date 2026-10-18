# Tutee Shopper — Demand-Side Review (dev, http://[REDACTED IP ADDRESS])

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

**Persona:** Singapore parent who has used an established agency, online marketplaces, and
agency WhatsApp groups. Comparison-shopping, slightly skeptical, time-poor.
**Date:** 2026-06-22 · **Browser:** Playwright MCP
**Account registered:** REDACTED email / [REDACTED PHONE] (fresh tutee)
**Console/500s:** No 500s, no render crashes anywhere in the journey. Only one benign 401
auth me interface operation while logged-out (expected auth probe).

---

## The one thing that would make me hesitate vs a competitor

**I cannot act on a tutor I like until an admin approves my post (up to "1 business day").**
I registered, posted a detailed A-Math request, then opened a 5-star A-Math tutor and hit both
"Request an Interview" and "Invite to my job" — both were **blocked**, because my post is still
"Pending Review". On an agency WhatsApp group I'd be talking to a coordinator in minutes; on
marketplaces like Tueet/Smile Tutor I can contact/shortlist instantly. Here, the moment my
intent is highest (I'm looking at the tutor I want), the product makes me **wait a day**. That's
the single biggest abandonment risk. (Detail S2-1 below — the gate is defensible for vetting,
but the *copy* makes it worse by claiming I have no request when I clearly do.)

---

## Findings

| ID | Sev | Screen / URL | What I saw | How a competitor does it / why it matters | Evidence |
|---|---|---|---|---|---|
| **S2-1** | S2 | Tutor detail the tutor's profile-detail page — interview & invite modals | With a freshly-posted request in **Pending Review**, the Interview modal says *"Post a request first … You don't have an active request yet."* and the Invite modal says *"No active requests — You need an active request to invite a tutor."* Both CTAs disabled. But I **do** have a request — it's awaiting approval. | The message is factually wrong from my point of view and sends me to /tutee/posts/new to post a **duplicate**. Competitors let me shortlist/contact immediately, or at least say *"Your request is awaiting approval — you can invite tutors once it's live."* Mislabelling my pending post as "no request" erodes trust and risks duplicate posts. | tutee-09-interview-gated-on-approval png |
| **S2-2** | S2 | Landing / — "Recent job requests" strip | The strip shows the **same job 7×** in a row: *"Primary 5 · Mathematics (PSLE) · 1 student · S$30–60/hr · Singapore · 2× per week · 1.5 hrs · Flexible start · posted today"* (only the 8th card differs — Sec 4 A-Math). | As a shopper this screams "fake/seed data" and kills the social-proof the section is meant to create. Real marketplaces show varied recent activity. Either dedupe or seed varied demand. The repetition is the most "demo-ish" thing on the site. | tutee-01-landing png |
| **S3-1** | S3 | My request details dialog /tutee/dashboard (View) | The "Request details" dialog shows Subject, Budget, Venue, Students, Status, Special requirements — but **omits the Schedule** (the 20 availability slots, 1.5 hr duration, Twice/week) I spent the most effort entering in Step 2. | I want to verify what I submitted, especially the part that took longest. A competitor echoes back everything. Showing budget/venue but hiding schedule feels like data was dropped (it wasn't — Step-4 review showed "20 slots · 1.5 hrs · Twice/week"). Add schedule to the detail dialog. | tutee-06-request-details-no-schedule png |
| **S3-2** | S3 | My request row /tutee/dashboard | Actions are only **View / View applicants / Close job**. There is **no Edit**. If I picked the wrong level or budget, my only path is Close + re-post (and wait for re-approval again). | Every platform I've used lets me edit a live request. No-edit forces a destructive close→repost loop and another 1-day approval wait. At minimum allow editing while "Pending Review". | tutee-06-request-details-no-schedule png |
| **S4-1** | S4 | Tutor detail — Availability the tutor's profile-detail page | Availability chips render with **no space**: "Mon Evening", "Tue Evening", "Sat Afternoon". | Looks unpolished on an otherwise strong profile page. Should read "Mon Evening" / "Mon · Evening". | tutee-08-tutor-detail png |
| **S4-2** | S4 | Post wizard Step 1 — Number of students | Dropdown still reads **"6 or more"** as the top option. (Project notes say Spec 009 changed group size to exact 6–20.) | Minor copy/spec drift — "6 or more" vs an exact count. As a parent of a 2-kid group I'd want to say exactly how many. Low impact. | tutee-02-wizard-step1 png |
| **S4-3** | S4 | Browse Tutors /tutors | Only **1 of 14** tutors has any review (a single 5-star). Most cards show only "X yrs experience". | Reviews are the #1 thing I shop on. Thin review coverage is expected pre-launch, but the listing leans almost entirely on self-declared experience. Not a defect — flag for go-live: seed/solicit reviews, or surface "vetted by our team" more prominently per card to compensate. | tutee-07-browse-tutors png |
| **S4-4** | S4 | Mobile post wizard Step 2 /tutee/posts/new @390px | The fixed **bottom nav bar overlaps** the availability grid (sits over the Fri row). Grid is still scrollable and the day-column sticks, so it's usable. | Minor visual overlap on small screens; consider bottom padding so the last grid rows clear the nav bar. | tutee-13-mobile-wizard-step2-grid-390.png |

---

## What earned trust / better than competitors ✓

- **Privacy is the headline and it's consistent.** "Contact info never shared — ever",
  "Meet your tutor before you commit", "No agency fee for parents", Personal Data Protection Act line + UEN in the
  footer. The promise is repeated at every decision point (landing, browse, tutor detail,
  wizard Step 3/4, profile). This is genuinely more reassuring than agency WhatsApp groups
  where my number gets passed around. (tutee-01, tutee-08)
- **The interview-before-commit hook is a real differentiator.** "Request a free 15-min video
  call with any tutor, admin coordinates everything." No other SG platform I've used offers a
  structured pre-match interview. Strong reason to choose this over a marketplace.
- **Post wizard is clean and safe.** 4 steps, clear progress, rich taxonomy (55 subjects /
  28 levels), schedule quick-presets ("Weekday evenings"), and explicit safety hints ("Do not
  include your phone, email, or home address here"). (tutee-02 / tutee-03 / tutee-05)
- **Budget validation works correctly.** min 80 > max 50 → field marked invalid, inline
  "Max budget must be ≥ min budget", and the wizard **blocked** me from proceeding. (tutee-04)
- **Step-4 Review does NOT auto-submit.** It shows a true summary with distinct Back / Submit
  buttons; submit produced a clean "Request submitted … you'll get an email once approved."
  (The known Spec-008 wizard bug is fixed in this build.) (tutee-05)
- **Tutor detail page is comprehensive:** subjects×levels, "NIE-trained, 6 years" qualification,
  teaching approach, availability, venue, rate band, group size, and a real review with text
  ("turned my son's A-Math around … from failing to a B3"). Enough to choose on. (tutee-08)
- **Tutor identity is privacy-first** — handles ("tutor hui ling goh"), no real names or
  contact on public listings.
- **Best-in-class contact-safety on my own profile.** Email/phone changes require my **current
  password** AND an **email-confirmation link**, with current details staying active until I
  confirm. Phone verification + WhatsApp opt-in with explicit, plain-English consent ("we never
  send marketing on WhatsApp, opt out whenever"). Soft-delete Danger Zone. My own email/phone
  shown in plaintext to me (correct — that's my data). This is more careful than most platforms
  I've used. (tutee-10)
- **Mobile @390px is solid** — nav collapses to hamburger + bottom tab bar; the availability
  grid gets a "Swipe to reach later hours →" hint with a sticky day column. (tutee-11 / tutee-13)
- **Outbound-requests dashboard exists** — "Invitations sent" and "Interview requests" sections
  are already on the dashboard, so once a request is approved my sent actions have a home to show
  up in (couldn't fully exercise this because of the S2-1 approval gate). (dashboard snapshot)

**Net:** the privacy/interview-first proposition and the careful account-security user experience are
genuinely differentiated and built trust. The thing that would make *this* shopper pause is
**S2-1** — being told I have "no request" and being unable to invite/interview the tutor I
came to hire until an admin approves me a day later — compounded by **S2-2** (the duplicated
"recent jobs" strip reading as fake) and the no-edit limitation (S3-2).
