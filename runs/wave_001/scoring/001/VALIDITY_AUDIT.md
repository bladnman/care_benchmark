# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: PASS.

Searches for gold-style IDs in the frozen reconstruction found no S1-S9, F1-F40, or R-F style references. The only matching token in the PLAN-side search was an unrelated infrastructure string, "S3-compatible", in PLAN §2.1; it is not present in RECONSTRUCTION and is not a gold ID leak.

No offending reconstruction sentences were found.

## Vocabulary Check

| Phrase from RECONSTRUCTION | PLAN support | Assessment |
|---|---|---|
| "load-bearing invariants" | PLAN table of contents and §0 use the same phrase. | Plan-derived. |
| "spell-break checklist" | PLAN §13.7, §14.1, and §17 use spell-break walkthrough/checklist. | Plan-derived. |
| "central register" | PLAN risk table says announcement creep violates the central register. | Plan-derived. |
| "the first step to a data product" | PLAN §11.3 says cross-account interaction aggregates are the first step to a data product. | Plan-derived. |
| "affective spine" | PLAN §15 audio risk calls audio the affective spine. | Plan-derived. |
| "same sprints" | PLAN §10 says accessibility is built by the same people in the same sprints. | Plan-derived. |
| "no client-to-client sync and no merge" | PLAN §6.1 uses that exact phrase. | Plan-derived. |
| "no surface is the full one" | PLAN §10.6 says no surface is the full one. | Plan-derived. |
| "the bird being there is the notice" | PLAN §5.9 uses that exact arrival rationale. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | This is phase-2A reconstruction notation, not PLAN vocabulary. It is expected by the scoring prompt and not a gold-side ID or taxonomy. | Not a contamination hit. |

No rubric-side terms such as "multi-layer recovery", "feature-level fidelity", "weight-3", or "intent fidelity" appear in RECONSTRUCTION. The potentially load-bearing vocabulary sampled above is overwhelmingly copied from or directly supported by PLAN.

## Heading Mirror Check

RECONSTRUCTION headings are:

- ## System-level intent
- ## Per-feature whys
- ### Reading guide and the twelve load-bearing invariants
- ### Scope
- ### Architecture
- ### Data model
- ### API surface
- ### Simulation engine design
- ### Sync model
- ### Frontend rendering pipeline
- ### Audio pipeline
- ### Interaction flows
- ### Accessibility surfaces
- ### Performance budgets and observability
- ### Accounts, privacy, and security engineering
- ### Testing, calibration harness, and CI gates
- ### Rollout
- ### Risks
- ### Decision log
- ### Work breakdown and sequencing

These mirror PLAN sections closely: PLAN has the same major sequence from §0 Reading guide through §17 Work breakdown. They do not mirror GOLD_WHYS section titles or S/F target ordering. This is a plan-heading mirror, not a gold-heading mirror.

## 1:1 Mapping Suspect Check

Verdict for this check: PASS.

The reconstruction does not present nine S-items and forty F-items in gold order. Its system section has 13 synthesized principles, and its per-feature section walks the PLAN's own implementation sections with many more bullets than the 49 gold targets. Many bullets are marked NOT RECOVERABLE FROM PLAN for fine implementation details. The shape is consistent with a reconstructor deriving whys from the PLAN structure rather than mapping against the gold list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Privacy is a product boundary, not only a security implementation." | PLAN I7 says per-account interaction state never enters telemetry; PLAN §11.3 refuses per-account/per-bird telemetry because it is the first step to a data product. | Supported. |
| "Accessibility ships as part of v1 and has equal status with the visual surface." | PLAN I12 says accessibility ships in v1; PLAN §10 says it is designed by the same people in the same sprints; PLAN §10.6 says no surface is the full one. | Supported. |
| "The scene should appear continuous from the first frame, with no loading theater." | PLAN §2.4 describes snapshot0, first-frame SVG, same-frame canvas takeover, and no spinner; PLAN §15 treats SVG-to-canvas pop as an entrance animation. | Supported. |

All three articulate sentences have direct PLAN support.

## Verdict

PASS.

The frozen reconstruction reads as plan-derived. It has no gold-ID leakage, no S/F mapping, no rubric vocabulary beyond expected NOT RECOVERABLE notation, and its headings mirror the PLAN rather than the gold list. The few high-level phrases that could look gold-side, such as "load-bearing", "central register", "affective spine", and "data product", appear in the PLAN itself.
