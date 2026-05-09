# VALIDITY_AUDIT - CARE run 001

## ID Leakage

PASS. A direct scan found no gold IDs or external taxonomy markers in the frozen reconstruction: no S1-S9, F1-F40, R-F identifiers, GOLD, RUBRIC, or fidelity terminology. The reconstruction uses plan-native headings and feature names rather than gold-list labels.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Naturalist product voice, not game pressure" | PLAN.md:3, 13, 16, 20-22 | Plan-derived synthesis. |
| "A calm, bounded aviary" | PLAN.md:10, 137-140 | Inference from 2/5/7 rollout and cap; not gold-side vocabulary. |
| "Server-authoritative, canonical simulation" | PLAN.md:28, 31, 96 | Directly plan-derived. |
| "Bird change is slow, monotonic, and non-punitive" | PLAN.md:77-81, 145 | Directly plan-derived. |
| "Procedural life over static playback" | PLAN.md:88-90, 105, 144 | Plan-derived synthesis. |
| "Social interaction is opt-in and read-only" | PLAN.md:15, 22, 70-71 | Directly plan-derived. |
| "Accessibility and performance are product constraints" | PLAN.md:16, 119-130 | Plan-derived synthesis. |
| "NOT RECOVERABLE FROM PLAN" | Phase-2A reconstruction convention, not a gold ID | Not a contamination signal by itself. |

No suspicious rubric-side phrases such as "multi-layer recovery," "feature-level fidelity," "weight-3," "load-bearing," or "intent fidelity" appear in the reconstruction.

## Heading Mirror

The top headings are the expected reconstruction contract: "System-level intent" and "Per-feature whys." Subheadings mirror the candidate PLAN structure: "Scope and Boundaries," "Architecture," "Data Model," "API Surface," "Simulation Engine Design," "Sync and Consistency," "Frontend Rendering Pipeline," "Audio Pipeline," "Accessibility Surfaces," and "Performance Budgets." They do not mirror GOLD_WHYS section titles or S/F identifiers.

## 1:1 Mapping Suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40 in order. It follows the candidate plan's own section order and reconstructs a broader set of plan items, including items that are not gold-why rows. Several gold targets are absent or marked not recoverable, which argues against a neat gold-list mapping.

## Plan-Derivation Spot Check

1. Reconstruction: "Clients are thin rendering and interaction-capture shells." Support: PLAN.md:28 states that exact client role.
2. Reconstruction: "The calibration target of ~3 weeks is there to avoid drift being too fast or too slow." Support: PLAN.md:81 gives the ~3 week visible-change target; PLAN.md:145 names the too-fast/Tamagotchi and too-slow/Static risk.
3. Reconstruction: "The rationale is to allow visits without creating a social network or allowing visitor authorship." Support: PLAN.md:15 limits social to read-only per-invite visits; PLAN.md:22 excludes social network features; PLAN.md:71 gives a read-only visitor snapshot.

## Verdict

PASS. I found no significant contamination signatures. The reconstruction reads as plan-derived: it uses plan headings, plan vocabulary, and plan-supported synthesis; it does not leak gold IDs, mirror the gold list, or use rubric-side recovery terminology.
