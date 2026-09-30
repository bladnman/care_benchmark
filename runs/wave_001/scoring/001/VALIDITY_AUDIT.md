# VALIDITY_AUDIT - CARE run 001

## ID Leakage

**Finding: PASS.** Mechanical search found no gold IDs or external rebuild IDs in `RECONSTRUCTION.md` (`F1`, `S1`, `R-F01`, etc.). No offending sentence was found, and the reconstruction does not use the gold taxonomy.

## Vocabulary Check

Sampled phrases against the plan:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "one private aviary per single-user account" | Exact phrase appears in PLAN §1. | Plan-derived |
| "starts already in motion" | Exact phrase appears in PLAN §1. | Plan-derived |
| "the sole writer" | PLAN §2 says the simulation worker is the sole writer. | Plan-derived |
| "Traits never decrease" | Exact phrase appears in PLAN §6/§12. | Plan-derived |
| "reviewed aesthetic" | PLAN §7 says reduced motion is a reviewed aesthetic. | Plan-derived |
| "never engagement/retention or population bird statistics" | Exact rollout language appears in PLAN §13. | Plan-derived |
| "zero visitor drift" | PLAN §13 exit gate says visitor attention has zero effect; reconstruction phrase is a close paraphrase. | Plan-derived |
| "NOT RECOVERABLE FROM PLAN" | This phrase came from the phase-2A prompt format, not the gold list. | Expected |

No scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity` appears in the reconstruction.

## Heading Mirror

The top-level headings are the required `## System-level intent` and `## Per-feature whys`. Subheadings under per-feature whys mirror the PLAN's section order: `Product boundary and decisions`, `Architecture and responsibility boundaries`, `Persistent data model`, through `Risks and mitigations`. They do not mirror the gold-list section titles or S/F ordering.

## 1:1 Mapping Suspect

**Finding: PASS.** The reconstruction does not provide a neat S1-S9/F1-F40 mapping. It lists 12 system principles, then many plan-derived feature bullets in the PLAN's own section order. This is not the gold order, and it contains many non-gold implementation details as well as several explicit `NOT RECOVERABLE FROM PLAN` rows.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The simulation worker is `the sole writer` of personality and canonical plans; the Owner API `never updates personality columns`; the client `never runs mood or drift simulation`."
   PLAN support: §2 responsibilities use those same phrases for the simulation worker, Owner API, and client.

2. Reconstruction sentence: "Traits never decrease, absence trends toward `ordinary ambient behavior, never mistrust, loss of color, or distress`."
   PLAN support: §1 decision table states exactly that long absence trends toward ordinary ambient behavior and never mistrust, loss of color, or distress.

3. Reconstruction sentence: "Accessibility is present now, not reserved for the end, and reduced motion is a `reviewed aesthetic` rather than paused animation."
   PLAN support: §13 says accessibility is present now; §7 says reduced motion is a reviewed aesthetic, not `animation: none`.

All three checked sentences are grounded in the PLAN.

## Verdict

**PASS.** The reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary, no gold-heading mirror, no 1:1 gold mapping, and spot-checked articulate claims trace back to the PLAN. The few polished phrases are either exact plan language or close paraphrases of plan sections.
