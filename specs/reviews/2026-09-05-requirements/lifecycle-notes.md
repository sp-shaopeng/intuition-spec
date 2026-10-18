# InTuition lifecycle requirements audit

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](corrections.md) and [current requirements](../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

Scope: matching, parent selection, interview negotiation, commission, cancellation/reopening, end of tuition and reviews. Source review only: no code edits, tests, runtime actions, or external actions. Repository root: REDACTED local path. Paths below are absolute. Recommendations are proposals, not approved requirements.

## Authority and interpretation

- **Confirmed by user during this audit:** “Mark commission paid” means the agency has received the tutor's commission. Parent-to-tutor tuition payment is a separate event. This is no longer an open decision. It agrees with the current FAQ, REDACTED local path, and original commission payer rule, REDACTED local path. M18's “parent pays → admin marks PAID” shorthand must be made explicit so it does not imply parent tuition payment is agency revenue.
- The code/reference application programming interface establish current behavior, but do not automatically supersede product requirements. In particular, product requirements document §20.3 explicitly says the money flow reuses §8.3/AC-05/BR-010 verbatim (REDACTED local path). INT-157 reconfirms the due-after-first-session trigger is unchanged (REDACTED local path). Thus the three-status implementation does not constitute a documented decision to remove first-session confirmation.
- Dates/status headings are not a reliable authority by themselves: specs 014, 017, 019 and 024 have HISTORICALLY MARKED IMPLEMENTED frontmatter but retain draft language and unchecked tasks. Their explicit folded decisions are more useful than old status boilerplate.
- Do not reopen resolved history: no-listing application is intentional; post-cancel application reopening is intentional; max three interviews means concurrent per parent posting; 72-hour unanswered-negotiation expiry is ratified by Boss conditions; Singapore time-only scheduling is deliberate; accepted-slot prefill is editable by design; COMPLETED rather than CLOSED is the authoritative match enum.

## Requirement inventory

| Area | Intended/current requirement | Primary evidence | Assessment |
|---|---|---|---|
| Parent choice | Parent selects a tutor or requests an interview; selection is mutable until admin confirms. Neither operation creates a match. Admin liaises and both parties confirm. | REDACTED local path; REDACTED local path | Settled model; evidence of “both confirm” remains an operational detail. |
| Matching | Admin-only; one PENDING/ACTIVE match per job; tutor can hold multiple matches. Both admin matching entry points converge on one assignment transition. | REDACTED local path; REDACTED local path | Settled. |
| Assignment effects | Job ACTIVE→MATCHED, winning application→MATCHED, other APPLIED/SHORTLISTED applications→SUPERSEDED. Contact becomes available after assignment. | REDACTED local path; REDACTED local path | Settled; other interviews are not included in the cross-object transition table. |
| Cancellation | PENDING/ACTIVE match can be cancelled; COMPLETED/CANCELLED are terminal. Reopen job and MATCHED/SUPERSEDED applications; cancel uncollected commission. | REDACTED local path; REDACTED local path | Clear current baseline; commercial exceptions are missing. |
| Commission amount | One-time 50% of agreed first-month plan, locked at activation; plan lives on commission; legacy plan fields nullable. No routine recomputation for actual later lessons. Amount override 1…fee remains available. | REDACTED local path; 73; 86 | Lock, storage, legacy handling and flexible-frequency input settled. Two commercial choices explicitly remain open. |
| Commission payer | Tutor pays agency commission. Parent pays tutor tuition separately. PAID records agency receipt of tutor commission. | User confirmation during audit; REDACTED local path | Confirmed, not open. |
| First-session obligation | No commission invoice before first session is confirmed by tutee or admin override; record confirmation; reminder/escalation after seven days. | REDACTED local path; REDACTED local path; REDACTED local path | Clear requirement currently unmet by inspected lifecycle implementation; do not ask if requirement exists. |
| Financial recording | Off-platform collection; admin marks PAID, guarded by identity/amount confirmation. Current payment also sets match COMPLETED and match Completed At. | REDACTED local path; REDACTED local path | Collection semantics now confirmed; correction policy and meaning of match completion are incomplete. |
| Interview initiation | One to three proposed 15-minute slots within seven days; max three active interviews per posting. Newer spec permits interviews while job PENDING; rejection must close pending invite/interview. | REDACTED local path; REDACTED local path; REDACTED local path | Explicit supersessions of older ACTIVE-only/per-account cap rules. |
| Negotiation | Alternating counterparty turn; accept one slot; NOT INTERESTED terminal; CANT MAKE IT counter remains active; agreement→PENDING SCHEDULING. | REDACTED local path; 150; 453 | Settled. |
| Interview expiry | PROPOSED/COUNTER PROPOSED expire after 72h inactivity; other active states consume cap but are not in this sweep. | REDACTED local path; 611 | Window/mechanism settled; real slot expiry and admin handoff deadline are missing. |
| Admin scheduling | Admin may force nonterminal interview to pending scheduling, change agreed time, supply link; terminal outcomes immutable. Accepted slot auto-fills editable control. | REDACTED local path; 463; REDACTED local path | Authority settled; safeguards when changing agreed appointments are unspecified. |
| Reviews | Each participant may review once per COMPLETED match; tutor ratings aggregate publicly. | REDACTED local path; REDACTED local path | Implemented/declared; older product requirements document Phase-2 review line is historical. Eligibility tied to agency payment creates an unaddressed product consequence. |
| Analytics | Interview conversion is completed interviews within the requested-at cohort; later periods may change cohort completion rate. Mark-paid confirmation on all surfaces. | REDACTED local path; REDACTED local path | Settled; not a new ambiguity. |

## Nine decisions / missing use cases worth resolving

### L1 — Cancellation after tuition starts: debt, refund, and replacement policy (high business impact)

**Evidence:** The first session triggers commission (product requirements document 49, business requirements document 473). product requirements document §8.2 requires explicit waiver/dispute decisions after invoice (REDACTED local path). Later cancellation spec makes every unpaid commission CANCELLED and expressly excludes paid refunds/clawbacks (REDACTED local path, 338). Current service cancels a PENDING commission whenever its match is cancelled (REDACTED local path). Public terms default to non-refundable unless separately stated (REDACTED local path). FAQ promises no long-term lock-in (REDACTED local path).

**Scenario:** Three lessons occur, tutor owes the agency S$270 but has not paid. Parent decides the tutor is unsuitable and asks for replacement. Cancel currently voids that receivable. If the tutor paid yesterday, the match is terminal, cancellation is refused, and there is no defined replacement/credit path. Identical service outcomes are treated differently based on collection timing.

**Decision question:** For cancellation before first lesson, after first lesson but before payment, and after payment, what commission is owed, waived/refunded/credited, and is a replacement match charged again? Does a free/unsuccessful trial count as the first qualifying session?

**Proposal:** Publish a small cancellation matrix keyed to service delivery, cause and refund/credit decision, not simply invoice PAID status. No qualifying lesson→no commission; post-start cancellation retains debt unless an explicit recorded waiver/credit applies. Preserve original collected transactions. Confirm the commercial amount/window with the founder before promising free replacements or refunds. “Refund tooling was outside a defect-fix scope” is not a complete customer policy.

### L2 — Correcting an honest money-entry error without rewriting history (high business impact)

**Evidence:** INT-157 locks fee and basis forever and defers renegotiation (REDACTED local path, 91). Spec019 labels Mark PAID irreversible (REDACTED local path). Current payment records current paid At and full amount, then completes the match (REDACTED local path). The current enum only has PENDING/PAID/CANCELLED (REDACTED local path).

**Scenario:** Admin locks nine sessions instead of eight, or records commission paid against the wrong tutor after seeing a bank transfer. Another common case: tutor pays S$100 against S$270 owed. Confirm dialogs do not eliminate these cases. Cancelling/rematching to repair a fee also triggers marketplace and application side effects.

**Decision question:** Who can correct activation/receipt mistakes, what evidence is required, how is a partial payment represented, and can receipt date differ from the entry date?

**Proposal:** Treat a correction as a separate audited adjustment/reversal linked to the immutable original, with reason, actor and receipt reference/date. Support a recorded outstanding balance or explicitly document partial payments as unsupported with an operator procedure. Keep agreed-plan changes separate from correction of a transcription error. No silent delete/reset of PAID.

### L3 — Four weeks versus 4.33 weeks remains explicitly unapproved (high money clarity impact)

**Evidence:** product requirements document says frequency×4: twice weekly→eight sessions (REDACTED local path). INT-157 v2 says provisional round using frequency×4.33: one/two/three→4/9/13 (REDACTED local path) and explicitly “NEEDS FOUNDER DECISION” on 4.33 versus four (104–109).

**Scenario:** S$60/hour, 1.5 hours, twice weekly gives S$720 first month/S$360 commission under four weeks, versus S$810/S$405 under the nine-session default. A rounded annualized average adds S$45 commission without an extra actual session being agreed.

**Decision question:** Is “first month” an agreed number of lessons, four weeks, calendar-month actual sessions, or an annualized 4.33-week estimate?

**Proposal:** Make explicitly agreed session count authoritative, show the dated/numbered plan before both parties commit, and choose one clearly labeled estimate for convenience. Until founder ratification, do not describe 4.33 as an approved policy. FLEXIBLE requiring manual entry, nullable legacy snapshots, and storing the snapshot on commissions are already resolved and should not be reopened.

### L4 — Negotiated rate outside advertised ranges and proof of agreement (medium/high money clarity impact)

**Evidence:** INT-157 explicitly leaves rate hard-clamping open; provisional default is suggested midpoint with no hard clamp (REDACTED local path). M18 says both parties confirm before assignment (REDACTED local path). Activation later locks the plan (REDACTED local path).

**Scenario:** Parent advertised S$40–50, tutor advertised S$60–80, coordinator agrees S$55 verbally then activates at S$65. There is no defined customer acknowledgment of the exact locked fee separate from profile/budget values and general signup terms.

**Decision question:** Can negotiated rates lie outside both advertised ranges, and what evidence records both parties' acceptance of the exact final plan? Activation is already the documented lock event in INT-157; “locked at match” needs copy alignment, not a new timing decision.

**Proposal:** Keep no hard clamp so legitimate negotiation is possible, but require an out-of-range explanation and capture both parties' acceptance of the exact plan before activation. Preserve activation as the documented amount-lock event and align “locked at match” copy to it. The proposed evidence/bounds policy still needs approval.

### L5 — Agency collection, actual end of tuition, replacement and review eligibility are conflated (high product impact)

**Evidence:** product requirements document calls match and commission independent dimensions (REDACTED local path) but its authoritative M18 lifecycle sets match COMPLETED on commission PAID (854, 2069). Current payment also stamps assignment “until” date (REDACTED local path). Reviews require COMPLETED (REDACTED local path). Terminal matches cannot cancel (REDACTED local path).

**Scenario:** Tutor pays agency after week one but teaches for another year. History marks the assignment ended at week one and reviews immediately unlock. A tutor who teaches well for two months but delays agency payment cannot be reviewed. A poor tutor who is replaced before commission settlement also cannot be reviewed. Later replacement cannot use normal cancellation of the completed match.

**Decision question:** Given the settled COMPLETED-on-PAID definition, how should actual teaching history end, reviews unlock, and replacement become available? The existing match enum is defined; a separate tuition lifecycle is missing.

**Proposal:** Track agency placement/collection and actual tuition status separately. Preserve current COMPLETED as placement completion, label it accordingly, and add actual started/ended outcomes only when evidenced. Consider basing review eligibility on a qualifying delivered lesson/arrangement experience; this would amend the current COMPLETED-only rule. Explicitly decide treatment of a cancelled arrangement that nevertheless delivered lessons and review edits/replies/moderation; do not silently switch the existing rule.

### L6 — Reopening after cancellation needs parent-intent and applicant-validity rules (medium/high product impact)

**Evidence:** Automatic reopen of job and MATCHED/SUPERSEDED applications is explicitly resolved (REDACTED local path; REDACTED local path). Current cancel applies it unconditionally to MATCHED jobs (REDACTED local path). Earlier business requirements document distinguishes a parent closing their own request from a tutor withdrawing (REDACTED local path, 876).

**Scenario:** Parent cancels because the child no longer needs tuition, but the app relists the request. Or parent cancels for tutor misconduct and the same former winner reappears as APPLIED. Previously superseded tutors may now be unavailable or already at their active-application quota.

**Decision question:** Which cancellation reasons reopen demand, whether the former tutor is excluded, and whether old applications need renewed interest after a long assignment?

**Proposal:** Preserve reopen as the replacement-request default, but capture “find another tutor” versus “no longer seeking tuition.” Restore old applicants as candidates needing availability reconfirmation; exclude explicitly rejected/unsafe former tutors. Define whether reopened rows may temporarily exceed quota. This is a missing exception policy, not a claim that the settled default reopen rule was an implementation error.

### L7 — Cross-lifecycle cleanup of interviews when a job is assigned, rejected or withdrawn (high wasted-work/consent impact)

**Evidence:** Assignment transition enumerates job, winning/other applications and match, but no interview outcomes (REDACTED local path). Current assignment expires pending invitations (REDACTED local path) but has no corresponding interview cascade. Newer spec021 explicitly requires pending invites/interviews to be voided after job rejection (REDACTED local path).

**Scenario:** Parent has interviews booked with tutors A and B, then admin assigns A. B still has a meeting and possibly a reminder for a job that is no longer available. Another parent starts negotiation while the post awaits approval; admin rejects it. Rejection cleanup is specified, but the outcome of already-SCHEDULED interviews and active negotiation on assignment or voluntary closure is not consistently stated.

**Decision question:** Which interviews survive assignment (including with the winner), who is notified, should meeting reminders be cancelled, and do cancelled pending negotiations revive when a job later reopens?

**Proposal:** Define a job×interview transition table: cancel unnecessary future interviews on assignment/withdrawal with a typed reason and cancel reminders; preserve historical completed outcomes; do not revive appointments automatically on reopening. Rejection of pending items is already required by spec021 and should be treated as requirement verification, not a new product vote. Whether a winner's future interview should still take place needs explicit handling.

### L8 — Real proposed-time expiry and the admin handoff have no complete deadline contract (medium/high scheduling impact)

**Evidence:** Propose/counter can offer any future slot within seven days (REDACTED local path). Accept validation checks ownership/state, not a separate time-freshness rule (456). Only PROPOSED/COUNTER PROPOSED age out at 72h (468); PENDING SCHEDULING consumes one of three slots (143). Current sweep confirms those two states only (REDACTED local path).

**Scenario:** Parent proposes tomorrow 9am. Tutor responds tomorrow 10am: interview is not 72h old, but every proposed time is past. Or both agree and admin misses the slot; the interview remains “Time agreed” and consumes capacity indefinitely. Seven-day slots and 72-hour inactivity are different clocks.

**Decision question:** Does a proposal expire when its last usable slot passes, what minimum lead time guarantees admin can schedule, and how long may an agreed interview wait for a link?

**Proposal:** Require acceptance of a still-future usable slot; when all offers pass, request a new round or mark expired with an explicit reason. Set a concrete admin handoff service-level agreement and escalate before the agreed time; if missed, reopen negotiation with parties notified rather than silently treating an expired agreement as valid. Keep the ratified 72-hour inactivity rule as an additional backstop, not the only expiry definition.

### L9 — Admin override/rescheduling authority is settled; re-consent and collision protections are not (medium/high scheduling impact)

**Evidence:** Negotiation spec explicitly permits admin override at any nonterminal state (REDACTED local path) and different scheduled time (463). Spec014 keeps accepted-time prefill editable (REDACTED local path). The old MVP procedure cancels/rebooks for time changes (REDACTED local path), while current force Pending accepts any nonterminal interview and schedule allows a new time (REDACTED local path, 522). Allowing overlapping proposals was proposed in negotiation D7 (REDACTED local path), but it does not settle overlapping confirmed appointments.

**Scenario:** Both agree to Tuesday 6pm; admin changes it to Wednesday 6pm, or force-pending moves an already scheduled interview back to ready-to-schedule. Another admin books the same parent/tutor at the same time for a different job. The system treats changed appointments as scheduled without a defined distinction between mutual agreement and coordinator override.

**Decision question:** Must material time changes be reaccepted, what reason/evidence permits overriding a participant agreement, and do confirmed appointments reserve parent/tutor time globally?

**Proposal:** Keep admin override but record who changed what and why; notify both parties explicitly of old/new time and require acknowledgment for a materially different slot unless consent was recorded off-platform. Permit overlapping tentative proposals; prevent or visibly warn on overlapping confirmed appointments across all jobs for either participant. Provide one documented reschedule procedure that also updates reminders instead of relying on force-pending as an undocumented workaround.

## Clear requirements that should not be presented as unanswered questions

1. **First-session confirmation exists as a requirement.** product requirements document AC-05 (REDACTED local path), §20.3 (2034) and INT-157 (REDACTED local path) all preserve it. Current activation builds commission PENDING (REDACTED local path), and payment advances based on invoice/match status only (REDACTED local path). This is an implementation/scope reconciliation item, not an open “should we require confirmation?” question. A narrow genuine conflict remains: business requirements document 480 allows admin override at any time; product requirements document AC-05 1507 describes an overdue match and product requirements document 59 says after seven days. Proposed default: admin may override earlier only with recorded evidence of first session; seven days is escalation, not permission to assert an unheld session occurred.
2. **Payer is confirmed.** Do not ask the founder to choose between parent and tutor again. Receipt of parent tuition is not sufficient to mark agency commission PAID.
3. **Pending-post rejection cleanup is explicit.** Spec021 24 already requires void/close of pending invite/interview. Any missing source path is a requirement defect, separate from the still-missing assignment/withdrawal cascade policy.
4. **Specs014/019 schedule visibility and mark-paid dialogs are clear fixes.** They add no commercial reversal, installment, or admin override policy. Do not mistake “confirmation dialog present” for those business questions being answered.
5. **Spec024 does not settle lifecycle billing.** Its relevant folds are tutor identity resolution for invitations and planning-area completeness. It does not supersede first-session confirmation, debt, refunds or tuition completion.

## Suggested decision order

First settle/reconcile first-session evidence and commercial cancellation debt rules (L1 plus the clear requirement). Next establish money corrections (L2), then explicitly ratify first-month count and rate negotiation (L3/L4). Define tuition end/review/replacement outcomes (L5/L6), then complete job/interview transitions and scheduling deadlines (L7/L8/L9). No speculative code should be required to answer these product questions.
