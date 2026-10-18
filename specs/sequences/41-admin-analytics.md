# 41 — Admin Analytics (M12)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authorize admin and validate the selected time range, including inverted-range rejection.
2. Interview conversion uses017’s completed subset of interviews requested in the period; numerator is a subset of denominator.
3. Agency revenue excludes unpaid debt. Average commission uses the definition recorded in017 and O02; a legacy count stub or “code is correct” statement does not settle the date cohort.
4. Prove posts and signups windows with recent,31–90-day and>90-day records under019/023.

## Authority

[Current requirements C02](../CURRENT-REQUIREMENTS.md) · [requirements md](../017-admin-analytics-and-financial-safety/requirements.md) · [spec md](../019-admin-money-safety-and-analytics/spec.md) · [spec md](../023-seed-data-variety/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 41 — Admin Analytics (M12)

## Trigger

An admin opens the analytics dashboard. A single endpoint returns platform-wide aggregate
metrics for an inclusive from, to date range (counts/funnels from database queries — not a
behavioural event pipeline; treat as a count-level stub, see status-briefing md §2.10).

## Participants

- **Admin**
- **frontend** admin analytics dashboard
- **Analytics Controller** — read operation for admin analytics?from&to (the pre authorize operation)
- **Analytics Service get Analytics**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Analytics Controller implementation reference; BASE PATH is admin analytics interface operation,
  DEFAULT LOOKBACK DAYS is 30.
- Both from/to optional; defaults to is today, from is to-30. 400 if the start date is after the end date.
- **Defence-in-depth authz:** the pre authorize operation on the class **and** the
  admin ** interface operation matcher in Security Config — either alone denies a non-admin.
- Count/funnel only — no behavioural event tracking; the dashboard is a stub, not product analytics.

## Failure modes

- 403 — non-ADMIN caller.
- 400 — from after to.

## Related flows

- [25-csrf-protected-write md](25-csrf-protected-write.md) — GET is read-only; admin writes elsewhere are cross-site request forgery protection-guarded
- [08-admin-approval md](08-admin-approval.md) — other admin-only surfaces under admin ** interface operation
