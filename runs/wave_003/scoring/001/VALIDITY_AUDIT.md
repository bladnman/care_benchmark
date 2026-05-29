# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: no leakage found. A search for gold-style identifiers (`S1`-`S9`, `F1`-`F40`, and `R-Fxx`) found no hits in the frozen reconstruction. The only rubric-adjacent-looking phrase surfaced by the same scan was `load-bearing`, and it appears directly in PLAN.md (`affective rules are load-bearing`; `Engineering invariants (the load-bearing rules)`).

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `load-bearing` | PLAN opening and section 0 use the same term | plan-derived |
| `feel as architecture, not decoration` | PLAN opening: feel constraints are architecture, not decoration | plan-derived |
| `structural absence over policy` | PLAN says features are not built so they cannot leak in; no toast/badge primitive | plan-derived paraphrase |
| `sole writer` | PLAN I1: server is sole writer of personality and canonical mood/state | plan-derived |
| `3-condition conjunction` | PLAN I4 and presence detector use visible/focused/recent activity | plan-derived |
| `not computed` | PLAN I8 says streak/visit-frequency metrics are not computed | plan-derived |
| `matter-of-fact voice` | PLAN I11 and auth/visit surfaces use matter-of-fact voice | plan-derived |
| `procedural-only` | PLAN I9 uses audio procedural-only | plan-derived |
| `launch-blocking` | PLAN I12 and rollout say accessibility is launch-blocking | plan-derived |
| `NOT RECOVERABLE FROM PLAN` | Not in PLAN; this is a phase-2A reconstruction marker, not gold/rubric content | acceptable notation |

I did not find gold-only vocabulary such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, or `intent fidelity` in RECONSTRUCTION.md.

## Heading Mirror Check

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope and non-goals`
- `### Architecture, data, and API`
- `### Simulation engine`
- `### Frontend rendering`
- `### Audio pipeline`
- `### Accessibility, performance, and rollout`

These do not mirror the gold-list S1-S9 or F1-F40 headings. The first two headings reflect the expected reconstruction artifact structure; the remaining headings follow the PLAN's organization rather than GOLD_WHYS.md's product-file grouping.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping pattern found. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not present 49 neat target rows. It follows the plan's structure, including broad plan-only architecture/API items and several `NOT RECOVERABLE FROM PLAN` entries. That shape is consistent with a plan-derived reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `The plan treats feel as architecture, not decoration.` | PLAN opening says affective rules are load-bearing and feel constraints are architecture, not decoration. | supported |
| `Presence must be honest, not inferred from a tab being open.` | PLAN I4 requires visibility, focus, and recent pointer/key activity; section 5.2 says tab open alone never emits. | supported |
| `Privacy is enforced by topology, not best intentions.` | PLAN I7 and architecture isolate telemetry by network path/IAM and exclude simulation DB from analytics. | supported |

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no gold IDs, no gold-order mirror, no rubric vocabulary beyond normal phase-2A notation, and the strongest phrases can be traced back to PLAN.md. Scores do not need contamination discounting.
