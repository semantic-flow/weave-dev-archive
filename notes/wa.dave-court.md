---
id: lw7a7nzv37gd7o405loyfa6
title: Dave Court
desc: 'Open decision cards for Dave — ungated; position is the status, swept at ruling time'
updated: 1785606064695
created: 1785606064695
---

This note is Dave's court: open decision cards only, one card per decision, each with a stated lean. Position is the status — a card's presence means the decision is open, and it is swept at ruling time, with the ruling recorded where it belongs (the owning task note and/or [[wd.decision-log]]). A ruling that spawns work becomes a task note and enters [[wd.queues]]. This note is deliberately not gate-enforced: its writers are Jimbo (cards) and Dave (rulings), and drift here is visible to its only reader. A task waiting on Dave may hold a [[wd.queues]] entry AND a card here — the two surfaces are allowed to name the same task.

## Markdown site pipeline — one question left

**RULED 2026-08-06**, three of four:

- **Wikilink forms: narrow now.** Target, alias, and heading anchor only. Note references, block references, and tags are out of v1.
- **Missing or unpublished target: disabled link.** Not a build failure, not plain text — the link renders in a disabled state, so the authoring intent stays visible without the page shipping a broken href.
- **Conversation transcripts: unpublished by default**, opt-in required.

Recorded on [[wa.task.2026.2026-08-06_0854-markdown-site-pipeline]] and in [[wd.decision-log]].

### Still open: what identity means, and how vaults get namespaced

Dave asked whether "Dendron identity" means the note's IRI. It means two things I conflated, and they can be answered differently:

**(a) The wikilink lookup key** — what `[[wd.todo]]` matches against in the note index. Dave's lean of the dot-hierarchy filename is straightforwardly right here: it is what authors actually type, and matching anything else would make wikilinks unwritable.

**(b) The published identity** — the designator path, and therefore the IRI, of the resulting Knop and page. This is where the lean has a cost worth ruling on deliberately.

**Per-vault namespacing: yes, and it dissolves the collision problem.** Mount each vault at its own designator path, and the dot-hierarchy becomes the path beneath it — `wd.todo` in the weave vault and `ac.todo` in accord land at different IRIs by construction. Cross-vault collision then cannot occur; the vault segment disambiguates before the hierarchy is consulted. This workspace already uses disjoint prefixes (`wd.`, `wa.`, `ac.`, `ont.`, `sf.`), but that is a convention, and a mount point makes it a guarantee.

**The cost the lean does not yet cover: renames break IRIs.** If the designator path derives from the filename, renaming `wd.todo` to `wd.backlog` moves the Knop, and every prior citation of the old IRI dangles. That matters more here than in ordinary Dendron publishing, because the whole point of a mesh is stable, citable, versioned identifiers — a resource whose IRI changes when someone tidies a filename is not really stable.

Dendron itself resolves this by keeping both: the frontmatter `id` is durable across renames, while the filename carries hierarchy and URL shape.

**Recommendation:** filename hierarchy determines the designator path and IRI (readable URLs, and the hierarchy is the site structure an SSG needs), **and** the frontmatter `id` is recorded on the Knop as a durable alias. Renames then remain traceable — the old identity is still resolvable through the alias rather than silently lost — without giving up human-readable IRIs. That is strictly more than either choice alone and costs one triple per note.

**To rule:** accept the recommendation, or accept filename-only and treat renames as republications with dangling prior IRIs.
