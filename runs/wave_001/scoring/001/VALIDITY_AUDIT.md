# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS.

Mechanical check found no gold IDs or external taxonomy tokens in either the frozen reconstruction or the assigned plan for patterns such as `F1`, `S1`, `R-F01`, or `R-S01`. The reconstruction does not appear to expose scorer-side identifiers.

## Vocabulary Check

Verdict: PASS.

No hits were found for scorer-side vocabulary: `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | Plan-derived support |
|---|---|
| "continuity" | PLAN closing criterion: continuity with same named birds and no score/notification loop. |
| "Server-authored canonical life" | PLAN section 2: only the server writes canonical identity, vectors, mood, weather, and notebook records. |
| "One state, many surfaces" | PLAN section 2: visible call, caption, and synthesized call describe the same event. |
| "quiet watching" | PLAN section 5: quiet watching counts; pointer/key activity is freshness, not click frequency. |
| "Privacy by design" | PLAN sections 1, 9, 10, 12: visit log as privacy surface, aggregate metrics, analytics boundary. |
| "Accessibility as a full scene" | PLAN sections 8 and 12: normal renderer, narration grammar, reduced motion, and status-feed avoidance. |
| "quiet field" | PLAN section 2: cold or delayed snapshot shows a quiet field without spinner. |
| "no resource meters" | PLAN section 6: still pool and song fragment are brief events, not resource meters. |

These are mostly concise rephrasings of plan language, not unexplained gold-list vocabulary.

## Heading Mirror

Verdict: PASS.

The reconstruction headings mirror the PLAN's implementation sections rather than the held-out gold taxonomy. Top-level headings are exactly the phase-2A required structure: `System-level intent` and `Per-feature whys`. The `###` headings match plan sections such as `Product boundary and decisions`, `System shape and ownership`, `Simulation design`, and `Accessibility as a full scene`.

No headings mirror the gold-list organization `System-level whys`, `Feature-level whys`, `Complete features list`, or the S/F why titles.

## 1:1 Mapping Suspect

Verdict: PASS.

The reconstruction does not provide a neat S1-S9 or F1-F40 list. It has 12 system-level bullets and then a long plan-section-ordered reconstruction. The per-feature ordering follows the PLAN structure, not the gold order: it includes many non-gold features and also marks several plan-derived items as `NOT RECOVERABLE FROM PLAN`. This is not a suspicious one-to-one mapping to the scorer target list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan's governing acceptance criterion is continuity: the same named birds persist and continue evolving on the server without turning the relationship into a score or a notification loop." | PLAN final paragraph: governing acceptance criterion is continuity; same named birds persist; no score or notification loop. | Plan-derived. |
| "The server owns canonical bird identity, hidden personality vector, mood, perch intent, current weather, and notebook records, while the browser receives a compact render projection." | PLAN section 2 states exactly this ownership split and projection boundary. | Plan-derived. |
| "The plan says to ship narration grammar and reduced-motion renderer with the normal renderer and to test whether alternate surfaces feel like a continuing aviary rather than a status feed." | PLAN principal risk response for accessible experience: ship narration grammar/reduced-motion with normal renderer and test that alternates feel continuing, not status feed. | Plan-derived. |

## Verdict

PASS.

The frozen reconstruction reads as plan-derived. It contains no gold ID leakage, no scorer/rubric vocabulary, no heading mirror of held-out gold sections, and no suspicious S/F one-to-one mapping. The strongest phrasing in the reconstruction can be traced back to plan passages.
