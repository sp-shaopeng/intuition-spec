# Regression Labs + "Regression Agent" — Proposal (choices, pre-deep-dive)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Status: **PROPOSAL for team review → founder choice** (NOT a build spec yet) ·
Author: Architect + quality assurance · Date: 2026-05-30

> Founder ask: hire a **regression agent** that manages a set of end-to-end test cases
> *individually* to ensure end-to-end flows don't break; runs in **continuous integration/continuous deployment**; unit tests
> are not enough; **separate regression labs for frontend and backend**; spec first; propose the
> tech/model for each; team-review; **present the choices before we deep-dive.**
> This doc presents the approach + options + a recommendation. No tooling is installed.

---

## 1. Why (the gap, evidenced)

Unit + web-slice tests have repeatedly passed while real flows broke:
- **M17 /public/assignments 500** — Postgres enum IS no value type-inference bug. Slipped
  because controller tests mock the service and the only real-database test (@Data Jpa Test)
  **self-skips when Docker/docker-java mismatches**. Caught by a human curl *after deploy*.
- **Showcase Lazy Initialization Exception** — degraded the landing; mocked-repo tests
  returned pre-hydrated entities and were structurally blind to it.
- **Login return To was broken for every protected route** — no test exercised the
  full login→bounce-back journey.

The common thread: **nothing exercises a full journey against a real, running stack in
continuous integration.** That is the regression lab's job.

## 2. What we're proposing

1. A new team role — **Regression Engineer ("Robin")** — added to the review panel and
   to continuous integration ownership. Owns the end-to-end suites, the flake budget, the gating policy, and (if we
   adopt the AI layer, §7) the regression agent.
2. **Two independent labs**, each with its own harness, runner, and continuous integration stage:
   - **frontend Regression Lab** — browser-driven end-to-end of the SPA against a running stack.
   - **backend Regression Lab** — application programming interface-level end-to-end against a running backend + **real**
     Postgres/Redis (not mocks, not self-skipping containers).
3. **Individually-managed test cases** — one file per end-to-end flow, tagged
   (@smoke, @auth, @marketplace, @admin), independently runnable and reportable,
   so a failure points at one journey, not a 2000-line suite.
4. **continuous integration/continuous deployment gate** — runs after the unit gates, against an **ephemeral full stack**
   (the documented operational check), on every pull request (smoke) + nightly (full). Hard gate on smoke;
   report-only initially on full, then promote to gate once stable.

## 3. Non-negotiable principles

- **Real dependencies.** Real Postgres + Redis in continuous integration (the self-skip is the enemy). The
  backend lab must hit the actual database query language that the enum-500 lived in.
- **Deterministic & isolated.** Seeded known state per run (reuse Dev Data Seeder or a
  dedicated regression seed); each test independent; no shared mutable order.
- **Flake-controlled.** Auto-wait/retries at the tool level, a tracked flake budget,
  quarantine lane for flaky cases (never silently @Ignore).
- **Fast feedback.** A <5-min @smoke subset blocks pull requests; the full suite runs nightly /
  pre-release.
- **Evidence on failure.** Traces, screenshots, video (frontend), request/response logs +
  database snapshot (backend), uploaded as continuous integration artifacts.

---

## 4. frontend Regression Lab — tech CHOICES

| Option | Pros | Cons | Fit |
|--------|------|------|-----|
| **Playwright** ⭐ | Fast, parallel, multi-browser; first-class **cross-origin** (needed for **M18 Google OAuth** redirect flows); traces/video/codegen; already used ad-hoc here for verification; great continuous integration | Team ramp (mild) | **Recommended** |
| Cypress | Loved DX, time-travel debug, big ecosystem | Historically weak at cross-origin / multi-tab (Google OAuth painful); slower at scale | Viable, but OAuth is a real wart |
| Webdriver IO / Selenium | Maximum browser coverage | Heavier, slower, more boilerplate | Overkill |

**Recommendation: Playwright** — the M18 Google sign-in end-to-end alone (cross-origin to
Google's consent screen) makes Playwright the low-risk pick; it's also already proven in
this repo.

## 5. backend Regression Lab — tech CHOICES

| Option | Pros | Cons | Fit |
|--------|------|------|-----|
| **REST Assured** (JVM) ⭐ | Lives in the Maven build; runs against a Testcontainers full Spring context **or** a deployed URL; Java team already fluent; real database query language exercised | Java-only authors | **Recommended (primary)** |
| Karate | BDD/Gherkin, low-code, readable by non-Java folks, built-in parallel + reports | Separate DSL to learn/maintain | Strong if we want PM-readable specs |
| Postman + **Newman** | A Postman collection **already exists** in the repo; lowest barrier; Newman runs it in continuous integration | Weaker assertions/maintainability at scale; JSON sprawl | Good as a stop-gap / smoke |
| Playwright application programming interface Request | Unifies frontend+backend tooling in one runner | Less idiomatic for deep JVM/database assertions | Viable if we want one tool |

**Recommendation: REST Assured against a Testcontainers-backed full stack** (real
Postgres/Redis), so the exact enum-IS no value database query language path runs in continuous integration. Optionally keep the
existing Postman collection driven by Newman as a quick smoke layer.

## 6. Cross-cutting option — contract testing (Pact)

The M14b/M17 wire contracts (e g. Showcase Request fields) are duplicated frontend↔backend.
**Pact** (consumer-driven contracts) would fail the build the moment the frontend's expected
shape and the backend's response drift. Optional add-on; complements (doesn't replace) end-to-end.

## 7. The "Regression Agent" — automation layer (optional, the part that makes it an *agent*)

Beyond a static suite, a **Claude-powered regression agent** (via the Claude Agent SDK,
runnable in continuous integration) could:
- **Triage failures** — read the trace/logs, classify real-regression vs flake, open a
  tagged issue with the suspected cause.
- **Author/maintain cases** — generate new end-to-end from a module spec (e g. turn the M18
  spec's flows into Playwright cases); update selectors when the user interface changes.
- **Flake detection** — track pass/fail history, auto-quarantine.

**Model proposal per task (cost vs capability):**
| Task | Model | Why |
|------|-------|-----|
| Routine smoke run orchestration / pass-fail triage | **Haiku 4.5** (claude-haiku-4-5-20251001) | Cheap, fast, high-volume per-pull request |
| Failure root-cause + test maintenance / selector repair | **Sonnet 4.6** (claude-sonnet-4-6) | Strong reasoning at moderate cost |
| Authoring complex multi-step flows / cross-cutting design from specs | **Opus 4.8** (claude-opus-4-8) | Deepest reasoning for the hard, rare jobs |

> This layer is **optional and incremental** — labs first (§4/§5), agent later. The
> founder may want just the labs, or the full agent.

## 8. continuous integration/continuous deployment integration (shape)


The illustration for . continuous integration/continuous deployment integration shape contains the following information:

- pull request pipeline:.
- lint unit (backend mvn / frontend vitest) → build images → spin ephemeral manage the required application services.
- → backend smoke (REST Assured) frontend smoke (Playwright) [HARD GATE, <5 min].
- Nightly / pre-release:.
- full end-to-end (both labs, all tags) → artifacts (traces/video/database snapshot) → report.


- Real Postgres+Redis via compose (kills the self-skip blind spot).
- Selective runs by tag; parallel shards; flake-quarantine lane.

## 9. Initial end-to-end flow catalog (what the labs would cover first)

Anon browse tutors/jobs → login-gated detail → login → bounce-back (return To);
register (tutee + tutor + commission); post a request → appears in public showcase;
tutor apply → admin review/approve → match → interview → review; admin create-admin;
**M18 Google sign-in (new + returning + link)** once built. Each = one managed case.

## 10. Decisions for the founder (bring these back after review)
- **D1 — frontend tool:** Playwright (rec) / Cypress / other.
- **D2 — backend tool:** REST Assured (rec) / Karate / Postman+Newman / Playwright-application programming interface.
- **D3 — Scope now:** labs only, or labs **+** the AI regression agent (§7)?
- **D4 — Contract testing (Pact):** include now, or defer?
- **D5 — Gating:** smoke = hard pull request gate from day 1, or report-only until stable?

## 11. Roadmap placement
New Tier-1/2 item **"Regression Labs (frontend + backend)"** — foundational quality infra; should
land before/with the next major feature (M18). Owner: Regression Engineer + Dev Ops.
