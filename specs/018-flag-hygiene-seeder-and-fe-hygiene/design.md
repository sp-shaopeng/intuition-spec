# Design — Flag hygiene, seeder, frontend hygiene (018)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> Owned by frontend-expert (routing/copy/CTA/flag toast) + software-engineer (seeder). Grounded 2026-06-22.

## REQ-001 — WhatsApp flag-gating

Phone Settings Section already reads the enabled flag for the modal (23). Extend the gate so the
profile-save/phone-change path does not emit a **WhatsApp** verification toast or trigger a **WhatsApp one-time passcode** send when
OFF. For a changed phone, show 012's pending/email-confirmation copy and retain the current phone until the
explicit confirmation commits. Show "phone updated" only after that commit. If a backend one-time passcode send is wired to the phone-change, confirm it is already
flag-gated (it should be — INT-082) and only the frontend toast leaks.

**Cross-wave (MANDATORY): channel-scope the suppression to WhatsApp/one-time passcode only.** The suppression key MUST be the
WhatsApp channel, NOT "any phone-change verification copy". Spec 012/REQ-004 introduces an **email-link**
phone-verification flow (a confirm link to the account email, independent of INT-082); 018/REQ-001's suppression MUST
NOT hide that email-link "we sent a confirmation link to your account email" copy. This is a Wave-3 build note for
this REQ and a Wave-2 build note for 012/REQ-004's author.

## REQ-002 — seeder tutor user id

Dev Data Seeder save Interview (1078-1097) sets tutor Post/tutee/tutee Post/status but never set Tutor User
(verified — only seeded rows are affected; live Interview Service create Interview already sets it). Add
the set tutor user operation. **Shared invariant:** seeded rows must satisfy the SAME invariant
the live path enforces — every interview has a non-null tutor user id equal to its tutor-post's tutor. Dev/test-data
only; no schema. Dev Data Seeder Test must ASSERT the invariant (not just run): every seeded interview has a non-null
tutor user id AND it equals the get tutor operation's id — so a future seeder change can't silently regress it.

## REQ-003 — seed variety (measurable threshold)

In the seeder's job/tutor generation, spread subject / level / budget-bucket / planning-area so no single
combination dominates the public strips. Inspect the showcase queries (active tutors / recent requests) so the
variety reaches the landing//assignments strips.

**Measurable acceptance threshold (was unmeasurable "looks realistic"):** among the OPEN (ACTIVE) tutee posts the
seeder produces for the showcase, the set SHALL satisfy ALL of:

- **≥ 8 distinct subject Code values**, and **≥ 5 distinct level Code values**, and **≥ 4 distinct planning-area
  values** appear across the open posts; and
- **no single subject Code, level Code, budget-bucket, planning-area tuple is more than 20%** of the open
  showcase posts (i e. the "8 identical [REDACTED PASSWORD] cards" symptom cannot recur).

The acceptance fixture must contain **at least 8 open showcase posts** (8 distinct subjects already imply this
minimum). If the existing fixture is smaller, expand it; never scale the thresholds down with the min operation.
Validate the shared taxonomy combinations and assert the counts over the eligible showcase population, not
over closed/ineligible fixtures. The ≤20% tuple bound and all three distinct-count floors remain fixed.

## REQ-004 — /dashboard redirect

Add a /dashboard route in App implementation reference (before the * catch-all, 155) that resolves the user's role and
redirects to the signed-in user’s role-specific dashboard, or to login when unsigned in (reuse the
existing auth-redirect/return-path logic).

## REQ-005 — step copy

Find the wizard step header that says "All optional" and reconcile it with the actual field requirements
(Create Request implementation reference; Subject * at 335). Accurate copy per step.

## REQ-006 — CTA intent

Route the ENUMERATED generic auth CTAs (see requirements REQ-006 in-scope list) to /login for anonymous visitors
(register reachable from there), consistent with M17. **FLIP set:** generic public /register CTAs with no
role/create-account intent — e g. Rate Guide implementation reference "Post a request". **KEEP set (regression-guard,
MUST NOT flip):** the explicit create-account CTA Nav implementation reference "Sign up" → /register, and the role-qualified
intent CTAs Nav implementation reference (?role is tutee, INT-172), Active Tutors Strip implementation reference / Recent Requests Strip implementation reference /
Landing implementation reference (?role is tutor). Browse Assignments implementation reference already → /login (reference pattern). The RTL test
enumerates both sets.

## Test / coverage

- backend gate ≥97.5/97.5 — Dev Data Seeder Test asserts tutor User set + variety.
- frontend gate ≥97.5/88 — flag-OFF no-toast, /dashboard redirect per role, step copy, CTA routing.
- Live: flag-OFF profile save; reseed → tutor interviews non-empty + varied showcase; /dashboard redirect; CTA → login.
