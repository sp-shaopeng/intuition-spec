> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

---
id: 001
slug: legal-and-compliance-consent-and-policy-pages
title: Legal and Compliance consent and policy pages
version: 1.3
status: HISTORICALLY MARKED IMPLEMENTED            # DRAFT → APPROVED → IN-PROGRESS → HISTORICALLY MARKED IMPLEMENTED  (or SUPERSEDED)
owner: requirements-engineer         # the engineer who owns this change (requirements-engineer for cross-cutting/product specs)
created: 2026-06-20
updated: 2026-09-05
---

> **2026-09-05 requirements correction (documentation only).** Policy consent now explicitly uses the displayed-version/exact-server-match contract, including stale-form rejection and renewed confirmation.
> The status and shipment/evidence entries below record the earlier implementation. This revision does not
> certify that the corrected contract is implemented, tested, or deployed; follow-up verification remains open.


# Legal and Compliance consent and policy pages

Brings the signup flow and the legal content pages to Personal Data Protection Act-/Terms-compliance. At registration the user is
shown **four consent checkboxes** (GH #198): Terms-of-Use + Privacy-Notice agreement [required], granular
Personal Data Protection Act data-use consent [required], marketing opt-in [**optional**], and a 18+/parent-guardian confirmation
[required]. Required consents block submission on the form and are enforced server-side (no consent-less
account can be created), and every consent is **persisted with an audit timestamp** on users via Liquibase
changeset 0058 (mirroring the existing whatsapp opt in boolean+timestamp+require pattern), plus a
CONSENT GIVEN audit row and the terms version/privacy version policy-version columns for Personal Data Protection Act
defensibility. The **full Privacy Policy** (GH #199) and **Terms & Conditions** (GH #200) copy is published on
/privacy and /terms. Per founder decision on GH #194, the company **street address is removed from the
public Footer and Contact page only** and **retained on the Privacy and Terms pages** (Personal Data Protection Act requires a
contactable DPO/business address). The tutor path captures consents on Register implementation reference Step 1 and carries them
via tutor Registration Draft into the Commission implementation reference account-create call (the #1 data-flow risk). Value: an
enforceable, auditable, version-stamped consent record and compliant public legal surfaces, closing the
Personal Data Protection Act/clickwrap gap before launch. 15 atomic requirements (REQ-001…REQ-015).

- **Requirements:** [requirements md](requirements.md) — EARS statements + Gherkin AC (each REQ-NNN)
- **Design:** [design md](design.md) — architecture, data model, error handling, testing, migration
- **Tasks:** [tasks md](tasks.md) — implementation tasks, each traced to a REQ-NNN + owning specialist
- **Evidence:** the private evidence directory for the requirement — captured proof per acceptance scenario

## Status

HISTORICALLY MARKED IMPLEMENTED — built, gated, evidence-complete, and team-reviewed (Boss SHIP verdict 2026-06-21). Live on the local
dev stack; pushed to the polyrepo sp-dev. See Changelog 1.2. Follow-ups (non-blocking) tracked on the delivery
board: F5 Testcontainers/Docker-29 local-skip (devops); marketing→WhatsApp AND-gate enforcement before any
WhatsApp-marketing enablement (security); 0058 prod-cutover lock note (runbook); consent-checkbox a11y@390;
pre-existing-user re-consent path before public launch; the private implementation folder policy-version lockstep doc.

## Review & Sign-Off

- [x] Counterpart — security / Personal Data Protection Act & consent (security-expert) — SIGN-OFF (conditions applied v1.1) / 2026-06-20
- [x] Counterpart — signup form & consent user experience (uiux-designer) — SIGN-OFF (conditions applied v1.1) / 2026-06-20
- [x] Counterpart — register application programming interface contract (bff-specialist) — SIGN-OFF (reject-code resolved v1.1) / 2026-06-20
- [x] Counterpart — content pages & footer/contact (frontend-expert) — SIGN-OFF (file path + footer strings fixed v1.1) / 2026-06-20
- [x] Counterpart — consent schema (dba) — SIGN-OFF (lock timeout + version cols v1.1) / 2026-06-20
- [x] Technical Writer — legal copy (#199/#200) fidelity & placeholders — SIGN-OFF (values resolved v1.1) / 2026-06-20
- [x] Boss (Rex) — APPROVED / 2026-06-20. Pre-launch follow-up (non-blocking): legal to supply a dedicated DPO email (replace [REDACTED EMAIL]) before public launch.

## Changelog

- 1.2 (2026-06-21) HISTORICALLY MARKED IMPLEMENTED — built (backend+frontend parallel lanes), gated, evidence-complete, team-reviewed → Boss SHIP.
  backend: migration 0058 (10 consent cols + 4 chk * consent ts CHECKs), User implementation reference fields, Register Tutee/Tutor Request
  @Assert yes consents + marketing Opt In, Auth Service apply Consents (server-set timestamps + Policy Versions
  versions) + atomic CONSENT GIVEN audit row. frontend: 4 #198 checkboxes in Register implementation reference with tutor-draft carry into
  Commission implementation reference, POLICY VERSIONS constant, full #199 Privacy / #200 Terms pages, address removed from
  Footer/Contact and retained on legal pages (#194). Gates: backend 1878 unit + jacoco-check green (Boss-reverified);
  frontend 2376 tests 98.27% line / 92.89% branch. Evidence REQ-001…015 all PASS under evidence implementation reference (register persist +
  server timestamps + versions, CONSENT GIVEN audit, 400 VALIDATION ERROR + no-row on missing consent, live database
  require-constraint proof, Playwright @1280/@390 of all four pages). REQ-007 proven via **live psql** (Testcontainers
  can't run vs Docker 29.5.2 locally — follow-up F5). Closes #198/#199/#200/#194.
- 1.1 (2026-06-20) MINOR — scoped spec-review SEND-BACK fixes (status stays DRAFT; Boss re-gates):
  - MODIFIED tasks md — corrected the tutor commission-step file path from the non-existent Register Commission* implementation reference to the real Commission implementation reference (POSTs register Tutor , payload ) across every task; added the tutor Registration Draft implementation reference carry.
  - MODIFIED REQ-006 — made the tutor consent data-flow explicit (the #1 risk): AC-a draft carries the 4 consent booleans; AC-b Step 1 blocks "Continue to commission terms →" when a required box is unticked; AC-c Commission implementation reference payload includes the booleans; declared as an explicit frontend dependency.
  - MODIFIED REQ-008 + design — RESOLVED the reject mechanism to ONE deterministic pattern: @Assert yes → Global error handler handle Validation → web request protocol 400 code:VALIDATION ERROR (confirmed Global error handler implementation reference; mirrors commission Acknowledged @Assert yes); removed the "400 VALIDATION ERROR / CONSENT REQUIRED" ambiguity and the dual service-thrown path.
  - MODIFIED tasks T14/T15 — specify clean footer and Contact statements after address removal, retaining the actual legal operator and registration number without orphaned commas or repeated punctuation.
  - MODIFIED tasks T12/T13 + REQ-013 — inline instruction that the Privacy (19) / Terms (18) DPO/contact block RETAINS COMPANY registered Address through the full-copy rewrite.
  - RESOLVED Q1–Q4 as Boss-ruled facts: effective date = deploy date stored as a single company implementation reference constant; DPO/support email = REDACTED email; address = COMPANY registered Address; pre-launch follow-up to get a dedicated DPO email (non-blocking). Marketing-consent precedence baked into REQ-002 (marketing opt in MASTER, whatsapp opt in channel-under-it; both required for WhatsApp marketing; columns independent).
  - ADDED REQ-014 — CONSENT GIVEN audit-log event (consent set + policy versions), mirroring USER REGISTERED/COMMISSION ACKNOWLEDGED.
  - ADDED REQ-015 — terms version/privacy version recorded at registration from a single source-of-truth constant.
  - MODIFIED REQ-007 / design / T1 — 0058 also adds the two version columns; each require the rule database migration step guarded by SET LOCAL lock timeout is 3s (parity with 0047); noted the full-table validate scan.
- 1.0 (2026-06-20) initial draft
