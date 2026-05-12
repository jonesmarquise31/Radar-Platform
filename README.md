# Workforce Radar

Public artifacts from the build of [Workforce Radar](https://workforceradar.com) — a career intelligence platform that diagnoses how the modern hiring system reads or misreads skilled professionals.

## What this repo is

This is the public build log. It contains technical write-ups, architecture notes, design-system documentation, and selected engineering patterns from the Radar build. The production codebase, diagnostic prompts, classification logic, archetype framework, and customer data live in a private repo and are not published here.

The intent is to show the work without giving away the work.

## What Radar is

Radar diagnoses signal-layer misalignment in the hiring market. A 5-minute diagnostic classifies professionals into one of six archetypes and identifies the specific reasons their target market is reading them incorrectly. The platform serves three buyer types through three portals — professionals diagnosing themselves, career coaches diagnosing their client books, and hiring teams querying a classified candidate pool — plus a standalone entry-tier product (the Decode) that delivers a personally-written read of how a single profile is being parsed.

The system was built on data from 250+ classified IT, cybersecurity, and finance professionals. The diagnostic engine reads hiring systems themselves rather than any single industry.

## Product tiers

The line is a single funnel: free diagnostic stack into one premium engagement.

```
Free      Quick Read            Two five-minute diagnostic entries —
                                perception axis and valuation axis.
                                Anonymous, no email.

Free      Full Diagnostic       Account-gated. Full archetype classification
                                + cross-dimensional signal breakdown +
                                structured verdict. The terminal free
                                surface.

Premium   The Radar Special     Four-week intelligence engagement for
                                directors, VPs, and C-suite operators.
                                Waitlist-gated. Cohorts capped.
```

The free funnel is the dataset feeder. Every submission compounds the platform's aggregate; every diagnosis reads from that aggregate before generating output. The premium tier is positioned at the altitude where the engagement actually creates leverage; access is by waitlist application, not by checkout.

The standalone Decode product on a separate domain was a V41-era surface; under the V44 funnel collapse, the entry-tier paid surfaces from the prior offer stack have been retired in favor of the single-engagement structure described above.

## Architecture

- **Frontend:** Static HTML, vanilla JS, Tailwind CSS on the platform; single-file static HTML/CSS/JS with no build step on the Decode landing
- **Backend:** Netlify Functions (Node.js)
- **Database:** Supabase (Postgres + Auth + RLS); RLS deny-all-except-service-role on all submission tables
- **Storage:** Supabase private buckets for uploaded artifacts (resumes, supporting documents)
- **Payments:** Stripe Checkout; server-side Stripe verification on every payment-gated write; Stripe webhooks for real-time buy events
- **Notifications:** Telegram bot for operator alerts on every paid Stripe session
- **AI:** Anthropic API (Claude) for diagnostic generation on the platform; the Decode is hand-written
- **Surfaces:** `/quiz` (perception-axis free intake), `/scope` (valuation-axis free intake), `/rds` (account-gated terminal diagnostic, free), `/radar-special` (premium engagement, waitlist), `/coach` (career coaches), `/hiring` (recruiters/hiring teams), `/admin` (operator command center, auth-gated — see [architecture/admin-portal.md](./architecture/admin-portal.md))

## Build log

Major releases and the technical decisions behind them are documented in [/build-log](./build-log).

- [**V44 — Funnel Collapse + Director-Tier Offer**](./build-log/v44-funnel-collapse.md) — dual-lane offer stack retired, single funnel into one premium engagement (The Radar Special), waitlist-gated cohorts
- [**V41 — The Radar Decode**](./build-log/v41-radar-decode.md) — standalone entry-tier product, separate-domain deploy, post-payment intake flow, real-time buy alerts via Stripe webhook → Telegram
- [**V40 — Funnel friction reduction**](./build-log/v40-funnel-friction.md) — name-only entry, results gated behind account creation, phone removal across paid flows

## Events

Public live sessions documented as structured architecture artifacts — audience composition, session architecture, mechanic, deliverable layers:

- [**The Underprice Read**](./events/2026-05-08-the-underprice-read.md) — 45-minute live diagnostic session on the structural pricing gap mechanic; four-block architecture, three deliverable layers, anonymized live read (May 8, 2026)

## Design

The visual system that runs across every Radar surface — typography hero, four-color palette, three-family type system, V39.1 stat-block component treatment, deliberate rejection of SaaS chrome:

- [**Brand system**](./design/brand-system.md) — colors, typography, components, what the system rejects

## Engineering patterns

Generalized patterns extracted from Radar functions, documented for reuse and review:

- [**Verify-before-write with Stripe**](./patterns/stripe-verify-before-write.md) — re-verify payment server-side at every payment-gated write
- [**Idempotent submissions**](./patterns/idempotent-submissions.md) — two-layer collapse via pre-check + unique-constraint catch
- [**Real-time buy alerts**](./patterns/realtime-buy-alerts.md) — Stripe webhook → Telegram, fire-and-forget with 200-to-Stripe regardless of chat success

## Architecture

Deeper write-ups of how individual layers of the platform are built:

- [**Single-funnel architecture**](./architecture/single-funnel.md) — V44 active architecture: two free diagnostic entry points feeding one free terminal diagnostic, with a single director-tier engagement above the free stack
- [**The admin portal**](./architecture/admin-portal.md) — `/admin` operator command center: CRM, DM pipeline, connections classifier, content intel, calendar, accounts. Where the platform actually gets run.

## Glossary

Definitions for terms used across the build log, design system, and patterns — product / methodology / engineering / design vocabulary in one place: [glossary.md](./glossary.md).

## Frameworks

The intervention methodology used in Radar client engagements:

- **3C Framework (Career Chessboard)** — tactical positioning methodology: CEO of Your Own Career, Consultant Mindset, Control the Narrative
- **Abundance Mindset Reframe (AMR)** — psychological operating system for career operators: data-driven market awareness, pipeline of power, scarcity signaling, owning the close

Frameworks are delivered to Full System buyers as part of the Radar engagement.

## Operating principles

A few non-obvious principles that show up across the build log and the patterns directory:

- **Audit before scope.** Before writing any code on a new release, enumerate every surface, function, and column the change might touch. Most of the work that *doesn't* need to happen gets carved out at this step. (See [V40](./build-log/v40-funnel-friction.md) and [V41](./build-log/v41-radar-decode.md).)
- **Ship in phases with explicit checkpoints.** Each phase produces a verifiable artifact before the next phase begins. Schema → function → UI → deploy → end-to-end.
- **Verification beats trust.** Client-side fields are inputs, not facts. Stripe sessions are verified server-side at the moment of write. Webhooks are verified by signature, not by URL secrecy.
- **Idempotency at the boundary.** Any write that can be retried (and they all can) is idempotent on a unique business identifier — pre-check plus unique-constraint catch.
- **Real-time signal substitutes for QA when launch windows are tight.** Telegram alerts on every paid Stripe session converted production traffic into the test cycle when the V41 launch window was shorter than a full E2E rehearsal.
- **Backwards-compatible field removal.** When removing data from a flow, stop writing the column, leave it nullable, update downstream readers — don't drop it. (See [V40](./build-log/v40-funnel-friction.md).)

## License

The contents of this public repo (build logs, design-system documentation, engineering patterns, and architectural notes) are © 2026 Marquise Jones. The Radar product, diagnostic engine, archetype framework, and all proprietary methodology are not licensed for external use.
