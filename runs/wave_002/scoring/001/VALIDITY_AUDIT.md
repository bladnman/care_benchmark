# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS.

Mechanical search of the frozen reconstruction found no gold IDs or external taxonomy tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. Because there were no hits, no PLAN cross-check was needed for leaked IDs.

## Vocabulary Check

Verdict: PASS.

Sampled phrases from the reconstruction read as plan-derived rather than rubric-derived:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "the only writer of canonical aviary state" | PLAN §1 and §2 use the same phrase for the server-side tick. | Plan-derived. |
| "clients never write personality state" | PLAN §2.2: clients never write personality state under any code path. | Plan-derived. |
| "ambient quietness, not regression" | PLAN §5.2 uses this exact rationale for neglect. | Plan-derived. |
| "noticing, not announcing" | PLAN §4.2 calls the notebook dot "noticing, not announcing." | Plan-derived. |
| "no rows in interaction_events" | PLAN §3.2 says visitor sessions produce no such rows. | Plan-derived. |
| "launch-blocking features" | PLAN §9.4 says reduced motion, narration, and captions are launch-blocking. | Plan-derived. |
| "signature preserved" | PLAN §8.1 uses this phrase for procedural calls. | Plan-derived. |
| "CI-enforced, not guidelines" | PLAN §11.1 uses this phrase for performance budgets. | Plan-derived. |

No suspicious scorer-side terms such as gold, rubric, weight-3, multi-layer recovery, feature-level fidelity, or system-level fidelity appeared in the reconstruction.

## Heading Mirror

Verdict: PASS.

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then plan-shaped section headings: Scope, Architecture, Data model, API Surface, Simulation Engine Design, Sync Model, Frontend Rendering Pipeline, Audio Pipeline, Accessibility Surfaces, Data Lifecycle Features, Performance Budgets and Observability, Rollout, and Risks and Mitigations.

These mirror the PLAN structure, not the held-out gold-list sections. They do not enumerate S1-S9 or F1-F40 and do not use gold-list headings.

## 1:1 Mapping Suspect

Verdict: PASS.

The reconstruction does not provide a neat item for every gold target in gold order. It follows the PLAN's own sections and bullet clusters, includes many implementation features outside F1-F40, and marks several items `NOT RECOVERABLE FROM PLAN`. That shape is consistent with blind reconstruction from the plan rather than contamination from the gold list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The intent is to make divergent simulation and absolute client writes unreachable by construction." | PLAN §§2.2 and 6: clients write events, no request type accepts absolute trait values, and overlapping device sessions fold into one vector. | Grounded. |
| "The plan treats neglect as absence of signal, not decay." | PLAN §5.2: neglect produces absence of signal, which produces absence of drift - ambient quietness, not regression. | Grounded. |
| "The rationale is that stale rendering would fake continuity, which the plan says is worse than an honest quiet field." | PLAN §6: no offline cached rendering; stale state would fake continuity, worse than an honest quiet field. | Grounded. |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction is detailed, but its vocabulary, headings, and evidence trail follow the PLAN rather than the held-out gold list or scoring rubric. Scores can be treated as valid for this run.
