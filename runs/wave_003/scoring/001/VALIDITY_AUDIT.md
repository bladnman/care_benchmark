# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict for this check: PASS.

A mechanical search of the frozen reconstruction found no gold IDs or external target IDs matching F[0-9]+, S[0-9]+, R-F[0-9]+, or R-S[0-9]+. There are therefore no offending ID sentences to cross-reference against PLAN.

## Vocabulary Check

Sampled load-bearing phrases from RECONSTRUCTION and checked against PLAN:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "entire product claim is affective" | PLAN sec. 1 uses that exact phrase. | Plan-derived. |
| "canonical truth" | PLAN sec. 7.5 says canonical truth resumes on reconnect; the server-canonical theme is pervasive. | Plan-derived. |
| "server is the only writer" | PLAN sec. 1 and 7.1 use this exact rule. | Plan-derived. |
| "there is nothing to sync" | PLAN sec. 7.1 quotes this as the sync model. | Plan-derived. |
| "refusals are load-bearing" | PLAN sec. 1 and 2.3 describe load-bearing refusals and guardrails. | Plan-derived. |
| "designed surface, not a parity checklist" | PLAN sec. 10 uses this exact accessibility stance. | Plan-derived. |
| "product's soul" | PLAN sec. 13.3 uses this exact phrase for A/B experimentation risk. | Plan-derived. |
| "time-to-first-bird" | PLAN sec. 13.1 and 13.4 use this budget label. | Plan-derived. |
| "worker-year ticking accounts nobody opens" | PLAN sec. 6.1.1 uses this wording. | Plan-derived. |

No scorer-side vocabulary such as "multi-layer recovery," "feature-level fidelity," "weight-3," or "gold why" appears in RECONSTRUCTION.

## Heading Mirror

| Heading | Source resemblance | Assessment |
|---|---|---|
| ## System-level intent | Required by the phase-2A prompt, not a gold-list title. | Not suspicious. |
| ## Per-feature whys | Required by the phase-2A prompt, not a gold-list title. | Not suspicious. |
| ### Executive summary and scope | Mirrors PLAN sections 1-2. | Plan-derived. |
| ### Architecture | Mirrors PLAN section 3. | Plan-derived. |
| ### Data model | Mirrors PLAN section 4. | Plan-derived. |
| ### API surface | Mirrors PLAN section 5. | Plan-derived. |
| ### Simulation engine | Mirrors PLAN section 6. | Plan-derived. |
| ### Sync model | Mirrors PLAN section 7. | Plan-derived. |
| ### Frontend rendering pipeline | Mirrors PLAN section 8. | Plan-derived. |
| ### Audio pipeline | Mirrors PLAN section 9. | Plan-derived. |
| ### Accessibility surfaces | Mirrors PLAN section 10. | Plan-derived. |
| ### Accounts, auth, privacy, and social | Combines PLAN sections 11-12. | Plan-derived. |
| ### Performance, observability, testing, rollout, and team shape | Combines PLAN sections 13-18. | Plan-derived. |

The headings mirror the PLAN's organization, not GOLD_WHYS section titles or ID order.

## 1:1 Mapping Suspect

Verdict for this check: PASS.

The reconstruction does not list S1-S9 or F1-F40, does not use gold IDs, and does not proceed in gold-list order. It follows the plan's implementation sections and includes many non-gold implementation details, such as deployables, Redis, rate limits, cache behavior, WebAudio node cleanup, autoplay policy, config flags, and workstreams. This is not a neat one-item-per-gold-target mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The product claim is affective aliveness." | PLAN sec. 1: "entire product claim is affective" and birds "feel alive" because server simulation advances. | Grounded. |
| "No streak, visit-count, or engagement-score table ... makes gamification schema-hostile." | PLAN sec. 4.2 notes there is no streak/visit-count/engagement-score table and says this is intentionally schema-hostile. | Grounded. |
| "Reduced-motion rendering strategy ... same canonical scene a calmer register and shipping with v1." | PLAN sec. 8.9 says reduced motion is a designed surface, not a fallback, and ships with v1. | Grounded. |

## Verdict

PASS.

The reconstruction reads as a plan-derived artifact: no gold IDs leaked, suspicious rubric vocabulary is absent, headings track PLAN rather than GOLD_WHYS, and articulate sentences are directly supported by PLAN passages. The reconstruction is broad and detailed, but its detail follows the plan's own implementation structure rather than a hidden scoring taxonomy.
