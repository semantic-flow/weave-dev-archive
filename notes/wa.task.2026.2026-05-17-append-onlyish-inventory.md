---
id: gx515t52uvyddqh14agkrhu
title: 2026 05 17 Append Onlyish Inventory
desc: ''
updated: 1779080314278
created: 1779079677519
---

## Goals

- Make inventory writes append-onlyish: normal operations append new settled facts, no-op when those facts already exist, and fail closed on conflicting settled facts.
- Stop treating "graph-preserving rewrite" as the target abstraction. A rewrite that preserves more triples is still a rewrite; inventory mutation should be fact-level append/no-op/fail unless an explicit repair, regeneration, or retraction mode is active.
- Keep current/progression pointers out of inventory working files. Inventory should describe settled membership and artifact structure; metadata/progression surfaces should say what is current, latest, or next.
- Make automated release reruns boring: `weave`, `weave version`, `weave generate`, and `weave prepare gh-pages` should not churn inventory files when source bytes, config, target history, and generated-page policy inputs have not changed.
- Preserve dereferenceability and local static-host usefulness without making generated ResourcePage facts a reason to rewrite settled inventory.

## Summary

Inventory is a ledger, not a freshly rendered report.

The current implementation is already halfway there for MeshInventory: `_mesh/_inventory/inventory.ttl` keeps stable history and state membership while `_mesh/_meta/meta.ttl` owns MeshInventory current/latest/next progression. The remaining work is to make that the general rule for MeshInventory, KnopInventory, payload histories, support-artifact histories, source-registry links, reference-catalog links, and generated ResourcePage facts.

The desired write primitive is:

- append missing settled triples or subject blocks
- leave existing matching triples byte-stable
- reject conflicting settled triples unless an explicit repair/regeneration/retraction mode was requested
- never silently remove unknown or older facts from inventory

This task supersedes the older TODO wording about "subject-level canonical rewrites with graph-preserving updates". The replacement principle is simpler and stricter: normal inventory operations do not rewrite existing inventory facts at all.

## Discussion

### Settled Inventory Facts

Inventory may append facts such as:

- a mesh has a Knop
- a Knop has support artifacts
- a Knop has a payload artifact
- an artifact has an ArtifactHistory
- an ArtifactHistory has a HistoricalState
- a HistoricalState has an ArtifactManifestation
- a state or manifestation has a LocatedFile
- a support artifact has a working LocatedFile
- a Knop inventory links a source registry, reference catalog, extraction-source pointer, or generated ResourcePage once that support surface exists

These are additive facts. Once the mesh has published them, rerunning the same operation should not restate the whole subject in a canonical order, remove unfamiliar predicates, or "clean up" older blocks.

### Current And Progression Facts

Facts such as `sflo:currentArtifactHistory`, `sflo:latestHistoricalState`, `sflo:nextHistoryOrdinal`, consumed next-state hints, and any current-selection convenience facts are mutable progression facts. They should be written to metadata/progression documents, not to inventory working files.

This is already the documented direction for MeshInventory: `_mesh/_inventory` keeps stable artifact-history and historical-state membership while `_mesh/_meta` advances `sflo:latestHistoricalState`, `sflo:nextStateOrdinal`, and related progression facts. Finish that split and apply the same shape to Knop-owned inventory and payload/support artifact histories.

The RDF subject can still be the artifact or history IRI. The important distinction is the graph/document home: settled membership facts live in inventory; mutable progression facts live in meta.

### ResourcePages

Generated ResourcePage files are derived outputs, but the fact that a ResourcePage exists for a resource can become a settled inventory fact once the page is materialized or promised by policy. ResourcePage policy should control whether new page facts are appended and whether page bytes are generated. It should not remove already-settled page facts from inventory during ordinary runs.

If renderer behavior, page config, or publication policy changes and historical pages need to be rebuilt, that is a regeneration mode. Regeneration may rewrite generated HTML bytes and may repair stale page facts if explicitly requested, but it should not be the default inventory mutation path for `weave` or `prepare gh-pages`.

### Retractions And Repairs

Append-onlyish does not mean "never correct anything". It means correction is explicit.

Legitimate non-append modes include:

- privacy or security retraction
- repair of invalid previous output
- deliberate regeneration after a config or ontology-contract change
- retargeting an ArtifactHistory or source binding with user intent

Those modes should be named, audited, and tested separately. Ordinary mesh growth should not smuggle repair behavior through a canonical renderer.

### Release Automation Consequence

For CI/CD, rerunning publication should be safe because the command either sees no new facts, appends genuinely new release/source facts, or fails before writes. A repeated release action against the same source bytes and the same `releases/vX.Y.Z` target should not touch inventory. A new version should append the new HistoricalState membership and move current/latest pointers in meta, not rewrite the old inventory blocks.

## Open Issues

- Which exact metadata document owns current/progression facts for Knop-owned payload and support histories? The likely target is `D/_knop/_meta/meta.ttl` for Knop-local artifact progression and `_mesh/_meta/meta.ttl` for MeshInventory progression.
- Should the ontology or config vocabulary name an explicit inventory write policy such as append-only/current-projection/repair, or is this initially a Weave runtime invariant?
- How should repair/retraction be exposed: CLI flags on existing commands, a separate `weave repair` surface, or an internal mode first?
- Should existing pre-v1 fixtures and branch-published meshes be regenerated into the new shape, or should the first implementation carry a one-time migration reader? Preference: regenerate fixtures and fail closed on stale published shapes unless a repair mode is invoked.
- Are generated ResourcePage facts always settled once present, or do some page promises belong in metadata until the page is actually materialized?

## Decisions

- Normal inventory operations append facts, no-op duplicate facts, and fail closed on conflicting facts.
- "Graph-preserving update" is not the desired endpoint; it is still too permissive because it allows silently replacing known subject blocks.
- Current/progression predicates belong in metadata/progression documents, even when their RDF subjects are inventory artifacts, payload artifacts, or ArtifactHistory resources.
- Inventory history/state membership facts are settled facts and may remain in inventory.
- Generated ResourcePage byte regeneration is a separate concern from inventory append semantics.
- `releases` is the preferred named ArtifactHistory segment for release histories. Singular `release` examples and tests should be corrected rather than propagated.

## Contract Changes

- `weave version` must not rewrite an inventory file when it has no new settled facts to append.
- `weave` must not rewrite an inventory file merely because the renderer can produce a prettier or more complete canonical block.
- `weave generate` must not remove existing inventory ResourcePage facts as a side effect of page generation policy; it should append new page facts only when the policy says the page surface is materialized or promised.
- `weave prepare gh-pages` must be idempotent for unchanged source bytes, source metadata, target designator, target `releases/<version>` state, and config.
- A requested append that contradicts a single-valued settled fact must fail before writes with an error that names the existing fact and the requested fact.
- Historical state files and historical inventory snapshots remain immutable once written.
- Metadata files may update mutable progression facts during ordinary runs, but those updates should be narrowly scoped to the predicates the operation owns.

## Testing

- Add unit coverage for an inventory append planner: duplicate triples no-op, new triples append, conflicting settled triples fail, unknown existing triples are preserved byte-for-byte.
- Add tests that normal `weave` over an already-settled workspace produces no inventory writes when source and config are unchanged.
- Add tests that `prepare gh-pages` rerun over the same source commit leaves publication inventory files unchanged.
- Add tests that a new release state appends `sflo:hasHistoricalState <D/releases/vX.Y.Z>` to inventory while writing `sflo:latestHistoricalState` and next-state progression to metadata.
- Add guardrail tests that newly generated inventory files do not contain mutable progression predicates such as `sflo:currentArtifactHistory`, `sflo:latestHistoricalState`, `sflo:nextHistoryOrdinal`, or `sflo:nextStateOrdinal`.
- Add regression coverage for the source-registry/reference-catalog case: adding `_sources` must not drop `_references`, and neither path should rewrite unrelated inventory facts.
- Update fixture expectations after the inventory/meta split; do not add long-lived compatibility shims for stale pre-v1 fixture shapes.

## Non-Goals

- Do not design a full merge/conflict-resolution language for arbitrary RDF graphs in this task.
- Do not make Turtle formatting canonicalization part of normal inventory writes.
- Do not promise immutable generated HTML bytes; ResourcePage regeneration has its own policy surface.
- Do not add backward-compatibility shims for old pre-v1 inventory shapes unless an implementation slice proves a very narrow temporary reader is unavoidable.
- Do not solve privacy/security retraction UX beyond reserving explicit repair/retraction modes.

## Required Code Changes

- Introduce a shared inventory append planner/writer, likely under `src/core/weave` or a small RDF utility module, that parses current inventory with the RDF parser, compares requested triples semantically, and produces append/no-op/conflict results.
- Replace inventory mutation paths that call subject-block replacement or canonical re-rendering with the append planner. Primary targets include MeshInventory update helpers, KnopInventory weave renderers, source-registry insertion in `src/runtime/deploy/gh_pages.ts`, and support-artifact preservation helpers around `renderKnopInventoryWithPreservedSupportArtifacts`.
- Move remaining current/progression writes out of inventory renderers and into metadata writers. MeshInventory is already partially split; KnopInventory, payload histories, ReferenceCatalog histories, ResourcePageDefinition histories, and source-registry-related progression need the same treatment.
- Update runtime readers so current/latest resolution reads metadata/progression graphs rather than assuming the current pointers live in inventory. The reader should reject conflicting metadata/inventory current-pointer facts rather than picking one silently.
- Change ResourcePage policy filtering so it prevents appending disallowed new page facts and controls generated bytes, but does not remove already-settled inventory facts during ordinary generation.
- Update `prepare gh-pages` source-registry handling so linking `_knop/_sources` from Knop inventory is append-only and idempotent, while source registry detail replacement/retargeting remains outside inventory.
- Correct CLI/test examples that use `--payload-history-segment release` for ArtifactHistory IRIs to `--payload-history-segment releases`.

## Required Documentation Changes

- Replace the stale TODO entry about graph-preserving subject rewrites with this append/no-op/fail-closed task.
- Update [[wd.codebase-overview]] after implementation to describe inventory as settled additive membership and metadata as the current/progression graph.
- Add a [[wd.decision-log]] entry before closing the task that records the append-onlyish inventory contract and the explicit exception modes.
- Update [[wu.cli-reference]] to use `releases` consistently for release ArtifactHistory examples, especially `prepare gh-pages`.
- Update or add user-facing release automation guidance that says automated reruns should use `prepare gh-pages`, stable source refs/commits, and explicit `releases/<version>` targets, and that unchanged reruns must not rewrite inventory.
- If the storage split becomes part of the portable Semantic Flow contract rather than only Weave runtime behavior, update the relevant SFLO ontology summary/spec notes to clarify settled inventory facts versus metadata-hosted progression facts.

## Implementation Plan

- [ ] Add focused tests for append/no-op/conflict inventory behavior before refactoring production writers.
- [ ] Implement the shared RDF-aware inventory append planner.
- [ ] Refactor MeshInventory writers to use append-only settled-fact writes and metadata-hosted progression only.
- [ ] Refactor KnopInventory and payload/support history writers to append settled facts and move current/latest/next progression to metadata.
- [ ] Refactor `prepare gh-pages` source-registry inventory updates to use the append planner.
- [ ] Adjust ResourcePage fact handling so normal generation never removes settled inventory facts.
- [ ] Update runtime current/latest readers to resolve from metadata and fail closed on conflicting current-pointer facts.
- [ ] Regenerate or update fixture expectations for the new inventory/meta split.
- [ ] Correct singular `release` examples/tests to `releases`.
- [ ] Update developer/user documentation and decision log.
- [ ] Run targeted tests, then `deno task fmt`, `deno task lint`, and the relevant CI gate before closing.
