---
id: nh1izfge5mialx3b9godo22
title: Knop supersession and rename
desc: 'Renames are supersessions, not moves: the old Knop keeps its payload history permanently citable, dcterms:isReplacedBy links it forward via weave supersede, and the old page offers a cancellable redirect rather than vanishing'
updated: 1786034947000
created: 1786034947000
---

## Goals

- Let a Knop be renamed without breaking any prior citation, including exact-state citations.
- Keep the superseded resource's content and history browsable, not replaced by a stub.
- Offer readers arriving at a superseded IRI a clear, cancellable path to the continuation.

## Summary

Cut 2026-08-06 from the identity ruling on [[wa.task.2026.2026-08-06_0854-markdown-site-pipeline]]. **This is a mesh-level concern, not a Markdown one** — any Knop can be renamed, so it does not belong inside the Markdown pipeline note even though that is where the question surfaced.

**RULED 2026-08-06 (Dave): a rename is a SUPERSESSION, not a move.**

Nothing relocates. The old Knop stays exactly where it is, with its full ArtifactHistory intact and permanently resolvable. A new Knop begins at the new designator path. The two are linked by a paired `dcterms:isReplacedBy` / `dcterms:replaces`, and the new Knop records an exact-state provenance pointer to the history it continues from.

**Precision, corrected 2026-08-06:** "frozen" overstated it. The old Knop's **payload states are immutable and permanently resolvable** — that is what preserves citations — but the Knop itself **mints a new state** to carry the `isReplacedBy` fact. Recording a supersession is a change to the old Knop, and Weave records changes by minting states. A superseded Knop is therefore still-growing metadata over never-changing payload history, not a sealed object.

The rejected alternative was a MOVE: relocating the whole history tree under the new path. That is a rewrite (against the append-onlyish direction), and its decisive flaw is that it changes **every historical state's IRI** — a citation of `…/wd/todo/_history001/_s0003`, an exact state, dangles. Exact-state citability is the thing a mesh exists to provide; the move model would have bought tidiness by breaking it.

Accepted cost: one authored note carries two identity chains. This is honest rather than unfortunate — a renamed note genuinely *is* a new identifier, and content continuity is better stated explicitly than implied by pretending the IRI never changed. It also makes the frontmatter `id` alias load-bearing: `id` is what asserts "same authored note" across the supersession boundary.

## Discussion

### The old page keeps its value

Dave's framing, 2026-08-06: *"the old page might have valuable information."* Under supersession that is not a concession — it is the point. The superseded page renders in full, with its own panels and history tree, exactly as before. It gains a supersession notice; it does not become a stub.

This is the substantive difference from a conventional redirect. A hard redirect assumes the old URL is worthless. In a mesh, the old URL names a resource whose settled states are still true, still cited, and still worth reading.

### The notice and the cancellable redirect

Dave's sketch: a popover reading roughly *"The latest states of this content have moved to IRI; you will be redirected in 5 seconds, or click cancel to stay here."*

That wording is semantically precise and should be preserved: **the latest states moved; the old states did not.** Only the continuation is elsewhere. Language implying the whole resource moved would be false under supersession.

Design consequences:

- **The notice is RDF-driven.** It renders from `dcterms:isReplacedBy` on the superseded Knop, as an ordinary page region in the document model — not a special case bolted onto the renderer.
- **It degrades correctly by construction.** The notice is server-rendered HTML; only the countdown and the cancel control need JS. With JS disabled the reader sees the notice and a plain link, and is never redirected — which is the better failure mode, since the content stays visible.
- **JS is established precedent, not a new capability.** Generated pages already carry an inline script (`src/runtime/weave/pages.ts:1514`) that normalizes trailing-slash URLs against the canonical link via `history.replaceState`. All 1,467 published SFLO pages contain it. The countdown extends an existing pattern.

### Proposed refinement: never auto-redirect an exact-state URL

A reader arriving at a superseded IRI wants one of two incompatible things, and the URL tells us which:

- A **resource-root** IRI (`…/wd/todo`) is ambiguous — could be a stale bookmark wanting the current thing, could be deliberate.
- An **exact-state** IRI (`…/wd/todo/_history001/_s0003`) is unambiguously a deliberate citation. Nobody bookmarks a state ordinal by accident.

Auto-redirecting the exact-state case would punish precisely the reader the mesh exists to serve. Recommendation: **show the notice everywhere, auto-redirect only from the resource root**, never from an exact state or a history index. Cheap to implement, and it preserves citation semantics.

### Accessibility

A timed auto-redirect engages WCAG 2.2.1 (Timing Adjustable), which requires the user be able to turn off, adjust, or extend a time limit. Dave's cancel control satisfies "turn off" — so the cancel button is a conformance requirement, not merely good manners, and must not be dropped as a simplification. The notice should also be focusable and announced, not a purely visual overlay.

### Vocabulary

**RULED 2026-08-06 (Dave): `dcterms:isReplacedBy`.** Established, precisely defined ("a related resource that supplants, displaces, or supersedes the described resource"), and it requires **no new SFLO vocabulary term**.

`owl:sameAs` was rejected: it asserts the two IRIs denote the same individual, which is arguably false once the Knops have divergent histories, and it drags in reasoning consequences we do not want.

**Dave 2026-08-06: SHACL can reference it, and SHACL updates are routine.** A shape editing `semantic-flow-core-shacl.ttl` does mean an SFLO release, but that is expected churn rather than a cost to design around — sequence the shape with whatever release is convenient. I had flagged the release as a caveat; Dave's ruling correctly removes it as a consideration.

## Decisions

- **2026-08-06 (Dave): a rename is a SUPERSESSION**, linked with `dcterms:isReplacedBy`. See Summary.
- **2026-08-06 (Dave): `weave supersede` is the primitive; `weave rename` is `mv` + `supersede`.** `repoint` was rejected as a name because it already means changing a `ResourcePageSource`'s target in this workspace.
- **2026-08-06 (Dave): a rename needs an explicit command.** A rename needs an explicit signal, because Weave cannot infer one. When an author renames a Dendron note in their editor, all Weave observes is that one designator's working file vanished and another appeared — indistinguishable from a delete plus an unrelated create. Nothing in the mesh distinguishes those cases, and guessing from content similarity would be a heuristic making identity claims, which is precisely what a mesh must not do. The command supplies the intent that the filesystem cannot.
- **2026-08-06 (Dave): store one hop.** If A is replaced by B and B by C, A records only `isReplacedBy B`. The chain resolves at render time, with cycle detection.

  **Corrected rationale (Dave, 2026-08-06):** I first justified this as avoiding a rewrite of settled facts. That was wrong. Recording a later hop would *mint a new state*, not rewrite an old one, and minting states is exactly how Weave records change — fully append-onlyish. The ruling stands, but on different grounds:

  - **Truth locality.** "A was replaced by B" is an observed fact about A. "A is superseded by C" is derived from a later event involving only B and C. A's record should carry what happened to A.
  - **Cost.** Full-closure means every rename mints a state on every ancestor — O(chain length) states per rename, each carrying downstream news rather than news about itself.
  - **It saves nothing operationally.** Under static publication, A's page must be regenerated for its notice to point at C whether the traversal happens at write time or render time. Storing the closure moves work rather than removing it.
- **2026-08-06 (Dave): SHACL updates are routine.** Constraining supersession in `semantic-flow-core-shacl.ttl` is not a cost to design around; SFLO SHACL is expected to change often. Sequence the shape with whatever SFLO release is convenient rather than deferring the work to avoid one.
- **2026-08-06 (Dave): the link is PAIRED.** The old Knop gets `dcterms:isReplacedBy <new>`; the new Knop gets `dcterms:replaces <old>`.

  **My earlier objection was wrong and is withdrawn.** I argued one-way on the grounds that pairing "would need writing to the old Knop as well, which is the settled-fact rewrite the ruling avoids." That was confused: the write to the old Knop is `isReplacedBy` itself, which happens regardless. Pairing adds a fact to the *new* Knop, which is brand new — free, and no rewrite of anything.

  The decisive argument for pairing is **locality under a file-per-resource model**. Weave publishes each resource's facts in its own file. A consumer reading the new Knop's inventory should be able to answer "what did this replace?" without scanning the mesh for an inbound `isReplacedBy`. An RDF reasoner could infer the inverse; Weave does not run one, and neither do its consumers. Denormalization is correct here.

  Pairing stays compatible with one-hop: both facts are written in the same operation, at supersession time, and neither is ever revised. In a chain A→B→C, B carries `isReplacedBy C` and `replaces A`, each written when it was true.

- **2026-08-06 (Dave): the new Knop records a provenance pointer to the superseded history.** It does not copy states.

  **Refinement worth ruling: point at the exact state, not just the old Knop.** A superseded Knop is not frozen — it mints states for the supersession itself, and it must remain correctable (the PII retraction ruling requires it). "I continue from that Knop" therefore becomes ambiguous as soon as the old Knop changes again. "I continue from `…/_history001/_s0003`" is precise, permanently true, and is exactly the exact-state citation the mesh is built to support.

- **2026-08-06 (Dave): an occupied target is refused, with no overwrite flag.** Refusal stands until a separate act removes the target Knop; `supersede` never deletes.

  Worth encoding in the diagnostic: superseding into an occupied path is not a rename at all, it is a **merge** — two identity chains converging on one designator. Merge semantics are undefined here and should not be invented as a side effect of a rename flag. The error should say so rather than suggesting an override that does not exist.

## Open Issues


- Auto-redirect default: on, off, or configurable per mesh? The exact-state refinement above assumes on-at-root.
- Should `weave validate` report a dangling `isReplacedBy` target, or a superseded Knop that still has pending work?
- Re-run safety: a second `supersede` with the same arguments should be a semantic no-op rather than minting a second supersession.

### Command shape — RULED 2026-08-06

**`weave supersede <old> <new>`** is the semantic primitive: it records the supersession and touches no files. **`weave rename <old> <new>`** is the convenience wrapper — move the working file, then supersede. `rename` = `mv` + `supersede`.

One semantic implementation, one set of tests against `supersede`; the filesystem step lives only in the wrapper.

Dave proposed splitting by who moved the file: `weave rename` moves it, and a second command records metadata for an already-moved file. **The split is right** — the two workflows are genuinely distinct:

- **Weave owns the file** — Weave performs the move and records the supersession.
- **The editor owns the file** — the Dendron case, and the realistic one for notes. Dendron's own rename refactor moves the file *and* updates wikilinks across the vault, which Weave should not try to replicate. Weave is told after the fact.

**But `repoint` is already taken.** It has an established meaning in this workspace: changing a `ResourcePageSource` or page region's target artifact. It appears in fixture-ladder rung descriptions (`wa.completed.2026.2026-05-25_0958-alice-14-19-ladder-redesign`, rungs 18–19 "repoint the page definition to the governed artifacts"), in the import task note, and in the maintenance log. Reusing it for supersession would collide with live vocabulary.

**Proposed instead:**

- **`weave supersede <old> <new>`** — the semantic primitive. Records the supersession; touches no files. Named for the act the ruling actually defines, and matching `dcterms:isReplacedBy`.
- **`weave rename <old> <new>`** — convenience: move the working file, then supersede. `rename` = `mv` + `supersede`.

This keeps one semantic implementation with one set of tests, adds a filesystem step in a thin wrapper, and leaves `repoint` meaning what it already means. It also reads correctly for non-note meshes, where "rename" is the natural word and the file move is wanted.

RULED 2026-08-06 (Dave): `supersede` accepted.

## Contract Changes

- New emitted RDF on superseded Knops (`dcterms:isReplacedBy`), and a `dcterms` prefix in generated output.
- A new page region in the document model for the supersession notice.
- Generated-page bytes change for superseded resources only. Unaffected resources must be byte-identical — this is testable and should be tested.
- A SHACL shape constraining supersession, sequenced with a convenient SFLO release (RULED 2026-08-06: SHACL churn is expected and is not a reason to defer).

## Testing

- A superseded Knop renders the notice; an ordinary Knop's bytes are unchanged.
- Exact-state and history-index URLs render the notice but emit no redirect timer.
- JS-disabled rendering shows the notice and a working link, with no redirect.
- Chain resolution and cycle detection.
- Byte-stable same-timestamp regeneration with a supersession present.
- The old Knop's full history remains resolvable after supersession — the regression that guards the whole ruling.

## Non-Goals

- Server-side redirects. The GitHub Pages profile requires `.nojekyll` (`src/runtime/publication/presets.ts:53`), which disables Jekyll and therefore `jekyll-redirect-from`; Pages offers no other redirect configuration. Client-side is the only option on the primary publication target.
- Merging identity chains, or any claim that the two Knops are the same individual.
- Retroactively superseding resources renamed before this ships.

## Implementation Plan

- [ ] `weave supersede <old> <new>`: record the supersession, refuse an occupied target, no-op on repeat.
- [ ] `weave rename <old> <new>`: move the working file, then delegate to `supersede`.
- [ ] Emit the paired `dcterms:isReplacedBy` / `dcterms:replaces` facts plus the exact-state provenance pointer; validate both targets resolve.
- [ ] Add the supersession region to the document model and render it from RDF.
- [ ] Countdown and cancel control, root-only, with the no-JS path proven.
- [ ] Chain resolution at render time with cycle detection (one hop stored).
- [ ] SHACL shape constraining supersession, with the next convenient SFLO release.
