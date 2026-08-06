---
id: oqbnhexpcy6ap5hoq2k0kjy
title: Markdown site pipeline
desc: 'Replace the hand-rolled regex renderer with a unified/remark/rehype pipeline so mesh-held Markdown DigitalArtifacts render as ResourcePages — the static-site-generator endpoint, eventually via the public API'
updated: 1786031652000
created: 1786031652000
---

## Goals

- Render mesh-held Markdown DigitalArtifacts — developer docs, user guides, conversation transcripts — as ResourcePage content rather than raw-source panels.
- Support Dendron flavour: hierarchical note identity, frontmatter, and `[[wikilinks]]` resolved against mesh state.
- Converge Markdown, HTML fragments, and `.txt` on one safe document representation.
- Eventually expose site building through the public API (RULED 2026-08-06 by Dave as the endpoint).

## Summary

Weave currently renders authored Markdown through a hand-rolled regex renderer (`renderMarkdownRegion` in `src/runtime/weave/pages.ts:3561`). It handles ATX headings, flat `-` lists, paragraphs, and a few inline forms; it silently misses or mangles fenced code, ordered/nested/task lists, blockquotes, images, tables, setext headings, autolinks, footnotes, thematic breaks, heading IDs, wikilinks, and nested inline markup. Raw HTML is escaped rather than parsed.

A blind codex evaluation on 2026-08-02 — brief withheld the prior decision and Dave's lean — chose **`unified`/`remark`/`rehype`** over `markdown-it`. Full reasoning, decision table, and the nine-slice migration plan are in [[wa.discussion.2026-08-08-markdown-renderer]]. The decisive axis is that markdown-it exposes a token stream rather than an AST, and this endpoint needs note indexing, mesh-state link resolution, and derived headings/link-graphs/text/RDF.

This reverses `sflo.conv.2025-11-29-rdf-storage-options`, which chose markdown-it — correctly, for the narrower "render Markdown to HTML" question it was answering. The scope grew; the answer moved with it.

**Priority: RULED 2026-08-06 — this sits BELOW the existing three queue items.** The Stagecraft-facing work keeps precedence; this proceeds in slices as capacity allows.

## Discussion

### Why this is a site generator, not a renderer swap

The `mesh-alice-bio` fixture shows the current mechanism straining. `/alice`'s page definition sources its `main` region from `<…/alice/bio>` — a peer Knop with its own identity, history, and page. The result:

| Page | `<h1>` | Panels |
|---|---|---|
| `/alice` | `Alice Ghostley` | rendered markdown + children, no source panel |
| `/alice/bio` | `bio` | raw source dump + history tree |

The artifact's own page renders it worst; the borrower renders it properly. `/alice`'s heading is governed by `alice/bio`'s frontmatter, and the same bytes are published at two URLs with no canonical relation declared.

That borrowing is not a design error — it is a workaround for a missing capability. Generic resource pages expose payloads as raw-source panels and cannot treat a Markdown DigitalArtifact as a page body. Once they can (slice 8), `/alice` simply owns its own markdown payload.

Dave's ruling on the pattern (2026-08-06): **compose from content, reference from identity.** A fragment that exists to be included (`mesh-content/sidebar`) is a fine `ResourcePageSource`; a resource that stands on its own should be linked, not inlined, and a page wanting a custom look should use a purpose-made source rather than borrowing a peer's payload.

### The security defect

`resolveMarkdownHref` (`src/runtime/weave/pages.ts:3734`) accepts any URI scheme — the guard `/^[a-zA-Z][a-zA-Z0-9+.-]*:/` matches and returns the href verbatim, so `[x](javascript:...)` produces a live `href`. Verified in source 2026-08-02.

Not exploitable today: mesh content is self-authored and the published site has no authored regions. This task's own goal is what converts it into a real vector — conversation transcripts and integrated third-party sources are not self-authored. `rehype-sanitize` fixes it as a side effect of the migration, so it is folded here rather than cut as a separate slice, **unless authored content starts publishing before this lands**, in which case take the three-line scheme allowlist immediately.

## Slice 2 receipts — packaging proof PROVEN (2026-08-06)

Branch `lane/spike-unified-packaging` at `e84fdba`, stacked on the ESM commit. **Not for merge** — see the temporary artifacts below.

**Verdict: the unified pipeline works under both Deno and dnt/Node.** Codex hit a DNS wall in its sandbox and correctly reported BLOCKED rather than fabricating results; the proof was completed directly.

Real Dendron markdown (`alice-bio.md`) renders correctly: frontmatter recognized and **excluded** from output (0 leaks), `[[user.bob-newhart]]` left **literal** for Weave-owned resolution, headings slugged with the namespaced prefix (`weave-overview`, `weave-bob-newhart`). dnt builds the full graph; the off-tree consumer renders under real Node v26.3.0.

**Determinism — all three comparisons byte-identical:**

| check | result |
|---|---|
| same process, rendered twice | identical |
| two fresh processes | identical |
| **Deno output vs Node output** | **identical** |

**Memory does not accumulate with batch size:**

| documents | Deno wall / peak RSS | Node wall / peak RSS |
|---:|---|---|
| 100 | 0.52s / 207 MB | 0.57s / 192 MB |
| 500 | 2.04s / 237 MB | 2.04s / 230 MB |
| 1,467 | 5.83s / **242 MB** | 5.61s / **233 MB** |

14.7× the documents costs ~17% more memory. Rendering the full SFLO page count is ~6 seconds and about a quarter gigabyte — worth contrasting with the weave path, which peaks at 3.79 GiB at N=1,700. Method: 63 distinct real sources (`alice-bio.md` plus top-level `documentation/notes/*.md`) cycled to reach each batch size, one reused processor per batch, HTML counted and discarded; wall and peak RSS from `/usr/bin/time` on a fresh process.

### Constraints found

1. **dnt entry points must live under `src/`** — recorded durably in [[wd.library-packaging]] § Packaging constraints discovered in practice, since it is a packaging fact rather than a Markdown one. Constrains npm subpath exports only, not imports or design. Relevant to slice 10.
2. **`remark-frontmatter`'s `yaml` node is not in the bare unified `Plugin` tree type.** A string-typed `visit(tree, "yaml", …)` narrows the node to `never` and fails type checking. Either import the mdast augmentation or visit untyped and discriminate by hand — the spike does the latter, with a comment explaining why. Slice 4 will hit this.

### Temporary artifacts — must be reverted before anything ships

- the `./unified-packaging-spike` subpath export in `scripts/build-npm-lib.ts`
- `src/runtime/content/spike_unified_render.ts` (placed under `src/` only because of constraint 1)
- `scripts/spikes/unified-packaging/{probe-deno,smoke-node}.ts`

None of this is on `main`. A temporary npm export reaching a release would be a real defect, so it is worth a guard — see Open Issues.

## Open Issues

Four gate the contract slice:

1. **Dendron identity** — is canonical identity the dot-hierarchy filename, frontmatter `id`, an RDF identifier, or a combination? Decides collision behavior across vaults.
2. **Wikilink forms contractual in v1** — recommendation: target, alias, and heading only; not note refs, block refs, or tags.
3. **Missing/unpublished target behavior** — build failure, warning plus plain text, disabled link, stub page, or link elsewhere?
4. **Default publish state for conversation transcripts** — a privacy decision wearing a rendering costume. Recommendation: unpublished by default, opt-in required.

Plus one raised by the spike itself:

5. **Where does scratch/spike code live, and what stops it shipping?** The spike had to put a module under `src/` (constraint 1) and declare a temporary npm export. Nothing structural prevented that reaching a release except branch discipline. Recommendation: a CI guard asserting `scripts/build-npm-lib.ts` declares exactly one entry point, which makes a stray export impossible to merge rather than merely discouraged — location conventions alone do not prevent the failure. See [[wd.general-guidance]] once ruled.

Further questions (frontmatter strictness and RDF precedence, embedded-HTML policy and sanitizer allowlist, `.txt` rendering shape, which DigitalArtifact roles become pages automatically, transient vs materialized render facts, byte-stability scope for third-party npm installs, shell-preserving vs deliberate regeneration) are enumerated in [[wa.discussion.2026-08-08-markdown-renderer]] § Open Issues.

## Decisions

- **2026-08-02 (blind codex evaluation):** `unified`/`remark`/`rehype` over `markdown-it`; `micromark` directly considered and rejected.
- **2026-08-06 (Dave):** the endpoint is building sites via the **public API**. Page generation is not currently exported from `src/api/mod.ts` — that export is its own contract decision, and given `wd.programmatic-validate-api` and `wd.programmatic-version-api` were both ratified before implementation, this surface deserves the same.
- **2026-08-06 (Dave):** ranks below the existing three queue items.

## Contract Changes

None until the API slice. The internal pipeline change is invisible to consumers except through generated page bytes, which will move — see byte-stability below. When site building joins the public API, that is a new ratified contract note, not an incidental export.

## Testing

- Byte snapshots for CommonMark, GFM, Dendron wikilink, HTML-fragment, `.txt`, and hostile-input fixtures (quotes, `<`, `&`, `</script><script>`, U+2028/U+2029, astral Unicode, `javascript:` hrefs).
- Same-timestamp double render of a representative site compared **by hash**, not by DOM.
- Deno and Node ESM builds both rendering real Markdown.
- Fail-on-old recorded for the current renderer's behavior before replacement.

## Non-Goals

- Retiring `ResourcePageSource` — it stays the right mechanism for genuinely shared fragments.
- Search-ranking or rich-result promises.
- HTML-to-Turtle round-trip: embedded output is a generated projection, never an import source.
- Enabling raw HTML inside Markdown, or adding `rehype-raw`, in the first cut.

## Implementation Plan

Nine independently landable slices, detailed in [[wa.discussion.2026-08-08-markdown-renderer]]:

- [ ] 1. Contract and fixtures — Dendron semantics, current-byte capture, hostile inputs. **Blocked on the four rulings above.**
- [~] 2. Packaging proof — exact pins, frozen lock, Deno + Node ESM builds, real render in both, time and memory measured. **ESM-only landed as PR #39 (2026-08-06), clearing the packaging gate; the spike proves the rest.**
- [ ] 3. Source/compiled model boundary, populated with the old renderer's output so bytes do not move.
- [ ] 4. Unified Markdown compiler behind unit tests, not routing production pages.
- [ ] 5. Two-pass Dendron indexing — discovery, duplicate detection, wikilink node, resolution policy, diagnostics.
- [ ] 6. Authored-page cutover behind an explicit renderer version; the direct fallback consumes the same compiled result.
- [ ] 7. HTML fragment and `.txt` adapters through the common tail.
- [ ] 8. DigitalArtifact site generation — Markdown artifacts become page content rather than raw-source panels. **This is the slice that removes the need for the alice/bio borrowing.**
- [ ] 9. Renderer epoch and cleanup — regenerate in scratch, review the publication diff, switch, then delete the regex renderer and the direct-rendering bypass.

Plus, sequenced after the pipeline proves out:

- [ ] 10. Site building joins the public API, under its own ratified contract note.
