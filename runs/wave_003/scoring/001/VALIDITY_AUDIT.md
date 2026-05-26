# VALIDITY_AUDIT - CARE run 001

## 1. ID leakage check

Verdict: no gold-ID leakage found.

Search hits in the frozen reconstruction found no S1-S9, F1-F40, R-Fxx, weight-3, multi-layer, feature-level fidelity, or intent-fidelity identifiers. The only rubric-like sentinel repeated in reconstruction is `NOT RECOVERABLE FROM PLAN`, which is an expected phase-2A output marker rather than gold taxonomy. The phrase `load-bearing` appears once in the reconstruction, but it also appears in the PLAN heading `Drift function (the load-bearing algorithm)`, so it is plan-derived rather than leakage.

## 2. Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "load-bearing algorithm" | PLAN §5 heading: "Drift function (the load-bearing algorithm)" | Plan-derived. |
| "No negative deltas" | PLAN §5 drift rule: "No negative deltas" | Plan-derived. |
| "Clients are dumb renderers" | PLAN §6 multi-device coherence uses the same phrase. | Plan-derived. |
| "aliveness spell" | PLAN §12 audio risk uses "aliveness spell." | Plan-derived. |
| "state-list readout" | PLAN §12 accessibility risk uses "state-list readout." | Plan-derived. |
| "quiet field" | PLAN §7 loading state uses "quiet field." | Plan-derived. |
| "no client-side state to merge" | PLAN §6 says there is no client-side state to merge. | Plan-derived. |
| "per-bird engagement funnels" | PLAN §10 deliberately does not measure them. | Plan-derived. |
| "multi-layer", "weight-3", "feature-level fidelity" | No hits in reconstruction. | No rubric-vocabulary concern. |

## 3. Heading mirror check

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then plan-shaped sections: Scope, Architecture, Data Model, API Surface, Simulation Engine Design, Sync Model, Frontend Rendering Pipeline, Audio Pipeline, Accessibility Surfaces, Performance Budgets and Observability, Rollout, Risks and Mitigations, Appendix Ambiguity Resolutions.

These mirror the PLAN's own section headings, not the gold-list headings. The only overlap with the gold list is the expected generic division between system-level and per-feature rationale. The reconstruction does not mirror S1-S9 or F1-F40 titles.

## 4. 1:1 mapping suspect check

Verdict: not suspect.

The reconstruction does not create a neat S1-S9 and F1-F40 sequence. It follows the PLAN's implementation outline and includes many non-gold implementation items such as API endpoints, render pipeline, rollout phases, and ambiguity resolutions. It also marks multiple plan items as `NOT RECOVERABLE FROM PLAN`, which is the opposite of a suspicious gold-aligned fill-in pattern.

## 5. Plan-derivation spot check

1. Reconstruction: "The client is a viewer and event emitter, not a simulation participant."  
   PLAN support: §2 boundary rule says the client renders snapshots, never computes drift, never writes personality values, and is "a viewer and event emitter."

2. Reconstruction: "The drift function is named 'the load-bearing algorithm' and 'the product's central promise.'"  
   PLAN support: §5 heading names the drift function "the load-bearing algorithm"; §12 drift calibration risk calls it the product's central promise.

3. Reconstruction: "Reduced motion preserves the feeling of aliveness while replacing frame-by-frame animation with cross-fades."  
   PLAN support: §7 reduced-motion mode replaces animations with cross-fades; §12 says reduced motion must not ship as "animations off" or break aliveness.

All three articulate sentences are directly grounded in the PLAN.

## 6. Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no rubric taxonomy, no 1:1 gold-order mapping, and its distinctive phrases are supported by the PLAN. Minor overlap with scorer vocabulary such as `load-bearing` is not a contamination concern because the exact phrase is present in the PLAN.
