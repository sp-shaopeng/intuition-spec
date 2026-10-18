# INT-064 (follow-up) — Composed Multi-Repo Regression / end-to-end continuous integration Lane

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Status:** SPEC v2 — revised per team-review + Boss SEND-BACK (2026-06-13)
**Date:** 2026-06-13
**Author:** devops-engineer
**Tracker:** [README implementation reference](../issues/README.md) — INT-064 (S1 → PARTIALLY DONE)
**Migration:** none · **App code:** none (continuous integration/infra only)
**Next milestone:** build the lane; **final verification is access-gated (see §5)**

> **Review history.** v1 → team-review → Boss **SEND BACK** (2026-06-13), Casey (Security) condition folded into v2:
> the wrapper docker-compose implementation reference carries **committed dev secrets** (Tink AES-SIV keyset, signed session token keyed message authentication, database/Redis
> passwords — status-briefing §7). "Commit the composing stack into the e2e repo" must **never** carry those values;
> the continuous integration compose sources all secrets from GitHub Actions secrets / a continuous integration-generated ephemeral keyset (§3.1, §3.4).

---

## 1. Problem & what is already done

INT-064 originally flagged that continuous integration was wired to none of the pushed subrepos. That is **mostly closed**:

**Done — do not redo (verified present):**
- ci implementation reference — run the full backend build and verification checks in continuous integration → Testcontainers repo
  tests RUN + JaCoCo ≥97.5/97.5. **Confirmed green** on the recorded revision (surefire 1714 + the Testcontainers
  failsafe band incl. Quota Concurrency IT — INT-077's race-safety evidence captured).
- The frontend CI workflow installs dependencies, checks formatting and code rules, builds the application, and runs its coverage tests. **Historically reported
  green** (frontend run #22, founder-verified on the Actions tab).
- regression implementation reference + docker-compose ci implementation reference — present in the e2e repo.

Getting the per-repo lanes green required INT-083 (postgres digest re-pin) + INT-086 (fixture cascade
+ the ci-no-lab profile that excludes the live-stack lab from the per-pull request gate, since it fail-hards
without a running stack).

**Remaining gap (this spec).** The **composed multi-repo lane** — the black-box *IT regression band
and the Playwright e2e suite — needs the full stack spanning all subrepos. The composing
docker-compose implementation reference lives in the **unversioned wrapper** (REDACTED local path),
which is not a git repo, so the e2e regression implementation reference cannot check it out. The cross-repo run (a
POLYREPO PAT-driven multi-checkout) is unverified.

---

## 2. Goal

A single continuous integration lane that:
1. Checks out all three subrepos (intuition-backend, intuition-frontend, e2e) at the right refs.
2. Brings up the full composed stack (Postgres + Redis + application programming interface + SPA) in the runner.
3. Runs the backend black-box *IT regression band **and** the Playwright e2e suite against it.
4. Publishes the Playwright report + IT surefire/failsafe reports as artifacts.
5. Is **observed green on the GitHub Actions tab** (the project Do D — claims trace to a seen run).

---

## 3. Design

### 3.1 Where the composing stack lives

The wrapper docker-compose implementation reference is unversioned and spans repos, so it cannot be the continuous integration source of
truth. Choose one (devops call at build time):

- **(A) Commit the continuous integration compose into the e2e repo** (recommended) — the e2e repo already owns
  docker-compose ci implementation reference; extend it to compose the full stack (build backend + frontend images from the
  checked-out sibling dirs, plus pg/redis). The e2e lane is the natural home: it already orchestrates
  the cross-cutting suite. **No dependency on the unversioned wrapper.**
- **(B) A dedicated infra repo** holding the composing stack — heavier; only if more than the e2e lane
  needs it. Not recommended for one consumer.

> **🔒 SECURITY GATE (Boss SEND-BACK #5 / Casey).** Do **NOT** copy the wrapper docker-compose implementation reference
> into the e2e repo — it embeds committed **dev secrets**: the Tink AES-SIV master keyset, the signed session token keyed message authentication
> key, and database/Redis passwords (docker-compose implementation reference, status-briefing §7). Committing it would publish
> those into another repo's history. The continuous integration compose must declare every secret as an
> environment-variable reference with no embedded default value; GitHub
> Actions supplies the values through its secret store (§3.4). Any encryption-component tests should use fresh, throwaway keys generated for the run, never development keys. This does not imply that current account contact fields are encrypted; they follow the plain-text amendment. A check should reject embedded credentials in committed test-service definitions.

### 3.2 Cross-repo checkout

In regression implementation reference, check out the three subrepos with actions/checkout using a
**least-privilege, repo-scoped POLYREPO PAT** secret (read access to the three private subrepos):


Check out the test, frontend and backend repositories into the sibling locations expected by the combined tests. Use a credential limited to reading those repositories. Record the exact revisions so a test result can be traced to the versions exercised.


(Pin refs to sp-dev or the triggering SHA per the trigger model decided in review.)

### 3.3 Compose up → run suites

- Supply required credentials through the CI environment and generate fresh throwaway encryption-test keys where needed (§3.4). The service definitions read those supplied values rather than containing literal credentials.
- manage the required application services
  (reuse the runbook health checks).
- Seed the regression fixtures (the lab seed; honor INT-137c's isolation once that lands).
- backend black-box *IT band against localhost (the live-stack lab profile — the one excluded from
  the per-pull request ci-no-lab gate; **this lane is exactly where it must run**).
- Run the Playwright browser tests and retain their report as a CI artifact.

### 3.4 Secret management (Boss SEND-BACK #5)

Two distinct secret concerns, both via GitHub Actions secrets — **never committed**:
- **POLYREPO PAT** — the cross-repo checkout token. Least-privilege **fine-grained** PAT, read-only
  (Contents: Read) on exactly the three subrepos; never a classic all-repo token. Document purpose,
  scope, owner, and rotation cadence in [dev-vm-deploy implementation reference].
- **Application secrets for the continuous integration stack** — database/Redis passwords + signed session token keyed message authentication come from Actions secrets;
  the **Tink keyset is generated fresh per run** (ephemeral, throwaway test data only). The committed
  docker-compose ci implementation reference contains **no secret literals** — enforced by the §3.1 grep guard. This keeps
  the dev keyset (committed in the wrapper compose, owed for rotation pre-prod) out of continuous integration entirely.

---

## 4. Tests / Do D

- The lane brings the stack to healthy, runs both suites, uploads reports.
- A negative-control: a deliberately failing e2e (then reverted) proves the lane actually **fails the
  build** on a red suite — a guard that can't fail is a false green (SHARED-LESSONS, INT-064 lineage).
- **Do D: a green run observed on the GitHub Actions tab.** Per "verify, never trust," the lane is not
  done until the run is seen green — not "YAML validated, should work."

---

## 5. Access blocker — must be surfaced to the founder

This workstation has **no gh CLI and no GitHub Actions visibility**. Therefore:
- Creating/rotating the POLYREPO PAT repo secret,
- triggering the lane,
- and confirming the green run on the Actions tab

**require founder action or credentials provisioned here.** The spec is buildable (the YAML + compose
can be authored and pushed), but the §4 Do D — the seen-green run — cannot be satisfied from this
workstation. Treat the green-run confirmation as a hand-off item, not an in-session deliverable.

---

## 6. Ship

Author the e2e compose + regression implementation reference changes → /team-review → Boss → push to the e2e repo
(main) and the spec (sp-dev). The green-run confirmation is a tracked follow-up gated on §5 access.
Update the INT-064 tracker row only when the composed lane is **seen green** — not on push.
