# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A targeted search of the frozen reconstruction for gold-style IDs and rubric terms (`S1`-`S9`, `F1`-`F40`, `R-F*`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `gold why`) returned no hits. The reconstruction uses candidate-plan feature names and grouped plan sections rather than gold IDs.

## Vocabulary Check

Sampled phrases and plan derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "neglect is quiet, not punished" | PLAN.md:446 says exactly that quietness is not punishment. | Plan-derived. |
| "Spinner culture" | PLAN.md:895 names spinner culture as first-frame risk. | Plan-derived. |
| "server is the only writer of canonical state" | PLAN.md:18 and PLAN.md:551-553 make server/worker single-writer explicit. | Plan-derived. |
| "Presence is evidence, not mere page-open engagement" | PLAN.md:40,333-335,401,873 reject tab-open and require three signals. | Paraphrase derived from plan. |
| "first-class aesthetic" | PLAN.md:658 uses this phrase for reduced motion. | Plan-derived. |
| "Privacy is a product boundary" | PLAN.md:42,103-108,805-814 establish technical privacy boundaries. | Paraphrase derived from plan. |
| "avoid aviary pings" | PLAN.md:30 and PLAN.md:370-372 constrain outbound notifications/email. | Plan-derived paraphrase. |
| "live-region rate limits" | PLAN.md:926 lists live-region rate limiter. | Plan-derived. |

I found no free use of rubric-side vocabulary such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, `gold why`, or `intent fidelity`.

## Heading Mirror Check

Reconstruction headings:

| Heading | Comparison |
|---|---|
| `## System-level intent` | Expected reconstruction template heading, not a gold-list mirror. |
| `## Per-feature whys` | Expected reconstruction template heading, not a gold-list mirror. |
| `### Scope and product surface` | Mirrors candidate PLAN scope/product grouping, not GOLD_WHYS sections. |
| `### Architecture, API, and data` | Mirrors candidate PLAN architecture/API/data grouping. |
| `### Simulation and sync` | Mirrors candidate PLAN simulation/sync grouping. |
| `### Frontend, audio, accessibility, performance, and rollout` | Mirrors candidate PLAN implementation sections. |

No heading is an exact or near-exact mirror of GOLD_WHYS section titles beyond the required reconstruction scaffold.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping pattern found. The reconstruction does not list S1-S9 or F1-F40, does not preserve gold order, and includes many plan-specific implementation items that are not gold-why anchors, such as Redis, object store, snapshot cache, TypeScript/Vite, build sequence, and feature flags. Its ordering follows the candidate plan's architecture and rollout structure rather than the gold list.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Presence is evidence, not mere page-open engagement."
   Plan support: PLAN.md:40 requires visible state, focus, and recent input; PLAN.md:333-335 defines pings only while all three hold; PLAN.md:401 gives a single ping 0s; PLAN.md:873 names no tab-open shortcut.
   Assessment: plan-derived paraphrase.

2. Reconstruction sentence: "Privacy is a product boundary, not only an implementation detail."
   Plan support: PLAN.md:42 forbids per-bird data in telemetry/training; PLAN.md:103-108 separates simulation DB and operational metrics; PLAN.md:805-814 excludes account/bird dimensions and relationship-reconstructing metrics.
   Assessment: plan-derived paraphrase.

3. Reconstruction sentence: "Audio should be procedural, recognizable, and alive rather than looped."
   Plan support: PLAN.md:701-710 defines procedural motif grammar and forbids looped buffers; PLAN.md:714-717 defines independent chorus schedulers; PLAN.md:883-885 names loopy/phasey audio risks and mitigations.
   Assessment: plan-derived paraphrase.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: it has no gold ID leakage, no rubric vocabulary, no gold-order 1:1 mapping, and its expressive phrases are traceable to the candidate plan. The few scoring misses are ordinary reconstruction compression, not contamination signatures.
