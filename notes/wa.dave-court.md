---
id: lw7a7nzv37gd7o405loyfa6
title: Dave Court
desc: 'Open decision cards for Dave — ungated; position is the status, swept at ruling time'
updated: 1785606064695
created: 1785606064695
---

This note is Dave's court: open decision cards only, one card per decision, each with a stated lean. Position is the status — a card's presence means the decision is open, and it is swept at ruling time, with the ruling recorded where it belongs (the owning task note and/or [[wd.decision-log]]). A ruling that spawns work becomes a task note and enters [[wd.queues]]. This note is deliberately not gate-enforced: its writers are Jimbo (cards) and Dave (rulings), and drift here is visible to its only reader. A task waiting on Dave may hold a [[wd.queues]] entry AND a card here — the two surfaces are allowed to name the same task.

## sflo published corpus is on legacy source-binding vocabulary

- Decision: what to do about the `sflo` `gh-pages` mesh, which current `main` can no longer generate. Discovered 2026-08-01 by the bite-3 probe: standalone `generate` fails in 0.2s at `loadDesignatorContexts` with "Could not resolve the current extracted source target for config/allowedRemoteOrigin" because the corpus binds with `sflo:hasTargetArtifact` / `sflo:hasRequestedTargetState` while current code resolves `sflo:targetArtifact` / `sflo:targetHistoricalState`. Nothing was mutated; this is a corpus-vintage issue, not an extractor defect. Note this is our own flagship published ontology site.
- Options: (a) regenerate/migrate the corpus to current vocabulary — pre-v1 posture ("avoid backward-compatibility shims") points here, and it would restore a genuine 379-Knop real-corpus test substrate; (b) deliberately support the legacy vocabulary in the resolver — contradicts the no-shims rule; (c) leave it stale and drop sflo as a test substrate.
- Lean: **(a) regenerate.** It fits the pre-v1 no-shims rule, unblocks the real-corpus arm the extractor work still needs, and republishing the ontology site is independently overdue. Wants its own task note if you agree — the regeneration touches a published gh-pages branch, so it is not a loop-wake bite.

## Send the Stagecraft reply

- Decision: send [[wd.consumer-feedback.0.5.1.reply]] — the canonical draft covering both feedback rounds (2026-07-28 and 2026-07-29); it tells them to skip 0.5.1 and pin 0.6.0.
- Waits on: verifying v0.6.0 is live on npm.
- Lean: verify and send as-is; the draft was written for exactly this moment.
