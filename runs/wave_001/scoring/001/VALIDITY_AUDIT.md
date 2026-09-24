# VALIDITY_AUDIT - CARE run 001

## ID leakage

Mechanical scan for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` in the frozen reconstruction found no hits. Because no IDs appeared, there were no PLAN cross-reference leakage hits.

## Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `observational and quiet` | PLAN section 1: "Keep the product observational and quiet." | Plan-derived; not rubric-side vocabulary. |
| `presentation projection, not a second model` | PLAN section 2: "presentation projection, not a copy of the server's model." | Plan-derived; not rubric-side vocabulary. |
| `server owns all canonical state` | PLAN section 2: "The server owns all canonical state." | Plan-derived; not rubric-side vocabulary. |
| `naturalist, lowercase, present-tense` | PLAN section 1 uses the same phrase for product prose. | Plan-derived; not rubric-side vocabulary. |
| `revocable, read-only visitor credential` | PLAN section 1: invite redemption becomes a "revocable, read-only visitor credential." | Plan-derived; not rubric-side vocabulary. |
| `low-pass filter over capped, positive daily inputs` | PLAN section 5 uses this drift description. | Plan-derived; not rubric-side vocabulary. |
| `age gates, never visit-count rewards` | PLAN section 1 uses the same age-gate wording. | Plan-derived; not rubric-side vocabulary. |
| `release gates` | PLAN section 9 makes accessibility paths release gates. | Plan-derived; not rubric-side vocabulary. |
| `first visual frame must not wait` | PLAN section 2 has this exact first-frame requirement. | Plan-derived; not rubric-side vocabulary. |
| `no endpoint that sets a bird vector, mood, perch, or simulation timestamp` | PLAN section 4 states this directly. | Plan-derived; not rubric-side vocabulary. |

I also scanned for scorer-side terms (`gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, `intent fidelity`) and found no hits.

## Heading mirror

Reconstruction headings are: `System-level intent`, `Per-feature whys`, then `1. Scope and product invariants` through `11. Principal risks and controls`. The numbered headings mirror the PLAN sections, not the gold-list files or S/F target order. There is no close mirror of `System-level whys`, `Feature-level whys`, or the F1-F40 target sequence.

## 1:1 mapping suspect

No 1:1 mapping to the gold targets is visible. The reconstruction does not enumerate S1-S9 or F1-F40, and it does not attempt exactly 49 gold-aligned entries. It follows the plan order and includes many implementation features outside the gold-why set. This is not suspect.

## Plan-derivation spot check

| Reconstruction sentence | PLAN support | Judgment |
|---|---|---|
| "Treat the browser scene as a presentation projection, not a second model." | PLAN section 2 says the snapshot is "a presentation projection, not a copy of the server's model," and the renderer consumes immutable cues without mutating canonical state. | Supported. |
| "Make change slow, calibrated, and never punitive." | PLAN section 5 describes low-pass capped positive daily inputs, week-one/week-three calibration, and never decrementing a trait for neglect. | Supported. |
| "Ship accessibility with v1 rather than treating it as a later layer." | PLAN section 9 starts "Ship accessibility with v1" and section 10 sequences accessibility settings/narration before release. | Supported. |

## Verdict

PASS. The reconstruction reads as plan-derived: it uses the plan's headings and vocabulary, shows no gold IDs, contains no rubric-side terminology, and the spot-checked articulate sentences are all grounded in the PLAN. Scores can be treated as valid for this run.

