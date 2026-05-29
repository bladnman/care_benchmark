# VALIDITY_AUDIT — CARE run 001

## ID Leakage

PASS. I found no gold IDs or rubric IDs in the frozen reconstruction: no `S1`-`S9`, `F1`-`F40`, `R-F`, `weight-3`, `multi-layer`, `intent fidelity`, or `feature-level fidelity` references. The only search hit that looked ID-like was ordinary prose in "Soft-then-hard deletion", not a gold identifier.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Aliveness is the product" | PLAN introduction uses the exact organizing belief. | Plan-derived. |
| "Load-bearing rules should be enforced mechanically, not by vigilance" | PLAN §0: "Vigilance does not survive a team; tests and types do." | Plan-derived. |
| "Notice, never announce" | PLAN invariant #2 uses the exact phrase. | Plan-derived. |
| "the actual product" for accessibility | PLAN §9: users get "the actual product". | Plan-derived. |
| "server-authored canonical state" | PLAN §0/§6 repeatedly says server is sole writer and canonical source. | Plan-derived. |
| "same product voice" | PLAN §5.7 says the shared prose engine gives users the same product voice. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | Not in PLAN; appears to be phase-2A reconstruction convention rather than gold-side vocabulary. | Not contamination by itself. |

No suspicious gold/rubric vocabulary appears beyond the allowed reconstruction convention.

## Heading Mirror

The reconstruction headings mirror the PLAN structure: `Scope`, `Architecture`, `Data model`, `API surface`, `Simulation engine`, `Sync model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Performance budgets and observability`, and `Guardrails, rollout, and risks`. They do not mirror GOLD_WHYS section titles such as system IDs, feature IDs, or targeted headroom groupings.

## 1:1 Mapping Suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40 in order. It walks the plan's sections and includes many more plan-derived bullets than the gold list. There is no neat one-item-per-gold-target mapping.

## Plan-Derivation Spot Check

1. Reconstruction: "Continuous tick for recently active aviaries plus deterministic catch-up for dormant ones keeps 'continues without the viewer' true without ticking millions of idle aviaries every minute."
   PLAN support: §5.1 states the same active/dormant tick split and the cost rationale.

2. Reconstruction: "Quantized behavior directives in the snapshot give renderer/audio enough to express birds while keeping raw personality floats off the wire."
   PLAN support: §4.1 snapshot schema and note under `behavior` make exactly this boundary.

3. Reconstruction: "Reduced-motion mode is a designed surface; a reduced-motion user gets a calmer Pocket Aviary, not broken-looking animations-off."
   PLAN support: §9 says reduced motion is not animations-off and describes the calmer designed rendering.

All three articulate sentences are grounded directly in PLAN.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no gold ID leakage, no heading mirror of the gold list, no 1:1 gold mapping, and its strongest phrases are traceable to the plan. The few `NOT RECOVERABLE FROM PLAN` markers are consistent with a reconstruction protocol rather than evidence of gold contamination.
