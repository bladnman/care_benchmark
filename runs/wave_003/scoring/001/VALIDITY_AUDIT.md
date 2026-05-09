# VALIDITY_AUDIT - CARE run 001

## ID leakage check

Verdict for this check: PASS.

A direct search of the frozen reconstruction found no gold IDs or rubric IDs: no `S1`-`S9`, no `F1`-`F40`, no `R-Fxx`, and no rubric phrases such as `weight-3`, `multi-layer`, or `intent fidelity`. The corresponding search in PLAN also found no such IDs. There are therefore no ID leakage hits to cross-reference.

## Vocabulary check

Sampled phrases from RECONSTRUCTION and plan derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `slow-burn, observational relationship` | PLAN.md:5 uses the same phrase | plan-derived |
| `monotonic toward expressive` | PLAN.md:8 and PLAN.md:69 | plan-derived |
| `Notice, never announce` | PLAN.md:9 | plan-derived |
| `naturalist observation log` | PLAN.md:10 | plan-derived |
| `authoritative simulation engine` | PLAN.md:27 | plan-derived |
| `sole writer of personality state` | PLAN.md:27 and PLAN.md:97 | plan-derived |
| `aviary continued without you` | PLAN.md:98 | plan-derived |
| `Time to First Bird` | PLAN.md:114 and PLAN.md:128 | plan-derived |
| `Too fast = Tamagotchi; Too slow = Screensaver` | PLAN.md:134 | plan-derived |
| `NOT RECOVERABLE FROM PLAN` | Not in PLAN, but it is a phase-2A reporting marker, not gold-side vocabulary | not suspicious |

No sampled phrase reads like gold/rubric-only vocabulary. The reconstruction does not use `load-bearing`, `feature-level fidelity`, `multi-layer recovery`, or similar rubric-side terms.

## Heading mirror check

The reconstruction headings mirror the PLAN structure rather than the gold list. Its top headings are `System-level intent` and `Per-feature whys`, followed by the plan's section headings: `Scope and v1 Definition`, `Architecture`, `Data Model`, `Simulation Engine & Drift Function`, `Frontend Rendering & Audio Pipeline`, `Sync & Conflict Model`, `Accessibility Design`, `Performance & Observability`, `Rollout Strategy`, and `Risks & Mitigations`.

These headings do not echo the gold-list section titles such as `System-level whys`, `Feature-level whys`, or individual S/F titles. The section order is plan-derived.

## 1:1 mapping suspect check

Verdict for this check: PASS.

The reconstruction does not produce a neat S1-S9 and F1-F40 list. It has seven system-level bullets and then plan-section bullets in the same order as PLAN.md. Several gold targets have no corresponding reconstruction item, while several reconstruction bullets refer to plan implementation details outside the canonical F1-F40 why list. This is not a 1:1 gold mapping.

## Plan-derivation spot check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `The plan emphasizes procedural personality drift, rules-based call grammar, WebAudio synthesis, micro-motion, parallax, interpolation, timing variation, and pitch variation.` | PLAN.md:8, PLAN.md:36, PLAN.md:81-90, PLAN.md:135 | grounded |
| `The plan's rationale is that clients send events, the server computes new state, and multiple devices can contribute without last-write-wins conflicts.` | PLAN.md:96-98 | grounded |
| `The rationale is privacy-preserving analytics: no aggregate tracking of specific per-bird interactions, only anonymized session duration and general health metrics.` | PLAN.md:119 | grounded |

The spot-checked articulate sentences are supported by the plan. They may compress or generalize, but they do not appear to introduce gold-only information.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no gold ID leakage, no rubric vocabulary, no heading mirror to the gold list, no 1:1 S/F mapping, and the sampled high-signal sentences are grounded in PLAN.md. The main scoring weakness is compression and non-recovery, not contamination.
