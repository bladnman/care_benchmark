# VALIDITY_AUDIT - CARE run 001

## ID Leakage

PASS. Mechanical search of the frozen reconstruction found no gold IDs or external scoring IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. There were therefore no offending IDs to cross-check against PLAN.md.

## Vocabulary Check

No scorer-side vocabulary appeared in the reconstruction. Mechanical search found no `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, `intent fidelity`, or `CARE` terms.

Sampled plan-derived phrases:

| Reconstruction phrase | PLAN support |
|---|---|
| "build a place that continues without the viewer" | PLAN executive summary/product one-liner uses the same phrase. |
| "measure attention, not clicks" | PLAN product one-liner uses the same phrase and the presence monitor implements it. |
| "Never punish absence" | PLAN product one-liner and drift section use monotonic expressive/no negative drift rules. |
| "No naturalist cosplay for failures" | PLAN §6.6 uses this exact system-voice rule. |
| "Hard wall" analytics DB role cannot read simulation tables | PLAN observability section uses this hard-wall privacy language. |
| "charming naturalist surfaces, not dumps" | PLAN success criteria say reduced-motion, SR, and captions are charming surfaces, not dumps. |
| "out of product-not a backlog item" | PLAN §16 says conflicts with core principles are out of product, not backlog items. |

## Heading Mirror

The reconstruction has only the two required headings:

| Heading | Assessment |
|---|---|
| `## System-level intent` | Required by the phase-2A prompt; not a gold-list mirror. |
| `## Per-feature whys` | Required by the phase-2A prompt; not a gold-list mirror. |

No `###` headings or gold-section-title mirrors appear.

## 1:1 Mapping Suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold list order. Its per-feature section follows the PLAN's own structure: Executive summary/scope, Architecture, Data model, API surface, Simulation engine, Sync model, Frontend, Audio, Accessibility, Performance, Rollout, and Risks/standards. The item count is far larger and differently grouped than the 49 gold whys, which argues against a neat scorer-side mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| "Presence validity requires visible, focused, and recent activity, while still leaning long because 'watching is stillness.'" | PLAN §1.3 sets a 180s activity window and says "watching is stillness"; PLAN §5.2 requires visible/focused/recent activity. | PASS |
| "Visitor snapshot deliberately omits host settings/PII." | PLAN §4.4 says visitor snapshots omit host settings/PII. | PASS |
| "A morning return should not be 'yesterday's dusk frozen incorrectly.'" | PLAN §5.3 uses this exact offline mood-path rationale. | PASS |

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no gold-ID leakage, no rubric vocabulary, no gold-heading mirrors, and no suspicious 1:1 gold mapping. The most articulate sampled claims are directly supported by PLAN.md. Scores are not flagged for contamination.
