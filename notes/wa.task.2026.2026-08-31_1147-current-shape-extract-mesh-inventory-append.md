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

Implemented in Weave commit `6b48c73`. The current-shape arm now prepares the original MeshInventory and routes those same three named-node fact groups through the shared append planner/renderer. Existing bytes—including comments, opaque facts, trailing whitespace, and carried blank-node source-locator data—remain the exact output prefix. Semantic duplicates no-op at the helper boundary, and a contradictory target Knop inventory locator refuses before runtime writes. Legacy-shape dispatch and rendering are unchanged.

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

Fail-on-old receipt: the exact-prefix and contradictory-locator tests produced 0 passed / 2 failed against unchanged production code. The legacy concatenator trimmed trailing bytes and did not detect the existing contradictory single-valued locator.

Implementation receipts at `6b48c73`:

- direct current-shape MeshInventory tests: 3 passed / 0 failed;
- existing floating-repository semantic-union test: 1 passed / 0 failed;
- extracted-weave unrelated-block preservation test: 1 passed / 0 failed;
- Bob and sidecar extract integrations: 2 passed / 0 failed;
- runtime conflicting-locator zero-write test: 1 passed / 0 failed, with byte-identical MeshInventory and no target directory;
- `deno task fmt:check`, `deno task lint`, and `deno task check`: green;
- `deno task test`: 909 passed / 0 failed;
- `deno task ci`: 909 passed / 0 failed, LCOV generated; only the known deleted-temporary-source coverage notices appeared;
- `git diff --check`: green.

The migrated output is intentionally not byte-identical to the old appended suffix: the shared renderer emits a self-contained compact suffix. Tests require exact carried-prefix bytes and exact RDF graph equality with the settled fixture, while retaining byte-for-byte fixture assertions for every unaffected file. This byte/no-op change is boarded here for the next release notes because no next-release stub exists yet.

## Non-Goals

- No legacy extract renderer, KnopInventory/source-registry, extraction lifecycle, `hasResourcePage`, integrate, repository-locator, progression-storage, ontology, CLI, or public API change.
- No new blank-node generation/acceptance, OS-level append, or repair/retraction surface.

## Implementation Plan

- [x] Record fail-on-old exact-prefix and conflict evidence, then add no-op and semantic-union coverage.
- [x] Replace only the current-shape string append with bounded append planning.
- [x] Add runtime zero-write conflict coverage and retain existing extract integrations.
- [x] Update developer guidance and board the byte/no-op change here for the next release notes; no next-release stub exists yet.
- [x] Run focused/full validation. Plan-level sequencing is unchanged: legacy extract remains a later progression-migration concern, while the two already-open rulings still block their respective versioned writers.
