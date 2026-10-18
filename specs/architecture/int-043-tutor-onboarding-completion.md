# INT-043 — Tutor Onboarding: Optional Profile Depth at Sign-up + Post-login Completion Gate

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Optional signup enrichment remains; [002](../002-tutor-profile-and-onboarding-ux/spec.md) adds optional profile fields/subjects and [012](../012-tutor-apply-and-profile-ux/requirements.md)/[024](../024-postfix-review-fixes/spec.md) settle the shared planning-area/subject Level completeness predicate. The older five-field/location Region/no-subject descriptions are replaced in that scope. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Status:** SPEC — pending team review  
**Date:** 2026-06-03  
**Author:** product-designer (lead), frontend-expert, software-engineer, uiux-designer  
**Tracker:** [README implementation reference](../issues/README.md) — INT-043  
**Related:** INT-017 (apply-gate 422 from incomplete seed profile, FIXED-DEV)  
**Governance:** /team-review required before implementation; Boss audit required before ship.  
**Next free migration:** 0032 (INT-025 note: 0031 = INT-015 soft-delete trigger; coordinate if INT-025 is built first).

---

## 1. Problem

Tutor sign-up currently requires qualifications and years Experience to be filled on
the commission step (Commission implementation reference → submit operation for auth register tutor). These are
**profile-depth fields** — they describe teaching credentials, not account identity.

Blocking sign-up on them creates friction at the exact moment conversion is most fragile:
a prospective tutor who doesn't have their CV to hand, or who is exploring whether
InTuition is worth their time, simply bounces. The commission terms acknowledgement is the
only logically-required gate at registration (it is a binding agreement).

Separately, Job Application Service assert Profile Complete To Apply gates job applications on
five fields (public handle, commitment, rate range, region, qualifications). INT-017 exposed
that even seeded tutors hit this wall — because their profiles were created without those
fields. The fix for INT-017 patched the seeder; the underlying product question — _when and
how does a tutor complete their profile?_ — remained open and is this spec's concern.

### Founder directive

> "Don't block sign-up on profile depth. After login, prompt the tutor to complete their
> profile if missing."

The directive has two parts:
1. Make qualification + years-experience optional at registration.
2. Post-login: surface a clear completion prompt if key profile fields are missing.

---

## 2. Proposed Design

### 2.1 Conceptual model

Three tiers of tutor completeness:

| Tier | Fields | Unlocks |
|------|--------|---------|
| **Account** (sign-up) | email, phone, password, display Name, commission ack | Account creation; can log in; can browse jobs; can create a tutor listing (Tutor Post) |
| **Identity** (apply-gate) | Handle, commitment, hourly rate min+max, location Region, qualifications | Applying to jobs (submit operation for tutor applications) |
| **Optional depth** | years Experience, bio | Shown on tutor cards/detail; no hard gate |

This maps to the existing assert Profile Complete To Apply logic (unchanged) and the
compute Completeness / GATE FIELD LABELS in Profile implementation reference (five fields: Handle,
commitment, rate, location Region, qualifications).

years Experience is already classified as "NOT a gate" in Profile implementation reference — this spec
aligns the sign-up step to that decision.

### 2.2 Registration flow after this change


Implementation example omitted. This public edition retains the written requirements in this section.


### 2.3 Post-login completion banner

Placement: inside Tutor Layout, rendered **above <Outlet /** (after the tab bar), so
it appears on every tutor dashboard page until the profile is complete. It is dismissible
per-session (session Storage) so an active tutor who chooses to finish their profile later
is not pestered on every tab switch, but will see it again on next login.

Reuse Profile Completeness Bar and Apply Gate Banner from
Profile Completeness Bar implementation reference. The banner already exists and already links
to /tutor/profile. The new work is:
- A thin Profile Completion Banner wrapper rendered in Tutor Layout that queries the tutor
  profile and checks completeness.
- Session-dismiss logic (the set item operation).

The profile completeness data comes from read operation for tutor profile (already used by
Profile implementation reference). To avoid an extra round-trip on every layout render, the query should be
cached via the existing Tan Stack Query tutor-profile key with a 5-minute stale Time.

The banner is **informational, not blocking**: a tutor can still browse listings, create
tutor posts, and navigate the dashboard without completing their profile. Only applying to
a job is gated (by the existing backend 422 and the Apply Gate Banner on the profile page and
on the apply flow). See the gating matrix in Section 4.

### 2.4 Design tokens (INT-049 alignment)

All new frontend surfaces must adopt the INT-049 refreshed visual language once that spec ships
(navy/clay/cream brand locked; updated spacing, shadow, type scale). For now, implement
using the existing design-token class names (bg-amber-50, border-amber-200, text-navy,
text-clay, etc. — consistent with the existing Apply Gate Banner). When INT-049
implementation lands, the banner gets the same polish pass as the rest of the tutor
dashboard.

---

## 3. Frontend Changes

### 3.1 Commission implementation reference — remove profile-depth fields

Remove from the zod schema:
- qualifications field (was a select control from QUALIFICATION OPTIONS)
- years Experience field (was a number input field)

Remove from the form JSX: the two form controls and their error messages.

Remove from the submit payload (the application programming interface call at line 94):
- qualifications: data qualifications
- years Experience: data years Experience

The QUALIFICATION OPTIONS import from constants/qualifications can be removed if it
is not used elsewhere (check all uses of the qualification options).

Update the page heading from "Commission Structure" to remain as-is; update the subheading
from "tell us about your teaching background" to remove the teaching-background reference
(e g. "read and acknowledge our commission terms to complete your account").

The COMMISSION TERMS VERSION constant and the commission acknowledgement user interface remain
unchanged — these are not profile-depth.

### 3.2 Register implementation reference — no change to Step 1

Step 1 (personal details) does not include qualifications or experience. No change
required here.

### 3.3 New Profile Completion Banner in Tutor Layout implementation reference

Add a new component (either in Profile Completeness Bar implementation reference as a named export or as its
own file Profile Completion Banner implementation reference) with this contract:


Implementation example omitted. This public edition retains the written requirements in this section.


Logic:
1. Read the get item operation — if set, return null.
2. Query read operation for tutor profile with stale Time: 5 * 60 * 1000, query Key: tutor-profile.
3. If profile is loading or the query errored, return null (non-blocking).
4. Run the compute completeness operation — same function already defined in Profile implementation reference (extract
   to a shared util, e g. tutor Completeness implementation reference, so both Profile and the banner can
   import it without duplication).
5. If completeness complete, return null.
6. Render: an amber banner (reuse Apply Gate Banner style) with:
   - Heading: "Complete your profile to start applying"
   - Body: "Still needed: missing fields joined with ', '."
   - CTA button: "Complete profile →" links to /tutor/profile
   - Dismiss button ("×" or "Remind me later") that sets the session Storage key and hides the banner.

Mount in Tutor Layout implementation reference immediately after the tab bar and before the main content:


Implementation example omitted. This public edition retains the written requirements in this section.


### 3.4 Subjects taught and tutor type

The INT-043 ticket mentions "subjects taught" and "tutor type" alongside qualification and
experience. The current sign-up flow does not collect subjects taught or tutor type
(commitment/PART TIME/FULL TIME) at registration — these already live in the profile form
and in the tutor listing (Tutor Post) form respectively. No change is needed for these: they
are already deferred to the post-login profile/listing flow.

If "tutor type" refers to a field not yet in the data model, it is an open decision (see
Section 7).

### 3.5 Apply-flow gate (frontend)

When a tutor clicks "Apply" on a job and their profile is incomplete, the current flow
shows Apply Gate Banner on the profile page. The completion banner in Tutor Layout gives
an earlier, dashboard-level signal. No change is needed to the apply-time gate — the
Apply Gate Banner on /tutor/profile remains the correct point-of-action gate for the
apply flow.

---

## 4. Backend Changes

### 4.1 Register Tutor Request — make profile-depth fields optional

File: Register Tutor Request implementation reference

Current:


Implementation example omitted. This public edition retains the written requirements in this section.


After:


Implementation example omitted. This public edition retains the written requirements in this section.


The @Not Blank constraint on qualifications becomes a @Size only (null is permitted;
if non-null it must be ≤ 1000 chars). years Experience becomes whole number so it can be
omitted from the payload (null means "not specified").

### 4.2 Auth Service register Tutor — handle nullable profile fields

File: Auth Service implementation reference

Current (lines 164–170):


Implementation example omitted. This public edition retains the written requirements in this section.


After:


Implementation example omitted. This public edition retains the written requirements in this section.


Empty string for qualifications signals "not yet filled" without changing the column
required constraint (migration 0002 has nullable is no). The assert Profile Complete To Apply
check uses the is present operation which treats a blank/empty string as
incomplete — so a tutor who registered without qualifications cannot apply until they fill
it in. This is the correct behaviour.

### 4.3 Tutor Profile model — no migration required

The tutor profiles qualifications column is TEXT required and years experience is
INT required DEFAULT 0 (migration 0002). Storing an empty string for qualifications and
0 for years experience is valid within these constraints. No Liquibase changeset is
needed for this change.

If the team decides that qualifications should be truly nullable at the database level (to
distinguish "not provided" from "provided as empty"), a migration is required:


Implementation example omitted. This public edition retains the written requirements in this section.


This is an open decision (see Section 7). The "empty string = not provided" approach avoids
a migration and keeps the existing is Present semantics; nullable adds a cleaner semantic
distinction at the cost of a migration and a nullable check in the model/Java persistence mapping.

### 4.4 read operation for tutor profile — expose profile completeness

The existing endpoint (used by Profile implementation reference) already returns all five gate fields. No
change is needed for the frontend to check completeness client-side. The backend does not need a
dedicated /profile/completeness endpoint; the frontend computes it locally using the same
compute Completeness helper already in Profile implementation reference.

### 4.5 assert Profile Complete To Apply — no change

The apply-time gate in Job Application Service is correct and intentional. It gates on:
Handle (non-blank) + commitment (non-null) + hourly Rate Min Sgd (non-null) +
hourly Rate Max Sgd (non-null) + location Planning Area (non-blank) + qualifications
(non-blank). This is unchanged by INT-043.

Note: the gate checks location Planning Area, not location Region. Profile implementation reference
compute Completeness checks location Region. These should be aligned — see Section 7,
Open Decision #3.

### 4.6 Auth Controller Test / Auth Service Test — update test fixtures

Remove qualifications and years Experience from the "happy-path register tutor" fixtures.
Add a test case: registering without qualifications/experience succeeds (201/200); the
created profile has qualifications= and years Experience is 0.

---

## 5. Gating Matrix

This matrix is the authoritative definition of what each action requires. It supersedes any
ad-hoc check in the codebase.

| Action | Account tier | Identity tier | Notes |
|--------|-------------|---------------|-------|
| Sign up (create account) | Required | Not required | Commission ack required |
| Log in | Required | Not required | |
| Browse tutor listings (own) | Required | Not required | /tutor/dashboard |
| Browse open job requests | Required | Not required | /requests |
| Create a tutor listing (Tutor Post) | Required | Not required | Listing has its own qualification field |
| Edit tutor profile | Required | Not required | |
| See profile completeness bar | Required | Not required | Shown on /tutor/profile |
| Be shown in public "Active Tutors" showcase (M14a) | Required | featured Consent is yes + at least one ACTIVE Tutor Post | |
| Apply to a job (submit operation for tutor applications) | Required | **All five identity-tier fields** | 422 TUTOR PROFILE INCOMPLETE if missing |
| Appear in admin selection-request queue | Required | Identity tier (applied → appears) | Downstream of apply gate |
| Contact reveal (post-match) | Required | Not required (already matched) | |
| Receive a review | Required | Not required | |

**Identity-tier fields** (all five required to apply):
1. Handle — non-blank
2. commitment — PART TIME or FULL TIME
3. hourly Rate Min Sgd — non-null
4. hourly Rate Max Sgd — non-null
5. qualifications — non-blank (≥ 10 chars per frontend schema; ≥ 1 char per backend is Present)
6. location Planning Area — non-blank (backend gate; location Region for frontend display — align per OD #3)

Note: years Experience and bio are never gated. They enrich the profile card but do not
block any action.

---

## 6. INT-017 Interaction

INT-017 (FIXED-DEV) patched Dev Data Seeder so seeded tutors carry a complete canonical
profile (Handle, commitment, hourly-rate range, planning area, qualifications). The
apply-gate now passes for seeded tutors.

INT-043 does not change the apply-gate. It changes what the sign-up flow collects. The two
interact as follows:

- A real tutor who registers via the new (slimmer) flow will have qualifications= until
  they fill in their profile. The Apply Gate Banner and Profile Completion Banner surface
  this gap clearly before the tutor attempts to apply.
- The Applications IT tutor apply To Job regression IT exercises the seeded tutor (which has
  a complete profile per the INT-017 fix). No change to that IT is required.
- A new IT should be added: register tutor without qualifications → log in → attempt apply
  → 422 TUTOR PROFILE INCOMPLETE → update profile → apply → 200. This validates the
  end-to-end flow introduced by INT-043.

---

## 7. Open Decisions

| # | Question | Options | Recommendation |
|---|----------|---------|----------------|
| OD-1 | Should tutor profiles qualifications be nullable at the database level? | (a) Keep required, store empty string for "not provided" (no migration). (b) Drop required, accept NULL (migration 0032). | **Option (a)** — avoids a migration; is Present semantics are unchanged; empty string is a valid "not yet filled" sentinel. Revisit only if analytics needs to distinguish "never provided" from "explicitly left blank". |
| OD-2 | Should the post-login banner be dismissible for the session only, or permanently? | (a) Session dismiss (session Storage — reappears on next login). (b) Permanent dismiss (local Storage or a backend flag). (c) No dismiss — always shown until complete. | **Option (a)** — balances persistence with completion nudge. A tutor who wants to ignore it will see it at most once per login session. Permanent dismiss risks tutors never completing; no dismiss is nagging. |
| OD-3 | Apply location/completeness | **Resolved by012/024.** | frontend and backend share location Planning Areas and the full predicate; a region label alone does not substitute for a required planning area. |
| OD-4 | "Tutor type" mentioned in INT-043: is this commitment (PART TIME/FULL TIME) or a separate enum? | Likely commitment — it is the only type-like field in the model. | Treat as commitment. If a distinct "tutor type" enum (e g. ACADEMIC/ENRICHMENT/LANGUAGE) is needed, file a separate issue with a schema design. |
| OD-5 | Subjects at signup/profile | **Superseded by002/012.** | Signup may optionally capture subject Levels; profile completeness at Apply requires the shared subject/level criterion. Listings remain optional. |
| OD-6 | Auto-redirect to /tutor/profile on first login? | (a) Banner only (recommended). (b) Hard redirect to profile on first login (intrusive). (c) Modal overlay. | **Option (a)** — banner is the least disruptive path; the tutor can choose to explore the dashboard first. An explicit welcome modal on the very first login (when qualifications is blank and no profile fields are set) is acceptable as an enhancement but not required in this spec. |

---

## 8. Acceptance Criteria

### 8.1 Registration (frontend + backend)

- AC-R1: A tutor can complete registration without providing qualifications or years of experience.
- AC-R2: The Commission page (/register/commission) shows only the commission terms and acknowledgement checkbox — no qualifications dropdown, no years-of-experience input.
- AC-R3: submit operation for auth register tutor with a payload that omits qualifications and years Experience returns 200/201 and a valid session.
- AC-R4: The created tutor profiles row has qualifications= and years experience is 0 when fields are omitted.
- AC-R5: A tutor who provides qualifications and years Experience at registration (legacy behaviour) is still accepted (backward compatibility).

### 8.2 Post-login banner (frontend)

- AC-B1: A logged-in tutor with an incomplete profile (missing any identity-tier field) sees the Profile Completion Banner on any tutor dashboard page.
- AC-B2: The banner lists the specific missing fields (e g. "Still needed: a public handle, your commitment, a rate range.").
- AC-B3: The "Complete profile →" CTA navigates to /tutor/profile.
- AC-B4: The banner has a dismiss affordance; clicking it hides the banner for the current session.
- AC-B5: After dismissing, navigating to another dashboard tab does not re-show the banner within the same session.
- AC-B6: After the tutor completes all identity-tier fields and saves, the banner does not appear on the next dashboard load (profile query returns complete).
- AC-B7: A tutor with a complete profile does not see the banner.

### 8.3 Apply gate (unchanged, verified)

- AC-A1: A tutor with an incomplete profile who attempts to apply to a job receives 422 TUTOR PROFILE INCOMPLETE.
- AC-A2: After completing their profile, the same tutor can apply successfully.
- AC-A3: The Apply Gate Banner on /tutor/profile continues to surface when profile is incomplete.

---

## 9. Test Plan

### 9.1 Backend (JUnit / Testcontainers)

| Test | File | Description |
|------|------|-------------|
| register Tutor without Qualifications And Experience succeeds | Auth Controller Test | POST register/tutor with no qualifications/years Experience → 201; profile row has qualifications is  and years experience is 0 |
| register Tutor with Qualifications And Experience succeeds | Auth Controller Test | Existing happy-path fixture — verify still passes after schema change |
| register Tutor with Qualifications toolong rejected | Auth Controller Test | qualifications >1000 chars → 400 VALIDATION ERROR (Size constraint still enforced) |
| apply To Job after Registration Without Qualifications 422 | Applications IT or new IT | Register without qualifications → attempt apply → 422 TUTOR PROFILE INCOMPLETE |
| apply To Job after Completing Profile 200 | Applications IT or new IT | Register without quals → complete profile via PUT → apply → 200 |

### 9.2 Frontend (Vitest)

| Test | File | Description |
|------|------|-------------|
| Commission page has no qualifications field | Commission test implementation reference | Render Commission page; assert no the get by label text operation |
| Commission page has no years Experience field | Commission test implementation reference | Render Commission page; assert no the get by label text operation |
| Commission submit calls application programming interface without qualifications/years Experience | Commission test implementation reference | Fill stash + submit → assert POST payload omits those fields |
| Profile Completion Banner renders when fields missing | Profile Completion Banner test implementation reference | Mock tutor-profile query with incomplete profile → banner visible; missing fields listed |
| Profile Completion Banner hidden when profile complete | Profile Completion Banner test implementation reference | Mock complete profile → banner not in DOM |
| Profile Completion Banner hides on dismiss | Profile Completion Banner test implementation reference | Click dismiss → banner removed; session Storage key set |
| Profile Completion Banner absent when dismissed in session | Profile Completion Banner test implementation reference | session Storage key pre-set → banner not in DOM on mount |
| Tutor Layout renders banner when profile incomplete | Tutor Layout test implementation reference | Incomplete profile mock → banner in layout output |
| Tutor Layout does not render banner when complete | Tutor Layout test implementation reference | Complete profile mock → no banner |

### 9.3 Coverage gate

After changes, the documented operational check must pass at ≥ 97.5% lines and ≥ 88% branches. the documented operational check must pass at ≥ 97.5% lines and ≥ 97.5% branches.

---

## 10. Implementation Order

1. **backend first:** Update Register Tutor Request + Auth Service register Tutor (nullable fields). Update Auth Controller Test / Auth Service Test. Run the documented operational check green.
2. **frontend Commission page:** Remove qualifications and years Experience fields. Update Commission test implementation reference. Run the documented operational check green.
3. **frontend completeness util:** Extract compute Completeness + GATE FIELD LABELS from Profile implementation reference to tutor Completeness implementation reference. Update imports in Profile implementation reference and new banner. Vitest green.
4. **frontend Profile Completion Banner:** Build and test the banner component. Vitest green.
5. **frontend Tutor Layout:** Mount banner. Vitest green.
6. **New IT:** apply To Job after Registration Without Qualifications end-to-end. the documented operational check green.
7. **Spotless + ESLint:** the documented operational check and the documented operational check before commit.
8. **Deploy:** Rebuild + deploy + reseed (no migration if OD-1 resolved as option (a)).
9. **Smoke:** Register a fresh tutor without qualifications → land on dashboard → see banner → complete profile → banner gone → apply to a job → 200.
