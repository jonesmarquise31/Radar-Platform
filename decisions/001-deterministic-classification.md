# 001 — Deterministic Classification, Not Model-Based

**Date:** May 3, 2026
**Status:** Accepted

## Context

Radar classifies professionals into archetype clusters from diagnostic submissions. Classification could be model-based (a language model reads the answers and assigns the cluster) or deterministic (fixed rules score the answers and assign the cluster).

The choice has runtime, cost, audit, and product implications. A model-based classifier produces softer, more nuanced judgments but at non-trivial inference cost per submission. A deterministic classifier is rigid but reproducible and free at the runtime margin.

## Decision

Deterministic rules. Each submission's answers map to a known signal level per dimension. Scores aggregate into a misalignment index. Cluster assignment follows a fixed rule order. The model is reserved for generating the diagnostic prose, not for computing the classification.

## Consequences

- **Reproducible.** The same submission produces the same archetype every time. There is no temperature, no sampling variance, no version drift.
- **Auditable.** A submission's classification rationale traces back to specific rule matches. The platform can answer the question "why did this submission get this archetype" with an exact rule trace.
- **Free of inference cost.** Classification consumes no model tokens. Volume scales linearly without a per-submission cost line.
- **Tunable without retraining.** A rule change is a code change. No data labeling, no model fine-tuning, no offline evaluation harness.
- **Trade-off acknowledged.** Rules are blunter than a model's judgment. A divergence log captures cases where the rule-based classifier disagrees with the model's classification of the same submission. Divergences feed ongoing rule calibration.

---

© 2026 Marquise Jones. All rights reserved.
