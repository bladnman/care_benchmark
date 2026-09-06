# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no leakage found.

Mechanical search of the frozen reconstruction found no tokens matching gold/scorer IDs such as F1-F40, S1-S9, R-Fxx, or R-Sxx. Because there were no reconstruction hits, no cross-reference hits were required in PLAN.md.

## Vocabulary Check

Mechanical search found no scorer-side terms in RECONSTRUCTION.md: no "gold", "rubric", "weight-3", "multi-layer recovery", "feature-level fidelity", "system-level fidelity", "load-bearing", or "intent fidelity".

Sampled phrases and plan support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Rejects gamified task-completion unlocks; honors the slow deepening" | PLAN.md line 539 uses the same rationale. | Plan-derived. |
| "First-frame aliveness" and "No static start poses" | PLAN.md lines 19 and 406-408. | Plan-derived. |
| "server" is the "Sole Author" | PLAN.md lines 370-372. | Plan-derived. |
| "Strict conjunction" of visibility, focus, and recent input | PLAN.md lines 21 and 303-310. | Plan-derived. |
| "Ghost Presence Leaks" and "background tab counts presence" | PLAN.md line 548. | Plan-derived. |
| "primary aesthetic surface" | PLAN.md line 471. | Plan-derived. |
| "canned audio artifacts" | PLAN.md line 465. | Plan-derived. |
| "Quiet Visits" and read-only host control | PLAN.md lines 27-30 and 288-295. | Plan-derived. |

## Heading Mirror Check

RECONSTRUCTION.md headings are:

- '## System-level intent'
- '## Per-feature whys'
- '### 1. Scope & Boundary Enforcement'
- '### 2. Architecture & Service Topology'
- '### 3. Data Model & Storage Schema'
- '### 4. API Surface & Protocols'
- '### 5. Simulation Engine Design'
- '### 6. Multi-Device Synchronization & Conflict Prevention'
- '### 7. Frontend Rendering Pipeline'
- '### 8. Audio Pipeline & Synthesis Architecture'
- '### 9. Accessibility Surfaces'
- '### 10. Performance Budgets & Observability'
- '### 11. Rollout & Aviary Scaling Strategy'
- '### 12. Risk Management & Failure Modes'

The top two headings match the required reconstruction output structure. The numbered subsection headings mirror PLAN.md's implementation sections, not the gold list's S1-S9/F1-F40 structure. No gold-list section-title mirroring was found.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping pattern found. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold-list order. It lists 10 system-level principles and then a much longer plan-section feature pass. That shape matches the PLAN.md section order and the phase-2A prompt's instruction to use the plan's own structure.

## Plan-Derivation Spot Check

1. Reconstruction: "Presence must mean actual attention." Supporting plan: PLAN.md line 21 defines the strict conjunction; lines 303-310 implement it; line 548 explains ghost presence risk. Result: grounded.

2. Reconstruction: "Accessibility is part of the aesthetic experience." Supporting plan: PLAN.md line 471 says accessibility is a primary aesthetic surface; lines 473-479 define narration; lines 489-495 cover keyboard/focus; line 551 makes screen-reader testing release-blocking. Result: grounded.

3. Reconstruction: "No recorded audio fallbacks" preserve bundle budget and avoid canned artifacts. Supporting plan: PLAN.md lines 461-465 specify graceful silent mode, captions, and no recorded fallbacks. Result: grounded.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction is very close to PLAN.md's vocabulary and section order, but that is evidence of plan derivation rather than held-out gold/rubric exposure: it has no gold IDs, no scorer vocabulary, no 1:1 S/F mapping, and its articulate claims are traceable to PLAN.md.
