---
id: lw7a7nzv37gd7o405loyfa6
title: Dave Court
desc: 'Open decision cards for Dave — ungated; position is the status, swept at ruling time'
updated: 1785606064695
created: 1785606064695
---

This note is Dave's court: open decision cards only, one card per decision, each with a stated lean. Position is the status — a card's presence means the decision is open, and it is swept at ruling time, with the ruling recorded where it belongs (the owning task note and/or [[wd.decision-log]]). A ruling that spawns work becomes a task note and enters [[wd.queues]]. This note is deliberately not gate-enforced: its writers are Jimbo (cards) and Dave (rulings), and drift here is visible to its only reader. A task waiting on Dave may hold a [[wd.queues]] entry AND a card here — the two surfaces are allowed to name the same task.

## Rename semantics — is a rename a MOVE or a SUPERSESSION?

Identity RULED 2026-08-06: filename hierarchy determines the designator path and IRI; frontmatter `id` is recorded on the Knop as a durable alias; each vault mounts at its own designator path so cross-vault collision cannot occur. Wikilink lookup matches the dot-hierarchy filename.

Dave then asked to consider **optional alias or redirect pages when filenames change**. That is a good idea, and pursuing it surfaces a prior question the identity ruling does not settle. What an alias page *is* depends entirely on the answer.

### The prior question

**MOVE** — identity is continuous. The whole ArtifactHistory relocates under the new designator path; the old path gets a redirect stub.

- Cost: relocating a history tree is a rewrite, which cuts directly against the append-onlyish direction in [[wa.task.2026.2026-05-17-append-onlyish-inventory]].
- Worse cost: **every historical state's IRI changes too.** A citation of `…/wd/todo/_history001/_s0003` — an exact state, the thing a mesh exists to make citable — dangles. The redirect stub only covers the resource root unless we emit one per state.

**SUPERSESSION** — nothing moves. The old Knop stays exactly where it is, frozen, with its full history intact and permanently resolvable. A new Knop begins at the new path. The two are linked by a supersession predicate, and the frontmatter `id` is what states they are the same authored note across the boundary.

- Nothing is rewritten; append-only by construction.
- Every prior citation, including exact-state citations, stays valid forever.
- Cost: the new Knop starts with no history, so one authored note has two identity chains.

**Lean: SUPERSESSION.** The cost is real but honest — a renamed note genuinely *is* a new identifier, and content continuity is better expressed by an explicit link than by pretending the IRI never changed. The move model buys tidiness by breaking exactly the guarantee the mesh exists to provide. It also makes the `id` alias do real work rather than being decorative.

Under supersession, the alias page becomes a thin courtesy artifact rather than a load-bearing identity mechanism — which is the right weight for it.

### Mechanism constraint, if we do emit redirect pages

**Server-side redirects are unavailable on the primary publication target.** The GitHub Pages profile requires `.nojekyll` (`src/runtime/publication/presets.ts:53`), which disables Jekyll entirely — so `jekyll-redirect-from` cannot work, and Pages offers no other redirect configuration.

That leaves a client-side stub: `<meta http-equiv="refresh" content="0; url=…">` plus `<link rel="canonical">` and `noindex`. It works on any static host. It is what Jekyll's redirect plugin actually emits. It is also a real page that exists and is crawlable, which the canonical and noindex mitigate but do not erase.

### Vocabulary

SFLO has **no** supersession or alias term today — the only adjacent thing is `referenceRole_deprecated`. Two paths:

- **`dcterms:isReplacedBy` / `dcterms:replaces`** — established, precisely defined ("supplants, displaces, or supersedes"), and needs **no SFLO release**. Cheap and correct.
- **Mint an SFLO term** — more expressive if rename semantics need mesh-specific nuance, but it is an ontology change requiring a versioned SFLO release before Weave can emit it.

Recommendation: `dcterms` first. Mint SFLO vocabulary only if a real need appears that dcterms cannot express. Avoid `owl:sameAs` — it asserts the IRIs denote the same individual, which drags in reasoning consequences we do not want and is arguably false once the two Knops have divergent histories.

**To rule:** (a) move or supersession; (b) whether redirect stubs are emitted at all, and if so opt-in per rename or automatic; (c) dcterms or a minted SFLO term.

## Markdown site pipeline — contract rulings

**RULED 2026-08-06**, three of four:

- **Wikilink forms: narrow now.** Target, alias, and heading anchor only. Note references, block references, and tags are out of v1.
- **Missing or unpublished target: disabled link.** Not a build failure, not plain text — the link renders in a disabled state, so the authoring intent stays visible without the page shipping a broken href.
- **Conversation transcripts: unpublished by default**, opt-in required.

Recorded on [[wa.task.2026.2026-08-06_0854-markdown-site-pipeline]] and in [[wd.decision-log]].

### Identity — RULED 2026-08-06

Filename hierarchy determines the designator path and IRI; frontmatter `id` recorded on the Knop as a durable alias; each vault mounts at its own designator path, so cross-vault collision cannot occur by construction. Wikilink lookup matches the dot-hierarchy filename.

The rename consequences this ruling exposes are carded separately above.
