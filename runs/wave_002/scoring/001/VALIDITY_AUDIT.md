# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The reconstruction reads as plan-derived: no gold IDs, rubric vocabulary, or gold-list ordering leaked into the frozen text. It mirrors the PLAN section structure and uses PLAN phrases heavily. A few phrases are interpretive rather than exact copies, but they are grounded in nearby PLAN passages and do not look like contamination.

## ID leakage check

Search target: gold IDs and rubric markers such as `S1`, `F1`, `F40`, `R-F`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `gold`, and `rubric`.

Result: **no hits** in either RECONSTRUCTION.md or PLAN.md. There is no offending gold ID or external taxonomy in the reconstruction.

## Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "feels alive, notice never announce, charm from specificity, restraint" | PLAN line 105 uses this exact principle bundle. | Plan-derived. |
| "strict conjunction definition" | PLAN lines 24, 36, and 102 define/constrain the three-signal presence rule. | Plan-derived. |
| "tab open corruption" | PLAN line 102 says tab open must not count as real presence. | Plan-derived. |
| "not stripped fallback" | PLAN line 79 uses this phrase for reduced motion. | Plan-derived. |
| "first-class designed surfaces" | PLAN line 101 uses "first-class designed surfaces." | Plan-derived. |
| "edge snapshot/minimal critical path" | PLAN line 87 says edge snapshot and minimal critical path. | Plan-derived. |
| "voice containment" | PLAN does not use this exact phrase, but lines 15, 49, 90, and 103 establish the voice split. | Benign inference. |
| "canned feel" | PLAN line 100 uses canned feel in audio mitigation. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | This is a reconstruction convention, not a gold term, and appears only where the plan lacks rationale. | Not contamination. |

No rubric-side vocabulary such as "multi-layer recovery," "weight-3," "feature-level fidelity," or "gold why" appears in the reconstruction.

## Heading mirror check

RECONSTRUCTION headings: `System-level intent`, `Per-feature whys`, then `Scope`, `Architecture`, `Data model`, `API surface`, `Simulation engine design`, `Sync model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Performance budgets and observability`, `Rollout`, `Risks and mitigations`.

Gold headings: `System-level whys`, individual `S1`-`S9` headings, feature groups by source file, and complete feature list sections.

Assessment: **not a gold mirror.** The reconstruction mirrors the PLAN's implementation sections, not the gold-list S/F taxonomy. The only broad overlap is the required phase-2A heading `System-level intent`, which is expected by the workflow.

## 1:1 mapping suspect check

The reconstruction does **not** provide neat S1-S9 or F1-F40 items in gold order. It uses PLAN order: scope, architecture, data model, API, simulation, sync, frontend, audio, accessibility, performance, rollout, risks. Some gold-bearing features appear in roughly similar topical order because the PLAN itself was organized by product subsystem, but the reconstruction also includes non-gold implementation items and marks several items NOT RECOVERABLE FROM PLAN. This is not a 1:1 gold mapping.

## Plan-derivation spot check

1. Reconstruction: "The plan says presence = real interaction (three-signal conjunction) and defines presence as visibilityState=visible + window focus + pointer/key activity."
   - PLAN support: lines 24 and 36 state presence is the three-signal conjunction and enumerate visibility, focus, and activity.
   - Assessment: grounded.

2. Reconstruction: "The plan avoids wake-up animation and spinner, giving a quiet field on slow load so the aviary feels already alive rather than announced."
   - PLAN support: line 63 says load pulls a snapshot, places birds mid-action, renders immediately, and has no wake-up animation/no spinner/quiet field.
   - Assessment: grounded; "already alive" is an inference from the plan's final principles.

3. Reconstruction: "The plan says Ship with all listed surfaces; no deferred a11y/perf, so v1 is complete rather than staged around accessibility or performance."
   - PLAN support: line 95 says exactly that; lines 77-83 and 85-90 list the accessibility/performance surfaces.
   - Assessment: grounded.

## Verdict rationale

PASS. The reconstruction is cleanly derived from the PLAN's own sectioning and wording. There is no ID leakage, no rubric vocabulary, no gold-order mapping, and the articulate claims sampled above have direct PLAN support. The main scoring weaknesses are ordinary plan compression and rule-without-why loss, not contamination.
