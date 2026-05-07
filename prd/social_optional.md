# Social — optional and quiet

The social feature in Pocket Aviary is one feature: a host can invite a friend to visit their aviary, read-only. That's it. This file specifies what visits are, what they deliberately are not, and the privacy controls around them.

The reason this file is short is that the social surface is small on purpose. Most product teams in this space build a social network and then defend the parts; we are building a single quiet affordance and refusing the rest, and the refusals are the substance of the file.

## Visits

A user (the host) can invite a friend (the visitor) to visit their aviary. The host enters the visitor's email; the system emails the visitor a one-time link. The visitor follows the link and sees the host's aviary in their browser as a read-only ambient view.

Invites are per-invite opt-in. There is no global "discoverable" flag, no friend-of-friend chain, no implicit sharing model. Each visitor is specifically named (by email) by the host, and the host issues each invitation deliberately. There is no automatic re-invitation, no "frequent visitor" status, no permanent visitor list that updates itself.

Invites are revocable. The host can revoke any outstanding or active invite from account settings. Revocation takes effect immediately — the next state-snapshot pull on the visitor's client returns a "visit no longer available" surface in the matter-of-fact voice.

Visits default OFF for new accounts. A user has to explicitly send an invite to use the social feature. There is no default visitor list, no "share with friends" prompt during onboarding, no public surface a new account is exposed to without choosing.

## Visit is read-only ambient

A visitor sees the host's aviary as it is. They can hear the calls, watch the birds, see the day/night state, see the current weather — exactly what the host would see at this moment, with no special rendering. The visitor cannot trigger anything: no greetings, no listen-in, no offers, no settle gesture, no notebook scrolling that affects the host. Their session is render-only.

The visit is observation, not co-presence. The visitor and the host are not in the aviary together. There is no shared cursor, no shared pointer, no live "your friend is here too" overlay. The visitor sees the host's aviary; the host doesn't see anything new.

The reason for not building co-presence is structural: co-presence would require designing a multi-user simulation where presence-time, interactions, and drift inputs come from multiple users at once, which is a different and much larger product than the one we're building. A read-only ambient view is the maximum social affordance the bird-engine model can support without rebuilding it. We chose to land the social feature inside that constraint rather than expand the engine to accommodate a different feature.

The simulation does not record presence-time or interaction events from a visitor. A visitor sitting and watching for an hour does not drift the host's birds; the host's drift comes from the host's presence only. This protects the host's relationship with their birds from being reshaped by visitor attention they didn't sign up for.

## What visits are not

A list of negative space, because the absences here are deliberate.

- **No chat.** No text overlay during a visit, no comment box, no message-the-host. If the visitor wants to talk to the host, they have other tools.
- **No avatars.** The visitor's identity is not represented in the aviary. There is no marker, no cursor highlight, no bird-shaped placeholder for the visitor.
- **No comments.** Visitors cannot leave notes on the host's aviary or on individual birds. The notebook is the host's; it is not annotatable by anyone.
- **No public discovery.** There is no directory of public aviaries, no "explore other aviaries" surface, no rating or featuring of aviaries. Aviaries are private to the host and their named invitees.
- **No leaderboards.** No ranking of any kind. No "most-visited aviaries," no "longest-running aviary," no "most birds." We don't even compute the underlying stats for this purpose.
- **No "show-off" mode.** Visitors see the host's aviary exactly as it is — same birds in the same moods at the same drift levels. There is no special rendering that prettifies the aviary for visitors. Faking the aviary for visitors would betray the relationship the host has with their birds; the entire point of letting a friend in is for the friend to see the actual birds, not a marketing rendering of them.

The leaderboards-and-discovery-feed cluster is the most reachable extension of the social feature, and every existing pattern in this space includes one or both. We are explicitly refusing them. Public surfaces would shift the user's relationship from "their birds" to "their birds compared to other people's birds," which is a different product. The user we are building for does not need their birds to be ranked; they need their birds to be theirs. This refusal cascades: no leaderboards means no underlying metrics aggregated across accounts, which means no telemetry pipeline that could later "just be exposed," which means the architectural absence makes the feature's reappearance harder rather than easier.

## No friend-visited notification

By default, the host gets no push, no email, no in-product notification when a friend visits. The visit is logged silently in the host's visit log; the host can choose to look at the log if they want.

This is a deliberate refusal of the obvious "social engagement" loop. A "your friend visited!" notification would convert the social feature into an attention-driver — the host gets pinged, opens the app to see who came by, sees an empty visit-list because they were invited too, and the loop that produces "engagement" is the loop that the rest of the product is built to refuse. Quietly logging the visit and letting the host find it on their own preserves the social feature as an affordance that the user reaches for, not a notification surface that reaches for the user.

The host can opt into visit notifications in settings if they want them — a small per-account toggle, off by default. The setting exists so that a host who specifically wants to know when a particular friend stops by can turn it on; it is not surfaced during onboarding and it is not enabled out of the box.

## Visit log

The host can see a visit log in account settings: a list of who visited and when, ordered by most recent. The log shows the visitor's email, the date and approximate duration of each visit, and any current outstanding invitations.

The visit log is reachable on demand, not pushed at the host. There is no badge on the settings icon when a new visit happens. The log is just there if the host wants to look at it. The reason we have it at all is transparency about what's been shared: the host should be able to see what's been visible to whom.

## Visit revocation surface

When the host revokes an invitation that is currently in use, the visitor's session is terminated at the next state-snapshot pull. The visitor sees a matter-of-fact surface explaining the visit is no longer available. There is no notification to the host that revocation succeeded; the absence of the visitor in the visit log is its own confirmation.

If the host revokes an invitation that hasn't been used yet, the link silently stops working. The visitor following an expired or revoked link sees the same matter-of-fact surface.

## Invite expiration

Outstanding invitations expire after 30 days if unused. An expired invitation cannot be revived; the host can issue a new one. The expiration window is long enough to absorb friends who don't get to it right away and short enough that abandoned invitations don't accumulate indefinitely.
