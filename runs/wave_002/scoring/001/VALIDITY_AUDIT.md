# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found.

A targeted search of the frozen reconstruction found no gold IDs or rubric IDs such as `S1`, `S9`, `F1`, `F40`, `R-F01`, `gold why`, or `intent fidelity`. The same targeted ID/rubric vocabulary search against the plan also produced no hits, so there were no reconstruction-only ID hits to cross-reference.

## Vocabulary Check

Sampled reconstruction phrases against PLAN.md:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "executable engineering plan" | PLAN opening sentence uses the same phrase. | plan-derived |
| "Absence is architectural" | PLAN says nothing in schema/API/telemetry computes comparative stats and calls absence structural. | plan-derived |
| "One canonical aviary, one writer" | PLAN scope and sync model repeatedly state one canonical aviary and server-only writer. | plan-derived |
| "logically continuous" hot/lazy tick | PLAN section 2.3 uses this exact tick model. | plan-derived |
| `catchUp(S, E, T) === composeTicks(S, E, T)` | PLAN section 2.3 contains the invariant exactly. | plan-derived |
| "one human has one attention" | PLAN presence section uses this phrase for the multi-device clamp. | plan-derived |
| "same product, not two products glued together" | PLAN prose-system section uses this exact phrase. | plan-derived |
| "network topology and schema review, not policy text" | PLAN architecture/privacy section uses this phrase. | plan-derived |
| "operational fallback, not product experimentation" | PLAN launch section says feature flags are for fallback, not A/B. | plan-derived |

No sampled phrase read like rubric-side language. Terms such as "multi-layer recovery," "feature-level fidelity," "weight-3," "intent fidelity," or "gold why" did not appear in the reconstruction.

## Heading Mirror

Reconstruction headings:

| Reconstruction heading | Closest source shape | Assessment |
|---|---|---|
| `## System-level intent` | Required reconstruction structure, not a gold title. | acceptable |
| `## Per-feature whys` | Required reconstruction structure, not a gold title. | acceptable |
| `### Scope` | Mirrors PLAN section 1. | plan-derived |
| `### Architecture` | Mirrors PLAN section 2. | plan-derived |
| `### Data model and API surface` | Merges PLAN sections 3 and 4. | plan-derived |
| `### Simulation engine` | Mirrors PLAN section 5. | plan-derived |
| `### Sync model` | Mirrors PLAN section 6. | plan-derived |
| `### Frontend rendering pipeline` | Mirrors PLAN section 7. | plan-derived |
| `### Audio pipeline` | Mirrors PLAN section 8. | plan-derived |
| `### Accessibility surfaces` | Mirrors PLAN section 9. | plan-derived |
| `### Performance, observability, and rollout` | Merges PLAN sections 10 and 11. | plan-derived |
| `### Risk-shaped implementation features` | Mirrors PLAN section 12. | plan-derived |

The headings mirror the plan outline, not the gold-list section titles or S/F taxonomy.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not proceed in the gold-list order. Its per-feature section is grouped by the plan's implementation sections and contains many plan-specific items that are not gold targets, such as Postmark, Redis, Cloudflare KV, Preact, AudioWorklet voice pools, and rollout gates. That shape is consistent with derivation from PLAN.md rather than from GOLD_WHYS.md.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "A bird's being is server state; a bird's appearance this frame is client interpolation." | PLAN section 2.2 contains the sentence almost verbatim under render pipeline boundary. | supported |
| "The plan wants the user-observable property that 'the aviary you return to is the aviary that has been running,' while avoiding useless work for dormant aviaries." | PLAN section 2.3 states the same property and the hot/lazy tick reason. | supported |
| "The flag checked at the query layer gives calibration one sanctioned exception without eroding the broader privacy boundary." | PLAN section 11.2 defines `calibration_consented` as the only accounts whose per-bird state a dashboard may read and says the flag is checked at the query layer. | supported |

## Verdict: PASS

The frozen reconstruction reads as plan-derived. I found no gold-ID leakage, no rubric vocabulary leakage, no gold-heading mirror, and no neat 1:1 mapping to the gold targets. The reconstruction's structure and wording closely track PLAN.md, including several exact phrases and plan-specific implementation calls.
