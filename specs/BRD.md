# Business Requirements Document

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).


**Project status:** InTuition is not yet live. Launch is paused because the founder has been busy with life. This document states business requirements and historical planning assumptions; it does not report customer adoption or revenue.

**Reading this record:** The original draft is dated 27 May 2026. The September 2026 decision register supersedes conflicting older rules. This public edit puts those current rules beside the affected requirements. It does not verify their implementation. See the [current requirements](CURRENT-REQUIREMENTS.md) and [business and product overview](../docs/business-and-product.md).

## InTuition — Singapore Private Tuition Marketplace Platform

**Document Version:** 1.3 — Rebranded to InTuition; Tech Stack: React + TypeScript / Java Spring Boot  
**Date:** 2026-05-27  
**Status:** Requirements record; implementation verification remains separate  
**Prepared by:** The founder with AI assistance. Historical specialist reviews were AI review roles, not employees or independent human certification.  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Business Context](#2-business-context)
3. [Business Model](#3-business-model)
4. [Business Goals](#4-business-goals)
5. [Stakeholders](#5-stakeholders)
6. [User Needs](#6-user-needs)
7. [Competitive Landscape](#7-competitive-landscape)
8. [Key Business Rules](#8-key-business-rules)
9. [Success Metrics](#9-success-metrics)
10. [Risks, Constraints & Non-Functional Requirements](#10-risks-constraints--non-functional-requirements)
11. [Out of Scope for MVP](#11-out-of-scope-for-mvp)
12. [Singapore-Specific Requirements](#12-singapore-specific-requirements)
- [Appendix A — Post State Transition Table](#appendix-a--post-state-transition-table)
- [Appendix B — Personal Information Field Register](#appendix-b--personal-information-field-register)
- [Appendix C — Data Governance and Retention Policy](#appendix-c--data-governance-and-retention-policy)
- [Appendix D — Role Permission Matrix](#appendix-d--role-permission-matrix)

---

## 1. Executive Summary

InTuition is a proposed Singapore tuition service supported by a two-sided marketplace. Parents and adult learners describe what they need; tutors describe what they can teach and apply to suitable requests. A coordinator reviews posts, helps arrange interviews, and confirms assignments after both parties agree.

The aim is to make the steps between an enquiry and a first lesson clearer. Users can express preferences, but a parent's selection does not itself create a match. A complete tutor profile is sufficient for application and assignment; a published tutor listing is optional.

The proposed revenue model is a one-time commission paid by the tutor to the agency. The current default is 50% of an agreed first-month fee. Activation locks the amount, first-session confirmation gates invoicing and due handling, and agency receipt permits recording the commission as paid.

## 2. Business Context

### 2.1 Singapore Private Tuition Market

The project began with the assumption that some families and tutors would value help finding a suitable arrangement. The original draft's market-size and participation estimates were not independently verified and are not used here as public facts. The product needs to test demand with prospective users.

The intended matching criteria are concrete: subject, level, general location, budget, availability, lesson length and frequency. Curriculum and terminology need maintenance as educational requirements change.

### 2.2 Tutor Supply Landscape

These are intended user groups, not measured shares of the tutor market:

| Segment | Possible needs to investigate |
|---|---|
| Undergraduate and other part-time tutors | Suitable assignments around study or work commitments |
| Full-time freelance tutors | A reliable way to find students and fill available teaching hours |
| Former teachers | A way to explain their experience, subjects and available schedule |
| Subject specialists | Requests that fit a particular subject, qualification or teaching approach |

### 2.3 Why Use an Agency-Mediated Model?

The proposed service gives the coordinator responsibility for reviewing posts, following up on interest and confirming the arrangement. Tutors can apply and parents can express a preference, so the coordinator does not have to discover every candidate unaided.

Public and pre-assignment views exclude private contact details. The participant interface reveals contact only for authorized active or completed matches. Reviewing a post is not a guarantee of teaching quality; any wider verification claim needs a defined process and evidence.

The business question is whether this help is valuable enough to justify the fee and the staff time involved. No comparison with another provider, reduction in unwanted contact, or improvement in match quality has been established by this record.

### 2.4 Pre-Match Interview Sessions

A parent or learner may request a short conversation, intended to last up to 15 minutes, before committing to a tutor. The purpose is to discuss suitability and expectations. The platform supports proposed times, counter-proposals, an agreed slot, a meeting link and a recorded outcome. See the current [interview requirements](modules/interview/spec.md).

The interview flow does not release phone numbers or email addresses. The platform provides the meeting details required by the participants; the external meeting service has its own account and display settings.

The benefits remain questions to test: does an interview help a parent make a decision, does it reduce unsuitable introductions, and how much coordination does it require? The original 15–30 minute staff-time estimate per interview was a planning assumption, not an observed average.

---

## 3. Business Model

### 3.1 Two-Sided Marketplace Structure

- Parents and learners create tuition requests; tutors maintain profiles and may publish optional listings.
- Public posts require moderation approval. Visitors can browse and filter bounded public results without signing in; details and actions have separate permission checks.
- Tutors with the required profile fields may apply to active requests without an approved listing. Parents may review applicants for their own request and express a preference.
- Only an authorized coordinator creates the match. A request has at most one pending or active match, while a tutor may teach several students.
- A matched request leaves open discovery. A permitted cancellation can reopen it and restore eligible applications.
- Private contact is absent from public, applicant and interview views. The participant interface reveals it only for authorized ACTIVE or COMPLETED matches, never match-PENDING. Staff access is separately authorized and audited.

### 3.2 Admin as Matchmaker

The coordinator reviews the request and candidate tutor profile, checks suitability and agreement with both parties, and confirms the assignment. A parent can choose a preferred tutor, but that choice is a request for staff action. It is not an automatic match or permission to reveal contact.

This updates the May 2026 model in which staff initiated every match from two published posts. The later assignment design permits user-led interest and a tutor profile without a listing; staff confirmation remains required.

### 3.3 Commission Structure

| Variable | Value |
|---|---|
| Invoicing prerequisite | Tutee confirms the first session, or the authorized override process follows the seven-day timeout |
| Amount lock | Activation records the agreed first-month rate, session count and hours per session |
| Payment recorded | Only after the agency receives the tutor commission |
| Default commission rate | 50% of first month's tuition fee |
| Commission basis | First-month fee is hourly rate × hours per session × agreed sessions, rounded to whole SGD; commission is that locked fee × 50%, rounded to whole SGD |
| Payer | Tutor, paying the agency; parent-to-tutor lesson payments are separate |
| Collection method | Off-platform invoice/transfer (MVP scope) |
| One-time charge | One commission per match; no recurring fees |
| Commission disclosure stage | Standalone acknowledgement step during tutor registration (before profile creation completes) |

**Illustrative agreed-plan example:** The tutor and parent explicitly agree to 12 first-month sessions of 1.5 hours at SGD 50 per hour. The agreed first-month fee is SGD 900 and a 50% agency commission is SGD 450, owed by the tutor. This is a fixed agreed plan, not an inference from a weekly frequency.

The permitted activation override is from S$1 to the locked first-month fee. Later lesson-plan changes do not automatically recalculate commission. The weekly-to-monthly suggestion (four or 4.33 weeks), limits on negotiated rates, refunds and partial receipts remain open decisions. See [INT-157](architecture/int-157-commission-first-month-locked-spec.md) and [the decision register](CURRENT-REQUIREMENTS.md#open-decisions).

### 3.4 Revenue Drivers

- Volume of successful matches per month
- Average hourly rate of matched assignments (level and subject mix)
- Sessions per week agreed in each arrangement
- Commission collection rate (% of matched arrangements that actually pay)

---

## 4. Business Goals

These are historical May 2026 planning targets, not achieved results or validated forecasts. Their clock has not started with a public launch. A future plan should reset the dates and test the assumptions behind them. A KPI is a measure used to check progress.

### Priority 1 — Establish Functional Marketplace

| Goal | KPI | 12-Month Target |
|---|---|---|
| Grow active tutor post inventory | Approved tutor listings live at any time | 500 active listings |
| Grow active tutee post inventory | Approved tutee requests live at any time | 300 active requests |
| Achieve admin throughput | Posts reviewed and actioned within service-level agreement | 95% of posts reviewed within 2 business days |

### Priority 2 — Generate Revenue Through Matches

| Goal | KPI | 12-Month Target |
|---|---|---|
| Generate confirmed matches | Admin-confirmed matches per month | 50 matches/month by Month 12 |
| Achieve match-to-commission conversion | % of confirmed matches where commission is collected | ≥ 80% |
| Commission revenue | Monthly commission revenue (SGD) | SGD 20,000/month by Month 12 |

### Priority 3 — Build Platform Reputation

| Goal | KPI | 12-Month Target |
|---|---|---|
| Time-to-match service-level agreement | Median days from tutee post approval to admin match | ≤ 7 days |
| User satisfaction (tutee) | Post-match survey NPS | ≥ 50 |
| User satisfaction (tutor) | Post-match survey NPS | ≥ 40 |
| Post quality | % of submitted posts requiring revision before approval | ≤ 20% |

### Priority 4 — Operational Efficiency

| Goal | KPI | 12-Month Target |
|---|---|---|
| Admin workload per match | Admin hours spent per confirmed match | ≤ 2 hours |
| Re-listing rate | % of tutee requests re-posted after no match in 30 days | ≤ 15% |

### Match Cardinality Business Rules

The following cardinality rules govern posts and matches:

1. **Tutee post — maximum one active/pending match at a time.** A tutee post may be in at most one match with status PENDING or ACTIVE simultaneously. This is enforced via a partial unique index on tutee post id where match status IN PENDING, ACTIVE. If that match is cancelled, the tutee post returns to Active status and becomes eligible for re-matching.

2. **Tutor — multiple simultaneous matches allowed.** One tutor profile may be linked to several arrangements. A listing may be attached but is not required.

3. **Matched tutee post removed from discovery index.** When a tutee post reaches Matched state, it is removed from the tutor-browseable discovery index. It reappears in the index only when the match is cancelled and the post returns to Active state.

4. **Commission status.** Commission billing is tracked separately from the match lifecycle. These retained billing stages are requirements; partial receipts, refunds and some dispute transitions remain unresolved in the current register:

| Commission Status | Definition |
|---|---|
| PENDING | Match confirmed by admin; awaiting first session to occur |
| AWAITING CONFIRMATION | Admin has marked match ACTIVE; tutee confirmation of first session is pending |
| CONFIRMED | Tutee has confirmed that the first session occurred |
| INVOICED | Commission invoice has been generated and sent to tutor |
| PAID | The agency received the tutor commission |
| DISPUTED | Commission is under active dispute |
| WAIVED | Commission waived by admin decision |

---

## 5. Stakeholders

### 5.1 Tutees — Parents and Adult Students

| Sub-segment | Profile | Primary Driver |
|---|---|---|
| Primary school parents | Children in P1–P6 preparing for PSLE | PSLE scoring anxiety; catch-up or enrichment |
| Secondary school parents | Children in Sec 1–5 (Express/NA/NT) | O-Level preparation; subject-specific weakness |
| JC/IP/IB parents | Children in JC1–JC2 or IP stream | A-Level/IB scoring; last-mile preparation |
| Polytechnic/University students | Adult learners seeking peer tutoring | Exam support, specific modules |
| Adult learners | Working adults pursuing part-time education or skills | Flexible scheduling; professional development |

### 5.2 Tutors

| Sub-segment | Profile | Motivation |
|---|---|---|
| Undergraduate part-timers | NUS/NTU/SMU/SUTD students | Supplementary income; flexible hours |
| Full-time freelance tutors | Degree holders building tuition practice | Primary income; steady client pipeline |
| Ex-MOE teachers | Retired or left teaching service | Leveraging MOE experience; income in retirement |
| Specialist tutors | IB-trained, overseas-educated, or subject experts | Premium positioning; specific niche |

### 5.3 Admin (Internal)

The proposed coordinator role is responsible for the following work. This is a staffing plan, not a claim that employees or contractors have been hired:
- Post moderation (approve/reject/request revision)
- Matching decisions
- Communicating with tutees and tutors via contact info stored in admin panel
- Commission tracking and invoicing
- Platform content management

**Historical staffing assumptions (May 2026; unmeasured):**
- One full-time equivalent coordinator at launch and two by Month 12 were budget assumptions.
- A capacity of 25 matches and up to 20 interview coordinations per person each month was an estimate to test.
- Admin accounts MUST use multi-factor authentication (time-based one-time passcode minimum) — see Section 10

---

## 6. User Needs

### 6.1 Tutee Needs

| Need | Description |
|---|---|
| Easy post creation | Simple form to describe tuition needs: level, subject, preferred schedule, location, budget |
| Transparent process | Clear status updates: "Post Pending Review", "Post Approved", "Match Found" |
| Privacy protection | Contact is not public or shown before assignment; only authorized participants can reveal it in active or completed matches |
| Fast matching | Admin contacts them promptly when a suitable tutor is identified |
| Clear review standards | Understand what staff reviewed and what has not been independently verified |
| Post management | Ability to edit, deactivate, or delete their posts |
| First-session confirmation | After admin marks a match as Active, tutee receives a notification with a "Confirm first session occurred" action (see BR-010) |

### 6.2 Tutor Needs

| Need | Description |
|---|---|
| Easy profile/post creation | Simple form to describe offerings: levels taught, subjects, availability, rate expectations, qualifications |
| Assignment discovery | Browse and filter bounded approved, anonymized tutee request lists without login; authenticated detail/action permissions remain separate |
| Privacy protection | Contact info not publicly visible; admin controls introduction |
| Match notification | Notified promptly when admin identifies a match |
| Commission clarity | Upfront, transparent commission terms acknowledged as a standalone step during registration (before profile creation completes) |
| Post management | Ability to update availability, rates, and subjects offered |

### 6.3 Admin Needs

| Need | Description |
|---|---|
| Post review queue | Centralized dashboard showing pending posts with full details including contact info |
| Full contact info access | View phone numbers and email addresses for all registered users (access is logged — see Section 10) |
| Match creation tool | Link a tuition request to a tutor profile and record an assignment; a tutor listing is optional |
| Match status tracking | Track pending and active arrangements, cancellations and agency commission receipt; completion does not imply teaching ended |
| Commission ledger | Record and track commission amounts due, invoiced, and received per match |
| User management | Ability to suspend, deactivate, or ban accounts |
| Communication log | Internal notes on each post or match for team handoffs |
| First-session override | Ability to override the tutee first-session confirmation after 7-day timeout, allowing commission invoice generation to proceed |
| multi-factor authentication enforcement | All admin accounts must use time-based one-time passcode-based multi-factor authentication |
| Interview coordination | Receive and manage interview requests from tutees; contact tutor to agree on a time slot; enter meeting link (any URL) into the platform; notify both parties |
| Interview queue | Centralized view of all pending interview requests, sorted by submission time; ability to mark scheduled/completed/cancelled |

### 6.4 Post Status Notification Table

For each post status transition, the following notifications are triggered:

| Post Status | Triggered By | Notification Sent To | Notification Content | Dashboard Display |
|---|---|---|---|---|
| Pending Review | User submits post | User (tutee or tutor) | "Your post has been received and is under review. We will notify you within 2 business days." | "Post Under Review" badge |
| Approved / Active | Admin approves post | User | "Your post has been approved and is now active on the platform." | "Active" badge; post visible in discovery |
| Revision Requested | Admin requests revision | User | "Admin has requested changes to your post. [Reason]. Please update and resubmit." | "Revision Required" badge with reason |
| Rejected | Admin rejects post | User | "Your post could not be approved at this time. [Reason]." | "Rejected" badge with reason |
| Matched (tutee post) | Admin creates match | Tutee | "Great news — admin has found a tutor match for your request. Admin will be in touch shortly." | "Matched" badge; post removed from tutor discovery index |
| Matched (tutor post) | Admin creates match | Tutor | "Admin has identified a tutee assignment that matches your profile. Admin will be in touch shortly." | "Matched" assignment visible on tutor dashboard |
| Active (match active) | Admin marks match Active | Tutee | "Your tuition arrangement is now active. Please confirm when your first session has occurred." [Confirm action button] | "First Session Confirmation Pending" |
| Active (match active) | Admin marks match Active | Tutor | "Your tuition arrangement has been confirmed as active. Commission will be invoiced after the first session is confirmed." | "Arrangement Active" |
| Cancelled (match) | Admin cancels match | Both tutee and tutor | "The match arrangement has been cancelled. [Reason if provided]." | Tutee post returns to "Active"; tutor post shows "Available" |
| Closed | User or admin closes post | User | "Your post has been closed and is no longer visible on the platform." | "Closed" badge |
| Expired | System auto-expiry | User | "Your post has expired after [X] days without a match. You may resubmit an updated post." | "Expired" badge |

### 6.5 Interview Request User Flow

1. A signed-in parent requests an interview linked to their tuition request. The current requirements permit initiation while that request awaits approval.
2. The participants may propose and counter-propose times under the negotiated interview flow. Choosing a time does not create a tuition assignment.
3. Once a slot is accepted, the coordinator schedules the interview and provides a meeting link.
4. Both participants receive the relevant time and join details. Rescheduling invalidates obsolete reminders; a changed meeting link must reach both participants.
5. Completion, cancellation and no-show are recorded. A participant may then ask staff to progress an assignment.

Phone and email are not released by this process. Required interview details and permitted request/profile information may be shared; the meeting link is not the only information needed to attend. The external meeting service is outside the platform's contact-disclosure controls.

The original May flow was staff-only scheduling. The current [negotiation design](architecture/int-039-interview-negotiation.md) and [request-flow amendment](021-tutee-request-flow/spec.md) supersede that limitation.

### 6.6 First-Session Confirmation User Flow

This flow applies after admin marks a match as ACTIVE:

1. **Admin action:** Admin marks match status as ACTIVE in the admin panel.
2. **Tutee notification:** System sends a notification to the tutee with a "Confirm First Session" action button. The commission status is set to AWAITING CONFIRMATION.
3. **Tutee confirmation:** Tutee clicks "Confirm First Session Occurred" within 7 days. Commission status advances to CONFIRMED. System is ready to generate the commission invoice.
4. **No response within 7 days:** If the tutee does not confirm within 7 calendar days, the system flags the match for admin review. Admin may:
   - Override and advance commission status to CONFIRMED manually, OR
   - Contact tutee to obtain confirmation, OR
   - Change match status (e.g., cancel if arrangement did not proceed)
5. **Commission invoice generation:** Invoice is generated only after commission status reaches CONFIRMED (via tutee action or admin override).

---

## 7. Competitive Landscape

### 7.1 Historical Research Limit

The original May 2026 draft named other providers and asserted differences without dated supporting evidence. Those comparisons are removed. This document does not establish that InTuition is unique, superior, more private, or more effective than any other service.

### 7.2 Questions for Future Research

| Question | Evidence needed |
|---|---|
| Do parents want a coordinator's help? | Interviews and observation of prospective users, including how they currently find tutors |
| Does a short interview help a decision? | Reasons for proceeding or declining, alongside the time spent coordinating |
| Will tutors accept the disclosed fee? | Price objections, acceptance and actual agency receipts in a properly prepared trial |
| Can staff provide the service economically? | Measured staff time, direct costs, collections, cancellations and disputes per arrangement |

### 7.3 Intended Positioning

The proposed offer is a clear sequence from a tuition request to an agreed arrangement, with staff review, optional interviews, private contact before assignment and transparent commission terms. These are design commitments. Whether prospective users value them is unproven.

### 7.4 Historical Staffing and Financial Assumptions

The May draft assumed one full-time equivalent coordinator at launch, two by Month 12, capacity of about 25 matches per person per month and an example average commission of S$400. It also assumed S$4,000 in monthly staff costs.

Twenty collected commissions at S$400 would produce S$8,000 in revenue. The old draft called this a break-even point, but did not provide a complete cost model. It is an arithmetic scenario, not a demonstrated break-even threshold. Actual planning must include staff time, acquisition, support, messaging, infrastructure, payment costs and uncollected invoices.

---

## 8. Key Business Rules

### BR-001: Contact Information Visibility

- Contact information (phone/email) is **never public or exposed in pre-assignment counterparty views**.
- Admin access is separately authorized and audited. Under the explicit M18 amendment, assigned participants may access contact through the participant-only endpoint; INT-125 keeps frontend reveal to ACTIVE/COMPLETED matches. Post approval or choosing alone does not release contact.
- This rule applies to both tutees and tutors, and to all post types
- Violation of this rule constitutes a platform privacy breach
- All admin access to contact fields is logged in an immutable audit trail (see Section 10)

### BR-002: Post Approval Gate

- A tutee post or tutor post submitted by a registered user does **NOT become publicly visible** until an admin explicitly approves it
- Posts in "Pending Review" are not publicly discoverable. Owners and authorised staff may view pending posts. An invited participant may see only the information permitted for that invitation or interview, without private contact details. Starting either process does not approve public publication or assignment.
- Admin may approve, reject, or request revision on any post
- If rejected or revision-requested, the user is notified with a reason
- A user may resubmit a revised post; the resubmission re-enters the approval queue

### BR-003: Admin-Confirmed Assignment

- Tutors may apply to active requests after completing the required profile fields; a published tutor listing is optional.
- A parent may review applicants for their own request, request an interview and choose a preferred tutor.
- Only an authorized coordinator creates the match after checking agreement. A parent's selection alone is not a match.
- The match links one tuition request to one tutor identity, with an optional listing reference.
- At most one pending or active match may exist for a request. Contact access follows BR-001; interest or selection alone does not release it.

The original staff-only expression-of-interest and mandatory-two-listing rules were superseded by PRD section 20 and current decisions C04–C05.

### BR-004: Commission Calculation

- The tutor owes a one-time commission to the agency. Parent-to-tutor lesson payments are separate.
- At activation, staff confirm the hourly rate, first-month session count and hours per session. Multiplying these and rounding to whole Singapore dollars gives the locked fee.
- The default commission is 50% of that fee, also rounded to whole Singapore dollars. An authorized activation override can set an amount from S$1 to the fee.
- A flexible schedule has no automatic session count; staff must enter a positive agreed count. The four-versus-4.33-week suggestion and limits on negotiated rates remain open.
- Later plan changes do not automatically rewrite the snapshot. First-session confirmation still gates invoicing and due handling.
- PAID requires actual receipt by the agency. An unpaid dispute decision or lesson payment is not receipt. Refunds, partial payments and early termination require the remaining explicit financial rules.
- Manual payment recording is not integrated payment collection. No recurring platform, subscription or per-session fee is defined by this model.

### BR-005: One Commission Per Match

- Each admin-confirmed match generates at most one commission event
- If the arrangement ends and the same tutee and tutor are re-matched later, a new commission may apply at admin discretion
- If a match is created but no arrangement commences (either party declines), no commission is owed

### BR-006: Communication and Controlled Contact Access

- The product does not provide a general private chat between parents and tutors.
- Applications, invitations, selections and interview-time proposals are permitted structured interactions.
- Staff coordinate the formal assignment. Contact remains absent from public and pre-assignment views.
- Authorized assigned participants can reveal phone/email through the active/completed match interface. Pending matches do not reveal it.
- External messaging is not evidence that platform delivery succeeded. WhatsApp remains off until its separately recorded enablement conditions are met.

### BR-007: Registration and Browsing

- Registration and the applicable account checks are required to create posts and use protected actions.
- Guests may browse and filter bounded public tutor and tuition-request results. Anonymous pages are numbered zero to four, with at most 20 results per page; later pages are empty. This limits each query's result set, not the number of searches a visitor can make.
- Detailed views and actions retain separate permission checks. Browsing does not authorize access to contact.
- Applying requires the tutor's required profile fields, not a published tutor listing.
- A user account does not simultaneously hold the tutee and tutor roles under the retained account model.

### BR-008: Post Lifecycle

- Posts have the following status states: Draft → Pending Review → Approved / Revision Requested / Rejected
- Approved posts may be: Active (visible), Matched (admin has created a match), Closed (user or admin deactivated), Expired (auto-expired after configurable period with no match activity)
- When a tutee post reaches Matched state, it is removed from the tutor-browseable discovery index
- See Appendix A for the complete post state transition table

### BR-009: Commission Disclosure at Registration

- During tutor registration, the platform MUST present a **standalone commission disclosure screen** as a mandatory step before profile creation can be completed
- The tutor must explicitly acknowledge this screen (checkbox or equivalent affirmative action) before proceeding
- The disclosure MUST include:
  - The commission rate (default: 50% of first month's tuition fee)
  - The calculation method: multiply the hourly rate, hours per session and agreed first-month session count, then round to whole Singapore dollars. Multiply that rounded monthly fee by the commission rate and round again to whole dollars.
  - A worked numeric example using realistic Singapore rates (e.g., SGD 50/hr × 1.5hr × 8 sessions = SGD 600/month → commission = SGD 300)
  - The commission trigger event (first session confirmed by tutee or admin override)
  - A statement that the commission is owed by the tutor and is a one-time charge per match
- The acknowledgement event MUST be logged as a system event with: tutor user ID, timestamp, version of commission terms shown, and the affirmative action recorded
- A tutor who has not completed this acknowledgement step cannot have their profile approved by admin

### BR-010: First-Session Confirmation Before Commission Invoice

- Commission invoice MUST NOT be generated until the first session has been confirmed
- **Trigger:** When admin marks a match as ACTIVE, the system automatically sends the tutee a "Confirm First Session" notification
- **Commission status at this point:** AWAITING CONFIRMATION
- **Tutee action:** Tutee clicks "Confirm First Session Occurred" in their dashboard. Commission status advances to CONFIRMED
- **Timeout:** If tutee does not confirm within **7 calendar days after activation**, the system flags the match for admin attention
- **Admin override:** After the seven-day timeout, authorized staff may follow the recorded override process, seek confirmation or cancel. Silence does not prove a session occurred.
- **Invoice generation:** The system generates a commission invoice only after commission status is CONFIRMED
- **Cancellation case:** If the match is cancelled before first session confirmation, commission status is set to WAIVED and no invoice is generated
- This business rule is in MVP scope

### BR-011: Interview Request Eligibility

- Only authenticated users with the appropriate role and ownership may initiate the interview flow.
- A parent may initiate from their own pending request under specification 021. Publication and assignment still require moderation approval.
- Interview caps apply per posting, using the current negotiation states, not the old per-account REQUESTED/SCHEDULED-only count.
- Completed/no-show cooldowns, cancellation, decline and expiry follow the current interview design. A rejected, removed or closed request must clean up its open intake.
- Whether an invitation may be accepted while its request is still pending is open decision O01; permission to initiate does not settle permission to accept.

The original listing-only, per-account and no-request-needed rules are historical. See [the current interview specification](modules/interview/spec.md).

### BR-012: Interview Duration Limit

- Each interview slot is a maximum of **15 minutes**
- Admin MUST record the scheduled duration when creating the meeting. If duration is not specified, the system defaults to 15 minutes
- The platform does not enforce the duration technically (no meeting timer or automatic disconnect). The 15-minute limit is communicated to both parties in the notification
- Admin may not schedule an interview slot longer than 15 minutes through the platform user interface

### BR-013: Interview Negotiation and Scheduling

- Participants may propose times and counter-propose under the current negotiation flow.
- The coordinator completes the accepted-slot scheduling step and supplies the meeting link.
- Replaced proposals cannot remain actionable. Cancellation and no-show outcomes must handle the permitted states.
- Staff must be able to tell whether the latest meeting details and reminders were delivered, pending or failed.

The original staff-exclusive scheduling rule is superseded by [INT-039](architecture/int-039-interview-negotiation.md).

### BR-014: Meeting Link Requirement

- Admin MUST provide a meeting link when scheduling an interview. The link may be any valid URL (Zoom, Google Meet, Microsoft Teams, Jitsi, or any other video conferencing service)
- The platform does not create or manage video conferencing accounts or rooms — admin provides an externally generated link
- The meeting link MUST be stored in the interview record and delivered to both parties via platform notification
- Admin may update the meeting link before the scheduled time; the changed link must be delivered to both participants.
- Meeting links are not shown publicly; they are sent only to the specific tutee and tutor involved in the interview

### BR-015: Contact Privacy During Interview

- The interview feature does NOT involve any exchange of contact information between tutee and tutor
- The tutee does NOT receive the tutor's phone number, email address, or personal details through the interview workflow
- The tutor does NOT receive the tutee's phone number, email address, or personal details through the interview workflow
- Share only the permitted profile/request summary, scheduling information and meeting link needed for the interview; do not include phone, email or exact home address.
- Admin contacts the tutor by external means (using admin-held contact info) to coordinate scheduling — this contact is not mediated through the platform
- Any attempt to circumvent this rule (e.g., embedding contact info in the meeting link notes field) is prohibited and subject to account suspension

---

## 9. Success Metrics

These are historical May 2026 targets, not observed results. They need a defined measurement period and a fresh plan before launch. Collected revenue must count agency receipts, not invoices or parent lesson payments.

### 9.1 Supply & Demand Health

| Metric | Definition | Target (Month 6) | Target (Month 12) |
|---|---|---|---|
| Active tutor listings | Approved, active tutor posts at any point in time | 250 | 500 |
| Active tutee requests | Approved, active tutee posts at any point in time | 150 | 300 |
| Supply-demand ratio | Tutor listings ÷ Tutee requests | 1.5–2.0x | 1.5–2.0x |
| New post submissions/week | Combined new tutee + tutor posts per week | 30 | 60 |

### 9.2 Operational Performance

| Metric | Definition | Target |
|---|---|---|
| Post review service-level agreement | % of submitted posts reviewed within 2 business days | ≥ 95% |
| Post approval rate | % of submitted posts approved (not rejected) | ≥ 85% |
| Revision request rate | % of posts requiring revision before approval | ≤ 15% |
| Time-to-match | Median days from tutee post approval to confirmed match | ≤ 7 days |
| Admin matches per month | Number of admin-confirmed matches per month | 50 by Month 12 |

### 9.3 Revenue

| Metric | Definition | Target (Month 6) | Target (Month 12) |
|---|---|---|---|
| Commission revenue (SGD/month) | Total commission collected per month | SGD 8,000 | SGD 20,000 |
| Commission collection rate | % of confirmed matches where commission is collected | ≥ 80% | ≥ 85% |
| Average commission per match | Mean commission value per collected match (SGD) | SGD 350 | SGD 400 |
| Match-to-arrangement rate | % of confirmed matches where arrangement actually starts | ≥ 75% | ≥ 80% |

### 9.4 User Quality

| Metric | Definition | Target |
|---|---|---|
| Tutee post-match NPS | Net Promoter Score from tutees after a match | ≥ 50 |
| Tutor post-match NPS | Net Promoter Score from tutors after a match | ≥ 40 |
| Tutee repeat post rate | % of tutees who post again within 12 months (new subject/child) | ≥ 25% |
| Tutor re-listing rate | % of tutors who update/maintain active listings after first match | ≥ 60% |

### 9.5 Interview Feature Performance

| Metric | Definition | Target (Month 6) | Target (Month 12) |
|---|---|---|---|
| Interview request rate | % of approved tutee-facing tutor posts that receive ≥1 interview request | ≥ 20% | ≥ 35% |
| Interview scheduling rate | % of interview requests that reach SCHEDULED status | ≥ 85% | ≥ 90% |
| Interview completion rate | % of scheduled interviews that reach COMPLETED (vs cancelled/no-show) | ≥ 80% | ≥ 85% |
| Interview-to-match conversion | % of completed interviews that result in a formal match within 14 days | ≥ 40% | ≥ 55% |
| Admin interview turnaround | Median hours from interview REQUESTED to SCHEDULED | ≤ 24 hrs | ≤ 12 hrs |

**Proposed admin scheduling requirement:** Admin must schedule any REQUESTED interview within **1 business day**. This is a target to validate against staff capacity before offering it to users. A P95 ≤ 24 hrs alarm on admin scheduling turnaround hours must be configured in the operations dashboard from Day 1.

---

## 10. Risks, Constraints & Non-Functional Requirements

### 10.1 Privacy Risk

| Risk | Description | Mitigation |
|---|---|---|
| Contact data breach | Admin panel exposes personal information; a breach would expose all user contact info | Enforce role-based access control; limit admin accounts; protect storage/backups under the plaintext-contact amendment; audit contact-field access |
| Accidental public exposure | A bug or misconfiguration publishes contact info on public pages | Code reviews and automated tests asserting contact fields are never serialised to public application programming interface responses |
| Personal Data Protection Act compliance | Singapore Personal Data Protection Act requires consent, purpose limitation, and breach notification | Include Personal Data Protection Act-compliant consent language at registration; appoint a Data Protection Officer; document data flows |

### 10.2 Supply-Demand Imbalance

| Risk | Description | Mitigation |
|---|---|---|
| Tutor-heavy inventory | More tutors than tutee requests; tutors feel matches are slow | Invest in tutee acquisition marketing (Facebook parent groups, school-adjacent channels) |
| Tutee-heavy inventory | More requests than tutor supply; tutee wait time increases | Run tutor recruitment campaigns; targeted outreach to university student populations |
| Subject/level gaps | High demand for niche subjects (e.g., IB Economics, Tamil O-Level) with insufficient tutor supply | Admin tracks unmatched requests by subject/level; targeted supply-side outreach |

### 10.3 Commission Collection Risk

| Risk | Description | Mitigation |
|---|---|---|
| Non-payment | Tutor accepts match but does not pay commission | Obtain signed commission agreement before introducing tutor to tutee; pursue via standard collection channels |
| Dispute over arrangement start | Tutor claims arrangement never started | BR-010: Require tutee confirmation of first session as commission trigger |
| Off-platform circumvention | Tutor and tutee agree to transact outside platform, bypassing commission | Commission agreement to include clause prohibiting circumvention for 12 months post-introduction |

### 10.4 Operational Constraints

| Constraint | Description |
|---|---|
| Admin bandwidth | All posts and matches are human-moderated; volume is constrained by admin headcount (1 FTE at launch, 2 FTEs by Month 12; ~25 matches/FTE/month) |
| Manual commission collection | MVP has no in-platform payment; commission tracking and collection is manual |
| Match suitability | Staff judgement, user preferences and accurate profile/request data all matter; none guarantees a successful teaching relationship |
| Regulatory | Confirm applicable tutor work-authorization requirements and the platform’s responsibilities before launch; do not infer an exemption from these planning notes |

### 10.5 Market Risks

| Risk | Description |
|---|---|
| Acquiring users | The cost of attracting suitable parents and tutors is unknown and must be measured |
| MOE policy changes | Changes to PSLE, streaming, or curriculum reduce demand for certain subjects/levels |
| Freelance tutor churn | Tutors stop listing once their schedule is full; active inventory may shrink without continuous acquisition |

### 10.6 Security Requirements

#### 10.6.1 Admin Multi-Factor Authentication

- All admin accounts MUST use multi-factor authentication with time-based one-time passcode (Time-based One-Time Password) as the minimum standard
- Hardware security keys (FIDO2/WebAuthn) are preferred for production admin accounts
- multi-factor authentication cannot be disabled for admin roles by any application-level configuration
- New admin accounts cannot access the admin panel until multi-factor authentication is configured and verified

#### 10.6.2 Immutable Audit Log for Contact Field Access

- Every access to a contact field (phone number, email address, home address) by an admin user MUST be recorded in an immutable audit log
- Each audit log entry MUST capture: timestamp (Coordinated Universal Time), admin user ID, record type accessed (tutee or tutor), record ID, fields accessed
- The audit log MUST be write-once and non-deletable by application code
- The audit log MUST be stored separately from the primary application database with restricted write access
- The original draft proposed five-year audit retention. Final retained fields and periods require the open O08 decision; see Appendix C.

### 10.7 Non-Functional Requirements

#### 10.7.1 Personal Information Storage

The founder changed the earlier encryption proposal. Email and phone are now stored as plain text in the account record, using the approved email lookup. Restrict database access, protect backups, record access and follow the current contact-sharing rules. Plaintext personal information values MUST NOT appear in application logs, error traces, or analytics data exports.

#### 10.7.2 Data Residency

All data processed and stored by this system must reside on infrastructure physically located in Singapore. Cross-border transfer of any data category (including backups, analytics exports, and third-party integrations) requires a written determination from the designated Data Protection Officer prior to implementation. This requirement applies to all environments including staging and development environments that handle real user data.

#### 10.7.3 Mobile-First User Interface

The tutee and tutor user interfaces MUST be designed and tested for mobile-first usage. The primary target viewport is 390px width and above. All core user flows (post creation, status viewing, first-session confirmation) must be fully functional and usable at 390px without horizontal scrolling. Automated visual regression tests MUST include mobile viewport breakpoints. Admin interfaces must also satisfy the later 015/019 requirements at 390px, including reachable navigation, focus management and no page-level horizontal overflow.

#### 10.7.4 Backup, Disaster Recovery, and Availability

| Parameter | Requirement |
|---|---|
| Recovery Point Objective (RPO) | Maximum 24 hours (maximum data loss in a disaster scenario) |
| Recovery Time Objective (RTO) | 4 hours (maximum time to restore service after a disaster) |
| Backup frequency | Daily automated backups |
| Backup retention | 30 days rolling |
| Public site availability service-level agreement | 99.5% uptime (measured monthly) |
| Admin panel availability service-level agreement | 99.9% uptime during Singapore business hours (Singapore time 08:00–22:00, 7 days) |
| Restore procedure testing | Quarterly — a full restore must be tested and the result documented |
| Backup storage location | Singapore region; subject to data residency requirement (10.7.2) |

---

## 11. Out of Scope for MVP

This table records the original May 2026 scope choices. Later amendments supersede it where noted; it is not a current inventory of missing features.

| Feature | Reason Excluded | Future Release Notes |
|---|---|---|
| Automated / algorithmic matching | Core differentiator is human curation; premature automation undermines brand | Phase 2 — assist admin with suggestions, not replace |
| In-platform payment processing | Commission collection via invoice/transfer is sufficient for MVP scale; payment integration adds compliance complexity (MAS regulations) | Phase 2 — integrate payment gateway (e.g., Stripe, PayNow) |
| Mobile app (iOS/Android) | Web-responsive is sufficient for MVP; native app development is costly | Phase 3 |
| Chinese / Malay / Tamil language user interface | English is the working language in Singapore tuition market; multilingual user interface adds localisation overhead | Phase 2 — simplified Chinese for potential mainland Chinese tutor segment |
| In-platform messaging / chat | Business rule BR-006 excludes this; introducing messaging changes the model fundamentally | Out of scope indefinitely for tutee-tutor direct communication |
| Tutor reviews and ratings | Originally deferred | Later development scope records this as built; the old exclusion no longer controls |
| Subscription or membership tiers | Single commission model is simpler to launch; tiered pricing adds sales complexity | Phase 3 |
| Zoom/Google Meet application programming interface integration (automated room creation) | Admin provides meeting links manually; automated room provisioning is not required for MVP | Phase 2 — auto-generate meeting links via Zoom/Google Meet application programming interface |
| Background or credential verification integration | No universal verification guarantee is defined | The evidence required for a verified-tutor claim remains open decision O05 |
| Analytics dashboard for tutors/tutees | Internal admin analytics only in MVP | Phase 2 |

---

## 12. Singapore-Specific Requirements

### 12.1 Education Levels Supported

The following is the May 2026 product-planning vocabulary, not a verified current curriculum list. The exact approved mapping remains decision O06; later subject/level integrity requirements control which combinations the form may accept.

| Level Group | Specific Levels | Subjects Typically Requested |
|---|---|---|
| **Kindergarten** | K1, K2 | Phonics, Early Literacy, Early Numeracy |
| **Primary** | P1, P2, P3, P4, P5, P6 (PSLE) | English, Mathematics, Science, Chinese, Malay, Tamil, Higher Mother Tongue |
| **Secondary — Express** | Sec 1, 2, 3, 4 (O-Level) | E-Maths, A-Maths, English, Physics, Chemistry, Biology, Combined Science, History, Geography, Literature, Social Studies, Chinese, Malay, Tamil, POA |
| **Secondary — Normal Academic** | Sec 1, 2, 3, 4, 5 (N-Level → O-Level) | Same as Express; N-Level syllabus variants |
| **Secondary — Normal Technical** | Sec 1, 2, 3, 4 | Mathematics, English, MT, Technical subjects |
| **Integrated Programme (IP)** | IP Year 1–4 (school-specific) | School-specific curricula (e.g., NUSH, RI, HCI) |
| **Junior College** | JC1, JC2 (A-Level) | GP, H1/H2 Maths, H2 Physics, H2 Chemistry, H2 Biology, H2 Economics, H2 History, H2 Geography, H2 Literature, H1/H2 Mother Tongue |
| **IB (International Baccalaureate)** | IB Year 1–2 (school-specific) | IB HL/SL — Maths AA/AI, Physics, Chemistry, Biology, Economics, History, English A/B, Chinese |
| **Polytechnic** | Year 1–3 (Diploma modules) | Engineering Maths, Business Statistics, module-specific |
| **University** | Year 1–4 (Degree modules) | Statistics, Calculus, Programming, Economics, Accounting, specific modules |
| **Adult / Non-traditional** | IGCSE, private candidates, adult learners | English, Maths, Mother Tongue, professional certifications |

### 12.2 Subject Reference List

**PSLE Level**

| Subject | Exam Board | Notes |
|---|---|---|
| English Language | SEAB | Compulsory |
| Mathematics | SEAB | Compulsory |
| Science | SEAB | P3–P6 |
| Foundation English / Maths | SEAB | For weaker students |
| Chinese Language | SEAB | Mother Tongue track |
| Malay Language | SEAB | Mother Tongue track |
| Tamil Language | SEAB | Mother Tongue track |
| Higher Chinese / Malay / Tamil | SEAB | Higher MT track |

**O-Level Level**

| Subject | Exam Board | Notes |
|---|---|---|
| English Language | SEAB | Compulsory |
| Elementary Mathematics | SEAB | Compulsory |
| Additional Mathematics | SEAB | Optional; common Sec 3–4 |
| Physics | SEAB | Pure or Combined Science |
| Chemistry | SEAB | Pure or Combined Science |
| Biology | SEAB | Pure or Combined Science |
| Combined Science (various) | SEAB | Phys/Chem, Chem/Bio, Phys/Bio |
| History | SEAB | Elective Humanities |
| Geography | SEAB | Elective Humanities |
| Literature in English | SEAB | Elective Humanities |
| Social Studies | SEAB | Compulsory in Express/NA |
| Principles of Accounts (POA) | SEAB | |
| Chinese / Malay / Tamil Language | SEAB | Mother Tongue |
| Higher Chinese / MT | SEAB | Higher MT |

**A-Level Level**

| Subject | Exam Board | Notes |
|---|---|---|
| General Paper (GP) | SEAB | Compulsory |
| H1/H2 Mathematics | SEAB | |
| H2 Further Mathematics | SEAB | Additional for strong students |
| H2 Physics | SEAB | |
| H2 Chemistry | SEAB | |
| H2 Biology | SEAB | |
| H2 Economics | SEAB | |
| H2 History | SEAB | |
| H2 Geography | SEAB | |
| H2 Literature in English | SEAB | |
| H1/H2 Mother Tongue Language | SEAB | |
| Knowledge & Inquiry (KI) | SEAB | Replaces GP in some schools |

### 12.3 Historical Pricing Examples

| Education Level | Undergraduate Tutor | Full-Time Freelance | Ex-MOE Teacher |
|---|---|---|---|
| Kindergarten | SGD 20–25/hr | SGD 25–35/hr | SGD 35–50/hr |
| Primary (P1–P4) | SGD 22–30/hr | SGD 30–45/hr | SGD 45–60/hr |
| Primary (P5–P6 / PSLE) | SGD 25–35/hr | SGD 35–55/hr | SGD 50–70/hr |
| Secondary (Sec 1–2) | SGD 28–38/hr | SGD 38–55/hr | SGD 55–75/hr |
| Secondary (Sec 3–5 / O-Level) | SGD 30–45/hr | SGD 45–65/hr | SGD 65–90/hr |
| JC / A-Level | SGD 50–65/hr | SGD 65–90/hr | SGD 80–120/hr |
| IB | SGD 55–70/hr | SGD 70–100/hr | SGD 90–130/hr |
| Polytechnic / University | SGD 35–50/hr | SGD 50–80/hr | N/A (rare) |

*These figures are unverified May 2026 planning examples, not current market benchmarks or recommended prices. Actual agreed rates are a matter for the participants and the approved commercial rules.*

### 12.4 Session Format & Venue Types

| Venue Type | Description | Common Usage |
|---|---|---|
| Tutee's home | Tutor travels to tutee's residence | Most common for Primary/Secondary |
| Tutor's home | Tutee travels to tutor's residence | Common for JC/specialist tutors |
| Public location | Library, community centre, café | Secondary and above; neutral ground |
| Online (video call) | Zoom, Google Meet, or equivalent | Growing post-COVID; all levels |
| Tutor's centre | If tutor has a dedicated teaching space | Specialist or ex-MOE tutors |

The platform post form must allow tutees and tutors to specify venue preference and online/in-person availability. Location should be captured by:
- MRT station (nearest)
- Planning area / district (e.g., Tampines, Buona Vista, Bishan)
- Broad region: Central, North, South, East, West

### 12.5 Scheduling Conventions

The original draft used the following scheduling assumptions. They are not a current exam calendar or measured demand pattern; check official dates before using them for an operating plan:

| Convention | Details |
|---|---|
| School terms | 4 terms/year; Jan–Mar, Apr–Jun, Jul–Sep, Oct–Nov; tuition demand peaks before exams |
| PSLE exam window | Typically September–October; demand surge P5–P6 from January |
| O-Level exam window | October–November; demand surge Sec 4 from March |
| A-Level exam window | October–November; demand surge JC2 from March |
| Weekday preference | Typically after school hours: 4pm–8pm weekdays |
| Weekend slots | High demand; Saturday and Sunday mornings most popular |
| Holiday intensive | June and November/December school holidays; common for intensive crash programmes |

### 12.6 Platform Language

- Primary language: **English**
- All system text, labels, notifications, and post templates: English
- Subject and level nomenclature: follow **SEAB/MOE official terminology** (e.g., "O-Level" not "GCSE", "H2 Chemistry" not "A-Level Chemistry")
- Mother tongue subjects listed in English with native script in parentheses where helpful (e.g., "Chinese Language (华文)")

### 12.7 Regulatory Considerations

The original draft identified data protection, tax registration and invoicing, tutor work authorization, education-related requirements and fair consumer terms as topics to resolve before launch. It did not establish legal conclusions or prove compliance.

The business requirements are to maintain a clear privacy notice and consent record, define retention and access processes, disclose commission terms, and obtain appropriate advice on applicable obligations. Do not infer an exemption or a statutory deadline from this planning document.

---

## Appendix A — Post State Transition Table

This appendix defines all valid post states, the trigger for each transition, the actor responsible, the resulting state, and any side effects.

### A.1 Tutee Post State Transitions

| From State | Trigger | Actor | To State | Side Effects / Notes |
|---|---|---|---|---|
| (none) | User saves post as draft | Tutee | Draft | Post not visible to anyone except tutee |
| Draft | User submits post for review | Tutee | Pending Review | Admin notified of new submission; tutee notified "under review" |
| Pending Review | Admin approves post | Admin | Active | Post becomes visible in tutor-browseable discovery index; tutee notified |
| Pending Review | Admin requests revision | Admin | Revision Requested | Tutee notified with reason; post not visible publicly |
| Pending Review | Admin rejects post | Admin | Rejected | Tutee notified with reason; post permanently closed |
| Revision Requested | Tutee updates and resubmits | Tutee | Pending Review | Post re-enters admin review queue |
| Active | Admin creates a match linking this post | Admin | Matched | Post removed from tutor discovery index; tutee notified of match; match record created with commission status PENDING |
| Active | Auto-expiry timer elapses (configurable; default 60 days) | System | Expired | Tutee notified; post hidden from discovery; tutee may resubmit |
| Active | Tutee deactivates post | Tutee | Closed | Post hidden; tutee notified; no match in progress |
| Active | Admin closes post | Admin | Closed | Post hidden; tutee notified |
| Matched | Admin cancels match | Admin | Active | Eligible post returns to discovery; both parties are notified and the match becomes CANCELLED. Commission treatment depends on billing stage: an invoiced amount needs an explicit waiver or dispute decision, and existing receipts are preserved (see A.3). |
| Matched | Tutee withdraws post while match is pending | Tutee | Closed | Match is also cancelled; tutor notified; commission status set to WAIVED |
| Matched | Admin marks arrangement ended | Admin | Closed | Match record closed; commission lifecycle proceeds per current commission status |
| Expired | Tutee resubmits post (creates new post) | Tutee | Pending Review (new record) | Original expired post remains in Expired state; a new post record is created |
| Any non-terminal state | Admin bans or suspends account | Admin | Closed | All active posts for the account are closed |

**Terminal states:** Rejected, Closed, Expired (expired posts may lead to a new post but the expired record does not reactivate).

### A.2 Tutor Post State Transitions

| From State | Trigger | Actor | To State | Side Effects / Notes |
|---|---|---|---|---|
| (none) | User saves post as draft | Tutor | Draft | Post not visible to anyone except tutor |
| Draft | User submits post for review | Tutor | Pending Review | Admin notified of new submission; tutor notified "under review" |
| Pending Review | Admin approves post | Admin | Active | Tutor profile and post visible to admin for matching purposes; tutor notified |
| Pending Review | Admin requests revision | Admin | Revision Requested | Tutor notified with reason |
| Pending Review | Admin rejects post | Admin | Rejected | Tutor notified with reason |
| Revision Requested | Tutor updates and resubmits | Tutor | Pending Review | Post re-enters admin review queue |
| Active | Admin creates a match using the tutor profile and optional post | Admin | Active (post unchanged) | Tutor notified of match; match record created; tutor post remains in Active state and can be matched again (tutor may have multiple simultaneous matches) |
| Active | Auto-expiry timer elapses | System | Expired | Tutor notified; post hidden; tutor may resubmit |
| Active | Tutor deactivates post | Tutor | Closed | Post hidden |
| Active | Admin closes post | Admin | Closed | Post hidden; tutor notified |
| Any non-terminal state | Admin bans or suspends account | Admin | Closed | All active posts closed |

**Note on tutor post cardinality:** A tutor post in Active state may participate in multiple simultaneous match records. The tutor post's own status field remains Active even when one or more matches referencing it are in PENDING or ACTIVE state.

### A.3 Match State Transitions

| From State | Trigger | Actor | To State | Commission Status Effect | Notes |
|---|---|---|---|---|---|
| (none) | Admin creates match record | Admin | PENDING | Set to PENDING | Links one tutee post to tutor user identity; tutor listing optional under INT-034; parties notified |
| PENDING | Admin marks match as active (arrangement confirmed) | Admin | ACTIVE | Set to AWAITING CONFIRMATION | Triggers tutee first-session confirmation notification (BR-010) |
| ACTIVE | Tutee confirms first session occurred | Tutee | ACTIVE (no change) | Set to CONFIRMED | Commission invoice can now be generated |
| ACTIVE | Admin uses the authorized override process after the seven-day timeout | Admin | ACTIVE (no change) | Set to CONFIRMED | Commission invoice can now be generated |
| ACTIVE | Commission invoice generated | System/Admin | ACTIVE (no change) | Set to INVOICED | Invoice sent to tutor |
| ACTIVE | Agency receives tutor commission; admin records receipt | Admin | COMPLETED | PAID | Job remains MATCHED; completion does not mean all teaching ended |
| PENDING or ACTIVE | Admin cancels match | Admin | CANCELLED | Depends on billing stage; invoiced commission needs an explicit waiver or dispute decision | Eligible request returns to discovery; both parties notified; existing receipts are not silently erased |
| PENDING | Either party withdraws (tutee closes post or tutor closes post) | Tutee / Tutor | CANCELLED | Set to WAIVED | Tutee post returns to Active if tutee did not close it |
| ACTIVE | Commission disputed | Admin | ACTIVE (no change) | Set to DISPUTED | Admin investigates; commission status resolved manually |
| ACTIVE | Admin waives commission | Admin | ACTIVE (no change) | Set to WAIVED | No invoice generated or invoice voided |

**Definition of "Expired":** A tutee or tutor post reaches Expired state when the configurable auto-expiry timer elapses without the post being matched, closed, or renewed. The default expiry period is 60 days from the Active date. Expiry is triggered by a scheduled system job. An expired post does NOT automatically cancel any match that may exist (an Active or Pending match prevents expiry if the tutee post is in Matched state). Only posts in Active state (not Matched) are subject to auto-expiry.

---

## Appendix B — Personal Information Field Register

This is an access and handling inventory. It is not a legal classification opinion or proof that every proposed field is collected. Detailed retention periods and erasure rules remain open decision O08.

| Information | Intended access | Handling requirement |
|---|---|---|
| User email and phone | The owner, authorized staff and authorized assigned participants under the contact rules | Stored in plaintext under the explicit amendment; restrict database access, protect backups, audit required access and exclude values from logs and analytics |
| Legal name, home address or date of birth, if collected | Only the owner and specifically authorized staff | Do not add to public discovery or assume assignment releases exact home address |
| Public handle | Public views where permitted | Use the permitted public identity; it must not reveal otherwise private fields |
| General area and nearest station on a public post | Bounded public discovery | Avoid exposing exact private location; consider whether combinations identify a person |
| Profile photo, if collected | Access determined by the approved profile design | Do not infer a public-photo entitlement from this register |
| Commission acknowledgement | Authorized staff and the relevant user-facing confirmation | Preserve terms version and acknowledgement details required by the contract |
| Contact-access audit | Authorized audit/review roles | Record who accessed what and when; prevent application users from rewriting the history |

**Historical note, May 2026:** The old register proposed encryption and fixed one-, two-, five- or seven-year periods for different rows. The plaintext-contact amendment supersedes the email/phone encryption requirement. The current register leaves final retention, retained audit fields and erasure timing unresolved; the historical numbers are not an instruction to delete or retain data for those periods.

## Appendix C — Data Governance and Retention Policy

### C.1 Decisions Required Before Launch

The project requires an approved data-handling plan before real-user operation. The founder must identify responsibility for privacy decisions and obtain appropriate review of notice, consent, access, correction, retention, erasure and incident response. This is a project gate, not a claim that review has occurred or compliance is established.

The plan must specify each data class, why it is needed, who can access it, the retention period, how erasure or anonymization works, and how backups, financial records and audit obligations affect the process. See open decision O08 in the [current register](CURRENT-REQUIREMENTS.md#open-decisions).

### C.2 Deletion and Anonymization

- Business deletion is logical: a record is marked deleted or closed and excluded from normal use.
- Logical deletion does not itself resolve the separate retention and erasure policy. No general rule in this document authorizes physical deletion after an assumed period.
- Anonymization must remove the ability to link records back to a person; replacing a name with a stable identifier alone does not establish that result.
- Erasure handling must account for approved audit and financial-record requirements and backup rotation. These details require an explicit policy rather than an automatic rewrite of history.

### C.3 Analytics Handling

Analytics must exclude private contact details and session secrets. The original draft proposed anonymizing identifiable analytics after 90 days; its exact retention and separation arrangements require reconciliation with O07 and O08 before implementation approval. Aggregated statistics must not be treated as anonymous merely because names are absent.

### C.4 User Requests and Incidents

Provide a process for people to request access or correction, withdraw consent where applicable, and request deletion. Record the request, explain any approved retention requirement and confirm the resulting action. Maintain an incident assessment and notification procedure reviewed against applicable obligations. This document does not state a universal legal response deadline.

### C.5 Third-Party Services

Record which services receive user information, for what purpose, under what access and retention arrangements, and in which locations. The project's Singapore-residency requirement applies unless the authorized review approves an exception. Review providers and data flows when the service changes.

---

## Appendix D — Role Permission Matrix

This matrix defines what each system role may read, write, create, or delete for each resource type. "Own" means records associated with the user's own account. "Any" means any record in the system.

### D.1 Role Definitions

| Role | Description |
|---|---|
| UNAUTHENTICATED | Visitor with no account or not logged in |
| TUTEE | Registered tutee with a verified account |
| TUTOR | Registered tutor subject to account checks and the required profile fields; a published listing is optional |
| ADMIN | Internal staff or contractor with admin panel access |

### D.2 Permission Matrix

These are access requirements, not a certification of current enforcement. Invitation and interview negotiation permissions also depend on ownership and state; they are not blanket edit rights.

| Resource | Action | UNAUTHENTICATED | TUTEE | TUTOR | ADMIN |
|---|---|---|---|---|---|
| **Tutor posts (public listing)** | Browse/filter approved listings without contact | Yes (bounded) | Yes | Yes | Yes |
| **Tutor posts (content)** | Read (own) | No | No | Yes | Yes |
| **Tutor posts (content)** | Read (any — admin view) | No | No | No | Yes |
| **Tutor posts (content)** | Create | No | No | Yes | Yes (on behalf) |
| **Tutor posts (content)** | Update (own) | No | No | Yes | Yes |
| **Tutor posts (content)** | Delete / Close (own) | No | No | Yes | Yes |
| **Tutee posts (content, anonymised — discovery index)** | Browse/filter ACTIVE posts without contact | Yes (bounded) | Yes | Yes | Yes |
| **Tutee posts (content, full — own)** | Read | No | Yes (own only) | No | Yes |
| **Tutee posts (content)** | Create | No | Yes | No | Yes (on behalf) |
| **Tutee posts (content)** | Update (own) | No | Yes | No | Yes |
| **Tutee posts (content)** | Delete / Close (own) | No | Yes | No | Yes |
| **Contact fields** (email, phone, address) | Read (any) | No | No | No | Yes (access logged) |
| **Assigned counterparty phone/email** | Reveal in participant interface | No | Yes, only for authorized ACTIVE/COMPLETED matches; hidden while pending | Same participant and state checks | Separately authorized and audited |
| **Contact fields** | Write / Update (own) | No | Yes (012 step-up and confirmation) | Yes (012 step-up and confirmation) | Yes (authorized, audited) |
| **Match records** | Read (own match) | No | Yes (own) | Yes (own) | Yes |
| **Match records** | Read (all) | No | No | No | Yes |
| **Match records** | Create | No | No | No | Yes |
| **Match records** | Update status | No | Yes (first-session confirmation only) | No | Yes |
| **Match records** | Delete / Cancel | No | No | No | Yes |
| **Commission records** | Read (own) | No | No | Yes (own — amount and status only) | Yes |
| **Commission records** | Read (all) | No | No | No | Yes |
| **Commission records** | Create / Update | No | No | No | Yes |
| **Interview records** | Read (own request) | No | Yes (own) | Yes (own — summary only, no tutee contact) | Yes |
| **Interview records** | Read (all) | No | No | No | Yes |
| **Interview records** | Create (request) | No | Yes | No | Yes |
| **Interview records** | Propose/counter/cancel under negotiation rules | No | Own permitted actions only | Own permitted actions only | Authorized actions |
| **Interview records** | Schedule accepted slot and record outcome | No | No | No | Authorized actions |
| **Meeting links** | Read (own interview) | No | Yes (own interview only) | Yes (own interview only) | Yes |
| **Audit logs** | Read | No | No | No | Yes (DPO / senior admin only) |
| **Audit logs** | Write | No | No | No | System only (immutable) |
| **User accounts** | Create own | Yes (registration flow) | N/A | N/A | Yes |
| **User accounts** | Read own profile | No | Yes | Yes | Yes |
| **User accounts** | Update own profile | No | Yes | Yes | Yes |
| **User accounts** | Suspend / Ban any | No | No | No | Yes |

### D.3 Enforcing Permissions

1. Permission checks MUST be enforced server-side. Front-end user interface controls (hiding buttons, collapsing fields) are usability aids only and do not constitute access control.
2. Contact field access by admin MUST trigger an audit log entry before the field value is returned (see Section 10.6.2).
3. Public and authenticated discovery of anonymized tutee post content applies only to posts in Active state. Posts in Matched, Closed, Expired, Pending Review, or Revision Requested states are not included in the public discovery index query results. Scoped owner/invitation views of PENDING posts under specification 021 are separate.
4. The TUTEE first-session confirmation is the only match record update action available to non-admin roles. The update is constrained to advancing commission status from AWAITING CONFIRMATION to CONFIRMED on the tutee's own matched post only.
5. Role assignment is mutually exclusive: a user account cannot hold both TUTEE and TUTOR roles simultaneously (enforced at registration per BR-007).

---

*End of Business Requirements Document*

*Historical note: May 2026 revisions added interview requirements and folded AI-assisted review suggestions. The role labels in those records do not identify employees or independent human reviewers. September decisions and this public edit clarify the current rules without certifying implementation.*

*This document is intended as input to the System Design Document (SDD), feature specifications, and development planning. Any changes to business rules (Section 8) must be reviewed and approved by the Business Owner before implementation. The Data Governance Appendix (Appendix C) is a go-live gate and must receive DPO sign-off before production launch.*
