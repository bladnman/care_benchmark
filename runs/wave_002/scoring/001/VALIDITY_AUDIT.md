# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A mechanical search for gold-style IDs (F1-F40, S1-S9, R-Fxx, R-Sxx) in the frozen reconstruction returned no hits. RECONSTRUCTION uses no gold IDs or external taxonomy labels.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION and PLAN grounding:

| Reconstruction phrase | PLAN grounding | Finding |
|---|---|---|
| "renderer and an event forwarder" | PLAN section 2.3 uses the same phrase. | plan-derived |
| "only writer of personality, mood, and notebook state" | PLAN section 7.1 uses the same phrase. | plan-derived |
| "quiet field" | PLAN sections 8.6 and 8.7. | plan-derived |
| "of the aviary, not of the user's behavior" | PLAN section 14.5. | plan-derived |
| "designed surface, not a fallback" | PLAN sections 8.5 and 10.4. | plan-derived |
| "Not v1.1. Not a retrofitted fix." | PLAN section 10.6. | plan-derived |
| "small social system rather than parallel independent birds" | PLAN section 6.7. | plan-derived |
| "architectural absence makes the feature's reappearance harder" | PLAN section 11.3. | plan-derived |
| "CALIBRATION" | PLAN section 1.3 and section 16. | plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Required phase-2A output convention, not gold-side vocabulary. | expected |

No scorer-side terms such as gold, rubric, weight-3, multi-layer recovery, feature-level fidelity, system-level fidelity, or intent fidelity appeared in RECONSTRUCTION.

## Heading Mirror

RECONSTRUCTION headings:

- ## System-level intent
- ## Per-feature whys
- ### Scope and product boundaries
- ### Architecture and storage
- ### Data model
- ### API surface
- ### Snapshot schema
- ### Simulation engine
- ### Sync model and error voice
- ### Frontend rendering pipeline
- ### Audio pipeline
- ### Accessibility surfaces
- ### Performance, observability, and privacy
- ### Rollout and implementation order

The first two headings are required by the phase-2A prompt. The remaining headings mirror PLAN section organization, not GOLD_WHYS headings. There is no near-exact mirror of S1-S9/F1-F40 gold titles.

## 1:1 Mapping Suspect

Verdict: not suspect. RECONSTRUCTION does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not give a neat 49-item list. Its system section has 10 plan-derived principles, and its per-feature section follows plan areas with many more than 40 items, including items outside the gold-why list. This is consistent with the PLAN structure.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "The plan repeatedly makes the browser a 'renderer and an event forwarder' and says the server is 'the only writer of personality, mood, and notebook state.'" | PLAN sections 2.3 and 7.1 contain those phrases. | grounded |
| "The plan excludes 'achievements, streaks, levels, scores, badges, counters' and any 'ping surface,' then reinforces the same intent through a 'quiet field' loading state." | PLAN sections 1.2 and 8.6 contain those phrases/rules. | grounded |
| "Accessibility is in v1 scope as 'first-class designed surfaces'; reduced motion is 'a designed surface, not a fallback'; and the plan says these surfaces ship 'Not v1.1. Not a retrofitted fix.'" | PLAN sections 1.1, 8.5, 10.4, and 10.6 support the sentence. | grounded |

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It uses the PLAN's headings and vocabulary, contains no gold-ID leakage or scorer-side rubric vocabulary, and its articulate claims can be grounded in PLAN text. I found no significant contamination signatures.
