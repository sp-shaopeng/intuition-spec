# Tutee/Parent re-run — deep logged-in flows (2026-06-22)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

External reviewer, LOCAL (frontend  / application programming interface ), Playwright MCP. Fresh login per identity.

## 1. Deep flows reached

| # | Flow | Identity | Result |
|---|------|----------|--------|
| 1 | Tutee match dashboard (completed match) | Mrs Lim | WORKED |
| 2 | Contact reveal on completed match | Mrs Lim | WORKED (minor href nit) |
| 3 | Leave a review (NEW flow) + appears on tutor public detail | Mrs Lim | WORKED |
| 4 | Interview — tutee side (scheduled) | Hidayah | WORKED (1 consistency defect) |
| 5 | Invitation sent (pending invite) | Bee Hoon Tay | BLOCKED — no pending invite exists in live data |

### Flow 1 — Match dashboard (WORKED)
/tutee/matches renders the completed match: "Mr Tan Wei Jie", headline "PSLE & Lower Sec Math Specialist — ex-MOE", subject "Mathematics (PSLE)", **Completed** badge, "Matched 22 Jun 2026", helper text "This match is complete; contact details remain available." Reveal-contact button + Leave-a-review link present. Evidence: evidence/tutee-rerun-01-matches png.

### Flow 2 — Contact reveal (WORKED)
Clicking "Reveal contact details" expands an inline card labelled "Contact (shared now you've committed)" with Name = Mr Tan Wei Jie, Phone [REDACTED PHONE] (tel link), Email [REDACTED EMAIL] (mailto link). Only available because the match is committed/completed — correct gating. Evidence: evidence/tutee-rerun-02-contact-reveal png.

### Flow 3 — Leave a review (NEW, WORKED end-to-end)
the review action for the selected match form: 5-star rating group, optional comment (0/2000 counter), Cancel/Submit. Submitted 5 stars + comment → toast "Thanks for your review!", redirect to matches, card now shows "✓ Review submitted" (link removed, blocks duplicate). Verified the review then appears on the tutor's PUBLIC detail page (/tutors/315e7b3b-...): aggregate became "5.0 (2 reviews)" and Mrs Lim's review renders "just now" with full comment text. Evidence: evidence/tutee-rerun-03-review-form png, 04-review-submitted png, 05-review-on-public-detail png.

### Flow 4 — Interview, tutee side (WORKED)
Hidayah → /tutee/interviews lists a **Scheduled** interview; detail page (the tutee's interview-detail page) shows: Scheduled badge, "Interview scheduled — 24 Jun 2026, 09:00 am Singapore time", a live **Join Meeting →** link (https://meet google.com/abc-defg-hij), and a **negotiation timeline** ("Round 1 · proposed by Tutee") with 3 proposed slots — Wed 24 Jun marked **Agreed**, the other two **Not chosen**. "Cancel this interview" + the privacy note are present. Clear and complete. Evidence: evidence/tutee-rerun-06-interview-detail png.

### Flow 5 — Invitation sent (BLOCKED — data state mismatch)
The brief expected Bee Hoon Tay to have a **PENDING** invitation to Wei Jie. Live data does not match: database job invitations shows her invitation to wei jie.tan is **ACCEPTED** (not PENDING), on tutee post a540f788… (Science PSLE), which is now **MATCHED**. There are **zero PENDING invitations in the entire database** — the admin pass that ran before this session accepted/matched it. So the "invitation-sent / pending" tutee state could not be exercised. What I could confirm instead: the accepted invitation **folded correctly into the apply→choose funnel** (INT-044 design) — on the applicants page for a tuition request the tutor renders as the single applicant with status "Assigned to you", and the job shows "This job has been assigned." See ISSUE-3 for a related structural gap. Evidence: evidence/tutee-rerun-07-applicants-assigned png.

## 2. Issues

### ISSUE-1 · tel link contains unescaped spaces · S4 (cosmetic/minor)
- Where: Mrs Lim → /tutee/matches → Reveal contact details → Phone link.
- Saw: anchor href = tel: REDACTED phone (raw spaces).
- Expected: a valid tel URI strips spaces, e g. tel: REDACTED phone. Some dialers/OSes fail to parse spaces in tel.
- Evidence: evidence/tutee-rerun-02-contact-reveal png.

### ISSUE-2 · Tutor shown as raw handle "tutor wei jie tan" on the interview + applicants screens · S3 (consistency/polish)
- Where: Hidayah → /tutee/interviews and /tutee/interviews/id ("with tutor wei jie tan"); also Bee Hoon Tay → /tutee/requests/id/applicants (applicant heading "tutor wei jie tan").
- Saw: the tutor is rendered as the raw system handle tutor wei jie tan.
- Expected: a friendly display name — the **Matches** screen for the same tutor correctly shows "Mr Tan Wei Jie", proving a display name is available. The interview and applicants surfaces are inconsistent with it, and the raw handle reads like a leaked internal identifier to a parent.
- Evidence: evidence/tutee-rerun-06-interview-detail png, evidence/tutee-rerun-07-applicants-assigned png.

### ISSUE-3 · No tutee-facing list of sent/pending invitations · S3 (user experience gap)
- Where: tutee app globally. After a parent sends a job invitation to a tutor, the "Invited" pill on the tutor detail page is **session-only** (invited Post Ids local state in Tutor Detail implementation reference); on reload it disappears. There is a withdraw endpoint (delete operation for tutee invitations id) and a quota endpoint, but **no GET to list the parent's invitations** (api Routes implementation reference — tutee side has post Invite, invite Quota, invitation Withdraw only; the LIST route tutor invitations interface operation is tutor-scoped).
- Saw: a parent who sends an invite, then comes back later, has no persistent place to see "I invited tutor X; status PENDING/ACCEPTED/DECLINED." The Invite Tutor Modal's "Manage pending invites" link points to the applicants page for a tuition request, but a PENDING (not-yet-applied) invite does not surface there as an invitation row.
- Expected: a parent can see the invitations they sent and their status, from a stable URL.
- Note: could not be demonstrated against live data because there are zero PENDING invitations seeded (see Flow 5). Filed as a structural/code-grounded observation, severity S3.

## 2a. 390px spot-checks (all PASS, no overflow/clipping)
- Applicants page: evidence/tutee-rerun-08-390-applicants png
- My Matches (completed card): evidence/tutee-rerun-09-390-matches png — note the tutor IS correctly "Mr Tan Wei Jie" here.
- Contact reveal card: evidence/tutee-rerun-10-390-contact png
- Interview detail (timeline + join link): evidence/tutee-rerun-11-390-interview png
- Dashboard requests table collapses to stacked cards; hamburger menu + bottom Primary nav both work. No horizontal scroll observed.

## 3. Still blocked
- **Flow 5 (invitation-sent / pending state)** — not testable: no PENDING invitation exists in the live database. Bee Hoon Tay's invite to Wei Jie is ACCEPTED and the job is MATCHED. Needs a re-seed with a genuinely PENDING tutee→tutor invitation (and ideally a separate OPEN job) to exercise the "invitation sent" tutee view.
