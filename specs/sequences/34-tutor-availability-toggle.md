# Sequence 34 — Tutor availability / showcase consent toggle

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate the tutor and update availability/featured consent under the existing predicates and cache invalidation contract.
2. Eligible public projections remain personal information-free and use public handle/humanized public identity, never the private display Name.
3. Revocation must follow the current consent/cache timing rules; a historical refresh observation is not a new unlimited delay allowance.

## Authority

[Current requirements C04](../CURRENT-REQUIREMENTS.md) · [assignment-fe-spec md](../architecture/assignment-fe-spec.md) · [spec md](../026-s3s4-polish-batch/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# Sequence 34 — Tutor availability / showcase consent toggle

**Module:** M14a Active Tutors showcase · **Actors:** Tutor
**Endpoint:** update operation for tutor profile featured-consent

Controls whether the tutor appears in the public pre-login Active-Tutors showcase strip.
The showcase requires BOTH the post being available AND the account featured consent is yes.

## Preconditions
- Tutor is authenticated.

## Flow


Implementation example omitted. This public edition retains the written requirements in this section.


## Postconditions
- featured consent updated. Showcase inclusion = post ACTIVE + available + consent is yes.
  Setting consent is no removes the tutor from the public strip.

## Error cases
- 401 unauthenticated; 403 non-tutor; 400 missing consent.

## Notes
- The showcase projection is personal information-free (display name + subjects/levels only; no contact, no
  applicant count). Consent is the tutor's control over public visibility.

## Related
- Showcase module: [../modules/landing-showcase/spec md](../modules/landing-showcase/spec.md)
- Regression: Showcase IT (degraded:false, personal information-free, lazy-collections serialize).
