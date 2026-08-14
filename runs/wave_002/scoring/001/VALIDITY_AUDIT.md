# VALIDITY_AUDIT — CARE run 001

## ID Leakage

Verdict: no ID leakage found. A direct scan of the frozen reconstruction found no gold IDs or rubric-style identifiers such as `S1`, `F1`, `F40`, `R-F01`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `gold`, or `rubric`. The plan also contained no such IDs, so there are no suspicious reconstruction-only ID hits to cross-reference.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "felt aliveness" | PLAN §1.1 uses "felt aliveness" | Plan-derived |
| "quiet, observational relationship" | PLAN §1.1 uses same phrase | Plan-derived |
| "honest idle presence" | PLAN §1.1 uses same phrase | Plan-derived |
| "The Server is the Sole Writer" | PLAN §6.1 exact heading/bullet | Plan-derived |
| "matter-of-fact tone" | PLAN §6.2 exact phrase | Plan-derived |
| "Zero Audio Files" | PLAN §8.1 exact phrase | Plan-derived |
| "No Spinners" | PLAN §7.2 exact phrase | Plan-derived |
| "dedicated reduced-motion mode" | PLAN §1.2 exact phrase | Plan-derived |
| "Zero aggregation or ML usage" | PLAN §1.2 exact phrase | Plan-derived |
| "read-only, non-co-present" | PLAN §1.2 exact phrase | Plan-derived |

No rubric-side vocabulary was found. The reconstruction does use the instruction-provided headings "System-level intent" and "Per-feature whys," but those are expected phase-2A output headings rather than gold-list leakage.

## Heading Mirror

The reconstruction headings mirror the candidate plan's section headings almost exactly: "Executive Summary & Scope," "System Architecture & Service Topology," "Data Model & Storage Schema," "API Surface & Communication Protocols," "Simulation Engine Design & Mechanics," "Multi-Device Sync & Conflict Prevention," "Client-Side Rendering Pipeline," "WebAudio Procedural Sound Architecture," "Accessibility Architecture," "Performance Budgets & Observability," "Rollout & Release Plan," and "Risk Matrix & Mitigation Strategies." These are plan headings, not GOLD_WHYS headings.

Gold headings are ID-based and why-based, e.g. "S1 — feels-alive-not-robotic," "F2 — drift-function," and "F40 — narration-cadence-slow." The reconstruction does not mirror that taxonomy.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not create one item per S1-S9 and F1-F40 in gold order. Instead, it follows the plan's architecture sections and enumerates plan bullets such as core simulation, bird population, APIs, rendering, WebAudio, accessibility, rollout, and risks. Some gold targets receive no neat counterpart, and several plan sections cover features that are not gold-why-bearing. This pattern is consistent with plan derivation.

## Plan-Derivation Spot Check

1. Reconstruction: "Felt aliveness is the core engineering imperative... canonical server-side tick... birds already in motion... non-repeating chorus acoustics... drift monotonically."
   PLAN support: §1.1 states the core engineering imperative is felt aliveness and lists the same tick, motion, audio, and drift mechanisms.

2. Reconstruction: "The rationale is operational health without behavioral surveillance... zero aggregation or ML over bird/account interaction data."
   PLAN support: §1.2 says zero aggregation or ML usage of per-bird/per-account interaction data; §10.2 lists operational-only metrics and strict telemetry exclusions.

3. Reconstruction: "The rationale is preventing ARIA live spam or desynchronization through throttled narration and priority interruption only for direct gestures."
   PLAN support: §9.1 sets a 30-60 second narration cycle with gesture-triggered updates; the risk matrix names ARIA live spam and priority interruption only for direct gestures.

All three articulate sentences are grounded in the plan.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs or rubric vocabulary appear, headings mirror the plan rather than GOLD_WHYS, there is no neat S/F 1:1 mapping, and spot-checked rationale sentences are supported by PLAN passages. The run's scores do not need contamination discounting.
