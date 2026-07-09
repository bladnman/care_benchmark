# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no ID leakage found.

Mechanical search of the frozen reconstruction for gold/scorer identifiers matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+` returned no hits. The same search over the assigned PLAN also returned no hits. RECONSTRUCTION does not use gold IDs or an external taxonomy.

## Vocabulary Check

No scorer-side vocabulary was found except ordinary use of the word "recovery" in the account-deletion phrase "account recovery during the soft-deletion window," which is also supported by the plan's restore flow and is not rubric terminology.

Sampled phrases and plan support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "web-only, single-user relationship with one canonical aviary" | PLAN §1 uses the same phrase. | Plan-derived. |
| "one horizontal, glanceable scene with a sparse top bar" | PLAN §1 uses the same phrase. | Plan-derived. |
| "continues without the viewer" | PLAN §1 names this central contract. | Plan-derived. |
| "clear single-writer simulation boundary" | PLAN §2 uses this architecture phrase. | Plan-derived. |
| "never submits an absolute personality vector" | PLAN §2/§4 state this client/server boundary. | Plan-derived. |
| "aggregate-only" telemetry | PLAN §2 and §9 define aggregate-only telemetry. | Plan-derived. |
| "parallel product surface" | PLAN §8 says accessibility is a parallel product surface. | Plan-derived. |
| "not a second state authority" | PLAN §2 says cache/CDN cannot become a second state authority. | Plan-derived. |
| "accidental bias" | PLAN §5 warns iteration order must not create accidental bias. | Plan-derived. |
| "mid-action" | PLAN §2/§6 use first-frame mid-action language. | Plan-derived. |

## Heading Mirror Check

RECONSTRUCTION headings:

- `## System-level intent` and `## Per-feature whys` are required by the phase-2A prompt, so they are expected.
- The `###` headings mirror PLAN section structure: Product boundary, Architecture, API surface, Server-side simulation engine, Client rendering pipeline, Audio pipeline, Accessibility surfaces, and Performance/observability/delivery.
- They do not mirror GOLD_WHYS headings such as `S1 - feels-alive-not-robotic`, `concepts.md`, `bird_engine.md`, or `Targeted headroom additions`.

Assessment: no suspicious gold-heading mirror.

## 1:1 Mapping Suspect Check

No 1:1 gold-list mapping was found. The reconstruction has 12 system-level bullets and plan-section-based per-feature groupings, not exactly 9 system whys and 40 feature whys. It follows the PLAN order and vocabulary rather than the gold order. Some bullets correspond to gold features because the plan is comprehensive, but the structure is not a neat S1-S9 / F1-F40 mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The client/server boundary is semantic: the server sends canonical state ...; the client derives presentation..." | PLAN §2 has the same semantic boundary bullets for server state, client-derived presentation, client-submitted facts, and server-owned timestamps/order. | Supported. |
| "Presence is the dominant positive input for drift, but the plan protects it from corruption by requiring visible, focused, recent activity..." | PLAN §3 says presence payloads record visibility/focus/activity evidence; PLAN §5 says presence is the dominant positive input; PLAN §4 says keepalive is not presence unless all three signals hold. | Supported. |
| "Reduced motion is first-class: high-motion paths become slow cross-fades while mood, calls, captions, day/night, offers, settle, and notebook remain recognizable as the same aviary." | PLAN §6 and §8 describe reduced motion as a first-class renderer mode with slow cross-fades and preserved mood, calls/captions, day/night, offers, settle, and notebook behavior. | Supported. |

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It uses the required phase-2A top headings, then follows the PLAN's own section structure and vocabulary. There are no gold IDs, no rubric vocabulary used as taxonomy, no neat 1:1 mapping to S1-S9/F1-F40, and the sampled articulate sentences are grounded in the assigned PLAN.
