# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict: no leakage found.

Searches for gold-style IDs and rubric terms in the frozen reconstruction found no `S1`-`S9`, `F1`-`F40`, `R-Fxx`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `gold`, or `rubric` hits. The PLAN likewise does not contain those gold IDs. There are therefore no offending ID sentences to cross-reference.

## Vocabulary check

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "the server is the only writer of canonical aviary state" | PLAN opening invariant | Plan-derived |
| "a place that was already running" | PLAN opening invariant | Plan-derived |
| "structural enforcement" | PLAN Scope closing paragraph | Plan-derived |
| "never-expose-the-vector rule" | PLAN snapshot boundary | Plan-derived |
| "quieter, not punished" | PLAN drift expression section | Plan-derived |
| "affective-perf bridge metric" | PLAN day-one instrumentation | Plan-derived |
| "watching without moving is the product" | PLAN presence window | Plan-derived |
| "no-lost-drift guarantee" | PLAN Architecture and Simulation | Plan-derived |
| "privacy boundary as architecture" | PLAN telemetry section heading | Plan-derived |
| "cheap version" | PLAN accessibility risks | Plan-derived |

No sampled phrase reads like rubric-side language. The reconstruction's strongest phrases are directly traceable to PLAN wording.

## Heading mirror

The reconstruction uses the required top-level headings `System-level intent` and `Per-feature whys`, then mirrors the PLAN's numbered implementation sections: Scope, Architecture, Data model, API surface, Simulation engine design, Presence/greeting/session semantics, Sync model, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Performance budgets and observability, Rollout, Risks and mitigations, and Open calls.

This is a PLAN mirror, not a GOLD_WHYS mirror. It does not echo gold section titles such as `System-level whys`, `Feature-level whys`, or individual S/F labels.

## 1:1 mapping suspect

Verdict: not suspect.

The reconstruction does not create neat S1-S9 or F1-F40 rows, and it is not ordered like GOLD_WHYS. It contains many more plan-derived bullets than the 49 gold targets and follows the plan's implementation section order. Some bullets naturally overlap gold targets because both derive from the same product, but the structure is not a 1:1 gold mapping.

## Plan-derivation spot check

1. Reconstruction: "Product boundaries are enforced structurally, not only by policy." PLAN support: Scope says banned items get "structural enforcement" and later implements absence of day-rollup tables, no trait-value API, no visit events endpoint, copy lint, schema review, and typed telemetry.

2. Reconstruction: "The plan repeatedly protects the never-expose-the-vector rule." PLAN support: the snapshot boundary says personality is not in the snapshot and the numbers physically never leave the server; the API section states no endpoint returns trait values.

3. Reconstruction: "Privacy is an architecture boundary." PLAN support: the telemetry section separates simulation/event data from telemetry, forbids account and bird dimensions, and gives the analytics warehouse no simulation database credentials.

All three articulate PLAN material rather than unexplained gold-list phrasing.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold ID leakage, no rubric vocabulary, plan-section heading structure, no 1:1 gold mapping, and spot-checked articulate sentences are directly supported by PLAN passages. The run's scores do not need a contamination caveat.
