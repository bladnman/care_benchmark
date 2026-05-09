# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no ID leakage found. A targeted search for gold-style IDs and rubric-side labels in `RECONSTRUCTION.md` found no `S1`-`S9`, `F1`-`F40`, `R-Fxx`, `multi-layer`, `weight-3`, `intent fidelity`, or `feature-level fidelity` references. The only suspicious-looking term, `load-bearing`, also appears in `PLAN.md`, so it is plan-derived vocabulary rather than gold/rubric leakage.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Finding |
|---|---|---|
| `already-running aviary` | PLAN first-frame strategy uses the same phrase. | Pass |
| `real, not policy` | PLAN telemetry boundary says the privacy boundary is real, not policy. | Pass |
| `too specific and load-bearing` | PLAN notebook generation uses the same phrase for voice. | Pass |
| `not learning to mistrust` | PLAN monotonic drift section uses the same phrase. | Pass |
| `age-only - never engagement-driven` | PLAN adding-birds section says pacing is age-only and never engagement-driven. | Pass |
| `silently delete drift` | PLAN no-last-write-wins section uses the same phrase. | Pass |
| `product remains complete` | PLAN audio-off section uses the same phrase. | Pass |
| `first harmless engagement feature cracks the rule` | PLAN risk section says a harmless engagement feature is added and the first one cracks the rule. | Pass |
| `same canonical state` / `naturalist voice` | PLAN screen-reader narration states both. | Pass |

No sampled phrase reads like rubric-side scoring language. The reconstruction vocabulary is dense, but it is grounded in PLAN phrasing.

## Heading Mirror Check

Reconstruction headings are `System-level intent`, `Per-feature whys`, and plan-shaped subsections such as `Scope, service shape, and data model`, `API surface`, `Simulation engine design`, `Sync model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Performance budgets and observability`, `Rollout`, `Risks`, and `Work breakdown, assumptions, and ship criteria`.

These do not mirror the gold list's S1-S9 or F1-F40 titles. `System-level intent` and `Per-feature whys` are expected phase-2A output sections, not evidence of gold-list access.

## 1:1 Mapping Suspect Check

Verdict: no 1:1 mapping. The reconstruction follows the candidate PLAN's section order and includes many plan-specific items that are not gold why rows, such as service shape, SQL tables, rate limits, render stack, rollout phases, and risk register items. It also misses or compresses several gold whys, which is inconsistent with access to the gold list.

## Plan-Derivation Spot Check

1. Reconstruction: `The server is the only writer of canonical state; the client is a renderer.`
   PLAN support: Sync model states exactly that summary and repeats the no client-side personality write invariant.

2. Reconstruction: `The plan says last-write-wins on personality would silently delete drift.`
   PLAN support: The no-last-write-wins section says this directly and removes any client personality write API.

3. Reconstruction: `Reduced motion is its own designed surface with cross-fades, not a broken renderer that merely skips rAF callbacks.`
   PLAN support: Reduced-motion section says it is not implemented as skipped rAF callbacks and has its own cross-fade aesthetic.

All three articulate claims are directly plan-derived.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as a dense but faithful transformation of the PLAN: no gold IDs, no rubric vocabulary, no gold heading mirror, no neat S/F mapping, and strong direct support for sampled high-signal phrases.
