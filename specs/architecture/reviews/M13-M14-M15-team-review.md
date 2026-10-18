# Team Review Record — M13 / M14 / M15

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-05-29 · **Mode:** Full (17 specialists + Tech Lead + Boss), real
multi-agent review · **Outcome:** SEND BACK with resequencing + hard gates.

> Supersedes the earlier M13-team-review md and M14-landing-showcase-plan-review md
> notes, which were written as an informal summary, not a real agent debate. This
> record reflects the actual review.

## Consensus
- **Resequence: M15 first → then M14 → M13 last.** Nobody defended the original
  order. M15 is buildable today; M14 and M13 are not.
- **Kill the M14 proxy applicant count.** Architect, Product, BA, Data, Writer,
  and **both Platform Users** independently called it dishonest/unmeasurable.
  Show a count only from real M15 applications.
- **M13 is not deployable now:** no database backups, the ops pii restricted database role
  doesn't exist in the single-database compose flow, and the exfil rate-alert is
  spec-only (Prometheus can't scrape a database audit row).
- **last active at must not write on every request** — guarded
  the now operation, Redis-gated, or autovacuum
  melts the hottest table.

## Verified facts (Boss read the code)
- ✅ **AES-SIV is deterministic** (Encryption Service encrypt Deterministically) →
  exact-match search already works over ciphertext; a plaintext store is only
  justified if **partial/substring** search is a proven operator need.
- ❌ **OPERATOR role does not exist** (User Role = TUTEE/TUTOR/ADMIN).
- ❌ No job applications, no ops pii, no last active at, no OPS personal information VIEW —
  all three modules are greenfield specs.
- ✅ No backups / continuous integration / staging / email sender / payments (per SCOPE md).

## Tech Lead final decision
- **M15 — BUILD FIRST.** Thin apply slice; uniqueness enforced among applications that have not been withdrawn;
  named enum; idempotent duplicate-apply; withdraw→re-apply; no count surfaced yet.
- **M13 — DEFER, then thin slice:** audited operator search + single-person contact
  card via **decrypt-on-the-fly** (no second store unless partial search proven);
  add OPERATOR; shared Protected Route.
- **M14 — CUT for now;** revisit when M15 yields a real count + a funnel exists.

## Boss (Rex) verdict: ⚠️ SEND BACK
Conditions:
1. **M15 before M14;** delete the proxy from the M14 spec (done).
2. **No ops pii backfill until verified database backup + restore exists.**
3. **M13 audit fail-closed (503 on audit-write failure)** + a **blocking** bulk-read
   cap, both asserted by integration test.
4. **OPERATOR role + ops ** interface operation gating + named-multi-factor authentication operator accounts** land
   together; no plaintext directory reachable without them.
5. Correct the record where the earlier summary misstated the decisions (done).

**Founder ruling (Boss):** building the *stored, viewable, secure* table honors the
founder's ask (just isolated); but the applicant count must be **real, not a proxy**
— the proxy is killed. Build M15 first, get a real count, then M14; build M13's
store only after backups exist and only if partial search is genuinely required.

## Per-discipline must-fixes (condensed)
- **SWE:** structural single-writer (outbox/same-tx), not nightly parity; last active at off hot path.
- **Architect:** dual-source-of-truth is the core flaw; prove partial search or drop the store; the random operation scaling cliff.
- **DBA:** soft-delete won't purge plaintext row; guarded last active at write; M15 partial unique + count index + named enum; OPERATOR enum add is run In Transaction is no.
- **Security:** ops pii isolation is partial theater (same instance/backups/credential/keyset); separate database role or decrypt-on-the-fly; blocking bulk-read cap; out-of-band WORM audit; named multi-factor authentication operators; Personal Data Protection Act erasure cascade.
- **Frontend:** lazy-load /ops; one shared Protected Route (add OPERATOR to User Role); locked card/skeleton dims + deterministic order.
- **frontend-facing service:** single batched Ops Person Detail Response; post Id-keyed batch count; split showcase endpoints; Page Response everywhere; additive-nullable count.
- **user experience:** disambiguating search rows; contact-first tabbed person view; freshness-first order; low-volume landing layout; hide count when zero.
- **Dev Ops:** backups before backfill; provision the ops pii role+GRANT or isolation is false; stand up monitoring.
- **quality assurance:** backfill parity 100% rows; per-read single audit row; authz negatives; 0-vs-hidden count; deterministic order tests; duplicate-apply idempotent; withdraw→re-apply.
- **Data:** showcase funnel events (impression→click→conversion); typed OPS personal information VIEW field Set; OPS ACTION TAKEN; M15 status-transition fact.
- **Writer:** stale index headers (still "M1–M12"); M13 endpoint data transfer objects + column types missing; operator runbook missing; pin the count query.
- **SRE:** backfill needs backup+rollback+resumable batches; audit fail-closed; cache stampede lock + Redis-down degraded mode.
- **Observability:** emit intuition ops pii view total + deploy scrape/alert before M13; showcase cache-hit/latency/write-rate metrics.
- **Wei (tutor):** random feed unhelpful (wants relevance); high counts discourage; abandons on silence after applying.
- **Priya (parent):** public card ok if never location/initials; real count reassures, proxy is "a lie"; "it's logged" insufficient — wants least-privilege + active log review + a plain-language privacy promise; quits on an unapproved cold call.
