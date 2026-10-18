# MOE Teacher Re-Evaluation — Tutor Signup & New Taxonomy

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

**Reviewer persona:** 15-yr MOE teacher / Ho D (core subject), full SG curriculum knowledge.
**Target:** Deployed dev site http://[REDACTED IP ADDRESS] (Playwright MCP).
**Date:** 2026-06-23.
**Account used:** fresh tutor REDACTED email, phone REDACTED phone, subject A-Math (O-Level) Sec 4 (Express).
**Scope covered:** register → onboarding profile → subject/level pickers (profile + listing + browse filter) → commission terms → build a listing → browse + apply to a job → profile-completeness banner → mobile 390px.

---

## Findings (lead with taxonomy + money/trust)

| ID | Sev | Screen / URL | What I saw (as a teacher) | Why it matters | Evidence |
|---|---|---|---|---|---|
| **MOE-T1** | **S2** | Subject→level matrix, all 3 pickers (/register/profile, /tutor/listings/new, /tutor/profile) | Every **Secondary-band subject is given the IDENTICAL full level set** — Sec 1–5 Express + Normal using Acad + Sec 1–4 Normal using Tech — with no per-subject restriction. So **A-Math (O-Level)** is offered at "Secondary 1 (Normal Tech)", and **Combined Humanities (O-Level)** is offered at "Secondary 1 (Normal Tech)". Neither combination exists in the real curriculum: A-Math is an upper-sec (Sec 3–4) Express/NA elective, never N using T; Combined Humanities is a Sec 3–4 O-Level subject, never lower-sec, never N using T. | A parent or admin filtering by these pairs gets nonsense matches; a teacher immediately distrusts a platform that thinks A-Math is a Sec-1 N using T subject. The band scoping (no Primary/JC for a Sec subject) IS correct — it's the *stream×year within Secondary* that is over-permissive. | moe-02, moe-03, moe-06 |
| **MOE-T2** | **S2** | Levels group header, listing wizard /tutor/listings/new (and profile matrix) | The Secondary level group is labelled **"SECONDARY (O-LEVEL)"** but it contains the **Normal using Acad and Normal using Tech** levels (Sec 1–5 NA, Sec 1–4 NT). N using A students sit the **N using A-Level**, N using T students the **N using T-Level** — NOT the O-Level. Calling these "(O-Level)" under one header is factually wrong to any SG educator. | Mislabels the entire Normal stream as O-Level. Combined with MOE-T1 this is the kind of error that signals "built by people who don't know the SG system" — directly undermines tutor/parent trust. | moe-06 |
| **MOE-T3** | **S2** | Browse list /assignments → list-card **Apply** button | Clicking **Apply** on a job card while my profile was incomplete returned submit operation for tutor applications → **422 TUTOR PROFILE INCOMPLETE** but the user interface showed **nothing** — no toast, no inline error, the button still read "Apply". The server message ("Complete your tutor profile before applying to jobs.") was swallowed. | Silent failure. As a tutor I can't tell if I applied, if it failed, or why. The **detail page** (the selected assignment's page) handles the SAME gate correctly (shows "⚠ Finish your profile to apply" + a Complete-profile button) — so it's an inconsistency: fix the list-card button to surface the gate like the detail page does. | moe-08 (422 body captured: TUTOR PROFILE INCOMPLETE) |
| **MOE-T4** | **S2** | /tutor/profile vs apply gate | The profile page reported **"Profile completeness 100%"** with the required **"Area *"** dropdown set (East) — yet apply was still blocked with "Complete your profile to apply: **your teaching areas**", which refers to the **Planning areas** chips that the form itself labels **"(optional)"**. Only after I added one optional chip (Bedok) did the apply button appear. | Three-way contradiction: completeness is 100%, field labelled "optional", but apply hard-requires it. A tutor sees "100% complete" and "you can't apply, finish your profile" at the same time — confusing and trust-eroding. Either make planning-area mandatory (and label it so, and don't show 100%), or drop it from the apply gate. | moe-09 |
| **MOE-T5** | **S3** | /register wizard | Progress bar advertises **4 steps** (Account · Profile · Terms · **First listing**), but "Create account" on the Terms step lands directly on /tutor/dashboard — **Step 4 "First listing" is never shown**. Likewise the listing wizard advertises a Step 4 "Review" that is skipped (Step-3 "Continue" submits straight to "Listing submitted"). | Minor, but the step counter over-promises. A first-timer expects a review screen before a "binding" commission submission and a guided first-listing step; both silently vanish. | moe-05, moe-07 |
| **MOE-T6** | **S3** | /tutor/profile save | Saving with the auto-suggested handle "Tan W." returned **409 PUBLIC HANDLE TAKEN**. The inline handle error ("⚠ That handle's taken. Try another.") is **correct and clear** — good — but **completeness briefly flashed "100%" and a "Setting up verification…" status appeared** even though the save was rejected and nothing persisted. | The momentary "100% / setting up verification" on a *failed* save is misleading. The auto-suggested handle also collides out of the box (derived purely from display name), so the very first save a tutor attempts fails. Consider suffixing the suggestion or checking availability before pre-filling. | moe (handle 409 body captured) |
| **MOE-T7** | **S4** | Subject labels, all pickers | Lower-primary maths/English/science/Chinese are only reachable via the **"…(PSLE)"**-suffixed subjects (e g. selecting "Mathematics (PSLE)" then ticking Primary 1–4). Labelling a P1–P4 subject "PSLE" (a P6 exam) reads oddly to a teacher — a P2 parent isn't preparing for PSLE. The P1–P4 levels themselves ARE present and correctly scoped, so this is cosmetic. | Naming nit; functionally the lower-primary years exist and scope correctly. A neutral suffix like "(Primary)" would read better than "(PSLE)" for the P1–P4 band. | moe-03 |
| **MOE-T8** | **S4** | /register wizard back-navigation | Using the browser/Nav to return to Step 1 **wipes all entered wizard fields** (name/email/phone/password/profile selections reset to blank). | Not a blocker, but a tutor who steps back to fix a typo loses everything and must re-enter. Persist wizard state across step navigation. | (observed; form re-fill required) |

---

## ✓ Correct / improved — including the NEW taxonomy + scoping verdict

**New taxonomy — present and correctly placed (the headline wins):**
- ✓ **Combined Humanities (O-Level)** — present in all pickers (subject band correct; only the within-Secondary stream/year scoping is loose, see MOE-T1/T2).
- ✓ **H1 sciences (A-Level)** — H1 Physics, H1 Chemistry, H1 Biology all present; **scope correctly to Junior College 1 / Junior College 2 only** (no Secondary, no Primary). Verified live. Also H1 Maths/Econ present.
- ✓ **N using T levels** — Secondary 1–4 Normal Tech are real, selectable level options.
- ✓ **Sec 5 N using A** — Secondary 5 Normal Acad present.
- ✓ **Lower-primary** — Primary 1–Primary 4 selectable (reachable under the PSLE-suffixed primary subjects); plus a full **Pre-School** band (Nursery 1/2, Kindergarten 1/2) and Pre-School subjects (Numeracy, Literacy/English, Phonics, Chinese).
- ✓ **Generic "Mathematics (O-Level)" is GONE** — O-Level maths is split into **E-Math (O-Level)** and **A-Math (O-Level)** only. Confirmed across all three pickers and the browse filter. This was the key ask and it's right.
- ✓ Mother Tongue coverage good: Chinese/Malay/Tamil at PSLE & O-Level, plus **Higher Chinese (PSLE/O-Level)**. IB band (Maths/Physics/Chem/Bio/English/Econ) and Poly/University tiers present.

**Subject→level SCOPING — works at the band level:**
- ✓ Pick **A-Math (O-Level)** → only Secondary levels offered (no Primary/JC/Poly/Uni). Verified.
- ✓ Pick **Mathematics (PSLE)** → only **Primary 1–6** offered. Verified.
- ✓ Pick **H1 Physics (A-Level)** → only **JC 1 / JC 2** offered. Verified.
- ✓ Pick **Combined Humanities** → only the Secondary group offered. Verified.
- ✓ The listing wizard collapses the full grouped level list down to just the relevant band once a subject is chosen.
- ⚠ The remaining gap is *within* a band (stream × year) — see MOE-T1. No truly cross-band nonsense pair (e g. A-Math + Primary) was selectable; the system blocks those by hiding them.

**Money / commission (the trust test) — clear and fair:**
- ✓ Commission is shown **before** account creation as its own dedicated Step 3 ("Commission terms"). "Create account" stays **disabled** until the acknowledgment box is ticked.
- ✓ Terms are explicit: **"50% of the first month"**, charged **once** after the first confirmed session, **"You keep 100% of every month after that."** A worked example is shown (S$60/hr × 4 × 1.5h = S$360/mo → **S$180 one-time**), plus "no subscription fees or hidden charges" and "invoice to your registered email." Terms version stamped (2026-01-01) with a "Read full terms" link.
- Minor ambiguity (not logged as a defect): commission = 50% of the *first month* but is "due after the first **confirmed session**" — a cautious tutor may wonder what they owe if the student drops after session 1–2. Worth a one-line clarification, but the headline number and model are transparent and fair.

**Other working flows:**
- ✓ Registration consent gating is correct: Terms+Personal Data Protection Act+age (18) required, marketing optional, with Personal Data Protection Act purposes itemised and address removed.
- ✓ Duplicate phone returns **409 with a clear inline alert** ("This phone number is already registered.").
- ✓ Listing wizard **pre-fills**: title auto-built from subject+level ("Secondary 4 (Express) A-Math Tutor"), and the **rate band carried over from the profile** (40–60). Availability grid is a clean time×day matrix with quick-set shortcuts ("Weekday evenings").
- ✓ Apply **happy path** works end-to-end once profile is complete: detail-page "Apply / Express interest" → "Express interest" dialog (optional pitch, optional attach-listing) → submit operation for tutor applications 200 → button flips to **"✓ Interest sent"**. No personal information exposed ("Contact details are never shown").
- ✓ Public handle auto-suggested from display name (the suggested Public Handle feature).
- ✓ Profile-completeness banner is broadly accurate (50% pre-fill → 100% after required fields) and links to the profile — only the planning-area mismatch (MOE-T4) and the failed-save flash (MOE-T6) mar it.
- ✓ **Mobile 390px:** profile page AND listing subject/level picker both have **zero horizontal overflow** (scroll Width  equals  390); chips wrap, level groups stack cleanly.

**Console / errors observed:** only expected ones — 401 /auth/me when logged out (benign), the 409/422 business responses above (each with a proper JSON error code+trace Id). No 500s, no render crashes, no uncaught JS exceptions anywhere in the flow.

---

### Teacher's bottom line
The new taxonomy is a real, visible improvement — the generic "Mathematics (O-Level)" is gone, H1 sciences/Combined Humanities/N using T/Sec5 N using A/lower-primary are all present, and **band-level subject→level scoping is correct**. The commission is transparent and the apply happy-path is clean. What still trips an educator's eye: the **"O-Level" header sitting over Normal-stream levels (MOE-T2)** and **A-Math/Combined-Humanities being offered at Sec-1 / Normal-Tech (MOE-T1)** — fix the within-Secondary stream×year scoping and that header, and the taxonomy would read as genuinely curriculum-accurate. The **silent list-card Apply failure (MOE-T3)** and the **100%-complete-but-can't-apply contradiction (MOE-T4)** are the two user experience issues most likely to make a real tutor abandon.
