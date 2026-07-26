# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict: **PASS**.

Searches for gold-style IDs and rubric-only labels found no `F1`-`F40`, `S1`-`S9`, `R-F*`, `gold`, `rubric`, `weight-2`, `weight-3`, `multi-layer`, `feature-level fidelity`, or `intent fidelity` hits in the frozen reconstruction. The only load-bearing-looking term hit was `load-bearing`, but it appears directly in PLAN (`PLAN.md:140`, `PLAN.md:285`) and is therefore not leakage.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "structurally refused" | PLAN §1.2 uses "structurally refused" for non-goals. | Plan-derived. |
| "server owns discrete state; the client owns continuous presentation" | PLAN §2.1 states this exact boundary. | Plan-derived. |
| "load-bearing systems trick" | PLAN §2.5 heading uses this phrase. | Plan-derived, not rubric leakage. |
| "absence ... in the DDL" | PLAN §1.2 says the absence is in the DDL, not policy. | Plan-derived. |
| "Accessibility ships with v1 or v1 does not ship" | PLAN §10.1 states this. | Plan-derived. |
| "implemented as network topology, not policy" | PLAN §12.3 states this. | Plan-derived. |
| "single most likely charm-destroying change" | PLAN risk section and reconstruction both refer to the toast risk in this wording. | Plan-derived. |
| "first painted frame is a frame from the middle" | PLAN §7.9 states this. | Plan-derived. |
| "one observer" | PLAN §9.1 describes one voice/observer across notebook, narration, captions. | Plan-derived. |
| "measured, none is a vibe check" | PLAN §14.6 uses this launch-gate phrasing. | Plan-derived. |

No sampled phrase reads as gold-side taxonomy rather than plan vocabulary.

## Heading Mirror Check

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture`
- `### Data model`
- `### API surface`
- `### Simulation engine design`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Voice kernel - notebook, narration, captions`
- `### Accessibility surfaces`
- `### Performance budgets and observability`
- `### Privacy and security engineering`
- `### Testing strategy`
- `### Rollout`
- `### Risks, decisions, and appendices`

The `###` headings mirror PLAN sections, not GOLD_WHYS section titles or S/F IDs. The two top-level reconstruction headings are expected reconstruction organization, not evidence of gold-list exposure.

## 1:1 Mapping Suspect Check

Verdict: **PASS**.

The reconstruction does not give a neat S1-S9 then F1-F40 list. Instead it contains a broad system-intent summary followed by many plan-section bullets, including non-gold implementation details such as deployable units, Redis, WebAudio memory pooling, chunks, load tests, and environment layout. The order follows the PLAN, not the gold list.

## Plan-Derivation Spot Check

1. Reconstruction: "Make the aviary continuous in the strong sense through deterministic replay."
   PLAN support: `PLAN.md:144-146` says the tick is logically continuous, physically tiered with exact catch-up, and bit-identical to continuous ticking.

2. Reconstruction: "No spinner component" and first frame as a middle frame of continuous animation.
   PLAN support: `PLAN.md:735-738` says no spinner exists and the first painted frame is from the middle of continuous animation.

3. Reconstruction: privacy as topology/type-system property.
   PLAN support: `PLAN.md:1075-1080` says the privacy commitment is network topology, no telemetry route to sim DB, no per-account metric dimension, and no ETL access.

All three articulate claims are directly supported by PLAN passages.

## Verdict

**PASS** — no significant contamination signatures. The reconstruction reads as plan-derived: vocabulary and headings trace to PLAN, there is no gold ID leakage, and the output does not map 1:1 to the gold list.
