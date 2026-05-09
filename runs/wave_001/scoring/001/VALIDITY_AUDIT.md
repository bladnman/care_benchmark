# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no ID leakage found.

A mechanical search for gold-side IDs and scorer terms (`F1`, `F40`, `S1`, `S9`, `R-F01`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `load-bearing`) found no hits in the frozen reconstruction. The reconstruction uses plan-local API paths and section labels, not gold identifiers.

## Vocabulary Check

Sampled reconstruction phrases against PLAN:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Browser-based virtual aviary" | PLAN Scope uses the same phrase. | plan-derived |
| "web-only" | PLAN non-goals and rollout repeat web-only v1. | plan-derived |
| "Ambient life over game pressure" | Derived from PLAN non-goals plus risks preserving the "alive" feel. | plan-derived synthesis |
| "Server-authored continuity" | PLAN says state changes are "server-authored" and server is source of truth. | plan-derived synthesis |
| "Naturalist voice as an interface layer" | PLAN names naturalist observations, prose, and captions. | plan-derived synthesis |
| "Accessibility as a designed surface" | PLAN Risks says accessibility is "a designed surface." | plan-derived |
| "Privacy and revocability" | PLAN has strict privacy boundary and explicit, revocable invitations. | plan-derived synthesis |
| "organic variation" | PLAN Audio risk says the grammar should ensure organic variation. | plan-derived |

No rubric-side vocabulary such as multi-layer recovery, feature-level fidelity, weight-3, gold why, or intent fidelity appears in the reconstruction.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture`
- `### Data Model`
- `### API Surface`
- `### Simulation Engine Design`
- `### Sync Model`
- `### Frontend Rendering Pipeline`
- `### Audio Pipeline`
- `### Accessibility Surfaces`
- `### Performance Budgets and Observability`
- `### Rollout`
- `### Risks`

These mirror the PLAN section structure, not the gold-list grouping. They do not echo gold section titles such as system-level whys, feature-level whys, or targeted headroom additions beyond the two required reconstruction section labels.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not create one item per S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold order. It follows PLAN sections and marks many plan-listed features as `NOT RECOVERABLE FROM PLAN`, which is consistent with a plan-derived blind reconstruction rather than a scorer-side target map.

## Plan-Derivation Spot Check

1. Reconstruction: "Because state changes are server-authored, devices natively stay in sync without client-side merging."
   PLAN support: Sync Model says all state changes are server-authored and devices stay in sync "without client-side merging."

2. Reconstruction: "Frame-by-frame animation is swapped for slow cross-fades between static poses."
   PLAN support: Frontend Rendering Pipeline says reduced-motion mode swaps animation for "slow cross-fades between static poses."

3. Reconstruction: "If WebAudio is unavailable or denied, the app falls back gracefully to silence with captions enabled."
   PLAN support: Audio Pipeline says the same fallback rule in nearly identical words.

All three articulate sentences are directly grounded in the plan.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary, no gold-order mapping, and headings track the candidate PLAN rather than the held-out gold list. The only mild concern is that a few system-level labels are polished syntheses, but each sampled phrase is grounded in PLAN wording and structure.
