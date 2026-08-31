---
id: 1003d33f-4d51-4cab-b62f-e3a74b32d46e
title: Current-Shape Extract MeshInventory Append
desc: 'Route normal extract MeshInventory registration through the shared append planner'
created: 1788202020000
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Replace `appendExtractMeshInventoryTurtle` string concatenation with the shared append planner/renderer.
- Preserve exact carried MeshInventory bytes while registering the extracted Knop and inventory file.
- No-op duplicate requested facts at the helper boundary and reject contradictory working-inventory locators before writes.

## Summary

Normal current-shape extract already appends three named-node fact groups, but it trims the current file and concatenates Turtle without semantic duplicate/conflict handling: `_mesh hasKnop`, target Knop type/working-inventory locator, and inventory-file types. The separate `renderLegacyExtractMeshInventoryTurtle` reconstructs a complete historical fixture shape and is outside this child.

## Discussion

Keep `assertMeshInventorySupportsExtract` and legacy-shape classification unchanged. Only replace the non-legacy append arm. Prepare the original MeshInventory, request the same three fact groups, treat `hasWorkingKnopInventoryFile` as single-valued, and render through the shared proof-checked suffix path.

The requested graph is named-node-only. Existing blank-node data remains untouched prefix bytes; no blank node is generated or accepted in requested facts.

## Open Issues

- The legacy extract renderer remains whole-document reconstruction and may share the first-Knop progression-migration problem. Report exact evidence; do not broaden this child into legacy fixture migration.

## Decisions

- Preserve existing current-versus-legacy dispatch and all extract lifecycle/source semantics.
- Migrate only the current-shape append arm and use the shared planner's single-valued conflict diagnostics.
- Keep public/runtime result paths and created Knop/source-registry artifacts unchanged.

## Contract Changes

- Current-shape extract preserves exact MeshInventory bytes and trailing whitespace instead of trimming before append.
- Semantic duplicate requests return exact input at the helper boundary; contradictory working-inventory locators refuse with requested/existing facts named.
- Owned RDF facts and public extract results remain unchanged.

## Testing

- Fail-on-old exact-prefix test with comments, opaque facts, trailing whitespace, and a carried blank-node subgraph.
- Fail-on-old semantic no-op and contradictory locator tests.
- Exact semantic-union test for the three owned fact groups.
- Runtime zero-write conflict coverage and existing root/nested/floating/extract-all-terms integrations.
- Run focused tests first, then `deno task fmt:check`, `deno task lint`, `deno task check`, `deno task test`, and `deno task ci` before landing.

## Non-Goals

- No legacy extract renderer, KnopInventory/source-registry, extraction lifecycle, `hasResourcePage`, integrate, repository-locator, progression-storage, ontology, CLI, or public API change.
- No new blank-node generation/acceptance, OS-level append, or repair/retraction surface.

## Implementation Plan

- [ ] Record exact-prefix, no-op, conflict, and semantic-union tests failing on current `main` before production edits.
- [ ] Replace only the current-shape string append with bounded append planning.
- [ ] Add runtime zero-write conflict coverage and retain existing extract integrations.
- [ ] Update developer guidance and board the byte/no-op change for the next release notes.
- [ ] Run focused/full validation and return any plan-level sequencing delta.
