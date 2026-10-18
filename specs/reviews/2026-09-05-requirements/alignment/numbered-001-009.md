# Specs001–009 requirement alignment audit

> Reading note: [Current requirements](../../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../../docs/publication-notes.md#history-and-review-roles).

> **Pre-correction audit snapshot** — reviewed source at REDACTED identifier on 2026-09-05. See the [correction report](../corrections.md) and [current requirements](../../../CURRENT-REQUIREMENTS.md) for the revised contracts. These verdicts and quoted clauses describe the earlier baseline. Old line numbers remain in paired JSON evidence; local file links below now navigate without stale line positions. This is not a current implementation verdict.

All36primary documents were read; folded changelogs/sign-offs override explicitly named earlier alternatives. Requirements omissions are partial coverage, not proof of implementation failure. Historical evidence cited below was read as documents, not rerun.

Coverage: 9 folders,36primary documents,80 named REQs. No runtime or implementation verdicts.

Verdicts: aligned = traced consistent behavior; partial = incomplete/weak coverage or unpropagated settled amendment; conflict = incompatible normative clauses; authority-gap = approval not established. Stale status prose alone never makes behavior conflicting.

## 001 — conflict

Legal consent behavior is largely aligned; policy-version ownership/payload contract needs reconciliation.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|aligned|Required consent booleans and server timestamps trace through persistence/schema tasks.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-002|aligned|Optional marketing false/null is preserved; master marketing AND channel opt-in is explicitly settled, with transactional messages unaffected.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-003|aligned|Unchecked required boxes block registration/advance and show inline errors.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-004|aligned|Four exact consent texts, links and granular purpose list are represented; 390px a11y confirmation is a tracked residual.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-005|aligned|Tutee request requires flags; service persists server-set timestamps.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-006|aligned|Tutor draft carry, pre-commission validation, and Commission payload are explicitly traced. 002 later adds optional capture between the account and commission pages without removing this consent obligation.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-007|aligned|0058 specifies defaults, timestamp CHECKs, lock timeout and version columns.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-008|aligned|400 VALIDATION ERROR through bean validation is explicitly settled; no second CONSENT REQUIRED mechanism.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-009|aligned|Sixteen-section Terms rewrite and retained contact address trace to tasks. Exact GH #200 source copy is external and was not supplied/read in this audit.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-010|aligned|Ten-section Privacy rewrite and retained DPO address trace to tasks. Exact GH #199 source copy is external and was not supplied/read.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-011|aligned|Remove address only from Footer, preserve name/UEN.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-012|aligned|Remove address only from Contact, preserve name/UEN.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-013|aligned|Retain address on both legal pages; surgical removal is explicit.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-014|aligned|CONSENT GIVEN audit is in the registration transaction and contains flags/versions.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|
|REQ-015|conflict|Conflict: client sends only booleans in design, but tasks require policy versions in payload; source-of-truth ownership and displayed-version versus persisted-version mismatch are not resolved.|[requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) · [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) · [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md)|

### 001-F1 — Policy-version contract has two incompatible ownership descriptions

Classification: **behavioral-contradiction**.

- [design md](../../../001-legal-and-compliance-consent-and-policy-pages/design.md) — “the client sends only booleans”
- [tasks md](../../../001-legal-and-compliance-consent-and-policy-pages/tasks.md) — “network panel shows the 4 booleans + versions in the POST body”
- [requirements md](../../../001-legal-and-compliance-consent-and-policy-pages/requirements.md) — “record the Terms and Privacy policy versions the user consented to ... single source-of-truth constant”

Proposal: define displayed version identifiers in the request, authoritative current versions on the server, and reject/reconfirm a mismatch; or explicitly document generated shared constants and atomic deployment. Do not silently infer either from shipped code.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full)

## 002 — conflict

Optional onboarding and fine-grained tutor availability are aligned after explicit amendments; prior-listing profile prefill crosses the canonical-profile boundary without a rule.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|aligned|1-hour cells are persisted as fine-grained tutor-listing slots; 009 later intentionally transposes shared grid layout.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-002|aligned|2-hour selection expands into contiguous 60-minute slots; mobile default 2h remains intentional.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-003|aligned|Junction rows preserve exact tutor availability. Original 0058 identifier is stale; shipped amendment pins 0059/0059b.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-004|partial|New slot-list round-trip and legacy compatibility are designed; tighten the stale alternative allowing legacy bitmask to be frozen rather than synchronized.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-005|aligned|frontend and backend canonical vocabularies are explicitly one source on each side, not an impossible single cross-language source.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-006|aligned|Exact 08–12 / 12–18 / 18–22 backfill mapping and bit decode are binding; migration references lag the shipped amendment.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-007|aligned|Optional multi-MRT junction/list application programming interface is designed, legacy field retained; shipped 0060 supersedes stale 0059 labels.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-008b|aligned|Optional new capture page, expanded draft and registration persistence explicitly reverse INT-043 only for optional capture; commission acknowledgement remains mandatory.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-008c|aligned|Omitting expanded fields must still register; completion is gated later at applying, not signup.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-008|conflict|Broad prior-listing-save prefill promise lacks design source/precedence and conflicts with parent profile-canonical/no-sync ownership; onboarding/profile persistence path is aligned.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-009|aligned|Bad day/hour/duration payloads rejected without persistence; separate validation from visual display window.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-010|aligned|Handle suggestion provided by GET, editable, with server validation; later 026 refines collision outcome.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-011|aligned|Unsafe/overlength/colliding handle is rejected without persistence. Later 026 explicitly permits autosuffix or clear inline collision; do not reopen that approved flexibility.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|
|REQ-012|aligned|Verification of multiple subject-level pairs, no claimed numeric cap; later 020/025 coherence rules refine valid pairs.|[requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) · [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) · [tasks md](../../../002-tutor-profile-and-onboarding-ux/tasks.md)|

### 002-F1 — Profile prefill promises prior listing values without ownership/precedence

Classification: **requirement-parent-conflict**.

- [requirements md](../../../002-tutor-profile-and-onboarding-ux/requirements.md) — “all ... must be pre-filled from whatever was persisted at onboarding ... or by any prior profile/listing save”
- [product requirements document md](../../../PRD.md) — “profile-canonical + listing-overrides-when-attached (no sync job)”
- [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) — “The frontend prefills ALL of these on the profile page from the GET response”

Proposal: limit routine profile prefill to persisted profile/onboarding and one-time documented backfill; separately design explicit user-controlled copy from a listing if desired.

### 002-F2 — Legacy availability compatibility contains an obsolete frozen alternative

Classification: **stale-design-alternative**.

- [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) — “kept in sync or frozen at migrated value”
- [design md](../../../002-tutor-profile-and-onboarding-ux/design.md) — “legacy columns kept and kept in sync”

The specific rollout rule resolves the choice to synchronization. Remove frozen alternative and update0058/0059 references using spec changelog; no new product approval needed.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full)

## 003 — conflict

Reset behavior and error mapping align; strict anti-enumeration wording exceeds the explicitly accepted residual. Production one-time passcode policy is superseded by027.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|partial|Reproduce before fixing and fill hypothesis table are aligned. Required raw database/Redis snapshots were replaced by behaviorally inferred state in task closure without formal evidence-waiver trace.|[requirements md](../../../003-password-reset-failure-fix/requirements.md) · [design md](../../../003-password-reset-failure-fix/design.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md)|
|REQ-002|aligned|Reset updates password, consumes one-time passcode, clears forced-change, permits new-password login and rejects old password.|[requirements md](../../../003-password-reset-failure-fix/requirements.md) · [design md](../../../003-password-reset-failure-fix/design.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md)|
|REQ-003|conflict|Strict response/timing non-enumeration promise conflicts with the design/reviewer accepted response-sequence oracle. Narrow the promised invariant or track closure; the exception is recorded, not a hidden implementation discovery.|[requirements md](../../../003-password-reset-failure-fix/requirements.md) · [design md](../../../003-password-reset-failure-fix/design.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md)|
|REQ-004|aligned|RED→GREEN test discipline and unconditional frontend gate were folded into the approved spec; earlier conditional wording is historical.|[requirements md](../../../003-password-reset-failure-fix/requirements.md) · [design md](../../../003-password-reset-failure-fix/design.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md)|
|REQ-005|aligned|D8 deliberately superseded local-only logging. Later 027 REQ-A0-5 explicitly requires production OFF before cutover; do not treat original prod logging as the current launch policy.|[requirements md](../../../003-password-reset-failure-fix/requirements.md) · [design md](../../../003-password-reset-failure-fix/design.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md) · [tasks md](../../../003-password-reset-failure-fix/tasks.md)|

### 003-F1 — Non-enumeration guarantee exceeds recorded accepted boundary

Classification: **recorded-exception-not-folded**.

- [requirements md](../../../003-password-reset-failure-fix/requirements.md) — “neither response nor timing reveals whether the account exists”
- [design md](../../../003-password-reset-failure-fix/design.md) — “OTP ATTEMPTS EXCEEDED is observable only for an account that holds a live reset token ... a soft existence/activity oracle”
- [Private supporting material excluded (publication note)](../../../../docs/publication-notes.md) — “KNOWN, accepted ... out of scope for issue #193”

Make the existing reviewed exception explicit in the normative requirement or open a security requirement to remove it; do not claim strict response-sequence non-enumeration is already proven.

### 003-F2 — Production one-time passcode logging has an explicit later launch override

Classification: **deliberate-supersession**.

- [requirements md](../../../003-password-reset-failure-fix/requirements.md) — “ALL profiles INCLUDING PROD ... REVISIT (turn OFF) before public launch”
- [spec md](../../../027-aws-production-deployment/spec.md) — “OTP DEBUG LOG must be false in the prod profile/task def”

027 governs production cutover. Preserve the local operational history; link003 to027 rather than reopening the D8 choice.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full)

## 004 — conflict

Refresh lifecycle/security rules are detailed;429 cookie handling contradicts itself and grace response semantics need tightening. Logout correction is explicitly authorized.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|aligned|1800-second access signed session token/cookie lifetime is pinned, including fallback guard.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-002|aligned|Public-at-filter refresh endpoint retains cross-site request forgery protection, sets new credentials on normal rotation.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-003|partial|CAS, grace and revoke-before-401 are specified. Strict successor reissue is weakened to plain 200 in AC/design; hash-only storage leaves successor unavailable if original rotation response is lost. Clarify grace response/recovery contract.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-004|aligned|Block/reset epoch  is at most  boundary, missing-epoch fail-safe and no mint are explicit.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-005|partial|Spec v1.4 explicitly supersedes cookie-dependent family revoke with ALL user families on logout/delete; design/task index should adopt that settled correction.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-006|aligned|Tutor/tutee credential entrypoints receive scoped host-only refresh cookie; admin exclusion is explicit.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-007|aligned|7-day idle/30-day absolute and absolute-before-rotation are explicit.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-008|aligned|Idle-expired or missing family rejected; INVALID versus EXPIRED explicitly allowed.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-009|aligned|Single-flight, one retry, refresh-before-logout and no recursion are fully traced; unchecked frontend tasks are stale status, not evidence of absence.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-010|aligned|Admin stays stateful sliding1800 and receives no refresh token; defined refresh rejection is pinned.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|
|REQ-011|conflict|Direct contradiction: rate-limit evidence/test says no Set-Cookie, but controller task/design says clear BOTH cookies on every failure including429.|[requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) · [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) · [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md)|

### 004-F1 — Rate-limited refresh cannot both preserve and clear cookies

Classification: **behavioral-contradiction**.

- [requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) — “asserting no Set-Cookie”
- [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) — “in all cases ... both cookies are cleared on rejection”
- [tasks md](../../../004-auth-session-hardening-30min-ttl-and-refresh/tasks.md) — “on any failure clear BOTH cookies + Auth Error Code 401/403/429 ... over-limit ... no Set-Cookie”

Proposal should not revoke or clear a healthy session; return retry After with no Set-Cookie. Enumerate terminal401/403 versus temporary429 explicitly and adjust frontend recovery contract.

### 004-F2 — Grace duplicate has two response contracts

Classification: **partial-contract**.

- [requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) — “returns the already-minted successor”
- [requirements md](../../../004-auth-session-hardening-30min-ttl-and-refresh/requirements.md) — “receives the same successor R2 (or a 200)”
- [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) — “stores only SHA-256(secret) ... NEVER the raw secret”
- [design md](../../../004-auth-session-hardening-30min-ttl-and-refresh/design.md) — “idempotent reissue or a plain 200 with the standing cookie”

Plain200 is explicitly allowed, so this is not a proven violation. Choose a single contract and specify recovery when the winning rotation response is lost before R2 is stored.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full)

## 005 — conflict

Deferred testimonial build is intentional; allowed founder quotes and personal information-safe attribution are not supported by the source design as written.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|aligned|Verified privacy glyph/copy and issue closure; later021 changes privacy copy, an explicit newer copy requirement rather than instruction to restore always-private.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|
|REQ-002|aligned|Adoption metric expressly deferred; no missing feature inferred.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|
|REQ-003|partial|Placement, semantic cards and no-carousel are designed, but explicit 1280 2–3-up/390 single-column dimensions remain in requirements rather than concrete design recipe.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|
|REQ-004|partial|Build is deliberately blocked on real content/consent; not a delivery defect. Founder-quote alternative needs source contract resolution with REQ005.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|
|REQ-005|conflict|Conflict: mandatory source AC only permits M11 review rows while REQ004/007 permit founder-supplied quotes. Source option A/B is deliberately pending content-unblock; no decision inferred.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|
|REQ-006|aligned|Populated reserved-footprint versus zero-item absent behavior is explicitly resolved by v1.1; stale coming-soon alternatives should be removed.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|
|REQ-007|aligned|Content/consent is an explicit blocker; founder quote or approved real review ID allowed.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|
|REQ-008|partial|Consent and attribution requirements are clear, but source A/B design only transports Review Response and does not specify consented alias/masking projection.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|
|REQ-009|partial|v1.1 expressly chooses zero-item absent/no-box, superseding earlier optional coming-soon prose still left in this REQ.|[requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) · [design md](../../../005-landing-page-trust-and-social-proof/design.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md) · [tasks md](../../../005-landing-page-trust-and-social-proof/tasks.md)|

### 005-F1 — Founder quotes are allowed but source design only accepts M11 rows

Classification: **behavioral-contradiction**.

- [requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) — “real M11 reviews curated, or founder quotes with consent”
- [requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) — “one of ... featured ... real reviews ... or ... real review IDs ... each item resolves to an existing review row”
- [design md](../../../005-landing-page-trust-and-social-proof/design.md) — “Two source options ... every card maps to a real review row”

Proposal: add a consented quote source variant (optional rating, consent reference, approved attribution), or narrow all REQs to M11 content. Pending source A/B decision is legitimate deferred scope.

### 005-F2 — Attribution protection lacks a designed projection

Classification: **partial-coverage**.

- [requirements md](../../../005-landing-page-trust-and-social-proof/requirements.md) — “no exact full name unless explicitly consented ... Verified parent / initials only”
- [design md](../../../005-landing-page-trust-and-social-proof/design.md) — “Review Response ... reviewer Display Name ... strip fetches ... renders them”

Specify a consented display-attribution field and rendering rule separate from raw reviewer Display Name, and how withdrawn consent suppresses a card. Withdrawal behavior is a proposal, not an existing requirement.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full)

## 006 — partial

Authorized reminder content and OFF/readiness gates align;007 supersedes the sweep/flag mechanism. Some settled decisions remain only in ship prose, and the required frontend consent-copy update was not carried into006’s backend-only design.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|aligned|Link-gated two-party schedule notification and Singapore time ordered parameters align. Exact Singapore time format is settled in spec status/changelog, despite stale Open Q1.|[requirements md](../../../006-whatsapp-interview-reminders/requirements.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md)|
|REQ-002|aligned|Nullable ordered body Params travels end-to-end, legacy constructor/path retained; no-body Params logging condition is folded in spec.|[requirements md](../../../006-whatsapp-interview-reminders/requirements.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md)|
|REQ-003|partial|Sweep/flag mechanism explicitly superseded by007; user-facing >24h reminder remains.  is at most 24h confirmation-only is settled in006spec37 and007, but old006REQUIREMENT AC/task recipes were not updated.|[requirements md](../../../006-whatsapp-interview-reminders/requirements.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md)|
|REQ-004|aligned|0061 add/default/index is historical migration, deliberately followed by007/0063 drop. No live requirement to reintroduce flag.|[requirements md](../../../006-whatsapp-interview-reminders/requirements.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md)|
|REQ-005|partial|Reschedule re-arm and same-time guard are original behavior; scheduler cancellation/re-registration in007 replaces flag mechanism;  is at most 24h exception governs changed-time bookings.|[requirements md](../../../006-whatsapp-interview-reminders/requirements.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md)|
|REQ-006|partial|Reminder event/config tasks align with the authorized feature and WhatsApp stays OFF. Cross-document coverage gap: INT082 requires event expansion to ship with updated frontend consent copy and copy-pinning test, but006 design explicitly says no frontend change. Carry forward that existing condition before enablement; this is not a new consent policy or a current-delivery violation.|[requirements md](../../../006-whatsapp-interview-reminders/requirements.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [int-082-whatsapp-otp-and-notifications-spec md](../../../architecture/int-082-whatsapp-otp-and-notifications-spec.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [requirements md](../../../006-whatsapp-interview-reminders/requirements.md)|
|REQ-007|aligned|Three exact template definitions/parameter order align. one-time passcode re-registration scope is explicitly settled in spec40–41; Meta approval remains activation evidence, not dormant-build evidence.|[requirements md](../../../006-whatsapp-interview-reminders/requirements.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md)|
|REQ-008|aligned|OFF means enqueue/drop, no send; recipient eligibility and paramless backcompat retained. Never infer activation from HISTORICALLY MARKED IMPLEMENTED.|[requirements md](../../../006-whatsapp-interview-reminders/requirements.md) · [design md](../../../006-whatsapp-interview-reminders/design.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) · [tasks md](../../../006-whatsapp-interview-reminders/tasks.md)|

### 006-F1 — Approved short-notice exception is absent from old sweep recipes

Classification: **deliberate-supersession-not-folded**.

- [spec md](../../../006-whatsapp-interview-reminders/spec.md) — “a booking made ≤24h out gets the schedule confirmation only; reminder sent is set true at schedule time”
- [tasks md](../../../006-whatsapp-interview-reminders/tasks.md) — “when scheduled At changes, set reminder Sent is no ... scheduled at ∈ (now, now+24h]”
- [requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) — “Scheduling  is at most 24h out registers NO reminder job”

The decision is settled and007 replaces the mechanism. Annotate obsolete006 recipes, including REQ005 flag semantics, with007; do not reopen short-notice reminder policy.


### 006-F2 — Reminder expansion omits the existing consent-copy propagation requirement

Classification: **cross-document-partial-coverage**.

- [int-082-whatsapp-otp-and-notifications-spec md](../../../architecture/int-082-whatsapp-otp-and-notifications-spec.md) — “Re-expansion requires founder cost approval + frontend consent-copy update; config + copy ship together, never events config alone.”
- [design md](../../../006-whatsapp-interview-reminders/design.md) — “No new web request protocol endpoint and no frontend change.”
- [requirements md](../../../006-whatsapp-interview-reminders/requirements.md) — “INTERVIEW REMINDER must be added to the delivered-events config.”

The reminder feature is authorized and remains OFF. Carry the already-required event-specific consent copy and copy-pinning acceptance coverage into006/current activation work; do not reopen the approved reminder policy or infer present delivery. Same finding as AUX1-F01.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] 2.md (full25line unlinked template stub; not normative)


## 007 — conflict

Scheduler registration/migration align, but once-only and reschedule-safe guarantees exceed the documented design; crash dedup is an explicit pre-enablement condition.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|aligned|Dependency, Liquibase-owned table and initializer-off align;10s/5threads and pinned database structure definitions are explicitly approved in changelog.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-002|aligned|Bean+handler registration and wrapper worked example align; v1.2 explicitly removes unused reschedule wrapper method, a settled simplification.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-003|conflict|Conflict: never-fires-twice/once-only guarantee contradicts recorded crash gap between outbox commit and scheduler row deletion; durable delivery dedup remains a mandatory pre-enablement condition.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-004|partial|Partial: headline promises zero sends for rescheduled interviews, but guard/AC only detect status change or removed link. No time/generation guard covers a stale job after a reschedule that remains linked/SCHEDULED.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-005|partial|Partial: every transition must remove/move scheduled row, but v1.2 catches currently-executing exceptions; no reconciled contract for re-registering the new time during this race.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-006|partial|Schema removal deliberately supersedes006; once-only claim attached to row deletion inherits REQ003 caveat.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-007|aligned|Recurring expiry/showcase preserve business logic, cadence and per-pool isolation.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-008|partial|Metrics/retry remain required; founder R4 expressly defers alert rule and fixes4 retries. Earlier mandatory-alert clauses/tasks are stale superseded instructions, not unresolved policy.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-009|aligned|Shed Lock removal plus retained continuous poller scheduling and single ownership align.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|
|REQ-010|partial|Parameter/gating/dormancy parity is clear; founder explicitly accepts pre-cutover misses (no backfill), narrowing broad unchanged-user-behavior claim. Crash duplicates are separately an unresolved enablement condition.|[requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) · [design md](../../../007-generic-scheduler-db-scheduler/design.md) · [tasks md](../../../007-generic-scheduler-db-scheduler/tasks.md)|

### 007-F1 — Once-only delivery promise conflicts with recorded crash window

Classification: **behavioral-contradiction-with-launch-condition**.

- [requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) — “scheduled tasks row is consumed ... so it never fires twice”
- [design md](../../../007-generic-scheduler-db-scheduler/design.md) — “db-scheduler guarantees once-only execution”
- [spec md](../../../007-generic-scheduler-db-scheduler/spec.md) — “crash ... between the handler’s outbox-commit and db-scheduler’s row-delete can re-fire ... Must add a delivery-done dedup key before WhatsApp enablement”

Add the explicitly required durable delivery-done dedup requirement/design/task and remove the unconditional once-only claim until that condition is satisfied. This is stated in the spec, not a new code finding.

### 007-F2 — Reschedule safety guarantee only tests removed-link case

Classification: **partial-coverage**.

- [requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) — “firing ... against a since-cancelled and a since-rescheduled interview must produce zero outbox rows”
- [requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) — “RESCHEDULED ... whose link was removed ... still SCHEDULED with a non-blank link ... exactly two”
- [spec md](../../../007-generic-scheduler-db-scheduler/spec.md) — “catch Task Instance Currently Executing Exception ... closes a real500-race”

Proposal: define which schedule generation owns a reminder, recheck current due window/generation atomically, and reconcile a new schedule if old task is executing. No serialized personal information is required; generation/version can remain non-personal information.

### 007-F3 — Alerts and cutover backfill were explicitly deferred

Classification: **deliberate-supersession-not-folded**.

- [requirements md](../../../007-generic-scheduler-db-scheduler/requirements.md) — “failure alert rule exists ... committed”
- [spec md](../../../007-generic-scheduler-db-scheduler/spec.md) — “NO startup backfill ...4 retries ... metrics-only for now — DEFER the Prometheus alert rule”
- [design md](../../../007-generic-scheduler-db-scheduler/design.md) — “registered as one-time jobs at deploy ... or accept ... founder confirms”

Founder R1–R4 resolve these choices. Fold them into REQ008/design/tasks; metrics and4retrygive-up remain required. Do not present absent alert or backfill as an unresolved product decision.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full)

## 008 — conflict

Core fix is explicitly settled in evidence; final intent/keyboard wording and textarea handler disagree, and budget correction AC needs a one-word correction.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|conflict|Actual click-morph root cause and distinct button keys are recorded in ship note/evidence but not folded into final design recipe. Explicit-click wording conflicts with designed final-step Enter submit; textarea-safe claim conflicts with unconditional form key handler.|[requirements md](../../../008-tutee-post-flow-bugfixes/requirements.md) · [design md](../../../008-tutee-post-flow-bugfixes/design.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md)|
|REQ-002|partial|Partner revalidation design is aligned, but AC is arithmetically impossible as worded: from max<min, raising min cannot make min is at most max; change correction to lowering min (or raising max).|[requirements md](../../../008-tutee-post-flow-bugfixes/requirements.md) · [design md](../../../008-tutee-post-flow-bugfixes/design.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) · [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md)|

### 008-F1 — Submission intent wording and keyboard design disagree

Classification: **behavioral-contradiction**.

- [requirements md](../../../008-tutee-post-flow-bugfixes/requirements.md) — “post is created only after the user clicks Submit Request on the Review step”
- [design md](../../../008-tutee-post-flow-bugfixes/design.md) — “On the final step Enter is allowed to submit”
- [tasks md](../../../008-tutee-post-flow-bugfixes/tasks.md) — “Enter on Review submits”

Proposal: define deliberate activation of visible final Submit by pointer or keyboard; preserve accessible button Enter/Space behavior and block implicit submission from unrelated fields.

### 008-F2 — Textarea exemption is stated but absent from handler recipe

Classification: **design-internal-contradiction**.

- The earlier [design](../../../008-tutee-post-flow-bugfixes/design.md) intercepted Enter before the final step, but did not explicitly exclude multi-line text fields or text composition. Its promise to preserve newlines therefore needed a clearer rule.

The form-level recipe has no target/composition guard yet claims textarea newlines unaffected. Add explicit textarea/IME/button handling to the design; this is a design flaw, no current-code claim.

### 008-F3 — Budget correction scenario is impossible as written

Classification: **acceptance-wording-defect**.

- [requirements md](../../../008-tutee-post-flow-bugfixes/requirements.md) — “max budget below min budget ... raise min budget to a value at or below max budget”

Replace raise-min with lower-min; retain reverse raise-min-above-max scenario. No product decision needed.

### 008-F4 — Actual root cause and ship evidence are not fully folded

Classification: **partial-design-and-evidence**.

- [spec md](../../../008-tutee-post-flow-bugfixes/spec.md) — “distinct keys ... jsdom can’t reproduce ... virtual machine deploy pending”
- [design md](../../../008-tutee-post-flow-bugfixes/design.md) — “TODAY (suspected): Enter ... STRUCTURAL GUARD”
- [requirements md](../../../008-tutee-post-flow-bugfixes/requirements.md) — “Live convergence is mandatory before SHIP ... then re-confirm on the virtual machine”

The key-remount fix is an explicit recorded correction. Update final design/task5 with it and distinguish dev shipment from virtual machine acceptance still recorded pending; do not infer current runtime status.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full)

## 009 — partial

Feature requirements align with the approved taxonomy amendment and later coherence specs; canonical tables and test-strength alternatives need consolidation.

|Requirement|Verdict|Assessment|Evidence|
|---|---|---|---|
|REQ-001|partial|Canonical subject list approved through explicit founder delegation; N LEVEL amendment supersedes original O LEVEL table rows. Sync-test design allows count/spot-check weaker than full equality requirement.|[requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) · [design md](../../../009-tutee-post-taxonomy-and-ux/design.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md)|
|REQ-002|partial|Distinct N LEVEL is a clear founder amendment; original no-new-enum prose/table/tasks remain stale. Later020/025 amend taxonomy further; no unapproved new decision inferred here.|[requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) · [design md](../../../009-tutee-post-taxonomy-and-ux/design.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md)|
|REQ-003|aligned|Known-code422 contract and matching edit guard align. Later020 explicitly strengthens to coherent subject-level pairs.|[requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) · [design md](../../../009-tutee-post-taxonomy-and-ux/design.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md)|
|REQ-004|aligned|Exact6–20 reveal,1–5 unchanged,BE1–20 ceiling and edit prefill align;022 verifies rather than redefines.|[requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) · [design md](../../../009-tutee-post-taxonomy-and-ux/design.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md)|
|REQ-005|partial|Layout-only day rows/hour columns and preserved21-bit tutee bitmask are explicit. Do not mislabel parent-hour precision loss as a missing002 tutor migration. Ship note still lists required390px evidence pending.|[requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) · [design md](../../../009-tutee-post-taxonomy-and-ux/design.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md)|
|REQ-006|aligned|FAQ/footer/wizard rates links and unchanged JSON-LD map to tasks; draft persistence is a separate parent requirement, not added to link wiring.|[requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) · [design md](../../../009-tutee-post-taxonomy-and-ux/design.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md) · [tasks md](../../../009-tutee-post-taxonomy-and-ux/tasks.md)|

### 009-F1 — N LEVEL is approved but canonical tables retain superseded mappings

Classification: **deliberate-supersession-not-folded**.

- [requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) — “ENG NLEVEL / MATH NLEVEL / SCI NLEVEL ... O LEVEL”
- [spec md](../../../009-tutee-post-taxonomy-and-ux/spec.md) — “Founder ... APPROVED VIA BOSS/TEAM DELEGATION ... N-Level = distinct N LEVEL”
- [spec md](../../../009-tutee-post-taxonomy-and-ux/spec.md) — “remaps the N-Level subjects/levels to N LEVEL”

Propagate the binding amendment into canonical tables/FEbands/design/task3; later020/025 further constrain taxonomy. This has authority and is not an unapproved taxonomy choice.

### 009-F2 — Sync-test alternative is weaker than required set parity

Classification: **partial-validation-design**.

- [requirements md](../../../009-tutee-post-taxonomy-and-ux/requirements.md) — “every value posted by the picker exists as a subjects code ... compares the frontend list against the seeded set”
- [design md](../../../009-tutee-post-taxonomy-and-ux/design.md) — “read ... a small fixture ... or assert the count + a spot-set”

Require exact approved-code set comparison and a seed-derived/checkable source; equal count plus selected members does not demonstrate full parity. No tests run in this audit.

Documents read: [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full), [REDACTED LOCAL PATH] (full)

## Parent/amendment coverage

- [REDACTED LOCAL PATH] selected sections198–232,677–689,2010–2025 plus earlier marketplace review
- [REDACTED LOCAL PATH] selected registration,commission,interview,privacy,non-functional requirement clauses446–471,501–522,623–652
- [REDACTED LOCAL PATH] selected1–45; dated historical baseline, not current delivery truth
- [REDACTED LOCAL PATH] selected45–54 production one-time passcode override
- [REDACTED LOCAL PATH] read in prior marketplace audit; profile/listing matrix authority
- GH issues#193–#200,#202–#206 and twinkly-percolating-glade plan: references contained in specs; external issue bodies/plan not independently read; exact-copy fidelity/independent approval provenance not certified

Supplemental amendment read: [INT082 consent-copy lockstep](../../../architecture/int-082-whatsapp-otp-and-notifications-spec.md), lines182–188.
