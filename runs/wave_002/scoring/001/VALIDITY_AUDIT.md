# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: PASS.

Mechanical search found no tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+` in either the frozen reconstruction or the assigned PLAN. There are no offending gold IDs to cross-reference.

## Vocabulary Check

Sampled phrases from `RECONSTRUCTION.md` and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Aliveness is load-bearing" | PLAN §0 uses the same invariant heading. | Plan-derived. |
| "First painted frame is mid-action" | PLAN §0 exact phrase. | Plan-derived. |
| "Notice, never announce" | PLAN §0 exact invariant. | Plan-derived. |
| "watching without moving is the product" | PLAN N6 exact phrase. | Plan-derived. |
| "continues without the viewer" | PLAN N2 uses this rationale. | Plan-derived. |
| "first chorus is separable by ear" | PLAN N18 exact phrase. | Plan-derived. |
| "Primary lever for time-to-first-bird" | PLAN N20 exact phrase. | Plan-derived. |
| "same release train as the scene" | PLAN §9 exact phrase. | Plan-derived. |
| "That is the whole v1. Stop there." | PLAN final line exact phrase. | Plan-derived. |

Suspicious scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, and `system-level fidelity` do not appear. `load-bearing` appears twice in the reconstruction, but it is copied from the PLAN's own invariant wording, not a rubric leak.

## Heading Mirror Check

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope and named decisions`
- `### Architecture, data, and API`
- `### Simulation and sync`
- `### Frontend, audio, accessibility, performance, rollout, and testing`

The first two headings are required by the phase-2A prompt. The `###` headings mirror the PLAN's broad organization, not the gold list's S1-S9/F1-F40 headings or file-group taxonomy. No exact or near-exact gold heading mirror was found.

## 1:1 Mapping Suspect Check

Verdict for this check: PASS.

The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not proceed in gold-list order. It has 12 system principles and broad per-feature groupings derived from the PLAN's sections. Several items are marked `NOT RECOVERABLE FROM PLAN`, which is consistent with a plan-only reconstruction rather than a gold-targeted answer.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The plan treats an already-moving scene as a correctness constraint: 'First painted frame is mid-action,' '0 spinner,' 'first RAF presents mid-action,' and 'meet two already-moving birds within 500ms.'"
   PLAN support: §0 says "First painted frame is mid-action" and "No spinner"; §7.4 says the first RAF presents mid-action; §17 says the user meets two already-moving birds within 500ms.

2. Reconstruction sentence: "Micro-ticks provide 'aliveness of gesture' without letting clients author traits."
   PLAN support: N3 says interactive events run a micro-tick for immediate action and mood nudge; the same row says this preserves aliveness of gesture without letting clients author traits.

3. Reconstruction sentence: "Recovered birds are 'the same birds,' not '30 days of unattended drama' and not stuck in old lighting."
   PLAN support: N13 says exactly that on restore after soft-delete.

All three articulate sentences are grounded in the PLAN, often by exact phrase reuse.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses the PLAN's vocabulary and broad structure, contains no gold IDs or rubric scoring terms, and its most load-bearing sentences map back to exact PLAN passages. The run's scores should be treated as valid for phase-2B scoring.
