# VALIDITY_AUDIT — CARE run 001

## ID Leakage

**Result: PASS.** Mechanical search found no gold IDs or rebuild IDs in `RECONSTRUCTION.md` (`F1`, `S1`, `R-F01`, `R-S01`, etc.). The same search also found no such IDs in the assigned PLAN, so there are no leakage hits to cross-reference.

## Vocabulary Check

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | Appears in / is grounded by PLAN? | Assessment |
|---|---|---|
| "architectural constraints, not review vigilance" | PLAN §1 Scope risk note uses the same wording. | Plan-derived. |
| "client is a rendering and input surface" | PLAN §2.2 states the client is a rendering/input surface and never owns state. | Plan-derived. |
| "Expose behavior, not numbers" | PLAN §2.4 says raw personality numbers are never serialized and only behavior is exposed. | Plan-derived paraphrase. |
| "the moment a count exists in the schema" | PLAN §3.6 exact rationale for anti-gamification schema. | Plan-derived. |
| "zero read access" | PLAN §3.7 telemetry boundary. | Plan-derived. |
| "structurally invisible to the tick's drift math" | PLAN §10.2 uses this phrasing for visit events. | Plan-derived. |
| "v1.1 fix" | PLAN §8.5 says reduced motion must not land as a v1.1 fix. | Plan-derived. |

I found no scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity` in the reconstruction.

## Heading Mirror

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Scope`
- `### 2. Architecture`
- `### 3. Data model`
- `### 4. Simulation engine design`
- `### 5. Sync model`
- `### 6. Field notebook / narrative generation pipeline`
- `### 7. Presence engine`
- `### 8. Frontend rendering pipeline`
- `### 9. Audio pipeline`
- `### 10. Visit-invitation flow`
- `### 11. Accessibility surfaces`
- `### 12. Performance budgets and observability`
- `### 13. Rollout`
- `### 14. Risks`

These mirror the PLAN's numbered headings, not the gold-list S/F taxonomy. The first two headings are required by the reconstructor prompt. No gold section title mirror was found.

## 1:1 Mapping Suspect

**Result: not suspect.** The reconstruction does not create a neat S1-S9 or F1-F40 sequence. It has 13 system principles and per-feature groups following the PLAN's 14 sections. Many PLAN features are merged, and some are marked `NOT RECOVERABLE FROM PLAN`. This is not a 1:1 mapping to the held-out gold list.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The client is a rendering and input surface; the server owns state."
   - PLAN support: §2.2 says the client renders snapshots, sends events, and never computes personality, mood, or drift; §2.1 says the simulation service is the only writer.
   - Assessment: supported.

2. Reconstruction sentence: "No schema stores visit counts, streaks, days active, or frequency aggregates."
   - PLAN support: §3.6 says no table stores derived visit count, streak length, days active, or frequency aggregate.
   - Assessment: supported.

3. Reconstruction sentence: "Visitor events are structurally invisible to the tick's drift math."
   - PLAN support: §10.2 says visit-session events are recorded for the host log only and filtered out of drift computation.
   - Assessment: supported.

## Verdict

**PASS.** The reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary, no gold-heading mirror, and no 1:1 mapping to S1-S9/F1-F40. Its organization and phrasing closely track the assigned PLAN's own sections and wording.
