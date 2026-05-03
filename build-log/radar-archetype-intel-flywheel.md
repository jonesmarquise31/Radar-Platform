# Radar Archetype Intel Flywheel

**Shipped:** May 3, 2026  
**Surface area:** /quiz (free diagnostic), /rds ($97 paid diagnostic)  
**Deploy hash:** 319ea07

## What It Is

A self-sharpening data layer. Every completed diagnostic feeds a population aggregate. Every new diagnostic reads from that aggregate to ground its output in real data.

## The Problem It Solves

Diagnostics that don't know what they've seen before deliver generic output. A platform with hundreds of completed diagnostics should not produce the same diagnosis as a platform with one. The flywheel makes the dataset itself a first-class part of every output.

## Architecture

Three layers:

1. **Classification layer** — Deterministic rules score every completed diagnostic across a set of positioning dimensions and assign one of six pattern archetypes. No model in the loop, no judgment calls at runtime.

2. **Aggregation layer** — A scheduled job rebuilds a global snapshot of the entire diagnosed population: distribution by archetype, average composite index, top positioning gaps, dominant blocking factors. Runs every six hours.

3. **Injection layer** — Every diagnosis call (free and paid) reads the latest snapshot and injects it into the model context before generating output. The diagnosis is grounded in real population data, not training priors.

## Build Decisions Worth Calling Out

- **Deterministic classification, not a model.** Rules are auditable, reproducible, and free of inference cost. The model still produces the diagnosis text; the classification that grounds it stays out of the model's hands.
- **Single-row aggregate for v1.** Multi-segment aggregation (industry × region × experience) is a future build. At current data volume it would slice the dataset thinner than the signal warrants.
- **Empty-table fallback.** If the aggregate has not yet refreshed, diagnoses ship without injection rather than fail. Silent degradation, no fabricated stats.
- **Divergence logging.** The deterministic classifier runs alongside the model's own classifier on the paid path. Disagreements are logged but not blocked. Diverging cases become tuning signal over time.
- **Database-native scheduler.** The refresh is pure SQL, scheduled inside the database itself. Adding an external workflow tool was unnecessary surface area.

## Outcome

Live in production as of May 3, 2026. First aggregate snapshot covers 56 diagnosed professionals. Diagnoses now reference the real dataset — buyers see themselves placed inside a measurable population, not handed a solo result.

## What It Unlocks

Every future diagnostic submission improves the next one. The more the platform is used, the sharper its output becomes. The dataset itself is now infrastructure.
