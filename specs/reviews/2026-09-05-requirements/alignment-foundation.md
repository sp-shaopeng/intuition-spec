# Foundation and domain specification alignment

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](corrections.md) and [current requirements](../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Date: 2026-09-05. Requirements comparison only. Later explicit decisions govern only the clauses they replace. User-confirmed filtering and agency-receipt semantics take precedence. Source code and historical deployment claims do not silently amend requirements.

## Findings

### F01 — Anonymous filter controls
**Classification:** Conflict with confirmed requirement.

The browse flow instructs the user interface to hide filters before login. The user confirmed anonymous limited filtering. Change that flow and its acceptance expectations; keep authenticated detail/action gates and bounded pagination.

Evidence: [23-public-browse-with-filters md](../../sequences/23-public-browse-with-filters.md); [m17-marketplace-ux-overhaul md](../../architecture/m17-marketplace-ux-overhaul.md); [audit md](audit.md).

### F02 — Parent lesson payment is not agency receipt
**Classification:** Conflict with confirmed requirement.

M18 design and product requirements document shorthand say parent pays then admin marks commission PAID. The user confirmed PAID means receipt by the agency of tutor commission. Replace the causal shorthand everywhere; retain parent-to-tutor payment as a separate event.

Evidence: [assignment-lifecycle-design md](../../architecture/assignment-lifecycle-design.md); [product requirements document md](../../PRD.md); [product requirements document md](../../PRD.md); [spec md](../../modules/commission/spec.md).

### F03 — Match completion event
**Classification:** Explicitly superseded, stale body.

The matching module defines COMPLETED on first-session confirmation. The authoritative M18 note defines COMPLETED on commission PAID. First-session confirmation still exists as a separate commission milestone; changing its effect does not delete its requirement.

Evidence: [spec md](../../modules/matching/spec.md); [spec md](../../modules/matching/spec.md); [product requirements document md](../../PRD.md); [assignment-be-spec md](../../architecture/assignment-be-spec.md).

### F04 — Contact disclosure boundary
**Classification:** Explicitly superseded, stale body.

Old product requirements document/business requirements document, matching module and user experience flows promise no counterparty contact ever. M18 explicitly extends BR-001 with post-assignment disclosure. Update all copy and permission tables to the assignment boundary; post approval and parent selection alone do not qualify.

Evidence: [product requirements document md](../../PRD.md); [business requirements document md](../../BRD.md); [spec md](../../modules/matching/spec.md); [user-flows md](../../domain/uiux/user-flows.md); [product requirements document md](../../PRD.md); [assignment-be-spec md](../../architecture/assignment-be-spec.md).

### F05 — Listing-free apply and parent applicant view
**Classification:** Explicitly superseded, stale body.

The applications module still requires an ACTIVE listing and says parents never see applicant identities. M18 removes the listing gate and adds a parent-owned personal information-free applicant view. Public applicant counts remain excluded by the folded decision; early paragraphs in the same module still promise those counts.

Evidence: [spec md](../../modules/applications/spec.md); [spec md](../../modules/applications/spec.md); [product requirements document md](../../PRD.md); [product requirements document md](../../PRD.md).

### F06 — Interview scope and state model
**Classification:** Explicitly superseded, stale body.

The interview module still uses a per-tutee cap and REQUESTED-only negotiation. Later decisions use three concurrent interviews per parent posting, add negotiation states, and allow PENDING parent requests to initiate. Preserve the later rules and update diagrams/guards without reopening the settled cap.

Evidence: [spec md](../../modules/interview/spec.md); [spec md](../../modules/interview/spec.md); [max-interviews-per-posting md](../../modules/interview/max-interviews-per-posting.md); [int-039-interview-negotiation md](../../architecture/int-039-interview-negotiation.md); [spec md](../../021-tutee-request-flow/spec.md).

### F07 — Commission lifecycle and scope
**Classification:** Unreconciled baseline.

business requirements document/product requirements document and the commission module require confirmation, invoice, dispute and waiver states. The code-derived flow document describes direct PENDING-to-PAID. M18 expressly preserves the earlier money path. Treat this as requirement-to-delivery reconciliation; user interface deferral for disputes/waivers does not remove their P0 state-model obligation.

Evidence: [product requirements document md](../../PRD.md); [spec md](../../modules/commission/spec.md); [assignment-be-spec md](../../architecture/assignment-be-spec.md); [11-admin-mark-commission-paid md](../../sequences/11-admin-mark-commission-paid.md).

### F08 — Session duration
**Classification:** Explicitly superseded, stale body.

The domain application programming interface says 24-hour tutor/tutee signed session token, the auth module says seven days, and spec004 requires 30-minute access plus refresh. The latter is the current requirement. The admin Redis idle-session model remains distinct.

Evidence: [api-spec md](../../domain/backend/api-spec.md); [spec md](../../modules/auth/spec.md); [requirements md](../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md).

### F09 — Contact storage and erasure schema
**Classification:** Explicitly superseded, stale body.

Schema/index/migration/sequence docs still use encrypted/keyed message authentication email and phone columns, although the explicit plaintext amendment replaces those fields. Some also direct physical account removal, contrary to the standing soft-delete policy. Update obsolete columns/procedures; resolve retention timing separately.

Evidence: [schema md](../../domain/database/schema.md); [indexes md](../../domain/database/indexes.md); [migrations md](../../domain/database/migrations.md); [27-encrypted-pii-write-and-lookup md](../../sequences/27-encrypted-pii-write-and-lookup.md); [plaintext-email-phone md](../../architecture/plaintext-email-phone.md); [int-094-batch-soft-delete-integrity-spec md](../../architecture/int-094-batch-soft-delete-integrity-spec.md).

### F10 — Migration tool and commands
**Classification:** Direct engineering contradiction.

The migration strategy calls the tool Spring Data Java persistence mapping Migrate but directs schema prisma and prisma migrate commands. The accepted ADR requires Liquibase XML changesets only. This is unsafe as a build instruction and needs replacement with the existing Liquibase workflow.

Evidence: [migrations md](../../domain/database/migrations.md); [ROADMAP md](../../ROADMAP.md).

### F11 — Quality gates
**Classification:** Explicitly superseded, stale body.

Domain test/workflow docs still require 97.5% frontend branches; the full product plan contains an 85/80 gate. Accepted ADR005 requires 97.5% frontend lines and 88% branches. Correct these execution instructions and link the current gate once.

Evidence: [test-strategy md](../../domain/backend/test-strategy.md); [dev-workflow md](../../domain/dev-workflow.md); [FULL-PRODUCT-PLAN md](../../FULL-PRODUCT-PLAN.md); [ROADMAP md](../../ROADMAP.md).

### F12 — Routes, auth and pagination contracts
**Classification:** Partially maintained reference.

The page inventory says its route list is still valid but lists public tutor detail. Domain application programming interface also lists a public slug endpoint and cursor envelopes while M17 gates detail and accepted ADR001 uses page metadata. Reference docs should identify endpoint-specific current contracts and archive obsolete examples, rather than mix generations.

Evidence: [pages md](../../domain/frontend/pages.md); [pages md](../../domain/frontend/pages.md); [api-spec md](../../domain/backend/api-spec.md); [api-spec md](../../domain/backend/api-spec.md); [m17-marketplace-ux-overhaul md](../../architecture/m17-marketplace-ux-overhaul.md); [ROADMAP md](../../ROADMAP.md).

### F13 — Selection versus interview promise
**Classification:** Partial cross-feature coverage.

user experience design-direction says the next step after choosing is that the agency will set up an interview. M18 describes choose-tutor OR request-interview routes before admin assignment. Define the success message from the actual chosen path; do not promise an automatically created interview unless that action is part of the contract.

Evidence: [design-direction md](../../domain/uiux/design-direction.md); [product requirements document md](../../PRD.md).

### F14 — Master index is incomplete and misleading
**Classification:** Traceability/authority gap.

requirements traceability matrix indexes only001–010 and carries contradictory draft notes for007/009. README calls older sequence counts and migration numbers current. Historical status notes are not release evidence; the source catalog/numbered matrix should replace this index after review.

Evidence: [requirements traceability matrix md](../../RTM.md); [requirements traceability matrix md](../../RTM.md); [README md](../../../README.md).

### F15 — Operational SLOs
**Classification:** Measurement contract conflict.

product requirements document and business requirements document use different admin business-day calendars; specific interview notices have a five-minute target while a general table assigns other events thirty minutes. Keep these as measurement decisions requiring reconciliation, not as evidence the chosen production topology is wrong.

Evidence: [product requirements document md](../../PRD.md); [product requirements document md](../../PRD.md); [business requirements document md](../../BRD.md).

### F16 — First-month amount
**Classification:** Explicit amendment plus open commercial detail.

INT157 locks an explicit plan at activation and preserves the due trigger, superseding mutable fee calculations. Its 4.33 multiplier and rate clamp remain explicitly founder-open. Do not classify the entire commission spec as either fully approved or wholly undecided.

Evidence: [spec md](../../modules/commission/spec.md); [int-157-commission-first-month-locked-spec md](../../architecture/int-157-commission-first-month-locked-spec.md); [int-157-commission-first-month-locked-spec md](../../architecture/int-157-commission-first-month-locked-spec.md).

## Domain document coverage

| Document | Verdict | Inspection depth | Assessment |
|---|---|---|---|
| [domain/the private implementation folder](../../domain/backend/api-spec.md) | conflict | Full read | Mixed 24-hour auth, cursor contract, public detail and newer012/021 sections. F08/F12; first-session route remains documented. |
| [domain/the private implementation folder](../../domain/backend/test-strategy.md) | partial | Full read | Real database and role/privacy tests align; frontend branch threshold and old contract examples need reconciliation. F11. |
| [domain/database/schema md](../../domain/database/schema.md) | conflict | Schema/enum/identity/lifecycle/erasure sections | Old CLOSED/APPROVED enum vocabulary, encrypted identities and hard-delete instructions conflict with later baselines. F03/F09. |
| [domain/database/indexes md](../../domain/database/indexes.md) | partial | Index inventory, predicates and relevant query patterns | Historical keyed message authentication and independent subject/level indexes do not describe later identity/matrix amendments. F09; retain applicable cardinality constraints. |
| [domain/database/migrations md](../../domain/database/migrations.md) | conflict | Tool/workflow, ledger, erasure and amendment sections | Prisma commands conflict with Liquibase-only ADR; erasure and encryption procedures obsolete; later0033–0037 ledger does not update earlier sections. F09/F10. |
| [domain/dev-workflow md](../../domain/dev-workflow.md) | partial | Amendments and contract/testing/deployment gates | Adopted lab-first amendments need to replace contradictory old stages; frontend branch gate differs from ADR005. F11. |
| [domain/the private implementation folder](../../domain/frontend/pages.md) | conflict | Full read | Layout supersession banner is useful, but its assertion that routes stay valid is false for auth gates and moved routes. F12. |
| [domain/the private implementation folder](../../domain/frontend/tech-stack.md) | partial | Full read | HttpOnly ownership and Singapore time formatting align. Old raw401 redirect, public-detail rendering, hosting and test-tool examples require newer references. |
| [domain/uiux/design-direction md](../../domain/uiux/design-direction.md) | partial | Full read | Shared states/navigation/accessible controls align with user interface requirements. Post-choose interview promise and choice/assignment wording need path-specific correction. F13/F04. |
| [domain/uiux/design-tokens md](../../domain/uiux/design-tokens.md) | partial | Normative token/visibility/accessibility and component sections | Preserved brand intent aligns; state vocabulary, coarse availability and first-session examples need mapping to current models. This is not a measured accessibility audit. |
| [domain/uiux/user-flows md](../../domain/uiux/user-flows.md) | conflict | Flow inventory and requirement-bearing steps/branches | Old privacy promise, registration/verification arc, commission flow and matching assumptions need explicit effective-version labels. F02/F04/F07. |
| [domain/uiux/wireframes md](../../domain/uiux/wireframes.md) | conflict | Screen inventory, action/permission/state/copy constraints | Public tutor detail and no-Apply rules are superseded; contact-copy and commission screens still mix generations. Wireframe pixel geometry was not revalidated. F04/F05/F12. |

## Product source coverage

product requirements document/business requirements document were assessed as requirement baselines, including their role, money, visibility, retention, acceptance and M18 amendments. ROADMAP decision records were checked for explicit supersession. FULL-PRODUCT-PLAN retains historical implementation instructions (including older coverage gates). SCOPE, business-update, status-briefing and work-plan are dated status snapshots; business-case-next and roadmap-next contain recommendations, backlog and folded updates. They are not interchangeable with an approved current acceptance baseline. requirements traceability matrix and README were checked as traceability indexes. Earlier review artifacts supply context and do not approve new policy.

Inspection used full reads for short files and targeted normative-section/transition scans for long historical artifacts. Supporting examples, archived reports and pixel geometry are not certified by a per-document verdict.
