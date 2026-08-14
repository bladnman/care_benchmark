# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A targeted search for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` in `RECONSTRUCTION.md` returned no hits. Because there were no offending IDs, no PLAN cross-reference was needed.

## Vocabulary Check

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, `fidelity`, or `recovery`.

Sampled reconstruction phrases and PLAN grounding:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "observational attention rather than custodial maintenance or gamified retention loops" | PLAN.md line 6 uses the same phrase. | grounded |
| "server-side personality drift driven by honest presence accounting" | PLAN.md line 6 uses the same phrase. | grounded |
| "Server" is the "sole author and writer of canonical state" | PLAN.md line 80 states this directly. | grounded |
| "no stats screens, debug toggles, or telemetry leakage" | PLAN.md strict non-goals include this refusal. | grounded |
| "silent visit logging" | PLAN.md social scope includes silent visit logging. | grounded |
| "Screen-reader running narration and call captioning built and tested in parallel from Phase 1" | PLAN.md line 464 states this mitigation. | grounded |
| "no two calls are identical" | PLAN.md line 361 states this for runtime variation. | grounded |

## Heading Mirror

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, and then plan-section headings such as `Executive Summary & Product Scope`, `System Architecture & Boundaries`, `Data Models & Database Schemas`, and `Simulation Engine Design`.

These headings do not mirror the held-out gold list headings (`S1 - feels-alive-not-robotic`, `F1 - presence-definition`, etc.). They mirror the PLAN structure, which is exactly what the phase-2A prompt requested.

## 1:1 Mapping Suspect

No 1:1 gold mapping was detected. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not proceed in gold-list order. It has seven system bullets and a plan-structured per-feature pass with many features marked `NOT RECOVERABLE FROM PLAN`, which is consistent with plan-derived reconstruction rather than gold-list alignment.

## Plan-Derivation Spot Check

1. Reconstruction: "Aliveness should be quiet, procedural, and slowly earned by honest presence."
   PLAN support: line 6 says aliveness is expressed through procedural calls, subtle mood-based idle animations, and server-side personality drift driven by honest presence accounting.

2. Reconstruction: "Canonical inner life belongs on the server; the client observes and renders."
   PLAN support: line 80 says the server is the sole author and writer of canonical state, personality vectors, mood transitions, and notebook entries; line 81 says the client is pure rendering, synthesis, and observation.

3. Reconstruction: "Accessibility and performance are part of the aviary, not an afterthought."
   PLAN support: section 1.2 includes ARIA narration, reduced motion, captions, and performance budgets in v1 scope; line 464 says screen-reader narration and call captioning are built and tested in parallel from Phase 1.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no scorer/rubric vocabulary, no gold heading mirror, and no neat S/F target mapping. Its strongest phrases are directly supported by the assigned PLAN, and the many `NOT RECOVERABLE FROM PLAN` markers are a positive sign that the reconstructor did not backfill missing rationale.
