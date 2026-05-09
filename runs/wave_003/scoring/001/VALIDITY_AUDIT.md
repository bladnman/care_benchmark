# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A targeted search for gold identifiers (F1-F40, S1-S9, R-Fxx-style ids) returned no hits in either the frozen reconstruction or the plan. The reconstruction does not use gold why IDs, rubric row labels, or external taxonomy names.

## Vocabulary Check

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | Plan support | Assessment |
|---|---|---|
| "invariants, not defaults" | PLAN opening invariant paragraph | plan-derived |
| "the substance of the product" | PLAN pinned summary | plan-derived |
| "server is the only writer of personality state" | PLAN sections 2, 6, 14 | plan-derived |
| "notice, never announce" | PLAN pinned rule 8 | plan-derived |
| "architectural; not a policy" | PLAN privacy-boundary risk | plan-derived |
| "affective fidelity" | PLAN WebSocket rationale | plan-derived, not rubric leakage |
| "load-bearing read shape" | PLAN AviarySnapshot section | plan-derived |
| "not load-bearing for identity" | PLAN sync model for names/prefs | plan-derived |
| "designed surface, not a checklist" | PLAN accessibility section | plan-derived |
| "never white, never a spinner" | PLAN first-frame section | plan-derived |

No sampled phrase looks gold-side only. The few rubric-adjacent words, especially "load-bearing" and "fidelity," appear in PLAN itself.

## Heading Mirror Check

The top-level headings are generic phase-2A output headings: "System-level intent" and "Per-feature whys." Subheadings under per-feature whys follow the plan's structure (Scope and product surface, Architecture and build shape, Data model and API surface, Simulation engine design, Sync model, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Performance/observability/rollout), not the gold-list grouping/order. No heading mirrors a gold ID or gold section title closely enough to flag.

## 1:1 Mapping Suspect Check

No 1:1 mapping pattern. The reconstruction has 10 system-level bullets rather than S1-S9, and the per-feature section is organized by plan domains with many more bullets than the 40 gold feature whys. It does not march through F1-F40 or use the same order as GOLD_WHYS.md.

## Plan-Derivation Spot Check

1. Reconstruction: "The product refuses gamification of any flavor, notifications by default, Welcome back surfaces, visit-frequency widgets, and social-network surfaces."
   Plan support: PLAN section 1 lists those non-goals explicitly and PLAN section 14 pins notice-never-announce/no gamification rules.

2. Reconstruction: "The tick keeps mood, drift, and call-event scheduling fresh while ensuring the aviary continues without the viewer."
   Plan support: PLAN section 5.1 describes live/dormant queues, catch-up ticks, and the aviary continuing without the viewer.

3. Reconstruction: "Reduced motion is a separate designed rendering path with cross-fades, no leaf drift, preserved calls, preserved mood/day-night, and focus without animation."
   Plan support: PLAN section 7.7 enumerates exactly those reduced-motion behaviors.

All three articulate sentences are directly derivable from PLAN.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no gold IDs, no gold-list order, no unexplained rubric vocabulary, and spot-checked high-signal sentences are grounded in PLAN. The run's scores do not need contamination caveats.
