# Concepts

The other files in this PRD lean on a small shared vocabulary. This file defines the terms once. If a term appears in any other file with a meaning that doesn't match what's here, this file wins.

## Glossary

- **Bird** — one of the animated creatures in the aviary. Always "bird," never "creature," "pet," "animal," or "character" in spec text. The user adopts birds; they don't acquire them.
- **Aviary** — the scene as a whole, and by extension the system as a whole for a given user. One aviary per account at v1.
- **Call** — a bird's vocalization. Always "call," never "song," "noise," or "chirp." Calls are procedural (see `bird_engine.md`); they are not audio loops.
- **Mood** — a fast-timescale emotional state for a bird. Mood resets on a daily-ish cadence, modulated by recent interactions, time of day, and ambient events. Wary, content, curious, drowsy are example mood states; the full set lives in `bird_engine.md`.
- **Personality vector** — the slow-timescale set of numerical traits per bird (boldness, social warmth, vocal frequency, plumage saturation, curiosity). The personality vector is the substrate of long-term character; mood is its short-term expression. The user never sees these numbers; this is a hard rule, not a default.
- **Drift** — the slow cumulative change in personality vector values over weeks of use. Drift is shaped by user behavior — primarily by presence (see below) and secondarily by interactions. Drift is monotonic toward expressive: traits move up on positive presence and never move down on neglect.
- **Presence** — the user's measured idle attention to the aviary. Presence is precisely defined: a presence-event is recorded when, simultaneously, (a) the document's `visibilityState` is `visible`, (b) the document has window focus, and (c) at least one pointermove or keypress event has occurred in the last few minutes (we'll calibrate the exact window during build). All three conditions must hold; any one alone is insufficient. Presence-events accumulate into presence-time, which is the dominant input to drift.

  This definition is sharper than it might seem necessary because the simulation engine uses presence-time as a primary drift input — and any laxer definition silently inflates the population's drift signal, making everyone's birds change faster than the design calibration intends. A common shortcut, "tab is open," would count a user who left their laptop open all night the same as a user who watched for an hour, which would corrupt the drift function across every account. Pinning presence to the conjunction of three independently-checkable signals is the load-bearing precision that makes "feels alive over weeks" actually true.

  Presence is the cleanest way the product expresses its central idea: idle attention is real interaction. A user who sits and watches without clicking is not doing nothing; the system sees them sitting, the birds drift toward expressive in response, and the next session reflects that attention. The flip side is also true and just as important — a user with the tab in a background window for two days has not interacted, and the engine must not record otherwise. The whole shape of the relationship the product is asking the user to form depends on this signal being honest.

- **Listen-in** — the user's interaction of focusing a single bird so its call rises in the audio mix while the others quiet to ambient. Verb: "to listen in on Pip." Noun: "the listen-in mix." Never "solo," "select," "highlight," or "pin."
- **Offer** — the user's interaction of giving the aviary a small gift: a seed, a song fragment from a small library, or a still pool of water. Verb: "to offer a seed." The bird's reaction is shaped by mood and personality.
- **Settle** — a user-initiated soft session-end gesture. The lighting shifts to evening, calls quiet, the aviary acknowledges the goodbye. "Settle" is also the adjective for the resulting evening lighting state (the aviary is "settled"). Settling is a deliberate, named gesture distinct from closing the tab; both are valid ways to leave. Defining it explicitly preserves the affordance for users who want to say goodbye without making tab-close feel like a failure or a missed step.
- **Field notebook** — the auto-generated log of naturalist observations the system writes about the aviary. Lowercase "notebook" when referring to the file or feature; capitalized "Field Notebook" only when it is a proper UI label.
- **Visit** — the social opt-in feature: a host invites a friend by email to view the aviary read-only. Verb: "to visit your friend's aviary." The visitor cannot interact and is not co-present with the host.
- **Tick** — the server-side simulation step. The aviary's canonical state advances on a tick at a slow cadence (~once per minute). Clients never tick; they pull snapshots and interpolate.

## Slow timescale vs. fast timescale

The product runs on two clocks at once and a reader who confuses them will get the engine wrong.

- **Slow timescale** — personality vector values drift across days and weeks. A typical bird should show measurable drift in instruments after about a week of regular visits and visible drift to the user after about three weeks. Drift never resets, and a single session never moves a personality value visibly.
- **Fast timescale** — mood resets on a daily-ish cadence and is modulated by the last few interactions, the time of day, and ambient events (a passing rain, the early-morning hush). Mood is what the user actually sees moment-to-moment; personality is what they feel after weeks.

Mood is to personality as weather is to climate. The clean way to think about it: mood is the visible motion, personality is the slow current underneath, and the user notices the current only by looking back at where the bird used to perch.

## Presence as the central idea

If a reader takes one thing from this concepts file, it should be that presence is real interaction. The product is not measuring engagement; it is measuring attention. The simulation does not care how many times the user clicks; it cares whether the user was there.

This is also why settle and tab-close are equivalent at the engine level — both end presence; neither punishes the user. The settle gesture is offered for users who want the affordance, not required for users who don't. Forcing a settle would convert a quiet leaving into a chore, which is the wrong tone for a product whose core claim is that absence is fine and presence is enough.
