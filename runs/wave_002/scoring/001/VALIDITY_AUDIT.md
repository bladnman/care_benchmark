# VALIDITY_AUDIT — CARE run 001

## ID Leakage Check

**Result: PASS.** I found no gold IDs, rubric IDs, or external scoring taxonomy in the frozen reconstruction. A targeted search for `S1`-style IDs, `F1`-style IDs, `R-F` IDs, `multi-layer`, `feature-level`, `intent fidelity`, `weight-2`, `weight-3`, `load-bearing`, `fidelity`, and `gold` returned no hits in either the PLAN or RECONSTRUCTION.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Verdict |
|---|---|---|
| "Core but bounded v1 scope" | PLAN §Scope says v1 includes core areas and strict non-goals. | Plan-derived synthesis |
| "Canonical server state" | PLAN §Sync Model: "Canonical state lives on the server." | Directly supported |
| "server tick as authority" | PLAN §Sync Model: "server simulation tick is the only writer." | Directly supported |
| "Append-only interaction history" | PLAN §Data Model: "Append-only list of interactions." | Directly supported |
| "Naturalist prose" | PLAN §Accessibility: "First-class narration (naturalist prose)." | Directly supported |
| "Procedural, web-delivered sensory experience" | PLAN §Client/Rendering names procedural rendering, canvas, WebAudio, 60fps. | Plan-derived synthesis |
| "Accessibility is part of the experience surface" | PLAN gives Accessibility its own section and flags "Accessibility design vs. standard ARIA automation." | Plan-derived synthesis |
| "Constrained social and telemetry surfaces" | PLAN §Scope has read-only social visits/no public social surfaces; §Rollout has aggregate telemetry. | Plan-derived synthesis |

No sampled phrase reads like rubric-side language. The reconstruction uses ordinary plan-derived synthesis rather than terms such as "multi-layer recovery," "feature-level fidelity," or "gold why."

## Heading Mirror Check

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then plan-structure headings such as `### Scope`, `### Architecture`, `### Data Model`, `### API Surface`, `### Simulation Engine`, `### Sync Model`, `### Rendering & Audio Pipeline`, `### Accessibility`, `### Performance`, and `### Rollout`. These mirror the PLAN headings, not the gold-list headings. They do not closely echo gold section titles like `feels-alive-not-robotic`, `notice-never-announce`, `presence-definition`, or `drift-function`.

## 1:1 Mapping Suspect Check

**Result: PASS.** The reconstruction does not enumerate S1-S9 or F1-F40, does not contain 49 neat target-aligned rows, and does not proceed in gold-list order. Instead it follows the candidate PLAN's compressed sections and only reconstructs whys for the features named there. This is exactly the expected shape for a plan-derived reconstruction.

## Plan-Derivation Spot Check

1. Reconstruction: "Canonical server state with the server tick as authority."  
   PLAN support: §Sync Model states "Canonical state lives on the server" and "server simulation tick is the only writer."  
   Verdict: supported.

2. Reconstruction: "Accessibility is part of the experience surface, not only compliance."  
   PLAN support: §Accessibility lists first-class narration, reduced-motion, and captioning; §Risks names "Accessibility design vs. standard ARIA automation."  
   Verdict: supported as synthesis.

3. Reconstruction: "Constrained social and telemetry surfaces."  
   PLAN support: §Scope includes read-only social visits and excludes public social-network surfaces; §Rollout restricts telemetry to aggregate operational metrics.  
   Verdict: supported as synthesis.

## Verdict

**PASS.** The frozen reconstruction reads as a faithful synthesis of the candidate PLAN's own headings and wording. It contains no gold ID leakage, no rubric vocabulary, no gold-heading mirror, and no 1:1 mapping to the gold target list. Its losses are ordinary non-recovery from a compressed plan, not contamination signatures.
