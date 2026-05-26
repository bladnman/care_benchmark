# VALIDITY_AUDIT - CARE run 001

## Gold ID leakage check

No gold IDs were found in the frozen reconstruction. I checked for S/F-style gold IDs and rubric IDs such as `F1`, `S1`, and `R-F01`; the reconstruction does not use them. The only rubric-adjacent phrase hit was `load-bearing`, which also appears in PLAN.md in the final sentence, so it is plan-derived rather than leakage.

## Vocabulary check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "load-bearing invariants" | yes | PLAN uses "load-bearing invariants across every layer". |
| "notice, never announce" | yes | Directly in PLAN final invariant. |
| "presence honesty" | yes | Directly in PLAN final invariant. |
| "No ownership of canonical bird state" | yes | Directly in PLAN Architecture. |
| "Server is sole writer of vectors" | yes | Directly in PLAN Architecture/Data. |
| "designed alternative surface (not stripped)" | yes | Directly in PLAN Accessibility. |
| "Time-to-first-bird <500ms" | yes | Directly in PLAN Performance. |
| "rule_without_why" / "multi-layer recovery" / "feature-level fidelity" | no | These rubric terms do not appear in the reconstruction. |

No suspicious rubric-side vocabulary is used freely. The reconstruction vocabulary tracks PLAN language closely.

## Heading mirror check

| Reconstruction heading | Comparison to gold/rubric headings | Assessment |
|---|---|---|
| `## System-level intent` | Similar to gold section `System-level whys`, but also the expected reconstruction structure. | Not suspicious. |
| `## Per-feature whys` | Similar to gold section `Feature-level whys`, but generic and expected for phase 2A. | Not suspicious. |

The headings mirror the phase task shape, not the gold list's detailed taxonomy.

## 1:1 mapping suspect check

The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not produce a neat 49-item mapping. Its per-feature section follows the PLAN's own section order: Scope, Architecture/Data, API, Simulation, Sync, Frontend, Audio, Accessibility, Performance, Rollout/Risks. Several gold targets are absent or marked `NOT RECOVERABLE FROM PLAN`. This is not a 1:1 gold-list mirror.

## Plan-derivation spot check

| Reconstruction sentence | PLAN support | Verdict |
|---|---|---|
| "The client is a 'thin renderer + input capture + state subscriber' with 'No ownership of canonical bird state.'" | PLAN Architecture uses the same phrases. | Supported. |
| "Accessibility appears in Scope as 'screen-reader narration, reduced-motion mode, call captions, WCAG AA.'" | PLAN Scope lists exactly those accessibility items. | Supported. |
| "The plan uses a 'procedural bird engine,' 'WebAudio procedural synthesis,' 'motif library per species,' 'personality-shaped timing/pitch variations,' and 'client-generated ornaments.'" | PLAN Scope, Audio Pipeline, Simulation Engine, and Frontend contain those phrases. | Supported. |

## Verdict: PASS

The reconstruction reads as plan-derived. It uses the PLAN's wording and structure, contains no gold ID leakage, avoids rubric scoring vocabulary, and does not map neatly onto the gold target list. Minor heading overlap is expected from the phase-2A output format and is not enough to flag the run.
