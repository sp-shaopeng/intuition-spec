# Specification correction report — 2026-09-05

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

The reviewed contradictions have been corrected across the requirements, designs, task lists, module contracts, application programming interface references and user flows. The [current requirements and decision register](../../CURRENT-REQUIREMENTS.md) is the starting point; the [requirements traceability matrix](../../RTM.md) indexes all **27 packages and 170 named requirements**.

This is a documentation correction. Application code, infrastructure, deployments and external notification delivery were not changed. Historical HISTORICALLY MARKED IMPLEMENTED/APPROVED labels remain dated source history; they do not certify the newly clarified acceptance criteria.

## Main corrections

| Area | Current requirement |
|---|---|
| Guest browsing | Guests can filter public listings without login. Results are bounded to pages 0–4, at most 20 per page; this is a result cap per query, not a lifetime search quota. Detail/action gates remain explicit. |
| Commission | The tutor owes the agency. PAID records the agency receiving that commission. Parent lesson payment and an unpaid dispute decision cannot mark it received. |
| Match lifecycle | Admin assignment creates the match; activation locks the agreed first-month plan; commission PAID completes the match. First-session confirmation and invoicing requirements remain. |
| Contact access | Approval, interviews, invitations and choosing do not release contact. Assigned participants use the authorized endpoint; frontend reveal is ACTIVE/COMPLETED only. |
| Tutor participation | A published tutor listing is optional. Canonical profile completeness and scoped parent applicant views remain required. |
| Account changes | Thirty-minute user access plus refresh; HttpOnly server logout and cookie-preserving 429 handling. Contact changes use the correct email destination, explicit confirmation and independently authorized revocation. |
| Reminders and notifications | Atomic in-app/outbox intent, durable per-generation reminder deduplication, stale-generation guards, recoverable registration and compatible rollback. Disabled delivery is not success. |
| Data and operations | Plaintext-contact amendment, logical deletion, Liquibase XML, consistent quality gates, retained service targets and Amazon Web Services production provenance. |

Recovered the original [approved Amazon Web Services v3 plan](../../027-aws-production-deployment/evidence/approved-plan-v3.md) and added an [approved-decision extract](../../027-aws-production-deployment/approved-decisions.md) incorporating later phase amendments. The recovered file matches its local historical source by SHA-256; historical cost/recovery estimates are not presented as current measurements.

## Coverage and disposition

All **135 distinct findings** in the preceding alignment audit have a matching disposition. The correction set contains 229 source documentation files (226 modified plus 3 new); audit reports and correction ledgers are additional review artifacts. Unchanged, already aligned packages/clauses were retained. Historical flow diagrams remain clearly labelled beneath corrected current contracts.

| Review scope | Detailed disposition ledger |
|---|---|
| Numbered 001–009 and auxiliary set 1 | [Ledger](corrections/001-009.md) |
| Numbered 010–018 | [Ledger](corrections/010-018.md) |
| Numbered 019–027 | [Ledger](corrections/019-027.md) |
| Auxiliary set 2 | [Ledger](corrections/aux2.md) |
| Auxiliary set 3 | [Ledger](corrections/aux3.md) |
| Foundation, operations and all 43 sequences | [Ledger](corrections/foundation-operations-sequences.md) |

Three specialist reviewers handled separate correction scopes and cross-reviewed the contracts for guest/contact access, money/security, scheduler reliability and operations. Their follow-through findings were resolved or recorded as a pending decision, then reviewed by the coordinating agent. The [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) records the complete 135-finding mapping and final peer-review dispositions.

## Decisions still needed

The [O01–O12 register](../../CURRENT-REQUIREMENTS.md) preserves the remaining dependencies: PENDING-post invitation acceptance; average-commission date cohort; provisional month/rate defaults; old-phone warning delivery; verified-badge evidence; taxonomy/model approval records; analytics-store separation; retention/erasure timing; parent lesson-duration default; financial adjustments; founder testimonial content/consent; and the first-session timeout service-level agreement clock.

These are visibly pending, with the retained requirement and affected acceptance boundary stated. They are not silently answered by existing code or a proposed default. The two questions already sent about invitation acceptance and average commission remain pending; the settled agency-receipt decision is applied throughout.

## Verification

**Passed:** all 170 requirement IDs preserved, all 27 packages indexed, 366 introduced local links resolve, no new Markdown structure errors, and whitespace checks pass.

The final [Private supporting material excluded (publication note)](../../../docs/publication-notes.md) records requirement-ID preservation, introduced local link checks, changed-document structure checks and whitespace validation. The 135 original finding IDs and 135 correction IDs match with no missing or extra IDs. Backend and frontend Git working trees are clean. No application tests, migration runs, provider calls or deployment checks were executed for this documentation task.

The document check also removed a stray historical code fence; inline HTML examples are treated as literal code when checking details markup.

Original audit verdicts and line evidence are preserved as a **pre-correction snapshot** at Git HEAD REDACTED identifier. Snapshot Markdown links omit stale line positions; paired JSON retains the original quoted evidence and line numbers.
