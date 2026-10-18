# INT-165…193 — Frontend/user interface/wording/SEO audit fixes — SPEC

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **Current contract correction (2026-09-05):** Where applicable, [005](../005-landing-page-trust-and-social-proof/spec.md) settles stats-band framing, [009](../009-tutee-post-taxonomy-and-ux/spec.md) retains1–5 plus6-or-more/exact6–20, and [022](../022-tutor-profile-and-fe-polish/spec.md) makes that a regression check. Testimonials remain gated on actual approved content/consent; these mechanical corrections approve no vetting/guarantee claim. Older code/schema examples are historical in the affected scope; they do not approve a policy override.


**Status:** SPEC · **Date:** 2026-06-18 · **Source:** multi-skill live audit (uiux-designer,
frontend-designer, frontend-expert, content-seo, web-competitor-analysis) of the public + conversion-funnel
surfaces. Findings recorded as INT-165…193 (GitHub #148–176).

## Context
A fresh evidence-driven audit (Playwright on the live local stack, desktop + 390px, + code review) surfaced
29 issues across data, a11y, SEO, mobile, user experience, copy, and positioning. This spec groups them into build
batches and separates **engineering fixes (build now, autonomously)** from **product/content items that
need a founder decision** (we must not invent pricing, guarantees, testimonials, or real metrics).

## Build batches (each batch: build → gate → team-review → Boss → deploy local+virtual machine → push)

### Batch 1 — infra/a11y high-severity (S2)
- **INT-165** (backend) Collector Controller: accept both JSON and plain-text request bodies;
  read the raw body → parse to Collect Event Request for text/plain; add Http Media Type Not Supported Exception
  → 415 in Global error handler. Live-verify text/plain → 204. (backend lane.)
- **INT-166** (frontend) Notification Bell: add enabled: !user to the unread-count query; add
  /notifications/unread-count to the should Trigger Logout exemption list (auth Redirect implementation reference).
- **INT-168** (frontend) skip link: sr-only focus:not-sr-only anchor as first child of each layout +
  a main content element that the skip link can focus on (Root Layout/Public Layout/Tutee/Tutor/Admin).
- **INT-169** (frontend) Tutee Layout implementation reference + Tutor Layout implementation reference px-8 → px-4 md:px-8 (match Admin);
  re-run the 390px overflow gate.

### Batch 2 — SEO (S2/S3)
- **INT-167** per-route page title element/meta: add react-helmet-async (or a tiny use Document Meta hook) and set
  unique title+description on the public routes (/, /tutors, /assignments, /rates, /about, /faq,
  /contact).
- **INT-181** add  per public route + a static Organization JSON-LD in index html
  (name/address/UEN/same As from COMPANY). (FAQPage JSON-LD lands with INT-187 if approved.)

### Batch 3 — copy / IA quick wins (S3/S4)
- **INT-191** footer Pte. Ltd.. → strip the trailing . before the copyright concat (Footer implementation reference).
- **INT-192** landing CTA → "Browse Tutors →" (Landing implementation reference).
- **INT-171** footer About/FAQ/Contact → /about /faq /contact (Footer implementation reference).
- **INT-172** "Post a Request" (logged-out) → /register?role is tutee with return To; de-dup the two ghost CTAs.
- **INT-174** one verb for register everywhere ("Sign up"); include the demand CTA in the mobile drawer;
  align the login-page account-create labels (Nav implementation reference, login page).
- **INT-182** drop named competitors in Landing implementation reference → category claim ("Other platforms…").
- **INT-193** compact level format for dense chip contexts (keep long labels in forms) — format implementation reference.

### Batch 4 — reusable-component a11y (S3)
- **INT-188** Combobox — render the  only when open (not hidden).
- **INT-189** Modal — capture document active Element on open, restore focus on close; dev-warn if no
  label Id/aria-label.
- **INT-190** Combobox clear button → min-h- 44px min-w- 44px flex-centered (optical-align via margin).

### Batch 5 — funnel user experience (S3)
- **INT-173** browse card: suppress/replace the bare "Singapore" location fallback; promote differentiators.
- **INT-177** landing hero: make "Find a Tutor" the single dominant primary; demote "Find tuition jobs".
- **INT-180** collapse the 3 completeness prompts to one canonical surface.
- **INT-179** split tutor Profile implementation reference into tabs (Profile · Security · Reviews · Account); isolate Danger Zone.

### Batch 6 — bigger user experience restructures (S3)
- **INT-175** detail pages: sticky bottom CTA on mobile; inline apply for incomplete-profile tutors; one
  focal rail action at a time.
- **INT-176** Create Listing step 1: group subject×level by level-band (Primary/O-Level/A-Level) cascade.

## ⚠️ NEEDS FOUNDER DECISION (content/policy — NOT built until answered)
These require business input; specced but parked with needs-design:
- **INT-170** pricing/commission transparency — what to publish (parent "no agency fee"; tutor "50% first
  month"?) + whether to surface a public rate guide.
- **INT-178** stats band — which REAL metric can we honestly show (tutor count? jobs posted?).
- **INT-183** testimonials — use real M11 reviews? which?
- **INT-184** "how we vet" copy — exact vetting steps to claim (must match the M3 reality).
- **INT-185** guarantee — is there a replacement / no-lock-in promise the business backs?
- **INT-186** WhatsApp/phone contact — which channel/number, or a contact form for now?
- **INT-187** public FAQ — the actual Q&A content (then wire /faq + FAQPage JSON-LD).

## Verification (per batch)
frontend gate (lint+build+test:ci ≥97.5/88) + backend gate where touched (the documented operational check ≥97.5/97.5) +
live re-check via Playwright (desktop+390px) for the visible fixes + e2e green. team-review (18-seat) →
Boss SHIP per batch. Deploy local+virtual machine, push the private implementation folder Mark each INT FIXED-DEV in the tracker (GitHub
issues left open per founder's standing call unless told otherwise).
