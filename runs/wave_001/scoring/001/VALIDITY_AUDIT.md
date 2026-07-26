# VALIDITY_AUDIT - CARE run 001

## ID leakage

PASS. A mechanical scan of the frozen reconstruction found no gold IDs such as F1-F40, S1-S9, R-F*, or R-S*. The reconstruction uses only the required top-level headings and PLAN-derived section names.

## Vocabulary check

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`. Sampled phrases are plan-derived:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "mechanically impossible" | PLAN introduction uses the same phrase for invisible failures. | ok |
| "continuing without the viewer" | PLAN section 2 and first-frame sections repeat this phrase. | ok |
| "quiet field" | PLAN sections 8.4 and 8.8 use this loading/adoption visual. | ok |
| "architectural absence" | PLAN section 1.3 uses this for refusing aggregation. | ok |
| "server owns discrete facts; client owns continuous ornament" | PLAN section 3.2 states this rule directly. | ok |
| "multi-device sync is the absence of a problem" | PLAN section 7.1 states this directly. | ok |
| "actual product, not a stripped variant" | PLAN section 11 states this directly for accessibility. | ok |
| "the product's temperament" | PLAN section 15.6 uses this for drift/mood constants. | ok |

## Heading mirror

PASS. The only `##` headings are the phase-2A required headings: `System-level intent` and `Per-feature whys`. The `###` headings mirror PLAN sections, such as `Scope`, `Architecture`, `Data model`, `API surface`, `Simulation engine`, and `Risks`. They do not mirror the gold-list headings S1-S9/F1-F40 or the rubric structure.

## 1:1 mapping suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve gold order, and does not provide a neat one-row-per-gold-why structure. Instead it follows the PLAN's table of contents and includes many implementation items beyond the 49 scored whys. That shape is consistent with a blind reconstruction from PLAN.md.

## Plan-derivation spot check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "The plan treats announcement surfaces as antithetical to the product." | PLAN section 2: no toast/banner/modal primitives, greeting is the only session-start surface; section 9.7 refuses audio prompts. | supported |
| "The server owns discrete facts; the client owns continuous ornament." | PLAN section 3.2 uses this sentence and explains server/client ownership. | supported |
| "The plan says accessible users get the actual product, not a stripped variant." | PLAN section 11 opens with this accessibility stance and implements it through shared renderer/voice surfaces. | supported |

## Verdict

PASS. I found no meaningful contamination signatures. The reconstruction contains no gold IDs or scorer-side terminology, follows PLAN organization rather than gold-list organization, and its load-bearing phrases trace back to PLAN.md. Scores can be treated as valid for this run.
