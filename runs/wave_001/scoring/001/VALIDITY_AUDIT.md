# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. Searches of the frozen reconstruction found no gold IDs or rubric IDs such as `S1`, `F1`, `F40`, `R-F01`, `gold`, or `rubric`. The same search against PLAN also returned no such IDs.

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "canonical state holder and simulation engine" | yes | Directly plan-derived. |
| "thin rendering layer" | yes | Directly plan-derived. |
| "Append-only event log processed sequentially by server tick" | yes | Directly plan-derived. |
| "Presence-logic integrity (ensuring attention-metric honesty)" | yes | Directly plan-derived from risks. |
| "No baked loops" | yes | Directly plan-derived. |
| "Naturalist, present-tense prose" | yes | Directly plan-derived. |
| "Designed alternate render mode" | yes | Directly plan-derived. |
| "first-class alternate experience" | no exact match | Mild inference from accessibility scope and designed alternate rendering; not gold-ID/rubric vocabulary in context. |
| "Procedural life rather than baked content" | no exact match | Inferred heading from procedural rendering/WebAudio/no baked loops; not suspect by itself. |

No rubric-side phrases such as "multi-layer recovery," "feature-level fidelity," "weight-3," "intent fidelity," or "load-bearing" appear in RECONSTRUCTION.md.

## Heading Mirror Check

RECONSTRUCTION headings are only:

- `## System-level intent`
- `## Per-feature whys`

These are expected phase-2A output sections and do not mirror the gold-list section headings beyond the benchmark's generic reconstruction structure. The subsection labels inside `Per-feature whys` mirror the PLAN's own headings (Scope, Architecture, Data Model, API Surface, Simulation Engine, Sync Model, Rendering Pipeline, Audio Pipeline, Performance, Rollout), not the gold list.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping is present. The reconstruction does not enumerate S1-S9 or F1-F40 and does not proceed in gold-list order. It instead follows the candidate plan's structure and omits many gold targets entirely, including return greetings, no welcome-back toast, account export/deletion, visit log, synthetic account ID, and slow narration cadence.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The client is a thin rendering layer that pulls state snapshots and interpolates."
   PLAN support: Architecture says, "The client is a thin rendering layer that pulls state snapshots and interpolates." Verdict: directly derived.

2. Reconstruction sentence: "Presence and interactions should move birds toward expression without punishment."
   PLAN support: Simulation Engine says drift is "Monotonic toward expressive" with "presence-time (primary), interactions (secondary)" and "no penalty for neglect." Verdict: derived synthesis.

3. Reconstruction sentence: "Accessibility is designed as a first-class alternate experience."
   PLAN support: Scope includes screen-reader narration, reduced-motion, and call captioning; Accessibility says reduced motion is a "Designed alternate render mode." Verdict: derived but slightly polished; no clear contamination.

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no gold IDs, no rubric vocabulary, no gold-order enumeration, and its headings and feature groupings follow the PLAN rather than GOLD_WHYS.md. A few polished phrases go beyond exact plan wording, but they are ordinary syntheses of nearby plan text rather than contamination signatures.
