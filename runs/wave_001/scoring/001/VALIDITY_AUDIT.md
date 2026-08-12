# VALIDITY_AUDIT - CARE run 001

## ID Leakage

**Result: PASS.** I found no gold IDs such as S1-S9, F1-F40, R-F01, rubric weights, or gold-list labels in the frozen reconstruction. The phrase `NOT RECOVERABLE FROM PLAN` appears several times in RECONSTRUCTION, but that is a phase-2A recovery marker, not a gold ID, and it does not expose gold taxonomy.

Search cross-check: the ID/vocabulary scan found only `NOT RECOVERABLE FROM PLAN` instances in RECONSTRUCTION and no S/F/R-F IDs. No matching hidden gold IDs were present in PLAN.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Verdict |
|---|---|---|
| "Aliveness is the product" | PLAN intro uses the same phrase. | plan-derived |
| "the aviary has been continuing without you" / "not a visual trick" | PLAN §3.1 uses these phrases. | plan-derived |
| "watching is the product" | PLAN §10.1 calibration comment says this. | plan-derived |
| "Clients never tick" | PLAN §3.1 says this directly. | plan-derived |
| "Non-goals are constraints, not backlog" | PLAN opening section says this. | plan-derived |
| "network ACL and a credential split, not a policy paragraph" | PLAN §3.2 uses this phrasing. | plan-derived |
| "a designed product surface" | PLAN §11 says accessibility is a designed product surface. | plan-derived |
| "Small on purpose" | PLAN §14 uses this for visits. | plan-derived |
| "two cameras on one aviary" | PLAN §7.1 uses this phrasing. | plan-derived |
| "stronger than a comment" | PLAN §17.3 uses this for DB column grants. | plan-derived |

I did not find suspicious rubric-side terms such as `multi-layer`, `feature-level fidelity`, `intent fidelity`, `weight-3`, `gold why`, or `rubric` in RECONSTRUCTION.

## Heading Mirror

RECONSTRUCTION headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope and locked decisions`
- `### Architecture and data`
- `### API and sync`
- `### Simulation engine`
- `### Frontend, audio, interactions, and accessibility`
- `### Performance, privacy, social, rollout, and risk controls`

Gold headings include S1-S9 titles, file-group headings such as `concepts.md`, `bird_engine.md`, and `Complete features list`. The reconstruction does not mirror the S/F gold titles or file-group headings. Its headings look derived from PLAN structure and the expected reconstruction output shape, not from GOLD_WHYS.

## 1:1 Mapping Suspect

**Result: no 1:1 gold mapping.** RECONSTRUCTION has 13 system-level bullets, not 9, and its per-feature section follows plan-domain groupings with many more items than the 40 gold feature whys. It does not enumerate F1-F40 or S1-S9, and it includes many plan features that are not gold-why-bearing. The order is closer to PLAN scope -> architecture -> API -> simulation -> frontend -> rollout/risk than to GOLD_WHYS.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "Canonical life belongs to the server; the browser is presentational." | PLAN intro says the server is the only writer; PLAN §3.1 says server owns time/personality/mood and clients never tick. | grounded |
| "A call is not a file." | PLAN §9.1-§9.2 states v1 ships no call samples and describes procedural WebAudio graphs. | grounded |
| "The plan avoids localStorage because it could become a second source of truth that replays days later." | PLAN §7.7 says not to put events in localStorage as a second source of truth that could replay days later. | grounded |

## Verdict

**PASS.** The reconstruction reads as a plan-derived artifact: no gold IDs leaked, no rubric vocabulary appears, headings do not mirror the gold taxonomy, and spot-checked articulate claims are directly grounded in PLAN. Scores are not flagged for contamination.
