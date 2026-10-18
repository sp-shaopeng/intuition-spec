# Consolidated External-Reviewer Findings — 2026-06-22 (de-noised + Boss-verified)

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

3 external reviewers (tutee / tutor / admin) tested LOCAL in parallel. **Caveat: they shared ONE Playwright-MCP
browser → cross-session collisions.** The Boss de-noised collision artifacts and verified the sharp claims live.
This is the input to the fix spec (founder decision 2026-06-22: **fix EVERYTHING S2+S3+S4+T-02**; **seed + sequential
re-run FIRST** to close the deep-flow coverage gap before finalizing).

## Boss verification (verify-don't-trust)
- **Authz intact** — tutor token → [REDACTED CREDENTIAL] & /admin/analytics = **403**. The "tutor hijack admin"
  screenshot is a shared-browser collision, NOT an authz hole. ❌ not a defect.
- **Deep-link/refresh auth holds** — e2e render-smoke does the goto operation + asserts
  "bounced to /login (auth lost?)" → **4/4 PASS**. The admin "S1 refresh-logout" is **refuted** as a universal bug
  (cookie is [REDACTED CREDENTIAL], same-site localhost → sent correctly; prod is single-origin anyway). ❌ not the S1 it looked like.

## Real defects (to fix — founder: ALL)

| ID | Sev | Area | Finding | Owner |
|----|-----|------|---------|-------|
| INT-210 | S2 | the private implementation folder | Availability grid overflows the PAGE at ≤390px (Availability Grid implementation reference wrapper not width-capped; overflow-x-auto never engages — missing min-w-0 on a flex/grid ancestor). Regression vs #206. e2e missed it (only checks Step-1). | frontend-expert |
| T-07 | S2 | frontend+backend | Tutor Apply with incomplete profile → bare 403 Access denied, **zero** frontend feedback (silent fail on the core money action). Gate is correct; need typed PROFILE INCOMPLETE + inline guidance / disabled-Apply-with-reason. | frontend-expert + bff-specialist |
| INT-211 | S3 | the private implementation folder | Error toast shows raw Tutee post not found: `<uuid>` (entity + id leak). Humanise; keep trace Id Reference. | frontend-expert + software-engineer |
| T-01 | S3 | frontend-routing | /dashboard (no role prefix) → hard 404. Redirect to role dashboard or login. | frontend-expert |
| T-04 | S3 | the private implementation folder | Desktop nav collapses to hamburger on /assignments + /tutor/* (links only in bottom mobile nav); /tutor/dashboard shows full nav — inconsistent breakpoint. | frontend-expert |
| T-08 | S3 | frontend+the private implementation folder | "Interview scheduled" badge + "View interview →" in My Pipeline, but /tutor/interviews + read operation for tutor interviews empty. Views disagree. (Root cause TBD by seeding engineer.) | software-engineer + frontend-expert |
| ADM-nav | S3 | the private implementation folder | Admin 8-section nav is a cramped ~154px scroll strip at 390px, no hamburger. | frontend-expert |
| CTA-intent | S3 | the private implementation folder experience | Generic /register + "Post a request" CTAs bounce to /login. | uiux-designer + frontend-expert |
| T-02 | S3* | frontend-auth | frontend "Log Out" races a silent token-refresh → a surviving refresh token can resurrect a session. Backend revocation is correct (logout w/ XSRF revokes). *Cross-user "hijack" needed the shared browser; single-user impact = "logout may not stick". Fix: logout reliably revokes + halts the silent-refresh loop before navigation. | software-engineer + security-expert |
| autofill | S3/S4 | frontend-auth | Auth forms don't resist stale browser autofill (saved creds override typed; auto-submitted a forced password-change). autocomplete handling. | frontend-expert |
| T-05 | S4 | the private implementation folder | Profile save fires a WhatsApp "Sending verification code…" toast while WhatsApp is OFF (flag not gating the frontend toast). | frontend-expert |
| T-03 | S4 | frontend-copy | Step-2 "All optional" header but Subjects & levels is *required — copy/validation mismatch. | frontend-expert |
| T-06 | S4 | frontend | Apply-rail bottom "Apply →" persists after "✓ Interest sent". | frontend-expert |
| ADM-400 | S4 | the private implementation folder | create-admin 400 doesn't name the missing field. | bff-specialist + frontend-expert |
| demo-data | S4 | data | Landing//assignments dominated by 8+ identical "Primary 5 · Math (PSLE) · S$30-60" cards — undercuts "real jobs now". Seeder variety, not code. | dba/data |

## De-escalated / not defects
- Admin "S1 refresh-logout" → refuted (see verification). Tutee /dashboard 404 → it's the unprefixed /dashboard
  route (T-01), real tutee login lands on /tutee/dashboard correctly. "tutor hijack admin" → collision artifact.

## Coverage gap → seed + SEQUENTIAL re-run (founder choice)
UNTESTED (no seed data + shared browser): interview accept/propose/counter/decline (INT-039), tutee→tutor invitation
accept/decline (INT-044), ACTIVE match dashboards + **contact reveal**, **commission payment → completion**,
reviews-received + public tutor detail. → Engineer seeds these; reviewers re-run **one at a time** (no shared browser).

## Verified WORKING (regression-positive, don't touch)
Registration + Personal Data Protection Act consent copy; create-post wizard taxonomy (54 subj/28 lvl incl. N-Level) + 422 UNKNOWN SUBJECT OR LEVEL
(no 500) + "6 or more"; apply/withdraw + 10-quota; profile/listing builders + multi-MRT + time×day availability grid
(sticky day col, swipe hint) — mobile-usable at 390px; Rate Guide + clear/fair 50%-first-month commission; admin
approve/reject-with-reason-guard + analytics + date-range + inverted-range guard + self/super-admin protection (403);
POST SUBMITTED fan-out; trace Id Reference on errors; personal information-free public cards; consistent privacy/trust messaging.

## Founder-filed GitHub issues #207–211 (pulled 2026-06-22) — fold into the same fix wave
Filed 2026-06-21 alongside #202–206 (which are HISTORICALLY MARKED IMPLEMENTED via 008/009 but still OPEN on GitHub pending close). These
overlap the reviewed flows and ADD scope. **Verified: NOT a single root cause** — live Interview Service create Interview
sets tutor User (line 268); only 3/40 interviews are NULL tutor user id (old seeded rows = T-08 seeder defect); the
admin list filters by **status** (find By Status In), not tutor user id. So #208 / #210.1 / T-08 are distinct.

| GH# | Item | Sev (prelim) | Notes / root-cause to confirm at spec |
|----|------|------|------|
| 207.1 | Tutee dashboard: no **edit / view post** action | S2 | NEW — can't edit a post after creating; verify if an edit route exists at all |
| 207.2 | Tutee dashboard shows only 2 venues though all selected | S3 | NEW — venue list truncation/display (or persistence) bug |
| 208 | From tutor public page, **invite-to-job** + **request-interview** don't appear in active requests | S2 | NEW — create succeeds but list-visibility gap (tutee side); confirm the create path + which list it should appear in |
| 209.1 | **Tutor profile subject dropdown** needs the expanded taxonomy (same as tutee #203) | S3 | verify whether tutor profile reuses the shared SUBJECT OPTIONS (009) — may already be fixed; confirm live |
| 209.2 | Registration **duplicate-phone** → generic "couldn't complete registration" | S3 | corroborates error-copy group — need a specific "phone already in use" message (don't leak enumeration: align with security) |
| 209.3 | Tutor profile **phone/email not editable** | S3 | user experience/decision — is personal information-immutable intended? if so, explain; else allow edit (re-verify path) |
| 210.1 | **Admin interview list** doesn't show an interview created→accepted live (but notification fires) | S2 | REAL — list Admin Interviews status-filter gap (admin saw the seeded PENDING SCHEDULING one; the founder's state isn't in the filter). Confirm which statuses the admin list includes |
| 210.2 | Scheduling should **auto-fill the accepted timeslot** | S3 | enhancement; ties to #211 |
| 210.3 | "Record No Show" button **clipped / not visible** next to Mark Complete | S3 | user interface overflow/clipping |
| 211 | Interview **date/time picker** unpleasant to use (all roles) | S3 | user experience of the schedule datetime control |
| (202–206) | shipped via 008/009 | — | **close on GitHub after founder verifies** (still OPEN) |

## Sequential re-run findings (deep flows — all core flows WORKED; seeded + one-at-a-time, no collision)
**TUTOR re-run:** all 6 deep flows WORKED (interview negotiate+accept INT-039, invitation accept INT-044, match dashboards,
contact reveal, 5★ reviews-received on profile+public, commission view). +**S4** notification bell polls
/notifications/unread-count while logged OUT → ~190× 401 (gate the poll on auth).
**ADMIN re-run:** all WORKED — commission **pay→match COMPLETED** (S$120 = 50% of S$240 first-month, math correct),
interview scheduling (Spec 006), activate/cancel→job-reopen, choose-tutor→assign-winner, approve/reject, analytics
date-range. +**S3** interview-funnel "Conversion 400%" (cohort mismatch), +**S3** "Mark Paid" has **no confirm dialog**
(highest-consequence financial action — every other mutation confirms), +**S3** admin 8-section nav cramped at 390px
(dup of ADM-nav), +**S4** "Avg Commission/Match" divides by active-matches (=grand total, label/calc mismatch),
+**S4** bare read operation for admin commission → 500 not 404/405.
**TUTEE re-run:** match dashboard, contact reveal, **leave-a-review WORKED end-to-end** (shows on public detail),
interview tutee-side WORKED. +**S3** tutor shown as raw handle tutor wei jie tan on Interview/Applicants screens (vs
proper name on Matches — reads like leaked id), +**S3** no tutee-facing **sent-invitations list** (the "Invited" pill is
session-only local state, no GET route — corroborates #208), +**S4** tel link has **unescaped spaces** (dialers fail).
(Invitation-sent view couldn't be demoed live — admin pass had converted the only PENDING invite; re-seed to test.)

**NOTE:** the re-run **mutated the seed** (commission paid, interview scheduled, a match activated-then-cancelled, a
selection assigned, a post approved + one rejected, the invitation accepted). Reset seed before any next live pass.
