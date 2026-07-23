# VALIDITY_AUDIT - run 001

## ID leakage

PASS. A regex scan of the frozen reconstruction found no gold IDs or external IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. The reconstruction does not use the scorer's gold taxonomy.

## Vocabulary check

Sampled phrases and source checks:

| Reconstruction phrase | PLAN support | Finding |
|---|---|---|
| "every load-bearing rule" | PLAN opening uses the same phrase. | Plan-derived. |
| "The product is what's left after these subtractions" | PLAN section 15 uses this sentence. | Plan-derived. |
| "clients render snapshots; the server simulates" | PLAN section 2.2 names this boundary. | Plan-derived. |
| "the absence of divergence" | PLAN section 6 uses this for sync. | Plan-derived. |
| "actual product" for accessibility | PLAN section 9 uses this framing. | Plan-derived. |
| "aggregate-only" telemetry | PLAN section 10.3 uses this boundary. | Plan-derived. |
| "The tick is the product" | PLAN section 5 says this directly. | Plan-derived. |

The word "golden" appears only as caption/narration test terminology inherited from the plan, not as benchmark gold-list language. No rubric terms such as "feature-level fidelity," "system-level fidelity," "weight-3," or "multi-layer recovery" appear.

## Heading mirror

The top headings are `System-level intent` and `Per-feature whys`, which were required by the phase-2A reconstruction prompt. Subheadings under per-feature whys are `Scope`, `Architecture and data model`, `API surface`, `Simulation engine and sync`, `Frontend rendering pipeline and audio`, and `Accessibility, performance, testing, rollout, and risks`. These mirror the PLAN's implementation structure, not GOLD_WHYS section titles or S/F ordering.

## 1:1 mapping suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40 and does not provide exactly 49 neat gold-target rows. It follows the plan's section order, includes many plan-only engineering items, and marks several items `NOT RECOVERABLE FROM PLAN`. This is not a 1:1 gold-list mapping.

## Plan-derivation spot check

1. Reconstruction sentence: "The product is intentionally made by refusal." PLAN support: sections 1.2 and 15 list the refused surfaces and close with the product being what remains after the subtractions.
2. Reconstruction sentence: "The core architecture is 'clients render snapshots; the server simulates.'" PLAN support: section 2.2 states this as the central boundary and sections 5-6 implement it.
3. Reconstruction sentence: "Privacy is structural rather than only policy." PLAN support: sections 1.3, 3, and 10.3 enforce DB-role separation, email isolation, telemetry credential boundaries, and banned metric fields.

All three articulate sentences have direct PLAN support.

## Verdict: PASS

No significant contamination signs were found. The reconstruction reads as a plan-derived artifact: no leaked gold IDs, no scorer-side vocabulary, no gold-list heading mirror, and no neat 1:1 mapping to the held-out why list.
