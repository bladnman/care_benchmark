# VALIDITY_AUDIT - CARE run 001

## ID Leakage

No gold ID leakage found. Searches for `F1`-style IDs, `S1`-style IDs, `R-Fxx`, and rubric terms such as `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, and `load-bearing` returned no hits in either frozen reconstruction or PLAN.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `low-key, observational relationship` | PLAN line 4 uses the exact phrase. | plan-derived |
| `high-fidelity affective depth and minimal UI chrome` | PLAN line 4 uses the exact phrase. | plan-derived |
| `thin-client, thick-server simulation` | PLAN line 24 uses the exact phrase. | plan-derived |
| `view-only renderer of snapshots` | PLAN line 26 uses the exact phrase. | plan-derived |
| `append-only event log` | PLAN line 27 uses the exact phrase. | plan-derived |
| `last-write-wins irrelevant for bird traits` | PLAN line 109 uses the exact phrase. | plan-derived |
| `not an afterthought` | PLAN does not use this phrase, but line 13 says accessibility surfaces are designed and lines 119, 141-151 detail them. | mild inference, not rubric vocabulary |
| `constrained, ambient social and privacy posture` | PLAN lines 12, 19, and 177 support the social/privacy summary. | plan-derived synthesis |

No suspicious rubric-side terms appear. The reconstruction uses ordinary implementation-summary language and many exact PLAN phrases.

## Heading Mirror

The frozen reconstruction headings are `## System-level intent`, `## Per-feature whys`, and numbered subsections matching PLAN sections: Scope, Architecture, Data Model, API Surface, Simulation Engine Design, Sync Model, Frontend Rendering Pipeline, Audio Pipeline, Accessibility Surfaces, Performance Budgets, and Rollout & Risk. These mirror the PLAN structure, not GOLD_WHYS section headings or IDs. No exact or near-exact gold-list heading mirror was found.

## 1:1 Mapping Suspect

No 1:1 mapping to S1-S9 or F1-F40 is present. The reconstruction contains 8 system-level bullets and many PLAN-section feature bullets, not 9 system whys plus 40 feature whys in gold order. Several gold targets are absent or marked `NOT RECOVERABLE FROM PLAN`, which also argues against gold-list exposure.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `The Architecture section states that the model exists "to ensure continuity and multi-device coherence."` | PLAN line 24 states the thin-client/thick-server model exists `to ensure continuity and multi-device coherence`. | supported |
| `Personality drift is "monotonic," "moving toward expressive," calibrated for visible change in "~3 weeks of regular presence," and "never negative."` | PLAN lines 90 and 96-98 contain moving-toward-expressive, low-pass drift, three-week visibility, and never-negative neglect. | supported |
| `Social is limited to "One-to-one email-based visit invitations (read-only, ambient)," while "No Social Network" excludes profiles, discovery feeds, and public aviaries.` | PLAN lines 12 and 19 contain those exact constraints. | supported |

## Verdict

**PASS.** The reconstruction reads as plan-derived: it mirrors PLAN structure, uses many exact PLAN phrases, contains no gold IDs or rubric vocabulary, and does not provide a neat gold-ordered 1:1 map. A few summary phrases are interpretive, but they are grounded in nearby PLAN language rather than in the gold list.
