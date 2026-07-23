# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found.

Mechanical search over the frozen reconstruction found no tokens matching gold IDs or rebuild IDs such as `F1`, `S1`, `R-F01`, or `R-S01`. The assigned PLAN also did not contain those IDs, so there were no cross-reference leakage hits to adjudicate.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `felt-aliveness` | PLAN.md:4 uses `felt-aliveness`. | Plan-derived. |
| `traditional gaming mechanics, quests, streaks` | PLAN.md:4 and :24 use this framing. | Plan-derived. |
| `strict triple-conjunction rules` | PLAN.md:139 uses this phrase. | Plan-derived. |
| `single server-authoritative simulation state` | PLAN.md:17 uses this phrase. | Plan-derived. |
| `append-only event logging` | PLAN.md:17 and :54 use this phrase. | Plan-derived. |
| `Opt-In & Quiet` | PLAN.md:19 uses this heading text. | Plan-derived. |
| `Naturalist field-notebook register` | PLAN.md:203 uses this phrase. | Plan-derived. |
| `Time-to-First-Bird Visible` | PLAN.md:212 uses this label. | Plan-derived. |

No scorer-side vocabulary was found: no `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity` hits appeared in the frozen reconstruction.

## Heading Mirror

The two required reconstruction headings are `## System-level intent` and `## Per-feature whys`. The subsequent `###` headings mirror the PLAN's section structure: `Scope`, `Architecture & System Topology`, `Data Model & Schema Specification`, `Simulation Engine Design`, `Sync & State Propagation Model`, `Frontend Layout, Rendering, & Audio Pipelines`, `Accessibility & UX Voice Architecture`, `Performance Budgets, Security, & Observability`, and `Rollout, Instrumentation, & Risk Mitigation`.

They do not mirror the gold-list groups or S/F target names. This supports plan derivation rather than gold-list contamination.

## 1:1 Mapping Suspect

No 1:1 gold mapping pattern is present. The reconstruction has seven system-level bullets and plan-section grouped per-feature bullets, not nine S-level items plus forty F-level items. It does not use gold IDs, gold ordering, or canonical gold feature labels. The order tracks the PLAN's implementation sections.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `The Executive Summary says Pocket Aviary prioritizes "felt-aliveness" over "traditional gaming mechanics, quests, streaks, or Tamagotchi-style custodial obligations."` | PLAN.md:4 contains that same priority statement. | Supported. |
| `The rationale is preventing personality vector overwrite and data loss; server ticks process timestamped events instead.` | PLAN.md:167 says both devices append immutable events and the server tick processes events in timestamp order; PLAN.md:235-236 names sync conflict data loss and zero client vector writes. | Supported. |
| `The rationale is naturalist access to the scene through an aria-live="polite" region with slow-cadence prose every 30-60 seconds.` | PLAN.md:192-195 specifies `aria-live="polite"`, 30-60 second naturalist prose updates, and lowercase present-tense descriptive prose. | Supported. |

## Verdict

PASS. The frozen reconstruction reads as plan-derived: it follows PLAN headings, repeats PLAN vocabulary, contains no gold/rubric IDs, and does not map neatly to S1-S9/F1-F40. Some reconstruction prose is compressed and occasionally stronger than the underlying PLAN, but the spot checks are grounded and any overcompression was handled in scoring rather than treated as contamination.
