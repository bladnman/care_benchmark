# VALIDITY_AUDIT - CARE run 001

## ID Leakage

**Result:** PASS. Mechanical search found no gold IDs or external taxonomy tokens in `RECONSTRUCTION.md` matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. Because no IDs were present in the reconstruction, there are no plan cross-reference leakage hits.

## Vocabulary Check

The reconstruction does not use scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Sampled plan-derived phrases:

| Reconstruction phrase | PLAN support |
|---|---|
| `one canonical aviary` | PLAN section 1 says each account owns exactly one canonical aviary. |
| `presence is the main relationship signal` | PLAN section 1 uses the same phrase. |
| `renderer and event producer, never a simulation authority` | PLAN section 2 uses this description of the web client. |
| `specific, lowercase naturalist prose` | PLAN section 1 states the product surface uses this prose style. |
| `privacy boundaries should be structural, not cosmetic` | PLAN sections 2, 8, and 9 require separate telemetry, no identifiers, and data-pipeline separation. |
| `release requirements for the same v1, not follow-up scope` | PLAN delivery sequence says accessibility and performance are same-v1 release requirements. |
| `first frame starts mid-motion` | PLAN section 6 says initialize from snapshots so the first rendered frame starts mid-motion. |
| `unioned presence intervals` | PLAN sections 3 and 10 describe unioning simultaneous owner-device intervals. |

## Heading Mirror

**Result:** PASS. Reconstruction headings are `## System-level intent`, `## Per-feature whys`, and `### 1` through `### 10` matching the plan's numbered sections. They do not mirror the held-out gold-list headings (`System-level whys`, `Feature-level whys`, or F/S identifiers). The heading structure reads like plan-derived organization, not scorer taxonomy.

## 1:1 Mapping Suspect

**Result:** PASS. The reconstruction has 9 system intent bullets and then a long plan-section walkthrough. The per-feature section follows the PLAN's 10 sections rather than the gold order of S1-S9 and F1-F40. It does not produce neat one-row mappings for every gold target, and it includes plan-specific items such as PRD conflict decisions and service components that are not gold IDs.

## Plan-Derivation Spot Check

1. Reconstruction: `The client is an expressive renderer, not an authority.`
   PLAN support: section 2 says the web client is a renderer and event producer, never a simulation authority.

2. Reconstruction: `Presence is the main relationship signal, but it is quiet and non-punitive.`
   PLAN support: section 1 says to treat presence as the main relationship signal and that absence is never a penalty.

3. Reconstruction: `Accessibility is part of v1 quality, not a fallback or later scope.`
   PLAN support: section 9 says accessibility and performance are release requirements for the same v1, not follow-up scope.

All three checked sentences are directly derivable from the assigned plan.

## Verdict

**PASS.** No significant contamination signatures were found. The reconstruction uses the plan's vocabulary and section order, contains no gold IDs or rubric vocabulary, and its articulate claims are supported by assigned-plan passages. Scores can be treated as valid for this run.
