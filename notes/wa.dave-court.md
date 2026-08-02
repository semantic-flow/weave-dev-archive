---
id: lw7a7nzv37gd7o405loyfa6
title: Dave Court
desc: 'Open decision cards for Dave — ungated; position is the status, swept at ruling time'
updated: 1785606064695
created: 1785606064695
---

This note is Dave's court: open decision cards only, one card per decision, each with a stated lean. Position is the status — a card's presence means the decision is open, and it is swept at ruling time, with the ruling recorded where it belongs (the owning task note and/or [[wd.decision-log]]). A ruling that spawns work becomes a task note and enters [[wd.queues]]. This note is deliberately not gate-enforced: its writers are Jimbo (cards) and Dave (rulings), and drift here is visible to its only reader. A task waiting on Dave may hold a [[wd.queues]] entry AND a card here — the two surfaces are allowed to name the same task.

## BLOCKER — sflo regeneration needs an SFLO v0.2.1 source release first

- Decision: the regeneration you ruled cannot run against the current released source. **SFLO v0.2.0's payload itself defines the legacy `sflo:hasTargetArtifact` / `sflo:hasRequestedTargetState` predicates**, while current Weave emits and resolves the renamed `sflo:targetArtifact` / `sflo:targetHistoricalState` — so regenerating from v0.2.0 would publish a mesh whose ontology defines one vocabulary while its own generated support artifacts use another. `next/v0.2.1` has the compatible vocabulary but still carries v0.2.0 release identities, so publishing those bytes at `releases/v0.2.0` would break immutable release identity.
- Consequence: **cut and tag an SFLO v0.2.1 source release first**, then regenerate from a detached worktree at that tag. This is a prerequisite task in the sflo repo, not part of the publication task.
- Lean: approve v0.2.1 as the source release and let me prepare it (sflo has its own `release:set-version` / `release:validate` tasks and a release runbook). The regeneration then runs against the tag.
- Also worth knowing: the vocabulary rename genuinely changes the term census — projected 379 → 362 Knops (60 retired, 43 added). That is expected, not drift, but it means surviving-term IRIs need spot-checking after publication.

## Three smaller sflo rulings (defaults are fine if you say nothing)

- `favicon.ico` came from the "lovely manual re-creation" commit and is the only host asset at risk. Lean: preserve it byte-for-byte. (`.nojekyll` is Weave-managed; there is no `CNAME` and no `assets/`.)
- The old two-commit chain: lean is an annotated `archive/gh-pages-before-regeneration-2026-08-01` tag so history stays auditable even though `gh-pages` becomes a single root commit. Say so if "first published version" should leave no discoverable archive ref.
- `riot` (the independent Turtle validator the recipe wants) is installed but unusable — no JDK on this box. Lean: install a JDK, else I substitute an equivalent parser gate and record the substitution.

## Send the Stagecraft reply

- Decision: send [[wd.consumer-feedback.0.5.1.reply]] — the canonical draft covering both feedback rounds (2026-07-28 and 2026-07-29); it tells them to skip 0.5.1 and pin 0.6.0.
- Waits on: verifying v0.6.0 is live on npm.
- Lean: verify and send as-is; the draft was written for exactly this moment.
