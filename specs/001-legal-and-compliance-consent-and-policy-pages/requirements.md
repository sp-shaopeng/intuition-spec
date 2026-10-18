# Requirements — Legal and Compliance consent and policy pages (001)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Each requirement is **atomic** (one independently testable behaviour), stated in **EARS**, with **Gherkin**
> acceptance criteria and the **exact evidence** that will prove each scenario. A compound requirement is two
> requirements — split it. See SKILL implementation reference for the EARS patterns and the evidence-type catalogue.
>
> **Scope.** Bring the signup flow and the legal content pages to Personal Data Protection Act-/Terms-compliance: four sign-up consent
> checkboxes (3 required, 1 optional) captured with an audit timestamp; the full Privacy Policy and Terms &
> Conditions copy published on their pages; and the company **street address removed from the public Footer and
> Contact page only** while it is **retained** on the Privacy and Terms pages (Personal Data Protection Act needs a contactable
> DPO/business address). Sources: GH #198 (checkbox wording), #199 (Privacy copy), #200 (Terms copy), #194 (address removal).
>
> **Founder decisions baked in (binding):**
> - #194: remove the agency’s registered business address from the **Footer and Contact page ONLY**; **KEEP** it on Privacy & Terms.
> - #198: the four checkboxes are (a) Terms+Privacy agreement [required], (b) granular Personal Data Protection Act data-use consent [required], (c) marketing opt-in [OPTIONAL], (d) 18+/guardian confirmation [required].
>
> **Project gates (apply to every REQ's test evidence):** backend JaCoCo ≥97.5% line AND ≥97.5% branch
> (the documented operational check); frontend Vitest ≥97.5% line AND ≥88% branch (the documented operational check); no magic strings
> (API ROUTES / typed enums both ends); Liquibase-only schema; soft-delete only (no requirement here physically
> deletes a row); consent persists with an audit timestamp.

---

## REQ-001 — When a user submits the registration form with all required consents ticked, the system shall persist the Terms-and-Privacy agreement, the Personal Data Protection Act data-use consent, and the age/guardian confirmation each with its own audit timestamp.

- **Type:** data
- **Priority:** Must
- **Source:** GH #198 (checkboxes a, b, d); Personal Data Protection Act consent-record obligation
- **Rationale / value:** A Personal Data Protection Act-defensible account needs durable, timestamped proof that the user actively gave each required consent at registration. Without the timestamp the consent is not auditable.
- **Owner (specialist):** dba (schema) + software-engineer (persistence wiring)
- **Acceptance criteria (Gherkin):**


- Scenario: Required consents are stored with timestamps on tutee registration.
- Given a visitor completing the tutee registration form.
- When they tick Terms+Privacy, Personal Data Protection Act data-use, and 18+/guardian, then submit.
- Then the new users row has terms accepted = TRUE with terms accepted at set.
- And pdpa consent = TRUE with pdpa consent at set.
- And age confirmed = TRUE with age confirmed at set.


- **Required evidence:** a private inspection of the new account showing accepted terms, privacy consent and age confirmation, each with a recorded time; the Liquibase changeset id (0058). Negative-path covered by REQ-008.
- **Dependencies:** REQ-007 (schema), REQ-005/006 (application programming interface contract)

---

## REQ-002 — When a user submits registration with the marketing opt-in ticked, the system shall persist marketing opt in = TRUE with marketing opt in at set; when it is left unticked, the system shall persist marketing opt in = FALSE with marketing opt in at NULL, and registration shall still succeed.

- **Type:** data
- **Priority:** Must
- **Source:** GH #198 (checkbox c, "Optional"); #199 §3 ("marketing consent will be separate from service-related consent")
- **Rationale / value:** Personal Data Protection Act requires marketing consent to be separate and genuinely optional. The user must be able to register without it, and the recorded value must reflect their actual choice with a timestamp only when opted-in (mirrors the existing whatsapp opt in + chk pattern).
- **Marketing-consent precedence (Boss ruling on Q2 — BINDING):** marketing opt in is the **MASTER marketing-consent gate**; the pre-existing whatsapp opt in (INT-082) is **channel-enablement UNDER it**. A user must have **marketing opt in yes AND whatsapp opt in yes** to receive WhatsApp **marketing**. The two columns stay **independent** in the schema (no superseding, no merge): marketing opt in = general marketing consent (#198 checkbox c, all channels: email/SMS/WhatsApp); whatsapp opt in = the specific WhatsApp-channel flag (INT-082). Any future WhatsApp **marketing** send MUST AND-gate on both flags (transactional WhatsApp notifications, e g. INTERVIEW SCHEDULED, are unaffected — they are not marketing).
- **Owner (specialist):** dba + software-engineer
- **Acceptance criteria (Gherkin):**


- Scenario: Marketing opt-in ticked is stored true with a timestamp.
- Given a visitor on the registration form.
- When they tick the marketing opt-in box and submit with all required boxes ticked.
- Then the users row has marketing opt in = TRUE and marketing opt in at is set.
- Scenario: Marketing opt-in left unticked still registers successfully.
- Given a visitor on the registration form.
- When they leave the marketing opt-in box unticked but tick all required boxes and submit.
- Then registration succeeds.
- And the users row has marketing opt in = FALSE and marketing opt in at IS NULL.


- **Required evidence:** Two database rows: one opted-in (TRUE + timestamp), one opted-out (FALSE + NULL), proving the field reflects choice and that opt-out does not block registration. curl log of the opted-out register returning success.
- **Dependencies:** REQ-007, REQ-005/006

---

## REQ-003 — While the registration form has the Terms-and-Privacy agreement checkbox unticked, the system shall block submission and surface a validation message.

- **Type:** user interface
- **Priority:** Must
- **Source:** GH #198 (checkbox a, required); #200 §1 (active clickwrap acceptance)
- **Rationale / value:** Clickwrap acceptance is only enforceable when the user must actively agree before the account is created. The form must not submit without it.
- **Owner (specialist):** frontend-expert (with uiux-designer on copy/placement)
- **Acceptance criteria (Gherkin):**


- Scenario: Submit blocked when Terms+Privacy unticked.
- Given a visitor who filled all fields and ticked Personal Data Protection Act + 18/guardian but NOT Terms+Privacy.
- When they click the submit button.
- Then the form does not submit (no register network call).
- And a validation message is shown on the Terms+Privacy checkbox.


- **Required evidence:** Playwright @1280 and @390 screenshot of the form with the Terms+Privacy box unticked, the inline error visible, and the network panel showing no register POST fired.
- **Dependencies:** REQ-004 (form renders the checkboxes)

---

## REQ-004 — When the registration form is rendered, the system shall display all four consent checkboxes with the exact wording from GH #198, with the Terms-of-Use and Privacy-Notice references as links to /terms and /privacy.

- **Type:** user interface
- **Priority:** Must
- **Source:** GH #198 (full wording of the four checkboxes)
- **Rationale / value:** The legally-binding text must be presented verbatim and the policy links reachable so consent is informed.
- **Owner (specialist):** frontend-expert + uiux-designer (with Technical Writer reviewing copy)
- **Acceptance criteria (Gherkin):**


- Scenario: Four consent checkboxes render with the issue wording.
- Given a visitor on the registration form (the account step, shown for both tutee and tutor).
- When the form loads.
- Then a checkbox reads "I have read and agree to the Terms of Use and Privacy Notice." with /terms and /privacy links.
- And a checkbox reads "I agree to receive marketing messages by email, SMS, or WhatsApp. (Optional)".
- And a checkbox reads "I confirm that I am at least 18 years old, OR I have verified parent/guardian consent.".


- **Required evidence:** Playwright @1280 and @390 screenshot of the registration form showing all four checkboxes with the verbatim wording and bullet list; DOM check that the two links point to /terms and /privacy.
- **Dependencies:** REQ-009 (Terms page exists), REQ-010 (Privacy page exists)

---

## REQ-005 — When the tutee-registration application programming interface receives a request, the system shall require the three required consent flags (terms accepted, pdpa consent, age confirmed) to be true and persist them with server-set timestamps.

- **Type:** functional
- **Priority:** Must
- **Source:** GH #198; Personal Data Protection Act consent-record obligation; existing Register Tutee Request contract
- **Rationale / value:** The server is the source of truth for consent. The frontend checkbox is not enough — the contract must carry, and the service must persist, the consents so a tampered/replayed client cannot create a consent-less account.
- **Owner (specialist):** bff-specialist (contract) + software-engineer (service)
- **Acceptance criteria (Gherkin):**


- Scenario: Tutee register with all required consents true persists them.
- Given a valid tutee registration payload with terms Accepted is yes, pdpa Consent is yes, age Confirmed is yes.
- When POST auth register tutee interface operation is called.
- Then the response is success (account created).
- And the persisted user has the three consent flags true with timestamps set by the server.


- **Required evidence:** curl log of submit operation for auth register tutee with the consent fields → success, followed by the database row showing the three flags + timestamps. (Tutor path is REQ-006.)
- **Dependencies:** REQ-007

---

## REQ-006 — When the tutor-registration application programming interface receives a request, the system shall require the three required consent flags and persist them with server-set timestamps, alongside the existing commission acknowledgement.

- **Type:** functional
- **Priority:** Must
- **Source:** GH #198; existing Register Tutor Request (commission ack) contract
- **Rationale / value:** Tutors register through a separate two-step path: consents are ticked on the **first** Register step (Register implementation reference Step 1, role is tutor), but the tutor account is **not created there** — it is created later on the **commission step** (Commission implementation reference, which POSTs application programming interface ROUTES auth.register Tutor at ~line 90 with a payload built ~lines 79–87 from the in-memory tutor Registration Draft). The four consent values captured on Step 1 must therefore travel through the draft to the create call, the same three required consents must be persisted, and the existing commission-acknowledgement contract must be undisturbed. **This consent carry-through is the #1 risk of this spec** (consents captured in one component, account created in another) — its three ACs below are explicit and non-negotiable.
- **Owner (specialist):** bff-specialist + software-engineer + frontend-expert (draft carry)
- **Acceptance criteria (Gherkin):**


- Scenario: Tutor register with all required consents true persists them.
- Given a valid tutor registration payload with terms Accepted is yes, pdpa Consent is yes, age Confirmed is yes, commission Acknowledged is yes.
- When POST auth register tutor interface operation is called.
- Then the response is success (account created).
- And the persisted user has the three consent flags true with timestamps, and the commission acknowledgement is unchanged.
- Scenario: The tutor draft carries the four consent booleans (AC-a).
- Given a visitor on Register implementation reference Step 1 with role is tutor who ticked the three required consents (and optionally marketing).
- When they advance to the commission step.
- Then the in-memory tutor Registration Draft (tutor Registration Draft implementation reference) carries terms Accepted, pdpa Consent, marketing Opt In, and age Confirmed alongside email/phone/display Name/password.
- Scenario: Step 1 blocks advancing to commission when a required consent is unticked (AC-b).
- Given a visitor on Register implementation reference Step 1 with role is tutor who left a required consent box (terms / Personal Data Protection Act / age) unticked.
- When they click "Continue to commission terms →".
- Then the form does not advance to the commission step (no draft is set / no navigation).
- And an inline validation message is shown on the unticked required box.
- Scenario: Commission implementation reference includes the four consent booleans in the register Tutor payload (AC-c).
- Given a complete tutor Registration Draft carrying the four consent booleans.
- When Commission implementation reference submits to application programming interface ROUTES auth register Tutor (~Commission implementation reference).
- Then the POST body built at ~Commission implementation reference-87 includes terms Accepted, pdpa Consent, marketing Opt In, and age Confirmed.


- **Required evidence:** curl log of submit operation for auth register tutor with consent + commission fields → success, then the database row showing the consent flags + timestamps AND the commission acknowledgement still recorded; a Playwright capture of Step 1 (role is tutor) blocking "Continue to commission terms →" with a required box unticked; the network-panel POST body from the commission step showing the four consent booleans present.
- **Dependencies:** REQ-007; **and an explicit frontend data-flow dependency** — tutor Registration Draft must carry the four consent booleans (AC-a), Step 1 must block advancing when a required box is unticked (AC-b), and Commission implementation reference must include them in the register Tutor payload (AC-c). Without all three, a tutor account is created with no recorded consent.

---

## REQ-007 — The system shall add the four consent column pairs PLUS the two policy-version columns to the users table via a Liquibase changeset (0058), each boolean defaulting FALSE and guarded by a require that the flag cannot be true without its timestamp, with the ALTERs bounded by SET LOCAL lock timeout is 3s.

- **Type:** data
- **Priority:** Must
- **Source:** Existing whatsapp opt in + chk whatsapp consent ts pattern (changeset 0047, which uses SET LOCAL lock timeout is 3s before its require the rule); GH #198
- **Rationale / value:** Consent state must be a first-class, integrity-guarded part of the schema — the require constraint makes a timestamp-less "true" impossible, exactly as chk whatsapp consent ts does today. Columns default FALSE so the migration is additive and backward-compatible with existing rows. The two **policy-version** columns (terms version, privacy version) record WHICH version of the copy was consented to — the Terms/Privacy copy is being introduced now and WILL change, so a bare consent timestamp is not enough to prove what the user actually agreed to (Personal Data Protection Act defensibility).
- **Owner (specialist):** dba
- **Acceptance criteria (Gherkin):**


- Scenario: Migration 0058 adds the consent columns, version columns, and require constraints.
- Given the changelog with new changeset 0058.
- When mvn verify applies migrations via Testcontainers.
- Then users has columns terms accepted, terms accepted at, pdpa consent, pdpa consent at, marketing opt in, marketing opt in at, age confirmed, age confirmed at.
- And users has columns terms version and privacy version (text, nullable).
- And the four booleans default FALSE and are required.
- And a require forbids each flag being TRUE while its timestamp IS NULL.
- Scenario: require rejects a timestamp-less true.
- Given the migrated schema.
- When an update records sets terms accepted = TRUE while terms accepted at IS NULL.
- Then the database rejects it with a constraint violation.
- Scenario: The ALTERs are lock-bounded (parity with 0047).
- Given changeset 0058.
- Then each database migration step that ADDs a CONSTRAINT is preceded by SET LOCAL lock timeout = '3s'.


- **Required evidence:** databasechangelog row for 0058; \d users (or information schema) output showing the eight consent columns + the two version columns + defaults + required on the booleans; a psql attempt to set a flag TRUE with NULL timestamp returning a require-constraint error; the changeset XML showing SET LOCAL lock timeout 3s ahead of each require the rule.
- **Dependencies:** none (foundation for REQ-001/002/005/006)
- **Note:** No pdpa consent column exists today (only whatsapp opt in); 0058 introduces all four consent pairs + the two version columns fresh — see Q1 (resolved: dba confirmed all-fresh) and dba residual D3 (the require the rule triggers a full-table validate scan on a populated users; negligible on the dev recreate-on-deploy stack, but note it for the production cutover).

---

## REQ-008 — If a registration request reaches the application programming interface without all three required consent flags set true, then the system shall reject it with a 4xx validation error and create no account.

- **Type:** security
- **Priority:** Must
- **Source:** GH #198 (required checkboxes); both-ends principle (guard the server, not just the form)
- **Rationale / value:** The frontend block (REQ-003) is bypassable. A direct application programming interface call missing a required consent must be rejected server-side so no consent-less account can ever exist — the negative/guard path Personal Data Protection Act actually depends on.
- **Owner (specialist):** bff-specialist (validation) + software-engineer
- **Reject mechanism — ONE deterministic pattern (RESOLVED, BINDING):** Each of the three required consents is guarded with **bean-validation @Assert yes** on the request data transfer object field — exactly mirroring the existing commission Acknowledged @Assert yes precedent in Register Tutor Request implementation reference. A no/absent value raises Method Argument Not Valid Exception, which the @response customization component Global error handler handle Validation (Global error handler implementation reference) maps **deterministically to web request protocol 400 with body a validation error carrying the required-consent message**. We do **NOT** add a second service-thrown the bad request operation path — that ambiguity (bean @Assert yes 400 vs the service-thrown COMMISSION ACK REQUIRED 400 that Auth Service register Tutor ~line 188 still throws for commission) is the exact dual-mechanism the gate flagged; for the **consents** there is one mechanism and one code. The asserted scenario asserts status equals 400 AND code equals VALIDATION ERROR.
- **Acceptance criteria (Gherkin):**


- Scenario: application programming interface rejects tutee register missing the Terms+Privacy consent.
- Given a tutee registration payload with terms Accepted is no (others true).
- When POST auth register tutee interface operation is called.
- Then the response status is 400.
- And the response body code is "VALIDATION ERROR".
- And no users row is created for that email.
- Scenario: application programming interface rejects tutor register missing the 18+/guardian confirmation.
- Given a tutor registration payload with age Confirmed is no (others true).
- When POST auth register tutor interface operation is called.
- Then the response status is 400 and the response body code is "VALIDATION ERROR".
- And no account is created.


- **Required evidence:** curl log of each rejected request showing **web request protocol 400** + body code equals VALIDATION ERROR; a follow-up database query proving no users row exists for that email. (Mechanism mirrors the commission Acknowledged @Assert yes → VALIDATION ERROR precedent, confirmed against Global error handler implementation reference.)
- **Dependencies:** REQ-005, REQ-006, REQ-007

---

## REQ-009 — When a visitor opens /terms, the system shall display the full Terms & Conditions copy from GH #200, including the company name + UEN and a contactable business address.

- **Type:** user interface
- **Priority:** Must
- **Source:** GH #200 (full Terms copy); #194 founder decision (KEEP address on Terms)
- **Rationale / value:** The placeholder Terms page must carry the real, enforceable Terms text, with a reachable legal contact (Personal Data Protection Act/clickwrap expectation). Address is intentionally retained here.
- **Owner (specialist):** frontend-expert (build) + Technical Writer (copy fidelity)
- **Acceptance criteria (Gherkin):**


- Scenario: Terms page shows the full Terms & Conditions.
- Given a visitor.
- When they navigate to /terms.
- Then all 16 sections from GH #200 are present (Acceptance of terms … Contact) including the SGD 100 liability cap and Governing law (Singapore).
- And the page shows the agency’s registered legal name (UEN the agency’s registration number) and the registered business address.


- **Required evidence:** Playwright @1280 and @390 screenshot using s of /terms covering the section headings, the contact block with the retained address, and the company/UEN; DOM/text check that the 16 section headings are present.
- **Dependencies:** none

---

## REQ-010 — When a visitor opens /privacy, the system shall display the full Privacy Policy copy from GH #199, including the DPO contact block with the retained business address.

- **Type:** user interface
- **Priority:** Must
- **Source:** GH #199 (full Privacy copy); #194 founder decision (KEEP address on Privacy)
- **Rationale / value:** The placeholder Privacy page must carry the real Personal Data Protection Act policy, and Personal Data Protection Act guidance expects the DPO contact (incl. address) to be publicly findable in the privacy policy. Address is intentionally retained here.
- **Owner (specialist):** frontend-expert (build) + Technical Writer (copy fidelity)
- **Acceptance criteria (Gherkin):**


- Scenario: Privacy page shows the full Privacy Policy with the DPO address.
- Given a visitor.
- When they navigate to /privacy.
- Then all 10 sections from GH #199 are present (Personal data we collect … Changes to this policy).
- And section 9 "Data protection officer" shows a DPO contact with the retained business address.
- And the page shows the agency’s registered legal name (UEN the agency’s registration number).


- **Required evidence:** Playwright @1280 and @390 screenshot using s of /privacy covering the 10 section headings and the DPO block with the retained address; DOM/text check of the section headings.
- **Dependencies:** none

---

## REQ-011 — The system shall remove the company street address from the public Footer.

- **Type:** user interface
- **Priority:** Must
- **Source:** GH #194 (remove address); founder narrowing (Footer + Contact only)
- **Rationale / value:** The founder wants the physical address off the general-purpose public surfaces; the Footer currently prints COMPANY registered Address in its operated-by line.
- **Owner (specialist):** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Footer no longer shows the street address.
- Given a visitor on any page with the global Footer.
- When they view the footer.
- Then the agency’s actual registered business address is not displayed.
- And the operated-by line still shows the legal name and UEN.


- **Required evidence:** Playwright @1280 and @390 screenshot of the footer; DOM/text assertion that the address string is absent and the legal name + UEN remain.
- **Dependencies:** none

---

## REQ-012 — The system shall remove the company street address from the public Contact page, while keeping the legal name and UEN.

- **Type:** user interface
- **Priority:** Must
- **Source:** GH #194; founder narrowing (Footer + Contact only)
- **Rationale / value:** The Contact page "Registered entity" row currently prints the full address; per #194 it must show only the legal entity + UEN (the email already serves as the contact channel).
- **Owner (specialist):** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Contact page no longer shows the street address.
- Given a visitor on /contact.
- When they view the Registered entity row.
- Then the agency’s actual registered business address is not displayed.
- And the row still shows the agency’s registered legal name (UEN the agency’s registration number).


- **Required evidence:** Playwright @1280 and @390 screenshot of /contact; DOM/text assertion that the address string is absent and legal name + UEN remain.
- **Dependencies:** none

---

## REQ-013 — While the address is removed from the Footer and Contact page, the system shall continue to display the company street address on the /privacy and /terms pages.

- **Type:** constraint
- **Priority:** Must
- **Source:** GH #194 founder decision (KEEP on Privacy & Terms for Personal Data Protection Act contactability)
- **Rationale / value:** This is the explicit guard against over-removal: a single COMPANY registered Address edit that nuked the address everywhere would break the Personal Data Protection Act DPO/business-address requirement. The address must remain reachable on the legal pages. (COMPANY registered Address must therefore stay in company implementation reference; only the Footer/Contact *usages* are removed.)
- **Owner (specialist):** frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: Address retained on legal pages after the Footer/Contact removal.
- Given the change removing the address from Footer and Contact has been applied.
- When a visitor views /privacy and /terms.
- Then the agency’s actual registered business address is displayed on both legal pages.


- **Required evidence:** Playwright @1280 screenshots of /privacy and /terms each showing the address present (the counterpart to REQ-011/012's "absent" assertions). This REQ exists specifically so the removal is verified as surgical, not global. The DPO/contact block on both pages must retain COMPANY registered Address (it already renders today: Privacy implementation reference, Terms implementation reference) — T12/T13 must not silently drop it during the full-copy rewrite.
- **Dependencies:** REQ-009, REQ-010, REQ-011, REQ-012

---

## REQ-014 — When a registration succeeds with the required consents, the system shall write a CONSENT GIVEN audit-log row recording the consent set and the policy versions, alongside the existing USER REGISTERED (and, for tutors, COMMISSION ACKNOWLEDGED) rows.

- **Type:** data
- **Priority:** Should
- **Source:** Personal Data Protection Act defensibility; existing audit pattern — Auth Service already saves USER REGISTERED (Auth Service implementation reference) and COMMISSION ACKNOWLEDGED (Auth Service implementation reference) rows in-transaction via Audit Log Repository
- **Rationale / value:** Column timestamps prove a consent flag is set, but an explicit audit event gives a single, queryable, append-only record of the whole consent decision (which consents, which policy versions, when, from which IP) for a DPA/regulator request — mirroring the audit rows the codebase already writes at registration. Append-only audit is stronger evidence than a mutable column.
- **Owner (specialist):** software-engineer (with data-engineer on the event shape)
- **Acceptance criteria (Gherkin):**


- Scenario: A CONSENT GIVEN audit row is written on successful registration.
- Given a successful tutee or tutor registration with the three required consents true.
- When the registration transaction commits.
- Then an audit log row exists with event type = "CONSENT GIVEN" for that user id.
- And its details JSON records terms Accepted, pdpa Consent, marketing Opt In, age Confirmed and the terms version + privacy version consented to.
- And it is saved in the same transaction as USER REGISTERED (consistent with Auth Service implementation reference INT-066 note).


- **Required evidence:** a private inspection of the user’s consent audit record, including the event, account identifier, consent choices and policy versions; confirmation the row is committed atomically with USER REGISTERED.
- **Dependencies:** REQ-001, REQ-005, REQ-006, REQ-015 (version values to record)
- **Note:** event type is a plain string on Audit Log event Type (no enum exists today) — use the literal CONSENT GIVEN as a shared constant, not a scattered raw string (no-magic-strings rule), consistent with how USER REGISTERED/COMMISSION ACKNOWLEDGED are used.

---

## REQ-015 — Registration records the exact displayed and accepted Terms and Privacy versions

- **Type:** data / application programming interface contract
- **Priority:** Should
- **Source:** original REQ-015 policy-version evidence requirement; clarified 2026-09-05.
- **Owner:** bff-specialist, software-engineer, frontend-expert, dba.
- **Contract:** the published policy bundle defines terms Version and privacy Version for the exact copy displayed. The frontend reads each from its single policy constant and carries both with the consent booleans, including through tutor Registration Draft to the tutor create request. The backend holds the authoritative current accepted pair and compares both submitted values for exact equality before account creation. It persists the submitted, validated values with server-set timestamps and the in-transaction consent audit. It must never replace an older submitted version with its current version and imply the user saw that copy.
- **Acceptance:**


- Scenario: Displayed current versions are accepted.
- Given the user sees and accepts the currently published Terms and Privacy copy.
- When either registration endpoint receives both displayed version IDs and required consents.
- Then it persists exactly those IDs, server timestamps and consent audit atomically.
- Scenario: A policy changes while the registration form or tutor draft is open.
- Given either submitted version differs from the server's current accepted version.
- When registration is attempted.
- Then it returns 409 POLICY VERSION STALE and creates no account or consent audit.
- And the user interface reloads the current policy copy, clears the required legal consent confirmations and asks the user to review and confirm again.
- And it retains non-secret registration fields and never retries with silently substituted versions.
- Scenario: Required version is missing.
- When either registration endpoint receives a missing or blank version ID.
- Then it returns 400 VALIDATION ERROR before account creation.


- **Required evidence:** both registration-path request/database/audit captures; stale tutor-draft and stale tutee-form tests; missing-version negative tests; matched the private implementation folder policy-release constants. Deploy copy, version constants, and backend accepted versions together. Constants may be mirrored across builds, but their exact equality is a release contract, not two independently chosen policy authorities.
- **Dependencies:** REQ-005/006/007/014. No change to the supplied legal text or founder-approved effective-date facts.

---

## Open questions for the spec-review gate (Phase 1.5 — feed back before build)

> **Status after the scoped spec-review (2026-06-20):** Q1, Q2, and Q3 are **RESOLVED** by Boss ruling (recorded as facts below / in REQ-002 + REQ-007 + REQ-014/015). Q4 is resolved and baked into REQ-006's three explicit ACs. Q5 remains a design-confirmation item for uiux/frontend.

### Resolved facts (Boss rulings — binding, no longer open)

- **Q2 (marketing vs WhatsApp) — RESOLVED.** marketing opt in is the MASTER marketing-consent gate; whatsapp opt in (INT-082) is channel-enablement UNDER it. WhatsApp **marketing** requires marketing opt in yes AND whatsapp opt in yes; the columns stay independent. Baked into REQ-002.
- **Q3 (legal placeholder values) — RESOLVED.** Effective date = **the deploy date**, stored as a **constant in company implementation reference** (single source — NOT an inline literal in both Privacy implementation reference and Terms implementation reference). DPO email = support email = **REDACTED email** (COMPANY contact Email). Business/DPO address = **COMPANY registered Address**. **Pre-launch follow-up (non-blocking for the dev build):** legal should supply a **dedicated DPO email** before PUBLIC launch — a generic admin@ mailbox is a Personal Data Protection Act weak point (the DPO contact should be a monitored, role-specific address). Track as a pre-launch residual.
- **Q1 (consent columns fresh) — RESOLVED.** No pdpa consent column exists today (only whatsapp opt in); 0058 introduces all four consent pairs + the two version columns fresh. Baked into REQ-007.
- **Q4 (tutor two-step consent carry) — RESOLVED.** Consents are collected on Register implementation reference Step 1; carried through tutor Registration Draft to the Commission implementation reference → register Tutor create call; not re-asked on the commission step. Baked into REQ-006 (AC-a/b/c).

### Still open (design confirmation)

- **Q5 (a11y — REQ-004):** Confirm each checkbox has an associated label and the granular Personal Data Protection Act purposes are a real  (not a sentence), for screen-reader clarity. uiux-designer/frontend-expert to confirm in the design.
