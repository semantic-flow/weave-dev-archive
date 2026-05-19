---
id: 5z3xczdod11ijqbxmkglfmt
title: 2026 05 19_0249 Integrate Working Source Binding
desc: ''
updated: 1779184306369
created: 1779184306369
---

## Goals

- Correct the earlier goal: ordinary floating working-source publication should not need a later `set source-binding` operation.
- Make `integrate` able to create working-only payload source bindings for external working files without repository ref, commit, path, or digest evidence.
- Keep exact source-state evidence separate from floating working-source bindings.
- Rewrite the branch-published first-release replay so already-integrated payloads are woven from their existing working source locators instead of re-integrated, `payload.update`d, or `prepare`d.
- Defer a true source-binding update/retarget surface until a workflow needs to change an existing payload's source locator or convert between working and exact source-state policy.

## Summary

This task started as a proposed `weave set source-binding` operation. That was too much surface for the immediate branch-published release problem.

For the default floating working-source case, `integrate` should do the source-binding work once. It should register the external working file as the payload's source locator, and later `weave`/`version` operations should read whatever bytes are currently materialized at that locator. If the source checkout is moved from one fixture/source rung to another, but the working file path stays the same relative to the publication mesh, no source-binding update is needed.

The concrete gap is narrower: the core integrate planner already supports a source binding without repository metadata, but the current CLI only exposes source-binding creation through the implementation-shaped `--source-binding-id` flag and then rejects that flag unless repository URL/ref/path are also supplied. That forces floating working bindings to carry exact-looking evidence that is not meaningful for the "follow this working file" policy.

The desired first-pass branch-published integration shape is:

```sh
weave integrate "$SOURCE_ROOT/ontology/fantasy-rules-ontology.ttl" ontology \
  --mesh-root "$PUBLICATION_ROOT" \
  --grant-source-directory "$SOURCE_ROOT"
```

That should create a Knop source registry with a deterministic internal source-binding id, `targetLocalRelativePath`, and `artifactResolutionMode_working`, without `sourceRepositoryRef`, `sourceRepositoryCommit`, `sourceRepositoryPath`, `expectsContentDigest`, or repository digest evidence. The common user should not have to name the source-binding node.

Later release materialization can then run the explicit weave/version surface without a source-binding update:

```sh
weave --mesh-root "$PUBLICATION_ROOT" \
  --payload-history-segment releases \
  --payload-state-segment v0.0.2 \
  --payload-manifestation-segment ttl \
  --target designatorPath=ontology \
  --target designatorPath=shacl
```

## Discussion

### What already exists

`integrate` can create a payload source binding while leaving bytes in place. The core planner can represent a working-only binding. When source-binding metadata is supplied, it writes a Knop-owned `_sources/sources.ttl` registry with:

- `sflo:hasSourceBinding` from the `KnopSourceRegistry`.
- an `sflo:ArtifactResolutionTarget` binding.
- `sflo:hasTargetArtifact` for the payload artifact.
- `sflo:targetLocalRelativePath` for the approved working source locator.
- `sflo:hasArtifactResolutionMode sflo:artifactResolutionMode_working`.
- optional exact-state evidence only when the caller explicitly selects an exact source state.
- optional repository locator facts only when they are part of an exact or otherwise deliberate repository-source contract.

`payload.update` also exists, but it is a local convenience mutation of existing working payload bytes. It resolves the current working file for a payload and replaces those bytes. It does not update Knop source registries, repository evidence, current inventories, histories, or ResourcePages.

`weave set extraction-source` exists, but it is scoped to `sflo:ExtractionSource` bindings for extracted term Knops. It proves the "set an existing source binding" pattern, but it is not the payload-source registry update we need here.

### Why this is `integrate` for new payloads

`integrate` is a creation operation for a new payload designator surface. It intentionally fails if the mesh inventory already registers that payload, Knop, or working file. That behavior is correct for first integration: it prevents accidental replacement of an existing governed payload surface.

For `03`, `04`, and `09` in the branch-published fixture, the payload designators are new. Those rungs should use `integrate` and should be able to record a working-only source binding. We should not use repository ref/commit/digest evidence merely to make the CLI accept a source binding.

For `11-first-release-woven`, the payload designators already exist. That rung should not run `integrate` again. It should materialize the source checkout at the desired source rung and then run `weave`/`version` against the existing working locators.

### Why this is not a `set source-binding` command yet

If the source locator is stable, there is nothing to update. The payload's Knop inventory and source registry already point to the external working file. The source worktree can contain different bytes over time; that is exactly what `artifactResolutionMode_working` means.

A future update surface still makes sense for different problems:

- changing the source locator for an already-integrated payload.
- converting a floating working binding into an exact source-state binding.
- clearing legacy evidence from old fixtures after a one-time model correction.
- migrating source bindings after a repository layout change.

Those are later retarget/migration operations. They should not be required by the ordinary release path.

### Why this is not `payload.update`

For a branch-published or sidecar external-source payload, the working source file is not necessarily inside the publication mesh. `payload.update` was designed around replacing bytes at the payload's existing mesh-resolved working file, and its current shape deliberately avoids source-provenance edits.

In the branch-published first-release case, the source bytes live in the source checkout. We do not want to copy them into the publication mesh. We also do not want `payload.update` to pretend the external source file is an internal mesh working file. The right model is: the payload is already integrated, its working locator points outside the publication mesh under local policy, and `weave` reads that working file.

### Why this is not `import`

`import` is the copy boundary. It brings bytes into the mesh or publication tree so the copy becomes governed local working content. The branch-published ontology and SHACL sources should not be copied into the publication branch just to make provenance refresh easier.

### Why not use the working file to specify the source?

We should. For this workflow, the working file is the source.

The current payload's working bytes are resolved from the payload/Knop inventory, and the source binding records an operational working-source locator with `sflo:targetLocalRelativePath`. In a branch-published fixture that locator is something like `../source/ontology/fantasy-rules-ontology.ttl`, resolved under local path policy. That is the working file.

That means the immediate work is to make initial `integrate` accurately record the working source contract:

- The source path is recorded as `targetLocalRelativePath`.
- The binding declares `artifactResolutionMode_working`.
- The binding id is generated deterministically by Weave, such as the existing internal default `payload-source`, unless a future multi-binding workflow needs user-directed names.
- Exact evidence is omitted unless the caller explicitly selects an exact source state.
- Repository ref/commit/digest evidence is not used for default floating working bindings.

The source registry should not silently rewrite those facts during `weave`, because `weave` should not fetch/copy/prepare/update provenance as a hidden side effect.

We should not model the external source file itself as `sflo:hasWorkingLocatedFile` of the Knop source registry. `hasWorkingLocatedFile` identifies the working file for a governed artifact such as the payload artifact, KnopInventory, or source-registry document. The external source locator in a source binding is an `ArtifactResolutionTarget`; `targetLocalRelativePath` plus `artifactResolutionMode_working` is the right model for "follow this approved working source file."

### Floating working vs exact source state

The default source binding for branch-published release automation should be floating working:

- it follows the file in the currently materialized source checkout.
- it does not persist `sourceRepositoryRef`, `sourceRepositoryCommit`, `expectsContentDigest`, or repository-locator `hasContentDigest`.
- it may be accompanied by runtime audit logs that report the observed source commit/digest, but those audit facts are not the durable source-resolution contract.

Exact evidence should be a separate explicit mode built around exact state selection:

- the caller pins to an exact source `HistoricalState` or manifestation, such as a future `--source-state` form.
- commit/digest evidence can support that exact state if the ontology/API needs it.
- digest mismatch should fail closed when explicit exact-state evidence is supplied.
- exact evidence should not be silently generated by the ordinary floating working update path.

### Later update surface

A later version may add a `set source-binding` or `source-binding update` command for changing the working locator itself. That broader mode should update all coupled current-state facts together:

- the payload's current working locator in the Knop inventory.
- any matching mesh-inventory working-file registration.
- the source binding's `targetLocalRelativePath`.
- local path policy grants when needed.
- dependent no-op/conflict reporting.

That is a bigger mutation than the first-release fixture needs, so it stays out of this task's immediate implementation slice.

## Open Issues

- Should common users ever provide a source-binding id? Recommendation: no. A source-binding id is just the local fragment id of the `ArtifactResolutionTarget` in the Knop source registry; Weave should generate a deterministic id for the single-source case.
- Should there be a user-facing flag such as `--record-source-binding`? Recommendation: avoid it for the branch-published working-source case. External working-source integration already needs the binding, so the command can infer it from source topology and local path policy.
- Should `integrate` automatically create a working-only source registry for any external source path? Recommendation: yes when the source is outside the mesh root or requires `--grant-source-directory`; that is the case where the source contract is otherwise easiest to lose. Keep mesh-local auto-binding as a separate decision if it would churn existing simple fixtures.
- Should an advanced source-binding id option exist later? Recommendation: only if multiple source bindings for one payload become real, or if migration tooling needs to preserve an existing id.
- Should exact source-state support be part of this task? Recommendation: no. This task is only for floating working source binding creation and branch replay correction.
- Should legacy repository evidence be removed from existing generated ladders now? Recommendation: defer to full fixture regeneration after the CLI and manifests are settled.

## Decisions

- Existing `weave set extraction-source` is not the general payload-source binding update surface; it is extraction-specific.
- `integrate` remains creation-only for new payload source bindings and should continue to reject already-integrated payloads.
- `integrate` should create a working-only source binding without repository metadata for external working sources that need local path policy.
- The common user should not provide a source-binding id; Weave should generate a stable internal id for the single-source case.
- The external source path supplied to `integrate` is the working source file. It is represented in the source binding as `targetLocalRelativePath` under working resolution mode, not as the source registry artifact's own `hasWorkingLocatedFile`.
- Floating working-source bindings should not persist ref/commit/digest constraints by default.
- Ref/commit/digest evidence is for exact source-state policy or runtime audit observation, not for the default "keep up with latest working file" policy.
- A later `set source-binding` command may be useful, but it is not needed for the ordinary branch first-release path when the working locator is stable.

## Contract Changes

- Update `integrate` CLI/runtime contract so external working-source integration creates a working-only source binding without repository metadata or a user-provided source-binding id.
- Update `integrate` help text so repository metadata is optional and only meaningful when deliberately selected.
- Remove or hide `--source-binding-id` from ordinary user-facing docs unless a later multi-binding workflow needs it.
- Update source-binding vocabulary/SHACL or implementation guidance so floating working source bindings are not forced to carry repository ref/commit/digest facts.
- Defer exact source-state binding support to a later task or to existing exact extraction-source behavior; it is not required for branch-published working-source integration.
- Keep [[sf.api]] focused on `integrate` for leaving source bytes in place; do not add a payload source-binding update surface yet.
- Update [[wu.cli-reference]] with syntax and examples once implemented.
- Update branch-published conformance replay for first-time integrations to use working-only source bindings.
- Update branch-published conformance replay for `11-first-release-woven` to remove `payload.update` and `prepare gh-pages`, then weave/version from the already-integrated working locators.

## Testing

- Unit-test `resolveIntegrateSourceBindingOptions`:
  - external working-source integration returns a source binding request with a deterministic internal id.
  - repository metadata remains accepted when URL/ref/path are all supplied.
  - partial repository metadata fails with a clear error.
  - `--source-digest` without exact or repository metadata is rejected or deferred.
- Core/runtime tests:
  - working-only `integrate` writes `_sources/sources.ttl`.
  - the source binding includes `targetLocalRelativePath` and `artifactResolutionMode_working`.
  - the source binding omits repository ref/commit/path/digest evidence.
  - grant behavior continues to work for external source checkouts.
- Fixture/conformance tests:
  - update branch `03`, `04`, and `09` manifests to use working-only source bindings.
  - replace the remaining `prepare gh-pages` and `payload.update` steps in branch `11-first-release-woven` with ordinary weave/version commands against existing working locators.
  - rerun focused fixture-ladder tests, then regenerate ladders in the separate regeneration pass.

## Non-Goals

- Do not make `weave` update source provenance implicitly while weaving.
- Do not make `integrate` mutate already-integrated payload surfaces.
- Do not copy source bytes into the publication mesh; that is `import`.
- Do not add network fetching or remote repository checkout behavior in this task.
- Do not implement exact source-state pinning for payload source bindings in this task.
- Do not implement a general payload source-binding retarget/update command in the first slice.
- Do not solve manifest-driven integrate here.
- Do not implement broad working-locator migration in the first slice.

## Implementation Result

Implemented on 2026-05-19 across `weave` and `semantic-flow-framework`.

- `weave integrate` now infers a working-only payload source binding for external working sources when the source path resolves outside the publication mesh under an approved source-directory grant.
- The single-source binding uses Weave's deterministic internal id, `payload-source`; ordinary users no longer provide `--source-binding-id`.
- Floating working bindings record `targetLocalRelativePath` and `artifactResolutionMode_working`, and omit repository ref, commit, path, digest, and `expectsContentDigest` evidence.
- Repository-backed source metadata remains supported only when callers provide a complete repository locator.
- `--source-digest` without repository-backed source metadata is rejected, because digest evidence is not part of the default floating working-source contract.
- Branch fixture manifests for `03`, `04`, and `09` now integrate with working-only source bindings and no source repository args.
- `11-first-release-woven` replay now weaves from the already-integrated working locators instead of using `payload.update`, `prepare gh-pages`, or a source-binding update.
- Focused CLI/runtime/integration tests, focused fixture tests, `deno task lint`, and `deno task check` passed.

Full fixture ladder regeneration has since been completed.

## Implementation Plan

- [x] Update `integrate` CLI option help and validation so external working-source integration creates a working-only source binding request without `--source-binding-id`.
- [x] Preserve repository metadata support only when the caller supplies a complete repository locator, and do not auto-compute digest evidence for floating working bindings.
- [x] Add focused CLI/runtime tests for working-only source binding creation.
- [x] Update branch `03`, `04`, and `09` manifests/tests to omit source repository ref/commit/path/digest from floating working bindings.
- [x] Update `11-first-release-woven.jsonld` replay commands to drop `payload.update` and `prepare gh-pages`, then run the intended weave/version commands from the already-integrated working locators.
- [x] Run focused fixture-ladder tests.
- [x] Regenerate the fixture ladders in the separate regeneration pass.
