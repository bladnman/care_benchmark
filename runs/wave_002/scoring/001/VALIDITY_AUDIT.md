# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Mechanical search for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, `R-S[0-9]+`, plus scorer-side terms returned no hits in `RECONSTRUCTION.md`. The reconstruction does not use gold IDs, rebuild IDs, or an external taxonomy absent from the plan.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Quiet, non-gamified ambient presence" | PLAN non-goals: no gamification, no push/notification, no social network features | Plan-derived summary phrase |
| "Gentle, non-punitive change over time" | PLAN says drift is monotonic and neglect creates ambient quietness, never hostility or sadness | Plan-derived |
| "Server-authoritative state with no client personality writes" | PLAN says server-side tick is canonical and zero client-side personality mutations | Plan-derived |
| "Privacy-preserving identity and observation" | PLAN uses synthetic UUIDs and excludes per-bird/user-bird relationship observability | Plan-derived |
| "Naturalist product voice" | PLAN repeatedly uses naturalist prose for notebook/narration/captions | Plan-derived |
| "First-frame and ongoing aliveness" | PLAN says already mid-motion, quiet field fallback, continuous micro-motion | Plan-derived |
| "Accessibility and graceful fallback as normal surfaces" | PLAN includes narration, reduced motion, captions, keyboard navigation, WebAudio fallback | Plan-derived |
| "Optional, quiet social access under host control" | PLAN says optional/quiet social, host-initiated invitations, revocation, silent visit logging | Plan-derived |

No suspect rubric-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity` appeared in the reconstruction.

## Heading Mirror Check

The top headings are `## System-level intent` and `## Per-feature whys`, which are required by the phase-2A prompt. The subordinate headings mirror the PLAN sections: Scope & Non-Goals, Architecture & System Topology, Data Model, API Surface & Protocols, Simulation Engine & Drift Runtime, Sync Model & State Propagation, Frontend Rendering Pipeline & Scene Composition, Procedural Audio Pipeline, Accessibility Surfaces, Performance Budgets & Observability, Rollout & Aviary Scaling Plan, and Risk Analysis & Mitigation Strategies.

They do not mirror the gold-list section titles or F/S ordering. This supports plan derivation rather than contamination.

## 1:1 Mapping Suspect Check

No 1:1 mapping to S1-S9 or F1-F40 is present. The reconstruction has 8 system bullets and then follows the PLAN section order with many plan features, including non-gold implementation details such as Redis cache/queue, API endpoints, DDL fields, rollout phases, and risk mitigations. This is not a neat gold-target list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "The plan later names the risk as `False Presence Inflation` and says backgrounded tabs can inflate presence hours." | PLAN Risk Analysis: `False Presence Inflation`; mitigation requires visibility, focus, and recent input | Supported |
| "The render pipeline draws birds immediately in mid-pose and uses a tranquil ambient sky if fetch exceeds 150ms." | PLAN Frontend Rendering: birds are drawn immediately in mid-pose; fallback after 150ms uses tranquil ambient sky | Supported |
| "If `AudioContext` fails, the system enters silent mode, enables captions, and loads `No canned or looped audio files`." | PLAN Procedural Audio: WebAudio fallback enters silent mode, enables captions, and loads no canned/looped files | Supported |

## Verdict

PASS

The reconstruction shows no significant contamination signatures. It uses plan headings, plan vocabulary, and plan-specific implementation details; no gold IDs, gold ordering, or scorer/rubric vocabulary were detected. The few polished abstractions read as summaries of the plan rather than leakage from held-out materials.
