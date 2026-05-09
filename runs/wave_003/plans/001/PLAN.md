# Comprehensive Implementation Plan - Pocket Aviary (V1)

## Scope
Pocket Aviary is a browser-based, single-user virtual aviary. V1 focuses on the core observational relationship between a user and their birds, strictly adhering to the principle of "notice, never announce."

- **In Scope:** 
  - 2 to 7 birds per aviary.
  - Multi-device sync (single-user).
  - Procedural call engine (WebAudio).
  - Personality drift and daily mood system.
  - Read-only visit invitation feature (opt-in).
  - Field notebook (naturalist, auto-generated).
  - Reduced-motion mode and call captioning.
- **Non-Goals (Absolute):** No gamification (streaks, badges, quests), no native apps, no social network discovery/profiles, no Tamagotchi-style neglect penalties, no notifications.

## Architecture
- **Client/Server Split:** Server-side simulation (canonical state, personality vectors, mood, drift calculations). Thin-client rendering and event-capturing.
- **Render Pipeline:** Single horizontal scene with parallax, no-chrome view, procedural animation, client-side WebAudio synthesis.
- **Service Shape:** 
  - Auth Service: Magic-link email auth.
  - Simulation Service: Periodic tick (~1min) processing event logs to update state.
  - Sync Service: Snapshot-based delivery to clients.

## Data Model
- **Account:** Synthetic UUID, magic-link auth, account settings.
- **Bird:** Stable internal ID, user-assigned name, species, personality vector (hidden), mood state (fast-timescale).
- **Presence:** Conjunction of (visibilityState: visible, window focus, pointer/key activity).
- **Interaction Log:** Append-only events (listen-in, offer, settle).
- **Notebook Entry:** Naturalist prose, generated at low frequency.

## Simulation Engine
- **Tick Engine:** Server-side, ~1min cadence. Computes drift from presence logs, handles mood transitions.
- **Drift Function:** Monotonic toward expressive; traits move up with presence, never down with neglect.
- **Audio Engine:** Procedural grammar, synthesized client-side, chorus mixing, WebAudio fallback (silence + captions).

## Sync Model
- **Canonical State:** Server-side record is the source of truth.
- **Updates:** Client writes interaction events; server applies deltas. No client-to-client or client-writes-state-directly allowed.

## Frontend Rendering
- **Pipeline:** Client interpolates between snapshots for smooth motion.
- **Constraints:** Initial JS bundle <2MB. First bird visible <500ms. 60fps idle motion. Memory growth zero over 30min session.

## Accessibility
- **Narration:** Naturalist running prose, low-frequency updates, screen-reader optimized.
- **Reduced-Motion:** Designed cross-fade sequences instead of frame-by-frame animation.
- **Captions:** Procedural call descriptions, written in naturalist voice.
- **Navigation:** Keyboard-navigable focus, high-contrast outlines for birds.

## Risks
- **Drift Calibration:** Risk of too fast (gamified feel) or too slow (static feel). Mitigation: Instrument, tune.
- **Audio Uncanniness:** If chorus effect breaks or sounds looped. Mitigation: Strict adherence to procedural synthesis.
- **Sync/State Divergence:** Mitigated by client-never-writes-state architecture.
