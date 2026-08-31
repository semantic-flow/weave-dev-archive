---
id: 4bcd02e4-d349-4657-8475-e4a8e1d11f03
title: Current-Only PageDefinition Inventory Append
desc: 'Preserve carried KnopInventory bytes while adding current-only ResourcePageDefinition page facts'
created: 1788200820000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Route `renderCurrentOnlyPageDefinitionWovenKnopInventoryTurtle` through the shared append planner.
- Preserve the exact carried KnopInventory while appending only the ResourcePageDefinition page claim and page type facts.
- Return exact input bytes for semantic no-op and retain the existing working-file precondition.

## Summary

The current-only PageDefinition renderer still splits Turtle blocks, locates the PageDefinition subject by serialization, checks the expected working file with substring matching, replaces that subject block to add `hasResourcePage`, and upserts the page subject. Unknown PageDefinition facts, comments, repeated blocks, and trailing bytes can be reordered or lost.

This is independent of both blocked MeshInventory questions: it adds no progression facts, repository locators, or blank-node requests. Existing carried blank-node subgraphs remain untouched prefix bytes.

## Discussion

Prepare the current KnopInventory once, verify the expected `sflo:hasWorkingLocatedFile` fact semantically, and request only two fact groups: `D/_knop/_page hasResourcePage D/_knop/_page/index.html` and page `rdf:type ResourcePage, LocatedFile`. Use `planInventoryAppend` / `renderInventoryAppendPlan` for append/no-op output.

## Open Issues

- None. The helper owns only named-node multi-valued page/type facts.

## Decisions

- Replace substring/block validation with RDF-fact validation without widening accepted working-file paths.
- Emit only named-node requested facts and preserve exact current bytes.
- Keep current-only PageDefinition routing, page generation, and public/runtime result paths unchanged.

## Contract Changes

- Current-only PageDefinition weave preserves arbitrary carried KnopInventory bytes and facts instead of replacing the PageDefinition block.
- Exact semantic duplicates return the exact current bytes; missing page facts append once.
- Malformed/mismatched working-file facts remain fail-closed, now through semantic validation.

## Testing

- Fail-on-old exact-prefix test with PageDefinition comments, opaque/repeated facts, trailing whitespace, and a carried blank-node subgraph.
- Fail-on-old exact no-op test requiring byte identity.
- Working-file mismatch test and exact semantic-union test.
- Existing current-only PageDefinition core/runtime integration coverage remains green and history-free.
- Run focused tests first, then `deno task fmt:check`, `deno task lint`, `deno task check`, `deno task test`, and `deno task ci` before landing.

## Non-Goals

- No versioned PageDefinition, MeshInventory, payload, extract, integrate, source-locator, progression-storage, ontology, CLI, or public API change.
- No new blank-node generation/acceptance, OS-level append, or repair/retraction surface.

## Implementation Plan

- [ ] Record exact-prefix, no-op, mismatch, and semantic-union tests failing on current `main` before production edits.
- [ ] Replace substring/block mutation with semantic precondition plus bounded append planning.
- [ ] Retain current-only PageDefinition integration behavior and absence of history writes.
- [ ] Update developer guidance and board the byte/no-op change for the next release notes.
- [ ] Run focused/full validation and return any plan-level sequencing delta.
