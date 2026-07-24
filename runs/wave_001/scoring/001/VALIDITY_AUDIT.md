# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A mechanical search of the frozen reconstruction for gold-style IDs (F1, S1, R-F01, R-S01, etc.) returned no hits. The reconstruction does not use the scorer/gold taxonomy, and the PLAN likewise does not introduce those IDs.

## Vocabulary Check

Sampled phrases from the reconstruction against the PLAN:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "prefer notice-over-announce, sparsity over feature density" | PLAN line 33 has the same ambiguity decision. | plan-derived |
| "Clients never tick and never write personality" | PLAN line 51 uses the same sentence. | plan-derived |
| "birds feel continuous across absence and across phone/laptop" | PLAN line 494 has the same success criterion. | plan-derived |
| "Canned audio / loops sneaks in" and "Kills aliveness" | PLAN line 447 contains this risk. | plan-derived |
| "accessible paths stay charming, not checklist shells" | PLAN line 383 contains this stance. | plan-derived |
| "per-bird interaction data in aggregate analytics / ML" | PLAN line 31 excludes this. | plan-derived |
| "unique wall-clock minutes" | PLAN line 281 contains the same decision. | plan-derived |
| "matter-of-fact upgrade message" | PLAN line 434 contains this copy stance. | plan-derived |

No scorer-side vocabulary such as gold, rubric, weight-3, multi-layer recovery, feature-level fidelity, system-level fidelity, load-bearing, or intent fidelity appeared in the reconstruction.

## Heading Mirror

The reconstruction headings are:

- ## System-level intent
- ## Per-feature whys
- ### Scope
- ### Architecture and data model
- ### API and interaction paths
- ### Simulation and sync model
- ### Frontend rendering, audio, and accessibility
- ### Performance, observability, and rollout

The first two are required by the reconstruction prompt. The subsection headings mirror the PLAN's structure, not GOLD_WHYS.md's gold-list taxonomy. They do not mirror S1-S9 or F1-F40 section titles.

## 1:1 Mapping Suspect

Verdict: not suspect. The reconstruction has no neat S1-S9/F1-F40 order, no gold IDs, and no 49-row gold-why mapping. Its per-feature section follows the candidate plan's own sections and includes many implementation items outside the 40 scored feature whys.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "Make continuity across absence and devices feel real." | PLAN lines 17, 104, 260, 285, and 493-494 cover canonical sync, motion phase, background return, and continuity across phone/laptop. | supported |
| "Presence is the dominant drift signal, but it is clamped and credited as unique wall-clock minutes." | PLAN lines 173-175, 220-225, and 279-281 cover triple-condition presence, drift input, and unique wall-clock credit. | supported |
| "Make accessible paths the same charming product, not a stripped-down state dump." | PLAN lines 19, 355-383, 449, and 497 cover naturalist narration, reduced motion, captions, and a11y users getting naturalist aliveness. | supported |

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no ID leakage, no scorer-side vocabulary, headings track the PLAN rather than the gold list, and articulate rationale sentences can be grounded in the PLAN. Scores are not flagged for contamination.
