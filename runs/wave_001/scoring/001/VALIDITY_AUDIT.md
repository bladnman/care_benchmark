# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS.

Mechanical search of the frozen reconstruction found no gold IDs or reconstruction IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. The same search in the PLAN also returned no matching gold IDs. There is no ID leakage hit to cross-reference.

## Vocabulary Check

Verdict: PASS.

A search for scorer-side terms found no instances of `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, or `load-bearing` in the reconstruction.

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "continuity of individually recognizable birds over weeks" | PLAN sec. 1 uses the same phrase as the engineering success criterion. | plan-derived |
| "private state must never enter a shared CDN cache" | PLAN sec. 2 says private state must never enter shared CDN cache. | plan-derived |
| "Simulation authority belongs to the server" | PLAN secs. 2, 6 and 7 repeatedly reserve canonical writes for server workers/functions. | plan-derived |
| "no term depends negatively on days absent" | PLAN sec. 6 states this exact absence rule. | plan-derived |
| "quiet watching is valid" | PLAN sec. 5 says quiet watching remains valid until the activity window expires. | plan-derived |
| "accessibility is not a post-launch follow-up" | PLAN sec. 14 states accessibility is not a follow-up. | plan-derived |
| "no fake placeholder birds" | PLAN sec. 8 forbids fake placeholder birds in startup fallback. | plan-derived |
| "separate capability and route group" | PLAN sec. 2 assigns visitors to a separate capability and route group. | plan-derived |

## Heading Mirror

Verdict: PASS.

The top headings `## System-level intent` and `## Per-feature whys` are required by the phase-2A prompt. The remaining headings mirror PLAN structure, not the held-out gold list:

| Reconstruction heading | Match assessment |
|---|---|
| `### 1. Delivery contract and scope` | Mirrors PLAN sec. 1. |
| `### Product ambiguities resolved for implementation` | Mirrors PLAN subsection. |
| `### 2. Architecture and ownership boundaries` | Mirrors PLAN sec. 2. |
| `### 3. Persistent model and invariants` | Mirrors PLAN sec. 3. |
| `### 4. API and account flows` | Mirrors PLAN sec. 4. |
| `### 5. Presence and interaction accounting` | Mirrors PLAN sec. 5. |
| `### 6. Server simulation and behavior design` | Mirrors PLAN sec. 6. |
| `### 7. Snapshot synchronization and failure recovery` | Mirrors PLAN sec. 7. |
| `### 8. Frontend scene and startup pipeline` | Mirrors PLAN sec. 8. |
| `### 9. Audio, captions and narration` | Mirrors PLAN sec. 9. |
| `### 10. Private visits, export and lifecycle` | Mirrors PLAN sec. 10. |
| `### 11. Privacy and observability boundaries` | Mirrors PLAN sec. 11. |
| `### 12. Performance budgets and operational gates` | Mirrors PLAN sec. 12. |
| `### 13. Verification matrix` | Mirrors PLAN sec. 13. |
| `### 14. Implementation sequence and rollout` | Mirrors PLAN sec. 14. |
| `### 15. Principal risks and response` | Mirrors PLAN sec. 15. |

No heading mirrors GOLD_WHYS section names such as specific S or F titles.

## 1:1 Mapping Suspect

Verdict: PASS.

The reconstruction does not present S1-S9 or F1-F40, does not enumerate 49 gold why targets, and does not use gold ordering. It follows the PLAN's own section order and includes several items marked `NOT RECOVERABLE FROM PLAN`, which is consistent with the reconstructor prompt. The per-feature section is broad and plan-derived rather than a neat gold-list mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN grounding | Result |
|---|---|---|
| "The first screen should feel alive, not like loading software." | PLAN sec. 8 requires mid-action first paint and no spinner/fake placeholder birds; sec. 15 names the risk that the initial scene reads as loading software. | supported |
| "Quiet attention matters more than engagement mechanics." | PLAN sec. 5 credits quiet watching under the presence conjunction and rejects invasive engagement mechanics; sec. 6 says repeated clicking cannot substitute for quiet attention. | supported |
| "Accessibility is a first-class version of the experience, not a degraded fallback." | PLAN sec. 1 includes first-class narration/reduced motion/captions; sec. 9 makes reduced motion a separately art-directed path; sec. 14 says accessibility is not post-launch. | supported |

## Verdict

PASS. I found no significant contamination signatures: no leaked gold IDs, no scorer/rubric vocabulary, no gold-heading mirror, no 1:1 gold mapping, and the sampled high-level reconstruction sentences are grounded in the assigned PLAN. The reconstruction reads as a plan-structured summary with honest non-recovery markers rather than a held-out-gold reconstruction.
