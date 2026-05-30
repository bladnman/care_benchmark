# VALIDITY_AUDIT - CARE run 001

## ID leakage

No gold IDs were found in the frozen reconstruction. It does not use IDs such as F1-F40, S1-S9, R-F01, feature-level fidelity labels, or rubric-specific target names. The reconstruction uses ordinary plan section numbering and plan-derived headings only.

## Vocabulary check

| Reconstruction phrase | PLAN support | Verdict |
|---|---|---|
| "feel alive before, during, and after a visit" | PLAN section 1 says the aviary "feels alive" and "continues without the viewer" | plan-derived |
| "already-running aviary" | PLAN section 1 success criterion uses "already-running aviary" | direct plan vocabulary |
| "quiet field rather than a spinner" | PLAN section 7 first-load experience uses the same phrase | direct plan vocabulary |
| "clients render and emit events; the server owns truth" | PLAN section 2 high-level system shape uses the same sentence | direct plan vocabulary |
| "monotonic toward expressive" | PLAN section 5 drift function uses the same phrase | direct plan vocabulary |
| "semantic events, never calculated deltas" | PLAN section 4 event contract uses this language | direct plan vocabulary |
| "accessible modes should be the real product" | PLAN section 1 success criteria and section 9 accessibility surfaces support it | plan-derived |
| "technically enforceable rather than policy-only" | PLAN section 1 success criteria uses this phrasing | direct plan vocabulary |

Rubric-side vocabulary such as "multi-layer recovery", "feature-level fidelity", "weight-3", "gold why", and "load-bearing" does not appear in the reconstruction.

## Heading mirror

The reconstruction headings are:

- ## System-level intent
- ## Per-feature whys
- ### 1. Scope and product boundaries
- ### 2. Architecture
- ### 3. Data model
- ### 4. API surface
- ### 5. Simulation engine design
- ### 6. Sync model
- ### 7. Frontend rendering pipeline
- ### 8. Audio pipeline
- ### 9. Accessibility surfaces
- ### 10. Performance, observability, security, and rollout

These mirror PLAN.md structure, not GOLD_WHYS.md section titles. No gold heading sequence is reproduced.

## 1:1 mapping suspect

The reconstruction does not provide a neat S1-S9 and F1-F40 mapping. It walks the plan sections and plan feature clusters in order. That is expected for a plan-derived reconstruction and is not suspicious. Some gold targets receive no direct item, while many non-gold plan items receive reconstruction bullets.

## Plan-derivation spot check

1. Reconstruction sentence: "The server owns truth, while clients render and emit events."
   PLAN support: section 2 says "clients render and emit events; the server owns truth" and says the client never writes trait values directly.

2. Reconstruction sentence: "Presence accounting based on visibility + focus + recent input conjunction: The why is honest attention."
   PLAN support: section 5 says the server constructs presence windows only where all three are true; section 13 identifies background tabs and dual-device sessions as drift-inflation risks.

3. Reconstruction sentence: "Accessible modes should be the real product."
   PLAN support: section 1 says accessible modes should feel like the real product, and section 9 says reduced motion is a designed mode rather than disabled animation.

## Verdict: PASS

The reconstruction shows no significant contamination signatures. It uses PLAN language, follows PLAN section structure, does not leak gold IDs or rubric vocabulary, and its most articulate claims are traceable to PLAN passages.
