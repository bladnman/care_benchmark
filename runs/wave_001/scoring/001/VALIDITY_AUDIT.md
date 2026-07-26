# VALIDITY_AUDIT - CARE run 001

## ID leakage

Mechanical check for gold/taxonomy IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`: **no hits** in the frozen reconstruction. No cross-reference to PLAN was needed because the reconstruction had no such tokens.

## Vocabulary check

| Reconstruction phrase | PLAN support | Verdict |
|---|---|---|
| "wrong thing hard to build" | PLAN 0: same phrase in the opening implementation approach | plan-derived |
| "load-bearing" | PLAN 4.2 uses this for missing streak/user-count columns | plan-derived |
| "affective metric, not a perf metric" | PLAN R8 uses the phrase for first-bird timing | plan-derived |
| "same product, not two products glued together" | PLAN 11.1 uses this wording for narration/state sharing | plan-derived |
| "a policy alone is not a boundary" | PLAN 12.1 uses the phrase for telemetry architecture | plan-derived |
| "north-star engagement metric" | PLAN 14.4 explicitly refuses one | plan-derived |
| "feature-level fidelity" / "system-level fidelity" / "multi-layer recovery" / "weight-3" | Not present in RECONSTRUCTION | no scorer-side vocabulary |

## Heading mirror

RECONSTRUCTION headings are: `System-level intent`, `Per-feature whys`, then plan-shaped groups: Scope and product boundary, Architecture, Domain model and data model, Simulation engine, API surface and sync model, Frontend rendering pipeline, Field notebook, Audio pipeline, Accessibility surfaces, Privacy and telemetry boundary, Testing/tooling/rollout/risks. These mirror PLAN section domains, not GOLD_WHYS sections or the S/F taxonomy.

## 1:1 mapping suspect

No. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold order. It is expansive and near-plan-ordered, with 14 system principles and many feature bullets grouped under the plan's implementation domains. That is not a neat 1:1 mapping to the held-out gold list.

## Plan-derivation spot check

1. Reconstruction: "Affective requirements should become engineering mechanisms." PLAN support: section 0 says the PRD is opinionated about feel and the plan converts those into structures that make wrong things hard to build; section 13.3 encodes product integrity as CI.

2. Reconstruction: "No visit, session, streak, days-active, or score columns" are load-bearing. PLAN support: section 4.2 has the same schema-absence list and explains why missing columns prevent a future streak counter from becoming easy.

3. Reconstruction: accessibility is "the product, not a fallback" and should be judged by whether it felt alive. PLAN support: section 11.7 makes that acceptance question a launch blocker, and M4 ships accessibility before launch.

## Verdict

**PASS.** The reconstruction is detailed, but the detail traces back to the plan's own headings, phrases, and guardrail vocabulary. I found no gold ID leakage, no scorer-side terminology, no heading mirror to the gold list, and no suspicious 1:1 mapping.
