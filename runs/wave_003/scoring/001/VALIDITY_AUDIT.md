# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: PASS. I found no gold IDs such as `S1`-`S9`, `F1`-`F40`, or `R-Fxx` in `RECONSTRUCTION.md`. A mechanical search for rubric/gold identifiers returned no ID hits. The only notable term hit was `load-bearing`, but that word also appears in `PLAN.md` (for example, the continuity band and template-grammar sections), so it is not leakage by itself.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "mostly affective" | PLAN §0 says correctness conditions are mostly affective. | Plan-derived. |
| "made structurally unrepresentable" | PLAN §0 uses this exact organizing conviction. | Plan-derived. |
| "The bird is the welcome" | PLAN §7.1 uses this exact sentence. | Plan-derived. |
| "the query does not exist" | PLAN INV-8 and §12.6 use this phrase for visit-frequency data. | Plan-derived. |
| "a designed presenter, not disabled animation" | PLAN INV-15 uses this exact reduced-motion framing. | Plan-derived. |
| "quiet field" | PLAN INV-11 and §11.4 use this loading/empty state phrase. | Plan-derived. |
| "load-bearing" | PLAN uses the term in greeting and voice sections. | Not rubric-only in this run. |
| "NOT RECOVERABLE FROM PLAN" | This is reconstruction protocol language, not gold taxonomy. | Not contamination. |

I found no free use of suspicious rubric-side phrases such as `intent fidelity`, `feature-level fidelity`, `multi-layer recovery`, or `weight-3` in the reconstruction.

## Heading Mirror Check

`RECONSTRUCTION.md` headings are `## System-level intent`, `## Per-feature whys`, and then `### 0` through `### 18`, matching the PLAN section structure. They do not mirror `GOLD_WHYS.md` headings such as `System-level whys`, `Feature-level whys`, S1-S9, or F1-F40. This pattern is expected for a plan-derived reconstruction.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction is organized by PLAN sections and invariant/product areas, not by the 49 gold targets. It does not enumerate S1-S9 or F1-F40 in order, and it includes many plan-specific items marked `NOT RECOVERABLE FROM PLAN` that are not gold-row mirrors.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| "The organizing conviction is that hard rules should be made structurally unrepresentable wherever possible, not merely documented." | PLAN §0 uses the same conviction and examples: no decrement accumulator, no UserFact type, no mood enum in narration, string import boundary. | Supported. |
| "The plan bans streak, score, level, badge, XP, rank, visit count, avoids visible cooldowns, discards excess drift drive instead of banking it..." | PLAN INV-8 bans counters; §6.3 discards excess drive; §7.3 makes cooldown invisible. | Supported. |
| "Reduced-motion is a designed presenter, not disabled animation, with identical drift, mood, calls, notebook, and narration." | PLAN INV-15 and §10.4 state the same presenter architecture and parity list. | Supported. |

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs leaked, headings mirror the PLAN rather than the gold list, vocabulary is traceable to PLAN language, and the most articulate sentences have direct support in the plan. Scores should be treated as valid for this slot.
