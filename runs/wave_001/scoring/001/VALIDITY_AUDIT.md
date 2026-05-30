# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: PASS.

Searches for gold/rubric identifiers (`S1`-`S9`, `F1`-`F40`, `R-F*`, `feature-level`, `weight-3`, `multi-layer`, `load-bearing`, `intent fidelity`, `gold`, `rubric`) returned no hits in `RECONSTRUCTION.md` or `PLAN.md`. The reconstruction does not expose gold IDs or rubric-only taxonomy.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `quiet, ambient experience` | PLAN has `Quiet Field` and `ambient "Visits"` | Plan-derived phrasing/inference. |
| `game loop` | PLAN excludes scores, streaks, levels, badges, achievements and Tamagotchi mechanics | Inferred from PLAN; not rubric-specific. |
| `stateless viewer of the server-side canonical state` | Exact PLAN wording at line 27 | Directly plan-derived. |
| `low-pass filters` | Exact PLAN wording at line 43 | Directly plan-derived. |
| `naturalist prose` | PLAN lines 13, 49, 92 | Directly plan-derived. |
| `primary, naturalist-voiced surface from day one` | PLAN line 113 | Directly plan-derived. |
| `read-only, ambient` | PLAN line 12 and line 22 | Directly plan-derived. |
| `NOT RECOVERABLE FROM PLAN` | Reconstruction convention, not in PLAN | Acceptable phase-2A marking; not gold leakage. |

No rubric-side phrases such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, `load-bearing`, or `intent fidelity` appear in the reconstruction.

## Heading Mirror Check

The reconstruction uses the required top-level headings `System-level intent` and `Per-feature whys`, then mirrors the PLAN's numbered section headings:

- `1. Scope & Non-Goals`
- `2. Architecture & Data Model`
- `3. Simulation Engine Design`
- `4. API Surface & Sync Model`
- `5. Frontend Rendering & Audio Pipelines`
- `6. Accessibility & Performance`
- `7. Rollout & Risks`

These headings mirror `PLAN.md`, not `GOLD_WHYS.md`. They do not match gold headings such as `S1 - feels-alive-not-robotic` or file-grouped feature sections.

## 1:1 Mapping Suspect Check

Verdict for this check: PASS.

The reconstruction does not provide a neat S1-S9/F1-F40 list, does not follow gold order, and has only seven system-level bullets. Its per-feature section is organized by PLAN section and PLAN bullets, not by the 49 gold targets. This is not a 1:1 gold mirror.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `The architecture centers on a canonical server state.` | PLAN lines 27-28 describe the client as a stateless viewer and server as authoritative source of truth. | Supported. |
| `Presence is carefully gated by visible, focused, recently active use.` | PLAN line 53 requires visible state, focus, and recent pointer/key activity. | Supported. |
| `The rationale is primary access in the same product voice, not a fallback.` | PLAN lines 14, 92, and 113 describe naturalist narration/reduced motion and accessibility as primary from day one. | Supported. |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived: it quotes or paraphrases PLAN headings and PLAN vocabulary, marks several areas `NOT RECOVERABLE FROM PLAN`, and does not leak gold IDs, rubric vocabulary, or a 1:1 gold-list structure.
