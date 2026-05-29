# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

No leakage found. A search of the frozen reconstruction for gold identifiers and rubric-side IDs found no S1-S9, F1-F40, R-Fxx, gold-list labels, or scoring taxonomy. The only potentially loaded term hit was "load-bearing," and the same phrase appears in PLAN.md as "load-bearing asymmetry," so it is plan-derived rather than gold-side leakage.

## Vocabulary Check

Sampled reconstruction phrases against PLAN.md:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "felt-aliveness is the product" | PLAN opening uses the exact phrase. | Plan-derived |
| "two registers, one hard line" | PLAN voice note uses the exact phrase. | Plan-derived |
| "not features layered on a CRUD app" | PLAN opening uses the exact phrase. | Plan-derived |
| "The product is what's left after the subtractions" | PLAN summary uses the exact phrase. | Plan-derived |
| "actual product that feels alive" | PLAN §9 uses the exact phrase. | Plan-derived |
| "single biggest lever" | PLAN §2.3 uses it for inlined snapshot. | Plan-derived |
| "load-bearing asymmetry" | PLAN §5.2 uses the exact phrase. | Plan-derived |
| "boring-with-teeth layer" | PLAN rollout uses the exact phrase. | Plan-derived |
| "a spinner says machine" | PLAN §7.4 uses the exact phrase. | Plan-derived |
| "the wrong word can't be typed" | PLAN glossary section uses the exact phrase. | Plan-derived |

No unsupported rubric-side vocabulary such as "intent fidelity," "feature-level fidelity," "weight-3," or "multi-layer recovery" appears in the reconstruction.

## Heading Mirror Check

Reconstruction headings are: System-level intent, Per-feature whys, then broad plan-derived groups: Scope, Interactions, Accounts Sync And Social, Architecture And Domain Model, Simulation Engine, API Surface, Frontend Rendering Pipeline, Audio Pipeline, Accessibility Surfaces, Performance And Observability, and Rollout And Risks.

These do not mirror the gold-list section titles or S/F identifiers. They mirror the candidate PLAN's own implementation organization more than GOLD_WHYS.md. System-level intent and Per-feature whys are expected phase-2A output headings, not leakage.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping detected. The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve gold order, and includes many implementation items that are not gold why anchors. Its per-feature section follows the PLAN's broad build areas and includes plan-only items such as Preact, Go tick worker, Redis, tick_consumed_at, and rollout sequencing. That shape is inconsistent with gold-list contamination.

## Plan-Derivation Spot Check

1. Reconstruction: "The privacy commitment is enforced by network policy and separate credentials, not by reviewer vigilance."
   PLAN support: §2.1 states that the two planes share no data store or ETL path and that the privacy commitment is enforced by network policy and separate credentials.
   Verdict: plan-derived.

2. Reconstruction: "A constantly opaque bar reads as an app frame; fading makes it an ignorable thin layer."
   PLAN support: §7.6 says the top bar fades nearly transparent and that a top bar always at full opacity reads as an app frame.
   Verdict: plan-derived.

3. Reconstruction: "The grammar is deterministic, naturalist, testable, privacy-clean, and voice-safe."
   PLAN support: §5.7 says v1 uses a deterministic template/grammar generator because it is testable, cheap, offline-safe, privacy-clean, and incapable of drifting into announcement voice.
   Verdict: plan-derived.

## Verdict: PASS

The reconstruction shows no significant contamination signatures. It uses no gold IDs, its distinctive vocabulary is traceable to PLAN.md, its headings do not mirror GOLD_WHYS.md, and its structure follows the plan rather than a neat S/F scoring map. The few scoring denials are ordinary plan/reconstruction evidence gaps, not contamination.
