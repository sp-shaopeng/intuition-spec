# Requirements — Flag hygiene, seeder, frontend hygiene (018)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> EARS + Gherkin per atomic requirement, grounded 2026-06-22. Source: CONSOLIDATED md (T-05, T-08, demo-data, T-01,
> T-03, CTA-intent).

## Scope note (verified)

- Phone Settings Section implementation reference reconciles the modal off a response indicating that WhatsApp is disabled, but profile-save phone-change
  surfaces verification messaging that should be suppressed when WhatsApp is OFF.
- Dev Data Seeder save Interview (1078-1097) never sets tutor User → NULL tutor user id on every seeded
  interview (live create Interview sets it, Interview Service implementation reference).
- The application originally sent an unmatched route to Not Found; role-specific dashboards existed, but there was no
  /dashboard route.
- Create Request implementation reference Subject is * required; a step header reads "All optional".
- CTA intent: M17 already defaults auth CTAs to login on detail pages; some generic /register/"Post a request"
  CTAs still bounce to /login.

---

## REQ-001 — No WhatsApp verification messaging while the feature is OFF (T-05)

- **Type:** functional / flag hygiene · **Priority:** Should · **Severity:** S4 · **Source:** T-05
- **Rationale:** Showing "Sending verification code…" when WhatsApp is OFF confuses users and implies a feature that
  isn't running.
- **Behaviour (EARS):** WHILE the WhatsApp feature flag is OFF (enabled:no), the system SHALL NOT trigger a
  WhatsApp **one-time passcode send** NOR show **WhatsApp** verification toasts on profile/phone save. A changed phone
  remains pending under 012 until explicit email-link confirmation; the current phone stays active. Show
  pending/email-confirmation copy, not "phone updated", until the commit succeeds. WHEN the flag is ON,
  current WhatsApp verification behaviour applies.
- **NOTE (cross-wave — Wave-2 condition, MANDATORY): the suppression is CHANNEL-SCOPED to WhatsApp/one-time passcode ONLY.** It
  MUST NOT suppress **spec 012/REQ-004's email-link phone-verification copy** (the email-link confirmation flow for a
  phone change routes a link to the account email and is INDEPENDENT of INT-082/WhatsApp). The suppression key is the
  WhatsApp/one-time passcode channel, NOT "any phone-change verification messaging". When 012/REQ-004 ships, a phone save that
  starts the email-link flow MUST still show its "we sent a confirmation link to your account email" copy even with
  WhatsApp OFF. The author of this REQ (Wave 3) MUST gate the suppression on the WhatsApp channel specifically.
- **Owner:** frontend-expert (· software-engineer if a backend send is triggered)
- **Acceptance criteria (Gherkin):**


- Scenario: No WhatsApp verification toast while the email-confirmation flow remains visible.
- Given the WhatsApp feature is OFF.
- When the user saves their profile / changes phone.
- Then no WhatsApp "Sending verification code…" toast appears and no WhatsApp one-time passcode is sent.
- And phone-change copy says a confirmation link was sent to the account email.
- And the old phone stays active until explicit email-link confirmation commits.
- And only after that commit may the user interface say the phone was updated.
- Scenario: Verification works when ON (regression).
- Given the WhatsApp feature is ON.
- Then verification messaging/one-time passcode behaves as before.


- **Required evidence:** RTL test (flag OFF → no WhatsApp toast/one-time passcode; email-link/pending copy retained; ON WhatsApp behavior unchanged); live capture with the flag
  OFF. Under REQ-001 implementation reference.
- **Dependencies:** INT-082 (WhatsApp flag).

## REQ-002 — Seeder sets tutor user id on seeded interviews (T-08)

- **Type:** functional / test-data · **Priority:** Should · **Severity:** low · **Source:** T-08
- **Rationale:** NULL tutor user id on seeded interviews makes the tutor interview list empty, so the "Interview
  scheduled" badge in My Pipeline disagrees with an empty /tutor/interviews (T-08). Live data is fine; only the
  seed is wrong.
- **Behaviour (EARS):** WHEN Dev Data Seeder save Interview creates a seeded interview, it SHALL set
  tutor User (= the tutor-post's tutor) so the row is reachable by find By Tutor User Id, matching the live
  create Interview invariant.
- **Owner:** software-engineer
- **Acceptance criteria (Gherkin):**


- Scenario: Seeded interviews have a tutor user id.
- Given a freshly reseeded dev stack.
- When a tutor opens /tutor/interviews.
- Then their seeded interviews appear (no NULL tutor user id rows).
- And the My Pipeline badge and the interviews list agree.
- Scenario: The seeder test asserts the invariant (no silent regression).
- Given Dev Data Seeder Test after seeding.
- Then every seeded interview has a non-null tutor user id.
- And that tutor user id equals the interview's tutor-post's tutor.


- **Required evidence:** Dev Data Seeder Test ASSERTING every seeded interview has a non-null tutor user id equal to
  its tutor-post's tutor (the shared invariant with live create Interview); live reseed + tutor interviews
  non-empty. Under REQ-002 implementation reference.
- **Dependencies:** none.

## REQ-003 — Seed-data variety on landing / assignments (demo-data)

- **Type:** non-functional / data · **Priority:** Could · **Severity:** S4 · **Source:** demo-data
- **Rationale:** ~8 identical "Primary 5 · Math (PSLE) · S$30-60" cards make the marketplace look fake. Variety
  (subject, level, budget, location) sells "real jobs now".
- **Behaviour (EARS):** WHEN the dev seeder creates the OPEN (ACTIVE) tutee posts for the public showcase, the set
  SHALL include **at least 8 open showcase posts** and satisfy a MEASURABLE variety bar: **≥ 8 distinct subject Code, ≥ 5 distinct level Code, ≥ 4 distinct
  planning-area** across the open posts, AND **no single (subject Code, level Code, budget-bucket, planning-area) tuple
  exceeds 20%** of them — so the landing//assignments showcase cannot be dominated by one repeated card.
- **Owner:** data (seeder) · software-engineer
- **Acceptance criteria (Gherkin):**


- Scenario: Varied showcase meets the measurable bar.
- Given a reseeded dev stack.
- When the open showcase posts are inspected.
- Then at least 8 open showcase posts exist.
- And ≥8 distinct subjects, ≥5 distinct levels, and ≥4 distinct planning-areas appear.
- And no single (subject, level, budget-bucket, planning-area) tuple is more than 20% of them.


- **Required evidence:** a seeder/inspection assertion against the variety thresholds above; live capture of the
  varied showcase. Under REQ-003 implementation reference.
- **Note (scope):** REQ-003 stays **Could**-priority. It is retained in Wave 1 because it now carries a measurable,
  assertable acceptance bar (the gate's condition); if the Boss prefers to trim Wave 1, it is the cleanest single
  deferral (dev-data cosmetics, no production-logic dependency) — but it is no longer the unmeasurable bullet the
  review rejected.
- **Dependencies:** none.

## REQ-004 — /dashboard redirects to the role dashboard (T-01)

- **Type:** functional / routing · **Priority:** Should · **Severity:** S3 · **Source:** T-01
- **Rationale:** A bare /dashboard is a natural URL/bookmark; today it hard-404s. It should route the user to
  their role dashboard (or login if unauthenticated).
- **Behaviour (EARS):** WHEN an authenticated user navigates to /dashboard, the system SHALL redirect to their
  role dashboard (/tutee/dashboard, /tutor/dashboard, or the admin dashboard). WHEN unauthenticated, it SHALL
  redirect to /login (preserving any post-login return). It SHALL NOT render the hard 404.
- **Owner:** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: /dashboard routes a tutee.
- Given an authenticated tutee.
- When they visit /dashboard.
- Then they are redirected to /tutee/dashboard (not a 404).
- Scenario: /dashboard while logged out → login.
- Given no authenticated user.
- When they visit /dashboard.
- Then they are redirected to /login.


- **Required evidence:** RTL/router test (each role + logged-out); live capture. Under REQ-004 implementation reference.
- **Dependencies:** none.

## REQ-005 — Step copy matches the actual required fields (T-03)

- **Type:** functional / copy · **Priority:** Should · **Severity:** S4 · **Source:** T-03
- **Rationale:** "All optional" over a step whose Subject field is *required is contradictory and erodes trust in
  the form.
- **Behaviour (EARS):** WHEN a create-post wizard step contains a required field (e g. Subject & level, *), the
  step header SHALL NOT claim "All optional"; the copy SHALL accurately reflect which fields are required vs
  optional.
- **Owner:** frontend-expert (· uiux-designer for copy)
- **Acceptance criteria (Gherkin):**


- Scenario: Step copy is accurate.
- Given a wizard step with a required field.
- Then the header does not say "All optional".
- And the required/optional state is accurately communicated.


- **Required evidence:** RTL/structural test of the corrected copy; live capture. Under REQ-005 implementation reference.
- **Dependencies:** specs 008/009 (wizard).

## REQ-006 — Unauthenticated CTAs default to login (CTA-intent)

- **Type:** functional / user experience · **Priority:** Should · **Severity:** S3 · **Source:** CTA-intent
- **Rationale:** Generic /register + "Post a request" CTAs that bounce to /login are jarring; M17 already
  established login-default for auth CTAs. Make the remaining generic CTAs consistent.
- **Behaviour (EARS):** WHEN an unauthenticated visitor activates a generic auth-required CTA, the system SHALL route
  them to **/login** by default (with register reachable from there), consistent with M17, rather than landing on
  /register and then bouncing to /login. The explicit create-account CTAs SHALL still go to /register.
- **In-scope CTAs (ENUMERATED, grounded 2026-06-22 — Wave-2 condition):**
  - **FLIP to /login** (generic auth-required CTAs that currently →/register and would bounce anonymous users):
    - Rate Guide implementation reference "Post a request" ().
    - Invite Tutor Modal implementation reference "Post a request →" — only when it routes an unauthenticated
      visitor; (if it is only reachable by an authed tutee, confirm at build and leave as-is).
    - Any other generic "Sign up"/"Get started"/"Post a request" CTA on a PUBLIC page that points at bare /register
      with no role/create-account intent (audit the public pages — Landing, Rate Guide, strips — at build).
  - **KEEP as-is (explicit create-account / role-intent — regression-guarded, MUST NOT flip):**
    - Nav implementation reference "Sign up" () — the explicit create-account CTA.
    - Nav implementation reference "Post a Request" → /register?role is tutee — INT-172 carries demand-side
      create-account intent deliberately (a role-qualified register, not a generic login-wall CTA); LEAVE.
    - Active Tutors Strip implementation reference, Recent Requests Strip implementation reference, Landing implementation reference →
      /register?role is tutor — role-qualified create-account intent; LEAVE.
    - Browse Assignments implementation reference already correctly points anonymous visitors to /login (INT-100) — no
      change, used as the reference pattern.
  - The RTL test SHALL enumerate the FLIP set (generic → /login) AND the KEEP set (explicit/role-qualified →
    /register) so neither regresses.
- **Owner:** uiux-designer · frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Generic auth CTA → login.
- Given an unauthenticated visitor.
- When they activate a generic auth-required CTA.
- Then they land on /login (not /register-then-bounce).
- And register is reachable from login.
- Scenario: Create-account CTA → register (regression).
- Given the explicit create-account CTA.
- Then it goes to /register.


- **Required evidence:** RTL/router test (generic CTA → login; create-account → register); live capture. Under
  REQ-006 implementation reference.
- **Dependencies:** M17.

---

## Traceability summary

| REQ | One-line | Type | Priority | Severity | Owner |
|---|---|---|---|---|---|
| REQ-001 | Suppress WhatsApp verification toast/one-time passcode while the feature is OFF | functional/flag | Should | S4 | frontend-expert · software-engineer |
| REQ-002 | Seeder sets tutor user id on interviews | functional/test-data | Should | low | software-engineer |
| REQ-003 | Varied seed data on landing/assignments | non-functional/data | Could | S4 | data · software-engineer |
| REQ-004 | /dashboard redirects to role dashboard (or login) | functional/routing | Should | S3 | frontend-expert |
| REQ-005 | Step copy matches required fields (no false "All optional") | functional/copy | Should | S4 | frontend-expert · uiux-designer |
| REQ-006 | Unauthenticated generic CTAs default to login (M17) | functional/user experience | Should | S3 | uiux-designer · frontend-expert |

## Open questions for the Boss / founder

1. **REQ-001: RESOLVED by 012 v1.3.** WhatsApp OFF suppresses only WhatsApp one-time passcode/copy. The confirmed
   email-link flow keeps the old phone until explicit confirmation; there is no silent-update default and no
   WhatsApp-dependent blanket block. Old-phone warning transport remains separately pending under 012.
2. **REQ-006 scope: RESOLVED (Wave-2 condition folded).** The in-scope CTAs are now ENUMERATED in REQ-006 above —
   FLIP set (generic public /register CTAs → /login, e g. Rate Guide implementation reference) vs KEEP set
   (explicit/role-qualified create-account CTAs → /register, e g. Nav implementation reference "Sign up", Nav implementation reference
   ?role is tutee, the ?role is tutor strips). The RTL test enumerates both so the explicit create-account path is not
   caught.
