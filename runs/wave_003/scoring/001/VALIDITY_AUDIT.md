# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found.

Mechanical search of the frozen reconstruction for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` returned no hits. The reconstruction does not use gold IDs or rebuild IDs.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "already alive" | PLAN.md:367 uses the "already alive" illusion framing. | Plan-derived. |
| "quietly continuous" | PLAN.md:5 says the aviary continues without the viewer; PLAN.md:204-205 uses quiet field/no spinner. | Plan-derived paraphrase. |
| "server is the only writer" | PLAN.md:43 states this exactly. | Plan-derived. |
| "stripped fallback" | PLAN.md:361 says accessibility may become a stripped fallback. | Plan-derived. |
| "observational and naturalist" | PLAN.md:258 uses the phrase for narration. | Plan-derived. |
| "relationship dynamics" | PLAN.md:305 excludes dashboards summarizing individual relationship dynamics. | Plan-derived. |
| "divergence should be structurally prevented" | PLAN.md:192 states this. | Plan-derived. |
| "per-bird recognizability" | PLAN.md:243 and 357 preserve this idea. | Plan-derived. |
| "PRD-level product contract" | PLAN.md:385 contains the same phrase. | Not leakage; copied from PLAN. |
| "emergent-feeling behavior" | PLAN.md:167-169 and 173-177 support emergence from bounded behavior, though not exact wording. | Mild paraphrase, not rubric/gold vocabulary. |

Suspicious scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, and `intent fidelity` do not appear. The lone `PRD-level product contract` phrase is present in the PLAN itself.

## Heading Mirror Check

| Reconstruction heading | Gold/template comparison | Assessment |
|---|---|---|
| `## System-level intent` | Matches required phase-2A output heading, not a gold-list section title leak. | OK. |
| `## Per-feature whys` | Matches required phase-2A output heading, not the hidden F1-F40 list structure. | OK. |

The reconstruction does not mirror gold section headings such as specific S or F titles.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction has 10 system-level principles and 60 per-feature items, not 9 system whys and 40 feature whys. The per-feature list follows the PLAN's implementation order (browser/account/simulation/data/API/render/audio/accessibility/rollout/testing), not the gold F1-F40 order. It also includes many features that have no gold feature-level why, such as cache, email integration, API surfaces, and engineering tactics.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "Canonical state belongs on the server, while the client is a disposable renderer and event emitter." | PLAN.md:43-44 says the server is the only writer and clients only emit events/request snapshots; PLAN.md:185 says client state is disposable. | Supported. |
| "Privacy and non-analytics boundaries are part of the product shape, not a later hardening pass." | PLAN.md:14 excludes per-account relationship data; PLAN.md:293-305 restricts telemetry to aggregates and excludes per-bird/per-account histories; PLAN.md:371-375 names privacy-boundary mitigation. | Supported. |
| "Accessibility is first-class and must keep the charm of the main product." | PLAN.md:13 lists first-class accessibility surfaces; PLAN.md:270-271 preserves captions/narration continuity; PLAN.md:361-363 says accessibility must not be a stripped fallback and is launch-blocking. | Supported. |

## Verdict: PASS

The reconstruction reads as plan-derived. It uses the required phase-2A headings, follows the PLAN's own structure rather than the gold taxonomy, contains no gold IDs, and its load-bearing vocabulary is either directly present in the PLAN or a close paraphrase of PLAN passages. No contamination signature is significant enough to flag the run.
