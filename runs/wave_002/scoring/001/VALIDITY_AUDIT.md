# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no gold-ID leakage found. A targeted search of `RECONSTRUCTION.md` found no `S1`-`S9`, `F1`-`F40`, `R-Fxx`, `gold why`, `gold list`, `intent fidelity`, `feature-level fidelity`, `multi-layer recovery`, or `weight-3` terms. The only rubric-sounding hit was `load-bearing`, and the same word appears in PLAN line 786, so it is plan-derived rather than leakage.

## Vocabulary Check

| Reconstruction phrase | Appears or is grounded in PLAN? | Finding |
|---|---|---|
| `load-bearing` | yes, PLAN line 786 | clean |
| `no client-owned simulation state` | yes, PLAN summary and sync sections | clean |
| `quiet field replaces a spinner` | yes, PLAN lines 448-450 | clean |
| `designed surface, not a stripped fallback` | yes, PLAN reduced-motion section | clean |
| `telemetry pipelines never touch the simulation DB` | yes, PLAN observability/privacy sections | clean |
| `same canonical record` | yes, PLAN sync model | clean |
| `NOT RECOVERABLE FROM PLAN` | reconstructor convention, not gold-specific | clean |
| `v1.1 fix` | yes, PLAN accessibility risk section | clean |

No sampled phrase reads like rubric-side scoring vocabulary except `load-bearing`, which is present in the PLAN.

## Heading Mirror

`RECONSTRUCTION.md` headings are `System-level intent`, `Per-feature whys`, and plan-section mirrors such as `Scope`, `Architecture`, `Data Model`, `API Surface`, `Simulation Engine Design`, `Sync Model`, `Frontend Rendering Pipeline`, `Audio Pipeline`, `Accessibility Surfaces`, `Performance Budgets and Observability`, `Rollout, Risks, and Tests`, and `Open Implementation Details`. These mirror the PLAN structure, not the gold-list taxonomy. They do not echo gold headings like `feels-alive-not-robotic`, `presence-definition`, or `drift-function`.

## 1:1 Mapping Suspect

No near-1:1 mapping to S1-S9 or F1-F40 is present. The reconstruction lists more than 40 per-feature items, follows the PLAN's own sections, includes non-gold implementation details, and marks many items `NOT RECOVERABLE FROM PLAN`. That shape is consistent with plan-derived reconstruction rather than gold-list access.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| `The plan makes this load-bearing in Scope, Architecture, Sync model, and Summary...` | PLAN line 786 names the two refusals load-bearing; lines 13, 56-69, and 385-410 ground server-only state. | supported |
| `The bootstrap HTML inlines a tiny initial snapshot... There is no spinner-resolves-into-aviary transition.` | PLAN lines 448-450 state the inline snapshot and no-spinner quiet field. | supported |
| `The telemetry line has aggregate-only RUM with no per-account or per-bird dimensions.` | PLAN lines 604-618 describe aggregate-only RUM and what not to measure. | supported |

## Verdict: PASS

The reconstruction reads as plan-derived. I found no gold IDs, no gold-order mapping, no gold-heading mirroring, and no unsupported rubric vocabulary. Some phrases are strong, but the spot checks tie them back to PLAN passages rather than contamination.
