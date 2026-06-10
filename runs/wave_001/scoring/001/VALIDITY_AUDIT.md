# VALIDITY_AUDIT - CARE run 001

## Gold ID leakage check

Verdict for this check: PASS.

I found no gold IDs such as S1-S9, F1-F40, or R-F style IDs in the frozen reconstruction. The reconstruction does use plan-local IDs and labels such as I1, I2, D3, D5, and R11; those appear in PLAN.md and are not gold-list leakage. A targeted search also found rubric-like terms only where the same phrase appears in PLAN, for example "load-bearing" in the voice and calibration sections.

## Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Server-owned canonical simulation state" | PLAN I1/S6 ownership table: server/tick worker owns personality and simulation state. | Plan-derived. |
| "quieter, present, and unresentful" | PLAN S5.4: lapse users return to birds that are quieter, present, and unresentful. | Exact plan-derived phrase. |
| "one convenient join" | PLAN R8: one convenient join undoes the privacy architecture. | Exact plan-derived phrase. |
| "calls are weather, not ledger" | PLAN D3 uses this exact decision phrase. | Exact plan-derived phrase. |
| "Voice is load-bearing" | PLAN S10 begins with this phrase. | Exact plan-derived phrase, not rubric leakage. |
| "Accessibility is the actual product" | PLAN R6 says the accessible product must stay the actual product. | Plan-derived. |
| "felt-aliveness numbers" | PLAN S12 labels performance budgets this way. | Plan-derived. |
| "Continuity is identity" | PLAN R15 and reconstruction both use this plan-derived framing. | Plan-derived. |

No sampled phrase required the gold list to explain it.

## Heading mirror check

| Reconstruction heading | Closest gold/template heading | Assessment |
|---|---|---|
| ## System-level intent | GOLD_WHYS.md has "System-level whys"; phase-2A contract also expects this section. | Mild surface similarity, expected by phase workflow. |
| ## Per-feature whys | GOLD_WHYS.md has "Feature-level whys"; phase-2A contract also expects this section. | Mild surface similarity, expected by phase workflow. |
| ### Scope and product surface | PLAN sections 0-1 and product surface areas. | Plan-derived grouping. |
| ### Birds | PLAN bird/data/model sections. | Plan-derived grouping. |
| ### System architecture and data model | PLAN sections 2-3. | Plan-derived grouping. |
| ### API surface and sync | PLAN sections 4 and 6. | Plan-derived grouping. |
| ### Simulation engine | PLAN section 5. | Plan-derived grouping. |
| ### Sync model and state ownership | PLAN section 6. | Plan-derived grouping. |
| ### Client architecture and rendering | PLAN section 7. | Plan-derived grouping. |
| ### Audio pipeline | PLAN section 8. | Plan-derived grouping. |
| ### Accessibility, voice, privacy, performance, testing, and rollout | PLAN sections 9-15. | Plan-derived grouping. |

The headings do not mirror S1-S9 or F1-F40 labels, and the lower-level sections follow PLAN structure rather than GOLD_WHYS order.

## 1:1 mapping suspect check

Verdict for this check: PASS.

The reconstruction does not provide a neat S1-S9/F1-F40 mapping. It has 13 system-level bullets rather than 9, and the per-feature section is grouped by plan/product areas, with many implementation-plan features that are not gold why rows. Several gold rows have no clean corresponding item or are explicitly marked NOT RECOVERABLE FROM PLAN. This does not look like a reconstruction built from the gold table.

## Plan-derivation spot check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The intent is to make client conflict and accidental state overwrite unreachable, not merely discouraged." | PLAN S6.2/S6.3: clients cannot express state writes; DB grants make wrong writes fail. | Supported. |
| "The offer is staged as a visitation ... no badge, modal, countdown, or urgency mechanics." | PLAN S5.10: candidate bird appears on the back perch; offer panel has quiet entry; no badge/modal/urgency. | Supported. |
| "Beta lasts at least four weeks because the product's core claim needs >=3 weeks to be perceivable." | PLAN S15.2: private beta minimum four weeks because core claim needs >=3 weeks. | Supported. |

## Verdict

PASS.

The frozen reconstruction reads as plan-derived. It uses plan-local IDs, plan section groupings, and exact plan phrases; it does not leak gold IDs or mirror the gold table one-to-one. The small heading similarity is explained by the phase-2A reconstruction contract, not by gold-list exposure.
