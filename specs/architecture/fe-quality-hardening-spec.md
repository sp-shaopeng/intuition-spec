# frontend Quality Hardening Spec

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Owners:** frontend-expert (Sam) + uiux-designer (Jamie)
**Scope:** intuition-frontend only. **No backend change** except the optional INT-061 backend alternative.
**Status:** DRAFT — doc only. Pending /team-review → Boss → build.
**Date:** 2026-06-05
**Gate (must hold on build):** the documented operational check 0, the documented operational check (tsc) 0, the documented operational check ≥97.5% line / ≥88% branch.

This spec collects the frontend-quality and user experience defects surfaced by the whole-codebase review. Each item is
grounded in a real file:line (re-read and confirmed against the working tree on 2026-06-05). Items are
independent and may be built in any order, but all land in one intuition-frontend lane (sequential builds —
same subtree, per CLAUDE md §A2 concurrency rule).

Severity legend: **S1** user-visible bug shipped in every flow · **S2** a11y/user experience correctness · **S3** hygiene/perf advisory.

---

## H1 — Double error toasts app-wide (S1)

### Problem
Every mutation error currently shows **two** toasts.

### Current state (file:line)
- query Client implementation reference installs a **global Mutation Cache on Error** that calls
  the show global error toast operation → the error operation for any non-401 error.
- The function's own JSDoc (query Client implementation reference) documents the invariant: *"call sites with their own
  tailored on Error toast set suppress the shared error message when the screen already handles it to avoid double toasts."*
- **29 call-site files** fire their own the error operation (64 call sites total), e g.
  Matching implementation reference, Applications implementation reference,
  Apply Button implementation reference, Create Request implementation reference, …
- A search for the error-suppression setting returns **only the three lines in query Client implementation reference** — i e. **zero**
  mutations set suppress the shared error message when the screen already handles it. The documented invariant is violated everywhere.

Net effect: a failed mutation runs the call-site's toast error **and** the global fallback → 2 toasts.
(Note query Client implementation reference is in the Vitest coverage exclude list, so the regression has been invisible
to the gate.)

### Design (chosen approach + the rejected one)
The mechanism in query Client implementation reference is correct; the bug is that no call site opts out, and the invariant is
unenforceable by hand across 64 sites. **Decision: invert the default — global fallback fires ONLY when a
mutation opts IN, and queries keep the existing opt-out fallback.**

- **Mutations:** the global Mutation Cache on Error becomes opt-**in**: it fires only when the mutation sets
  show a shared error message for this action (a mutation with no meta → the call site owns the toast; no global toast).
  This matches reality — virtually every mutation already has a tailored on Error (better copy, e g.
  "Job assigned." failure context) — and eliminates the double-toast for all 29 files with **zero per-site
  edits**. A small number of "fire-and-forget" mutations that have NO on Error (audit during build) get an
  explicit show a shared error message for this action so they still surface a toast (no silent failure — INT-004).
- **Queries:** keep the *current* opt-**out** behaviour (Query Cache on Error fires unless
  suppress the shared error message when the screen already handles it) — queries rarely have tailored handlers and a silent failed fetch is
  worse than a redundant toast. Document the asymmetry in the JSDoc.
- Keep the status equals 401 skip (logout/redirect path owns it).
- **Remove query Client implementation reference from the Vitest coverage exclude list** (vite config implementation reference) so the
  toast policy is gate-covered going forward. (If full coverage of the Query Client construction is
  impractical, cover the two on Error reducers via the extracted helpers below; do not silently re-exclude.)

Rejected alternative: *"set suppress the shared error message when the screen already handles it on all 29 files."* — 64 hand-edits, no
compile-time guarantee a new mutation remembers it, and the invariant drifts again on the next pull request. Inversion
makes the safe default (no double toast) the *zero-config* default.

Refactor for testability: extract the two cache on Error bodies into named pure helpers
(the should toast mutation error operation, the should toast query error operation) returning boolean, so they
unit-test without constructing a Query Client.

### Acceptance criteria
- AC1: a mutation **without** meta that errors shows **exactly one** toast (the call site's), never two.
- AC2: a mutation with show a shared error message for this action and **no** on Error shows exactly one (the global) toast.
- AC3: a 401 error shows **zero** toasts from either cache.
- AC4: a query error (no meta) shows one toast; suppress the shared error message when the screen already handles it shows zero.
- AC5: a search for the error-display settings shows the policy is referenced by name, and no
  call site relies on the *old* (unset) opt-out for correctness.

### Test notes
- New query Client test implementation reference: unit-test should Toast Mutation Error / should Toast Query Error across
  no meta, opt-in/opt-out meta, 401, 500 → asserts the boolean policy (covers AC1–AC4 cheaply).
- One integration test mounting a component whose use Mutation has a tailored on Error + forced 500 →
  spy on toast error (or count rendered toasts via Toast Container) → **exactly one** call (AC1, the
  load-bearing regression guard).
- Verify the vite config implementation reference exclude removal keeps the gate green.

---

## H2 — Button primary is cream-on-cream → invisible CTA on light surfaces (S1, WCAG 1.4.11)

### Problem
The primary button is **cream fill with navy text** — on the app's cream/white page surfaces it has almost no
fill-vs-background contrast, so the single most important action on a screen reads as a flat label, not a
button. Violates WCAG 2.2 **1.4.11 Non-text Contrast** (user interface component needs ≥3:1 against adjacent colour) and
the one-primary-action / Von Restorff principle (the CTA should be the highest-contrast thing on the view).

### Current state (file:line)
- Button implementation reference:
  primary: bg-cream text-navy hover:bg-cream-muted border border-cream.
- Tokens (tailwind config implementation reference): cream #FBF6 EE, cream muted #F2 EBE0 — i e. the primary fill is the
  **same family as the page background** (most pages sit on cream/white). Border is cream too, so the
  button has no visible edge either.
- secondary (16) is cream **text** on transparent (designed for dark/navy surfaces); ghost (17) is
  navy text on a cream-border outline. There is **no** solid, high-contrast CTA variant for light surfaces.
- Real consequence on a light surface: Login's submit <Button type is submit … size is lg>
  (Login implementation reference) and Register's submit are cream-on-cream — the primary conversion CTA is the
  lowest-contrast element on the page.

### Design
Treat cream primary as the **on-dark** variant (it IS correct on the navy hero / dark sections) and add a
**token-backed solid CTA for light surfaces** as the new default primary.

- Add variant **on Dark** = the *current* primary recipe (bg-cream text-navy …) for use on navy/hero
  sections (Landing hero CTAs that sit on navy).
- Redefine **primary** as a solid, ≥4.5:1 CTA on light surfaces: bg-clay text-white border border-clay
  hover:bg-clay-lightclay#C84B31 on white → ~4.6:1 for white text; clay vs cream background ≫ 3:1 for the
  user interface-component contrast). clay is already the brand action accent (used by danger, focus ring, the active
  nav underline border-clay), so this reads as "the action colour," not a new token.
  - If clay-as-primary collides visually with danger (also clay), differentiate danger to a darker/again
    distinct treatment OR keep danger clay and make primary bg-navy text-white (navy #0 F2138 on white →
    ~14:1). **Resolve in team-review**; both are token-backed and pass 1.4.11. Default proposal: primary =
    navy solidleavingclay exclusively for destructive — cleaner semantic separation (Similarity law:
    same colour = same function).
- Keep secondary/ghost as-is. Audit every variant is primary usage that actually sits on a **dark**
  surface and switch those to on Dark (grep variant is primary + check the parent section bg; Landing hero
  is the main one).
- No new dependency; pure Tailwind token classes.

### Acceptance criteria
- AC1: primary button fill has ≥4.5:1 text contrast and ≥3:1 fill-vs-(white|cream) background contrast
  (1.4.11). State the measured ratios in the pull request.
- AC2: there is a documented variant (on Dark) for navy/hero surfaces; the Landing hero CTAs use it and
  remain legible.
- AC3: Login + Register submit buttons are visibly solid CTAs on their light cards.
- AC4: focus-visible ring (focus-visible:ring-clay) still reads against the new fill (if primary becomes
  clay, switch its ring to ring-navy so the ring isn't ring-on-same-colour).

### Test notes
- Button test implementation reference: assert the primary variant emits the new solid class (a class-presence
  assertion is the load-bearing guard — it stops a refactor silently reverting to cream-on-cream). Assert
  on Dark emits the old cream recipe. Assert danger unchanged.
- a11y: jsdom can't measure rendered contrast; record the computed ratios in the spec/pull request and add an axe pass
  on a Login render (jest-axe/vitest-axe if already present) asserting no color-contrast violation on
  the submit button. If axe isn't wired, the manual ratio note + class assertion is the floor.

---

## H3 — Input a11y: error not wired to the control (S2, WCAG 3.3.1 / 4.1.2)

### Problem
The Input renders a visible error message but **does not associate it with the input** — a screen reader
user tabs into the field and hears the label only, never that it's invalid or why. Sighted-only error user experience.

### Current state (file:line)
- Input implementation reference — the <input> has id, focus styles, and (37) error  and 
  'border-clay'colour-only error indication , but **noaria-invalid** and **noaria-describedby**.
- 43-44 — hint and error render as  siblings with **no id**, so nothing can point at them.
- error is colour-only at the field boundary → WCAG **1.4.1 Use of Color** as well (colour is the only
  programmatic invalid signal).

### Design
- Give the hint and error s stable ids derived from input Id: a unique identifier for the field error, a unique identifier for the field hint.
- On the <input>: mark the field invalid only when it has an error and
  associate the field with its error if present, otherwise its hint if present (error takes precedence over hint, matching
  the existing hint and !error render guard at 43).
- Keep the visible ⚠ glyph (it already pairs an icon with colour → satisfies 1.4.1; ensure the ⚠ has
  aria-hidden since the adjacent text carries the meaning, consistent with Toast implementation reference).
- No application programming interface change for consumers — input Id is already derived (13).

### Acceptance criteria
- AC1: when error is set, the input exposes aria-invalid is yes and aria-describedby pointing at the
  error 's id.
- AC2: with only hint (no error), aria-describedby points at the hint id; with neither, the attr is absent.
- AC3: error is conveyed by more than colour (text + icon already present; assert text is programmatically linked).

### Test notes
- Input test implementation reference: render with error is Required → the get by role operation has
  aria-invalid is yes and its aria-describedby resolves to the element containing "Required". Render with
  hint only → describedby = hint id. Render with neither → no aria-describedby/aria-invalid.

---

## H4 — Error toasts are role is status / polite → screen readers may miss them (S2, WCAG 4.1.3)

### Problem
The toast region is role is status + aria-live is polite. Polite announcements are queued behind whatever
the SR is currently reading and can be **dropped** if the user navigates. For an **error** ("Could not assign
job"), the user must be told *now* — polite is wrong for the assertive class.

### Current state (file:line)
- Toast implementation reference — the container is a single
  role is status aria-live is polite region wrapping **all** toast types (success, info, **error**).
- Per WCAG 4.1.3 / ARIA APG, errors and time-sensitive failures should be assertive (role is alert),
  not polite.

### Design
Split announcement politeness by toast type without changing the visual layout.

- Render **two** stacked live regions in the container: a role is status aria-live is polite region for
  success/info, and a role is alert aria-live is assertive region for error toasts. Partition
  toasts by type and render each into its region (visual stacking unchanged — both fixed bottom-right;
  order errors above or interleave by timestamp — keep current visual stacking, this is purely the live-region
  wrapper).
- Alternative (simpler, acceptable): keep one container but set aria-live/role **per toast item** —
  however nested live regions are fragile; the two-region split is the APG-blessed pattern. Decide in review;
  two-region is the recommendation.
- Keep aria-label is Notifications; keep the dismiss button's aria-label.

### Acceptance criteria
- AC1: an error toast lives inside a role is alert (assertive) region.
- AC2: success/info toasts live inside a role is status (polite) region.
- AC3: visual position/stacking and dismiss behaviour unchanged.

### Test notes
- Toast test implementation reference: push an error toast → assert it renders within an element with
  role is alert; push a success toast → within role is status. Existing dismiss test stays green.

---

## H5 — Login (and Register) have no auto Complete → autofill/password-managers broken (S2)

### Problem
The login form has **no auto Complete attributes**, so browsers and password managers can't reliably autofill
email/password — friction on the highest-traffic conversion surface (mobile especially; Baymard-class drop-off
and Jakob's Law: users expect autofill to work). Register has the same gap.

### Current state (file:line)
- Login implementation reference — the email and password <Input>s pass type, required, placeholder,
  error but **no auto Complete**. grep confirms zero auto Complete in Login implementation reference (the only
  auto Complete usages in the app are Reset Password implementation reference, Forgot Password implementation reference, Otp Input implementation reference,
  Combobox implementation reference — Login/Register were missed).
- Register implementation reference — display Name, email, phone, password, confirm Password inputs,
  none with auto Complete.

### Design
Input already spreads ...props onto the <input> (Input implementation reference), so auto Complete is a pass-through —
no component change. Add the standard tokens:

- Login: email auto Complete is email + input Mode is email; password auto Complete is current-password.
- Register: display Name → auto Complete is name; email → auto Complete is email + input Mode is email;
  phone → auto Complete is tel + input Mode is tel; password → [REDACTED CREDENTIAL];
  confirm Password → auto Complete is new-password.

### Acceptance criteria
- AC1: every credential/identity field in Login + Register carries the correct auto Complete token.
- AC2: email fields set input Mode is email, phone sets input Mode is tel (correct mobile keyboard).

### Test notes
- Login test implementation reference / Register test implementation reference: assert the email input has auto Complete is email and password has
  current-password (Login) / new-password (Register). These are cheap attribute assertions but guard
  against a refactor dropping them.

---

## H6 — No RHF mode: on Blur anywhere → submit-only validation (S2)

### Problem
Every form validates **only on submit**. A user fills a long form (Register, Create Request, Create Listing),
hits submit, and gets a wall of errors at once — instead of inline, on-blur feedback as they leave each field.
On-blur inline validation cuts abandonment ~10% (research lesson) and matches the Doherty/feedback principle.

### Current state (file:line)
- A search for forms configured to validate on blur or change → **no matches**. All 11 the use form operation call sites omit
  mode, so RHF defaults to on Submit.
- Confirmed at Login implementation reference (connect the form to its validation rules) and
  Register implementation reference — neither sets mode.

### Design
- Adopt mode: on Blur on the form-heavy, conversion-critical screens first: Register, Create Request
  (post a job), Create Listing, Profile forms, Commission. This validates a field when the user leaves it,
  surfaces the error adjacent to the field (already wired once H3 lands), and keeps submit as the final net.
- Pair with re Validate Mode: on Change (RHF default) so once a field has errored, fixing it clears the error
  live — the standard "error on blur, clear on change" user experience.
- **Keep no Validate on the <form>** (Form Layout already sets it; SHARED-LESSONS Wave-4 rule) so zod owns
  validation, not native popups.
- Login is lower value (2 fields) — apply for consistency but it's optional; the win is the long forms.
- This is additive config; no schema or copy change. Error copy improvements (adaptive "what + how to fix")
  are out of scope here (tracked separately) — this item only changes *when* validation fires.

### Acceptance criteria
- AC1: the listed forms set mode: on Blur.
- AC2: blurring a required field empty shows its error before submit; typing a valid value clears it.
- AC3: submit still blocks and shows all errors (final net unchanged).

### Test notes
- For one representative form (Register): focus the email input, type an invalid value, blur → assert the zod
  error renders **without** clicking submit. Then type a valid value → assert the error clears (re Validate
  on Change). Existing submit-validation tests stay green.

---

## H7 — Dead code: Nav,Footer implementation reference (S2 — INT-054-class trap)

### Problem
Two **unused duplicate** components shadow the live ones. They are a maintenance trap: a future fix applied to
the wrong file changes nothing (exactly the INT-054 "fixed the lookalike, not the real surface" failure).
Worse, the orphan Nav implementation reference is **stale and wrong** — it links to a non-existent /tutor/requests
route and calls the use auth store operation (the live store/flow uses perform Logout), so if anyone ever wired
it up it would break.

### Current state (file:line)
- Nav implementation reference and Footer implementation reference exist.
- A search for imports of the obsolete navigation and footer → **zero imports**. The real ones are
  Nav implementation reference and Footer implementation reference, imported by Public Layout implementation reference,
  Tutee Layout implementation reference, Tutor Layout implementation reference (2-3 each).
- Nav implementation reference references /tutor/requests (dead route) and 8 logout from the
  store — divergent from the live nav.

### Design
- **Delete** Nav implementation reference and Footer implementation reference (the modules/ dir is
  otherwise empty → remove it too). These are physical *source artifacts*, not data — the soft-delete-only
  directive applies to **data rows**, not dead source files, so a hard delete is correct here.
- Confirm no test imports them (grep test/) before deletion.

### Acceptance criteria
- AC1: the private implementation folder no longer exists; a search for imports of the obsolete navigation and footer in the application and tests → empty.
- AC2: the documented operational check (tsc) + test:ci stay green (proving nothing depended on them).

### Test notes
- No new test; the build + a repo grep are the verification. (Optionally an ESLint no-unused-modules/
  import/no-unresolved pass to catch future orphans — advisory, not required.)

---

## H8 — Logo-home target diverges from login Landing For Role (S2, INT-037/038 consistency)

### Problem
Clicking the **logo** when logged in as a tutor sends them to /tutor/dashboard ("My Listings", a *More*
overflow tab), but **logging in** sends a tutor to /requests ("Find Work", primary 0). Two different
"home" answers for the same role → the user lands somewhere whose nav tab isn't even active. The codebase
already centralised the landing rule (login Landing For Role) precisely to stop this drift; the logo wasn't
migrated.

### Current state (file:line)
- Nav implementation reference hardcodes dashboard Path:
  user role equals TUTOR ? /tutor/dashboard : role equals ADMIN ? /admin/review : /tutee/dashboard.
- 109 logo send signed-in users to their dashboard and guests to the homepage and 172 the display Name link also use dashboard Path.
- nav Tabs implementation reference login Landing For Role returns each role's primary 0:
  TUTOR → /requests (TUTOR NAV primary 0, 46), TUTEE → /tutee/dashboard, ADMIN → /admin/review.
- So for **TUTOR** the two diverge (/tutor/dashboard vs /requests); TUTEE/ADMIN happen to match today
  but only by coincidence — nothing enforces it.
- nav Tabs implementation reference JSDoc already states the landing tab MUST stay in sync with the login redirect.

### Design
- Route the logo (and the display Name link) **through the login landing for role operation** instead of the
  local dashboard Path constant. Delete the hardcoded dashboard Path ternary (Nav implementation reference) and replace
  its two usages (109, 172) with the login landing for role operation.
- This makes "logo home"  equals  "login landing"  equals  primary 0 for **all** roles, by construction, sourced
  from one function — they can't drift.

### Acceptance criteria
- AC1: logged-in logo target  equals  the login landing for role operation for TUTOR, TUTEE, ADMIN.
- AC2: logged-out logo target = /.
- AC3: a tutor clicking the logo lands on /requests (the active primary tab), not the demoted listings page.

### Test notes
- Nav implementation reference is currently in the Vitest coverage **exclude** list (vite config implementation reference), so it's untested.
  Add nav Coupling test implementation reference (unit, no render) asserting **for every role**:
  the login landing for role operation (the coupling the holistic-review condition #3 demands).
  This guards H8 *and* future nav edits without needing to un-exclude Nav implementation reference.
- Optionally render Nav for a TUTOR and assert the logo href resolves to /requests (would require
  removing the Nav exclude — nice-to-have, not required; the unit coupling test is the load-bearing guard).

---

## H9 — MAX PRIMARY TABS declared but unenforced (S2)

### Problem
MAX PRIMARY TABS 5 is documented as a "hard cap" (Hick/Miller) but is **only a comment** — nothing
asserts any role's primary array obeys it. A future tab addition can silently overflow the header/bottom bar.
(SHARED-LESSONS holistic-pass rule: a cap that isn't tested is just a comment.)

### Current state (file:line)
- nav Tabs implementation reference the documented operational check — referenced in JSDoc only; no runtime/test
  enforcement. Today TUTEE has 4 primary (31-36), TUTOR 4 (46-51), ADMIN 5 (63-69) — all within cap,
  so the gap is latent.
- Sibling MAX BOTTOM TABS 4 (95) IS applied via the slice operation (98-99) — but a slice
  truncates silently rather than failing, so it also deserves an assertion that the source set is sane.

### Design
- No source change to nav Tabs implementation reference. Add a test that asserts the invariant across **all** roles.
- (Optional hardening, decide in review: a dev-only console warn or a module-load assertion if
  primary length MAX PRIMARY TABS — but the test is the required deliverable; a thrown error at module
  load could take down the app, so prefer the test + an advisory warn at most.)

### Acceptance criteria
- AC1: a test asserts group primary.length is at most MAX PRIMARY TABS for TUTEE, TUTOR, ADMIN (and the EMPTY set).
- AC2: adding a 6th primary tab to any role makes that test fail.

### Test notes
- Extend nav Coupling test implementation reference (or a nav Tabs test implementation reference): iterate TUTEE NAV, TUTOR NAV, ADMIN NAV,
  assert primary length is at most MAX PRIMARY TABS; also assert bottom Tabs For User never exceeds MAX BOTTOM TABS.

---

## H10 — No route code-splitting / manual Chunks → one large eager bundle (S3, advisory)

### Problem
Every route component is **statically imported** in App implementation reference, so the entire app (all dashboards, all admin
pages, charts, wizards) ships in one initial chunk. Only the Landing showcase strips are lazy. Larger LCP/INP
on first load, especially on mobile/3G; Vite will emit the >500 kB chunk advisory.

### Current state (file:line)
- App implementation reference — all pages imported eagerly (Login from ./pages/Login,
  Register …, all pages/admin/*, pages/tutor/*, pages/tutee/*, pages/browse/*).
- Only lazy usage in the app: Landing implementation reference (Active Tutors Strip, Recent Requests Strip behind
  <Suspense> at 105).
- vite config implementation reference has **no build rollup Options output.manual Chunks** and no build block at all (it's a
  Vitest config; the prod build uses Vite defaults → a single vendor+app chunk).

### Design
Apply the SHARED-LESSONS Vite bundle-splitting playbook (split by change cadence, not just package):

- **Route-level React lazy** for the heavy / role-gated route groups behind a shared <Suspense> boundary
  in each layout (Admin Layout, Tutor Layout, Tutee Layout Outlets). Prime split points: the entire
  admin/* group (incl. Analytics + any charts), the wizard flows (Create Request/Create Listing), and the
  authenticated dashboards. Public/auth pages (Landing, Login, Register) can stay eager (first-paint critical).
- Provide a Suspense fallback that **reserves layout footprint** (skeleton, fixed height) to keep CLS < 0.1
  (per the Landing strip pattern at Landing implementation reference).
- Add build rollup Options output.manual Chunks to pin stable vendor deps into a long-cached chunk:
  react/react-dom/react-router-dom in one vendor chunk, @tanstack/react-query in another, any charting
  lib separate. Split by change cadence so a frequently-edited util doesn't bust the vendor hash.
- **Measure before/after** (the documented operational check chunk report) — this is an advisory perf item; success = the
  initial chunk drops below the advisory and admin/charts load on demand. Don't over-split (too many tiny
  chunks hurt web request protocol overhead).

### Acceptance criteria
- AC1: admin route group (and charts) load in a separate chunk, fetched only when an admin navigates there.
- AC2: manual Chunks pins react/react-router and react-query into vendor chunks; build emits them as distinct
  hashed files.
- AC3: no >500 kB single-chunk advisory after the split (or a documented justification if one remains).
- AC4: lazy boundaries have layout-reserving fallbacks (no CLS regression).

### Test notes
- Lazy routing is covered by existing route render tests (they must still pass through <Suspense> — wrap
  test renders accordingly). Add a build-artifact check in the pull request notes (chunk sizes before/after). No unit
  test asserts chunk layout; the gate is the documented operational check succeeding + the documented chunk report.

---

## H11 — INT-061: admin Matching "Job" label is empty when job Title is null (S3)

### Problem
Live /admin/matches returns job Title: no value for seeded/real posts (parents describe a job by
subject/level/area, not a free-text title), so the Matching table's **Job** cell renders **empty** — the link
target still works, but there's no visible label to click. (Filed as INT-061.)

### Current state (file:line)
- Matching implementation reference — the Job cell links to
  the applicants page for the match’s tuition request and renders the match’s job title directly (414); when
  job Title is null it's blank.
- Adjacent fields available on the row: match tutee Subject (406), match tutee Name (402). A
  planning-area field may also be present on the data transfer object (confirm against Admin Match Item Response during build).

### Design
- **frontend fallback (quick win, this spec):** render
  [REDACTED PASSWORD] Subject using match tutee Subject job the end of the display condition — or a literal **"View job"** if subject
  is also absent — so the cell always has a meaningful, clickable label. Use the existing friendly subject
  label map (taxonomy implementation reference/status implementation reference), not the raw enum (INT-027 "labels not codes" for
  display). Prefer subject (+ planning-area if present on the data transfer object) over a bare "View job".
- **Optional backend alternative (out of scope unless review escalates):** derive a display title from
  subject+level on Admin Match Item Response (cf. INT-052 derived-title). frontend fallback is the agreed quick win;
  do not block on backend.

### Acceptance criteria
- AC1: when job Title is null, the Job cell shows a non-empty, meaningful label derived from subject (e g.
  "Secondary Math job") and remains a working link to the applicants page.
- AC2: when job Title is present, it renders unchanged.
- AC3: the fallback uses friendly labels, never a raw enum code.

### Test notes
- Matching test implementation reference: render a row with job Title: no value + a known tutee Subject → assert
  the cell shows the friendly fallback text AND the link href is still
  the administrator’s applicants page for the selected tuition request. Render with job Title: Foo → asserts "Foo".

---

## Cross-cutting notes

- **Coverage exclusions touched:** H1 and H8 reference removing query Client implementation reference (and discuss Nav implementation reference) from
  the Vitest coverage exclude (vite config implementation reference). Removing an exclusion can drop coverage — budget for
  the tests above; do **not** re-add an exclusion to paper over a gap (CLAUDE md gate rule). If un-excluding
  Nav implementation reference is out of scope, the nav Coupling unit test still enforces H8/H9 without rendering Nav.
- **No magic strings:** any new route literal must go through nav Tabs/application programming interface ROUTES/typed enums; the fallback
  labels in H11 must come from the canonical taxonomy label map.
- **Soft-delete directive** applies to data, not source — H7's file deletion is correct.
- **Build lane:** all items are intuition-frontend-only (except optional H11 backend) → one sequential frontend lane.

## Definition of done
Per-item ACs met + the listed tests green; the documented operational check 0, the documented operational check (tsc) 0,
the documented operational check ≥97.5% line / ≥88% branch; deploys+runs locally (manual contrast/autofill spot-check on
Login + a tutor logo click landing on /requests); then push on sp-dev (no branch change).

→ team-review → Boss → build.
