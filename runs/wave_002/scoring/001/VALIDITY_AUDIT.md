# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Mechanical check found no gold IDs or external taxonomy tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+` in the frozen reconstruction. The reconstruction uses ordinary numbered headings that mirror the PLAN sections, not the gold why IDs.

Verdict for this check: PASS.

## Vocabulary Check

Sampled reconstruction phrases and plan grounding:

| Reconstruction phrase | PLAN grounding | Assessment |
|---|---|---|
| "quiet, opt-in, read-only invitation flow" | PLAN line 5 uses the same phrase. | Plan-derived. |
| "default is silence" | PLAN line 7 says default is silence for visit notification. | Plan-derived. |
| "naturalist observation, not a game dashboard" | PLAN line 7 excludes progress counters/streaks and requires naturalist prose; PLAN line 74 excludes user-behavior logs. | Plan-derived synthesis. |
| "presentation instructions, never displayed as statistics" | PLAN line 23 uses the same wording. | Plan-derived. |
| "canonical, server-authored, and durable" | PLAN lines 21 and 64 make the database/worker canonical. | Plan-derived synthesis. |
| "aggregate-only operations pipeline" | PLAN line 21 uses this phrase. | Plan-derived. |
| "No launch waiver for accessibility" | PLAN line 108 uses the same phrase. | Plan-derived. |
| "input evidence, not an engagement metric" | PLAN line 66 uses the same phrase. | Plan-derived. |

Mechanical vocabulary check found no matches for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Verdict for this check: PASS.

## Heading Mirror

The only `##` headings are `System-level intent` and `Per-feature whys`, which were required by the phase-2A prompt. The `###` headings mirror the PLAN's eleven section headings: product contract, architecture, persistent data, API, canonical tick/presence, bird behavior/audio, frontend/accessibility, sync/security/privacy, performance/tests, delivery/rollout, and risks.

These do not mirror `GOLD_WHYS.md` section titles such as "System-level whys" or "Feature-level whys" and do not follow S1-S9/F1-F40 ordering.

Verdict for this check: PASS.

## 1:1 Mapping Suspect

The reconstruction does not provide a neat item for every S1-S9 or F1-F40 target. It follows the PLAN's structure and includes many plan-only items, such as exact config values marked `NOT RECOVERABLE FROM PLAN`, API endpoint rationale, migration details, and rollout sequencing. Several gold targets are merged into broader plan-derived bullets, and several feature whys are absent.

Verdict for this check: PASS.

## Plan-Derivation Spot Check

1. Reconstruction line 6 says the simulation is "canonical, server-authored, and durable." PLAN line 21 says the database is authoritative and the worker alone advances timelines; PLAN line 64 describes the durable scheduled tick. This is plan-derived.

2. Reconstruction line 10 says accessibility is "a first-class aviary experience, not a stripped fallback." PLAN line 17 requires all sensory paths to retain the aviary experience; PLAN lines 92-94 define reduced motion, narration, keyboard, and screen-reader behavior; PLAN line 108 says there is no launch waiver for accessibility. This is plan-derived.

3. Reconstruction line 141 says telemetry is aggregate-only with no account, bird, trait, action, or per-bird state fields. PLAN line 102 states the same boundary and adds that the simulation store has no read path into analytics. This is plan-derived.

No sampled sentence required held-out gold or PRD knowledge beyond the plan.

## Verdict: PASS

The frozen reconstruction shows no significant contamination signatures. It contains no leaked gold IDs, no scorer-side rubric vocabulary, no gold-list heading mirror, and no 1:1 mapping to the gold why table. It is detailed, but its detail tracks the PLAN's section order and wording.
