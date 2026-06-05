# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A direct search of the frozen reconstruction found no standalone gold IDs such as `S1`-`S9`, `F1`-`F40`, or `R-F..`. Because there were no hits, there was no ID to cross-check against the plan.

## Vocabulary Check

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | Plan support | Finding |
|---|---|---|
| "continuous, private, and alive" | PLAN.md opening product interpretation uses the same phrase. | supported |
| "game, pet-care loop, social network, or notification product" | PLAN.md opening paragraph uses the same exclusion framing. | supported |
| "not direct bird command" | PLAN.md event payloads describe offer_seed as "not direct bird command." | supported |
| "like an app waking up" | PLAN.md performance risk says loading can feel "like an app waking up." | supported |
| "server as the canonical author" | PLAN.md repeatedly says the server/database is canonical and the only writer for simulation state. | supported paraphrase |
| "first-class" accessibility | PLAN.md says first-class accessibility surfaces and QA ship with v1. | supported |
| "behavioral dossiers" | Exact phrase is not in PLAN.md, but it paraphrases the plan's ban on per-account history, per-bird telemetry, and behavioral dashboards. | minor paraphrase, not gold/rubric language |
| "NOT RECOVERABLE FROM PLAN" | This is a reconstruction convention, not a gold phrase; used sparingly for unrecoverable plan items. | acceptable |

No rubric-side vocabulary such as "multi-layer," "feature-level fidelity," "weight-3," "gold why," or "intent fidelity" appears in the reconstruction. The single word "recovery" appears only in the ordinary system-outage phrase "recovery replaces descriptors."

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Product interpretation and v1 scope`
- `### System architecture`
- `### Data model`
- `### API surface`
- `### Simulation engine design`
- `### Sync model and conflict prevention`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance and observability`
- `### Rollout plan`
- `### Testing strategy`
- `### Security and privacy plan`
- `### Voice and copy governance`
- `### Key risks and mitigations`
- `### Engineering work breakdown and definition of done`

These mirror the PLAN.md section structure, not the gold-list structure. They do not echo gold titles such as "feels-alive-not-robotic," "notice-never-announce," or F1-F40 labels.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not provide a neat S1-S9/F1-F40 list, does not use gold IDs, and does not proceed in gold-list order. It follows the plan's implementation sections and includes many plan-only implementation items. Several gold targets are absent or marked not recoverable, which is inconsistent with a contaminated 1:1 gold mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "Make the aviary 'continuous, private, and alive' while explicitly keeping it from becoming 'a game, pet-care loop, social network, or notification product.'" | PLAN.md opening paragraph contains the same phrasing. | supported |
| "Presence qualifies only with visible document, focused window, and recent pointer/key activity." | PLAN.md §6.4 lists exactly those three conditions. | supported |
| "Reduced-motion mode is a parallel presentation layer... preserves calls, captions, mood, notebook, drift, and interactions." | PLAN.md §7.5 lists the same reduced-motion preservation rules. | supported |

## Verdict: PASS

The reconstruction reads as plan-derived. It has no gold ID leakage, no rubric vocabulary beyond ordinary non-technical usage, headings that mirror PLAN.md rather than GOLD_WHYS.md, and spot-checkable support for its most articulate claims. The only minor note is occasional paraphrase such as "behavioral dossiers," but it is grounded in the plan's telemetry prohibitions and does not indicate contamination.
