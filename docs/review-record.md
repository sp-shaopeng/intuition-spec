# Review record

**Latest scope, 18 October 2026:** a new review checks high-level sequence diagrams and the AI coding workflow. Its findings and final checks are recorded at the end of this page. Earlier approvals apply only to their dated publication scope; they did not prevent later reviewers from finding defects.

**Later alignment review:** the six reviews below assessed the original publication. Three new independent AI reviewers subsequently compared it with the current application and source specifications and found real mismatches. Read the [current-file alignment report](current-file-alignment.md) for the corrections and remaining application gaps. The original publication passes were not approval of application compliance.

The original reviewers' activation-clock wording below was later found incomplete. The source uses activation, email sending and notification receipt as different starting events for the seven-day period. No later controlling decision settles that disagreement. Separately, the current register's O12 remains open about the reminder deadline and detection schedule; the earlier review did not settle either question. This is one reason later reviews must check the source decisions as well as earlier conclusions.

Six AI reviewers examined this publication from separate roles on 18 October 2026. Each received its own brief and worked without reading the other reviewers' reports. These were AI review sessions, not reviews by six human teams or independent organisations.

**All six final rechecks passed within their stated publication-review scopes.** The first round required changes in every role. The table below records the main findings and the corrections that were checked again. This was a review of the documentation, not approval to launch the service.

| Review role | What the reviewer challenged | Correction made and rechecked |
|---|---|---|
| Business | An older experiment named the wrong commission payer; a monthly example conflicted with its weekly description; the customer-acquisition plan was thin. | Name the tutor as the agency-fee payer, use an explicitly agreed monthly plan, and state the untested acquisition and supply assumptions. |
| Product | The seven-day clock had no clear starting event; interview coordination and fee acknowledgement were underexplained. | The initial recheck used activation as the clock start; the later source comparison reopened that choice. It also distinguished agreed interview time from staff scheduling and explained fee acknowledgement at signup. |
| Technical lead | Encryption and message-recovery claims exceeded the inspected implementation; combined tests did not necessarily exercise each application pull request. | Describe plain-text contact storage, specific delivery gaps, and the actual boundaries of combined checks. |
| Developer | The export retained fragments of source and commands; some headings and links no longer matched. | Replace fragments with meaningful prose and check the final Markdown paths and heading links. |
| QA | Early cleanup removed real requirements and damaged tables; navigation needed verification after the format changed. | Restore every current rule, compare table contents with the source, and validate the final Markdown collection. |
| AI skeptic | Some converted text had lost its meaning; older claims implied proven benefits or human-team approval; evidence labels were misleading. | Rewrite damaged passages from the source, remove unsupported comparisons, explain AI roles locally, and identify private evidence as excluded. |

## Final verification

The original reviewed collection contained 357 Markdown files, including 346 specification documents. Both specification indexes accounted for all 346 documents. Checks found no broken local document links or heading links, inconsistent table widths, non-Markdown publication files or symbolic links. The later alignment report adds one guide and records corrections beyond those original checks.

The developer recheck examined the whole collection with content scans and manually inspected reported source fragments. Commands, query examples and damaged conversion markers were rewritten into prose. Pattern scans found no confirmed unredacted credentials or private contact details. A separate comparison of 100 candidate credential values found in the private files found no matches in the public draft. These are bounded checks, not a guarantee against every unknown secret format.

The business recheck also required cancellation wording to preserve existing receipts and distinguish billing stages. The product recheck preserved the difference between server authorisation and the website's contact-display rule. The technical recheck required explicit independent approval of the specification before implementation. QA caught a placeholder that Markdown interpreted as a broken link. The skeptical-reader recheck originally chose activation as the seven-day clock start. The later source comparison found conflicting clauses, so that choice is no longer presented as settled.

Publication uses a new repository and fresh history. Only the reviewed Markdown documents are staged; original source history and private evidence are excluded.

## What the review can establish

The reviewers checked the public explanation, selected source documents and, for the technical role, relevant private implementation. The publication checks cover document inventory, relative links, tables, file types and selected sensitive-content patterns. The private implementation and original reports are not included here.

The original publication review did not rerun the application suites. A later limited backend baseline is recorded in [Testing and release](testing-and-release.md#follow-up-baseline-check), with the excluded live lab and unfinished frontend check stated explicitly. Review does not establish production readiness, security certification, revenue, customer demand or a successful public launch. InTuition remains unlaunched.

## How findings are handled

A finding needs a concrete correction and a recheck. A passing publication review means the reviewed documentation is acceptable within its stated scope. It does not close the implementation gaps described in [testing and release](testing-and-release.md), or settle the business decisions in the [current requirements](../specs/CURRENT-REQUIREMENTS.md).

## Follow-up: project workflow, user flows and Markdown conversion

The follow-up addresses how the project is managed with AI: approved written requirements, small assigned tasks, issue and evidence tracking, independent review, and maintenance of the regression lab. Product journeys are documented separately. No application feature was changed by this documentation work.

| Independent review | Finding and correction | Scope |
|---|---|---|
| Product and conversion reviewer | Required tutor commission acknowledgement at signup; explicit parent confirmation, billing and cancellation exceptions; faithful missing-rate and navigation details in the historical mockups. | Compared the new journeys and Markdown archives with private source requirements and original bundled designs. Five findings were corrected and independently rechecked. |
| Regression-lab reviewer | Found coverage summaries that count references rather than executed journeys, checks not wired into the combined workflow, shared-data and target risks, and unenforced quarantine rules. | Read the actual test-lab files. The plan records these gaps and proposed controls; this review did not run the full lab. |
| Workflow and skeptical-reader reviewer | Required a clear failing-test/fix/test cycle, results before independent final review, accurate release ordering, and explanations of test data and required checks. | Independently rechecked those corrections and the five earlier content findings. Approval covers the documentation, not application behaviour or release readiness. |
| Publication and privacy reviewer | Found leftover element/component markup in older Markdown specifications and two editing-tool closing tokens. | Required prose replacements preserving control type, labels, navigation and access restrictions, plus a final inventory, link, format and sensitive-content recheck. |

The final source comparison also found acceptance scenarios incorrectly omitted from the public PRD as if they were implementation examples. An independent reviewer compared all three restored journey maps, the commission flow and all 19 acceptance sections, confirming that all 51 source scenarios remain. The restored text preserves the source requirements and explicitly identifies conflicts with later controlling rules. It does not turn historical wording into a new product decision.

The collection now contains 362 Markdown documents, including the same 346 specification documents, 21 main user journeys linking all 43 numbered sequences, and two historical HTML designs converted into Markdown. Private repair drafts created after the initial specification snapshot remain excluded and are not approved implementation work.

Follow-up content rechecks passed within those scopes. Final collection checks found no broken local document or heading links, malformed table widths, non-Markdown files, embedded HTML or rendered code blocks. Literal field placeholders remain visible as notation. A fresh comparison of the 100 private credential candidates found no matches; separate checks of the six private company legal/contact values also found no exact or punctuation-normalized matches. These bounded checks do not guarantee that every possible unknown secret format has been detected, remove older remote objects or revoke credentials.

## Fresh six-role review of the project workflow

Six separately briefed AI reviewers examined the publication at revision 655a894 and selected current private sources. They worked in parallel batches within the available agent capacity. Their first-pass reports were completed before the shared rewrite. They were not six human teams. The business reviewer encountered historical sign-off text while searching sources and explicitly excluded it from its conclusions; the other reviewers did not consult one another's reports before their first verdicts. This was independent work within the same AI-assisted process, not independence from the model provider or project owner.

Every perspective required changes. The documentation author then combined duplicate findings into the [improvement plan](workflow-improvement-plan.md). The coordinating assistant checked the cited files and the read-only GitHub settings. No application tests, settings changes or deployments were performed in this round.

| Perspective | Main finding | Documentation or proposed action |
|---|---|---|
| Business | Admin-panel uptime had become a staffing promise; known billing decisions were missing from the register; the plan did not account for founder capacity and business learning. | Correct C17, register the conflicts, budget founder time and distinguish early learning, technical readiness and trial admission. |
| Product | The register omitted three policy conflicts and the main billing flow omitted fourteen-day escalation. Completed tasks did not establish a complete service journey. | Add O13–O15 without choosing policy; restore staff override-or-cancel with no automatic cancellation; define an acceptance record and a required-journey list. |
| Technical lead | Build paths and a protected identity endpoint block reliance on the combined lab. Required checks and deployment approvals are not configured as the prose might suggest. | Disclose the source findings and observed settings. Put startup first, then exact-version approval, compatible interfaces, release checks and demonstrated recovery. |
| Developer | A commit label can hide uncommitted changes; interrupted work lacks a dependable handoff; a shallow interface comparison and a green subset of tests can miss required behaviour. | Bind source and build identities to evidence, retain one resumable private record, require both sides of interface changes to be checked, and reconcile required cases with actual results. |
| QA | Skipped or retry-pass cases can earn report credit; an inventory mismatch does not fail the script; evidence expires or omits successful runs. | Define accurate case outcomes, a deliberate failure example for each blocking check and retained success/failure summaries. These controls still need implementation. |
| AI skeptic | The linked historical workflow contradicts itself; process prose repeats; totals lack a compact dated example and the founder's workload is understated. | Label conflicting historical instructions, keep W01 for founder reconciliation, shorten repeated explanations, add the dated request-form evidence trail and use one bounded pilot. |

All six correction rechecks are READY within their stated documentation and proposed-plan scopes. Product recheck required populated draft acceptance rows, which were added and checked again. Technical recheck caught a moved heading link, which was repaired and reopened. The developer later authored the historical workflow cleanup; the technical-lead and skeptic reviewers checked that file independently, and the developer's own final verdict excludes it.

Final collection checks cover 362 Markdown documents, including 346 specifications and 7,115 local document/heading links. There are no broken local links, malformed table widths, non-Markdown publication files, rendered HTML or code blocks. Comparisons found no matches to the 100 private credential candidates or six private company legal/contact values. These remain bounded checks, not a guarantee about unknown secret formats.

The proposed operating controls, unresolved business decisions and application repairs are not approved or completed by this documentation review. The first startup pilot is UNVERIFIED and billing remains BLOCKED. InTuition is still not live.

## AI-assisted project delivery as the main story

The founder clarified that the main purpose is to show how they use AI to deliver project work. The README and working-method page now distinguish founder direction, AI execution, independent review, corrections and observed results. Business context and detailed specifications remain available through their existing guides.

The application example retains its dated historical result and outstanding keyboard requirements. The separate completed example concerns publication of the Markdown specification repository at a04514d; it does not imply that the application has launched. The founder's recorded directions are separated from checks performed by AI assistants.

Six separately briefed AI reviewers checked the two entry pages without reading one another's current reports or editing the pages. Business, product, technical lead, developer and QA returned READY within their respective scopes. The AI skeptic required a plain-English correction: phrases such as "bounded issues" and "installed governance" became "tasks with clear limits" and a description of rules enforced by tools. The skeptic and technical lead independently reread the corrected pages and returned READY. These are AI reviews of the account, not human-team certification or a new application test run.

The coordinator opened all six reports and checked the final files. Collection checks cover 362 Markdown documents, including 346 specifications, 43 numbered sequences and 7,097 local document/heading links. No broken local links, malformed tables, non-Markdown publication files, symbolic links, rendered HTML or code blocks were found. Comparisons found no matches to the 100 private credential candidates or six private company legal/contact values. Those checks cover the described files and values; they do not guarantee detection of every unknown secret format. The 7,115-link figure in the completed case belongs to the earlier a04514d publication.

No application or test-lab code, repository settings or business policies are changed by this revision. The not-live status and outstanding application work remain explicit.

## Sequence diagrams and AI coding workflow

Six separately briefed AI reviewers examined the unchanged 4bc40f1 publication from business, product, technical-lead, developer, QA and skeptical-reader perspectives. They completed their first reports before the authors began the shared revision and did not read one another's current reports. All six requested changes for the new diagram scope. This does not turn their earlier prose-only approval into a claim that diagrams already existed.

The review found no rendered diagrams in the 362-document baseline, despite the sequence index promising them. The index also retained superseded contact-encryption and message-recovery descriptions. The coding walkthrough needed clearer specification approval, test-first work, ownership of shared browser/test resources, complete-case checks, final coordinator audit and development-delivery handoffs. The written method must preserve W01 and separate completed development work from approval for a public release.

All six role rechecks returned READY within their documentation scopes. The technical recheck required a failed or missing local startup/flow check to stop before pushing; the corrected sequence then handles an unconfirmed push separately. Actual rendering caught three message-label semicolons that Mermaid treated as statement separators; these were replaced with commas. The final wording pass also replaced an unnecessary process term with plain English.

The revised collection has 363 Markdown documents, including the same 346 specifications. Eleven high-level sequence diagrams cover the main coding, test-lab, product and system handoffs. Coverage tables account for all 21 user journeys and all 43 detailed records, while explicitly identifying text-only details. This is documentation coverage, not passing acceptance coverage. Local checks found no broken links among 7,301 document/heading links, malformed tables, HTML or code blocks other than the eleven reviewed Mermaid sequences. Comparisons found no matches to 100 private credential candidates or six private company legal/contact values.

All eleven diagrams parsed and rendered with Mermaid 12.2.0 in local Chromium. The render record identifies the exact source and each diagram; empty, stale or unexpected diagram inventories fail validation. QA opened all eleven full-chart images, and the skeptical-reader review inspected the main workflow, service and billing views. Narrow-screen samples show wider charts that need horizontal scrolling or the supplied prose alternatives. These checks do not certify GitHub's mobile layout. Privacy comparison checks were also hardened and exercised with synthetic matches and missing-input examples; no real private value was exposed by those checks.

These are documentation changes. No application behaviour, business policy or tool-enforced project rule is changed by drawing them, and no new application test pass or public launch is claimed.
