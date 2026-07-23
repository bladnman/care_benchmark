# VALIDITY_AUDIT - CARE run 001

## ID leakage

PASS. A mechanical search for gold-style IDs (`F1`, `S1`, `R-F01`, `R-S01`, etc.) found no hits in the frozen reconstruction. The reconstruction does not use the external gold taxonomy.

## Vocabulary check

PASS. I sampled the following reconstruction phrases and verified they are plan-derived rather than rubric-side vocabulary:

| Reconstruction phrase | PLAN support | Finding |
|---|---|---|
| "server is the only place time passes" | PLAN.md:58 | direct plan phrase |
| "the aviary continues without the viewer" | PLAN.md:67 | direct plan phrase |
| "declarative scene-state + parameterized-intent" | PLAN.md:63 | direct plan phrase |
| "strict no-recorded-audio rule" | PLAN.md:386 | direct plan phrase |
| "1-week-measurable / 3-weeks-visible" | PLAN.md:333, 376 | plan-derived calibration phrase |
| "copy-voice lint" | PLAN.md:35, 357, 430 | plan-derived enforcement phrase |
| "relationship with their aviary" | PLAN.md:333 | direct plan phrase |
| "CI-enforced gates, not guidelines" | PLAN.md:316 | direct plan phrase |

No scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity` appear in the reconstruction.

## Heading mirror

PASS. Reconstruction headings are `System-level intent`, `Per-feature whys`, then plan-derived groups: `Scope`, `Architecture and sync`, `Data model and API surface`, `Simulation engine`, `Frontend rendering and audio`, and `Accessibility, observability, rollout, and risks`. These mirror PLAN.md's organization, not GOLD_WHYS.md's system/feature ID ordering or file-group taxonomy.

## 1:1 mapping suspect

PASS. The reconstruction does not map neatly to S1-S9 and F1-F40. It has 12 system-intent bullets rather than 9, no gold IDs, and many per-feature bullets grouped by the plan's own sections. Some gold targets are split, merged, or marked `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction rather than a gold-list mirror.

## Plan-derivation spot check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| "The intended feeling is that 'the aviary continues without the viewer' and the returning user meets 'continuity, not a reset.'" | PLAN.md:65-67 and 204-209 describe server-side continuation and mood continuity. | grounded |
| "drift is a 'slow low-pass filter,' 'presence-time' is dominant... calibration aims for '1-week-measurable / 3-weeks-visible'" | PLAN.md:195-202 and 373-376 give the drift formula, calibration band, and risk framing. | grounded |
| "telemetry 'never reads the simulation database'... anything reconstructable into 'a user's relationship with their aviary'" | PLAN.md:328-340 states aggregate-only telemetry and separation from simulation data. | grounded |

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It contains no gold ID leakage, no rubric vocabulary, no 1:1 gold-list mapping, and its headings follow the plan's own structure. The reconstruction is sometimes very articulate, but the strongest phrases are traceable to PLAN.md rather than to held-out scoring artifacts.
