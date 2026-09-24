# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict: no leakage found. A mechanical search of the frozen reconstruction found no gold IDs such as S1-S9, F1-F40, R-S*, or R-F*. The only ID-like hit in the paired search was in the PLAN, not the reconstruction: "S3-compatible object storage," which is unrelated to gold IDs.

## Vocabulary check

No scorer-side vocabulary was found in the reconstruction for: gold, rubric, weight-3, multi-layer recovery, feature-level fidelity, system-level fidelity, intent fidelity, or similar terms.

Sampled reconstruction phrases and PLAN derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Canonical state belongs to the server" | PLAN 0.1: server is the only writer; PLAN 6.1 one canonical record | plan-derived |
| "Drift is monotonic by construction" | PLAN 0.1 decision 2; PLAN 5.5 drift formula | plan-derived |
| "Presence means honest attention, not an open tab" | PLAN 0.1 decision 7; PLAN D6; PLAN 5.4 | plan-derived |
| "Notice, never announce" | PLAN 0.2 PRD commitment table and multiple guardrails | plan-derived |
| "Privacy is an architectural rule, not policy" | PLAN 2.1 Zones section uses this phrase | plan-derived |
| "Accessibility is a designed surface, not a fallback" | PLAN 0.2 accessibility proof; PLAN 7.9 | plan-derived |
| "First contact should feel alive, not loaded" | PLAN 0.1 first frame; PLAN 7.1 boot/quiet field | plan-derived |
| "Changes after launch move forward and blend" | PLAN 14.6 post-launch change policy | plan-derived |

## Heading mirror

The reconstruction headings are:

- ## System-level intent
- ## Per-feature whys
- ### 0. Orientation and 1. Scope
- ### 1.2 Out of v1 and 1.3 decisions
- ### 2. Architecture, 3. Data model, and 4. API surface
- ### 5. Simulation engine
- ### 6. Sync model
- ### 7. Frontend rendering pipeline
- ### 8. Audio pipeline
- ### 9. Accessibility surfaces
- ### 10. Voice and content system
- ### 11. Privacy, security, and account lifecycle
- ### 12. Performance, 13. Testing, 14. Rollout, and 15-16 risks/open items

These mirror the PLAN's organization, not the held-out gold list. They do not mirror S1-S9 or F1-F40 headings.

## 1:1 mapping suspect

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold order. Its per-feature section is much larger than the 49 gold whys and follows PLAN sections from orientation through rollout.

## Plan-derivation spot check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan uses that exact phrase for the Zone A/B/C topology." | PLAN 2.1: "That is the PRD's architectural rule, not policy." | supported |
| "The plan rejects an LLM for notebook, narration, and captions because a grammar gives voice control, sparsity control, and output we can lint." | PLAN 0.1 decision 13 says notebook, narration, and captions use an authored grammar, not an LLM, for voice/sparsity/lint control. | supported |
| "The tick is a deterministic function, RNG streams are keyed so adding a subsystem does not shift others, math is vendored to avoid engine/version dependence." | PLAN 0.1 decision 4 and PLAN 5.2 determinism specify deterministic advance, keyed RNG streams, and vendored pure-TS math. | supported |

## Verdict: PASS

The reconstruction reads as plan-derived. I found no gold ID leakage, no scorer vocabulary, no heading mirror of the held-out gold taxonomy, and no 1:1 mapping to the gold list. The few phrases that sound load-bearing are traceable to the PLAN's own language and section structure.
