# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no gold-ID leakage found. I searched the frozen reconstruction for gold-style tokens such as `F1`, `F40`, `S1`, `S9`, `R-F01`, plus scorer-side labels. The reconstruction does not use the gold IDs or an external feature taxonomy. The only notable hits were ordinary words also present in the plan, such as `PRD`, and the word `weight` in "game engine weight".

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "The server computes meaning, the client computes appearance" | PLAN opening principle uses the exact sentence. | plan-derived |
| "No entry animation, ever" | PLAN 2.4 uses the exact heading. | plan-derived |
| "architectural absences, not feature flags" | PLAN Scope uses the exact phrase. | plan-derived |
| "boring with teeth" | PLAN 6.5 uses the phrase for the pull model. | plan-derived |
| "same aviary, not a stripped fallback" | PLAN 8.4/10 describe same-data reduced motion and not a stripped fallback. | plan-derived |
| "StateFactExtractor" | PLAN 9.1 names the shared module. | plan-derived |
| "build deliverable, not a deploy-time guess" | PLAN 5.4 uses the phrase about calibration. | plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Required by the phase-2A reconstruction prompt; used for plan gaps. | expected |

I did not find scorer-side vocabulary such as "multi-layer recovery", "feature-level fidelity", "intent fidelity", "gold why", or "weight-3" in the frozen reconstruction.

## Heading Mirror Check

The reconstruction top-level headings are `## System-level intent` and `## Per-feature whys`, which are required by the phase-2A prompt. Subheadings mirror the plan's own section order: Scope, Architecture, Data model, API surface, Simulation engine design, Sync model, Audio pipeline, Frontend rendering pipeline, Naturalist text generation, Accessibility surfaces, Performance budgets and observability, Rollout, Risks. They do not mirror the gold list's S1-S9/F1-F40 headings.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not follow the gold feature order, and does not create one neat item per gold target. It follows the plan's sections and includes plan-specific implementation decisions such as Postgres event-log choice, `EmailSender`, tick leases, and `StateFactExtractor`.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The snapshot boundary in section 2.3 carries the same intent: everything left of the snapshot is server-owned; everything right of it is a pure function of latest snapshot, wall-clock time, per-entity render seed."
   Plan support: PLAN 2.3 states the boundary and the same pure-function inputs. Supported.

2. Reconstruction sentence: "A hand-rolled renderer keeps engine weight out of the bundle and avoids carrying unused physics, tilemaps, and particle systems."
   Plan support: PLAN 8.1 says no general-purpose game engine and lists unused physics, tilemaps, and particle systems. Supported.

3. Reconstruction sentence: "If AudioContext fails or is blocked, scheduling still runs and calls become captions with no recorded-audio fallback."
   Plan support: PLAN 7.5 describes caption-only mode, default captions, and no recorded-audio fallback. Supported.

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It uses the plan's vocabulary and section structure, contains no gold IDs, does not map 1:1 to the held-out gold list, and the strongest sentences spot-check cleanly against PLAN.md. No contamination signatures rose above normal plan-language reuse.
