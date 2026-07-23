# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: PASS.

A regex check for gold/rubric identifiers in the frozen reconstruction found no matches for gold IDs or rebuild IDs such as F1-F40, S1-S9, R-F*, or R-S*. The reconstruction does not use external feature IDs. It names features in PLAN terms such as "Web-Only Client", "Server-Side Simulation Tick", and "Screen-Reader Narration".

## Vocabulary Check

Verdict: PASS.

Sampled load-bearing phrases from RECONSTRUCTION and checked them against PLAN vocabulary:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Web-Only Client" | PLAN 1.1 uses the exact heading. | Plan-derived. |
| "No Native Applications" | PLAN 1.2 uses the exact heading. | Plan-derived. |
| "strictly monotonic toward expressive" | PLAN 1.2 uses this phrase. | Plan-derived. |
| "sole writer" | PLAN 6.1 says the server tick is the sole writer. | Plan-derived. |
| "Strict Privacy Isolation Boundary" | PLAN 10.2 uses the exact heading. | Plan-derived. |
| "naturalist observations" | PLAN 1.1/9.1 use naturalist observation/prose language. | Plan-derived. |
| "Zero Recorded Audio" | PLAN 8.1 uses the exact phrase. | Plan-derived. |
| "Complete accessibility implementation" | PLAN 11.1 uses the exact phrase. | Plan-derived. |
| "mechanical UI state logs" | PLAN 12.4 uses the phrase. | Plan-derived. |
| "artificial presence inflation" | PLAN 6.2/12.3 discuss preventing artificial/5x presence inflation. | Plan-derived. |

No scorer-side vocabulary such as "gold", "rubric", "weight-3", "multi-layer recovery", "feature-level fidelity", or "system-level fidelity" appears in RECONSTRUCTION.

## Heading Mirror Check

Verdict: PASS.

The top-level headings "System-level intent" and "Per-feature whys" are required by the phase-2A prompt. The third-level headings mirror PLAN sections: "Scope & System Boundaries", "Explicit Non-Goals", "Architecture & Service Topology", "Data Model & Schema Definitions", "API Surface & Web Protocols", "Simulation Engine & Algorithm Specifications", "Multi-Device Sync & Conflict Prevention", "Frontend Rendering Pipeline & Visual Design System", "Audio Pipeline & WebAudio Runtime", "Accessibility Implementation Strategy", "Performance Budgets & Observability", "Rollout & Operations Strategy", and "Engineering Risks & Mitigation Strategies".

They do not mirror GOLD_WHYS section titles or S/F IDs.

## 1:1 Mapping Suspect Check

Verdict: PASS.

The reconstruction is not a neat S1-S9/F1-F40 mapping. It contains 9 system-level bullets and many per-feature entries, but the per-feature entries follow the PLAN's own section order and include plan-specific architecture/API/schema/risk items that are not the gold feature list. Several gold-bearing features are absent or marked NOT RECOVERABLE FROM PLAN, which is inconsistent with contamination-style one-to-one recovery.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan treats the server tick as the 'sole writer' of personality vectors and canonical snapshots." | PLAN 6.1: "The server simulation tick is the sole writer of personality vectors ..." | Directly supported. |
| "One bird's call is raised in the mix while others drop 'to an ambient floor' and are 'never muted.'" | PLAN 1.1 Listen-In and 8.2 Listen-In Mix Dynamics specify ambient floor/never muted behavior. | Directly supported. |
| "The plan's rationale is preventing users with 5 tabs from inflating presence time '5x.'" | PLAN 12.3 names the multi-tab presence duplication risk as users opening 5 tabs and inflating presence time 5x. | Directly supported. |

## Verdict

PASS.

No significant contamination signs were found. The reconstruction uses PLAN headings, PLAN vocabulary, and PLAN-specific implementation details; it does not leak gold IDs or rubric vocabulary, and it does not present a suspicious one-to-one mapping to the held-out gold list. Some system bullets are articulate, but their supporting phrases are traceable to PLAN passages rather than to the gold/rubric side.
