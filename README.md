# Workforce Radar

Public artifacts from the build of [Workforce Radar](https://workforceradar.com) — a career intelligence platform that diagnoses how the modern hiring system reads or misreads skilled professionals.

## What this repo is

This is the public build log. It contains technical write-ups, architecture notes, and selected patterns from the Radar build. The production codebase, diagnostic prompts, classification logic, and customer data live in a private repo and are not published here.

The intent is to show the work without giving away the work.

## What Radar is

Radar diagnoses signal-layer misalignment in the hiring market. A 5-minute diagnostic classifies professionals into one of six archetypes and identifies the specific reasons their target market is reading them incorrectly. The platform serves three buyer types through three portals: professionals diagnosing themselves, career coaches diagnosing their client books, and hiring teams querying a classified candidate pool.

The system was built on data from 250+ classified IT, cybersecurity, and finance professionals. The diagnostic engine reads hiring systems themselves rather than any single industry.

## Architecture

- **Frontend:** Static HTML, vanilla JS, Tailwind CSS
- **Backend:** Netlify Functions (Node.js)
- **Database:** Supabase (Postgres + Auth + RLS)
- **Payments:** Stripe Checkout
- **AI:** Anthropic API (Claude) for diagnostic generation
- **Three portals:** /quiz (professional), /coach (career coaches), /hiring (recruiters/hiring teams)

## Build log

Major releases and the technical decisions behind them are documented in [/build-log](./build-log).

- [V40 — Funnel friction reduction](./build-log/v40-funnel-friction.md) — name-only entry, results gated behind account creation, phone removal across paid flows

## Frameworks

The intervention methodology used in Radar client engagements:

- **3C Framework (Career Chessboard)** — tactical positioning methodology: CEO of Your Own Career, Consultant Mindset, Control the Narrative
- **Abundance Mindset Reframe (AMR)** — psychological operating system for career operators: data-driven market awareness, pipeline of power, scarcity signaling, owning the close

Frameworks are delivered to Full System buyers as part of the Radar engagement.

## License

The contents of this public repo (build logs, write-ups, and architecture notes) are © 2026 Marquise Jones. The Radar product, diagnostic engine, archetype framework, and all proprietary methodology are not licensed for external use.
