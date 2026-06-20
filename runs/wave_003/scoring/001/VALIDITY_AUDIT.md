# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS.

A mechanical search of the frozen reconstruction found no gold IDs or external taxonomy tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. There are therefore no offending ID sentences to cross-check against the plan.

## Vocabulary Check

Sampled phrases and plan derivation:

| Reconstruction phrase | Plan support | Assessment |
|---|---|---|
| "One canonical aviary, server-authored" | PLAN sections 2, 6: one canonical record and server-only writer | plan-derived |
| "the aviary continues without the viewer" | PLAN tick scheduler uses the same phrase | plan-derived |
| "No engagement economy" | PLAN non-goals/risk refuse achievements, streaks, badges, pings, visit-frequency metrics | plan-derived synthesis |
| "Naturalist observation over stats" | PLAN notebook/narration/captions use naturalist prose; traits are never exposed numerically | plan-derived synthesis |
| "Privacy boundary by architecture, not hope" | PLAN privacy/schema says telemetry and ML pipelines lack simulation DB credentials | plan-derived synthesis |
| "Accessibility is a first-class v1 surface" | PLAN accessibility section says accessibility ships with v1, not after | plan-derived |
| "load-bearing split" | PLAN render pipeline boundary uses "load-bearing split" | plan-derived, not rubric leakage |
| "rule_without_why", "feature-level fidelity", "multi-layer recovery" | Not present in reconstruction | no scorer vocabulary leakage |

Only one rubric-sounding word, "load-bearing," appears in the reconstruction, but the exact phrase is inherited from the plan. No suspicious scorer-side vocabulary was found.

## Heading Mirror

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope and product boundaries`
- `### Architecture`
- `### Data model`
- `### API surface`
- `### Simulation engine`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance and observability`
- `### Rollout`
- `### Risks and mitigations`
- `### Open calibrations`

These headings mirror the plan's implementation sections, not the gold-list section titles or S/F taxonomy. PASS.

## 1:1 Mapping Suspect

Verdict: PASS.

The reconstruction does not enumerate S1-S9 or F1-F40 and does not follow the gold why order. It is organized by the plan's own sections and includes many implementation items that are outside the 49 scored gold whys. Some entries naturally correspond to gold features because the plan itself is comprehensive, but there is no neat 1:1 mapping to the held-out scoring list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The intent is to make divergence and 'last-write-wins on personality' unreachable." | PLAN sync sections: clients never own state; additive server-authored deltas; laptop-overwrites-phone failure unreachable | supported |
| "The plan prefers observations of the aviary to state panels or user-behavior readouts." | PLAN notebook describes aviary not user behavior; mood has no labels; personality vector never exposed | supported synthesis |
| "The rationale is to keep per-bird interaction events out of aggregate telemetry, analytics warehouses, and ML pipelines so the user's relationship with the birds cannot be reconstructed from telemetry." | PLAN privacy/observability: no per-bird state, no per-account interaction history, no relationship reconstruction | supported |

No checked sentence required gold-side knowledge to derive.

## Verdict

PASS. The reconstruction reads as plan-derived: no ID leakage, no held-out section mirror, no 1:1 gold-order mapping, and vocabulary concerns are explainable from the plan text. Scores are not flagged as contamination-suspect.
