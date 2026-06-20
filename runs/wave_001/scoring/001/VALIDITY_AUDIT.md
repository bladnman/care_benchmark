# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found.

I searched the frozen reconstruction for gold-side identifiers such as `S1`-`S9`, `F1`-`F40`, and `R-Fxx` patterns. No gold IDs appear in `RECONSTRUCTION.md`. Because there were no hits, there was no offending ID to cross-reference against `PLAN.md`.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Relationship deepening, not attention metrics" | PLAN Rollout: bird pacing "matches relationship deepening, not attention metrics" | Plan-derived |
| "Canonical state sovereignty" | PLAN Sync Model heading uses the same phrase | Plan-derived |
| "same affective core" | PLAN reduced-motion section uses the phrase | Plan-derived |
| "checklist parity" | PLAN scope says accessibility surfaces are designed for charm, not checklist parity | Plan-derived |
| "product opening promise" | PLAN performance risk says >500ms breaks the product opening promise | Plan-derived |
| "Privacy boundary separates operational health from user relationship data" | PLAN performance boundaries uses the same sentence | Plan-derived |
| "Server is only source of truth" | PLAN canonical state sovereignty says this directly | Plan-derived |
| "graceful silence with captions on by default" | PLAN WebAudio fallback says this directly | Plan-derived |

I found no rubric-only vocabulary such as "intent fidelity", "feature-level fidelity", "multi-layer recovery", "weight-3", or gold IDs. The one phrase that looked benchmark-like, "checklist parity", is present in the assigned PLAN.

## Heading Mirror

| Reconstruction heading | Closest PLAN heading | Gold/rubric mirror concern |
|---|---|---|
| `## System-level intent` | Phase-two reconstruction structure, not PLAN | Expected scaffold, not gold leakage |
| `## Per-feature whys` | Phase-two reconstruction structure, not PLAN | Expected scaffold, not gold leakage |
| `### Scope` | `## Scope` | Mirrors PLAN |
| `### Explicit out-of-scope boundaries` | `**Explicit out-of-scope**` | Mirrors PLAN |
| `### Deliberately limited v1 details` | `**In scope but deliberately limited**` | Mirrors PLAN |
| `### Architecture` | `## Architecture` | Mirrors PLAN |
| `### Data model and API surface` | `## Data Model` / `## API Surface` | Mirrors PLAN grouping |
| `### Simulation engine and sync model` | `## Simulation Engine Design` / `## Sync Model` | Mirrors PLAN grouping |
| `### Frontend rendering, audio, accessibility, and rollout` | `## Frontend Rendering Pipeline`, `## Audio Pipeline`, `## Accessibility Surfaces`, `## Rollout` | Mirrors PLAN grouping |

The headings do not echo gold-list section titles or S/F labels. They mostly compress PLAN headings.

## 1:1 Mapping Suspect

Verdict: not suspect.

The reconstruction does not list S1-S9 or F1-F40, does not preserve the gold order, and does not create a neat 49-item mapping to the gold whys. Its per-feature bullets follow the PLAN section order and include many items outside the 40 gold why anchors. That shape is consistent with derivation from PLAN rather than from the gold list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Relationship deepening, not attention metrics." | PLAN bird ramp: "Pacing matches relationship deepening, not attention metrics"; relationship safety flags gamification and counters. | Supported |
| "Server-owned canonical state." | PLAN Architecture says the server owns all mutable state; Sync Model says "Server is only source of truth for personality vectors, moods, and drift." | Supported |
| "Accessibility is part of the affective core." | PLAN scope says accessibility is designed for charm; reduced-motion says "same affective core"; rollout ships all accessibility surfaces with product. | Supported |

The sampled articulate sentences are traceable to exact PLAN language.

## Verdict

PASS.

The frozen reconstruction reads as plan-derived: no gold IDs leaked, vocabulary is supported by the assigned PLAN, headings mirror PLAN structure rather than the gold list, and the reconstruction is not arranged as a 1:1 gold-target map. Scores should not be treated as contamination-suspect.
