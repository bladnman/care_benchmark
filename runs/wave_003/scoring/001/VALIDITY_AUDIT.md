# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found.

Mechanical search of the frozen reconstruction found no gold/scorer IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. Because there were no hits, no plan cross-check was needed for offending IDs.

## Vocabulary Check

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, or `load-bearing`.

Sampled reconstruction phrases and plan derivation:

| Reconstruction phrase | Appears exactly in PLAN? | Assessment |
|---|---|---|
| `Canonical server-authored life` | no | Synthesized heading, but grounded in PLAN phrases `server is the only writer` and `canonical state`. |
| `Calm care without game pressure` | no | Synthesized, but grounded in non-goals and monotonic/no-punishment drift. |
| `Naturalist voice, not dashboard voice` | no | Synthesized from naturalist notebook/narration and matter-of-fact system surfaces. |
| `Designed accessibility as part of the feature, not a pass afterward` | no | Closely grounded in PLAN rollout: accessibility ships with described features, not later. |
| `Privacy and restraint at the schema boundary` | no | Synthesized from synthetic IDs, encrypted email, aggregate-only telemetry, and analytics separation. |
| `Performance-first living scene` | no | Synthesized from first-bird budget, inlined snapshot, mid-motion boot, and no spinner. |
| `Recognizability over scale` | no | Synthesized from seven-bird cap and chorus recognizability degradation checks. |
| `NOT RECOVERABLE FROM PLAN` | no | Expected phase-2A protocol phrase, not scorer/gold vocabulary. |

The reconstruction uses some polished abstractions, but they are derived from plan content rather than scorer taxonomy.

## Heading Mirror

Reconstruction headings:

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
- `### 12. Risks and mitigations`

The `##` headings are exactly the required phase-2A output headings. The `###` headings mirror the assigned PLAN structure, not `GOLD_WHYS.md` section titles or S/F taxonomy. No contamination signal here.

## 1:1 Mapping Suspect

Verdict: not suspect.

The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve gold order, and does not create one neat item per gold why. It has seven system-level bullets and then a plan-section-based reconstruction. Several gold-bearing features are marked `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction rather than gold-list matching.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| `The render pipeline boundary is sharp: client input -> events -> server tick -> snapshot -> client interpolation.` | PLAN section 2 has the same boundary phrase. | Directly plan-derived. |
| `Presence-time is the dominant drift signal, and the presence-signal risk says a laxer definition like tab-open would silently inflate drift population-wide.` | PLAN sections 5 and 12 state presence-time is dominant and tab-open inflation is a risk. | Directly plan-derived. |
| `Accessibility surfaces ship with the features they describe -- not a later pass.` | PLAN section 11 uses this phrasing in rollout. | Directly plan-derived. |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived: it mirrors the PLAN's own section structure, contains no gold IDs, avoids scorer/rubric vocabulary, and includes honest `NOT RECOVERABLE FROM PLAN` markers instead of filling gaps from the gold list.
