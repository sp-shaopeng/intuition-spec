# Tasks — Tutor apply feedback + profile user experience (012)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Boss dispatches each owning specialist. Mixed backend+frontend lane; the frontend apply control + Assignment Detail are this spec's
> files (no overlap with 011's api implementation reference/auth Store). Check a box only when evidence is under the private evidence directory for the requirement.

## Governance gate (FIRST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 1 | Scoped /team-review (bff-specialist, software-engineer, frontend-expert, uiux-designer, **security-expert, dba** for REQ-004, tech-lead) + Boss audit → sign-off → status: APPROVED. REQ-001 source-of-truth RESOLVED (payload signal). **REQ-004 founder override RESOLVED (build self-edit + re-verify) — email-link fallback and request-table design are resolved; review the revoke contract and separately track the unresolved old-phone warning transport.** | all | Boss | boxes checked; status APPROVED; email-link/revoke contract recorded; old-phone transport explicitly pending | [ ] |

## backend lane (bff-specialist · software-engineer)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 2 | REQ-001: extract the apply-completeness check (Job Application Service assert Profile Complete To Apply, the 7 fields: Handle, commitment, rate Min, rate Max, planning Areas, qualifications [no length floor], subject Levels) into ONE shared predicate; expose server-computed profile Complete + missing Fields on the GET tutor-profile / profile-me response from that SAME predicate; keep the 422 TUTOR PROFILE INCOMPLETE gate (driven by the shared predicate). **Aligned-constant fallback REJECTED unless a concrete blocker is recorded.** | REQ-001 | bff-specialist · software-engineer | tutor-profile data transfer object/controller, Job Application Service implementation reference | one predicate, two callers; frontend+backend cannot diverge | [ ] |
| 3 | REQ-001: confirm whether the bare 403 Access denied is a security-layer role/auth 403 vs the service 422; ensure both are surfaced (no silent path) | REQ-001 | software-engineer | Security Config, Job Application Service implementation reference | 403 vs 422 path documented + both handled | [ ] |
| 4 | backend tests for the completeness signal (field matrix) | REQ-001 | software-engineer | Job Application Service Test + profile data transfer object tests | the documented operational check ≥97.5/97.5 | [ ] |

## frontend lane (frontend-expert · uiux-designer)

| # | Task | Traces | Owner | Files | Verify | Status |
|---|------|--------|-------|-------|--------|--------|
| 5 | REQ-001: **delete the frontend is Profile Complete predicate** (Tutor Apply Control implementation reference) and consume the server profile Complete/missing Fields instead; banner lists missing fields; add a clear 403/auth branch to on Error (no bare "Access denied"). Add the BLOCKING frontend↔backend parity test (gate renders iff profile Complete equals no) | REQ-001 | frontend-expert | Tutor Apply Control implementation reference, Profile Completeness Bar/Apply Gate Banner | frontend predicate deleted; parity test green; incomplete→banner; 422→toast+banner; 403→message; none silent | [ ] |
| 6 | REQ-002: sticky rail in Assignment Detail mirrors applied/gated/inactive state (hide/relabel) | REQ-002 | frontend-expert | Assignment Detail implementation reference (207-225), Tutor Apply Control implementation reference | sticky rail matches control state | [ ] |
| 7 | REQ-003: test the tutor subject selector derives from SUBJECT OPTIONS; live screenshot; note GH #209.1 to close | REQ-003 | frontend-expert | Create Listing implementation reference, Subject Level Matrix Editor implementation reference, tests | options  equals  shared taxonomy | [ ] |
| 8a | REQ-004 (override) — schema: Liquibase migration for the pending-contact representation (use the decided contact change request table: old/new contact snapshots following the plaintext-contact amendment, channel, separate confirm/revoke hashes, status including REVOKED, expiry and audit; current email/phone unchanged until commit) | REQ-004 | dba | db/changelog/** | migration applies via Testcontainers; no raw database structure definitions | [ ] |
| 8b | REQ-004 (override) — backend flow: edit endpoint stores PENDING value + issues verification (EMAIL change: link to NEW email; PHONE change: link to EXISTING verified account email; explicit landing-page POST); verify endpoint enforces uniqueness at commit (reuse spec 011 REQ-004 dup handling) → atomic swap + audit row + clear pending; unverified/expired requests retain old value and terminal audit state; OLD value used until commit | REQ-004 | software-engineer · bff-specialist | user/profile service, new contact-change endpoints + data transfer objects, api Routes implementation reference | pending held; old-until-verify; verify→swap+audit; expiry discards | [ ] |
| 8c | REQ-004 (override) — security: require step-up/re-auth to START a change; single-use short-expiry token; record OLD-email and OLD-phone warning intents/outcomes; implement separate purpose-bound revoke token and POST, REVOKED state and atomic confirm-vs-revoke; old-phone transport remains pending, not waived | REQ-004 | security-expert | contact-change endpoints, tests | step-up and revoke proven; warning intent/attempt/delivery distinguished; old-phone channel acceptance pending | [ ] |
| 8d | REQ-004 (override) — frontend: editable email/phone on tutor Profile + "current value stays active until you confirm" pending banner + email-link phone copy + explicit Confirm/Revoke pages and profile revoke action | REQ-004 | uiux-designer · frontend-expert | Profile implementation reference (547-552), pending-state user interface | edit→pending→verify→commit; no silent discard | [ ] |
| 9 | frontend tests for REQ-001/002/003/004 | REQ-001..004 | frontend-expert | *.test implementation reference | lint and build and test:ci green ≥97.5/88 | [ ] |

## Evidence + Do D

| # | Task | Traces | Owner | Private evidence requested | Expected result | Status |
|---|---|---|---|---|---|---|
| 10 | Live Playwright: incomplete-profile apply (banner + missing fields), forced 422 toast, sticky-rail-after-applied, expanded subject list, **REQ-004 email edit→pending→verify→commit + phone edit→pending→EMAIL-link-confirm→commit with WhatsApp OFF; revoke-before-confirm and replay/race outcomes** | REQ-001,002,003,004 | qa-engineer · frontend-expert | captures under evidence/ | gate never silent; rail correct; contact change re-verified | [ ] |
| 11 | Do D: backend + frontend gates green; local deploy; live captures; e2e green; virtual machine re-confirm; commit+push current branch; close GH #209.1 (REQ-003) | all | software-engineer · frontend-expert | gates + push | green + pushed | [ ] |

## Governance gate (LAST)

| # | Task | Traces | Owner | Verify | Status |
|---|------|--------|-------|--------|--------|
| 12 | /team-review (full) → Boss verdict; fix SEND-BACK in place; SHIP → status: HISTORICALLY MARKED IMPLEMENTED + requirements traceability matrix | all | Boss | Boss SHIP; requirements traceability matrix updated; evidence opened | [ ] |

## Outstanding delivery decision — 2026-09-05

- [ ] Record the approved old-phone warning channel/fallback and failure behavior in [current open decisions](../CURRENT-REQUIREMENTS.md), then prove both-channel delivery acceptance. Email-link confirmation is already decided; do not revert to WhatsApp/pending-only or report queued warnings as delivered.
- [ ] Prove verify/revoke purpose separation, owner authorization, no mutation on GET, idempotent revoke, terminal conflict and concurrent confirm/revoke serialization. These acceptance additions have not been executed in this documentation correction.
- [ ] Verify the confirm link supplies request Id and token to the authenticated POST; initiate/pending/profile responses expose the request id without tokens; profile revoke sends the contact-change request identifier, warning-link revoke sends the contact-change request identifier and its separate revocation token, and successful/idempotent revoke returns 204. Assert EMAIL and PHONE pending copy names the distinct confirmation destination.
