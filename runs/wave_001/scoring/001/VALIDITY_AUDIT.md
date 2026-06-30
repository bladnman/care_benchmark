# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no leakage found. A mechanical search of the frozen reconstruction found no gold IDs or rebuild IDs such as `F1`, `F40`, `S1`, `S9`, `R-F01`, or `R-S01`. The reconstruction references only plan-facing section numbers and product terms.

## Vocabulary Check

Sampled vocabulary against the assigned plan:

| Reconstruction phrase | Present in PLAN? | Assessment |
|---|---|---|
| "load-bearing rules" | yes, PLAN §2.1 uses the same phrase | Not suspicious |
| "hard ownership boundaries" | yes, PLAN §2.1 | Plan-derived |
| "thin renderer and an event emitter" | yes, PLAN §2.2 | Plan-derived |
| "by construction" | yes, PLAN §§2.1/4/12 | Plan-derived |
| "quiet field" | yes, PLAN §7.3 | Plan-derived |
| "Tamagotchi-by-accident" | yes, PLAN §12 | Plan-derived |
| "aggregate-only" | yes, PLAN §§1/10/13 | Plan-derived |
| "structural defense" | yes, PLAN §12 | Plan-derived |
| "feature-level fidelity" / "multi-layer recovery" / "weight-3" | no hits in reconstruction | No scorer-side vocabulary |

The only rubric-adjacent phrase, "load-bearing," is explicitly present in the plan and therefore is not a leakage signal.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Scope`
- `### 2. Architecture`
- `### 3. Data model`
- `### 4. API surface`
- `### 5. Simulation engine design`
- `### 6. Sync model`
- `### 7. Frontend rendering pipeline`
- `### 8. Audio pipeline`
- `### 9. Accessibility surfaces`
- `### 10. Performance budgets and observability`
- `### 11. Rollout`
- `### 12. Risks`
- `### 13. Privacy and telemetry boundary`

The first two headings match the required phase-2A output structure. The numbered headings mirror the PLAN structure, not the gold list. They do not mirror `GOLD_WHYS.md` sections such as System-level whys, Feature-level whys, or the F1-F40 taxonomy.

## 1:1 Mapping Suspect Check

Verdict: not suspicious. The reconstruction does not create neat S1-S9 or F1-F40 rows, does not use gold identifiers, and does not follow the gold ordering. It creates 10 system-level principles and then walks the implementation plan section-by-section. That is consistent with plan derivation.

## Plan-Derivation Spot Check

1. Reconstruction: "Treat the server as the canonical state owner and the client as a thin renderer and an event emitter."
   Plan support: PLAN §2.2 says the client is "a thin renderer and an event emitter," and PLAN §6 says only the tick writes personality and mood.

2. Reconstruction: "Make accessibility a v1 product surface, not a late repair."
   Plan support: PLAN §11.1 says accessibility and core engine are not phased apart, and PLAN §12 says future additions have two render paths and narration/caption obligations.

3. Reconstruction: "Keep privacy and telemetry aggregate-only."
   Plan support: PLAN §13 says per-bird interaction events exist only for the account simulation and the analytics warehouse never reads the simulation database.

All three sampled sentences are directly supported by the plan.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived: it uses the plan's section structure, vocabulary, and implementation emphasis; it does not contain gold IDs, scorer-side taxonomy, or a 1:1 mapping to the held-out gold list.
