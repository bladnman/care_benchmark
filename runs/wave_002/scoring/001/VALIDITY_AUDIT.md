# VALIDITY_AUDIT - run 001

## Verdict

PASS. The frozen reconstruction reads as derived from `PLAN.md`: it follows the plan's section order, uses plan-local invariant vocabulary, includes rule-only and NOT RECOVERABLE calls, and does not present a neat gold-list mapping.

## 1. ID leakage check

No gold IDs were found in `RECONSTRUCTION.md`: no `S1`-`S9`, no `F1`-`F40`, and no `R-Fxx` style IDs. The only ID-like taxonomy is the plan's `INV-n` vocabulary, which appears in `PLAN.md` and is therefore not leakage.

Search note: `load-bearing` appears once in the reconstruction around bird identity and appears in the plan as plan vocabulary, so it is not a gold-side-only hit.

## 2. Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "principles behave like invariants, not aspirations" | PLAN section 0 uses the same phrasing | plan-derived |
| "first frame already in motion" | PLAN sections 1.1 and 10.4 | plan-derived |
| "no load state", "no canned cue", "no looped audio", "no cycle animation" | PLAN invariant register INV-1 | plan-derived |
| "visible and focused and recent input" | PLAN INV-8 and section 7.2 | plan-derived |
| "per-bird interaction data never leaves the simulation boundary" | PLAN INV-9 and section 11.3 | plan-derived |
| "discrete versus continuous state" | PLAN section 2.2 | plan-derived |
| "too-fast drift is unrecoverable and too-slow drift is a config change" | PLAN section 7.4 and R1 | plan-derived |
| "accessibility is a designed surface, not a fallback" | PLAN sections 10 and 12.6 | plan-derived |

No rubric-side phrases such as `intent fidelity`, `weight-3`, `multi-layer recovery`, or `gold why` appeared in the reconstruction.

## 3. Heading mirror check

The top headings are the expected reconstruction headings: `## System-level intent` and `## Per-feature whys`. Subheadings mirror PLAN sections (`0. How to read this plan`, `1. Scope`, `2. Architecture`, through `16. Team shape and sequencing note`), not GOLD_WHYS sections. This is a plan mirror, not a gold-list mirror.

## 4. 1:1 mapping suspect check

No suspicious 1:1 mapping to the gold targets. The reconstruction has 16 system-intent bullets, then many plan-section bullets in plan order. It does not enumerate S1-S9 or F1-F40, and it includes many non-scored plan details alongside scored ones. This supports plan derivation.

## 5. Plan-derivation spot check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan says the PRD has 'five principles that behave like invariants, not aspirations,' and it converts each refusal into schema grants, lint rules, contract tests, CI gates, and infra-policy checks." | PLAN section 0 invariant register and sections 12.5/13 infra policy | supported |
| "The visitor path is a different route, not a flag." | PLAN section 6.5 uses the same sentence and explains the separate visitor route | supported |
| "The plan states 'too-fast drift is unrecoverable and too-slow drift is a config change,' so alpha ships at 0.6x fitted and is raised only after dogfood signal." | PLAN section 7.4 and decision 12 | supported |

## 6. Verdict rationale

PASS: no gold ID leakage, no rubric vocabulary leakage, no gold-heading mirror, no neat S/F mapping, and spot-checked articulate sentences are directly supported by PLAN passages. The reconstruction's omissions and NOT RECOVERABLE calls also look like honest plan-only reconstruction rather than gold-aware backfilling.
