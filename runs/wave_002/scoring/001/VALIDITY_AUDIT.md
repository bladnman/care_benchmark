# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS.

Mechanical search of the frozen reconstruction found no gold IDs or rebuild IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. There are no offending ID sentences to cross-check against the PLAN.

## Vocabulary Check

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "load-bearing architecture" | PLAN has "Privacy architecture (load-bearing)" and the decision rule "sole simulation writer." | Plan-derived phrasing; not a leakage hit. |
| "sole simulation writer" | PLAN decision rule says "keeps the server as sole simulation writer." | Directly plan-derived. |
| "Same product, different register - not a stripped mode" | PLAN accessibility section says the same phrase with an em dash. | Directly plan-derived. |
| "not 24 hours" | PLAN presence calibration says 20 minutes/day should not become "24 hours." | Directly plan-derived. |
| "no LWW" | PLAN sync section says "no LWW." | Directly plan-derived. |
| "relationship reconstruction" | PLAN observability section says product analytics should not become "relationship reconstruction." | Directly plan-derived. |
| "breaks the product thesis" | PLAN first-frame risk says spinner/fade breaks the thesis. | Directly plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | Required by the phase-2A prompt, not gold vocabulary. | Expected reconstruction marker. |

No rubric-side terms such as "gold," "weight-3," "multi-layer recovery," "feature-level fidelity," or "system-level fidelity" appeared in the reconstruction.

## Heading Mirror

RECONSTRUCTION headings:

| Heading | Gold/rubric comparison | Assessment |
|---|---|---|
| `## System-level intent` | Similar only to the phase-2A required output section; gold uses "System-level whys." | Not suspicious. |
| `## Per-feature whys` | Similar only to the phase-2A required output section; gold uses "Feature-level whys." | Not suspicious. |

No `###` headings or gold-list grouping headings appear.

## 1:1 Mapping Suspect

Verdict: PASS.

The reconstruction does not create a neat S1-S9 or F1-F40 mapping. It follows the PLAN's structure: system principles first, then PLAN groupings such as Scope, Architecture, Data model, API surface, Simulation engine, Sync model, Frontend rendering, Audio pipeline, Accessibility surfaces, Performance/rollout, and Risks. It also includes more than forty per-feature bullets and several `NOT RECOVERABLE FROM PLAN` markers, which is consistent with the phase-2A prompt rather than a leaked gold taxonomy.

## Plan-Derivation Spot Check

1. Reconstruction: "Server-canonical simulation is the load-bearing architecture." PLAN support: product summary says "Server is the only writer of personality," the decision rule keeps the server as sole simulation writer, API never writes personality vectors/moods, and sync says laptop and phone are snapshot readers. Supported.

2. Reconstruction: "Accessibility is a designed v1 surface, not a later accommodation." PLAN support: the decision rule says accessibility ships as a designed surface on day one; scope includes naturalist narration, designed reduced motion, captions, AA chrome, and keyboard; rollout says reduced motion cannot slip. Supported.

3. Reconstruction: "Performance budgets protect the product thesis." PLAN support: the performance section defines first-bird and bundle gates; first-frame risk says spinner/fade breaks the product thesis; definition of done requires two birds mid-motion in under 500ms and no spinner. Supported.

## Verdict

PASS. The reconstruction reads as plan-derived: no ID leakage, no held-out taxonomy, no gold/rubric vocabulary, and the articulate load-bearing sentences can be traced to PLAN wording or structure. The only mild concern is use of the phrase "load-bearing," but that term appears in the PLAN itself and is not sufficient to flag contamination.
