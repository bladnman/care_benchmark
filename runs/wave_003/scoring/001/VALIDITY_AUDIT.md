# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no ID leakage found. A targeted search of the frozen reconstruction for gold IDs and rubric-like identifiers (`S1`-`S9`, `F1`-`F40`, `R-F*`, `weight-3`, `feature-level fidelity`, `intent fidelity`, `multi-layer`, `gold`, `rubric`) returned no hits. The reconstruction does not use the gold why IDs or an external taxonomy absent from the plan.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Server-owned canonical state" | PLAN.md:10,29 name server-managed canonical state and server source of truth. | Plan-derived. |
| "clients reading snapshots" | PLAN.md:11,20,29 say snapshots are pulled/read for rendering. | Plan-derived. |
| "Append-only log of user interactions" | PLAN.md:17 names an append-only interaction log. | Exact plan vocabulary. |
| "toward expressive based on presence-time, listens, and offers" | PLAN.md:26 uses this phrasing. | Exact plan vocabulary. |
| "native mobile apps, payments, shared aviaries, social network surfaces, and gamification" | PLAN.md:7 and 46 contain the same exclusions/web-only rollout. | Plan-derived. |
| "Accessibility is treated as a product surface rather than a late add-on" | PLAN.md:6,35-38 include accessibility surfaces; PLAN.md:47 names register alignment risk. | Mild inference, grounded in plan structure. |
| "Naturalist prose stream for screen readers" | PLAN.md:36 contains the phrase. | Exact plan vocabulary. |
| "phase-canceling or repetition" | PLAN.md:47 contains the phrase. | Exact plan vocabulary. |

No rubric-side vocabulary appears freely in the reconstruction. The headings `System-level intent` and `Per-feature whys` are the required phase-2A output scaffold, not evidence of gold-list exposure.

## Heading Mirror Check

The reconstruction headings are: `System-level intent`, `Per-feature whys`, then `Scope`, `Architecture`, `Data Model`, `API Surface`, `Simulation Engine`, `Sync Model`, `Rendering & Audio Pipeline`, `Accessibility`, `Performance`, and `Rollout & Risks`.

These mirror the PLAN headings and the required reconstruction scaffold, not the gold-list titles. They do not echo gold headings such as `feels-alive-not-robotic`, `notice-never-announce`, `presence-definition`, `drift-function`, or `no-gamification-non-goal`.

## 1:1 Mapping Suspect Check

No 1:1 mapping suspect pattern. The reconstruction has six system-level bullets and a plan-section walk-through. It does not enumerate S1-S9, F1-F40, or the 120-feature list, and it does not follow the gold order.

## Plan-Derivation Spot Check

1. Reconstruction: "Server-owned canonical state, with clients reading snapshots rather than owning simulation truth."
   - PLAN support: PLAN.md:10 says the server manages canonical state and the simulation tick; PLAN.md:29 says canonical state is owned by the server and clients read snapshots.
   - Assessment: plan-derived.

2. Reconstruction: "Expressive birds are procedural, state-driven, and tied to presence, listens, and offers."
   - PLAN support: PLAN.md:6 includes procedural audio/animation and drift/mood systems; PLAN.md:24-26 describes server-side drift/mood updates and drift based on presence-time, listens, and offers; PLAN.md:32-33 gives procedural motion/audio.
   - Assessment: plan-derived synthesis.

3. Reconstruction: "Accessibility is treated as a product surface rather than a late add-on."
   - PLAN support: PLAN.md:6 includes accessibility surfaces in v1 scope; PLAN.md:35-38 gives dedicated narration, reduced-motion, and caption surfaces; PLAN.md:47 lists accessibility register alignment as a risk.
   - Assessment: plan-derived inference, not contamination.

## Verdict: PASS

The reconstruction reads as a conservative plan-derived summary. It contains no gold IDs, no rubric vocabulary, no gold-heading mirroring, and no neat S/F mapping. Its frequent `NOT RECOVERABLE FROM PLAN` labels are a healthy sign for this contamination check, because the reconstructor did not appear to backfill missing whys from outside the plan.
