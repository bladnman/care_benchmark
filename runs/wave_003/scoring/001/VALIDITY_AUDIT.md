# VALIDITY_AUDIT — CARE run 001

## Verdict

**PASS.** No significant contamination signatures were found. The reconstruction reads as derived from PLAN.md: it uses the plan's headings, phrases, and implementation clusters; it does not leak gold IDs, rubric vocabulary, or a neat S1-S9/F1-F40 mapping.

## ID leakage

Search target: gold IDs and rubric-ish identifiers such as `S1`, `F1`, `F40`, `R-F01`, `weight-3`, `multi-layer`, and `intent fidelity`.

- Result: no hits in RECONSTRUCTION.md.
- Cross-reference: because no offending IDs appear in RECONSTRUCTION.md, there is no PLAN absence/presence leakage hit to adjudicate.

## Vocabulary Check

Sampled load-bearing phrases from RECONSTRUCTION.md and checked against PLAN.md:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "no surface anywhere observes the user's behavior back at them" | PLAN §1.2 uses the same sentence. | Plan-derived. |
| "notice, never announce" | PLAN §12 risk calls out "Notice, never announce" erosion. | Plan-derived. |
| "zero drift" | PLAN §1.3 voice governance says "zero drift." | Plan-derived. |
| "what is true" / "how it looks and sounds" | PLAN §2.2 boundary rule uses these phrases. | Plan-derived. |
| "THE bird; never regenerated" | PLAN §3.1 birds table comment uses this phrase. | Plan-derived. |
| "neither penalized" | PLAN §5.2 says settle and tab-close are neither penalized. | Plan-derived. |
| "no account dimension anywhere" | PLAN §3 and §10.4 use this telemetry boundary. | Plan-derived. |
| "boring with teeth" | PLAN §2.1 uses this exact infrastructure framing. | Plan-derived. |
| "engineered, never canned" | PLAN §5.6 names the return-greeting this way. | Plan-derived. |
| "cheap version" | PLAN §9 rejects the cheap accessibility version. | Plan-derived. |

I found no reconstruction vocabulary such as rubric-side "multi-layer recovery," "feature-level fidelity," "weight-3," or "intent fidelity."

## Heading Mirror

RECONSTRUCTION.md headings:

- `## System-level intent`
- `## Per-feature whys`

These are the expected phase-2A output headings, not gold-list section titles. The reconstruction does not mirror gold headings such as "System-level whys," "Feature-level whys," or the S/F IDs.

## 1:1 Mapping Suspect

No 1:1 mapping pattern was found. The reconstruction has 12 system bullets and many plan-clustered feature bullets, not exactly 9 system items and 40 feature items. Its feature order follows the PLAN structure: scope, architecture, data/API, simulation, sync, frontend, audio, accessibility, performance, rollout. Some gold targets are absent or marked `NOT RECOVERABLE FROM PLAN`, which is the opposite of a suspiciously neat mapping.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "State truth belongs to the server; the client realizes it."
   - PLAN support: §2.2 says server owns personality, mood, drift, perch selection, weather, notebook, presence accounting, adoption offers, and persistence; the server sends "what is true" while the client decides "how it looks and sounds this exact frame."
   - Assessment: supported.

2. Reconstruction sentence: "Performance is affective."
   - PLAN support: §10.2 says the first-bird metric is treated as an affective metric and that below 500ms the aviary feels already there.
   - Assessment: supported.

3. Reconstruction sentence: "Calibration replaces per-user behavioral analytics."
   - PLAN support: §11.4 describes synthetic in-house accounts and says the privacy boundary forces this method; §10.4 excludes per-account interaction telemetry.
   - Assessment: supported.

## Verdict Rationale

PASS: the reconstruction is broad and articulate, but its wording and organization are traceable to PLAN.md. There are no gold IDs, no rubric vocabulary, no heading mirror, no exact 49-item gold mapping, and no spot-check sentence requiring gold-side knowledge.
