---
id: dde59667-d0d6-4942-9b00-443403d8c428
title: Mesh Support Page-Only Inventory Append
desc: 'Preserve carried MeshInventory bytes when only mesh-support ResourcePage facts are missing'
created: 1788203991838
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Route the page-only, non-initial-history arm of `planMeshSupportResourcePages` through the shared append planner/renderer.
- Preserve the exact carried MeshInventory while adding only mesh-support `hasResourcePage` claims and ResourcePage/LocatedFile types.
- Keep exact semantic no-op behavior and omit unchanged inventory updates from the returned plan.

## Summary

When no mesh-support artifact needs initial history creation, `planMeshSupportResourcePages` still splits the complete MeshInventory into Turtle blocks, edits each support-artifact subject, upserts each page subject, and rejoins the document. Comments, repeated blocks, unknown facts, prefixes, and trailing bytes can therefore be reordered or lost even though this arm owns only page claims.

This child is independent of both open MeshInventory rulings. It does not touch the initial-history arm, progression pointers, repository locators, integrate, or legacy extract. Its requested graph is named-node-only; carried blank-node subgraphs remain exact prefix bytes and are never submitted as requested facts.

## Discussion

Keep support-resource discovery and the `needsInitialSupportHistory` dispatch unchanged. In the false arm, verify that `_mesh`, `_mesh/_meta`, `_mesh/_inventory`, and optional `_mesh/_config` subjects exist, prepare the original MeshInventory, request each resource's page link plus the page's `ResourcePage` and `LocatedFile` types, and render through the shared proof-checked suffix path.

There is no single-valued predicate in the requested graph, so this child has no meaningful conflict class. A pre-existing different page link is an additional settled page, not a contradiction. Exact duplicates must no-op semantically; the returned `VersionPlan` must omit a byte-identical update as it does today.

## Open Issues

- The initial mesh-support history path still reconstructs the document and writes inventory-owned progression. It remains outside this child pending the existing progression-storage ruling.

## Decisions

- Migrate only the no-initial-history page-fact arm.
- Preserve current resource-page generation policy behavior: policy controls derived HTML, not settled inventory retraction.
- Request no blank nodes and accept no new locator identity policy.

## Contract Changes

- Page-only mesh-support planning preserves arbitrary carried MeshInventory bytes instead of rejoining subject blocks.
- Missing page facts append once; a semantic no-op yields no inventory update.
- Owned RDF facts, support-page discovery, and generated-page policy remain unchanged.

## Testing

- Fail-on-old exact-prefix test with comments, opaque/repeated facts, trailing whitespace, and a carried blank-node subgraph.
- Semantic-union and exact no-op tests for `_mesh`, metadata, inventory, and optional config pages.
- Missing-support-subject refusal coverage.
- Existing mesh-support planner/runtime integrations remain green.
- Run focused tests first, then `deno task fmt:check`, `deno task lint`, `deno task check`, `deno task test`, and `deno task ci` before landing.

## Non-Goals

- No initial support history, metadata progression, versioned writer, integrate, extract, KnopInventory, ontology, CLI, public API, or fixture-topology change.
- No blank-node generation/acceptance, single-valued conflict policy, OS-level append, or repair/retraction surface.

## Implementation Plan

- [ ] Record fail-on-old exact-prefix and semantic no-op evidence.
- [ ] Replace only the page-only block-mutation arm with bounded append planning.
- [ ] Retain missing-subject and generation-policy behavior.
- [ ] Update developer guidance and board the byte/no-op change for the next release notes.
- [ ] Run focused/full validation and return any plan-level sequencing delta.
