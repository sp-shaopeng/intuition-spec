# 37 — Contact Reveal (post-assignment counterparty personal information)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

**Current flow contract — corrected 2026-09-05.** This describes the required behavior after approved amendments; implementation verification is separate.

## Required sequence

1. Authorize the caller as an assigned participant; the application programming interface’s permitted PENDING/ACTIVE/COMPLETED states remain distinct from frontend eligibility. Cancelled/nonparticipant access is not disclosed.
2. The frontend requests/renders contact only for ACTIVE/COMPLETED and hides match-PENDING. Approval, interview and selection alone do not release contact.
3. Read current plaintext fields and audit the reveal; old decrypt helpers and historical500 observations are not current contract requirements.

## Authority

[Current requirements C04](../CURRENT-REQUIREMENTS.md) · [int-125-batch-closeout-spec md](../architecture/int-125-batch-closeout-spec.md) · [plaintext-email-phone md](../architecture/plaintext-email-phone.md)


Historical implementation snapshot — superseded clauses are not current acceptance criteria

The original code-derived description below is retained for traceability. Its code line numbers, missing-feature observations and superseded branches describe the earlier snapshot. Implement against the required sequence and linked current specifications above.

# 37 — Contact Reveal (post-assignment counterparty personal information)

## Trigger

After a match exists (post-assignment), a **participant** (the match's tutor or tutee)
opens their match and requests the other party's real contact details. This is the
**only** endpoint that crosses the counterparty-personal information line, and only post-assignment
(M18 D3).

## Participants

- **Participant** (the match's TUTOR or TUTEE)
- **frontend** match dashboard / detail
- **Match Contact Controller** — read operation for matches id contact (the pre authorize operation)
- **Contact Reveal Service reveal Contact** (the transactional operation)
- **Match Repository, User Repository, Tutor Profile Repository, Tutee Profile Repository**

## Diagram


Implementation example omitted. This public edition retains the written requirements in this section.


## Notable details

- Controller: Match Contact Controller implementation reference. Service: Contact Reveal Service implementation reference.
- **Participant-only:** caller must be the match's tutor or tutee (50-54) else 403.
- **Revealable window:** PENDING (post-assign) / ACTIVE / COMPLETED reveal; CANCELLED or
  missing → **404** (deliberately indistinguishable — no status enumeration, 56-65).
- Returns the **counterparty's** plaintext name/phone/email (decrypted from the AES-SIV
  columns); the caller's own details are never the payload. This is the single sanctioned
  personal information-crossing surface.

## Failure modes

- 401 — anonymous.
- 403 FORBIDDEN — authenticated non-participant.
- 404 MATCH NOT FOUND — missing match OR not-yet/never-revealable (pre-assign, CANCELLED).
- **⚠️ Known defect INT-016 (Sev-1):** on the live dev stack this path can 500 with
  Lazy Initialization Exception (lazy Tutor Post/profile deref outside the session under
  open-in-view is no). Tracked in issues/README md + BATCH3-REMEDIATION-PLAN md (P1).

## Related flows

- [09-admin-match-creation md](09-admin-match-creation.md) / [10-admin-match-activation md](10-admin-match-activation.md) — how a match comes to exist
- [38-choose-tutor-selection md](38-choose-tutor-selection.md) — the parent-driven path into a match
- [27-encrypted-pii-write-and-lookup md](27-encrypted-pii-write-and-lookup.md) — how the plaintext is stored/decrypted
- [17-tutee-views-matches md](17-tutee-views-matches.md) / [18-tutor-views-matches md](18-tutor-views-matches.md)
