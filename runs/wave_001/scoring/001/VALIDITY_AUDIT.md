# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no leakage found.

I searched the frozen reconstruction for gold/rubric identifiers and related labels (`S1`-`S9`, `F1`-`F40`, `R-Fxx`, `weight-2`, `weight-3`, `multi-layer`, `intent fidelity`, `feature-level fidelity`, `load-bearing`). There were no hits. The PLAN search for the same terms also returned no hits, so there is no ID leakage to cross-reference.

## Vocabulary Check

Sampled reconstruction phrases and PLAN grounding:

| Reconstruction phrase | PLAN grounding | Assessment |
|---|---|---|
| "Observational, not custodial or gamified" | PLAN conclusion: "observational, not custodial" and out-of-scope gamification | Plan-derived |
| "Quiet, not announced" | PLAN conclusion: "noticed rather than announced at"; quiet notification service | Plan-derived |
| "A place that continues without the viewer" | PLAN success criteria: "feel the aviary continues without them" | Plan-derived |
| "Attention changes the relationship over weeks" | PLAN drift and success criteria: personality drift over weeks | Plan-derived |
| "Continuity is protected by server authority" | PLAN server-is-sole-writer and sync sections | Plan-derived |
| "Birds must remain individually knowable" | PLAN recognizability and 7-bird cap language | Plan-derived |
| "Accessibility is the same product, not a degraded fallback" | PLAN reduced-motion "designed surface, not fallback" and same affective experience | Plan-derived |
| "Telemetry should not reconstruct the user's relationship" | PLAN "Any telemetry that could reconstruct user's relationship" | Plan-derived |

No sampled phrase used rubric-side vocabulary such as "multi-layer recovery," "feature-level fidelity," "weight-3," "load-bearing," or "intent fidelity." The phrase "NOT RECOVERABLE FROM PLAN" is expected phase-2A output format, not gold leakage.

## Heading Mirror Check

Reconstruction headings:

| Heading | Comparison to gold headings | Assessment |
|---|---|---|
| `## System-level intent` | Matches phase-2A required structure, not specific S1-S9 titles | OK |
| `## Per-feature whys` | Matches phase-2A required structure, not specific F1-F40 titles | OK |
| `### Scope` | Mirrors PLAN section, not gold grouping | OK |
| `### Architecture` | Mirrors PLAN section | OK |
| `### Data Model and API Surface` | Condenses PLAN sections 3 and 4 | OK |
| `### Simulation Engine Design` | Mirrors PLAN section | OK |
| `### Sync Model and Rendering Pipeline` | Condenses PLAN sections 6 and 7 | OK |
| `### Audio Pipeline` | Mirrors PLAN section | OK |
| `### Accessibility Surfaces` | Mirrors PLAN section | OK |
| `### Performance Budgets and Observability` | Mirrors PLAN section | OK |
| `### Rollout` | Mirrors PLAN section | OK |
| `### Risks, Success Criteria, and Design System` | Condenses PLAN sections 12, 14, and 16 | OK |
| `### Open Questions` | Mirrors PLAN section | OK |

The headings are PLAN-shaped, not gold-list-shaped.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not present S1-S9 or F1-F40 in order. It uses 10 system bullets and then PLAN-section groupings with many more and many fewer items than the gold sections. Several gold targets are merged, omitted, or marked `NOT RECOVERABLE FROM PLAN`, and non-gold implementation details such as React/Preact, worker threads, service worker caching, browser support, and open questions appear. That shape is consistent with a reconstruction derived from PLAN, not a neat mapping to the gold list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan ties this to server-side tick, persistent mood, initial load with birds 'mid-action,' ambient motion, and the success criterion that users 'feel the aviary continues without them.'" | PLAN sections 5, 7, and 14 contain server-side tick, mood persistence, first frame with birds mid-action, ambient motion, and the quoted success criterion. | Supported |
| "Reduced-motion mode is 'not animations off' but 'its own designed surface.'" | PLAN sections 7, 9, and 12 explicitly use "Not 'animations off'" and "Designed surface, not fallback." | Supported |
| "Aggregate-only real user monitoring measures performance and errors without a per-account dimension or relationship-reconstructing telemetry." | PLAN section 10 lists aggregate RUM and "What We Don't Measure," including per-account interaction history and relationship-reconstructing telemetry. | Supported |

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary, no gold-order mapping, PLAN-shaped headings, and the sampled articulate sentences are grounded in PLAN text. The run's scores do not need a contamination flag.
