# Non-goals

The product is shaped as much by what it refuses as by what it includes. This file states the four explicit out-of-scope categories for v1, with the reasons. Each is a paragraph rather than a bullet because the reasoning is the substance — a contributor reading this file should leave knowing why each absence is deliberate, not just that it's absent.

## Out of scope: native mobile app

V1 is web-only. There is no iOS app, no Android app, no plan for either at v1. The web-only constraint is a deliberate scoping choice tied to what one team can build at quality, not a permanent prohibition. Browsers are good enough — modern WebAudio, the rendering pipeline we need, multi-device sync via standard auth — and the team's time spent on a second native client would come out of time we'd otherwise spend on the bird engine, which is the load-bearing surface. We may revisit native apps later; we are not planning for them in v1, and we are not designing the data model or the protocols with native-client constraints in mind.

## Out of scope: gamification

No achievements. No streaks. No levels. No scores. No badges. No "birds adopted: 2" counter. No green-dot calendar. No XP. No rank. No tier. No "you've been here every day this week" surface anywhere in the product. Not as a quiet setting toggle, not as an opt-in dashboard, not as a "harmless" celebration on a milestone. None of these exist in v1, and none will be added in any future version that still calls itself Pocket Aviary.

The temptation to add at least one is strong and predictable — every adjacent product in this space has one, the implementation is cheap, and the engagement metric it produces is easy to point to in a meeting. The reason we are refusing all of them is that each one teaches the user that presence is for the counter, not the birds. The rotation is small in any single instance — "I'm not visiting because of the streak, I just like the streak" — but its cumulative effect on the relationship the product is shaping is total. The user starts checking the streak before the birds. The next session, they feel the streak's pull before they feel the bird's notice. The session after that, they feel a small obligation when the streak is at risk, and an obligation is the wrong shape for a relationship with a virtual bird. Once the rotation has happened, every other decision in the product — the procedural calls, the field notebook prose, the slow drift, the noticing-not-announcing — is fighting against a counter the user is now optimizing for, and the rest of the product reads as decoration around the counter.

This is also the rule that compounds. "Just one streak counter" is the foothold that makes the next harmless engagement feature easier to argue for. Six months on, there's a notification, a public profile, a leaderboard, and the product is no longer Pocket Aviary; it's a different product with birds in it. The rule has to be loud here so that it survives every reasonable-looking pitch to relax it. The cumulative effect of "just one" is exactly the failure mode every gamified product slides into without noticing, and the only durable defense is to refuse the first one.

## Out of scope: Tamagotchi-style mechanics

Birds do not die. Birds do not get hungry. Birds do not show distress. Birds do not have a happiness meter that decays over time. The relationship is observational, not custodial.

The Tamagotchi model — neglect produces visible suffering, sustained attention produces visible flourishing — is the obvious mechanical engine to reach for in a product like this, and it is exactly the wrong engine. It punishes absence, which is the user behavior we are explicitly refusing to punish. It teaches the user that they owe the birds something, which converts the relationship into an obligation. It frames the user's role as caretaker, which is a different and more demanding role than observer. The entire drift mechanic — monotonic toward expressive, no negative drift on neglect — is built specifically to refuse this model, and surface-level Tamagotchi mechanics would contradict the engine they sit on.

A user who closes Pocket Aviary for two weeks and comes back should find birds that are quieter than they were, not birds that are sick or sad or angry at them. The shape of the welcome on return is "the birds are still here, just less expressive than they could be" — which gives the returning user something to ease back into rather than a guilt surface to apologize to.

## Out of scope: social network surfaces

No profiles. No follows. No public feed. No shared discovery. No "explore other aviaries." No friend-of-friend chains. No mutual visits. No comments on visits. The visit feature exists as a quiet opt-in (see `social_optional.md`); social-network surfaces beyond that single affordance are out of scope.

Public surfaces would shift the product's center of gravity from "the user's birds" to "the user's birds compared to other people's birds," which is a different product with a different relationship at its center. The user we are designing for does not need their birds to be ranked, featured, or visible to strangers; they need their birds to be theirs. A discovery feed of public aviaries would also create a social pressure to perform the aviary — to make it look good for visitors — which betrays the entire premise that what the visitor sees is what the host sees. Refusing the social-network surfaces is a refusal to take on the failure modes those surfaces always carry: comparison, performance, the slow conversion of a private surface into a social one.
