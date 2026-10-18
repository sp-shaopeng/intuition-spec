# INT-045 — Admin Analytics Date-Range Selector

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** [017](../017-admin-analytics-and-financial-safety/requirements.md) settles the conversion cohort and forbids collected÷current-ACTIVE average; its exact average-commission date cohort still awaits the business-owner record. [019](../019-admin-money-safety-and-analytics/spec.md)/[026](../026-s3s4-polish-batch/spec.md) extend labels and acceptance. This document's older code/schema examples are historical where the cited amendment changes them; they do not approve a policy override.


**Status:** SPEC — pending team-review + Boss audit before implementation
**Date:** 2026-06-03
**Authors:** data-engineer (lead) · bff-specialist · frontend-expert · software-engineer
**Tracker:** [README implementation reference](../issues/README.md) — INT-045
**Next milestone:** /team-review → Boss → implementation sprint

---

## 1. Problem

read operation for admin analytics already accepts optional from/to query params (introduced in M12). The controller defaults them to the last 30 days and the service enforces a 366-day cap. However:

1. The frontend Analytics implementation reference already ships a date-range picker with presets (Last 7/30/90 days + Year to Date) and custom from/to inputs. The frontend is ahead of where this INT thinks it is.
2. Several metrics are **not scoped to the selected range** in the service layer — they are unconditional all-time counts. When the admin selects "Last 7 days", totals like total Tutee Posts, total Tutor Posts, match funnel total/pending/active/completed/cancelled, and all revenue figures are computed against the entire dataset, not the window. The frontend labels them "All time" or "Lifetime collected" — honest, but the inconsistency makes it hard to answer "how did we perform in the last 30 days?"
3. Key indexes are missing: users created at, matches matched at, interviews requested at, tutee posts created at (global, not owner-scoped), tutor posts created at (global), and commissions paid at — none are indexed in their relevant analytics-query shape. The daily-metrics loop issues one count By Created At Between per day per table — up to 366 × 5 = 1830 unindexed sequential scans.

**Founder requirement:** the dashboard must answer "how did the platform perform in a selected window?" clearly. The picker user interface exists; we need the metric scoping and the indexes to be correct.

---

## 2. Proposed Design

### 2.1 Schema changes — none

This is a read-only analytics change. No new tables, no new columns. The from/to parameters already exist in the controller. The work is:

- **Service:** make point-in-time metrics range-aware where meaningful (see §3).
- **Indexes:** add targeted BRIN/BTree indexes on the timestamp columns used by the analytics queries (see §5).
- **frontend:** clean up stat-card subtitles to be accurate given the scoping model (see §4).

Confirmation: no Liquibase migration is needed for the service or frontend changes. A Liquibase migration is needed for the new indexes (read-only, no structural change). Next free migration number is **0037**.

### 2.2 Service scoping changes (the data-engineer call)

The Analytics Service mixes two kinds of metrics. This spec formalises that distinction and corrects the inconsistencies:

| Concern | Rule |
|---|---|
| **Range-scoped** | "how many events happened in the window?" → always filtered by from Ts, to Ts |
| **Point-in-time** | "what is the platform's current state?" → unconditional counts; window is irrelevant |

The the build interview funnel operation and the build revenue summary operation methods currently take no timestamps, making them implicitly point-in-time. The spec calls this correct for certain fields (see §3) and wrong for others.

**the build interview funnel operation — use the017 requested-at cohort:**

The conversion denominator is every interview requested in the selected interval; the numerator is the subset of those same interviews whose current status is COMPLETED. The denominator is not the current REQUESTED-status count. Zero denominator yields0%; numerator is naturally a subset, with no clamp. Requested/Completed funnel tiles and rate use this same cohort per019/026. Additional status gauges may remain point-in-time only if clearly labelled and never substituted into that ratio.

**the build revenue summary operation — split into point-in-time balance + range-scoped period revenue:**

total Collected Sgd (all PAID commissions) and outstanding Sgd (all PENDING commissions) are correct as point-in-time totals. active Matches Count is a correct point-in-time gauge. The old avg Commission Per Match Sgd collected / active definition is superseded by 017: numerator and denominator must use the same approved commission/match population and never the unrelated current ACTIVE gauge. The exact average date cohort remains pending the owner's recorded decision in017; existing code does not settle it.

Add a new field to Revenue Summary: collected In Period Sgd — the sum of commissions whose paid at falls inside from Ts, to Ts. The existing sum Paid Commission Amount Between repository query already exists. Wire it in the build revenue summary operation.

**the build match funnel operation — already correct:**

total/pending/active/completed/cancelled are point-in-time state counts; matches Created In Period is already range-scoped via the count by matched at between operation. No change needed.

**the build user counts operation — already correct:**

total Users/total Tutees/total Tutors are point-in-time; new Signups In Period is already range-scoped. No change needed.

**the build posting funnel operation — already correct:**

total Tutee Posts/total Tutor Posts are point-in-time stock counts; pending Review/active/rejected are point-in-time status gauges; posts Created In Period is already range-scoped. No change needed.

### 2.3 Summary of service changes

1. Add collected In Period Sgd to Revenue Summary record.
2. Pass from Ts/to Ts into build Revenue Summary and populate it via the sum paid commission amount between operation.
3. Update Analytics Service Test for the new field.
4. Update the frontend Revenue Summary type and render the new card (§4.3).

No other service method changes.

---

## 3. Metric Scoping Table

| Metric | Field in data transfer object | Scope | Rationale |
|---|---|---|---|
| Total users | users total Users | Point-in-time | Cumulative registered users; the selected window does not change what "total" means |
| Total tutees | users total Tutees | Point-in-time | Same |
| Total tutors | users total Tutors | Point-in-time | Same |
| New signups | users new Signups In Period | **Range-scoped** (created at) | "How many signed up in the window?" — already implemented |
| Total tutee posts | posting total Tutee Posts | Point-in-time | Running stock |
| Total tutor posts | posting total Tutor Posts | Point-in-time | Running stock |
| Pending review | posting pending Review | Point-in-time | Current queue depth — actionable right now |
| Active posts | posting active | Point-in-time | Current live listings |
| Rejected posts | posting rejected | Point-in-time | Historical state count |
| Posts created | posting posts Created In Period | **Range-scoped** (created at) | Already implemented |
| Interviews requested | Current funnel denominator | Selected requested at cohort | Every interview requested in the period, regardless of current status |
| Interviews scheduled | interviews scheduled | Point-in-time | Status gauge |
| Interviews completed | Current funnel numerator | Same requested at cohort | Subset now COMPLETED |
| Interviews no-show | interviews no Show | Point-in-time | Status gauge |
| Interviews cancelled | interviews cancelled | Point-in-time | Status gauge |
| Conversion rate | interviews conversion Rate | Same selected requested at cohort | Numerator/denominator above;0% if empty |
| Match total | matches total | Point-in-time | Running count |
| Matches pending | matches pending | Point-in-time | Current state |
| Matches active | matches active | Point-in-time | Current active relationships |
| Matches completed | matches completed | Point-in-time | Historical count |
| Matches cancelled | matches cancelled | Point-in-time | Historical count |
| Matches created | matches matches Created In Period | **Range-scoped** (matched at) | Already implemented |
| Total collected (lifetime) | revenue total Collected Sgd | Point-in-time | Cumulative business revenue |
| Collected in period | revenue collected In Period Sgd | **Range-scoped** (paid at) | **NEW** — "how much commission did we collect in this window?" |
| Outstanding | revenue outstanding Sgd | Point-in-time | Current receivables |
| Active matches count | revenue active Matches Count | Point-in-time | Current billable base |
| Avg commission/match | revenue avg Commission Per Match Sgd | Exact date cohort pending017 owner record | Same population numerator/denominator; never current ACTIVE count |
| Daily signups | daily Metrics .new Signups | **Range-scoped** (per-day created at) | Already implemented |
| Daily interviews | daily Metrics .interviews Requested | **Range-scoped** (per-day requested at) | Already implemented |
| Daily matches | daily Metrics .matches Created | **Range-scoped** (per-day matched at) | Already implemented |
| Daily commission | daily Metrics .commission Paid Sgd | **Range-scoped** (per-day paid at) | Already implemented |

**Data-engineer note:** the earlier all-time-only recommendation is superseded by017. Excluding interviews requested before the window from BOTH numerator and denominator is intentional cohort consistency. Do not substitute completion-time counts or clamp an inconsistent percentage.

---

## 4. application programming interface Contract

### 4.1 Endpoint (unchanged)


The illustration for endpoint unchanged contains the following information:

- GET admin analytics interface operation.
- Authorization: ROLE ADMIN (cookie).


**Query parameters** (both optional):

| Param | Type | Default | Validation |
|---|---|---|---|
| from | YYYY-MM-DD | today - 30 days | Must be ≤ to; range ≤ 366 days |
| to | YYYY-MM-DD | today | Must be ≥ from |

Errors: 400 INVALID DATE RANGE if the start date is after the end date ; 422 INVALID DATE RANGE if range > 366 days. (Already implemented.)

### 4.2 Response shape change

The only breaking change to Analytics Response is adding collected In Period Sgd to Revenue Summary:


Implementation example omitted. This public edition retains the written requirements in this section.


All existing fields are preserved. The frontend must add collected In Period Sgd to the Revenue Summary TypeScript type (see §4.4).

### 4.3 frontend-facing service contract — no magic strings

The analytics endpoint is already registered in application programming interface ROUTES:


Implementation example omitted. This public edition retains the written requirements in this section.


The frontend already calls it via application programming interface ROUTES admin.analytics with the start and end of the selected date range. No change needed to api Routes implementation reference.

The query key is already parameterised:


Implementation example omitted. This public edition retains the written requirements in this section.


No new routes or query keys are needed.

### 4.4 TypeScript type update


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 5. Frontend Date-Range Picker

### 5.1 Current state (already correct, minor fixups needed)

Analytics implementation reference already implements:

- **Quick-range presets:** Last 7, Last 30, Last 90 days + Year to date (the YTD preset exceeds the founder's required 90-day max but is useful and within the 366-day server cap; retain it).
- **Custom from/to:** two native date input elements with ARIA labels.
- **Client-side inversion guard:** is Range Inverted prevents a round-trip for an obviously wrong range.
- **Query disabled** when either date is empty or range is inverted.
- **Error and loading states.**

This is the correct lightweight approach. No third-party date-picker library is needed — the native date input is accessible, zero-bundle-cost, and sufficient for an admin-only screen. Do not introduce a heavy dependency (e g. react-datepicker, date-fns) for this feature.

### 5.2 Required frontend changes

**a) Add the new Revenue card:**

In the Revenue section, add a Stat Card for collected In Period Sgd:


Implementation example omitted. This public edition retains the written requirements in this section.


**b) Fix stat-card subtitles for point-in-time metrics:**

Current labels like "All time" (on total Tutee Posts) and "Lifetime collected" (on total Collected Sgd) are accurate but should be consistent. Proposed standard language for point-in-time cards: subtitle All time. Range-scoped cards: subtitle In selected period (already used for posts Created In Period and new Signups In Period).

Apply this standard to total Collected Sgd (change "Lifetime collected" → subtitle All time), outstanding Sgd (change "Awaiting payment" → keep as-is, it's accurate without period context), and active Matches Count / avg Commission Per Match Sgd (keep current subtitles, they are accurate).

**c) INT-049 design token alignment:**

The picker controls (date input, preset buttons) should use the same border/focus/text tokens defined in the INT-049 redesign spec (intuition-spec/specs/uiux/redesign-modern-readable-spec md). Specifically:

- Border: border-cream-border (already used — correct).
- Focus ring: focus:border-navy (already used — correct).
- Text: text-ink for input values, text-sand for labels (already used — correct).
- Preset buttons: border border-cream-border rounded with hover:underline — these are lightweight; if INT-049 introduces a standard ghost-button variant, migrate at that time (out of scope for INT-045).

No immediate changes are needed for design token compliance; the current implementation already aligns. Note this alignment as a verification item in the acceptance criteria.

**d) No heavy dependency:**

Do not add react-datepicker, @mui/x-date-pickers, or any other date-picker library. The native date input is sufficient, accessible, and has zero bundle cost.

---

## 6. Performance

### 6.1 Index audit

The analytics service runs the following time-bounded queries:

| Table | Column queried | Existing index covers? | Action |
|---|---|---|---|
| users | created at between | idx users active covers id when deleted at IS no value — does NOT cover created at range scans | **Add** idx users created at |
| tutee posts | created at between | idx tutee posts owner covers tutee id, created at DESC — owner-scoped, not useful for global range scan | **Add** idx tutee posts created at |
| tutor posts | created at between | idx tutor posts owner covers tutor id, created at DESC — same issue | **Add** idx tutor posts created at |
| interviews | requested at between | No index on requested at | **Add** idx interviews requested at |
| matches | matched at between | No index on matched at ; the existing status and creator indexes do not support this date filter | **Add** idx matches matched at |
| commissions | paid at between | No index on paid at ; the existing invoice-status index does not support this date-range lookup | **Add** idx commissions paid at |

### 6.2 Index type

Use plain **BTree indexes** (PostgreSQL default) on TIMESTAMPTZ columns. BRIN is smaller but only pays off for very large append-only tables (millions of rows, monotonically written). At InTuition's current scale, BTree is the right default and is the pattern already established for audit log created at.

Sort order: DESC on all timestamp indexes — the analytics service queries BETWEEN from Ts AND to Ts which PostgreSQL can satisfy in either direction, but DESC matches the natural "most-recent first" read pattern used in listing queries and avoids a future sort operator in adjacent contexts.

### 6.3 Daily-metrics loop

The existing implementation iterates day by day (up to 366 iterations), each firing a constant number of count By Created At Between calls. With the indexes added in §6.1, each index seek becomes an index-range scan (O using matching rows rather than O using table size). At current data volumes this is acceptable. If the table grows to hundreds of thousands of rows and latency becomes a problem, a future optimisation would be to replace the per-day loop with a single the date trunc operation grouped by query per table — but this is out of scope here.

### 6.4 Liquibase migration: 0037

Migration 0037-analytics-timestamp-indexes implementation reference adds the six indexes. Use SET LOCAL lock timeout 3s per project convention. All indexes are CONCURRENTLY-safe (Liquibase XML create Index with no concurrent flag is fine in dev; if a concurrent build is needed for prod zero-downtime, note it in the runbook). Example changeset structure:


Implementation example omitted. This public edition retains the written requirements in this section.


**Verify with EXPLAIN ANALYZE, BUFFERS** after migration that count By Created At Between queries use the new indexes (look for Index Scan on the new index name, not Seq Scan).

---

## 7. Acceptance Criteria

| ID | Criterion |
|---|---|
| AC-01 | read operation for admin analytics?from is 2026-05-01&to is 2026-05-31 returns 200 with revenue collected In Period Sgd = sum of commissions with paid at in May 2026 |
| AC-02 | read operation for admin analytics (no params) defaults to last-30-days and includes revenue collected In Period Sgd |
| AC-03 | Preset "Last 7 days" sets the date inputs and triggers a fresh query; results reflect the 7-day window |
| AC-04 | Preset "Last 30 days" and "Last 90 days" work correctly |
| AC-05 | Custom from/to: entering a future to date or a wide range (≤ 366 days) succeeds; > 366 days returns 422 |
| AC-06 | Inverted range (start date after end date) shows the "End date must be on or after start date" alert and suppresses the query |
| AC-07 | Point-in-time metrics (total users, total posts, match funnel totals, revenue totals) return the same value regardless of the selected date range |
| AC-08 | Range-scoped metrics (new Signups In Period, posts Created In Period, matches Created In Period, collected In Period Sgd, all daily metrics) change correctly when the range changes |
| AC-09 | EXPLAIN ANALYZE on any count By Created At Between/count By Matched At Between/count By Requested At Between/sum Paid Commission Amount Between query shows an Index Scan using the new indexes, not a Seq Scan |
| AC-10 | the documented operational check passes at ≥ 97.5% lines and ≥ 97.5% branches (JaCoCo) after service changes |
| AC-11 | the documented operational check passes at ≥ 97.5% lines and ≥ 88% branches (Vitest) after frontend changes |
| AC-12 | No raw string literal for the analytics URL path outside application programming interface ROUTES admin.analytics |
| AC-13 | frontend date inputs, preset buttons, and stat cards use existing design tokens (border-cream-border, text-navy, text-ink, text-sand) — no new CSS variables introduced |
| AC-14 | No third-party date-picker library added to package implementation reference |

---

## 8. Test Plan

### 8.1 Backend unit tests (Analytics Service Test)

| Test | What it asserts |
|---|---|
| build Revenue Summary includes Collected In Period | With two PAID commissions — one paid at inside the range, one outside — collected In Period Sgd equals the inside one's amount only |
| build Revenue Summary collected In Period zero When None In Range | No PAID commissions in range → collected In Period Sgd equals 0 (null-safe) |
| get Analytics response Includes Collected In Period | Integration-level: the get analytics operation returns a non-null Revenue Summary with a collected In Period Sgd field |
| Existing tests | All existing Analytics Service Test cases pass unchanged |

### 8.2 Backend controller / IT

Analytics Controller Test does not need new cases beyond confirming the new field is serialised. If an existing IT (Analytics IT or equivalent) exercises the full endpoint, confirm revenue collected In Period Sgd is present in the JSON response body.

### 8.3 Frontend unit tests (Vitest)

Existing Analytics test implementation reference (or equivalent) mocks Analytics Response. Add:

| Test | What it asserts |
|---|---|
| renders collected In Period Sgd card | Given mock data with collected In Period Sgd: 500, a "Collected in Period" card renders with the formatted SGD value |
| preset last7 sets correct date range | Clicking "Last 7 days" sets from to 6 days before today (Coordinated Universal Time) and to to today |
| preset ytd sets correct date range | Clicking "Year to date" sets from to Jan 1 of this year and to to today |
| inverted range shows alert suppresses query | Setting from after to shows the alert div and does not call the application programming interface |
| empty from does not fire query | While editing (empty from), the prior result remains visible and no new query fires |

Existing tests for the picker presets (range For Quick) if present are updated to include the new collected In Period Sgd field in mock payloads.

### 8.4 DBA verification (post-migration)

After running migration 0037 on the dev stack:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 9. Open Decisions

| # | Decision | Options | Recommendation |
|---|---|---|---|
| OD-1 | Interview funnel cohort | **Resolved by 017**: completed among interviews requested in the selected period; rejected alternatives remain rejected. | Apply017 and019/026 labels; no all-time-only default. |
| OD-2 | **Revenue: should total Collected Sgd be deprecated in favour of collected In Period Sgd only?** | (A) Keep both (all-time + period-scoped, two cards); (B) Drop all-time, show only period | Recommend (A). Both are useful. All-time total is a business health metric; period revenue is a trend metric. Two cards side-by-side is the right admin user experience |
| OD-3 | **Index type: BTree vs BRIN for timestamp columns?** | BTree (default) vs BRIN (smaller, monotonic-only) | BTree per §6.2. Revisit only if a specific table grows past ~1M rows and BRIN savings are warranted |
| OD-4 | **Daily-metrics loop: per-day queries vs single grouped by per table?** | Per-day loop (current) vs bulk grouped by then pivot in application code | Per-day loop is correct and readable at current scale. Add a note in build Daily Metrics Javadoc to revisit at >100k rows per table |
| OD-5 | **YTD preset: keep or remove?** | Keep (useful, within 366-day cap); remove (founder spec said last 7/30/90 days + custom only) | Keep. YTD is already implemented, does not violate any constraint, and is a standard analytics preset. No work to remove it; document it in the frontend |

---

## 10. Out of Scope

- Charting / visualisation (the existing bar-chart table is retained as-is).
- CSV/PDF export of analytics data.
- Per-subject or per-level analytics breakdown.
- Notifications or alerts on metric thresholds.
- Interview funnel range-scoped metrics (deferred to OD-1 above).
- Frontend caching strategy beyond the existing Tan Stack Query range-keyed cache (expiry period defaults are sufficient for an admin-only screen).
