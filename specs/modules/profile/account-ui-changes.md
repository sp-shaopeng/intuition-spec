# Account / Profile — user interface Changes (2026-05-29)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The blanket counterparty-never wording is corrected below. Public/pre-match contact remains private; the explicit ACTIVE/COMPLETED matched-participant exception and advance mutual-visibility disclosure are required by [INT-090/097](../../architecture/int-090-batch-m18-funnel-completion-spec.md). Contact edits follow [012](../../012-tutor-apply-and-profile-ux/requirements.md) email-link confirmation and old-value preservation; formatting alone does not authorize disclosure.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** BUILT + DEPLOYED — email/phone shown plaintext to authorized viewers (own/admin/match), format Phone spacing, qualification dropdown, portfolio-completion prompt all shipped. Ratified personal information-visibility model: own is plaintext, admin is authorized, public/pre-match is none, ACTIVE/COMPLETED matched counterparty is explicit participant-gated reveal.
**Author:** Product + frontend (founder directives, 2026-05-29).
**Scope:** Frontend-focused account/profile user experience. Four directives below.

> **Critical boundary (Personal Data Protection Act — do not cross):** these changes affect only views where
> email/phone are **already returned to an authorized viewer** (a user's own profile,
> admin review detail, active-match contact). They do **NOT** add email/phone to any
> public, pre-login, or pre-match counterparty surface, and they do **NOT** touch the personal information-free
> M14 public showcase. "Unmask" means "show the full value the application programming interface already returned to
> someone allowed to see it," never "expose a value that was previously hidden by role."
> At-rest email/phone storage follows the later [plaintext-email-phone amendment](../../architecture/plaintext-email-phone.md); this display change does not itself choose storage policy.

---

## 1. Show email & phone in plaintext (remove masking)

**Directive:** "all phone number and email should not be masked, plaintext."

**Current:** the user's own Profile implementation reference and Profile implementation reference render their phone
through the mask phone operation (e g. 9123 ****). Admin review detail and match-contact views
already show plaintext.

**Change:** remove mask Phone/mask Email from all display sites; render the full value
returned by the application programming interface. Delete the mask helpers (or leave unused) once no caller remains.

**Acceptance:** a user viewing their own profile sees their full email and full phone; no
*/• masking remains anywhere a value is already authorized; the public showcase and any
pre-match counterparty view still exposes **no** email/phone. ACTIVE/COMPLETED matched participants may use the
separately authorized mutual contact reveal, with the required advance disclosure (INT-090/097).

## 2a. Format phone numbers with spacing

**Directive:** "format the phone number with spacing."

**Change:** add the format phone operation to format implementation reference. Singapore mobile = 8 digits →
display XXXX XXXX; if a +65 country code is present, display +65 XXXX XXXX. Apply at
every phone **display** site (own profile, admin review detail, match contact). Inputs/storage
remain raw digits; formatting is display-only.

**Acceptance:** the format phone operation; the format phone operation; non-8-digit input degrades gracefully (returned as-is).

## 2b. Qualification → dropdown

**Directive:** "qualification in the apply account should be a drop down."

**Current:** tutor listing creation (Create Listing implementation reference / the /register/commission
step) has qualifications as a free-text input.

**Change:** replace the free-text input with a **select** backed by a typed constant
QUALIFICATION OPTIONS (no magic strings):
O-Level, A-Level, Diploma, Undergraduate, Bachelors Degree, Masters Degree,
Ph D, NIE-Trained Teacher, Other. The selected label is sent in the existing
qualifications string field (no the private implementation folder change; backend continues to store the string).
Keep an optional free-text "details" only if needed; the primary field is the dropdown.

**Acceptance:** the field renders as a <select> with the defined options; submitting persists
the chosen value; existing free-text values still display.

## 3. Prompt tutor to complete their portfolio after registration

**Directive:** "prompt tutor to fill up their portfolio after account registration."

**Current:** a tutor registers → /register/commission → listing creation. After that, there's
no nudge to complete a rich profile/portfolio (qualifications, experience, teaching approach,
headline, etc.).

**Change:** after a tutor completes registration, route/prompt them into portfolio completion:
- On first landing in the tutor area with an incomplete profile/listing, show a dismissible
  **"Complete your portfolio"** banner/CTA (and/or redirect to the profile setup) listing what's
  missing (headline, qualifications, experience, teaching approach, availability).
- Compute "incomplete" from the tutor's profile/listing fields already returned by the application programming interface.
- Dismissible, re-appears until the portfolio is reasonably complete.

**Acceptance:** a newly-registered tutor with an incomplete portfolio sees the prompt; once the
key fields are filled, the prompt no longer appears; a tutee never sees it.

---

## Out of scope / guardrails
- No change to at-rest encryption or to the public M14 showcase (stays personal information-free).
- No new exposure of email/phone to unauthorized roles.
- Backend unchanged except (optional) accepting the dropdown's string value (already does).

**Later disclosure amendment:** [INT-125/129](../../architecture/int-125-batch-closeout-spec.md) keeps the
participant reveal controls on ACTIVE and COMPLETED indefinitely; match-PENDING stays hidden in the frontend.
This does not widen unrelated ACTIVE-only sections or public/pre-assignment disclosure.
