# VALIDITY_AUDIT - CARE run 001

## ID Leakage

**Result: PASS.** Mechanical search found no gold-side IDs such as `F1`-`F40`, `S1`-`S9`, `R-F01`, or `R-S01` in the frozen reconstruction. The only structured labels are the reconstruction's own plan-derived headings and feature names. Cross-reference: the PLAN also does not expose gold IDs.

## Vocabulary Check

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "single-screen, non-panning horizontal aviary scene" | PLAN §1.1 has the exact phrase. | plan-derived |
| "continuous time-of-day/ambient weather rendering" | PLAN §1.1 has the exact phrase. | plan-derived |
| "No Tamagotchi Mechanics" | PLAN §1.2 has the exact phrase. | plan-derived |
| "server is the sole writer" | PLAN §6.2 has the exact phrase. | plan-derived |
| "Immutable append-only log for raw interaction events" | PLAN §2.1 data stores has the exact phrase. | plan-derived |
| "Naturalist prose as the product voice" | PLAN repeatedly says naturalist prose for notebook, narration, and captions. | plan-derived synthesis |
| "physically isolated from simulation databases" | PLAN §9.2 has the exact phrase. | plan-derived |
| "generic state logs" | PLAN risk matrix says accessible surfaces should not feel like generic state logs. | plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Not in PLAN, but required by the phase-2A prompt; not gold/rubric leakage by itself. | acceptable operator vocabulary |

No scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity` appear in the reconstruction.

## Heading Mirror

The reconstruction headings mirror the PLAN, not the gold list:

| Reconstruction heading | Closest source | Assessment |
|---|---|---|
| `## System-level intent` | Required phase-2A output format. | acceptable |
| `## Per-feature whys` | Required phase-2A output format. | acceptable |
| `### Scope & System Boundaries` | PLAN §1 heading. | plan-derived |
| `### Architecture & Service Topology` | PLAN §2 heading. | plan-derived |
| `### Data Model & Schema Definitions` | PLAN §3 heading. | plan-derived |
| `### API Surface & Protocols` | PLAN §4 heading. | plan-derived |
| `### Simulation Engine Design` | PLAN §5 heading. | plan-derived |
| `### Presence & Multi-Device Sync Protocol` | PLAN §6 heading. | plan-derived |
| `### Frontend & Audio Rendering Pipeline` | PLAN §7 heading. | plan-derived |
| `### Accessibility Surfaces` | PLAN §8 heading. | plan-derived |
| `### Performance Budgets & Observability` | PLAN §9 heading. | plan-derived |
| `### Rollout & Staging Strategy` | PLAN §10 heading. | plan-derived |
| `### Risk Matrix & Mitigations` | PLAN §11 heading. | plan-derived |

No heading mirrors the gold-list sections S1-S9 or F1-F40.

## 1:1 Mapping Suspect

**Result: PASS.** The reconstruction does not enumerate S1-S9 or F1-F40 in order and does not create a neat gold-target mapping. It follows the PLAN's section order and includes many non-gold implementation features such as Edge Gateway, Redis cache, API endpoints, rollout phases, and risk mitigations. This structure is consistent with plan derivation rather than gold-list exposure.

## Plan-Derivation Spot Check

1. Reconstruction: "The plan repeatedly makes the server the authority: 'single canonical server-side simulation tick,' 'No Client State Ownership,' 'server is the sole writer,' and 'No Last-Write-Wins.'"
   - PLAN support: §1.1 says sync is driven by a canonical server-side tick; §1.2 says no client state ownership; §6.2 says the server is the sole writer and no LWW.
   - Verdict: supported.

2. Reconstruction: "The Field Notebook is 'naturalist, prose-based observation log generated server-side.' The same voice appears in 'Lowercase naturalist prose,' 'Screen-Reader Narration,' procedural call captions..."
   - PLAN support: §1.1 names the field notebook phrase; §3.4 stores lowercase naturalist prose; §8 specifies naturalist screen-reader narration and call captions.
   - Verdict: supported.

3. Reconstruction: "The account model uses 'Synthetic UUID account keying (PII isolation),' encrypted email, a blind hash index, and a 'Strict Privacy Rule' that telemetry pipelines are 'physically isolated from simulation databases.'"
   - PLAN support: §1.1 names synthetic UUID/PII isolation; §3.1 has encrypted email and blind hash; §9.2 states telemetry isolation.
   - Verdict: supported.

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It uses the PLAN's headings, feature order, and vocabulary; it contains no gold IDs or rubric-side scoring language; and its most articulate claims can be grounded directly in the PLAN. The repeated `NOT RECOVERABLE FROM PLAN` markers are consistent with the reconstructor prompt rather than contamination.
