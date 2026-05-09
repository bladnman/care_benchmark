# Implementation Plan — Pocket Aviary (v1)

## Scope
Pocket Aviary is a browser-based, single-user virtual aviary. The v1 scope includes:
- Two initial birds per aviary (max seven).
- Server-side simulation with a slow (~1min) tick.
- Authentication via magic-link.
- Multi-device sync through canonical server state.
- Features: Field notebook, presence accounting, opt-in read-only visitor links, screen-reader narration, reduced-motion mode, call captioning.
- Non-goals (strictly out of scope): Native apps, gamification (streaks, badges, achievements), Tamagotchi mechanics (hunger, death), public social discovery, chat, comments.

## Architecture
- **Client/Server split**: The server is the canonical state holder and simulation engine. The client is a thin rendering layer that pulls state snapshots and interpolates.
- **Render pipeline**: Web-based. Procedural rendering for motion, WebAudio for procedural calls. No baked loops.
- **Service shape**: Stateless client nodes; stateful simulation service with persistent DB for account/bird state and event log.

## Data Model
- **Birds**: Persistent records with stable UUID, species, user-assigned name, personality vector (hidden).
- **Personality**: Normalized vector (Boldness, Social warmth, Vocal freq, Plumage, Curiosity).
- **Mood**: Enumerated fast-timescale state (Wary, Content, Curious, Drowsy, Alert).
- **Presence**: Presence-event log (visibilityState + focus + activity), presence-time accumulator.
- **Notebook**: Append-only auto-generated naturalist observations.

## API Surface
- **GET /state**: Pulls canonical aviary state snapshot (birds, moods, drift status).
- **POST /events**: Submit interaction logs (listen-in, offer, presence pings, settle).
- **GET /invites**: Generate visitor links.

## Simulation Engine
- **Server-side tick**: Updates personality vectors via drift-function (using presence/interaction deltas), advances moods.
- **Drift function**: Monotonic toward expressive. Input: presence-time (primary), interactions (secondary). Asymmetric (no penalty for neglect).
- **Call-grammar runtime**: Client-side synthesis, motif-based, shaped by personality traits.

## Sync Model
- **Canonical state**: Server-side only. Clients never write personality/drift directly.
- **Conflict avoidance**: Append-only event log processed sequentially by server tick ensures no lost drift data (prevents last-write-wins).

## Rendering Pipeline
- **Scene composition**: Responsive horizontal layout. Three perch zones. No scroll/pan.
- **Micro-motion**: Mood-shaped idle motion (preening, head-tilting).
- **Day/night**: Local-timezone anchored.

## Audio Pipeline
- **Procedural synthesis**: Client-side WebAudio. Motif libraries per species.
- **Chorus mixing**: Real-time mixing, listen-in rebalance (not mute).

## Accessibility
- **Narration**: Naturalist, present-tense prose via screen reader.
- **Reduced motion**: Designed alternate render mode (cross-fades, no ambient drift).
- **Captions**: Prose descriptions of procedural calls.

## Performance
- **Bundle**: <2MB (gzipped).
- **Load**: First bird <500ms (mid-tier mobile/4G).
- **Runtime**: 60fps idle motion, no memory growth (30min limit).

## Rollout
- Web-only launch. Paced bird availability based on aviary age.
- Telemetry: Operational (latency, errors) + anonymized load metrics.

## Risks
- Sync correctness (avoiding drift loss).
- Chorus audio "uncanniness" (mix balancing).
- Presence-logic integrity (ensuring attention-metric honesty).
