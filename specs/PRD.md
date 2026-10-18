# Product Requirements Document (product requirements document)

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements correction:** BR-001/M18 disclosure, tutor-to-agency receipt, retained first-session obligations and notification/availability measurement windows are reconciled. Historical feature and delivery snapshots below do not supersede explicit later amendments. See [current requirements](CURRENT-REQUIREMENTS.md). This corrects the document; it is not a new runtime verification.
## InTuition — Singapore Private Tuition Marketplace Platform

**Document Version:** 1.3 — Rebranded to InTuition; Tech Stack: React + TypeScript / Java Spring Boot  
**Date:** 2026-05-27  
**Status:** Draft  
**Prepared By:** Product Management  
**Source business requirements document:** business requirements document v1.2 — Interview Feature Added (2026-05-27)

---

## Table of Contents

1. [Overview & Vision](#1-overview--vision)
2. [User Personas](#2-user-personas)
3. [Feature Inventory — Mo S Co W Priority](#3-feature-inventory--mo-s-co-w-priority)
4. [User Journey Maps](#4-user-journey-maps)
5. [Post Field Specifications](#5-post-field-specifications)
6. [Approval Workflow Spec](#6-approval-workflow-spec)
7. [Matching Workflow Spec](#7-matching-workflow-spec)
8. [Commission Workflow](#8-commission-workflow)
9. [Notification Plan](#9-notification-plan)
10. [Public Pages (Unauthenticated)](#10-public-pages-unauthenticated)
11. [Authenticated Pages by Role](#11-authenticated-pages-by-role)
12. [Full Routes Inventory & application programming interface Contracts](#12-full-routes-inventory--application-programming-interface-contracts)
13. [Acceptance Criteria (P0 Features)](#13-acceptance-criteria-p0-features)
14. [Non-Functional Requirements](#14-non-functional-requirements)
15. [MVP vs Phase 2 Scope](#15-mvp-vs-phase-2-scope)
16. [Analytics Events](#16-analytics-events)

---

## 1. Overview & Vision

### 1.1 Product Vision

To be Singapore's most trusted private tuition agency by operating a curated, human-mediated marketplace — where every listing is vetted by admin, every match is made by a qualified coordinator, and no contact information is ever exposed between parties without a confirmed introduction.

### 1.2 Product Summary

The platform is a two-sided marketplace connecting tutees (parents seeking tutors for their children, or adult learners) with tutors (undergraduates, full-time freelancers, ex-MOE teachers, and specialists) across all MOE education levels and SEAB subject areas.

Unlike volume-based self-service platforms (Smile Tutor, Mind Flex, Start Tuition), this platform operates as a **concierge agency at web scale**:

- Every tutor post and tutee post is reviewed by admin before publication (per BR-002)
- Contact information — phone and email — is **never shown publicly**. Admin access is separately authorized/audited; assigned participants may see counterparty contact through the match-state gates in §20.6 (BR-001, amended by M18/INT-125)
- Visitors can browse and filter bounded anonymised lists without login; details/actions require the appropriate authentication and contact remains assignment-gated
- All matches are made exclusively by admin staff, not by algorithm (per BR-003)
- Commission is charged once per match, paid by the tutor, triggered after tutee confirms the first session (per BR-004, BR-010)

### 1.3 Business Context

Historical market assumption: demand for private tuition motivated the product proposal. Market-size and participation estimates in the original draft were not independently verified and are not presented as public facts.

### 1.4 Revenue Model

- **Commission:** 50% of the first month's tuition fee, paid by the tutor
- **Calculation:** commission hourly rate × hours per session × sessions per month × 0.5
- **Trigger:** Tutee confirms first session occurred (or admin override after 7-day timeout)
- **Collection:** Off-platform invoice/transfer in MVP
- **Target:** SGD 20,000/month commission revenue by Month 12 (50 matches/month × ~SGD 400 average)

### 1.5 Success Metrics (12-Month Targets)

| Metric | Target |
|---|---|
| Active tutor listings | 500 |
| Active tutee requests | 300 |
| Admin-confirmed matches/month | 50 |
| Commission collection rate | ≥ 85% |
| Commission revenue/month | SGD 20,000 |
| Post review service-level agreement (within 2 business days) | ≥ 95% |
| Median days tutee post → match | ≤ 7 days |
| Tutee post-match NPS | ≥ 50 |
| Tutor post-match NPS | ≥ 40 |

### 1.6 Scope Adjustments (v1.1)

**Priority Re-classifications:**

- **Account deletion (Personal Data Protection Act right-of-erasure) moved from P1 to P0.** Personal Data Protection Act right-of-erasure is a legal requirement. Admin-manual deletion is acceptable as a Day 1 workaround if automated deletion is not complete at launch, but must be documented and resolved within 30 days post-launch.
- **service-level agreement monitoring dashboard (amber/red queue indicators) moved from P0 to P1.** service-level agreement monitoring is valuable but not launch-blocking; queue time data is still captured and accessible in the review queue view.
- **Commission DISPUTED and WAIVED status workflow user interface moved from P0 to P1.** Admin can manually handle via notes in MVP; the status transitions (DISPUTED, WAIVED) remain P0 in the data model and commission state machine.

**Scope Exclusions:**

- Go-to-market strategy and tutor/tutee acquisition channels are out of scope for this document. Supply acquisition is a marketing function and must be addressed in a separate GTM plan.

---

## 2. User Personas

### 2.1 Persona: Parent Tutee — "The Anxious Primary School Parent"

**Name (representative):** Mei Lin, 38, Tampines  
**Role on Platform:** Tutee (submits tutee request post)

| Attribute | Detail |
|---|---|
| Situation | Mother of a P5 daughter preparing for PSLE. Daughter is weak in Science. |
| Primary Goal | Find a reliable, MOE-aligned Science tutor quickly without being spammed by tutor cold-calls. |
| Secondary Goal | Understand the matching process; receive progress updates without having to chase. |
| Pain Points (current alternatives) | Has tried Facebook parent groups — got 20 tutor Whats Apps in 48 hours, overwhelmed. Worried about tutor credibility. |
| Platform Value Proposition | Admin handles shortlisting. Contact info never shared without match. Tutors are vetted before listing. |
| Tech Comfort Level | Moderate. Uses mobile for everything. Comfortable with government portals (My Info, Parents Gateway). Expects mobile-first user experience. |
| Key user experience Requirements | Simple post form (≤10 min to complete). Clear status badge. One-click first-session confirmation. |
| Privacy Sensitivity | High. Does not want phone number or home address shared with any third party. |

---

### 2.2 Persona: Adult Student Tutee — "The Self-Improving Working Adult"

**Name (representative):** Darren, 28, Bishan  
**Role on Platform:** Tutee (submits tutee request post for self)

| Attribute | Detail |
|---|---|
| Situation | Polytechnic graduate retaking O-Level A-Maths as a private candidate. Works full-time, available only evenings. |
| Primary Goal | Find a tutor experienced with private candidates who can work around his schedule. |
| Secondary Goal | Budget-conscious; needs a tutor in the SGD 35–50/hr range. |
| Pain Points | Existing platforms list 100+ tutors with no easy filtering for private-candidate experience. |
| Tech Comfort Level | High. Desktop and mobile comfortable. Familiar with form-heavy platforms. |
| Key user experience Requirements | Granular schedule selector (evening-only); budget range input; clear post status tracking. |

---

### 2.3 Persona: Undergraduate Part-Time Tutor — "The NTU Student Side-Hustler"

**Name (representative):** Wei Jian, 21, Jurong West  
**Role on Platform:** Tutor (submits tutor offering post)

| Attribute | Detail |
|---|---|
| Situation | NTU Year 2 Engineering student, tutoring Sec 3–4 A-Maths and E-Maths on weekends for supplementary income. Charges SGD 30–40/hr. |
| Primary Goal | Find 2–3 steady students, earn SGD 500–800/month extra income without cold-calling parents. |
| Secondary Goal | Build a track record. Prefers assignments near Jurong West or online. |
| Pain Points | Other platforms require him to aggressively bid on assignments. Gets ghosted after profile views. |
| Commission Awareness | Low initially. Needs clear commission disclosure at registration so there are no surprises. (per BR-009) |
| Tech Comfort Level | High. Mobile-first. Expects quick form submission. |
| Key user experience Requirements | Commission disclosure step must be unambiguous. Availability grid (weekly slots). Post status notifications via email. |

---

### 2.4 Persona: Full-Time Freelance Tutor — "The Professional Freelancer"

**Name (representative):** Siti, 32, Bedok  
**Role on Platform:** Tutor (submits tutor offering post)

| Attribute | Detail |
|---|---|
| Situation | Full-time freelance tutor with 7 years' experience, specialising in O-Level Chemistry and A-Level H2 Chemistry. Charges SGD 65–80/hr. Manages 8–10 students. |
| Primary Goal | Fill 1–2 new student slots per month consistently. Prefers assignments in East Singapore or online. |
| Secondary Goal | Maintain professional credibility. Values a platform that has vetted tutees (not time-wasters). |
| Pain Points | Illustrative research hypothesis: users may receive unwanted contact or poorly matched enquiries. This scenario is not a verified claim about another provider. |
| Commission Awareness | Experienced; understands agency commission. Wants calculation to be fully transparent. |
| Tech Comfort Level | Moderate-High. Prefers desktop but expects mobile compatibility. |
| Key user experience Requirements | Rich post with multi-subject selection. Rate min–max field. Qualifications text. Match notification by email. |

---

### 2.5 Persona: Ex-MOE Teacher — "The Retired Educator"

**Name (representative):** Mr Rajendran, 58, Clementi  
**Role on Platform:** Tutor (submits tutor offering post)

| Attribute | Detail |
|---|---|
| Situation | Retired MOE secondary school teacher with 28 years of experience. Teaches GP, H2 History, and Social Studies. Charges SGD 90–120/hr. Selective — max 4 students. |
| Primary Goal | Attract serious JC students; does not want to deal with low-ball rate expectations. |
| Secondary Goal | Maintain dignity of process; does not want his contact details circulated. |
| Pain Points | Illustrative research hypothesis: users may receive unwanted contact or poorly matched enquiries. This scenario is not a verified claim about another provider. |
| Commission Awareness | High. Expects the rate to be stated upfront. |
| Tech Comfort Level | Low-Moderate. Prefers simple forms. May need guidance completing post. |
| Key user experience Requirements | Clear field labels with examples. Teaching approach free text. Profile photo upload. Strong privacy guarantee visible on registration page. |

---

### 2.6 Persona: Admin Staff — "The Agency Coordinator"

**Name (representative):** Cheryl, 27, Agency HQ  
**Role on Platform:** Admin

| Attribute | Detail |
|---|---|
| Situation | Joined the agency as a coordinator. Manages post review, matching, and commission tracking. Handles 25 matches/month at full capacity. |
| Primary Goal | Review posts efficiently, make quality matches, collect commission without chasing. |
| Secondary Goal | Keep both tutors and tutees informed so they don't email in asking for status updates. |
| Pain Points | Without a proper tool, she uses spreadsheets to track posts and matches. Missing notifications and audit trail. |
| Tech Comfort Level | High. Desktop-first. Comfortable with table-heavy admin dashboards. |
| Key user experience Requirements | multi-factor authentication on login. Review queue sorted by submission time. Filters on matching dashboard. Commission ledger with invoice generation. Immutable audit log for contact field access. |

---

## 3. Feature Inventory — Mo S Co W Priority

Features are tagged P0 (Must Have — MVP), P1 (Should Have — MVP stretch or immediate post-launch), and P2 (Nice to Have — Phase 2+). P0 features constitute the launch-critical scope.

### 3.1 Authentication & Account Management

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Tutee registration with email verification | P0 | BR-007 |
| Tutor registration with email verification | P0 | BR-007 |
| Role enforcement: mutually exclusive tutee/tutor roles | P0 | BR-007 |
| Admin login with time-based one-time passcode multi-factor authentication enforcement | P0 | BR-001, §10.6.1 |
| Password reset flow | P0 | — |
| Account profile edit (own details, non-contact fields) | P0 | Appendix D |
| Contact field edit (own email/phone via012 confirmation; visibility under amended BR-001) | P0 | BR-001, Appendix B |
| Account deletion / data subject withdrawal | **P0** | Appendix C §C.4 |
| Admin user management (suspend, deactivate, ban) | P0 | Appendix D |

### 3.2 Tutor Commission Disclosure

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Standalone commission disclosure screen at tutor registration (mandatory step) | P0 | BR-009 |
| Affirmative acknowledgement (checkbox) logged as system event | P0 | BR-009 |
| Disclosure must include: rate, calculation method, worked example, trigger, payer | P0 | BR-009 |
| Tutor profile not approvable by admin until acknowledgement is logged | P0 | BR-009 |

### 3.3 Post Creation & Management

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Tutee: create tuition request post (all fields per §5.2) | P0 | BR-002, BR-007 |
| Tutor: create tuition offering post (all fields per §5.1) | P0 | BR-002, BR-007 |
| Post save as draft before submission | P0 | Appendix A |
| Submit post for admin review | P0 | BR-002 |
| Tutee: edit own post (when in Draft or Revision Requested state) | P0 | BR-008 |
| Tutor: edit own post (when in Draft or Revision Requested state) | P0 | BR-008 |
| Tutee/Tutor: close own active post | P0 | BR-008 |
| Tutee/Tutor: view own post status and history | P0 | §6.4 |
| Auto-expiry of unmatched active posts after configurable period (default 60 days) | P0 | Appendix A §A.3 |
| System notification on auto-expiry | P0 | §6.4 |
| Multiple active posts per tutor (tutor may hold several active offering posts) | P1 | Appendix A §A.2 |

### 3.4 Admin Post Moderation

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Admin review queue (all pending posts, sorted by submission time) | P0 | BR-002 |
| Per-post view including full contact info (admin only) | P0 | BR-001 |
| Approve action | P0 | BR-002 |
| Reject action with mandatory reason from enum | P0 | BR-002, §6 |
| Request Revision action with message | P0 | BR-002 |
| Immutable audit log entry on every contact field access | P0 | §10.6.2, Appendix B |
| Post revision re-enters approval queue | P0 | BR-002 |
| Internal notes per post (admin comments for handoff) | P1 | §6.3 |
| Bulk action on review queue (approve/reject multiple) | P2 | — |

### 3.5 Public Tutor Listing (Unauthenticated Browse)

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Publicly browseable approved tutor listings (no contact info) | P0 | §3.1 |
| Filter by subject, level, venue type, location area | P0 | §3.1 |
| Individual tutor post detail page (no contact info shown) | P0 | BR-001 |
| Pagination of listing results | P0 | — |

> **Login Gate Principle:** Unauthenticated users may browse all public pages (tutor listings, rate guide, landing). Any action that creates, modifies, or applies for a post — including "Request Interview", "Post a Request", "Create Listing" — must require the user to log in or register first. Clicking a gated action redirects to sign-in while retaining the intended destination so the user is returned to their intent after authentication.

### 3.5a Tuition Rate Guide (Public Reference Page)

A publicly accessible reference page showing typical Singapore private tuition rates. No login required.

| Feature | Priority | Notes |
|---|---|---|
| Rate guide page at /rates (publicly accessible) | P0 | No login required |
| Rate table by education level: Pre-school, Lower Primary, Upper Primary, Lower Sec, Upper Sec, JC, IB, Poly/Uni | P0 | Data sourced from Singapore market benchmarks |
| Rate breakdown by tutor type: Part-time (undergrad/student), Full-time freelancer, Ex-MOE/school teacher | P0 | Three-column table per level |
| Reference data from market (Smile Tutor, Mind Flex benchmarks) with "actual rates may vary" disclaimer | P0 | Not InTuition's own commission structure |
| CTA row: "Browse Tutors" (public) and "Post a Request" (login-gated) | P0 | Login gate applies to "Post a Request" |
| FAQ section: 3–5 common rate questions (why rates vary, group vs 1-on-1, ex-MOE premium) | P1 | — |
| Link from Homepage stats section and from tutor/tutee post creation forms | P0 | Contextual rate guidance at point of form entry |

### 3.6 Tutee Request Discovery (Authenticated Tutors)

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Approved tutors (logged in) can browse anonymised tutee request posts | P0 | §3.1, Appendix D |
| Tutee posts shown in discovery index only when in Active state (not Matched, Closed, etc.) | P0 | BR-008, Appendix D §D.3 |
| No contact info shown on tutee discovery posts (name shown as pseudonym only) | P0 | BR-001 |
| Filter tutee requests by subject, level, venue, area, budget range | P0 | §7 |
| **Free-market apply — tutor applies to any ACTIVE job, NO published listing required** | P0 | M18 §20 (removes TUTOR NO ACTIVE POST guard) |
| **Apply gated only by profile completeness** (rate/area/PT-FT/qualifications) | P0 | M18 §20 (replaces listing gate; throttles spam) |

### 3.7 Admin Matching Dashboard

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Two-panel matching dashboard: tutee requests (left) + tutor offerings (right) | P0 | §7 |
| Filter both panels by subject, level, venue type, area/MRT, budget/rate overlap | P0 | §7 |
| Create Match action linking one tutee post to one tutor post | P0 | BR-003 |
| Cardinality enforcement: tutee post → max 1 pending/active match | P0 | §4 (match cardinality) |
| Tutor post → multiple simultaneous matches permitted | P0 | §4 (match cardinality) |
| Match lifecycle status tracking: PENDING → ACTIVE → COMPLETED/CANCELLED | P0 | Appendix A §A.3 |
| Admin cancel match action | P0 | Appendix A §A.3 |
| Matched tutee post removed from tutor discovery index | P0 | BR-003, §3.1 |
| Re-entry of tutee post into discovery index on match cancellation | P0 | §3.1, Appendix A §A.3 |
| Admin internal notes per match | P1 | §6.3 |
| **Parent-facing applicant view (personal information-free)** — parent sees who applied to their own job | P0 | M18 §20.6 (distinct personal information-free data transfer object + owner-auth) |
| **Parent "choose tutor" selection request** → admin queue | P0 | M18 §20.3 (submit operation for tutee posts id select) |
| **Admin "close job" = atomic assign** (winner→MATCHED, others→SUPERSEDED, job→MATCHED) | P0 | M18 §20.4 (set-based + optimistic-lock) |
| **Reopen on cancel** (job→ACTIVE, superseded apps→APPLIED) | P0 | M18 §20.4 |
| **Stuck-PENDING-commission admin view + age alert** | P0 | M18 §20.8 (pay-after-first-session has no system deadline) |

### 3.8 Commission Workflow

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Commission calculation displayed when admin marks match ACTIVE | P0 | BR-004, §8 |
| Tutee first-session confirmation email with action link | P0 | BR-010 |
| 7-day timeout notification and admin alert if tutee does not confirm | P0 | BR-010 |
| Admin override to advance commission to CONFIRMED | P0 | BR-010 |
| Commission invoice generation (admin-triggered after CONFIRMED) | P0 | §8 |
| Invoice fields: tutor display name, tutee initials, subject/level, monthly fee, rate, amount, due date | P0 | §8 |
| Admin marks commission PAID manually | P0 | §8 |
| Commission status: PENDING / AWAITING CONFIRMATION / CONFIRMED / INVOICED / PAID / DISPUTED / WAIVED | P0 | §3.3 |
| Commission status visible to tutor (amount + status only, per Appendix D) | P0 | Appendix D |
| Commission DISPUTED and WAIVED status workflow user interface | **P1** | §8 (admin handles via notes in MVP) |
| In-platform payment collection | — | Out of scope (Phase 2) |

### 3.9 Notifications

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Email notification on all 15+ trigger events (see §9 Notification Plan) | P0 | §6.4 |
| In-platform notification inbox (bell icon) | P1 | — |
| SMS notifications | P2 | — |

### 3.10 Security & Compliance

| Feature | Priority | business requirements document Reference |
|---|---|---|
| personal information encryption at rest (email, phone — deterministic AES-SIV or equivalent) | P0 | §10.7.1, Appendix B |
| Contact fields excluded from all public application programming interface responses | P0 | BR-001, Appendix B |
| Personal Data Protection Act-compliant consent language at registration | P0 | §10.1 |
| Data residency: all data in Singapore region | P0 | §10.7.2 |
| role-based access control enforcement server-side | P0 | Appendix D §D.3 |
| Immutable audit log for contact field access (separate store, 5-year retention) | P0 | §10.6.2 |
| Commission acknowledgement log (write-once, 5-year retention) | P0 | BR-009, Appendix B |
| Mobile-first user interface (tutee and tutor flows, 390px primary viewport) | P0 | §10.7.3 |
| Account deletion / right to erasure flow | **P0** | Appendix C §C.4 |
| Data export on request (Personal Data Protection Act right of access) | P1 | Appendix C §C.4 |
| DPO sign-off on data governance before go-live | P0 (gate) | Appendix C |

### 3.11 Public Content Pages

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Landing page with value proposition and CTAs | P0 | §10 |
| How It Works pages (tutee flow, tutor flow) | P0 | §10 |
| About page | P0 | §10 |
| FAQ page | P0 | §10 |
| Contact page | P0 | §10 |
| Privacy Policy page | P0 | Personal Data Protection Act |
| Terms & Conditions page | P0 | §12.7 |

### 3.12 Interview Sessions (USP)

| Feature | Priority | business requirements document Reference |
|---|---|---|
| Tutee can request interview from approved tutor post detail page | P0 | BR-011 |
| Interview concurrency limit: max 3 open requests per tutee at any time | P0 | BR-011 |
| Interview cooldown: no new request for same tutor post within 30 days of completed/no-show | P0 | BR-011 |
| Admin interview queue: all REQUESTED interviews sorted by submission time | P0 | BR-013 |
| Admin schedules interview: sets date/time and meeting link (any URL) | P0 | BR-013, BR-014 |
| Meeting link storage and delivery to both tutee and tutor via platform notification | P0 | BR-014 |
| Interview duration display: 15 minutes stated in notification copy (not technically enforced) | P0 | BR-012 |
| Admin updates meeting link before scheduled time | P0 | BR-014 |
| Admin marks outcome: COMPLETED / CANCELLED / NO SHOW | P0 | BR-013 |
| Tutee: my interviews list (status badge, meeting link for SCHEDULED) | P0 | — |
| Tutor: my interview requests list (date/time, meeting link, tutee request summary — no contact info) | P0 | — |
| No contact information shared through interview workflow | P0 | BR-015 |
| Interview analytics events (request, schedule, complete, no-show, conversion-to-match) | P0 | business requirements document §9.5 |
| Admin: update interview record (reschedule link, change time) | P1 | — |
| Automated reminder to both parties 24h before scheduled interview | P1 | — |

---

## 4. User Journey Maps

### 4.1 Tutee Journey


These steps preserve the original journey. They describe requirements, not verified delivery. Current requirements C01 and C04 allow bounded public browsing and restrict contact disclosure to authorized assigned participants. C06 and C07 replace the older interview eligibility and limit rules noted below.

1. **Register.** The parent or adult learner chooses “Post a Request” or “Sign Up as Tutee” and provides their name, email, phone and password. The original journey then sends an email verification link, activates the account when it is followed, and presents privacy consent. The current consent and registration sequence takes priority over this historical order; see [parent registration](sequences/01-register-tutee.md). Contact storage follows the readable-text amendment, with disclosure restrictions unchanged.
2. **Understand privacy.** On first login, show an explanation that contact stays private while browsing and choosing, and becomes available to the assigned participants after activation. Record the user's “I Understand” acknowledgement.
3. **Create a request.** Collect the fields in §5.2, allow a draft to be saved, and submit the request as Pending Review. Show “Post Under Review” and acknowledge receipt by email with the stated two-business-day review expectation.
4. **Receive the review outcome.** Approval makes the request Active and sends an approval email. A revision request includes a reason and lets the parent edit and resubmit for review. Rejection includes the recorded reason. The original approved-tutors-only discovery wording is historical; C01 governs public browsing and restricted details/actions retain their own access checks.
5. **Wait and consider tutors.** The live request exposes subject, level, area, schedule and budget without private name, phone or email. Staff may review suitable tutors while the parent browses tutor listings.
6. **Optionally request an interview.** The original listing-based path lets the parent choose a tutor and add a note of up to 300 characters. A REQUESTED interview alerts staff through N-21. Staff contact the tutor to agree a 15-minute slot, enter a meeting link and date/time, and mark it SCHEDULED. N-22 gives the parent the link, time and instructions; N-23 gives the tutor the link, time and anonymised request summary. Both join independently. Staff record COMPLETED, CANCELLED or NO_SHOW and follow up about a formal match. No interview alone releases contact details. The original limit of three open interviews per account, counted only as REQUESTED or SCHEDULED, is superseded by C07's per-post limits and negotiation states; the original 30-day repeat restriction after completion or no-show is retained for traceability. C06 also allows initiation from the parent's own pending request.
7. **Receive an admin-created match.** The original path pairs a parent request with a tutor post; under C05 a tutor listing is optional and assignment uses the eligible tutor profile. Assignment moves the request to Matched, removes it from discovery and creates commission in PENDING. The parent receives email and a dashboard notice that a coordinator will contact them within one business day.
8. **Agree the arrangement.** Staff use their authorized, audited access to contact both people off-platform and discuss the introduction. Staff then mark the match ACTIVE. Under C03 activation fixes the agreed first-month plan; under C04 the website can now reveal contact to the assigned participants.
9. **Confirm the first session.** Activation starts the confirmation workflow and commission becomes AWAITING_CONFIRMATION. The parent receives the confirmation email and confirms that the session occurred; commission then becomes CONFIRMED. No response leads to the seven-day reminder and admin alert, with an authorized override path. The start of those seven days is unresolved; see AC-05. Confirmation alone does not complete the match.
10. **After confirmation.** Staff generate the invoice and send it to the tutor. The source's general “closed when arrangement ends” wording is superseded by C02/C03: agency receipt marked PAID completes the match, which does not mean all teaching has ended. A satisfaction survey remains a post-MVP or manually triggered possibility.


---

### 4.2 Tutor Journey


These steps retain the original tutor journey with explicit later corrections. They do not show that the website has implemented every step.

1. **Register.** The tutor chooses “List as Tutor” or “Sign Up as Tutor” and supplies their name, email, phone and password. The original diagram placed account creation, email verification and privacy consent before commission acknowledgement. That order is superseded by [the current tutor registration sequence](sequences/02-register-tutor.md): collect the required legal and commission acknowledgements before final account creation, offer optional and skippable profile capture, and save acknowledgements and account audit together atomically. Later actions still require the approved profile completeness.
2. **Read and acknowledge commission terms.** The disclosure remains a mandatory standalone step. Explain the 50% first-month commission, calculation method, a worked numerical example, the first-session confirmation trigger, the tutor as payer and the one-time-per-match charge with no recurring fee. An unchecked acknowledgement blocks progression. Record tutor identity, timestamp, terms version and affirmative action; staff cannot approve a profile without that record. See §8.1 for calculation details and [the UX wireframes](domain/uiux/wireframes.md) for presentation.
3. **Optionally publish an offering.** The original journey required a tutor post; C05 makes publication optional for application and assignment. If the tutor chooses to publish, collect §5.1 fields, permit saving a draft, and submit it as Pending Review. Acknowledge receipt by email with the two-business-day review expectation.
4. **Receive the listing review.** Staff see the post, contact information and commission acknowledgement. Approval makes the listing Active and sends a “listing is live” email; the public listing excludes private contact. Revision and rejection include reasons. These publication checks do not impose a listing requirement on an otherwise eligible tutor.
5. **Browse and apply.** The original discovery screen shows level, subject, area, budget and schedule without parent contact. C01 governs bounded public browsing; application and detail access have their own checks. Applying expresses interest only, parent choice requests staff action, and only staff assignment creates the match. No pre-assignment contact is released.
6. **Receive an assignment.** Staff assign the tutor to a request, using the eligible profile whether or not a tutor listing exists. The tutor receives email and a dashboard message that a coordinator will contact them within one business day.
7. **Agree the arrangement.** Staff contact the tutor using authorized stored details and facilitate the introduction. Activation fixes the first-month plan and makes eligible participant contact available. The dashboard says the arrangement is active and commission will be invoiced after first-session confirmation.
8. **Complete the commission process.** The parent confirms the first session, or staff use the authorized override process. Staff send the tutor an invoice due within 14 days and record PAID only when the agency receives payment. A parent paying for lessons is a separate event. A later satisfaction survey is optional, as in the original journey.
9. **Maintain availability and any listing.** A published tutor post remains Active after a match because a tutor may have multiple simultaneous matches. Changes to availability, rates or subjects return the listing to review under the applicable moderation rules. C05 keeps listing-specific overrides separate from the main tutor profile.


---

### 4.3 Admin Journey


The following is the original operational journey with current-rule caveats. Each step remains a requirement unless explicitly superseded; the list is not a statement that all admin screens are available.

1. **Sign in.** The original flow requires email, password and a time-based one-time passcode before admin access, with no bypass. C09 and [production account setup](027-aws-production-deployment/spec.md) keep admin authentication and session controls specific to the environment; the original universal wording does not replace those rules.
2. **Review posts.** The queue lists Pending Review posts oldest first. Detail includes name, email, phone, subject, level, rate or budget, availability, location and qualifications, plus the tutor's commission acknowledgement. Every contact-field view creates an immutable audit entry. Approve makes the post Active and notifies its owner. Reject records a selected reason and notifies the owner. Request Revision records a message and notifies the owner. C04 separately governs assigned participant access; “admin only” in the old diagram means this staff view, not a ban on authorized participant contact after activation.
3. **Manage interviews alongside post review.** The original queue lists REQUESTED interviews by submission time, showing parent display name and note, tutor summary and audited parent contact. Staff agree a slot with the tutor, enter a future date/time, required meeting link and optional notes, then schedule it. Send N-22 to the parent and N-23 to the tutor. Afterward, record completion and follow up off-platform, cancellation, or no-show with the absent person identified. The old diagram names only the parent for N-24; AC-19 requires cancellation notice to both participants. C06/C07 add pending-post initiation, per-post limits and negotiation states to this earlier queue description.
4. **Assign a tutor.** The original matching dashboard places active requests and tutor offerings side by side, filtered by subject, level, venue, area or MRT and budget/rate overlap. C05 allows assignment without a published offering. Staff choose an eligible tutor for one request. Block a second pending or active match for the same request. Notify both people, mark the request Matched and remove it from discovery.
5. **Manage the match.** Keep arrangement status PENDING, ACTIVE, COMPLETED or CANCELLED separate from commission status. Activation fixes the plan, requests first-session confirmation and advances commission to AWAITING_CONFIRMATION. Monitor the seven-day follow-up, whose start is unresolved in AC-05. Staff can use the authorized confirmation override, invoice after CONFIRMED, and record payment actually received by the agency. Receipt marked PAID completes the match; first-session confirmation alone does not. Cancellation notifies both people and reopens an eligible request under §20.4. After invoicing, staff must explicitly decide WAIVED or DISPUTED for commission; cancellation must not silently choose a billing outcome.
6. **Review the commission ledger.** Show matches, commission states and amounts, with status, date-range and tutor filters. Accounting export remains a future feature. Dedicated dispute/waiver screens remain P1 while the required states and manual notes remain in MVP scope (§1.6).
7. **Manage users.** Show parent and tutor accounts with authorized, audited contact access. The original actions are suspend, reactivate, ban and force-close posts. Apply the current related-request closure and record-retention rules when changing account or post status.
8. **Review audit records.** Senior staff or the data-protection role can inspect contact-access and commission-acknowledgement logs. They cannot edit or delete those records through an application interface.


---

## 5. Post Field Specifications

### 5.1 Tutor Offering Post Fields

| Field | Type | Required | Constraints | Visibility |
|---|---|---|---|---|
| **Title / Headline** | Text input | Yes | 100 chars max | Public (no contact info) |
| **Subjects Taught** | Multi-select (SG taxonomy) | Yes | Min 1 selection; see SG subject list below | Public |
| **Levels Taught** | Multi-select (MOE levels) | Yes | Min 1 selection; see MOE level list below | Public |
| **Hourly Rate (min)** | Number (SGD) | Yes | Positive integer; must be ≤ rate max | Public |
| **Hourly Rate (max)** | Number (SGD) | Yes | Positive integer; must be ≥ rate min | Public |
| **Venue Types** | Multi-select | Yes | Options: Student's Home / Tutor's Home / Public Place / Online / Tuition Centre | Public |
| **Group Size** | Multi-select | Yes | Options: 1-on-1 Only / Small Group (2–4) / Both | Public |
| **Availability** | Weekly grid | Yes | Mon–Sun × Morning / Afternoon / Evening; at least 1 slot selected | Public |
| **Qualifications** | Free text | Yes | 500 chars max | Public |
| **Years of Tutoring Experience** | Number | Yes | 0–50 range | Public |
| **Teaching Approach** | Free text | No | 500 chars max | Public |
| **Location / Area** | Select (MRT station or planning area) | Conditional | Required if any venue type that is in-person is selected; optional if Online only | Public (area only, not home address) |
| **Profile Photo** | Image upload | No | JPEG/PNG/Web P; max 5 MB; stored with storage-level encryption per Appendix B; admin and own user only — excluded from all non-authenticated and non-admin application programming interface responses | Admin + own user |
| **Contact Email** | Email input | Yes | Must match registered email or alternate; **private before assignment per amended BR-001**; plaintext user-contact amendment applies | Own user/admin; assigned participants under match-state gates |
| **Contact Phone** | Phone input | Yes | Singapore mobile format (+65 8/9XXXXXXX preferred); **private before assignment per amended BR-001**; plaintext user-contact amendment applies | Own user/admin; assigned participants under match-state gates |

**Subject Taxonomy (Singapore — abbreviated; full list per §12.2):**

| Level Group | Sample Subjects |
|---|---|
| Kindergarten | Phonics, Early Literacy, Early Numeracy |
| PSLE (P1–P6) | English, Mathematics, Science, Chinese Language, Malay Language, Tamil Language, Higher Chinese/Malay/Tamil, Foundation English, Foundation Maths |
| O-Level | English Language, Elementary Mathematics, Additional Mathematics, Physics, Chemistry, Biology, Combined Science (Phy/Chem, Chem/Bio, Phy/Bio), History, Geography, Literature in English, Social Studies, Principles of Accounts (POA), Chinese/Malay/Tamil Language, Higher Chinese/MT |
| A-Level (JC) | General Paper (GP), H1/H2 Mathematics, H2 Further Mathematics, H2 Physics, H2 Chemistry, H2 Biology, H2 Economics, H2 History, H2 Geography, H2 Literature in English, H1/H2 Mother Tongue, Knowledge & Inquiry (KI) |
| IB | IB HL/SL Maths AA, Maths AI, Physics, Chemistry, Biology, Economics, History, English A/B, Chinese |
| Polytechnic / University | Engineering Mathematics, Business Statistics, Calculus, Statistics, Programming, Economics, Accounting, module-specific |
| Adult / Private Candidate | IGCSE, Private Candidate O-Level, Adult English, Adult Mathematics |

**MOE Level List (for multi-select):**

Kindergarten K1, Kindergarten K2 | Primary P1, P2, P3, P4, P5, P6 | Secondary Sec 1 (Express), Sec 2 (Express), Sec 3 (Express), Sec 4 (Express) | Secondary Sec 1–5 (Normal Academic) | Secondary Sec 1–4 (Normal Technical) | IP Year 1–4 | JC1, JC2 (A-Level) | IB Year 1–2 | Polytechnic Year 1–3 | University Year 1–4 | Adult / Private Candidate

---

### 5.1a Tutor Profile Fields — canonical identity (M18, free-market)

In the free marketplace a tutor applies to jobs **without** a published listing, so the **profile**
(not the post) is the canonical public identity. The applicant card and Browse Tutors read the
profile; an attached listing's rate/area **overrides** for that card when present.

| Field | Type | Required | Validation / Notes |
|-------|------|----------|--------------------|
| Display name | Text | Yes | existing |
| Qualifications | Free text | Yes | existing, 500 max |
| Years of experience | Number | Yes | existing, 0–50 |
| Bio | Free text | No | existing, 1000 max |
| **Commitment (Part-time / Full-time)** | Enum | **Yes** | **NEW** — the tutor's part-time or full-time commitment; **dropdown, required at profile setup AND editable later** |
| **Hourly rate range (min–max)** | Number–Number | **Yes** | **NEW on profile** — S$10–1000, max≥min (today only on post) |
| **Location (planning area / region)** | Select | **Yes** | **NEW on profile** — coarse area for personal information-safe card (today only on post) |

> **Migration:** backfill profile rate/area from each tutor's most-recent Tutor Post; tutors with no
> prior post get null until they edit their profile. Profile-completeness (rate + area + commitment +
> qualifications) is the **gate to apply** (replaces the old active-listing requirement). Identity
> model = **profile-canonical + listing-overrides-when-attached** (no sync job).

---

### 5.2 Tutee Request Post Fields

| Field | Type | Required | Constraints | Visibility |
|---|---|---|---|---|
| **Post Title** | Text input | No | 150 chars max; auto-generated if blank (e g., "P6 Science Tutor Needed — Tampines") | Public (anonymised) |
| **Subject** | Single select (SG taxonomy) | Yes | One subject per post | Public |
| **Level** | Single select (MOE levels) | Yes | One level per post | Public |
| **Number of Students** | Select | Yes | Options: 1 / 2 / 3 or more | Public |
| **Student Relationship** | Select | Yes | Options: Parent's child / Self / Other | Public |
| **Budget (min, SGD/hr)** | Number | Yes | Positive integer; must be ≤ budget max | Public |
| **Budget (max, SGD/hr)** | Number | Yes | Positive integer; must be ≥ budget min | Public |
| **Preferred Venue Types** | Multi-select | Yes | Same options as tutor offering: Student's Home / Tutor's Home / Public Place / Online / Tuition Centre | Public |
| **Preferred Schedule** | Weekly grid | Yes | Mon–Sun × Morning / Afternoon / Evening; at least 1 slot | Public |
| **Session Duration** | Select | Yes | Options: 1 hr / 1.5 hr / 2 hr / 2.5 hr | Public |
| **Frequency per Week** | Select | Yes | Options: 1× / 2× / 3× / Flexible | Public |
| **Preferred Start Date** | Date input | No | Must be today or future | Public |
| **Special Requirements** | Free text | No | 500 chars max | Public |
| **Location / Area** | Select (MRT station or planning area) | Yes | MRT station name or planning area (e g., Tampines, Buona Vista, Bishan) — not home address | Public (area only) |
| **Contact Email** | Email input | Yes | **Private before assignment per amended BR-001**; plaintext user-contact amendment applies | Own user/admin; assigned participants under match-state gates |
| **Contact Phone** | Phone input | Yes | Singapore mobile format; **private before assignment per amended BR-001**; plaintext user-contact amendment applies | Own user/admin; assigned participants under match-state gates |

**Note:** The tutee's display name on the discovery index is a pseudonym (e g., initials or first name only). Public discovery uses the sanctioned public handle rather than private full name. Contact email/phone remain hidden before assignment and use participant-only match-state disclosure afterward (amended BR-001/INT-125).

---

### 5.3 Location Input Specification

The platform must provide a structured location selector supporting:

1. **MRT Station** — dropdown of all Singapore MRT/LRT stations (EWL, NSL, CCL, DTL, TEL, NEL, etc.)
2. **Planning Area** — dropdown of all 55 Singapore URA planning areas (e g., Tampines, Buona Vista, Bishan, Clementi, Jurong East)
3. **Broad Region** — derived automatically from planning area: Central / North / South / East / West

The matching dashboard uses all three levels for filter compatibility.

---

## 6. Approval Workflow Spec

### 6.1 Review Queue

- Admin navigates to /admin/posts/review
- Queue displays all posts in Pending Review state
- Default sort: submission timestamp ascending (oldest first, to ensure service-level agreement compliance)
- Each row shows: post type (Tutee/Tutor), submitter display name, subject, level, submission timestamp, time-in-queue
- service-level agreement indicator: posts approaching or past 2 business days highlighted in amber/red (P1 — see §1.6)
- service-level agreement target: 95% of posts reviewed within 2 business days (per §9.2)

### 6.2 Per-Post Review View

Admin clicks a post to open the full review view. This view renders:

- All post content fields (as submitted)
- **Contact information section** (admin-only panel): full name, email, phone — access to this section triggers an immutable audit log entry per §10.6.2
- For tutor posts: commission acknowledgement log entry (tutor user ID, timestamp, version, affirmative action)
- Post history: any previous submissions, revision requests, rejection reasons
- Internal admin notes field (free text, not visible to user)

### 6.3 Admin Actions

| Action | Trigger | Result | User Notification |
|---|---|---|---|
| **Approve** | Admin clicks Approve | Post status → Active; post enters public listing / discovery index | Email: "Your post has been approved and is now live." |
| **Reject** | Admin selects reason from enum + clicks Reject | Post status → Rejected (terminal) | Email: "Your post could not be approved. Reason: [selected reason]." |
| **Request Revision** | Admin enters message + clicks Request Revision | Post status → Revision Requested; user sees reason in dashboard | Email: "Revision required for your post: [admin message]." |

### 6.4 Rejection Reason Enum

The following values are the only permissible rejection reasons. Admin must select one from this list (free text may be appended as a note but the enum is required):

| Code | Label |
|---|---|
| INCOMPLETE INFO | Incomplete information |
| UNREALISTIC RATE | Unrealistic rate |
| INAPPROPRIATE CONTENT | Inappropriate content |
| DUPLICATE POST | Duplicate post |
| SPAM TEST | Spam or test post |
| MISSING QUALIFICATIONS | Missing qualifications |
| RATE OUT OF MARKET | Rate out of market range |

### 6.5 Resubmission Flow

- After a Revision Requested status, the user edits the post and clicks "Resubmit for Review"
- Post status reverts to Pending Review and re-enters the review queue
- The queue entry shows the resubmission count and the original submission date
- Admin sees the revision history inline in the review view

### 6.6 service-level agreement Monitoring

- System calculates "time in queue" for each post (business hours only: Singapore time 09:00–18:00 Mon–Fri)
- If a post exceeds 1 business day without action: amber indicator (P1)
- If a post exceeds 2 business days without action: red indicator + optional email alert to admin (P1)
- service-level agreement report (% reviewed within 2 business days, rolling 7-day) accessible on admin dashboard (P1)

### 6.7 Admin multi-factor authentication Requirement

- time-based one-time passcode-based multi-factor authentication (Google Authenticator or equivalent) is the minimum requirement for all admin accounts
- Hardware security keys (FIDO2) are supported but not required
- Admin users who have not enrolled multi-factor authentication must be redirected to multi-factor authentication enrollment before accessing any admin route (see AC-07)
- The /admin/login page label reads "Admin Login" with time-based one-time passcode as the primary multi-factor authentication method

---

## 7. Matching Workflow Spec

### 7.1 Matching Dashboard Layout

Admin navigates to /admin/matching. The screen is divided into two scrollable panels:

| Left Panel | Right Panel |
|---|---|
| Tutee Request Posts (Active, not currently Matched) | Tutor Offering Posts (Active) |
| Filterable | Filterable |

### 7.2 Filter Set (Both Panels)

| Filter | Type | Values |
|---|---|---|
| Subject | Select (SG taxonomy) | Full subject list |
| Level | Select (MOE levels) | Full level list |
| Venue Type | Multi-select | Student's Home / Tutor's Home / Public Place / Online / Tuition Centre |
| Area / MRT | Text search + select | MRT station or planning area |
| Budget / Rate Overlap | Range slider | SGD min–max; filters show tutee posts whose budget overlaps with tutor's rate range |

Applied filters affect both panels simultaneously (admin may also pin a filter to one panel only via a toggle).

### 7.3 Match Creation

1. Admin clicks a tutee post in the left panel → post highlighted
2. Admin clicks a tutor post in the right panel → "Create Match" button activates
3. System checks cardinality constraint: tutee post must not already have a PENDING or ACTIVE match
   - If constraint violated: system shows error "This tutee request already has an active match. Cancel the existing match first."
4. Admin clicks "Create Match" → confirmation modal:
   - Shows tutee post summary (subject, level, area, budget) and tutor post summary (name, subjects, rate, area)
   - "Confirm Match" button
5. On confirmation:
   - Match record created with status PENDING
   - Commission record created on commissions table with status PENDING (see §7.8 Data Model)
   - Tutee post state → Matched (removed from discovery index)
   - Tutor post state remains Active (may participate in further matches)
   - Both parties receive match notification email (see §9)

### 7.4 Match Cardinality Rules

| Rule | Enforcement |
|---|---|
| Tutee post: max 1 pending or active match at a time | Partial unique index on tutee post id when match status IN PENDING, ACTIVE |
| Tutor post: multiple simultaneous matches permitted | No cardinality constraint on tutor post |
| Matched tutee post removed from discovery index | Filter applied to discovery index query: exclude posts when status MATCHED |

### 7.5 Match Lifecycle

| Status | Description | Admin Action Available |
|---|---|---|
| PENDING | Match created; admin has not yet confirmed arrangement is underway | Mark ACTIVE / Cancel |
| ACTIVE | Admin confirmed arrangement is underway; tutee confirmation pending | Cancel / Close |
| CANCELLED | Match cancelled before arrangement confirmed → job reopens (§20.4) | (terminal) |
| COMPLETED | Commission marked PAID after first session (§8.3); arrangement complete | (terminal) |

Commission status tracks separately on the commissions table (see §7.8). They are separate dimensions with an explicit transition rule: recording received agency commission as PAID completes the match.

> **Terminology note (M18):** the code's terminal match status is **COMPLETED** (set when the
> commission is marked PAID), not "CLOSED". Earlier drafts of this table and the match closed
> analytics event (§16.4) said CLOSED — **COMPLETED is authoritative**. Separately, the *job*
> (Tutee Post) uses Post Status MATCHED while assigned; "close the job" in §20 means the job left
> ACTIVE (→ MATCHED), which is distinct from the match reaching COMPLETED.

### 7.6 Match Notification Content

On match creation, both parties receive the following email:

> **Subject:** Tuition Match Found — Action Required  
> **To:** [Tutee or Tutor email — sent by admin via system, contact info accessed from admin panel]  
> **Body:** "We are pleased to inform you that we have identified a potential tuition match for your request. Our coordinator will contact you within 1 business day to facilitate the introduction. Please ensure your phone is reachable."

No contact information of the other party is included in this notification.

### 7.7 Scheduler Specifications

#### 7.7.1 First-Session Confirmation Timeout Job

**Separate unresolved start, O13:** The activation-based seven-day clock below conflicts with AC-05's email-send start and the original BR-010 notification-receipt start. See [AC-05](#ac-05-first-session-confirmation-before-commission-invoice-br-010). No start is selected here; this is distinct from O12's detection-versus-delivery timing question.

> **Timing dependency O12:** This retained six-hour detector/window conflicts with the five-minute-from-trigger target when “trigger” means the seven-day threshold. The service-level agreement clock must be reconciled in [CURRENT-REQUIREMENTS md](CURRENT-REQUIREMENTS.md) before timing acceptance is closed. Both values below remain source requirements pending that decision.

- **Job name:** first session confirmation timeout
- **Schedule:** Every 6 hours, Singapore time timezone (Coordinated Universal Time+8)
- **Logic:** Query matches when status ACTIVE AND commission status AWAITING CONFIRMATION AND more than seven days have elapsed since match activation
- **On trigger (7-day threshold reached):**
  - Create and send notification N-12 (reminder email to tutee)
  - Create and send notification N-13 (admin alert email)
  - Flag match with confirmation overdue yes in admin panel
- **After 14 days without confirmation:** Escalate to admin-required override; system does NOT auto-cancel the match. Admin must manually choose: override confirmation or cancel match.
- **Idempotency:** Keyed on match id, trigger date sgt. The same match can only trigger reminder notifications once per calendar day (Singapore time). Duplicate triggers within the same calendar day are silently no-ops.
- **Missed-fire handling:** If the job was down, on startup it runs a backfill query for all matches where the 7-day threshold has been exceeded and confirmation overdue is not yet set.

#### 7.7.2 Post Auto-Expiry Cron Job

> **Source conflict O15 remains open in the [decision register](CURRENT-REQUIREMENTS.md#open-decisions):** this paragraph measures lifetime from submission, while the retained posting and approval module designs use an approval-based expiry timestamp. Restoring the prose does not decide between those clocks. An explicit decision and matching acceptance example are needed before claiming one consistent expiry rule.

- **Job name:** post auto expiry
- **Schedule:** Daily at 02:00 Singapore time (Coordinated Universal Time+8)
- **Logic:** Query posts when status ACTIVE AND the submission timestamp plus the configured expiry period is earlier than the current time; expiry days is platform-configurable (default 60)
- **Processing:** Batches of 100 posts to prevent table lock
- **On expiry:** Set post status EXPIRED; send notification N-18 to submitter; create post expired event in analytics event store
- **Idempotency:** Status transition is idempotent; re-running on an already-expired post is a no-op

### 7.8 Data Model

#### 7.8.1 matches Table

Key fields for the matches table:

| Field | Type | Notes |
|---|---|---|
| match id | unique identifier PK | |
| tutee post id | unique identifier FK → posts | |
| tutor post id | unique identifier FK → posts | |
| status | ENUM | PENDING / ACTIVE / CANCELLED / COMPLETED |
| agreed rate | INTEGER (SGD) | Admin-entered override of the tutor's stated rate; used for commission calculation when set |
| match activated at | TIMESTAMPTZ | Set when admin marks match ACTIVE |
| created by admin id | unique identifier FK → users | |
| created at | TIMESTAMPTZ | |
| updated at | TIMESTAMPTZ | |

**Index recommendation:**


The executable SQL index example is omitted from this documentation-only edition. Its purpose is to support lookup by tutor post and match status.


This index supports efficient querying of all active matches per tutor post.

#### 7.8.2 commissions Table

Commission data is stored in a separate commissions table, not as an enum column on the matches table. One commission record per match (enforced by unique constraint on match id for non-WAIVED records — see BR-005 / AC-14).

| Field | Type | Notes |
|---|---|---|
| commission id | unique identifier PK | |
| match id | unique identifier FK → matches | |
| tutor post id | unique identifier FK → posts | Denormalized for query efficiency |
| tutee post id | unique identifier FK → posts | Denormalized for query efficiency |
| agreed hourly rate sgd | INTEGER | Admin-entered; overrides post rate for commission calculation |
| agreed hours per session | DECIMAL using 3,1 | e g., 1.5 |
| agreed sessions per month | INTEGER | Admin enters this when frequency = "Flexible" (required before commission calculation) |
| commission rate | DECIMAL using 4,3 | Default 0.50; configurable per match by admin |
| calculated amount sgd | DECIMAL using 10,2 | Computed: rate × hours × sessions × commission rate |
| status | ENUM | PENDING / AWAITING CONFIRMATION / CONFIRMED / INVOICED / PAID / DISPUTED / WAIVED |
| invoice reference | VARCHAR | System-generated sequential invoice ID |
| invoice sent at | TIMESTAMPTZ | |
| payment received at | TIMESTAMPTZ | |
| notes | TEXT | Admin notes (required when status = DISPUTED or WAIVED) |
| created by admin id | unique identifier FK → users | |
| created at | TIMESTAMPTZ | |
| updated at | TIMESTAMPTZ | |

**Flexible frequency handling:** When a tutee post has frequency Flexible, the agreed sessions per month field on the commission record is required and must be entered by admin before the commission amount can be calculated. The match detail admin form includes a required "Agreed sessions per month" input field in this case.

#### 7.8.3 commission acknowledgements Table

Commission acknowledgement is stored at the user id level in a separate commission acknowledgements table (not on the post record).

| Field | Type | Notes |
|---|---|---|
| id | unique identifier PK | |
| user id | unique identifier FK → users | The tutor who acknowledged |
| acknowledged at | TIMESTAMPTZ | ISO 8601 |
| terms version | VARCHAR | Semantic version string, e g., "1.0.0" |
| ip address | INET | Stored encrypted; subject to anonymization after 90 days in analytics store only; immutable in audit log |
| affirmative action type | VARCHAR | e g., "checkbox checked" |

This table is write-once. No update records or DELETE is permitted via any application programming interface. It is part of the immutable audit store.

#### 7.8.4 Availability Grid Schema

Weekly availability is stored as a **21-bit integer bitmask**. Bits 0–20 map to the 21 availability slots (3 time periods × 7 days):

| Bit | Slot |
|---|---|
| 0 | Monday Morning |
| 1 | Monday Afternoon |
| 2 | Monday Evening |
| 3 | Tuesday Morning |
| 4 | Tuesday Afternoon |
| 5 | Tuesday Evening |
| 6 | Wednesday Morning |
| 7 | Wednesday Afternoon |
| 8 | Wednesday Evening |
| 9 | Thursday Morning |
| 10 | Thursday Afternoon |
| 11 | Thursday Evening |
| 12 | Friday Morning |
| 13 | Friday Afternoon |
| 14 | Friday Evening |
| 15 | Saturday Morning |
| 16 | Saturday Afternoon |
| 17 | Saturday Evening |
| 18 | Sunday Morning |
| 19 | Sunday Afternoon |
| 20 | Sunday Evening |

A bit value of 1 = available; 0 = not available. Example: a tutor available only Saturday and Sunday mornings would have availability bitmask 0b100001000000000 33024. Stored as INTEGER in the database.

#### 7.8.5 Analytics / Audit Separation

**The analytics event store and the audit log are SEPARATE data stores.**

- The 90-day anonymization job runs ONLY on the analytics event store.
- The immutable audit log (business requirements document §10.6.2) is exempt from anonymization and must never be targeted by the anonymization pipeline.
- The commission acknowledgements table is part of the immutable audit log; it must never be anonymized or modified.
- Analytics events may contain surrogate identifiers (post id, match id, user id). After 90 days, these fields in the analytics event store must be replaced with anonymized surrogate IDs that cannot be linked back to the operational database.

---

## 8. Commission Workflow

### 8.1 Commission Calculation

When admin marks a match ACTIVE, the system calculates the expected commission:


The illustration for commission calculation contains the following information:

- expected commission = agreed hourly rate × hours per session × sessions per month × commission rate.


- agreed hourly rate: taken from tutor's stated rate; admin may override via the agreed rate field on the match record or the agreed hourly rate sgd field on the commission record (commission record takes precedence)
- hours per session: taken from tutee's session duration field (1 / 1.5 / 2 / 2.5)
- sessions per month: taken from tutee's frequency field × 4 weeks (1×/week = 4, 2×/week = 8, 3×/week = 12); when frequency = "Flexible", admin must enter agreed sessions per month on the commission record before calculation proceeds
- commission rate: default 0.50; configurable per match by admin

The calculated commission is displayed in the match detail view and saved on the commissions table record.

**Example:** SGD 65/hr × 1.5hr × 8 sessions × 0.5 = **SGD 390 commission**

### 8.2 Commission Status Flow


1. **PENDING:** Creating a match creates its commission record in PENDING.
2. **AWAITING_CONFIRMATION:** Staff mark the match ACTIVE, advancing commission and sending the parent's confirmation email. The source diagram puts the seven-day window beside this email event; AC-05, §7.7.1/AC-11 and BR-010 disagree on whether the clock starts at send, activation or receipt. No start is selected here.
3. **CONFIRMED:** The parent confirms that the first session occurred. Staff may then generate the invoice.
4. **No response within the seven-day window:** Staff may override confirmation to CONFIRMED, contact the parent for confirmation, or cancel the match and explicitly waive commission. Silence alone does not confirm the session.
5. **INVOICED:** Send the invoice to the tutor with payment due in 14 days.
6. **PAID, DISPUTED or WAIVED:** Record PAID when the agency receives payment. A dispute moves commission to DISPUTED for staff resolution with notes. A staff waiver moves it to WAIVED with notes. A dispute decision without receipt cannot be recorded as PAID; unresolved financial adjustments remain O10.
7. **Cancellation after invoicing:** Staff must explicitly set commission to WAIVED or DISPUTED. Do not apply an automatic terminal billing state because the match was cancelled.


**Match status and commission status are separate fields with coupled transitions.** Agency receipt recorded as commission PAID moves the match to COMPLETED; first-session confirmation alone does not. Match status tracks the arrangement lifecycle (PENDING → ACTIVE → COMPLETED/CANCELLED). Commission status tracks the billing lifecycle on the commissions table.

### 8.3 Tutee First-Session Confirmation Flow (per BR-010)

1. Admin marks match ACTIVE in admin panel
2. System sets commission status → AWAITING CONFIRMATION on the commissions record
3. System sends tutee email:
   - Subject: "Please confirm your first tuition session"
   - Body: Explanation + action button "Confirm First Session Occurred"
   - Button links to: the first-session confirmation page for the user's match (authenticated, tutee-only)
4. Tutee clicks link → lands on confirmation page → clicks "Yes, my first session has taken place"
5. Commission status → CONFIRMED; confirmation timestamp logged
6. The confirmation endpoint is idempotent: if commission status is already CONFIRMED or later, subsequent calls return web request protocol 200 with current commission status and do not create duplicate records (see AC-13)
7. If no action within 7 calendar days:
   - Day 7: reminder email to tutee (N-12); system flags match for admin review (N-13)
   - Admin sees "Awaiting Tutee Confirmation — Overdue" in matches list
   - Admin chooses: override (→ CONFIRMED), contact tutee, or cancel match (→ CANCELLED)

### 8.4 Invoice Fields

When admin generates a commission invoice (after commission status reaches CONFIRMED), the invoice must contain:

| Field | Value |
|---|---|
| Invoice Number | System-generated sequential ID |
| Invoice Date | Date generated |
| Agency Name & Contact | Platform legal entity: the agency’s registered legal name (UEN the agency’s registration number), the agency’s registered business address; GST registration number (if applicable). Sourced from COMPANY in company implementation reference. |
| Tutor Display Name | Tutor's display name (not full legal name on invoice sent to tutor) |
| Tutee Reference | Tutee first name only (first word of display name, truncated to 10 characters). This identifier is deterministic and consistent across all invoice renders for the same tutee. Example: if display name is "Mei Lin Tan", tutee reference is "Mei" + subject + level (e g., "Mei — P6 Science"). |
| Agreed Hourly Rate | SGD X/hr |
| Sessions per Month | X sessions × Y hr/session = Z hr/month |
| Agreed Monthly Fee | SGD amount |
| Commission Rate | 50% |
| Commission Amount Due | SGD amount |
| Payment Due Date | Invoice date + 14 calendar days |
| Payment Instructions | Bank transfer details (or Pay Now UEN) |
| Dispute Contact | Agency email for disputes |

### 8.5 Commission Tracking (Admin)

- Admin commission ledger at /admin/commission shows all matches with:
  - Match ID, tutor display name, tutee reference, subject, level
  - Commission status, commission amount, invoice date, payment date
- Admin marks PAID manually after receiving off-platform payment
- DISPUTED status triggers internal note requirement (admin must document dispute)
- WAIVED status triggers internal note requirement (admin must document reason)

---

## 9. Notification Plan

All notifications are delivered via transactional email (system-generated). In-platform notifications (bell icon) are P1. All emails must comply with Personal Data Protection Act — no personal information of the other party is included.

| # | Trigger Event | Recipient | Channel | Subject Line | Content Summary |
|---|---|---|---|---|---|
| N-01 | User submits post (tutee or tutor) | Submitter | Email | "We've received your post" | "Your post is under review. We'll notify you within 2 business days." |
| N-02 | Admin approves tutee post | Tutee | Email | "Your tuition request is now live" | "Your post has been approved and is now active. Tutors can view your request (anonymised)." |
| N-03 | Admin approves tutor post | Tutor | Email | "Your tutor listing is now live" | "Your listing is approved and visible on the platform. You can now browse tutee requests." |
| N-04 | Admin requests revision (tutee) | Tutee | Email | "Revision needed for your post" | "Please update your post: [admin message]. Log in to edit and resubmit." |
| N-05 | Admin requests revision (tutor) | Tutor | Email | "Revision needed for your listing" | "Please update your listing: [admin message]. Log in to edit and resubmit." |
| N-06 | Admin rejects post (tutee) | Tutee | Email | "Update on your tuition request" | "Your post could not be approved at this time. Reason: [enum label]. You may submit a new post." |
| N-07 | Admin rejects post (tutor) | Tutor | Email | "Update on your tutor listing" | "Your listing could not be approved. Reason: [enum label]." |
| N-08 | Admin creates match (tutee) | Tutee | Email | "Exciting news — a match has been found!" | "We have identified a potential tutor match. Our coordinator will contact you within 1 business day." |
| N-09 | Admin creates match (tutor) | Tutor | Email | "A tutee match has been identified for you" | "We've matched a tutee request to your profile. Our coordinator will contact you within 1 business day." |
| N-10 | Admin marks match ACTIVE → tutee first-session confirmation | Tutee | Email | "Please confirm your first tuition session" | "Your arrangement is underway. Click below to confirm when your first session has taken place. [Confirm button]" |
| N-11 | Admin marks match ACTIVE → tutor informed | Tutor | Email | "Your tuition arrangement is active" | "Your arrangement is confirmed as active. Commission will be invoiced after the first session is confirmed." |
| N-12 | 7-day confirmation timeout — tutee reminder | Tutee | Email | "Reminder: Please confirm your first session" | "We haven't received your confirmation. Please confirm or contact us if the arrangement has not commenced." |
| N-13 | 7-day confirmation timeout — admin alert | Admin | Email (system) | "[ALERT] Tutee confirmation overdue: Match #id" | Admin receives summary; match flagged in admin panel. |
| N-14 | Tutee confirms first session | Admin | System event (no email) | — | Commission status advanced to CONFIRMED; match visible in commission ledger for invoice generation. |
| N-15 | Commission invoice generated | Tutor | Email | "Commission Invoice — Payment Due in 14 Days" | Invoice PDF or HTML attached; payment instructions; due date. |
| N-16 | Match cancelled | Tutee | Email | "Your tuition match has been cancelled" | "The match arrangement has been cancelled. [Reason if provided.] Your post is now active again and eligible for a new match." |
| N-17 | Match cancelled | Tutor | Email | "The tutee match has been cancelled" | "The match arrangement has been cancelled. [Reason if provided.]" |
| N-18 | Post auto-expired (no match within 60 days) | Submitter (tutee or tutor) | Email | "Your post has expired" | "Your post has expired after 60 days without a match. You may log in to create a new post." |
| N-19 | Post closed by user | Submitter | Email | "Your post has been closed" | "Your post has been closed and is no longer visible on the platform." |
| N-20 | Account suspended by admin | User | Email | "Important notice regarding your account" | "Your account has been suspended. Please contact [REDACTED EMAIL] for further information." (contact email sourced from COMPANY contact Email, company implementation reference) |
| N-21 | Tutee submits interview request | Admin | Email (system) | "[ACTION] New interview request for tutor post #tutor post id" | Tutee request summary, tutor post summary, tutee note (if any). Admin must schedule within service-level agreement. |
| N-22 | Admin schedules interview → tutee | Tutee | Email | "Your interview is confirmed — join details inside" | Date/time (Singapore time), 15-minute duration notice, meeting link URL, instructions ("Join on time — the meeting link is your only entry point"). |
| N-23 | Admin schedules interview → tutor | Tutor | Email | "Interview request scheduled — join details inside" | Date/time (Singapore time), 15-minute duration notice, meeting link URL, tutee request summary (subject/level/area/budget — no tutee contact info). |
| N-24 | Interview cancelled by admin | Tutee + Tutor | Email | "Interview session cancelled" | "The scheduled interview has been cancelled. [Reason if provided.] You may submit a new interview request." |
| N-25 | Interview completed — tutee follow-up prompt | Admin | Email (system) | "[ACTION] Follow up with tutee after interview: Match #tutor post id" | Tutee contact info included. Prompts admin to follow up off-platform. Sent when admin marks COMPLETED. |
| N-26 | 24-hour interview reminder (P1) | Tutee + Tutor | Email | "Reminder: Your interview is tomorrow" | Meeting link reminder, date/time, 15-minute duration reminder. |

**Delivery service-level agreement:** N-10, N-12, N-13 are commission-critical notifications and must be delivered within 5 minutes of their trigger event. N-22 and N-23 (interview schedule confirmations) must also be delivered within 5 minutes of admin scheduling action. N-01 and N-08 (non-critical) must deliver within 30 minutes. See §14 NFRs for full delivery service-level agreement specification.

---

## 10. Public Pages (Unauthenticated)

These pages are accessible to all visitors without login. No contact information of any user is ever rendered on these pages.

### 10.1 Landing Page (/)

**Purpose:** Convert visitors into registered tutees or tutors.

**Required content elements** (refer to the user experience Design Specification at specs/domain/uiux/ for wireframes, copy guidelines, and visual hierarchy; the product requirements document specifies WHAT must be communicated, not HOW it is visually presented):

1. **Hero Section** — Must communicate:
   - The platform's primary value proposition (admin-vetted, privacy-first, human matching)
   - Two distinct calls-to-action: one for tutees seeking a tutor, one for tutors seeking students

2. **How It Works — For Tutees** — Must communicate a 3-step process:
   1. Post a tuition request (level, subject, schedule, budget)
   2. Admin reviews and approves the request
   3. The agency matches the tutee with a vetted tutor and contacts them directly

3. **How It Works — For Tutors** — Must communicate a 3-step process:
   1. Register and list a tutoring profile
   2. Browse tutee requests once approved
   3. The agency makes the match and handles the introduction

4. **Trust Signals** — Must communicate:
   - Admin review of every listing
   - Contact details never shared without the user's knowledge
   - Human matching (no algorithms, no spam)

5. **Browse Tutors CTA** — Links to /tutors

6. **Footer** — Must include links to: About, FAQ, Contact, Privacy Policy, Terms & Conditions

---

### 10.2 Browse Tutor Listings (/tutors)

**Purpose:** Allow any visitor to browse approved, active tutor profiles.

**Features:**
- **Profile-based (M18):** lists any tutor with a complete profile (rate/area/commitment per §5.1a) —
  a published listing is **optional**, not required (free-market). Source = tutor profiles.
- Filters: Subject, Level, Venue Type, Location/Area
- Each card shows: subjects, levels, rate range, location area, **YOE, Part-time/Full-time**, rating,
  availability indicators — **NO real/display name** (M18 personal information; lead with subjects)
- **No contact information shown** (no email, phone, real name, or exact location per BR-001) —
  M18 profile-based browse **extends** AC-06 anonymisation; coarse area only
- **Profile photo URL MUST be excluded from all non-authenticated and non-admin-role application programming interface responses** for tutor post detail and listing endpoints (per BR-001 and §7.8.1)
- Individual tutor post detail page at the public tutor page for the selected listing — renders all public fields, CTA to post a tuition request
- Pagination (default 20 per page)
- No login required

---

### 10.3 About Page (/about)

- Agency story, mission, and values
- How the admin-mediated model differs from self-service platforms
- Trust and privacy commitment

### 10.4 FAQ Page (/faq)

**Required content** (copy and visual presentation per user experience Design Specification):
- Sections: For Tutees | For Tutors | Commission & Pricing | Privacy & Data
- Key questions that must be addressed:
  - Expected time to find a match (≤7 days median target)
  - Whether users can contact tutors/tutees directly (No — admin facilitates all introductions per BR-006)
  - When contact details are shared (only with the assigned participant under match-state gates; never in public or pre-assignment views)
  - What the commission is (50% of first month's fee, paid by tutor, one-time per match)
  - How to confirm the first session (via email link after admin marks arrangement active)

### 10.5 Contact Page (/contact)

- Agency contact form (name, email, message) — inquiry goes to admin inbox
- Admin email address and/or WhatsApp for direct enquiries
- Operating hours

### 10.6 Privacy Policy (/privacy-policy)

- Personal Data Protection Act-compliant; covers: data collected, purpose, access controls, retention, data subject rights, breach notification, contact for DPO

### 10.7 Terms & Conditions (/terms)

- Covers: platform rules, commission terms (for tutors), no-circumvention clause (12 months post-introduction), MOM work pass disclaimer, no guarantee of tutor qualifications

---

## 11. Authenticated Pages by Role

### 11.1 Tutee — Authenticated Pages

| Page | Description |
|---|---|
| **Dashboard** (/tutee/dashboard) | Overview: active posts, match status, pending first-session confirmation, recent notifications |
| **My Posts** (/tutee/posts) | List of all tutee posts with status badges |
| **Create Post** (/tutee/posts/new) | Tutee request post creation form (§5.2) |
| **Post Detail / Edit** (the owner's tuition-request page) | View post; edit if in Draft or Revision Requested state; close post action |
| **My Matches** (/tutee/matches) | List of all matches associated with tutee posts; status, match date |
| **Match Detail** (the user's match-detail page) | Match summary (subject, level, area); counterparty contact uses the separately authorized ACTIVE/COMPLETED reveal; commission status label; first-session confirmation button (if AWAITING CONFIRMATION) |
| **Confirm Session** (the first-session confirmation page for the user's match) | One-click first-session confirmation page (per BR-010) |
| **My Interviews** (/tutee/interviews) | List of all interview requests with status badges (REQUESTED / SCHEDULED / COMPLETED / CANCELLED / NO SHOW); meeting link shown for SCHEDULED |
| **Interview Detail** (the user's interview-detail page) | Interview summary: tutor post link, status, date/time, meeting link (if scheduled); cancel action for INT-039-defined non-terminal states |
| **Account Settings** (/tutee/account) | Update display name, password; view own email/phone (own data only) |
| **Notifications** (/tutee/notifications) | (P1) In-platform notification inbox |
| **Browse Tutors** (/tutors) | Accessible while logged in (same public page, no change); "Request Interview" button visible on authenticated the public tutor-detail page when logged in as TUTEE |

---

### 11.2 Tutor — Authenticated Pages

| Page | Description |
|---|---|
| **Dashboard** (/tutor/dashboard) | Overview: active listings, active matches, commission status summary, recent notifications |
| **My Listings** (/tutor/posts) | List of all tutor offering posts with status badges |
| **Create Listing** (/tutor/posts/new) | Tutor offering post creation form (§5.1); commission disclosure must be completed before first post |
| **Listing Detail / Edit** (the tutor's own listing page) | View listing; edit if Draft or Revision Requested; close listing action |
| **Browse Tutee Requests** (/tutor/requests) | Anonymised tutee request discovery index (Active posts only; no contact info); filterable by subject, level, venue, area, budget |
| **Tutee Request Detail** (the selected tuition-request detail page) | Full anonymised tutee request: subject, level, number of students, venue, schedule, budget, area — no name, phone, or email shown |
| **My Matches** (/tutor/matches) | List of matches; status; commission status label |
| **Match Detail** (the tutor's match-detail page) | Match summary (subject, level — no tutee contact info shown); commission amount and status (admin-set) |
| **Commission Terms** (/tutor/commission-terms) | View accepted commission terms; re-display acknowledgement log entry |
| **Account Settings** (/tutor/account) | Update display name, password, profile photo; view own email/phone |
| **My Interviews** (/tutor/interviews) | List of scheduled interview requests where this tutor's post is involved; date/time, meeting link; tutee request summary (no tutee contact info) |
| **Notifications** (/tutor/notifications) | (P1) In-platform notification inbox |

---

### 11.3 Admin — Authenticated Pages

| Page | Description |
|---|---|
| **Admin Login** (/admin/login) | Email + password + time-based one-time passcode multi-factor authentication (Google Authenticator or equivalent); FIDO2 hardware keys supported |
| **Admin Dashboard** (/admin/dashboard) | KPI overview: pending queue count, service-level agreement health, active matches, commission revenue, new registrations |
| **Post Review Queue** (/admin/posts/review) | All pending posts sorted by submission time; service-level agreement indicator (P1); bulk actions (P2) |
| **Post Review Detail** (the staff review page for the selected post) | Full post + contact info panel (access logged) + internal notes + action buttons (Approve / Reject / Request Revision) |
| **All Posts** (/admin/posts) | Full post list; filterable by status, type, subject, level |
| **Matching Dashboard** (/admin/matching) | Two-panel layout; tutee requests (left) + tutor offerings (right); shared filters; Create Match action |
| **All Matches** (/admin/matches) | Full match list; filterable by status, subject, level, date; match detail link |
| **Match Detail** (the staff match-detail page) | Full match record: linked tutee post, linked tutor post, match status, commission record (status, calculated commission, history), internal notes, action buttons |
| **Commission Ledger** (/admin/commission) | All commission records with status; filter by status, date, tutor; mark PAID action |
| **Invoice Generator** (the staff invoice page for the selected commission) | Generate and preview invoice PDF/HTML; send to tutor |
| **Users — Tutees** (/admin/users/tutees) | All tutee accounts; contact info accessible (access logged); status; post count |
| **Users — Tutors** (/admin/users/tutors) | All tutor accounts; contact info accessible (access logged); commission acknowledgement log; status; listing count |
| **User Detail** (the staff detail page for the selected user) | Full account record; actions: suspend, reactivate, ban, close all posts |
| **Interview Queue** (/admin/interviews) | All interview requests; REQUESTED tab (action required) + SCHEDULED tab + history (COMPLETED/CANCELLED/NO SHOW); sorted by submission time |
| **Interview Detail** (the staff interview-detail page) | Full interview record: tutee info (contact visible, access logged), tutor post details, status, meeting link, admin notes; action buttons: Schedule / Mark Completed / Mark Cancelled / Mark No-Show |
| **Audit Log** (/admin/audit) | Immutable contact field access log; commission acknowledgement log; read-only; DPO/senior admin access only |
| **Platform Settings** (/admin/settings) | Configurable: post auto-expiry days, rejection reason enum, commission rate defaults |

---

## 12. Full Routes Inventory & application programming interface Contracts

| # | Path | Page Name | Auth Required | Roles Allowed |
|---|---|---|---|---|
| R-01 | / | Landing Page | No | All |
| R-02 | /tutors | Browse Tutor Listings | No | All |
| R-03 | the public tutor page for the selected listing | Tutor Post Detail | No | All |
| R-04 | /about | About | No | All |
| R-05 | /faq | FAQ | No | All |
| R-06 | /contact | Contact | No | All |
| R-07 | /privacy-policy | Privacy Policy | No | All |
| R-08 | /terms | Terms & Conditions | No | All |
| R-09 | /register/tutee | Tutee Registration | No (redirects if logged in) | UNAUTHENTICATED |
| R-10 | /register/tutor | Tutor Registration — Step 1: Account | No (redirects if logged in) | UNAUTHENTICATED |
| R-11 | /register/tutor/commission | Tutor Registration — Step 2: Commission Disclosure | Yes (partial — completing registration) | New TUTOR (pre-approval) |
| R-12 | /register/tutor/profile | Tutor Registration — Step 3: Create Listing | Yes | New TUTOR |
| R-13 | /verify-email | Email Verification | No | All |
| R-14 | /login | Login | No (redirects if logged in) | All |
| R-15 | /forgot-password | Forgot Password | No | All |
| R-16 | /reset-password | Reset Password | No (token-based) | All |
| R-17 | /tutee/dashboard | Tutee Dashboard | Yes | TUTEE |
| R-18 | /tutee/posts | My Tuition Requests | Yes | TUTEE |
| R-19 | /tutee/posts/new | Create Tuition Request | Yes | TUTEE |
| R-20 | the owner's tuition-request page | Tuition Request Detail / Edit | Yes | TUTEE (own), ADMIN |
| R-21 | /tutee/matches | My Matches (Tutee) | Yes | TUTEE |
| R-22 | the user's match-detail page | Match Detail (Tutee View) | Yes | TUTEE (own), ADMIN |
| R-23 | the first-session confirmation page for the user's match | First Session Confirmation | Yes | TUTEE (own match only) |
| R-24 | /tutee/account | Tutee Account Settings | Yes | TUTEE |
| R-25 | /tutee/notifications | Tutee Notifications (P1) | Yes | TUTEE |
| R-26 | /tutor/dashboard | Tutor Dashboard | Yes | TUTOR |
| R-27 | /tutor/posts | My Tutor Listings | Yes | TUTOR |
| R-28 | /tutor/posts/new | Create Tutor Listing | Yes | TUTOR |
| R-29 | the tutor's own listing page | Tutor Listing Detail / Edit | Yes | TUTOR (own), ADMIN |
| R-30 | /tutor/requests | Browse Tutee Requests | Yes | TUTOR (approved only) |
| R-31 | the selected tuition-request detail page | Tutee Request Detail (Anonymised) | Yes | TUTOR (approved only) |
| R-32 | /tutor/matches | My Matches (Tutor) | Yes | TUTOR |
| R-33 | the tutor's match-detail page | Match Detail (Tutor View) | Yes | TUTOR (own), ADMIN |
| R-34 | /tutor/commission-terms | Commission Terms View | Yes | TUTOR |
| R-35 | /tutor/account | Tutor Account Settings | Yes | TUTOR |
| R-36 | /tutor/notifications | Tutor Notifications (P1) | Yes | TUTOR |
| R-37 | /admin/login | Admin Login (multi-factor authentication) | No (redirects if logged in) | ADMIN |
| R-38 | /admin/dashboard | Admin Dashboard | Yes | ADMIN |
| R-39 | /admin/posts/review | Post Review Queue | Yes | ADMIN |
| R-40 | the staff review page for the selected post | Post Review Detail | Yes | ADMIN |
| R-41 | /admin/posts | All Posts | Yes | ADMIN |
| R-42 | /admin/matching | Matching Dashboard | Yes | ADMIN |
| R-43 | /admin/matches | All Matches | Yes | ADMIN |
| R-44 | the staff match-detail page | Match Detail (Admin View) | Yes | ADMIN |
| R-45 | /admin/commission | Commission Ledger | Yes | ADMIN |
| R-46 | the staff invoice page for the selected commission | Invoice Generator | Yes | ADMIN |
| R-47 | /admin/users/tutees | Tutee User List | Yes | ADMIN |
| R-48 | /admin/users/tutors | Tutor User List | Yes | ADMIN |
| R-49 | the staff detail page for the selected user | User Detail | Yes | ADMIN |
| R-50 | /admin/audit | Audit Log | Yes | ADMIN (DPO / senior admin) |
| R-51 | /admin/settings | Platform Settings | Yes | ADMIN |
| R-52 | /tutee/interviews | My Interview Requests (Tutee) | Yes | TUTEE |
| R-53 | the user's interview-detail page | Interview Detail (Tutee View) | Yes | TUTEE (own) |
| R-54 | /tutor/interviews | My Scheduled Interviews (Tutor) | Yes | TUTOR |
| R-55 | /admin/interviews | Interview Queue | Yes | ADMIN |
| R-56 | the staff interview-detail page | Interview Detail (Admin View) | Yes | ADMIN |

### 12.1 application programming interface Routes

All application programming interface endpoints must enforce server-side role-based access control (per Appendix D §D.3). Contact fields must be excluded from all responses to non-admin roles. application programming interface paths follow the  interface operation prefix convention.

**Profile photo URL MUST be excluded from all non-authenticated and non-admin-role application programming interface responses** for tutor post detail and listing endpoints. Authenticated tutor endpoints return the photo URL only for the tutor's own profile. Admin endpoints return the photo URL as a signed URL per §14 NFRs.

#### 12.1.1 Match Lifecycle Endpoints

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | tutee matches id confirm-session interface operation | Tutee confirms first session. Idempotent: subsequent calls when commission status is already CONFIRMED or later return web request protocol 200 with current commission status. Returns web request protocol 409 if match is not in AWAITING CONFIRMATION state (and not already confirmed). | TUTEE (own match) |
| POST | matches id timeout-flag interface operation | System-internal cron trigger for 7-day confirmation timeout. Creates N-12 and N-13 notifications. Idempotent per match id, trigger date sgt. | SERVICE ACCOUNT only |
| POST | admin matches id override-confirmation interface operation | Admin override of tutee first-session confirmation. Advances commission status to CONFIRMED. Logged with admin user id and timestamp. | ADMIN role |

#### 12.1.2 Listing Endpoints — Pagination Spec

All listing endpoints (tutor posts, tutee posts, matches, commission records) support cursor-based pagination:

| Query Parameter | Type | Default | Max | Description |
|---|---|---|---|---|
| page size | integer | 20 | 100 | Number of records per page |
| cursor | string | (none — first page) | — | Composite cursor encoding created at + id; opaque to client |
| subject | string | — | — | Filter by subject |
| level | string | — | — | Filter by MOE level |
| venue type | enum | — | — | Filter by venue type |
| area | string | — | — | Filter by planning area or MRT station |
| budget min | integer (SGD) | — | — | Filter by minimum budget (tutee posts) / rate (tutor posts) |
| budget max | integer (SGD) | — | — | Filter by maximum budget / rate |

Responses include a next cursor field (null if no further pages). Cursor encodes created at (ISO 8601) and id (unique identifier) as a base64-encoded JSON string.

#### 12.1.3 application programming interface Versioning Policy

- All endpoints are prefixed  interface operation
- Non-breaking additions (new optional response fields, new optional query parameters) are backwards compatible and do not require a version bump
- Breaking changes (removal of fields, changed field types, changed semantics) require a new prefix  interface operation and a minimum 90-day deprecation notice with continued support of  interface operation during the deprecation window
- Deprecation notices are communicated via application programming interface response headers (Deprecation, Sunset) and documented in the application programming interface changelog

---

## 13. Acceptance Criteria (P0 Features)

### AC-01: Public privacy and assigned-participant contact access (amended BR-001)

**Feature:** Contact field access control


1. **GIVEN** a registered parent or tutor has stored email and phone details, **WHEN** a public, discovery, applicant or other pre-assignment interface returns their profile or post to another user, **THEN** it must not expose private email, phone or full name.
2. **GIVEN** the requester is an authorized participant of the assigned match, **WHEN** they request the other person's contact, **THEN** the interface enforces ownership and an eligible match state of PENDING, ACTIVE or COMPLETED. The website reveals phone/email only for ACTIVE or COMPLETED. Post approval, invitation acceptance and selection alone never release contact.
3. **GIVEN** an administrator opens the contact section of a post, **WHEN** the contact details are accessed, **THEN** an immutable audit entry records the admin identity, Singapore-time timestamp, record type, record identity and fields accessed.
4. **GIVEN** an unauthenticated visitor browses tutor listings, **WHEN** the listing is rendered, **THEN** no tutor email, phone, home address or profile photo URL is present in either the page content or the underlying response.


---

### AC-02: Post Approval Gate (BR-002)

**Current scope:** Moderation still gates publication. C01 replaces the older approved-tutors-only browsing assumption with bounded public access; C05 permits eligible tutors to apply or be assigned without a published tutor listing. The original cases below do not override those changes.

**Feature:** Post visibility gated on admin approval


The illustration for ac-02: post approval gate br-002 contains the following information:

- GIVEN a tutor submits a new tutor offering post.
- WHEN the post status is Pending Review.
- THEN the post does NOT appear on the /tutors public listing page or in the /tutor/requests tutor discovery index.
- GIVEN admin approves a tutee request post.
- WHEN the approval action is saved.
- THEN the post appears in the /tutor/requests discovery index for logged-in approved tutors, AND the tutee receives an approval notification email within the system's email delivery service-level agreement.
- GIVEN admin selects "Reject" with reason "Incomplete information" on a tutor post.
- WHEN the rejection is confirmed.
- THEN the tutor receives an email containing the rejection reason, AND the post status is set to Rejected (terminal state), AND the post does not appear in any listing.


---

### AC-03: Commission Disclosure at Tutor Registration (BR-009)

**Feature:** Standalone commission disclosure step


**Current sequence takes priority:** [Tutor registration sequence 02](sequences/02-register-tutor.md) requires legal and commission acknowledgements before final account creation and saves them with the account audit atomically. Profile capture is optional and skippable. The first historical case below documents the earlier order, not permission to create an account before acknowledgement. The final case retains the guard for a missing acknowledgement; it does not authorize such incomplete signup.

1. **Historical order, superseded:** **GIVEN** a user selects “Register as Tutor”, **WHEN** they finish account creation and email verification, **THEN** the original case presents mandatory standalone commission disclosure as step two and blocks progress to profile creation. Replace this order with the current sequence above; keep disclosure mandatory.
2. **GIVEN** the tutor is on the disclosure screen, **WHEN** they continue without checking acknowledgement, **THEN** prevent progression and show “You must accept the commission terms to continue”.
3. **GIVEN** the tutor checks acknowledgement and continues, **WHEN** the form is submitted, **THEN** record tutor identity, an ISO 8601 acknowledgement timestamp, commission terms version and the affirmative checkbox action in an immutable audit record.
4. **GIVEN** an existing tutor account lacks commission acknowledgement, **WHEN** staff view its post in the review queue, **THEN** show “Commission acknowledgement not yet completed” and disable approval.


---

### AC-04: Admin-Only Matching (BR-003)

**Feature:** Match creation is admin-exclusive


**Current listing rule:** C05 permits application and assignment without a published tutor listing. The original “tutor post B” case below retains its historical wording for traceability; an eligible tutor profile is sufficient under the current rule.

1. **GIVEN** a logged-in tutor opens a parent request from discovery, **WHEN** its detail is shown, **THEN** show neither a “Contact Tutee” nor a “Request Match” action and expose no private contact. An “Apply / Express interest” action is available under M18, but it creates only an application and never a match.
2. **GIVEN** staff create a match between parent request A and tutor post B in the original listing-based flow, **WHEN** that match is confirmed, **THEN** mark request A Matched, remove it from discovery and email both people. Apply the current listing-optional rule above.
3. **GIVEN** request A already has a PENDING match, **WHEN** staff attempt another match for it, **THEN** block creation and show “This tutee request already has an active match”.
4. **GIVEN** a parent chooses a tutor, **WHEN** that selection is recorded, **THEN** it is only a request for staff action. No match exists until staff create it, and §7.4 cardinality still applies.


> **M18 note:** the free-market change lets a tutor *apply* (/tutor/applications), and a parent
> *choose/request* — but **match creation stays admin-exclusive**. AC-04 is unchanged; §20 layers a
> parent→admin request step *before* the admin match action.

---

### AC-05: First-Session Confirmation Before Commission Invoice (BR-010)

**Feature:** Tutee first-session confirmation gate


**Unresolved start of the seven-day window:** Case 3 below counts from the confirmation email being **sent**. Section 7.7.1 and AC-11 count from match **activation**. The original [BR-010](BRD.md#br-010-first-session-confirmation-before-commission-invoice) counts from the parent **receiving** the notification. These are three different events; no approved rule has reconciled them. Preserve the confirmation, reminder and admin follow-up obligations, but do not claim one start is settled. This is separate from O12, which asks whether the five-minute delivery target starts at the seven-day threshold or when the six-hour detector notices it.

1. **GIVEN** staff mark a match ACTIVE, **WHEN** the change is saved, **THEN** commission becomes AWAITING_CONFIRMATION, a “Confirm First Session” email is sent to the parent within the five-minute delivery target in §14, and the admin ledger shows “Awaiting Tutee Confirmation”.
2. **GIVEN** the parent follows “Confirm First Session Occurred” from the email, **WHEN** they submit confirmation on the authenticated confirmation page for their match, **THEN** commission becomes CONFIRMED, the confirmation timestamp is recorded, and the ledger shows “Commission Due: Ready to Invoice”.
3. **Historical email-send start, unresolved:** **GIVEN** seven calendar days have passed since the confirmation email was sent without parent action, **WHEN** the timeout is reached, **THEN** send a parent reminder and admin alert, and show “Overdue Confirmation” in the admin panel. The start remains subject to the conflict above.
4. **GIVEN** staff choose “Override Confirmation” on an overdue match, **WHEN** they confirm the override, **THEN** commission becomes CONFIRMED and the audit records the admin identity and timestamp.
5. **GIVEN** a match is cancelled before commission reaches CONFIRMED, **WHEN** cancellation is saved, **THEN** commission becomes WAIVED, no invoice is generated, and both people receive cancellation email.


---

### AC-06: Tutor Discovery Index — Anonymisation (BR-001, BR-003)

**Current scope:** The privacy and matched-request removal requirements remain. C01 and C05 replace the older assumption that all discovery access requires an approved tutor listing.

**Feature:** Tutees browseable to tutors without contact info


The illustration for ac-06: tutor discovery index — anonymisation br-001, br-003 contains the following information:

- GIVEN an approved tutor is logged in and navigates to /tutor/requests.
- WHEN the discovery index page loads.
- THEN all tutee posts are rendered with: subject, level, number of students, venue type, schedule, budget range, location area — AND the tutee's full name, email address, and phone number are absent from both the rendered HTML and the underlying application programming interface response.
- GIVEN a tutee post's status changes to Matched.
- WHEN an approved tutor views /tutor/requests.
- THEN the matched tutee post is no longer visible in the listing (removed from discovery index).


---

### AC-07: Admin multi-factor authentication Enrollment Enforcement

**Environment scope:** Read these original admin security cases with C09 and specification 027. Admin authentication rules are defined separately for each environment; the original universal wording does not erase that distinction.

**Feature:** Admin multi-factor authentication requirement


The illustration for ac-07: admin multi-factor authentication enrollment enforcement contains the following information:

- GIVEN an admin user has not enrolled multi-factor authentication (time-based one-time passcode).
- WHEN they attempt to access any admin route (any path under /admin/ except /admin/login and /admin/mfa/enroll).
- THEN they must be redirected to the multi-factor authentication enrollment flow at /admin/mfa/enroll before access is granted.
- GIVEN an admin session attempts to bypass multi-factor authentication at the application layer.
- WHEN the bypass is attempted via any mechanism.
- THEN the server returns web request protocol 403 Forbidden and the action is logged in the audit log.


---

### AC-08: Audit Log Immutability on Contact Field Access

**Feature:** Immutable audit log for admin contact field access


1. **GIVEN** staff view a phone or email field on any admin page, **WHEN** the page renders and includes that field in its response, **THEN** create an audit entry with admin identity, Singapore-time timestamp in ISO 8601 format, resource type such as tutor post or user, resource identity, and the particular phone and/or email fields accessed.
2. **GIVEN** an audit entry exists, **WHEN** any application interface, including an admin interface, attempts to modify or delete it, **THEN** return an error and leave the entry unchanged. No application interface permits either action.


---

### AC-09: Admin Session Idle Timeout

**Environment scope:** These original admin session cases remain subject to C09 and the separately specified admin environment rules. They do not define the parent or tutor session contract.

**Feature:** Admin session controls


The illustration for ac-09: admin session idle timeout contains the following information:

- GIVEN an admin session has been idle for 30 minutes (no authenticated request made).
- WHEN any authenticated request is subsequently made using the idle session token.
- THEN the server returns web request protocol 401 Unauthorized and the session is invalidated.
- GIVEN an web request protocol 401 is returned due to idle session expiry.
- WHEN the admin attempts to access any admin route.
- THEN they must re-authenticate from /admin/login, including completing the time-based one-time passcode multi-factor authentication step.


---

### AC-10: Terms Version Schema for Commission Acknowledgement

**Feature:** Commission acknowledgement terms versioning


1. **GIVEN** a tutor completes commission acknowledgement, **WHEN** the record is saved, **THEN** include the user identity, ISO 8601 acknowledgement timestamp, semantic terms version such as 1.0.0, IP address and the affirmative checkbox action.
2. **GIVEN** an acknowledgement record is being created, **WHEN** its terms version is set, **THEN** it must equal the active version in platform configuration. A mismatch raises an error and prevents creation.


---

### AC-11: Reminder Email Timing (N-12)

**Separate unresolved start, O13:** This case starts the seven days at activation, while AC-05 starts them at email send and original BR-010 at notification receipt. See [AC-05](#ac-05-first-session-confirmation-before-commission-invoice-br-010). O12 below concerns the later detection and delivery window, not which of these three events starts the seven days.

**Feature:** 7-day first-session confirmation reminder timing

**O12 pending:** This historical six-hour acceptance window and §14.2’s five-minute trigger target need one explicit clock definition; this scenario is not independently sufficient for sign-off.


The illustration for ac-11: reminder email timing n-12 contains the following information:

- GIVEN a tutee has not confirmed their first session.
- WHEN 7 calendar days have elapsed since match activated at (measured in Singapore time).
- THEN a reminder email (N-12) is sent within 6 hours of the 7-day mark, consistent with the cron schedule running every 6 hours (per §7.7.1).


---

### AC-12: Tutee Pseudonym Format on Invoice

**Feature:** Consistent tutee identifier on commission invoices


The illustration for ac-12: tutee pseudonym format on invoice contains the following information:

- GIVEN a match is confirmed and a commission invoice is generated.
- WHEN the invoice displays the tutee reference.
- THEN the tutee is identified as the first word of their display name, truncated to a maximum of 10 characters.
- GIVEN the same tutee is referenced on multiple invoices across different matches.
- WHEN the invoices are rendered.
- THEN the tutee pseudonym is identical across all renders (deterministic based on display name).


---

### AC-13: First-Session Confirmation Endpoint Idempotency

**Feature:** Idempotent tutee session confirmation


1. **GIVEN** the parent has already confirmed the first session and commission is CONFIRMED or later, **WHEN** they submit confirmation again for the same match, **THEN** return HTTP 200 with the current commission status.
2. **GIVEN** that repeat confirmation is processed, **WHEN** the request is handled, **THEN** create no duplicate commission record and emit no duplicate events.


---

### AC-14: One Commission Per Match — BR-005

**Feature:** Commission uniqueness per match


The illustration for ac-14: one commission per match — br-005 contains the following information:

- GIVEN a confirmed match already has a commission record in any status other than WAIVED.
- WHEN an admin attempts to create a second commission record for the same match id.
- THEN the system returns an error (web request protocol 409 Conflict) and prevents creation of the duplicate commission record.


---

### AC-15: No Direct Messaging — BR-006

**Current disclosure rule:** The ban on a direct messaging feature remains. The historical phrase “contact-exchange endpoint” below must not prohibit C04's authorized assigned-participant contact interface in AC-01; pre-assignment contact remains hidden.

**Feature:** No messaging or contact exchange between tutees and tutors


The illustration for ac-15: no direct messaging — br-006 contains the following information:

- GIVEN any authenticated user (TUTEE or TUTOR role).
- WHEN they call any messaging, contact-exchange, or direct-communication endpoint.
- THEN the application programming interface returns web request protocol 404 Not Found.
- No messaging endpoint exists in the production application programming interface surface. This is not a permissions error (403) — the endpoint must not exist.


---

### AC-16: Interview Request — Eligibility Guards (BR-011)

**Feature:** Interview request eligibility enforcement


**Historical eligibility cases:** C06 and [specification 021](021-tutee-request-flow/spec.md) also allow initiation from the parent's own pending post. C07 replaces the old per-account cap and REQUESTED/SCHEDULED-only counting with per-post limits and the current negotiation states. Retaining the old cases below does not reinstate those superseded restrictions.

1. **GIVEN** a logged-in parent views an approved tutor listing, **WHEN** that listing is ACTIVE, **THEN** show “Request Interview” on its detail page. This listing-based entry point is not the only permitted current entry point.
2. **Historical account cap, superseded:** **GIVEN** a parent already has three open interviews counted as REQUESTED or SCHEDULED, **WHEN** they submit another request for any tutor post, **THEN** the original case returns HTTP 422 and asks them to wait until one completes. Apply C07's per-post limit and current open states instead of this account-wide predicate.
3. **GIVEN** an interview with a tutor post ended COMPLETED or NO_SHOW, **WHEN** fewer than 30 calendar days have passed since that interview's completion, **THEN** a new request for the same tutor post returns HTTP 422 and asks the parent to wait 30 days before requesting another interview.
4. **GIVEN** a visitor is not logged in, **WHEN** they view a tutor listing detail page, **THEN** replace “Request Interview” with “Log in to Request Interview”, subject to the current authentication rule for viewing the detail itself.


---

### AC-17: Interview Scheduling — Meeting Link Required (BR-013, BR-014)

**Feature:** Admin interview scheduling with mandatory meeting link


The illustration for ac-17: interview scheduling — meeting link required br-013, br-014 contains the following information:

- GIVEN admin opens an interview in REQUESTED status and clicks "Schedule Interview".
- WHEN the scheduling form is submitted without a meeting link.
- THEN the system returns a validation error: "A meeting link is required to schedule an interview".
- GIVEN admin enters a valid date/time and a meeting link URL and submits the schedule form.
- WHEN the schedule action is saved.
- THEN the interview status transitions to SCHEDULED, notification N-22 is sent to the tutee (within 5 minutes), notification N-23 is sent to the tutor (within 5 minutes), and the meeting link is stored on the interview record.
- GIVEN an interview is in SCHEDULED status.
- WHEN admin updates the meeting link before the scheduled time.
- THEN the interview record is updated with the new link AND notifications are resent to both tutee and tutor with the updated link.


---

### AC-18: Interview — No Contact Info Shared (BR-015)

**Feature:** Contact privacy preserved throughout interview workflow


**Disclosure boundary:** These interview cases retain privacy before assignment. Under C04, holding or scheduling an interview never releases contact. Authorized contact access after assignment and activation follows the separate participant rule in AC-01.

1. **GIVEN** an interview is SCHEDULED, **WHEN** the parent views its detail page, **THEN** show the tutor post title, subjects, levels, rate range, area, date/time and meeting link, but no tutor full name, phone or email.
2. **GIVEN** an interview is SCHEDULED, **WHEN** N-23 is sent to the tutor, **THEN** include the parent request's subject, level, area and budget, but no parent full name, phone or email.
3. **GIVEN** staff view interview details, **WHEN** parent phone or email is displayed in the admin panel, **THEN** create an immutable audit entry meeting AC-08.


---

### AC-19: Interview Outcome Tracking (BR-013)

**Feature:** Admin marks interview outcomes


The illustration for ac-19: interview outcome tracking br-013 contains the following information:

- GIVEN an interview is in SCHEDULED status and the scheduled time has passed.
- WHEN admin marks the interview as COMPLETED.
- THEN the interview status transitions to COMPLETED and notification N-25 (admin follow-up prompt) is sent to admin.
- GIVEN an interview is in REQUESTED or SCHEDULED status.
- WHEN admin cancels the interview.
- THEN the interview status transitions to CANCELLED, notification N-24 is sent to both tutee and tutor.
- GIVEN an interview was scheduled but one or both parties did not join.
- WHEN admin marks NO SHOW.
- THEN the interview status transitions to NO SHOW and the no show actor field is set (TUTEE / TUTOR / BOTH).
- GIVEN an interview has reached status COMPLETED, CANCELLED, or NO SHOW (terminal states).
- WHEN any attempt is made to change the interview status.
- THEN the system returns web request protocol 422: "Interview is in a terminal state and cannot be modified".


---

## 14. Non-Functional Requirements

### 14.1 Availability

| Component | Target |
|---|---|
| Public web application | 99.5% monthly uptime |
| Admin panel | 99.9% during Singapore time business hours (08:00–22:00 Singapore time, seven days per week; retained business requirements document §10.7.4 window) |

### 14.2 Notification Delivery service-level agreement

| Notification Type | Events | Delivery service-level agreement |
|---|---|---|
| Commission-critical (transactional) | N-10 (first-session confirmation email), N-12 (tutee reminder), N-13 (admin alert) | Within 5 minutes of trigger event (N-12/N-13 clock reconciliation pending O12) |
| Interview scheduling | N-22 and N-23 (both-party schedule confirmations) | Within 5 minutes of scheduling, per §9 |
| Non-critical (informational) | N-01 (registration confirmation), N-08 (match notification) and other events without an explicit shorter service-level agreement | Within 30 minutes of trigger event |

### 14.3 Email Provider Requirements

- Must support Singapore data residency OR have a signed Data Processing Agreement (DPA) covering Personal Data Protection Act cross-border transfer requirements
- Recommended providers: Amazon Web Services SES (ap-southeast-1 region), Postmark (with executed DPA)
- Provider must support transactional email delivery tracking (bounce, delivery, open events)

### 14.4 Object Storage (Profile Photos)

| Requirement | Specification |
|---|---|
| Region | Singapore-region S3-compatible storage (Amazon Web Services ap-southeast-1 or equivalent) |
| Read access | Signed URLs with 1-hour expiry (no permanent public URLs) |
| Encryption | Server-side encryption (server-sent events-S3 or server-sent events-key management service) |
| Deletion | Deletion pipeline triggered on account deletion; photo must be deleted within 24 hours of account deletion |
| Max file size | 5 MB |
| Accepted formats | JPEG, PNG, Web P |

Profile photo signed URLs must not appear in any non-authenticated or non-admin application programming interface response (see §10.2 and §12.1).

### 14.5 Analytics Event Sink

- Must be Singapore-region compliant
- Acceptable options: self-hosted Post Hog on Singapore infrastructure, or a cloud analytics provider with a Singapore region and a signed DPA covering Personal Data Protection Act requirements
- Subject to 90-day anonymization job per business requirements document Appendix C and §7.8.5 of this document
- The analytics event sink is a separate data store from the operational database and the immutable audit log

### 14.6 Response Latency Targets

| Endpoint Type | P50 Target | P99 Target |
|---|---|---|
| All user-facing application programming interface endpoints | < 300ms | < 1,500ms |
| Admin matching dashboard (two-panel filter) | < 500ms | < 2,000ms |

Latency measured at the server (excluding client network time).

### 14.7 Rate Limiting

| Endpoint / Context | Limit |
|---|---|
| Registration endpoints (/register/*) | 10 registrations per IP per hour |
| Public tutor listing (/tutors, tutors * interface operation) | 200 requests per IP per minute |
| Admin endpoints (/admin/*, admin * interface operation) | 500 requests per admin session per minute |

Rate limit violations return web request protocol 429 Too Many Requests with a Retry-After header.

---

## 15. MVP vs Phase 2 Scope

### 15.1 MVP Scope (Launch-Critical — P0)

The MVP delivers a fully functional admin-mediated marketplace with:

| Capability | Status |
|---|---|
| Tutee and tutor registration with email verification | MVP |
| Personal Data Protection Act-compliant consent and privacy policy | MVP |
| Commission disclosure standalone step at tutor registration | MVP |
| Post creation (tutee and tutor), draft, submit, resubmit | MVP |
| Admin post review queue (approve / reject / request revision) | MVP |
| Public tutor listings with bounded anonymous browse/filter; authenticated detail and separate action gates | MVP |
| Bounded anonymous browse/filter of approved, anonymized requests; authenticated detail/action gates | MVP |
| Admin matching dashboard (two-panel, filter, create match) | MVP |
| Match cardinality enforcement (tutee: 1 active match; tutor: many) | MVP |
| Match lifecycle management (PENDING → ACTIVE → COMPLETED/CANCELLED) | MVP |
| First-session confirmation flow and 7-day timeout | MVP |
| Admin override of first-session confirmation | MVP |
| Commission calculation, invoice generation, PAID tracking | MVP |
| Email notification suite, including retained first-session and interview schedule notifications | MVP |
| User email/phone plaintext storage per founder amendment; restricted access, protected storage/backups and no personal information in logs | MVP |
| Immutable audit log for contact field access | MVP |
| Commission acknowledgement log | MVP |
| Admin multi-factor authentication (time-based one-time passcode required; FIDO2 supported) | MVP |
| Role-based access control, server-side enforcement | MVP |
| Mobile-first tutee/tutor user interface (390px primary viewport) | MVP |
| Data residency in Singapore region | MVP |
| Auto-expiry of unmatched posts (default 60 days) | MVP |
| Admin user management (suspend, ban, close posts) | MVP |
| Account deletion / right to erasure (admin-manual Day 1 workaround acceptable; must be resolved within 30 days post-launch) | MVP |
| DPO sign-off on data governance (go-live gate) | MVP |
| Interview request by tutee from tutor post detail page | MVP |
| Interview concurrency and cooldown guards (BR-011) | MVP |
| Admin interview queue and scheduling (date/time + meeting link) | MVP |
| Interview notifications N-21 through N-25 | MVP |
| Interview outcome tracking (COMPLETED / CANCELLED / NO SHOW) | MVP |
| No contact info shared during interview workflow (BR-015) | MVP |
| Interview analytics events | MVP |

### 15.2 Explicitly Out of MVP Scope

| Feature | Phase | Notes (per §11 business requirements document) |
|---|---|---|
| Automated / algorithmic matching | Phase 2 | Admin-assist suggestions only; does not replace human decision |
| In-platform payment (Stripe, Pay Now integration) | Phase 2 | MAS compliance complexity; off-platform invoice sufficient at MVP scale |
| Mobile app (iOS / Android) | Phase 3 | Web-responsive is sufficient |
| Multilingual user interface (Simplified Chinese, Malay, Tamil) | Phase 2 | English-first for MVP |
| In-platform tutee-tutor messaging or chat | Never (indefinitely) | Business rule BR-006; would change the model fundamentally |
| Tutor review / rating system | Phase 2 | Requires match volume; gaming risk without moderation |
| Subscription / membership tiers | Phase 3 | Single commission model for MVP |
| Automated Zoom/Google Meet room creation via application programming interface | Phase 2 | Admin provides meeting links manually in MVP; automated provisioning deferred |
| Credential verification via My Info / Singpass | Phase 2 | Manual admin review is sufficient at MVP scale |
| Analytics dashboard for tutees / tutors | Phase 2 | Admin-only analytics in MVP |
| In-platform payment dispute resolution | Phase 2 | Manual dispute handling via admin in MVP |
| GST invoice automation | Phase 2 (trigger: revenue > SGD 1M) | Monitor revenue; register and update invoices when threshold reached |
| In-platform notification inbox (bell icon) | P1 (post-launch) | Email-only in MVP |
| Personal Data Protection Act data export on user request | P1 (post-launch) | Admin-assisted export in MVP |
| Background check integration | Phase 2 | Manual credential review by admin for MVP |
| service-level agreement monitoring dashboard (amber/red queue indicators) | P1 (post-launch) | Queue time data captured; dashboard user interface deferred |
| Commission DISPUTED/WAIVED workflow user interface | P1 (post-launch) | Status transitions remain in data model; admin handles via notes in MVP |

---

## 16. Analytics Events

All events are captured server-side. The analytics event store and the immutable audit log are SEPARATE data stores (see §7.8.5). The 90-day anonymization job runs ONLY on the analytics event store. The immutable audit log (business requirements document §10.6.2) is exempt from anonymization and must never be targeted by the anonymization pipeline.

**Anonymization policy for analytics events older than 90 days:**
- user id replaced with anonymized user hash
- post id replaced with anonymized post hash
- match id replaced with anonymized match hash
- admin user id replaced with anonymized admin hash

These hashes are NOT reverse-linkable to operational database records. After anonymization, the original identifiers are destroyed in the analytics store.

Events must not capture personal information fields (email, phone, full name) at any time.

### 16.1 Registration & Onboarding Events

| Event Name | Trigger | Properties | business requirements document §9 KPI |
|---|---|---|---|
| user registered | New account created (POST /auth/register/tutee or /tutor succeeds) | role: tutee\|tutor, timestamp | business requirements document §9.4 new user acquisition |
| login success | User successfully authenticates | role: tutee\|tutor\|admin, timestamp | Session analytics, abuse detection |
| login failed | Authentication fails (wrong password or locked account) | failure reason: INVALID CREDENTIALS\|ACCOUNT LOCKED, timestamp | Abuse detection, security monitoring |
| registration started | User clicks "Register as Tutee" or "Register as Tutor" | role: tutee\|tutor, source page: string, timestamp | business requirements document §9.4 new user acquisition |
| registration completed | Email verified; account activated | role: tutee\|tutor, timestamp | business requirements document §9.4 registration conversion |
| user registered returning | Registration where email domain matches a previously closed account (detected at email hash level; email itself is not stored in analytics) | role: tutee\|tutor, previous role: tutee\|tutor | business requirements document §9.4 tutee repeat post rate, tutor re-listing rate |
| commission disclosure viewed | Tutor reaches commission disclosure screen | user id (anonymized after 90d), terms version: string, timestamp | business requirements document §9.4 tutor onboarding funnel |
| commission disclosure accepted | Tutor checks acknowledgement box and continues | user id (anonymized after 90d), terms version: string, timestamp | business requirements document §9.4 tutor onboarding funnel |
| registration abandoned | User starts registration but does not complete email verification within 24h | role: tutee\|tutor, step abandoned: string, timestamp | business requirements document §9.4 registration funnel drop-off |

### 16.2 Post Lifecycle Events

| Event Name | Trigger | Properties | business requirements document §9 KPI |
|---|---|---|---|
| post draft saved | User saves post as draft | post type: tutee\|tutor, post id (anonymized after 90d), timestamp | — |
| post submitted | User submits post for review | post type, post id (anonymized after 90d), subject, level, timestamp | business requirements document §9.2 admin review volume |
| post approved | Admin approves post | post type, post id (anonymized after 90d), subject, level, time in queue hours: float, timestamp | business requirements document §9.2 post review service-level agreement |
| post rejected | Admin rejects post | post type, post id (anonymized after 90d), rejection reason: enum, time in queue hours, timestamp | business requirements document §9.2 rejection rate |
| post revision requested | Admin requests revision | post type, post id (anonymized after 90d), time in queue hours, timestamp | business requirements document §9.2 revision rate |
| post resubmitted | User resubmits after revision | post type, post id (anonymized after 90d), resubmission count: int, timestamp | — |
| post closed by user | User closes own post | post type, post id (anonymized after 90d), days active: int, had match: bool, timestamp | — |
| post expired | System auto-expires post (created by post auto expiry cron) | post type, post id (anonymized after 90d), days active: int, timestamp | business requirements document §9.1 inventory churn |
| active inventory snapshot | Daily cron at midnight Singapore time | snapshot date: date, active tutor posts count: int, active tutee posts count: int, supply demand ratio: float | business requirements document §9.1 supply-demand ratio |
| admin review action taken | Admin approves, rejects, or requests revision | admin user id (anonymized after 90d), post id (anonymized after 90d), action: APPROVED\|REJECTED\|REVISION REQUESTED, review duration seconds: int | business requirements document §9.2 admin workload per match |

### 16.3 Discovery & Browse Events

| Event Name | Trigger | Properties | business requirements document §9 KPI |
|---|---|---|---|
| tutor listing page viewed | Any user views /tutors | filter subject: string\|no value, filter level: string\|no value, filter area: string\|no value, results count: int, timestamp | business requirements document §9.4 top-of-funnel traffic |
| tutor post detail viewed | Any user views /tutors/slug | post id (anonymized after 90d), subject, level, timestamp | business requirements document §9.4 tutor profile engagement |
| tutee request discovery viewed | Approved tutor views /tutor/requests | filter subject: string\|no value, filter level: string\|no value, filter area: string\|no value, results count: int, timestamp | business requirements document §9.1 tutor engagement with inventory |
| tutee request detail viewed | Approved tutor views /tutor/requests/id | post id (anonymized after 90d), subject, level, timestamp | business requirements document §9.1 demand signal visibility |
| tutor listing page cta clicked | Visitor clicks "Post a Request" or sign-up CTA from /tutors | cta location: string, destination: string, timestamp | business requirements document §9.4 CTA conversion |

### 16.4 Matching Events

| Event Name | Trigger | Properties | business requirements document §9 KPI |
|---|---|---|---|
| match created | Admin creates a match | match id (anonymized after 90d), tutee post subject, tutee post level, tutee post area, timestamp | business requirements document §9.3 match rate |
| match arrangement started | Match moves to ACTIVE status (distinct from commission status changed) | match id (anonymized after 90d), subject, level, area, timestamp | business requirements document §9.3 match-to-arrangement rate |
| match activated | Admin marks match ACTIVE | match id (anonymized after 90d), days since match created: int, timestamp | business requirements document §9.3 time to activation |
| match cancelled | Admin cancels match | match id (anonymized after 90d), match status at cancellation: string, days since match created: int, timestamp | business requirements document §9.3 cancellation rate |
| match closed | Admin closes match record (terminal = COMPLETED, §7.5) | match id (anonymized after 90d), commission status at close: string, days active: int, timestamp | business requirements document §9.3 match duration |
| tutor selected (M18) | Parent chooses a tutor (interview or choose) | tutee post id, tutor id, via: interview\|choose, timestamp | selection funnel |
| job assigned (M18) | Admin closes job = atomic assign (§20.4) | tutee post id, tutor id, match id, superseded count: int, timestamp | assignment rate |
| application superseded (M18) | Job assigned to another tutor | tutee post id, tutor id, application id, timestamp | bumped-applicant volume |

> M18 also adds a listing attached: bool property to the existing application submitted event
> (§16.3) to measure free-market applications-per-job / spam.

### 16.5 Commission Events

| Event Name | Trigger | Properties | business requirements document §9 KPI |
|---|---|---|---|
| first session confirmation sent | System sends tutee confirmation email | match id (anonymized after 90d), timestamp | business requirements document §9.3 confirmation initiation |
| first session confirmed by tutee | Tutee submits confirmation | match id (anonymized after 90d), days since confirmation sent: int, timestamp | business requirements document §9.3 tutee confirmation rate |
| first session confirmation overridden | Admin overrides confirmation | match id (anonymized after 90d), days overdue: int, timestamp | business requirements document §9.3 override rate |
| commission confirmation timed out | 7-day timeout reached with no tutee action | match id (anonymized after 90d), timestamp | business requirements document §9.3 timeout rate |
| invoice generated | Admin generates commission invoice | match id (anonymized after 90d), commission amount sgd: float, timestamp | business requirements document §9 commission revenue |
| commission marked paid | Admin marks commission PAID | match id (anonymized after 90d), days since invoice: int, timestamp | business requirements document §9 commission collection rate |
| commission disputed | Admin marks commission DISPUTED | match id (anonymized after 90d), timestamp | business requirements document §9 dispute rate |
| commission waived | Admin waives commission | match id (anonymized after 90d), timestamp | business requirements document §9 waiver rate |
| nps survey submitted | Tutee or tutor submits post-match NPS | match id (anonymized after 90d), user role: TUTEE\|TUTOR, nps score: int 0-10 | business requirements document §9.4 NPS |

### 16.6 Interview Events

| Event Name | Trigger | Properties | business requirements document §9.5 KPI |
|---|---|---|---|
| interview requested | Tutee submits interview request | interview id (anonymized after 90d), tutor post id (anonymized after 90d), has note: bool, timestamp | business requirements document §9.5 request rate |
| interview scheduled | Admin schedules interview (enters link + date/time) | interview id (anonymized after 90d), hours since request: float, meeting platform domain: string (e g., "zoom us" — derived from URL, not full link), timestamp | business requirements document §9.5 scheduling rate, turnaround |
| interview completed | Admin marks interview COMPLETED | interview id (anonymized after 90d), hours since request: float, minutes from scheduled time: int, timestamp | business requirements document §9.5 completion rate |
| interview cancelled | Admin or tutee marks interview CANCELLED | interview id (anonymized after 90d), cancelled by: admin\|tutee, from status: REQUESTED\|SCHEDULED, timestamp | business requirements document §9.5 cancellation rate |
| interview no show | Admin marks interview NO SHOW | interview id (anonymized after 90d), no show actor: TUTEE\|TUTOR\|BOTH, timestamp | business requirements document §9.5 no-show rate |
| interview meeting link updated | Admin updates meeting link on SCHEDULED interview | interview id (anonymized after 90d), meeting platform domain: string, timestamp | Meeting link stability tracking |
| interview converted to match | Match created within 14 days of completed interview (derived event, computed in analytics layer) | interview id (anonymized after 90d), match id (anonymized after 90d), days interview to match: int, timestamp | business requirements document §9.5 interview-to-match conversion |

**Note on meeting platform domain:** The full meeting link URL must NEVER be stored in analytics (it may be a single-use link or contain authentication tokens). Only the hostname (e g., "zoom us", "meet google.com") is captured for platform usage analytics.

---

### 16.7 Funnel Conversion Events

| Event Name | Trigger | Properties | business requirements document §9 KPI |
|---|---|---|---|
| landing page viewed | User views / | source: string utm source, medium: string utm medium, timestamp | business requirements document §9.4 top-of-funnel |
| registration cta clicked | User clicks any registration CTA | cta variant: string, user role intent: tutee\|tutor, source page: string, timestamp | business requirements document §9.4 CTA conversion |
| post to approval funnel | Track: submitted → approved → matched | Derived from post submitted + post approved + match created events | business requirements document §9.3 post-to-match funnel |
| tutee post approved to match days | Days between tutee post approval and first match created | Derived metric; computed in analytics layer | business requirements document §9.3 time-to-match |

### 16.8 Analytics Retention Rules

Per Appendix C §C.3 and §7.8.5 of this document:
- All analytics events older than 90 days must be anonymized: user id, post id, match id, and admin user id replaced with non-reversible anonymized hash values; IP addresses and session tokens removed
- Anonymized surrogate IDs cannot be linked back to the operational database
- Aggregate counts, conversion rates, and funnel summaries may be retained indefinitely in their aggregated form
- No personal information field values (email, phone, full name) may appear in any analytics event property at any time
- The 90-day anonymization pipeline must ONLY target the analytics event store; it must never modify the immutable audit log or the commission acknowledgements table

---

## 20. Assignment Lifecycle & Free Marketplace (M18)

**Status:** ✅ team-reviewed (full debate) + Boss-cleared (2026-05-31); cross-section consistency
fixes applied (§7.5/§9/§10.2/§12/§13/§16 reconciled). Ready to drive the private implementation folder specs. Encodes the
team-reviewed + Boss-cleared design
architecture/assignment-lifecycle-design md. Drives the private implementation folder specs that follow.
**Source of truth:** the service code; all claims verified against it 2026-05-31.

### 20.1 The model — free marketplace (founder)

Any tutor can express interest / **apply to any ACTIVE job with NO published listing required**.
Applying is open expression of interest. The old TUTOR NO ACTIVE POST 403 guard
(Job Application Service) is **removed**; the only remaining apply guard is "job must be ACTIVE"
(TUTEE POST NOT ACTIVE 422). Drive-by spam is throttled by a **profile-completeness gate** (rate +
area + commitment + qualifications) instead. tutor Post Id on an application stays **optional**.

### 20.2 Canonical identity = tutor profile (not listing)

Because applicants may have no listing, the **profile** carries the public identity: rate range,
area, YOE, **Part-time/Full-time**, rating. Listings become optional showcases. Model =
**profile-canonical + listing-overrides-when-attached** (no sync job). tutor profiles gains
commitment, hourly rate min/max sgd, location planning area, location region (none exist
today) + a **backfill** from each tutor's most-recent post. See §5.1a.

### 20.3 Assignment flow (parent-initiated, admin-confirmed)

1. Tutor **applies** to an active job (free market) → application APPLIED.
2. Parent views applicants (personal information-free) and either **requests an interview** or **chooses a tutor**
   (choose-tutor selection request).
3. Admin receives the request → **liaises** with the tutor.
4. **Both confirm → admin "closes the job"** = atomic assign (§20.4); **N-15** "assignment confirmed".
5–6. **Retain §8.3 / AC-05 / BR-010.** Activation locks the agreed first-month plan. The first lesson
   and **tutee first-session confirmation** gate invoicing/due handling. The tutor owes commission to
   the agency; **only agency receipt of that commission permits admin to mark PAID**, which completes
   the match (§7.5). Parent-to-tutor lesson payment is separate. The shortened PENDING→PAID collection
   summary does not remove confirmation/invoice obligations or define advance, partial or refunded receipts.

> **Cardinality note (reconciles §7.4):** parent choose-tutor is a *selection request*, NOT a match.
> The match is still **admin-created** (AC-04 holds); §7.4's "max 1 PENDING/ACTIVE match per tutee
> post" is unchanged — assign creates exactly one.

### 20.4 The atomic "assign" transition (one a database transaction)

Extends Admin Match Service create Match (inject Job Application Repository):
1. Chosen tutor's application → **MATCHED**; link match source application id.
2. Job (Tutee Post) ACTIVE → MATCHED (**reuse existing Post Status MATCHED** — no new enum;
   drops from public browse, blocks new applies).
3. **All other APPLIED/SHORTLISTED applications for that job → SUPERSEDED** (NEW status) via a
   **single set-based update records** (never a read-loop; index on tutee post id, status).
4. **Optimistic-version check** on the Tutee Post so two admins can't double-assign (2nd → 409).
5. Notifications fan-out (superseded / matched / parent) is **after-commit + failure-isolated** — a
   notification error must not roll back the assignment.
6. On **cancel** (pre-complete): reopen — job MATCHED→ACTIVE, superseded apps →APPLIED (Q1).

**Both entry points converge on ONE transition (the solution architect):** the existing admin 2-panel Match Creation
(§7.3) and the new parent-selection-driven assign both call the *same* extended
Admin Match Service create Match — there is **no second match path**. The parent selection request
(§20.3) is just a pre-step that tees up which (tutee, tutor) the admin confirms.

**Inline state-transition table (job × application × match):**

| Event | Job (Post Status) | Winning application | Other applications | Match (Match Status) |
|---|---|---|---|---|
| tutor applies (free market) | ACTIVE | — (this app → APPLIED) | — | — |
| parent requests interview / **chooses** | ACTIVE | — | — | — (selection request only) |
| **admin closes job** (assign) | ACTIVE → **MATCHED** | APPLIED → **MATCHED** | APPLIED/SHORTLISTED → **SUPERSEDED** | create **PENDING** (source application id set) |
| admin activates | MATCHED | MATCHED | SUPERSEDED | PENDING → **ACTIVE** (commission PENDING) |
| **agency receives tutor commission → admin marks PAID** | MATCHED | MATCHED | SUPERSEDED | ACTIVE → **COMPLETED** (commission PAID) |
| admin cancels (pre-complete) | MATCHED → **ACTIVE** | MATCHED → **APPLIED** | SUPERSEDED → **APPLIED** | → **CANCELLED** |
| tutor withdraws own app | (unchanged) | — | APPLIED → **WITHDRAWN** | — |

### 20.5 Application status (Job Application Status)

Actual enum today: the historical application states: applied, withdrawn, shortlisted, rejected and matched —
SHORTLISTED/REJECTED/MATCHED are **dead** (seeder-only; no service sets them). This delivery wires
APPLIED→SHORTLISTED→MATCHED, →WITHDRAWN, and **adds SUPERSEDED**. No explicit parent "decline"
in v1 — **REJECTED stays reserved, not wired** (decline is implicit via SUPERSEDED). Migration
guidance (Morgan): M18 changesets start at **0025** (head is 0024); adding SUPERSEDED to the
job application status PG enum is ALTER TYPE ... ADD VALUE — isolate in its own changeset with
run In Transaction is no; new tutor profiles columns + backfill + match source application id
ship in the same M18 pull request (schema-coupling rule). Tutor "My Applications" display vocabulary: Applied /
Shortlisted / Interview scheduled / **Assigned to you** (MATCHED) / **No longer available**
(SUPERSEDED) / Not selected (REJECTED) / Withdrawn.

### 20.6 personal information contract (binding — extends BR-001)

- **Parent applicant view is a DISTINCT personal information-free data transfer object** (NOT Admin Application Response) with its own
  **contract test**: handle, subjects, levels, rate, **coarse** area, YOE, PT/FT, rating — **NO real
  name, email, phone, or exact location**.
- **Child's name + parent's phone are hidden from applicants until the parent commits** (assignment).
- **Owner-authorization** on the applicant-list + choose-tutor endpoints (IDOR → 403).
- The public **assignment record** on a tutor profile shows subject / level / period / status only —
  **never the counterparty's identity**.

### 20.7 Endpoints (new/changed)

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | tutor applications interface operation | TUTOR | **Free-market apply** (listing-gate removed) |
| GET | tutee posts id applicants interface operation | TUTEE (owner) | **NEW** parent applicant view (personal information-free) |
| POST | tutee posts id select interface operation | TUTEE (owner) | **NEW** choose-tutor selection request |
| POST | admin matches interface operation | ADMIN | Extended = atomic assign (§20.4) |
| POST | admin commissions id pay interface operation | ADMIN | Mark PAID (existing) |

### 20.8 Observability & ops acceptance criteria (binding)

- Metrics: applications superseded total, tutor selections pending (gauge = admin backlog),
  assignments total, **commissions pending age** (stuck-revenue).
- **Alert** on commissions pending age over threshold — the pay-after-first-session step is a
  **manual admin action with no system deadline**; this alert is the only revenue-leak safety net.
- **Admin view** of stale-PENDING commissions.
- Analytics events: application superseded, tutor selected, job assigned, commission paid;
  application submitted carries whether a listing was attached (free-market spam metric).

### 20.9 Known limit (accepted v1)

Admin liaison per selection makes **admin throughput the ceiling** for assignment. Accepted for v1;
tutor selections pending makes the backlog visible. Auto/self-confirm is future work.

### 20.10 Bug fixes folded into M18

| # | Bug | Fix |
|---|---|---|
| ① | showcase tutor rate "undefined" | frontend: Showcase Tutor type + card read hourly Rate Min Sgd/Max (data transfer object field) |
| ② | "Reg Tutor Post" rows polluting browse | clean reseed (no test run after); reg-band purge follow-up |
| ③ | reset / admin password "only old works" | **Mailhog dev SMTP** so the one-time passcode outbox delivers (backend reset logic already correct — proven live new is 200/old is 401) |
| ④ | signup "gone" | frontend: add **Register** to desktop nav (/register already works) |

### 20.11 Test impact

Retire Guard Negatives IT apply Without Active Listing is403. Add: **positive free-market apply**
(no-listing tutor → 201), assign→winner-MATCHED / others-SUPERSEDED / job-MATCHED,
apply-after-assigned→422, cancel→reopen, choose-tutor happy + IDOR-403, applicant-list owner-scope +
personal information-shape contract test (~8–10 IT cases). Concurrency double-assign = unit-test territory.

---

*End of Product Requirements Document*

*product requirements document Version 1.2 — Interview Feature Added — 2026-05-27. Authored by Product Management. References business requirements document v1.2 (2026-05-27). All business rule citations (BR-001 through BR-015) reference the approved business requirements document. Changes to business requirements document business rules require corresponding product requirements document update. Data governance appendices (business requirements document Appendix C) are a go-live gate requiring DPO sign-off.*
