# Parent-shopper review — InTuition SG (deployed dev http://[REDACTED IP ADDRESS])

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

**Persona:** veteran SG parent, has used an established agency + online marketplaces + agency WhatsApp groups.
**Date:** 2026-06-23. **Account:** REDACTED email (fresh tutee, [REDACTED PHONE]).
**Browser:** Playwright MCP. Evidence: parent-NN-* implementation reference.

---

## Would-I-abandon-for-a-competitor (lead findings)

The two things that would actually make me hesitate are both **silent failures** — the system tells me nothing
is wrong while quietly doing nothing:

1. **The "Invite a tutor to my job" button does nothing and says nothing (S2).** I opened a tutor, clicked
   *Invite to my job*, picked my (pending) request, wrote a message, hit *Send Invitation*. The dialog closed as if
   it worked — **no error, no toast**. But the network call submit operation for tutee posts id invite returned **404**,
   and my dashboard's *Invitations sent* list still says *"You haven't invited any tutors yet."* On an agency
   WhatsApp group I'd at least know my message didn't go through. Here I'd assume the tutor was invited and wait
   for a reply that never comes. (Notably the *quota* call GET …/invite-quota returns 200 — so the frontend is hitting
   a route the backend doesn't expose.)

2. **The post wizard can silently jam on Step 1 with no error (S2).** If I pick "6 or more" students (which reveals
   an *Exact number* box) and then change my mind back to "1 student", the **Continue button stops working** — it
   highlights but never advances to Step 2, and there is **no message** telling me why. I thought the site was
   broken. (Reproduced deterministically.)

Everything *else* about the demand-side journey is genuinely good — arguably better than the agencies I've used on
privacy and on the "meet before you commit" promise.

---

## Findings

| ID | Sev | Screen / URL | What I saw | How a competitor does it / why it matters | Evidence |
|----|-----|--------------|------------|--------------------------------------------|----------|
| P1 | **S2** | Tutor detail the tutor's profile-detail page → *Invite to my job* | Click *Send Invitation* → dialog closes silently. submit operation for tutee posts id invite → **404**. Dashboard *Invitations sent* still empty ("You haven't invited any tutors yet"). The quota GET (…/invite-quota) returns 200, so only the POST route is wrong/missing. The headline new "invite while pending" feature is broken at submit. | Marketplaces show "Invitation sent ✓" and list it. A swallowed 404 = I believe I contacted a tutor and I didn't. Worst kind of fail for a money/trust flow. | parent-04, parent-06 |
| P2 | **S2** | Post wizard /tutee/posts/new Step 1 | Select "6 or more" students (reveals *Exact number* field), then switch back to "1 student" → **Continue is dead**, stays on "Step 1 of 4", **no error shown**. Confirmed repro with Maths using PSLE+P5. A stale required-state on the now-hidden exact-number field blocks submit invisibly. | Every decent wizard either re-validates on change or shows the blocking field in red. A button that does nothing with no message reads as "site is broken" — I'd leave. | parent-02, parent-10 |
| P3 | S3 | Browse/landing tutor cards /tutors | Most tutors display as raw seed handles — "tutor joanne chua", "tutor faizal hassan", "tutor rachel teo". Only one was nicely initialised ("Tan W."). | Agencies show "Joanne C." / "Mr Faizal H." Raw tutor * usernames look like un-finished test data and cheapen trust on the very page where I'm judging quality. | parent-09 |
| P4 | S3 | Wizard review (Step 4) vs. dashboard *View* dialog | Step-4 review and the *View request* dialog **omit session duration (1.5 hrs) and frequency (Twice/week)** that I entered — the View dialog only lists subject, budget, venue, schedule using days, students, status, notes. Wizard review shows "Schedule: 20 slots · 1.5 hrs · Twice/week" but the saved-record view drops duration/frequency. | I want to confirm what I asked for. Dropping the cadence I set makes me unsure it was captured. | parent-03 |
| P5 | S3 | Wizard Step-2 schedule (mobile + desktop) | The "Weekday evenings" preset selected my availability but the review summarises it as **"20 slots"** — an opaque count, not "Mon–Fri evenings". (The dashboard *View* dialog DOES render it nicely as Mon–Fri Evening, so the data is fine — just the wizard-review wording is cryptic.) | "Mon–Fri, 6–9pm" is what a parent understands; "20 slots" is engineer-speak. | parent-03, parent-08 |
| P6 | S4 | Mobile wizard Step-2 /tutee/posts/new @390px | The fixed bottom nav bar (My Requests / Find Tutors / Interviews / Matches) overlaps the schedule grid — the Fri row sits partly behind it; needs scrolling around the bar. Also the time-block size defaults to **"2 hours"** which is an odd default for most 1-on-1 parents. | Minor; bottom-bar should not occlude form content, and 1-hour granularity is the more common starting point. | parent-08 |
| P7 | S4 | Profile /tutee/profile (change-password form) | Console DOM warnings: password form lacks a hidden username field and autocomplete is current-password attributes (a11y / password-manager hint). | Cosmetic; password managers and screen readers prefer these. | (console log) |
| P8 | S4 | Register /register?role is tutee | Stepper says "Step 1 of 2 — Account / Profile" but tutee registration completes in **one** step (submitted straight to dashboard, no Profile step). | Mild expectation mismatch; either drop the 2-step indicator for tutees or show the profile step. | parent-01 (flow) |

---

## What worked — better than competitors / improved ✓

- **Trust on landing is strong and specific.** "Meet your tutor before you commit", "your contact details are
  never shared with any tutor, ever", "Every tutor vetted", a trust band ("100% admin-vetted", "Contact info —
  always private", "Personal Data Protection Act Compliant"), and an honest **commission line** stated plainly: *"Our only charge is a
  one-time commission of 50% of the first month's tuition fee, billed to the tutor after a successful match — no
  subscriptions, no listing fees,"* plus *"No agency fee for parents."* That's clearer than most agencies, which
  hide the cut. (parent-01)
- **New taxonomy is there and sensible.** Combined Humanities (O-Level), the N-Level subjects (English/Maths/Science
  N-Level), the **H1 sciences** (H1 Physics/Chemistry/Biology), Pre-School/lower-primary, Poly and University —
  all present and selectable. Picking an O-Level subject correctly **filters the Level list** to Secondary levels
  only. (snapshots)
- **"6 or more" → exact count works** — reveals "Exact number of students" with helper "Enter a number between 6
  and 20." (the #205 fix). The only problem is the revert path (P2).
- **Budget min>max is caught** — "⚠ Max budget must be ≥ min budget", blocks Continue, and **re-clears both ways**
  when fixed. (the cross-field fix)
- **Step-4 Review does NOT auto-submit** — it's a read-only summary with a distinct *Submit Request*; submit gives
  honest copy: "Our team reviews all requests within 1 business day. You'll get an email once it's approved and
  tutors can start applying." (parent-03)
- **Post is visible + editable while PENDING.** Dashboard shows it as "Pending Review" with View / Edit / View
  applicants / Close job, and the View dialog renders schedule as Mon–Fri Evening. (dashboard snapshots)
- **Invite/Interview-while-pending copy is honest and reassuring.** Both dialogs label my request
  "(awaiting approval)" and say *"Your request is awaiting approval — you can invite/request now. The tutor's
  contact is only shared once your request is approved."* Exactly the transparency a skeptical parent wants. The
  **interview** path actually works end-to-end and persists ("Awaiting tutor with Tan W. — Awaiting scheduling by
  our team" in My Interviews) — it's only the **invite** path that 404s (P1). (parent-04, parent-05)
- **Contact-safety is excellent.** Tutor detail repeats "Contact details are never shown." Changing my own
  email/phone requires my **current password** AND a **confirmation link emailed to my account email**, with the
  old value staying active until confirmed — strong account-takeover protection, better than typical agencies.
  WhatsApp notifications are opt-in/off with clear consent copy. (profile snapshots)
- **Mobile @390px parity is good** — hamburger + bottom nav, single-column cards, schedule grid transposed to
  time×day with a sticky day column and a "← Swipe to reach later hours →" hint. (parent-07/08/09)

## Console / network notes
- read operation for auth me → 401 when logged out (benign).
- submit operation for tutee posts 476120b8-… invite → **404** (P1, the real defect); …/invite-quota → 200.
- No JS exceptions or 500s observed anywhere in the journey.
