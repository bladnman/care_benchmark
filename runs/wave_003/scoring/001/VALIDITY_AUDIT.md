# VALIDITY_AUDIT - CARE run 001

Verdict: **PASS**

## ID Leakage Check

No gold IDs or rubric IDs were found in the frozen reconstruction. Searches for `S1`-`S9`, `F1`-`F40`, `R-F##`, `weight-3`, `multi-layer`, and `intent fidelity` produced no leakage hits. The reconstruction uses the heading `## Per-feature whys`, but that is the required phase-2A output shape rather than a gold ID; no corresponding S/F target IDs appear.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "explicit refusals" | PLAN.md:27-42 `Out of scope ... explicit refusals` | Plan-derived |
| "the shape of v1 is what is left after those subtractions" | PLAN.md:42 exact phrase | Plan-derived |
| "server-canonical state" / "only writer" | PLAN.md:60-63 and 355-369 | Plan-derived |
| "naturalist, quiet, and not user-addressing" | PLAN.md:342-347, 489-491, 640 | Plan-derived synthesis |
| "designed surface, not a fallback" | PLAN.md:481-495 | Plan-derived |
| "Privacy boundaries are architectural, not policy-only" | PLAN.md:56, 176-180, 556-563 | Plan-derived synthesis |
| "the aviary is still the aviary" | PLAN.md:413 exact phrase | Plan-derived |
| "load-bearing", "multi-layer", "weight-3", "intent fidelity" | Not present in reconstruction | No suspect rubric vocabulary found |

The reconstruction uses ordinary words like `why` and `PRD`, but both are present in the PLAN itself (`PLAN.md:3`, `PLAN.md:656-671`) and not used in a rubric-like way.

## Heading Mirror Check

| Reconstruction heading | Comparison | Assessment |
|---|---|---|
| `## System-level intent` | Required reconstruction section; not a GOLD_WHYS section title. | OK |
| `## Per-feature whys` | Required reconstruction section; not a gold ID list. | OK |
| `### Scope`, `### Architecture`, `### Data model`, `### API surface`, `### Simulation engine design`, `### Sync model`, `### Frontend rendering pipeline`, `### Audio pipeline`, `### Accessibility surfaces`, `### Performance budgets and observability`, `### Rollout`, `### Risks`, `### Defensible calls where the PRD is silent`, `### What this plan is not` | These mirror PLAN.md sections, not GOLD_WHYS.md section titles or S/F labels. | Plan-derived |

No reconstruction heading mirrors `S1-S9`, `F1-F40`, or the gold-list grouped file headings in a suspicious way.

## 1:1 Mapping Suspect Check

The reconstruction does **not** map neatly to all S1-S9 and F1-F40 targets. It instead walks the PLAN structure and includes many non-gold implementation items: web-only SPA, service split, lazy AudioContext creation, one-time-use visit tokens, API endpoints, AudioWorklet node reuse, rollout phases, and browser fragmentation. It also marks several plan details as `NOT RECOVERABLE FROM PLAN`. The order is roughly the PLAN order, not the gold-list order.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Finding |
|---|---|---|
| "The plan says the shape of v1 is what is left after those subtractions and that adjacent feature pitches should be rejected at the design step." | PLAN.md:42 exact support | Grounded |
| "The narration generator runs in the same process as the renderer and reads from the same state cache, so screen-reader output stays voice-continuous with the visual surface." | PLAN.md:69 exact support | Grounded |
| "A canned fallback would feel canned, and a procedural fallback at recorded quality would blow the bundle budget." | PLAN.md:471 exact support | Grounded |
| "A toggle implies the feature can come back." | PLAN.md:606 exact support | Grounded |

## Verdict

**PASS.** The frozen reconstruction reads as derived from `PLAN.md`: no gold IDs leak, suspect rubric vocabulary is absent, headings mirror the PLAN rather than the gold list, and articulate claims spot-check cleanly against plan text. The `Per-feature whys` heading is expected output structure and not, by itself, contamination.
