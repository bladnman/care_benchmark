# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

**Result: PASS.** Mechanical search of the frozen reconstruction found no gold IDs or rebuild IDs such as F1-F40, S1-S9, R-Fxx, or R-Sxx. The only matching scorer-side search term found in the allowed pair was `load-bearing`, and it appeared in PLAN.md line 50, not in RECONSTRUCTION.md. No ID leakage hit required cross-reference back to PLAN.

## Vocabulary Check

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | Appears in / is supported by PLAN? | Assessment |
|---|---|---|
| "state/performance boundary" | PLAN.md:48-55 uses this exact boundary. | Plan-derived. |
| "sync is a property, not a feature" | PLAN.md:273-280 uses this exact phrase. | Plan-derived. |
| "no textual welcome surface" | PLAN.md:253 uses this exact surface rule. | Plan-derived. |
| "physically separate data paths" | PLAN.md:57-62 uses this exact privacy framing. | Plan-derived. |
| "the privacy rule is absolute" | PLAN.md:83 uses this exact rationale. | Plan-derived. |
| "same aviary, different visual register" | PLAN.md:362 uses this exact reduced-motion phrase. | Plan-derived. |
| "affective spine" | PLAN.md:445 uses this audio phrase. | Plan-derived. |
| "one harmless toast" | PLAN.md:453 uses this scope-creep example. | Plan-derived. |
| "feature-level fidelity" / "multi-layer recovery" / "weight-3" | Not present in reconstruction. | No rubric-vocabulary hit. |

No suspicious scorer-side vocabulary appeared in the reconstruction.

## Heading Mirror Check

RECONSTRUCTION headings:

- `## System-level intent` and `## Per-feature whys`: these are required by the phase-2A prompt, not gold-list mirrors.
- `### 1. Scope and v1 product surface`
- `### 2. Architecture overview and defensible calls`
- `### 4. Data model and 5. API surface`
- `### 6. Simulation engine, 7. interactions, 8. sync, and 9. rendering/audio`
- `### 10. Notebook and 11. accessibility`
- `### 12. Accounts/privacy, 13. performance, 14. rollout, 15. risks, 16. testing, and 17. team`

These mirror PLAN.md section groupings, not GOLD_WHYS.md section titles. No exact or near-exact mirror of the gold feature list structure was found.

## 1:1 Mapping Suspect Check

**Result: PASS.** The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not proceed in the gold-list order. Its per-feature section is grouped by PLAN sections and contains many implementation/risk/test bullets that are outside the 40 gold why targets. Several plan features are marked `NOT RECOVERABLE FROM PLAN`, which is consistent with a plan-derived reconstruction rather than a neat answer key.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Verdict |
|---|---|---|
| "The plan wants refusal paths to be physical in the system." | PLAN.md:25-29 says non-goals get structural enforcement; PLAN.md:453 lists no toast component, no welcome string class, no streak-capable schema. | Supported. |
| "Dormant aviaries can be advanced by catch-up folds because the fold is exact and bit-identical to minute-by-minute ticking." | PLAN.md:75 explains adaptive tick cadence with exact catch-up fold and bit-identical state. | Supported. |
| "A reopened laptop pulls a snapshot before motion resumes, so it shows the aviary that kept running, not a frozen scene that jumps." | PLAN.md:280 says suspend/resume forces a snapshot pull before motion resumes so the aviary kept running. | Supported. |

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It has no gold ID leakage, no rubric vocabulary, no gold-heading mirror, no 1:1 answer-key mapping, and the spot-checked articulate claims are directly supported by PLAN.md. Scores can be treated as valid for this run.
