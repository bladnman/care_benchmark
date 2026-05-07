# Bird engine

The bird engine is the behavioral core of the product — the personality model, the mood system, the call grammar, and the drift function that makes the relationship between the user and a bird actually evolve over weeks. This file specifies what each piece is, how the pieces relate, and what the calibration targets are.

## Personality vector

Each bird has a hidden personality vector — a small set of slowly-drifting numerical traits. The traits are:

- **Boldness** — how readily this bird approaches the front perch versus retreating to the back. High boldness reads as a bird that comes near the viewer; low boldness reads as a bird that watches from a distance.
- **Social warmth** — how often this bird greets the user first; how it reacts to other birds' calls. High warmth is a bird that calls back, that greets first, that perches near other birds.
- **Vocal frequency** — how often the bird calls when unobserved; how readily it joins a chorus.
- **Plumage saturation** — the visual richness of the bird's color and feather detail. Plumage drifts up with sustained attention and (per the rule below) never drifts down.
- **Curiosity** — likelihood to investigate offered items versus ignore them; head-tilting toward new sounds.

The traits are scalar, normalized to a small range, and stored server-side as part of the bird's persistent record. The exact ranges and the seed values for newly-adopted birds are an implementation detail that lives in the simulation service.

## Personality drift

Personality is not static. It drifts. Drift is the cumulative change in trait values across days and weeks of use, and it is the spine of the product's "feels alive over weeks" promise.

Drift is implemented as a low-pass filter over presence-and-interaction signals. The filter is slow on purpose: no single session shifts a trait visibly, but three weeks of regular visits produce a change the user can feel without being told. A drift function that's too fast turns Pocket Aviary into a Tamagotchi where the user can move a number by clicking; a drift function that's too slow turns it into a screensaver where nothing the user does seems to matter. The product lives in the narrow band between those, and the calibration is part of this PRD because no other system in our stack pins it down.

The calibration target is named so it can be tested against: a typical bird should show measurable drift in instruments after about one week of regular visits — small numerical changes the test harness can pick up — and visible drift to the user after about three weeks. The instruments-vs-user gap is intentional. We don't want users noticing changes session-by-session; we want them noticing changes when they look back.

Drift inputs, in rough order of weight:

- **Presence-time** — the dominant input. A user who sits and watches drifts the birds toward expressive.
- **Listen-in** — focusing a bird is a strong signal of attention; that bird's social warmth and vocal frequency drift accordingly.
- **Offers** — accepting an offer is a small drift toward curiosity; offering near a bird at all is a small drift toward boldness.
- **Settle** — the settle gesture is a small mood-quieting signal; it does not push drift in any particular direction beyond ending the presence-window cleanly.

Drift is monotonic toward expressive. Traits move up on positive presence; they do not move down on neglect. A bird that gets ignored does not become more wary, more silent, or less colorful — it becomes ambient: still alive, still calling, but greeting less often because less often is what's been observed.

This is a deliberate departure from the symmetric-drift model a developer would naturally reach for, and it is the load-bearing implementation of the "no Tamagotchi" rule. Punishing absence is the central mistake of every product in this space; it teaches the user that being away is bad, which is exactly the relationship we are not building. The user should be able to leave for two weeks and come back to birds that are quieter than they were, not birds that have learned to mistrust them. The asymmetry is a strong claim and it is right at the engine level — get this wrong and every other charm-protecting decision in this PRD becomes cosmetic.

## Mood

Layered on top of personality is mood — a fast-timescale emotional state that resets on a daily-ish cadence. Mood is what the user actually sees moment-to-moment; personality is what they feel over weeks.

Mood is a small enumerated state per bird (wary, content, curious, drowsy, alert; the exact set is finalized in implementation). Mood transitions are shaped by:

- Recent interactions in the current session (an offer just accepted nudges toward content)
- Time of day in the user's local timezone (drowsy near dusk; alert in early morning)
- Ambient events in the aviary (a passing rain dampens vocal frequency; another bird's alarm call shifts nearby birds toward wary)
- The bird's own personality vector (a high-boldness bird is less likely to enter wary even on the same input)

Mood persists across sessions: the mood a bird has at session-end is the mood it has at the next session-start, modulo whatever the server-side tick has done in the interim. Birds don't reset to neutral when the user opens the tab.

## Calls

Calls are procedural. This is non-negotiable.

Each bird has a procedural call grammar — a small set of motifs combined and varied at runtime, with personality-shaped timing and pitch. Two birds calling at once produce a real chorus, not stacked audio loops. The calls are synthesized client-side via WebAudio (see `accessibility_perf.md` for the constraint behind this).

Procedural rather than recorded matters for two related reasons. First, looped audio is the audible signature of dead software — once the user hears the same call twice, exactly the same way, the spell breaks and they don't recover, even if everything else in the product is doing its job. Second, the chorus mechanic depends on real-time per-call variation: stacking two recorded loops in a mixer produces a characteristic phase-canceling artifact that the ear catches even when each individual call is procedural-sounding. The audio is the affective spine of the product, and the procedural-call rule is what keeps that spine alive.

Each bird's call signature is recognizable across mood and personality drift. A user who has spent two weeks with Pip should know Pip's call by ear, even when Pip is in a different mood, even when Pip's vocal frequency has drifted up. Recognizability is what makes seven the cap on bird count and what makes the per-bird relationship the product is selling actually possible. Calls are timing-shaped by the vocal-frequency trait — a bird with high vocal frequency calls more often when unobserved and joins the chorus more readily.

## Idle motion

Birds are never still in a way that reads as paused. Idle micro-motion includes preening, scanning the scene, head-tilting toward sounds, the small body-shuffle a perched bird does to reset its weight. Idle motion runs continuously regardless of user attention; it does not pause when the tab loses focus from the user's perspective (though see the rendering notes — the client may stop rendering when the tab is hidden, but the simulation continues server-side).

Idle motion is mood-shaped. A wary bird perches further back and scans more; a content bird preens; a curious bird tilts toward sounds and watches passing leaves; a drowsy bird sits low on the perch with feathers fluffed. The point of mood-shaped idle is that the user reads mood from the motion without being told — they see that Pip is wary today and they don't need a label, a tooltip, or a status icon to confirm it. Idle motion is the visible surface of mood, and the moment the user has to be told what a bird is feeling, the product has failed at one of its primary affective contracts.

## Bird species pool

V1 ships with a small species pool — about six species, designed to feel like a coherent set of birds you might see in one place. Each species has its own visual silhouette, default plumage palette, and call-grammar motif library. New birds adopted into an aviary draw from the same pool; species rarity is not a feature.

## Bird naming

Each bird has a user-assigned name. The user picks names at adoption (default suggestions provided; the user can change them). Names are renameable from the bird's settings at any time — Pip can become Pippa next week with no effect on personality, mood, or call.

## Adoption flow

A new account starts with two starter birds. The system selects the two species from the pool; the user does not pick from a catalog. The user can name them at adoption and then again later. The reason for not offering a catalog is small but real: the first encounter should be meeting an animal, not configuring an avatar. The two starters are presented as the birds that arrived, not as the birds the user chose.

## Bird count cap

An aviary tops out at seven birds. The cap is empirical, not arbitrary: seven is the ceiling at which procedural call signatures remain individually recognizable to a typical listener. Above that, the chorus blurs into ambient and the per-bird relationship — knowing Pip from Wren by ear — collapses, which would unravel the rest of the product. We may revisit the cap if future audio-mix work raises the recognizability ceiling, but seven is the v1 limit and it is built into the engine.

## Adding a third bird and beyond

New birds become available based on aviary age. Not visit count, not interaction score, not paid tier — age. A new species offer appears in the user's flow at intervals tied to how long the aviary has existed. The pacing is meant to match the rhythm of a relationship deepening: an aviary a few months old offers a third bird; a year-old aviary may have grown to five or six. The mechanic deliberately refuses to teach the user that more attention earns more stuff, which is the gamification trap the product is shaped to avoid.

## Bird identity

Each bird carries a stable internal identifier separate from its name and species. Renaming a bird, syncing across devices, or any future changes to the species pool never replace one bird with another. The bird the user adopted on day one is *that* bird, with that personality, that drift history, that name, for as long as the account exists.

Identity continuity is the foundation of the personality drift's perceived validity. If the user could ever be told that a bird has been "reset," "regenerated," or "swapped out for an updated version," the entire premise of weeks-long drift evaporates retroactively — what was the point of three weeks of presence-time if the bird isn't the same bird anymore? The stable-id rule lives at the engine layer because it has to be invariant across every client, every sync, every internal migration.

## Personality vector persistence

The personality vector is persisted server-side. It is never derived from session history at runtime, never recomputed from event logs, never rebuilt by the client. It is stored, it is updated by the server-side tick, and it is canonical.

Losing a personality vector amounts to deleting the bird the user has been getting to know — which is the worst possible failure of this product, and the failure most likely to be invisible until a user notices. A reset bird wouldn't fail any unit test; it would just gradually un-reveal itself to a user who would feel something was wrong without being able to name it. This drives the multi-device-canonical model and the no-last-write-wins rule downstream in `accounts_sync.md`: the client never owns personality state, the server is the only writer, and conflict resolution is built around preserving drift, not arbitrating it.

## Personality vector is never exposed numerically

The user never sees personality vector values. Not in a stats panel, not in a debug view, not in a "show me how my bird is doing" surface, not at any version, not in any tier. There is no toggle for it.

The moment a user can see "boldness: 0.62," the bird becomes a number, and the relationship the product is asking the user to form collapses into a stat-management exercise. This is a deliberate exception to the general transparency-is-good principle a developer would otherwise apply: hiding the numbers protects what the numbers are for. Personality should be felt by watching the bird, not read off a dashboard. We are not building a product where the user optimizes traits; we are building a product where the user notices that Pip is bolder than she used to be, and the noticing is the whole point.

## Bird-to-bird interaction

Birds interact with each other, not just with the user. Calls from one bird can prompt responses from another; a wary mood in one bird tends to spread; chorus events emerge when two or more birds with high vocal frequency happen to be calling in the same window. Bird-to-bird interaction is what makes the aviary feel like a small social system rather than a row of independent NPCs.

## Mood persistence across sessions

Mood does not reset when the user opens the tab. The mood a bird is in at session-end is the mood it is in at session-start, modulated by whatever the server-side tick has computed in the interim. A bird that ended yesterday's session in drowsy and was at dusk-time on the server will likely be settled or sleeping by morning; a bird that ended in wary will likely have softened toward content if the time-of-day signal pushed that way. The user should never notice mood "snapping" to a default on tab open.
