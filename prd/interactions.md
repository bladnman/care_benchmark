# Interactions

This file describes what a session looks like — what the user can do, what counts as interaction, what the system does in response. Most of these are short descriptions, because most of the affective intent is already established in `product_brief.md` and inherited silently. A few interactions carry specific intent worth landing here.

## Return-greeting

When the user opens the aviary tab — fresh navigation, return from another tab, return after long absence — one bird notices them within the first second or two. Not all the birds, one bird. The greeting takes a different form depending on bird boldness, current mood, and absence length: it might be a glance up from preening, a quiet two-note call, a head-tilt and a step toward the front perch, a longer call followed by another bird's response. The greeting is procedurally varied so it is never identical twice.

The return-greeting is the anchor moment of the session, and it earns the prose space it gets. It varies by absence length so that a quick return after stepping away for coffee feels different from a return after two days — short absences produce a glance; longer absences produce something closer to a re-orientation, a bird coming closer or calling longer. It varies by bird boldness so the same bird greets the user the same way across visits while different birds greet differently from each other; the bolder bird greets first, the warier bird greets later or not at all on a given day. It varies procedurally inside those rules so the user is never being shown a canned cue.

This is the surface where "notice, never announce" pays off most visibly, and it is the easiest one for a planner to compress into a generic "play arrival animation" — which would be the wrong product. The reason this gets multi-paragraph treatment in this file is that the implementation matters: the absence-length signal has to be wired through, the bird-selection (which bird greets first today?) has to honor boldness and mood, and the procedural variation has to be real variation rather than three pre-recorded variants in rotation. If any one of those falls back to a canned cue, the user feels it on the first session and the rest of the product reads as theater.

When multiple birds would greet, they stagger by a randomized small offset rather than firing in unison. A simultaneous chorus on cue would announce the user's arrival to the aviary, which is exactly the wrong affective register; staggering makes the arrival feel like the aviary noticing, one bird at a time.

## No "Welcome back!" toast or banner

There is no textual welcome on return. No toast in the corner. No banner across the top. No greeting modal. No "great to see you" sentence anywhere on the surface.

The bird greeting is the entire welcome. Adding a textual welcome would announce the user's arrival at exactly the moment the bird is meant to notice it, which is the single most damaging violation of "notice, never announce" the product can suffer. This is the application of the principle that is most likely to be quietly added by a well-meaning contributor — "just a small toast saying hi" — and the temptation has to be foreclosed by being explicit here. A toast in this position would re-frame the session: instead of "the bird noticed me," it becomes "the system told me I was back, and here are the birds." Different product, in one move.

The rule extends to every variant of the same idea: no "you've been gone X days" surface anywhere, no calendar-of-visits decoration, no friendly text inviting the user back into the aviary. The aviary is the welcome.

## Listen-in

The user can focus a single bird — by clicking, tapping, or keyboard-focusing it — to bring its call up in the audio mix while the others quiet to ambient. The interaction is called listen-in.

Listen-in is the user's way of paying attention to one bird specifically. The mix change is gradual on engage and on disengage — a slow rise in the focused bird's mix level, a slow drop in the others. The interaction must feel like listening, not like switching channels: a hard cut would convert the aviary into a UI of soloable tracks, which is a different kind of audio surface from the one we're building.

Other birds drop in mix but never go silent. Silencing them entirely would teach the user that the aviary is a set of things to switch between rather than a place where multiple things are happening at once. The listen-in mix is a re-balance, not a mute.

Listen-in disengages when the user clicks the focused bird again, focuses a different bird, clicks empty space in the aviary, or moves keyboard focus away. The mix returns to ambient on disengage with the same slow ramp.

## Offer

The user can offer the aviary something — a seed, a song fragment from a small library, a still pool of water. Offers are reachable from a small affordance in the top bar (see `aviary_layout.md`); they are not reached by clicking on a bird directly.

Each offer produces a different reaction depending on the receiving bird's mood and curiosity. A curious, content bird approaches a seed; a wary bird waits and eventually comes near; a drowsy bird may not approach at all. The song-fragment offer is a melodic motif played softly in the aviary; the bird's response — joining in, going quiet, calling against — is shaped by its vocal frequency and current mood. The still-pool offer drops a soft reflective surface into the front of the scene; some birds drink, some bathe, some watch.

Offers have a per-bird cooldown of a few minutes. The cooldown is functional, not punitive — without it, curiosity-trait drift would saturate within a single session and the engine would collapse. With it, an offer reads as a gesture rather than a button-mash, which is the affective register the rest of the product is in.

## Settle

Settle is a user-initiated soft session-end gesture. The user triggers it from the top bar; the lighting shifts to evening over a slow few seconds, calls quiet, and the aviary acknowledges the goodbye. The aviary is now in a "settled" state, and remains there until the user closes the tab or actively re-engages.

Settle is opt-in. Closing the tab without settling is also fine — the engine treats the end of presence the same way regardless of whether the user clicked settle first. The settle gesture is offered for users who want the affordance, not required for users who don't, because making it required would convert a charming small ritual into a chore and would penalize users who close the tab for ordinary reasons (the meeting starting, the laptop battery dying, the phone going into a pocket). A required goodbye would also break the "presence is real interaction" model — presence ends when presence ends, with or without ceremony.

The settle gesture has a small undo affordance: any click anywhere in the aviary within five seconds of triggering settle reverses the lighting shift and returns the aviary to its normal state. This is a small mercy for accidental clicks; it is not a feature in itself.

## Field notebook

The field notebook is the system's running observation log of the aviary. It is auto-generated, read-only, and lives behind a notebook icon in the top bar.

Notebook entries are written in naturalist field-notebook prose, lowercase, present-tense, specific to the moment. An entry might read:

> tuesday — pip greeted before wren today, first time this week.

> wren is fluffed against the cool air, watching the back perch. low calls only.

> a long stretch of quiet this morning. pip preened for several minutes without looking up.

Entries are not generic event logs. They are not "a bird greeted you" or "Pip's vocal frequency changed by 0.03." They are written to feel like an observer's notes from a real morning of watching real birds. The notebook is the surface where the product's voice is most concentrated and most visible, and a stock event-log treatment would break the spell across the entire product. A user who opens the notebook and finds "session started at 7:43" instead of "wren is fluffed against the cool air" has been told, in one entry, that the rest of the product's voice is performance.

Entries are rare. Roughly one entry every few days for a regularly-visited aviary, more often when something noteworthy happens, not on every session. The notebook is not a feed; entries are observations, and an observation per session would dilute the entries that matter into noise. We will tune the entry-generation logic to preserve sparsity even for very active users.

The notebook is read-only. The user cannot edit, delete, or annotate entries. The notebook is an observer's record, not a journal — making it editable would invite the user to curate, which is a different product entirely. The user can scroll back through entries indefinitely; old entries do not get archived or hidden.

## Presence accounting

Presence is the dominant input to slow personality drift, and the implementation has to be precise. Per the definition in `concepts.md`, a presence-event is recorded only when the document's `visibilityState` is `visible` AND the document has window focus AND a pointermove or keypress has occurred in the last few minutes. All three conditions, simultaneously.

The reason this has to be exact is that the drift function uses presence-time as a primary input, and any laxer definition silently corrupts drift across the entire user base. If the engine counted "tab open" as presence, a user who left their laptop with the aviary in a background window for two days would generate as much drift as a user who actively watched for two hours. Birds across thousands of accounts would drift faster than the design calibration intends; "feels alive over weeks" would become "your birds change visibly between sessions." The failure would be silent — no test would catch it — and would only show up as users reporting that their birds felt different than they expected.

The conjunction of three independently-checkable signals is what makes presence honest. Visibility alone catches "tab is in front" but misses windows that are in front but not focused. Focus alone catches "user is looking at this window" but misses minimized windows. Pointer-or-key activity alone catches "user is at their computer" but misses the case where the laptop is open with the user away. All three together approximate "user is sitting and watching" closely enough that the drift signal carries the right meaning.

The activity window for the pointer-or-key check is a few minutes; we will calibrate exact value during build, leaning toward the longer side because watching birds without moving is the actual product. Presence is not lost the moment the user stops moving the mouse; it's lost when the user has stopped showing any sign of being there for a while.

Settling and tab-close are both terminal: both end the presence window. Closing the tab without settling is not penalized at the engine level. There is no "you didn't settle" recovery surface and no notification.

When the tab is in the background or hidden, the client stops rendering — there is nothing for the user to see and rendering wastes battery — but the simulation continues server-side at the slow tick cadence. The aviary the user comes back to is the aviary that has been running, not the aviary as it was when they left.

## No streak counter

There is no streak counter. There is no "days visited." There is no calendar of green dots. There is no "you've been here every day this week" surface. There is no widget anywhere in the product that surfaces visit-frequency to the user.

This is a deliberate refusal of the most reachable engagement feature in this product's adjacent design space. Surfacing visit-frequency would teach the user that presence is for the counter, not the birds — a tiny rotation of the user's intention from "I want to see my birds" to "I want to maintain my streak." The rotation is small and looks harmless from the outside, but its cumulative effect on the relationship the product is shaping is total: the user is no longer noticing the aviary, they are managing a number, and the rest of the product becomes furniture for the number. This is the biggest temptation a contributor will have to add a "harmless" engagement feature, and it must be explicitly rejected so it doesn't leak in. There is no version of a streak counter that doesn't break the product. The rule is absolute.

The same rule rules out adjacent disguises: a quiet calendar in the settings, an exportable visit log, a notebook entry that reads "you visited every day this week." The notebook can write that Pip greeted first today; it cannot write that the user has been here every day this week. The line is between observations of the aviary and observations of the user's behavior; the product makes the first kind, never the second.
