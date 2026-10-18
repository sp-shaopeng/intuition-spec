# M18 — Assignment Lifecycle & Free Marketplace — user interface/user experience Design Spec (with ASCII wireframes)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Use [CURRENT-REQUIREMENTS](../CURRENT-REQUIREMENTS.md) for guest filters and agency receipt, [012](../012-tutor-apply-and-profile-ux/requirements.md) for completeness, and [INT-125](../architecture/int-125-batch-closeout-spec.md) for assigned-participant contact with ACTIVE/COMPLETED frontend reveal. Public identity remains the approved handle/public field, never an implicit private display Name release. This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Owner:** user interface/user experience (Jamie) · **Date:** 2026-05-31 · **Status:** draft → for team-review → frontend handoff
**Drives:** the frontend spec for M18.
**Ground truth:** architecture/assignment-lifecycle-design md (the approved design that drives product requirements document
§20), product requirements document §2 personas, §4 journeys, §10.2 (Browse Tutors), §11 (authenticated pages by role), §13
AC-04 / §3.6 AC-20 / §3.7 AC-21, §20.1–20.11.
**Design-system source:** the live intuition-frontend — this spec REUSES existing primitives and
tokens; it does **not** introduce a new design language. Everything marked "(verified)" was read from
the actual component source.

---

## 0. Design system this spec builds on (verified in code)

Every wireframe annotation maps to something that already exists:

| Token / primitive | Real value (from frontend source) | Used in this spec for |
|---|---|---|
| Colors | cream (page bg), cream-border/sand (borders/muted), ink (body text), navy (headings/links), clay (accent / danger fill) | all surfaces |
| Headings | font-serif text-navy (cards use font-serif text-lg text-navy) | every screen title / card subject line |
| Button | variants primary using cream bg / navy text secondary using transparent / cream border ghost using transparent / navy / cream-border danger using clay bg / white text; sizes sm/md/lg; loading prop shows inline spinner + auto-disables; disabled:opacity-50; full Width | all actions |
| Badge | variants pending using amber live using green rejected using red draft/cancelled using gray scheduled using blue completed using green default using cream-muted; rounded-full text-xs | status pills, PT/FT chip — mapped to these REAL variants in the table below |
| Modal | native dialog, aria-modal aria-labelledby, the show modal operation focus-trap, Esc + backdrop close, max-w-md; sub-components Modal Header/Body/Footer | Choose-tutor confirm, Apply sheet, assign confirm |
| Empty State | centered; props icon using emoji, default 📭 + heading using navy + description?(sand) + action? | every empty list |
| Spinner / Skeleton | sizes sm/md/lg | loading states |
| Star Rating | props value + size; cards render it only when average Rating differs from no value and review Count 0 | all cards |
| toast | the success operation / the error operation transient | every mutation result |
| format Sgd Range, format Location, format Date, application Status Label, application Status Badge Variant (in utils/format) | existing formatters; application Status Label/application Status Badge Variant are the EXISTING status→label/variant mappers to EXTEND for SUPERSEDED + interview pseudo-status | never render a raw enum to a user |
| Nav | sticky h- 84px bg-cream; desktop links hidden md:flex; container max-w-7xl | §H |
| Constants | JOB APPLICATION STATUS (constants/status), application programming interface ROUTES (lib/api Routes) — no magic strings | enums + routes |

**New components this spec introduces (see Handoff):** Tutor Identity Card (redesigned nameless card),
Commitment Badge, Status Pill (the display-vocab mapper, built on application Status Label),
Applicant Card, Choose Tutor Modal, Apply Interest Sheet, Pii Notice, Contact Reveal Card,
Selection Request Queue, Close Job Assign Modal, Stale Commission List, Profile Completeness Bar.

### Display vocabulary — single source of truth (NEVER render the raw enum)

Extend the existing application Status Label / application Status Badge Variant mappers:

| Job Application Status | User-facing label | Badge variant | Meaning |
|---|---|---|---|
| APPLIED | "Applied" | pending (amber) | sent; awaiting parent/admin |
| SHORTLISTED | "Shortlisted" | scheduled (blue) | parent flagged interest |
| (interview exists for this job — has Interview flag, NOT a status) | "Interview scheduled" | scheduled (blue) | M6 interview booked. Status Pill takes both the application status and whether an interview exists — see Handoff |
| MATCHED | "Assigned to you" | **assigned (emerald) — NEW Badge variant** | this tutor won the job (distinct from live/completed, which are byte-identical green) |
| SUPERSEDED | "No longer available" | cancelled (gray) — warm copy, NOT a faded row (§D) | job went to someone else |
| REJECTED | "Not selected" | draft (gray) | enum exists; unused by parent flow in v1 (Q5) |
| WITHDRAWN | "Withdrawn" | draft (gray) | tutor pulled their own application |

| Post Status (job) | User-facing | Badge variant |
|---|---|---|
| ACTIVE | "Open · accepting applicants" | live (green) |
| MATCHED | "Position filled" (parent) / "Assigned" (admin) | scheduled (blue) |
| CLOSED | "Closed" | cancelled (gray) |

Tutor Commitment: PART TIME→"Part-time", FULL TIME→"Full-time" (chip on every tutor-facing card).
**RESOLVED:** add a dedicated **assigned (emerald) Badge variant** for MATCHED — visually distinct
from live/completed (which are byte-identical bg-green-100) so "Assigned to you" reads as a win,
not a schedule. This is a one-variant design-system addition (frontend component inventory).

---

## A. Tutor — Profile setup / edit (add PT/FT + rate range + area; profile is the apply gate)

**Maps to:** product requirements document §5.1a, §20.2; design §5. **Persona:** Wei (NTU part-timer) — wants YOE + part-time
status clear, hates friction. **Route:** /tutor/profile.

The profile now carries the **public identity** (a tutor can apply with no listing). The new fields
drive the applicant card and Browse Tutors: **Public handle**, **Commitment (PT/FT)**, **Hourly rate
range**, **Coarse area**. Profile-completeness is the **gate to apply** (not "own a listing").

**The *-required set MUST equal the real apply gate (backend §3.2/§12 #5) — exactly:** handle,
commitment, hourly rate (min+max), coarse area, qualifications. **Years-of-experience is NOT a
gate** (no *). The completeness bar counts these five only — no "100% then bounced at Apply" (Wei).


Implementation example omitted. This public edition retains the written requirements in this section.


**Apply-gate banner** (renders on job-detail / My Applications when profile incomplete):


The illustration for a. tutor — profile setup / edit add pt/ft + rate range + area profile is the apply gate contains the following information:

- ⚠ Finish your profile to apply.
- Add your rate range and commitment so parents know what.
- to expect. [ Complete profile → ].


- **Fields/actions:** * = required = part of the completeness gate (commitment, rate min/max, area,
  qualifications). Save profile = primary; Cancel = ghost; Preview = ghost link rendering
  the §B card with the tutor's own data.
- **States:**
  - *Loading:* Skeleton row per field; Save disabled.
  - *Empty / first-time:* completeness 0%; banner explains apply is gated until complete.
  - *Validation (inline):* rate Minimum cant be higher than maximum.; commitment "Pick part-time
    or full-time."; for an empty required field, display"This helps parents decide — please fill it in."
  - *Error (save):* keep entered values; the error operation
  - *Success:* the success operation; completeness bar animates; if now ≥ gate the apply
    banner disappears.
- **Responsive:** desktop single column ~max-w-2xl, stacked groups; mobile full stack, rate min/max
  stay on one row (flex gap-2), select controls full-width.
- **Accessibility:** every input has a label associated with its control; commitment + area are native select controls (keyboard/AT
  native); required aria-required is yes; inline errors via aria-describedby + aria-invalid;
  completeness bar role is progressbar aria-valuenow.
- **Microcopy:** Commitment helper "How parents see your availability." Display-name helper (verbatim
  above) states the personal information boundary.
- **personal information boundary:** display name + full qualifications = admin/self only. Rate / coarse area / YOE /
  commitment / rating = public-card-safe. Area must be **coarse** (region), never street/postcode.
- **OPEN (UIUX):** Area = single coarse region select control vs region→planning-area cascade (design §5
  lists both location planning area + location region). Recommend **coarse region** for v1.

---

## B. Public tutor card redesign — /tutors browse + landing showcase strip (hide name; fix S$undefined)

**Maps to:** product requirements document §10.2 / §10.1, §20.2; design §6; bug ①. **Persona:** Wei (no name pre-deal), Priya
(needs rate/area/YOE/PT-FT to vet). **personal information rule: NO real or display name on these cards. Ever.**

Lead with **subjects → levels**, then Rate / Area / YOE / **PT-FT** / rating. The card's identity is its
subject expertise, not a person.

### B1. /tutors browse card (redesign of Tutor Card implementation reference → Tutor Identity Card)


Implementation example omitted. This public edition retains the written requirements in this section.


- **Remove** the initials avatar circle and the tutor tutor Display Name text node (lines 25, 122–129
  of Tutor Card implementation reference). The level-three heading/subject  becomes the lead. Keep the favourite heart (already
  aria-pressed, TUTEE-only) and the grid.
- Card grid unchanged: desktop 3-up, tablet 2-up, mobile 1-up.

### B2. Landing showcase strip card (Showcase Tutor Card implementation reference)


The illustration for b2. landing showcase strip card showcase tutor card implementation reference contains the following information:

- English ★ 4.6 <- subjects rating. NO name.
- Rate S$28–40/hr <- BUG ① FIX (see below).
- Area East.
- [ Full-time ] [Available] <- NEW PT/FT chip existing "Available" live Badge.


- **Bug ① fix (annotation for frontend):** Showcase Tutor Card currently calls
  the format sgd range operation (line 69) but the data transfer object sends
  hourly Rate Min Sgd/hourly Rate Max Sgd → renders "S$undefined". Align the Showcase Tutor type + card to
  hourly Rate Min Sgd/Max. Also **remove** the trailing tutor tutor Display Name line (line 75) and the
  initials block (lines 26–31, 42–47) — that name is the personal information leak; add commitment to the type +
  render the chip.
- Showcase card stays non-link pre-login (existing behavior).

**States (both cards):**
- *Loading:* Skeleton card (subject bar + 2 muted bars) ×N.
- *Empty (browse):* Empty State heading is No tutors match yet" description is Try widening your filters."
  action=Clear filters control.
- *No rating:* hide Star Rating (existing average Rating equals no value guard).
- *Missing PT/FT (back-filled tutor):* omit the chip rather than show "undefined"; never block the card
  (back-fill may leave it null — design §5).
- *Error:* page-level retry, not per-card.

**Responsive:** chips wrap; the grid rows already stack on mobile.
**Accessibility:** since there's no name to read, the View-Profile link / card gets an aria-label
composed from subjects + area + rate, e g. Mathematics, Physics tutor in Central, S 30–45 per hour.
PT/FT meaning is in the chip TEXT (color is decorative).
**Microcopy:** rate an hourly price range in Singapore dollars; experience the number of years, with correct singular or plural wording (existing pluralization).
**personal information boundary (call-out):** subjects / levels / rate / coarse area / YOE / PT-FT / rating ONLY. No
name, contact, or exact location. The detail page behind the card is login-gated (M17) and stays
nameless until assignment.

---

## C. Tutor — Job detail → "Apply / Express interest" (free market; apply ≠ match)

**Maps to:** product requirements document §3.6 AC-20 (apply allowed, no listing), §13 AC-04 (apply does NOT create a match);
design §0/§4. **Persona:** Wei. **Route:** job detail /assignments/:id (login-gated, M17).


Implementation example omitted. This public edition retains the written requirements in this section.


**Apply panel (Modal on desktop / bottom-sheet on mobile):**


The illustration for c. tutor — job detail → apply / express interest free market apply ≠ match contains the following information:

- Express interest [x].
- This sends your profile to the parent. <- profile-sourced (§A/§B).
- Applying does not book the job — the parent <- AC-04 honesty (REQUIRED line).
- and our team decide next steps.
- Your pitch (optional).
- [ Hi! I've prepped 10 students for O-… ] <- optional textarea char counter.
- Attach a listing (optional) [ none v] <- optional select control of tutor's OWN active listings.
- Showcases your rate/availability for this. "none" = profile values used (§5 override rule).
- [ Cancel ] [ Send interest ].


- **CTA gating:** Apply / Express interest enabled only if the §A completeness gate passes; otherwise
  it's replaced by the apply-gate banner (Complete profile →). Pitch + attach-listing both
  optional. Attaching a listing means that listing's rate/area **overrides** the profile values on the
  parent's applicant card (design §5) — say so in the helper line.
- **States:**
  - *Already applied:* CTA → disabled secondary ✓ Interest sent + link "View in My Applications".
  - *Profile incomplete:* gate banner instead of CTA.
  - *Job not ACTIVE (filled while viewing):* CTA disabled, status flips to "Position filled", inline
    note "This job has been assigned." (mirrors backend TUTEE POST NOT ACTIVE 422).
  - *Submitting:* Send interest loading.
  - *Success:* close sheet, the success operation,
    CTA → "Interest sent".
  - *Error:* keep sheet open, the error operation
- **Responsive:** desktop = Modal max-w-md; mobile = full-width bottom sheet, page CTA sticky to
  viewport bottom.
- **Accessibility:** sheet = shared Modal (dialog, focus-trap, Esc). Char counter aria-live is polite.
  Attach-listing is a native select control.
- **personal information boundary:** job detail shows NO parent/child name, exact address, or contact (counterparty rule)
  — only coarse area + bucketed budget. The tutor's pitch is the only free text they add.
- **Microcopy emphasis:** "Applying does not book the job" is REQUIRED (Wei + AC-04 honesty).

---

## D. Tutor — "My Applications" (display vocabulary; SUPERSEDED = reassuring closure)

**Maps to:** product requirements document §3.7 AC-21, §20.5; design §4 tutor 2 + §2.2. **Persona:** Wei — "No longer available"
must read as closure, not a cold grey-out. **Route:** /tutor/applications (EXISTS — extend).

> The page already renders the application status label operation in a Badge and only allows Withdraw on
> APPLIED. M18 adds the SUPERSEDED label/variant + the interview pseudo-status + the warm SUPERSEDED
> treatment.


Implementation example omitted. This public edition retains the written requirements in this section.


**SUPERSEDED row treatment (Wei's explicit ask — reassuring, NOT cold):**


The illustration for d. tutor — my applications display vocabulary superseded reassuring closure contains the following information:

- Physics · Sec 4 Jurong [ No longer available ] <- gray Badge, but row NOT opacity-50'd.
- This job has been filled by another tutor. Thanks for <- warm line, FULL-contrast ink text.
- putting yourself forward — keep an eye on new jobs.
- [ Browse open jobs → ] <- forward action, not a dead end.


- NOT a faded/disabled row. Full-contrast text; only the status Badge is gray. The copy explains WHY
  (filled), thanks the tutor, and gives a forward action — the difference between rejection and closure.
- **Exact microcopy:**
  - SUPERSEDED pill: **"No longer available"**
  - SUPERSEDED body: **"This job has been filled by another tutor. Thanks for putting yourself forward —
    keep an eye on new jobs."**
  - MATCHED body: **"You won this job 🎉"** + View match →
  - Withdraw confirm toast: **"Application withdrawn."** (existing copy)
- **States:**
  - *Loading:* Spinner (existing) or skeleton rows.
  - *Empty:* Empty State icon is 📝" heading is No applications yet" description is Browse open jobs and
    express interest." action=Browse Requests → (existing pattern; note
    tutor job browse is /requests).
  - *Withdraw:* only on APPLIED/SHORTLISTED; existing confirm. SUPERSEDED/MATCHED/WITHDRAWN have no
    Withdraw.
  - *Error:* existing red alert + refresh.
- **Responsive:** desktop = status pill right-aligned (existing flex items-start justify-between);
  mobile = stacked, action full-width.
- **Accessibility:** status conveyed by the pill TEXT, not color alone (Wei's "not a grey-out" also
  satisfies WCAG color-independence). Each row ; actions are real buttons/links.
- **personal information boundary:** rows show only the job subject/level + coarse area — never the parent/child.

---

## E. Parent — "My Job → Applicants" (NEW personal information-safe applicant view + Request Interview / Choose)

**Maps to:** product requirements document §3.7 (parent applicant view + choose-tutor), §20.6 personal information contract, §13 AC-04; design §4
parent 2 + §3.2 + §5a. **Persona:** Priya (anxious parent) — wants to see who applied, vet, not feel
railroaded, see the fee before committing. **Route:** /tutee/requests/:job Id/applicants (owner-scoped;
403 if not the job owner — §20.6 IDOR).


Implementation example omitted. This public edition retains the written requirements in this section.


**"Choose this tutor" confirm Modal (Priya: explain it goes to admin; fee upfront):**


Implementation example omitted. This public edition retains the written requirements in this section.


- **Actions per applicant:**
  - Request interview (secondary/ghost) → existing M6 flow scoped to this job (date/mode → admin
    queue). the success operation
  - Choose tutor (primary) → confirm Modal → submit operation for tutee posts id select
    (§20.7). Creates a **selection request** (admin actions it). Does NOT itself match (AC-04).
- **States:**
  - *Loading:* skeleton applicant cards.
  - *Empty (no applicants yet):* Empty State icon is 📭" heading is No applicants yet" description is When
    tutors express interest in this job, they'll show up here. We'll notify you." (no action — passive
    wait).
  - *Already chose (selection pending):* chosen card shows Chosen · awaiting our team Badge
    (scheduled) + Change choice (mutable until admin confirms); other cards keep
    Request interview, and their Choose tutor becomes Choose instead.
  - *Job already assigned:* banner "This job has been assigned."; cards read-only (see §F).
  - *Error:* page retry; failed select → keep modal + toast error("Couldn't save your choice. Try
    again.")
  - *IDOR (not owner):* route guard → 403 page "You don't have access to this job."
- **Responsive:** desktop = applicant cards single-column ~max-w-3xl, actions inline-right; mobile =
  stacked, both actions full-width.
- **Accessibility:** handle is the card heading (level-three heading); each card is an article labelled with the applicant’s public handle; use buttons for actions and the shared confirmation dialog. Rating absence shown as "(no reviews
  yet)" text, not silence.
- **personal information boundary (CRITICAL — §20.6):** card shows **handle, subjects, levels, rate, COARSE area, YOE,
  PT/FT, rating, optional pitch** — and NOTHING ELSE. No real name, email, phone, exact location. This
  is a **distinct personal information-free data transfer object** (read operation for tutee posts id applicants), NOT the admin data transfer object.
  Reciprocally the tutor never saw the parent/child. Contact crosses ONLY after assignment (§F).
- **Microcopy:** Pii Notice (verbatim above). Count "n tutors have expressed interest." / singular "1
  tutor has expressed interest."
- **RESOLVED:** placement fee = **50% of one month's tuition, charged only after the first session**
  (backend provides the computed value from commission config; the modal shows the real statement — NEVER a
  literal "S$XX" placeholder, which reads as broken/hiding to a parent at the highest-intent moment).

---

## F. Parent — "Position filled" + assignment states (trust copy; contact only after commit)

**Maps to:** product requirements document §20.6 counterparty rule, §3.7 AC-21; design §4 parent 3–4 + §5a. **Persona:** Priya —
reassurance the deal is real and the fee is clear.

**State 1 — selection pending (parent chose, admin not yet confirmed):**


The illustration for f. parent — position filled + assignment states trust copy contact only after commit contains the following information:

- Mathematics · Sec 3 [ Chosen · awaiting our team ].
- You chose @maths wiz 88. Our team is confirming with them.
- You'll get their contact once it's confirmed.
- [ Change choice ].


**State 2 — assigned / position filled (admin closed the job = MATCHED):**


The illustration for f. parent — position filled + assignment states trust copy contact only after commit contains the following information:

- Mathematics · Sec 3 [ Position filled ] <- scheduled/blue.
- ✓ @maths wiz 88 is your tutor.
- Contact (shared now you've committed) <- contact ONLY appears here.
- Name: Wei Tan.
- Phone: [REDACTED PHONE].
- Email: [REDACTED EMAIL].
- Placement fee: 50% of one month's tuition — charged after <- real fee, post-session timing.
- your first session. We'll send payment details then.
- [ View match → ].


- **Annotated:** the **Contact** block is the FIRST and ONLY place the counterparty's real name/phone/
  email appears, gated on assignment (§20.6). Before this, every parent surface showed only the handle.
  Fee restated with "after your first session" to kill cost-surprise anxiety (Priya).
- **States:**
  - *Awaiting admin:* State 1 (mutable; Change choice).
  - *Assigned:* State 2 (contact revealed; Change choice gone).
  - *Cancelled / reopened (admin cancel → job ACTIVE again, §20.4):* banner "This assignment was
    cancelled — your job is open for applicants again." → back to §E list; contact block removed.
  - *Loading/Error:* skeleton / page retry.
- **Responsive:** contact is a bordered sub-card; mobile stacks label/value pairs.
- **Accessibility:** contact reveal announced aria-live is polite on assigned load; phone/email are
  tel / mailto links.
- **personal information boundary:** the deliberate personal information crossing point. Before = handle-only; here = full contact both
  directions (the tutor sees the parent's contact in their match view too).
- **Microcopy:** header "Contact (shared now you've committed)" makes the privacy model legible — Priya
  understands WHY she sees it now and not before.

---

## G. Admin — three surfaces (Approval / Interviews & Selections / Matching+Assignment + stale-commission)

**Maps to:** product requirements document §11.3, §6, §7, §8, §20.4/§20.8; design §4 admin + §3 + §10. **Persona:** Maya (ops) —
efficient queues, clear state, audited actions. Existing pages: /admin/matching, /admin/commission,
/admin/interviews, /admin/review.

### G1. Approval Queue (EXISTS — unchanged; listed for IA completeness)
Post review queue: approve/reject tutor + tutee posts. No change in M18.

### G2. Interviews & Selections (interview requests + NEW choose-tutor selection requests)
**Route:** /admin/interviews (extend) — two tabs.


Implementation example omitted. This public edition retains the written requirements in this section.


- "Liaise → close job (assign)" triggers the assign confirm (G3). The verb is **"close the job"** =
  assign (§20.4) — copy reflects ops mental model.
- **Selection-request count** in the tab = the tutor selections pending gauge made visible (§20.8).

### G3. Matching / Assignment (close is assign, activate, mark PAID, cancel/reopen + applicant pool)
**Route:** /admin/matching.


Implementation example omitted. This public edition retains the written requirements in this section.


**"Close job & assign" confirm Modal (the §20.4 atomic transition):**


The illustration for g3. matching / assignment close is assign, activate, mark paid, cancel/reopen + applicant pool contains the following information:

- Close job & assign to @maths wiz 88? [x].
- This will:.
- • Mark @maths wiz 88 as assigned (winner).
- • Set the job to "Position filled".
- • Notify the other 2 applicants their <- the SUPERSEDED fan-out (after-commit).
- applications are no longer available.
- This can be undone by cancelling the match.
- [ Cancel ] [ Close job & assign ].


- **Concurrency (§20.4 optimistic lock):** if another admin already assigned this job, the action 409s →
  the error operation + auto-refresh. Never a
  silent double-match. Use danger variant on irreversible-feeling actions.

### G4. Stale-PENDING commission view (revenue-leak safety net — §20.8, Seb)
**Route:** /admin/commission (add a filter/tab).


Implementation example omitted. This public edition retains the written requirements in this section.


- The human-facing twin of the commissions pending age alert (§20.8) — the only safety net for the
  manual "mark PAID" step. Sort by age desc; amber/red ⚠ on rows over threshold.

**Admin states (all G surfaces):** loading skeletons; per-queue empty ("No selection requests right
now."); error retry; success toasts (Job assigned., Marked as paid., "Match cancelled — job
reopened.").
**Responsive:** admin is desktop-primary (queues/tables); mobile = card-stacked rows, actions
full-width. **Accessibility:** tabs role is tablist + aria-selected; destructive actions (cancel,
close-job) use danger + confirm Modal; counts are text.
**personal information boundary:** admin **legitimately sees both real identities** (ops console, §20.6) — the ONE role
allowed full personal information. Still never renders a raw enum; still audited per existing ops rules.

---

## H. Desktop Register link (bug ④)

**Maps to:** product requirements document §20.10 bug ④; design §7. **Verified in Nav implementation reference:** the mobile drawer already has a
Register link → /register (line 106), but the **desktop logged-out cluster** (hidden md:flex,
lines 62–69) has only two Button variant is ghost items — **"Post a Request"** and **"Log In"** —
**both pointing at /login**, with NO register affordance. That is the gone-signup bug.


The illustration for h. desktop register link bug ④ contains the following information:

- Desktop nav (logged out) — BEFORE (verified):.
- InTuition Find a Tutor Find Tuition Jobs How It Works About [Post a Request] [Log In].
- (→/login) (→/login).
- Desktop nav (logged out) — AFTER:.
- InTuition Find a Tutor Find Tuition Jobs How It Works About [Log In] [ Sign up ].
- (→/login) ^ Link to '/register'.
- Button variant is primary size is sm.


- **Annotated:** add a “Sign up” link with a small primary button treatment
  to the desktop hidden md:flex auth cluster; recommend it leads visually (primary) while "Log In"
  stays ghost. The existing "Post a Request" ghost (→ /login) is the M17 intent CTA — keep, not in
  scope to remove here.
- **Responsive:** desktop-only (hidden md:flex); the mobile drawer already has Register → do NOT
  duplicate. **Accessibility:** use a link with the accessible name “Sign up”.
- **Microcopy:** nav label "Sign up" (destination page keeps M17's "Create account").
- **OPEN (UIUX):** "Sign up" vs "Register" vs "Create account" — recommend nav "Sign up", page "Create
  account". Consistency sign-off needed.

---

## Handoff to frontend

### Component inventory
**New components:**
- Tutor Identity Card — redesign of Tutor Card (subjects-first, no name/initials, + PT/FT row). (§B1)
- Commitment Badge — Badge wrapper rendering Part-time/Full-time from Tutor Commitment. (§A/§B/§E)
- Status Pill — wraps application Status Label/application Status Badge Variant (+ SUPERSEDED +
  interview pseudo-status) per the §0 table. (§D/§E/§F)
- Applicant Card — parent-facing personal information-safe applicant card + two actions. (§E)
- Choose Tutor Modal — confirm modal: next-steps + fee. (§E)
- Apply Interest Sheet — apply Modal / bottom-sheet (pitch + attach-listing). (§C)
- Pii Notice — inline trust strip ("names private until you choose"). (§E/§F)
- Contact Reveal Card — post-assignment counterparty contact block. (§F)
- Selection Request Queue — admin selection-requests tab. (§G2)
- Close Job Assign Modal — admin assign confirm. (§G3)
- Stale Commission List — admin stale-PENDING view + tab. (§G4)
- Profile Completeness Bar + apply-gate banner. (§A)

**Reused (no change):** Button, Badge, Modal using +Header/Body/Footer, Empty State, Spinner,
Skeleton, Star Rating, toast, Input, format Sgd Range/format Location/format Date.
**Modified existing:** Showcase Tutor Card (bug ① field rename + drop name + PT/FT), Tutor Card→
identity redesign, Nav (desktop Sign up), tutor Profile (3 new fields + completeness),
tutor Applications (SUPERSEDED label/variant + warm treatment + interview status), admin
Interview Queue/Matching/Commission pages, application Status Label/application Status Badge Variant
mappers.

### New / changed routes (pages)
| Route | Page | New? |
|---|---|---|
| /tutee/requests/:job Id/applicants | Parent applicant view (§E) | NEW |
| /tutor/profile | + PT/FT + rate + area + completeness (§A) | CHANGED |
| /assignments/:id | + Apply/Express-interest CTA + sheet (§C) | CHANGED |
| /tutor/applications | + SUPERSEDED + interview status + warm treatment (§D) | CHANGED |
| /admin/interviews | + Selection-requests tab (§G2) | CHANGED |
| /admin/matching | + close-job/assign + cancel/reopen (§G3) | CHANGED |
| /admin/commission | + stale-PENDING tab (§G4) | CHANGED |
| /tutors, / showcase | nameless redesigned cards (§B) | CHANGED |
| nav | desktop Sign up (§H) | CHANGED |

### application programming interface shapes the screens need (cross-ref product requirements document §20.7)
- read operation for tutee posts id applicants — **NEW, personal information-safe Applicant Card Response** (backend §5.1).
  AUTHORITATIVE shape:  id, tutor Public Handle, job Subject, job Level, hourly Rate Min Sgd,
  hourly Rate Max Sgd, coarse Area, years Experience, commitment, rating|null, review Count,
  application Status, applied At, pitch?|null, also Teaches?[] . Lead subject/level = the **applied-to
  job's** job Subject/job Level; also Teaches = attached-listing subjects (NO structured
  attached Listing object; rate/area override applied server-side into the flat fields). NO
  name/email/phone/exact-location. Paged (Page Response, size 20). Owner-scoped (403). Contract test (§20.6).
- submit operation for tutee posts id select — **NEW** selection request (§E). tutor Id is a
  **unique identifier string**. Mutable (re-select = 200 update, create = 201); owner-scoped; does NOT create a
  match (AC-04).
- read operation for matches id contact — **NEW** (backend §13 D3). Powers the §F contact block. Participant-
  only; returns counterparty name, phone number and email address; gated server-side on assignment (MATCHED/ACTIVE/
  COMPLETED); 404 before assignment and after cancel (contact revoked both sides). The §E Choose
  step does NOT call this — contact reveals only after admin-confirmed assignment.
- GET tutor My-Applications list — each row carries has Interview (bool) + interview Id (nullable
  unique identifier) precomputed server-side (backend §13 D2), so §D's "Interview scheduled" pill is not an N+1.
- submit operation for tutor applications — apply (free-market, no listing;
  AC-20). (§C)
- Tutor profile PUT — add commitment, hourly Rate Min Sgd/Max, location Region using /Area. (§A)
- Public tutor card / showcase data transfer object — must carry hourly Rate Min Sgd/Max (fix ①) + commitment; must NOT
  carry name. (§B)
- submit operation for admin matches — extended = atomic assign (§20.4); returns updated job/app states; may
  409 on concurrent assign. (§G3)
- submit operation for admin commissions id pay — mark PAID (existing). (§G3/§G4)
- Post-assignment match view — exposes counterparty contact. (§F)

### Status / enums the user interface must render (always via the §0 mapper — never raw)
- Job Application Status: APPLIED, SHORTLISTED, MATCHED, SUPERSEDED, REJECTED, WITHDRAWN
  (via JOB APPLICATION STATUS constant).
- Post Status (job): the matched operation (others admin-only).
- Tutor Commitment: PART TIME, FULL TIME.
- Commission: PENDING, PAID (+ stale = PENDING past age threshold, derived).

---

## OPEN (UIUX) items — for team-review / frontend-backend to resolve
1. **Area granularity** (§A): single coarse region select control vs region→planning-area cascade. Recommend
   coarse region for v1 (personal information). Design §5 has both columns.
2. **Placement-fee display** (§E/§F): exact value + flat-vs-% format from commission config; modal shows
   a placeholder until backend confirms.
3. **Nav label** (§H): "Sign up" vs "Register" vs "Create account" — recommend nav "Sign up",
   destination keeps "Create account" (M17). Needs consistency sign-off.
4. **Handle source** (§E): tutors have display Name (admin/self) — what is the public **@handle**? If no
   handle field exists today, backend must derive/add one (anonymised, stable). Possible schema gap (design
   §5 lists profile columns but no public handle).
5. **Stale threshold** (§G4): wireframe uses "threshold days"; must equal the commissions pending age
   alert threshold (§20.8) — single source.
6. **Assigned vs interview Badge** (§0): both currently scheduled-blue/green-share; decide whether to
   add a dedicated assigned-green Badge variant.

---

## FOUNDER DECISIONS RESOLVED (2026-05-31) — supersedes the OPEN table above

| # | Item | RESOLUTION |
|---|---|---|
| 4 | Public handle source | **NEW real field tutor profiles public handle** (founder; backend spec §12 #8). The card/applicant view/showcase render handle (e g. "Mr Tan T."), NEVER display Name. Not a schema gap anymore — backend adds the column + setup-form input. Profile setup (§A) gains a **required "Public handle" text input** with helper copy: "Shown publicly instead of your full name." |
| 1 | Area granularity | **Coarse region select control** for v1 (personal information). Single dropdown, no cascade. |
| 3 | Nav label | Nav button reads **"Sign up"**, destination page keeps "Create account" (M17). |
| 5 | Stale threshold | **14 days** — single source shared with backend commissions pending age alert (backend spec §12 #10). Replace "threshold" in the §G4 wireframe with "14 days". |
| 2 | Placement-fee display | Show the real commission value from config (backend provides it); modal copy: fee is **50% of one month's tuition**, **charged only after the first session** (Priya's "fee clear before commit" + reuses §8 commission model). No flat placeholder. |
| 6 | Assigned Badge | **Add a dedicated "Assigned" green Badge variant**, distinct from interview-blue, so "Assigned to you" reads as a win, not a schedule. |

> **Card identity (cross-spec):** with the real handle field, the redesigned card (§B) leads
> with **handle + subjects** (handle is personal information-safe). The applicant card (§E) shows handle + subject/level
> **of the applied-to job** + rate/area/YOE/PT-FT/rating (backend spec §12 #7) — never the real name, phone,
> or exact location until the parent commits.

---

## TEAM-REVIEW RESOLVED (2026-05-31) — 10 sub-agent reviewers + Boss

Verdicts: Quinn APPROVE; Jamie/Sam/Casey/Dana/Reese/Wei/Riley APPROVE-WITH-CONDITIONS; **Chris SEND
BACK** (applicant-card contract divergence). Priya (parent): would trust + use it. All conditions
resolved below; binding for the frontend spec. (backend-side items mirrored into assignment-be-spec md §13.)

### Blockers fixed
- **B1 — §E applicant card reconciled to the backend Applicant Card Response contract (Chris/Riley/Reese).**
  The §E wireframe + Handoff data transfer object are AUTHORITATIVELY the backend §5.1 shape:
   id, tutor Public Handle, coarse Area, job Subject, job Level, hourly Rate Min Sgd/Max, years Experience,
  commitment, rating, review Count, application Status, applied At, pitch?, also Teaches?[] .
  - Lead subject/level on the card = **the applied-to job's** job Subject/job Level (NOT a tutor
    multi-subject profile). If the applicant attached a listing, show that listing's subjects as a
    secondary **"also teaches"** line (also Teaches) — there is **no structured attached Listing
    object**; rate/area override is applied server-side and arrives in the flat hourly Rate*/coarse Area.
  - Field name is **coarse Area**, not area. The card MAY render application Status as a small chip
    (Applied/Shortlisted) — it is returned; rendering it is a SHOULD.
  - **backend delta:** add pitch (the optional applicant message) to Applicant Card Response so §E's pitch
    line has a source (mirrored to backend §13).
- **B2 — handle personal information-leak validation = MUST (Casey/Jamie).** §A "Public handle" input gets
  server-side validation: reject values matching SG phone an eight-digit Singapore mobile number beginning with 8 or 9, email/URL patterns; length cap
  (≤40); uniqueness; profanity screen. Helper microcopy: "A short public name (e g. 'Mr Tan T.') —
  not your full name, phone, or email." Enforcement is server-side (backend TR4), the form mirrors it.
- **B3 — profile-completeness fields must MATCH the real apply gate (Wei).** The gate (backend §12 #5) is
  **public handle + commitment + rate range + area + qualifications**. The §A form *-required set
  and the completeness bar use EXACTLY these five. **qualifications is required (add the *);
  years-of-experience is NOT a gate** (drop its *). No "100% then bounced at Apply."

### Conditions (frontend-spec binding)
- **C1 — §0 display-vocab table reconciled in place (Jamie/Sam):** MATCHED→"Assigned to you"→a NEW
  dedicated **assigned Badge variant** (distinct emerald token, NOT the existing live/completed
  green which are byte-identical). Update the §0 table itself; delete the superseded OPEN note.
- **C2 — backend-first / types-first sequencing note (Sam):** SUPERSEDED, commitment/Tutor Commitment,
  handle exist in **neither frontend nor backend today**. application Status Label/…Badge Variant are
  exhaustive Record<Job Application Status,… — adding SUPERSEDED is a compile-breaking cross-layer
  change. frontend §D/§A/§B/§E build **after** backend ships the enum + columns + data transfer object. State this in the frontend spec.
- **C3 — Status Pill real prop shape (Sam/Reese/Riley):** the "Interview scheduled" pseudo-status is
  NOT a Job Application Status. Status Pill takes both the application status and whether an interview exists. **backend delta:** the
  My-Applications list data transfer object carries a precomputed has Interview/interview Id (kills the §D N+1).
- **C4 — §F contact-reveal needs an owning endpoint + data transfer object (Riley):** the post-assignment counterparty
  contact block (name/phone/email, gated server-side on MATCHED, revoked on cancel — both sides) is a
  real application programming interface the wireframe needs. **backend delta:** specify it (mirrored to backend §13). Selection (Choose)
  must NOT reveal contact — reveal only after admin-confirmed MATCHED (mutable-selection harvest guard).
- **C5 — showcase bug ① is a data transfer object-vs-type contract bug (Sam):** Showcase Tutor type currently declares
  rate Min Sgd/Max while the wire sends hourly Rate*. Fix the type AND update the showcase tests that
  mock the wrong shape (they're green while prod is broken).
- **C6 — Modal label Id/Header id wiring** on every confirm modal (§C/§E/§G3); add aria-live
  polite region for §E/§F selection-state transitions (Jamie/Sam).
- **C7 — §E pagination (Riley/Reese):** applicant list is paged (Page Response, size 20); wireframe
  the pager + the "chose a tutor who's on another page" case + the chosen card's full action set
  (Request-interview button: removed once chosen).
- **C8 — admin button copy (Jamie):** "close job" → **"Assign tutor & close job"** (don't let it read
  as delete-the-posting). §G4 stale-commission: draw the **empty** state ("No stale commissions").
- **C9 — tutor Id is a unique identifier string** everywhere (backend §5.2 123 is a stub) (Chris/Riley).

### Trust copy (Priya) — MUST appear on the parent surfaces
- **Show the REAL fee number**, not "S$XX" (backend provides it) — a literal "S$XX" reads as broken/hiding.
- **Mirror personal information sentence on §E:** "Your contact details stay private before assignment. After assignment and confirmation, you and your tutor can access each other's contact details." Apply participant authorization and the ACTIVE/COMPLETED frontend gate; post approval alone does not release contact.
- **"Reviewed by our team" reassurance** (NOT "verified" — there is no identity-verification
  workflow today; backend §13 D6) so a handle reads as safe, not anonymous-scary.

### Deferred (not v1 blockers)
- Dana: explicit GIVEN/WHEN/THEN ACs + surface the §8.3 first-session-confirmation touchpoint on the
  parent fee journey → fold into frontend spec.
- Quinn: the optional "attach a listing" affordance is a *want* — keep, but pressure-test post-v1.
- Jamie: SUPERSEDED/WITHDRAWN both render grey pills (text-differentiated) — known limitation, v1-ok.
