# Requirements — Interview scheduling user experience + admin visibility (014)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> EARS + Gherkin per atomic requirement, grounded 2026-06-22. Source: CONSOLIDATED md (#210.1/.2/.3, #211,
> raw-handle finding).

## Scope note (verified)

- Admin list Interview Service list Admin Interviews (457-462) = find All when no status filter → correct.
  The gap is the frontend tab map Interview Queue implementation reference: ACTION REQUIRED: REQUESTED, SCHEDULED: SCHEDULED,
  HISTORY: COMPLETED,CANCELLED,NO SHOW, ALL:no value. The Interview Status enum has **10 values**
  (Interview Status implementation reference): REQUESTED, SCHEDULED, COMPLETED, CANCELLED, NO SHOW, PROPOSED, COUNTER PROPOSED,
  TUTOR DECLINED, PENDING SCHEDULING, **EXPIRED**. Missing from every tab except ALL in the current frontend map:
  PROPOSED, COUNTER PROPOSED, PENDING SCHEDULING, TUTOR DECLINED, **and EXPIRED** → the fix must place all
  10 (EXPIRED → history).
- Admin Interview Response ALREADY carries tutor Display Name + tutor Public Handle (43-44) AND the accepted slot
  agreed Timeslot Id + timeslots (38-39). REQ-002 + REQ-005 are therefore frontend-only — no backend field work.
- Admin No-Show buttons live in a flat button row (Interview Detail implementation reference, 5 buttons) → overflow/clipping.

---

## REQ-001 — Admin interview queue surfaces ALL live statuses (no invisible interviews)

- **Type:** functional · **Priority:** Must · **Severity:** S2 · **Source:** #210.1
- **Rationale:** An accepted-but-unscheduled interview (PENDING SCHEDULING) — exactly the state needing admin
  action — is invisible in the action queue because no tab includes it. Admins miss work that the system told them
  (via notification) exists.
- **Behaviour (EARS):** WHEN the admin views the interview queue, the system SHALL place EVERY one of the 10
  Interview Status values (Interview Status implementation reference) in exactly one non-ALL tab: the action/needs-scheduling tab
  SHALL include REQUESTED, PROPOSED, COUNTER PROPOSED, PENDING SCHEDULING, and TUTOR DECLINED (the states
  awaiting admin action); the scheduled tab SCHEDULED; and history the terminal states COMPLETED, CANCELLED,
  NO SHOW, **and EXPIRED** (EXPIRED is terminal — its home is history). The union of the tab status-filters SHALL
  equal the full Interview Status enum set (no status orphaned outside ALL — EXPIRED included). The tab map SHALL
  be DERIVED FROM / ASSERTED AGAINST the live Interview Status enum so a new (or re-introduced-gap) status cannot
  silently fall through.
- **Owner:** frontend-expert (· bff-specialist if a server-side "needs action" grouping is preferred)
- **Acceptance criteria (Gherkin):**


- Scenario: An accepted-but-unscheduled interview appears in the action tab.
- Given an interview in PENDING SCHEDULING.
- When the admin opens the interview queue action tab.
- Then that interview is listed.
- Scenario: Every status is reachable (enum-driven, set-equality).
- Given the set of all Interview Status values read from the enum at test time.
- And the union of every non-ALL tab's status-filter.
- And EXPIRED appears in the history tab.
- Scenario: Negotiation states are visible.
- Given interviews in PROPOSED / COUNTER PROPOSED / TUTOR DECLINED.
- Then each appears in an appropriate (non-ALL) tab.


- **Required evidence:** an **enum-driven** contract test that computes the enum set from the live Interview Status
  and asserts SET-EQUALITY with the union of the tab maps (NOT a frozen snapshot — it must fail if a value is
  unplaced); RTL test (PENDING SCHEDULING shows in the action tab); live capture of an accepted interview in the
  queue. Under REQ-001 implementation reference.
- **Dependencies:** none.

## REQ-002 — Scheduling auto-fills the accepted timeslot

- **Type:** functional / user experience · **Priority:** Should · **Severity:** S3 · **Source:** #210.2
- **Rationale:** When a tutor has accepted a proposed slot, the admin should not re-type it; pre-filling reduces
  error and effort.
- **Behaviour (EARS):** WHEN the admin opens the schedule control for an interview that has an accepted/agreed
  timeslot, the system SHALL pre-fill the datetime control with that accepted slot (editable). IF no slot is
  agreed, the control SHALL open empty. The accepted slot is ALREADY on the admin payload
  (Admin Interview Response implementation reference agreed Timeslot Id + timeslots) — **frontend-only**, resolve the slot by
  agreed Timeslot Id from timeslots; no backend field work.
- **Owner:** frontend-expert (frontend-only — data already present; no bff task)
- **Acceptance criteria (Gherkin):**


- Scenario: Schedule pre-fills the accepted slot.
- Given an interview with a tutor-accepted timeslot.
- When the admin opens the schedule control.
- Then the datetime field is pre-filled with that slot (editable).
- Scenario: No agreed slot opens empty (guard).
- Given an interview with no agreed slot.
- When the admin opens the schedule control.
- Then the field opens empty.


- **Required evidence:** confirm the accepted slot is on the admin payload (or add it); RTL test of pre-fill; live
  capture. Under REQ-002 implementation reference.
- **Dependencies:** REQ-004 (the datetime control it pre-fills).

## REQ-003 — Record-No-Show (and all interview actions) are fully visible, not clipped

- **Type:** functional / user experience · **Priority:** Should · **Severity:** S3 · **Source:** #210.3
- **Rationale:** A clipped/invisible "Record No Show" means an admin can't perform a real lifecycle action.
- **Behaviour (EARS):** WHEN the admin interview detail renders its action buttons (Interview Detail implementation reference:
  Set Meeting Link, Mark Complete, No Show Tutee, No Show Tutor, Cancel), the system SHALL lay them out so every
  action is fully visible and reachable at all supported viewports (wrap/stack/group rather than clip), with no
  button cut off next to another.
- **Owner:** frontend-expert · uiux-designer
- **Acceptance criteria (Gherkin):**


- Scenario: All interview actions are visible.
- Given the admin interview detail at desktop and at 390px.
- Then Mark Complete, No Show (Tutee), No Show (Tutor), Cancel, and Set Meeting Link are all fully visible.
- And none is clipped or overlapped.


- **Required evidence:** component test / visual check; live captures at desktop + 390px. Under REQ-003 implementation reference.
- **Dependencies:** none.

## REQ-004 — A usable date/time picker for scheduling (all roles)

- **Type:** functional / user experience · **Priority:** Should · **Severity:** S3 · **Source:** #211
- **Rationale:** The current datetime control is unpleasant to use everywhere it appears (admin schedule, tutee/
  tutor slot proposal). A clear, low-friction control reduces scheduling errors.
- **Behaviour (EARS):** WHEN a user sets an interview date/time (admin schedule; tutee/tutor slot proposal), the
  system SHALL present a usable date+time control (clear affordance, sensible Singapore time defaults, mobile-friendly,
  keyboard-accessible) that prevents obviously-invalid entries (e g. past datetimes for a future interview) with a
  clear message. The change SHALL NOT alter the wire format each endpoint expects: admin
  scheduled At stays an Singapore time ISO local datetime without offset/Z; proposal/counter slots stay Coordinated Universal Time Z
  instants under INT-039. Convert explicitly between Singapore time display and Coordinated Universal Time wire values, independent of browser timezone.
- **Owner:** uiux-designer · frontend-expert
- **Acceptance criteria (Gherkin):**


- Scenario: The datetime control is usable and validated.
- Given any role setting an interview datetime.
- Then the control is clear, mobile-friendly, and keyboard-accessible.
- And a past datetime (for a future interview) is rejected with a clear message.
- And the value sent to the backend matches the existing expected format.


- **Required evidence:** RTL/a11y test of the control + past-datetime guard + **a BLOCKING wire-format-unchanged
  test** (PINNED): the picker is frontend-only, so the value submitted to submit operation for admin interviews id schedule MUST
  remain a Local Date Time-shaped ISO-8601-local string (no timezone offset, no Z) that the backend
  Schedule Interview Request scheduled At (java time.Local Date Time, 14-17) parses unchanged — assert the exact
  string the frontend sends for a chosen datetime equals the pre-change format (a picker swap cannot silently change the
  payload to e g. an offset/Z-suffixed instant). Separately assert proposer/counter slots use Coordinated Universal Time Z
  instants (e g. 14:00 Singapore time → 06:00Z) and agreed-slot prefill converts Coordinated Universal Time to Singapore time correctly in any browser timezone.
  Both existing backend data transfer object contracts remain unchanged. Live
  captures on admin + a proposer flow. Under REQ-004 implementation reference.
- **Dependencies:** none (REQ-002 pre-fills into it).

## REQ-005 — Show the tutor's display name, not the raw handle, on admin Interview/Applicants screens

- **Type:** functional / user experience · **Priority:** Should · **Severity:** S3 · **Source:** raw-handle finding
- **Rationale:** Rendering tutor wei jie tan reads like a leaked internal id and is inconsistent with the Matches
  screen, which shows a proper name. The data already exists (Admin Interview Response tutor Display Name, 43).
- **Behaviour (EARS):** WHEN an **admin** Interview or Applicants screen shows a tutor, the system SHALL display the tutor's
  display name (tutor Display Name) and SHALL NOT show the raw handle as the primary identity. IF a
  display name is absent, the system SHALL fall back to a human-friendly label (not a raw tutor … slug).
  This requirement does not authorize adding names to tutee-facing responses or screens; those retain their
  counterparty-personal information rules and public-handle identity until an applicable disclosure rule permits otherwise.
- **Owner:** frontend-expert (frontend-only for the admin screens — Admin Interview Response tutor Display Name 43
  already populated; no backend task)
- **Acceptance criteria (Gherkin):**


- Scenario: Admin Interview screen shows the display name.
- Given an interview with a tutor Display Name.
- Scenario: Admin Applicants screen shows the display name.
- Given an applicant tutor with a display name.
- Then the admin applicants screen shows the name, not a raw tutor_ slug.
- Scenario: Missing name falls back gracefully on the admin screen (guard).
- Given a tutor with no display name.
- Then a human-friendly label is shown (never a raw tutor_ slug as the primary identity).


- **Required evidence:** RTL test (name shown, not handle; fallback); live capture on the admin Interview +
  Applicants screens. Under REQ-005 implementation reference. (No backend field task — Admin Interview Response tutor Display Name
  already present.)
- **Dependencies:** none (data already present on Admin Interview Response).

---

## Traceability summary

| REQ | One-line | Type | Priority | Severity | Owner |
|---|---|---|---|---|---|
| REQ-001 | Admin queue tabs cover ALL live statuses (PENDING SCHEDULING etc. no longer invisible) | functional | Must | S2 | frontend-expert |
| REQ-002 | Schedule auto-fills the accepted timeslot | functional/user experience | Should | S3 | frontend-expert · bff-specialist |
| REQ-003 | All interview action buttons visible, not clipped | functional/user experience | Should | S3 | frontend-expert · uiux-designer |
| REQ-004 | Usable, validated datetime picker (all roles) | functional/user experience | Should | S3 | uiux-designer · frontend-expert |
| REQ-005 | Show tutor display name, not raw handle, on Interview/Applicants | functional/user experience | Should | S3 | frontend-expert |

## Resolved decisions (were open questions — closed at design, 2026-06-22)

1. **REQ-001 grouping — RESOLVED:** frontend drives the tab map from the Interview Status enum (set-equality contract
   test). All 10 values placed: action=REQUESTED,PROPOSED,COUNTER PROPOSED,PENDING SCHEDULING,TUTOR DECLINED,
   scheduled=SCHEDULED, history=COMPLETED,CANCELLED,NO SHOW,EXPIRED. The v1.0 omission of EXPIRED is fixed. (A
   server-side "needs-action" grouping is not needed for this slice — the enum-driven frontend map + contract test prevents
   the orphan class.)
2. **REQ-002/REQ-005 — RESOLVED (no backend task):** Admin Interview Response already carries agreed Timeslot Id+
   timeslots (38-39) and tutor Display Name+tutor Public Handle (43-44). Both REQs are frontend-only; the two
   v1.0 "confirm/add if absent" backend hedges are removed.

## Open question for the founder (REQ-004 only — non-blocking)

1. **REQ-004 picker:** native date-and-time input (zero-dep, a11y, mobile) vs a custom/library control.
   uiux-designer recommends; frontend-expert vets bundle/a11y. Default lean: native unless a gap is shown. (Does not
   block REQ-001 — the SEND-BACK item.)
