# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A search of the frozen reconstruction for `F#`, `S#`, `R-F#`, and `R-S#` style identifiers returned no hits. The plan search also returned no such identifiers, so there were no reconstruction-only gold IDs to cross-reference.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION.md against PLAN.md:

| Phrase in reconstruction | PLAN support | Assessment |
|---|---|---|
| "server-side simulation is the load-bearing way" | PLAN intro: "single most important property" and "load-bearing rules" | Plan-derived |
| "renderer and an event-ingestor" | PLAN Client/server split uses this exact phrase | Plan-derived |
| "notice, never announce" | PLAN §13 and scope repeat the rule | Plan-derived |
| "load-bearing refusal of the gamification trap" | PLAN §11 uses this exact phrase | Plan-derived |
| "quieter than it was, not sadder than it was" | PLAN drift function uses this exact phrase | Plan-derived |
| "felt, not inspected numerically" | PLAN API-boundary rule supports the phrase; wording is a close paraphrase | Plan-derived |
| "privacy is structural, not cosmetic" | PLAN privacy and telemetry sections support this; phrase is a paraphrase, not scorer vocabulary | Plan-derived |
| "different aesthetic, not a degraded one" | PLAN reduced-motion section uses this exact phrase | Plan-derived |
| "affective spine" | PLAN audio risk and procedural-call sections use this wording | Plan-derived |

Suspicious scorer-side terms (`gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`) did not appear. The word `load-bearing` appears several times, but it also appears repeatedly in PLAN.md, so it is not a contamination signal here.

## Heading Mirror

The reconstruction headings are: `System-level intent`, `Per-feature whys`, then plan-like sections (`Scope`, `Decisions`, `Architecture`, `Data model`, `API surface`, `Simulation engine design`, `Sync model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Performance budgets and observability`, `Rollout`, `Risks and mitigations`, `Load-bearing refusals and out-of-scope confirmations`).

`System-level intent` and `Per-feature whys` are required by the phase-2A prompt. The remaining headings mirror PLAN.md structure, not GOLD_WHYS.md section titles. No near-exact mirror of the S1-S9/F1-F40 gold list structure was found.

## 1:1 Mapping Suspect

No 1:1 gold mapping detected. The reconstruction does not list S1-S9 or F1-F40, uses no gold IDs, and does not proceed in gold-list order. It reconstructs a broad plan-shaped inventory with many more entries than the 49 scored whys. This is consistent with derivation from PLAN.md.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Canonical state belongs on the server; the client is 'a renderer and an event-ingestor.'"
   PLAN support: Client/server split says the client is a renderer and event-ingestor and never owns personality, mood, perch, notebook, or drift math.
   Result: supported.

2. Reconstruction sentence: "The intended feeling is that an unattended bird is 'quieter than it was, not sadder than it was.'"
   PLAN support: Drift function says neglect is not punished and a bird left alone is quieter, not sadder.
   Result: supported.

3. Reconstruction sentence: "Visit email re-validation on each open: the plan's rationale is 'privacy-correct vs. low-friction' because the host invited a specific email, not a URL."
   PLAN support: Visit-link replay mitigation uses the same privacy-correct vs. low-friction framing and says the host invited a specific email, not a URL.
   Result: supported.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold identifiers, no scorer-side vocabulary, plan-mirroring headings, and spot-checked articulate claims are grounded in PLAN.md. The few strong-sounding phrases are present in or directly paraphrased from the plan.
