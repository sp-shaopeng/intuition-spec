# Sequence 33 — Profile edit

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authenticate and edit only the caller’s profile under canonical field/taxonomy rules. Contact changes go through012 confirmation, not direct profile mutation.
2. Public cards/showcase use public handle or its sanctioned humanized rendering; private display Name is not the public identity source.
3. Listing overrides do not synchronize back into the profile. Profile edits cannot change a locked agreed commission plan.

## Authority

[Current requirements C04](../CURRENT-REQUIREMENTS.md) · [requirements md](../012-tutor-apply-and-profile-ux/requirements.md) · [spec md](../026-s3s4-polish-batch/spec.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# Sequence 33 — Profile edit

**Module:** M7 Profile · **Actors:** Tutee, Tutor
**Endpoints:** GET/PUT tutee profile interface operation, GET/PUT tutor profile interface operation

## Preconditions
- User is authenticated; role matches the profile namespace (a tutor cannot touch
  /tutee/profile → 403, and vice versa).

## Flow


Implementation example omitted. This public edition retains the written requirements in this section.


## Postconditions
- Profile fields updated. Tutor edits to qualifications/display Name flow through to the
  public tutor card + showcase projection.

## Error cases
- 401 unauthenticated; 403 wrong role for the namespace; 400 validation.

## Related
- Account user interface note: [Private supporting material excluded (publication note)](../../docs/publication-notes.md)
- Availability/showcase consent: [34-tutor-availability-toggle md](34-tutor-availability-toggle.md)
- Reference: [backend-api md](../reference/backend-api.md) §M7.
