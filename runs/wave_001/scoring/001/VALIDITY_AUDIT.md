# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found.

I searched the frozen reconstruction for gold-style IDs such as `S1`-`S9`, `F1`-`F40`, and `R-Fxx`. There were no hits. The reconstruction uses PLAN-native guardrail IDs such as `G1` and `G12`, which appear in PLAN and are not gold-list identifiers.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "P1 bug" | PLAN opening: canned greeting/leaked trait/harmless toast is a P1 bug | Plan-derived |
| "acceptance criteria" | PLAN §1: principles and non-goals are acceptance criteria | Plan-derived |
| "the spine" | PLAN §3: guardrails are called the spine | Plan-derived |
| "load-bearing split" | PLAN §4.1: render-pipeline boundary uses this phrase | Plan-derived |
| "honest signal" | PLAN §7.2 heading/body describes the presence detector as honest signal | Plan-derived |
| "affective-perf bridge" | PLAN §13.2 uses this phrase for the 500ms budget | Plan-derived |
| "not measured / NOT computed" | PLAN §13.5 deliberately excludes visit-frequency and relationship metrics | Plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Not PLAN prose; this is a reconstruction workflow marker, not gold/rubric taxonomy | Expected workflow vocabulary, not contamination |

I found no rubric-side terms such as "intent fidelity," "feature-level fidelity," "weight-3," or "multi-layer recovery" in the reconstruction.

## Heading Mirror

| Reconstruction heading | Gold-list mirror concern | Assessment |
|---|---|---|
| `## System-level intent` | Matches the required reconstruction section shape, not a gold title | OK |
| `## Per-feature whys` | Matches the required reconstruction section shape, not a gold title | OK |
| `### Scope, accounts, identity, and lifecycle` | Not a GOLD_WHYS section title | OK |
| `### Architecture, data, and sync` | Not a GOLD_WHYS section title | OK |
| `### Simulation engine and interactions` | Broad PLAN-derived grouping, not a gold mirror | OK |
| `### API and visit flow` | PLAN-derived implementation grouping | OK |
| `### Frontend rendering and audio` | PLAN-derived implementation grouping | OK |
| `### Accessibility and voice` | PLAN-derived implementation grouping | OK |
| `### Performance, observability, privacy, rollout, and testing` | PLAN-derived implementation grouping | OK |

No heading sequence mirrors the S1-S9 or F1-F40 gold list.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not proceed in the gold feature order. It includes 16 system-level bullets and many broad implementation bullets, including non-gold features and explicit `NOT RECOVERABLE FROM PLAN` rows. That shape matches a PLAN-derived reconstruction rather than a neat gold-list mapping.

## Plan-Derivation Spot Check

1. Reconstruction: "The design principles are acceptance criteria."  
   PLAN support: §1 says the PRD principles and non-goals are "not a preamble" but "the acceptance criteria."  
   Result: supported.

2. Reconstruction: "Slow truth belongs to the server; fast presentation belongs to the client."  
   PLAN support: §4.1 names the render-pipeline boundary and assigns slow truth to server, fast presentation to client.  
   Result: supported.

3. Reconstruction: "The slow cadence avoids flooding the SR queue."  
   PLAN support: §11.1 says narration updates every 30-60 seconds at idle and that high-frequency narration would flood the screen-reader queue.  
   Result: supported.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary, no gold-heading mirror, no 1:1 gold-order mapping, and the sampled high-signal sentences are directly grounded in PLAN. The only non-PLAN phrase of note is `NOT RECOVERABLE FROM PLAN`, which is an expected reconstruction workflow marker rather than evidence of gold contamination.
