# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict for this check: no leakage found.

Mechanical search of the frozen reconstruction for gold-style IDs (`F1`, `S1`, `R-F01`, `R-S01`, etc.) produced no hits. The plan also contains no such IDs, so there are no reconstruction-only gold identifiers to explain.

## Vocabulary check

No rubric-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity` appears in `RECONSTRUCTION.md`.

Sampled reconstruction phrases and plan derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "browser-only, single-aviary relationship surface" | PLAN §0 product sentence uses the same phrase. | plan-derived |
| "Drift is monotonic toward expressive" | PLAN §0 and §5.2 state this directly. | plan-derived |
| "ambient, not mistrust" | PLAN §5.2 uses this exact contrast. | plan-derived |
| "wallet of truth" | PLAN §6.3 uses the same phrase for shared tick state. | plan-derived |
| "A11y as labels-only" | PLAN §14 risk table uses this label. | plan-derived |
| "Social network gravity" | PLAN §14 risk table uses this phrase. | plan-derived |
| "toasty confetti" | PLAN §5.10 uses this phrase for adoption offers. | plan-derived |
| "no visitor POST events" | PLAN §4.5 says visitor cannot POST events. | plan-derived |

## Heading mirror

The reconstruction's top-level headings are the two required prompt headings: `## System-level intent` and `## Per-feature whys`. Its subheadings mirror the plan's section structure (`0. Purpose and posture / 1. Scope`, `2. Architecture`, `3. Data model / 4. API surface`, etc.), not the gold list's S1-S9/F1-F40 taxonomy. No held-out gold section title mirror was found.

## 1:1 mapping suspect

No 1:1 mapping to the gold list is present. The reconstruction has 13 system-level bullets and many plan-section grouped feature bullets, including several `NOT RECOVERABLE FROM PLAN` entries. It does not enumerate S1-S9, F1-F40, the 120-feature table, or anything in the same order as `GOLD_WHYS.md`.

## Plan-derivation spot check

1. Reconstruction sentence: "The plan defines presence as 'the conjunction of visibility + focus + recent input activity - not 'tab open.''"
   PLAN support: §0 non-negotiable consequence 2 states presence is the conjunction of visibility, focus, and recent input activity, not tab open. Verdict: grounded.

2. Reconstruction sentence: "The plan requires procedural calls because recorded call loops, including fallback loops, would violate the product posture."
   PLAN support: §0 requires procedural calls only and no recorded loops; §8.3 says WebAudio fallback is silence plus captions and no MP3 pack. Verdict: grounded.

3. Reconstruction sentence: "The 'three week club' slows clocks if drift is visible day-to-day and bumps `k_p` if invisible at day 21."
   PLAN support: §13.4 contains that calibration rule nearly verbatim. Verdict: grounded.

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no leaked gold IDs, no rubric/scorer vocabulary, no gold-list heading mirror, and no neat S/F mapping. The strongest phrases in the reconstruction trace directly to the plan's own wording and structure.
