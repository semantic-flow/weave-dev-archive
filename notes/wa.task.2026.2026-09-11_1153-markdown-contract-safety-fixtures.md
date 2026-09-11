---
id: 2e5318ea-2445-4394-9c75-9b89bbbd0b61
title: Markdown Contract, Link Safety, And Baseline Fixtures
desc: 'Fail closed on active Markdown link schemes and establish the byte-stable contract/fixture baseline for the Markdown site pipeline'
created: 1789152780000
---

## Parent Plan

[[wa.plan.2026.2026-08-06_0854-markdown-site-pipeline]]

## Goals

- Prevent authored Markdown links from emitting active or unrecognized URI schemes through the current renderer.
- Preserve the existing relative, mesh-root-relative, fragment, HTTP(S), and mailto link behavior that the first pipeline release intends to support.
- Capture a reusable contract matrix and an exact-byte/hash baseline before introducing the compiled-content boundary or unified dependencies.
- Record the already-ruled Dendron v1 semantics precisely enough that the later parser/index child can implement them without reopening product decisions.

## Summary

`resolveMarkdownHref` in `src/runtime/weave/pages.ts` currently treats every syntactically valid URI scheme as acceptable. An authored `[label](javascript:...)` therefore becomes a live anchor. The issue is latent on publications without authored regions, but custom ResourcePages already accept Markdown and this plan deliberately expands authored and imported content. Close the unsafe boundary before making that path more common.

This child also establishes the before-state oracle for later migration. It does not add unified packages or implement Dendron parsing. It captures representative current output by exact bytes/hash, adds hostile link cases to the executable page-renderer suite, and records the future Dendron fixture cases alongside the contract they will test.

Likely implementation files are `src/runtime/weave/pages.ts`, `src/runtime/weave/pages_test.ts`, a small renderer-focused fixture/helper under `tests/fixtures` or `tests/support` if that improves reuse, and [[wu.resource-pages]] for the user-visible link rule. Avoid expanding `tests/integration/weave_test.ts` unless the safety boundary cannot be proven at the focused ResourcePage renderer layer.

## Discussion

The temporary legacy-renderer rule must match the future sanitizer boundary closely enough that the compiler migration does not reopen a vulnerability:

- Permit fragment references, ordinary relative paths, mesh-root-relative paths, and explicit `http:`, `https:`, and `mailto:` links.
- Reject case-insensitively all other schemes, including `javascript:`, `data:`, and `vbscript:`.
- Reject protocol-relative `//host/path` links rather than treating them as mesh-root-relative paths.
- Render a rejected explicit Markdown link as escaped label text with no `href`; do not preserve an active URL in another HTML attribute.
- Apply the same rule to inline links and reference-definition links.

The later Dendron fixture matrix must cover the ruled forms: direct target, aliased target, and target plus heading anchor; duplicate identities; a missing/unpublished target rendered disabled; per-vault identity prefixes; filename-derived designator paths; and retained frontmatter `id` aliases. Note refs, block refs, tags, and actual wikilink compilation remain out of this child's implementation.

The baseline should use a fixed generated timestamp and compare exact UTF-8 output or a SHA-256 digest derived from it. Do not introduce a broad snapshot framework merely for one page: reuse current page-renderer entry points and keep the fixture small enough that a later renderer epoch produces an intelligible diff.

## Open Issues

None. If an existing test demonstrates a relied-upon link form outside the allowlist, report it as a plan delta rather than silently widening the policy.

## Decisions

- Unsafe explicit Markdown links degrade to escaped, unlinked label text.
- `http`, `https`, and `mailto` are the only allowed explicit URI schemes in this child.
- Relative, mesh-root-relative, and fragment links retain their current resolution behavior; protocol-relative links are not mesh-root-relative.
- The safety rule is part of the durable authored-content contract and must remain true after the unified cutover.
- Dendron cases are specified and fixture-shaped here, but their parser/index implementation belongs to the later Dendron child.

## Contract Changes

- Generated HTML no longer emits anchors for explicit Markdown links using active, unrecognized, or protocol-relative schemes.
- Safe existing link forms retain their current generated hrefs.
- No RDF, CLI, filesystem layout, template contract, or public TypeScript API changes.

## Testing

- Record fail-on-old evidence for inline and reference-definition `javascript:` links.
- Cover mixed-case active schemes plus representative `data:` and `vbscript:` inputs.
- Cover escaped malicious labels and prove rejected URL text does not leak into `href`, `src`, style, or event-handler attributes.
- Cover allowed `http:`, `https:`, `mailto:`, fragment, relative, and mesh-root-relative links.
- Cover protocol-relative rejection explicitly.
- Capture one representative fixed-timestamp authored ResourcePage baseline by exact bytes or SHA-256 and prove two renders match.
- Keep the Dendron v1 fixture matrix checked by a lightweight structural test if stored as data; do not add unused fixtures.
- Run focused page-renderer tests, `deno task fmt`, and `deno task ci`.

## Non-Goals

- Adding unified, remark, rehype, micromark, or Dendron parser dependencies.
- Implementing wikilink parsing, cross-document indexing, slugging, HTML-fragment parsing, or plain-text adapters.
- Introducing `CompiledContentRegion` or changing production page routing.
- Exposing page or site generation through `src/api/mod.ts`.
- Broad ResourcePage visual or template changes.
- Regenerating published fixture branches or changing the renderer epoch.

## Implementation Plan

- [ ] Add focused fail-on-old coverage for unsafe inline and reference-definition links.
- [ ] Refactor Markdown href resolution/rendering so rejected links cannot produce anchors while safe existing paths retain their exact href behavior.
- [ ] Add the hostile/allowed scheme matrix, including protocol-relative inputs and escaping assertions.
- [ ] Add the smallest reusable Dendron v1 contract fixture matrix with a structural test.
- [ ] Capture and verify a representative fixed-timestamp authored-page byte/hash baseline.
- [ ] Document the authored Markdown link policy in [[wu.resource-pages]].
- [ ] Run focused tests, formatting, and full CI; record receipts and any plan delta.
