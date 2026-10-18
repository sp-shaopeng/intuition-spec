# Design — Legal and Compliance consent and policy pages (001)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> How the requirements in [requirements md](requirements.md) will be built. Owned with solution-architect
> for non-trivial design; record significant choices as ADRs (Context / Decision / Consequences).

## Architecture

Three independent, non-overlapping work lanes (parallelisable per the concurrency rule — distinct subtrees/files):

1. **Schema lane (backend, dba)** — one additive Liquibase changeset 0058 adds four consent column-pairs to
   users, mirroring the existing whatsapp opt in + chk whatsapp consent ts pattern from 0047. Serves
   REQ-007, foundation for REQ-001/002/005/006/008.

2. **Register-application programming interface lane (backend, bff-specialist + software-engineer)** — extend the two request data transfer objects
   (Register Tutee Request, Register Tutor Request) with the consent fields, add bean-validation guards
   (@Assert yes for the three required), and persist them through Auth Service build User* / the register paths
   with **server-set** timestamps, the policy-version values (REQ-015), and a CONSENT GIVEN audit row written
   in-transaction (REQ-014). Serves REQ-005/006/008/014/015 (+ the persistence half of REQ-001/002).

3. **Frontend lane (frontend-expert + uiux-designer + Technical Writer)** — three independent file groups:
   - **Register implementation reference + Commission implementation reference + tutor Registration Draft implementation reference** — replace the single pdpa Consent checkbox
     on Register implementation reference Step 1 with the four #198 checkboxes (3 required Zod refinements, 1 optional); Step 1
     **blocks "Continue to commission terms →"** if a required box is unticked. For the tutee path the four
     booleans go straight into the register Tutee payload. For the **tutor** path the account is created later
     on Commission implementation reference (POSTs application programming interface ROUTES auth.register Tutor ~line 90, payload built ~lines 79-87), so the
     four booleans must be carried in tutor Registration Draft (tutor Registration Draft implementation reference)
     and added to the Commission implementation reference payload — **not** re-asked on the commission step. Serves REQ-003/004/006
     (+ frontend half of REQ-001/002).
   - **Privacy implementation reference / Terms implementation reference** — replace placeholder copy with the full #199/#200 text via Article Layout.
     Serves REQ-009/010 (+ REQ-013 retention).
   - **Footer implementation reference / Contact implementation reference** — remove the COMPANY registered Address usage. Serves REQ-011/012/013.

**Reused patterns/utilities (do not reinvent):**
- Consent column pattern: whatsapp opt in BOOLEAN required default no + whatsapp opt in at TIMESTAMPTZ
  + require flag no OR flag at has a value — changeset 0047, mirrored entity fields in User implementation reference.
- @Assert yes required-consent validation: already used for commission Acknowledged in Register Tutor Request.
- frontend form: react-hook-form + zod a validation rule requiring the consent value to be true (today's pdpa Consent); application programming interface ROUTES auth.*
  (no raw paths); tutor Registration Draft in-memory draft for the tutor two-step flow.
- Content pages: Article Layout + Article Layout Section; company facts from the single source company implementation reference.

## Data model

New columns on users (changeset **0058**, additive, all booleans required DEFAULT no; each timestamp
TIMESTAMP WITH TIME ZONE nullable):

| Column | Type | REQ | Notes |
|---|---|---|---|
| terms accepted | boolean required default false | REQ-001/005/006/007 | #198 (a) Terms+Privacy agreement |
| terms accepted at | timestamptz null | REQ-001/007 | audit timestamp |
| pdpa consent | boolean required default false | REQ-001/005/006/007 | #198 (b) granular Personal Data Protection Act data-use |
| pdpa consent at | timestamptz null | REQ-001/007 | audit timestamp |
| marketing opt in | boolean required default false | REQ-002/007 | #198 (c) OPTIONAL |
| marketing opt in at | timestamptz null | REQ-002/007 | set only when opted-in |
| age confirmed | boolean required default false | REQ-001/005/006/007 | #198 (d) 18+/guardian |
| age confirmed at | timestamptz null | REQ-001/007 | audit timestamp |
| terms version | text null | REQ-015 | Terms copy version consented to |
| privacy version | text null | REQ-015 | Privacy copy version consented to |

Four require constraints (one per consent pair), e g.:
a database constraint requiring a consent timestamp whenever terms have been accepted
(and chk pdpa consent ts, chk marketing consent ts, chk age consent ts). Each database migration step that runs an
require the rule is preceded by a three-second database lock timeout for the migration (parity with 0047 lines 36-40). The
require the rule performs a full-table validate scan on a populated users — negligible on the dev
recreate-on-deploy stack, noted for the production cutover (dba residual D3). Each changeset has a matching
the reviewed database rollback dropping the constraints + all ten columns (mirrors 0047). terms version / privacy version
have no require (they are free-text version tags, populated only from the submitted displayed versions after exact comparison against the backend current accepted pair — REQ-015).

User implementation reference gains the four boolean + four Offset Date Time fields with @Column + @Builder Default no,
identical to the whatsapp Opt In / whatsapp Opt In At pair, plus two nullable text fields terms Version /
privacy Version (REQ-015). **marketing opt in is the MASTER marketing-consent gate; whatsapp opt in
(INT-082) is channel-enablement UNDER it** (Boss ruling on Q2, baked into REQ-002): WhatsApp **marketing**
requires marketing opt in yes AND whatsapp opt in yes; the two columns stay independent in the schema.

No table is dropped or row physically deleted (soft-delete rule N/A here — purely additive).

## application programming interface / contracts

Endpoints unchanged (application programming interface ROUTES auth.register Tutee = auth register tutee interface operation,
application programming interface ROUTES auth.register Tutor = auth register tutor interface operation). Request bodies extend:

Register Tutee Request (consents plus displayed version IDs):


Implementation example omitted. This public edition retains the written requirements in this section.


Register Tutor Request — same consent and version fields added, keeping existing commission Acknowledged + commission Terms Version.

- **Timestamps are server-set** (the client sends consent booleans and displayed version IDs, never timestamps). On register, the service sets
  the now operation for each flag that is yes; marketing opt in at stays null when no.
  The service compares both submitted versions exactly with the authoritative current accepted pair before creating the account, rejects mismatch with 409 POLICY VERSION STALE, and persists the submitted validated terms version / privacy version (REQ-015)
  and writes a CONSENT GIVEN audit row in-transaction with USER REGISTERED (REQ-014).
- Success: existing Auth User Response shape — no response change.
- **Negative/guard (REQ-008) — ONE deterministic mechanism:** each required consent is a data transfer object @Assert yes
  field (mirroring commission Acknowledged @Assert yes in Register Tutor Request implementation reference). A no/absent
  value raises Method Argument Not Valid Exception, which Global error handler handle Validation
  (Global error handler implementation reference) maps **deterministically to web request protocol 400, body  code:
  a validation-error response containing the required-consent message ** — confirmed against the handler. No account is
  created (validation is pre-service). We deliberately do **NOT** add a second service-thrown
  the bad request operation for the consents: that would re-create the dual-mechanism
  ambiguity the gate flagged (bean @Assert yes 400 vs the service-thrown COMMISSION ACK REQUIRED 400 that
  Auth Service register Tutor ~line 188 still throws for commission only). One mechanism, one code
  (VALIDATION ERROR) for all three consents.

## Error handling

| Failure | Where | Result |
|---|---|---|
| Required checkbox unticked (frontend) | Register implementation reference Zod .refine | submit blocked, inline message on the box, no network call (REQ-003) |
| Required consent false/missing (application programming interface) | @Assert yes bean validation → Global error handler handle Validation (159-173) | deterministic web request protocol 400, body code: VALIDATION ERROR, no account (REQ-008) — single mechanism, no service-thrown CONSENT REQUIRED |
| Required consent unticked on tutor Step 1 (frontend) | Register implementation reference Step 1 Zod .refine | "Continue to commission terms →" blocked, inline message, no draft set / no nav (REQ-006 AC-b) |
| Flag true but timestamp null (database) | require chk * consent ts | constraint violation — defence-in-depth, should never fire from app code (REQ-007) |
| Address accidentally removed from legal pages | Playwright assertion in REQ-013 | caught in review — removal must be surgical (Footer/Contact only) |

## Testing strategy

- **REQ-007 (schema):** the documented operational check applies 0058 via Testcontainers; an integration/repository test asserts the
  columns + defaults and that a timestamp-less yes update records is rejected. Evidence: databasechangelog row +
  \d users + require-violation log.
- **REQ-005/006/008 (application programming interface):** controller/service tests (and Abstract Api IT live-stack checks) for: all-consents
  → persisted with timestamps; each required consent false → 400 + no row. Evidence: curl logs (happy + each
  negative path) + database row. Backend gate ≥97.5% line AND branch (new data transfer object branches + service branches covered).
- **REQ-001/002 (persistence):** assert the persisted booleans + timestamp nullability for opted-in vs opted-out.
- **REQ-014/015 (audit + versioning):** assert a CONSENT GIVEN audit log row is written in-transaction with
  USER REGISTERED (details JSON = consent set + policy versions), and that terms version / privacy version
  are persisted from the single frontend policy-version constant. Tutor path additionally asserts the four consent
  booleans survive the tutor Registration Draft round-trip into the Commission implementation reference register payload (REQ-006).
- **REQ-003/004 (form):** Vitest unit/RTL — four checkboxes render with #198 wording; submit blocked when a
  required box is unticked; optional box leaves submit enabled. frontend gate ≥97.5% line / ≥88% branch (note: RTL
  aria-pressed/checkbox assertions use to Have Attribute/to Be Checked correctly per project lessons).
- **REQ-009/010/011/012/013 (content):** Vitest text-presence tests + live Playwright @1280/@390 screenshots
  (the evidence). REQ-013 is the explicit "address still present on /privacy & /terms" counter-assertion.

Each REQ's Gherkin scenario maps 1:1 to a test or a live capture named in its **Required evidence** line.

## Migration / rollout

- Single additive changeset 0058; no backfill (existing rows default FALSE with null timestamps — pre-existing
  accounts simply have no recorded consent, which is accurate). Backward-compatible; the reviewed database rollback provided.
- No feature flag — consent capture is mandatory from deploy. Content/address changes are static frontend edits.
- Rollout order: **migrate (0058) → deploy backend (data transfer object+service) → deploy frontend**. The booleans default FALSE
  so a backend deployed before the frontend sends them still validates correctly only once the frontend sends yes;
  therefore ship frontend + backend together (the @Assert yes guard would 400 a frontend that doesn't yet send the fields).
- COMPANY registered Address stays in company implementation reference (REQ-013 depends on it for the legal pages); only the
  Footer/Contact usages are deleted.

## Decisions (ADR log)

- 2026-06-20 — **Boolean+timestamp+require per consent, not a separate consent table.** Context: could model
  consents as rows in a user consents audit table. Decision: mirror the existing whatsapp opt in column
  pattern (0047) — flat columns on users + require. Consequences: consistent with the codebase, simplest to
  query/persist, sufficient for the single capture-at-registration event; if per-consent history/withdrawal
  auditing is later required, a user consents ledger can be added without breaking these columns. (Revisit if
  Personal Data Protection Act withdrawal-with-history per #199 §8 becomes a tracked requirement.)
- 2026-06-20 — **Server sets consent timestamps; client sends booleans and displayed policy-version IDs.** Context: trusting a
  client-supplied timestamp is forgeable. Decision: the now operation set in the service. Consequences: audit
  timestamps are trustworthy; matches the whatsapp opt in at precedent.
- 2026-06-20 — **Address removal is surgical (Footer + Contact only), guarded by REQ-013.** Context: a single
  edit to COMPANY registered Address would remove it everywhere and break Personal Data Protection Act contactability. Decision: edit
  only the Footer/Contact *usages*, keep the constant, and add REQ-013 to assert retention on the legal pages.
- 2026-06-20 (v1.1) — **ONE deterministic reject mechanism for the consents: @Assert yes → 400
  VALIDATION ERROR.** Context: the codebase has two reject patterns — bean @Assert yes (→ VALIDATION ERROR
  via Global error handler) and a service-thrown the bad request operation (e g.
  COMMISSION ACK REQUIRED in Auth Service ~line 188). Shipping both for the consents would make the rejected
  body code non-deterministic. Decision: the three required consents use **only** @Assert yes (mirroring the
  commission Acknowledged @Assert yes precedent in Register Tutor Request implementation reference); the asserted code is
  VALIDATION ERROR. Consequences: a single, testable failure shape; no new error code for required-checkbox rejection. The separate stale-policy-version rejection is409 POLICY VERSION STALE under the2026 correction.
- 2026-06-20 (v1.1) — **Policy versions stored on a single frontend constant, recorded per registration.** Context: a
  consent timestamp can't prove which policy text was agreed to, and the copy will change. Decision: add
  terms version / privacy version columns (0058), populate from one POLICY VERSIONS-style constant in
  company implementation reference (mirroring COMMISSION TERMS VERSION), and record an append-only CONSENT GIVEN
  audit row. Consequences: version-provable consent (REQ-014/015); no inline literals duplicated across pages.

## 2026-09-05 policy-version contract clarification

Both frontend registration paths carry the displayed terms Version/privacy Version; the tutor draft preserves them unchanged. Blank/missing IDs use data transfer object 400 VALIDATION ERROR; stale IDs use the shared POLICY VERSION STALE code (409), not the required-checkbox validation path. No registration, audit or session cookie is produced on rejection. The user interface reloads current policy content and requires fresh legal confirmations; it keeps non-secret fields and re-prompts for credentials where needed. Never auto-update a draft's IDs or silently retry acceptance. Verify a policy release between Step 1 and Commission as well as an already-open parent form. Both builds release the same policy bundle/version pair together.
