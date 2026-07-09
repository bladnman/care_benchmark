# VALIDITY_AUDIT - run 001

## Gold ID Leakage Check

Verdict: no leakage found. A search of `RECONSTRUCTION.md` for gold IDs and related forms (`F1`-`F40`, `S1`-`S9`, `R-Fxx`) returned no hits. The same search against `PLAN.md` returned no hits, so there are no reconstruction-only ID leaks to cross-reference.

## Vocabulary Check

Sampled phrases from `RECONSTRUCTION.md` and their plan grounding:

| Reconstruction phrase | PLAN support | Finding |
|---|---|---|
| "continuing place, not progression" | PLAN line 5 uses the same phrase. | plan-derived |
| "The server is the only writer" | PLAN line 9 uses the same phrase. | plan-derived |
| "visible open tab alone never creates presence time" | PLAN line 10 uses the same phrase. | plan-derived |
| "durable opaque ID independent of name/species" | PLAN line 12 uses the same phrase. | plan-derived |
| "naturalist, lowercase, specific, and observational" | PLAN line 14 uses the same phrase. | plan-derived |
| "Per-account interaction data remains inside the simulation domain" | PLAN line 15 uses the same phrase. | plan-derived |
| "never show a spinner, static app shell, fade-in, or wake-up sequence" | PLAN line 130 uses the same phrase. | plan-derived |
| "same normalized snapshot" | PLAN line 154 supports the accessibility wording. | plan-derived |

Suspicious rubric/gold-side vocabulary check: searches for `multi-layer`, `feature-level fidelity`, `intent fidelity`, `weight-3`, `load-bearing`, `gold why`, and `rule_without_why` returned no hits in the reconstruction.

## Heading Mirror Check

`RECONSTRUCTION.md` headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Delivery boundary and product invariants`
- `### Service architecture and responsibilities`
- `### Canonical data model`
- `### Public API and event contracts`
- `### Simulation engine`
- `### Sync, integrity, and lifecycle`
- `### Frontend scene and interaction pipeline`
- `### Audio and accessibility surfaces`
- `### Performance, reliability, privacy, and observability`
- `### Implementation sequence and rollout`
- `### Validation matrix and principal risks`

The `###` headings mirror the plan's section structure, not the gold-list headings (`System-level whys`, `Feature-level whys`, PRD file group headings, or F/S IDs). The two required reconstruction top-level headings are expected by the phase-2A format and are not leakage evidence.

## 1:1 Mapping Suspect Check

No suspect 1:1 mapping. The reconstruction does not enumerate S1-S9 or F1-F40 and does not proceed in gold-list order. Instead it expands the plan section-by-section, with many more than 49 bullets and plan-specific headings. Some bullets correspond naturally to gold features because the plan itself is comprehensive, but the artifact is not a neat gold-target mapping.

## Plan-Derivation Spot Check

1. Reconstruction: "The opening says Pocket Aviary is 'a relationship with a continuing place, not a progression system.'"
   PLAN support: line 5 contains the quoted phrase in the opening delivery boundary paragraph.
   Finding: plan-derived.

2. Reconstruction: "Qualifying presence requires visible document, focused window, and recent pointer/key activity."
   PLAN support: line 10 says qualifying host presence requires visible document, focused window, and pointer/key activity.
   Finding: plan-derived.

3. Reconstruction: "Metric schema validation must reject bird IDs, account IDs, emails, names, events, payloads, state versions tied to accounts, and notebook prose."
   PLAN support: line 171 lists the same metric schema rejection categories.
   Finding: plan-derived.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses plan vocabulary, mirrors plan headings, avoids gold IDs and rubric terminology, and its most articulate sentences are directly supportable from the plan. The run's scores should be treated as valid under the phase-2B contamination audit.
