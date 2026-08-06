---
id: lw7a7nzv37gd7o405loyfa6
title: Dave Court
desc: 'Open decision cards for Dave — ungated; position is the status, swept at ruling time'
updated: 1785606064695
created: 1785606064695
---

This note is Dave's court: open decision cards only, one card per decision, each with a stated lean. Position is the status — a card's presence means the decision is open, and it is swept at ruling time, with the ruling recorded where it belongs (the owning task note and/or [[wd.decision-log]]). A ruling that spawns work becomes a task note and enters [[wd.queues]]. This note is deliberately not gate-enforced: its writers are Jimbo (cards) and Dave (rulings), and drift here is visible to its only reader. A task waiting on Dave may hold a [[wd.queues]] entry AND a card here — the two surfaces are allowed to name the same task.

## Markdown site pipeline — contract rulings

**RULED 2026-08-06**, three of four:

- **Wikilink forms: narrow now.** Target, alias, and heading anchor only. Note references, block references, and tags are out of v1.
- **Missing or unpublished target: disabled link.** Not a build failure, not plain text — the link renders in a disabled state, so the authoring intent stays visible without the page shipping a broken href.
- **Conversation transcripts: unpublished by default**, opt-in required.

Recorded on [[wa.task.2026.2026-08-06_0854-markdown-site-pipeline]] and in [[wd.decision-log]].

### Identity — RULED 2026-08-06

Filename hierarchy determines the designator path and IRI; frontmatter `id` recorded on the Knop as a durable alias; each vault mounts at its own designator path, so cross-vault collision cannot occur by construction. Wikilink lookup matches the dot-hierarchy filename.

Rename consequences were ruled the same day and are no longer open: a rename is a SUPERSESSION, linked with `dcterms:isReplacedBy`. Cut as [[wa.task.2026.2026-08-06_0949-knop-supersession-and-rename]].

*(No other open cards.)*
