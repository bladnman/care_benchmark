# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: PASS. A mechanical search of the frozen reconstruction found no held-out gold IDs or external taxonomy tokens such as `F1`, `F40`, `S1`, `S9`, `R-F01`, or `R-S01`. There are therefore no ID hits to cross-reference against the PLAN.

## Vocabulary Check

Sampled phrases from `RECONSTRUCTION.md` and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "one browser-based, private aviary" | PLAN.md:5 uses the same phrase | plan-derived |
| "never a publicly cacheable account response" | PLAN.md:13 uses the same phrase | plan-derived |
| "database is the sole source" | PLAN.md:15 uses the same phrase | plan-derived |
| "simulation worker is the only writer" | PLAN.md:15 uses the same phrase | plan-derived |
| "lowercase, present-tense, specific naturalist observations" | PLAN.md:7 uses the same phrase | plan-derived |
| "honesty mechanism, not antifraud surveillance" | PLAN.md:56 uses the same phrase | plan-derived |
| "soft quiet field" | PLAN.md:76 uses the same phrase | plan-derived |
| "aggregate operational counters/histograms" | PLAN.md:96 uses the same phrase | plan-derived |

No scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity` appeared. `NOT RECOVERABLE FROM PLAN` appears several times, but that phrase is explicitly required by the reconstructor prompt and is not a contamination signal.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Product contract and decisions`
- `### 2. System boundaries`
- `### 3. Persistent model and retention`
- `### 4. API and authorization`
- `### 5. Presence, event ordering, and simulation`
- `### 6. Sync and failure behavior`
- `### 7. Client rendering and audio`
- `### 8. Accessibility and copy acceptance`
- `### 9. Budgets, instrumentation, and rollout`
- `### 10. Verification and risk controls`

These mirror the PLAN's own numbered section headings, not the held-out gold list section titles. No near-exact mirror of `GOLD_WHYS.md` sections was found.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction follows the plan's ten-section order and natural feature grouping. It does not contain S1-S9 or F1-F40 labels, does not enumerate exactly 49 gold targets, and includes plan-specific operational rows that are not gold why rows. This is consistent with a plan-derived reconstruction rather than a gold-list-shaped reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "Attention is qualified without becoming surveillance." | PLAN.md:54 defines visible/focus/activity presence; PLAN.md:56 calls interval union "an honesty mechanism, not antifraud surveillance" and stores no coordinates/keystrokes. | grounded |
| "The first frame should feel ongoing, calm, and resilient." | PLAN.md:76 draws the first bird before hydration with elapsed animation phase, no spinner/fade/static; PLAN.md:72 keeps last frame during transient fetch; PLAN.md:76 uses a soft quiet field on lag. | grounded |
| "Operational data collection must remain privacy-minimal." | PLAN.md:96 allows only aggregate counters/histograms and forbids account/bird/event dimensions, relationship data, and telemetry/model-training access to private events. | grounded |

## Verdict: PASS

The reconstruction shows no gold ID leakage, no rubric vocabulary, no gold-heading mirror, and no neat one-to-one mapping to the held-out why list. Its structure and vocabulary are strongly tied to the PLAN, with several honest `NOT RECOVERABLE FROM PLAN` entries. Scores can be treated as valid for this run.
