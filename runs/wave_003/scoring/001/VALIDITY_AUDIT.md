# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict: no leakage detected.

A mechanical search for gold-style IDs (`F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, `R-S[0-9]+`) found no hits in `RECONSTRUCTION.md` and no corresponding hits in `PLAN.md`. The reconstruction does not use gold IDs or an external numbered taxonomy.

## Vocabulary check

Sampled phrases from `RECONSTRUCTION.md` and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Ambient aviary, not a game" | PLAN §1.2 excludes gamification, Tamagotchi mechanics, and social network surfaces | Plan-derived |
| "watching without moving is the product" | PLAN §1.3 uses this exact rationale for the 4-minute presence window | Plan-derived |
| "Canonical state belongs on the server" | PLAN §2.2 assigns canonical state and vector writes to the server | Plan-derived |
| "Relationship becomes optimization" | PLAN §12 risk for personality vector exposure uses this exact phrase | Plan-derived |
| "lowercase present-tense prose" | PLAN §5.7 notebook generation uses the phrase | Plan-derived |
| "Accessibility ships as part of the product" | PLAN §1.1 includes a11y; §12 warns retrofit means broken product | Plan-derived |
| "Product identity collapse" | PLAN §12 gamification creep impact | Plan-derived |
| "co-presence scope creep" | PLAN §12 visit risk | Plan-derived |
| "aggregate health without collecting intimate state" | PLAN §10.3 aggregate-only observability and never-collect list | Plan-derived |

Suspicious scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, and `intent fidelity` do not appear. `PRD` appears several times, but every use is copied from or directly supported by PLAN text that itself says "per PRD" or "PRD sparsity requirement".

## Heading mirror

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope and v1 deliverables`
- `### Defensible calls on ambiguity`
- `### Architecture and render boundary`
- `### Data model`
- `### API surface`
- `### Simulation engine`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance and observability`
- `### Rollout and workstreams`

The first two top-level headings were required by the phase-2A prompt. The remaining headings mirror PLAN.md sections (Scope, Architecture, Data Model, API Surface, Simulation Engine, Sync Model, Frontend Rendering Pipeline, Audio Pipeline, Accessibility Surfaces, Performance Budgets and Observability, Rollout), not GOLD_WHYS.md titles. They do not mirror S1-S9 or F1-F40 headings.

## 1:1 mapping suspect

No 1:1 gold mapping detected. The reconstruction has 11 system-level principles and a plan-section-oriented feature list. It does not enumerate S1-S9 or F1-F40, does not follow the gold order, and includes many implementation details that are not gold why anchors. This structure is consistent with the PLAN's own sections.

## Plan-derivation spot check

1. Reconstruction: "The 4-minute presence window 'leans long' because 'watching without moving is the product.'"
   PLAN support: §1.3 presence activity window rationale says exactly that.

2. Reconstruction: "Canonical state belongs on the server."
   PLAN support: §2.2 assigns canonical state and personality vector writes to the server; §6.1 shows Event Log -> Tick -> Canonical DB.

3. Reconstruction: "Visits are 'Opt-in,' 'read-only ambient view,' and 'off by default.'"
   PLAN support: §1.1 social deliverable says opt-in read-only visits are off by default; §7.8 disables interaction in visitor mode.

All three articulate plan content rather than held-out gold phrasing.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no gold ID leakage, no scorer-side rubric vocabulary, headings follow the PLAN rather than GOLD_WHYS, and the per-feature organization is not a neat 1:1 mapping to the hidden target list. Minor PRD references are copied from the PLAN itself and are not contamination signs.
