# VALIDITY AUDIT — CARE run 001

## ID leakage

PASS. Mechanical search of the frozen reconstruction found no gold IDs or external taxonomy tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. The reconstruction uses plan-native numbering such as `### 1. Scope` and feature names from the plan.

## Vocabulary check

No scorer-side vocabulary hits were found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`. Spot checks of plan-derived phrases:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Only the server tick writes personality" | PLAN invariant 1 | Plan-derived |
| "Personality is monotonic toward expressive" | PLAN invariant 2 | Plan-derived |
| "Notice, never announce" | PLAN invariant 6 and §12 | Plan-derived |
| "first frame is the aviary, already in motion" | PLAN invariant 10 | Plan-derived |
| "analytics has no network route and no credentials" | PLAN §11.6 | Plan-derived |
| "a designed surface" for reduced motion | PLAN §7.8 | Plan-derived |
| "does it feel alive?" accessibility testing prompt | PLAN §10.7 | Plan-derived |
| "not a button with a timer" for offer cooldown | PLAN D-11 | Plan-derived |

## Heading mirror

PASS. Reconstruction headings are `## System-level intent`, `## Per-feature whys`, then `### 1. Scope` through `### 17. Decisions made where the plan records ambiguity or tension`. Those headings mirror the candidate PLAN's numbered implementation sections, not GOLD_WHYS headings such as "System-level whys", "Feature-level whys", or F/S IDs.

## 1:1 mapping suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40, and it does not provide a neat 49-item gold-order mapping. Instead, it follows the PLAN structure and includes many plan-only items without gold why anchors, such as magic-link token handling, event outbox, renderer choice, workstreams, and risk mitigations. That shape is consistent with blind plan-derived reconstruction.

## Plan-derivation spot check

1. Reconstruction: "The plan repeatedly protects one source of truth." PLAN support: invariant 1, §6.1 canonical state, and §5.1 tick transaction. Supported.
2. Reconstruction: "A newcomer bird appears in the scene; the notebook may note it; adoption is in settings so the user notices and chooses without being prompted." PLAN support: §5.10 Arrivals and D-10. Supported.
3. Reconstruction: "Hosted LLMs would send per-bird state to a third party ... grammar is testable by lint." PLAN support: §9 opening paragraph. Supported.

## Verdict

PASS. I found no significant contamination signatures: no gold IDs, no rubric vocabulary, no gold-list heading mirror, no 1:1 mapping to the scorer taxonomy, and spot-checked articulate claims are grounded in the PLAN. Scores can be treated as valid under the benchmark's frozen-reconstruction contract.
