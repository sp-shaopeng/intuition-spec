# Amazon Web Services deployment: retained approved decisions and verification

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Version0.1 · 2026-09-05 · documentation correction authorized by the user.

## Authority and recovery

The original budget-v3 plan is restored verbatim as [approved-plan-v3 md](evidence/approved-plan-v3.md). It is an archived source, with historical cost estimates and implementation claims, not a fresh certification. The original027 changelog and founder sign-off record approval of this v3 plan. This extract and spec md apply the later phase amendments and retained product requirements.

- Original source path: REDACTED local path.
- Recovered backup: 59c2737d7602147a@v3 in Claude session REDACTED record identifier file history.
- The session's file-history-snapshot metadata maps that backup to the original source path, version3, backup time 2026-06-24 T16:42:24.098 Z.
- SHA-256 of the restored original bytes: REDACTED identifier.
- The original source path currently contains an unrelated continuous integration plan; it is no longer a normative dependency.
- [Original sequencing/Verify table](evidence/approved-plan-v3.md) and [retained/dropped review conditions](evidence/approved-plan-v3.md) are now durable. Earlier premium-v2 topology is not restored as the current requirement.

## Retained architecture

| Concern | Current retained decision | Source |
|---|---|---|
| Budget and topology | ≤$200/month target; one Amazon Web Services account in ap-southeast-1; no NAT, application load balancer, interface endpoints or on-Amazon Web Services staging; retain local staging. Historical estimates must be refreshed before spending; this correction does not purchase resources. | archived v3 context/topology and spec027 approval |
| Compute/ingress | arm64 Fargate, two main tasks, each with cloudflared sidecar; public subnet for outbound access, task security group has no inbound; private managed relational database/Valkey reachable only from task security group. | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) |
| Data | Postgre SQL15 t4g micro single-AZ, gp3; PITR14 days selected in Phase1; deletion protection/prevent destroy; Valkey t4g micro one node, TLS/AUTH, noeviction. Single-AZ/single-node has no automatic failover. | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) |
| infrastructure as code home and secrets | intuition-spec/infra/terraform/; S3/Dynamo database state; scoped OIDC roles and SSM Secure text; one CMK with break-glass deletion control. Sensitive Terraform state is encrypted and access-restricted; do not claim provider-read secret values can never enter state. | [Private supporting material excluded (publication note)](../../docs/publication-notes.md), [Private supporting material excluded (publication note)](../../docs/publication-notes.md) |
| Edge/provider choice | Cloudflare Pages/Tunnel; retain the Phase3 paid-plan toggle decision as pending until recorded. Do not assert a particular current provider plan entitlement from the historical estimate. Resend is the selected transactional-email path. | [Private supporting material excluded (publication note)](../../docs/publication-notes.md), spec027 REQ-CUTOVER |
| Deployment | Backend and frontend deployment workflows remain manual-only before cutover; existing continuous integration workflows unchanged. Use the1-task tunnel canary,2-hour alarm-gated bake, then promote/rollback; prod environment reviewers and single-flight deployment apply. | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) |
| Canary correction | Check canary rollout state and running/desired task count at boot and end of bake; failed/unhealthy canary rolls back without changing main. Alarm-clean alone is insufficient. | [Private supporting material excluded (publication note)](../../docs/publication-notes.md) |
| Observability |90-day encrypted logs; Phase4 owns5xx/ERROR log-metric filters. Phase5 owns ops alarms/SNS/dashboard; SNS uses alias/aws/sns under its recorded review amendment. Confirm email subscription and alarm delivery at apply. | [Private supporting material excluded (publication note)](../../docs/publication-notes.md), [Private supporting material excluded (publication note)](../../docs/publication-notes.md) |
| Deferred premium services | Multi-account, Multi-AZ/replicas, on-Amazon Web Services staging, default Guard Duty/Config/Security Hub, application load balancer/Cloud Front/native blue-green remain deferred, not mandatory additions from the older plan. | archived v3 folded conditions |

## Current phase acceptance

Historical phase evidence records authored/reviewed Terraform and offline validation; it does not certify apply or cutover. Record current execution evidence separately.

| Phase | Required verification before advancing |
|---|---|
| A0 | All ten REQs in spec md, including in-flight drain/reclaim, trusted proxy, no pre-multi-factor authentication admin action, bootstrap-secret deletion retry, and delivery-preserving stream capacity. /version returns only the deployed revision identifier; anonymous /actuator/info is401. |
|0 Foundations | Review actual plan; state encryption/locking and IAM scopes verified; fresh params exist without exposing values; Cloud Trail emits; no inbound/NAT/application load balancer resources accidentally added. |
|1 Data | Migration runner applies to scratch database and exits0; database/Valkey TLS connects; noeviction/deletion guards attached; PITR restore to scratch measured against retained recovery requirements below. |
|2 Compute/tunnel | Two healthy main tasks; tunnel connects; deployed /version SHA matches expected image; direct origin access fails; untrusted forwarded headers cannot choose a limiter bucket. |
|3 Edge | SPA and application programming interface route correctly through the chosen provider plan; web application firewall/rate/health capabilities match the recorded plan selection; no claim of Free/Pro approval without its decision record. |
|4 continuous integration/continuous deployment | Successful short-bake rehearsal promotes; forced alarm and boot/end-of-bake unhealthy canary each roll back without changing main; concurrent deploy queues; provenance check passes. Retain the standard2-hour production bake; record any authorized emergency override. |
|5 Observability | Inject a representative actual-schema5xx/ERROR log and demonstrate the intended metric/filter/alert, confirm SNS subscription and received alert, query trace/correlation ID, and verify the billing guard. |
|6 Cutover | Super-admin multi-factor authentication + tutor/tutee/admin journey + real transactional email; SPF/DKIM/DMARC; secret custody and bootstrap cleanup; origin isolation; zero production one-time passcode logging; restore drill; documented rollback; retained non-functional requirement evidence below. |

## Retained product/non-functional requirement acceptance

These requirements are not waived by the budget-v3 approval. They refine the generic restore/smoke checks; they do not introduce premium topology.

- [business requirements document backup/recovery](../BRD.md): show measured RPO≤24h and RTO≤4h, daily backups,30-day rolling retention, Singapore backup location, and the quarterly restore procedure. The selected14-day PITR window does not by itself prove30-day backup retention. Record the additional retained backup arrangement or an explicit approved replacement before claiming this requirement met. The historical10–15-minute restore estimate is not a guaranteed RTO.
- Preserve business requirements document's public99.5% monthly and admin99.9% availability during08:00–22:00 Singapore time, seven days per week; identify the measurement, reporting window and owners. Single-AZ approval changes architecture, not a measured service-level agreement result.
- [product requirements document latency](../PRD.md): at the server, user-facing application programming interface P50<300ms/P99<1500ms; admin matching P50<500ms/P99<2000ms. Record workload, dataset and percentile sample; health200 alone does not prove these targets.
- Record message-trigger-to-delivery latency against [PRD14.2](../PRD.md) for enabled channels and agreed actual events; off/disabled transport is not a delivered message. Preserve retry/replay and terminal-failed-message queue visibility in the transactional-messaging contract.
- [business requirements document residency](../BRD.md) and [product requirements document provider requirements](../PRD.md): Amazon Web Services regional selection alone does not establish the processing locations of Cloudflare, Resend or every integration. Attach the required internal provider/residency or cross-border determination/DPA record and delivery-event tracking evidence before claiming compliance. This document makes no external provider or legal determination.

## Remaining recorded choices

The original architecture/Verify source is recovered. Unrecorded commercial/provider plan selection, measured acceptance evidence and any recovery-retention replacement remain explicit open records; the archived plan's historical estimates or HISTORICALLY MARKED IMPLEMENTED flags do not settle them. Drain/stream/bootstrap acceptance in the corrected spec still needs implementation evidence; this documentation pass supplies none.
