# VALIDITY AUDIT - CARE run 001

Verdict: **PASS**

## ID Leakage

No gold IDs or rubric IDs were found in the frozen reconstruction. A targeted search for `S1`-style IDs, `F1`-style IDs, `R-F`, `canonical #`, `feature-level fidelity`, `multi-layer recovery`, `weight-3`, `load-bearing`, `intent fidelity`, and `gold` returned no hits in either `RECONSTRUCTION.md` or `PLAN.md`.

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| `Notice, never announce` | yes, PLAN.md:4 | Plan-derived |
| `Feels alive, not robotic` | yes, PLAN.md:4 | Plan-derived |
| `The Heartbeat` | yes, PLAN.md:65 | Plan-derived |
| `quiet field` | yes, PLAN.md:102 | Plan-derived |
| `visible` + `focused` + `activity` | yes, PLAN.md:88 | Plan-derived |
| `single source of truth for personality and mood` | yes, PLAN.md:87 | Plan-derived |
| `naturalist and not robotic` | yes, PLAN.md:147 | Plan-derived |
| `too fast feels like a toy; too slow feels like a screensaver` | yes, PLAN.md:144 | Plan-derived |
| `phase-canceling or repetitive artifacts` | yes, PLAN.md:146 | Plan-derived |

I did not find rubric-side vocabulary used freely in the reconstruction. Terms such as `feature-level fidelity`, `multi-layer recovery`, `weight-3`, `load-bearing`, and `intent fidelity` were absent.

## Heading Mirror

The reconstruction headings are `System-level intent`, `Per-feature whys`, and plan-shaped subsections such as `Scope`, `Architecture`, `Data Model`, `Simulation Engine Design`, `Sync & Interaction Model`, `Frontend Rendering Pipeline`, `Performance & Observability`, and `Rollout & Risks`. These mirror the PLAN structure, not the gold-list S/F taxonomy. They do not echo gold section titles such as `S1 - feels-alive-not-robotic` or `F1 - presence-definition`.

## 1:1 Mapping Suspect

No 1:1 gold mapping was detected. The reconstruction has 8 system bullets rather than S1-S9, and its per-feature section follows the candidate plan headings and many plan bullets. It does not enumerate F1-F40, does not preserve gold order, and includes many `NOT RECOVERABLE FROM PLAN` entries for plan-local bullets rather than neat gold targets.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `The aviary should have one reliable truth across devices.` | PLAN.md:22-34 and 85-88 describe server-side canonical state, thin clients, snapshots, and clients that never write state. | Plan-derived synthesis |
| `Performance is part of the alive feeling.` | PLAN.md:102, 125-128, 143-147 tie first snapshot/quiet field, 500ms TTFB, 60fps, memory, and sync/audio/accessibility risks to alive/not-robotic behavior. | Plan-derived synthesis |
| `Accessibility surfaces should preserve the same naturalist voice rather than becoming a separate mechanical layer.` | PLAN.md:11 and 113-119 describe naturalist narration, captions, reduced-motion cross-fades, and keyboard support; PLAN.md:147 warns narration must stay naturalist and not robotic. | Plan-derived synthesis |

## Verdict

**PASS.** The reconstruction reads as a plan-derived synthesis. It uses the candidate plan's headings, phrases, and omissions; it does not leak gold IDs, rubric vocabulary, or a neat S1-S9/F1-F40 mapping. Some sentences are articulate syntheses, but each sampled synthesis has clear support in the PLAN.
