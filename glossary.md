# Glossary

Terms used across the Radar build log, design system, and pattern documents. Listed in rough order from product / methodology terms to technical / build terms, then internal vocabulary.

---

## Product and methodology

**Workforce Radar.** The career intelligence platform documented in this repo. Diagnoses how the modern hiring system reads or misreads skilled professionals. Three portals serve three buyer types: professionals, career coaches, hiring teams. Live at [workforceradar.com](https://workforceradar.com).

**The Radar Decode.** The standalone $47 entry-tier product. A one-page operator brief, written by hand, anchored to a buyer's actual LinkedIn profile and resume. Delivered as a PDF within 48 hours of purchase. Live at [radar-decode.netlify.app](https://radar-decode.netlify.app). The full launch context is in the [V41 build log](./build-log/v41-radar-decode.md).

**RDS.** The $97 mid-tier product. The full Radar diagnostic: structured intake, archetype classification, and a written report naming the specific signals causing the buyer to be filtered or undervalued in their target market. Stands for "Radar Diagnostic System."

**Full System.** The $1,497 engagement tier. Combines the diagnostic with delivery support and the application of the 3C Framework and Abundance Mindset Reframe to the buyer's actual situation.

**Archetype.** One of six classifications produced by the diagnostic. Each archetype names a distinct *signal pattern* that recruiters and hiring systems read off a candidate's surface (LinkedIn, resume, public history). The archetype framework itself is proprietary and is not published.

**Signal-layer.** The layer of meaning the hiring system reads from a candidate's surface artifacts — LinkedIn profile structure, resume formatting, headline phrasing, project descriptions, recommendation patterns. Signal-layer misalignment is when the candidate's actual capability is being parsed incorrectly by recruiters and ATS systems because of how the surface artifacts present, not because of the underlying skills. Radar diagnoses this layer specifically.

**Operator.** The audience term for Radar's professional buyer. Operator-class IT, cybersecurity, or finance professional — typically mid-to-senior, technically credible, frustrated by being filtered or underpaid. The brand voice ("operator brief," "operator intelligence") is calibrated to this reader.

**3C Framework.** Also called Career Chessboard. The tactical positioning methodology used in Radar engagements: **CEO of Your Own Career** (own the strategy), **Consultant Mindset** (sell the outcome), **Control the Narrative** (steer how the market reads you). Delivered to Full System buyers.

**Abundance Mindset Reframe (AMR).** The psychological operating system layer of the engagement: data-driven market awareness, pipeline of power, scarcity signaling, owning the close. Companion to the 3C Framework.

## Build and engineering

**Build log.** The /build-log directory. Major releases each get an entry naming the decision, the architecture, the trade-offs, and what was learned. See [V40](./build-log/v40-funnel-friction.md) and [V41](./build-log/v41-radar-decode.md).

**Audit phase.** The first phase of any Radar release. Read-only enumeration of every form, function, and column the change might touch — *before* writing code. Most of the work that doesn't need to happen gets carved out at this step. Used in V40 and V41 (see both build log entries).

**Phased deploy.** The release pattern: schema → function → UI → deploy → verification, with explicit checkpoints between phases. Each phase produces a verifiable artifact before the next phase begins.

**Intake form.** The post-payment form that buyers complete after Stripe Checkout to provide artifacts (LinkedIn URL, resume) needed for delivery. On the Decode, the form lives at `/thanks` and is gated on a valid Stripe `session_id` from the redirect URL.

**Verify-before-write.** A required pattern: before any database write that depends on a Stripe payment, retrieve the session via the Stripe API and confirm it is real, paid, and matches the expected price. Documented as a [pattern](./patterns/stripe-verify-before-write.md).

**Idempotent submission.** A submission flow that can be retried, double-submitted, or hit by concurrent requests without producing duplicates or partial state. Achieved through a pre-write lookup plus a unique-constraint catch at the database level. Documented as a [pattern](./patterns/idempotent-submissions.md).

**Orphan rate.** The rate at which a buyer pays but does not complete the post-payment intake form (data lost across browser tabs, devices, or cleared storage). Introduced as an architectural metric in [V40](./build-log/v40-funnel-friction.md). Watched as a signal, not assumed away.

**Buy alert.** A real-time Telegram notification fired by a Stripe webhook the moment a `checkout.session.completed` event arrives. Includes amount, buyer email, time, and truncated session ID. Documented as a [pattern](./patterns/realtime-buy-alerts.md).

**Live mode.** Stripe's production mode, where charges are real money. Distinguished from test mode. The Decode launched directly in live mode — the V41 build log explains the trade-off.

## Design system

**Operator brief.** The visual posture documented in the [brand system](./design/brand-system.md). Reads like an internal operations document — terse, dense, signal-forward — rather than like a marketing site. Typography is the hero; SaaS chrome (gradient backgrounds, illustrated heroes, feature grids with green checkmarks) is explicitly rejected.

**V39.1 stat block.** The component treatment used for "WHAT YOU GET" lists, HOW IT WORKS steps, and any structured callout. Specs: 3px gold left border, 20px left padding, navy background, Cormorant Garamond white 18px text. Named "V39.1" because it was introduced in that release of the brand system. In code: `.stat-block` (and `.step-block` for the numbered variant). Specs in [brand-system.md](./design/brand-system.md).

**Brand bar.** The two-element flex row at the top of every Radar surface: page-constant `WORKFORCE RADAR` mark on the left, page-specific slug (e.g. `DECODE / 001`) on the right, both in DM Mono gold. Specs in [brand-system.md](./design/brand-system.md).

**Reassurance line.** The small DM Mono gold line that sits directly under each primary CTA, naming the operational reality of what the buyer is committing to (e.g. `48-HOUR TURNAROUND  ·  PDF DELIVERY  ·  ONE PAGE`). Always uses `·` (middle dot) as the separator. Em-dashes are not used as separators in this system.

## Internal vocabulary

**Surface.** Any user-visible artifact that carries the brand: a page, a deliverable PDF, an outbound email template, a social card. New surfaces inherit the brand system without modification.

**Tier.** One of the three pricing levels in the Radar product line: Decode ($47), RDS ($97), Full System ($1,497). Used both in pricing language and in describing where a buyer sits in the funnel.

**Tighten.** Common verb in the build log: to remove copy, layers, or fields rather than add them. Default move when a flow feels noisy.
