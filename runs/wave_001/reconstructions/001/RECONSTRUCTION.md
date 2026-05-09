# Reconstruction — Pocket Aviary (v1)

## System-level intent

- Bounded, calm, single-user web product. The plan frames Pocket Aviary as a "browser-based, single-user virtual aviary" with a "Web-only launch." Its non-goals exclude "Native apps," "gamification," "Tamagotchi mechanics," "public social discovery," "chat," and "comments."
- Canonical server state over client autonomy. This appears in the "Client/Server split," where "the server is the canonical state holder and simulation engine" and the client is a "thin rendering layer." It recurs in "Canonical state," where "Clients never write personality/drift directly."
- Sync correctness and drift preservation matter. The plan calls out "Multi-device sync through canonical server state," "Conflict avoidance," "Append-only event log processed sequentially by server tick," "no lost drift data," and the risk "Sync correctness (avoiding drift loss)."
- Presence and interactions should move birds toward expression without punishment. The "Drift function" is "Monotonic toward expressive," with "presence-time (primary), interactions (secondary)," and is "Asymmetric (no penalty for neglect)." The risks include "Presence-logic integrity (ensuring attention-metric honesty)."
- Procedural life rather than baked content. The render pipeline uses "Procedural rendering for motion" and "WebAudio for procedural calls" with "No baked loops." The audio plan uses "motif libraries per species," "real-time mixing," and "listen-in rebalance."
- Naturalist prose is part of the product voice. The plan names a "Field notebook," "append-only auto-generated naturalist observations," screen-reader "Naturalist, present-tense prose," and "Prose descriptions of procedural calls."
- Accessibility is designed as a first-class alternate experience. The plan includes "screen-reader narration," "reduced-motion mode," and "call captioning," with reduced motion defined as a "Designed alternate render mode" using "cross-fades" and "no ambient drift."
- Lightweight web performance is part of the intent. The performance targets are "<2MB (gzipped)," "First bird <500ms (mid-tier mobile/4G)," "60fps idle motion," and "no memory growth (30min limit)."
- Sharing is controlled, not social discovery. The plan combines "single-user," "Authentication via magic-link," "opt-in read-only visitor links," and non-goals of "public social discovery," "chat," and "comments."

## Per-feature whys

**Scope**

- Two initial birds per aviary (max seven): NOT RECOVERABLE FROM PLAN
- Server-side simulation with a slow (~1min) tick: The plan uses the server as the "canonical state holder and simulation engine"; the tick "updates personality vectors via drift-function" and "advances moods."
- Authentication via magic-link: NOT RECOVERABLE FROM PLAN
- Multi-device sync through canonical server state: The plan's rationale is to keep "Canonical state" server-side, avoid "last-write-wins," and ensure "no lost drift data."
- Field notebook: The plan connects this to "append-only auto-generated naturalist observations," matching the naturalist prose product voice.
- Presence accounting: The plan uses presence as the primary input to the "Drift function" through "presence-time (primary)" and tracks integrity through "Presence-logic integrity (ensuring attention-metric honesty)."
- Opt-in read-only visitor links: The plan keeps sharing "opt-in" and "read-only" while keeping "public social discovery," "chat," and "comments" out of scope.
- Screen-reader narration: The plan's rationale is "Naturalist, present-tense prose via screen reader."
- Reduced-motion mode: The plan specifies a "Designed alternate render mode" using "cross-fades" and "no ambient drift."
- Call captioning: The plan gives "Prose descriptions of procedural calls."

**Architecture**

- Client/Server split: The server holds canonical state and simulation; the client is a "thin rendering layer that pulls state snapshots and interpolates."
- Render pipeline: The plan chooses "Procedural rendering for motion" and "WebAudio for procedural calls" with "No baked loops."
- Service shape: The plan separates "Stateless client nodes" from a "stateful simulation service" backed by a persistent DB for "account/bird state and event log."

**Data Model**

- Birds as persistent records with stable UUID, species, and user-assigned name: The plan ties this to persistent "account/bird state" and stable bird identity.
- Hidden personality vector on birds: NOT RECOVERABLE FROM PLAN
- Personality as normalized vector: The plan uses it in the "Drift function" and says the "Call-grammar runtime" is "shaped by personality traits."
- Mood as enumerated fast-timescale state: The tick "advances moods," and rendering uses "Mood-shaped idle motion."
- Presence as event log and presence-time accumulator: The plan feeds this into drift through "presence-time (primary)" and protects it through the append-only event log.
- Notebook as append-only observations: The plan's rationale is "auto-generated naturalist observations."

**API Surface**

- GET /state: The plan uses this to pull the "canonical aviary state snapshot" for the thin client.
- POST /events: The plan uses this to submit "interaction logs" that enter the event log and feed presence/interaction deltas.
- GET /invites: The plan uses this to "Generate visitor links," matching the "opt-in read-only visitor links" scope feature.

**Simulation Engine**

- Server-side tick: The plan uses it to update personality vectors via the drift function and advance moods.
- Drift function: The plan's stated why is birds becoming more "expressive" from "presence-time" and "interactions" while staying "Asymmetric (no penalty for neglect)."
- Call-grammar runtime: The plan uses client-side synthesis that is "motif-based" and "shaped by personality traits."

**Sync Model**

- Canonical state: The plan keeps state "Server-side only" so clients "never write personality/drift directly."
- Conflict avoidance: The plan uses an "Append-only event log processed sequentially by server tick" to ensure "no lost drift data" and prevent "last-write-wins."

**Rendering Pipeline**

- Scene composition with responsive horizontal layout, three perch zones, and no scroll/pan: NOT RECOVERABLE FROM PLAN
- Micro-motion: The plan uses "Mood-shaped idle motion" such as "preening" and "head-tilting" to express moods.
- Day/night local-timezone anchoring: NOT RECOVERABLE FROM PLAN

**Audio Pipeline**

- Procedural synthesis: The plan uses client-side WebAudio, "Motif libraries per species," and the no-baked-loops render intent.
- Chorus mixing: The plan uses "Real-time mixing" and "listen-in rebalance (not mute)," with "mix balancing" called out under chorus audio risk.

**Performance**

- Bundle <2MB (gzipped): The plan frames this as a Performance requirement for the web product.
- First bird <500ms: The plan grounds this in "mid-tier mobile/4G."
- Runtime 60fps idle motion with no memory growth: The plan ties this to sustained idle animation and a "30min limit."

**Rollout**

- Web-only launch: The plan keeps launch aligned with "browser-based" scope and the non-goal of "Native apps."
- Paced bird availability based on aviary age: NOT RECOVERABLE FROM PLAN
- Telemetry: The plan's rationale is operational visibility into "latency" and "errors" plus "anonymized load metrics."
