# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: PASS.

A mechanical search of the frozen reconstruction for gold-style IDs (F1-F40, S1-S9, R-Fxx, R-Sxx) returned no hits. The corresponding PLAN search also returned no hits. There is no ID leakage evidence.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION and plan support:

| Reconstruction phrase | PLAN support | Assessment |
| --- | --- | --- |
| "relationship with a window" | PLAN intro uses the same phrase. | plan-derived |
| "Ship a window that was already going" | PLAN closing sentence uses the same phrase. | plan-derived |
| "Devices are projectors" | PLAN section 6.1 uses the same phrase. | plan-derived |
| "No announcement, score, punishment, or neediness" | PLAN intro and sections 1.2, 10.3, 12.6 reject announcements, scores, and making birds needier. | plan-derived |
| "Accessibility is a designed surface" | PLAN section 0 says accessibility ships as designed surfaces, not fallbacks. | plan-derived |
| "Privacy is pipeline shape" | PLAN section 10.4 is titled Privacy = pipeline shape. | plan-derived |
| "Aliveness is the release gate" | PLAN section 11.1 says aliveness is the release gate, not feature count. | plan-derived |
| "canonical" | PLAN repeatedly uses server-canonical state and canonical snapshot language. | ordinary plan vocabulary |
| "score" | Appears in PLAN as a product/gamification term, not as scorer vocabulary. | not suspicious |
| "recovery" | Appears only in deletion/recover account context, not benchmark recovery vocabulary. | not suspicious |


Suspicious scorer-side terms such as gold, rubric, weight-3, multi-layer recovery, feature-level fidelity, and system-level fidelity did not appear in the reconstruction.

## Heading Mirror Check

RECONSTRUCTION headings are:

- System-level intent
- Per-feature whys
- 1. Scope
- 2. Architecture
- 3. Data model
- 4. API surface
- 5. Simulation engine
- 6. Sync model
- 7. Frontend rendering pipeline
- 8. Audio pipeline
- 9. Accessibility surfaces
- 10. Performance, observability, rollout, risks, testing, and team shape

The first two headings are required by the phase-2A prompt. The numbered headings mirror PLAN.md's implementation structure, not GOLD_WHYS.md's section structure or S/F taxonomy. No gold heading mirror was found.

## 1:1 Mapping Suspect Check

Verdict: PASS.

The reconstruction does not enumerate S1-S9 or F1-F40, does not produce 49 neat target rows, and does not follow the gold order. It has 12 system-level bullets and then plan-section groupings with many merged feature rationales plus several explicit NOT RECOVERABLE FROM PLAN entries. This reads as a reconstruction of the PLAN, not a 1:1 mapping to held-out gold targets.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
| --- | --- | --- |
| "The product intent is change through presence without punishment." | PLAN sections 0 and 5.2: drift monotonic toward expressive; neglect gives Delta x = 0; traits hold. | grounded |
| "The split is the product." | PLAN section 2.4 says this exact sentence after explaining client/server simulation boundaries. | grounded |
| "Reduced-motion is not a skipped animation flag because accessibility is first-class." | PLAN sections 2.3 and 7.7 say reduced-motion is a renderer swap, not a skip-rAF flag, and section 0 says accessibility is first-class. | grounded |


## Verdict

PASS.

The frozen reconstruction shows no meaningful contamination signatures. It uses plan vocabulary, plan headings, and plan-local quotes; it does not leak gold IDs, rubric terms, or a 1:1 gold taxonomy. The minor vocabulary hits are ordinary product terms already present in PLAN.md.
