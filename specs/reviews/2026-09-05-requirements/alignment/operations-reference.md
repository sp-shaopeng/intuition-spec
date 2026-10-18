# Operations, references and governance alignment

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Operational and reference contract checks; evidence documents inspected only for role and scope. No commands from runbooks were executed.

## OPS-01 — Runbook says no failed-message queue, then documents both DLQs

**Internal contradiction and settled supersession**. The email outage procedure says there is no dead-letter queue, while the same runbook later documents email and WhatsApp DLQs. It also recommends unconditional stream trimming without a pending-delivery preservation rule. Replace the stale outage paragraph with the current failed-message queue/replay procedure and connect stream bounds to the unresolved delivery-loss contract in027. No Redis operation was performed.

Sources: [ops/runbook md](../../../ops/runbook.md), [ops/runbook md](../../../ops/runbook.md), [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md).

## OPS-02 — Reminder enablement runbook still directs the old sweep

**Explicitly superseded mechanism**. The operations instructions and template reference still describe reminder sent plus a15-minute sweep.007 explicitly replaces that mechanism with per-instance db-scheduler jobs. Fold the replacement into the enablement, diagnostics and rollback instructions; template content and existing consent conditions remain applicable. Enablement still needs the durable deduplication condition recorded in007.

Sources: [ops/runbook md](../../../ops/runbook.md), [ops/runbook md](../../../ops/runbook.md), [reference/whatsapp-templates md](../../../reference/whatsapp-templates.md), [007-generic-scheduler-db-scheduler/spec md](../../../007-generic-scheduler-db-scheduler/spec.md).

## OPS-03 — Monitoring is still framed around Railway and Vercel

**Explicitly superseded production topology**. The monitoring strategy and log-routing/retention instructions target the older provider stack.027 explicitly approves the lean Amazon Web Services topology. Map retained SLO, alerting and retention requirements to the actual Amazon Web Services components and recover the approved deployment plan. The OCI document explicitly describes a dev environment, so OCI dev versus Amazon Web Services production is not itself a conflict. This is an internal contract comparison, not a provider or legal assessment.

Sources: [ops/monitoring md](../../../ops/monitoring.md), [ops/monitoring md](../../../ops/monitoring.md), [ops/monitoring md](../../../ops/monitoring.md), [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md), [ops/dev-vm-deploy md](../../../ops/dev-vm-deploy.md).

## OPS-04 — Unauthenticated deployment SHA check points at the protected endpoint

**Explicitly superseded endpoint policy**. The runbook directs an anonymous /actuator/info request.027 explicitly keeps /actuator/info protected and adds a minimal public /version endpoint. Update the operational verification recipe and continuous integration documentation together. No live endpoint was probed.

Sources: [ops/runbook md](../../../ops/runbook.md), [027-aws-production-deployment/spec md](../../../027-aws-production-deployment/spec.md).

## REF-01 — application programming interface reference also conflates post approval with contact release

**Conflict with retained assignment gate**. The invitation section says contact is Match-gated until approval, whereas its own M18 endpoint section says only post-assignment. Use separate post moderation and match-assignment conditions; invitation initiation on a PENDING post does not authorize contact release.

Sources: [reference/backend-api md](../../../reference/backend-api.md), [reference/backend-api md](../../../reference/backend-api.md), [reference/backend-api md](../../../reference/backend-api.md), [021-tutee-request-flow/spec md](../../../021-tutee-request-flow/spec.md).

## REF-02 — Dev seeder operations still require encrypted account contacts

**Superseded historical data-model premise**. The seeder guide describes encrypted email/phone and the old small actor set. The explicit plaintext directive supersedes that account-field premise, while018/023 govern newer seed variety. Preserve dev-only guards and refer to current fixtures instead of treating old example counts as the current acceptance target.

Sources: [ops/dev-sample-data md](../../../ops/dev-sample-data.md), [ops/dev-sample-data md](../../../ops/dev-sample-data.md), [architecture/plaintext-email-phone md](../../../architecture/plaintext-email-phone.md), [018-flag-hygiene-seeder-and-fe-hygiene/requirements md](../../../018-flag-hygiene-seeder-and-fe-hygiene/requirements.md).

## File coverage

| Document | Assessment | Review depth |
|---|---|---|
| [ops/runbook md](../../../ops/runbook.md) | conflict | Targeted operational procedures: provenance, email outage/replay/failed-message queue, scheduler enablement; shell commands were not executed. |
| [ops/monitoring md](../../../ops/monitoring.md) | partial | Targeted normative stack, logging/retention, SLO and alert sections; Prom QL was not executed. |
| [ops/local-stack md](../../../ops/local-stack.md) | partial | Targeted environment, migration, smoke and production-topology descriptions; local Compose is not proof of the Amazon Web Services production topology. |
| [ops/dev-vm-deploy md](../../../ops/dev-vm-deploy.md) | historical/dev-specific | Targeted environment scope, migration/seed, verification and security clauses; the documented OCI dev environment is not a conflicting choice for Amazon Web Services production. |
| [ops/dev-sample-data md](../../../ops/dev-sample-data.md) | partial | Normative data model, enablement, actor set and safety sections. |
| [reference/backend-api md](../../../reference/backend-api.md) | conflict | Targeted authentication, public/detail boundaries, invitations, M18 contact, messaging and contact-change sections; code-derived reference is not product approval. |
| [reference/frontend-routes md](../../../reference/frontend-routes.md) | partial | Routes, auth boundaries, nav and completion-status tables; main M17 gates align, historical completion labels are not independently verified. |
| [reference/postman md](../../../reference/postman.md) | partial | Targeted contract-change notes, public caps, contact/reminder additions and route examples; collection completeness was not revalidated. |
| [reference/whatsapp-templates md](../../../reference/whatsapp-templates.md) | partial | Template content/order and normative trigger/format sections; scheduler trigger is superseded, no provider approval verification. |
| [reference/branch-coverage-matrix md](../../../reference/branch-coverage-matrix.md) | historical-evidence | Header/scope and coverage claims inspected as historical evidence; no current coverage certification. |
| [reference/five-factor-audit md](../../../reference/five-factor-audit.md) | historical-evidence | Header/method inspected as historical evidence; no rescoring of old test classes. |
| [governance/feature-ownership md](../../../governance/feature-ownership.md) | aligned | Ownership matrix and pre-state/action/post-state plus negatives/role-based access control/transition depth requirements reviewed; this review does not deliver a module. |
| [governance/boss-audit-2026-06-14-coverage md](../../../governance/boss-audit-2026-06-14-coverage.md) | historical-evidence | Scope, verdict and reachability-versus-flow distinction reviewed; old coverage verdict is not a present runtime result. |
