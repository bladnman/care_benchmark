# VALIDITY_AUDIT - CARE run 001

## ID leakage

**Result: PASS.** Mechanical search of the frozen reconstruction found no tokens matching gold IDs such as `F1`, `S1`, `R-F01`, or `R-S01`. The same check against the assigned PLAN also found no such IDs, so there are no reconstruction-only taxonomy leaks to report.

## Vocabulary check

**Result: PASS.** Mechanical search found none of these scorer-side terms in the reconstruction: `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Sampled plan-derived phrases from the reconstruction and supporting PLAN locations:

| Reconstruction phrase | PLAN support |
|---|---|
| "quiet everyday session" | PLAN §1: "Build the quiet everyday session first" |
| "server alone owns persistent mood and personality" | PLAN §1 invariant with the same phrase |
| "presence-based personality drift" / "neglect never subtracts traits" | PLAN §1 and §5 drift invariants |
| "private interaction history never enters analytics" | PLAN §1 and §8 privacy/telemetry sections |
| "stable immutable ID" | PLAN §1 and §3 bird identity fields |
| "Ship accessibility surfaces with v1" | PLAN §7 uses the same sentence |
| "constrained templates and state facts" | PLAN §5 notebook prose generation |
| "optional read-only visits" | PLAN §1 and §8 visit scope |
| "release gates rather than deferred polish" | PLAN §9 uses the same phrase |
| "Product voice has two registers" | PLAN §7 naturalist vs matter-of-fact copy split |

## Heading mirror

**Result: PASS.** The reconstruction headings mirror the PLAN headings, not the gold list. It has `## System-level intent`, `## Per-feature whys`, then `### 1. Product scope and invariants` through `### 11. Risks and mitigations`, exactly matching the PLAN's own top-level sectioning. These do not mirror held-out gold sections such as system-level whys, feature-level whys, or the 120-feature list.

## 1:1 mapping suspect

**Result: PASS.** The reconstruction does not create a neat S1-S9 / F1-F40 ledger. It contains a plan-shaped decomposition with many more per-feature bullets than the 40 gold feature whys, in PLAN section order rather than gold order. There are no gold IDs, no weight labels, and no evidence that every gold target received a matching reconstruction row.

## Plan-derivation spot check

| Reconstruction sentence | PLAN grounding | Assessment |
|---|---|---|
| "The plan's invariants say 'the server alone owns persistent mood and personality' and 'clients submit events, never state.'" | PLAN §1 contains both phrases in the invariants paragraph. | Plan-derived. |
| "Presence is the main input because the plan wants accumulated personality change from regular presence, while requiring monotonic deltas, no negative drift on absence, no per-session jump, and no exposed numbers." | PLAN §§1 and 5 state presence-based drift, monotonic additive deltas, no negative drift, no session-sized jump, and no exposed trait values. | Plan-derived synthesis. |
| "Narration exists so screen-reader users receive one coherent scene from the same snapshot, in naturalist prose, without raw state lists, trait values, duplicate speech or a flooded queue." | PLAN §7 says narration is generated from the same snapshot, in lowercase naturalist prose, with no trait values/raw state lists and no duplicate/flooded speech. | Plan-derived. |

## Verdict

**PASS.** The reconstruction reads as a faithful plan-derived expansion. It has no ID leakage, no scorer/rubric vocabulary, no gold-order mapping, and its headings/phrases are traceable to the assigned PLAN. Scores can be treated as valid under the phase-2B contamination audit.
