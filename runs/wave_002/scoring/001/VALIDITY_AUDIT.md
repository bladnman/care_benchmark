# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS. The frozen reconstruction reads as plan-derived: it follows the plan's section order, uses plan-local vocabulary, contains no gold IDs or rubric vocabulary, and marks many items `NOT RECOVERABLE FROM PLAN` rather than inventing gold-side rationales.

## ID Leakage Check

No gold ID leakage found. Targeted scan of `RECONSTRUCTION.md` found no `S1`-`S9`, `F1`-`F40`, `R-Fxx`, `gold`, `rubric`, `weight-3`, `multi-layer`, `feature-level fidelity`, or similar benchmark taxonomy. The headings and bullets are plan-facing labels such as `Scope and Boundaries`, `System Architecture & Component Topology`, and `Data Schema & Persistence Model`.

## Vocabulary Check

Sampled load-bearing phrases from the reconstruction against the plan:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `quiet` / `Optional & Quiet` | PLAN Scope uses `quiet` and `Social (Optional & Quiet)` | plan-derived |
| `presence honesty` | PLAN Architecture says the split ensures `multi-device consistency, presence honesty, and zero client-owned personality states` | exact plan-derived phrase |
| `zero client-owned personality states` | Same architecture sentence in PLAN | exact plan-derived phrase |
| `monotonic toward expressive` | PLAN Drift says `Drift is monotonic toward expressive` | exact plan-derived phrase |
| `Naturalist prose observation in lowercase present tense` | PLAN notebook schema uses the same phrase | exact plan-derived phrase |
| `Looped Sound Perception` | PLAN risk matrix names `Audio Uncanniness / Looped Sound Perception` | exact plan-derived phrase |
| `strict PII/per-bird data isolation` | PLAN Accessibility & Observability scope uses this phrase | exact plan-derived phrase |
| `Accessibility as a first-class rendering path` | PLAN does not use the exact phrase, but lists reduced motion, captions, WCAG, keyboard navigation, and `aria-live` narration | acceptable synthesis from plan |
| `Budgeted reliability across devices and browsers` | PLAN Performance Budgets table and browser matrix ground the phrase | acceptable synthesis from plan |

No suspicious rubric-side phrases appeared in the reconstruction.

## Heading Mirror Check

`RECONSTRUCTION.md` headings mirror the PLAN, not the gold list:

| Reconstruction heading | Closest source | Assessment |
|---|---|---|
| `## System-level intent` | Phase-2A output format | expected |
| `## Per-feature whys` | Phase-2A output format | expected |
| `### 1. Scope and Boundaries` | PLAN `## 1. Scope and Boundaries` | plan mirror |
| `### 2. System Architecture & Component Topology` | PLAN same heading | plan mirror |
| `### 3. Data Schema & Persistence Model` | PLAN same heading | plan mirror |
| `### 4. Simulation Engine & Personality Drift Math` | PLAN same heading | plan mirror |
| `### 5. Client Rendering, Audio & Animation Pipeline` | PLAN same heading | plan mirror |
| `### 6. Accessibility & Interaction Specifications` | PLAN same heading | plan mirror |
| `### 7. Performance Budgets & Technical Constraints` | PLAN same heading | plan mirror |
| `### 8. Release Strategy & Rollout Plan` | PLAN same heading | plan mirror |
| `### 9. Risk Matrix & Mitigation Strategies` | PLAN same heading | plan mirror |

No heading mirrors gold-list section names like `feels-alive-not-robotic`, `notice-never-announce`, or `presence-definition`.

## 1:1 Mapping Suspect Check

No 1:1 mapping to the gold target list. The reconstruction does not enumerate S1-S9 or F1-F40, does not use their order, and does not provide one neat item per gold why. Instead it walks the plan's sections and individual plan bullets. The many `NOT RECOVERABLE FROM PLAN` entries also argue against gold-list exposure: a contaminated reconstruction would be more likely to produce gold-aligned rationale for those slots.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| `The architecture separates client rendering from server-side state simulation to ensure multi-device consistency, presence honesty, and zero client-owned personality states.` | PLAN Architecture introduction uses this sentence almost exactly. | directly plan-derived |
| `The personality model is monotonic toward expressive; traits shift UP on presence and NEVER decay on neglect.` | PLAN Drift section says `Traits shift UP on presence` and `Traits never decrease`; non-goals reject negative drift. | directly plan-derived |
| `The plan uses Naturalist prose observation in lowercase present tense, Naturalist field notes, and screen-reader naturalist prose narration.` | PLAN notebook schema and SR narration section contain those phrases. | directly plan-derived |

## Final Verdict

PASS. There are no significant contamination signatures. The reconstruction is sometimes articulate, but its articulation is traceable to PLAN wording and structure, not to gold IDs, rubric vocabulary, or a gold-list-shaped outline.
