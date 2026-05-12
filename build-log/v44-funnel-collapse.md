# V44 — Funnel Collapse + Director-Tier Offer

**Date:** May 2026

## What Changed

The dual-lane offer stack introduced in V42 has been retired. The platform now operates a single funnel: two free diagnostic entry points feeding one free terminal diagnostic, with a single premium engagement positioned above the free stack.

## Why

The dual-lane stack carried five paid surfaces: an entry-tier paid diagnostic, two strategy-session upsells (one per lane), an engagement-tier paid product, and a recurring subscription tier. Each surface justified its own sales page, checkout flow, post-purchase routing, and operator follow-up.

The result was conversion friction without a corresponding conversion lift. Buyers who came in through a free diagnostic faced three different paid CTAs across the system, each with its own price point and its own value framing. None of the paid tiers ran at the cadence that would have justified its existence as a separate product.

V44 cuts the offer stack down to its load-bearing parts:

- The diagnostic layer is what the platform's dataset is built on. It stays free, and it stays the primary surface.
- The engagement layer is what operators actually buy when they're ready to move. It consolidates into a single product positioned at the level the engagement actually delivers value at: director and above.

Everything between the free diagnostic and the director-tier engagement was load that the platform was carrying without revenue justification. V44 stops carrying it.

## The Free Funnel

Two free entry points feed one free terminal diagnostic. The entry points are the existing five-minute reads on the two diagnostic axes: how a professional is being read, and how a professional is being valued. The terminal diagnostic is the full archetype classification, account-gated and free.

The free funnel keeps the platform's data flywheel intact. Every submission compounds the dataset; every diagnosis reads from that dataset before generating output. The free tier is the platform's primary asset and stays priced at the level its actual function justifies: zero.

## The Radar Special

The premium engagement is a four-week intelligence engagement for directors, VPs, and C-suite operators. It is built around a structural problem the platform has documented across its dataset: senior operators running scope at one altitude while being read by the market at a different altitude. The engagement repositions the operator's signal layer to match the work they are actually doing.

Access is by waitlist only. Cohorts are capped. The platform makes no attempt to scale the engagement — the offer is deliberately constrained because the work is delivered by one operator, in calendar time, against the buyer's actual market position.

The waitlist surface is shipped. The backend that captures waitlist submissions ships in the next phase.

## What This Retires

V44 retires the following surfaces and their associated infrastructure:

- The entry-tier paid diagnostic page
- Both strategy-session upsell products
- The engagement-tier paid product page
- The standalone strategy-call booking page
- The legacy diagnostic sales pages from the pre-dual-lane era
- The associated checkout, intake, and confirmation routes for each retired product
- The dashboard's lane-progress tracker

The free funnel surfaces — the two entry quizzes and the terminal diagnostic — are unchanged. The dashboard continues to surface a buyer's results across both diagnostic axes; what's removed is the funnel-progress tracker that pointed at retired paid surfaces.

## Decisions Carried Forward

A few decisions from prior releases continue to hold under V44:

- **The diagnostic is a dataset feeder, not a revenue tier.** Pricing the diagnostic gates the asset that matters most to the platform. V42's "free entry for both lanes" carries forward.
- **Account creation as currency on the terminal diagnostic.** V42's account-gating on the full diagnostic stays. The terminal diagnostic remains free in exchange for sign-up.
- **Cross-axis result surfacing.** Buyers who complete both diagnostic axes see results across both on a single dashboard surface.

## Decisions Reversed

A few decisions from V42 are explicitly reversed:

- **Two strategy-session products.** V42 introduced a separate strategy-session product per lane to preserve lane-specific metadata for downstream analytics. V44 retires both products. The volume of those purchases was below the threshold where the metadata distinction justified the operational overhead of running two separate products.
- **Dashboard as a lane-progress tracker.** The prior dashboard rebuild surfaced buyer progress across both lanes as a stack of step cards pointing at paid CTAs. With the paid CTAs removed, the tracker had no surface to point at. The dashboard returns to its V41-era function: a results display, not a sales surface.

## Outcome

The platform now runs a simpler line: free funnel into one premium engagement. The dataset compounds across the free tier exactly as it did before. The premium tier is positioned at the altitude where the engagement actually creates leverage.

The intent is not to maximize the number of paid offers visible to a buyer. The intent is to make the one paid offer that exists carry the weight that five separate offers were failing to carry.

---

© 2026 Marquise Jones. All rights reserved.
