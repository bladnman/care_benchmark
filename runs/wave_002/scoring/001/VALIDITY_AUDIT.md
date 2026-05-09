# VALIDITY_AUDIT - run 001

## ID Leakage

Verdict: no leakage found.

I searched the frozen reconstruction for gold IDs and taxonomy markers (`S1`, `F1`, `F40`, `R-F`, `gold`, `rubric`, `feature-level fidelity`, `weight-3`). The reconstruction does not use gold IDs or the scoring taxonomy. It uses `NOT RECOVERABLE FROM PLAN`, but that is an expected phase-2A recovery marker and not a gold-list ID. The phrase `load-bearing` appears in both PLAN and RECONSTRUCTION, so it is plan-derived rather than leakage.

## Vocabulary Check

Sampled phrases and PLAN support:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "Server is the only writer of personality state" | PLAN §0 rule 1 uses the same phrase. | pass |
| "Aliveness is the product" | PLAN §0 rule 2 uses the same phrase. | pass |
| "Privacy is architectural, not policy" | PLAN §0 rule 3 uses the same phrase. | pass |
| "what's left after a long list of refusals" | PLAN §14 uses this phrase. | pass |
| "aviary age only, never interaction-gated" | PLAN §1.3/§5.8 state this. | pass |
| "actual product, not a stripped variant" | PLAN §9 uses this phrase. | pass |
| "spinner-then-fade" | PLAN §0/§7.8/§12.7 mention this refusal. | pass |
| "NOT RECOVERABLE FROM PLAN" | Not in PLAN; it is a reconstruction-status marker, not content vocabulary. | acceptable marker |

No suspicious rubric-only vocabulary such as `multi-layer recovery`, `feature-level fidelity`, or `weight-3` appears.

## Heading Mirror

Reconstruction headings:

| Reconstruction heading | Gold/template comparison | Result |
|---|---|---|
| `## System-level intent` | Similar to the required phase-2A section, not a gold title mirror. | pass |
| `## Per-feature whys` | Similar to the required phase-2A section, not a gold ID list. | pass |
| `### 1. Scope` through `### 12. Risks` | Mirrors PLAN section order, not GOLD_WHYS S/F order. | pass |

The reconstruction does not mirror gold-list section titles such as individual S1-S9 or F1-F40 entries.

## 1:1 Mapping Suspect

Verdict: not suspect. The reconstruction has many more plan-derived bullets than the 49 scored gold whys and follows the PLAN's section order: Scope, Architecture, Data model, API surface, Simulation engine, Sync model, Frontend, Audio, Accessibility, Performance, Rollout, Risks. It does not create neat S1-S9/F1-F40 items or preserve the gold ordering.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "The plan says personality vectors are 'never set by clients,' client events are append-only, the tick 'consumes them in order,' and there is 'no last-write-wins on personality, ever.'" | PLAN §0 rule 1 and §5.1 tick transaction. | pass |
| "The cross-fade rendering uses the same pose-keyframes the standard renderer uses... rather than an 'animations off' stub." | PLAN §7.6 reduced-motion mode. | pass |
| "The schema must not be able to answer 'how many days in a row has this user visited,' because the existence of the answer is a foothold for surfacing it." | PLAN §3.11 says this nearly verbatim. | pass |

## Verdict: PASS

The reconstruction reads as plan-derived. It uses no gold IDs, no gold-order mapping, and no rubric-side scoring vocabulary beyond the expected recoverability marker. Its headings and most articulate claims track the PLAN structure and wording closely.
