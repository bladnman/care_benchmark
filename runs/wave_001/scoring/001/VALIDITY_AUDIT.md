# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no ID leakage found.

Searches for gold/rubric identifiers such as `S1`-`S9`, `F1`-`F40`, `R-F`, `weight-3`, `feature-level fidelity`, `multi-layer`, `load-bearing`, and `intent fidelity` returned no hits in either the frozen reconstruction or the plan. The reconstruction uses plan-native headings and prose rather than gold identifiers.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Attention-responsive aviary" | PLAN.md:4 "respond to user attention" | Plan-derived paraphrase. |
| "Server-authoritative canonical state" | PLAN.md:19,25-27 "Canonical simulation state", "Server-Authoritative", "only writer" | Plan-derived. |
| "Append-only history as the source of change" | PLAN.md:9,15-16,19,53 append-only/event-log/order language | Plan-derived synthesis. |
| "Expressive change without punitive reinforcement" | PLAN.md:4,20 negative reinforcement exclusion and monotonic expressive drift | Plan-derived synthesis. |
| "Calm, accessible presentation" | PLAN.md:30,33,40-43 single scene, reduced-motion, narration, captioning, navigation | Plan-derived synthesis. |
| "Alive naturalist voice across modalities" | PLAN.md:31,36-37,41-42,52 mood motion, procedural audio, naturalist prose, "feels alive" | Plan-derived, slightly interpretive but not rubric-side. |
| "Performance on constrained devices" | PLAN.md:45-48 performance targets | Plan-derived. |

No suspect rubric-side vocabulary appears freely in the reconstruction.

## Heading Mirror Check

The reconstruction has two markdown headings: `## System-level intent` and `## Per-feature whys`, matching the phase-2A output contract rather than gold-list titles. Its unmarked section labels - Scope, Architecture, Data Model, Simulation Engine, Sync Model, Frontend Rendering Pipeline, Audio Pipeline, Accessibility, Performance - mirror PLAN.md headings, not GOLD_WHYS.md sections. No gold-list heading mirror was found.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40 and does not contain 49 neatly ordered gold-target entries. It has 7 system bullets plus plan-section feature bullets in PLAN order. That shape mirrors the candidate plan, not the gold taxonomy.

## Plan-Derivation Spot Check

1. Reconstruction: "The plan repeatedly protects one canonical truth." Support: PLAN.md:19 "Canonical simulation state", PLAN.md:25 "Server is the only writer of state", PLAN.md:26 "canonical snapshots", PLAN.md:27 server-side additive deltas.
2. Reconstruction: "Expressive change without punitive reinforcement." Support: PLAN.md:4 excludes "Tamagotchi-style negative reinforcement" and PLAN.md:20 says drift moves "toward expressive".
3. Reconstruction: "Alive naturalist voice across modalities." Support: PLAN.md:41-42 naturalist prose narration and prose call captions, PLAN.md:31 mood-driven idle motion, PLAN.md:36-37 procedural calls/chorus, and PLAN.md:52 procedural synthesis "feels alive" risk.

All three sampled sentences are grounded in PLAN wording, though the third is a synthesis across several plan bullets.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as a plan-derived compression: it mirrors PLAN structure, avoids gold IDs and rubric vocabulary, and marks many items NOT RECOVERABLE rather than manufacturing gold-side rationales.
