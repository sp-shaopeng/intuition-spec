# Feature ownership matrix

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Founder directive (2026-06-14):** every agent OWNS a slice of the product — its **code AND its test
cases** (backend unit/IT, frontend unit, and the e2e/ flow specs). The owner is accountable for that module's
correctness, coverage depth (pre→action→post + negatives + role-based access control + transitions), and lessons. Every
module below has exactly one primary owner; co-owners advise. No module is unowned.

## Module ownership

| Module | Primary owner | Co-owner using s | Scope (code + tests) |
|---|---|---|---|
| **M1 Auth** — register, login, multi-factor authentication, forgot/reset, change-pw, delete-account, first-login gate | **security-expert** | qa-engineer | authn/authz, lockout, one-time passcode, session, password-change gate |
| **M2 Posting (tutee+tutor)** — create, read, update and delete, submit, close, availability | **bff-specialist** | dba | post contracts, status machine, soft-delete |
| **M2 Public Browse** — public tutors/requests lists | **frontend-expert** | bff-specialist | browse surface, pagination, render |
| **M3 Approval** — pending queue, approve/reject, revision, active | **business-analyst** | software-engineer | moderation workflow + acceptance criteria |
| **M4 Matching** — create/activate/cancel/reopen | **software-engineer** | dba | match state machine, atomicity |
| **M5 Commission** — pay, completion | **software-engineer** | dba | money path, optimistic lock, idempotency |
| **M6 Interview** — request/schedule/complete/no-show + INT-039 negotiation | **qa-engineer** | software-engineer | the most stateful machine; turn/round, slot grid, cap |
| **M7 Profile** — tutee/tutor profile, canonical identity | **uiux-designer** | frontend-designer | identity, forms, completeness gate |
| **M8 Match Dashboard** — tutee/tutor match views | **frontend-expert** | data-engineer | dashboards, humanisation (INT-137a) |
| **M9 Notifications** — list, unread, mark, fan-out | **operator** | data-engineer | in-app fan-out, POST SUBMITTED, outbox |
| **M10 Favourites** — add/probe/list/remove | **product-designer** | bff-specialist | feature scope |
| **M11 Reviews** — submit, my-reviews, public | **product-designer** | qa-engineer | reviews on COMPLETED matches |
| **M12 Admin Analytics** — overview, date range | **data-engineer** | business-analyst | metrics, measurement, INT-045 |
| **M14 Landing Showcase** — active-tutors/recent-requests strips, featured-consent | **frontend-designer** | uiux-designer | personal information-free showcase, consent |
| **M15 Applications** — apply/list/withdraw/applicants | **bff-specialist** | business-analyst | apply funnel |
| **M16 Admin Management** — create/list/delete admin, super-admin, first-login | **security-expert** | operator | admin authz, super-admin protection |
| **Admin Moderation (INT-065)** — users, block/unblock, delete posting | **security-expert** | operator | abuse handling, soft-delete |
| **M17 Marketplace Detail** — tutor/assignment detail, login gate, 404 | **frontend-expert** | bff-specialist | login-gated detail, 401/404 |
| **M18 Assignment Lifecycle** — applicants, select, assign, contact, invitations | **software-engineer** | bff-specialist | assign atomicity, selection, reveal |
| **INT-082 WhatsApp / phone** — verify, consent, webhooks (feature-flagged) | **devops-engineer** | security-expert | external integration, flag, webhook signing |

## Cross-cutting ownership (no single module, owned end-to-end)

| Area | Owner |
|---|---|
| Schema, migrations, data integrity (Liquibase) | **dba** |
| continuous integration pipelines, deploy, infra, the e2e continuous integration lanes | **devops-engineer** |
| System design, ADRs, failure modes, scalability | **solution-architect** |
| Test strategy, the e2e/ lab + the coverage metric, quality gates | **qa-engineer** |
| Measurement / event instrumentation across modules | **data-engineer** |
| Mo S Co W / scope / MVP discipline | **product-designer** |
| Synthesis, delivery-vs-quality trade-offs | **tech-lead** |
| Code-review pass on every change | **code-reviewer** |
| Adversarial bug-hunting on the live system (INT-NNN) | **bug-founder** |
| Governance: audit + SHIP/SEND-BACK verdicts | **boss** |

## What "owning the tests" means (the depth bar)
Every owner's module tests MUST follow **pre-state → action → post-state**, not just an action's status:
- Capture the BEFORE state (e g. login with pw1 succeeds), perform the action, assert the AFTER state
  (logout, login with pw2 succeeds; pw1 now fails).
- Cover the **negative + role-based access control** matrix: wrong-role → 403, non-owner → 403/404, invalid-state → 409/422,
  missing/blocked → 401/404 — not only the happy path.
- Cover **state transitions** (the full enum machine), **idempotency replay**, **quota exhaustion**, and
  **soft-delete preservation** where the module has them.
Owners keep their module at the depth bar; the Boss audits owners against it.
