# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict: no ID leakage found.

A bounded search of the frozen reconstruction found no gold IDs such as S1-S9, F1-F40, R-F01, and no rubric labels such as "feature-level fidelity," "multi-layer recovery," "weight-3," "intent fidelity," or "gold." The same search returned no hits in the reconstruction or plan, so there are no offending sentences to cross-reference.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "executable engineering specification" | PLAN opening says the plan is an "executable engineering specification." | Plan-derived. |
| "client is a renderer, not a simulation participant" | PLAN Sync Model uses the exact sentence. | Plan-derived. |
| "not optimized for engagement time" | PLAN Observability says the product is not optimized for engagement time. | Plan-derived. |
| "new birds must never feel earned by attention" | PLAN Ramping birds uses this exact rationale. | Plan-derived. |
| "cross-fade rendering, not stripped fallback" | PLAN Scope and Reduced-motion sections specify cross-fades and not stripped fallback. | Plan-derived. |
| "No sudden snap" | PLAN Transitions says mood changes have no sudden snap. | Plan-derived. |
| "per-bird behavioral analytics" | PLAN deliberately does not measure per-bird behavioral analytics. | Plan-derived. |
| "as if they have always been there" | PLAN Loading state says birds are placed as if they have always been there. | Plan-derived. |
| "Ambient product, not engagement product" | This exact heading is B synthesis, but it is supported by PLAN non-goals, no engagement funnels, no notifications, and no gamification primitives. | Not suspect. |

No rubric-side vocabulary was present. The reconstruction does use polished synthesis headings, but they are grounded in plan language rather than gold/rubric terms.

## Heading Mirror

The reconstruction headings are:

- System-level intent
- Per-feature whys
- Scope
- Architecture
- Data model
- API surface
- Simulation engine design
- Sync model
- Frontend rendering pipeline
- Audio pipeline
- Accessibility surfaces
- Performance budgets and observability
- Rollout
- Risks
- Engineering discipline and guardrails
- Ambiguities and defensible calls

These mirror the PLAN section structure, not GOLD_WHYS section titles. The first two headings are required by the reconstruction workflow shape and do not reveal gold-side IDs or taxonomy.

## 1:1 Mapping Suspect

No 1:1 gold mapping detected. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not preserve gold order. Instead, it follows the plan sections and produces many plan-derived bullets beyond the 49 gold why targets. Several gold targets are absent or marked NOT RECOVERABLE FROM PLAN, which is inconsistent with contamination by the gold list.

## Plan-Derivation Spot Check

1. Reconstruction: "New birds are based on aviary_age_days, not interaction, and must never feel earned by attention; they feel like the aviary maturing."
   Plan support: Ramping birds says new birds are based on aviary_age_days, not user interaction, and "must never feel earned by attention; they feel like the aviary maturing."
   Result: plan-derived.

2. Reconstruction: "The Simulation Service is the canonical state owner, client never computes personality drift, and the client is a renderer, not a simulation participant."
   Plan support: Architecture and Sync Model contain those exact statements.
   Result: plan-derived.

3. Reconstruction: "Reduced-motion mode is cross-fade rendering, not stripped fallback; calls, mood changes, drift, notebook, and interactions remain."
   Plan support: Scope and Reduced-motion mode say cross-fades replace motion and preserve call audio, mood changes, drift, notebook, and interaction affordances.
   Result: plan-derived.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold ID leakage, no rubric vocabulary, headings mirror the PLAN rather than GOLD_WHYS, and articulate sentences have direct plan support. Minor synthesis headings are not exact plan phrases, but they are supported by plan content and do not suggest contamination.
