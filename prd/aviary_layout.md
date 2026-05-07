# Aviary layout

This file specifies the visual scene — what the aviary looks like, how it's laid out, and the rules that keep it feeling like a place rather than an app surface.

## Single horizontal scene

The aviary is one horizontal scene that fits on one screen, at any reasonable browser viewport. There is no panning, no scrolling, no zooming. The user takes the aviary in at a glance. The depth lives in the birds, not in the geography.

This is a conscious limitation. A larger explorable scene would shift the user's attention from the birds to the space — looking around instead of looking at — which would make the per-bird relationship the product is selling thinner. One screen, no panning, is the right size for a small social system the user is meant to know intimately.

## Three perch zones

The scene has three perch zones — front, middle, and back — that shape proximity to the viewer. A bird on the front perch reads as close, attentive, available; a bird on the back perch reads as distant, more ambient. Birds choose their perch based on mood and personality; a wary bird sits further back, a bold bird comes forward.

The user does not arrange the birds. There is no drag-to-place affordance, no "send Pip to the front" command, no perch-selection panel. Perch position is a *signal* the user reads, not a layout the user controls. The birds choosing where to sit is part of how the user reads them; converting it into a user-controlled placement would erase the signal.

## Day/night cycle

The aviary follows the user's local time. Morning in the user's timezone is morning in the aviary; their evening is the aviary's evening. The cycle runs continuously: sunrise warms the palette gradually over the early hours, midday is the brightest, evening shifts toward warmer hues with quieter calls, night dims most of the aviary.

A fixed-server-time cycle would feel arbitrary — the user's morning shouldn't be the aviary's afternoon. Local-time anchoring makes the aviary feel like a place that shares the user's day, which is a small but durable part of the felt-aliveness.

The evening palette warms the scene, calls quiet generally, and most birds drift toward drowsy or settled. At full night, most birds are settled (eyes closed, low on the perch); one species in the pool — a nightjar-like call signature — remains active and may call into the late hours. Night is not a dead state.

## Ambient weather

Rare ambient weather passes through the aviary: a few times a week, a short rain; occasional soft wind that ripples leaves through the scene. Weather is never assertive — there is no thunderstorm, no snow, no weather event the user has to notice. A passing rain is enough to make the aviary feel like it has its own moments without becoming a weather feature, and that's the calibration.

Weather affects mood in small ways. Rain dampens vocal frequency briefly across the aviary; wind makes some birds more alert and others more wary. The effects are short-lived and are mostly visible as small mood shifts during and just after the weather event.

## Ambient micro-motion

Small ambient motion runs continuously across the scene: leaves drift through frame occasionally, a feather catches the light and falls; the foreground and background have subtle parallax (gentle, not parallax-heavy — the scene is not a layered illustration trying to show off). The ambient motion is the visual cue that the aviary is continuing without the viewer.

## No UI chrome inside the aviary

The aviary scene itself carries no buttons, no badges, no hover-tooltips, no overlay icons, no inline labels. The visual surface is birds and place. UI chrome lives in a thin top bar that sits above the aviary scene proper.

The top bar contains a small set of icons: account/settings, accessibility settings, the field notebook, and the offer affordance. Nothing else. The top bar is intentionally sparse.

## Top bar fade

After a few seconds of cursor stillness, the top bar fades nearly to transparent. It returns to full opacity on cursor movement or keyboard activity. The fade is the small cost of putting controls anywhere on the surface, and it pays off in the felt-stillness of the scene — a top bar that's always at full opacity reads as an app frame; a top bar that fades reads as a thin layer the user can ignore unless they need it.

## Aviary scene loads with motion already in progress

The first frame the user sees has birds mid-action. A bird is mid-preen, another is calling softly from the high perch, a leaf drifts through frame. There is no "wake up" animation, no fade-from-static, no entry sequence, no spinner-resolves-into-aviary transition. The aviary appears with its motion already in progress, and that motion continues without interruption.

This is the central conceit of the product made concrete in the loading sequence. The aviary is meant to feel like it has been continuing without the viewer; an entry animation would tell the user explicitly that it just started, which is the wrong story. The implementation falls out of the server-side simulation tick — the client pulls a state snapshot, places birds at their current positions in their current motions, and starts rendering as if it's been rendering all along.

A planner who falls back on a spinner-then-fade-in transition because it's the safe pattern has compromised the central conceit of the product, and the rest of the affective design starts to drag against this single failure. The first frame is not a load state; it is the aviary, which has been there.

When the state snapshot takes a beat to load (slow connection, cold cache), the loading state is a quiet field — soft sky color, perhaps one or two faint motion cues — not a spinner. A spinner says "machine," and we are not selling a machine. The quiet field reads as the aviary catching up rather than the product loading.

## Empty-aviary state

Briefly, between adoption flow and the first bird appearing, the aviary is empty. This is a real state — the user has just signed up, named two starter birds, and the visual scene needs to handle the moment before the birds appear in it. The empty-aviary state is the same quiet field as the loading state; the first bird then enters with a soft fly-in to its starting perch. From that point forward, the user never sees an empty aviary again.

## Color palette

The aviary palette is calm and naturalist — soft blues, greens, warm browns, muted ochres. Saturated UI accent colors — bright reds, electric blues — are deliberately absent. The palette is meant to feel like a window on a real morning, not a designed UI surface.

The palette specifications, including specific colors and contrast ratios for any text overlaid on the aviary, are in the design system spec (separate document; lives with the visual designer). Contrast on user-copy text passes WCAG AA at minimum; see `accessibility_perf.md`.

## Responsive scene

The aviary is responsive to viewport size. On a narrow phone viewport, the scene compresses horizontally without cropping any bird out of frame; on a wide desktop viewport, the scene widens with more space between perches. Aspect ratio is preserved in a way that keeps all birds visible at all times — never crop a bird out, never let one drift offscreen. The maximum and minimum viewport handling are implementation details that live in the rendering spec.

## Foreground and background

The scene has a quiet foreground/background separation — birds and perches sit on a middle plane; soft background foliage and sky sit behind; an occasional foreground branch or leaf passes through. The parallax is subtle. The product is not parallax-heavy.

## Ambient leaf and feather drift

Ambient leaves and feathers drift through the scene at slow random intervals. These are not driven by the simulation tick (no per-leaf state); they are pure rendering ornaments, generated client-side at idle cadence to keep the scene feeling like it has its own ambient motion. They are part of how the aviary continues looking alive even between bird actions.
