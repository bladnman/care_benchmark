# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict for this section: PASS.

I found no gold IDs or rubric IDs in the frozen reconstruction. The reconstruction uses the PLAN's own section structure and feature names, not `S1`-`S9`, `F1`-`F40`, `R-F01`, or similar held-out identifiers. PLAN also contains no such gold IDs, so there is no ID leakage hit to list.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "ambient, not distressed" | PLAN §3.3 says this is the engine-level implementation of "ambient, not distressed." | Plan-derived |
| "watching without moving is the product" | PLAN D1 uses the exact phrase. | Plan-derived |
| "Every host device is a projector" | PLAN §6.1 uses the exact phrase. | Plan-derived |
| "Build the quiet window, not a game around it" | PLAN §16 uses the exact phrase. | Plan-derived |
| "Privacy as architecture" | PLAN §2.4 heading. | Plan-derived |
| "designed aesthetic" | PLAN §6.6 describes reduced motion this way. | Plan-derived |
| "already alive" | PLAN §15.5 and §7.1 use this conceit. | Plan-derived |
| "load-bearing" | PLAN §11 calls `presence/monitor.ts` load-bearing. | Plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Required by the reconstructor prompt for absent rationale. | Expected, not leakage |

I did not find scorer-side vocabulary such as "multi-layer recovery," "feature-level fidelity," "weight-3," or "gold why" in RECONSTRUCTION.

## Heading Mirror Check

RECONSTRUCTION headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 0. Decisions and scope`
- `### 2. Architecture`
- `### 3. Data model`
- `### 4. API surface`
- `### 5. Simulation engine`
- `### 6. Sync model`
- `### 7. Frontend rendering pipeline`
- `### 8. Audio pipeline`
- `### 9. Accessibility surfaces`
- `### 10. Performance budgets and observability`
- `### 11. Frontend module map`
- `### 12. Security, privacy ops, mail`
- `### 13. Testing strategy`
- `### 14. Rollout`
- `### 15. Risks`
- `### 16. Team notes`

These mirror PLAN section headings, not the gold-list groupings. The two required top headings came from the phase-2A prompt. PASS.

## 1:1 Mapping Suspect Check

Verdict for this section: PASS.

The reconstruction does not enumerate S1-S9 or F1-F40, and it does not create a neat 49-row list in gold order. It follows the PLAN's own sections, including Decisions, Architecture, Data model, API surface, Simulation engine, Sync model, Frontend, Audio, Accessibility, Performance, Testing, Rollout, Risks, and Team notes. Some entries align with gold whys because the PLAN itself was comprehensive, but the structure is plan-derived rather than gold-derived.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The browser never advances mood, perch intent, weather, or personality." | PLAN D22 says exactly this under No client sim. | Grounded |
| "The gate implements 'neglect without punishment'; a bold bird away for two weeks becomes 'quieter, not warier' and snaps back 'by being watched - not by apology.'" | PLAN §3.3 contains the expressiveness gate and the quoted language. | Grounded |
| "No cold analytics copy exists to forget." | PLAN §6.5 says this under Hard-delete. | Grounded |
| "The monitor is 'load-bearing' because it owns the three-signal conjunction, 4-minute window, 30s pings, `session_end`, and hidden-tab stop behavior." | PLAN §11 labels `presence/monitor.ts` load-bearing and lists those checklist items. | Grounded |

## Verdict: PASS

The frozen reconstruction reads as a plan-derived artifact. It uses PLAN headings, PLAN vocabulary, and exact PLAN phrases; it does not expose gold IDs, rubric scoring language, or a suspicious 1:1 gold mapping. The few `NOT RECOVERABLE FROM PLAN` markers are consistent with the reconstruction prompt rather than contamination.
