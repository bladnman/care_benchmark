# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no ID leakage found. I searched the frozen reconstruction for gold-style IDs and rubric markers (`F1`, `F40`, `S1`, `S9`, `R-F`, `weight-3`, `feature-level`, `intent fidelity`, `rubric`, `gold`). The reconstruction does not use S/F IDs or rubric taxonomy. The only `load-bearing` hit in the slot artifacts is in PLAN 1.3, not in RECONSTRUCTION.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Presence is relational, not a counter" | PLAN 1.3 asks whether a feature teaches "presence is for a counter rather than the birds" | Plan-derived synthesis; not suspicious. |
| "server-authoritative simulation with stateless clients" | PLAN 2.1 uses the exact phrase | Directly plan-derived. |
| "single canonical aviary state" | PLAN 2.1 and 6.1 describe the single canonical state | Directly plan-derived. |
| "Vector values are felt through behavior, not read as numbers" | PLAN 3.2 uses the exact sentence | Directly plan-derived. |
| "Change should be slow, sparse, and meaningful" | PLAN 5.2 names slow drift; PLAN 3.4 says sparsity is a design constraint | Plan-derived synthesis. |
| "Accessibility is first-class design, not a fallback" | PLAN 12.4 says accessibility surfaces are first-class, not a checklist; PLAN 7.4 says not stripped fallback | Plan-derived synthesis. |
| "Procedural audio is part of the spell of liveness" | PLAN 8.1 says looped audio breaks the spell and is dead software | Plan-derived synthesis. |
| "affective-perf bridge" | PLAN 10.2 uses the exact phrase | Directly plan-derived. |

No rubric-side vocabulary is used freely in RECONSTRUCTION.

## Heading Mirror Check

RECONSTRUCTION headings are `System-level intent`, `Per-feature whys`, then plan-shaped sections: `Scope and non-goals`, `Architecture, data, and API surface`, `Simulation and sync`, `Frontend rendering and audio`, and `Accessibility, performance, observability, and rollout`. These mirror the reconstruction task and PLAN organization, not the gold-list section titles. There is no heading-by-heading echo of S1-S9 or F1-F40.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and is ordered by PLAN sections rather than gold order. Some items naturally align with gold rows because the PLAN is broad, but the mapping is not a neat one-item-per-gold-target list.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN grounding | Assessment |
|---|---|---|
| "Bird state should be felt through behavior, not read as numbers." | PLAN 3.2: "Vector values are felt through behavior, not read as numbers." | Directly grounded. |
| "Loading state as a quiet field rather than a spinner: 'A spinner says machine; we are not selling a machine.'" | PLAN 7.3 contains the same quiet-field and spinner rationale. | Directly grounded. |
| "The narration cadence is slow -- high-frequency narration would overwhelm the screen reader's queue." | PLAN 9.1 states the slow cadence and queue-overwhelm reason. | Directly grounded. |

## Verdict: PASS

The frozen reconstruction reads as a plan-derived synthesis. It contains no gold ID leakage, no free rubric vocabulary, no gold-order reconstruction, and its most articulate claims trace cleanly to PLAN passages. The run's scores do not need contamination discounting.
