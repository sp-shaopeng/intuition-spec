# Business Update — Product → Biz Team

> Reading note: [Current requirements](CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../docs/publication-notes.md#history-and-review-roles).

**Date:** 2026-05-29 · **From:** Product · **Audience:** Business / founder /
commercial. Plain language, no jargon. Source of truth: [SCOPE md](SCOPE.md).

## Where the product stands

The platform works end-to-end **inside the building**: people can sign up, post
tutor listings and tuition requests, get approved, be matched, interview, and have
commission recorded. Twelve feature areas (M1–M12) are built and tested to a high
quality bar.

## The honest caveats (please read before promising dates)

- **No emails actually go out yet.** The wiring exists but nothing sends. Password
  resets and match notifications are in-app only until we build the email sender.
- **No real payments.** Commission is recorded by staff by hand; we have not
  connected card or Pay Now.
- **Not production-hardened.** No automated deploys, no staging copy, no database
  backups yet. These are the things that must exist before we take real users.

These are known, scoped, and on the roadmap as launch blockers.

## What we just decided to build (founder-driven)

1. **An Operations Console for our staff (M13).** Today our team can't easily see
   a person's full details to contact and verify them. We're building a secure,
   access-logged console where the operations team (a new "Operator" role) can
   search and see everything they need — stored safely and isolated, with every
   view recorded for compliance.
2. **A livelier landing page (M14).** First-time visitors will see 10–20 fresh
   tuition jobs and 10–20 active tutors, so the marketplace looks alive from
   second one. Real data only — nothing faked.
3. **"Apply" to jobs (M15).** Tutors will be able to apply to tuition jobs. This
   gives us a real "X tutors applied" number on each job (the stat the founder
   asked for) and a genuine pipeline for our team to match from — instead of cold
   manual matching.

## Why these matter commercially

- **M13** makes the operations team actually able to run the business and stay
  Personal Data Protection Act-compliant — fewer manual workarounds, an audit trail we can stand behind.
- **M14 + M15** attack the cold-start problem: visible activity + a real apply
  loop is how a two-sided marketplace builds trust and liquidity.

## What we still need from the business

- A decision/lead time on **email provider** and **payment provider** — these are
  the two hard blockers to taking paying users.
- Personal Data Protection Act/legal sign-off on the operations console handling personal data before it
  goes live in any shared environment.
