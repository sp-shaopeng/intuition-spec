# Wave 1 — Scoped SDD Spec-Review (pre-build gate)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> Mode: team-review **Spec Review mode** (DRAFT specs, before status: APPROVED) → Boss sign-off per spec.
> Reviewers (voiced): frontend-expert (Sam), bff-specialist (Riley), software-engineer (Jordan), dba (Morgan),
> uiux-designer (Jamie), security-expert (Casey), qa-engineer (Reese), tech-lead (the technical lead), Boss (Rex).
> Every load-bearing claim below was re-grounded against the real source (file:line cited). This is a gate, not a rubber-stamp.
> Date: 2026-06-22. Scope: Wave 1 slices only (later REQs noted, not gated).

---

## Ground-truth verification (the load-bearing claims I re-opened)

| Claim under review | Verified against source | Verdict |
|---|---|---|
| 012 R-001: frontend↔backend apply-completeness mismatch | backend Job Application Service implementation reference checks location Planning Areas (257), no qualifications length floor; frontend Tutor Apply Control implementation reference checks location Region (49) + the trim operation (51) | ✅ REAL — verbatim mismatch confirmed |
| 012 R-001: 422 already handled, not silent | Tutor Apply Control implementation reference has a TUTOR PROFILE INCOMPLETE branch + toast | ✅ confirmed (the silent path is the *role 403*, still unproven) |
| 014 R-001: frontend tab-map orphans negotiation states | Interview Queue implementation reference — ACTION REQUIRED: REQUESTED only; SCHEDULED, HISTORY as described | ✅ REAL |
| 014 R-001: enum has 10 values | Interview Status implementation reference — incl. PROPOSED, COUNTER PROPOSED, TUTOR DECLINED, PENDING SCHEDULING, **EXPIRED** | ⚠️ design's proposed union OMITS EXPIRED (see SEND-BACK) |
| 014 R-002: accepted slot on payload | Admin Interview Response implementation reference already carries agreed Timeslot Id + timeslots | ✅ data already exists — spec should say so |
| 014 R-005: display name on payload | Admin Interview Response implementation reference carries tutor Display Name + tutor Public Handle | ✅ confirmed |
| 013 R-003: no tutee sent-invitations GET | Tutee Invitation Controller implementation reference has POST invite / GET invite-quota / DELETE withdraw — **no list GET** | ✅ REAL gap |
| 017 R-003: Mark-Paid has no confirm | Commission implementation reference the mutate operation direct; cancel uses cancel Confirm Id + dialog (325-348) | ✅ REAL asymmetry |
| 018 R-002: seeder omits tutor User | Dev Data Seeder save Interview sets tutor Post/tutee/tutee Post/status — **never set Tutor User** | ✅ REAL |

---

## Spec 013 — Tutee dashboard + invitations (#207.1, #207.2, #208, sent-invitations)

**Verdict: SEND-BACK** (Wave-1 slice cannot reach APPROVED until the conditions below are closed; the diagnosis is sound, the contract is underspecified.)

Strengths: REQ-001/002 are accurately scoped frontend-only (backend PUT/GET exist); REQ-003's "no tutee GET for sent invitations" is verified true; soft-delete-awareness and own-only authz are called out.

**SEND-BACK conditions:**
1. **(Riley/Morgan — REQ-003 contract) Define the response shape explicitly.** The design says "personal information rules" but never specifies the Job Invitation Response fields the tutee-side list returns. A reused Job Invitation Response may carry tutor-side fields the tutee shouldn't see. Enumerate the exact fields (tutor display identity, target job, status, timestamp) and confirm none leak. There is already a Job Invitation Response data transfer object (used by the tutor-side read operation for tutor invitations) — state whether it's reused as-is or a tutee-scoped projection is needed.
2. **(Riley/Jordan — REQ-003 query owner field) Resolve the query key.** Design says query by created By caller (Job Invitation implementation reference). Confirm Job Invitation actually has a created By/inviter column that equals the tutee user id (not the tutor) and that an index exists for the own-only scan; if absent, this becomes a DBA task, not "frontend-only + one endpoint." Add the route constant to api Routes implementation reference (named, no magic string).
3. **(Jamie/Sam — REQ-001 editable-statuses) The open question is load-bearing and must be closed before build, not deferred.** "Gate Edit on the statuses the backend PUT accepts" with the actual list TBD means the frontend can't be built deterministically. Task 3 (bff confirms the editable-status list) must complete and the list recorded in design md before REQ-001 frontend work starts. Without it the "non-editable → reason surfaced" Gherkin is untestable.
4. **(Jamie — REQ-003/004 dashboard IA) The "where does the outbound request surface" question is unresolved.** The open question (#2) admits "one unified 'My outbound requests' area vs separate sections — uiux-designer to decide." This is exactly the #208 root issue (created → not visible). A spec that defers when the surface lives is underspecified — the IA decision (single outbound area vs three sections, and which dashboard region) must be in design md before APPROVED, or REQ-004 has no acceptance target.

**BLOCKING tests this spec must carry:** (a) backend: own-only authz — tutee A's token never returns tutee B's invitations (the REQ-003 guard scenario); (b) backend: soft-delete/withdrawn excluded by default, included with ?status is ALL; (c) frontend RTL: "Invited" state is server-backed and survives a refresh (replaces session-only invited Post Ids, Tutor Detail implementation reference); (d) live Playwright: invite → refresh → still shown.

**Boss verdict — 013:** ⚠️ SEND BACK. Diagnosis verified and correct; the **read-endpoint contract + the dashboard IA placement** are the two gaps that make REQ-003/REQ-004 unbuildable as written. Close conditions 1–4, then re-sign.

---

## Spec 012 — Tutor apply feedback + profile user experience (Wave-1 = REQ-001 only)

**Verdict: APPROVED for REQ-001 with one binding pre-build decision** (the single-source-of-truth choice). REQ-002/003/004 are out of Wave 1 — not gated here.

Strengths: REQ-001 is the strongest-grounded slice in the wave. The frontend↔backend mismatch is real and verified verbatim (location Region vs location Planning Areas; the is at least 10 floor backend lacks). The two-part fix (single source + never-silent) is correct framing.

**Binding condition before build (resolve at design, then APPROVED stands):**
1. **(Riley — single source of truth) Pick the backend-signal route, not the aligned-constant fallback.** Cross-cutting recommendation (below) applies here squarely: have the **backend expose profile Complete: boolean + missing Fields: string** on the GET tutor-profile response, and the frontend consume it directly — delete the frontend is Profile Complete predicate rather than re-aligning it. The "align the constant" fallback re-creates the exact drift that caused this bug and will rot again the next time the backend gate changes. The spec lists this as "preferred" but leaves it open; the gate's call: **the payload-signal is the requirement, the aligned-constant is rejected unless Riley shows a concrete blocker.**
2. **(Jordan/Casey — the 403 path) The bare 403 Access denied is still a hypothesis, not a root cause.** Spec design §403-vs-422 says "confirm at build whether…". That's acceptable as a build task ONLY if REQ-001's acceptance explicitly requires the 403 reproduction to be identified and handled (it does — the "403 never silent" Gherkin). Keep it; do not let the build close REQ-001 on the 422 path alone while the actual 403 the reviewer saw stays unexplained.

**BLOCKING tests REQ-001 must carry:** (a) the **frontend-verdict  equals  backend-verdict parity test over a field matrix** — this is the non-negotiable one; if the backend returns the signal, the test asserts the frontend renders the gate iff profile Complete equals no; (b) RTL: incomplete → banner (not an enabled CTA); 422 → toast+banner; **403 → explicit message, never an unhandled "Access denied"**; (c) live Playwright: incomplete-profile apply shows the banner + a forced 422 shows the toast.

**Boss verdict — 012 (REQ-001):** ✅ SHIP the spec **conditioned on** the backend-signal decision being recorded in design md before build (not deferred to the implementer). The parity test is the gate that proves the fix; it must exist. REQ-004 stays correctly blocked on the founder editability decision.

---

## Spec 014 — Interview scheduling user experience + admin visibility (Wave-1 = REQ-001 only)

**Verdict: SEND-BACK** (REQ-001 has a correctness bug in its own design — the proposed tab union does not cover the enum it claims to cover.)

Strengths: the frontend-not-backend root-cause is verified correct (list Admin Interviews is find All-correct; the bug is the frontend status Params map at 52-57). The "drive from the enum so a new status can't orphan" instinct is exactly right.

**SEND-BACK conditions:**
1. **(Sam/Reese — REQ-001 union is incomplete) The design's proposed union omits EXPIRED.** Design md proposes action is REQUESTED,PROPOSED,COUNTER PROPOSED,PENDING SCHEDULING,TUTOR DECLINED, scheduled is SCHEDULED, history is COMPLETED,CANCELLED,NO SHOW. The enum (Interview Status implementation reference) ALSO has **EXPIRED** (terminal). That union is **9 of 10** — EXPIRED is orphaned outside ALL, the *exact* defect this REQ exists to kill. Decide where EXPIRED belongs (history is the natural home — it's terminal) and fix the design. The acceptance scenario "union of all tab status-filters covers every Interview Status value" would FAIL against the current proposed map; that is a SEND-BACK by the spec's own test.
2. **(Reese — the union equals enum test) Make the test enforce real coverage, not a frozen snapshot.** The "union  equals  enum" test must compute the union from the live Interview Status enum and assert set-equality, so adding a future status (or the already-present EXPIRED) fails the test until it's placed. Specify it as a contract test over the enum, not an assertion against a hardcoded list.

**Non-blocking tightening (carry into design, not a SEND-BACK on its own):** REQ-002 says "confirm whether Admin Interview Response exposes the accepted slot; if not, add it." It already does — agreed Timeslot Id + timeslots (Admin Interview Response implementation reference). Update design md to "data already present; frontend-only pre-fill" so no phantom backend task is dispatched.

**BLOCKING tests REQ-001 must carry:** (a) the enum-driven **union  equals  every Interview Status value** contract test (will catch the EXPIRED gap); (b) RTL: a PENDING SCHEDULING interview appears in the action tab; (c) live: an accepted-but-unscheduled interview is visible in the queue action tab.

**Boss verdict — 014 (REQ-001):** ⚠️ SEND BACK. The root cause is right but the proposed fix reintroduces the same class of bug (EXPIRED orphaned). Fix the union + make the test enum-driven, then re-sign. This is precisely why the gate reads the enum, not the prose.

---

## Spec 015 — Mobile/responsive fixes (Wave-1 = REQ-001 only)

**Verdict: APPROVED for REQ-001.** REQ-002/003 are Wave 2 — not gated here.

Strengths: the diagnosis is textbook and correct — Availability Grid implementation reference has overflow-x-auto around a min Width table, but a flex/grid ancestor with default min-width:auto won't shrink, so the page scrolls instead of the grid (classic flexbox min-content overflow; min-w-0 on the shrinkable track is the right fix). The "e2e missed it because it only checks Step-1" insight is the real lesson.

**Conditions (tighten before build, not blockers):**
1. **(Sam — name the ancestor) The fix target is "the form layout column wrapping the grid" — identify the exact file:line.** "Add min-w-0 to the shrinkable track using s between the grid and the page container" is correct in principle but the implementer needs the specific ancestor element. Have Sam walk the DOM from Availability Grid up to the page container in the create-post wizard and record the exact element using s to patch, so the build is deterministic and the change is minimal (over-applying min-w-0 can break unrelated layouts).
2. **(Reese — the e2e guard is the most valuable artifact here) Make it a real gate, not a one-off.** The 390px body scroll Width is at most client Width assertion on the availability step is the BLOCKING test and it must advance the wizard **to the step that renders the grid** (the e2e gap was that it only checked Step-1). Specify which step index renders the availability grid and assert no page overflow *there*.

**BLOCKING tests REQ-001 must carry:** (a) the 390px Playwright e2e asserting body scroll Width is at most body client Width **on the availability-grid wizard step** (advancing past Step-1), with the grid itself still scrolling internally; (b) structural/RTL test that the min-w-0 class is on the intended ancestor; (c) regression: ≥sm the grid fits without the scroll affordance and the encoded bitmask is unchanged.

**Boss verdict — 015 (REQ-001):** ✅ SHIP the spec. Condition: the exact ancestor element is recorded in design md and the e2e advances to the grid step. This slice is clean; the only risk is an over-broad min-w-0 — pin it.

---

## Spec 017 — Admin analytics + financial safety (Wave-1 = REQ-003 only)

**Verdict: APPROVED for REQ-003** (Mark-Paid confirm). REQ-001/002 (analytics calc) are Wave 2 — not gated here, and correctly flagged as needing a business-analyst/data-engineer cohort decision first.

Strengths: REQ-003 is verified and clean — the asymmetry is real (Commission implementation reference fires the pay mutation directly; cancel at 325-348 already confirms). Marking money PAID with no confirm while a lesser action confirms is a legitimate financial-safety defect. The design reuses the existing confirm pattern (good — consistency + low risk) and retains the idempotency key.

**Conditions (tighten, not blockers):**
1. **(Jamie/Casey — REQ-003 dialog content) Spec the dialog copy precisely.** "naming the match/amount" — confirm the dialog shows the **commission amount and the match/tutor identity** so the admin can verify *which* commission before an irreversible action. There are three Mark-Paid call sites (199, 246, 312) — the confirm must gate **all three**, not only the first. Make that explicit in the task (today the task cites 199 only).
2. **(Reese — guard test) The cancel-path test is the safety proof.** Acceptance already has it ("cancel → no request sent"); keep it BLOCKING.

**BLOCKING tests REQ-003 must carry:** (a) RTL: clicking Mark-Paid opens the dialog and the pay mutation fires ONLY after explicit confirm — across **all three** call sites; (b) RTL: cancelling sends no request and the commission stays unpaid; (c) live capture of the dialog before a pay.

**Boss verdict — 017 (REQ-003):** ✅ SHIP the spec. Condition: the task covers all three Mark-Paid call sites (not just 199) and the dialog names amount + match. Money action with a confirm gate and a cancel-no-op test — this is the bar.

---

## Spec 018 — Flag hygiene, seeder, frontend hygiene (Wave-1 = REQ-002 + demo-data REQ-003)

**Verdict: APPROVED for REQ-002 (seeder tutor user id); REQ-003 (demo-data variety) APPROVED with a measurable acceptance condition.**

Strengths: REQ-002 is verified exactly — Dev Data Seeder save Interview never calls set Tutor User, so find By Tutor User Id returns empty for seeded rows; live create Interview sets it (268), so only seed data is affected. The fix (the set tutor user operation) is one line, dev/test-data only, no schema, low risk — correct scoping. This is the root cause of the T-08 "views disagree" symptom.

**Conditions:**
1. **(Jordan — REQ-002 test) The seeder test must assert the invariant, not just run.** Dev Data Seeder Test must assert every seeded interview has a non-null tutor user id (and ideally that it equals the tutor-post's tutor), so the invariant can't silently regress again. The live create Interview invariant and the seed must be the same — note it as the shared invariant.
2. **(Reese — REQ-003 is currently untestable as written) "no dominant duplicate" / "looks realistic" needs a measurable bar.** The acceptance "no single subject·level·budget combination dominates" has no threshold. Define a concrete, assertable rule (e g. "no single subject·level·budget·location tuple is >X% of seeded showcase jobs" or "≥N distinct subjects and ≥M distinct levels appear in the public strips"). Without a number this REQ can't pass a gate and isn't a real acceptance criterion. REQ-003 stays Could-priority — but if it ships in Wave 1 it needs the threshold; otherwise defer it.

**BLOCKING tests:** (REQ-002) Dev Data Seeder Test asserting non-null tutor user id on seeded interviews + a live reseed showing tutor /tutor/interviews non-empty; (REQ-003, if shipped) a seeder/inspection assertion against the defined variety threshold.

**Boss verdict — 018 (REQ-002/003):** ✅ SHIP REQ-002 (clean, verified, low-risk, one-line + test). REQ-003: ✅ conditioned on a measurable variety threshold — an unmeasurable "looks realistic" is not an acceptance criterion. The other 018 REQs (R-001 flag toast, R-004 /dashboard, R-005 copy, R-006 CTA) are not Wave 1 — not gated here.

---

## Cross-cutting risks (apply across the wave)

1. **Single-source-of-truth for the apply gate (012 R-001) — make the backend the authority, not two predicates.**
   This is the highest-leverage cross-cutting call. The bug exists *because* the frontend re-implements a completeness
   predicate that drifted from the backend's. The fix must **eliminate the second predicate**, not re-sync it: backend returns
   profile Complete + missing Fields on the tutor-profile GET; frontend consumes the boolean and renders missing Fields
   in the banner. The "align the frontend constant" fallback is rejected at this gate — it guarantees the bug returns. The
   same principle recurs in 013 (don't re-derive invitation state client-side — invited Post Ids session state is the
   same anti-pattern; the server list is the source of truth) and 014 (drive the tab map from the Interview Status
   enum, not a hardcoded copy of it). **One rule for the wave: client state mirrors server truth; it never re-derives it.**

2. **The #208 "created → not visible" family (013 R-003/R-004) is fundamentally a create-then-list-visibility design,
   and the spec defers the "where" — that's the actual defect.** A read endpoint alone does not fix #208 unless the
   dashboard *surfaces* it in a defined place. The IA placement (single "My outbound requests" area covering
   applications-sent + invitations-sent + interview-requests, vs three separate sections) MUST be decided in design
   before APPROVED. A user who invites a tutor and sees nothing on their dashboard is the bug; an endpoint nobody
   renders doesn't close it.

3. **"Union  equals  enum" must be enum-driven everywhere it's claimed (014).** The 014 design proved the danger itself by
   omitting EXPIRED from a union it called complete. Any spec that claims exhaustive coverage of an enum must back
   it with a contract test that reads the enum at test time. Hardcoded "we listed them all" lists rot.

4. **Phantom backend tasks from un-verified "add if absent" hedges.** 014 R-002 ("confirm the accepted slot is on the
   payload; add if absent") and R-005 (display name) both reference fields that ALREADY exist
   (Admin Interview Response implementation reference). Specs should resolve these at authoring time (I did — they exist) so
   the Boss doesn't dispatch a backend agent for a no-op. Cheap to verify, expensive to leave ambiguous.

5. **Money + irreversibility (017 R-003) — confirm-gate ALL call sites.** There are three Mark-Paid invocations; a
   confirm on one is a false sense of safety. The financial-safety REQ is only met when every path to the pay
   mutation is gated.

---

## Summary table — per-spec Wave-1 verdict

| Spec | Wave-1 slice | Verdict | Must-fix before APPROVED |
|---|---|---|---|
| 013 | #207.1/.2, #208, sent-invites | ⚠️ **SEND-BACK** | Read-endpoint response contract + own-field query/index; editable-status list resolved (not deferred); dashboard IA placement for outbound requests decided |
| 012 | REQ-001 (apply-gate parity) | ✅ **APPROVED** w/ condition | Record the backend-signal (profile Complete/missing Fields) decision in design (reject aligned-constant); parity test + 403-handling test are blocking |
| 014 | REQ-001 (admin tab-map) | ⚠️ **SEND-BACK** | Proposed union omits EXPIRED — place it; make union equals enum an enum-driven contract test |
| 015 | REQ-001 (390px grid overflow) | ✅ **APPROVED** w/ condition | Name the exact min-w-0 ancestor file:line; e2e must advance to the grid step |
| 017 | REQ-003 (Mark-Paid confirm) | ✅ **APPROVED** w/ condition | Gate all three Mark-Paid call sites; dialog names amount + match; cancel-no-op test blocking |
| 018 | REQ-002 (+demo-data R-003) | ✅ **APPROVED** w/ condition | R-002 test asserts non-null tutor user id; R-003 needs a measurable variety threshold or defer |

**Two SEND-BACKs (013, 014), four conditional APPROVEs (012, 015, 017, 018).** No spec is rubber-stamped: 012/015/017/018 ship on named, testable conditions recorded in design md; 013/014 return to the author with specific gaps. The blocking tests are listed per spec above and are non-negotiable for the build phase.

---

_Reviewed by the scoped Wave-1 panel; Boss (Rex) final sign-off recorded per spec above. Author self-approval remains a process failure — these verdicts are the gate, not the author's say-so._
