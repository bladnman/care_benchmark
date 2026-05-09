# VALIDITY_AUDIT — CARE run 001

## 1. Gold ID Leakage Check

Verdict: no ID leakage found. A targeted scan found no `S1`-`S9`, `F1`-`F40`, or `R-Fxx` style gold identifiers in either the frozen reconstruction or the plan. RECONSTRUCTION uses plan section labels and ordinary feature names, not gold IDs.

## 2. Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "ambient, evolving relationships" | yes | Directly plan-derived from the opening sentence. |
| "Non-game, non-punishing, non-social product boundaries" | yes, semantically | Derived from the explicit non-goals: games/gamification, Tamagotchi/punishing neglect, social networks. |
| "Server-authored canonical state" | yes, semantically | Derived from canonical server state and server-only authorship. |
| "Procedural change from presence, interaction, time, personality, and events" | yes | Derived from drift, mood modulation, procedural rendering/audio, and event log. |
| "Accessible parallel surfaces" | yes, semantically | Derived from narration, reduced motion, captions, and keyboard reachability. |
| "Operationally limited analytics and account data" | yes, semantically | Derived from synthetic UUID/encrypted email and operational-health-only telemetry. |
| "NOT RECOVERABLE FROM PLAN" | no | This is reconstructor scoring vocabulary, not gold-side vocabulary; it is not a contamination sign by itself. |

I did not find rubric-side phrases such as "multi-layer recovery", "feature-level fidelity", "weight-3", "gold why", or "intent fidelity" in the reconstruction.

## 3. Heading Mirror Check

RECONSTRUCTION headings are:

| Reconstruction heading | Closest source | Assessment |
|---|---|---|
| `## System-level intent` | Phase-2A expected output shape | Generic reconstruction section, not a gold heading. |
| `## Per-feature whys` | Phase-2A expected output shape | Generic reconstruction section, not a gold heading. |
| `### 1. Scope` through `### 11. Rollout` | PLAN section headings | Mirrors PLAN, not GOLD_WHYS. |

No heading mirrors gold section titles such as "feels-alive-not-robotic", "notice-never-announce", or feature-level F-title wording.

## 4. 1:1 Mapping Suspect Check

No 1:1 mapping to the gold list was detected. The reconstruction has seven system-level bullets and plan-section feature groupings, while the gold list has S1-S9 and F1-F40 in a different taxonomy. The reconstruction order follows PLAN sections (`Scope`, `Architecture`, `Data Model`, etc.), not gold order.

## 5. Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "The plan frames Pocket Aviary V1 as 'a browser-based virtual aviary focused on ambient, evolving relationships between users and birds.'" | PLAN opening sentence contains the same phrase. | Plan-derived. |
| "The architecture and sync model repeatedly center server authorship: the server is the 'Canonical state manager,' 'Canonical state lives on the server,' clients 'consume snapshots,' and there is 'no last-write-wins for personality (server-only authorship).'" | PLAN §§2 and 6 contain these exact or near-exact phrases. | Plan-derived. |
| "Accessibility is not a single add-on in the plan. It includes 'Naturalist prose' via screen reader, 'Reduced-Motion' cross-fade, 'Procedural prose captions for calls,' and 'Full reachability' by keyboard." | PLAN §9 lists narration, reduced-motion, captioning, and keyboard. | Plan-derived, with mild synthesis. |

## 6. Verdict

PASS. The frozen reconstruction reads as a plan-derived synthesis: it mirrors PLAN structure, uses plan vocabulary, contains no gold IDs, and does not present a neat S1-S9/F1-F40 mapping. Some phrases are inferential summaries, but they are grounded in the assigned PLAN rather than in gold-list language.
