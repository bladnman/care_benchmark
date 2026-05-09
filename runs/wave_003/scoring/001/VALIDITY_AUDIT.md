# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict: no gold-ID leakage found.

Searches over the frozen reconstruction for `S1`-style, `F1`-style, and `R-F` identifiers returned no hits. The same search over the PLAN also returned no hits, so there is no reconstruction-only gold identifier to cross-reference.

## Vocabulary Check

| Phrase sampled from RECONSTRUCTION | Appears in PLAN? | Assessment |
|---|---|---|
| "Server-canonical, client-presentational" | no exact phrase | Synthesized from PLAN lines "Server owns" and "Client owns"; not rubric/gold vocabulary. |
| "one canonical server record, no merge needed" | yes | Direct PLAN scope phrase. |
| "No last-write-wins, no vector clocks, no merge" | yes | Direct PLAN sync phrase. |
| "monotonic-toward-expressive drift" | yes | Direct PLAN scope phrase. |
| "naturalist prose" | yes | Direct PLAN scope/data/narration phrase. |
| "cross-fade poses, not animations-off" | yes | Direct PLAN scope phrase. |
| "Aggregate operational telemetry only" | yes | Direct PLAN scope phrase. |
| "No recorded audio" | yes | Direct PLAN non-goal phrase. |
| "core experience, not a separate mode" | no exact phrase | A reconstructor abstraction from v1 scope/accessibility surfaces; not gold-only wording. |
| "NOT RECOVERABLE FROM PLAN" | no | Valid phase-2A convention, not a gold/rubric scoring term leak by itself. |

No suspicious rubric-side terms such as "multi-layer recovery," "feature-level fidelity," "weight-3," or "intent fidelity" appear. The word "load-bearing" does not appear.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture`
- `### Data model`
- `### API Surface`
- `### Simulation Engine Design`
- `### Sync Model`
- `### Frontend Rendering Pipeline`
- `### Audio Pipeline`
- `### Accessibility Surfaces`
- `### Performance Budgets and Observability`
- `### Rollout`
- `### Risks`

These mirror PLAN headings and sections, not the GOLD_WHYS IDs or titles. GOLD_WHYS headings such as `S1 — feels-alive-not-robotic`, `F1 — presence-definition`, and `F40 — narration-cadence-slow` are not mirrored.

## 1:1 Mapping Suspect Check

Verdict: no neat 1:1 gold mapping. The reconstruction has 10 system-level bullets and then follows the PLAN's implementation sections. It does not enumerate S1-S9 or F1-F40, does not preserve gold order, and includes many plan-specific implementation items outside the 49 gold whys. This shape is consistent with plan-derived reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "The plan carries 'one canonical server record, no merge needed' from scope into the sync model." | PLAN §Scope says "one canonical server record, no merge needed"; §Conflict prevention says "No last-write-wins, no vector clocks, no merge." | Supported. |
| "The first-frame strategy reinforces this by having the client 'join the aviary mid-scene' with 'No entry animation' and 'No fade-from-black.'" | PLAN §First-frame strategy uses those exact phrases. | Supported. |
| "The narration voice is identical to the notebook voice. A screen-reader user moving from the aviary view to the notebook hears the same register." | PLAN §Screen-reader narration says "The narration voice is identical to the notebook voice" and "same register." | Supported. |

## Verdict

PASS. The reconstruction contains no gold IDs, no rubric vocabulary, no gold-heading mirror, and no 1:1 gold-order structure. Its headings and most articulate claims trace directly to PLAN sections, and its repeated `NOT RECOVERABLE FROM PLAN` entries read as conservative plan-derived behavior rather than contamination.
