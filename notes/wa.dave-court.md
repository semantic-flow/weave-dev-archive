---
id: lw7a7nzv37gd7o405loyfa6
title: Dave Court
desc: 'Open decision cards for Dave — ungated; position is the status, swept at ruling time'
updated: 1785606064695
created: 1785606064695
---

This note is Dave's court: open decision cards only, one card per decision, each with a stated lean. Position is the status — a card's presence means the decision is open, and it is swept at ruling time, with the ruling recorded where it belongs (the owning task note and/or [[wd.decision-log]]). A ruling that spawns work becomes a task note and enters [[wd.queues]]. This note is deliberately not gate-enforced: its writers are Jimbo (cards) and Dave (rulings), and drift here is visible to its only reader. A task waiting on Dave may hold a [[wd.queues]] entry AND a card here — the two surfaces are allowed to name the same task.

## Merge PR #31 — nested extraction sources without root Knops

- Decision: merge https://github.com/semantic-flow/weave/pull/31 (extractor bite 1: assertion fix + fail-on-old regression; full ci + build:npm-lib green locally; implemented by Kim/codex under the standing grant, reviewed and landed by the planning seat). Merge GO stays yours.
- Lean: merge on green checks — the change is exactly the carved bite, and the extractor queue entry keeps its place for the remaining slices.

## Binary-payload task: park or schedule?

- Decision: whether [[wa.task.2026.2026-08-01_1411-binary-payload-advancement]] is worth doing at all right now. Dave challenged the premise 2026-08-01 ("why would we do binary payload advancement now? was there a stagecraft use I missed?") — audit answer: **no consumer ask exists.** Neither feedback round mentions binary payloads; the "press includes non-text artifacts" line is a Stagecraft PM-seat Open Issue in the 07-21 note that also deferred press consumption to their own lane. Requirement 4 in [[wa.task.2026.2026-06-30_1108-stagecraft-driven-semantic-flow-requirements]] has been reclassified accordingly.
- Lean: **PARK the task, do not schedule it** — but keep the note, because the underlying code defect is real and now documented (later advancement decodes bytes; later renderers mistype binary payloads as `sflo:RdfDocument`). Trigger to revive: a real binary payload hitting later advancement through CLI `version`, or the Stagecraft press lane landing. The five spec adjudications inside the note wait for that trigger; nothing is owed now.
- If instead you want it scheduled, the five leans are in the note's Open Issues (include standalone update + binary overwrite, one shared classifier, explicit binary plan fields, next-minor release).

## Send the Stagecraft reply

- Decision: send [[wd.consumer-feedback.0.5.1.reply]] — the canonical draft covering both feedback rounds (2026-07-28 and 2026-07-29); it tells them to skip 0.5.1 and pin 0.6.0.
- Waits on: verifying v0.6.0 is live on npm.
- Lean: verify and send as-is; the draft was written for exactly this moment.


