---
id: lw7a7nzv37gd7o405loyfa6
title: Dave Court
desc: 'Open decision cards for Dave — ungated; position is the status, swept at ruling time'
updated: 1785606064695
created: 1785606064695
---

This note is Dave's court: open decision cards only, one card per decision, each with a stated lean. Position is the status — a card's presence means the decision is open, and it is swept at ruling time, with the ruling recorded where it belongs (the owning task note and/or [[wd.decision-log]]). A ruling that spawns work becomes a task note and enters [[wd.queues]]. This note is deliberately not gate-enforced: its writers are Jimbo (cards) and Dave (rulings), and drift here is visible to its only reader. A task waiting on Dave may hold a [[wd.queues]] entry AND a card here — the two surfaces are allowed to name the same task.

## Send the Stagecraft reply

- Decision: send [[wd.consumer-feedback.0.5.1.reply]] — the canonical draft covering both feedback rounds (2026-07-28 and 2026-07-29); it tells them to skip 0.5.1 and pin 0.6.0.
- Waits on: verifying v0.6.0 is live on npm.
- Lean: verify and send as-is; the draft was written for exactly this moment.

## Rename the two landed 07-29 task notes

- Decision: rename `wa.task.2026.2026-07-29_1219-programmatic-validate-mesh-api` and `wa.task.2026.2026-07-29_1220-whole-mesh-validate-bounded-memory` to `wa.completed.*` (renames are Dave's act per `AGENTS.md`; update affected wikilinks with the rename).
- Lean: rename both — v0.6.0 shipped their content; the `closure` groom duty flags them daily until then.

## Arm the first supervised loop wake

- Decision: schedule the dry run — one `/loop 10m` cycle with Dave watching, the last open implementation-plan item on [[wa.task.2026.2026-07-31_1014-planning-loop-infrastructure]]. Seating order is in [[wd.read-in.jimbo]] § Loop prompt.
- Lean: next working session; adjust prompt/read-in from what the wake actually needed rather than what was predicted.

## Ratify D1, D4, D5, D6, D8

- Decision: the five still-proposed decisions on [[wa.task.2026.2026-07-31_1014-planning-loop-infrastructure]] (queue name/sections; 10m cadence; no CI wiring for the gate itself; the named SURVEY repo set; mechanical wake/groom stamps) are implemented as specified — D2/D3/D7 were ruled 2026-07-31.
- Lean: ratify as built; the deviations found in review r1 are recorded in the task note's disposition section.
