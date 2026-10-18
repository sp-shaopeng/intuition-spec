# User flows and their current limits

Snapshot: 18 October 2026. This guide covers the main public, parent, tutor, administrator and account journeys. "Parent" includes the tutee role used by the application. The [current requirements](../specs/CURRENT-REQUIREMENTS.md) control over older examples; open decisions O01 through O12 remain open.

A flow below describes the required journey. "Present" means supported by the source review or documented module, with that scope stated; it does not mean this guide ran the journey. The [current-file review](current-file-alignment.md) is the baseline for gaps G01 through G12. Changes made after that review need new evidence before this guide's status changes.

The main chain is: browse → sign in → parent request → moderation → tutor application or invitation → parent selection → staff assignment → activation → parent confirms the first session → invoice to tutor → agency receives commission. Interviews support the selection process. Approval, an accepted invitation or a selection request alone does not release contact details.

## Public discovery

### UF01: browse tutors or assignments

- Start: a visitor needs a tutor or teaching work.
- Steps: open the relevant public list, set subject/level/area or other supported filters, inspect bounded results, then choose a detail or action. Sign in when the chosen detail/action requires it and retain the intended destination.
- Outcome: the visitor finds relevant public summaries without exposing the other person's contact details.
- Exceptions: anonymous pages 0 through 4 have at most 20 results each; later pages are empty. This is a bound per query, not a lifetime search quota. An empty result should allow another search.
- Present/missing: public lists exist; the baseline website hides guest filters/page controls (G01). The guest Post a Request destination differs from the retained login-first rule (G12). Public showcase and review content must not imply an unsupported verification claim (O05) or fabricated testimonial (O11).
- Rules and detail: C01/C04; [public browse](../specs/sequences/23-public-browse-with-filters.md), [detail login gate](../specs/sequences/36-marketplace-detail-login-gated.md), [showcases](../specs/sequences/40-public-showcase-strips.md), [landing requirements](../specs/005-landing-page-trust-and-social-proof/requirements.md).

## Accounts and access

### UF02: register, consent and complete a profile

- Start: a new parent or tutor chooses account creation.
- Steps: choose the role, enter account details and accept the required terms/privacy/age declarations. The tutor path has optional, skippable profile capture and a separate required step to read and acknowledge the commission terms before account creation. Create the account and session, then complete any deferred profile fields needed for later actions. Marketing consent is optional.
- Outcome: the account has its legal consent recorded; tutor commission acknowledgement is recorded with it atomically. The user can enter their role's journey. Tutor profile fields may be deferred at signup; action-specific profile completeness still applies later.
- Exceptions: duplicate identity, invalid data, missing mandatory legal consent or missing tutor commission acknowledgement must be reported without implying successful registration. Subject/level choices must follow the approved taxonomy; unresolved finer mapping stays O06.
- Present/missing: registration, consent fields and profile-completion mechanisms are documented. This guide does not establish a fresh end-to-end registration run or close every taxonomy acceptance case.
- Rules and detail: C05/C13; [parent registration](../specs/sequences/01-register-tutee.md), [tutor registration](../specs/sequences/02-register-tutor.md), [consent](../specs/001-legal-and-compliance-consent-and-policy-pages/requirements.md), [onboarding](../specs/002-tutor-profile-and-onboarding-ux/requirements.md).

### UF03: sign in, renew a session and sign out

- Start: a returning user opens login or a protected action.
- Steps: enter credentials, complete any applicable MFA challenge, return to the permitted destination, renew the session when required, and use server logout to revoke/clear the managed session.
- Outcome: access follows the user's role and the correct session policy. Parent/tutor access sessions use the specified 30-minute protected cookie and refresh lifecycle; admin policy is environment-specific.
- Exceptions: invalid credentials, blocked accounts, expired sessions, forbidden roles and rate limits have distinct outcomes. A refresh 429 must leave both cookies unchanged. Client script cannot clear an HttpOnly cookie.
- Present/missing: core session mechanisms exist. The website cannot complete an MFA challenge (G11). A filter interaction on renewal may violate the unchanged-cookie rule (G10); that combination needs an integration test.
- Rules and detail: C09; [login/MFA](../specs/sequences/03-login-with-mfa.md), [session requirements](../specs/004-auth-session-hardening-30min-ttl-and-refresh/requirements.md), [auth robustness](../specs/011-auth-robustness/requirements.md).

### UF04: recover or change a password

- Start: a user forgot the password, wants to change it, or must replace a temporary admin password.
- Steps: request email recovery and follow the valid reset link, or authenticate and enter the current/new password; complete the change and apply the required session revocation.
- Outcome: the new password is usable and obsolete access is rejected according to the current session contract.
- Exceptions: invalid, expired or replayed recovery tokens and incorrect current passwords do not change the account. Email delivery must remain pending on a recoverable outage; password recovery remains email-only.
- Present/missing: reset/change paths are documented. The general email loss paths in G08 prevent treating an enqueue or acknowledgment as proof that recovery email reached the user.
- Rules and detail: C09/C12; [password recovery](../specs/sequences/04-forgot-reset-password.md), [password change](../specs/sequences/29-change-password.md), [reset requirements](../specs/003-password-reset-failure-fix/requirements.md).

### UF05: edit a profile or change contact details

- Start: an authenticated user opens their profile/account settings.
- Steps: save permitted profile fields. For contact changes, supply the current password, request the change, then explicitly confirm a single-use expiring link. Phone changes are confirmed through the already-verified account email; email changes through the new email. Warn the old contact channels and support owner revocation.
- Outcome: the old value remains until confirmation commits. Tutor profile remains the canonical identity; an optional listing override does not rewrite it.
- Exceptions: wrong password, invalid/new duplicate contact, expired/replayed/revoked token and competing confirm/revoke actions must have safe outcomes. Inaccessible-channel recovery and old-phone warning transport remain O04.
- Present/missing: profile editing exists. G03 records incomplete templates, wrong destinations and missing revocation. Listing nullable overrides need further checking. The transport dependency prevents a complete contact-change claim.
- Rules and detail: C05/C08; [profile editing](../specs/sequences/33-profile-edit.md), [contact-change requirement](../specs/012-tutor-apply-and-profile-ux/requirements.md).

### UF06: delete an account

- Start: an authenticated account owner requests deletion.
- Steps: reconfirm credentials, apply logical deletion and required cascades/audit, then revoke sessions across devices.
- Outcome: the account loses access; retained business/financial records are not physically erased by this action.
- Exceptions: wrong password preserves the current session; unauthorized users cannot delete another account. Retention, anonymization timing and retained audit fields remain O08.
- Present/missing: sampled paths use logical deletion. This guide has no fresh evidence for every cascade or cross-device revocation and does not claim an approved erasure timetable.
- Rules and detail: C09/C13; [account deletion](../specs/sequences/30-account-deletion.md).

## Parent journeys

### UF07: create, edit, close or remove a tuition request

- Start: a signed-in parent needs lessons.
- Steps: enter the student's subject/level needs, location, schedule and budget; review the request; deliberately submit; follow moderation status. Edit the owned PENDING post when permitted. Close/remove an eligible request when no longer needed.
- Outcome: submission creates intake for moderation. Approval is required before public publication/assignment. Closing/removing a request cleans up associated open intake with the correct recorded reason.
- Exceptions: invalid combinations or missing fields block progression appropriately. Continue must not submit; text areas, text composition and Enter on the final step require the specified keyboard handling. Parent defaults and taxonomy decisions remain O09/O06.
- Present/missing: the main wizard and PENDING edit/initiation paths exist. G07 records unfinished keyboard cases. Every rejection/removal/closure cascade has not been reverified.
- Rules and detail: C06/C13; [creation](../specs/sequences/06-tutee-post-creation.md), [edit/close](../specs/sequences/32-post-edit-and-close.md), [wizard requirements](../specs/008-tutee-post-flow-bugfixes/requirements.md), [request flow](../specs/021-tutee-request-flow/spec.md).

### UF08: review applicants and choose a tutor

- Start: a parent opens their own job and its applicants.
- Steps: compare permitted tutor summaries, inspect available profile information, and submit a selection request for the chosen tutor. Staff then review and assign through UF14.
- Outcome: the parent's choice enters the staff queue. Choosing does not itself create an active match or reveal contact.
- Exceptions: another parent's applicant list is inaccessible. Duplicate, no-longer-eligible or competing choices must follow the selection/state rules; private contact remains hidden.
- Present/missing: parent applicant and selection paths are supported by the baseline. Complete browser-to-staff acceptance remains to be evidenced; misleading approval/contact text remains G06.
- Rules and detail: C04/C05; [choose tutor](../specs/sequences/38-choose-tutor-selection.md), [dashboard requirements](../specs/013-tutee-dashboard-and-invitations/requirements.md).

### UF09: invite a tutor and handle the response

- Start: a parent has an owned live PENDING or ACTIVE request and a tutor to invite.
- Steps: send the invitation; view or withdraw it; the tutor accepts or declines. Acceptance reuses the application and selection-request journey.
- Outcome: acceptance reaches staff selection handling, without a separate automatic matching path.
- Exceptions: only the inviter/recipient may act. Assignment, rejection, close or removal updates open invitations with the defined cause. There is no newly invented background timeout. Whether acceptance may occur before approval is O01; the baseline currently requires ACTIVE for acceptance.
- Present/missing: invitation initiation and response paths exist in inspected files. Pending acceptance remains an unresolved business decision; cascade completeness needs verification. Contact stays protected.
- Rules and detail: C04/C06; [invitation flow](../specs/sequences/43-tutee-tutor-job-invitation.md), [request amendment](../specs/021-tutee-request-flow/spec.md).

### UF10: request, negotiate or cancel an interview

- Start: a parent wants to speak with a potential tutor for a specific post.
- Steps: request an interview, propose times, accept/decline with a reason or counter; replaced slots become withdrawn. Agreed times reach staff scheduling. The owner can cancel in the defined non-terminal states.
- Outcome: an interview becomes ready to schedule, or ends with an explicit reason. It does not release private contact or create a match.
- Exceptions: the cap is three active interviews per posting; terminal states cannot be edited. Cancellation releases the cap and invalidates reminders. User-facing times are Singapore time even on a computer in another timezone.
- Present/missing: negotiation/scheduling exist. G04 affects time input. Stale-reminder and recovery checks remain G09. The staff outcome/link-change gaps are covered in UF15.
- Rules and detail: C07/C10/C11; [request](../specs/sequences/13-tutee-request-interview.md), [negotiation](../specs/sequences/42-interview-timeslot-negotiation.md), [parent cancellation](../specs/sequences/35-tutee-cancel-interview.md).

## Tutor journeys

### UF11: maintain a teaching profile and optional listing

- Start: a tutor wants suitable work and an accurate public presence.
- Steps: complete the canonical profile, choose valid teaching subjects/levels and availability, save rates/areas, and optionally create or edit a listing for moderation. Manage the availability/showcase-consent controls.
- Outcome: eligible tutors can apply without a published listing. Listing-specific optional values are overrides; they do not synchronize back into the profile.
- Exceptions: incomplete action-required profile data, invalid taxonomy and unapproved listing publication remain guarded. Availability alone is not a credential-verification claim.
- Present/missing: free-market/profile support and availability controls are documented. Some listing-form requirements may conflict with nullable overrides; that boundary is unverified. Exact finer taxonomy approval remains O06.
- Rules and detail: C05; [listing creation](../specs/sequences/07-tutor-post-creation.md), [availability](../specs/sequences/34-tutor-availability-toggle.md), [profile/onboarding](../specs/002-tutor-profile-and-onboarding-ux/requirements.md), [apply/profile](../specs/012-tutor-apply-and-profile-ux/requirements.md).

### UF12: apply, withdraw or respond to an invitation

- Start: a tutor finds a suitable assignment or receives an invitation.
- Steps: open the assignment, meet the canonical profile requirements, apply, track the application, or withdraw when eligible. Use UF09 for an invitation response and UF10 for an interview proposal.
- Outcome: the parent/staff can consider the tutor. Applying requires no published listing and does not expose private contact or guarantee selection.
- Exceptions: closed/matched/ineligible jobs and duplicate or terminal actions follow the state rules. Assignment chooses one winner and updates competing applications atomically; cancellation may restore only eligible applications.
- Present/missing: application and withdrawal paths are supported. A fresh full browser journey through parent selection and staff assignment is still needed to verify the combined experience.
- Rules and detail: C04/C05; [apply/withdraw](../specs/sequences/28-tutor-apply-withdraw.md), [applications module](../specs/modules/applications/spec.md).

## Administrator journeys

### UF13: moderate a submitted parent request or tutor listing

- Start: a permitted administrator receives a submission or opens the approval queue.
- Steps: inspect the submission, approve eligible content or reject it using the available moderation action; record the required reason/audit and notifications.
- Outcome: approved content can become public; rejection leaves the user an accurate status and cleans up dependent open intake where required.
- Exceptions: unauthorized staff and invalid/repeated transitions cannot bypass moderation. This flow does not invent a separate request-changes action. Approval does not by itself reveal contact or create a match.
- Present/missing: approve/reject actions and submission notifications are documented. The review did not verify every rejection cascade or external delivery.
- Rules and detail: C04/C06/C12; [admin approval](../specs/sequences/08-admin-approval.md), [approval module](../specs/modules/approval/spec.md).

### UF14: assign and activate a match

- Start: staff review an approved job, eligible tutor and any parent selection request.
- Steps: inspect applicants, create the assignment, verify agreed first-month plan fields, then activate. Assignment updates the job, winner and competing applications atomically. Activation locks the plan used for commission.
- Outcome: participants see the match; the website reveals contact only when ACTIVE or COMPLETED. Staff access remains separately authorized/audited.
- Exceptions: concurrent assignments must not produce two winners; stale choices receive a conflict. A listing is optional. Flexible frequency requires admin entry. The month multiplier and posted-rate bounds remain O03.
- Present/missing: matching, activation and plan lock are supported in inspected paths. The server separately allows authorized assigned participants in PENDING; the website hides that state's contact. G06 is misleading text, not proof of a website contact leak.
- Rules and detail: C03/C04/C05; [applicant view](../specs/sequences/39-admin-applicant-view.md), [match creation](../specs/sequences/09-admin-match-creation.md), [activation](../specs/sequences/10-admin-match-activation.md), [contact access](../specs/sequences/37-contact-reveal-post-assignment.md).

### UF15: schedule an interview and record its outcome

- Start: staff have an interview ready to schedule or a scheduled interview to manage.
- Steps: schedule the agreed Singapore time, set the meeting link, notify both participants, then record completion, cancellation or the correct no-show party. Reschedule or replace a link when needed and notify both participants of a new link immediately under the retained policy.
- Outcome: both participants have the correct arrangements; outcome, audit and reminder state agree.
- Exceptions: no-show includes BOTH participants. Terminal interviews reject invalid edits. A replaced schedule cannot fire an old reminder; a near-term link change cannot rely only on the 24-hour reminder.
- Present/missing: baseline staff screens cannot mark BOTH absent; link changes lack the required immediate notice/audit (G05). Staff prefill depends on browser timezone (G04); scheduler recovery remains G09.
- Rules and detail: C07/C10/C11/C12; [scheduling](../specs/sequences/14-admin-schedule-interview.md), [meeting link](../specs/sequences/15-admin-set-meeting-link.md), [outcomes](../specs/sequences/16-interview-completion-paths.md), [scheduling UX](../specs/014-interview-scheduling-ux-and-admin-visibility/requirements.md).

## Shared billing and cancellation

The parent, tutor and administrator each have a part in the following billing journey.

### UF16: first session, invoice, commission receipt and completion

- Start: a match is active with its first-month plan locked.
- Steps: after activation, the parent receives a confirmation email, signs in and confirms that the first lesson took place. That confirmation permits staff to invoice the tutor, with payment due in 14 days. The tutor pays the agency; staff record PAID only on actual agency receipt. PAID completes the match in this lifecycle. Repeated parent confirmation must not create duplicates.
- Outcome: financial records distinguish the tutor's debt to the agency from the parent's payment for teaching. Match COMPLETED does not certify that all lessons have ended.
- Follow-up: after seven calendar days without parent confirmation, the required reminder and staff-review flag lead to a staff choice: override confirmation, contact the parent, or cancel with an explicit waiver decision. The source disagrees on when those seven days start: activation in PRD §7.7.1/AC-11, sending the confirmation email in AC-05, or receiving the notification in BRD BR-010. That starting event is unresolved. The BRD also permits earlier staff discretion to override, whereas other clauses describe override after timeout; that permission needs a decision. O12 separately leaves the reminder’s delivery deadline versus detection schedule unresolved; this guide does not promise a measured delivery time.
- Exceptions: an invoiced commission can enter DISPUTED for staff resolution or be explicitly WAIVED; staff notes are required for either. Post-invoice match cancellation requires an explicit WAIVED or DISPUTED decision (UF17). A parent paying a tutor, or an agency-favour dispute decision with no receipt, must not mark commission received. Advance/partial receipts, refunds, the post-resolution unpaid state and early termination remain O10. O03 affects commercial defaults.
- Present/missing: activation, pending commission and manual receipt recording exist. The parent confirmation, reminder/follow-up, invoice and due stages remain missing (G02); the simple payment action does not implement that journey. Required dispute/waiver handling is retained, while the PRD defers its dedicated workflow UI. These exception paths are not newly verified here. Manual staff recording is not bank reconciliation.
- Rules and detail: C02/C03/C17; [commission receipt](../specs/sequences/11-admin-mark-commission-paid.md), [commission module](../specs/modules/commission/spec.md), [financial safety](../specs/017-admin-analytics-and-financial-safety/requirements.md), [billing requirements](../specs/PRD.md).

### UF17: cancel a match and reconcile its effects

- Start: staff need to cancel an eligible pre-completion match.
- Steps: check eligibility and the billing stage, cancel the match, apply the permitted pre-invoice commission outcome, and reopen the job/eligible applications as specified. After an invoice, staff must explicitly choose WAIVED or DISPUTED and record notes; cancellation cannot silently choose the debt outcome. Preserve cause-specific closed/expired states and save the audit/notification intent atomically.
- Outcome: participants see the cancellation and the job has the correct intake status. Cancelled uncollectable amounts do not inflate outstanding totals.
- Exceptions: this path cannot cancel COMPLETED matches or erase collected PAID commission. A refund, reversal or early teaching termination is not silently treated as ordinary cancellation; O10 still applies.
- Present/missing: cancellation/reopening have historical implementation evidence. Full current cascade, race and notification behaviour needs fresh combined-revision testing; delivery remains affected by G08.
- Rules and detail: C02/C03/C12; [cancel/cascade](../specs/sequences/12-admin-match-cancel-cascade.md), [matching module](../specs/modules/matching/spec.md).

## Administrator reporting and access

### UF18: inspect financial and activity analytics

- Start: permitted staff need a dated operational view.
- Steps: choose the supported date window, inspect activity/commission measures, and interpret each metric using its defined population and timestamp.
- Outcome: collected revenue counts agency receipts; unpaid debt stays outstanding. Recent, 31-to-90-day and older records distinguish window results.
- Exceptions: cancelling a pending debt must not erase paid receipts. Average commission's cohort remains O02; the separate analytics-store requirement remains O07. Old seed data cannot prove a recent-window filter.
- Present/missing: date-range and paid-commission paths exist; achieved results for every metric and fixture boundary were not reverified.
- Rules and detail: C02/C15; [analytics flow](../specs/sequences/41-admin-analytics.md), [money safety](../specs/019-admin-money-safety-and-analytics/spec.md).

### UF19: onboard or remove an administrator

- Start: authorized staff need another administrator, or a super-admin needs to remove an eligible administrator.
- Steps: create the account under the temporary-password contract; enforce first password change and applicable production MFA before privileged work. For removal, check super-admin authority, logically delete the target, purge its sessions and retain audit.
- Outcome: only eligible staff can use administrative actions.
- Exceptions: ordinary admins cannot remove admins, protected/self-removal rules apply, and duplicate identities/weak temporary passwords fail. Local fixtures are not production bootstrap.
- Present/missing: management and backend MFA mechanisms are documented. The missing website MFA challenge (G11) limits the complete production login journey; launch/bootstrap acceptance is unverified.
- Rules and detail: C09/C13/C16; [admin management](../specs/sequences/31-admin-create-delete-admin.md), [admin module](../specs/modules/admin-management/spec.md), [production package](../specs/027-aws-production-deployment/spec.md).

## Shared follow-up journeys

### UF20: view matches, favourites and reviews

- Start: a signed-in parent/tutor returns to their dashboard, or a visitor reads permitted public reviews.
- Steps: view owned matches and status, open authorized contact at the website's allowed states, manage favourites, and submit a review when eligible. Public readers see permitted review content/aggregates.
- Outcome: the user can follow an engagement and find saved tutors; eligible feedback contributes to the displayed review result.
- Exceptions: participants cannot access unrelated matches; reviews require the eligible completed match and reject duplicates/invalid ratings. COMPLETED follows commission receipt, not the end of every lesson. Public reviews do not make all tutor detail public.
- Present/missing: dashboard, favourites and reviews are documented. This guide has not rerun role isolation or duplicate-submission races; G06 still affects earlier contact promises.
- Rules and detail: C03/C04; [parent matches](../specs/sequences/17-tutee-views-matches.md), [tutor matches](../specs/sequences/18-tutor-views-matches.md), [review submission](../specs/sequences/19-tutee-submits-review.md), [public reviews](../specs/sequences/20-public-tutor-detail-reviews.md), [favourites](../specs/sequences/22-favourite-toggle.md).

### UF21: receive notifications and recover failed delivery

- Start: a committed business event requires an in-app or external message.
- Steps: commit the business state, in-app row and durable delivery intent together; show the notification centre/bell; deliver externally after commit; retry recoverable failures and expose explicit pending/dead-letter outcomes.
- Outcome: the intended recipient sees an accurate event, and delivery work remains recoverable until success or a recorded permanent outcome.
- Exceptions: disabled/missing-key delivery is not success; queue limits cannot discard unfinished work. Replayed work cannot create duplicate committed intent. WhatsApp remains OFF pending its enablement conditions. Five-minute critical and thirty-minute informational targets remain requirements, not measured results; O12 leaves one timeout clock unresolved.
- Present/missing: in-app and durable delivery mechanisms exist, but G08 records loss paths and G09 records reminder safeguards still missing. OFF does not establish delivery success. Password reset remains email-only.
- Rules and detail: C11/C12/C17; [notification centre](../specs/sequences/21-notification-bell-flow.md), [email delivery](../specs/sequences/26-email-fanout-on-event.md), [reminders](../specs/006-whatsapp-interview-reminders/requirements.md), [scheduler](../specs/007-generic-scheduler-db-scheduler/requirements.md).

## Coverage and use

The 21 flows cover public discovery; registration, sessions, passwords, profiles, contact changes and deletion; parent requests, applicants, invitations and interviews; tutor listings, availability and applications; moderation, assignment, activation, interview operations, billing, cancellation, analytics and staff management; dashboards, favourites, reviews and notifications.

Supporting cross-cutting sequences for [CSRF bootstrap](../specs/sequences/05-csrf-bootstrap.md), [rate limits](../specs/sequences/24-rate-limited-endpoint.md), [protected writes](../specs/sequences/25-csrf-protected-write.md) and [contact storage](../specs/sequences/27-encrypted-pii-write-and-lookup.md) support these journeys. Despite the historical filename, C13's plaintext-contact amendment controls the storage description. Together these references cover all 43 numbered sequence documents. This is high-level journey coverage, not a claim that every acceptance case is tested.

The [specification index](specification-index.md) links the detailed packages and the [route reference](../specs/reference/frontend-routes.md) names actual screens. The [workflow plan](workflow-improvement-plan.md) explains how each changed flow should gain a requirement-to-test evidence record. Preserve this snapshot until evidence for a later version has been checked.
