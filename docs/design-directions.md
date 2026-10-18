# InTuition SG: two design directions

Historical design proposal, converted to Markdown on 18 October 2026. The original supplied no approval date or final choice between its alternatives. This archive preserves its content and interaction ideas; it does not claim that either design was implemented or that its commercial/marketing copy was approved.

The source contained six desktop artboards: homepage, five-step parent request form and tutor search for each of two directions. Each artboard had its own miniature navigation. The document included sample tutors, invented reviews, indicative rates, browser/canvas controls and adjustable design settings. The relevant text and behaviour are retained below; executable presentation machinery is omitted.

## Two alternatives

| Choice | Direction A: Corporate Trust | Direction B: Modern Editorial |
|---|---|---|
| Intent | Familiar, credibility-led agency presentation, aimed at a parent of a secondary-school student | Premium magazine-like presentation with more space and editorial typography |
| Main colours | Blue #1E56A0 and amber #F5A623 | Navy #0F2138 and clay #C84B31 on warm paper |
| Typography and shape | Inter, rounded 8-pixel cards/buttons | Instrument Serif headings with Inter body text; sharper geometry |
| Homepage | Hero, statistics, three-step explanation, quick-start form, subject grid, featured tutors, testimonials, rates, trust cards and final request prompt | Split hero, inline statistics, three-step explanation, quick-start form, subject list, featured tutors, large parent story, rates and final request prompt |
| Request form | Horizontal progress/step treatment, main form with supporting trust/next-step content | Editorial step rail and main form with story/support content |
| Tutor search | Filter bar, card grid, clear-all and mid-page matching prompt | Editorial header, filter bar, card grid and direct-matching prompt |

Both directions used 1280 by 860 desktop artboards. Adjustable settings included primary/accent colours, the editorial serif font, compact/regular/comfy density, card radius from 0 to 20 pixels, and switches for WhatsApp calls to action and footers. Regular density used an 80-pixel section setting; compact and comfy used 48 and 112. Font alternatives were Instrument Serif, Playfair Display, DM Serif Display, Source Serif Pro and Newsreader. These are historical design controls, not current application requirements.

### Additional design choices retained from the controls

| Setting | Alternative values in the source |
|---|---|
| A primary | #1E56A0, #0F766E, #6D28D9, #BE185D, #0B6E4F |
| A accent | #F5A623, #DC2626, #16A34A, #7C3AED |
| B primary | #0F2138, #1A1614, #2A2317, #1F3A2F, #1A0F2E |
| B accent | #C84B31, #B8860B, #6B7F4F, #6B4F8E, #9B2C2C |

The sample badge treatments were gold for ex-teachers, blue for full-time tutors and grey for undergraduate tutors. The latter were categorized as part-time in filtering. A card's initial letters were an avatar placeholder, not a photograph or verified identity. The sample pagination showed pages 1, 2, 3, an ellipsis and 8 without a working page transition.

## Intended journey and actual mockup limits

The proposed acquisition journey was request a tutor → coordinator selects a tutor → trial lesson → continue if suitable. The quick-start fields were student level, subject and mobile number. Clicking their action opened the full form; the mockup did not transfer those quick-start values into the form or submit them to a server.

The navigation included home, tutor discovery, subjects, rates, about/support, tutor joining/login and the parent request action. Several navigation items were placeholders returning to an existing artboard, rather than complete destinations. The tutor-profile action returned to the tutor list; the request action opened the form without preserving a selected tutor. WhatsApp buttons and the numbered pagination were visual controls without a completed external delivery or paging flow.

Search stored subject, level, tutor type, region and availability filters. The mock data was actually filtered by subject text, type, region and availability; the level control did not restrict results. Sort options were best match (existing order), highest rating and lowest/highest starting hourly rate. No-results content suggested clearing filters or requesting a direct match. Cards displayed sample initials/name, category badge, credentials/experience, review score/count, subjects, rate range and availability. Direction A called an unavailable tutor Busy; Direction B called that state Booking. None of this proves working search or contact protection in the current application.

## Five-step form

| Step | Fields and choices | Validation or default in the mockup |
|---|---|---|
| Student | Student name, current level, optional school, one student or sibling group | Name and level required; sibling answer starts as one student |
| Subjects/goals | Multiple subjects; exam preparation, foundations, enrichment or homework support; current and target grade | At least one subject required; main goal stored as one choice |
| Schedule/location | Once, twice, three times weekly or intensive; one hour, 1.5 hours, two hours or flexible; family home, tutor home, online or hybrid; North/South/East/West/Central region | Frequency required; session length starts at 90 minutes and location at family home. A also requires region for family-home lessons; B does not validate region. Only A exposes a proposed start date. |
| Tutor preferences | Part-time, full-time, ex-teacher or open; gender preference; Mandarin/Malay/Tamil/Hindi/Hokkien/Cantonese; free-text needs | Optional step with Skip. Category price descriptions are sample copy, not an approved quotation. |
| Contact/summary | Parent name, mobile, optional email, request summary and contact-consent checkbox linked to privacy policy | Name, phone and contact consent required. Phone check accepts at least eight characters consisting of digits, plus, spaces or hyphens; it does not establish ownership. |

Back/Previous returns to an earlier step. Direction B also lets the user jump to any earlier completed step through its progress rail; Direction A's progress indicator is display-only. Continue/Next checks the current step, and the optional preferences step can be skipped. Editing a field clears its error. Final submit changes local display state after checking the final step; it does not store a request or produce a real reference. Success screens interpolate the entered name/subjects/level and display a random reference with a request prefix. A offers submit-another-request, which resets the step/success view but retains the entered data. Both show tutor browsing and faster-response WhatsApp prompts.

The final consent text authorizes contact about the request by phone, WhatsApp or email and references the privacy policy. This historical single checkbox is not a replacement for current registration consent requirements.

## Historical claims and current decision boundaries

The mockup claimed 500+ verified tutors, 3,000+ matched students, a 4.9 Google rating from 1,247 reviews, a 24-hour match/response, zero agency fees for parents, trial lessons and replies within an hour. It also used an establishment-year claim and hours of 9am to 9pm. These are illustrative claims, not measured achievements or approved business policy.

Direction A describes shortlisting; Direction B describes one hand-picked tutor and says it is a matching service rather than a marketplace. The current documented product includes applications and parent selection subject to staff assignment. Neither mockup changes that rule. Sample copy alternates between a no-obligation first lesson and a free trial; no free-lesson entitlement is approved by this archive.

The current requirements retain tutor-to-agency commission, WhatsApp OFF until its enablement conditions are met, an 08:00 to 22:00 SGT admin availability requirement, and unresolved O05/O11 verification-badge/testimonial content. The historical 90-minute form default does not decide O09's current parent-default question. Use the linked current register for governing rules; use this archive only to understand the design exploration.

## Sample subjects and levels

Subjects: English, Maths, Science, Chinese, Physics, Chemistry, Biology, History, Geography, Economics, GP and Literature. The full form also offered Higher Chinese, A-Math, E-Math and Combined Sci.

Levels: Preschool; Primary 1 to 3; Primary 4 to 6 / PSLE; Secondary 1 to 2; Secondary 3 to 5 / O-Level; Junior College / A-Level; IB / IGCSE; Polytechnic; University; AEIS. These display options are historical examples, not the approved taxonomy mapping.

## Sample rate table

The source called these hourly Singapore rates. Preserve them as mock design data only; no current market-rate research or approval was supplied.

| Level | Part-time | Full-time | Ex-teacher |
|---|---|---|---|
| Preschool (N to K2) | $20 to $30 | $30 to $40 | $50 to $70 |
| Primary 1 to 3 | $25 to $35 | $35 to $50 | $55 to $75 |
| Primary 4 to 6 / PSLE | $30 to $40 | $40 to $60 | $60 to $80 |
| Secondary 1 to 2 | $35 to $45 | $45 to $65 | $65 to $90 |
| Secondary 3 to 5 / O-Level | $40 to $55 | $55 to $80 | $70 to $100 |
| Junior College | $60 to $80 | $80 to $100 | $90 to $130 |
| IB / IGCSE | $60 to $100 | $90 to $130 | $100 to $150 |
| Polytechnic / University | $50 to $80 | $80 to $120 | Not supplied in the source |

## Sample tutor cards

All rows below come from illustrative mockup data. They are not real endorsements, live availability, verified qualifications or evidence for a universal badge.

| Sample name | Category | Rating / review count | Credential/experience example | Subject examples | Hourly rate | Availability / region |
|---|---|---|---|---|---|---|
| Sample tutor 1 | Ex-MOE | 4.9 / 42 | BSc (NUS) · NIE-trained; 8 yrs | Sec Maths, A-Level Maths, PSLE Maths | $65 to $90 | Available; East |
| Sample tutor 2 | Full-Time | 5 / 67 | MEd (NIE) · 11 yrs exp; 11 yrs | O-Level Chem, A-Level Chem, IB Chem HL | $60 to $85 | Available; Central |
| Sample tutor 3 | Full-Time | 4.9 / 38 | BA Hons (NTU) Lit; 6 yrs | O-Level Eng, GP, Literature | $55 to $75 | Available; West |
| Sample tutor 4 | Part-Time | 4.8 / 19 | NUS Y3 · PSLE 273; 3 yrs | P1 to P6 Maths, P1 to P6 Science | $30 to $40 | Busy/booking; North |
| Sample tutor 5 | Ex-MOE | 4.9 / 54 | PGDE (NIE) · MOE 9 yrs; 12 yrs | Sec Physics, A-Level Physics | $70 to $100 | Available; Central |
| Sample tutor 6 | Full-Time | 4.7 / 28 | BA Chinese (NTU); 5 yrs | PSLE Chinese, O-Level CL, Higher Chinese | $50 to $70 | Available; East |
| Sample tutor 7 | Ex-MOE | 5 / 73 | MA Econs (NUS) · NIE; 14 yrs | A-Level Econs, IB Economics | $90 to $130 | Busy/booking; Central |
| Sample tutor 8 | Full-Time | 4.8 / 31 | BSc Bio (NUS); 7 yrs | Sec Bio, O-Level Bio, IB Bio | $55 to $75 | Available; West |
| Sample tutor 9 | Part-Time | 4.9 / 22 | SMU Y4 · 4× SYF Gold; 2 yrs | P1 to P6 English, Lower Sec Eng | $28 to $38 | Available; North |

## Sample testimonial content

These invented testimonials are retained as design history. None is approved for live publication. The current content/consent dependency remains O11.

### Sample story 1

My son went from D7 to A1 for O-Level Chemistry in just 4 months. Sample tutor 2 explained organic chem from scratch, patiently.

Attribution in mockup: Sample parent 1, Parent of Sec 4 student; O-Level Chemistry; D7 → A1 in 4 months; Mar 2025; 5 stars.

### Sample story 2

Daughter's PSLE Math jumped from 68 to AL2 after working with Sample tutor 1. The trial lesson convinced us: no hard sell, just teaching.

Attribution in mockup: Sample parent 2, Parent of P6 student; PSLE Mathematics; 68 → AL2 in 6 months; Feb 2025; 5 stars.

### Sample story 3

Found a tutor within 18 hours of submitting. Zero agency fee, transparent rate, and the tutor was a perfect personality fit.

Attribution in mockup: Sample parent 3, Parent of Sec 3 student; A-Maths; C5 → A2 in 5 months; Jan 2025; 5 stars.

### Sample story 4

Sample tutor 5 pinpointed the exact topics my daughter was weak in. Physics went from a fail to A2 by prelims.

Attribution in mockup: Sample parent 4, Parent of Sec 4 student; O-Level Physics; F9 → A2 in 7 months; Nov 2024; 5 stars.

### Sample story 5

Sample tutor 7's notes are gold. My son scored an A for A-Level Econs after struggling with case studies.

Attribution in mockup: Sample parent 5, Parent of JC2 student; A-Level Economics; U → A in 9 months; Dec 2024; 5 stars.

## Screen-copy inventory

The following preserves distinct labels, messages, supporting copy and field choices from both alternatives. Some fragments belong around dynamic names, counts or selected values; their full intended meaning is explained in the journey/form sections above. Repeated identical text within one screen is listed once. Punctuation-only fragments and internal navigation keys are omitted.

### Direction A: navigation, tutor cards and footer

- Home
- Find a Tutor
- Our Tutors
- Subjects ▾
- Tuition Rates
- About
- FAQ
- iT
- InTuition
- SG
- Join as Tutor
- Request a Tutor
- For Parents
- Browse Tutors
- How It Works
- Subjects A to Z
- Join as a Tutor
- Open Assignments
- Tutor Login
- Pay Rates
- Support
- Help Centre
- Contact
- WhatsApp Us
- Mon to Sun 9am to 9pm
- Email
- Legal
- Privacy Policy
- Terms of Service
- Refund Policy
- PDPA Statement
- Sitemap
- InTuition SG
- Singapore's trusted home tuition matching service. Free for parents, verified tutors only.
- UEN: company registration value omitted · Est. 2018
- WhatsApp mobile number placeholder
- © 2025 the agency legal name All rights reserved.
- ⭐ 4.9 from 1,247 Google Reviews
- /hr
- View Profile
- Request

### Direction A: homepage

- Singapore's Trusted Home Tuition Agency
- Find the right tutor for your child,
- in 24 hours.
- Free to request. Verified tutors only. Matched by subject, level, and location.
- Find a Tutor Now
- WhatsApp Us
- ⭐ 4.9/5 from 1,247 Google Reviews · Zero agency fees · Trial lesson included
- Tutor + student photo
- 500+
- Verified Tutors
- Matched within 24hrs
- 4.9 · 1,247 reviews
- on Google
- Students Matched
- Google Rating
- Agency Fee
- How It Works
- Get a tutor in 3 simple steps
- Submit your request
- Tell us your subject, level, and schedule. Takes 2 minutes.
- We match your tutor
- We shortlist verified tutors and contact you within 24 hours.
- Start your first lesson
- Trial lesson with no obligations. Continue only if you're happy.
- Quick Start
- Get matched in 2 minutes
- Free. No obligations. We'll call within 24 hours.
- Student Level
- Subject
- +65 Phone Number
- Get Matched Now →
- Subjects
- What does your child need help with?
- View all subjects →
- Our Tutors
- Meet some of our tutors
- Browse All Tutors
- Real Results
- From Singapore families
- Pricing
- Transparent rates. Zero hidden fees.
- All rates paid directly to your tutor. Our matching service is completely free to parents.
- Level
- Part-Time
- Full-Time
- Ex-MOE
- /hr
- See full rates table →
- Why Choose Us
- Built on trust
- Profile checks, credential verification, and our 3-tier VerifiedTutor badge system.
- 24-Hour Matching
- Average matching time. Most parents hear from us the same day.
- Zero Agency Fees
- You pay the tutor directly. We never take a cut from parents.
- Trial Lesson
- No obligation after first session. Switch tutors anytime.
- Ready to find your child's tutor?
- 500+ verified tutors available right now. Response within 24 hours.
- Request a Tutor

### Direction A: five-step request form

- About the Student
- Subjects & Goals
- Schedule & Location
- Tutor Preferences
- Your Contact Details
- Please enter the student's name
- Select a level
- Choose at least one subject
- How often would you like lessons?
- Tell us your region
- Invalid phone
- You must agree to be contacted
- You're all set,
- We've received your request for
- ). Expect a call or WhatsApp from us within 24 hours.
- WhatsApp Us for Faster Response
- Browse Our Tutors
- Submit another request
- Reference:
- #REQ-
- Request a Tutor
- Takes under 2 minutes. Free. Matched within 24 hours.
- Free
- 24hr Response
- No Obligations
- Trial Lesson Available
- Step
- of
- Student's name
- e.g. student first name
- Current level
- Select level...
- School (optional)
- Helps us match a tutor familiar with your school's syllabus
- e.g. school name
- Any siblings who also need tutoring?
- No, just one student
- Yes, sibling group
- Which subjects?
- Tap to add. You can pick more than one.
- English
- Maths
- Science
- Physics
- Chemistry
- Biology
- Chinese
- Higher Chinese
- History
- Geography
- Economics
- Literature
- GP
- A-Math
- E-Math
- Combined Sci
- Main goal
- What outcome matters most?
- Exam preparation
- PSLE, O/N-Level, A-Level, IB
- Build foundations
- Strengthen weak areas
- Enrichment
- Push beyond syllabus
- Homework support
- Regular help with assignments
- Current grade
- e.g. C5 or 65%
- Target grade
- e.g. A2 or 80%
- How often?
- 1× per week
- 2× per week
- 3× per week
- Intensive
- Pre-exam crash
- Session length
- 1 hour
- 1.5 hours
- 2 hours
- Flexible
- Where will lessons be held?
- At our home
- At tutor's home
- Online only
- Hybrid
- Mix of home + online
- Region
- North
- South
- East
- West
- Central
- When would you like to start?
- Tutor type
- Not sure? Skip. We'll suggest the right fit.
- Part-Time
- Undergrads / recent grads. $20 to $55/hr.
- Full-Time
- Career tutors, 5+ yrs. $35 to $130/hr.
- Ex-MOE Teacher
- NIE-trained. $50 to $150/hr.
- Open to all
- Match by best fit, not category.
- Tutor gender preference
- No preference
- Female
- Male
- Language preference (besides English)
- Tap any that apply
- Mandarin
- Malay
- Tamil
- Hindi
- Hokkien
- Cantonese
- Anything else we should know?
- Learning differences, exam dates, preferred teaching style, etc.
- e.g. My child is preparing for prelims in August...
- Your name
- e.g. Sample parent 2
- Mobile number
- We'll WhatsApp or call you within 24 hours
- Email (optional)
- email address placeholder
- Your request summary
- Student:
- Subjects:
- Schedule:
- Location:
- Tutor type:
- open to all
- I agree to be contacted by InTuition SG via phone/WhatsApp/email about my request. I have read the
- Privacy Policy
- ← Back
- Skip this step
- Next →
- Submit Request
- What happens next
- Submit
- Your request reaches our matching team.
- Match
- We shortlist verified tutors who fit your needs.
- Trial
- Free trial lesson. No commitment, switch anytime.
- "From D7 to A1 in 4 months. Best decision we made for our son's O-Levels."
- Attribution: Sample parent 1, Parent · Chemistry
- 500+
- tutors ready to be matched
- WhatsApp Us Instead
- Mon to Sun 9am to 9pm · We reply within the hour

### Direction A: tutor search

- Our Tutors
- Find your tutor
- Browse verified Singapore tutors by subject, level, and location.
- All subjects
- All levels
- Any tutor type
- Part-Time
- Full-Time
- Ex-MOE
- All regions
- North
- South
- East
- West
- Central
- Available now
- Clear all (
- Sort: Best Match
- Sort: Highest Rated
- Sort: Rate ↑
- Sort: Rate ↓
- found
- · filtering by
- No tutors match those filters
- Tell us what you need and we'll find one for you.
- Request a Tutor
- Can't find the right tutor?
- Let us match you manually based on your specific needs.
- WhatsApp Us

### Direction B: navigation, tutor cards and footer

- Home
- Tutors
- Subjects
- Rates
- Stories
- About
- InTuition
- SG
- Tutor Login
- Request a Tutor →
- A new kind of tuition agency. Built for Singapore parents who care about fit, not just credentials.
- WhatsApp mobile number placeholder
- UEN company registration value omitted · Operating Mon to Sun, 9am to 9pm
- Parents
- Request a Tutor
- Browse Tutors
- How It Works
- Join as Tutor
- Open Assignments
- Pay Rates
- Login
- Company
- About Us
- Contact
- Privacy & Terms
- © 2025 the agency legal name
- ★ 4.9 from 1,247 Google Reviews
- reviews
- , with
- teaching.
- per hour
- Profile
- Request

### Direction B: homepage

- Singapore home tuition · est. 2018
- The right tutor,
- matched in 24 hours.
- Free to request. Verified tutors only. Matched by subject, level, location, and the way your child actually learns.
- Request a Tutor
- WhatsApp
- 4.9 from 1,247 Google Reviews
- Zero agency fees
- Hero: tutor portrait
- "From D7 to A1 in four months."
- Sample parent 1, Sec 4 parent
- O-Level Chemistry · 2024
- 500+
- Verified tutors
- Students matched
- 24 hr
- Average match time
- Agency fee to parents
- The Process
- Three steps,
- start to finish.
- We're not a marketplace. We're a matching service: one tutor, hand-picked for your child's level, subject, and personality. Free, with a no-obligation trial lesson.
- Submit your request
- Tell us about your child: level, subjects, schedule, and any specific goals. Takes two minutes.
- We hand-pick a tutor
- Our coordinator matches one verified tutor based on subject mastery, teaching style, and location.
- Trial, then continue
- First lesson is a no-commitment trial. Continue only if you and your child are happy with the fit.
- Start Here
- Tell us what
- you need.
- A short form: three fields to start, the rest is optional. We'll respond within 24 hours.
- Prefer to chat? Message us on WhatsApp, anytime 9am to 9pm.
- Level
- Choose level
- Subject
- Choose subject
- Your mobile number
- Get matched. It's free
- Free. No obligations. Response within 24 hours.
- Subjects
- Every
- subject,
- every level.
- View all subjects
- P · O · A
- Our Tutors
- Hand-picked,
- not auto-listed.
- Browse all tutors →
- A Parent's Story
- "From
- D7
- to
- A1
- in four months.
- We finally found a tutor who fit."
- "Sample tutor 2 didn't just drill past papers. He explained organic chemistry from scratch and rebuilt my son's confidence. Worth every cent."
- L
- Sample parent 1
- Parent · O-Level Chemistry · Tampines
- Transparent Rates
- Paid to the tutor.
- Never to us.
- Standard Singapore market rates. We don't take a cut from parents. Our matching service is genuinely free. Tutors keep 100% of what you pay.
- Part-Time
- Full-Time
- Ex-MOE
- /hr
- View full rates table →
- Ready to start
- Find your child's
- right tutor
- today.
- 500+ verified tutors. Matched in 24 hours. Free, with a trial lesson.
- Request a Tutor →
- WhatsApp Us

### Direction B: five-step request form

- The Student
- Subjects
- Schedule
- Preferences
- Contact
- Progress
- 0
- We'll need the student's name
- Choose the current level
- Pick at least one subject
- Roughly how often?
- Doesn't look like a phone number
- Please tick the box to continue
- Request received
- Thank you
- We've received your request for
- at
- . One of our coordinators will be in touch within 24 hours via your preferred channel.
- WhatsApp for faster response
- Browse Tutors
- Reference number
- REQ-
- Tutor Request
- Tell us about
- your child.
- Two minutes. Free. Matched within 24 hours.
- Step
- of
- About
- the student.
- & goals.
- When
- & where?
- Tutor
- preferences.
- Your
- contact details.
- Student's first name
- e.g. student first name
- Current level
- Choose level...
- School (optional)
- Helps us match a tutor familiar with your school's syllabus and culture
- e.g. school name
- Siblings who also need tutoring?
- Just one student
- Yes, sibling group
- Subjects needed
- English
- Maths
- Science
- Physics
- Chemistry
- Biology
- Chinese
- Higher Chinese
- History
- Geography
- Economics
- Literature
- GP
- A-Math
- E-Math
- Combined Sci
- The main goal
- It's fine to pick more than one priority later.
- Exam preparation
- PSLE, O/N-Level, A-Level, IB
- Build foundations
- Strengthen weak areas
- Enrichment
- Push beyond the syllabus
- Homework support
- Regular help with assignments
- Current grade
- e.g. C5
- Target grade
- e.g. A2
- How often?
- 1× per week
- 2× per week
- 3× per week
- Intensive
- Pre-exam crash
- Session length
- 1 hour
- 1.5 hours
- 2 hours
- Flexible
- Where
- At our home
- At tutor's home
- Online only
- Hybrid
- Region
- North
- South
- East
- West
- Central
- Tutor type
- Not sure? Skip. We'll recommend the best fit.
- Part-Time
- Undergrads · $20 to $55/hr
- Full-Time
- Career tutors · $35 to $130/hr
- Ex-MOE Teacher
- NIE-trained · $50 to $150/hr
- Open to all
- Match by best fit
- Gender preference
- No preference
- Female
- Male
- Language preference (besides English)
- Mandarin
- Malay
- Tamil
- Hindi
- Hokkien
- Cantonese
- Anything else we should know?
- Learning style, exam dates, special considerations...
- Your name
- e.g. Sample parent 2
- Mobile
- We'll WhatsApp or call within 24 hours
- Email (optional)
- email address placeholder
- Summary
- Student:
- Subjects:
- Schedule:
- Location:
- I agree to be contacted by InTuition SG via phone, WhatsApp, or email about this request. I've read the
- Privacy Policy
- ← Previous
- Skip
- Continue →
- Submit Request →
- A parent's story
- "Found a tutor in 18 hours, and she stayed with us through PSLE."
- SAMPLE PARENT 2 · P6 PARENT
- Need a faster answer?
- Skip the form. WhatsApp our coordinators directly; we typically reply within an hour.
- WhatsApp Us
- 500+
- Verified tutors, ready to be matched.

### Direction B: tutor search

- The Roster
- Find
- your
- tutor.
- Browse our verified Singapore tutors. Each profile is hand-checked for credentials, references, and teaching track record before going live.
- Active tutors
- 4.9
- Average rating
- 24h
- Avg. response
- Filter
- Any subject
- Any level
- Any tutor type
- Part-Time
- Full-Time
- Ex-MOE
- Any region
- North
- South
- East
- West
- Central
- Available now
- Clear (
- Sort
- Best match
- Highest rated
- Rate, low to high
- Rate, high to low
- · filtered
- No matches
- Nothing
- for those filters.
- Try clearing some, or let us match you directly.
- Request a Tutor →
- A Hand-Picked Match
- Can't find the right fit?
- We'll match you directly.
- WhatsApp Us
- Page

## Conversion record

The original bundled document was inspected, including its eight content/design resources, shared sample data and application settings. The archive retains both alternatives and their differences; no final design approval was found in the source. Fonts, SVG path instructions, styles, browser chrome and bundler/runtime code were omitted.

The public edition replaces sample personal names, direct contact placeholders and the legal company value with descriptions. The private archive retains the original display text.

[Current requirements](../specs/CURRENT-REQUIREMENTS.md) · [Current user flows](user-flows.md) · [Cover concept](cover-concept.md)
