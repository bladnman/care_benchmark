# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS. Mechanical search of the frozen reconstruction found no gold identifiers or runner IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. It also found no scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, or `intent fidelity`.

No offending IDs were present, so no PLAN cross-reference leakage hits were needed.

## Vocabulary Check

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `one continuing aviary` | PLAN section 1: `one continuing aviary per signed-in account` | Plan-derived |
| `recognizable birds already in motion` | PLAN section 1 and first-navigation section use the same phrase/concept | Plan-derived |
| `Leaving causes no penalty` | PLAN section 1 exact phrase | Plan-derived |
| `server-authored reaction plan` | PLAN decision table and reaction-plan section use this phrase | Plan-derived |
| `specific and alive` | PLAN definition of completion: product remains `specific and alive` in accessible modes | Plan-derived |
| `service health only` | PLAN rollout/observability section: dashboards are service health only | Plan-derived |
| `not a neutral loading pose` | PLAN first-navigation section exact phrase | Plan-derived |
| `no toast/badge framework` | PLAN risk table: no toast/badge framework on product events | Plan-derived |

No sampled phrase reads like held-out rubric language. The word `load-bearing` did not appear in the reconstruction.

## Heading Mirror

The reconstruction headings mirror the PLAN's own implementation sections rather than the gold-list taxonomy:

| Reconstruction heading | Closest PLAN heading | Gold-list mirror concern |
|---|---|---|
| `## System-level intent` | Required phase-2A output format | Not a gold title |
| `## Per-feature whys` | Required phase-2A output format | Not a gold title |
| `### Product contract and scope` | `## 1. Product contract and scope` | Plan mirror |
| `### Decisions where the PRD needs interpretation` | Same PLAN subheading | Plan mirror |
| `### Architecture and responsibilities` | `## 2. Architecture and responsibilities` | Plan mirror |
| `### Data model and invariants` | `## 3. Data model and invariants` | Plan mirror |
| `### API and authorization contracts` | `## 4. API and authorization contracts` | Plan mirror |
| `### Canonical simulation and immediate responses` | `## 5. Canonical simulation and immediate responses` | Plan mirror |
| `### Presence, session behavior, and interactions` | `## 6. Presence, session behavior, and interactions` | Plan mirror |
| `### Snapshot synchronization and failure handling` | `## 7. Snapshot synchronization and failure handling` | Plan mirror |
| `### Frontend scene and rendering pipeline` | `## 8. Frontend scene and rendering pipeline` | Plan mirror |
| `### Audio pipeline and call captions` | `## 9. Audio pipeline and call captions` | Plan mirror |
| `### Accessibility and product voice` | `## 10. Accessibility and product voice` | Plan mirror |
| `### Field notebook generation` | `## 11. Field notebook generation` | Plan mirror |
| `### Account lifecycle, visits, and privacy operations` | `## 12. Account lifecycle, visits, and privacy operations` | Plan mirror |
| `### Performance budgets and observability` | `## 13. Performance budgets and observability` | Plan mirror |
| `### Verification and acceptance suite` | `## 14. Verification and acceptance suite` | Plan mirror |
| `### Delivery sequence and rollout` | `## 15. Delivery sequence and rollout` | Plan mirror |
| `### Risks, mitigations, and launch blockers` | `## 16. Risks, mitigations, and launch blockers` | Plan mirror |
| `### Definition of v1 completion` | `## 17. Definition of v1 completion` | Plan mirror |

No heading mirrors S1-S9 or F1-F40 gold titles.

## 1:1 Mapping Suspect Check

Verdict: PASS. The reconstruction does not provide a neat S1-S9 then F1-F40 sequence. Its system section contains 14 plan-derived principles, and the per-feature section follows the PLAN's implementation sections with many more than 40 bullets. This is the opposite of a held-out gold-list-shaped reconstruction and is explainable by the PLAN's own section structure.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| `Make the product feel alive immediately, not loaded or restarted.` | PLAN section 1 says the first frame contains recognizable birds already in motion; first-navigation section rejects neutral loading pose, spinner, skeleton, and rerun fly-ins. | Supported |
| `The plan uses this to keep visits read-only and non-influential.` | PLAN visits section removes owner controls, return commands, and presence controller; visit end cannot submit simulation events. | Supported |
| `Operational telemetry restricted to performance, availability, and anonymous aggregate health: The rationale is a privacy boundary.` | PLAN section 1 states telemetry is restricted to performance, availability, and anonymous aggregate health; privacy section says metrics cannot include per-bird or per-account relationship data. | Supported |

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no scorer vocabulary, headings mirror the PLAN rather than the gold list, and spot-checked articulate sentences have direct PLAN support. The run's scores do not need contamination discounting.
