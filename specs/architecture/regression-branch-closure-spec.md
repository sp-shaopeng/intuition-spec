# Regression Branch-Closure Spec — close the high-value untested branches

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

## Current requirements correction — 2026-09-05

This regression list must follow [M18 in the product requirements document](../PRD.md): applying without a published tutor listing is allowed when the profile and other current gates pass. B4 is corrected accordingly; do not restore TUTOR NO ACTIVE POST as a regression expectation. Other dated test counts/fixtures are historical.

This is a documentation correction. Dated implementation observations, code/database query language examples, screenshots, test
counts and HISTORICALLY MARKED IMPLEMENTED labels below are historical evidence, not verification of this correction. Where they
conflict with the effective requirements above, they are superseded; no source code or runtime action was
performed for this change.

## Historical technical record and retained detail

**Status:** Draft → team review → implement · **Date:** 2026-05-31
**Owner:** Regression Eng (Robin) + Tech Lead · **Source of truth:** service code
**Drives from:** reference/branch-coverage-matrix md (136 branches; ~78 untested).

## 1. Why + scope

The branch audit found ~78 untested service branches. This spec closes the **high- and
medium-consequence** ones as new/extended regression ITs (all test-only, no product change, black-box
vs the live dev stack, reg-band + self-created precursors). It deliberately does **not** chase every
symmetric * NOT FOUND 404 twin — those are listed as "lower value, optional".

**Excluded by design (unchanged):** all multi-factor authentication branches (#03 deferred), all rate-limit 429s (limiter
raised in dev). Not in scope.

## 2. Tranche A — Priority (the founder-flagged close-order). MUST build.

| Test (new) | IT class | Branch closed | Precursor | Assert |
|---|---|---|---|---|
| A1 | Interview Limit IT (new) | INTERVIEW LIMIT REACHED (422) | 1 reg tutee post + **4** active reg tutor posts; tutee requests on 3 → all 201; 4th → 422 | 3×201 then 422 error code is INTERVIEW LIMIT REACHED |
| A2 | Interview Limit IT | DUPLICATE INTERVIEW (422) | 1 tutee post + 1 tutor post; request same pair twice | 1st 201, 2nd 422 |
| A3 | Review Submit Journey IT (extend) | INVALID STATUS (422) | ACTIVE (not COMPLETED) match | review → 422 error code is INVALID STATUS |
| A4 | Review Submit Journey IT | DUPLICATE REVIEW (422) | COMPLETED match; same reviewer submits twice | 1st 201, 2nd 422 DUPLICATE REVIEW |
| A5 | Review Submit Journey IT | tutor-as-reviewer path (201) | COMPLETED match; the **tutor** actor reviews | 201, rating echoed |
| A6 | Admin Matching IT (extend) | TUTEE POST NOT ACTIVE (422) | ACTIVE tutor post + **PENDING** tutee post → create match | 422 (twin of existing tutor-not-active) |
| A7 | Admin Matching IT | MATCH NOT CANCELLABLE (422) | match → activate → pay→COMPLETED → cancel | 422 MATCH NOT CANCELLABLE |
| A8 | Post Lifecycle IT (new) | post FORBIDDEN edit/delete by non-owner (403) | owner A creates post; actor B (other reg user) PUT/DELETE it | 403 |
| A9 | Post Lifecycle IT | post INVALID STATE delete non-deletable (409) | create+approve → ACTIVE post; owner DELETEs | 409 INVALID STATE |
| A10 | Post Lifecycle IT | post edit happy (PUT 200) + close happy (DRAFT/REJECTED→CLOSED) | owner edits own; owner deletes a REJECTED post | 200 / 200 |
| A11 | Account Lifecycle IT (new) | change-password wrong-current (INVALID CREDENTIALS 401) | reg user logged in; change with wrong current | 401 |
| A12 | Account Lifecycle IT | change-password happy (200) + old-pw-rejected | reg user changes pw → login new 200, old 401 | 200 / 401 |
| A13 | Account Lifecycle IT | account deletion happy (200) + ACCOUNT DELETION REQUESTED/login-after (401) | reg user deletes self → login → denied | 200 then login 401 |
| A14 | Account Lifecycle IT | delete account wrong password (401) | wrong confirm Password | 401 |

## 3. Tranche B — Medium (admin + ownership guards). SHOULD build.

| Test | IT class | Branch | Assert |
|---|---|---|---|
| B1 | Admin User Mgmt IT (extend) | CANNOT REMOVE SUPER ADMIN (403) | super-admin tries to delete a super-admin target → 403 |
| B2 | Admin Post Approval IT (extend) | tutee-post ALREADY REVIEWED (409) | approve a tutee post twice (the tutee-post variant of the tested tutor-post one) |
| B3 | Job Applications IT/Applications IT (extend) | TUTEE POST NOT ACTIVE (422) apply to non-active job | 422 |
| B4 | Applications IT | M18 listing-optional application: complete tutor profile, no published listing, ACTIVE target job | Apply is allowed subject to the normal profile/quota/authorization gates; do not restore the removed TUTOR NO ACTIVE POST guard |
| B5 | Favourites IT (extend) | TUTOR POST NOT ACTIVE (422) favourite a PENDING post | 422 |

## 4. Tranche C — Lower value (symmetric 404/not-found twins). OPTIONAL, defer unless cheap.

* NOT FOUND on: match create bad tutor/tutee id, activate/cancel unknown match, favourite unknown
post, interview request bad posts, application withdraw unknown, review unknown match. These are
defensive 404s with a tested sibling; batch later if time permits. Listed in the matrix as ✗ but
**not required** by this spec.

## 5. Conventions (binding)

- Error checks inspect the code inside the response’s error information, rather than assuming it appears at the top level. A status-only assertion is sufficient only where the specific code is not part of the behaviour being verified.
- New precursor helper if needed: the new active tutor post operation ×N for the interview cap.
- COMPLETED-match helper already exists in Review Submit Journey IT completed Match Id — reuse/lift to base.
- Flake protocol (A1 of dev-workflow): reset ratelimit:* + failed login attempts before the gate run.
- Tags: headline/security cases @smoke; the rest @full.

## 6. Acceptance / Do D

- All Tranche A built + green; Tranche B built + green (or explicitly deferred with reason).
- Full backend regression suite green **×3 consecutive**, Spotless clean (lab-first gate, dev-workflow A1).
- branch-coverage-matrix md updated: flipped rows ✗→✓ with the owning IT method; recompute the
  tested/gap counts.
- DONE-DEV (no prod-affecting change). No new PROD-DEPLOY-OWED.

## 7. Out of scope / risks

- multi-factor authentication + rate-limit branches stay excluded (declared).
- Tranche C 404-twins deferred (low value).
- Shared-stack order-sensitivity mitigated by the flake protocol + ×3, not eliminated.
