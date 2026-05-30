# VALIDITY AUDIT - CARE run 001

## Gold ID Leakage Check

Mechanical search found no gold IDs or rebuild IDs in `RECONSTRUCTION.md`; there were no matches for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. Because there were no hits, there was no need to cross-reference against `PLAN.md` for ID reuse.

## Vocabulary Check

Sampled vocabulary reads as plan-derived rather than rubric-derived:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "continuously alive" | PLAN planning posture says the aviary feels continuously alive | Plan-derived |
| "already ongoing" | PLAN risk says performance misses break the already-alive conceit | Plan-derived |
| "one canonical source of truth and one simulation writer" | PLAN system shape uses the exact phrase | Plan-derived |
| "tightly scoped, high-quality experience" | PLAN release framing uses the exact phrase | Plan-derived |
| "game, notification, and social-network patterns" | PLAN planning posture/definition of done uses this framing | Plan-derived |
| "privacy boundaries are product boundaries" | This is a synthesis of PLAN data-governance and quiet-social privacy-risk sections | Acceptable plan-derived synthesis |
| "first-class render mode" | PLAN reduced-motion section says first-class render mode | Plan-derived |
| "naturalist prose" | PLAN notebook/narration sections use this phrase | Plan-derived |
| "matter-of-fact" | PLAN account/error surfaces use this phrase | Plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Required by the phase-2A reconstruction prompt, not gold/rubric leakage | Expected workflow vocabulary |

A mechanical suspicious-vocabulary search found one incidental hit for `recovery` in "30-day recovery" in the deletion-flow bullet. It is ordinary product vocabulary from the plan, not rubric leakage. No uses of `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, or `intent fidelity` appeared.

## Heading Mirror Check

`RECONSTRUCTION.md` headings follow the required phase-2A shape and the PLAN's implementation sections, not the gold list:

| Reconstruction heading | Closest source | Mirror concern |
|---|---|---|
| `## System-level intent` | Required phase-2A output section | No concern |
| `## Per-feature whys` | Required phase-2A output section | No concern |
| `### Scope and product surface` | PLAN section 1 plus product-surface summary | No concern |
| `### Product architecture` | PLAN section 2 | No concern |
| `### Data model and governance` | PLAN section 3 | No concern |
| `### API surface` | PLAN section 4 | No concern |
| `### Simulation engine design` | PLAN section 5 | No concern |
| `### Sync and consistency model` | PLAN section 6 | No concern |
| `### Frontend rendering pipeline` | PLAN section 7 | No concern |
| `### Audio pipeline` | PLAN section 8 | No concern |
| `### Accessibility surfaces` | PLAN section 9 | No concern |
| `### Performance, observability, security, and deletion` | PLAN sections 10-11 | No concern |
| `### Delivery, rollout, testing, and risks` | PLAN sections 12-15 | No concern |

No headings mirror `GOLD_WHYS.md` titles such as `feels-alive-not-robotic`, `presence-definition`, or `drift-function`.

## 1:1 Mapping Suspect Check

The reconstruction does not present S1-S9 or F1-F40 IDs, and it does not create a neat 49-item gold-order ledger. It follows the PLAN order: scope, architecture, data model, API, simulation, sync, frontend, audio, accessibility, performance/security, and delivery/testing/risks. That shape is consistent with plan derivation. It is detailed, but the detail count follows the plan's implementation bullets rather than the gold taxonomy.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Canonical continuity belongs to the server." | PLAN says to use "one canonical source of truth and one simulation writer" and that clients only read snapshots and send events. | Grounded |
| "Reduced-motion rendering: pose cross-fades, dissolves, removed drift, and slower color transitions preserve readability and comfort as a first-class mode." | PLAN reduced-motion section lists pose-state cross-fades, perch-to-perch dissolves, removal of leaf/feather drift, slowed color transitions, and says to build it as a first-class render mode. | Grounded |
| "Quiet-social mitigation: separate endpoints and schemas, explicit invites, notifications off, no badges, and copy/telemetry review guard against privacy leaks or engagement wedges." | PLAN risk mitigation for quiet social says to keep endpoints/schemas separate, require explicit invites, default notifications off, avoid badges, and review copy/telemetry fields. | Grounded |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses the PLAN's structure, vocabulary, and implementation grouping; it contains no gold IDs, no rubric-specific scoring terminology, and no 1:1 mirror of the gold list. The few load-bearing phrases that sound evaluator-like are either exact plan phrases or required phase-2A workflow language.
