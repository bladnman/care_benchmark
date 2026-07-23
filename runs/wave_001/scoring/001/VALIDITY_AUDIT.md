# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A mechanical scan of the frozen reconstruction for tokens matching gold IDs such as F1-F40, S1-S9, R-Fxx, or R-Sxx returned no hits. Because no offending ID appeared in RECONSTRUCTION.md, no PLAN cross-reference leakage hit was needed.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION.md and PLAN derivation:

| Phrase from reconstruction | PLAN support | Assessment |
| --- | --- | --- |
| single horizontal aviary scene | PLAN.md §1.1 uses the same phrase. | plan-derived |
| quiet field | PLAN.md §7.4 says boot path is a quiet field without spinner. | plan-derived |
| small birds under a quiet window | PLAN.md §18 contains the same phrase. | plan-derived |
| identity death invisible | PLAN.md §12 risk table uses this phrase. | plan-derived |
| No naturalist evasion | PLAN.md §6.7 uses this phrase for failure surfaces. | plan-derived |
| A11y treated as ARIA stickers | PLAN.md §12 risk table uses this phrase. | plan-derived |
| 500ms lived experience | PLAN.md §12 bundle-bloat risk uses this phrase. | plan-derived |
| cultural backdoor | PLAN.md §12 uses this phrase for internal gamification metrics. | plan-derived |

No scorer-side phrases such as gold, rubric, weight-3, multi-layer recovery, feature-level fidelity, or system-level fidelity appeared. Hits for words like scores/weights were plan-derived uses such as engagement scores and calibration weights, not rubric vocabulary.

## Heading Mirror Check

The top-level reconstruction headings are the phase-2A required headings: "System-level intent" and "Per-feature whys." The subheadings under per-feature whys mirror the PLAN's structure in collapsed form: Scope, Resolved ambiguities, Architecture, Data model and storage, API surface, Simulation engine, Sync model, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Performance and observability, Rollout, Risks/testing/privacy/success, and Open implementation tickets.

These headings do not mirror GOLD_WHYS.md section titles or the S1-S9/F1-F40 taxonomy. They are plan-derived.

## 1:1 Mapping Suspect Check

No 1:1 mapping to the gold target list was found. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and follows the plan's own implementation order with many more items than the 49 gold why rows. This is not a neat held-out-list-shaped reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
| --- | --- | --- |
| The plan protects stable identity forever, species hotfixes never recycle bird.id, and success depends on recognizing birds by call alone. | PLAN.md §§3.3 and 17: Bird.id is stable identity forever; species hotfixes never recycle it; dogfood success requires recognizing birds by call. | grounded |
| The plan separates naturalist prose for narration, notebook, captions, and offer prompts from matter-of-fact auth, errors, settings, and unavailable visits. | PLAN.md §15 defines copy/naturalist and copy/system, and §6.7 requires matter-of-fact failure surfaces. | grounded |
| The plan rejects recorded-audio call libraries and says canned audio breaks the spell. | PLAN.md §§1.2, 8, and 12 ban recorded call libraries and list canned audio as a spell-breaking risk. | grounded |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived: it uses the plan's headings, quotes, and vocabulary; it does not leak gold IDs or rubric terms; and spot-checked articulate claims are directly grounded in PLAN.md.
