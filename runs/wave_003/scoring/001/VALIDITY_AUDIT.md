# VALIDITY_AUDIT - CARE run 001

## Gold ID leakage check

Verdict: PASS.

I found no explicit gold IDs such as `S1`, `S9`, `F1`, `F40`, or `R-F01` in `RECONSTRUCTION.md`. The only load-bearing phrase search hits were phrases also present in `PLAN.md`, such as "the relationship is the user's". There is no evidence that the reconstruction copied the gold taxonomy.

## Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "aliveness is the product" | PLAN.md line 5 uses the same phrase. | Plan-derived. |
| "architectural property a reviewer can check" | PLAN.md line 3 uses the same phrase. | Plan-derived. |
| "simulation owns meaning; client owns motion" | PLAN.md line 84 uses the same phrase. | Plan-derived. |
| "harder, not a flag-flip" | PLAN.md §1.2/§1.4 states non-goals are structurally hard to re-add. | Plan-derived. |
| "drift's perceived validity" | PLAN.md line 143 uses the same phrase. | Plan-derived. |
| "audible signature of dead software" | PLAN.md §7.1 uses the same phrase. | Plan-derived. |
| "actual product, not a stripped variant" | PLAN.md line 436 uses the same phrase. | Plan-derived. |
| "matter-of-fact" | PLAN.md §§4,10 repeatedly require matter-of-fact system errors. | Plan-derived. |
| "the relationship is the user's" | PLAN.md line 510 uses the same phrase. | Plan-derived. |
| "above 500ms the user notices loading" | PLAN.md §9.2 states the same threshold rationale. | Plan-derived. |

I did not find rubric-side vocabulary used as reconstruction scaffolding: no "multi-layer recovery", "feature-level fidelity", "weight-3", or gold ID labels appear in the reconstruction.

## Heading mirror check

`RECONSTRUCTION.md` has only two headings: `## System-level intent` and `## Per-feature whys`. These match the reconstruction artifact's expected structure, not the gold-list headings. It does not mirror gold entries such as `S1 - feels-alive-not-robotic`, `F1 - presence-definition`, or the grouped PRD-file headings in `GOLD_WHYS.md`.

## 1:1 mapping suspect check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, and it does not follow the gold order. Its per-feature section starts with scope/account surface items and then follows the plan's own implementation sections. That breadth is consistent with PLAN.md, which is itself organized by scope, architecture, data model, engine, rendering, audio, accessibility, social, privacy, performance, rollout, and risks.

## Plan-derivation spot check

1. Reconstruction: "Aliveness is the product." Support: PLAN.md line 5 says "aliveness is the product" and defines it as four engine invariants. Result: supported.
2. Reconstruction: "The server is the sole writer of canonical life." Support: PLAN.md lines 68-80 state that `sim` is the only writer of personality/mood/scene state and clients only emit events/render snapshots. Result: supported.
3. Reconstruction: "Accessibility surfaces deliver the actual product." Support: PLAN.md line 436 says accessibility surfaces deliver "the actual product, not a stripped variant" and ship with v1. Result: supported.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no near-1:1 gold mapping, no rubric vocabulary, and the most articulate phrases are directly traceable to PLAN.md. Minor scoring-level over-inference appears in F19, but that is a normal reconstruction-grounding issue rather than contamination evidence.
