# VALIDITY_AUDIT - run 001

## ID leakage

Verdict: clean. A direct search of `RECONSTRUCTION.md` for gold IDs (`F1`, `S1`, `R-F01`, `R-S01`, etc.) returned no hits. The same search against the assigned PLAN also returned no hits, so there is no unexplained ID overlap.

## Vocabulary check

Sampled reconstruction phrases and PLAN grounding:

| Reconstruction phrase | PLAN support | Status |
|---|---|---|
| "server-side simulation tick" | Architecture and Tick Engine use this phrase directly. | plan-derived |
| "single source of truth" | Architecture and Sync Model use this phrase directly. | plan-derived |
| "no client-side personality ownership" | Sync Model states this directly. | plan-derived |
| "visible drift expected after 3 weeks" | Drift Function states this directly. | plan-derived |
| "naturalist prose, lowercase, present tense" | Notebook data model and accessibility sections use this language. | plan-derived |
| "maintained charm" | Reduced-motion accessibility section uses this phrase. | plan-derived |
| "Aggregate-only monitoring (no per-bird state)" | Service boundaries and observability use this language. | plan-derived |
| "can never infer interaction history" | Privacy boundary states this directly. | plan-derived |

No scorer-side phrases such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity` appeared in the reconstruction.

## Heading mirror

The only top-level headings are `System-level intent` and `Per-feature whys`, matching the phase-2A output contract rather than the held-out gold list. The lower headings under per-feature whys are PLAN section headings (`Scope`, `Architecture`, `Data Model`, `API Surface`, `Simulation Engine Design`, `Sync Model`, `Frontend Rendering Pipeline`, `Audio Pipeline`, `Accessibility Surfaces`, `Performance Budgets and Observability`, `Rollout`, `Risks`). They do not mirror the gold-list organization by source file or the S1-F40 taxonomy.

## 1:1 mapping suspect

Verdict: not suspect. The reconstruction does not provide neat S1-S9 or F1-F40 rows, does not use gold IDs, and does not follow the gold order. It instead expands the PLAN section by section and includes many plan-specific implementation items, including items that are not gold why anchors. Many entries are marked `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction rather than gold-list fitting.

## Plan-derivation spot check

1. Reconstruction: "The plan repeatedly frames personality as something that drifts over time, not instantly." PLAN support: Data Model and Simulation Engine describe server-computed deltas, low-pass drift, instruments at ~1 week, and visible drift after 3 weeks. Supported.

2. Reconstruction: "Accessibility ships with the main product and keeps the affective core." PLAN support: Scope includes screen-reader narration, reduced-motion, captions, keyboard support; Risks says accessibility work ships with the main product and not as an afterthought. Supported.

3. Reconstruction: "Privacy boundary around bird state and interaction history." PLAN support: Analytics/Telemetry is aggregate-only; Observability says no per-bird state, isolated simulation database, and telemetry cannot infer interaction history. Supported.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as derived from the assigned PLAN: its vocabulary is traceable to PLAN phrases, its headings mirror the PLAN rather than the gold taxonomy, and it lacks gold IDs or rubric-side terminology. Scores can be treated as valid under the phase-2B contract.
