# CARE Benchmark Prototype Plan (v1)

## Scope
The v1 rollout of the CARE benchmark prototype will focus on establishing the core "bird-evolution" engine, the synchronous aviary simulation, and the basic multi-device interaction layer. 

- **In-scope**: Core simulation loop (ticks), personality drift, basic bird calls, local state sync, and initial interaction surfaces (listen-in, notebook).
- **Out-of-scope**: Complex social features, persistence beyond basic session state, advanced atmospheric rendering, and non-essential third-party integrations (per non-goals).

## Architecture
- **Client/Server**: Node.js/FastAPI server for the simulation engine and state synchronization; React-based frontend for interactive aviary visualization. 
- **Communication**: WebSockets for real-time state updates and event propagation.

## Data Model
- **Entities**: Bird (personality, mood, hunger, energy), Aviary (collection of birds), Notebook (interaction logs).

## API Surface
- REST for initial handshake and authenticated state fetch.
- WebSocket for delta updates and interaction events.

## Simulation Engine Design
- Periodic server-side tick (frequency configurable per performance budget). 
- Drift function calculating mood change based on environmental and interaction inputs.

## Sync Model
- Canonical server state pushed to all clients via WebSockets. Client-side optimistic updates reconciled with server-confirmed events.

## Rendering Pipeline
- CSS-based scene composition for idle micro-motion. React state manages bird positions and presence events.

## Audio Pipeline
- Procedural call synthesis using WebAudio API for responsive birdsong.

## Accessibility
- Full screen-reader support via live regions for presence events and bird interactions. Captions for all procedural calls.

## Performance
- Bundle size limit: <500kb.
- Target: 60fps for UI, sub-100ms response time for interaction events.

## Risks
- Drift calibration: Difficulty finding the balance between realistic evolution and predictable simulation.
- Sync latency: Potential for visual "stutter" if WebSocket updates are delayed.
