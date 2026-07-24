# VALIDITY_AUDIT - run 001

## ID Leakage

Verdict: no leakage found.

Mechanical search for gold-id style tokens (`F1`, `S1`, `R-F01`, `R-S01`, etc.) found no matches in either `runs/wave_001/reconstructions/001/RECONSTRUCTION.md` or `runs/wave_001/plans/001/PLAN.md`. The reconstruction does not use the gold IDs or rubric taxonomy.

## Vocabulary Check

Sampled phrases from the reconstruction were traceable to the plan rather than scorer-side vocabulary:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "calm interaction surface" | PLAN scope uses the exact phrase for presence, listen-in, offer, settle, and notebook. | plan-derived |
| "no guilt UX" | PLAN success criteria say return to "quieter, intact birds--no guilt UX." | plan-derived |
| "Server is sole writer" | PLAN hard invariant uses the exact sentence. | plan-derived |
| "Simulation DB is not joined into analytics warehouse" | PLAN hard invariant uses the exact sentence. | plan-derived |
| "first-class art pass" | PLAN reduced-motion section says this exactly. | plan-derived |
| "quiet field sky--not a spinner" | PLAN boot sequence has the same loading-state phrase. | plan-derived |
| "Variation every call so no identical loop" | PLAN call grammar runtime says this exactly. | plan-derived |
| "Age-only, not engagement" | PLAN defensible calls and adoption-offer sections use this phrasing. | plan-derived |
| "Social net gravity" | PLAN risk table uses this failure mode. | plan-derived |
| "TTFA <500ms" | PLAN edge bootstrap callout gives this rationale. | plan-derived |

I also searched for rubric/gold-side terms including `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, and `load-bearing`. No suspicious scorer-side terms appeared in the reconstruction.

## Heading Mirror

Reconstruction headings:

| Reconstruction heading | Comparison | Assessment |
|---|---|---|
| `## System-level intent` | Required by the reconstructor prompt, not a gold-list heading. | ok |
| `## Per-feature whys` | Required by the reconstructor prompt, not a gold-list heading. | ok |
| `### Scope and defensible calls` | Mirrors PLAN section 1.3, not GOLD_WHYS. | ok |
| `### Architecture` | Mirrors PLAN section 2. | ok |
| `### Data model and API surface` | Combines PLAN sections 3 and 4. | ok |
| `### Simulation engine` | Mirrors PLAN section 5. | ok |
| `### Sync model` | Mirrors PLAN section 6. | ok |
| `### Frontend rendering pipeline` | Mirrors PLAN section 7. | ok |
| `### Audio pipeline` | Mirrors PLAN section 8. | ok |
| `### Accessibility surfaces` | Mirrors PLAN section 9. | ok |
| `### Performance and observability` | Mirrors PLAN section 10. | ok |
| `### Interaction implementation notes` | Mirrors PLAN section 11. | ok |
| `### Social visits` | Mirrors PLAN section 12. | ok |
| `### Rollout, privacy, and calibration` | Mirrors PLAN sections 13-16. | ok |

No heading appears to mirror the held-out gold-list sections or why IDs. The per-feature grouping follows the plan's architecture, not the gold order exactly.

## 1:1 Mapping Suspect

Verdict: not suspect.

The reconstruction does not provide a neat S1-S9 or F1-F40 mapping. It has 12 system-level bullets and a long plan-derived per-feature list with items such as API routes, rate limits, tick locks, rendering details, and rollout decisions that are not the gold why list. Several gold why-bearing features are marked `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction rather than target-list matching.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Reduced motion is a 'first-class art pass,' 'not animation: none,' and a11y 'ships day-one with visuals/audio--not v1.1.'" | PLAN §7.5: "This is a first-class art pass"; PLAN §9.5: "A11y ships day-one with visuals/audio-not v1.1." | grounded |
| "Boot starts with 'quiet field sky--not a spinner'; birds paint 'mid-pose immediately'; there is no 'aviary powering on.'" | PLAN §7.1 boot sequence contains all three boot constraints. | grounded |
| "Drift 'never decreases on absence,' personality 'only rises,' bird unlocks are 'Age-only, not engagement,' and calibration targets make week-one changes measurable but week-three changes human-visible." | PLAN §§1.2, 1.3, 5.2, and 17 state non-decreasing drift, age gates, and calibration targets. | grounded |

All three spot-check sentences are directly plan-derived. They do not appear to import gold/rubric wording.

## Verdict: PASS

The reconstruction shows no significant contamination signatures. It contains no gold ID leakage, no scorer-side rubric vocabulary, no heading mirror to the gold list, and no 1:1 mapping to S/F targets. Its structure and phrasing track the PLAN closely, including explicit `NOT RECOVERABLE FROM PLAN` entries where rationale was not available.
