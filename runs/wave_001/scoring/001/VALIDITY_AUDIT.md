# VALIDITY_AUDIT - CARE run 001

## ID Leakage

PASS. Mechanical search found no gold IDs or external score-taxonomy IDs in the frozen reconstruction. There were no matches for tokens shaped like `F1`, `S1`, `R-F01`, or `R-S01` in either the reconstruction or plan.

## Vocabulary Check

PASS. I checked reconstruction phrases against the plan and did not find scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`. Sampled plan-derived phrases include:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "Treat product scope as release invariants" | PLAN §1: "Treat these as release invariants" | plan-derived |
| "server the canonical source of truth" | PLAN §1/§2: server is sole canonical state writer | plan-derived |
| "synthetic UUID" | PLAN §§2,3,11 | plan-derived |
| "specific lowercase naturalist prose" | PLAN §1 | plan-derived |
| "reduced motion remains a designed aviary" | PLAN §1 | plan-derived |
| "small measurable vector movement" | PLAN §5 | plan-derived |
| "per-invite read-only visits" | PLAN §1/§9 | plan-derived |
| "primary drift input, not click count" | PLAN §6 | plan-derived |
| "first-bird-visible under 500ms" | PLAN §7 | plan-derived |
| "not invitations to add engagement features" | PLAN §10 | plan-derived |

## Heading Mirror

PASS. The reconstruction headings are `## System-level intent`, `## Per-feature whys`, and numbered `### 1` through `### 11` sections that mirror the assigned PLAN structure. They do not mirror GOLD_WHYS section names or the S1-S9/F1-F40 taxonomy.

## 1:1 Mapping Suspect

PASS. The reconstruction does not provide neat S1-S9 and F1-F40 entries in gold-list order. It follows the plan's 11 sections and many implementation bullets, including items outside the 49 scored whys. This pattern is plan-derived rather than gold-list-derived.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Judgment |
|---|---|---|
| "Make the server the canonical source of truth." | PLAN §1 says the server is the sole canonical state writer, and §2 says the tick worker is the only writer for canonical simulation outputs. | grounded |
| "Presence accounting: the plan makes presence 'the primary drift input, not click count,' and requires visible, focused, recently active intervals so an open or background tab does not count." | PLAN §6 states those three signals and says presence is the primary drift input, not click count. | grounded |
| "Treat accessibility as part of the core aviary, not a fallback." | PLAN §§1,8,10,11 require accessibility paths at launch, reduced motion as a designed aviary, and accessibility acceptance gates. | grounded |

## Verdict

PASS. I found no significant contamination signs. The reconstruction is highly aligned to the PLAN's structure and vocabulary, contains no leaked gold IDs or rubric vocabulary, and its articulate statements are supported by plan passages.
