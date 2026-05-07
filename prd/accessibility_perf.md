# Accessibility and performance

This file specifies two things that the rest of the product depends on quietly: how the aviary works for users who experience it differently from the default visual-and-audio path, and how fast the product loads and renders. These are usually two separate documents, but for Pocket Aviary they share a bias: in both cases, the rule is that the user gets the actual product, not a stripped-down fallback.

## Accessibility — first-class, not a checklist

Accessibility surfaces in Pocket Aviary are designed for charm, not parity-by-checklist. A screen-reader user, a reduced-motion user, a user with audio off — all of them should experience an aviary that *feels alive*. They should not get a degraded variant of the product where the affective core has been stripped out and replaced with semantic markup that announces the same states as the visual.

This is a deliberate design stance and it costs more than the alternative. The cheap way to do accessibility on a product like this is to label every visual state, expose the personality vector to ARIA attributes, and ship a reduced-motion fallback that turns off the animations and shows a static scene. We are not doing the cheap version. The narration is naturalist prose. The reduced-motion mode is its own designed surface, not a fallback. Captions for procedural calls are written in the same voice as the field notebook. The reason this matters more than usual is that the product's value is largely affective — if the accessible surface flattens that, we have built a product where access to its actual quality is rationed by sensory ability, which is a worse failure than missing a contrast threshold. The work to do this right is real and is named here because it has to be planned in from day one rather than retrofitted.

The downstream consequence of treating accessibility as a designed surface is that the accessibility work has to ship with the rest of the product, not after. A reduced-motion mode that lands two months after launch as a "v1.1 fix" is a v1 launch that quietly told reduced-motion users that the product wasn't for them.

## Screen-reader narration

The screen reader hears a running narration of the aviary state — naturalist prose updated on a slow cadence. Not a list of state changes, not "Pip is at perch 2," not "Wren mood: content." Running prose:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

The narration is generated server-side or client-side from the same state the visual surface reads from, expressed in the same voice as the field notebook. A screen-reader user has the same right to *feel* the aviary as a sighted user, and a state-list narration produces a fundamentally different — worse — product for them. The implementation has to honor this; a planner who treats the narration as ARIA-label automation has built the wrong feature.

Narration cadence is slow: roughly one prose update per 30–60 seconds at idle, faster only on user-initiated events (a successful offer, a settle gesture, a return-greeting). High-frequency narration would overwhelm the screen reader's queue and force the user to silence it, which would be the system pushing the accessibility surface aside. The pacing matches the slow rhythm of the visual aviary — birds aren't doing things every five seconds, and the narration shouldn't act like they are.

Narration prose uses the same naturalist field-notebook voice as the rest of the product. Lowercase, present-tense, specific:

> a warbler perches on the high branch, calling softly.

Not announcement style:

> warbler perched at high branch.

The voice continuity matters because a screen-reader user moving between the aviary surface and the notebook surface should hear the same product, not two products with different personalities glued together.

User-initiated events get a small priority bump in the queue: a return-greeting on session start gets narrated promptly, an offer reaction gets narrated as it happens. But even these prioritized events are written as observations, not as state transitions.

## Reduced-motion mode

Users with `prefers-reduced-motion` set, or who opt in via accessibility settings, get a reduced-motion mode. This mode is not "animations off." It is a different rendering of the same aviary.

In reduced-motion, micro-motion is replaced by slow cross-fades between still poses. A bird that would normally be preening is rendered in a sequence of preen-poses that cross-fade slowly, not animated frame-by-frame. Flight transitions become cross-fades between perches rather than animated paths. Ambient leaf drift is removed; ambient color shifts (day to evening) remain, slowed.

Calls still play at full quality (or caption, per the user's audio settings). Birds still drift. Mood still changes. The field notebook still notices things. The aviary is still the aviary; it just renders in a different visual register.

The reason reduced-motion is its own designed surface rather than a stripped fallback is that a stripped fallback would tell the user that their accessibility preference cost them the product. The cross-fade rendering is its own quiet aesthetic — it has its own charm, and a user who set `prefers-reduced-motion` for vestibular reasons gets a Pocket Aviary that is calmer and slower, not a Pocket Aviary that looks broken.

## Captioning for calls

Users can opt into call captions from accessibility settings. Captions are short prose descriptions of what each call sounds like in the bird's current mood:

> a soft three-note rise

> a low trill, paused, low trill again

> a single sharp call from the back perch

Captions appear as small text near the calling bird, fading in and out with the call. They use the same naturalist voice as the rest of the product. The captions are useful for users with audio off, hearing differences, noisy environments, or any situation where the audio isn't getting through.

The caption text is generated from the procedural call grammar at runtime, not stored as a fixed string per call. Each call's caption matches what was actually played.

## WCAG AA contrast

All user-copy text — top bar labels, settings, account surfaces, error surfaces, captions, narration when displayed visually — passes WCAG AA contrast. This is the floor, not the ceiling; the design system specifies actual ratios per surface. The aviary scene itself does not contain user copy except in the top bar, so the contrast constraint applies primarily to the chrome rather than to the scene.

## Keyboard navigation

All interactive surfaces are reachable by keyboard. Tab moves through the top bar items; entering the aviary scene with Tab focuses the first bird; arrow keys move focus between birds; Enter triggers listen-in on the focused bird; Escape exits listen-in. The offer affordance opens with a top-bar shortcut and is itself fully keyboard-navigable. The settle gesture is reachable from the top bar.

Focus indicators are visible against the aviary background — a soft, high-contrast outline that reads against both bright and dim aviary states. The visual designer specifies the exact treatment.

---

## Performance

Two budgets and one observability story.

### Initial JS bundle <2MB

The initial JS bundle, at first paint, is capped at 2MB (gzipped). Above this, the time-to-first-bird budget is unrecoverable on a typical mid-tier mobile device over 4G, and the central conceit of the product — the aviary appearing already in motion — falls back to a load state the user notices.

The 2MB cap drives several downstream choices. Procedural audio synthesis client-side is partly because we can't carry recorded audio at the variation we need within this budget. Bird visual assets are generated procedurally where possible and are otherwise small SVGs or compact bitmaps. Code-splitting is used aggressively for surfaces the user reaches less often (account settings, accessibility settings, the visit-invitation flow).

### Time to first bird visible <500ms

On a mid-tier mobile device over a 4G connection, the first bird is visible within 500ms of navigation. This is the threshold below which the aviary feels like it was already running and above which it feels like it's loading. The threshold is the affective-perf bridge — the point at which a performance metric becomes a felt-aliveness metric. Above 500ms, the user notices the load; below it, they don't.

Hitting this requires the bundle budget, fast initial state-snapshot delivery (a small payload delivered from a CDN edge with the HTML), and a render path that doesn't wait for non-critical assets before drawing the first bird. The full implementation lives in the rendering spec.

### 60fps idle motion on a 5-year-old laptop

Idle motion runs at 60fps on a five-year-old mid-range laptop. This is a runtime budget, not just a launch one — the constraint applies to a 30-minute session, not just the first minute.

### No memory growth over 30 minutes

A 30-minute session does not show memory growth in the client. Procedural audio buffers are reused; no per-call allocation that isn't freed. Notebook entries scrolled into view do not retain references after scroll-out. Worker threads and audio contexts are bounded. The "no memory growth" rule is a real test in CI, not a guideline.

### Procedural audio synthesized client-side

Calls are synthesized from the motif library client-side via WebAudio. Not downloaded as audio files. The bundle budget alone forces this — we cannot ship recorded audio at the variation the chorus mechanic requires within 2MB — but procedural synthesis is also what makes the chorus mechanic possible at all. Two recorded loops layered are not a chorus; two procedural calls mixed at runtime are.

### WebAudio fallback

If WebAudio is unavailable (older browser, audio context permission denied, hardware issue), the aviary plays in graceful silence with captions on by default. We do not ship a recorded-audio fallback path. The "no recorded audio" rule is unconditional — recorded audio at any quality below the procedural variation we need would feel canned, and at the quality we'd need to match, the bundle budget collapses. Silence with captions is a better fallback than canned audio.

### Performance observability

We instrument synthetic performance checks (a fleet of automated browsers running the aviary on a schedule from common geographies) and aggregate-only Real User Monitoring — page load timings, first-bird-render timings, render-frame timings, audio-context errors, simulation-tick latencies. None of this telemetry includes per-bird state or per-account interaction history; the privacy boundary in `accounts_sync.md` is honored at the metric definition.

Error budget: simulation-tick latency p99 alarms if it exceeds 5 seconds. The tick is supposed to take much less; an alarm at p99 5s catches degradation early, before users notice the aviary "running slow."

### Browser support

Last two major versions of Chrome, Safari, Firefox, and Edge. Older browsers receive a matter-of-fact unsupported-browser surface explaining what's needed. We deliberately do not maintain compatibility paths for very old browsers; the cost-benefit doesn't justify the bundle bloat.
