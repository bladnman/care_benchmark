# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A mechanical search of `RECONSTRUCTION.md` for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` produced no hits. No gold IDs, canonical why IDs, or rebuild IDs appear in the frozen reconstruction. The reconstruction also avoids external target-list labels.

## Vocabulary Check

The sampled vocabulary reads as plan-derived rather than rubric-derived:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "architectural exclusions" | PLAN §1: exclusions are "architectural exclusions" | Plan-derived |
| "a field that exists is a field someone eventually surfaces" | PLAN §1 uses the same sentence | Plan-derived |
| "cannot ship late" | PLAN §1 says narration/reduced-motion/captioning cannot ship late | Plan-derived |
| "server is the sole writer of personality and mood" | PLAN §7 uses this sentence | Plan-derived |
| "never the raw `[0,1]` scalars" | PLAN §5 uses this phrase | Plan-derived |
| "five deployables, deliberately not more" | PLAN §2 uses this phrase | Plan-derived |
| "event-sourced/Kafka pipeline" | PLAN §2 contrasts Postgres with Kafka/event-sourcing | Plan-derived |
| "same product, voiced two ways" | PLAN §10 uses this phrase | Plan-derived |
| "ObservedFact" | PLAN §3 and §6 define/use ObservedFact | Plan-derived |
| "synthetic `Account.id` UUID" | PLAN §12 uses this phrase | Plan-derived |

Suspicious scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, and `CARE` do not appear in `RECONSTRUCTION.md`.

## Heading Mirror Check

`RECONSTRUCTION.md` headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture`
- `### Data model`
- `### API surface`
- `### Personality vector exposure`
- `### Simulation engine design`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance budgets and observability`
- `### Privacy and identifier hygiene`
- `### Rollout`
- `### Risks and mitigations`

These mirror the plan's section headings, not `GOLD_WHYS.md` headings. There are no headings like `S1`, `F1`, `feels-alive-not-robotic`, `presence-definition`, or `Feature-level whys (40 entries)`.

## 1:1 Mapping Suspect Check

No 1:1 gold-list mapping is present. The reconstruction is organized by plan sections and implementation features, not by S1-S9 and F1-F40. It includes many plan-specific items not present as gold whys, such as BFF service shape, object storage, code splitting, WebSocket avoidance, DOM/SVG birds, and WebAudio graph shape. It also omits or marks as `NOT RECOVERABLE FROM PLAN` several gold-bearing features, which argues against hidden gold-list access.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Treat v1 scope and non-goals as architecture, not absence."  
   PLAN support: §1 says exclusions are "standing constraints" and "architectural exclusions," with the example that no consecutive-days field should exist because fields are later surfaced.  
   Result: grounded.

2. Reconstruction sentence: "Keep the bird world canonical on the server, but make liveliness local."  
   PLAN support: §7 says the server is the sole writer of personality and mood; §2 says moment-to-moment liveliness is a client-local procedural simulation driven by slow-changing server parameters.  
   Result: grounded.

3. Reconstruction sentence: "Screen-reader narration... same product, voiced two ways, not two products."  
   PLAN support: §10 uses the same phrase and ties narration to the same snapshot as the visual surface.  
   Result: grounded.

## Verdict: PASS

The frozen reconstruction shows no significant contamination signatures. It has no gold ID leakage, no rubric-side vocabulary, no heading mirror of the gold list, no neat S/F target mapping, and its most articulate claims are directly supported by the assigned PLAN. Scores can be treated as valid for this slot.
