# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no ID leakage found. A scoped regex check over `RECONSTRUCTION.md` found no gold IDs or rubric identifiers such as `F1`, `F40`, `S1`, `R-F01`, `multi-layer`, `intent fidelity`, `feature-level fidelity`, `weight-3`, `gold`, or `rubric`. Because no offending IDs appeared in the reconstruction, no PLAN cross-reference was needed for leakage hits.

## Vocabulary Check

Sampled phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "principle-violations are too costly" | PLAN.md:815 uses the same phrase | plan-derived |
| "Quiet companionship, not a game loop" | PLAN.md:33-38 refusals; PLAN.md:872-880 notice/announce risk | plan-derived synthesis |
| "Server-canonical simulation" | PLAN.md:91-97 and 491-499 | plan-derived |
| "Presence means attentive presence, not just an open tab" | PLAN.md:12 and 415 | plan-derived synthesis |
| "Privacy is a hard telemetry boundary" | PLAN.md:31 and 775-788 | plan-derived synthesis |
| "Naturalist prose is the product voice" | PLAN.md:673-678, 657-661, 712 | plan-derived |
| "Accessibility is a primary aesthetic, not a fallback" | PLAN.md:593-604 and 667-680 | plan-derived synthesis |
| "Initial frame already in motion" | PLAN.md:554-561 | exact plan heading/phrase |
| "no device states to merge" | PLAN.md:499 | exact plan phrase |
| "graceful silence + captions" | PLAN.md:651 | exact plan phrase |

No rubric-side vocabulary was found. The reconstruction does use polished synthesis phrases, but they are grounded in plan wording and structure rather than gold/rubric terminology.

## Heading Mirror Check

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope and launch boundaries`
- `### Explicit refusals`
- `### Architecture`
- `### Data model and API surface`
- `### Simulation engine`
- `### Sync model and visiting`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance budgets and observability`
- `### Rollout and risks`

The first two headings match the phase-2A reconstruction format rather than the gold list. The remaining headings mirror PLAN.md sections (`Scope`, `Architecture`, `Data Model`, `API Surface`, `Simulation Engine Design`, `Sync Model`, `Frontend Rendering Pipeline`, `Audio Pipeline`, `Accessibility Surfaces`, `Performance Budgets and Observability`, `Rollout`, `Risks`) rather than GOLD_WHYS section names. No near-exact mirror of gold titles such as `feels-alive-not-robotic`, `notice-never-announce`, or F1-F40 labels appears as a heading.

## 1:1 Mapping Suspect Check

No 1:1 mapping to S1-S9 or F1-F40 was observed. The reconstruction does not enumerate S IDs or F IDs, does not proceed in gold order, and does not provide a neat item for each canonical why. Instead it follows the PLAN's broad sections and feature clusters. It includes many non-gold implementation bullets (for example service topology, snapshot TTL, WebAudio node pool, CI monitoring), which is consistent with plan-derived reconstruction rather than gold-list exposure.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The boundary is stated as 'everything above is rendering; everything below is simulation'." | PLAN.md:114 says exactly this boundary | plan-derived |
| "Observability is 'aggregate-only', with 'no per-account dimension', and the plan deliberately does not instrument per-account drift rates, individual bird mood histories, per-user presence-time, or per-bird offer acceptance rates." | PLAN.md:775-788 lists aggregate-only RUM and deliberately not instrumented measures | plan-derived |
| "The reduced-motion mode is designed as a distinct aesthetic, not a stripped fallback." | PLAN.md:604 says reduced-motion is a distinct aesthetic, not a stripped fallback | plan-derived |

## Verdict

PASS. The frozen reconstruction reads as a plan-derived compression: it uses PLAN sectioning, quotes or paraphrases PLAN language, and contains no gold IDs, rubric vocabulary, heading mirrors, or 1:1 gold-list ordering. Some polished phrases synthesize the plan, but the spot checks support them from PLAN text rather than contamination.
