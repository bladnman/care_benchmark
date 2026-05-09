# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no ID leakage found. A targeted search for gold-side identifiers and rubric vocabulary in `RECONSTRUCTION.md` found no `S1`-`S9`, `F1`-`F40`, `R-F..`, `gold why`, `intent fidelity`, `feature-level fidelity`, `multi-layer recovery`, `weight-3`, `load-bearing`, or `rubric` hits. The reconstruction uses required phase-2A labels such as `NOT RECOVERABLE FROM PLAN`, but no gold IDs.

## Vocabulary Check

Sampled phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `sole canonical writer` | PLAN service shape and sync model use the same phrase for the Simulation Service. | Plan-derived. |
| `No client-side state is ever authoritative` | PLAN client/server split says exactly this for personality or mood. | Plan-derived. |
| `aviary age (not interaction count, not paid tier)` | PLAN scope and rollout use this wording for unlocks. | Plan-derived. |
| `not a stripped fallback` | PLAN reduced-motion scope says reduced motion is not a stripped fallback. | Plan-derived. |
| `preserve presence without vestibular risk` | PLAN reduced-motion bird rigs use this phrasing. | Plan-derived. |
| `affective quality of the accessible surface` | PLAN accessibility audit uses this phrase. | Plan-derived. |
| `No spinner. No Loading text.` | PLAN quiet-field loading state says no spinner and no Loading text. | Plan-derived. |
| `without code deploy` | PLAN config artifacts/unlock schedule use this phrase. | Plan-derived. |
| `read-mostly, HTTP/2-friendly` | PLAN ambiguity decision for SSE uses this phrase. | Plan-derived. |
| `telemetry is forbidden from joining on account UUID` | PLAN privacy boundary says telemetry pipelines are forbidden from joining on account UUID. | Plan-derived. |

No sampled phrase looks imported from the gold list rather than the plan. A few phrases are synthesized summaries, but they are ordinary paraphrases of nearby PLAN language.

## Heading Mirror Check

`RECONSTRUCTION.md` headings are:

| Heading | Comparison |
|---|---|
| `## System-level intent` | Required reconstruction structure, not a gold-list title. |
| `## Per-feature whys` | Required reconstruction structure, not a gold-list title. |
| `### Scope and user-facing surfaces` | Mirrors PLAN sections/scope, not GOLD_WHYS group titles. |
| `### Architecture, data, and API` | Mirrors PLAN architecture/data/API sections. |
| `### Simulation and sync behavior` | Mirrors PLAN simulation/sync sections. |
| `### Frontend rendering and audio` | Mirrors PLAN frontend/audio sections. |
| `### Accessibility, performance, rollout, and risk controls` | Mirrors PLAN accessibility/performance/rollout/risk sections. |

No heading mirrors `GOLD_WHYS.md` system or feature ID headings.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not present exactly 49 corresponding targets in gold order. Its per-feature section follows the candidate PLAN's organization and includes many plan features that are not gold-why rows, plus several explicit `NOT RECOVERABLE FROM PLAN` entries.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `The plan repeatedly says the server is the sole canonical writer... No client-side state is ever authoritative...` | PLAN `Client/server split` and `Sync Model` state server ownership and server-only personality writes. | Supported. |
| `Screen-reader narration uses naturalist prose; reduced-motion is not a stripped fallback...` | PLAN `Accessibility Surfaces` specifies naturalist prose narration and cross-fade reduced-motion rendering. | Supported. |
| `Internal references use synthetic UUIDs, email is encrypted and stored once... telemetry is forbidden from joining on account UUID...` | PLAN `Account record`, privacy boundary, and telemetry sections specify UUID-only references, encrypted email, and IAM restrictions. | Supported. |

## Verdict: PASS

The reconstruction reads as plan-derived. I found no gold ID leakage, no rubric-side vocabulary usage, no gold-heading mirror, and no neat 1:1 mapping to the gold targets. The reconstruction is sometimes articulate, but the articulate sentences spot-check cleanly against PLAN language.
