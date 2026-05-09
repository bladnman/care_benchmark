# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as plan-derived. It contains no gold IDs, rubric IDs, phase-two scoring vocabulary, or neat S1-S9/F1-F40 mapping. Its headings mirror the PLAN's implementation sections rather than GOLD_WHYS, and the strongest sentences are directly supportable from PLAN passages.

## ID leakage

No gold ID leakage found.

- Search target: S1-S9, F1-F40, R-F style IDs, canonical-number markers, and rubric vocabulary such as weight-3 or feature-level fidelity.
- RECONSTRUCTION hits: none.
- PLAN hits for the same ID/rubric pattern: none.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Judgment |
|---|---|---|
| "already alive" | PLAN §14: risk that spinner or slow first bird breaks the already-alive conceit | plan-derived |
| "honest presence and small gestures" | PLAN §1 uses the same phrase | plan-derived |
| "only writer of personality vectors" | PLAN §2 simulation worker bullet uses the same phrase | plan-derived |
| "low-pass filter over weekly-scale signal aggregates" | PLAN §5 personality drift uses the same phrase | plan-derived |
| "naturalist" / "matter-of-fact" voice split | PLAN §§4,9 use this split repeatedly | plan-derived |
| "relationship data" | PLAN §§1,10,14 use per-bird/per-account relationship-data language | plan-derived |
| "ships with v1" | PLAN §9 says accessibility ships with v1 | plan-derived |
| "read-only, off by default, revocable" visits | PLAN §1 uses the same visit framing | plan-derived |
| "quiet field" | PLAN §§6,7,14 use quiet-field loading language | plan-derived |
| "deterministic enough to test" | PLAN §5 uses the same phrase | plan-derived |

No suspect rubric-side phrases such as "multi-layer recovery," "feature-level fidelity," "weight-3," or "intent fidelity" appear in RECONSTRUCTION.

## Heading Mirror

RECONSTRUCTION headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Product boundary and v1 scope`
- `### System architecture`
- `### Core data model and API surface`
- `### Simulation engine design and sync`
- `### Frontend, audio, and accessibility`
- `### Privacy, performance, rollout, and guardrails`

These headings do not mirror GOLD_WHYS section titles. The per-feature headings are broad PLAN-derived implementation groupings, not the gold files or F1-F40 ordering.

## 1:1 Mapping Suspect

No 1:1 gold mapping found. RECONSTRUCTION has 11 numbered system-intent items rather than S1-S9, and its per-feature section is organized into six broad PLAN categories with mixed bullets. It does not enumerate 40 feature whys or proceed in the GOLD_WHYS order.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The aviary should feel already alive, not launched by the user."
   - PLAN support: §1 says the aviary appears to have been continuing before arrival; §7 says birds start mid-action with no wake-up animation; §14 names the already-alive conceit.
   - Result: supported.

2. Reconstruction sentence: "Canonical bird identity and behavior belong on the server."
   - PLAN support: §1 says all identity, mood, and drift state is canonical on the server; §2 states the client may not decide drift or overwrite bird state; §5 names the simulation worker as the only writer.
   - Result: supported.

3. Reconstruction sentence: "Privacy protects the user's relationship with their aviary."
   - PLAN support: §10 says per-bird events, vectors, mood, notebook sources, and relationship history are stored only for that account's aviary and excluded from analytics/training/recommendation/population dashboards.
   - Result: supported.

## Final Verdict

PASS. The reconstruction uses strong, sometimes polished language, but the phrasing is traceable to the PLAN and not to gold/rubric artifacts. No ID leakage, rubric vocabulary, gold-heading mirroring, or suspicious one-to-one target mapping was found.
