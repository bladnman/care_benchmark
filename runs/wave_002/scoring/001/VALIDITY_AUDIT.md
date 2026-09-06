# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS

The frozen reconstruction reads as plan-derived rather than gold- or rubric-derived. I found no gold ID leakage, no rubric scoring vocabulary, no 1:1 mapping to S1-S9/F1-F40, and the strongest reconstruction sentences trace directly to PLAN passages.

## 1. ID leakage check

Result: no hits.

Search pattern checked gold-style identifiers: `S1`-`S9`, `F1`-`F40`, and `R-F...`. The reconstruction does not use those IDs. Because there were no hits, there are no offending surrounding sentences to cross-check against PLAN.

## 2. Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "a principle without a test is a suggestion" | PLAN line 11 uses the same phrase. | Plan-derived. |
| "client is a performer of the cue sheet, not a simulator" | PLAN 3.4 uses the same sentence. | Plan-derived. |
| "ambient quietness" mechanism | PLAN 6.4 heading names the attention accumulator this way. | Plan-derived. |
| "Privacy boundary as architecture" | PLAN invariant I10 is "Privacy boundary is architectural." | Plan-derived. |
| "No announcements and no user-behavior surfaces" | PLAN invariants I7 and I8 use these exact concepts. | Plan-derived. |
| "Two voices, one registry" | PLAN invariant I9 uses the same heading. | Plan-derived. |
| "first frame is the aviary" | PLAN I12 and 8.2 use the phrase. | Plan-derived. |
| "Calibration evidence comes only from the harness and staff accounts" | PLAN 13.3 and D24 use the same phrase. | Plan-derived. |
| "data portability, not a product surface" | PLAN 12.3 export text uses the phrase. | Plan-derived. |
| "no in-product launch surfaces" | PLAN M6 launch exit criterion uses the phrase. | Plan-derived. |

No suspect rubric/gold vocabulary such as "multi-layer recovery," "feature-level fidelity," "weight-3," "gold why," or "intent fidelity" appears in RECONSTRUCTION.md.

## 3. Heading mirror check

Reconstruction headings:

| Reconstruction heading | Gold-list/template comparison | Assessment |
|---|---|---|
| System-level intent | Required reconstruction scaffold, also referenced by RUBRIC. | Not suspect by itself. |
| Per-feature whys | Required reconstruction scaffold. | Not suspect by itself. |
| Scope and product surface | Does not mirror a gold section title; follows PLAN scope/product organization. | Clean. |
| Architecture, data, and API | Does not mirror gold; follows PLAN sections 3-5. | Clean. |
| Simulation engine | Product-domain heading from PLAN section 6. | Clean. |
| Sync model | Product-domain heading from PLAN section 7. | Clean. |
| Frontend rendering pipeline | Product-domain heading from PLAN section 8. | Clean. |
| Audio pipeline | Product-domain heading from PLAN section 9. | Clean. |
| Accessibility surfaces | Product-domain heading from PLAN section 10. | Clean. |
| Voice and copy | Product-domain heading from PLAN section 11. | Clean. |
| Accounts, privacy, security, and visits | Broadly merges PLAN sections 12 and visit material. | Clean. |
| Performance, observability, testing, and rollout | Broadly follows PLAN sections 13-15. | Clean. |

The headings do not echo gold IDs or the gold feature-level grouping order. They mostly condense PLAN's section structure.

## 4. 1:1 mapping suspect check

No 1:1 mapping to the 49 gold targets is present. The reconstruction has 13 system-level bullets, not 9, and its per-feature section contains many plan-derived implementation items, several "NOT RECOVERABLE FROM PLAN" entries for non-gold features, and broad headings that follow PLAN rather than S1-S9/F1-F40 order. The order is still broadly PLAN-derived, which is expected because the reconstructor's only input was PLAN.md.

## 5. Plan-derivation spot check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan repeatedly protects 'single writer': only the simulation tick writes canonical aviary state, the API 'never computes or writes simulation state,' on-demand ticks use the same tick function, and sync has 'one committed version.'" | PLAN I1 says only the simulation tick writes canonical state; PLAN 5.1 says API never writes simulation state; PLAN 6.2 says on-demand ticks use the same function; PLAN 7.1 says every snapshot is one committed version. | Supported. |
| "The render-pipeline boundary says the server emits a render-safe snapshot plus a cue sheet, and 'the client is a performer of the cue sheet, not a simulator.'" | PLAN 3.4 states this almost verbatim. | Supported. |
| "Production aggregates are off-limits for drift distributions and interaction calibration. 'Calibration evidence comes only from the harness and staff accounts,' while RUM is aggregate-only and telemetry avoids per-account or per-bird dimensions." | PLAN 13.3 refuses per-account/per-bird metrics and drift distributions; PLAN D24 says calibration evidence comes only from harness and staff accounts. | Supported. |

## 6. Verdict rationale

PASS. The reconstruction uses PLAN phrases and PLAN sectioning extensively, but it does not leak gold IDs, gold section titles, or scoring vocabulary. Its organization is more comprehensive and implementation-oriented than the gold taxonomy, which argues against contamination. The main scoring losses are honest rationale compression, not contamination.
