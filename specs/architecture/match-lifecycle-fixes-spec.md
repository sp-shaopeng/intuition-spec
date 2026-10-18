# Match Lifecycle Fixes — INT-072 (Cancel Match) & INT-073 (Activate / Commission Basis)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

The cancellation cascade preserves a historical paid receipt and cancels an unpaid pending commission; it does not itself decide refund/clawback entitlement. Earlier D-1/D-2 commission-basis alternatives are superseded by [INT-157 v2](int-157-commission-first-month-locked-spec.md) agreed-plan activation locking. Only its explicit commercial defaults and the separate refund/end-of-tuition policies remain in [open decisions](../CURRENT-REQUIREMENTS.md). Do not reopen settled storage, FLEXIBLE or lock timing.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** SPEC — DRAFT (doc only; no code, no migration in this deliverable)
**Authors:** software-engineer (Jordan), business-analyst (Dana), uiux-designer (Jamie)
**Date:** 2026-06-05
**Owners on build:** software-engineer + frontend-expert + qa (INT-072); business-analyst + product-designer + software-engineer + bff-specialist + uiux (INT-073)
**Source issues:** intuition-spec/specs/issues/README md (INT-072), 562 (INT-073)
**Flow:** spec → **team-review → Boss → (founder decision) → build**

This spec covers two founder-walkthrough defects on the admin match lifecycle that share the
same surfaces (/admin/matches Matching Dashboard, /admin/commission Commission Ledger) and the
same service (Admin Match Service):

- **INT-072** — "Cancel Match" reported non-functional. **Fix-now** (reproduce → classify → fix +
  gate frontend + regression test).
- **INT-073** — Activate-match hardcodes a "Monthly fee" and a flat 50% commission. **Founder
  decision required** on fee basis + commission rule; ships a low-risk first step that needs no
  decision and no model change.

> **Hard rule (diagnose-before-patch):** INT-072 MUST be reproduced on the LIVE dev stack and
> classified by when it fails BEFORE any code is written. The "backend truth" below is a
> *hypothesis to confirm against the running system*, not a license to patch blind. A tracker-filed
> root cause is a hypothesis, not fact (SHARED-LESSONS / INT-016).

---

## Part 1 — INT-072: "Cancel Match" doesn't work in practice

### 1.1 Problem statement

Founder report (2026-06-05 walkthrough): a match **cannot be cancelled** — "technically we can't
cancel a match" — despite a "Cancel Match" / "Cancel" button being present on multiple admin
surfaces. Severity **S2** (a core admin lifecycle action is reported broken; a stuck PENDING/ACTIVE
match cannot be reversed, so the job it locked cannot be reopened → liquidity + ops blocker).

The report does not say when it fails (no click → nothing happens? error toast? wrong button on
the wrong row?). That classification is the first deliverable.

### 1.2 Current state (grounded in source)

**Backend — Admin Match Service cancel Match (Admin Match Service implementation reference):**

- Allows cancel only when the get status operation ∈ CANCELLABLE STATUSES = PENDING, ACTIVE
  (73). Terminal statuses (COMPLETED, CANCELLED) → throw at 534-537.
- On a terminal status it throws the unprocessable entity operation.
  **unprocessable Entity maps to web request protocol 422** (App Exception implementation reference) — **NOT 409.**
  > ⚠️ **Doc/code discrepancy to fix:** the INT-072 issue note (README md), the brief, and the
  > controller Javadoc (Admin Match Controller implementation reference says "422 if COMPLETED or already
  > CANCELLED" — correct) disagree with anything that calls this a "409". The spec, the frontend handling,
  > and the regression test MUST treat the non-cancellable rejection as **422 MATCH NOT CANCELLABLE**.
  > Confirm the real status code in the reproduction step (1.4) before writing the test assertion.
- On success: match status → CANCELLED (540); if the job is MATCHED, it is reopened to
  ACTIVE and its applications reopened (567-575); a **PENDING** commission is flipped to
  CANCELLED (578-582) while a **PAID** commission is **intentionally left untouched** (the
  agency already received payment — 519-522 Javadoc). After-commit, failure-isolated notification
  to the former winner tutor (586-596). The whole path was hardened against the INT-016
  Lazy Initialization Exception 500 (all lazy values snapshotted pre-detach, 543-563).
- Controller: submit operation for admin matches id cancel → 200 with Admin Match Item Response
  (Admin Match Controller implementation reference); ADMIN-only (class @Pre Authorize + Security Config glob).

**Frontend — two surfaces expose cancel, and they behave differently:**

| Surface | File | Gating | Confirm dialog? | Error feedback |
|---|---|---|---|---|
| Matching Dashboard | Matching implementation reference | can Cancel status equals PENDING \|\| ACTIVE (385) — button rendered only when cancellable (451-460) | **Yes** — cancel Confirm Id opens a modal component whose "Cancel match" button fires cancel Mutation mutate (334-367, 356-360) | Generic: the error operation (135-136) — does **not** read the error code |
| Commission Ledger | Commission implementation reference | Same PENDING \|\| ACTIVE gate (367-368) | **No** — clicking "Cancel Match" fires the mutate operation **directly, no confirm** (370) | Generic: the error operation (92) |

- Route constant exists and is correct: the match cancel operation →
  admin matches id cancel interface operation (api Routes implementation reference). Both pages POST it with an empty value object
  (Matching implementation reference, Commission implementation reference).
- **No centralized frontend error-code → message map exists** (grep of lib/ found none): both pages emit
  a single catch-all toast regardless of whether the failure was 422 MATCH NOT CANCELLABLE, 404,
  403, 500, or a network error. So when cancel *does* legitimately fail (e g. someone else already
  cancelled it → terminal status → 422), the admin sees "Please try again" — which never succeeds
  and reads as "can't cancel." (Jamie: this is a Nielsen heuristic-9 violation — the error neither
  diagnoses nor tells the user what to do.)
- Interview surfaces (Interview Queue implementation reference, Interview Detail implementation reference) have **no** Cancel-Match action
  (grep found none). The founder's "/admin/interviews" mention does not correspond to a cancel
  button there; the two real surfaces are **Matching** and **Commission**. Confirm during repro
  whether the founder was on one of these two.

### 1.3 Hypotheses (the three buckets — classify, do not assume)

Per the issue note (README md):

- **(a) frontend action/dialog not wired → nothing happens.** Largely ruled out by reading source:
  Matching's confirm modal Confirm button is wired to the mutation (356-360); Commission fires
  directly (370). But a regression (e g. cancel Confirm Id set but Confirm handler guarded out,
  or a Modal that renders children-always so the dialog never visibly opens) must be ruled out
  *live*, not by reading.
- **(b) Button shown on a non-cancellable status → 422 with unclear feedback.** Both pages gate to
  PENDING/ACTIVE in the rendered row, so a *stale* list is the real risk: the row was ACTIVE when
  rendered, someone (or the same admin twice) cancelled it, the list wasn't refetched, the admin
  clicks again → 422 MATCH NOT CANCELLABLE → generic "try again" toast → "can't cancel." **This is
  the most likely real defect** given the gating already exists and the error handling is generic.
- **(c) A regression on the cancel endpoint** (500/403). Confirm by reading the *live* web request protocol status +
  body and the backend log; INT-016 specifically broke this path with a Lazy Init 500 once.

### 1.4 Required reproduction (BEFORE writing any fix) — acceptance gate for starting the build

Run against the live dev stack (Abstract Api IT-style; SHARED-LESSONS "regression ITs hit the live
stack"). Capture the **real web request protocol status + response body + any toast + backend log** for each:

1. **Happy path:** activate a PENDING match → it becomes ACTIVE; click **Cancel** on that ACTIVE row
   on /admin/matches, confirm the modal. Expect 200 and status: CANCELLED, job reopened to
   ACTIVE. Record what actually happens.
2. **Same on /admin/commission** (no confirm dialog there) — record status + toast.
3. **Stale-row path (bucket b):** cancel a match, then WITHOUT refetch click Cancel again (or open
   two tabs). Expect 422 MATCH NOT CANCELLABLE. Confirm the toast text the admin sees.
4. **PAID-commission path:** activate → mark commission PAID → click Cancel. Per 519-522 the match
   SHOULD still cancel (200) and the PAID commission stays PAID. Confirm — the founder's "we can't
   cancel" may be the (correct, intentional) refusal to cancel a COMPLETED match, which IS a 422 by
   design. If so, the fix is **user experience/messaging**, not the endpoint.
5. Read the documented operational check for any stack trace on the failing click; classify
   code-vs-config from when it throws.

**Deliverable of this step:** a one-paragraph classification (which bucket, with the captured
status/body) appended to INT-072 in the issues README, before the fix pull request.

### 1.5 Design / fix (applies regardless of bucket — all are low-risk and additive)

**frontend-1 — Add a typed error-code → message mapper and use it on the cancel mutations.**
Create a small shared helper (e g. api Error implementation reference) that extracts
err response data error code and maps known match-lifecycle codes to human, actionable copy:

| Code | web request protocol | Admin-facing message (Nielsen-9: say what + what to do next) |
|---|---|---|
| MATCH NOT CANCELLABLE | 422 | "This match can no longer be cancelled — it's already completed or cancelled. Refreshing the list." (then refetch) |
| MATCH NOT FOUND | 404 | "This match no longer exists. Refreshing the list." (then refetch) |
| (network / 500 / unknown) | — | "Couldn't cancel the match. Please try again." (existing fallback) |

On MATCH NOT CANCELLABLE / MATCH NOT FOUND, the on Error handler also calls
refresh the cached match lists after a change so the stale row disappears and the
admin sees current truth (addresses bucket b directly). Magic-string rule: the codes live as typed
constants (mirror MATCH STATUS/INVOICE STATUS enums already in types implementation reference), not inline literals.

**frontend-2 — Gate the button to live status + add a confirm to Commission.**
- Keep the existing can Cancel PENDING or ACTIVE gate on both pages (it is already correct —
  Matching implementation reference, Commission implementation reference); add the same gate as a guard inside the mutation
  trigger so a stale render cannot fire a doomed request without feedback.
- Bring Commission implementation reference to parity with Matching implementation reference: route its "Cancel Match" through the **same
  confirm modal** pattern (cancel is destructive + reopens the job — Fitts/Nielsen error-prevention:
  destructive actions get a confirm step). Reuse the existing Matching cancel modal copy.
- Where a row is in a terminal/non-cancellable state, do **not** render a dead "Cancel" button (both
  pages already hide it — keep it hidden, not disabled-without-reason).

**backend — no change expected.** The service contract (PENDING/ACTIVE→cancel, terminal→422, PENDING
commission→CANCELLED, PAID untouched) is correct and intentional. Only touch backend if reproduction
proves bucket (c) (a genuine 500/regression) — in which case follow the INT-016 pre-detach-snapshot
pattern and reproduce the exact stack first.

**Doc fix:** correct the INT-072 issue note (README md) and any prose that says "409
MATCH NOT CANCELLABLE" to **422** (the code and controller Javadoc are already 422).

### 1.6 Acceptance criteria (Given/When/Then — behavioural, survive a redesign)

- **AC-1 (happy path, both surfaces):** *Given* a PENDING or ACTIVE match, *When* an admin confirms
  Cancel on /admin/matches **or** /admin/commission, *Then* the match becomes CANCELLED, the
  job reopens to ACTIVE, any PENDING commission becomes CANCELLED, a PAID commission stays
  PAID, and a success toast ("Match cancelled — job reopened.") shows.
- **AC-2 (terminal path):** *Given* a match that is already COMPLETED or CANCELLED, *When* a
  cancel request reaches the server (stale row), *Then* the application programming interface returns **422 MATCH NOT CANCELLABLE**
  and the admin sees a specific message ("already completed or cancelled") and the list refetches to
  remove the stale row — never the generic "please try again" loop.
- **AC-3 (confirm-on-destructive):** *Given* the Commission Ledger, *When* an admin clicks "Cancel
  Match", *Then* a confirm dialog appears before any request is sent (parity with Matching).
- **AC-4 (gating):** *Given* any match list, *Then* the Cancel control is rendered only for
  PENDING/ACTIVE rows on both surfaces.

> Caps at 4 ACs because INT-072 is a defect fix, not a new story; if reproduction reveals bucket (c)
> add ONE more AC for the specific regression.

### 1.7 Test notes (qa)

- **backend regression (live-stack IT, Admin Match Controller Test / a live IT):**
  (1) **success** — POST cancel on an ACTIVE match → 200, body status is CANCELLED, PENDING commission
  → CANCELLED; (2) **422 path** — POST cancel on a COMPLETED (or already CANCELLED) match → 422
  with error code equals MATCH NOT CANCELLABLE; (3) **PAID-untouched** — cancel an ACTIVE match whose
  commission is PAID → 200, commission still PAID. Assert the **status code is 422, not 409**
  (guards the doc discrepancy). Live-stack ITs need a reseed/rebuild before the documented operational check reflects the
  fix (SHARED-LESSONS).
- **frontend (Vitest):** assert the error mapper renders the specific MATCH NOT CANCELLABLE message and
  triggers a refetch (mock 422); assert the Commission confirm modal opens before the mutation fires
  (modal close-path coverage per INT-046 — assert the open attribute, not text removal); assert
  Cancel is absent on a COMPLETED/CANCELLED row. Coverage gates: frontend ≥97.5% line / ≥88% branch; backend
  ≥97.5% line+branch.

---

## Part 2 — INT-073: Activate-match assumes "Monthly fee"; fee basis + commission must be explicit

### 2.1 Problem statement

The Activate-match modal collects a single **"Monthly fee (SGD)"** input
(Matching implementation reference activate modal) and the backend derives commission as a
flat **50% of that monthly fee** (Admin Match Service activate Match,
DEFAULT COMMISSION RATE 0.5 at 76). Founder: **"not monthly, also the commission."**

Two distinct concerns:

1. **Fee basis isn't always monthly.** SG tuition is frequently **hourly × hours**, **per-session**,
   or **package / fixed-term**, not a flat monthly retainer. Labelling everything "Monthly fee"
   mislabels the agreement and can mis-size the commission.
2. **Commission is implicit.** The admin types a fee; the 50% is never shown before activation. The
   Commission Ledger headers assert "50% of the monthly fee" — an assumption baked into the user interface,
   the business requirements document, and the data model.

Severity **S2** — it touches the revenue/commission model (M5). Dana: **we cannot ship a model
change here without a stated business rule** — what we charge on, and whether it is always 50% of
the first month, is a *business decision*, not an engineering one.

### 2.2 Current state (grounded in source)

- **Commission entity (Commission implementation reference):** columns monthly fee sgd int (32-33),
  commission rate double (35-36), commission amount sgd int (38-39),
  invoice status enum (41-44). The fee column is **named and typed as "monthly"** — there is no
  fee basis / hours / sessions column. One commission per match (One To One unique on match id,
  28-29).
- **activate Match (467-510):** signature the activate match operation;
  the monthly fee multiplied by 50%, rounded to a whole Singapore dollar (481); builds the Commission with
  commission Rate 0.5 (487). Requires match PENDING else 422 MATCH NOT PENDING
  (475-479).
- **data transfer object Activate Match Request:** record Activate Match Request(@Not no value @Positive @Max using 50000
  Integer monthly Fee Sgd)Activate Match Request implementation reference).
- **frontend modals** collect a single monthly Fee Input, validate 0, POST the first-month fee in Singapore dollars
  (Matching implementation reference). **No commission shown before activation.**
- **Match Status** is PENDING/ACTIVE/COMPLETED/CANCELLED (Match Status implementation reference) — unaffected by
  this change; listed here only because the issue references it.

### 2.3 Low-risk FIRST STEP (no founder decision, no model change) — ship now

**Surface the computed commission live in the activate modal.** As the admin types the fee, show:

> **Commission (50%): S$200**

computed client-side as 50% of the entered fee, rounded to a whole Singapore dollar (mirrors activate Match), updating on every
keystroke (Doherty: <400ms feedback). This makes the billed amount unambiguous **before**
activation, costs no schema/data transfer object/endpoint change, and is reversible. Add it to BOTH activate modals
(Matching implementation reference and Commission implementation reference) for parity.

- Keep the field label "Monthly fee (SGD)" until the founder decides 2.4 — do **not** silently
  relabel, since the basis is still officially monthly.
- The 50% rate is a typed constant on the frontend (no magic number), kept in sync with the backend
  DEFAULT COMMISSION RATE; document that they must move together if the rate ever changes.
- **Empty/zero state:** when the field is blank or ≤0, show "Enter the fee to see the commission"
  (not a stray "S$0") — Jamie: the four states apply to a single computed line too.

This step alone satisfies the founder's "show the commission" ask and de-risks every activation. It
is independent of the fee-basis decision below.

### 2.4 Open founder decision (BLOCKS the model change — present options + recommendation)

The broader change requires the founder to choose. **Do not build 2.5 until these are answered.**

**Decision D-1 — Which fee bases must InTuition support?**

| Option | What the admin enters | Commission base | Build cost | Notes |
|---|---|---|---|---|
| **A. Monthly only (status quo)** | monthly fee | 50% of first month | none | Mislabels hourly/package deals; founder already said "not monthly." |
| **B. Monthly + Hourly×Hours** | rate + hours/period | 50% of (rate × hours) → first-period value | small (1 enum col + hours col) | Covers the two commonest SG cases. **Recommended minimum.** |
| **C. Full set: Monthly \| Hourly×Hours \| Per-session \| Package/fixed** | basis-specific fields | 50% of the first-period / package value | medium (enum + 1–2 nullable numeric cols + per-basis validation) | Most accurate; more user interface + validation surface; risk of over-build if the business only does A/B. |

**Recommendation (Dana + product-designer):** **Option B now**, with the fee basis column modelled
as an enum from day one so C is an additive enum value later (no second migration of the shape). Do
**not** build C speculatively — confirm with real deal data that per-session/package occur often
enough to justify the user interface (business-case-before-build).

**Decision D-2 — Is commission ALWAYS 50% of the first month/period, or does it vary?**

- Today: hardcoded 0.5 (76, 487) of a monthly fee.
- Options: (i) **always 50% of the first period** (whatever the basis) — simplest, matches business requirements document
  intent; (ii) **50% but admin may override** the computed amount at activation (captures one-off
  negotiated deals; the commission rate/commission amount sgd columns already exist to store an
  override); (iii) **a different rule per basis** (e g. fixed agency fee for packages).
- **Recommendation:** (i) as the default rule + (ii) an optional admin override of the **amount**
  (not the rate) at activation, since the columns already support storing whatever amount is agreed.
  Avoid (iii) until a real package deal exists.

**Decision D-3 — Show/confirm commission in the activate modal?**
- **Recommended: YES** — and 2.3 ships this immediately for the monthly case regardless of D-1/D-2.

### 2.5 Design of the model change (build ONLY after D-1/D-2) — sketch, not final

Contingent on Option B/C + D-2. Indicative shape:

- **Schema (Liquibase, new changeset — number to be allocated by dba at build time; leave gaps for
  reserved numbers per SHARED-LESSONS):** add fee basis VARCHAR (string-enum, **never** a named-PG
  ALTER TYPE — INT-062/INT-065 crash-loop rule) defaulting MONTHLY for backfill; add nullable
  fee rate sgd / fee hours (or sessions) columns as the chosen bases require. Keep
  monthly fee sgd as a generic "first-period fee" or rename via a forward-only changeset — decide
  with dba; an applied changeset is immutable (fix-forward only).
- **data transfer object/frontend-facing service:** Activate Match Request gains fee Basis + basis-specific fields with per-basis
  validation (bff-specialist owns the contract; a sealed/typed request per basis or a single record
  with conditional @Assert yes validation). The computed commission can be returned in the activate
  response so the frontend confirms server truth, not just its own arithmetic.
- **Service:** activate Match computes the first-period value from basis, amount, hours/sessions,
  then multiplies the first-period value by the commission rate and rounds to a whole Singapore dollar; persists fee basis + inputs for audit.
- **frontend:** the activate modal becomes a small basis selector (Hick's: ≤ the chosen N options) that
  reveals only the fields for the selected basis (progressive disclosure), with the live
  "Commission (50%): S$X" line from 2.3 recomputing per basis.
- **Ledger labels:** replace the hardcoded "50% of the monthly fee" header with basis-aware copy
  (match-between-system-and-real-world; user's language, not "monthly" for an hourly deal).

### 2.6 Acceptance criteria

**First step (2.3) — ships now, no decision needed:**
- **AC-5:** *Given* the activate modal on /admin/matches or /admin/commission, *When* the admin
  types a positive fee, *Then* a live "Commission (50%): S$X" line shows 50% of the entered fee, rounded to a whole Singapore dollar within
  one render; *When* the field is empty/≤0, *Then* a prompt ("Enter the fee…") shows instead of a
  stray amount, and Activate stays disabled.
- **AC-6:** activating still POSTs the existing the first-month fee in Singapore dollars contract and the persisted
  commission amount sgd equals the amount the admin saw (frontend arithmetic  equals  backend arithmetic).

**Model change (2.5) — only after D-1/D-2 are answered:** ACs to be written against the chosen
basis set (one AC per supported basis: entered inputs → expected first-period value → expected
commission), plus a backfill AC (existing commissions read as MONTHLY with the same amount).

### 2.7 Test notes (qa)

- **First step:** frontend tests assert the live commission line (typed fee → expected S$X), the empty
  state, and that the persisted/POSTed amount matches what was shown (payload-stability test — the
  commission is a revenue/wire-contract value, INT Wave-6 lesson). No backend change → no new backend test.
- **Model change:** backend unit tests per basis for the first-period→commission arithmetic (rounding at
  rounding the first-period value multiplied by the commission rate to a whole Singapore dollar); a migration backfill IT (databasechangelog head + fee basis is MONTHLY
  for pre-existing rows); contract test that an invalid basis/field combo → 400. Gates unchanged
  (backend ≥97.5% line+branch; frontend ≥97.5% line / ≥88% branch).

---

## 3 — Measurement / instrumentation (data-engineer to wire at build)

- **INT-072:** counter for cancel outcomes by result — match-cancellation counts grouped by success, not-cancellable or error
  — so the team can SEE whether the "can't cancel" report recurs after the fix (and whether bucket b
  is the real driver). North-star guardrail: cancels that reopen a job feed back into liquidity.
- **INT-073:** when the model change ships, record fee basis on the activation analytics event so
  the business can see the real distribution of bases (validates whether Option C is ever justified).

## 4 — Out of scope (explicit)

- Refunds / clawback of a PAID commission on cancel (intentionally untouched, 519-522).
- Multi-commission per match / instalment billing.
- Interview surfaces gaining a Cancel-Match action (none today; not requested).
- Any change to Match Status values.

---

**Current follow-up:** use INT-157 v2 for the resolved agreed-plan commission basis. Do not reopen the
earlier D-1/D-2 alternatives. Track only its explicitly pending commercial defaults and the separate
cancellation/refund policies in CURRENT-REQUIREMENTS; new implementation requires its own review/evidence.
