# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict for this section: PASS. I found no gold IDs or benchmark taxonomy tokens in the frozen reconstruction. Searches for `F1`-style IDs, `S1`-style IDs, `R-F01`, `feature-level`, `system-level`, `weight-3`, `multi-layer`, `gold`, and `rubric` returned no hits in `RECONSTRUCTION.md`.

The reconstruction does use ordinary plan headings such as `Scope and Core Identity`, `Architecture`, and `Data Model`, but those headings also appear in `PLAN.md` and are not gold-side identifiers.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "long-term observational relationships rather than short-term gamified engagement" (line 3) | PLAN.md:5 uses the same phrase. | Plan-derived. |
| "Server acts as the canonical simulation engine" / "stateless rendering" (line 7) | PLAN.md:27 uses the same wording. | Plan-derived. |
| "procedural call grammar" and "personality-shaped timing" (line 9) | PLAN.md:8 and :84. | Plan-derived. |
| "Naturalist prose" (line 11) | PLAN.md:13, :80, :109. | Plan-derived. |
| "one-to-one read-only visit invitations (opt-in only)" (line 13) | PLAN.md:12. | Plan-derived. |
| "critical for 'already in motion' feel" (line 15) | PLAN.md:117. | Plan-derived. |
| "soft-exit gesture" (line 33) | PLAN.md:9. | Plan-derived. |
| "strict Synthetic UUID rule" (line 83) | PLAN.md:135. | Plan-derived. |

I did not find rubric-side vocabulary such as "multi-layer recovery", "feature-level fidelity", "weight-3", "intent fidelity", or "load-bearing" in the reconstruction.

## Heading Mirror Check

| Reconstruction heading | Source comparison | Assessment |
|---|---|---|
| `## System-level intent` | Required reconstruction format, not a gold heading. | Not suspect. |
| `## Per-feature whys` | Required reconstruction format, not a gold heading. | Not suspect. |
| `### Scope and Core Identity` | Mirrors PLAN section `## 1. Scope and Core Identity`. | Plan-derived. |
| `### Included in V1` | Mirrors PLAN subsection. | Plan-derived. |
| `### Explicit Non-Goals` | Mirrors PLAN subsection. | Plan-derived. |
| `### Architecture` | Mirrors PLAN section. | Plan-derived. |
| `### Data Model` | Mirrors PLAN section. | Plan-derived. |
| `### API Surface` | Mirrors PLAN section. | Plan-derived. |
| `### Simulation Engine Design` | Mirrors PLAN section. | Plan-derived. |
| `### Audio Pipeline: Call-Grammar Runtime` | Mirrors PLAN subsection. | Plan-derived. |
| `### Frontend Rendering Pipeline` | Mirrors PLAN section. | Plan-derived. |
| `### Sync Model` | Mirrors PLAN section. | Plan-derived. |
| `### Accessibility Surfaces` | Mirrors PLAN section. | Plan-derived. |
| `### Performance Budgets` | Mirrors PLAN section. | Plan-derived. |
| `### Rollout and Risks` | Mirrors PLAN section. | Plan-derived. |

The heading structure mirrors the PLAN, not GOLD_WHYS.md. No near-exact gold-list section title mirroring was found beyond generic required reconstruction headings.

## 1:1 Mapping Suspect Check

Verdict for this section: PASS. The reconstruction does not map neatly to S1-S9 and F1-F40 in gold order. Instead, it follows the PLAN's ten-section implementation outline and emits many plan-local items such as React/TypeScript, Canvas/WebGL/SVG/DOM, Redis magic-link session management, and rollout phases. It also marks several details `NOT RECOVERABLE FROM PLAN`, which is more consistent with a plan-derived reconstruction than a contaminated gold-list pass.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "The system shape says the 'Server acts as the canonical simulation engine' and the 'Client acts as a stateless rendering and interaction-capture surface.'" | PLAN.md:27 has the same sentence. | Directly plan-derived. |
| "The Bird Engine includes 'procedural call grammar,' 'personality drift (monotonic toward expressive),' and mood systems." | PLAN.md:8 has the same list. | Directly plan-derived. |
| "Performance budgets emphasize 'Time-to-First-Bird,' 60fps, and a bundle size that is 'critical for already in motion feel.'" | PLAN.md:117-119 list bundle, TTFB, and runtime; :117 contains the already-in-motion rationale. | Plan-derived synthesis. |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction's vocabulary and heading structure are tightly traceable to `PLAN.md`, it does not expose gold IDs or rubric vocabulary, and it does not create a neat S1-S9/F1-F40 mapping. The run's low recovery appears to come from the compact plan and honest non-recovery, not from contamination.
