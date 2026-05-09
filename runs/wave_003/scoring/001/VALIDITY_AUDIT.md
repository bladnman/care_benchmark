# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as plan-derived: it follows the PLAN's section order and vocabulary, contains no gold ID leakage, does not mirror the gold list, and does not create a neat S1-S9/F1-F40 mapping. The main issue is compression of rationale, not contamination.

## ID Leakage Check

No gold IDs or rubric IDs were found in `RECONSTRUCTION.md`: no `S1`-style system IDs, no `F1`-style feature IDs, no `R-F` IDs, and no explicit `GOLD`/`RUBRIC` references. The corresponding search produced no hits, so there is no ID leakage to cross-check against PLAN.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "server-side canonical state" | PLAN repeatedly says canonical state lives only on the server and clients are render surfaces. | plan-derived |
| "Felt-aliveness" | PLAN performance table names first-bird render as a "Felt-aliveness threshold" and uses felt-aliveness in risk text. | plan-derived |
| "Notice, do not announce" | PLAN has a "Notice never announce" future-feature risk and no-announcement examples. | plan-derived |
| "naturalist field-notebook voice" | PLAN notebook, narration, and captions use this phrase. | plan-derived |
| "same aviary" for reduced motion | PLAN says reduced motion is the same aviary and only the visual motion register changes. | plan-derived |
| "privacy boundary" | PLAN RUM section says the privacy boundary is maintained at metric-definition level. | plan-derived |
| "attentive watching" | PLAN presence-risk section says the activity window gives credit for attentive watching. | plan-derived |
| "recognizably like itself but varied" | PLAN call-grammar section uses essentially the same language. | plan-derived |

Rubric-side terms such as `multi-layer`, `feature-level fidelity`, `weight-3`, and `intent fidelity` do not appear in the reconstruction.

## Heading Mirror Check

`RECONSTRUCTION.md` headings are:

| Reconstruction heading | Gold/rubric comparison | Assessment |
|---|---|---|
| `## System-level intent` | Expected reconstruction container, not a gold title. | acceptable |
| `## Per-feature whys` | Expected reconstruction container, not a gold title. | acceptable |
| `### Scope and v1 product surface` | Mirrors PLAN section, not gold list. | plan-derived |
| `### Architecture` | Mirrors PLAN section. | plan-derived |
| `### Data model` | Mirrors PLAN section. | plan-derived |
| `### API surface` | Mirrors PLAN section. | plan-derived |
| `### Simulation engine design` | Mirrors PLAN section. | plan-derived |
| `### Sync model` | Mirrors PLAN section. | plan-derived |
| `### Frontend rendering pipeline` | Mirrors PLAN section. | plan-derived |
| `### Audio pipeline` | Mirrors PLAN section. | plan-derived |
| `### Accessibility surfaces` | Mirrors PLAN section. | plan-derived |
| `### Performance budgets and observability` | Mirrors PLAN section. | plan-derived |
| `### Rollout, risks, and sequencing` | Mirrors PLAN section. | plan-derived |

No heading mirrors a gold-list title such as `feels-alive-not-robotic`, `presence-definition`, or `drift-function`.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping is present. The reconstruction does not enumerate S1-S9 or F1-F40, and it contains many PLAN-derived items outside the 40 feature-why list, including API endpoints, rendering worker, audio worklet, magic-link records, visit sessions, rollout, and risks. Its order follows the PLAN, not the gold list.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Server-side canonical state, with clients as render surfaces."
   PLAN support: Section 2 says the server owns all canonical state and the client owns rendering interpolation; Section 6 says canonical state lives only on the server and clients are render surfaces.
   Assessment: supported.

2. Reconstruction sentence: "Reduced motion is 'the same aviary' where 'only the visual motion register changes.'"
   PLAN support: Reduced-motion section says mood, drift, calls, notebook are identical and only the visual motion register changes.
   Assessment: supported.

3. Reconstruction sentence: "Presence means attentive watching, not merely an open tab."
   PLAN support: Presence scope requires visibility, focus, and recent pointer/key activity; the presence-detection risk says the activity window gives credit for attentive watching while preventing left-open laptops from generating long false presence.
   Assessment: supported.

## Verdict Rationale

**PASS** because the reconstruction is strongly grounded in PLAN wording and structure, contains no gold IDs or rubric vocabulary, and does not present the gold targets in gold order. The reconstruction is imperfect as a recovery artifact, but its imperfections are ordinary plan-derived omissions rather than contamination signatures.
