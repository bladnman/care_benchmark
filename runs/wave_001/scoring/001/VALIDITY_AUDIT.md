# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as plan-derived. I found no gold ID leakage, no 1:1 gold-list mapping, and no rubric vocabulary used as a scoring taxonomy. A few phrases such as "load-bearing" are high-signal, but they also appear in the PLAN itself.

## 1. ID leakage check

Search target: gold IDs and benchmark-side labels such as `S1`, `F1`, `F40`, `R-F01`, `gold`, `rubric`, `multi-layer`, `feature-level fidelity`, `weight-3`, and `intent fidelity`.

- Result: no gold IDs or benchmark taxonomy labels appear in `RECONSTRUCTION.md`.
- The only notable benchmark-adjacent phrase was "load-bearing", but `PLAN.md` also uses it in section 5.4 and in the time-to-first-bird mitigation, so this is not leakage.

## 2. Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Notice, never announce" | PLAN section 1.3 uses the exact hard-rule heading. | Plan-derived. |
| "load-bearing product philosophy" | PLAN uses "load-bearing" for asymmetry and bootstrap path. | Plan-derived vocabulary, not gold-only. |
| "simulation service is the spine" | PLAN section 2.1 says the simulation service is the spine. | Plan-derived. |
| "privacy and authority boundary" | PLAN section 2.1 uses this exact service-topology rationale. | Plan-derived. |
| "designed surface, not a stripped fallback" | PLAN section 1.3 says reduced-motion is a designed surface, not stripped fallback. | Plan-derived. |
| "produced fresh every time" | PLAN section 7.1 uses this exact procedural-call rationale. | Plan-derived. |
| "observations that matter, not a feed" | PLAN section 5.9 uses this exact notebook rule. | Plan-derived. |
| "the aviary continues without the viewer" | PLAN section 6.9 states this on visible resume. | Plan-derived. |
| "not building microservices for taste" | PLAN section 2.1 uses the exact phrase. | Plan-derived. |
| "No account import in v1" | PLAN section 12.7 states this and explains why. | Plan-derived. |

I did not find rubric-only phrases such as "intent fidelity", "feature-level fidelity", "multi-layer recovery", or "weight-3" in the reconstruction.

## 3. Heading mirror check

`RECONSTRUCTION.md` headings:

| Reconstruction heading | Gold-list heading comparison | Assessment |
|---|---|---|
| `## System-level intent` | Near the expected reconstruction section, not a gold title. | Not suspicious. |
| `## Per-feature whys` | Near the expected reconstruction section, not a gold item list. | Not suspicious. |
| `### Scope and account foundation` | Does not mirror GOLD file group names. | Plan-structure-derived. |
| `### Architecture, data model, and API` | Combines plan sections; not a gold heading. | Plan-structure-derived. |
| `### Simulation engine` | Mirrors PLAN section 5. | Plan-derived. |
| `### Frontend rendering pipeline` | Mirrors PLAN section 6. | Plan-derived. |
| `### Audio pipeline` | Mirrors PLAN section 7. | Plan-derived. |
| `### Accessibility surfaces` | Mirrors PLAN section 8. | Plan-derived. |
| `### Performance, observability, and calibration` | Mirrors PLAN sections 9-10. | Plan-derived. |
| `### Architectural invariants, sync, and rollout` | Mirrors PLAN sections 11-13. | Plan-derived. |

The headings follow the PLAN's implementation sections, not `GOLD_WHYS.md` section titles or F1-F40 order.

## 4. 1:1 mapping suspect check

The reconstruction does not create a neat S1-S9 or F1-F40 list. It has 14 system bullets and many plan-ordered feature bullets, including several items that are not gold why anchors (for example render pipeline boundary, API versioning, species pool configuration, event push batching, and iOS Safari audio unlock). Some gold-bearing features are merged into broader plan clusters and some are explicitly marked `NOT RECOVERABLE FROM PLAN`. This is not a 1:1 gold-list mapping.

## 5. Plan-derivation spot check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "We are not building microservices for taste; the service split draws a privacy and authority boundary between simulation state and telemetry." | PLAN section 2.1: same phrase and same privacy/authority-boundary rationale. | Plan-derived. |
| "If frame budget burns out, idle micro-motion drops to 30fps so birds preen more slowly rather than the scene stuttering." | PLAN section 6.4 states sustained overruns halve idle micro-motion and the user sees slower preening rather than stutter. | Plan-derived. |
| "The aviary does not show an enable-audio prompt; audio simply joins in after the first gesture." | PLAN section 7.10 states there is no prompt and audio joins once the user engages. | Plan-derived. |

## 6. Verdict

**PASS.** The reconstruction is articulate, but its vocabulary, headings, ordering, and strongest sentences trace back to the PLAN. There are no gold IDs, no rubric/scoring taxonomy, and no near-1:1 mapping to S1-S9/F1-F40. Treat this run's scores as valid, subject to normal scoring subjectivity.
