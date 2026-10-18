# Design — Interview scheduling user experience + admin visibility (014)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Owned by frontend-expert + uiux-designer + bff-specialist + software-engineer. Grounded 2026-06-22.

## REQ-001 — admin queue tab coverage

Interview Queue implementation reference status Params is the bug. The Interview Status enum has **10 values**
(Interview Status implementation reference): REQUESTED, SCHEDULED, COMPLETED, CANCELLED, NO SHOW, PROPOSED, COUNTER PROPOSED,
TUTOR DECLINED, PENDING SCHEDULING, EXPIRED. **Every one must land in exactly one non-ALL tab** — including
EXPIRED, which the v1.0 draft union omitted (that omission was the exact orphan-class defect this REQ exists to
kill). Redefine the tabs so the union  equals  the full enum:

| Tab | Statuses | Rationale |
|---|---|---|
| **ACTION REQUIRED** (action / needs-scheduling) | REQUESTED, PROPOSED, COUNTER PROPOSED, PENDING SCHEDULING, TUTOR DECLINED | admin action/follow-up grouping; TUTOR DECLINED is terminal and frees capacity but is deliberately retained here for admin follow-up |
| **SCHEDULED** | SCHEDULED | confirmed, upcoming |
| **HISTORY** | COMPLETED, CANCELLED, NO SHOW, **EXPIRED** | terminal states — EXPIRED is terminal (negotiation lapsed / job left ACTIVE → cap slot freed), so HISTORY is its natural home |
| **ALL** | no value (no filter) | every status, unfiltered |

Coverage check across the 10 values: ACTION REQUIRED using 5 + SCHEDULED using 1 + HISTORY using 4 = **10/10**, no orphan.

The tab map MUST be **derived/asserted against the live Interview Status enum**, not a hand-copied list — so a
future status (or a re-introduced gap) fails the test until it is placed. No backend change (find By Status In + find All
already support any subset).

## REQ-002 — auto-fill accepted slot

**Data already present — NO backend task.** Admin Interview Response implementation reference already carries agreed Timeslot Id +
the list of proposed time slots. REQ-002 is **frontend-only**: the schedule control
(Interview Detail implementation reference schedule action 44) resolves the accepted slot from timeslots by agreed Timeslot Id and
pre-fills the datetime when present, empty otherwise. Feeds REQ-004's control. (The v1.0 "confirm/add if absent"
hedge is removed — it would dispatch a no-op backend agent.)

## REQ-003 — button layout

Interview Detail implementation reference flat row of 5 buttons → wrap/stack/group (flex-wrap + gap, or a grouped action menu)
so nothing clips at any viewport, including 390px. Pure CSS/layout.

## REQ-004 — datetime picker

Use a keyboard/mobile-accessible picker with explicit **Asia/Singapore (Singapore time)** display and validation. A shared
control does not imply a shared wire serializer. Preserve each existing endpoint contract:

| Operation | Wire contract | Example for 5 June 2026, 14:00 Singapore time |
|---|---|---|
| Admin schedule: submit operation for admin interviews id schedule | scheduled At is Singapore time ISO local datetime, no offset or Z, for Local Date Time | 2026-06-05 T14:00:00 |
| Tutee propose / either party counter / decline with counter-slots | slots contains Coordinated Universal Time instants with Z, per INT-039 §4.3 | 2026-06-05 T06:00:00 Z |

Convert an accepted Coordinated Universal Time slot into Singapore time for admin prefill, then serialize using the admin local contract. Convert
proposer Singapore time selections to Coordinated Universal Time instants; never append Z to a local value without conversion. Results must be
independent of the browser's local timezone. A native datetime-local value is only an input/display value.
Keep proposal validation: 1–3 slots, future within seven days, 15-minute grid, no duplicates. This picker change
does not change either data transfer object contract or the existing slot-validation rules.

**Blocking contract checks for a later implementation:** exact admin local string; exact Coordinated Universal Time proposer string;
accepted Coordinated Universal Time→Singapore time prefill; equivalent results in Singapore time and a non-Singapore time browser; past/out-of-window/off-grid feedback.
Source: [INT-039 proposal data transfer object and validation](../architecture/int-039-interview-negotiation.md).

## REQ-005 — display name not handle

**Data already present on the admin response — NO backend task for the admin screen.**
Admin Interview Response implementation reference already carries tutor Display Name + tutor Public Handle. The admin
Interview/Applicants raw-handle display is purely a frontend choice (it renders the handle instead of the name). frontend:
render tutor Display Name on the admin Interview + Applicants screens; fall back to a friendly label, never the raw
tutor slug. The v1.0 "add if missing" hedge for the admin payload is removed (the field exists).

> Scope note: this REQ targets the **admin** Interview/Applicants screens, where Admin Interview Response already
> supplies the name. The tutee-facing Interview Response counterpart-display is OUT of this REQ's scope (it is the
> counterparty-personal information surface, governed separately) — REQ-005 does not touch it, so no tutee-facing backend field work is
> in scope here either.

## Test / coverage

- **No backend change in this spec** — REQ-002 (agreed Timeslot Id+timeslots) and REQ-005 (tutor Display Name) both
  read fields that already exist on Admin Interview Response. No backend gate impact unless a future change adds a field.
- frontend gate ≥97.5/88 — **enum-driven** tab-union contract test (compute the union from the live Interview Status
  enum/typed constant and assert SET-EQUALITY with the union of the tab maps — NOT a frozen snapshot list; this
  catches the EXPIRED gap and any future status); PENDING SCHEDULING visibility in the action tab; schedule
  pre-fill from the agreed slot; button-layout; datetime guard; display-name render.
- Live: accepted interview in queue; pre-filled schedule; all buttons visible at 390px; usable picker;
  display-name on Interview/Applicants.
