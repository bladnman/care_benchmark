# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no ID leakage found. Mechanical searches of the frozen reconstruction for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` returned no hits. The corresponding search in the PLAN also returned no hits, so there were no reconstruction-only gold IDs to cross-reference.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION and checked them against PLAN vocabulary:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "Presence is the primary product signal" | PLAN section 1 says presence, not click volume or reward loop, is the primary signal. | plan-derived |
| "feel continuing and alive between visits" | PLAN section 1 says the aviary should feel as though it continues between visits. | plan-derived |
| "server owns canonical reality" | PLAN section 1 says the server owns the one canonical aviary; section 2 says server authoritative. | plan-derived |
| "bounded non-negative deltas" | PLAN section 5 uses bounded non-negative deltas toward expressive ends. | plan-derived |
| "specific lowercase naturalist prose" | PLAN section 4 uses that phrase for product-facing surfaces. | plan-derived |
| "physically and logically apart" | PLAN section 2 says operational telemetry is physically and logically apart from simulation state. | plan-derived |
| "Ship accessibility as part of v1" | PLAN section 8 uses that sentence. | plan-derived |
| "read-only visitor sessions" | PLAN sections 4 and 9 describe narrowly scoped read-only visitor sessions. | plan-derived |
| "compact 2D Canvas/SVG scene" | PLAN section 6 prefers a compact 2D Canvas/SVG scene. | plan-derived |

Mechanical searches found no scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity` in the reconstruction.

## Heading Mirror

The top-level headings `System-level intent` and `Per-feature whys` are required by the phase-2A prompt, so they are not suspicious by themselves. The subsequent headings mirror the PLAN structure without numbering: Product contract and scope, Architecture and boundaries, Data model, API surface and user flows, Simulation engine and calibration, Frontend scene and interaction pipeline, Procedural audio, Accessibility and performance, Security/privacy/observability, and Delivery sequence/release gates. They do not mirror the gold-list headings (`System-level whys`, `Feature-level whys`, `Complete features list`) or the S/F taxonomy.

## 1:1 Mapping Suspect

No 1:1 gold mapping detected. The reconstruction has 10 system-level principles, not S1-S9, and it organizes per-feature material by the PLAN's section order rather than the gold F1-F40 sequence. It contains many more per-feature bullets than the 40 gold why anchors, includes plan-specific operational items, and marks several items `NOT RECOVERABLE FROM PLAN`, which is consistent with plan-derived reconstruction rather than target-list fitting.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Presence is the primary product signal, not activity volume." PLAN support: section 1 says "Presence-not click volume or a reward loop-is the primary signal," and section 5 makes presence dominant for drift.

2. Reconstruction sentence: "Privacy and data minimization are cross-cutting boundaries, not analytics afterthoughts." PLAN support: section 2 separates telemetry from simulation state; section 9 excludes interaction records from analytics, training, recommendation, and population analysis.

3. Reconstruction sentence: "Accessibility is part of v1 and comes from the same world model as rendering." PLAN support: section 8 says "Ship accessibility as part of v1" and states narration/captions derive from the same snapshot/grammar used by visual/audio rendering.

## Verdict: PASS

The reconstruction reads as plan-derived. There are no gold IDs, no scorer-side vocabulary, no 1:1 S/F mapping, and the headings follow the PLAN rather than the gold package. The strongest sentences checked above are directly supported by PLAN language.
