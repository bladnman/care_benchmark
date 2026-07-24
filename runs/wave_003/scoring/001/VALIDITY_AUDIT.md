# VALIDITY_AUDIT - run 001

## ID Leakage Check

Verdict: PASS. Mechanical search of the frozen reconstruction found no gold IDs or rebuild IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. No cross-reference hits against PLAN were needed because there were no offending IDs.

## Vocabulary Check

Sampled phrases read as plan-derived rather than rubric-derived:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Concept poison" | PLAN.md:L935 | Exact plan vocabulary. |
| "Lost relationship history" | PLAN.md:L930 | Exact plan vocabulary. |
| "Instrument health, not exploitation" | PLAN.md:L765 | Exact plan vocabulary. |
| "server-canonical state" | PLAN.md:L9, L520-L531 | Plan vocabulary. |
| "equal product quality" | PLAN.md:L619 | Exact plan vocabulary. |
| "never spinner-as-brand" | PLAN.md:L547 | Exact plan vocabulary. |
| "privacy-bounded telemetry" | PLAN.md:L18 | Exact plan vocabulary. |
| "matter-of-fact" system errors | PLAN.md:L288, L398-L400 | Plan vocabulary. |
| "fidelity" at RECONSTRUCTION.md:L247 | PLAN.md:L664-L666 | Ordinary product wording about captions matching synthesized calls, not scorer-side intent fidelity. |

No suspicious scorer-side phrases such as "gold", "rubric", "weight-3", "multi-layer recovery", "feature-level fidelity", or "system-level fidelity" appear in the reconstruction. The word "recovery" appears only in account deletion context.

## Heading Mirror Check

The top-level headings are the mandated `## System-level intent` and `## Per-feature whys`. The `###` headings mirror PLAN sections (`Scope`, `Architecture`, `Data model`, `API surface`, `Simulation engine design`, etc.), not GOLD_WHYS section titles or S/F taxonomy. This is expected plan-derived structure.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold order. It follows the PLAN order and includes many non-gold implementation features marked `NOT RECOVERABLE FROM PLAN`, which is consistent with the phase-2A prompt and inconsistent with a neat held-out mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Server-canonical continuity instead of client-owned simulation." | PLAN.md:L9, L91, L531-L535 | Directly grounded. |
| "Reduced motion is equal product quality, not muted broken checkbox." | PLAN.md:L615-L619 | Directly grounded. |
| "Privacy-bounded data and telemetry... simulation DB not CDC'd into warehouse." | PLAN.md:L740-L755 | Directly grounded. |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses the PLAN's own headings, vocabulary, and ordering, contains no gold IDs, and its most articulate claims trace to exact PLAN passages. Scores can be treated as measuring plan carry-through rather than reconstruction contamination.
