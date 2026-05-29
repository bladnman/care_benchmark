# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: PASS.

A targeted search of the frozen reconstruction for gold IDs and rubric identifiers found no uses of `S1`-`S9`, `F1`-`F40`, `R-Fxx`, `GOLD`, `RUBRIC`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, or `weight-2/weight-3`. There are no offending sentences to cross-reference against PLAN.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION and their PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `load-bearing architectural fact` | PLAN opening uses `load-bearing architectural fact`. | Plan-derived. |
| `render-only consumers of snapshots plus append-only event producers` | PLAN opening and section 2.2 use this architecture. | Plan-derived. |
| `noticed, never by announcing` | PLAN opening names this as the second shaping fact. | Plan-derived. |
| `instrument-measurable at ~1 week` / `user-visible at ~3 weeks` | PLAN 1.3 and 5.6 name these calibration targets. | Plan-derived. |
| `naturalist, lowercase, present-tense, specific` | PLAN 5.7 uses this notebook voice language. | Plan-derived. |
| `matter-of-fact` | PLAN 6.4 and 8.6 use this for system surfaces. | Plan-derived. |
| `aggregate-only` / `physically separate` telemetry | PLAN 11.1 names aggregate-only telemetry and physical separation. | Plan-derived. |
| `audible signature of dead software` | PLAN 5.8 and 7.1 use this phrase for looped audio. | Plan-derived. |
| `designed surface, not a checklist fallback` | PLAN 8 introduces accessibility in this language. | Plan-derived. |
| `NOT RECOVERABLE FROM PLAN` | This is a reconstruction marker, not a gold ID or rubric-side taxonomy. | Not contamination by itself. |

I found no rubric-side vocabulary used freely. `load-bearing` is also a gold-side word, but it appears directly in PLAN, so it is not a leakage hit.

## Heading Mirror Check

RECONSTRUCTION headings are:

- `System-level intent`
- `Per-feature whys`
- `Core relationship engine`
- `Account & sync`
- `Session surface`
- `Accessibility`
- `Social`
- `Performance & observability`
- `Architecture`
- `Data model`
- `API surface`
- `Simulation engine design`
- `Sync model`
- `Audio pipeline`
- `Frontend rendering pipeline`
- `Affective-constraint enforcement`
- `Observability & privacy boundary`

These mirror the PLAN organization and scope buckets, not the GOLD_WHYS headings such as `S1 - feels-alive-not-robotic`, `F1 - presence-definition`, or `Feature-level whys (40 entries, grouped by file)`. The first two headings match the expected reconstruction output shape, not a gold list mirror.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not provide a neat S1-S9 or F1-F40 list and does not proceed through the gold IDs. It contains many plan-derived implementation bullets, including non-gold architecture, API, data-model, and performance items. It also marks several ordinary PLAN features as `NOT RECOVERABLE FROM PLAN`, which is inconsistent with gold-list access. The ordering follows PLAN scope and implementation sections, not the gold taxonomy.

## Plan-Derivation Spot Check

1. Reconstruction sentence: `The plan's load-bearing architectural fact is that "the server is the only writer of canonical aviary state," the simulation advances on a "slow server-side tick whether or not a client is connected," and clients are "render-only consumers of snapshots plus append-only event producers."`
   Supporting PLAN passage: the opening paragraph states the same three-part architecture nearly verbatim.
   Assessment: plan-derived.

2. Reconstruction sentence: `The plan does not merely hide streaks, levels, scores, badges, XP, ranks, "birds adopted: N," visit counters, or green-dot calendars; it says the system "doesn't compute" them.`
   Supporting PLAN passage: section 10 says no gamification data exists to surface and that the system does not compute these fields.
   Assessment: plan-derived.

3. Reconstruction sentence: `Reduced motion is not "animations off"; it uses cross-fades and slowed shifts so vestibular users get "a calmer Pocket Aviary, not less of one."`
   Supporting PLAN passage: section 8.4 says reduced motion is not animations off and ends with the calmer-not-less formulation.
   Assessment: plan-derived.

## Verdict

PASS. The frozen reconstruction reads as a strong condensation of PLAN.md rather than a contaminated mirror of GOLD_WHYS.md or RUBRIC.md. It uses PLAN vocabulary, follows PLAN structure, contains no gold IDs, and includes misses/`NOT RECOVERABLE FROM PLAN` markers that would be unlikely if the reconstructor had seen the gold list.
