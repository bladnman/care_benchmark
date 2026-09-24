# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict: PASS.

A mechanical search of the frozen reconstruction for gold-side IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` returned no hits. The reconstruction uses plan-native labels such as H1, D-02, G-A, M0, and R-01, all of which are present in the PLAN or are plan section/risk labels rather than gold taxonomy.

## Vocabulary check

Verdict: PASS.

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "survive a year of well-meaning contributions" | PLAN line 13 has the same phrase. | Plan-derived. |
| "Reviewer memory isn't enough" | PLAN line 13 has the same sentence. | Plan-derived. |
| "first frame is mid-action" | PLAN H4 line 20 and §7.1. | Plan-derived. |
| "sitting still and watching is the product" | PLAN line 430. | Plan-derived. |
| "NO credentials for and NO network route to sim_db" | PLAN architecture diagram line 149. | Plan-derived. |
| "create the data product" | PLAN line 1286. | Plan-derived. |
| "the actual product" | PLAN line 1041. | Plan-derived. |
| "calmer and deliberate, never as broken" | PLAN line 840. | Plan-derived. |
| "golden replays" | PLAN quality section uses this term; the word "golden" is not gold-list leakage. | Benign plan vocabulary. |

No rubric-side phrases such as "multi-layer recovery", "feature-level fidelity", "system-level fidelity", or "weight-3" appear in the reconstruction.

## Heading mirror

Verdict: PASS.

The top-level headings are exactly the required phase-2A output headings: `## System-level intent` and `## Per-feature whys`. The `###` headings under per-feature whys mirror PLAN sections (`0. The hard rules and refusals`, `1. Scope and interpretive decisions`, `2. Architecture`, etc.), not held-out gold-list sections. They do not mirror S1-S9 or F1-F40 names or ordering.

## 1:1 mapping suspect

Verdict: PASS.

The reconstruction does not provide a neat S1-S9 and F1-F40 mapping. It has 13 system-level principles and a plan-section-based per-feature list. Several items are marked `NOT RECOVERABLE FROM PLAN`, and many plan implementation areas appear that have no gold-why counterpart. This structure follows PLAN.md rather than the held-out gold list.

## Plan-derivation spot check

1. Reconstruction: "Affective promises must be architectural, not remembered."
   PLAN support: line 13 says the promises must "survive a year of well-meaning contributions" and that "Reviewer memory isn't enough," then maps them to invariants enforced by architecture, DB permissions, lints, and CI.

2. Reconstruction: "Privacy is a product boundary, not a reporting choice."
   PLAN support: the hard rule H5 requires synthetic UUIDs, encrypted email in one table, and per-account interaction data never leaving sim_db; the architecture diagram states the ops plane has no credentials or network route to sim_db; §13.4 says collecting engagement data would create the refused data product.

3. Reconstruction: "Accessibility is a designed surface of the same product."
   PLAN support: line 1041 says every accessibility surface gets "the actual product," and §7.9 says reduced motion must be "calmer and deliberate, never as broken."

All three spot-checked sentences are directly derived from the plan.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived, uses plan structure and plan vocabulary, contains no gold IDs, and does not map one-to-one to the scorer's held-out target list.
