## System-level intent

- Core-first v1 prototype scope: The plan frames the rollout as "v1" and says it will focus on "establishing the core" engine, simulation, and interaction layer. This shows up in `Scope`, especially the contrast between "In-scope" core pieces and "Out-of-scope" complex, advanced, persistent, or third-party work.
- Real-time, server-centered synchronization: The plan repeatedly carries "synchronous aviary simulation," "state synchronization," "real-time state updates," "event propagation," "delta updates," and "Canonical server state pushed to all clients." This shows up across `Scope`, `Architecture`, `API Surface`, and `Sync Model`.
- Bird behavior evolves through drift, but must stay predictable: The plan uses "bird-evolution," "personality drift," and a "Drift function calculating mood change based on environmental and interaction inputs." The `Risks` section makes the philosophy explicit as a balance between "realistic evolution and predictable simulation."
- Responsive interaction and birdsong: The plan connects "basic bird calls," "interaction events," "listen-in," "notebook," "responsive birdsong," and "sub-100ms response time." This shows up in `Scope`, `Audio Pipeline`, and `Performance`.
- Accessible presence and calls are part of the product surface: The plan calls for "Full screen-reader support via live regions for presence events and bird interactions" and "Captions for all procedural calls" in `Accessibility`.
- Performance budgets shape implementation choices: The plan names a tick "frequency configurable per performance budget," "CSS-based scene composition," "Bundle size limit: <500kb," "60fps for UI," and "sub-100ms response time." This appears in `Simulation Engine Design`, `Rendering Pipeline`, and `Performance`.

## Per-feature whys

### Scope

- Core "bird-evolution" engine: The v1 rollout focuses on "establishing the core" engine.
- Synchronous aviary simulation: The v1 rollout focuses on the "synchronous aviary simulation," with "local state sync" and later "Canonical server state pushed to all clients."
- Basic multi-device interaction layer: The v1 rollout focuses on the "basic multi-device interaction layer" and "initial interaction surfaces."
- Core simulation loop (ticks): The plan includes ticks as part of the core simulation loop; later it specifies a "Periodic server-side tick" with frequency "configurable per performance budget."
- Personality drift: The plan includes "personality drift" and later defines drift as calculating "mood change based on environmental and interaction inputs."
- Basic bird calls: The plan includes "basic bird calls" and later uses "Procedural call synthesis using WebAudio API for responsive birdsong."
- Local state sync: The rationale is real-time synchronization: "Canonical server state pushed to all clients via WebSockets."
- Listen-in: NOT RECOVERABLE FROM PLAN
- Notebook: The data model defines "Notebook (interaction logs)," so its articulated purpose is logging interactions.
- Complex social features: The plan excludes them from v1 because the v1 focus is the core engine, synchronous aviary simulation, and basic multi-device interaction layer.
- Persistence beyond basic session state: The plan excludes it from v1 as "Out-of-scope," keeping only "basic session state."
- Advanced atmospheric rendering: The plan excludes it from v1 as "Out-of-scope" while using "CSS-based scene composition for idle micro-motion."
- Non-essential third-party integrations: The plan excludes them from v1 as "Out-of-scope" and names them as "non-essential" and "per non-goals."

### Architecture

- Node.js/FastAPI server: The plan says the server is "for the simulation engine and state synchronization."
- React-based frontend: The plan says the frontend is for "interactive aviary visualization."
- WebSockets: The plan says WebSockets are for "real-time state updates and event propagation."

### Data Model

- Bird entity: NOT RECOVERABLE FROM PLAN
- Aviary entity: NOT RECOVERABLE FROM PLAN
- Notebook entity: The plan defines it as "interaction logs."

### API Surface

- REST: The plan says REST is for "initial handshake and authenticated state fetch."
- WebSocket: The plan says WebSocket is for "delta updates and interaction events."

### Simulation Engine Design

- Periodic server-side tick: The plan says tick frequency is "configurable per performance budget."
- Drift function: The plan says it calculates "mood change based on environmental and interaction inputs."

### Sync Model

- Canonical server state pushed to all clients: The articulated rationale is shared state across clients: "pushed to all clients via WebSockets."
- Client-side optimistic updates: The plan says they are "reconciled with server-confirmed events."

### Rendering Pipeline

- CSS-based scene composition: The plan says it is for "idle micro-motion."
- React state: The plan says it "manages bird positions and presence events."

### Audio Pipeline

- Procedural call synthesis: The plan says WebAudio API is used "for responsive birdsong."

### Accessibility

- Live regions: The plan says they provide "Full screen-reader support" for "presence events and bird interactions."
- Captions: The plan says captions are for "all procedural calls."

### Performance

- Bundle size limit: NOT RECOVERABLE FROM PLAN
- 60fps UI target: NOT RECOVERABLE FROM PLAN
- Sub-100ms response time: The plan says the target is for "interaction events."

### Risks

- Drift calibration: The plan says the risk is "Difficulty finding the balance between realistic evolution and predictable simulation."
- Sync latency: The plan says the risk is visual "stutter" if "WebSocket updates are delayed."
