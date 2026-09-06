# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Mechanical search over the frozen reconstruction for `F#`, `S#`, `R-F#`, and `R-S#` tokens found no hits. The same search over the assigned PLAN found no such gold IDs either. There is no evidence that the reconstruction leaked gold identifiers or copied the scoring taxonomy.

Verdict for this check: PASS.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION.md and plan-derivation status:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "invariants with enforcement, not as tone guidance" | PLAN sec. 0 uses this phrasing nearly verbatim. | Plan-derived. |
| "clients submit events, never values" | PLAN invariant I1 says this directly. | Plan-derived. |
| "The server decides what happens; the client decides how it looks and sounds" | PLAN sec. 2.2 says this directly. | Plan-derived. |
| "visible and focused and recent pointer/key activity" | PLAN sec. 0 I3 and sec. 7.9 define this conjunction. | Plan-derived. |
| "nothing, anywhere, announces" | PLAN sec. 14 ends with this phrase. | Plan-derived. |
| "mail cannon" | PLAN sec. 4.4 uses this phrase for invite abuse. | Plan-derived. |
| "golden vectors" | Appears in PLAN sec. 5.13 and launch gates; this is engineering terminology, not gold-list leakage. | Benign. |
| "NOT RECOVERABLE FROM PLAN" | This marker is not in PLAN, but it is required by the phase-2A reconstructor prompt. | Benign workflow marker. |

No scorer-side phrases such as "multi-layer recovery", "feature-level fidelity", "weight-3", or "intent fidelity" appeared in the reconstruction. The only `gold` substring was "golden vectors", also present in PLAN.

## Heading Mirror Check

RECONSTRUCTION headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture and data model`
- `### API surface`
- `### Simulation engine`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance, observability, rollout, and operations`

The first two headings match the required phase-2A output structure, not GOLD_WHYS section titles. The remaining headings mirror the PLAN's own major sections more closely than the gold list. They do not mirror the S1-S9/F1-F40 taxonomy or the gold grouped-by-file headings in a suspicious way.

## 1:1 Mapping Suspect Check

The reconstruction does not assign S1-S9 or F1-F40 identifiers, does not proceed in gold-list order, and includes many plan-level implementation bullets outside the 40 gold why anchors. It is organized by the PLAN's sections and implementation domains. Some bullets correspond naturally to canonical features because the PLAN itself covers those features, but there is no neat one-item-per-gold-target mapping.

Verdict for this check: PASS.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The render boundary is philosophical as well as architectural: 'The server decides what happens; the client decides how it looks and sounds.'" | PLAN sec. 2.2 states the same boundary before listing canonical, server-computed, client-derived, and device-local layers. | Grounded. |
| "Privacy is a hard product boundary. 'Per-account interaction data never leaves the simulation database'; RUM is 'aggregate-only'; there is no analytics role." | PLAN invariant I7, sec. 2.5 trust boundaries, sec. 10.4 closed RUM schema, and sec. 10.5 deliberate non-measurement all support this. | Grounded. |
| "Age-based arrivals up to seven birds: The ladder 'ramps itself,' gives dogfood and beta aviaries earlier canary counts, and lets audio identification studies gate larger groups before public aviaries reach them." | PLAN sec. 11.4 says the age-based ladder ramps birds per aviary and dogfood/beta cohorts cross thresholds earlier; it also gates larger counts on identification studies. | Grounded. |

## Verdict

PASS. The frozen reconstruction reads as derived from the PLAN: no gold IDs leaked, scorer vocabulary is absent, headings follow the PLAN/workflow rather than the gold taxonomy, and spot-checked articulate sentences have clear PLAN support. The only non-PLAN phrase of note is the mandated `NOT RECOVERABLE FROM PLAN` marker, which is not contamination.
