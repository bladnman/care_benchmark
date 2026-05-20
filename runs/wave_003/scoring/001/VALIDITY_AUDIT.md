# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found.

Searches for gold-style IDs and rubric markers in the frozen reconstruction found no S1-S9, F1-F40, R-Fxx, weight-tier, multi-layer, fidelity, rubric, or gold-list language. The reconstruction uses plan-native headings and feature names rather than gold IDs. A cross-check against PLAN is therefore not needed for any offending ID because there were no offending IDs in RECONSTRUCTION.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Concern |
|---|---|---|
| "relationship deepening over engagement pressure" | PLAN.md:10 says "relationship deepening" and "rather than user interaction metrics"; PLAN.md:453 says "avoiding gamified engagement triggers" | no concern |
| "quiet ambient consequences" | PLAN.md:28 says neglect results in "quiet ambient behavior" | no concern |
| "calm, non-gamified product surface" | PLAN.md:27 bans gamification elements and displayed metrics | no concern |
| "server-authoritative continuity" | PLAN.md:36 says server is the single source of truth; PLAN.md:315 says clients never submit absolute values | no concern |
| "privacy and hidden internal mechanics" | PLAN.md:82 names PII leakage; PLAN.md:165 hides raw personality values; PLAN.md:443-446 excludes telemetry fields | no concern |
| "naturalist observer voice" | PLAN.md:17 says naturalist field-notebook voice; PLAN.md:403-405 gives cohesive prose narration | no concern |
| "equivalent access" | PLAN.md:399 says narration provides an "equivalent experience" | no concern |
| "procedural, smooth, living ambience" | PLAN.md:301-309 call grammar, PLAN.md:342-348 idle motion, PLAN.md:363 synthesized procedural audio | no concern |

No suspect rubric-side vocabulary such as "multi-layer recovery", "feature-level fidelity", "weight-3", "gold why", or "intent fidelity" appears in RECONSTRUCTION.

## Heading Mirror Check

RECONSTRUCTION headings:

| Reconstruction heading | Closest source shape | Assessment |
|---|---|---|
| `## System-level intent` | Required reconstruction structure from phase 2A, not a gold title | acceptable |
| `## Per-feature whys` | Required reconstruction structure from phase 2A, not a gold title | acceptable |
| `### Scope and Boundaries` | PLAN.md section 1 | plan-derived |
| `### System Architecture, Data Model, and API Surface` | PLAN.md sections 2-4 | plan-derived |
| `### Simulation Engine and Sync` | PLAN.md sections 5-6 | plan-derived |
| `### Frontend Rendering, Audio, Accessibility, and Observability` | PLAN.md sections 7-10 | plan-derived |

The headings do not mirror GOLD_WHYS titles such as "feels-alive-not-robotic", "presence-definition", or "drift-function". They mirror the candidate plan's section organization.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not enumerate S1-S9 or F1-F40, does not contain 49 neat entries, and does not follow the gold list order. It follows the plan order: scope, system/data/API, simulation/sync, frontend/audio/accessibility/observability, and risks. Some bullets align with gold-covered features because the plan itself covers those features, but the structure is not a gold-list mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "This shows up in Bird Adoption & Population Scaling, where more birds unlock by calendar age ... rather than user interaction metrics." | PLAN.md:10 says additional birds are paced by calendar age rather than user interaction metrics; PLAN.md:453 says growth avoids gamified engagement triggers. | supported |
| "The sync model says the client never submits absolute state values; instead, an event queue and server tick dedupe and apply deltas." | PLAN.md:315-321 explicitly rejects absolute client values and uses an event queue with server deduped delta updates. | supported |
| "Screen-reader narration is described as an equivalent experience; reduced motion replaces paths with slow cross-fades." | PLAN.md:399 describes an equivalent experience; PLAN.md:350-357 defines reduced-motion cross-fades. | supported |

## Verdict

PASS.

The reconstruction reads as a plan-derived synthesis. It contains no gold ID leakage, no rubric vocabulary, no near-1:1 gold mapping, and its articulate claims are supported by PLAN passages. Some wording is polished and abstract, but it is traceable to the candidate plan rather than to the gold list.
