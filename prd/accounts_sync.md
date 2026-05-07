# Accounts, sync, and privacy

This file specifies the account model, the sync architecture, and the privacy commitments that shape how the system stores and uses interaction data. The shape of this file is "boring with teeth" — most decisions here look like ordinary infrastructure decisions, and a few of them are load-bearing for the rest of the product working at all.

## Accounts

V1 accounts are single-user. One account, one aviary, one user. There is no shared aviary, no team aviary, no household-with-multiple-profiles model.

Sign-in is by email plus magic link. No passwords. The user enters their email; the system emails them a magic link; clicking the link signs them in on that browser. SSO and password-based login are deferred — magic-link is the v1 auth model because it's low-friction and matches the calm tone of the rest of the product, and because there's nothing in the product worth attacking by stuffing credentials.

Magic links expire after 15 minutes. A used link is invalidated immediately on consumption. A user can request a new link as often as they like; rate-limiting is per-email at a reasonable threshold (implementation detail).

Per-device session tokens are issued on successful sign-in and are revocable from account settings. A user who notices an unfamiliar device in the session list can revoke that session.

Email change requires verification of the new address before the switch is committed. The old email continues to work until the new address verifies.

## Synthetic account ID

Internal references to an account — in the database, in inter-service messages, in telemetry, in logs, in any partition or sharding key — use a synthetic UUID generated at account creation. The email is stored once, on the account record, encrypted; it is never used as the identifier anywhere else.

This is the single most important boring detail in the PRD. Email is PII. Any service that derives identifiers from email leaks PII into every log line, every Kafka partition key, every shard map, every aggregate analytics event, every error message. We've seen this exact failure mode produce compliance findings in our other systems — an engineer reaches for email as a "convenient unique identifier," and six months later, PII is sprayed across observability tooling no one can fully audit. The synthetic UUID rule cuts that off at the source: email lives in exactly one place, and every other reference is the UUID. This rule is non-negotiable and is the kind of thing that's easy to honor at design time and impossible to retrofit.

## Account export

A user can export a JSON snapshot of their aviary state — birds, names, current personality vectors, current moods, notebook entries, account settings — from account settings. The export is generated on demand and emailed to the verified address as a download link. This is a quiet quality-of-life feature, not a marketed one; the rationale is that the user's relationship with their birds is theirs, and they should be able to take a copy if they want.

## Account deletion

Deletion is soft for 30 days, then hard. A user who initiates account deletion sees their account marked for deletion immediately; they can sign in during the 30-day window and recover the account (clicking "I changed my mind" on any signed-in page restores it). After 30 days, the deletion is hard: birds, vectors, notebook, telemetry, every record tied to the account, gone.

Soft-deletion protects against the regret of an accident — clicking delete and immediately wishing they hadn't. Hard-deletion after the window honors the privacy commitment that interaction history is the user's, not ours to keep indefinitely.

---

## Server-side simulation tick

The aviary's canonical state advances on a server-side simulation tick at a slow cadence (~once per minute; exact cadence calibrated during build). The tick reads the recent interaction-event log, updates personality vectors, transitions moods, advances mood timers, and writes the new canonical state. The tick runs whether or not any client is connected to the account.

This is the architectural decision that makes "the aviary continues without the viewer" actually true. The aviary the user comes back to after a day away is the aviary that has been ticking on the server for a day, not the aviary as it was when they left, frozen and resumed. Personality drifts during the user's absence based on inputs from before they left, not on inputs invented at the moment they return; mood transitions through morning, afternoon, evening as time passes; the aviary has its own continuity independent of the client.

The same architecture is also what makes multi-device sync coherent. Because the server is the only writer of personality state, a user signing in from a phone after using the laptop sees the same canonical aviary, in the same mood, with the same drift history. The client renders snapshots, never owns state. There is no syncing of personality between devices because there is nothing to sync — both clients are reading the same record.

If the simulation tick were on the client, the entire model collapses. A laptop that's been closed for a day and a phone that's been closed for a day would each present "their" version of the aviary on next open, and either the user picks one (which throws away the other) or the system tries to merge two divergent simulations (which corrupts both). The server-side tick is what avoids ever being in that position. It's the implementation rule that turns the product's central conceit into a real architectural property.

## How clients consume state

The client opens the aviary, requests the current state snapshot from the server, and begins rendering. The snapshot includes per-bird positions, current moods, current call timing, and any active animations or transitions. The client interpolates between snapshots for smooth motion — a bird at perch A in snapshot N and perch B in snapshot N+1 is rendered moving smoothly between them, not teleporting.

The client pulls a fresh snapshot on visibility change (tab becoming visible after being hidden), on long render-frame gaps (handling the case where a laptop has been suspended), and on a low-frequency keepalive while the tab is visible. Snapshots are small — kilobytes, not megabytes — so the cost of pulling is low.

The client writes interaction events (offer, listen-in start/end, settle, presence pings) to an append-only event log on the server. Clients never write personality state directly. The simulation tick consumes the event log on its next pass.

## Multi-device sync

Multi-device sync is a property of the architecture, not a separate feature. Because the server is the only source of canonical state and the only writer of personality, the user's aviary on their laptop and on their phone is the same aviary. They sign in on both devices, both devices pull the same snapshots, both devices show the same moods and drift.

There is no client-to-client sync, no client-side state to merge, no eventual consistency to reconcile. The two clients are both reading from one canonical record.

## No last-write-wins for personality state

Personality drift is implemented as additive, server-authored deltas — never as client-submitted absolute values. The simulation tick computes a delta from the recent event log and applies it to the existing personality vector. A client never sends "set boldness to 0.62"; a client sends "user listened in to Pip for 3 minutes," and the server decides what that means for boldness.

A last-write-wins model on personality state would let one device overwrite drift recorded from a previous session on another device. Imagine: laptop session in the morning writes a personality update; phone session at lunch (which started before the laptop session ended) writes its own version of the same vector based on its older read; the lunch write wins, the morning's drift is silently deleted. The user never sees the failure — they just have a bird that's drifting more slowly than it should — and there is no log entry that says "we lost data here." Additive server-authored deltas, processed in event-log order, make this failure mode unreachable. This is the implementation rule that makes the server-side simulation actually correct rather than a label.

Concretely: only the server simulation tick writes personality vectors. Clients write interaction events into the append-only event log. The tick consumes the log in order. No client mutates personality directly under any code path.

## Sync conflict surface

In rare cases — magic-link replay, an in-flight session timing out mid-write, a server-side outage — the user may encounter a sync conflict surface or an account-level error. The error surface drops out of the naturalist voice and into matter-of-fact tone:

> We couldn't sign you in. The link may have expired. Try requesting a new link.

> Your session timed out. Sign in again to keep watching.

> Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.

Naturalist phrasing here would read as evasive — a user who's been blocked from signing in needs to know what happened and what to do, and they need it without the product affecting a tone. This is the named exception to the naturalist voice rule, and it applies anywhere the user is talking to the system as a system: sign-in, account settings, sync errors, accessibility settings.

---

## Privacy

The privacy commitment is short and substantive. Per-bird interaction events — every offer, every listen-in, every presence ping — are stored only to drive that user's own simulation. They are never aggregated for model training, never used to build recommendation features for other users, never shared with any third party, never used to inform population-level analysis of how birds are typically interacted with.

This constraint shapes the entire telemetry boundary downstream. Aggregate telemetry — request counts, latencies, error rates, anonymized session-duration histograms — is collected for operational health. Per-bird, per-account interaction state is never part of that telemetry. The line between "is this account having errors" (allowed) and "what is this account's bird doing" (not allowed for any aggregate purpose) is hard, named, and respected at the data-pipeline level rather than at the policy level.

The reason this commitment is substantive content of the PRD rather than boilerplate is that the user's per-bird interaction history is their relationship with their birds. Aggregating it for any purpose — even an anodyne "average drift across all accounts" dashboard — converts a private relationship into a data product, and the product is precisely the thing the user is trusting us not to do. The constraint is also load-bearing for the engineering boundary it implies: telemetry pipelines never touch the per-account simulation database; the simulation database is never read by the analytics warehouse; ML training, if it ever exists for any feature, never receives per-bird fields. This reads as a privacy claim and behaves as an architectural rule.

## Aggregate telemetry

We do collect aggregate operational telemetry. Counts of requests, latencies on simulation-tick computations, error rates, session-duration histograms (anonymized, no per-account dimension), client-side render-frame timing, audio-pipeline error counts. None of this contains per-bird state, per-account interaction history, or anything that could be used to reconstruct a user's relationship with their aviary.

The privacy policy lives in account settings as a link, in plain text, naming the aggregate categories and explicitly excluding per-bird interaction state.
