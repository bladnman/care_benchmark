# VALIDITY_AUDIT - run 001

## ID Leakage

Verdict: no leakage found.

Mechanical search of the frozen reconstruction for gold/rubric IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+` returned no hits. A paired search over the assigned PLAN and RECONSTRUCTION likewise showed no ID tokens to cross-check.

## Vocabulary Check

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Sampled plan-derived phrases:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "quiet sky field" | PLAN boot sequence uses quiet field/sky shell before snapshot. | plan-derived |
| "NO spinner" | PLAN boot sequence says first frame uses no spinner. | plan-derived |
| "no client-authored personality writes" | PLAN scope/sync repeats client cannot write personality state. | plan-derived |
| "watching without moving is the product" | PLAN presence activity window rationale uses this phrase. | plan-derived |
| "Age not visit count" | PLAN defensible implementation calls use this phrase. | plan-derived |
| "aggregate only" | PLAN observability section is privacy-safe aggregate-only telemetry. | plan-derived |
| "Ship a11y with v1, not after" | PLAN risk mitigation uses this phrase. | plan-derived |
| "No audio files shipped" | PLAN audio pipeline uses this phrase. | plan-derived |

## Heading Mirror

RECONSTRUCTION headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope and defensible implementation calls`
- `### Architecture and data model`
- `### API surface`
- `### Simulation engine design`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance, observability, rollout`

These mirror the PLAN's sections, not the held-out gold-list sections. The only two required phase-2A headings are the expected wrapper headings. No heading closely mirrors `GOLD_WHYS.md` sections such as system-level whys, feature-level whys, or complete features list beyond the generic requested phase-2A output structure.

## 1:1 Mapping Suspect

No. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not create exactly 49 target rows. It groups material by the plan's own implementation sections and includes many plan features that are not gold-why-bearing, plus several `NOT RECOVERABLE FROM PLAN` markers. The shape is consistent with plan-derived reconstruction rather than a neat target-list mapping.

## Plan-Derivation Spot Check

1. Reconstruction: "Server-authoritative simulation with the client as renderer and event emitter."
   PLAN support: authority boundaries say the simulation tick owns vectors, mood, perch assignments, weather, notebook generation, and canonical positions; the client owns rendering, audio, presence detection, and UI, and never writes personality state.
   Result: grounded.

2. Reconstruction: "Presence is calibrated longer because 'watching without moving is the product.'"
   PLAN support: the defensible implementation call for presence activity window says 5 minutes, tune via shadow metrics, because watching without moving is the product.
   Result: grounded.

3. Reconstruction: "If AudioContext fails, calls are disabled, captions auto-enable, and a matter-of-fact banner appears."
   PLAN support: WebAudio fallback lists all call playback disabled, captions auto-enable, matter-of-fact banner, and no recorded fallback.
   Result: grounded.

## Verdict

PASS. The reconstruction reads as plan-derived: no ID leakage, no rubric vocabulary, headings follow the plan rather than the gold list, the mapping is not 1:1, and spot-checked articulate claims are grounded in the assigned PLAN. Scores can be treated as valid for this run.
