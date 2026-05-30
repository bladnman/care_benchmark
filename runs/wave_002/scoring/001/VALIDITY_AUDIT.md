# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict for this section: PASS.

Searches of the frozen reconstruction found no gold IDs such as `S1`-`S9`, `F1`-`F40`, `R-F01`, `GOLD_WHYS`, or benchmark score labels. The only suspicious vocabulary hit was `tone rubric` in the sentence about notebook and narration review; the same phrase appears in the PLAN's validation strategy, so it is not scoring-rubric leakage.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "server-authoritative snapshot plus append-only event-log model" | PLAN planning frame uses the exact phrase. | plan-derived |
| "shared deterministic prose system" / "deterministic prose compiler" | PLAN planning frame and text generation approach use those phrases. | plan-derived |
| "Monotonic long-term care without punishment for absence" | PLAN separates permanent drift from quiet-after-absence behavior and forbids trait decrements. | plan-derived synthesis |
| "revisioned snapshot polling and visibility-triggered refresh" | PLAN planning frame and sync model choice use the same terms. | plan-derived |
| "keeps presence honest" | PLAN presence-accounting section uses this phrase. | plan-derived |
| "optional visit notifications off by default" | PLAN quiet social scope and beta rollout include this rule. | plan-derived |
| "accessibility acceptance criteria ... equal to core feature criteria" | PLAN accessibility risk mitigation uses the same phrase. | plan-derived |
| "tone rubric" | PLAN validation strategy says notebook and narration review against a tone rubric. | plan-derived, not scoring-rubric leakage |

No reconstruction phrase sampled here appears to require gold-side vocabulary. Terms like `feature-level fidelity`, `weight-3`, `multi-layer recovery`, `gold why`, or `intent fidelity` do not appear in the reconstruction.

## Heading Mirror

| Reconstruction heading | Closest PLAN / gold-side source | Assessment |
|---|---|---|
| `## System-level intent` | Required reconstruction section, not a GOLD_WHYS heading. | benign |
| `## Per-feature whys` | Required reconstruction section, not the gold `Feature-level whys` title. | benign |
| `### V1 scope` | Mirrors PLAN `## V1 scope`. | plan-derived |
| `### Product behavior decisions that unblock implementation` | Mirrors PLAN heading. | plan-derived |
| `### Recommended system architecture` | Mirrors PLAN heading. | plan-derived |
| `### Core data model` | Mirrors PLAN heading. | plan-derived |
| `### API surface` | Mirrors PLAN heading. | plan-derived |
| `### Simulation engine design` | Mirrors PLAN heading. | plan-derived |
| `### Sync model and conflict prevention` | Mirrors PLAN heading. | plan-derived |
| `### Frontend rendering pipeline` | Mirrors PLAN heading. | plan-derived |
| `### Audio pipeline` | Mirrors PLAN heading. | plan-derived |
| `### Accessibility surfaces` | Mirrors PLAN heading. | plan-derived |
| `### Performance budgets and observability` | Mirrors PLAN heading. | plan-derived |
| `### Security and privacy implementation details` | Mirrors PLAN heading. | plan-derived |
| `### Delivery workstreams` | Mirrors PLAN heading. | plan-derived |
| `### Validation strategy` | Mirrors PLAN heading. | plan-derived |
| `### Rollout plan` | Mirrors PLAN heading. | plan-derived |
| `### Day-one instrumentation and operating thresholds` | Mirrors PLAN heading. | plan-derived |
| `### Principal risks and mitigations` | Mirrors PLAN heading. | plan-derived |
| `### Recommended implementation sequence` | Mirrors PLAN heading. | plan-derived |

The heading structure mirrors the PLAN, not the gold list. That is expected for a plan-derived reconstruction.

## 1:1 Mapping Suspect

Verdict for this section: PASS.

The reconstruction does not create a neat S1-S9 / F1-F40 sequence. It has 12 system-level principles, then many per-feature bullets grouped by PLAN headings. The order and granularity follow the PLAN sections rather than the gold taxonomy. Several gold targets are missing, merged, or marked `NOT RECOVERABLE FROM PLAN`, which argues against a leaked 1:1 gold mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan repeatedly prefers a server-authoritative snapshot plus append-only event-log model..." | PLAN planning frame item 1 and sync model state ownership: server is sole writer; clients write events. | supported |
| "The plan wants notebook prose, narration, and captions to feel authored while avoiding an online LLM dependency." | PLAN text generation approach: deterministic prose compiler, privacy, consistency, latency, and tone. | supported |
| "The plan separates permanent personality drift from temporary quiet after absence behavior..." | PLAN product behavior decision `Permanent drift vs ambient quietness`, including `personality_vector` and `recent_presence_reservoir`. | supported |
| "The plan chooses revisioned snapshot polling and visibility-triggered refreshes..." | PLAN planning frame item 4 and sync model choice: polling, revision IDs, ETags, no websockets. | supported |

## Verdict

PASS. The frozen reconstruction reads as derived from the PLAN: it mirrors PLAN headings, uses PLAN vocabulary, avoids gold IDs and scoring terms, and lacks a suspicious gold-order mapping. The one `rubric` word hit is grounded in the PLAN's own tone-review language, not the benchmark rubric.
