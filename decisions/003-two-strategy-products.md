# 003 — Two Strategy Session Products, Not One

**Date:** May 3, 2026
**Status:** Accepted

## Context

The strategy session upsell tier sits above both diagnostic lanes. The decision was whether to model it as a single product (one offer, two referring lanes feeding it) or two parallel products (one per lane, each with its own metadata, copy, and landing page).

A single product is simpler operationally — one product card, one price, one webhook code path. Two products preserve lane-specific signal and let the pricing or framing of each evolve independently.

## Decision

Two parallel products, one per lane. Same price point, different metadata, different landing pages, different framing.

## Consequences

- **Webhook routing stays clean.** Payment metadata identifies which lane the buyer came from. The webhook can route to lane-specific downstream actions without inferring the lane from referrer or amount.
- **Lane-specific signal preserved.** Strategy session purchases land in the database with the source lane explicitly recorded. Downstream analytics — which lane converts at higher rates, which lane produces stickier customers, which lane warrants more investment — become possible without retroactive tagging.
- **Per-lane copy and offer framing.** Each lane's strategy session translates a structurally different diagnosis into a structurally different next move. Forcing both into one offer description blurs the value proposition for both.
- **Independent evolution.** Pricing, framing, or product structure can change for one lane without disturbing the other. The two are decoupled by construction.
- **Trade-off acknowledged.** Two product cards in the payment processor dashboard, two price IDs to manage in the runtime config, two webhook branches. The operational overhead is small and the analytic upside justifies it.

---

© 2026 Marquise Jones. All rights reserved.
