## System-level intent

- Attention-responsive aviary: The plan centers Pocket Aviary as a "browser-based virtual aviary featuring animated birds that respond to user attention." This shows up again in "presence-time and interaction signals," "offer, listen-in, presence," and Presence as a conjunction of visible state, focus, and recent user activity.
- Server-authoritative canonical state: The plan repeatedly protects one canonical truth. It names "server-side tick," "Server-Authoritative," "Server is the only writer of state," "canonical snapshots," "No client-client sync," and "Additive Deltas" to "prevent race conditions."
- Append-only history as the source of change: The plan uses append-only logs for both "system-generated observations" and "interaction gestures." The simulation "advances based on interaction event log," and Sync Correctness depends on "Maintaining event log order."
- Expressive change without punitive reinforcement: Birds have a "personality vector (drift substrate)" and a "mood (current expression)." Drift "monotonically drifts toward expressive," while v1 excludes "gamification" and "Tamagotchi-style negative reinforcement (birds dying, decaying happiness)."
- Calm, accessible presentation: The front end is a "Single horizontal scene, no panning/zoom," with "Reduced-Motion," "screen-reader narration," "captioning," and "Keyboard focusable perches/birds."
- Alive naturalist voice across modalities: The plan pairs "Naturalist prose narration," "Prose descriptions of procedural calls," "Idle micro-motion driven by mood," and the risk of ensuring procedural synthesis "feels alive."
- Performance on constrained devices: The plan makes performance part of the product contract with "<2MB (gzipped) initial JS load," "TTFBird: <500ms on 4G mid-tier mobile," and "60fps on 5-year-old laptop."

## Per-feature whys

Scope

- Browser-based virtual aviary: NOT RECOVERABLE FROM PLAN
- Animated birds that respond to user attention: The plan makes "user attention" the response target, with presence-time and interaction signals feeding Drift.
- Two to seven birds: NOT RECOVERABLE FROM PLAN
- Multi-device sync: The plan's rationale is canonical consistency: "Server-Authoritative," "All clients pull canonical snapshots," and "No client-client sync."
- Field notebook: The rationale given is to keep an "Append-only log of system-generated observations."
- Presence accounting: Presence is needed because Drift is based on "presence-time and interaction signals." The plan defines presence as `visibilityState: visible`, window focus, and recent pointer/key activity.
- Opt-in read-only visits: NOT RECOVERABLE FROM PLAN
- Screen-reader narration: The plan says narration is "generated for screen readers" and uses "Naturalist prose narration."
- Reduced-motion: The plan provides an alternative to frame animation: "Cross-fading still poses instead of frame-animated paths."
- Captioning: The plan captions audio through "Prose descriptions of procedural calls."

Architecture

- React/TypeScript SPA architecture: NOT RECOVERABLE FROM PLAN
- Node.js/FastAPI backend with server-side tick: The backend hosts the "simulation engine running server-side tick," and the Server Tick advances "canonical simulation state."
- REST for snapshot delivery: The plan uses REST to deliver snapshots that clients pull as "canonical snapshots."
- Append-only event log for interaction submissions: Interaction submissions enter an append-only log; the server tick advances from that event log, and ordered event history prevents personality drifts from conflicting.
- Browser-based rendering with Canvas/SVG: NOT RECOVERABLE FROM PLAN
- Client interpolation between server snapshots: The plan says interpolation exists "for smooth motion."

Data Model

- Account with Synthetic UUID: NOT RECOVERABLE FROM PLAN
- Magic-link auth: NOT RECOVERABLE FROM PLAN
- Bird canonical identity (UUID): NOT RECOVERABLE FROM PLAN
- Bird species: NOT RECOVERABLE FROM PLAN
- Personality vector: The rationale is explicit: it is the "drift substrate."
- Mood: The rationale is explicit: it is the bird's "current expression."
- Notebook append-only log: The rationale is to record "system-generated observations" as an append-only log.
- Events append-only log: The rationale is to capture "interaction gestures (offer, listen-in, presence)" for interaction submissions and simulation.

Simulation Engine

- Server Tick: It advances "Canonical simulation state" from the "interaction event log."
- Drift: It makes the personality vector "monotonically" move "toward expressive" based on presence-time and interaction signals, with calibration balancing user-visible change in "~3 weeks" against testing speed.
- Mood: It is a "Fast-timescale state" influenced by personality, time-of-day, ambient events, and interactions.
- Presence: It combines `visibilityState: visible`, window focus, and recent user activity so presence can serve as an interaction signal.

Sync Model

- Server-Authoritative: The rationale is that "Server is the only writer of state."
- No client-client sync: The rationale is that "All clients pull canonical snapshots from server."
- Additive Deltas: The rationale is to "prevent race conditions."

Frontend Rendering Pipeline

- Single horizontal scene with no panning/zoom: NOT RECOVERABLE FROM PLAN
- Idle micro-motion: It is "driven by mood," making mood the driver of animation.
- Transitions: Interpolation between canonical snapshots exists "for smooth motion."
- Reduced-Motion: It replaces frame-animated paths with "Cross-fading still poses."

Audio Pipeline

- Procedural call synthesis using WebAudio: NOT RECOVERABLE FROM PLAN
- Runtime-varied motif combination: The plan uses it for "Multi-bird chorusing."
- Listen-in: The mix re-balance raises the "focused bird" and quiets the others.

Accessibility

- Naturalist prose narration: The rationale is "for screen readers."
- Captioning: The rationale is to provide "Prose descriptions of procedural calls."
- Keyboard focusable perches/birds: The rationale is keyboard navigation.
- Top-bar shortcut reachability: The rationale is shortcut reachability.

Performance

- <2MB gzipped initial JS load: NOT RECOVERABLE FROM PLAN
- TTFBird under 500ms on 4G mid-tier mobile: NOT RECOVERABLE FROM PLAN
- Idle Motion at 60fps on 5-year-old laptop: The rationale is smooth idle motion on older hardware.
