# VALIDITY_AUDIT — CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as plan-derived. I found no gold-ID leakage, no external gold/rubric taxonomy in headings, no neat S1-S9/F1-F40 mapping, and no reconstruction content that required PRD/gold access. One vocabulary hit, "golden replays," is plan-derived engineering language rather than benchmark leakage.

## ID Leakage

Mechanical search for `F#`, `S#`, `R-F#`, and `R-S#` tokens in `RECONSTRUCTION.md` returned no hits. The same search in `PLAN.md` also returned no hits. There is therefore no gold ID leakage.

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "Aliveness is the product" | yes, planning stance | Plan-derived |
| "the aviary was already going when you opened the tab" | yes, planning stance | Plan-derived |
| "Privacy is architecture, not a policy add-on" | yes, PLAN §2.5 heading | Plan-derived |
| "same product in another register" | yes, PLAN §9 | Plan-derived |
| "stops the client from becoming a stats console" | yes, PLAN §1.4 | Plan-derived |
| "watching without moving is the product" | yes, PLAN §1.4 | Plan-derived |
| "golden replays" / "golden-file tick replays" | yes, PLAN §13.1 uses golden-file testing | False positive, not gold-list leakage |
| "feature-level fidelity", "system-level fidelity", "weight-3", "multi-layer recovery" | no hits in reconstruction | No scorer-side vocabulary |

## Heading Mirror

`RECONSTRUCTION.md` headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture`
- `### Data model and defensible calls`
- `### API surface`
- `### Simulation engine`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance budgets and observability`
- `### Rollout`
- `### Risks and implementation notes`

The two top-level headings are required by the reconstructor prompt. The subsection headings mirror the PLAN's own structure, not `GOLD_WHYS.md` sections. No heading mirrors S1-S9 or F1-F40 titles.

## 1:1 Mapping Suspect

No suspect 1:1 mapping. The reconstruction does not enumerate gold IDs, does not produce exactly 9 system rows plus 40 feature rows, and does not follow the gold-list order. It groups by PLAN sections and includes many implementation-plan bullets that are not gold-why rows.

## Plan-Derivation Spot Check

1. Reconstruction: "The server owns the living state; the client performs the place."
   PLAN support: §2.2 says the server owns account, birds, mood, weather, notebook, and the client owns scene graph, WebAudio, presence monitoring, and chrome.

2. Reconstruction: "Privacy is architecture, not a policy add-on."
   PLAN support: §2.5 is titled "Privacy architecture (not a policy add-on)" and defines separate Simulation and Operations planes with no shared readers.

3. Reconstruction: "Accessibility ships as the same product in another register, not a bolt-on state dump."
   PLAN support: §9 states accessibility ships "as the same product in another register, not a bolt-on state dump," then details narration, reduced motion, captions, keyboard, and visitor accessibility.

## Final Rationale

The reconstruction is broad and articulate, but its language and organization are traceable to the plan. It contains no gold IDs, no benchmark-rubric vocabulary, and no gold-list ordering pattern. PASS.
