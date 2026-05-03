# V42 — Dual-Lane Funnel

**Shipped:** May 3, 2026
**Surface area:** Homepage, /quiz, /rds, /scope (new), /dashboard, /strategy-call (new), /read flow
**Deploy hash:** 16941e3

## What Changed

Radar pivots from a single-lane archetype diagnostic platform to a dual-lane diagnostic system. The original archetype lane (how you're being read) is now joined by a scope lane (how you're being valued). Each lane has its own free entry quiz, a paid product, and a strategy session upsell. Both lanes share the dataset flywheel infrastructure shipped earlier.

## Why This Build

As the dataset grew across late 2025 and early 2026, a second pattern surfaced in commenter and reactor profiles on the underprice content cluster: scope-versus-compensation mismatch was the dominant pain, distinct from the positioning-versus-perception read the original platform addressed. Workers were not only being read incorrectly by hiring systems — they were being valued incorrectly by their current employers. Diagnosing one axis without the other left half of the buyer's actual problem unaddressed.

The platform needed to diagnose both axes, with a single shared dataset compounding across both.

## Architecture

Three layers added to the existing platform:

1. **Scope diagnostic layer.** A free five-question quiz that maps a buyer's actual scope against the title they hold. The classifier is fully deterministic — title is matched against a tier ladder by keyword, scope is aggregated from four orthogonal signals, and the verdict is computed by comparing the two. The output is a one-line verdict across five bands, ranging from aligned to inverted to multiple tiers underleveled. No model in the loop for the classification itself.

2. **Combined context builder.** The dataset injection mechanism, originally fed by a single archetype aggregate, now pulls from both archetype and scope aggregates in parallel. Every diagnosis call across either lane grounds its output in both axes of the population dataset. A buyer taking the archetype quiz sees scope-aware language in their diagnosis; a buyer taking the scope quiz sees archetype-aware language. The dataset is the spine; the lanes are entry points into the same body of intelligence.

3. **Strategy session upsell tier.** A new paid tier above the existing single-product offers. Two parallel products, one per lane, each combining the paid deliverable with a thirty-minute personal session. Webhook routing carries lane-specific metadata, alert routing fires lane-specific notifications, and the row schema preserves the source lane for future analytics.

## Build Decisions Worth Calling Out

- **The full diagnostic becomes free, account-gated.** The original paid full diagnostic now costs nothing in exchange for account creation. The trade is deliberate: account creation captures email and creates an asset that compounds, while the diagnostic itself becomes a dataset feeder rather than a revenue tier. The revenue moves up to the strategy session layer, which is structurally harder to commoditize.

- **Two separate strategy session products, not one.** The archetype lane and scope lane upsell to structurally different sessions — one translates a positioning read into market moves, the other translates a scope gap into a thirty-day plan. Modeling them as separate products in the payment processor keeps webhook routing clean, preserves lane-specific metadata for future analytics, and prevents one product's evolution from constraining the other.

- **Live aggregate stats on the homepage.** Legacy aspirational stats were replaced with real numbers from the production dataset. Smaller numbers, sharper claims. The headline statistic is the dataset size itself; subsequent statistics describe what the dataset actually shows. Honest framing beats round-number framing.

- **Homepage restructured around the dual axes, not the single funnel.** The two-card layout makes the platform's structure legible at first glance: how you're being read on the left, how you're being valued on the right. The previous audience-segmentation layout (three cards by buyer type) was preserved in the markup but hidden — symmetric to the rollback path used in earlier dashboard work.

- **Webhook collision defense was hardened.** Adding a new $197 price point created a second shared price tier (the existing $97 tier already had three products competing for the same amount). The webhook now refuses to route any unmatched-by-metadata payment that lands at a known shared amount, alerting for manual triage instead of guessing the product from the dollar value alone.

## Outcome

Live in production as of May 3, 2026. The platform now operates two diagnostic entry points feeding one dataset, with clean separation between the read-correctly axis and the valued-correctly axis. The homepage and all four diagnostic exit pages are wired with cross-lane CTAs so a buyer entering through one quiz is shown the existence of the other.

A post-deploy fix patched a missing route configuration that had silently broken the new free scope quiz on production. The miss came from per-function route registration in the runtime config; the existing alert wiring in the function itself swallowed the upstream rejection without surfacing, which extended the time-to-detection. Both the route and the alert-handling pattern are flagged for the next build's audit phase.

## What This Unlocks

Two distinct buyer entry points. Two distinct upsell ladders. One shared dataset that compounds across both lanes. Every free quiz submission across either lane sharpens every subsequent diagnosis on either lane.

The dataset was infrastructure when the flywheel shipped. With both lanes feeding it, it now becomes load-bearing in two directions at once.
