# M18 — Assignment Lifecycle & Free Marketplace — Frontend Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

Preserve the folded assignment user experience decisions. Apply [012](../012-tutor-apply-and-profile-ux/design.md) for server-computed profile Complete/missing Fields (delete independent frontend gate logic), [017](../017-admin-analytics-and-financial-safety/requirements.md) and 019 for explicit agency-receipt confirmation, and INT-157 for the immutable plan display. Earlier source-code examples do not waive first-session confirmation or define new policy.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Module:** M18 · **Layer:** Frontend (React 19 / Vite / TypeScript SPA)
**Status:** ✅ team-reviewed (9 real sub-agents) + all 6 OPEN-frontend resolved (backend follow-up 0030 shipped & verified) + Round-2 conditions §8a folded in → **APPROVED to build** (2026-06-01)
**Owner:** Frontend Tech Lead
**Date:** 2026-06-01

**Source of truth (read in full before building):**
1. specs/uiux/assignment-uiux-spec md — the Boss-cleared UIUX spec (areas A–H + FOUNDER DECISIONS
   RESOLVED + TEAM-REVIEW RESOLVED C1–C9 + Handoff). Every screen/state/microcopy is there; this frontend
   spec implements it 1:1 and never invents copy.
2. specs/architecture/assignment-be-spec md §5 + §12 + §13 — the LIVE backend contracts. The frontend
   matches these EXACTLY (field names, unique identifier ids, status codes, error envelope a structured error containing its error code).
3. product requirements document §20.

> **Conventions (binding, from CLAUDE md):** No magic strings — all application programming interface paths via application programming interface ROUTES
> (api Routes implementation reference), all enum values via typed unions + the status implementation reference registries.
> State = Tan Stack Query (use Query/use Mutation) + zustand auth Store. Coverage gate
> **≥97.5% line / ≥88% branch (Vitest the documented operational check)**. **the documented operational check (tsc) MUST pass** — type
> errors break the Docker build; the SUPERSEDED exhaustive-Record change is the prime build risk.
> ESLint must pass (the documented operational check). Reuse the existing design system — NO new design language.

---

## 0. Sequencing & build risk (TEAM-REVIEW C2 — read first)

SUPERSEDED, commitment/Tutor Commitment, handle, has Interview, the parent applicant
endpoint, and the contact-reveal endpoint exist in **neither frontend nor backend before M18**. Per backend spec these
ship in the live backend M18 release. **The frontend work in §A/§B/§D/§E/§F builds only AFTER the backend has
shipped** the enum + columns + data transfer objects. The contract-breaking moment is purely client-side:

application Status Label / application Status Badge Variant (format implementation reference) are **exhaustive
Record<Job Application Status, …**. The instant SUPERSEDED is added to the Job Application Status
union (index implementation reference) **and** the JOB APPLICATION STATUS constant (status implementation reference),
TypeScript fails the build on those two Records until both gain a SUPERSEDED key. **All three edits
land in one commit** (union → constant → both Records) or tsc (hence the documented operational check and the Docker
build) is red. This is the single most important ordering rule in this spec.

---

## 1. TypeScript type changes (index implementation reference) — the breaking set

| # | Change | File | Breaking? |
|---|---|---|---|
| T1 | Add SUPERSEDED to the Job Application Status union | index implementation reference | **YES** — forces T2 + both Records (§3) |
| T2 | Add SUPERSEDED: SUPERSEDED to JOB APPLICATION STATUS | status implementation reference | **YES** — Record<Job Application Status,… exhaustiveness |
| T3 | New Tutor Commitment PART TIME \| FULL TIME union + TUTOR COMMITMENT constant registry | index implementation reference + status implementation reference | no (additive) |
| T4 | **Bug ① fix:** Showcase Tutor rate Min Sgd/rate Max Sgd → hourly Rate Min Sgd/hourly Rate Max Sgd; add commitment?: Tutor Commitment \| no value; **remove tutor Display Name** (personal information leak) | index implementation reference | **YES** — Showcase Tutor Card implementation reference + its tests reference the old names |
| T5 | Tutor Post: add commitment?: Tutor Commitment \| no value; add Handle?: string \| no value (the public-card identity) | index implementation reference | no (additive/optional) |
| T6 | Job Application (My-Applications row): add has Interview: boolean + interview Id: string \| no value (backend §13 D2) | index implementation reference | no (additive) |
| T7 | New Applicant Card Response interface — backend §5.1 shape (see §5.E) | index implementation reference | no (new) |
| T8 | New Selection Request Response application identifier, tuition-request identifier, tutor identifier, status and creation time (backend §5.2) | index implementation reference | no (new) |
| T9 | New Contact Reveal Response the other participant’s name, phone and email (backend §13 D3) | index implementation reference | no (new) |
| T10 | New Selection Request Status CREATED \| CONSUMED \| SUPERSEDED \| EXPIRED | index implementation reference | no (new) |

> **Field-name discipline:** the applicant data transfer object field names are AUTHORITATIVELY (backend §5.1/§13 D1)
> tutor Public Handle, coarse Area, job Subject, job Level, hourly Rate Min Sgd/Max, pitch,
> also Teaches — NOT handle/area/subjects/levels. tutor Id is a **unique identifier string**
> everywhere (C9 / backend D5) — never a number.

---

## 2. application programming interface ROUTES additions (api Routes implementation reference) — no magic strings


Implementation example omitted. This public edition retains the written requirements in this section.


Reused as-is: applications create (submit operation for tutor applications, §5.C), applications my List
(§5.D), applications withdraw, profile update Tutor (PUT, §5.A), admin matches,
admin match Cancel, admin commission Pay, interviews admin List, public showcase Tutors,
tutors detail. No new admin route constants are needed: the stale-commission view is a **filter on
the existing commission Pay/commission list** (?status is PENDING&older Than Days is 14, backend §12 #12) and the
selection-request queue reads from the extended interviews admin List surface (G2) — confirm under
OPEN-frontend-3.

---

## 3. Display-vocabulary mappers (format implementation reference) — extend, never replace

Both mappers are exhaustive Record<Job Application Status, …; they MUST gain a SUPERSEDED key in the
same commit as T1/T2 (§0). Final state matching the UIUX §0 table:


Implementation example omitted. This public edition retains the written requirements in this section.


Changes vs current code: MATCHED label Matched→Assigned to you, variant live→assigned
(NEW variant, §4); REJECTED label Not Selected→Not selected, variant rejected→draft;
add SUPERSEDED. Add a the commitment label operation → PART TIME→Part-time,
FULL TIME→Full-time mapper (exhaustive Record<Tutor Commitment,string>). The "Interview scheduled"
state is **NOT** a Job Application Status — it is derived from has Interview inside Status Pill (§4),
not added to these Records.

---

## 4. Component inventory

### 4.1 NEW components

| Component | Purpose | Props (shape) | UIUX |
|---|---|---|---|
| **assigned Badge variant** | emerald token distinct from live/completed (byte-identical green) so "Assigned to you" reads as a win | extends Badge Variant union in Badge implementation reference + variant Classes (bg-emerald-100 text-emerald-800) | §0/C1 |
| Status Pill | the single display-vocab renderer | application status and an optional indication that an interview exists — if an interview exists and the application is applied or shortlisted → render "Interview scheduled" scheduled pill; else delegate to the mappers (§3) | §D/§E/§F, C3 |
| Commitment Badge | PT/FT chip | an optional part-time or full-time tutoring commitment — renders nothing when null (back-fill safe) | §A/§B/§E |
| Tutor Identity Card | redesigned /tutors browse card (replaces Tutor Card body) | the tutor listing and whether it is saved as a favourite | §B1 |
| Applicant Card | parent-facing personal information-safe applicant card + 2 actions | the applicant, chosen and pending states, and actions to request an interview or choose the tutor | §E |
| Choose Tutor Modal | confirm modal: next-steps + real fee | whether the dialog is open, close and confirm actions, the tutor handle and submission state | §E |
| Apply Interest Sheet | apply Modal/bottom-sheet (pitch + attach-listing) | whether the dialog is open, close and submit actions, an optional pitch and listing choice, available listings and submission state | §C |
| Pii Notice | inline trust strip (bidirectional privacy copy) | the content displayed inside the component or fixed-copy variant | §E/§F |
| Contact Reveal Card | post-assignment counterparty contact block | the authorised contact details | §F |
| Profile Completeness Bar | progress bar + apply-gate banner | the completion percentage and missing profile fields (+ <Apply Gate Banner/ companion) | §A/§C |
| Selection Request Queue | admin selection-requests tab content | selection requests and actions to view applicants or assign a tutor | §G2 |
| Close Job Assign Modal | admin "Assign tutor & close job" confirm (danger) | dialog state, close and confirm actions, tutor handle, other-applicant count and submission state | §G3 |
| Stale Commission List | admin stale-PENDING (>14d) tab + rows | the list items | §G4 |

### 4.2 REUSED unchanged
Button, Modal using +.Header/.Body/.Footer, Empty State, Spinner, Skeleton, Star Rating,
Input, toast. format Sgd Range, format Date, format Phone, subject Label, level Label,
format Location. Native select control (there is **no Select user interface primitive** — confirmed; forms use
native select controls per the accessibility notes). Forms use react-hook-form + zod + @hookform/resolvers/zod
(the Profile pattern).

### 4.3 MODIFIED existing
Badge implementation reference (+assigned), format implementation reference (mappers + commitment Label, §3), Showcase Tutor Card implementation reference
(bug ① + drop name + PT/FT, §B2), Tutor Card implementation reference→Tutor Identity Card (§B1), Nav implementation reference (desktop Sign up,
§H), Profile implementation reference (§A), Applications implementation reference (§D), Assignment Detail implementation reference
(§C), Interview Queue implementation reference (§G2), Matching implementation reference (§G3),
Commission implementation reference (§G4), index implementation reference, status implementation reference.

---

## 5. Per-screen specs

Each screen lists: route · components · states (loading/empty/error) · Tan Stack query keys + mutations ·
a11y · wireframe mapping.

### 5.A — Tutor profile setup/edit (UIUX §A) · route /tutor/profile (CHANGED)

**Components:** extend Edit Profile Section in Profile implementation reference. New fields above the existing
ones: **Public handle** (input field *), **Commitment** (native select control *, placeholder "Select…",
options Part-time/Full-time), **Hourly rate min/max** (two numeric input fields on one row,
*), **Area (coarse region)** (native select control *, the 5 Location Region values). qualifications
keeps its *. **Years of experience loses its *** (NOT a gate). Add a profile-completeness bar at the
top + the application eligibility banner companion.

- **Required/gate set (B3, exactly five):** Handle, commitment, hourly Rate Min Sgd +
  hourly Rate Max Sgd, location Region, qualifications. Completeness % = (filled gate fields / 5)·100.
  YOE, bio, display name, featured Consent are NOT counted.
- **zod schema additions:** the enum operation/Max
  coerced ints (S$10–1000), the refine operation;
  the enum operation. **Client validation
  mirrors the server but the server is authoritative** (D4) — do not trust the client gate alone.
- **PUT body** → application programming interface ROUTES profile.update Tutor adds Handle, commitment, hourly Rate Min Sgd,
  hourly Rate Max Sgd, location Region to the existing display Name/qualifications/YOE/bio payload.
- **Handle validation errors (server, D4):** map the error envelope code on save failure:
  422 INVALID PUBLIC HANDLE → inline field error "Use a short public name — not your full name,
  phone, email, or a link."; 409 PUBLIC HANDLE TAKEN → "That handle's taken. Try another." Keep
  entered values, set the error on the Handle field (set Error), no toast for these two.
- **States:** *loading* skeleton rows, Save disabled; *first-time* completeness 0% + banner;
  *inline validation* per copy above (rate, commitment, required-empty "This helps parents decide —
  please fill it in."); *save error (generic)* the error operation,
  keep values; *success* the success operation, bar animates, banner disappears if gate met.
- **Query/mutations:** the use query operation (existing) now returns the new fields; the save
  use Mutation invalidates tutor-profile + auth Keys me. The completeness bar reads the query
  data (or live form values for instant feedback).
- **a11y:** every field has a label associated with its control; native select control for commitment + region; aria-required;
  inline errors via aria-describedby + aria-invalid; bar role is progressbar aria-valuenow/min/max.
- **Apply-gate banner** is reused on §C and §D when the profile is incomplete (surface the requirement
  **before** the Apply button, TR10 — never bounce at apply with an error).

### 5.B — Public tutor cards (UIUX §B) · /tutors + landing showcase (CHANGED)

**B1 Tutor Identity Card** (/tutors): subjects-first lead (subject Label), levels muted
(level Label), then the 2-col grid Rate (the format sgd range operation) / Area
(format Location) / Experience / **Commitment** (commitment badge). **Remove** the initials avatar
block (Tutor Card implementation reference-129) and any name node. Keep the favourite heart (TUTEE-only, aria-pressed,
existing optimistic mutation) and its query wiring untouched.

**B2 Showcase Tutor Card** (landing strip): **bug ① fix** — change format Sgd Range(tutor rate Min Sgd,
tutor rate Max Sgd)line 69 →(tutor hourly Rate Min Sgd, tutor hourly Rate Max Sgd); **remove** the
initials block (lines 26-31, 42-47) and the the tutor’s display name line (line 75); add
commitment badge next to the existing "Available" live Badge. **C5: update the showcase tests** —
the mock fixtures currently set rate Min Sgd/Max; they are green while prod renders "S$undefined". Fix
the fixtures to hourly Rate Min Sgd/Max, drop the name assertion, add a PT/FT assertion.

- **States (both):** *loading* Skeleton cards ×N; *empty (browse)* Empty State heading is No tutors
  match yet" description is Try widening your filters." action=Clear filters control; *no rating* hide
  Star Rating (existing average Rating equals no value guard); *missing PT/FT* commitment badge renders
  nothing (never "undefined"); *error* page-level retry.
- **Query keys:** unchanged — the list operation (browse), the tutors operation (strip).
- **a11y:** no name → aria-label composed from subjects + area + rate, e g. "Mathematics, Physics
  tutor in Central, S$30–45 per hour"; PT/FT meaning is in the chip TEXT (colour decorative).

### 5.C — Tutor job-detail Apply / Express interest (UIUX §C) · /assignments/:id (CHANGED, login-gated)

**Components:** primary CTA Apply / Express interest (sticky on mobile) opening Apply Interest Sheet
(Modal max-w-md desktop / full-width bottom sheet mobile). Sheet body: REQUIRED honesty line
"Applying does not book the job — the parent and our team decide next steps." (AC-04); optional pitch
a multi-line text field with aria-live is polite char counter; optional **Attach a listing** native select control of
the tutor's OWN active Tutor Posts ("none" = profile values used; helper: attaching overrides
rate/area on the parent's card).

- **Mutation:** POST application programming interface ROUTES applications.create body the tuition-request identifier, optional message and optional tutor-listing identifier.
  On 422 TUTOR PROFILE INCOMPLETE → close sheet, route to/scroll the apply-gate banner (don't show a
  raw error) — but primarily this is **pre-gated**: if the profile is incomplete the CTA is replaced by
  the application eligibility banner Complete profile → so the 422 is the belt-and-braces fallback.
- **States:** *already applied* → disabled secondary ✓ Interest sent + "View in My
  Applications" link; *profile incomplete* → gate banner instead of CTA; *job not ACTIVE* → CTA
  disabled, status flips to "Position filled", inline "This job has been assigned." (mirrors
  422 TUTEE POST NOT ACTIVE); *submitting* → Send interest loading; *success* → close,
  the success operation, CTA → "Interest sent";
  *error* → keep sheet open, the error operation
- **Query keys:** the detail uses the existing assignment-detail query; the tutor's own active listings
  for the listing selection control via the mine operation (existing). On success invalidate application Keys all.
- **a11y:** sheet = shared Modal (focus-trap, Esc, label Id + Modal Header id, C6); char counter
  aria-live is polite; attach = native select control.

### 5.D — Tutor My Applications (UIUX §D) · /tutor/applications (CHANGED)

**Components:** replace the inline badge with **status indicator based on the application status and interview flag**. Add per-status bodies/actions:
- MATCHED → body "You won this job 🎉" + View match → (link to /tutor/matches).
- SUPERSEDED → **warm treatment, NOT a grey-out:** row keeps full-contrast ink (no opacity-50);
  only the pill is gray; body "This job has been filled by another tutor. Thanks for putting yourself
  forward — keep an eye on new jobs." + Browse open jobs → (→ /requests).
- has Interview (APPLIED/SHORTLISTED) → pill "Interview scheduled" + View interview →
  (/tutor/interviews, using interview Id when present).
- Withdraw remains, gated to APPLIED/SHORTLISTED only (existing confirm + "Application withdrawn.").
  SUPERSEDED/MATCHED/WITHDRAWN have no Withdraw.

- **States:** *loading* Spinner/skeleton (existing); *empty* Empty State icon is 📝" heading is No
  applications yet" description is Browse open jobs and express interest." action=
  Browse Requests →; *error* existing red alert + refresh.
- **Query/mutations:** unchanged — the application-list cache entry for the chosen page and size (the row data transfer object now carries
  has Interview/interview Id, no extra query, no N+1, C3/D2); withdraw mutation unchanged.
- **a11y:** status conveyed by pill TEXT not colour (also satisfies the "not a grey-out" WCAG ask); each
  row ; actions real buttons/links.

### 5.E — Parent My Job → Applicants (UIUX §E) · /tutee/requests/:job Id/applicants (NEW)

**Route (App implementation reference):** add under Tutee Layout children:
the tutee’s applicants screen for a specific tuition request → new page
Applicants implementation reference. Owner-scope is **server-enforced** (403 IDOR, D7) — render a 403 page on
403, not a client route guard.

**Applicant Card Response (T7 / backend §5.1) — AUTHORITATIVE shape:**


Implementation example omitted. This public edition retains the written requirements in this section.


**Applicant Card:** heading = tutor Public Handle (level-three heading); lead line = subject Label using job Subject ·
the readable level label. The next row shows the price range, broad area and years of experience; show a star rating when
rating differs from no value and review Count>0 else "(no reviews yet)"; commitment badge; optional pitch (quoted);
optional "Also teaches: also Teaches join using ". MAY render application Status as a small chip via
status indicator (SHOULD). Two actions: Request interview (secondary/ghost) + Choose tutor
(primary).

**Pii Notice strip** (verbatim): "The tutor never sees your name or phone number. You'll see each
tutor's contact only after you choose one and our team confirms. All tutors are **reviewed by our
team**." ("reviewed", NOT "verified", D6.)

**Choose Tutor Modal** (confirm before select): next-steps bullets + **Fee** block with the REAL
statement "One-time placement fee: 50% of one month's tuition, charged only AFTER your first session.
No charge before that." (NEVER a literal "S$XX") + "You can change your choice until our team confirms."
Footer Not yet Choose tutor.

- **Mutations:** Choose → the post select operation body the tutor-account identifier (**unique identifier
  string**); 201 create / 200 re-choose — treat both as success (does NOT create a match, AC-04).
  Request interview → existing M6 interview-create flow scoped to this job; toast success("Interview
  requested. We'll confirm a time."). Choose success →toast success-style confirmation + invalidate
  the applicants query so the chosen card flips.
- **Pagination (C7):** Page Response, size 20; reuse the Applications implementation reference pager pattern
  (← Previous / Page n of m / Next →, aria-label). Handle "chose a tutor on another page": the
  chosen state is keyed by selection-request, not by the visible page.
- **States:** *loading* skeleton applicant cards; *empty* Empty State icon is 📭" heading is No applicants
  yet" description is When tutors express interest in this job, they'll show up here. We'll notify you."
  (no action); *already chose (selection pending)* chosen card shows Chosen · awaiting our team
  scheduled Badge + Change choice, **its Request-interview button is removed** (C7), other
  cards' Choose tutor → Choose instead; *job already assigned* banner "This job has been assigned.",
  cards read-only (→ §F); *select error* keep modal + toast error("Couldn't save your choice. Try
  again."); *403 IDOR* "You don't have access to this job." page.
- **Count copy:** "n tutors have expressed interest." / singular "1 tutor has expressed interest."
- **Query keys (NEW):** add to query Keys implementation reference an applicant Keys namespace, e g.
  the applicant-list cache entry for that job, page and size and the selection operation. Reuse
  the by tutee post operation for invalidation cross-refs if appropriate.
- **a11y:** each applicant card is labelled with its public handle; the handle is a level-three heading; actions real buttons; modal
  label Id/Header id (C6); add an aria-live is polite region announcing selection-state transitions
  (C6).
- **personal information (CRITICAL §20.6):** this is the **distinct personal information-free data transfer object** — handle, subjects, levels, rate,
  COARSE area, YOE, PT/FT, rating, optional pitch ONLY. Never render name/email/phone/exact location;
  the type has no such fields (backend contract test guards it server-side).

### 5.F — Parent assignment + contact reveal (UIUX §F)

Surfaced on the parent's job/match view (the /tutee/matches detail or the applicants page when the job
is MATCHED). Two states:

- **State 1 — selection pending:** Chosen · awaiting our team scheduled Badge + "You chose
  handle. Our team is confirming with them. You'll get their contact once it's confirmed." +
  Change choice. **No contact block. Choose does NOT call the contact endpoint.**
- **State 2 — assigned (MATCHED):** Position filled scheduled Badge + "✓ handle is your
  tutor." + **contact-reveal card** + Fee restatement "Placement fee: 50% of one month's tuition —
  charged after your first session. We'll send payment details then." + View match →.

**Contact Reveal Card:** header "Contact (shared now you've committed)"; renders
counterparty Name, counterparty Phone (tel link, format Phone), counterparty Email (mailto).

- **Mutation/Query:** contact via the use query operation-adjacent or a dedicated
  the by match operation) → the contact operation. **Only enabled when the
  job/match is in an assigned state** (enabled: is Assigned) so it isn't fetched pre-assignment (backend
  returns 404 pre-assignment + post-cancel — treat 404 as "not revealable yet", not an error toast).
- **States:** *awaiting admin* State 1; *assigned* State 2; *cancelled/reopened* banner "This assignment
  was cancelled — your job is open for applicants again." → back to §E list, contact removed;
  *loading/error* skeleton / page retry (404 handled as above, not a red error).
- **a11y:** contact reveal announced via aria-live is polite on assigned load; phone/email are real
  tel/mailto links.

### 5.G — Admin surfaces (UIUX §G)

**G2 Interviews & Selections** · /admin/interviews (CHANGED): add a tablist
Interview requests n Selection requests m (role is tablist + aria-selected). New
Selection Request Queue renders rows: "Job: subject·level (#id) ACTIVE / Parent chose: handle
(changed Xm ago) / Requested Xh ago" + View applicants Assign tutor & close job (C8 copy —
NOT "close job" alone). Counts are TEXT (the tutor selections pending gauge made visible).

**G3 Matching / Assignment** · /admin/matching (CHANGED): per-job row shows Applicants using n + Selection
handle + View applicant pool Assign tutor & close job; MATCHED rows show Tutor/Match/
Commission state + Mark PAID Cancel & reopen. **Close Job Assign Modal** (danger,
confirm): bullets "Mark handle as assigned (winner). / Set the job to Position filled. / Notify the
other n applicants their applications are no longer available. / This can be undone by cancelling the
match." Footer Cancel Assign tutor & close job.
- **Concurrency (409):** assign → POST application programming interface ROUTES admin.matches; on 409 JOB ALREADY ASSIGNED
  the error operation + invalidate/refetch — never
  a silent double-match. Cancel → application programming interface ROUTES admin.match Cancel. Mark PAID → admin commission Pay.
- **Success toasts:** "Job assigned." / "Marked as paid." / "Match cancelled — job reopened."

**G4 Stale commission** · /admin/commission (CHANGED): add a filter tab All ⚠ Stale n.
Stale Commission List = PENDING past **14 days** (single source, backend §12 #10), via the existing
commission list query with ?status is PENDING&older Than Days is 14; sort age desc, amber/red ⚠.
**Draw the empty state (C8):** Empty State heading is No stale commissions.

- **Query keys:** interview Keys (selections tab — confirm shape under OPEN-frontend-3),
  pending commissions older than fourteen days (stale tab).
- **a11y:** tabs role is tablist + aria-selected; destructive actions danger + confirm Modal
  (label Id, C6); counts are text.
- **personal information:** admin legitimately sees both real identities (ops console) — still render via mappers, never
  a raw enum.

### 5.H — Desktop Register/Sign up link (UIUX §H, bug ④) · Nav implementation reference (CHANGED)

In the desktop logged-out cluster (hidden md:flex, Nav implementation reference-69) keep Log In (ghost →
/login) and the existing Post a Request (ghost → /login); ADD
a “Sign up” link with a small primary button treatment leading visually
(primary). Mobile drawer already has Register (line 106) — **do NOT duplicate**. Accessibility: use a link with the accessible name “Sign up”.

---

## 6. New / changed routes (App implementation reference)

| Route | Page | New? |
|---|---|---|
| /tutee/requests/:job Id/applicants | Applicants implementation reference (§E) | **NEW** |
| /tutor/profile | + handle/PT-FT/rate/area/completeness (§A) | CHANGED |
| /assignments/:id | + Apply/Express-interest sheet (§C) | CHANGED |
| /tutor/applications | + Status Pill/SUPERSEDED/interview/warm treatment (§D) | CHANGED |
| /admin/interviews | + Selection-requests tab (§G2) | CHANGED |
| /admin/matching | + assign-and-close + cancel/reopen (§G3) | CHANGED |
| /admin/commission | + stale (>14d) tab (§G4) | CHANGED |
| /tutors, / showcase | nameless redesigned cards (§B) | CHANGED |
| nav | desktop Sign up (§H) | CHANGED |

Only one NEW route element (Applicants) is added to the App implementation reference table; the rest are in-page changes.

---

## 7. Testing & coverage

- **Gate:** the documented operational check ≥97.5% line / ≥88% branch; the documented operational check (tsc) green; the documented operational check
  clean. Run the documented operational check BEFORE declaring done (type errors don't surface in test:ci).
- **Key Vitest cases:** Status Pill matrix (all 6 statuses × has Interview true/false) — exhaustively
  covers the new mapper branches incl. SUPERSEDED and the interview pseudo-status; Commitment Badge
  null → renders nothing; Applicant Card no-rating "(no reviews yet)" + pitch/also Teaches present vs
  absent + chosen/selection-pending action variants; Choose Tutor Modal real-fee copy assertion (NO
  "S$XX"); Profile Completeness Bar 0/partial/full + each missing-gate-field branch; profile handle
  errors (422 INVALID PUBLIC HANDLE / 409 PUBLIC HANDLE TAKEN) set inline field error; Apply Interest Sheet
  AC-04 honesty line present + already-applied/job-not-active disabled states; contact-reveal enabled
  only when assigned (404 not treated as error); admin 409 concurrency toast + refetch; stale empty
  state; **the fixed Showcase Tutor Card tests** (C5 — hourly Rate Min Sgd/Max fixtures, no name, PT/FT).
- a11y assertions: aria-pressed via the to have attribute operation (NOT
  to Be Checked); multiple matches via find All By Text + length; role is progressbar/aria-live/
  Modal label Id presence.

---

## 8. OPEN (frontend) — RESOLVED 2026-06-01 (team-reviewed; backend follow-up shipped + verified)

The frontend team review (9 real sub-agents) SENT BACK because 4 of these 6 were genuine backend contract gaps.
The backend follow-up (migration 0030 + 4 data transfer object/endpoint changes) shipped, full suite 1107/0/0, 0030 applied
+ verified on the live dev database. All 6 now RESOLVED with the real shipped field names — bind the frontend to these:

- **OPEN-frontend-1 RESOLVED:** read operation for tutor profile → Tutor Profile Response now returns, in addition
  to display Name, qualifications, years Experience, bio, featured Consent: **Handle (string),
  commitment (Tutor Commitment|null), hourly Rate Min Sgd/hourly Rate Max Sgd (int|null),
  location Planning Area (string|null), location Region (Location Region|null)**. Wire the
  tutor-profile query + completeness bar to these exact names. Extend the Tutor Profile Data TS
  interface to match.
- **OPEN-frontend-2 RESOLVED (incl. Casey personal information fix):** the showcase identity field tutor Display Name is now
  sourced from the validated **handle** (never display name), null/blank → Tutor
  placeholder. Showcase Tutor Response gains **commitment**. The browse-card source is
  **Tutor Post Response** (Public Browse Controller read operation for tutors) — it gains **tutor Public Handle**
  + **commitment** (both from the tutor profile, never display name). frontend: **show tutor Public Handle
  on browse + showcase cards** (handle LEADS — resolves the 3-reviewer + Priya "blank card" finding);
  fix bug ① in Showcase Tutor (rate Min Sgd/Max → hourly Rate Min Sgd/Max, +commitment,
  +tutor Public Handle, − real name). The personal information-free contract test on the backend side is extended.
- **OPEN-frontend-3 RESOLVED:** new **read operation for admin selection-requests** (admin-only) →
  a page of administration selection requests of CREATED requests, newest-first. data transfer object:
  **selection Request Id, tutee Post Id, job Subject, job Level, tutor Public Handle, tutor User Id,
  job Application Id, status, created At** (personal information-free — handle + opaque tutor User Id only). §G2
  Selection Request Queue reads this. §G4 stale-commission filter param names: confirm on the existing
  admin commission list when building §G4 (lower-risk; not a blocker).
- **OPEN-frontend-4 RESOLVED:** the My-Applications row data transfer object (Application Response) has **no area field** →
  **§D drops the area column**; render a readable subject and level label only.
- **OPEN-frontend-5 RESOLVED:** has Interview is *defined as* interview Id differs from no value (same lookup) — so
  **interview Id is always populated when has Interview is true**. The §D "View interview →"
  deep-link by interview Id is always valid; no fallback needed.
- **OPEN-frontend-6 RESOLVED:** submit operation for interviews now makes tutor Post Id **optional** + adds optional
  **tutor User Id**; a listing-less applicant is interviewed via tutee Post Id, tutor User Id (422
  NOT AN APPLICANT if they didn't apply; 400 TUTOR REQUIRED if neither id given). **Applicant Card Response
  now carries tutor User Id** (opaque unique identifier, personal information-safe) so §E "Request interview" has the id to call with.

### 8a. Round-2 conditions (Sam/Jordan) — binding for the implementation commit
- **C-test-matrix (Sam):** the atomic SUPERSEDED/re-map commit MUST also rewrite the test matrices that
  hard-code the old labels: format test implementation reference (MATCHED→'Matched', REJECTED→'Not Selected' /
  variants) and Applications test implementation reference (rendered 'Matched'/'Not Selected'). tsc
  green ≠ done — the documented operational check is red until these are updated.
- **C-pii-regression (Sam):** Showcase Strip Pii Regression test implementation reference asserts the
  display name renders as a mount-proof — removing the name node breaks it. Fix it (assert on the
  handle/subjects instead) in the same commit; it's a Boss-cleared regression test, do not silently break it.
- **C-withdraw-gating (Jordan):** UIUX/older draft said withdraw allowed on APPLIED+SHORTLISTED, but the
  live code + test (Applications implementation reference, Applications test implementation reference) gate it to **APPLIED only**. KEEP
  APPLIED-only (no behaviour change); do not flip it.
- **C-cache (Jordan):** after choose-tutor, pin applicant Keys (or the actual applicants query key) as
  the single invalidation source so the chosen card flips — do NOT rely on the unused
  application Keys by Tutee Post.
- **C-crosspage (Reese):** add an explicit Vitest case — chose a tutor on a non-visible page → all
  visible cards still render "Choose instead", no Request-interview leak on the chosen one.
- **C-nav-coverage (Sam):** Nav implementation reference is coverage-excluded (vite config implementation reference), so §H needs a manual/build check.

---

## 9. Summary of breaking type-changes (build-gating)

1. Job Application Status union += SUPERSEDED (index implementation reference).
2. JOB APPLICATION STATUS constant += SUPERSEDED (status implementation reference).
3. application Status Label Record += SUPERSEDED + MATCHED/REJECTED re-mapped (format implementation reference).
4. application Status Badge Variant Record += SUPERSEDED + MATCHED→assigned (format implementation reference).
5. Badge Variant union += assigned + variant Classes entry (Badge implementation reference).
6. Showcase Tutor: rate Min Sgd/Max → hourly Rate Min Sgd/Max, +commitment, +tutor Public Handle,
   −real-name (index implementation reference) → **breaks Showcase Tutor Card implementation reference + Showcase Tutor Card test implementation reference +
   Showcase Strip Pii Regression test implementation reference** (must fix all three in same commit, C5/C-pii-regression).
7. Browse Tutor Post/Tutor Post Response TS type += tutor Public Handle + commitment (card now leads
   with the handle). My-Applications: NO area field (drop the column). Applicant Card Response TS type
   += tutor User Id. Create Interview Request TS: tutor Post Id optional + add tutor User Id.

Items 1–4 are one atomic commit or tsc/the documented operational check/Docker build is red (§0). Item 6 is the bug ①
contract fix and its masking tests.
