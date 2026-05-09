# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS. The frozen reconstruction reads as plan-derived: it mirrors the PLAN section structure, uses no gold IDs or rubric vocabulary, and often says NOT RECOVERABLE FROM PLAN rather than filling gaps with gold-like rationale. Minor overlap with gold concepts is explained by the plan itself using product terms such as presence, server-side tick, and naturalist prose.

## ID Leakage Check

No leakage found. Searches of `RECONSTRUCTION.md` found no standalone gold IDs or rubric-side IDs such as `S1`, `F1`, `F40`, `R-F01`, `GOLD`, or `RUBRIC`. The same search over `PLAN.md` also found no such IDs, so there are no reconstruction-only ID hits to cross-reference.

## Vocabulary Check

| Reconstruction phrase | In PLAN? | Assessment |
|---|---|---|
| "aviary continues without the viewer" | yes, PLAN.md:23 | Plan-derived. |
| "thin-client, thick-server architecture" | yes, PLAN.md:23 | Plan-derived. |
| "Care is non-punitive" | partially, PLAN.md:18 and 57 | Inference from no Tamagotchi/zero drift, not gold-vocabulary leakage. |
| "Single Canonical State" | yes, PLAN.md:62 | Plan-derived. |
| "quiet, observational, and naturalist" | partially, PLAN.md:44, 59, 81 | Plan-derived synthesis of naturalist surfaces. |
| "core surface, not an afterthought" | partially, PLAN.md:13 and 105 | Inference from v1 scope and accessibility risk. |
| "strict privacy boundary" | yes, PLAN.md:93 | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | not in PLAN | Expected phase-2A reconstruction marker, not gold/rubric content. |

No suspicious terms such as "multi-layer recovery", "feature-level fidelity", "weight-3", "load-bearing", or "intent fidelity" appear in the reconstruction.

## Heading Mirror Check

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then `### 1. Scope` through `### 12. Risks`. The `###` headings mirror the PLAN sections, not the gold list. They do not mirror gold headings such as `S1 - feels-alive-not-robotic`, `F1 - presence-definition`, or the gold file-group headings.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping pattern found. The reconstruction has 8 system bullets, not S1-S9, and its per-feature section follows the candidate PLAN's 12 implementation sections rather than the 40 gold why IDs or the 120-feature table. The ordering is plan-shaped and contains many plan-local items that are not gold why rows.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| "The aviary continues without the viewer." | PLAN.md:23 says the architecture enforces the "aviary continues without the viewer" rule; PLAN.md:26 says the tick runs regardless of client connectivity. | Supported. |
| "Clients send actions, not absolute state, to prevent last-write-wins conflicts between devices." | PLAN.md:63 says clients never send absolute state, only actions, and this prevents last-write-wins conflicts. | Supported. |
| "Screen reader narration may sound robotic, so the plan treats narration generation as a core writing task and shares logic with the Field Notebook." | PLAN.md:105 says narration sounding robotic is a risk and mitigation is treating narration generation as a core writing task, sharing generation logic with Field Notebook. | Supported. |

## Verdict Rationale

PASS: the reconstruction shows no significant contamination signature. Its vocabulary, headings, and item ordering derive from the PLAN; where the PLAN is thin, the reconstruction usually refuses recovery rather than matching the gold list. Scores can be treated as valid for this run.
