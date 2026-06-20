# VALIDITY_AUDIT - CARE run 001

## ID leakage

PASS. Mechanical search found no gold IDs or reconstruction/taxonomy IDs in `RECONSTRUCTION.md`: no `F1`-style, `S1`-style, `R-F` or `R-S` tokens appeared. The reconstruction uses PLAN-facing names such as "Server-Side Tick", "Sync Model", and "Reduced-Motion Mode" rather than gold-list identifiers.

## Vocabulary check

Sampled phrases all trace to the PLAN rather than the scorer rubric or gold list:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "no achievements, streaks, scores" | PLAN.md:17 | Direct PLAN phrase. |
| "feels alive over weeks" | PLAN.md:278 | Direct PLAN phrase. |
| "Server owns" personality state | PLAN.md:32-33 | Direct PLAN wording. |
| "first-class designed surface" | PLAN.md:41 | Direct PLAN phrase. |
| "Running naturalist prose" | PLAN.md:188-192 | Direct PLAN phrase. |
| "Aggregate-only Real User Monitoring" | PLAN.md:232-237 | Direct PLAN phrase. |
| "No marketing push or social promotion" | PLAN.md:252 | Direct PLAN phrase. |
| "Bird recognizability maintained up to 7 birds in chorus" | PLAN.md:240 | Direct PLAN phrase. |
| "Additive server-authored deltas only" | PLAN.md:119 | Direct PLAN phrase. |
| "No hard channel switching" | PLAN.md:148 | Direct PLAN phrase. |

Mechanical vocabulary search found no rubric-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, or `intent fidelity`.

## Heading mirror

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope / What's in v1`
- `### Architecture`
- `### Data Model and Relationships`
- `### API Surface and Access Control`
- `### Simulation Engine Design`
- `### Sync Model`
- `### Frontend Rendering Pipeline`
- `### Audio Pipeline`
- `### Accessibility Surfaces`
- `### Performance Budgets and Observability`
- `### Rollout Strategy`

The first two headings are required by the phase-2A prompt. The remaining headings mirror PLAN sections, not the gold-list groupings. They do not mirror the gold S1-S9 or F1-F40 structure.

## 1:1 mapping suspect

PASS. The reconstruction does not produce a neat 49-item S/F mapping in gold order. It follows the PLAN's own section order and includes many PLAN-specific items outside the gold why list, including APIs, service workers, bundle budgets, keyboard navigation, synthetic performance tests, and rollout metrics. Several gold targets are not individually reconstructed and are instead marked `NOT RECOVERABLE FROM PLAN`, which is consistent with plan-derived reconstruction rather than contamination.

## Plan-derivation spot check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "Make the aviary feel alive over weeks, not session by session." | PLAN.md:276-279 names drift too slow/too fast and the "feels alive over weeks" promise; PLAN.md:93-98 gives slow drift calibration. | Plan-derived. |
| "Treat accessibility as a designed surface, not a fallback." | PLAN.md:41 calls reduced motion first-class; PLAN.md:291-294 says accessibility included from day one and designed surfaces instead of fallbacks. | Plan-derived. |
| "Keep telemetry aggregate-only and privacy-bounded." | PLAN.md:232-237 says aggregate-only RUM, anonymized histograms, and no per-account interaction history; PLAN.md:296-299 calls per-bird leakage a trust violation. | Plan-derived. |

## Verdict: PASS

No significant contamination signs were found. The reconstruction reads as PLAN-derived: it uses PLAN headings and wording, contains no gold IDs or rubric vocabulary, does not map one-to-one to the held-out gold list, and its articulate claims can be traced to visible PLAN passages.
