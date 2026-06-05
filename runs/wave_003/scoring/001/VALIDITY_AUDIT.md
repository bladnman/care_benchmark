# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A scoped search of the frozen reconstruction for gold IDs and rubric markers (`S1`-`S9`, `F1`-`F40`, `R-F*`, `weight-3`, `feature-level fidelity`, `multi-layer recovery`, `gold`, and `rubric`) returned no hits. There are no offending ID-bearing sentences to cross-reference against PLAN.

## Vocabulary Check

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "core value is felt aliveness over time" | PLAN §1 uses the exact phrase. | plan-derived |
| "small place that continues without the viewer" | PLAN §1 uses the exact phrase. | plan-derived |
| "the bird greeting is the welcome surface" | PLAN §1 and §7 use this rule. | plan-derived |
| "only writer of canonical aviary state" | PLAN §2 assigns this to the simulation service. | plan-derived |
| "not raw ARIA state dumps" | PLAN §10 uses this phrase for narration. | plan-derived |
| "Accessibility ships with v1, not as a patch" | PLAN §10 uses the exact phrase. | plan-derived |
| "quiet field with faint motion cues" | PLAN §8 describes this slow-snapshot state. | plan-derived |
| "privacy architecture is part of the product" | PLAN §11 states this directly. | plan-derived |

Suspicious rubric-side vocabulary was absent. The reconstruction does not use terms such as `intent fidelity`, `feature-level fidelity`, `weight-3`, `multi-layer recovery`, or gold IDs.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Product Interpretation And Scope`
- `### System Architecture`
- `### Core Data Model`
- `### API Surface And Sync`
- `### Simulation Engine Design`
- `### Frontend Rendering Pipeline`
- `### Audio Pipeline`
- `### Accessibility Plan`
- `### Privacy, Performance, Delivery, Copy, Tests, Operations, Security, And Risks`

The `###` headings mirror PLAN section groupings, not GOLD_WHYS section titles. The first two headings are the expected reconstruction scaffold rather than gold-list mirrors. No exact or near-exact gold heading mirrors were found.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping pattern was found. The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve gold order, and includes many plan-derived items outside the 49 scored whys, including architecture, API, milestones, risks, security, and baseline implementation choices. Several bullets are marked `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction behavior rather than gold-list completion.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Performance is part of the feeling of aliveness." | PLAN §8 and §12 connect first-frame motion, no spinner, quiet field, and first-bird timing. | grounded inference from PLAN |
| "Privacy is not only a compliance layer; it is 'part of the product.'" | PLAN §11 says, "The privacy architecture is part of the product." | directly grounded |
| "Accessibility ships as full product behavior, not a patch or fallback." | PLAN §10 says, "Accessibility ships with v1, not as a patch," and reduced motion preserves product behavior. | directly grounded |

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, rubric vocabulary, gold-order mapping, or gold-heading mirror were found. Its strongest phrases are either exact PLAN phrases or reasonable compression of adjacent PLAN passages.
