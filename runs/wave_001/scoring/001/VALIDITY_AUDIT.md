# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: PASS.

I searched the frozen reconstruction for gold IDs and rubric-style IDs: S1-S9, F1-F40, R-Fxx, plus obvious scoring terms. No gold IDs or per-gold taxonomy labels appeared in `RECONSTRUCTION.md`. The reconstruction mentions PRD claims, but PLAN.md also repeatedly mentions PRD source material and PRD claims, so that is not leakage.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "structurally true rather than true-by-policy" | PLAN.md section 0 opening sentence uses the same phrase. | Plan-derived. |
| "Load-bearing decisions" | PLAN.md section 0 heading. | Plan-derived. |
| "schema boundaries, not policy" | PLAN.md load-bearing decision 10. | Plan-derived. |
| "the tick is the only writer" | PLAN.md load-bearing decision 2 and data model permissions. | Plan-derived. |
| "the aviary continues without the viewer" | PLAN.md load-bearing decision 1. | Plan-derived. |
| "a place, not a soundscape" | PLAN.md call scheduling calm cap. | Plan-derived. |
| "a v1 deliverable with the same milestone gates" | PLAN.md accessibility section. | Plan-derived. |
| "nothing to overwrite" | PLAN.md sync conflict explanation says no client submits bird state. | Plan-derived paraphrase. |
| "NOT RECOVERABLE FROM PLAN" | This is a phase-2A marker, not PLAN prose or gold ID. | Not a leakage concern by itself. |
| "multi-layer recovery", "feature-level fidelity", "weight-3", "intent fidelity" | Not present in the reconstruction. | No rubric vocabulary leakage found. |

## Heading Mirror Check

The required top-level headings, `## System-level intent` and `## Per-feature whys`, resemble the phase-2A output contract, not the gold list. The subsection headings under `Per-feature whys` mirror PLAN sections rather than GOLD_WHYS sections:

| Reconstruction heading | Closest PLAN heading | Closest GOLD heading | Assessment |
|---|---|---|---|
| Load-bearing decisions | PLAN.md section 0 | none | Plan mirror, acceptable. |
| Scope | PLAN.md section 1 | none | Plan mirror, acceptable. |
| Architecture and stack | PLAN.md section 2/2.4 | none | Plan mirror, acceptable. |
| Data model | PLAN.md section 3 | none | Plan mirror, acceptable. |
| API surface | PLAN.md section 4 | none | Plan mirror, acceptable. |
| Simulation engine design | PLAN.md section 5 | none | Plan mirror, acceptable. |
| Sync model | PLAN.md section 6 | none | Plan mirror, acceptable. |
| Frontend rendering pipeline | PLAN.md section 7 | none | Plan mirror, acceptable. |
| Audio pipeline | PLAN.md section 8 | none | Plan mirror, acceptable. |
| Accessibility surfaces | PLAN.md section 9 | accessibility_perf.md file area only | Plan mirror, acceptable. |
| Performance budgets and observability | PLAN.md section 10 | accessibility_perf.md file area only | Plan mirror, acceptable. |
| Testing, rollout, and sequencing | PLAN.md sections 11-15 | none | Plan mirror, acceptable. |

## 1:1 Mapping Suspect Check

Verdict for this check: PASS.

The reconstruction does not enumerate S1-S9 or F1-F40, does not use the gold order, and does not produce one neat item per gold why. It follows the PLAN structure: load-bearing decisions, scope, architecture, data model, API, engine, sync, frontend, audio, accessibility, performance, and rollout. The order is explainable from PLAN.md and is not a gold-list mirror.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "The plan says the tick is 'the only writer' of bird state, while the responder writes only events and plans..." | PLAN.md load-bearing decision 2 says the tick is the only writer and responder writes only to the append-only event log and plans table. | Supported. |
| "The snapshot 'never carries the personality vector'; it carries only 'quantized, named presentation parameters'..." | PLAN.md load-bearing decision 3 says the snapshot never carries the vector and clients receive expression profiles. | Supported. |
| "The DOM accessibility overlay, reduced-motion renderer, captions, screen-reader narration, keyboard navigation, contrast gates, and outside AT sessions..." | PLAN.md load-bearing decision 5 and sections 7.4, 9, 11.4, and 9.7 list those mechanisms. | Supported. |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses many PLAN-specific phrases and headings, but that mirroring follows PLAN.md rather than GOLD_WHYS.md or RUBRIC.md. It contains no gold IDs, no 1:1 gold mapping, and no free use of scoring vocabulary. The run's scores are not flagged as contamination-suspect.
