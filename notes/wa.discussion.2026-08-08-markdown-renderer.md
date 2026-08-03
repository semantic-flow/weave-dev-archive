---
id: k9pisw9k12x7ke5y9x1c8yu
title: 2026 08 08 Markdown Renderer
desc: 'Blind codex evaluation of markdown-it vs unified/remark/rehype for the static-site-generator endpoint — recommends unified, reversing the 2025-11-29 pick under a changed scope'
updated: 1785726776715
created: 1785726776715
---

## Summary

A codex read-only evaluation on 2026-08-02 compared `markdown-it` against `unified`/`remark`/`rehype` for Weave's Markdown pipeline. The brief was deliberately **blind**: it did not mention the 2025-11-29 decision, its reasoning, or Dave's lean, so the result is independent rather than confirmatory.

**Result: `unified`/`remark`/`rehype` wins. `markdown-it` is the named loser.** This reverses `sflo.conv.2025-11-29-rdf-storage-options`, which picked `markdown-it` and rejected unified as "extremely powerful but very heavy."

**Both conclusions are correct for their own premise, and the premise is what changed.** The 2025 evaluation asked "what should render Markdown to HTML?" The 2026 brief asked "what should power a static site generator over mesh-held DigitalArtifacts, with Dendron flavour, semantic extraction, and HTML/text inputs alongside Markdown?" The analysis says so explicitly and unprompted: markdown-it *is* the better choice if the real scope is only "replace this regex renderer with safe, conformant Markdown-to-HTML." So this is not a contradiction to resolve — it is evidence that the scope grew, and the answer moved with it.

The decisive axis is architectural, not quality: markdown-it deliberately exposes a **token stream** rather than an AST. Weave's stated endpoint needs to index notes, resolve links against mesh state, derive headings/link graphs/extracted text/RDF, and converge Markdown, HTML fragments, and `.txt` on one safe representation. Unified supplies typed mdast/hast trees built for exactly that; with markdown-it, Weave would end up building that layer itself or bolting on a second HTML pipeline.

No third option dominates. Direct `micromark` was considered and rejected: remark already uses micromark and adds the mdast layer, so using it directly would mean recreating unified's orchestration.

## SECURITY FINDING — act on this independent of the library choice

`resolveMarkdownHref()` in `src/runtime/weave/pages.ts:3734` accepts **any** URI scheme. The guard is `/^[a-zA-Z][a-zA-Z0-9+.-]*:/`, which matches and returns the href verbatim — there is no allowlist. Authored Markdown containing `[text](javascript:...)` therefore produces a live `href="javascript:..."`. HTML escaping does not help; the scheme is the vector.

Verified directly in the source, 2026-08-02.

Currently latent: the published SFLO site has no authored regions, so the Markdown branch is not exercised there. **The goal described in this note is precisely what would activate it** — rendering notes, user guides, and especially *conversation transcripts* as pages means arbitrary authored text flowing into published `href` attributes at scale.

This is a small fix (scheme allowlist: `http`, `https`, `mailto`, and relative/fragment forms; reject the rest) and it should not wait for the pipeline decision.

## The decision table

Rows marked ★ drove the outcome; ◇ are implementation gates; ≈ were secondary.

| Constraint | Winner | Note |
|---|---|---|
| ★ AST access | unified, decisively | mdast/hast are conventional typed ASTs; markdown-it tokens are not, so cross-document analysis becomes application-owned machinery |
| ★ Multiple syntaxes | unified, strongly | Markdown→hast via `remark-rehype`; HTML fragments start as hast; text constructed as hast — one safe tail. markdown-it needs a second parser, AST, and sanitizer for HTML |
| ★ Dendron wikilinks | unified | Both can defer URL resolution to Weave. unified wins because syntax, resolution, diagnostics, and extraction stay separate tree transforms |
| ★ Security | unified | `rehype-sanitize` gives an allowlist sanitizer over the same hast used for both Markdown and HTML. markdown-it's safe Markdown defaults are good, but standalone HTML sanitization is outside its ecosystem |
| ★ Ecosystem cohesion | unified, strongly | Official maintained packages cover GFM, frontmatter, mdast→hast, HTML parsing, sanitization, slugs, traversal, and Shiki. markdown-it's equivalents are more fragmented |
| ◇ Deno + dnt | markdown-it | unified packages are ESM-only and will not survive the current unchanged CJS dnt build. See below — this is the real gate |
| ◇ Byte stability | markdown-it, slightly | Fewer moving parts. Both need exact pins, a renderer epoch, and byte-golden tests |
| ◇ Migration cost | markdown-it near-term | Closest drop-in. But the larger unified refactor is work the site-generator endpoint requires anyway |
| ≈ Performance | markdown-it, likely | Architectural reasoning, not measured — and at 1,467 pages Shiki and I/O may dominate. A timing/memory measurement is required in the packaging slice |
| ≈ TypeScript | unified, modestly | Both cores are typed; third-party markdown-it plugin typing varies |

Both projects are healthy. This is an architecture decision, not a maintenance-vitality one.

## The packaging gate — the one thing that could sink this

unified's packages are **ESM-only**. The current npm library builder does not disable dnt's CJS output (`scripts/build-npm-lib.ts:58`), so unified would not survive an unchanged build as external `require()` dependencies.

Crucially, **this problem already exists** and is merely masked: the generated CJS page module already contains `require("shiki")`, and Shiki is ESM-only too. It is invisible today only because `src/api/mod.ts` exports validation and versioning APIs, not page generation. Exposing page generation through the library would hit this with either library choice.

Recommended: `scriptModule: false`, `frozenLockfile: true`, publish ESM for Node 20, and add a Node smoke test that actually renders. "Leave the builder unchanged" is not viable for unified — and is already dubious for exported page rendering.

**This needs a ruling before implementation** (see Open Issues): may `weave-lib` become ESM-only for Node 20?

## What the pipeline would look like

Two passes, because wikilink resolution cannot be correct during an isolated one-file render — the target index and publish state must already be known.

```
Pass 1: DigitalArtifacts → classify syntax → parse frontmatter → validate identity
        → build immutable NoteIndex → reject ambiguous identities/output paths

Pass 2: Markdown ──→ mdast ─────────┐
        HTML fragment ─→ hast ──────┼→ sanitize → heading IDs/facts → Shiki → stringify
        plain text ─→ built hast ───┘
                                     ↓  CompiledContentRegion
                                     ↓  ResourcePageDocumentModel → shared shell
```

Markdown front end, fixed order: `remark-parse` → `remark-frontmatter` → `remark-gfm` → Weave's Dendron wikilink micromark/mdast extension → Weave frontmatter validation (existing `@std/yaml`) → Weave wikilink resolver against the NoteIndex → Weave fact collector → `remark-rehype({ allowDangerousHtml: false })`.

Common tail for every input: `rehype-sanitize` (checked-in schema) → `rehype-slug` (fixed namespaced prefix, per its DOM-clobbering warning) → Weave traversal collecting headings/links/text → `@shikijs/rehype/core` (one pre-created highlighter, matching the project's existing Shiki 4.0.2) → `rehype-stringify` with fixed options. Sanitize *before* slugging and highlighting.

The wikilink node stays Weave-owned:

```ts
interface DendronWikiLink extends Literal {
  type: "dendronWikiLink";
  target: string;
  alias?: string;
  heading?: string;
}
```

Resolution is a separate transformer that converts resolved nodes to ordinary mdast links and records an outbound-link fact. No third-party wiki plugin invents URLs.

unified ASTs stay a runtime implementation detail; the core boundary receives a data-only `CompiledContentRegion` (safe HTML plus frontmatter/headings/links/text), preserving the existing core/runtime split.

## Nine migration slices, each landable alone

1. Contract and fixtures — Dendron identities, wikilink forms, missing-target behavior, HTML trust, slug rules; capture current bytes plus hostile inputs. No behavior change.
2. Packaging proof — exact deps on an isolated branch, frozen lock, Deno + Node ESM builds, real render in both, timing/memory measured. Settle CJS before exposing the API.
3. Source/compiled model boundary — populate it with the *old* renderer's output first, so bytes do not move.
4. Unified Markdown compiler behind unit tests, not routing production pages yet.
5. Two-pass Dendron indexing — discovery, duplicate detection, wikilink node, resolution policy, diagnostics.
6. Authored-page cutover behind an explicit renderer version; the direct fallback consumes the same compiled result.
7. HTML and text adapters through the common tail.
8. DigitalArtifact site generation — Markdown artifacts become page content rather than raw-source panels.
9. Renderer epoch and cleanup — regenerate in scratch, review the diff, switch, then delete the regex renderer and the direct-rendering bypass.

Because the live site has no authored regions, a shell-preserving first cut could plausibly leave all 1,467 published files byte-unchanged.

## Byte-stability requirements

Neither candidate guarantees stability across upgrades, so determinism has to be a first-class renderer contract: pin every direct package exactly, frozen lock in dnt, plugin order in one module with no runtime discovery, LF normalization, **explicit code-point path sorting** (the analysis flags existing `localeCompare()` calls in `page_model_assembly.ts:142` and `page_definition.ts:640` as an avoidable ICU/locale determinism hazard worth auditing regardless), fixed serializer/sanitizer/Shiki/theme/language settings, byte snapshots for CommonMark/GFM/Dendron/HTML/text/hostile fixtures, and a same-timestamp double render compared by hash rather than DOM.

Every parser, serializer, sanitizer, slugger, or Shiki upgrade is a **renderer epoch** requiring reviewed golden diffs.

Use the existing render-provenance vocabulary — `renderedFromSourceState`, `renderedWithConfig`, `hasRendererVersion` already exist in the config ontology — rather than inventing parallel metadata.

## Open Issues — Dave rules

1. Is a Dendron note's canonical identity its dot-hierarchy filename, frontmatter `id`, an RDF identifier, or a combination? What resolves cross-vault collisions?
2. Which wikilink forms are contractual in the first release? Recommendation: target, alias, and heading only — not note refs, block refs, or tags.
3. Missing/unpublished target behavior: build failure, warning plus plain text, disabled link, stub page, or link elsewhere?
4. Are `id`/`title`/`desc` mandatory? When frontmatter conflicts with RDF metadata, which wins?
5. Is embedded HTML inside Markdown forbidden? For standalone fragments, what does the sanitizer allow beyond prose? (Scripts and event handlers stay unconditionally forbidden.)
6. Does `.txt` render as literal `<pre>` or get paragraphized?
7. Which DigitalArtifact roles/media types become navigable pages automatically? **What is the default publish state for conversation transcripts?**
8. Are extracted headings/links/text transient render facts, or materialized as RDF? If materialized, which vocabulary and provenance?
9. **May `weave-lib` become ESM-only for Node 20, or must CJS remain?** Should page generation join its public API?
10. Does byte-stability cover only official lockfile builds, or fresh third-party npm installs too? (The latter likely needs bundling or shrinkwrap.)
11. Preserve the current shell for a near-zero existing-site diff, or accept one deliberate 1,467-page regeneration?

## Provenance

Blind codex read-only evaluation, 2026-08-02, against `main` at `89023e7`. The brief withheld the 2025-11-29 decision, its reasoning, and Dave's stated lean. No files modified, no commands run against the live publication checkout. Full return retained in the session scratchpad.
