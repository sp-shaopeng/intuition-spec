# Tutor (supply-side) re-test — deep logged-in flows

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

**Reviewer:** external quality assurance (Playwright MCP, local stack frontend  / application programming interface )
**Date:** 2026-06-22
**Account:** [REDACTED EMAIL] (Mr Tan Wei Jie, role TUTOR) — fresh login, identity confirmed via /auth/me (id 7ef74ed7…, role TUTOR). No stale-session contamination.

## 1. Deep flows reached + result

| # | Flow | Result | Notes |
|---|------|--------|-------|
| 1 | **Live INTERVIEW negotiation** (/tutor/interviews) | **WORKED** | Interview APPEARS in list (T-08 confirmed FIXED). Detail page shows full INT-039 user interface: 3 timeslots w/ Accept, "Can't make these" (decline), "Suggest other times" (counter), negotiation timeline (Round 1 · proposed by Tutee), "Decline & cancel". Status "Awaiting tutor" + "It's your turn". Walked the counter-propose dialog (quick-pick Today/Tomorrow/Weekend × 4/5/7pm + custom time + add-to-list + Send disabled until ≥1 time) — works. Then **accepted** Wed 24 Jun slot → state transitioned cleanly: badge → "Time agreed", banner "Time agreed — our team will confirm the meeting link shortly", timeline marked Wed is Agreed / others is Not chosen, all action buttons removed. |
| 2 | **Tutee→tutor INVITATION** (/tutor/invitations) | **WORKED** | 1 PENDING invite (Science PSLE · P5, Central, S$30–45/hr, personal message, Accept/Decline). **Accepted** → row left pending list; "Show all history" shows it as **Accepted** + "Already applied" badge + "You accepted this invitation." + "View in My Applications →". Confirmed it FOLDED into the funnel: new "Applied" row appears at top of My Pipeline (/tutor/applications) — INT-044 verified end-to-end. |
| 3 | **Match dashboard** (/tutor/matches) | **WORKED** | 2 matches render correctly. COMPLETED (PSLE Math intensive, Pasir Ris) → "Completed" + "Leave a review →" + Reveal contact. ACTIVE (P6 Math, Tampines) → "Live" + S$240/mo + "Commission: S$120" + commission block + reveal. Filter tabs All/Active/Completed present. |
| 4 | **Post-match CONTACT REVEAL** | **WORKED** | On the Live match, "Reveal contact details" revealed parent: Name **Mrs Lim**, Phone **[REDACTED PHONE]** (tel:), Email **[REDACTED EMAIL]** (mailto:), header "Contact (shared now you've committed)". read operation for matches id contact via user interface works; only shown after commit. |
| 5 | **Reviews received** | **WORKED (both surfaces)** | Profile **Reviews tab**: "Reviews About Me" → Mr Pillai 5★ with full text. **Public tutor detail** (/tutors/315e7b3b-…): header "5.0 (1 review)" + Reviews section with the same 5★ Mr Pillai review. Public list application programming interface reports average Rating, review Count. |
| 6 | **Commission (tutor view)** | **WORKED** | On the Live match the tutor sees "Commission: S$120", badge "Commission pending", and clear copy: "We'll contact you to collect the one-time placement fee after your first session — nothing to do yet." + "The fee applies only once lessons begin — if the arrangement ends early, email [REDACTED EMAIL]…". Honest, no payment action demanded of the tutor. (Admin pays in a later pass — not exercised, as instructed.) |

**All 6 deep flows reached and functional.** T-08 (interview not appearing) is confirmed fixed.

## 2. Issues

### ISSUE-T-RR-01 · Notification bell polls /notifications/unread-count while logged OUT → 401 storm · **S4 (low)**
- **Repro:** Open any public/login page (not authenticated). The notification poller fires read operation for notifications unread-count on its interval with no session.
- **Expected:** No authenticated polling on unauthenticated routes (poller should be gated on auth state).
- **Saw:** ~190 console 401 errors on /notifications/unread-count accumulated over the pre-login window (login page sat for a short time). Once logged in the endpoint returns a successful response reporting nine results correctly — so this is wasted requests + console noise on unauthenticated pages, not a functional break.
- **Evidence:** evidence/ console log captured at 401-flood; live re-check returned 200.
- **Impact:** Cosmetic/perf hygiene. Could mask real 401s and adds needless load. Worth gating the poll on is Authenticated.

### Observations (not defects)
- personal information discipline is consistently correct on the supply side: interviews show "Parent · Bedok" (not "Hidayah"); invitation shows the message but not "Bee Hoon Tay"; contact only revealed post-commit. Good.
- T-05 (WhatsApp toast while OFF): **not observed** on any tutor screen exercised (interviews accept, invite accept, contact reveal) — no spurious WhatsApp toast fired. WhatsApp stays dormant as expected.
- T-06 (persistent Apply button): not in scope of these screens; not observed.
- T-04 (nav breakpoint): at 390px the bottom tab nav (Find Work / My Pipeline / Interviews / Matches) renders correctly; matches + interviews pages stack cleanly with no overflow.

## 3. 390px re-confirmation
- /tutor/matches @390px — clean: cards stack, commission block + contact reveal usable, bottom nav present (tutor-rerun-10-matches-390px.png).
- /tutor/interviews @390px — clean (tutor-rerun-11-interviews-390px.png).

## 4. Still blocked / not exercised
- **Admin commission payment** — intentionally not exercised (different role; admin pays in a later pass). Tutor-side view confirmed.
- **Counter-propose / decline-with-reason send** — user interface walked and verified enabled, but NOT submitted (would consume the seeded PROPOSED interview before testing Accept). Accept path was chosen as the positive transition to verify. Counter/decline send-paths remain unconfirmed end-to-end (user interface present and correct up to the Send button).

## Evidence
- evidence/tutor-rerun-01-interviews-list png — interview appears in list (T-08 fixed)
- evidence/tutor-rerun-02-interview-detail png — INT-039 negotiation user interface
- evidence/tutor-rerun-03-interview-time-agreed png — accept → "Time agreed"
- evidence/tutor-rerun-04-invitation-pending png — pending invite
- evidence/tutor-rerun-05-pipeline-after-accept png — invite folded into pipeline
- evidence/tutor-rerun-06-matches png — both matches
- evidence/tutor-rerun-07-contact-reveal png — Mrs Lim contact revealed
- evidence/tutor-rerun-08-reviews-tab png — profile Reviews tab (Mr Pillai 5★)
- evidence/tutor-rerun-09-public-detail-review png — public detail review
- evidence/tutor-rerun-10-matches-390px.png / 11-interviews-390px.png — 390px
