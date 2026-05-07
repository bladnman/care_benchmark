# Product brief — Pocket Aviary

## What this is

Pocket Aviary is a tiny, browser-based virtual aviary. You adopt a small handful of animated birds — two to start, room for up to seven — that live in a single horizontal scene rendered in your browser tab. The birds notice you. Over days and weeks their personalities drift in response to the small ways you show up: how long you sit and watch, whether you offer a seed or a still pool of water, whether you mute the calls or let them play.

There is no game state in the conventional sense. There are no quests, no scores, no streaks. The product is closer to a low-key relationship with a window than to a Tamagotchi. The user opens the tab, the aviary is already in motion, a bird notices them, and they sit with it for a few minutes. That's the surface. The depth is in what those small interactions accumulate into.

The product runs only in modern web browsers. Single-user accounts, signed in via email magic link, syncing across devices so the laptop in the morning and the phone at night show the same aviary in the same mood. There is no native app and no plan to ship one.

## A session, sketched

The user opens the tab. Within the first second or two, one bird notices them — perhaps Pip cocks her head, perhaps Wren calls once and looks. From there the user can:

- Sit and watch (idle attention is itself an interaction; the birds register it)
- **Listen in** on a bird — focus it, and its call rises in the audio mix while the others quiet to ambient
- **Offer** something — a seed, a song fragment from a small library, a still pool — and watch the bird's reaction shaped by its current mood and personality
- **Settle** the aviary — a soft lighting shift to evening, the user's polite way of saying "I'm leaving, take care"
- Browse the **field notebook** — naturalist observations the system has written about the aviary's small moments

That's the surface. A typical session is short and uneventful by design.

---

## Design philosophy

Five principles shape the product. They are not slogans; they are constraints with teeth, and most of the rest of this PRD inherits from them silently.

### Feels alive, not robotic

The headline. The aviary should feel like a place that has been continuing without the viewer, not an app that woke up when the tab opened. Animations breathe. Calls vary every time. Greetings are never identical twice. Idle motion is slow and personality-keyed. There is no entry animation, no "ready" pop, no fade-from-static; the first frame the user sees has birds mid-action, ambient drift already in motion, calls already audible.

The reason this is the headline and not the third bullet is that aliveness is the whole product. If the birds feel canned, every other feature collapses — the relationship the product is asking the user to form is with something that responds, not something that performs. Procedural calls instead of looped audio, mood-shaped idle motion instead of cycle animations, server-side simulation that advances on a slow tick whether or not anyone is watching: these are all expressions of the same principle, and getting them right is what makes a user feel accompanied rather than entertained. Failing this principle in even one place — a stock loading spinner, a canned greeting, a strobing micro-animation — leaks into the rest of the product as a kind of staleness the user will not name but will feel, and they will leave.

### Notice, never announce

The system notices the user but never announces. There are no "Welcome back!" toasts on return. There are no level-up confetti or badge popups. There is no "your friend visited!" notification. The bird greeting is the entire welcome surface — that's the whole point of having birds. A textual welcome would announce arrival at exactly the moment the bird is meant to notice it, which is the wrong product in one move.

This is the principle the developer instinct will most reliably violate, because announcement-style UI is what most products are built out of. The discipline here is not minimalism for its own sake; it's the recognition that announcing a thing is a different affective register from being noticed for it. A user who is announced at feels processed. A user who is noticed feels seen. The product earns attention by being noticed, not by demanding it, and the cumulative effect of refusing every "harmless" announcement surface is what makes the product feel different from the things it's adjacent to. If even one toast slips in, the rest of the surface starts to read like the product is trying — which it isn't, and shouldn't.

### Charm comes from specificity

The field notebook says "Pip greeted before Wren today, first time this week." Not "Achievement unlocked: First Greeter." The narration says "a warbler perches on the high branch, calling softly." Not "warbler perched at high branch." The product's voice is naturalist, specific to the bird and the moment, and it never reaches for gamification language even when the gamification language would be quicker.

Specificity is the product's only real charm engine. Generic phrasing — "your bird is happier!" — flattens every moment into the same moment and teaches the user that the product is a system showing them states. Specific phrasing — "Pip greeted before Wren today" — preserves the small particular and trusts the user to feel the difference.

### Restraint over richness

Two birds at start. Maximum seven. One screen. Calm color palette. No UI chrome inside the aviary view. The aviary fits at a glance.

This is a constraint, not a phase. Adding more birds would dilute per-bird recognizability — the audio system can support roughly seven distinct call signatures before the chorus blurs into ambient. Adding panning or scrolling would shift attention from the birds to the geography. Adding a richer color palette would make the screen feel like an app instead of a place. The depth lives in the birds, not in the variety.

### Naturalist voice for the product, matter-of-fact for the system

The product surface — the aviary, the notebook, the narration, the offer prompts — uses a naturalist field-notebook voice. Lowercase by default. Present-tense. Specific. Bird-related verbs (notice, perch, settle, listen in, offer) preferred where possible.

System surfaces — sign-in, sync conflicts, account settings, accessibility settings — drop into a quietly matter-of-fact register. Naturalist phrasing in an error context reads as evasive; the user needs system-clarity there, not charm. This is a deliberate, named exception, and it should be obvious to a careful reader of any PRD where the line falls.

---

## Voice and tone — style samples

The split between voices is load-bearing. Two short style samples, so the difference is visible and not just stipulated.

**Naturalist (product surface).**

> wren is on the low perch this morning, fluffed against the cool air. pip greeted first today — only by a beat, but first. a leaf drifted down past the back perch and neither bird looked up.

Note: lowercase, present-tense, bird-named, specific to the moment, no exclamation, no "you," no announcement framing.

**Matter-of-fact (system surface).**

> We couldn't sign you in. The link may have expired. Try requesting a new link.

Note: capitalized as normal English, direct, no naturalist phrasing, no warmth pretending to be useful. The user trying to sign in needs to know what happened and what to do, and they need it without the product affecting a tone that doesn't fit.

The exception is named so that nobody, anywhere, in any future feature, has to argue about it again. Account, error, sync, and accessibility-settings surfaces use matter-of-fact voice. Everything else uses naturalist voice. If a new surface lands in between (say, a billing flow that doesn't exist yet), the rule is that any surface where the user is explicitly engaging with the system as a system — money, identity, errors, settings — drops out of the naturalist register.

---

## What this is not

A short list, because each item rules out a whole class of features that an enthusiastic team would otherwise reach for.

- **Not a game.** No win condition. No score. No quests. No streaks. No gamification of any flavor.
- **Not a Tamagotchi.** Birds do not die. Neglect produces ambient quietness, not visible distress. The relationship is observational, not custodial.
- **Not a social network.** Visits are quiet, opt-in, and revocable. There are no profiles, no follows, no comments, no public discovery.
- **Not a pet simulator.** No hunger meter. No feeding schedule. The seeds in the offer interaction are gestures, not sustenance.
- **Not gamified.** No achievements, no badges, no levels, no "birds adopted: 2," no calendar of green dots. None of these. The temptation is real and the rule is absolute.
- **Not a notification surface.** Pocket Aviary does not push, ping, or email the user about the aviary. The aviary lives where the user visits it.
- **Not a native app.** Web-only at v1.

The product is what's left after these subtractions. That's the shape.

---

## Scope statement

V1 starts with two birds per aviary and caps at seven. The cap is not arbitrary: per-bird call signatures must remain individually recognizable to the listener, which is the load-bearing affordance of the audio system, and somewhere around seven the chorus blurs into ambient and the per-bird relationship collapses. Two at the start is the inverse rule — one bird is too lonely to model a small social system, and three or more at the gate overwhelms the first encounter. Two is companionship without overload.

V1 ships single-user accounts, magic-link sign-in, single canonical aviary per account, multi-device sync, the field notebook, presence accounting, the visit-invitation feature (off by default, opt-in per invite), screen-reader narration, reduced-motion mode, and call captioning. It does not ship: native apps, payments, shared aviaries, customizable scenes, multi-aviary accounts, public discovery, leaderboards, achievements, push notifications. See `non_goals.md` for the explicit list.

---

## Who this brief is for

This file is the spine. The concepts, mechanics, interactions, layout, accounts, social, accessibility, and non-goals files all hang off it. A reader who has internalized this brief should be able to predict, with reasonable accuracy, the rough shape of the next file before opening it.
